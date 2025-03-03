# Integration mit der Umgebung

Eine Webapplikation sollte niemals isoliert betrachtet werden. Auch wenn
die vorgestellten Komponenten perfekt umgesetzt werden, ergibt sich aus
der Interaktion zwischen der theoretischen Webapplikation und der realen
Umgebung immer ein Gefahren- bzw. Verbesserungspotential.

Hierbei kann es sich z.B. um nicht-funktionale Elemente wie Logging
handeln, oder aber auch um Aspekte wie Programmiersprache-inhärente
Muster, die einen negativen Einfluss auf die Sicherheit der
Webapplikation besitzen können.

## Using Components with Known Vulnerabilities

Die Verwendung vorhandener und gewarteter externer Komponenten wie
Bibliotheken oder Frameworks besitzt sicherheitstechnisch viele
Vorteile: man kann von Fehlern anderen lernen bzw. muss das Rad nicht
neu erfinden.

Damit wird allerdings auch der Nachteil eingekauft, dass eine
Verwundbarkeit innerhalb einer integrierten externen Komponente
automatisch auch eine Verwundbarkeit der eigenen Applikation impliziert.
Daher müssen aktiv und regelmäßig verwendete Komponenten auf bekannte
Schwachstellen hin überprüft, und ggf. die betroffenen Komponenten
aktualisiert werden. Falls über eine externe Komponente eine
Schwachstelle ,,eingefangen” wird, wird dies *Supply-Chain Attack*
genannt: der Angriff erfolgt nicht direkt gegen die Applikation selbst,
sondern über die inkludierten Komponenten, quasi dem Zulieferer (engl.
Supply Chain).

Um den Aufwand dieser Überprüfungen zu reduzieren und damit
optimaler-weise deren Frequenz zu erhöhen gibt es automatisierte Tools
wie den [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/) oder *OWASP Dependency-Track*. Diese
analysieren automatisch Projekte auf Dependencies (z.B. über Ruby
Gemfiles, NPM package-lock.json Files, Maven Projektbeschreibungen),
extrahieren automatisch dependencies und korrelieren diese mit
öffentlichen Verwundbarkeitsdatenbanken. Wird hier nun eine potentiell
anwendbare Schwachstelle gefunden, wird der Entwickler via Email oder
Slack notifiziert.

### Typo-Squatting und Dependency Confusion

Bei *Supply-Chain Angriffen* plazieren Angreifer Schadcode zumeist in
neuen Versionen bereits verwendeter Bibliotheken. Der Angreifer muss
also Zugriff auf den Source Code einer häufig verwendeten Bibliothek
erlangen. Die Geschichte zeigt, dass dies durchaus einfacher als gedacht
ist[2].

Ein Angreifer kann Supply-Chain Angriffe auch ohne Erlangen eines
bestehenden Projektes durchführen. Bei *Typo-Squatting* Angriffen
kopiert der Angreifer eine bestehende Bibliothek, reichert diese um
Schadcode an, und läd diese unter einem neuen Namen hoch. Der neue Name
ist sehr ähnlich dem ursprünglichen Namen der Bibliothek gewählt, der
Angriff zielt darauf ab, dass das Opfer beim Integrieren der eigentlich
gewünschten Bibliothek sich vertut oder vertippt und dadurch die falsche
Bibliothek samt Schadcode inkludiert.

Ein weiterer Angriffsvektor ist *Dependency Confusion*[3]. Hier
entwickelt das Opfer eine Software, die sowohl öffentliche als auch
interne (private) BBibliotheken verwendet. Der Angreifer kennt den Namen
einer internen Bibliothek und läd eine Schadsoftware mit diesem Namen
auf ein öffentliches Paketverzeichnis (welches von der Opfersoftware
verwendet wird) hoch. In Abhängigkeit von der verwendeten
Programmiersprache, der verwendeten Paketverwaltung und dessen
Konfiguration können zwei potentielle Schwachstellen auftreten:

