# Security Principles

Während sich Technologien und Architekturen permanent wandeln und
verändern, gibt es Sicherheitsprinzipien die quasi allgemein gültig als
Grundlage für alle weiteren Entscheidungen dienen. Einige dieser werden
in diesem Kapitel erläutert.

## Minimalprinzip

Die Applikation sollte nur jene Operationen und Funktionen beinhalten,
die für die Erfüllung der Kundenanforderungen zwingend benötigt werden.
Alle weiteren Funktionen und Operationen sollten deaktiviert bzw.
entfernt werden.

Durch diese Reduktion des Funktionsumfangs wird implizit die
Angriffsfläche verringert und dadurch Angriffe erschwert. *Was nicht
vorhanden ist, kann nicht angegriffen werden*. Zusätzlich wird der
langfristige Wartungsaufwand reduziert.

Die Minimierung kann und sollte an mehreren Stellen durchgeführt werden,
einige Beispiele:

-   Reduktion benötigter Operationen: ist eine Operation wirklich für
    den Kunden notwendig oder könnte der Kundenwunsch mit bereits
    implementierten Operationen ebenso befriedigt werden?

-   Reduktion der gesammelten und gespeicherten Daten: was ist das
    minimale Datenset, dass für die Bereitstellung der Operationen
    benötigt wird. Dies entspricht auch der Datenminimierung die durch
    die DSGVO[1] vorgeschrieben wird. Hier gibt es einen Wandel der
    Kultur: von big-data (alles speichern, vielleicht kann man das
    später verwenden) Richtung toxic-data (Daten sind gefährlich, wie
    komme ich mit möglichst wenig Daten aus).

-   Komponentenebene: welche Komponenten sind für den Betrieb notwendig?

-   Funktionale Ebene: welche Funktionen und Features können innerhalb
    von Komponenten deaktiviert werden?

### Security Misconfiguration

In den OWASP Top 10 kommen häufiger *Security Misconfiguration* als
Beispiel für Verstösse gegen das Minimalprinzip vor.

Wie bereits erwähnt, ist die Grundidee, dass im Produktionsbetrieb nur
Komponenten und Features vorhanden sind, die auch für die Umsetzung
eines Kundenwunsches benötigt werden. Beispiele für Software, die nicht
am Server vorgefunden werden sollte:

-   Entwicklungstools wie phpmyadmin. Diese besitzen meistens getrennte
    Zugangsdaten (verwenden also nicht die Zugangsdaten/Berechtigungen
    der Web-Applikation) und sind daher potentiell ein *alternate
    channel* über den auf eine Webapplikation zugegriffen werden kann.

-   Debug Mode bei verwendeten Frameworks, dieser erlaubt teilweise im
    Fehlerfall die Verwendung von interaktiven Shells direkt innerhalb
    der Webapplikation. Dies würde es einem Angreifer erlauben, direkt
    Programmcode abzusetzen.

-   Debug Toolbars bei Verwendung von Frameworks. Diese erlauben es
    zeitweise die letzten Sessions aller Benutzer anzuzeigen und
    erleichtern auf diese Weise Identity Theft.

-   Stacktraces mit Detailinformationen im Produktivbetrieb. Ein
    normaler Anwendern kann mit diesen Informationen nichts anfangen,
    ein Angreifer kann durch diese allerdings genaue Systeminformationen
    (Bibliotheksversionen, Pfade, etc.) erhalten welche weiter Angriffer
    erleichtern können.

-   phpinfo.php liefert genaue Informationen über die verwendete
    PHP-Version, verfügbare Module, System- und
    Konfigurationsinformationen die im Produktivbetrieb nicht öffentlich
    verfügbar sein müssen.

Beispiele für Metadaten, die nicht am Server vorgefunden werden sollten:

-   Beispielscode wie z.B. ein
    <a href="/example" class="uri">/example</a> Verzeichnis. Dieser kann
    zeitweise ebenso Sicherheitsfehler enthalten und auf diese Weise
    Zugang zu dem System erlauben. Auch Beispielscode ohne serverseitige
    Exekution kann missbraucht werden, siehe z. B. DOM-basierte
    XSS-Angriffe.

-   .git, .svn Verzeichnisse: diese beinhalten den gesamten Source-Code
    samt Versionshistory. Ein Angreifer kann auf diese Weise sowohl
    interne Credentials erhalten als auch den verwendeten Source Code
    analysieren.

-   Credentials im Dateisystem oder in Repositories. Da Repositories
    häufig auf öffentlichen Webservern gespeichert wird (z.B. private
    gitlab, githab oder bitbucket Repositories) gespeichert wird, können
    diese im Falle einer Fehlkonfiguration auch potentiell öffentlich
    zugreifbar gemacht werden. In diesem Fall besitzt ein Angreifer
    credentials mit denen er potentiell auf sensible Aktivitäten oder
    Daten zugreifen kann.

