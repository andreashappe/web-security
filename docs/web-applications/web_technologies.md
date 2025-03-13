# Web Technologien

Der Titel dieses Dokumentes ist Web Security, dementsprechend sind
unsere Ziele/Patienten auch Webapplikationen. Eine Definition fällt
nicht einfach — allgemein betrachtet ist eine Webapplikation eine auf
der Client-Server-Architektur basierte Applikation die als
Kommunikationsprotokoll HTTP verwendet.

Bei einer Client-Server Applikation versendet der Client einen Auftrag
an einen Server; letzterer führt diesen im Namen des Clients aus und
sendet die Antwort zurück. Im Zusammenhang mit Webapplikationen gehen
wir von einem Webbrowser als Client aus. Die meisten vorgestellten
Probleme betreffen auch Web-API Clients, auf diese wird allerdings nicht
explizit eingegangen.

## HTTP

Das Hypertext Transfer Protocol (HTTP[1]) ist ein textbasiertes
Protokoll welches primär zur Kommunikation zwischen Webservern und
Web-Clients (wie z.B. Webbrowsern) verwendet wird. HTTP 1.0 wurde 1996
als RFC 1945 als expliziter Non-Standard veröffentlicht. 1999 wurde das
Protokoll mit dem Update auf HTTP 1.1 (RFC 2616) modernisiert, es wurde
z.B. HTTP Pipelining (die Übertragung mehrerer Dateien innerhalb einer
HTTP Verbindung) in den Standard aufgenommen.

2015 wurde HTTP/2 im RFC 7540/7541 definiert: Verbesserungen betreffen
das Multiplexing von Anfragen, server-seitige Push-Nachrichten und die
Kompression der übertragenen Daten. Per Stand 2020 kann davon
ausgegangen werden, dass Zwei-Drittel bis Drei-Viertel der
Webkommunikation bereits über HTTP/2 abgewickelt wird.

HTTP verwendet zumeist TCP auf Port 80, die verschlüsselte Variante
HTTPS verwendet Port 443. Häufig verwendete Ports für weitere
HTTP-basierte Services sind 3000, 8000, 8080 und 8081.

Das Protokoll basiert auf Nachrichten, die zwischen Client (Browser) und
Server übertragen werden. Dabei folgt auf den initialen Request des
Clients immer eine Response des Servers.

Aktuell wird HTTP/3 basierend auf QUIC entwickelt. Diese
Protokollversion wird vieles verändern, so wird z.B. ein Umstieg von TCP
auf UDP diskutiert, auch die Verwendung von Port 443 bleibt eventuell
nicht mehr bestehen.

### HTTP Request

Bei HTTP 1.0/1.1 werden Anfragen von Webbrowsern an Webserver als
mehrzeilige Textdokumente verschickt. Die erste Zeile dieses Dokuments
beinhaltet als erstes Wort das zu verwendete HTTP Verb gefolgt von dem
aufgerufenen Pfad und der verwendeten HTTP-Version. Jede weitere Zeile
beinhaltet einen HTTP Header, diese sind immer als *Key: Value*
strukturiert.

Bei folgendem Beispiel versucht ein Webbrowser auf die Datei
*/index.html* eines Webservers lesend (Verb: GET) zuzugreifen:

    GET /index.html HTTP/1.1
    Host: snikt.net
    User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:65.0) Gecko/20100101 Firefox/65.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Connection: close
    Upgrade-Insecure-Requests: 1

#### HTTP Request Verbs/Methoden

Ein Request beginnt immer mit einem HTTP Verb (auf Englisch auch HTTP
Request Method genannt), dieses beschreibt die Aktion die der Client
gerne hätte. Häufig verwendete Verben werden in Tabelle
<a href="#tbl:http_verbs" data-reference-type="ref"
data-reference="tbl:http_verbs">1.1</a> gelistet. Ein verwendetes HTTP
Verb kann sowohl *safe* als auch *idempotent* sein. Safe Verben sollten
niemals Resourcen verändern (also Daten am Server modifizieren).

| Verb    | safe | idempotent | Name                                                                                                                                            |
|:--------|:-----|:-----------|:------------------------------------------------------------------------------------------------------------------------------------------------|
| GET     | ja   | ja         | Beschreibt einen Lesezugriff bei dem es zu keiner Veränderung des serverseitigen States kommen sollte.                                          |
| HEAD    | ja   | ja         | Entspricht einem HTTP GET, allerdings wird kein HTTP Body übertragen. Diese Operation wird häufig verwendet um Meta-Daten zu erfragen.          |
| POST    |      |            | Ist eine datenverändernde Operation und wird verwendet um ein neues Objekt zum Server zu übertragen (also um quasi ein neues Objekt anzulegen). |
| PUT     |      | ja         | Ist eine datenverändernde Operation welche ein Objekt am Server ersetzt, also quasi aktualisiert.                                               |
| DELETE  |      | ja         | Löscht ein Objekt/Datei vom Server.                                                                                                             |
| PATCH   |      |            | Ist eine datenverändernde Operation welche einen Teil eines bestehenden Objektes modifiziert.                                                   |
| CONNECT |      |            | Wird verwendet um einen Tunnel aufzubauen.                                                                                                      |
| OPTIONS | ja   | ja         | Listet alle erlaubten Kommunikationsoptionen für eine Resource auf.                                                                             |
| TRACE   |      |            | Führt zu Debug-Zwecken einen loop-back Test aus.                                                                                                |

