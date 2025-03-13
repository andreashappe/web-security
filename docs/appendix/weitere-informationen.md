# Weiterführende Informationen

Dieses Dokument kann nur eine grobe Einführung in Sicherheitsthemen
bieten. Es ist als kurzweiliges Anfixen gedacht und soll weitere
selbstständige Recherchen motivieren. Aus diesem Grund will ich hier auf
einige weitere Fortbildungsmöglichkeiten verweisen. Diese sollen als
erste Anlaufstelle für ein potentielles Selbststudium dienen.

## What to read?

Für weitere Informationen sind die Dokumente des [OWASP](https://www.owasp.org)
empfehlenswert. OWASP selbst ist eine Non-Profit Organisation welche
ursprünglich das Ziel hatte, die Sicherheit von Web-Anwendungen zu
erhöhen, mittlerweile aber auch im Mobile Application bzw. IoT Umfeld
tätig ist. Das bekannteste OWASP-Dokument sind wahrscheinlich die [OWASP
Top 10](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project) welche eine Sammlung der 10 häufigsten
Sicherheitsschwachstellen im Web sind.

Der [OWASP Application Security Verification Standard](https://www.owasp.org/index.php/Category:OWASP_Application_Security_Verification_Standard_Project), kurz ASVS,
bietet eine Checkliste die von Penetration-Testern bzw.
Software-Entwicklern verwendet werden kann, um Software auf die
jeweiligen Gegenmaßnahmen für die OWASP Top 10 Angriffsvektoren zu
testen. Der [OWASP Testing Guide](https://www.owasp.org/images/1/19/OTGv4.pdf) liefert zu jedem Angriffsvektor
Hintergrundinformationen, potentielle Testmöglichkeiten als auch
Referenzen auf Gegenmaßnahmen. Dieser Guide sollte eher als Referenz und
nicht als Einführungsdokument verwendet werden.

Prinzipiell ist es für Personen im Security-Umfeld höchst erstrebenswert
sowohl Programmier- als auch Softwarearchitektur-Kenntnisse zu besitzen.
Für ersteres bietet sich das Studium von JavaScript (z. B. über
<https://javascript.info>) an. Diese Sprache wird sowohl server- als
auch client-seitig (z. B. innerhalb eines Webbrowsers) eingesetzt, das
Erlernte kann dadurch an verschiedenen Stellen relevant werden.

## What to hack?

Web Security kann nicht ausschließlich theoretisch gelehrt werden, wenn
man in dem Umfeld aktiv sein will muss man hands-on Praxisbeispiele
sehen und auch versuchen. Das Gefühl, bei einer Web-Applikation
permanent mit dem Kopf gegen die Wand zu laufen, immer weider neue
Angriffe erfolglos zu versuchen bis man einen funktionierenden Angriff
gefunden, und sich nach erfolgter Ausnutzung zufrieden zurücklehnen
kann, kann nur erlebt werden. Glückerlicher Weise gibt es mittlerweile
eine Vielzahl an gratis bzw. freemium-basierten Webangeboten welche
genau diese Gelegenheit bieten.

Eine Auflistung dieser kann in folgender Tabelle vorgefunden werden[7]. Die
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
| [Web Security Academy](https://portswigger.net/web-security) | ja               | x      |     |     |
| [Vulnhub](https://www.vulnhub.com/)              | nein             |        |     | x   |
| [Pentester lab](https://pentesterlab.com/)       | ja               | x      |     | x   |
| [Hack the Box](https://www.hackthebox.eu/)        | ja               |        | x   |     |

Online-Angebote für Hacking-,,Praxisbeispiele”

Im Scope unterscheiden sich die gelisteten Lösungen ebenso. Während *Web
Security Academy* und *Pentester Lab* sich an Security-Schulungen
anlehnen und Theorie bzw. Hintergrundinformationen bieten, steht bei
*VulnHub* und *Hack the Box* das ,,Doing”, also das Hacken von
Maschinen, im Vordergrund. Die beiden letztgenannten Plattformen bieten
weniger Hintergrundinformationen, diese können aber im Normalfall durch
Suche im Internet gefunden werden.

## What to attend?

OWASP selbst ist in Städte-zentrische Chapters organisiert, ich bin zum
Beispiel bei dem [OWASP Chapter Klagenfurt](https://www.meetup.com/owasp-klagenfurt-chapter/) (Austria) aktiv.

[7] Ich habe mich bei dieser Liste auf Angebote welche, zumindest
teilweise, gratis nutzbar sind, beschränkt, daher fehlt hier z.B.
[Offensive Security](https://www.offensive-security.com)
obwohl diese von mir hoch geschätzt werden.
