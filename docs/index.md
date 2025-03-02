# Allgemeines

Ich will jetzt nicht mit der ,,Software ist
allgegenwärtig”-Standardfloskel beginnen. Ich glaube, dass dies die
Lebensrealität jedens ist, der dieses Buch liest. Der freie Zugriff auf
Informationen und das neue Level an Vernetztheit führen zu sozialen und
ökonomischen Entwicklungen deren Auswirkungen teilweise nicht absehbar
sind. Es sind interessante Zeiten, in denen wir leben; als Informatiker,
Hacker, etc. sind wir sind Teil einer privilegierten Schicht und dürfen
auch den Anspruch erheben, Teil dieses Wandels zu sein. Im
ursprünglichen Sinn des Wortes waren Hacker Personen, die Spaß an der
Arbeit mit neuen Technologien hatten und diese auch zweckentfremdeten —
*The Street will find its own uses for things* wie William Gibson
richtig bemerkte.

Technologie verbessert das Leben der Menschen, beinhaltet aber auch
Risiken. Durch die Allgegenwärtigkeit von Software wurden und werden
Personen von dieser abhängig. Fehler gefährden Menschen und Ökonomie.
Gerade weil Software so vielseitig ist, können auch vielseitige Fehler
entstehen. Wenn diese bösartig ausgenutzt werden[1] ist der Schritt vom
Hacker zum Cracker vollzogen. *With great power comes great
responsibility* — dies gilt auch für Softwareentwickler. Ich selbst
hielt mich für einen guten Softwareentwickler, wurde Penetration-Tester
und sah meinen ehemaligen Code mit neuen Augen. Meine Meinung über mich
selbst änderte sich rapide.

Im Frühjahr 2019 erhielt ich das Angebot, an der FH/Technikum Wien einen
Kurs *Web Security* zu halten und hoffe, dass ich damit einen kleinen
Teil beitrage die sub-optimale Sicherheitssituation zu verbessern.
Dieses Dokument dient als Skript, auch weil ich befürchte, während des
Vortrags wichtige Punkte zu übersehen bzw. als Möglichkeit
Basisinformationen aus der Vorlesung auszulagern. Es gibt leider zu
viele Schwachstellen und zu wenig Zeit um jede durchzugehen. Ein
Beweggrund für mich auf der Fachhochschule zu unterrichten ist, dass wir
alle Fehler machen. Unser Ausbildungsniveau sollte zumindest so hoch
sein, dass wir zumindest innovative Fehler begehen.

Ich spüre aber auch die Angst, etwas zu veröffentlichen das potentiell
Fehler beinhaltet oder auch teilweise meine Meinung widerspiegelt. In
der Webentwicklung gibt es keine perfekte Wahrheit, Dinge ändern sich.
Ich habe dieses Skript nach der zweiten Iteration meiner Vorlesung, nach
positivem Feedback in öffentlichen Foren als auch durch Studenten, 2020
offiziell höchst-nervös veröffentlicht.

Ich hoffe, dass die schlimmsten Missverständnisse bereits durch meine
Studenten erkannt, und von mir ausgebessert, wurden. Wenn nicht, würde
ich mich um ein kurzes Feedback unter
<a href="mailto:andreashappe@snikt.net"
class="uri">mailto:andreashappe@snikt.net</a> freuen. Ich stufe Feedback
als essentiell dafür ein, dass meine zukünftigen Studenten einen guten
Unterricht erhalten.

Die aktuelle Version dieses Buchs ist unter <https://snikt.net/websec/>
unter einer Creative-Commons Lizenz verfügbar. Der idente Inhalt wird
auch periodisch als Amazon Kindle eBook veröffentlicht. Auf Anfrage
einzelner Studenten ist dieses Skript auch als Buchversion verfügbar.
Leider ist das Update eines Papierbuchs nicht so einfach möglich. Da
Web-Technologie lebendig ist, überarbeite ich dieses Skript jedes Jahr
neu — aus diesem Grund habe ich einen niedrigen Buchpreis gewählt und
hoffe, dass dies als fair empfunden wird.

## Struktur dieses Dokumentes

Zur besseren Verständlichkeit wurde ein Top-Down-Approach gewählt. Im
Zuge der Vorlesung bewährte sich dies, da auf diese Weise die Studenten
vom Allgemeinen in die jeweiligen Spezialfälle geführt werden können.