Häufig verwendete HTTP Methoden bzw. Verben

Idempotente Verben sollten auch bei wiederholtem Aufruf auf eine
Resource das idente Ergebnis liefern. Sie können also beliebig häufig
aufgerufen, und wiederholt werden. Wenn z.B. während eines DELETE
Aufrufs ein Timeout geschieht, kann der Client die Operation wiederholen
ohne einen undefinierten server-seitigen State zu erzeugen. Aus diesem
Grund wird z.B. ein Update eines bestehenden Datensatzes gerne über das
PUT Verb implementiert: wird ein Update mit den gleichen übergebenen
Daten ausgeführt, kann es beliebig häufig wiederholt werden und der
server-seitige State sollte ident sein.

Die Verwendung des richtigen Verbs besitzt rein semantische Natur und
muss von der Web-Applikation umgesetzt werden. Nichts hindert einen
Programmierer, eine Operation mit einem unpassenden HTTP Verb
anzubieten. Allerdings gehen mehrere Komponenten (wie z.B. Web Proxies,
Caches oder Web Application Firewalls) von der richtigen Verwendung der
jeweiligen Verben aus, wird ein falsches Verb verwendet kann dadurch
inkorrektes Verhalten provoziert werden.

#### Representational State Transfer (REST)

Das REST-Paradigma wurde von Roy Fielding 2000 im Zuge seiner
Dissertation veröffentlicht. Das Paradigma versucht es, zustandlose APIs
über eine einheitliche Schnittstelle anzubieten. Jede gespeicherte
Ressource sollte eine eindeutige URL besitzen, als Kommunikationssprache
wird häufig HTTP eingesetzt.

Tabelle <a href="#tbl:rest" data-reference-type="ref"
data-reference="tbl:rest">1.2</a> zeigt wie häufig benötige
CRUD-Funktionalität[2] auf HTTP Verben umgelegt wird.

| Verb   | Operation     | Beispiel                                    | Beschreibung                                                                          |
|:-------|:--------------|:--------------------------------------------|:--------------------------------------------------------------------------------------|
| GET    | READ          | <a href="/notes/1" class="uri">/notes/1</a> | Fordert die Ressource vom Server an. Diese Operation sollte safe und idempotent sein. |
| POST   | CREATE        | <a href="/notes" class="uri">/notes</a>     | Erstellt eine neue Ressource am Server, deren URI wird zurück gegeben.                |
| PUT    | CREATE/UPDATE | <a href="/notes/2" class="uri">/notes/2</a> | Erstellt oder ersetzt eine Ressource an der angegeben URI.                            |
| PATCH  | UPDATE        | <a href="/notes/2" class="uri">/notes/2</a> | Die angegebene Ressource wird verändert, Nebeneffekte sind erlaubt.                   |
| DELETE | DELETE        | <a href="/notes/2" class="uri">/notes/2</a> | Die angegebene Ressource wird gelöscht.                                               |
| HEAD   | READ          | <a href="/notes/2" class="uri">/notes/2</a> | Liefert Meta-Daten für die angegebene Ressource.                                      |

Verwendung von HTTP Verben bei RESTful-Architekturen

#### Request Host Header

Der übergebene *Host*-Header kann sicherheitsrelevant sein: dieser
Header wird nicht verwendet um auf der Netzwerkebene das Ziel zu
identifizieren, sondern wird erst vom Zielwebserver verwendet. Einige
Webserver verwenden diesen Header um Adressen innerhalb der Antwortseite
zu generieren.

### HTTP Response

Der Server liefert nun ein Antwortdokument:

    HTTP/1.1 302 Found
    Date: Sun, 03 Mar 2019 22:03:21 GMT
    Server: Apache/2.4.25 (Debian)
    Location: https://snikt.net/
    Content-Length: 277
    Connection: close
    Content-Type: text/html; charset=iso-8859-1

    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>302 Found</title>
    </head><body>
    <h1>Found</h1>
    <p>The document has moved <a href="https://snikt.net/">here</a>.</p>
    <hr>
    <address>Apache/2.4.25 (Debian) Server at snikt.net Port 80</address>
    </body></html>

Hier fällt zuerst der Statuscode (302) auf. Prinzipiell beschreiben
Codes aus dem 100er Bereich *Continue*, Codes im 200er Bereich Erfolg
(*success*), Code im 300er Bereich sind Redirects, Codes im 400er
Bereich sind clientseitige Fehler und Codes im 500er Bereich beschreiben
serverseitige Fehler.