1.  Öffentliche Quelle werden gegenüber privaten Quellen bevorzugt: es
    wrid nun der Schadcode aus dem öffentlichen Repository geladen.

2.  Bei der Auswahl der Bibliothek aus internen und öffentlichen Quellen
    ,,gewinnt” das Paket mit der höchsten Version. In diesem Fall muss
    der Angreier nur ein Paket mit einer höheren Versionnummer in das
    öffentliche Repository hochladen um seinen Schadcode in die
    Opfersoftware zu integrieren.

## Insufficient Logging and Monitoring

Diese Schwachstelle wurde im Jahre 2017 neu bei den OWASP Top 10
aufgenommen. Es handelt sich hierbei weniger um eine Schwachstelle
während der Exekution, sondern eher um die Schaffung der Möglichkeit
nach einem Angriff aufgrund der vorhandenen Log-Dateien das Vorgehen des
Angreifers und die betroffenen Daten zu erkennen.

Folgende groben Anforderungen an das Log-System werden gestellt:

-   Es muss mit verteilten Applikationen umgehen können. Eine
    Webapplikation ist potentiell auf mehrere Computersysteme verteilt
    (Webserver, Applikationsserver, Datenbankserver). Die Logdaten der
    gesamten Systeme sollten an einer Stelle aggregiert werden.

-   Es muss die Integrität der Logdaten schützen: ein Angreifer sollte
    keine Möglichkeit besitzen, die geloggten Daten zu beeinflussen.
    Würden z.B. Logdaten direkt am Webserver gespeichert werden, könnte
    ein Angreifer der den Webserver gehackt hat, ebenso die Logdaten
    modifizieren. Dies impliziert, dass der Log-Server über eine genau
    definierte API erreichbar sein sollte.

-   Es muss die Vertraulichkeit der Daten schützen. Da der Logserver
    Detailinformationen über betriebliche Abläufe speichert, müssen
    diese Daten mindestens ebenso sicher wie die ursprünglichen Daten
    gespeichert werden.

-   Das Log-System muss Möglichkeiten zur nachträglichen Auswertung der
    gesammelten Daten bieten. Bonuspunkte, wenn man ein automatisiertes
    Monitoring mit dem Log-System betreiben kann.

Die jeweiligen loggenden Systeme sollten alle sicherheitsrelevanten
Events (z.B. Input Validation Fehler, Authentication Fehler,
Authorization Fehler, Applikations-Fehler) an das zentrale Log-System
schicken. Diese Daten sollen um Business Process Events angereichert
werden. Diese dienen dazu, relevante geschäfts-relevante Prozesse und
Ereignisse mit den Sicherheits-Events zu korrelieren. Weitere
Datenquellen sind z.B. Anti-Automatisierungssysteme, welche Brute-Force
Angriffe erkennen, Datenverarbeitungssysteme (können auch Batch-Systeme
sein, die z.B. einen Daten-Export oder Backups ausführen) und alle
direkt und indirekt involvierten Services, wie z.B. Mailserver,
Datenbankserver, Backupdienste. Falls vorhanden, sollten die
Loginformationen sicherheitsrelevanter Komponenten (HIDS, NIDS, WAFs)
auf jeden Fall inkludiert werden.

Bei dem Loggen sollte darauf geachtet werden, dass, wenn möglich,
standardisierte Log-Formate wie CEF oder LEEF verwendet werden. Dadurch
wird das Konvertieren der jeweiligen Datenquellen auf ein gemeinsames
Format vermieden.

Welche Daten sollten pro Event erfasst werden?

-   Wann hat sich der Vorfall ereignet? Bei einer verteilten Applikation
    sollte hier darauf geachtet werden, dass Timestamps die Zeitzone
    beinhalten (und auch auf Zeitumstellungen achten). Grundlage für das
    temporale korrelieren von Events ist es, dass alle beteiligten
    Server eine idente Systemzeit besitzen (z.B. durch die Verwendung
    von ntp).

