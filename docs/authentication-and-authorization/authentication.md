# Authentifikation

Sobald eine Webapplikation sensitive bzw. privilegierte Operationen und
Daten bereitstellt, besteht die Notwendigkeit die Identität des
zugreifenden Benutzers zu erheben und zu verifizieren.

Authentifikation kann als die Verifikation einer behaupteten
Benutzeridentität über zuvor ausgetauschte Details (wie z.B. das während
der Registrierung angegebenen Passwort) definiert werden. Nach erfolgtem
Login wird zumeist eine langfristige Verbindung (Session) zu dem
Benutzer aufgebaut. Bei Folgezugriffen wird dieses Vertrauensverhältnis
verwendet, um den Benutzer sowohl zu identifizieren als auch
authentifizieren.

## Identifikation und Authentifikation

Bei der Identifikation claimed der Benutzer seine Identität, z.B. durch
Angabe eines zuvor registrierten Benutzernamens. Weitere Möglichkeiten
wären z.B. SmartCards oder biometrische Methoden. Die Identifikation
wird zumeist mit einer Authentifikation kombiniert.

Die Authentifikation dient zur Validierung der behaupteten Identität des
Benutzers. Es gibt mehrere Möglichkeiten (Faktoren) über welche ein
Benutzer seine Identität authentifizieren kann, Tabelle
<a href="#tbl:factors" data-reference-type="ref"
data-reference="tbl:factors">1.1</a> gibt eine kurze Übersicht häufig
genutzter Faktoren.

| Faktor              | Art                                             |
|:--------------------|:------------------------------------------------|
| Passwort            | Something you know                              |
| Hardware-Tokens     | something you have                              |
| Biometrie           | something you are                               |
| Soziale Beziehungen | someone you know                                |
| Email-Konto         | z.B. Slackanmeldung mittels Link in Email       |
| PostIdent           | Verifikation am Postamt mittels Ausweis         |
| VideoIdent          | Verifikation mit Ausweis mittels Videokonferenz |
| PhotoIdent          | Verifikation über zugeschicktes Ausweisbild     |

Verschiedene Faktoren zur Authentication

Bei der initialen Registrierung und bei nachfolgenden Anmeldungen können
unterschiedliche Faktoren verwendet werden. Z. B. VideoIdent bei der
Registrierung, bei Folgeanmeldungen Passwörter.

Durch die Kombination mehrere Faktoren erhält man eine
Multifaktor-Authentifikation (MFA), häufig wird als
Zweifaktoren-Authentifikation (2FA) ein Passwort mit einem Token
kombiniert. Wichtig bei der MFA ist die Wahl von Faktoren aus
unterschiedlichen Klassen. Es macht z.B. wenig Sinn eine
Fingerprint-Authentifikation mit einer Iris-Authentifikation zu
kombinieren. Ein schönes Beispiel, bei dem Faktoren verschiedener
Klassen schlecht durch einen User kombiniert werden wäre es, wenn der
Benutzer einer Bankomatkarte seinen PIN (something you know) auf seine
Bankomatkarte (something you have) schreibt.

## Login- und Logout

Wenn ein Login- und Logout innerhalb der Applikation implementiert
werden, müssen gewisse Grundfunktionen abgedeckt sein.

### Login-Formular

Das Login-Formular sollte entsprechend dem KISS-Prinzip als einfaches
HTML-Formular implementiert werden. Hauptgrund dafür ist, dass das
Login-Formular mit Passwort-Managern kompatibel sein sollte. Dies
impliziert, dass das Login-Formular aus Textfeldern für Benutzername und
Passwort als auch einem Login-Button bestehen sollte.

Negative Beispiele die den Einsatz von Passwort-Managern erschweren:

-   Benutzername und Passwort-Feld sind nicht innerhalb der gleichen
    Seite

-   Password-Feld wird erst angezeigt, nachdem ein Benutzername
    eingegeben wurde

-   Verwendung von Flash-, Silverlight- oder Java-Applets

