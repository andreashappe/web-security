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

## Reflektionsfragen (need to split those up)

1. Wie funktioniert eine SQL union-based Injection? Womit können
   SQL-Injections vermieden werden?

2. Wie funktioniert eine SQL time-based Injection? Womit können
   SQL-Injections vermieden werden?

3. Warum sollten SQL prepared statements verwendet werden?

4. Was versteht man unter einer Serialisierungs-Schwachstelle? Welche
   negativen Auswirkungen können Serialisierungsangriffe auf eine
   Applikation besitzen?

5. Was versteht man unter XML External Entity Attacks? Welche negativen
   Auswirkungen auf die Applikation können erzielt werden und welche
   Gegenmaßnahmen sind möglich?

6. Welche Probleme können beim Upload eines Files auf einen Webserver
   auftreten? Welche Best-Practises im Zusammenhang mit File-Uploads
   sollten beachtet werden?

7. Unterschied der Angriffsvektoren mit einem File, dass serverseitig
   exekutierten Code enthält und einem File, dass client-seitig
   exekutierten Code enthält?

8. Wie können Path-Traversal Angriffe eingesetzt werden?

9. Erläutere LDAP-Injections.

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
