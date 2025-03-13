# Datenbank-Injections

Datenbank-Injections gehören zu den selteneren, dafür aber
schwerwiegenderen, vorkommenden Sicherheitsfehlern. Das Grundproblem
ist, dass Datenbankabfragen unter Zuhilfename von Benutzereingaben
gebaut werden. Durch bösartige Benutzereingaben versuchen Angreifer nun,
das Datenbanksystem zur Freigabe zusätzlicher Daten zu bringen,
unbeabsichtigt Daten zu verändern oder sogar aus dem Datenbanksystem auf
das Betriebssystem auszubrechen.

## SQL

SQL[2] ist die bekannteste Abfragesprache für relationale Datenbanken.
Im Zuge dieser Vorlesung werden nur einfache SQL-Features benötigt. Ein
Beispiel für ein einfaches SQL Statement:

```sql
select column1, column2 from table1, table2
where column1 = 'xyz'
order by column1 asc/desc
limit 1;
```

In diesem Fall werden zwei Spalten *column1* und *column2* aus zwei
Tabellen *table1* und *table2* ausgelesen. Mittels der where-Klausel
wird eine Bedingung zur Filterung der Daten hinzugefügt, mittels *order
by* die Daten entweder aufsteigend oder absteigend sortiert und mittels
*limit* die Anzahl der Datensätze auf einen Datensatz limitiert.

SQL bietet die Möglichkeit die Ausgaben zweier Queries zu einer
Gesamtausgabe zu kombinieren. Hierfür wird das *UNION* Kommando
verwendet:

```sql
select column1, column2 from table1, table2
union all
select column3, column4 from table3, table4;
```

Dies ist nur möglich, wenn beide verwendeten SQL-Queries die idente
Anzahl von Spalten zurück liefern.

## Arten von SQL-Injections

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

```java
String query = "select * from users where email = '" +email+ "' and password = '" +password +"' limit 1;";
```

Die Email-Adresse und das Passwort wird als Teil der Datenbank-Abfrage
verwendet, wird ein Datensatz zurückgegeben wird der erste Datensatz
vermutlich zur Befüllung der Benutzersession verwendet. Wird kein
Datensatz zurückgegeben nimmt die Applikation an, dass der Login nicht
erfolgreich war.

Ein Angreifer würde nun z.B. folgendes Fragment als Passwort übergeben:

```text
1' or '1'='1
```

Durch diesen Ausdruck würde folgendes SQL-Kommando entstehen:

```java
String query = "select * from users where email = 'ah@coretec.at' and password = '1' or '1'='1' limit 1;";
```

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

### Stacked Queries

Die grundsätzliche Methode an eine bestehende SQL-Abfrage zusätzliche
(ungewollte) Queries anzuhängen und dadurch Code auszuführen wird
Stacked Query genannt. Das klassische Beispiel für eine solche ist:

```text
'; drop table users; --
```

Mittels des ersten Zeichens `'` wird versucht aus dem vorgesehenen
SQL-Ausdruck auszubrechen. Das Semikolon dient zum Beenden des
eigentlichen Kommandos und der Angreifer kann ein beliebiges
SQL-Kommando anhängen — in diesem Fall ein *drop table* Kommando,
welches eine Datenbank löschen würde. Zum Schluss wird mit einem
weiteren Semikolon der eingeschleuste Befehlt beendet und durch die
beiden Bindestriche ein Kommentar eingeleitet. Auf diese Weise wird
potentiell nachfolgender SQL-Code auskommentiert.

### UNION-based SQL-Injection

Bei UNION-basierten SQL-Angriffen wird versucht mittels des *UNION*
Kommandos ein zusätzliches *SELECT* Statement an ein bestehendes
Select-Statement anzuhängen. Häufig wird dies verwendet, wenn eine
Web-Applikation eine Tabellen-ähnliche Datenauflistung bietet.

Beispiel: eine Webapplikation stellt eine Liste von Personen als
HTML-Tabelle dar. Ein Benutzer kann diese Liste durch Eingabe einer ID
einschränken. Es wird daher angenommen, dass die Daten der dargestellten
HTML-Tabelle durch eine SQL-Abfrage der Form:

```sql
SELECT Name, Phone, Address FROM Users WHERE Id=$id
```

bereitgestellt wird. Der Parameter *$id* wird durch den Benutzer
bereitgestellt. Ein Angreifer kann nun versuchen, hier eine
SQL-Injection durchzuführen. Beispielsweise könnte dafür folgendes
Fragment verwendet werden:

```text
1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable
```

Dieses Fragment wird durch die Webapplikation für *$id* eingesetzt (da
ID in diesem Fall ein Zahlenwert ist, muss, verglichen mit dem Ausbruch
aus einem String-Wert, werden hier keine Quoting-Zeichen wie ’ benötigt)
und erzeugt auf diese Weise die folgende SQL-Abfrage:

```sql
SELECT Name, Phone, Address FROM Users WHERE Id=1
UNION ALL
SELECT creditCardNumber,1,1 FROM CreditCardTable
```