-   Authentication through EMail a la Slack (Email mit Bestätigungslink
    dient als Passwortersatz)

-   HTTP BASIC basierte Authentifikation

### User Enumeration Angriffe

Eine User Enumeration liegt vor, wenn ein Angreifer gezielt
Informationen über das Vorhandensein eines Benutzers erzielen kann.
Zumeist geschieht dies über schlecht gewählte Fehlermeldungen. So kann
ein Angreifer bei der Fehlermeldung “Passwort invalid” davon ausgehen,
dass ein Benutzername dem System bekannt ist. Lösung: Verwendung
generischer Fehlermeldungen wie “Benutzer/Passwort-Kombination nicht
bekannt”.

Während dies bei einem Login-Formular leicht zu bewerkstelligen ist,
sind weitere Operationen komplexer:

-   “Passwort vergessen”-Funktion: hier muss meistens eine Email-Adresse
    angegeben werden. Falls die Email-Adresse dem System nicht bekannt
    ist, sollte keine Fehlermeldung ausgegeben werden, sondern ein
    Hinweis, dass an die angegebene Email eine Benachrichtigungsemail
    versendet wurde.

-   Bereits vorhandene Email-Adresse bei Registrierung: hier sollte
    ebenso eine neutrale Erfolgsmeldung innerhalb der Webseite
    ausgegeben, und anschließend in einer Bestätigungsemail der Benutzer
    darauf hingewiesen werden, dass er bereits ein Konto mit der
    Email-Adresse angelegt hatte.

-   Bereits vorhandener Login bei Registrierung: hier muss dem User eine
    Fehlermeldung angezeigt werden.

Generell ist dieser Bereich einer derjenigen, bei denen Usability und
Security potentiell konträre Ziele besitzen.

### Brute-Force Angriffe gegen Login-Formular

Brute-Force Angriffe versuchen mittels automatisierter Anfragen eine
valide Kombination von Benutzernamen und Passwort zu erraten. Durch die
Kenntnis bekannter Benutzernamen können Brute-Force Angriffe
beschleunigt werden (z.B. durch eine zu vorige User Enumeration).

Technisch sind Brute-Force Angriffe einfach umzusetzen, Tool-Support ist
massiv vorhanden. Die erreichte Geschwindigkeit befindet sich meistens
bei mehreren Zehntausend Versuchen pro Stunde.

Brute-Force Angriffe versuchen entweder eine Kombination des gesamten
Testbereichs (Buchstaben, Zahlen, Sonderzeichen) oder verwenden
vorbereitete Passwortlisten. Diese können auf verschiedene Arten
bereitgestellt werden:

-   Sammlung von Passwörtern von etwaigen Password Leaks.

-   Automatisch generierte Liste basierend auf den öffentlichen Seiten
    der zu testenden Homepage.

-   Deep-Learning basierte Verfahren, die basierend auf existierenden
    Passwortlisten neue Passwortlisten generieren.

Gegenmaßnahmen zielen auf eine Verlangsamung des Angriffs bzw. auf eine
Sperre betroffener Konten ab:

-   Rate-Limits bzw. Verlangsamung bei Fehlerseiten.

-   Sperre von Benutzeraccounts bzw. IP-Adressen nach einer definierten
    Anzahl von Fehlversuchen.

-   Einsatz einer Mehrfaktorauthentication. Durch die benötigte manuelle
    Interaktion wird eine Brute-Force Attacke ausgebremst. Hier ist die
    Wahl eines geeigneten Faktors und eine geeignete Integration
    notwendig.

### Logout

Symmetrisch zum Login sollte auch eine Logout-Operation implementiert
werden. Dadurch kann der Benutzer seine Session beenden und dadurch das
mögliche verwendbare Zeitfenster gegenüber Angriffen (z.B. gegenüber
CSRF-Angriffen) verkleinern.

Bei neueren Standards wie der ÖNORM A77.00 gibt es die Anforderung, dass
der Benutzer nicht nur seine aktuelle, sondern auch alle seine
bestehenden Sitzungen beenden kann.

