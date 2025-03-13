# HTML5 PostMessage als Angriffskanal

Eine Webapplikation wird innerhalb eines Browser-Tabs geöffnet, ihre
Einflussmöglichkeiten (z.B. mittels Javascript) beschränken sich auf
Inhalte innerhalb des Browser-Tabs. Es gibt Use-Cases, bei denen eine
Applikation mit einer Webseite innerhalb eines anderen Browser-Tabs bzw.
Browser-Fensters interagieren will. Ein Beispiel sind web-basierte
Präsentationsframeworks. Hier gibt es meistens zwei Browserfenster:
eines für die aktuell dargestellte Präsentationsfolie und ein Fenster
mit Notzien für den Vortragenden. Wird die Folie gewechselt sollten im
zweiten Fenster ebenso die Kommentare für die aktuell angezeigte Folie
dargestellt werden.

Eine moderne Implementierungsmöglichkeit für diese Funktion ist *HTML5
postMessage*. Die Webseite, welche eine Aktion ausführen will, kann eine
Nachricht via Javascript absenden. Diese Nachricht beinhaltet die
message, einen *Target-Origin* (kann auch das Wildcard *\** sein) und
eine Liste von serialisierten Objekten (deren Owernship an den Empfänger
übergehen). Die empfangende Webseite kann einen Callback-Handler für
empfangene Webseiten registrieren und auf diese Weise auf die Nachricht
reagieren.

Ein Beispiel für einen Message-Handler:

```html
<script>
function messageHandler(event){
    from = "From: " + event.origin;
    data = "Data: " + event.data;
    alert(from);
    alert(data)
}
// Register the handler
window.addEventListener("message", messageHandler)
</script>
```

Das Beispiel zeigt bereits eine Schwachstelle von HTML5 postMessage: der
*origin* wird nicht durch den Empfänger überprüft, sondern durch den
Code des Empfängers.

Wie kann eine Nachricht gesendet werden?

```javascript
otherWindow.postMessage(message, targetOrigin, [transfer])
```

Die jeweiligen Variablen wären:

- **otherWindow** gibt den Empfänger an. Dieser kann z.B. *parent*,
  ein Iframe, *window.opener* oder*window.source* sein.

- **message** ist der String der als Nachricht an den Empfänger
  übertragen wird.

- **targetOrigin** gibt die origin des Empfängers an, kann aber auch
  als Wildcard (*\**) ausgeführt sein.

- **tranfer** ist eine Liste von übertragenen Objekten. Diese können
  vom Sender nicht mehr verwendet werden und gehen in den Besitz des
  Empfängers über.

Hier ergeben sich zwei Angriffsszenarien:

1. Eine Webseite akzeptiert Nachrichten von beliebigen Quellen. Dies
   könnte z.B. im Zuge eines XSS-Angriffs ausgenutzt werden.

2. Beim Senden der Nachricht werden sensible Daten versendet ohne dass
   der Empfänger eingeschränkt wird. Dies geht zumeist mit einer
   *TargetOrigin* von *\** herein.
