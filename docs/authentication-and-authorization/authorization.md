# Authorization

Unter Authorization versteht man die Kontrolle der Benutzerrechte vor
der Ausführung der jeweiligen Operation.

Bei einer fehlerhaften Authorization muss ein Angreifer sich zumindest
authentifizieren. Im Fehlerfall kann der Betreiben den Login- und
Registrierungsprozess unterbinden, alle Benutzer ausloggen und erhält
auf diese Weise eine Plattform, auf die ein trusted User wieder zulassen
kann um dadurch einen eingeschränkten Betrieb zu ermöglichen (während
der grundlegende Fehler behoben wird). Im Gegensatz dazu, muss bei einer
fehlenden Authentication die Plattform deaktiviert werden, da diese
Möglichkeit zur Einschränkung auf vertrauenswürdige Benutzer entfällt.

## Probleme bei der Berechtigungsüberprüfung

In diesem Kapitel wird auf mehrere Probleme im Zusammenhang mit
Bereichtungsüberprüfungen eingegangen.

### Keine/Fehlende Authorization

Die Applikation überprüft zwar die Authentication aber jeder eingeloggte
Benutzer darf alle Operationen aufrufen.

Da ein Benutzer sich initial authentifizieren muss, kann aufgrund des
Logverlaufs der Fehler zumindest analysiert und der bösartige User
ausgesperrt werden. Eine Gefährdung durch Crawler findet ebenso nicht
statt, da diese keine valide Authentication durchführen können.

Analog zu den Fehlern bei der Authentication, kann es zu Problemen
kommen wenn eine Applikation innerhalb mehrere
Programmiersprachen/Frameworks implementiert wurde bzw. weitere
Komponenten zu einer bestehenden Lösung hinzugefügt wurden. Hauptproblem
ist die Integration der Authorization in das Bestandssystem. Als
Pen-Tester sollte man immer diese Komponenten gezielt auf
Authentication- und Authorization-Fehler hin testen.

*There is a special place in hell for developers that think that
not-displaying UI elements is a kind of authorization*. Häufig werden in
Abhängigkeit der Benutzerrolle nur Teile der Funktionalität angezeigt.
Ein Angreifer der einen zweiten Benutzeraccount mit diesen Rechten (oder
Log-Dateien) besitzt, erhält allerdings Informationen über die
Operationen und kann diese direkt aufrufen. Dies ist ein Fall von
Security by Obscurity.

Das erzwungene Aufrufen von Webseiten über URLs wird auch *Forceful
Browsing* genannt. Wird direkt auf Ressourcen, wie z. b. Downloadlinks,
zugegriffen, wird dies *direct object reference* genannt.

### Unterschiedliche Authorization in Alternate Channels

Falls eine Webapplikation Daten oder Operationen auf unterschiedliche
Arten und Weisen anbietet, müssen die Schutzmechanismen auf den
verschiedenen Kanälen synchronisiert werden.

Wird z.B. eine Operation direkt mittels der Webseite als auch über eine
SOAP Webservice-Schnittstelle (z.B. für eine mobile Applikation)
angeboten, müssen auf beiden Schnittstellen die gleichen
Zugriffsberechtigungen überprüft werden. Häufig kann man während
Penetration-Tests verminderte Schutzmaßnahmen bei
Webservice-Schnittstellen feststellen.

#### Probleme bei Verwendung von WebSockets

WebSockets unterliegen nicht der Same-Origin-Policy moderner Webbrowser.
Es wird daher empfohlen, bei öffnen des WebSockets serverseitig den
*Origin*-Header auf valide aufrufende Webseiten hin zu überprüfen.
Während des initialen Handshakes wird ein *Web-Socket-Key* übertragen.
Dieser dient nur zur Identifikation des Browsers gegenüber dem Webserver
und darf nicht zu Authentications- bzw. Authorization-Zwecken verwendet
werden.

