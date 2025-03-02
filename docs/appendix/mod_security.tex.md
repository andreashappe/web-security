# WAF: mod\_security

*mod\_security* ist eine Web-Application Firewall — dies bedeutet, dass
sie laut IOS/OSI Model auf Applikationsebene (Level 7) arbeitet und
daher auch die Sprache der Applikation (in diesem Fall HTTP) spricht und
versteht. Sie ist eine der bekanntesten Open-Source WAFs und wird daher
im Zuge dieses Skripts betrachtet.

Laut dem eigenen Selbstverständnis bietet sie *Real-Time Application
Security Monitoring and Access Control*: ein- und ausgehende HTTP
Anfragen und Antworten können in Echtzeit analysiert und aufgrund von
Regeln kontrolliert (z. B. blockiert) werden. *mod\_security* blockiert
by Default keinen Traffic, alle Entscheidungen müssen explizit über
Regeln konfiguriert werden. Ein häufig verwendeter Regelsatz ist de
OWASP Core Rule Set (CRS). Die WAF führt kein Input-Sanititzing durch:
ein Request wird also entweder akzeptiert oder blockiert. Dies war eine
Design-Entscheidung des Entwicklers um sich auf eine Kernaufgabe zu
konzentrieren und diese gut zu lösen (do one thing and do this well).