-   Wo ist das Event passiert? Hierfür können Systemnamen, Servicenamen,
    Containernamen oder Applikationsnamen verwendet werden.

-   Für welchen Benutzer ist das Event passiert? Hier können
    Systembenutzer (mit denen das Service läuft) oder feingranular der
    gerade eingeloggte Benutzer protokolliert werden.

-   Was ist passiert? Dies wird immer applikations- und event-spezifisch
    sein. Viele Systeme verwenden zumindest eine idente Klassifizierung
    der Wichtigkeit des Events.

Die Verwendung von personenbezogenen Daten kann das Logging
verkomplizieren. Ein Unternehmen sollte klare Regeln erstellen, welche
Daten geloggt werden und, falls notwendig, Anonymisierung oder
Pseudonymisierung verwenden um sensible Daten zu maskieren. Ein
ähnliches Problem tritt auf, wenn Log-Informationen zwischen Unternehmen
geteilt werden sollte (z.B. im Zuge eines Informations-Lagebilds). Da
diese Daten unter anderem personen-bezogene Informationen als auch
Betriebsgeheimnisse inkludieren können, wird davon meistens abgesehen.

Die erfassten Daten sollten im Zuge einer Auswertung verwendet werden.
Hier werden häufig "normale" Texteditoren in Verbindung mit regulären
Ausdrücken verwendet. Fortgeschrittene Lösungen wären ELK-Stacks, Kibana
und Logstash und z.B. Splunk.

In einem ähnlichem Umfeld arbeiten SIEM-Systeme (Security Information
and Event Management). Diese werden zumeist als weiterer Schritt nach
Log-Management angesehen. Zusätzlich zum Log-Management wird zumeist
auch Security Event Management (real-time monitoring), Security
Information Management (long-term storage of security events) und
Security Event Correlattion durchgeführt.

## DevOps und Tooling

DevOps ist eine neuere Strömung die versucht, Development und Operations
zu vereinen.

Webapplikationen werden zumeist von Entwickler erstellt und dann einem
Administratoren-Team zur Installation übergeben. Teilweise wird die
Applikation auch von den Entwicklern installiert und dann von den
Administratoren langfristig gewartet. In größeren Unternehmen wird die
Installation und Wartung teilweise auf zwei unterschiedliche
Administratorenteams aufgeteilt.

Dies führt zu getrennten Teams mit getrennten Wissensstand und kann im
worst-case auch z.B.nkerdenken — *,,us vs. them”*— führen. Diese
Trennung behindert den Informationsfluss und verhindert, und führt
künstliche Schranken im Verantwortlichkeitsgefühl ein (,,die Admins sind
dafür verantwortlich”). Im Fehlerfall führt dieses Bunkerdenken auch zum
Herum schieben der Verantwortung zwischen Parteien.

DevOps versucht nun, wie der Name schon sagt, die Trennung von
Entwicklung (,,Development”) und Administration (,,Operations”) zu
beenden. Prinzipiell ist DevOps mehr eine Philosophie/gelebte
Firmenkultur die stark von der Kultur der kontinuierlichen Verbesserung
(z.B. Kanban in Japan) geprägt ist. Bei der Umsetzung bindet es stärker
Entwickler in klassische Operations-Bereiche wie Deployment ein.

Eine gute Beschreibung ist, dass DevOps agile Entwicklungsmethoden mit
agilen Deployment kombiniert.

### Agile Methoden

Agile Methoden sind ein neueres Projektmanagement-Muster, welches im
Agile Manifesto[4] folgende Grundsätze definiert:

-   Individuals and interactions over processes and tools

-   Working Software over comprehensive documentation

-   Customer collaboration over contract negotiation

-   Responding to change over following a plan

Umgesetzt führt dies zumeist dazu, dass monolithische Projekte in kleine
minimale Teile transformiert werden. Diese werden dann, in Reihenfolge
der Kundenprioitisierung, abgearbeitet und regelmäßig der Fortschritt
mit dem Kunden besprochen. Im Zuge des Projektes kommt es häufiger zu
Änderungswünschen, diese können dann als weiteres Teilprojekt/Schritt
inkludiert werden, die Geschwindigkeit des Teams kann über Projektdauer
immer genauer eingeschätzt werden.