Webserver können mehrere optionale HTTP Header inkludieren und auf diese
Weise dem Webbrowser Informationen mitteilen. Diese Möglichkeit wird
häufig im Zuge des Browser-Hardenings verwendet: hierbei teilt der
Webserver Securityannahmen dem Client mit. Dieser kann dadurch effizient
gegen client-seitige Angriffe innerhalb des erhaltenen Contents
vorgehen.

#### Information Disclosure durch HTTP Header

Die optionalen Header können einen negativen Sicherheitsimpact besitzen,
häufig kommt es z.B. zu einer Information Disclosure. Bei dieser erhält
der Angreifer durch gesprächige Server Informationen, die ein normaler
Benutzer eigentlich nicht benötigen sollte aber einem Angreifer
behilflich sind. Im gezeigten Antwortdokument teil der Server den
verwendeten Webserver (*Apache*), das verwendete Betriebssystem
(*Debian*) und die Versionsnummer des Webservers (*2.4.25*) über den
*Server* Header mit. Dies erlaubt es einem Angreifer, gezielt nach
Schwachstellen für diese Softwarekomponente zu suchen. Im Zuge des
Hardenings werden solche Versionsinformationen zumeist maskiert.

## Transportlevel-Sicherheit

Eine Webapplikation sollte immer und ausschließlich über das gesicherte
HTTPS-Protokoll kommunizieren. Um die Sicherheit des Transports zu
gewährleisten sollte TLS[3] eingesetzt werden, die Abkürzung TLS steht
dementsprechend auch für Transport Level Security.

### TLS

Beim Einsatz von TLS sollte eine aktuelle Version (aktuell TLSv1.2)
verwendet werden, innerhalb von TLS sollten sichere Algorithmen
(AES-256-GCM oder ChaCha20-Poly1305) bereitgestellt werden. Aktuell wird
TLSv1.2 von ca. 95-96% der Webserver angeboten. Jeder HTTP/2 kompatible
Client muss ebenso TLSv1.2 unterstützen.

Wenn möglich sollten ältere TLS-Versionen vermieden werden, da durch
diese schlechtere Kryptographie in Kauf genommen werden muss. So
schreibt der TLS-Standard vor Version 1.2 vor, dass der Cipher
*3DES-CBC* zwingend in einer Standard-konformen Implementierung
angeboten werden muss. Dieser Cipher ist zwar noch sicher, wird aber
teilweise schon als *legacy* klassifiziert — sollte also bei neuen
Implementierungen nicht mehr verwendet werden. Mit TLSv1.2 wird nicht
mehr *3DES-CBC* sondern *AES-128-CBC* als notwendiger Cipher
vorgeschrieben. Mit TLSv1.3 wurde der CBC-Modus entfernt: dies ist aus
Sicherheitssicht stark begrüßenswert, allerdings ist diese Version des
Standards noch nicht veröffentlicht.

### Welche Kanäle müssen beachtet werden?

Die Entwickler und Administratoren müssen darauf achten, dass alle
Kommunikationswege auf die gleiche Art und Weise geschützt werden. Es
muss vermieden werden, dass z.B. ein Webserver mit einer sicheren
TLS-Konfiguration konfiguriert wurde, aber die identen Operationen
mittels eines Webservices ungesichert über HTTP bereitgestellt werden.

Ein häufiger Diskussionspunkt ist, welche Verbindungen durch TLS
abgesichert und verschlüsselt werden müssen. Prinzipiell sollte jegliche
Übertragung über öffentliche Kanäle gesichert erfolgen. Der Einsatz von
Verschlüsselung innerhalb des Rechenzentrums, z.B. zwischen
Applikationsserver und Datenbanken, wird allerdings teilweise
diskutiert. Die Verwendung der Verschlüsselung bewirkt geringere
Performance, höhere Kosten und verhindert teilweise die Verwendung
anderer Sicherheitstechniken (z.B. von Network-based IDSen) — daher wird
teilweise ein Rechenzentrum als a-priori sicher angenommen und innerhalb
dessen keine Verschlüsselung erzwungen. Die jeweilige Entscheidung muss
dokumentiert und durch das Management unterzeichnet werden.

### Perfect Forward Secrecy

Perfect Forward Secrecy (PFS) ist eine optionale Eigenschaft von
Key-Exchange Protokollen und kann z.B. bei TLS zum Einsatz kommen. TLS
verwendet einen Langzeitschlüssel — während des Verbindungsaufbau wird
basierend auf diesem ein Sitzungsschlüssel ausgemacht. Zeichnet ein
Angreifer die verschlüsselte Kommunikation auf und erhält auf irgendeine
Weise den Langzeitschlüssel, kann er die Verschlüsselung aufbrechen.
Dies ist problematisch, da der Langzeitschlüssel auch Jahre nach der
eigentlich erfolgten Kommunikation verloren gehen könnte.

Bei Verwendung von PFS kann mit dem Langzeitschlüssel der
Sitzungsschlüssel nicht mehr rekonstruiert werden. Dadurch wird die
Gefahr einer späteren Offenlegung der Kommunikation durch Verlust des
Langzeitschlüssels gebannt.

### HSTS