Über WebSockets werden zumeist Nachrichten verschickt. Der Server wird
auf den Erhalt einer Nachricht hin eine Operation starten, vor dieser
muss der Server eine Berechtigungsüberprüfung durchführen. Die
Berechtigungen zwischen WebSockets und HTTP-basierte Kommunikation
müssen auf jeden Fall synchron gehalten werden (siehe auch, Different
Authorization in Alternate Channels). Ein häufiges Problem ist, dass der
Client vor der Authentication des WebBrowsers gegenüber dem Client
bereits einen WebSocket öffnet. In diesem Fall wird meistens eine
parallele Session-Verwaltung auf server-seite aufgebaut: innerhalb einer
Serverdatenbank wird für den WebSocket-Client eine Session-Id oder eine
Token-Id generiert, in der Datenbank der betreffende Web-Benutzer dem
Token zugeordnet und dem Client das token/die session mitgeteilt. Der
Webbrowser muss nun bei jeder WebSocket Anfrage dieses Secret mit
übertragen und der Server kann den User über dieses Token
identifizieren.

### Hint: Update Operationen

Anhand eines Beispiels soll gezeigt werden, warum auch einfache
Operationen komplexe Sicherheitsfragen aufwerfen können. Bei dem
konkreten Beispiel sollen Benutzerdaten aktualisiert werden. Hierfür
wird folgende Update-Operation aufgerufen:

    POST /user/update/1 HTTP/1.1

Als Parameter wird ein JSON-String mit den neuen Werten übergeben:

    {
        "id" : "1",
        "name" : "happe"
    }

Bei diesem Beispiel fallen folgende sicherheitsrelevanten Fragen an:

-   kann ich durch Setzen einer anderen ID (statt 1) in der URL auf
    einen anderen Datensatz schreibend zugreifen?

-   was passiert, wenn man die ID im Datensatz ändert? Teilweise
    überprüfen Webapplikationen nur die ID innerhalb der URL und
    ignorieren die IDs innerhalb des Datensatzes. Mit Glück kann man
    diesen Missmatch zum Überschreiben anderer Datensätze verwenden.

-   was passiert, wenn im Datensatz keine ID vorkommt und der Angreifer
    manuell ein ID-Element in das JSON-Dokument hinzufügt?

-   was passiert, wenn der Angreifer ein neues JSON-Element namens
    *“Admin”: “true”* hinzufügt?

-   was passiert, wenn der Angreifer statt HTTP POST eine HTTP GET
    Operation verwendet? HTTP GET sollte eigentlich eine read-only
    Operation sein, deswegen werden GET requests teilweise von
    Web-Application Firewalls nicht kontrolliert und man kann auf diese
    Weise Firewall-Regeln umgehen.

### Mass-Assignments

Moderne Web-Frameworks versuchen die Effizienz von Programmierern zu
verbessern. Ein Feature, welches potentiell negativen Einfluss auf die
Sicherheit einer Applikation besitzt ist *mass assignments*.

Unter Mass-Assignment versteht man das automatisierte Zuweisen von
Werten aus einem HTTP Request zu einem Datenbank-Objekt. So könnten z.B.
bei einem User-Update die übergebenen HTTP-Parameter automatisch
gegenüber den bekannten Datenbankfeldern gematched werden und Parameter
wie z.B. *vorname* oder *nachname* werden automatisch auf das
Datenbankfeld *vorname* und *nachname* des betroffenen Datensatzes
gemapped und aktualisiert.

In Ruby on Rails würde der betroffene Code folgendermaßen aussehen:

        @user = User.find(params[:id])
        @user.update(params[:user])

Die erste Zeile des Beispiels verwendet den übergebenen *id* Parameter
um aus der Datenbank ein User-Objekt zu laden. In der zweiten Zeile
werden nun die vorhandenen HTTP-Parameter automatisch den vorhandenen
Feldern des User-Objektes zugewiesen.

Dies ist aus Sicherheitssicht problematisch. Ein Angreifer kann
versuchen potentielle Datenbankfelder zu erraten und diese mittels
mass-assignment zuzuweisen. Beispiele wären z. b. das Setzen von
*role=admin* oder *admin=true*.