Beispiel: Benutzer besitzt einen Desktop und einen Laptop. Der Laptop
wird gestohlen, es sollte möglich sein eine offene Web-session am Laptop
über den Desktop zu beenden.

### Deaktivieren/Sperren/Löschen von Accounts

Wird ein Benutzeraccount gelöscht oder deaktiviert stellt sich die
Frage, wie mit den gelöschten Daten des Benutzers umzugehen ist. Wurde
ein Account gesperrt muss dafür Sorge getragen werden, dass:

-   bereits ausgestellte Recovery-Codes den Account nicht reaktivieren
    können

-   aktive Benutzersessions beendet werden

-   der Benutzer sich nicht mehr einloggen kann

Die Hauptfrage bei einem zu löschenden Account ist, welche Daten
gelöscht, und welche Daten persistiert werden müssen (beides primär aus
rechtlichen Gründen).

## Behandlung von Passwörtern

Die grundsätzliche Strategie wäre, keine Passwörter in der Applikation
zu speichern, einzugeben oder zu verarbeiten. Wenn die Applikation
niemals Zugriff auf Passwörter hat, können diese auch nicht verloren
werden. Falls dies nicht möglich ist, müssen beim Umgang mit Passwörtern
gewisse Grundregeln eingehalten werden.

Genauere Informationen zur sicheren Speicherung von Passwörtern können
im Kapitel *Sensitive Data Exposure*
(<a href="#password_storage" data-reference-type="ref"
data-reference="password_storage">[password_storage]</a>) gefunden
werden.

Prinzipiell können Angriffe gegen Passwörter in drei Kategorien
eingeteilt werden:

1.  Disclosure tritt auf, wenn das Passwort unbeabsichtigt
    “veröffentlicht” wird. Dies kann z.B. durch Notizzettel, Wikis oder
    auch durch phishing geschehen.

2.  Online Attacks sind Angriffe gegenüber einem Login-System. Diese
    können durch das Websystem erkannt werden.

3.  Offline Attacks sind Angriffe gegenüber geleakten Passwort-Hashes.
    Diese können durch das Websystem nicht erkannt werden.

### Passwort-Qualität

Kann ein neues Passwort in der Applikation gesetzt werden, sollte dieses
gewisse Mindestanforderungen erfüllen. 2018 wurden die NIST 800-63-3:
Digital Identity Guidelines[1] veröffentlicht, diese inkludieren mehrere
Best-Pracises im Umgang mit Passwörtern:

-   Minimale Passwortlänge: 8 Zeichen. Ein Unicode Zeichen ist ein
    Zeichen.

-   Falls ein Benutzer ein längeres Passwort eingibt, müssen mindestens
    64 Zeichen gespeichert werden.

-   Das periodische Neusetzen von Passwörtern wird nicht mehr gefordert.
    Diese Maßnahme bewirkte schwächere Passwörter.

-   Komplexitätsregeln bei Passwörtern (mindestens ein Sonderzeichen und
    ähnliches) wurden entfernt.

-   Neu eingegebene Passwörter müssen gegen eine Liste von bekannten
    Passwort-Leaks und gegen bekannte Standard bzw. häufig genutzte
    Passwörter getestet werden.

-   Passwort-Hints dürfen nicht mehr verwendet werden.

Um eingegebene Passwörter gegen eine Liste von geleakten Passwörtern zu
überprüfen, kann z.B. von <https://haveibeenpwned.com> (im Folgenden
immer haveibeenpwned genannt) eine ca. 10 Gigabyte große Liste an
Passwort-Hashes heruntergeladen werden. Alternativ bietet haveibeenpwned
einen Passwort-Check Service an. Bei diesem werden Passwörter nicht als
Hash übermittelt (ansonsten würde der Serverbetreiber Wissen über die
verwendeten Passwörter erhalten), sondern es wird das Passwort gehashed,
die ersten 5 Zeichen des Hashes übertragen und anschließend eine Liste
aller gefundenen Hashes an den Client zurück übertragen.

