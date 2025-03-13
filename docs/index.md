# Web Security

## Struktur dieses Dokumentes

Zur besseren Verständlichkeit wurde ein Top-Down-Approach gewählt. Im
Zuge der Vorlesung bewährte sich dies, da auf diese Weise die Studenten
vom Allgemeinen in die jeweiligen Spezialfälle geführt werden können.

Im ersten Part *Einführung* versuche ich das Umfeld von Security zu
beleuchten. Da meine Welt ursprünglich die Softwarenentwicklung war,
gebe ich hier auch einen groben Überblick wie Security während der
Entwicklung beachtet werden kann. Zusätzlich versuche ich unser
Zielumfeld, Web-Applikatoinen, etwas genauer zu betrachten. Auf diese
Weise soll auch sicher gestellt werden, dass Studenten bzw. Leser einen
ausreichenden Wissensstand vor Beginn der eigentlichen Security-Themen
besitzen.

Der nächste Part (*Authentication und Autorisierung*) behandelt
high-level Fehler bei der Implementierung der Benutzer- und
Berechtigungskontrolle. Drei Kapitel (*Authentication*, *Authorization*,
*Federation/Single Sign-On*) beschreiben Gebiete, die applikationsweit
betrachtet werden müssen — falls hierbei Fehler auftreten, ist zumeist
die gesamte Applikation betroffen und gebrochen.

Im darauf folgenden Part (*Injection Attacks*) wird auf verschiedene
Injection-Angriffe eingegangen. Hier wurde zwischen Angriffen, die
direkt gegen den Webserver, und Angriffen die einen Client (zumeist
Webbrowser) benötigen, unterschieden. Während auch hier Schutzmaßnahmen
am besten global für die gesamte Applikation durchgeführt werden
sollten, betrifft hier eine Schwachstelle zumeist einzelne Operationen
und kann dadurch auch lokal korrigiert werden.

## Out-of-Scope für dieses Skript

Auf drei wichtige Bereiche wird im Zuge dieses Skripts nicht explizit
eingegangen:

### Denial-of-Service Angriffe

Denial-of-Service Angriffe zielen darauf ab, die Verfügbarkeit einer
Applikation zu beeinträchtigen. Dadurch kann der Dienst nicht mehr
benutzt bzw. konsumiert werden und dem Betreiber entstehen Kosten, z.B.
Verdienstentgang durch einen ausgefallenen Webshop.

Ein DoS-Angriff zielt entweder auf eine Applikations-bezogene Ressource
wie z.B. erlaubte Verbindungen pro Applikationsbenutzer oder eine
fundamentale Systemressource wie z.B. CPU-Zeit, Speicher oder
Netzwerkbandbreite ab. Als Applikationsentwickler kann man bei
Ressourcen-intensiven Operationen mittels Rate-Limits die Situation
entschärfen.

In diesem Dokument wird nicht tiefer auf DoS-Angriffe eingegangen, da
diese quasi die Holzhammermethode darstellen. Gerade gegenüber Angriffen
gegen die Netzwerkbandbreite kann nur über kommerzielle Cloud- bzw.
Rechenzentrenbetreiber entgegengewirkt werden. Diese sind kostspielig
und es entsteht eine Asymmetrie: die Abwehr des Angriffs kann
kostspieliger als der Angriff selbst werden. Somit wird aus einem
technischen DoS ein monetärer DoS.

### Security und Usability

Es gibt das Vorurteil, dass Sicherheit und Usability konträr zueinander
sind. Während dies in wenigen bedauerlichen Einzelfällen gegeben sein
kann, sollte dies nicht als Pauschalausrede missbraucht werden.

Der Benutzer will primär eine Aufgabe erledigen. Im Zuge der Erledigung
dieser Aufgabe sollte Sicherheit nicht im Weg stehen. Stattdessen sollte
der offensichtliche Weg der Aufgabenerledigung sicher implementiert sein
und den Benutzer über einen sicheren Weg zur Erledigung der Aufgabe
leiten. Falls sicherheitsrelevante Benutzerentscheidungen notwendig
sind, sollten diese möglichst früh erfolgen — wird dies während der
Abarbeitung einer Aufgabe durchgeführt, kann der Benutzer so fokussiert
sein, dass die Sicherheitsentscheidung nur peripher beachtet wird.

Ebenso sollte der Benutzer nicht mit irrelevanten Fragen bombardiert
werden da daruch nur der “Meldung-wegklicken”-Reflex des Benutzers
konditioniert wird. Die Willigkeit eines Benutzers, auf Sicherheit
Rücksicht zu nehmen ist begrenzt, vergleichbar mit einer Batterie. Wenn
diese erschöpft ist, wird weniger (oder gar keine) Rücksicht auf die
Security genommen.

Ein besserer Weg ist es, per default sichere Prozesse zu implementieren
und im Bedarfsfall unsichere Operationen durch den Benutzer explizit zu
erlauben. Die dabei verwendeten Benutzerinteraktionen sollten dem
NEAT-Prinzipien genügen:

- Necessary: kann die Applikation, anstatt den Benutzer zu fragen, das
  Problem auf eine andere sichere Art und Wiese lösen?

- Explained: besitzt der Benutzer das notwendige Wissen um eine
  informierte Entscheidung zu treffen?

- Actionable: kann der Benutzer überhaupt sinnvoll auf die
  dargestellte Meldung reagieren?

- Tested: ist die Meldung innerhalb der UX sinnvoll und wurde
  getestet, ob sie in jeglicher Form von Benutzerfluss sinnvoll ist?

Im Zuge der DSGVO/GDPR wurde bestimmt, dass Software *secure by design
and default* sein muss. Dies bedeutet, dass Software die Möglichkeit
einer sicheren Konfiguration bieten, und diese im Auslieferungszustand
auch sicher konfiguriert sein muss. Ein dagegen verstossendes Beispiel
wäre der Einsatz von Default-Passwörtern.

### Ethical Web Development

Technik an sich ist wertneutral. Sobald diese allerdings in Berührung
mit der Realität kommt, entsteht ein ethischer Impact. Web Applikationen
sind hier keine Ausnahme. Im Zuge des Skripts wird auf ethischen Impact
nicht explizit eingegangen, da der Inhalt der Vorlesung das Werkzeug und
nicht das Ziel des erstellten Werks ist.

Um die ethische Dimension nicht vollständig zu ignorieren, ein paar
[Richtlinien der EDRi](https://edri.org/ethical-web-dev/):

Allow as much data processing on an individual’s device as possible.  
Dies würde im Web-Umfeld den Einsatz von JavaScript bedingen, da nur auf
diese Weise Daten direkt im Browser des Benutzers verarbeitet werden
können.

Where you must deal with user data, use encryption.  
Dies inkludiert sowohl Transport-Level Encryption (wie TLS) als auch
Verschlüsselung der bearbeiteten Daten.

Where possible also use data minimisation methods.  
Das Minimalprinzip sollte auch auf die gespeicherten Daten angewendet
werden. Daten die eine Applikation nicht besitzt sind Daten, die auch
nicht entwendet oder zweckentfremdet werden können.

Use first-party resources and avoid using third-party resources.  
Es besteht die Sorge, dass externe Ressourcen modifiziert werden
könnten. Dies soll durch die Verwendung eigener Ressourcen vermieden
werden. Falls notwendig, können CSP-Direktiven bzw. Subresource
Integrity verwendet werden um die Integrität externer Ressourcen
sicherzustellen.

[1] Subjektiv im Auge des Betrachters.
