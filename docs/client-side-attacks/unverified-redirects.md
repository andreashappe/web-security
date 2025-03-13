# Unverified Forwards and Redirects

Diese Schwachstelle war in den OWASP Top 10 2013 vorhanden, wurde
allerdings 2017 aus der Liste der Top 10 entfernt. Die Schwachstelle
entsteht, falls eine Operation einer Webapplikation den Benutzerbrowser
auf eine weitere Seite weiterleitet und das Ziel über einen Parameter
bestimmt wird. Ein Angreifer kann nun versuchen, das Opfer auf eine
externe Seite zu leiten um dies im Zuge eines Social Engineering
Angriffs auszunutzen. Eine verwundbare Operation würde z.B.
folgendermaßen aussehen: `http://example.com/example.php?url=http://malicious.example.com`.

Besonders gefährlich ist es, wenn die verlinkte URL nicht über ein HTTP
Redirect aufgerufen wird, sondern wenn die übergebene URL als Ziel eines
eingebetteten IFrames verwendet wird. Auf diese Weise kann der Angreifer
Inhalte auf der (vermeintlichen) Opferwebseite platzieren, die meisten
Enduser werden nicht bemerken, dass sie gerade Daten in einem Iframe und
nicht in der Opfer-Webseite eingeben.

Falls es wirklich notwendig sein sollte, dass eine Zieladresse über
einen URL-Parameter übergeben wird, sollte penibles Whitelisting der
erlaubten URLs betrieben werden.