Der *HTTP Strict Transport Security* (HSTS, RFC 6797) Header teilt dem
Webbrowser mit, dass Folgezugriffe auf die Webseite immer über ein
sicheres Protokoll zu erfolgen haben. Bei Angabe des Headers wird eine
Laufzeit in Sekunden[4] für diese Regel angegeben:

    Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

Sobald dieser Header vom Browser interpretiert wird, werden potentielle
zukünftige HTTP-Aufrufe automatisch vom Browser auf HTTPS hochgestuft.
Zusätzlich schützen Webbrowser (bei Verwendung von HSTS) Benutzer vor
unüberlegten Entscheidungen und erlauben nicht mehr das Akzeptieren von
defekten oder invaliden Zertifikaten.

HSTS kann durch zwei Optionen erweitert werden. Durch
*includeSubDomains* inkludiert Subdomains in den HSTS Schutz. Dies ist
wichtig, da ein Angreifer von einer Subdomain auf die Cookies der
Hauptdomain zugreifen kann und dadurch auf HSTS-geschützte Cookies
zugreifen könnte.

Durch das Setzen von *preload* wird der Wunsch der Webseite mitgeteilt
in Google Chrome’s HTTPS preload Liste aufgenommen zu werden[5]. Dies
ist eine Liste von Webseiten, die ausschließlich über HTTPS verfügbar
sind. Wird dieser Header gesetzt, ist die Seite effektiv über Chrome nie
wieder über HTTP erreichbar.

### Verbindungssicherheit bei WebSockets

Eine WebSocket-URL beinhaltet das zu verwendende Protokoll, dieses kann
entweder *ws* (WebSocket) oder *wss* (WebSocket Secure) sein. Aus
Sicherheitssicht sollte ausschließlich *wss* verwendet werden.

## Sessions and Cookies

Eine Session ist eine stehende Verbindung zwischen einem Client und
einem Server. Innerhalb der Session kann der Server Zugriffe einem
Client zuordnen. Eine Session wird häufig verwendet um nach erfolgten
Login am Server die darauffolgenden Operationen dem eingeloggten
Benutzer zuordnen zu können.

HTTP ist ein zustandsloses Protokoll: jeder Zugriff ist alleinstehend.
Die Session muss daher auf einer höheren Ebene implementiert werden. Im
Web-Umfeld werden zumeist Cookie-basierte Sessions verwendet, andere
Möglichkeiten wären z.B. Token-basierte Systeme.

Ein Cookie ist ein kleines Datenpaket welches im Zuge des
Session-Managements vom Server dem Client mitgeteilt wird. Der Client
speichert nun dieses Cookie und inkludiert es in jedem Folgeaufruf zu
dem setzenden Webserver. Ein Cookie besteht aus einem Namen, Wert,
Ablaufdatum und einem Gültigkeitsbereich (Domain und/oder Pfad).

Wird eine Domain für ein Cookie gesetzt, wird das Cookie für diese
Domain und alle Subdomains übertragen. Dies ist überraschend unsicherer
als keine Domain zu setzen: in diesem Fall würde das Cookie nur an die
idente Domain (nicht an die Subdomains) übertagen werden.

Eine wichtige Cookie-Option ist das Setzen eines Gültigkeitspfades. Wird
dieser gesetzt, dann wird das Cookie nur für Ressourcen übertragen,
deren Pfad “unter” diesem Pfad liegen. Auf diese Wiese können mehrere
Applikationen auf unterschiedlichen Pfaden auf einem Webserver betrieben
werden während keine Applikation auf die Cookies einer anderen
Applikation zugreifen kann.

Zusätzlich zu den Cookie-Einstellungen gibt es spezielle
sicherheitsrelevante Cookie-Flags:

### secure-Flag

Durch das *secure*-Flag wird die Übertragung des Cookies mittels HTTPS
erzwungen. Bei potentiell auftretenden HTTP-Zugriffen wird kein Cookie
übermittelt, der Request allerdings abgesendet. Dies erlaubt es dem
Webserver auf sichere Weise ein HTTP 300 Redirect von HTTP auf HTTPS
durchführen.

### httpOnly-Flag

Das *httpOnly*-Flag verbietet es Webbrowsern den Zugriff mittels
Javascript auf das Cookie. Falls das Cookie nur zur Bildung der
Benutzersession verwendet wird, kann dieses Flag durch den Webserver
gesetzt, und damit Javascript-basierte Identity Theft Angriffe stark
erschwert werden. Achtung: dieses Flag besitzt keinen Einfluss auf die
Verwendung des HTTP- oder HTTPS-Protokolls.

Problematisch ist in diesem Zusammenhang die HTTP TRACE Methode. Diese
dient zu Analysezwecken und kopiert den eingehenden Request als Content
in das Antwortdokument. Falls der Angreifer nicht mittels Javascript auf
das Session-Cookie zugreifen kann, aber die Möglichkeit besitzt per
Javascript einen HTTP TRACE Aufruf auf den Opfer-Webserver abzusetzen,
kann er auf diese Weise das Session-Cookie extrahieren:

    <script>
      var xmlhttp = new XMLHttpRequest();
      var url = 'http://127.0.0.1/';

      xmlhttp.withCredentials = true; // send cookie header
      xmlhttp.open('TRACE', url, false);
      xmlhttp.send();
    </script>