### Passwort-Reset

Ein wichtiger Grundsatz ist *Account recovery not password recovery*.
Dieser sagt aus, dass der Benutzer wieder Zugang zu seinem Account
erhält, aber nicht sein bestehendes Passwort einsehen kann. In einer
korrekt implementierten Applikation sollte das bestehende Passwort
nirgends unverschlüsselt gespeichert werden, daher sollte diese
Möglichkeit prinzipiell nicht technisch möglich sein.

Meistens wird man aus Gründen der Usability dem User eine Möglichkeit
des Passwort-Resets geben. Dies wird normalerweise über eine Email mit
einem Passwort-Reset Link implementiert. Folgende
Implementierungshinweise:

-   Dem User sein bestehendes Passwort zuzusenden ist ein epic fail da
    hierfür das Passwort unverschlüsselt gespeichert werden müsste.

-   Dem Benutzer ein neues Passwort per Email zuzuschicken sollte
    vermieden werden.

-   Der generierte Link sollte nur einmalig verwendbar sein, und auch
    nur das Updaten des aktuellen (vergessenen) Passworts erlauben.

-   der generierte Link sollte nur für den betreffenden User verwendbar
    sein.

Hinweis: die aktuellen NIST Richtlinien verbieten explizit die
Verwendung von “Passwort Fragen” (“In welcher Straße bist du
aufgewachsen, etc.”) zum Zurücksetzen des Passworts. Grund: diese Fragen
waren bei bekannteren Personen einfach nachzuforschen.

Die Verifikation kann auch über Alternate Transports geschehen. Ein
Beispiel wäre die österreichische Sozialversicherung, bei der ein neues
Passwort über einen eingeschriebenen Brief an den User verschickt wird.
Dadurch wird eine Identitätsfeststellung des Empfängers erzwungen.

Sobald ein neues Passwort gesetzt wurde sollte der Benutzer über mehrere
Wege über diese Passwortänderung notifiziert, und ihm eine Möglichkeit
der Account-Sperre gegeben, werden. Typische Nachrichtenwege wären z.B.
Emails oder SMS.

### Ändern von Passwörtern

Der Benutzer sollte die Möglichkeit besitzen, sein Passwort neu zu
setzen. Für eine sichere Operation muss folgendes gegeben sein:

-   der User muss aktuell authenticated sein

-   der Benutzer kann nur sein eigenes Passwort ändern

-   im Zuge der Operation, die das neue Passwort setzt, muss auch das
    alte Passwort erfragt werden.

Das bestehende Password wird erfragt, damit ein Angreifer mit Zugriff
auf die Session nicht ein neues Passwort setzen kann (und dadurch
unbegrenzten Zugriff auf das Benutzerkonto erhält). Im einfachsten Fall
geschieht so ein Angriff indem der Angreifer auf einem nicht-gesperrten
Computer ein neues Passwort innerhalb einer eingeloggten Webapplikation
eingibt.

Damit diese Schutzmaßnahme funktioniert, müssen sowohl das alte als auch
das neue Passwort im gleichen Schritt übermittelt werden. Ebenso
verhindert dies CSRF-basierte Angriffe.

### Passwörter für Dritt-Dienste

Teilweise können Passwörter nicht gehashed innerhalb der Applikation
gespeichert werden. Dies tritt zum Beispiel auf, wenn das Passwort an
eine Drittapplikationen weitergegeben werden muss – eine Webapplikation
welche zur Darstellung eines IMAP-Emailkontos dient muss z.B. innerhalb
der Applikation die Zugangsdaten für das externe Email-System speichern.
Falls dieser Email-Server das Passwort in plain-text benötigt, muss die
Applikation nun auch das Passwort in plain-text speichern und kann daher
keine Einweg-Hashfunktion anwenden.