Damit die Projektsteuerung bei agiler Methodik funktioniert, darf ein
einmal erledigter Schritt/Problem nicht immer wieder (durch Bugs) kosten
verursachen. Aus diesem Grund wird hier stark auf automatisierte Tests
gesetzt. Sofern diese Tests erfolgreich durchlaufen wird davon
ausgegangen, dass das Produkt funktioniert. Der *master* oder
*production*-Branch der Software sollte niemals fehlerhafte Testcases
besitzen, kann daher jederzeit an den Kunden ausgeliefert werden.

#### Anwendbarkeit agiler Methoden

Agile Methoden sind natürlich nicht für alle Projekte geeignet und
werden eher bei Startup-Projekten bzw. explorativen Projekten
angetroffen. Bei Anwendungen mit hoher Sicherheitsrelevanz gibt es
Zeitweise sehr genau ausspezifizierte Lasten-/Pflichtenhefte, diese
müssen dann auch dementsprechend umgesetzt werden.

### Infrastructure as Code

Ein Grundsatz Agiler Methoden ist ,,*Working Software over comprehensive
documentation*”.

Der Fokus auf *Working Software* anstatt auf Dokumentation schlägt sich
auch bei dem Deployment (dem Installieren der Software) nieder: dies
wird zumeist automatisiert als Skript durchgeführt und nicht als
Dokumentation ausgeliefert (und entspricht dadurch bereits dem
DevOps-Gedanken).

Da im Zuge von Agilen Methoden versucht wird möglichst früh und
möglichst häufig lauffähigen Code beim Kunden bereitzustellen (bzw. als
Testservice dem Kunden zur Verfügung zu stellen) passieren
Installationsvorgänge regelmäßig. Um hier nun Redundanzen zu vermeiden
(bzw. um Konfigurationsfehler zu verhindern) wurden hier (historisch
betrachtet) Installationsanweisungen immer stärker durch automatisierte
Skripts ersetzt. Danach wurden dezidierte Deploymentstools (wie z.B.
*capistrano*) für das Setup der Applikation konfiguriert und verwendet.
Im Laufe der Zeit wurden diese Tools nicht nur für die Applikation
selbst, sondern auch für Datenbanken, Systemservices, etc. angewandt;
die historische Evolution sind mittlerweile dezidierte Frameworks die
zum Setup der Systeme dienen (wie z.B. Puppet, Chef oder Ansible).

Ein weiterer Vorteil dieses Ansatz ist, dass die verwendete
Konfiguration innerhalb der (hoffentlich) verwendeten Source Code
Versionierung automatisch versioniert und Veränderungen dokumentiert
werden. Dies erlaubt das einfachere Debuggen von Regressionen.

### Continuous Integration and Continuous Delivery

Die Verwendung von Tests zur Sicherung der Softwarequalität ist ein
essentieller Bestandteil Agiler Methoden. Dem Kunden werden regelmäßig
neue Programmversionen mit erweiterten Features zugestellt und anhand
des Kundenwunsches neue Features selektiert und im nächsten
Programmier-Sprint hinzugefügt. Würden Features fertig gestellt werden,
die Fehler beinhalten und müsste man nachträglich diese Fehler immer
wieder neu korrigieren, würde dadurch die Geschwindigkeit des
Programmierteams stark leiden. Um dies zu verhindern, werden massiv
Softwaretests geschrieben, die überprüfen ob die gewünschten
Kundenfeatures ausreichend implementiert wurden. Diese Tests werden
aufgerufen, bevor ein neues Feature in die, dem Kunden übermittelten,
Version integriert wird. Auf diese Weise wird automatisch eine Kontrolle
der Qualität durchgeführt.

