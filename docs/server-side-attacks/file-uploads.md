# File Uploads

Wenn ein Benutzer bei einer Webseite Dateien hochladen und der idente
Benutzer (oder ein anderer Benutzer) danach wieder auf diese Dateien
zugreifen kann, ergeben sich zwei Gefährdungsmomente. Einerseits kann
der idente Benutzer versuchen, mit dem hochgeladenen File den Server
direkt anzugreifen (z.B. um Code am Server auszuführen), auf der anderen
Seite kann ein Angreifer versuchen, auf diese Weise einen anderen
Benutzer anzugreifen (z.B. um dessen Session zu übernehmen).

## Das Upload-Verzeichnis

In einer sicherheitstechnisch guten Webapplikation sind alle Dateien und
Verzeichnisse schreibgeschützt — Angriffe, die serverseitig Dateien
erstellen oder modifizieren müssen, werden dadurch erschwert. Die
einzige Ausnahme sollte das Upload-Verzeichnis sein in welches die
Webapplikation (bzw. der Systemuser der Webapplikation) schreibend
zugreifen darf.

Dieses Verzeichnis sollte niemals unterhalb des Webroots liegen, falls
z.B. der Webroot `/var/www/html` ist, sollte das Uploadsvereichnis sich nicht unter
`/var/www/html/uploads`
befinden. Würde das Verzeichnis so situiert sein, kann der Webserver bei
einem Zugriff auf ein hochgeladenes File schwer unterscheiden, ob eine
hochgeladene Datei zum “normalen” Umfang der Webapplikation gehört, oder
ob es sich um eingeschleusten Schadcode handelt. Zusätzlich sollten
Directory-Listings für dieses Verzeichnis deaktiviert werden und das
Verzeichnis auf einer Partition mit aktivierter *noexec* Mount-Option
(bei Verwendung von Linux) platziert werden.

Der Dateiname, unter dem ein hochgeladenes File abgelegt wird, sollte
niemals durch den User bestimmt werden. Dies würde path traversal
Angriffe erlauben[1] bzw. könnte ein Angreifer den bekannten Pfad zu
einem File im Zuge von weiteren Injection-Angriffen verwenden.

Ein architekturelles Problem ist durch die Struktur von Webapplikationen
bedingt. Eine deployte Webapplikation besteht meistens aus einem
Webserver und einem Applikationsserver. Ersterer ist für die Zustellung
statischer Dateien optimiert, letzterer beinhaltet die Applikationslogik
inkl. der Zugriffskontrollen. Werden Dateien direkt über ein
Upload-Verzeichnis bereitgestellt, übernimmt diese Aufgabe der Webserver
(der für diese statische Zustellung optimiert ist) und nicht der
Applikationsserver — in diesem Fall kann es passieren, dass die
Authentication und Authorization nicht überprüft wird. Um dies zu
vermeiden, sollte ein Download immer mittels einer dezidierten
Downloadoperation, z.B. mittels
`https://example.local/download?file_id=xxx`, durchgeführt, und auf
diese Weise durch den Applikationsserver ausgeführt werden. Dabei
sollten serverseitig die benötigten Zugriffsrechte überprüft werden, als
Id wird die Verwendung einer zufälligen ID wie z.B. einer UUID
empfohlen.

## Upload von Malicious Files

Ein einfacher Angriff ist der der Upload von Dateien, die Code zur
serverseitigen Ausführung beinhalten — z.B. das Hochladen von einer
*.php* Datei bei Verwendung einer PHP Webapplikation. Der Angreifer
würde nach dem Upload auf die Datei zugreifen und dadurch den Code am
Server zur Ausführung bringen. Bei einem File-Upload sollten daher die
möglichen Dateitypen durch eine Whitelist auf Dateitypen, die nicht am
Server exekutiert werden, beschränkt werden. Ebenso sollte mittels dem
*Content Disposition* HTTP Header dem Browser mitgeteilt werden, dass
eine bezogene Datei explizit heruntergeladen sollte (und nicht als Teil
der Webapplikation ausgeführt werden sollte).

Eine weitere Empfehlung ist die Verwendung eines server-seitigen
Virenscanners. Diese arbeiten zumeist auf Dateisystem-Basis — wird ein
File mit bösartigem Code hochgeladen, wird dieses gescannt und
gegebenenfalls unter Quarantäne gestellt bzw. gelöscht. Da die
Webapplikation dies nicht automatisch bemerkt, kann es dabei zu
Inkonsistenzen zwischen dem Dateisystem und verlinkten Dateien in der
Webapplikation kommen. Eine saubere, aber aufwendige, Lösung wäre die
Integration des Virenscanners in den Upload-Prozess der Webapplikation
(z.B. über ein API des Virenscanners). Ein workaround wäre es, falls der
Virenscanner beim Löschen einer Datei eine gleichnamige Datei mit einem
Löschhinweis hinterlegt. Auf diese Werden werden die toten Dateilinks
innerhalb der Applikation vermieden.

Besondere Beachtung sollte der Upload von gepackten Dateien (*zip*,
*rar*) erhalten. Hier muss auf der einen Seite beachtet werden, dass
Archive entpackt und der Inhalt des Archivs ebenso analysiert wird, auf
der anderen Seite muss darauf geachtet werden, dass während des
Entpackvorgangs kein Sicherheitsfehler passiert.

Ein Spezialfall des Uploads von malicious Dateien ist der Upload von
Dateien, die bösartiten JavaScript-Code beinhalten. Da diese Angriffe
gegen andere Clients (zumeist Webbrowser) abzielen, werden diese im
Kapitel *Client-seitige Injection Angriffe* behandelt.

## Sandboxing

Falls eine Webapplikation nicht-vertrauenswürde Dateien verarbeiten
muss, muss diese Dateien aus nicht-vertrauenswürdigen Quellen (Benutzer)
analysieren. Dies ist eine notorisch gefährliche Operation und wird
selten vollkommen sicher implementiert werden können. Um das potentielle
Schadmass zu reduzieren kann Sandboxing verwendet werden. Dabei wird der
Parse-Code in einem abgeschotteten Bereich des Systems ausgeführt, im
Falle eines erfolgreichen Angriffs wird zumindest nicht das Gesamtsystem
kompromittiert.

Wird server-seitig ein Virenscanner verwendet um hochgeladene Dateien
auf Schadcode hin zu überprüfen, sollte dieser ebenso vom Rest des
Systems abgeschottet werden. In den letzten Jahren wurden vermehrt
Angriffe gegen Virenscanner festgestellt. Diese sind für Angreifer sehr
lohnende Ziele, da sie alle incoming Dateien vor gereicht bekommen und
zumeist mit administrativen Rechten ausgestattet sind.

Techniken in diesem Umfeld beinhalten chroots, Jails, Container und
Microservice-Architekturen.