Prinzipiell ist hier das Grundproblem, dass das sensible Passwort an
eine externe, potentiell nicht vertrauenswürdige, Applikation übergeben
werden muss.

## Alternativen zu Passwort-basierten Logins

Benutzer sind notorisch schlecht bei der Wahl sicherer Passwörter. Um
diese Gefahrenquelle zu minimieren wird versucht, entweder die
Sicherheit des Login-Vorgangs mit einem zweiten Faktor zu verstärken,
oder Passwörter vollkommen durch physikalische Tokens zu ersetzen.

TOTP ist ein Verfahren, dass zur Implementierung eines zweiten Faktors
eingesetzt werden kann. Im Gegensatz dazu, werden die Protokolle der
FIDO-Allianz häufig für die Implementierung Passwort-loser
Authentifizierungsvorgänge genutzt.

### TOTP

Time-based One-Time Passwords (TOTP, RFC 6238) ist ein häufig
verwendetes Verfahren zur Implementierung eines weiteren
Authentication-Faktors. Es ist ein Zusammenspiel zwischen Authenticator
(meist eine mobile Applikation) und einer Webapplikation.

Initial wird ein shared secret key zwischen Authenticator und
Webapplikation ausgetauscht. Wird nun eine Authentifikation benötigt
wird nun auf beiden Seiten die aktuelle Systemzeit (in Sekunden seit
Beginn der UNIX Epoche) auf 30 Sekunden gerundet und ein MAC (unter
Zuhilfenahme des shared secret keys) gebildet. Dieser MAC wird nun auf
31 bit gekürzt und in einen 6 oder 8 stelligen Zahlencode verwandelt.
Dieser wird am Authenticator angezeigt und muss vom User in der
Webapplikation als weiterer Faktor eingegeben werden. Sofern beide
berechnete Werte ident sind, wird die Authentifikation erfolgreich
durchgeführt.

Ein Vorteil dieses Verfahrens ist, dass nach dem initialen
Schlüsselaustausch keine Netzwerkverbindung zwischen Authenticator und
Applikation benötigt wird. Ein Nachteil ist, dass die Systemuhren der
betroffenen Systeme synchronisiert werden müssen. Ebenso kann bei TOTP
kein *device-binding* durchgeführt werden: die Webapplikation kann nicht
feststellen, auf wie vielen devices ein TOTP-Secret eingespielt wurde.
Ebenso ist der Vorgang der initialen Secret-Verteilung gefährlich: wird
hier z.B. von einem Benutzer ein Selfie inklusive dem
QR-Code/Secret-Code erstellt und veröffentlicht, wurde auf diese Weise
die gesamte Sicherheit des Verfahrens kompromittiert.

### Protokolle der FIDO-Alliance

Die FIDO-Alliance ist eine nicht-kommerzielle Vereinigung von über 150
Unternehmen und Behörden mit dem Ziel, offene und lizenzfreie
Authentifizierung-Industriestandards zu schaffen. Ihre Mitglieder
beinhalten u.a. Alibaba, Google, Microsoft, Samsung und YubiCo. Die
Abkürzung FIDO steht dabei für *Fast IDentity Online*.

Ende 2014 wurde FIDO 1.0 veröffentlicht, dieser Standard umfasste:

-   U2F (Universal Second Factor) standardisiert den Einsatz von
    physikalischen Tokens (wie z.B. einem Yubikey). Sofern die
    Webapplikation und der verwendete Webbrowser U2F unterstützen kann
    der Benutzer sich mit einem Hardware-Token authentifizieren (z.B.
    durch Knopfdruck auf einem USB-Stick oder durch Antappen eines
    NFC/BLE Tokens).

-   UAF (Universal Authentication Framework) dient zur Implementierung
    eines Passwort-losen Logins. Der Benutzer muss über ein
    UAF-kompatibles Endgerät verfügen (z.B. Windows 10) und registriert
    quasi sein Endgerät bei der Webapplikation.