Um sicher zu stellen, dass diese Tests auch wirklich aufgerufen werden,
werden diese Tests automatisiert aufgerufen. Dies wird im Zuge des
Continuous Integration Prozess durchgeführt: nach jeder Änderung wird
versucht, die Software zu bauen (builden) und anschließend werden die
vorhandenen Tests ausgeführt. Falls ein Fehler auftritt, wird der
betroffene Entwickler sofort notifiziert.

Während Unit-Tests primär auf das Testen von Funktionen abzielen, werden
auch statische Source Code Tests integriert. Diese überprüfen die
Qualität des gelieferten Codes (z.B. Coding Guidelines) und sind n
zweites Standbein automatisierter Tests.

In einem finalen Schritt kann auch Continuous Delivery angewandt werden,
dies ist quasi die Ausdehnung des Continuous Integration Prozesses auf
das installieren der fertigen Software (in einer Testumgebung oder, im
Extremfall, direkt beim Kunden). Hier kann nach erfolgtem Bauen und
Testen der Software diese auf Knopfdruck (oder vollautomatisiert) in
einem Test- bzw. Produktivsystem eingespielt werden. Der Fluss von der
Entwicklung über die Kontrolle bis zur Installation ist somit vollzogen.
Falls dies alles durch den Entwickler konfiguriert wurde, kommt kein
klassischer System-Administrator im Prozess vor. Damit gibt es keine
Teilung der Kompetenzen und Verantwortung mehr, der Schritt zu DevOps
ist vollzogen.

## DevOps and Security

Die Abkürzung DevOps besteht aus Development und Operations, das Wort
Security kommt dabei nicht vor.

Sicherheit in DevOps zu integrieren ist ein ähnliches Problemfeld wie
das ursprüngliche DevOps-Problem der Trennung zwischen Admins und
Entwicklern. Wenn die Sicherheitsverantwortlichen ein getrenntes Team
sind, dann ergibt sich wieder Bunkerdenken, Wissensinseln als auch ein
fehlendes Verantwortungsgefühl.

Es wird nun Versucht das Sicherheitsteam in das DevOps-Team zu
integrieren und dadurch Security als gemeinsame Verantwortung zu
etablieren. Die Grundidee ist schön in folgendem Satz ausgeführt:
,,*Security is built into the System instead of being applied upon the
finished product*”.

Häufiger wird zwischen verschiedenen Ansätzen um Security einzubauen
unterschieden:

-   DevOpsSec: es wird ein Produkt entwickelt, danach wird die
    Administration durchgeführt. Final wird Security gewährleistet: ein
    Beispiel dafür wäre es, dass nach Inbetriebnahme das Security-Team
    Security-Patches einspielt. Das klassische Beispiel wäre das
    Anpassen und der Betrieb von Standardsoftware.

-   DevSecOps: zuerst wird entwickelt, danach wird Security betrachtet
    und danach die Administration fortgesetzt. Dieser Ansatz wird
    aktuell (2019) häufig gesehen und ist zumindest besser als die
    Security generell zu ignorieren. Ein Beispiel hierfür wäre es, die
    Inbetriebnahme von der erfolgreichen Durchführung einer
    Sicherheitsüberprüfung abhängig zu machen.

-   SecDevOps: betrachtet initial die Security (z.B. schon während der
    Planung der Software). Dadurch durchdringt Security die gesamte
    Entwicklung als auch die Administration.

Security bewirkt meistens einen Mehraufwand für die Entwickler. Um
diesen Mehraufwand zu begrenzen, wird auch hier (im DevOps-Spirit) stark
auf Automatisierung gesetzt. So werden z.B. Sicherheitstests als
automatisierte Testprogramme implementiert und während der Testphase
innerhalb des Continuous Integration Prozesses ausgeführt. Dadurch
werden die Sicherheitstests automatisch Teil des
Abnahme-/Verifikationsprozess und durchdringt auf diese Weise die
gesamte Entwicklung.

### Automatisierung

