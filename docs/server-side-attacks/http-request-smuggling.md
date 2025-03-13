# HTTP Request Smuggling

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

1. Der HTTP-Header *Content-Lenght* beinhaltet die Länge des
   Request-Bodies.

2. Wird als HTTP-Header *Transfer-Encoding: chunked* verwendet, gibt es
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

```http
 POST / HTTP/1.1
 Host: vulnerable-website.com
 Content-Length: 13
 Transfer-Encoding: chunked

 0

 SMUGGLED 
```

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

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0
```

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

```http
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
```

Unter der Annahme, dass das Frontend *Content-Length* und das Backend
*Chunked-Encoding* verwendet würde folgendes passieren:

- Das Frontend kopiert den gesamten Request auf die Verbindung zum
  Backend.

- Das Backend versucht den Request zu erkennen, verwendet dafür
  *Chunked-Encoding* und beendet den Request mit dem 0-Chunk.

- Das Backend liest die nächste Zeile Text und interpretiert diese als
  Beginn eines neuen Requests. In diesem Fall wäre dies ein POSt
  Request auf `/post/comment`.

- Da der Angreifer selbst ein Konto erstellen konnte, konnte dieser
  eine valide Session als auch ein valides CSRF-Token generieren. Die
  Session wird als HTTP-Header eingeschleust. Innerhalb des
  eingeschleusten Requests wird auch das bekannte CSRF-Token gesetzt.

- Innerhalb des eingeschleusten Requests wird mit einem Request-Body
  begonnen. In diesem werden mehrere Parameter gesetzt. Der
  eingeschleuste Request endet mit einem *comment=*, also dem Wert der
  als Kommentar vom User eingegeben werden sollte.

- Da das Backend nicht weiss, dass der Request beendet ist, liest es
  die nächsten Daten aus der eingehenden Verbindung. In diesem Fall
  ist dies ein Request eines anderen Benutzers, der zeitgleich
  abgegeben und vom Frontend direkt nach dem eingeschleusten Reqeust
  in die Verbindung kopiert wurde.

- Die Applikationslogik interpretiert nun den Folgerequest als Inhalt
  des Parameters *comment*, also als Kommentar der als Kommetar
  gepostet werden sollte (da die Operation `/comment/post` war).

- Der Angreifer überprüft nun, ob ein neuer Kommentar mit sensiblen
  Daten eines anderen Users an der betroffenen Stelle auftaucht.
  Sensible Daten könnte z.B. Credentials im Zuge einer Login-Operation
  oder auch ein HTTP-Header mit Session-Cookies sein. Falls möglich,
  würde der Angreifer diese Verwenden um serverseitig die Identität
  des anderen Benutzers anzunehmen.

- Falls kein Kommentar erscheint oder der Kommentar nur sinnlose
  Informationen beinhaltet, wiederholt der Angreifer den
  Angriffsrequest (potentiell würde er ein neues CSRF-Token initial
  generieren).

Was kann man gegen HTTP Request Smuggling Angriffe unternehmen? Mehrere
Möglichkeiten würden diese unterbinden;

- Sicherstellen, dass Front- und Backend die Request-Länge ident
  interpretieren.

- Am Frontend potentiell mehrdeutige Request-Längen mit eindeutigen
  Längen ersetzen.

- Zwischen Front- und Backend für jeden Client-Request eine neue
  Verbindung aufbauen. Dies wird zumeist aus Performance-Gründen nicht
  durchgeführt.

- Zwischen Front- und Backend HTTP/2 verwenden. Diese Protokollversion
  besitzt die Mehrdeutigkeit der Requestlängen nicht mehr.
