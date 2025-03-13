# Integration externer Komponenten

Werden externe Inhalte innerhalb der eigenen Seite inkludiert, erhöht
sich die Angriffsfläche: ein Angreifer mit Zugriff auf die externe Seite
kann über diese Schadcode in die, eigentlich sichere, eigene Webseite
einschleusen. Falls die Einbindung externer Inhalte zwingend benötigt
wird, kann das Gefahrenpotential durch Einsatz folgender Techniken
reduziert werden:

## IFrame: sandbox-Flag

Wird eine externe Resource über das HTML *iframe* Tag eingebunden kann
durch Verwendung des *sandbox*-Attributes die Sicherheit erhöht werden.
Bei Verwendung dieses Attributes werden folgende Einschränkungen
aktiviert:

- JavaScript wird für die eingebundene Resource deaktiviert.

- Die eingebundene Seite bekommt einen eigenen Origin; dadurch kann
  dieses nicht mehr auf die einbindende Seite zugreifen, auch wenn
  diese auf dem identen Server abgelegt waren.

- Die eingebundene Seite kann keine neuen Fenster bzw. Dialoge öffnen.

- Es können keine Formulare abgeschickt werden.

- Plugins werden für die eingebettete Seite deaktiviert.

- Autoplay wird deaktiviert.

Die Einschränkungen können durch mehrere Optionen aufgeweicht werden,
der Namen dieser Optionen beginnt mit *allow-*. Ein abschließendes
Beispiel für einen per Iframe eingebundenen Twitter-Button:

```html
<iframe sandbox="allow-same-origin allow-scripts allow-popups allow-forms"
src="https://platform.twitter.com/widgets/tweet_button.html"
style="border: 0; width:130px; height:20px;"
</iframe>
```

## Subresource Integrity (SRI)

Webapplikationen lagern statische Dateien häufig auf externe Server aus.
Ein Beispiel hierfür wäre z.B. das Auslagern von statische Javascript-
oder CSS-Dateien auf ein CDN-Netzwerk. Dies wird zumeist zur Erhöhung
der Performance bzw. Reduktion der Latenzzeit durchgeführt.

Ein Angreifer, der Zugriff auf einen externen Server erlangt, kann auf
diesen bösartigen JavaScript- oder CSS-Code hinterlegen. Lädt nun eine
Webseite diese Dateien, wird diese automatisch infiziert. Um diesen
Angriffsvektor zu vermeiden, kann Subresource Integrity verwendet
werden. Bei dieser Technik wird ein Hashwert für jede eingebundene
Resource berechnet und innerhalb der eigenen Webseite angegeben. Wird
nun eine externe Resource angefordert, berechnet der Webbrowser den
Hashwert der empfangenen Resource und vergleicht diesen mit dem
konfigurierten Hashwert (innerhalb der Webseite). Die Resource wird nur
verwendet, wenn diese Hashwerte ident sind.

Ein Beispiel:

```html
<script src="https://example.com/example-framework.js"
integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
crossorigin="anonymous">
</script>
``  `

Mittels CSP kann die Verwendung von Subresource Integrity erzwungen
werden, folgendes Beispiel erzwingt die Angabe von Hashsummen für alle
inkludierte JavaScript- und CSS-Dateien.

```http
Content-Security-Policy: require-sri-for script style;
```
