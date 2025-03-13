# Reverse Tab Nabbing

Bei einem Reverse Tab Nabbing navigiert der Benutzer zuerst auf eine
Opferseite. Diese öffnet nun einen Link auf eine bösartige Seite in
einem neuen Fenster. Die aufgerufene bösartige Seite verwendet nun
Javascript um die Adresse der aufrufenden Seite (die wahrscheinlich
gerade im Hintergrund ist) zu verändern, der Webbrowser führt nun ein
redirect auf die neu verlinkte Seite im Hintergrund vor (während der
Benutzer noch immer die neu geöffnete Seite betrachtet). Wenn der
Benutzer nun das geöffnete Fenster schließt befindet er sich
vermeintlich auf der ursprünglichen Seite, welche den Link öffnete,
befindet sich allerdings in Wirklichkeit auf einer Seite, die vom
Angreifer bestimmt wurde.

Ein Beispiel für Reverse Tab Nabbing, folgende Opfer Seite:

```html
<html>
  <body>
    <li><a href="bad.example.com" target="_blank">Vulnerable target using html link to open the new page</a></li>
    <button onclick="window.open('https://bad.example.com')">Vulnerable target using javascript to open the new page</button>
  </body>
</html>
```

Die Opferwebseite öffnet eine externe Seite über einen Link (mittels
*target=\_blank* wird ein neues Fenster geöffnet) bzw. alternativ über
Javascript (*onclick*). Als neue Webseite verwendet der Angreifer
folgendes:

```html
<html>
  <body>
    <script>
      if (window.opener) {
        window.opener.location = "https://phish.example.com";
      }
    </script>
  </body>
</html>
```

Der Angreifer setzt über *window.opener* die Adresse der aufgerufenen
Seite und ändert dadurch (im Hintergrund) die im Webbrowser dargestellte
Seite. Wenn der Benutzer die geöffnete Seite schließt, gelangt er
dadurch auf eine vom Angreifer modifizierte Webseite.

Als Gegenmaßnahme sollte bei ausgehenden Links immer das *rel* Attribute
auf *noopener noreferrer* gesetzt werden. Dadurch kann die geöffnete
Seite nicht mehr über *windows.opener* auf die Location der öffnenden
Seite zugreifen. Zusätzlich kann über die *Referrer-Policy* das
Übermitteln des Referrer-Headers an die aufgerufene Webseite unterbunden
werden.

**Update 2020**: mehrere Browser bieten mittlerweile automatische
Verteidigungsmassnahmen gegenüber Reverse Tabnabbing Angriffe. Firefox
(seit 2016), Microsoft Edge und Firefox sollten out-of-the-box nicht
mehr gegenüber diesem Angriff verwundbar sein (sie setzen das
*noopener*-Flag automatisch). In zukünftigen Google Chrome Versionen ab
2021 sollte diese Browserfamilie dies auch durchführen und auf diese
Weise Tabnabbing-Angriffe unterbinden.