Die Tabelle wird nun initial mit den Daten des Users mit der Id 1
befüllt, zusätzlich werden alle Kreditkartennummern der Tabelle
CreditCardTable angehängt (bei diesen Daten werden Spalten 2 und 3 mit
der Konstanten 1 gefüllt).

Da bei einem UNION-Select die Spaltenanzahl der jeweiligen Queries ident
sein muss, muss der Angreifer initial die richtige Spaltenanzahl
erraten. Dies wird zumeist über Brute-Force Angriffe durchgeführt.

### Boolean-based Blind SQL Injection

Eine SQL-Injection ist auch ohne direkten Antwortkanal möglich. Ein
Beispiel hierfür sind Boolean-based blind SQL-Injections.

Ein Beispiel: gegeben eine Produktseite `opfer.local/product/1`
die ein Produkt anzeigt. Der Angreifer hat bereits erkannt, dass bei
Eingabe von `opfer.local/product/1 and 1=1` die Produktseite ebenso
angezeigt wird und bei `opfer.local/product/1 and 1=0` kein Produkt gefunden
wird. Dadurch besteht die Annahme, dass der Angreifer einen Ausdruck and
die Produkt-Id (1) anhängen kann und dass dieser Ausdruck auch
exekutiert wird (der Ausdruck 1=0 ergibt immer *false*, durch die
Und-Verknüpfung mit *false* wird kein Produkt mehr geliefert). Dies kann
nun ausgenutzt werden, um mit einzelnen Abfragen den Datenbankinhalt
auszulesen. Beispielsweise kann der Angreifer folgenden Ausdruck bilden:

```sql
SELECT field1, field2, field3 FROM Users WHERE Id='1' AND ASCII(SUBSTRING(username,1,1))=97
```

An den Suchausdruck wird also eine Substring-Abfrage hinzugefügt. Diese
extrahiert die erste Stelle des Benutzernamens, verwandelt diese über
die *ASCII*-Funktion in einen ASCII-Wert und überprüft, ob die erste
Stelle des Benutzernamens ein A ist. Wird nun die Produktseite des
Produkts 1 zurückgeliefert, weiß der Angreifer, dass das erste Zeichen
des Benutzernamens ein A ist. Wird keine Produktseite geliefert, würde
der Angreifer versuchen ob der ASCII Wert dem Zeichen B entspricht.
Durch mehrere (tausende) Anfragen kann der Angreifer auf diese Weise die
gesamte Datenbank rekonstruieren.

### Time-based Blind SQL Injection

Ähnlich wie bei einer boolean based blind SQL-Injection gibt es bei
dieser Angriffsart keinen direkten Antwortkanal für die extrahierten
Informationen. Anstatt wird ein side-channel Angriff auf das
Zeitverhalten der Antwort angewandt.

Der Angreifer besitzt die Möglichkeit ein SQL-Fragment an eine Anfrage
anzuhängen und zur Exekution zu bringen. Wieder wird eine IF-Abfrage
verwendet, in dem konkreten Fall wird, falls die Abfrage erfolgreich
ist, die Antwort um 10 Sekunden verzögert:

```url
http://www.examplecom/product.php?id=10 AND IF(ASCII(SUBSTRING(username,1,1))=97, sleep(10), ‘false’))--
```

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

### Error-based Injections

Bei *Error-based Injections* wird absichtlich ein Fehler eingebaut um
über den ausgegebenen Fehlertext Informationen zu extrahieren.

Ein Beispiel in MySQL: es gibt in Mysql die mathematische Funktion *exp*
welche ab einem übergebenen Dezimalwert von ca. 260 einen Fehler
ausgibt. Ebenso gibt es den Operator ‘w̃elcher ein Bitweises Kompliment
bildet. Wird dieser Operator auf das Ergebnis eines Selects angewandt,
ist das Ergebnis eine sehr große Zahl.

Ein Angreifer kann dieses Verhalten für eine Datenextraktion nutzen,
z.B.:

```text
exp(~(select * from (select user()) x)
```

Es wird also in einem sub-select die Funktion *user()* aufgerufen, die
den aktuellen Benutzernamen zurück gibt. Auf dieses Ergebnis wird ein
bitweises Kompliment angewandt, es wird eine große Zahl erzeugt; diese
Zahl wird dann an die *exp*-Funktion übergeben und wird einen Fehler
werfen.

Die generierte Fehlermeldung:

```text
mysql>select exp(~(select * from (select user()) x ));
ERROR 1690(22003): DOUBLE value is out of range in'exp(~((select 'root@localhost' from dual)))'
```

In der Fehlermeldung wurde allerdings der innere SQL-Ausdruck
exekutiert, dadurch wird der Benutzername *root@localhost* ausgegeben
und eine Datenextraktion ist erfolgt.

Dies ist ein weiterer Grund, warum auf einer Webseite keine
detaillierten Fehlermeldungen ausgegeben werden sollten.