*mod\_security* kann entweder als Apache Modul (daher der Name=
innerhalb eines bestehenden Apache Webservers integriert werden, oder
als eigenständiger Reverse-Proxy (als eigenständiger Apache-Server) vor
mehrere Applikationsserver vorgeschalten werden.

Wie bereits erwähnt ist der primäre Einsatzzweck von *mod\_security* ist
das Monitoren und Kontrollieren von HTTP Traffic in Echtzeit und das
Erstellen von Zugriffsentscheidungen aufgrund dieses Traffics. Konkreter
kann z. B. *Virtual Patching* durchgeführt werden. Dabei wird eine
Sicherheitslücke gepatched ohne, dass die Web-Applikation selbst
angepasst wird. Beispielsweise könnte es sich bei dem Sicherheitsproblem
um einen 0day (ohne verfügbaren Patch), oder bei der Applikation um eine
ungewartete proprietäre Applikation handeln. In diesem Fall würde das
Angriffsmuster analysiert und für den Angriffsvektor eine Regel
hinterlegt werden, welche bösartige Angriffe automatisiert erkennt und
die jeweiligen Requests blockiert. Eine weitere Möglichkeit ist die
Verwendung vordefinierter Regelwerker zum Hardening bestehender
Applikationen (dadurch wird eine weitere Schutzschicht über eine
eigentlich sichere Applikation eingezogen).

Eine unterschätzte Möglichkeit von *mod\_security* ist der Einsatz als
Log-Quelle. Webserver protokollieren im Normalfall nur die zugegriffene
URL (also keinen Request-Body und auch keine Antwortdokumente). Mittels
*mod\_security* kann ein Administrator nun genau definieren, welche
Requests in welcher Tiefe protokolliert werden sollten.

## Functionality and Transactions

Grundsätzlich kann die implementierte Funktionalität in vier Bereiche
aufgeteilt werden:

-   parsing: eingehende und ausgehende HTTP Nachrichten müssen
    analysiert werden.

-   buffering: teilweise werden Nachrichten auf mehrere Teilnachrichten
    aufgeteilt. *mod\_security* sammelt diese, fügt diese zu einer
    gemeinsamen Nachricht zusammen und ruft erst danach die Filterregeln
    auf.

-   rule-engine: ist die aktive Komponente welche die bereitgestellten
    Regeln auf die gesammelten HTTP-Nachrichten anwendet.

-   logging: führt das Logging durch.

Hier kann man auch schon den Overhead von *mod\_security* erkennen: auf
der einen Seite gibt es einen CPU-Overhead da alle Nachrichten
analysiert werden müssen, auf der anderen Seite müssen für das Buffering
alle ein- und aus-gehenden Nachrichten im Speicher gesammelt werden und
führen so zu Speicher-Overhead. Falls zu viel Speicher verbraucht wird,
müssen Nachrichten auf der Festplatte/SSD zwischengespeichert werden.
Beides erhöht die Latenz-Zeit die ein Benutzer wahrnimmt.

Ein eingehender HTTP Request und die dazugehörende Response durchläuft
innerhalb von *mod\_security* folgende fünf Phasen:

1.  request headers : cheap, decide if you want to look at body content

2.  request body : think POST-Requests, attached Files

3.  response headers : cheap, decide if you want to look at body content

4.  response body, attached Files

5.  logging : decide if the request/response will be logged or not

In Phase 1 und 2 kann das Weiterleiten des Requests zum
Applikationsserver blockiert werden. Innerhalb der phase 3 und 4 kann
das Antwortdokument zum Client blockiert werden, die Operation wird
allerdings am Applikationsserver exekutiert (dies kann z. B. verwendet
werden um Data-Extraktion zu verhindern). Inder Phase 5 kann ein Request
nicht mehr blockiert werden.

### Beispielstransaktion

Anhand der Debug-Logs von *mod\_security* wird hier nochmal eine
Transaktion erklärt.

Folgender Input Request mit jeweils einem Parameter in der URL (*a*) und
einem Parameter im Body (*b*).

    POST /?a=test HTTP/1.0
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 6

    b=test

Dies führt zu folgendem Antwortdokument mit der Nachricht *Hello World!*
im Content:

    HTTP/1.1 200 OK
    Date: Sun, 17 Jan 2010 00:13:44 GMT
    Server: Apache
    Content-Length: 12
    Connection: close
    Content-Type: text/html

    Hello World!

Folgende Phasen können nun in den Log-Dateien erkannt werden:

    [4] Initialising transaction (txid SopXW38EAAE9YbLQ).
    [5] Adding request argument (QUERY_STRING): name "a", value "test"
    [4] Transaction context created (dcfg 8121800).
    [4] Starting phase REQUEST_HEADERS.

Für jeden eingehenden HTTP-Request wird zunächst eine eindeutige
Transaktions-ID vergeben. Anschließend wird der HTTP Header analysiert
und der Parameter *a* extrahiert. Abschließend werden die Regeln der
Phase 1 (Request-Header) angewandt.

    [4] Second phase starting (dcfg 8121800).
    [4] Input filter: Reading request body.
    [9] Input filter: Bucket type HEAP contains 6 bytes.
    [9] Input filter: Bucket type EOS contains 0 bytes.
    [5] Adding request argument (BODY): name "b", value "test"
    [4] Input filter: Completed receiving request body (length 6).
    [4] Starting phase REQUEST_BODY.

Falls die Nachricht noch nicht blockiert wurde, wird nun der Content des
Requests analysiert und dabei der Parameter *b* extrahiert. Jetzt
besitzt *mod\_security* Zugriff auf alle Parameter und die Regeln der
Phase 2 (Request Body) werden angewendet.

    [4] Hook insert_filter: Adding input forwarding filter (r 81d0588).
    [4] Hook insert_filter: Adding output filter (r 81d0588).

    [4] Input filter: Forwarding input: mode=0, block=0, nbytes=8192 (f 81d2228, r 81d0588).
    [4] Input filter: Forwarded 6 bytes.
    [4] Input filter: Sent EOS.
    [4] Input filter: Input forwarding complete.

Wenn auch diese Regeln den Request nicht verwerfen, wird der Request an
den Applikationsserver/die Applikation weitergeleitet. *mod\_security*
kann nun abwarten, bis ein entsprechendes Antwortdokument empfangen
wird.

    [9] Output filter: Receiving output (f 81d2258, r 81d0588).
    [4] Starting phase RESPONSE_HEADERS.

    [9] Output filter: Bucket type MMAP contains 12 bytes.
    [9] Output filter: Bucket type EOS contains 0 bytes.
    [4] Output filter: Completed receiving response body (buffered full - 12 bytes).
    [4] Starting phase RESPONSE_BODY.

Analog zu der Request-Phase wird nun das Antwortdokument analysiert und
die Regeln der jeweiligen Phasen (3 und 4) angewendet. Hier sieht man
auch schön, dass auch das Antwortdokument gespeichert wird — werden z.
B. viele große Dokumente zugestellt (oder Dateien heruntergeladen)
müssen diese zwischengespeichert, und auf diese Weise serverseitig viele
Ressourcen verbraucht, werden.

    [4] Output filter: Output forwarding complete.
    [4] Initialising logging.
    [4] Starting phase LOGGING.
    [4] Audit log: Ignoring a non-relevant request.

Abschließend wird noch die Log-Phase durchspielt und aufgrund dieser
erkannt, ob der Request bzw. die Response in die Logdatei aufgenommen
werden sollte.

## Rules

Das Herz einer *mod\_security* Installation sind die verwendeten
Filterregeln. Diese besitzen immer das idente Format:

    SecRule ARGS OPERATOR ACTIONS

Die jeweiligen Bereiche sind einfacher erklärt:

ARGS  
: beschreibt auf welchen Bereich (z. B. auf welchen Parameter) eine
jeweilige Operation angewendet werden soll.

OPERATOR  
: definiert die Abfrage/den Filter der auf die ARGS angewandt werden
soll.

ACTIONS  
: beschreibt die Aktivitäten (z. B. blockieren und loggen) die angewandt
werden, falls der Operator erfolgreich auf die Argumente angewandt
werden konnte.

### Rules: Args

Beispiele für mögliche Argumente:

-   ARGS (GET+POST arguments)

-   ARGS\_GET, ARGS\_POST: die Parameter dre Header/des Contents

-   FILES: hochgeladene Dateien

-   FULL\_REQUEST: der gesamte Request

-   QUERY\_STRING, REQUEST\_BODY: raw data

-   REQUEST\_HEADERS: Header Werte

-   REQUEST\_METHOD: die Verwendete HTTP Methode (POST/GET)

-   REQUEST\_URI

-   REQUEST\_LINE

-   REMOTE\_ADDRESS: die Adresse des zugreifenden Clients

#### Operatoren

Operatoren beginnen meistens mit einem @-Symbol, einige Beispiele:

-   "@contains &lt;script &gt;": überprüft ob in den Arguments ein
    script-Tag vorkommt

-   "@detectSQLi": überprüft die Arguments auf SQL-Injection-Muster

-   "@detectXSS": überprüft die Arguments auf XSS-Muster

-   "@inspectFile /path/to/util/runav.pl": die Arguments werden an das
    angeführte Skripts übergeben, das Ergebnis des Operators ist das
    Ergebnis des Skripts. Auf diese Weise kann z. B. jedes hochgeladene
    File mittels eines Virenscanners überprüft werden, bevor dieses File
    an den Applikationsserver übergeben wird.

-   "@ipMatch 192.168.1.100": Vergleich auf eine IP-Adresse

-   "@validateDTD /path/to/xml.dtd": überprüft ob das übergebene
    Argument einem definierten XML-Schema entspricht.

#### Actions

Actions definieren die Operationen, die exekutiert werden sollten falls
die Operatoren erfolgreich angewandt werden konnten. Diese Operatoren
können lt. *mod\_security* Dokumentation in verschiedene Bereiche
eingeteilt werden:

-   disruptive: verändern den Datenfluss, z. B. *allow* oder *deny*

-   flow: beziehen sich auf die Regeln selbst, z. B. *skip* oder *chain*

-   meta-data: dienen zur Dokumentation

-   variable: für komplexe Regeln können Variablen gesetzt bzw.
    ausgelesen werden

-   logging: Definition der Daten, die geloggt werden sollten

-   special

-   miscellaneous: everything else

#### Beispiele

Ein einfaches primitives Beispiel:

    SecRule ARGS "@contains <script>" log,deny,status:404

Diese Regeln verwendet als Datenquelle alle extrahierten Argumente
(*ARGS*) und überprüft, ob in diesen ein HTML script-Tag vorkommt. In
dem Fall wird die Anfrage mit einem 404 Fehler blockiert und geloggt.

Eine komplexere Regel:

    SecRule REQUEST\_LINE|REQUEST\_HEADERS|REQUEST\_HEADERS\_NAMES "@contains () \{" "id:420008,phase:2,t:none,t:lowercase,deny,status:500,log,msg:'Malware expert - user-agent: Bash ENV Variable Injection Attack'"

Diese Regel sucht in verschiedenen Datenquellen nach dem String *() {*.
Dieser String beschreibt einen Bash-Funktionsaufruf und sollte in
regulärem Verkehr nicht vorkommen. Falls dieser String gefunden wird,
wird der Request mit einem 500er Statuscode abgelehnt und protokolliert.

    SecRule REMOTE\_ADDR "@ipMatch 192.168.1.101" id:102,phase:1,t:none,nolog,pass,ctl:ruleEngine:off

Diese Regel würde Verkehr von der IP-Adresse 192.168.1.101 immer
akzeptieren (durch ruleEngine:off werden keine Folgeregeln mehr
verwendet), diese Requests werden ebenso nicht geloggt. Dadurch wird ein
Whitelisting der IP-Adresse durchgeführt.

    SecRule ARGS:username “@streq admin” chain,deny
    SecRule REMOTE\_ADDR “!streq 192.168.1.111”

Das letzte Beispiel zeigt, wie zwei Regeln mittels *chain* verknüpft
werden. Es wird zuerst die erste Regel angewendet (*für das Argument
username wurde der Wert admin verwendet*) und dann mit der Regel *die
IP-Adresse ist nicht 192.168.1.111* kombiniert. Falls beide Regeln wahr
sind, wird die Anfrage abgelehnt. Dies entspricht also der Regel *Ein
Administrator darf sich nur ausgehend von der IP-Adresse 192.168.1.111
einloggen*.

### Log Format

Das *mod\_security* Format entspricht nicht 100% anderen bekannten
Log-Formaten. Eine protokollierte Transaktion wird über mehrere Sublogs
beschrieben. Jedes Sublog beginnt mit einem MIME-ähnlichem Header,
dieser besteht aus einer eindeutigen ID über welchen die Sublogs
aggregiert werden können (in dem Beispiel *f9adec1d*) und einer
Log-Kategorie (dies ist der angehängte Buchstabe).

    --f9adec1d-A--
    [25/Sep/2016:18:42:50 +0300] V@fwen8AAQEAAA2TDbQAAAAK 127.0.0.1 35965 127.0.0.1 443
    --f9adec1d-B--
    GET / HTTP/1.1
    Host: malware.expert
    Accept: */*
    User-Agent: () {

    --f9adec1d-F--
    HTTP/1.1 500 Internal Server Error
    Content-Length: 613
    Connection: close
    Content-Type: text/html; charset=iso-8859-1

    --f9adec1d-E--

    --f9adec1d-H--
    Message: Access denied with code 500 (phase 2). String match "() {" at REQUEST_HEADERS:User-Agent. [file "/etc/modsecurity/malware_expert.conf"] [line "97"] [id "420008"] [msg "Malware expert - user-agent: Bash ENV Variable Injection Attack"]
    Action: Intercepted (phase 2)
    Stopwatch: 1474818170070960 1010 (- - -)
    Stopwatch2: 1474818170070960 1010; combined=177, p1=25, p2=147, p3=0, p4=0, p5=5, sr=55, sw=0, l=0, gc=0
    Response-Body-Transformed: Dechunked
    Producer: ModSecurity for Apache/2.9.0 (http://www.modsecurity.org/).
    Server: Apache
    Engine-Mode: "ENABLED"

    --f9adec1d-Z--

In Abhängigkeit der Transaktion können unterschiedliche Log-Kategorien
befüllt werden. *A* beinhaltet immer den Zeitstempel und die Adressen
der betroffenen Kommunikationspartner, *B* ist der eingehende Request,
*F* die erzeugte Antwort; *H* beschreibt die Regel die angewandt wurde.

Leider ist es nicht so einfach diese Informationen mittels *grep* zu
parsen.

## OWASP Core Ruleset

Es macht natürlich wenig Sinn, für jede *mod\_security*-Installation
getrennte Regelwerke für common Angriffsvektoren zu erstellen. Aus
diesem Grund haben sich Default-Regelwerke gebildet die als Initiale
Regeln eingespielt werden können. Das bekannteste Beispiel hierfür ist
das *OWASP Core Rule Set* (auch CRS genannt), die aktuelle Version ist
3, dieses Regelwerk wird gerne zum Hardening von Webservern verwendet.

Dieses Ruleset beinhaltet Regeln welche die Angriffsvektoren der OWASP
Top 10 (als auch weitere Angriffsvektoren) abdecken. Um dieses Regelwerk
zu aktivieren, lädt der Administrator den Regelsatz als auch ein
Konfigurations-File (*crs-setup.conf*) vom OWASP-Server herunter und
inkludiert diese innerhalb der Konfiguration des Apache Webservers.
Dabei kann der Admin entweder alle Regeln inkludieren oder nur einzelne
Subbereiche (z. B. nur Regeln für SQL-Injection Angriffe).

Ein häufiges Problem von Web Application Firewalls ist, dass valide
Anfragen als Schadcode erkannt werden (diese Fehler werden auch als
*false positives* bezeichnet). Eine hohe Rate an false positives
zerstört die Benutzerakzeptanz und führt im Normalfall dazu, das die WAF
aus Akzeptanzgründen wieder deaktiviert wird. Um dem entgegenzuwirken
besitzt das OWASP Core Rule Set eine Paranoia-Einstellung welche von 1
(default) bis 4 gesetzt werden kann. Je höher das Paranoia-Level umso
mehr Schadcodes werden erkannt, allerdings steigt damit auch das Risiko
von False-Positives. In Abhängigkeit der Gefährdung der Applikation kann
nun ein geeignetes Paranoia-Level gewählt werden.

## Reflektionsfragen

1.  Was versteht man unter einer Web-Application Firewall? Welche
    Einsatzzwecke bzw. Verwendungsmöglichkeitne gibt es für solche?

2.  Aus welchen Teilen besteht eine mod\_security Regel? Erläutere die
    jeweiligen Teile.

3.  Was wird durch folgende Regel: *SecRule ARGS "@contains
    &lt;script&gt;" log,deny,status:404* durchgeführt?

<!-- -->

1.  Was ist der Grundgedanke dabei, DevOps und Security zu verbinden?

2.  Welche Sicherheitsmaßnahmen können im Zuge von SecDevOps
    automatisiert durchgeführt werden? Erläutere die jeweiligen
    Maßnahmen.