Um die konsistente und regelmäßige Verwendung der Sicherheitstests zu
enforcen wird die Ausführung der Sicherheitstests stark automatisiert.
Dieses Kapitel führt häufig verwendete automatisierte Tests an:

#### Automated Unit Tests

Unit Tests sind minimale Tests die ein Feature verifizieren. Die meisten
Software-Frameworks erlauben es, Tests auf Controller-Ebene zu schreiben
für deren Ausführung die Applikation nicht als gesamtes gestartet werden
muss.

Alternativ kann ein Unit-Test z.B. als einfaches Shellskript,
Python-Requests2 skript oder als JUnit-Test unter Verwendung von
Java-Bibliotheken durchgeführt werden. In dem Fall muss im Zuge der
Tests ein Webserver gestartet werden, diese Tests sind daher
zeitaufwendiger.

Integrations-Tests testen die Funktionalität des Gesamtsystems. Hierbei
kann unter anderem auf Browser-basierte Frameworks wie Selenium oder
Capybara zurückgegriffen werden. Durch diese wird ein oder mehrere
Benutzer mittels eines virtuellen Browsers simuliert — die Tests
beschreiben die Benutzernavigation und -operationen innerhalb der
Webseite. Hier kann man z.B. zwei Benutzer simulieren: ein Benutzer legt
Daten an und ein zweiter Benutzer versucht (invalid) auf diese Daten
zuzugreifen. Diese Tests sind um einiges Ressourcen- (da ein Webbrowser
und Webserver benötigt wird) als auch zeitaufwendiger und werden daher
zumeist nicht nach jeder Sourcecode-Änderung durchgeführt.

#### Static Source Code Tests

Analog zur Analyse der Qualität des Source Codes (während des normalen
DevOps-Prozesses) können auch Tools zur statischen Analyse des Source
Codes inkludiert werden. Diese prüfen den Source Code auf bekannte
Programmiermuster und -fehler die sicherheitsrelevante Konsequenzen
besitzen können.

Hier gibt es meistens Programmier- und Framework-abhängige Tools wie
z.B. *bandit* für *Python*, *Brakeman* für *Ruby on Rails* und
*SpotBugs* für *Java*. Eine Ebene über diesen Einzeltools funktioniert
*OWASP SonarCube*. Dieses Tool kann intern die gesamten erwähnten
Subtools anwenden und besitzt auch Plugins um mittels *OWASP
dependency-check* eine Überprüfung der verwendeten Abhängigkeiten (z.B.
Bibliotheken) auf Schadcode hin durchzuführen.

#### Dynamic Application Scans

Zusätzlich zur statischen Source-Code Analyse kann man auch dynamische
Scans verwenden. Hierbei wird zumeist die Applikation in einer
Testumgebung (z.B. *staging*) automatisiert installiert und danach
mittels automatisierter Web Application Security Scanner gescripted ein
Test durchgeführt. Im Falle einer Regression werden die Entwickler
benachrichtigt. Beispiele hierfür wäre z.B. das automatisierte Scannen
einer Applikation mittels *OWASP ZAP* unter Zuhilfename des
*full-scan.py*-Skripts.

## Reflektionsfragen

1.  Was ist der Grundgedanke dabei, DevOps und Security zu verbinden?

2.  Welche Sicherheitsmaßnahmen können im Zuge von SecDevOps
    automatisiert durchgeführt werden? Erläutere die jeweiligen
    Maßnahmen.

3.  Wie kann während der Continuous Integration (CI) oder Continuous
    Delivery (CD) auf die Sicherheit eines Softwareprodukts Rücksicht
    genommen werden?

4.  Was sind Supply-Chain Angriffe und wie können diese geschehen?

[1] <https://jeremylong.github.io/DependencyCheck/analyzers/index.html>

[2] <https://www.fireeye.com/blog/threat-research/2020/12/evasive-attacker-leverages-solarwinds-supply-chain-compromises-with-sunburst-backdoor.html>

[3] <https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610>

[4] <https://agilemanifesto.org/>