Das Grundprinzip basiert auf public key Kryptographie. Wenn ein
Authenticator (z.B. Android Gerät) als Gerät eines Benutzers registriert
wird, wird im Gerät ein public/private key pair generiert und der public
key dem FIDO Server mitgeteilt. Im Falle einer Benutzerauthentication
wird die Anfrage des Servers vom Client mit dem private key signiert,
mit dem serverseitig hinterlegten public key verglichen und damit die
Identität des Benutzers verifiziert. Lokal werden meistens biometrische
Methoden zum Schutz der Tokens verwendet.

FIDO2 kombiniert mehrere Projekte um eine passwortlose Authentication zu
erlauben. Das vom W3C standardisierte WebAuthn wird von Webbrowsern
implementiert und erlaubt es Webapplikationen (mittels JavaScript) eine
FIDO Benutzerauthentication durchzuführen. Das
Client-to-Authenticator-Protocol (CTAP) standardisiert das
Kommunikationprotokoll zwischen Authenticator (Hardware-Tokens) und dem
Webbrowser (Client). Es gibt zwei Varianten CTAP1 und CTAP2 wobei CTAP1
dem FIDO U2F Standard entspricht.

### Gegenüberstellung FIDO/TOTP

Wird FIDO mit TOTP verglichen, können konzeptionelle Unterschiede
erkannt werden:

-   FIDO1/2 überträgt nur einen öffentlichen Schlüssel während der
    Registrierung eines neuen Authenticators. TOTP überträgt ein shared
    secret. Bei FIDO verlässt der geheime Schlüssel niemals den
    Authenticator.

-   TOTP benötigt im Gegensatz zu FIDO während der Authentifizierung
    keine aktive Netzwerkverbindung zwischen Authenticator und Service.
    Stattdessen benötigt TOTP eine synchronisierte Systemzeit zwischen
    allen beteiligten Parteien.

-   Da bei FIDO der geheime Schlüssel nicht den Authenticator verlässt,
    gibt es ein Pairing zwischen dem Device und dem Service. Bei TOTP
    kann ein Benutzer das idente shared secret mit mehreren
    Authenticators verwenden, eine Zuordnung zu einem dedizierten
    Authenticator ist daher nicht möglich.

-   TOTP besitzt keine Hardware-Requirements und kann daher gratis in
    Software implementiert werden. Während FIDO ein freier Standard ist,
    setzt es einen Hardware-Token voraus — dadurch ist der Einsatz von
    FIDO mit Hardware-Kosten verbunden und ist tendenziell nicht
    “gratis”.

## Authentication von Operationen

Um eine serverseitige Rechtekontrolle durchführen zu können ist sowohl
eine Benutzer-Identifikation, -Authentifikation und Authorization
notwendig. Es muss sowohl die Benutzeridentität als auch dessen
Berechtigung (Authorization) überprüft werden. Dies muss vor Exekution
der aufgerufenen Operation serverseitig durchgeführt werden.

Da für jede Überprüfung der Authorization eine Feststellung der
Benutzeridentität notwendig ist, werden beide meistens zusammengefasst
durchgeführt. Ein wichtiger Unterschied ist, dass bei einem Fehler
innerhalb der Authorization ein Angreifer eine Operation trotz fehlender
Berechtigung aufrufen kann, dieser Aufruf allerdings einem bestehenden
Benutzerkonto zugeordnet werden kann (audit/log trail). Bei Fehlern in
der Authentifikation kann jeder anonyme Internetbenutzer auf die
betroffenen Operationen und Daten zugreifen. Dies gilt auch für
automatisierte Scantools, Search Bots und Crawler. Falls bei der
Transportlevel-Sicherheit auch Fehler vorhanden sind, besteht ebenso
großes Risiko durch Man-in-the-Middle Proxies (MitM-Proxies).

### Probleme im Umfeld der Authentication