Aus diesem Grund wird empfohlen, auf Webservern immer HTTP TRACE zu
deaktivieren.

### sameSite-Flag

Das *sameSite*-Flag dient zur Vermeidung von CSRF-Angriffen[6]. Das Flag
unterrichtet den Browser, unter welchen Umständen ein Session-Cookie an
eine Webseite übertragen werden soll.

Bei Verwendung von *strict* wird niemals ein Session-Cookie im
cross-domain Kontext übertragen. Dies bedeutet, dass das Cookie nur
übertragen wird, wenn der Benutzer von der Webseite auf einen Link/eine
Operation auf der identen Webseite navigiert. Wird z.B. ein Link auf die
Webseite von einer externen Quelle angeklickt (z.B. innerhalb eines
Forums oder ein Link innerhalb einer Email), wird bei dem URL-Aufruf
kein Cookie übergeben. Hier muss beachtet werden, dass falls die
Webseite eine *Unvalidated Forward or Redirect*-Lücke besitzt, der
Angreifer diese ansteuern kann und bei dem durchgeführten zweiten Aufruf
der Browser das Cookie inkludiert und dadurch potentiell bösartige
Aktionen ausgeführt werden können.

Bei Verwendung von *lax* darf der Browser bei dem cross-site Zugriff auf
die Webseite das Cookie übertragen, dies aber nur wenn eine sichere HTTP
Methode (nicht daten-verändernd) verwendet wird und das Ziel eine
*top-level navigation* ist (sprich die Webseite aufgerufen wird und
nicht eine Operation innerhalb der Webseite).

Google wollte mit Chrome 80[7] seinen Umgang mit dem *SameSite*-Flag
verschärfen: als default würde *SameSite=Lax* als Default verwendet
werden, der Wert *SameSite=None* würde vom Webbrowser ignoriert werden.
Aufgrund der Corona/Covid-19 Situation wurden diese Änderungen
verschoben.

### Beispiel für Cookies

Ein einfaches Cookie-Beispiel bei dem das Cookie *sessionid* gesetzt
wird. Der Zugriff mittels JavaScript wurde durch *httpOnly* verboten,
das Cookie ist für alle Pfade gültig. Da kein Ablaufdatum (*Expires*)
bzw. Lebenszeit (*Max-Age*) angegeben wurde, wird das Cookie beim
Schließen des Browsers gelöscht:

    Set-Cookie: sessionid=38afes7a8; HttpOnly; Path=/

Das folgende Cookies mit Namen *id* wird vor der unsicheren Übertragung
mittels HTTP (*Secure*) als auch vor Zugriffen mittels JavaScript
(*httpOnly*) geschützt. Die Lebensdauer wurde mit einem absoluten Datum
angegeben:

    Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly

Ein Beispiel für das Setzen der sicherheitsrelevanten Header:

    Set-Cookie: CookieName=CookieValue; SameSite=Strict; httpOnly; Secure;

## JavaScript

Wird über Webprogrammierung gesprochen fällt früher oder später der Name
der Programmiersprache *JavaScript* (auch teilweise mit JS abgekürzt).
Im Jahr 1995 war das Web noch statisch[8], der vorherrschende Webbrowser
war Netscape Navigator, Microsoft Internet Explorer war gerade in einer
ersten Version erschienen. Um dynamische client-seitige Webseiten zu
ermöglichen beauftrage Netscape Brendon Eich damit eine neue
Programmiersprache für die Exekution innerhalb des Webbrowsers zu
entwicklern. Diese wurde initial *LiveScript* getauft, aus
Marketing-Gründen — Java war damals der aktuelle Hype — wurde diese
Sprache in JavaScript umbenannt.

Im Laufe der Zeit entwicklete[9] Microsoft für den Internet Explorer
einen Klon: *JScript*. Leider waren die Netscape- und
Microsoft-Versionen teilweise inkompatibel zueinander. Netscape
versuchte diesen Wildwuchs durch eine Standardisierung der Sprache durch
ECMA namens ECMAScript entgegen zu wirken, da mittlerweile Microsoft
Internet Explorer marktbeherrschend war, und das damalige Microsoft noch
weniger offener, wurde dieser Standard nie von der breiten Masse
angenommen.

Dies änderte sich 2008: seit 2005 wurde AJAX vermehrt verwendet und
basierte auf JavaScript, ebenso gab es mittlerweile drei große
JavaScript-Familien: Mozilla/Firefox, Microsoft/Internet Explorer und
Google/Chrome. Die Hersteller entschlossen sich, JavaScript zu
standardisieren und gemeinsam an ECMAScript zu arbeiten. Ende 2009 wurde
als Ergebnis ECMAScript5 veröffentlicht. Mit ECMAScript6 entstand 2015
die ,,moderne” JavaScript Sprache, seitdem entscheiden jährlich neue
ECMAScript-Versionen.

