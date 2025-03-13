# Clientseitige Angriffe

Client-seitige Angriffe zielen auf den Web-Browser des Benutzers ab. Da
eine Interaktion des Benutzers bei vielen Angriffen benötigt wird,
werden sie zumeist im Zuge von Social-Engineering Angriffen eingesetzt.
Webserver besitzen die Möglichkeit, mittels optionaler HTTP Header den
Clients Sicherheitspolicies und Verwendungshinweise mitzuteilen; Clients
können auf diese Weise Schadcode erkennen und filtern.

## Reflektionsfragen

1. Erkläre Reflected-, Stored- und DOM-Based XSS Angriffe. Welche
   Gegenmaßnahmen gibt es und erläutere diese.

2. Was sind unvalidated Forwards und Redirects? Wie kann dagegen
   geschützt werden?

3. Was sind Reverse Tab Nabbing Angriffe? Welche Absicherungsmaßnahmen
   gibt es dagegen?

4. Welche Sicherheitsprobleme können bei HTML5 Local Storage auftreten?

5. Wie funktionieren Clickjacking-Angriffe und wie können diese
   verhindert werden?

6. Wie funktioniert ein CSRF-Angriff? Erläutere zwei potentielle
   Gegenmaßnahmen?

[1] Das Document-Object-Model beschreibt eine Programmierschnittstelle
welche HTML/XML-Daten als Baumstruktur darstellt. Mittels Javascript
kann das DOM modifiziert werden um beispielsweise Elemente bzw. deren
Attribute hinzuzufügen, entfernen oder zu modifizieren; Eventhandler zu
setzen bzw. Events zu feuern; bzw. um CSS zu verändern.

[6] Dies wird als UXSS bezeichnet, siehe auch
<https://blog.innerht.ml/the-misunderstood-x-xss-protection/>.