Um dies zu verhindern besitzen die meisten Frameworks ein Möglichkeit
Attribute für das Mass-Assignmentexplizit zu verbieten (black-list) oder
zu erlauben (white-list). Aus Sicherheitssicht ist das automatische
Ablehnen von Attributen und die explizite Freigabe einzelner Attribute
(also das White-Listing) vorzuziehen.

In Ruby on Rails würde der betroffene Code folgendermaßen aussehen:

        @user = User.find(params[:id])
        @user.update(params.require(:user).permit(:full_name))

In diesem Fall werden nur die Felder *full\_name* für das Objektes
*user* mittels mass-assignment aktualisiert.

## Scoping von Daten

Unter Scoping von Daten versteht man das Einschränken der verfügbaren
Daten auf einen Subbereich. Eine Web-Operation wird meistens für einen
Benutzer aufgerufen, die potentiell verfügbaren Daten sollten so früh
wie möglich auf die für den Benutzer verfügbaren Daten eingeschränkt
werden.

Beispiel: eine Applikation verwaltet Rechnungen, jede Rechnung hat einen
Benutzer als Autor. Mittels einer Update-Operation kann eine Rechnung
bearbeitet werden. Dies geschieht mittels der Operation
<a href="/invoice/1/update" class="uri">/invoice/1/update</a>, in der
Applikation ist der gerade angemeldete autorisierte User als
*current\_user* bekannt, mittels *current\_user.invoices* kann man auf
die Rechnungen des aktuellen Users zugreifen, mittels *Invoice* auf alle
Rechnungen die dem System bekannt sind.

Die Update-Operation sollte nun folgendermaßen aussehen:

        # hier sollte NICHT Invoice.find(params[:id]) stehen
        @invoice = current_user.invoices.find(params[:id])

        # normaler Update-Code
        @invoice.update(the_data_which_will_be_updated)
        @invoice.save

Durch die Verwendung des User-Scopes wird implizit der Zugriff auf
Rechnungen des aktuellen Benutzers erzwungen. Dadurch müssen
Zugriffsberechtigungen nicht zusätzlich explizit kontrolliert werden.

## Time of Check, Time of Use (TOCTOU)

Der Zeitpunkt der Berechtigungsüberprüfung ist essentiell. So genannte
*Time of Check, Time of Use* Angriffe nutzen racing conditions zwischen
der Überprüfung von Operationsberechtigungen und der Ausführung von
Operationen aus.

Ein gutes Beispiel für die TOCTOU-Problematik sind zumeist
Token-basierte Systeme. Hier wird die Berechtigung eines Anwenders
überprüft und danach ein Token mit einer definierten Laufzeit
ausgestellt. Wird das Token während der Laufzeit an eine Operation
überreicht, wird dieses valide Token als Zugangsberechtigung akzeptiert
und die Operation ausgeführt auch wenn zwischenzeitlich die
Berechtigungen des Benutzer modifiziert wurden und der Benutzer die
Operation eigentlich nicht mehr ausführen dürfe.

## Reflektionsfragen

1.  Erkläre den Unterschied zwischen Identifikation, Authentication und
    Authorization?

2.  Was versteht man unter Authorization? Wann und wo sollte diese
    durchgeführt werden? Welches Sicherheitsproblem versteht man unter
    Insecure Direct Object Reference bzw. unter Forced Browsing?

3.  Welche Probleme können im Zusammenhang mit Mass-Assignment
    auftreten?

4.  Gegeben ein Webshop mit einem Downloadlink für Rechnungen
    <http://shop.local/invoices/1/download>. Welche
    sicherheitsrelevanten Fehler können hier nun auftreten?

5.  Gegeben eine Profil-Updatefunktion welche als POST
    <a href="/user/1/update" class="uri">/user/1/update</a>
    implementiert wurde, als Parameter werden die Felder *id*, *email*,
    *new\_password* und *rolle* (mit Wert *user*) übergeben. Erkläre
    zumindest drei Sicherheitsprobleme die während des Updates eines
    Benutzers auftreten können.
