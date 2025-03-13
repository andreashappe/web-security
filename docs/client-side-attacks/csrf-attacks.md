# CSRF Angriffe

CSRF-Angriffe nutzen ein bestehendes Vertrauensverhältnis zwischen dem
Web-Browser des Opfers und einem Webserver aus. Das grundsätzliche
Problem ist, das Webbrowser, bei Requests zu bereits eingeloggten
Webservern, automatisch Sessions anhängen. Dabei wird nicht überprüft,
ob der ausgehende Request wirklich vom Benutzer in Auftrag gegeben
worden ist.

Folgende Schritte würden bei einem typischen CSRF-Szenario passieren:

1. Der Benutzer (im Folgenden das Opfer genannt) loggt sich bei einem
   Webserver ein. Der Webbrowser des Benutzers speichert sich das
   Session-Cookie für Folgezugriffe auf diesen Webserver.

2. Der Benutzer surft im Internet und besucht dabei einen durch den
   Angreifer kontrollierten Webserver.

3. Auf diesem Webserver befindet sich ein Formular, welches eine
   Operation auf dem Webserver, auf dem das Opfer eingeloggt ist,
   aufruft.

4. Der Browser des Opfers lädt die Webseite vom Webserver des
   Angreifers. Das Formular wird entweder durch den unbedarften
   Anwender oder durch Javascript automatisch abgesendet.

5. Der Browser des Opfers hängt automatisch das Session-Cookie zu dem
   ausgehenden Request hinzu.

6. Der Webserver (auf dem das Opfer eingeloggt war) erhält nun einen
   Request mit einer validen Session ausgehend vom Webbrowser des
   Opfers. Da dieser Request vollkommen korrekt aussieht, wird dieser
   auch exekutiert.

Ein Beispiel für ein HTML Formular welches der Angreifer auf seinem
Webserver hinterlegen würden:

```html
<form action="http://bank.com/transfer.do" method="POST">
    <input type="hidden" name="acct" value="MARIA"/>
    <input type="hidden" name="amount" value="100000"/>
    <input type="submit" value="View my pictures"/>
</form>
```

![Beispiel für einen CSRF-Angriff](/images/csrf.svg)

In diesem Fall wird die Operation `http://bank.com/transfer.do` mit den
Parametern *acct* und *amount* aufgerufen. Bei diesem Beispiel wurden
die Felder versteckt und der Button mit einem *ablenkenden* Text
beschriftet. Alternativ könnte der Angreifer das Formular auch in einem
1x1 Pixel großem IFrame verstecken und automatisiert mittels Javascript
abschicken.

## Gegenmaßnahmen

### Synchronizer Token Pattern

Es gibt mehrere Gegenmaßnahmen gegen CSRF-basierte Angriffe,
sicherheitstechnisch ist das so genannte *Synchronizer Token*-Pattern
vorzuziehen. Bei diesem fügt der Webserver bei jedem Formular ein
verstecktes HTML-Feld hinzu, in dieses schreibt der Server einen
zufälligen Zahlenwert. Wird eine Operation am Server aufgerufen wird
dieses Feld an den Server übertragen und dieser vergleicht den
übertragenen Zahlenwert mit dem vom Server erwarteten Zahlenwert. Falls
diese übereinstimmen, wird die Operation ausgeführt, ansonsten wird die
Operation verworfen. Dieser Schutz funktioniert, da der Angreifer auf
seinem remote Server den Zahlenwert erraten und in das Angriffs-Formular
einfügen müsste.

Damit dieser Schutz verlässlich funktioniert, muss der Zahlenwert
regelmäßig erneuert werden, bevorzugterweise sollte für jede potentielle
Operation ein neuer Zufallswert generiert werden. In der Praxis wird
diese Anti-CSRF Maßnahme häufig vollkommen transparent und automatisch
durch das verwendete Web-Framework implementiert.

Wichtig ist, dass eine Operation die einen CSRF-Check implementiert
nicht nur überprüft, ob ein potentiell übergebener CSRF-Wert mit dem
server-gespeicherten CSRF-Wert übereinstimmt, sondern auch überprüft ob
überhaupt ein CSRF-Wert übergeben wurde. Während Tests wurde häufig das
fehlerhafte Verhalten vorgefunden, dass wenn der CSRF-Parameter einfach
gelöscht wird, die Operation ausgeführt wird (also CSRF-Tokens nur
verglichen werden, wenn beim Aufruf zumindest ein CSRF-Wert übergeben
wird).

### SameSite-Flag bei Session Cookies

Eine weitere Schutzmaßnahme (im Sinne des Hardening) ist der Einsatz des
*SameSite* Cookie-Flags. Bei korrektem Setzen dieses Flags erlaubt der Web-Browser des
Opfers das Übertragen der Session-Id nur, wenn sowohl das Formular als
auch das Ziel des Formulars sich auf dem identen Webserver befinden.