Im ersten Part *Einführung* versuche ich das Umfeld von Security zu
beleuchten. Da meine Welt ursprünglich die Softwarenentwicklung war,
gebe ich hier auch einen groben Überblick wie Security während der
Entwicklung beachtet werden kann. Zusätzlich versuche ich unser
Zielumfeld, Web-Applikatoinen, etwas genauer zu betrachten. Auf diese
Weise soll auch sicher gestellt werden, dass Studenten bzw. Leser einen
ausreichenden Wissensstand vor Beginn der eigentlichen Security-Themen
besitzen.

Der nächste Part (*Authentication und Autorisierung*) behandelt
high-level Fehler bei der Implementierung der Benutzer- und
Berechtigungskontrolle. Drei Kapitel (*Authentication*, *Authorization*,
*Federation/Single Sign-On*) beschreiben Gebiete, die applikationsweit
betrachtet werden müssen — falls hierbei Fehler auftreten, ist zumeist
die gesamte Applikation betroffen und gebrochen.

Im darauf folgenden Part (*Injection Attacks*) wird auf verschiedene
Injection-Angriffe eingegangen. Hier wurde zwischen Angriffen, die
direkt gegen den Webserver, und Angriffen die einen Client (zumeist
Webbrowser) benötigen, unterschieden. Während auch hier Schutzmaßnahmen
am besten global für die gesamte Applikation durchgeführt werden
sollten, betrifft hier eine Schwachstelle zumeist einzelne Operationen
und kann dadurch auch lokal korrigiert werden.

## Weiterführende Informationen

Dieses Dokument kann nur eine grobe Einführung in Sicherheitsthemen
bieten. Es ist als kurzweiliges Anfixen gedacht und soll weitere
selbstständige Recherchen motivieren. Aus diesem Grund will ich hier auf
einige weitere Fortbildungsmöglichkeiten verweisen. Diese sollen als
erste Anlaufstelle für ein potentielles Selbststudium dienen.

### What to read?

Für weitere Informationen sind die Dokumente des OWASP[2]
empfehlenswert. OWASP selbst ist eine Non-Profit Organisation welche
ursprünglich das Ziel hatte, die Sicherheit von Web-Anwendungen zu
erhöhen, mittlerweile aber auch im Mobile Application bzw. IoT Umfeld
tätig ist. Das bekannteste OWASP-Dokument sind wahrscheinlich die OWASP
Top 10[3] welche eine Sammlung der 10 häufigsten
Sicherheitsschwachstellen im Web sind.

Der OWASP Application Security Verification Standard[4], kurz ASVS,
bietet eine Checkliste die von Penetration-Testern bzw.
Software-Entwicklern verwendet werden kann, um Software auf die
jeweiligen Gegenmaßnahmen für die OWASP Top 10 Angriffsvektoren zu
testen. Der OWASP Testing Guide[5] liefert zu jedem Angriffsvektor
Hintergrundinformationen, potentielle Testmöglichkeiten als auch
Referenzen auf Gegenmaßnahmen. Dieser Guide sollte eher als Referenz und
nicht als Einführungsdokument verwendet werden.

Um auf den aktuellen Stand im Bereich Web Security zu bleiben ist ein
Besuch von *The Daily Swig*[6] von PortSwigger empfehlenswert. Die
großen Topics dieser Nachrichtenseite sind Data Breaches,
Vulnerabilites, Ransomware und technische Deep Dives.