-   Backup files (.bak, .tmp) innerhalb des Dateisystems, diese werden
    z.B. durch Texteditoren angelegt. Wird z.B. auf einem PHP-System
    eine PHP-Datei am Webserver abgelegt und ein Angreifer greift darauf
    zu, wird der Code am Server ausgeführt und der Angreifer erhält nur
    das Ergebnis der Operation. Falls der Angreifer eine Backup-Datei am
    Server findet, kann er auf diese zugreifen, herunterladen und
    analysieren und kann auf diese Weise Fehler innerhalb des Source
    Codes suchen.

## Least Privilege

Jeder Benutzer und jede Funktion sollte nur jene minimalen Rechte und
Privilegien besitzen, die für die Ausführung seiner Ausgabe zwingend
benötigt werden. Jerome Saltzer[2] definierte diesen, als Least
Privilege bekannten, Ansatz als:

> Every program and every priviledged user of the system should operate
> using the least amount of priviledge necessary to complete the job.

Wird dieses Prinzip bereits während des Designs beachtet, führt dies
zumeist zu Systemen, welche aus mehreren Komponenten bestehen. Diese
Komponenten kommunizieren über wohl-definierte Interfaces und können
nicht “direkt” auf die Daten anderer Komponenten zugreifen. Dies
verbessert die Testbarkeit der einzelnen Komponenten, da diese getrennt
voneinander überprüft werden können. Aus Sicherheitssicht ist diese
Architektur ebenso stark zu bevorzugen da eine kompromittierte
Komponente nicht automatisch ein kompromittiertes Gesamtsystem zur Folge
hat.

Um diese Trennung zu ermöglichen, müssen Komponenten mit
unterscheidbaren Identitäten und mit zuweisbaren Ressourcen betrieben
werden. Dies inkludiert sowohl Benutzer- und Zugriffsrechte als auch
Entitlements auf Ressourcen (RAM, CPU, Speicher, Netzwerkbandbreite).
Weiters inkludiert dies Netzwerkzugriffsrechte: die Applikation sollte
nur auf jene remote Server zugreifen können, die auch wirklich zwingend
für den Betrieb notwendig sind.

## Separation of Duties

Separation of Duties besagt, dass zur Ausführung einer Operation die
Zustimmung von mehr als einer Person benötigt wird. Ein klassisches
Beispiel hierfür wäre die Aktivierung eines Atomsprengkopfes für das
mehrere Personen ihre Zustimmung geben müssen. Das Ziel von Separation
of Duties ist auf der einen Seite die Vermeidung von Insider-Threats,
auf der anderen Seite soll dadurch die Entdeckungsrate von
nicht-gewollten Aktivitäten erhöht werden. Grundsätzlich sollte ein
kompromittierter Benutzer nicht die Möglichkeit besitzen, das
Gesamtsystem zu korrumpieren.

Eine Anwendung dieser Idee ist das Vier-Augen-Prinzip bei dem sensible
Operationen vor Ausführung zuerst durch zumindest zwei Personen
bestätigt werden müssen.

Um diese Prinzipien anwenden zu können, müssen Anwender zweifelsfrei
identifiziert, authentifiziert und für die auszuführende Operationen
autorisiert werden. Aus diesem Grund werden
Mehr-Faktor-Authentifizierungslösungen häufig im Umfeld des Separation
of Duties Prinzips gefunden.

## Defense in Depth/Hardening

Das Zwiebelmodel der Sicherheit vergleicht die Gesamtsicherheit einer
Applikation mit einer Zwiebel. Im Inneren der Zwiebel befindet sich das
schützenswerte Gut (Daten, Operationen), rundherum gibt es einzelne
Sicherheitsschichten, analog zu den Schichten einer Zwiebel. Solange
zumindest eine Schutzschicht vorhanden ist, ist die Sicherheit des
Gesamtsystems gewährleistet.

Essentiell ist, dass die einzelnen Schutzschichten voneinander
unabhängig sind. Würde die gleiche Schutzschicht mehrfach verwendet
werden (z.B. zweimal die gleiche Web-Application-Firewall mit dem
identen Regelwerk der Applikation vorgeschalten werden), würde ein
Fehler in einer Schutzschicht automatisch auch den Schutz der zweiten
Schutzschicht neutralisieren.

Zusätzlich zum erhöhten Schutz des schützenswerten Gutes wird durch die
Zwiebelschichten auch Zeit im Fehlerfall erkauft. Da das System noch
nicht vollständig kompromittiert ist, besteht z.B. Zeit die Auswirkungen
eines potentiellen Updates zu testen.

## Fail-Open vs. Fail-Closed

Fail-Open (auch Fail-Safe genannt) und Fail-Close (auch Fail-Secure
genannt) beschreiben das Verhalten eines Systems im Fehlerfall. Bei
Fail-Open wird die Operation durchgeführt, bei Fail-Close wird diese
verhindert.

