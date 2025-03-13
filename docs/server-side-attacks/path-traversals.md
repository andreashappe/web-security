# Path Traversals

Bei einem Path Traversal wird versucht, über modifizierte Parameter auf
Ressourcen außerhalb des Webroots einer Webapplikation zuzugreifen. Auf
diese Weise kann versucht werden, auf applikations-externe Ressourcen
lesend oder schreibend zuzugreifen bzw. kann versucht werden,
ausführbare Dateien am Server zu starten.

Ein Beispiel für eine potentiell angreifbare Operation wäre
`https://opfer.local/GetImage.jsp?file=diagram.jpg`. Ein Angreifer
könnte versuchen, über den Wert `./../../../../etc/passwd` für den
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