Während JavaScript-Engines ursprünglich nur im Browser eingesetzt
wurden, änderte sich dies 2010 mit dem Erscheinen von *node.js*. Diese
Umgebung verwendete zwar die Google V8 JavaScript-Engine, wurde aber als
ein Server gestartet. Durch diese Entwicklung konnte in JavaScript auch
server-seitig entwickelt werden. Da ,,die Branche” immer einen Mangel an
Programmierern hat und viele ,,Entwickler” JavaScript ,,konnten” wurde
diese Möglichkeit gerne genutzt.

Auch außerhalb von Browsern wurden Einsatzgebiete für JavaScript
entdeckt. *PhoneGap* (mittlerweile *Apache Cordova*) erlaubte das
Entwickeln von mobilen Anwendungen mittels HTML, JavaScript und CSS seit
2009, seit 2010 können mittels *Electron* traditionelle
Desktop-Applikationen in JavaScript entwickelt werden.

### Die Sprache JavaScript

JavaScript selbst ist eine Multi-Paradigma Programmiersprache und war
initial primär für imperative und funktionale Programmierstile
ausgelegt. Mit späteren ECMAScript-Versionen wurden die
Objekt-orientierten Ansätze ausgebaut, es blieb allerdings bei einer
Prototyp-basierten Vererbung. Auch das Typsystem ist noch immer
dynamisch, optionales static-typing wird durch Projekte wie *TypeScript*
oder *flow* bereitgestellt. Source Code kann in mehrere Module
strukturiert werden, auch dies ist mittlerweile Teil des ECMAScript
Standards.