Das schwerwiegendste Problem wäre es, wenn keine Kontrolle der
Authentication durchgeführt wird. Nach einem Login kann jeder anonyme
Benutzer auf alle Operationen und Daten zugreifen, eine Zuordnung der
Operationsausführung zu einem eingeloggten Benutzer findet nicht statt.
Prinzipiell handelt es sich hierbei um Security-by-Obscurity da die
Sicherheit der Operationen und Daten nur davon abhängt, dass der
Angreifer die URL der Operationen nicht kennt. Dies ist allerdings
selten der Fall, da MitM-Proxies und Crawler zur Identifikation der
Operationen verwendet werden können. Ebenso stellen die meisten
API-Server automatisch generierte Dokumentation der bereitgestellten
Operationen zur Verfügung.

In abgeschwächter Form kann eine ähnliche Schwachstelle teilweise bei
historisch gewachsenen Applikationen gefunden werden. Hier haben sich im
Laufe der Zeit Technologietrends, Firmen-Guidelines oder
Programmierteams verändert und die Gesamtapplikation besteht aus
Komponenten, die in verschiedenen Programmiersprachen/Frameworks
implementiert wurden. Da Authentication-Daten zumeist über das Framework
abgehandelt werden, passiert es hier nun häufig, dass bei einem Teil der
Applikationen die Authentication vergessen wird.

Ein weiteres häufiges Problem sind selbst geschriebene Komponenten.
Ähnlich wie bei dem letzten Fehlerfall wird hier eine bestehende
Applikation um eine weitere Funktion erweitert, auch hier kann dies zeit
verzögert durch neue Programmierer geschehen. Bei den neu geschriebenen
Komponenten wird gerne auf die Authentication vergessen — eine mögliche
Ausrede wäre es, dass externe Programmierer eventuell das bestehende
System nicht gut kennen.

Beispiel: eine Kundenwebseite erlaubt den Download von Rechnungen. Die
gesamte Webseite ist in JSP geschrieben, die Downloadseite allerdings in
ASP.Net. Rechnungen können über die URL /documents/download/123 bezogen
werden. Bei Tests wurde festgestellt, dass über freie Wahl der ID (Zahl)
beliebige Kundenrechnungen heruntergeladen werden konnten, da keine
Authentication implementiert wurde. Bei Analyse der Logdateien wurde
weiters festgestellt, dass die betroffenen Daten bereits vom Google
SearchBot indiziert wurden und somit im Suchindex aufgenommen waren.

Gegenbeispiel: die Webseite eines Personentransportunternehmens
verschickt eine Email mit einen Link auf das gekaufte Ticket. Beim
Ticket-Download wird keine Authentication durchgeführt, Begründung: bei
der Kontrolle gab es immer wieder Probleme, dass Kunden ihr Ticket nicht
herunterladen konnten da sie ihre Zugangsdaten vergessen hatten. Um das
Risiko zu senken wurden als IDs große Zufallszahlen gewählt.

## Reflektionsfragen

1.  Was versteht man unter Multi-Faktor-Authentication?

2.  Wie funktionieren TOTP und FIDO U2F? Worin liegen konzeptionelle
    Unterschiede?

3.  Welche Regeln sollten bei der Speicherung von Passwörtern und zu der
    Sicherung der Qualität der Passwörter beachtet werden?

4.  Was ist der Unterschied zwischen Identification und Authentication?
    Nenne zumindest vier Beispiele wie ein Benutzer identifiziert werden
    kann.

5.  Wie sollte ein Login-Formular gestaltet sein? Von welchen Techniken
    sollte man Abstand nehmen?

6.  Was ist eine User Enumeration und wie kann man sich dagegen
    schützen? Was sind komplexere Applikationsfunktionen die schwer
    gegenüber User Enumeration absicherbar sind?

7.  Was sind Brute-Force Angriffe und wie werden die dabei verwendeten
    Daten erzeugt? Welche Gegenmaßnahmen gibt es?

8.  Auf welche Gefahren sollten bei der Implementierung der
    Passwort-Vergessen Funktion geachtet werden?

[1] <https://pages.nist.gov/800-63-3/>