Prinzipiell ist es für Personen im Security-Umfeld höchst erstrebenswert
sowohl Programmier- als auch Softwarearchitektur-Kenntnisse zu besitzen.
Für ersteres bietet sich das Studium von JavaScript (z. B. über
<https://javascript.info>) an. Diese Sprache wird sowohl server- als
auch client-seitig (z. B. innerhalb eines Webbrowsers) eingesetzt, das
Erlernte kann dadurch an verschiedenen Stellen relevant werden.

### What to hack?

Web Security kann nicht ausschließlich theoretisch gelehrt werden, wenn
man in dem Umfeld aktiv sein will muss man hands-on Praxisbeispiele
sehen und auch versuchen. Das Gefühl, bei einer Web-Applikation
permanent mit dem Kopf gegen die Wand zu laufen, immer weider neue
Angriffe erfolglos zu versuchen bis man einen funktionierenden Angriff
gefunden, und sich nach erfolgter Ausnutzung zufrieden zurücklehnen
kann, kann nur erlebt werden. Glückerlicher Weise gibt es mittlerweile
eine Vielzahl an gratis bzw. freemium-basierten Webangeboten welche
genau diese Gelegenheit bieten.

Eine Auflistung dieser kann in Tabelle
<a href="#tbl:online_hacking" data-reference-type="ref"
data-reference="tbl:online_hacking">1.1</a> vorgefunden werden[7]. Die
Spalten ,,online”, ,,VPN” und ,,VM” sollten darstellen, wie das
jeweilige Angebot genutzt werden kann. ,,Online” sind Kurse, bei denen
eine verwundbare Webapplikation direkt über den Browser des Benutzers
getestet werden kann: es muss nicht zwingend am lokalen Rechner eine
Virtualisierungslösung oder ähnliches installiert werden. Lösungen der
Spalte ,,VM” sind das genaue Gegenteil: hier kann zumeist eine virtuelle
Maschine bezogen und lokal installiert werden. In dieser virtuellen
Maschine befindet sich die zu testende Software. In diesem Fall benötigt
man zwar lokal installierte Virtualisierungssoftware, ist dafür
allerdings von der Internet-/Netzwerkverbindungsqualität großteils
unabhängig. ,,VPN”-Lösungen sind eine Mischform: bei diesen erhält man
Zugangsdaten für einen VPN-Einwahlknoten und gelangt über diesen zu
einem virtuellen Labornetzwerk in welchem sich virtuelle Maschinen mit
verwundbarer Software befinden. In diesem Fall muss man zwar lokal einen
VPN-Client installieren, diese ist allerdings leichtgewichtiger als eine
volle Virtualisierungslösung. Zusätzlich bieten ,,VPN”-basierte Ansätze
auch teilweise größere Netzwerke in denen man auch Post-Exploitation
Tätigkeiten wie Lateral Movement trainieren kann.

| Name                    | auch kommerziell | Online | VPN | VM  |
|:------------------------|:-----------------|:-------|:----|:----|
| Web Security Academy[8] | ja               | x      |     |     |
| Vulnhub[9]              | nein             |        |     | x   |
| Pentester lab[10]       | ja               | x      |     | x   |
| Hack the Box[11]        | ja               |        | x   |     |

Online-Angebote für Hacking-,,Praxisbeispiele”

Im Scope unterscheiden sich die gelisteten Lösungen ebenso. Während *Web
Security Academy* und *Pentester Lab* sich an Security-Schulungen
anlehnen und Theorie bzw. Hintergrundinformationen bieten, steht bei
*VulnHub* und *Hack the Box* das ,,Doing”, also das Hacken von
Maschinen, im Vordergrund. Die beiden letztgenannten Plattformen bieten
weniger Hintergrundinformationen, diese können aber im Normalfall durch
Suche im Internet gefunden werden.

### What to attend?

OWASP selbst ist in Städte-zentrische Chapters organisiert, ich bin zum
Beispiel bei dem Chapter Vienna (Austria) aktiv[12]. Aktuell finden
aufgrund der anhaltenden COVID-19 Situation keine Stammtische statt, es
gibt allerdings unregelmässige virtuelle Meetupgs.

## Out-of-Scope für dieses Skript

Auf drei wichtige Bereiche wird im Zuge dieses Skripts nicht explizit
eingegangen:

### Denial-of-Service Angriffe

Denial-of-Service Angriffe zielen darauf ab, die Verfügbarkeit einer
Applikation zu beeinträchtigen. Dadurch kann der Dienst nicht mehr
benutzt bzw. konsumiert werden und dem Betreiber entstehen Kosten, z.B.
Verdienstentgang durch einen ausgefallenen Webshop.

Ein DoS-Angriff zielt entweder auf eine Applikations-bezogene Ressource
wie z.B. erlaubte Verbindungen pro Applikationsbenutzer oder eine
fundamentale Systemressource wie z.B. CPU-Zeit, Speicher oder
Netzwerkbandbreite ab. Als Applikationsentwickler kann man bei
Ressourcen-intensiven Operationen mittels Rate-Limits die Situation
entschärfen.

In diesem Dokument wird nicht tiefer auf DoS-Angriffe eingegangen, da
diese quasi die Holzhammermethode darstellen. Gerade gegenüber Angriffen
gegen die Netzwerkbandbreite kann nur über kommerzielle Cloud- bzw.
Rechenzentrenbetreiber entgegengewirkt werden. Diese sind kostspielig
und es entsteht eine Asymmetrie: die Abwehr des Angriffs kann
kostspieliger als der Angriff selbst werden. Somit wird aus einem
technischen DoS ein monetärer DoS.

### Security und Usability

Es gibt das Vorurteil, dass Sicherheit und Usability konträr zueinander
sind. Während dies in wenigen bedauerlichen Einzelfällen gegeben sein
kann, sollte dies nicht als Pauschalausrede missbraucht werden.

Der Benutzer will primär eine Aufgabe erledigen. Im Zuge der Erledigung
dieser Aufgabe sollte Sicherheit nicht im Weg stehen. Stattdessen sollte
der offensichtliche Weg der Aufgabenerledigung sicher implementiert sein
und den Benutzer über einen sicheren Weg zur Erledigung der Aufgabe
leiten. Falls sicherheitsrelevante Benutzerentscheidungen notwendig
sind, sollten diese möglichst früh erfolgen — wird dies während der
Abarbeitung einer Aufgabe durchgeführt, kann der Benutzer so fokussiert
sein, dass die Sicherheitsentscheidung nur peripher beachtet wird.

Ebenso sollte der Benutzer nicht mit irrelevanten Fragen bombardiert
werden da daruch nur der “Meldung-wegklicken”-Reflex des Benutzers
konditioniert wird. Die Willigkeit eines Benutzers, auf Sicherheit
Rücksicht zu nehmen ist begrenzt, vergleichbar mit einer Batterie. Wenn
diese erschöpft ist, wird weniger (oder gar keine) Rücksicht auf die
Security genommen.

Ein besserer Weg ist es, per default sichere Prozesse zu implementieren
und im Bedarfsfall unsichere Operationen durch den Benutzer explizit zu
erlauben. Die dabei verwendeten Benutzerinteraktionen sollten dem
NEAT-Prinzipien genügen:

-   Necessary: kann die Applikation, anstatt den Benutzer zu fragen, das
    Problem auf eine andere sichere Art und Wiese lösen?

-   Explained: besitzt der Benutzer das notwendige Wissen um eine
    informierte Entscheidung zu treffen?

-   Actionable: kann der Benutzer überhaupt sinnvoll auf die
    dargestellte Meldung reagieren?

-   Tested: ist die Meldung innerhalb der UX sinnvoll und wurde
    getestet, ob sie in jeglicher Form von Benutzerfluss sinnvoll ist?

Im Zuge der DSGVO/GDPR wurde bestimmt, dass Software *secure by design
and default* sein muss. Dies bedeutet, dass Software die Möglichkeit
einer sicheren Konfiguration bieten, und diese im Auslieferungszustand
auch sicher konfiguriert sein muss. Ein dagegen verstossendes Beispiel
wäre der Einsatz von Default-Passwörtern.

### Ethical Web Development

Technik an sich ist wertneutral. Sobald diese allerdings in Berührung
mit der Realität kommt, entsteht ein ethischer Impact. Web Applikationen
sind hier keine Ausnahme. Im Zuge des Skripts wird auf ethischen Impact
nicht explizit eingegangen, da der Inhalt der Vorlesung das Werkzeug und
nicht das Ziel des erstellten Werks ist.

Um die ethische Dimension nicht vollständig zu ignorieren, ein paar
Richtlinien der EDRi[13]:

Allow as much data processing on an individual’s device as possible.  
Dies würde im Web-Umfeld den Einsatz von JavaScript bedingen, da nur auf
diese Weise Daten direkt im Browser des Benutzers verarbeitet werden
können.

Where you must deal with user data, use encryption.  
Dies inkludiert sowohl Transport-Level Encryption (wie TLS) als auch
Verschlüsselung der bearbeiteten Daten.

Where possible also use data minimisation methods.  
Das Minimalprinzip sollte auch auf die gespeicherten Daten angewendet
werden. Daten die eine Applikation nicht besitzt sind Daten, die auch
nicht entwendet oder zweckentfremdet werden können.

Use first-party resources and avoid using third-party resources.  
Es besteht die Sorge, dass externe Ressourcen modifiziert werden
könnten. Dies soll durch die Verwendung eigener Ressourcen vermieden
werden. Falls notwendig, können CSP-Direktiven bzw. Subresource
Integrity verwendet werden um die Integrität externer Ressourcen
sicherzustellen.

[1] Subjektiv im Auge des Betrachters.

[2] Open Web Application Security Project

[3] <https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project>

[4] <https://www.owasp.org/index.php/Category:OWASP_Application_Security_Verification_Standard_Project>

[5] <https://www.owasp.org/images/1/19/OTGv4.pdf>

[6] <https://portswigger.net/daily-swig>

[7] Ich habe mich bei dieser Liste auf Angebote welche, zumindest
teilweise, gratis nutzbar sind, beschränkt, daher fehlt hier z.B.
Offensive Security (<a href="www.offensive-security.com"
class="uri">www.offensive-security.com</a>) obwohl diese von mir hoch
geschätzt werden.

[8] <https://portswigger.net/web-security>

[9] <https://www.vulnhub.com/>

[10] <https://pentesterlab.com/>

[11] <https://www.hackthebox.eu/>

[12] <https://www.meetup.com/de-DE/OWASP-Vienna-Chapter/>

[13] <https://edri.org/ethical-web-dev/>