## Ausbruch aus dem Datenbanksystem

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

```sql
postgres-# CREATE TABLE temp(t TEXT);
postgres-# COPY temp FROM '/etc/passwd';
postgres-# SELECT * FROM temp limit 1 offset 0;
```

MySQL bietet auch die beiden Zusätze *into outfile* bzw. *into dumpfile*
an. Damit wird das Resultat einer SQL-Query in eine Datei gespeichert.
Falls der Datenbankserver mit einer hohen Berechtigunggstufe läuft (z.B.
als *root* oder *www-data* Benutzer) kann dies verwendet werden um
Dateien im Filesystem (z.B. im Web-Root) abzulegen und auf diese Weise
eine Webshell hochzuladen (diese würde dann durch den Angreifer über den
Webserver geöffnet werden).

## Gegenmaßnahmen

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

```java
String custname = request.getParameter("customerName");
String query = "SELECT account_balance FROM user_data WHERE user_name = ?";

PreparedStatement pstmt = connection.prepareStatement(query);
pstmt.setString(1, custname);

ResultSet results = pstmt.executeQuery();
```

Die dynamische SQL-Query befindet sich im String *query* und beinhaltet
einen dynamischen Parameter der mit einem *?* markiert wird. Durch die
Methode *setString* wird nun der 1te Parameter auf den Wert der Variable
*custname* gesetzt und auf diese Weise die Benutzereingabe in einer
sicheren Art und Weise in die SQL-Query eingebaut.

Ein weiteres Beispiel in PHP unter Verwendung von PDOs:

```php
$id = 1;
$sth = $DBH->prepare("SELECT * FROM juegos WHERE id = :id");
$sth->bindParam(':id', $id, PDO::PARAM_INT);
$STH->execute();
```

Bei diesem Beispiel werden die dynamisch inkludierten Daten mittels
eines Platzhalters (*:id*) identifiziert und mittels der Methode
*bindParam* gesetzt. Diese Art der Zuweisung hat den Vorteil, dass *:id*
innerhalb der Query an mehreren Stellen gesetzt werden kann. Ebenso wird
durch das Hinzufügen eines weiteren dynamischen Parameters die Position
der dynamischen Parameter nicht verändert[3].

Ein Problem mit prepared statements ist, dass nicht alle Elemente einer
SQL-Abfrage auf diese Weise dynamisch befüllt werden können. Häufige
Ausnahmen sind:

- Tabellennamen
- Spaltennamen
- die Sortierrichtung (*ASC*, *DESC*)

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

## Object-Relational Mapping

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

```javascript
models.Items.findAll({
    limit: '1',
})
```

In dem Beispiel wird ein Objekt des Typs Items erstellt. Problematisch
bei ORMs ist, dass im Hintergrund zumeist SQL-Kommandos erstellt werden
und daher SQL-Injections weiterhin möglich sind, hier ein Beispiel:

```javascript
models.Items.findAll({
  limit: '1; DELETE FROM Items WHERE 1=1; --',
})
```

An den Limit-Parameter wird eine Stacked-Query angehängt und auf diese
Weise eine SQL-Injection ausgeführt. Anhand diese Beispiels kann erkannt
werden, das ORMs kein Allheilmittel für SQL-Injections sind.

## NoSQL-Injections

In den letzten Jahren werden vermehrt NoSQL-Datenbanken eingesetzt.
Diese verwenden nicht SQL als Abfragesprache, sondern meistens
eigenständige Abfragesprachen oder exekutieren JavaScript-Snippets als
Query. Hier ein Beispiel in MongoDB:

```javascript
db.myCollection.find( { active: true, $where: function() { return obj.credits - obj.debits < $userInput; } } );
```

Bei diesem Beispiel wird als Query der aktuellen Kontostand berechnet
(*credits - debits*), falls dieser unter einer benutzerdefinierten
Schranke liegt (*$userInput*) wird der behalten, ansonsten ausgefiltert.
Die Abfrage ist als JavaScript implementiert und nicht als SQL.

Die grundsätzliche Problematik einer Injection bleibt ident. In dem
gewählten Beispiel wird z.B. die Benutzereingabe nicht escaped, ein
Angreifer kann daher auf diese Weise Schadcode einfügen:

```javascript
    "(function(){var date = new Date(); do{curDate = new Date();}while(curDate-date<10000); return Math.max();})()"
```

Hier wird nun innerhalb der Abfrage eine Javascript-Funktion definiert
und sofort danach aufgerufen. Die Funktion macht nichts anderes, als 10
Sekunden lang eine Endlosschleife aufzurufen. Falls der MongoDB-Server
nach dem Absetzen dieser Query für 10 Sekunden nicht antwortet und eine
CPU zu 100% ausgelastet ist, hat man also eine datenbankseitige
Injection erreicht.

Wie man an dem Beispiel sehen kann, ist der alleinige Einsatz von
NoSQL-Datenbanken nicht ausreichend um eine Datenbank-Injection zu
vermeiden.
