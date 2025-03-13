# Clickjacking

Clickjacking wird auch teilweise *UI redress attack* genannt. Bei diesem
Angriff will der Angreifer einen unbedarften Benutzer dazu bringen, eine
Webseite zu bedienen. Um dies durchzuführen, baut der Angreifer eine
eigene, harmlos aussehende, Webseite, welche den identen Bedienfluss wie
die Webseite besitzt, die der Angreifer gerne fernsteuern würde. Mittels
eines IFrames wird die Opferwebseite über die erstellte Webseite des
Angreifers gelegt, die Transparenz der Opfer-Webseite wird auf 100%
gesetzt.

Wenn nun der Benutzer die vermeintliche (vom Angreifer erstellte)
Webseite bedient, werden in Wirklichkeit alle Benutzereingaben an die
transparente Opfer-Webseite übertragen und dadurch diese durch den
Benutzer ferngesteuert.

Eine gute Abwehrmassnahme gegen Clickjacking ist der *X-Frame-Options*
HTTP Header.

## X-Frame-Options

Der *X-Frame-Options* Header wird verwendet um dem Webbrowser
mitzuteilen, innerhalb welcher Webseiten die eigene Webseite eingebunden
werden darf. Dadurch werden Clickjacking-Angriffe unterbunden.

Der Webserver kann über das Setzen des *X-Frame-Options* Header auf
folgende Werte das Webbrowser-Verhaltensmuster beeinflussen:

- DENY: die Webseite darf nicht von anderen Webseiten mittels IFrames
eingebunden werden.
- SAMEORIGIN: die Webseite darf von allen Webseiten mit dem identen Origin
eingebunden werden.
- ALLOW-FROM domain: die Webseite darf explizit von der Domain *domain* eingebunden werden.

Die Verwendung von *X-Frame-Options* ist allerdings nicht problemlos.
EIn häufiger Fehler ist es, dass bei *ALLOW-FROM* mehr als ein Origin
angegeben wird. Dies kann z.B. geschehen, falls der Entwickler das
Inkludieren ausgehend von zwei externen Seiten, oder das Inkludieren
ausgehend von der eigenen und von einer externen Seite erwünscht. Dies
ist mittles *X-Frame-Options* nicht abbildbar, wird dieses Verhalten
gewünscht, muss eine Content-Security Policy angewendet werden.

Ein weiteres Problem ist *Double Framing*. Eine Webseite versucht durch
Einsatz von *SAMEORIGIN* das Einbinden durch eine externe Seite zu
unterbinden. Der *X-Frame-Options* Header bezieht sich allerdings immer
auf das ,,äußerste” IFrame. Wird z.B. auf der eigenen Seite ein IFrame
mit einer externen Seite inkludiert, und diese externe Seite inkludiert
selbst über ein IFrame die eigene Seite, wird diese angezeigt auch wenn
dies durch den gesetzten Header als unterbunden gedacht wurde. Auch dies
ist nicht einfach über *X-Frame-Options* abbildbar.
