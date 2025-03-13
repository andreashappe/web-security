# Referrer-Policy

Ein Standard-Header der von Webbrowsern gesetzt wird ist der
*Referer*-Header. Dieser inkludiert bei jedem Seitenaufruf die URL der
aufrufenden Seite. Dies kann einen negativen Security-Impact haben,
falls die URL der aufrufenden Seite sensible Informationen (wie z.B.
eine Session-Id oder auch sensible Benutzerdaten) beinhaltet. Potentiell
wird der Header vom Empfangsserver und, bei unverschlüsselter
Kommunikation, von allen verbundenen Geräten entlang des
Kommunikationspfades gesehen.

Web-Server können das gewünschte Verhalten durch Verwendung des Headers
*Referrer-Policy*[1] mitteilen, valide Werte sind:

- no-referrer: der Referer-Header wird nicht übertragen.
- no-referrer-when-downgrade: die URL wird als Referer übertragen sofern die Folgeseite nicht ein
unsichereres Protokoll verwendet. Dadurch wird z.B. eine Übertragung der
URL beim Übergang von HTTPS zu HTTP verboten. Dies ist häufig das
Default-Verhalten der Webbrowser.
- origin: es wird immer nur der Origin übertragen.
- origin-when-cross-origin: es wird die volle URL übertragen, sofern man sich innerhalb des
identen Origins befindet. Falls es zu einem Origin-Wechsel kommt (z.B.
durch Navigation auf eine externe Seite) wird nur der Origin übertragen.
- same-origin: solange man sich innerhalb des Origins befindet, wird die URL gesetzt,
ansonsten wird kein Referer-Header versendet.
- strict-origin: es wird immer nur der Origin als Referer versendet, und dies auch nur
falls das Sicherheitslevel ident ist (es wird also beim Übergang von
einer HTTPS auf eine HTTP Seite kein Referer gesetzt).
- strict-origin-when-cross-origin: es wird die volle URL innerhalb des identen Origin verwendet, wird auf
Webseiten mit unterschiedlichen Origin zugegriffen (also z.B. beim
Übergang auf externe Webseiten) wird nur der Origin als Referer
verwendet. Falls die Sicherheit der Kommunikation schlechter wird (also
z.B. beim Übergang von HTTPS auf HTTP) wird überhaupt kein
Referer-Header versendet.
- unsafe-url: die gesamte URL wird im Referrer-Header immer übertragen.

Es wird empfohlen, eine *Referrer-Policy* zu wählen, die nicht
*no-referrer-when-downgrade* bzw. *unsafe-url* ist.
