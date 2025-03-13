# Command Injection

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

- `;ls`
- `$(ls)`
- `‘ls‘`

Ein ähnliches Verhalten kann ausgenutzt werden, wenn der Verdacht
besteht, dass Dateien mittels Systembefehlen ausgegeben werden und die
auszugebende Datei über HTTP Parameter übermittelt wird.

Beispiele hierfür:

- `http://sensitive/cgi-bin/userData.pl?doc=/bin/ls|`
- `http://sensitive/something.php?dir=%3Bcat%20/etc/passwd`

Um Command Injection Probleme zu umgehen wird empfohlen,
Programmierbibliotheken anstatt von Kommandozeilenaufrufen zu verwenden.
Da hierbei nun keine getrennte Shell geöffnet wird, kann an dieser
Stelle auch kein Kommando eingefügt werden.