Die Definition des gewünschten Verhaltens kann nur durch den Kunden
geschehen. Beispiel: ein Smart-Türschloss welches über eine
Mobilapplikation gesteuert werden kann. Das Verhalten im Falle eines
Batteriefehlers kann unterschiedlich implementiert werden. In einigen
Fällen (Notausgang) wäre es sinnvoll, das Schloss zu öffnen; in einigen
Fällen (Tresor) wäre es sinnvoll, das Schloss zu blockieren. Diese
Auswahl kann nur vom Kunden durchgeführt werden.

## No-Go: Security by Obscurity

Die Sicherheit eines Systems darf niemals von dessen Intransparenz
abhängig sein. Ein besserer Ansatz ist z.B. Shannons: *The Enemy Knows
the System*.

Ein motivierter Angreifer besitzt zumeist Möglichkeiten die
Intransparenz zu lüften:

-   Kauf und Reverse-Engineering der Software

-   Diebstahl eines Systems

-   Verlust der Obscurity durch Unfall (z.B. Selfies mit sichtbaren
    Schlüsseln im Hintergrund)

Analog gibt es in der Kryptographie das Kerckhoffsche Prinzip: die
Sicherheit eines Algorithmus darf nur von der Geheimhaltung des
Schlüssels und nicht durch die Geheimhaltung des Algorithmus abhängig
sein. 2020 konnte man die Problematik an Hand des Solarwind-Leaks sehen:
hier konnten Angreifer Zugriff auf Microsofts Quellcode erlangen.
Aktuell (Stand Jänner 2021) wurden hier allerdings keine
Masterpasswörter oder Backdoors bekannt.

## Keep it Simple Stupid (KISS)

*Complexity is the enemy of security*. Ein komplexes System mit vielen
Komponenten bzw. Interaktionen zwischen Komponenten besitzt automatisch
eine größere Angriffsfläche und bieten daher Angreifern mehr
Möglichkeiten.

Man sollte Simplicity nicht mit primitiven Lösungen verwechseln. Der
Grundgedanke stammt von Kelly Johnson der bei Skunk Works Chefingenioer
war, also Leiter jenes Ingenieursteams welches einige der
hoch-technologischsten Aufklärungsflugzeuge des Kalten Krieges entwurf
(U2, SR-71).

## Security-by-Design bzw. Security-by-Default

Analog zu dem in der DSGVO/GDPR verankerten privacy-by-design bzw.
privacy-by-default wird mittlerweile auch gerne von security-by-design
und security-by-default gesprochen. Ersteres bedeutet, dass eine
Software so geschrieben sein sollte, dass ein sicherer Betrieb
prinzipiell möglich ist. Letzteres bedeutet, dass, wenn ein sicherer
Betrieb möglich ist, dieser auch per-default so konfiguriert sein
sollte. Dies soll Sicherheitsfehler aufgrund von “vergessener” bzw.
unterlassender Konfiguration vermeiden, Sicherheitslücken müssen
explizit geöffnet werden.

Beides sind keine direkten Entwicklugsprinzipen aber quasi Vorgaben an
welche sich die Entwicklung halten muss und wurden aus diesem Grund hier
erwähnt.

## Trennung von Daten- und Programm-Logik

Ein typischer Vorfall der zu einer Sicherheitslücke führt ist, wenn ein
Programm bei der Zuordnung zwischen Daten und Programmlogik verwirrt
wird. Eine SQL-Injection fällt in dieses Muster: eine Benutzereingabe
(Daten) wird durch inadäquate Programmierung als Programmlogik bzw.
Programmcode interpretiert und ausgeführt.

Historisch gab es gegenläufige Computerarchitekturen: die
Harvard-Architektur verwendete eine strikte Hardware-Trennung zwischen
Daten- und Programmspeicher. Die konkurrierende Von-Neumann-Architektur
verwendete einen Speicher sowohl für Daten als auch für Programmcode und
setzte sich aufgrund der höheren Effizienz durch. Mittlerweile gibt es
mehrere Sicherheitsvorkehrungen um eine konzeptionelle Trennung von
Programmcode und Daten auch in diesen Architekturen durchzuführen.

## Reflektionsfragen

1.  Erläutere das Minimalprinzip mit zumindest drei Beispielen für
    jenes.

2.  Erläutere Least Privilege und Separation of Duties.

3.  Erläutere Defense in Depth.

4.  Erkläre den Unterschied zwischen Fail-Open und Fail-Closed.

5.  Welche Probleme können durch die Vermischung von Applikationlogik
    und Eingabedaten entstehen?

[1] Datenschutzgrundverordnung, siehe auch
<https://de.wikipedia.org/wiki/Datenschutz-Grundverordnung>

[2] Jerome Saltzer war an der Entwicklung von Multics involviert und
leitete später Projekt Athena am MIT. Dieses Projekt war massgeblich an
der Entwicklung graphischer Oberflächen und Netzwerktechnologien wie
z.B. Kerberos involviert.