Betreffend der Nebenläufigkeit wurde ein Event-basierter Ansatz gewählt.
Dies führte zu Problemen bei langlaufenden Prozessen/Requests, daher hat
sich innerhalb von JavaScript-Programmen die Verwendung von
Callback-Funktionen etabliert. In ECMAScript6 wurden ,,elegantere”
Konzepte wie *Promises* und *Futures* eingeführt, mit ECMAScript7 dieser
Einsatz durch Schlüsselwörter wie *async*/*await* vereinfacht.

Aktuell kann davon ausgegangen werden, dass JavaScript nicht so schnell
von der Bildfläche verschwinden wird. Für weiterführende Informationen
zu der Programmiersprache, bzw. zu deren moderneren Versionen, wird die
Lektüre von *The Modern JavaScript Tutorial*[10] bzw. von *You don’t
know JS*[11] empfohlen.

## Same-Origin-Policy (SOP)

Die Same-Origin-Policy ist fixer Bestandteil moderner Browser. *Origin*
ist definiert als die Kombination von Schema, Domainname und Port[12]
(z.B. <https://snikt.net:443>). Die Same-Origin-Policy moderner Browser
sagt aus, dass ein Skript das von Seite 1 geladen wurde, nur auf
Ressourcen auf Seite 2 zugreifen darf, wenn beide Seiten den identen
Origin besitzen.

Die Same-Origin-Policy ist essentiell für die Sicherheit. Würde es diese
nicht geben, könnte Javascript ausgehend von einer Seite auf die Daten
(DOM) einer externen Seite zugreifen. Da dieser Zugriff durch den
Webbrowser geschieht, würde bei diesem Zugriff das Session-Cookie mit
übertragen werden und die Operation würde mit der Identität des
Webbrowser-Benutzers durchgeführt werden.

### Cross-Origin Resource Sharing (CORS)

Während die Same-Origin-Policy aus Sicherheitssicht begrüßenswert ist,
muss sie teilweise aufgeweicht werden. Zum Beispiel könnte eine Webseite
aus Sicherheitsgründen auf mehrere Teilserver aufgeteilt worden sein:
*www.evil.com* beinhaltet die klassische Webseite, während
*api.evil.com* Operationen anbietet, die von *www* aus aufgerufen werden
sollten.

Um diese Zugriffe sauber zu erlauben, wird Cross-Domain Resource Sharing
verwendet. Hierbei werden zusätzliche Browser-Header verwendet, über
diese wird dem Webbrowser signalisiert, auf welche Operationen
zugegriffen werden darf.

Da diese Information über HTTP Header mitgeteilt wird, muss vor der
eigentlichen Operation eine Kommunikation zwischen dem Webbrowser und
dem API Server geschehen (wenn der Header erst als Antwort auf die
ausgeführte Operation übertragen worden wäre, wäre es etwas spät…).
Dafür wird die so genannten *preflight authorization* verwendet.

Operationen, bei denen Webbrowser CORS durchführen sollten:

-   alle HTTP Methoden ausser HTTP GET und HTTP POST

-   HTTP POST, abhängig vom verwendeten Content-Type

-   AJAX-Requests

-   Web Fonts

Ist eine CORS-Authorization notwendig, werden folgende Schritte
durchgeführt:

1.  Der Webbrowser eines Benutzers erkennt, dass er ausgehend von *www*
    auf *api* zugreifen will und hierfür ein CORS Authorization
    notwendig ist.

2.  Um diese durchzuführen, versendet er einen HTTP OPTIONS request auf
    die gewünschte Operation auf *api* und setzt dabei den Origin
    Header: *Origin: http://www* auf die aufrufende Webseite.

3.  Der *api* Webserver antwortet nun mit dem HTTP Header
    *Access-Control-Allow-Origin: http://www* und signalisiert dem
    Webbrowser dass der Zugriff auf *api* ausgehend von *www* erlaubt
    ist.

4.  Der Webserver führt nun die Operation auf *api* durch.

### JSONP

Vor der Verfügbarkeit von *CORS* wurde häufig *JSONP* zum Datenaustausch
mittels Javascript bzw. zum Umgehen der SOP verwendet. Bei diesem
Verfahren werden die gewünschten Daten über einen GET-Request als
Javascript-Datei geladen. Damit die geladenen Daten an Javascript
übergeben werden, wird beim Inkludieren der Daten eine Callback Funktion
mit übergeben.

Ein Beispiel, mittels der URL <a href="/php/jsonp.php?callback=callback"
class="uri">/php/jsonp.php?callback=callback</a> wird eine Operation
aufgerufen, der Parameter *callback* gibt an, wie die lokale
Javascript-Callback Funktion heißt. Das Antwortdokument ist z.B.:

    callback('{ "name":"John", "age":30, "city":"New York" }');

In der aufrufenden Webseite würde der Call nun folgendermaßen aussehen:

    <script>
    function callback(data) {
        console.log(data);
    }
    </script>
    <script src="/jsonp.php?callback=callback></script>

Es wird also vom Server ein Funktionsaufruf (auf die Callback-Funktion)
mit den serverseitigen-Daten erzeugt und dadurch im Webbrowser die
jeweilige Javascript-Funktion aufgerufen. In diesem Fall greift die SOP
nicht (da der src-Aufruf ein GET-Request ist), allerdings muss bei der
Eingabeprüfung innerhalb der callback-Funktion darauf geachtet werden,
ob Schadcode enthalten ist. Aus diesem Grund wird empfohlen, immer CORS
anstatt von JSONP einzusetzen (abgesehen davon, dass eine
JSONP-Operation meistens für alle Teilnehmer des Internets verfügbar
ist).

## WebSockets

WebSockets bieten eine bidirektionale Verbindung zwischen Webbrowser und
Webserver. Im Gegensatz zu klassischen HTTP (1.0/1.1) kann der Server
auch Nachrichten zum Client pushen, ebenso ist der Overhead geringer, da
keine dezidierten HTTP-Header pro übertragener Nachricht anfallen.

Der WebSocket wird durch einen Client-Request aufgebaut. Bei diesem
teilt der Client mit, dass er gerne die Verbindung auf einen WebSocket
upgraden will, im Erfolgsfall entgegnet der Server mit einem HTTP 101
Code. Bei der initialen Client-Anfrage werden alle HTTP Header mit
übertragen, der Server erlangt so Zugriff auf etwaige
Authorization-Header und/oder Session-Cookies und kann so Clients
identifizieren.

## WebAssembly

WebAssembly erlaubt es, hoch-performante Programme zu schreiben, welche
innerhalb eines Webbrowsers ausgeführt werden. Einzelne Funktionen, die
in WebAssembly geschrieben werden, können über JavaScript aufgerufen
werden.

Hier wären zwei Angriffsmöglichkeiten offensichtlich:

-   Verwendung von WebAssembly um einen hoch-performanten Crypto-Miner
    im Browser auszuführen.

-   Verwendung von WebAssembly zum Auslagern von Teilen von
    JavaScript-Schadcode um auf diese Weise die Detektion durch
    Anti-Malware Tools zu umgehen.

Aktuell (Stand Anfang 2020) waren keine großflächigen Angriffe mittels
WebAssembly bekannt.

## HTML5 / Progressive Web Appplications

Im Laufe der Zeit wurden die Möglichkeiten der Webbrowser zur
Interaktion mit Benutzern bzw. ihrer Umgebung immer stärker erweitert.
Diese neuen Möglichkeiten können zumeist über JavaScript innerhalb des
Browsers angesprochen werden. Indirekt werden sie allerdings weiterhin
über die Webserver kontrolliert, da diese den JavaScript Code ja zuerst
auf den jeweiligen Webseiten platzieren müssen.

### HTML5 WebStorage/LocalStorage

HTML5 LocalStorage oder WebStorage bietet Webapplikationen die
Möglichkeit lokal größere Datenmengen (um die fünf Megabyte) zu
speichern. Im Gegensatz dazu sollte bei Cookies davon ausgegangen
werden, dass maximal 4096 Byte gespeichert werden können.

Ähnlich wie bei Cookies kann ein Angreifer mit einer Javascript-Lücke
die lokalen Daten modifizieren. Eine Webapplikation muss daher auf jeden
Fall die Integrität der gespeicherten Daten vor jedem Zugriff
überprüfen. Da LocalStorage im Klartext abgelegt wird, muss bei
sensiblen Daten die Webapplikation selbst für die Vertraulichkeit der
Daten durch Verschlüsselung sorgen.

Der Entwickler kann zwischen SessionStorage und LocalStorage
unterscheiden. Ersteres wird beim Schließen des Browser-Fensters
verworfen, letzteres wird wirklich langfristig persistiert. Wenn möglich
sollte SessionStorage verwendet werden.

Verglichen zu Cookie ist WebStorage und SessionStorage stärker gegenüber
XSS-Angriffen verwundbar. Dies ist durch zwei Probleme bedingt:

1.  Cookies können durch Verwendung des *httpOnly*-Flags den Zugriff
    durch JavaScript unterbinden. Bei WebStorage wurde dieser
    Sicherheitsmechanismus nicht vorgesehen.

2.  Cookies können auf einen Unterpfad gescoped werden. Werden z.B. zwei
    Webapplikationen auf einem gemeinsamen Server Betrieben, kann ein
    Cookie z.B. für <https://example.local/app1> und ein Cookie für
    <a href="https.//example.local/app2"
    class="uri">https.//example.local/app2</a> ausgestellt werden. Eine
    Applikation besitzt keinen Zugriff auf das Cookie der anderen
    Applikation. Bei WebStorage kann dies nicht durchgeführt werden,
    WebStorage ist immer für die Domain gültig. Das heißt, dass ein
    Angreifer mit einem XSS-Fehler in app1 auf den WebStorage der app2
    zugreifen kann.

Aus diesen Gründen verbieten einige Sicherheitsrichtlinien den Einsatz
von HTML5 Web- und SessionStorage.

### HTML5 WebWorkers

Die Javascript-Umgebung eines Webbrowers ist eine Single-Thread
Umgebung. Wenn ein Javascript länger läuft, blockiert es die Ausführung
aller anderen JavaScripts auf der Seite.

Um long-running JavaScripts zu erlauben, wurden WebWorker eingeführt.
Dies sind JavaScript Programme die analog zu einem Background-Thread
gestartet werden und an welche Nachrichten von der Webseite aus
verschickt werden können.

Da WebWorker selbst AJAX-Requests (XMLHttpRequest) ausführen, und
potentiell die CPU des Hosts auslasten können, muss bei deren
Entwicklung darauf geachtet werden, dass ein Angreifer nicht Zugriff auf
diese erhält. Insbesondere sollten keine benutzer-bereitgestellten Daten
direkt an Webworker übergeben werden.

### Interaktionsmöglichkeiten

Webbrowser ersetzen aktuell Desktop-Applikationen, benötigen hierfür
allerdings erweiterte Interaktionsmöglichkeiten. Diese werden durch neue
Standards geschaffen, zum Beispiel:

-   WebRTC erlaubt die peer-to-peer Kommunikation zwischen Browsern und
    wird z.B. für Audio- oder Videokonferenzen verwendet.

-   WebNFC erlaubt die Verwendung von NFC über Webbrowser.

-   WebBluetooth erlaubt es, JavaScript auf konfigurierte BlueTooth LE
    devices zuzugreifen und wird für SmartHealth bzw. SmartHome
    Anwendungen benötigt.

Bei diesen Schnittstellen sind aktuell weniger Sicherheits-, sondern
eher Privatsphäre-Gefährdungen bekannt. Generell kann hier das
Gefährdungspotential mit jenem von Mobilapplikationen auf Smartphones
verglichen werden.

## Reflektionsfragen

1.  Wie können HTTP Header im Zuge einer Information Disclosure
    verwendet werden?

2.  Was versteht man unter SOP? Warum und wie kann dieses Prinzip mit
    CORS aufweichen?

3.  Was sind HTTP Methoden? Erkläre *safe* und *idempotente* HTTP
    Methoden.

4.  Welche Vor- und Nachteile besitzt die Verwendung von Perfect Forward
    Secrecy?

5.  Welche Flags sollten bei Verwendung von HTTP Cookie-basierter
    Sessions gesetzt werden?

6.  HTTP Cookies als auch HTTP5 localStorage/sessionStorage können zur
    Speicherung von lokalen Daten verwendet werden. Erläutere die
    Unterschiede.

[1] Da das P in HTTP bereits für Protocol steht, macht die Bezeichnung
,,HTTP Protokoll” wenig Sinn.

[2] Create, Update, Read and Delete of Resources.

[3] ältere und unsichere Versionen hießen SSL.

[4] z.B. 31536000 entspricht einem Jahr

[5] Genauere Informationen können unter <https://hstspreload.org/>
gefunden werden.

[6] siehe auch Seite

[7] voraussichtliches Veröffentlichungsdatum: Februar 2020.

[8] Im Sinne von: alle Webseiten wurden von Servern zugestellt, die
Webbrowser zeigten diesen Content nur statisch an.

[9] eigentlich reverse-engineerte…

[10] <https://javascript.info/>

[11] <https://github.com/getify/You-Dont-Know-JS>

[12] Achtung: der Internet Explorer Browser ignoriert den Port bei der
Bestimmung des Origins!
