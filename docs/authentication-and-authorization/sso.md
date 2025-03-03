# Federation/Single-Sign on

Werden mehrere Webapplikationen betrieben, entsteht schnell der Wunsch,
Benutzerkonten zwischen diesen Applikationen zu synchronisieren. Die
Grundidee ist es, ein unified Authentication/Authorization-Konzept über
mehrere Server hinweg zu implementieren. Potentielle Gründe für den
Einsatz einer Single-Sign On oder Federation Lösung sind:

- SSO erlaubt es mehreren Applikationen eine gemeinsame
  Login/Logout-Lösung zu verwenden. Dadurch können redundante Lösungen
  eingespart und duplizierte Sicherheitsprobleme vermieden werden.
  Nachteil: der Login-Server ist ein Single-Point-of-Failure.

- Das gesamte Passwort-Management kann aus der Webapplikation
  ausgelagert werden. Es müssen keine Passwörter mehr selbst erhoben,
  bearbeitet oder gespeichert werden.

- Die User-Experience ist angeblich besser. Der Aufwand für einen
  Benutzer einen neuen Account anzulegen wird minimiert.

- Durch den Login-Server können weitere Authenticationsservices
  implementiert worden sein, z.B. Ausweiskontrolle oder eine
  Multi-Faktor-Authentication.

## Festival-Beispiel

Eine gute Analogie ist ein Musikfestival bei welchem Besucher auf dem
Festivalgelände Getränke erwerben können. Je nach Alter darf ein Kunde
alkoholische oder nicht-alkoholische Getränke kaufen. Müsste nun jeder
Getränkestand bei jeder Bestellung Eintrittskarte und den Ausweis
(Altersnachweis) des Besuchers kontrollieren, würde dies zu starken
Verzögerungen führen.

Um die Situation zu verbessern, werden beim Festivaleingang die Besucher
einmalig bei einem Registrationszelt kontrolliert. Jeder Besucher erhält
ein Armband um zu beweisen, dass er eine Eintrittskarte besass. In
Abhängigkeit vom Alter bekommen Minderjährige Besucher ein blaues
Armband, erwachsene Besucher ein rotes Armband. Anhand dieses Armbands
(Token) können nun die Getränkestände schnell kontrollieren, ob ein Gast
alkoholische Getränke konsumieren darf. Zusätzlich wird über den Besitz
des Bands überprüft, ob ein Besucher eine Eintrittskarte besass. Falls
ein Besucher bei einem Getränkeshop ohne Band ein Getränk kaufen will,
wird er zu dem Registrationszelt verwiesen.

In diesem Beispiel ist das Registrationszelt der Identity Provider, die
Getränkehändler sind Service Provider, das Armband ein Token und der
Besucher der Client.

Das Beispiel zeigt auch zwei Probleme von token-basierten Lösungen. Das
Armband gilt für die gesamte Dauer des Festivals (im Folgejahr werden
andere Farben verwendet). Es gibt keine Möglichkeit einen Teil der
Armbänder nach dem ersten Festivaltag zu invalidieren. Ebenso wird klar,
dass jegliche Form von Zugriffskontrolle durch das Armband ersetzt wird.
Wollen z.B. ein Vater und Sohn beide Alkohol kaufen, kann der Vater
initial den Identitätscheck durchführen und dann sein Armband an seinen
Sohn weitergeben. Der Vater geht danach wiederholt zum Registrationszelt
und kauft sich ein zweites Zugangsband. Mit dem ersten Band kann nun der
Sohn beliebig Alkohol kaufen ohne dass auffällt, dass sein Alter dies
eigentlich verbieten sollte. Der alleinige Zeitpunkt der Überprüfung
(Authorization) geschieht während der Bandausgabe.

## OAuth2

[OAuth2](https://tools.ietf.org/html/rfc6749) erlaubt es einem Benutzer (Resource
Owner) einer Applikation (Client) Zugriff auf Resourcen/Operationen auf
einem Server (Resource Server) zu erteilen. Ein Authorization Server
wird verwendet um ein Zugriffs-Token für einen definierten Bereich
(scope) am Resourcen-Server auszustellen. Zusätzlich zu dem
Zugriffs-Token wird zumeist auch ein Refresh-Token ausgestellt mit dem
ein Client ein neues Zugriffstoken ohne Benutzerinteraktion generieren
kann.

Da die Rechte eines ausgestellten Token im Normalfall nur während der
Ausstellung überprüft werden und danach für die gesamte Lebenszeit des
Tokens gültig sind, wird bestenfalls eine sehr kurze Lebenszeit im
Minutenbereich gewählt. Läuft das Token ab, kann mit den Refresh-Token
ein neues Access-Token angefordert werden. Da hierfür keine
Benutzerinteraktion benötigt wird, kann dies automatisiert und
transparent für den Endbenutzer erfolgen. Der Vorteil liegt darin, dass
während der Ausstellung durch den Authorisationsserver die angeforderten
Rechte des Tokens wiederholt überprüft werden.

![Beispiel für einen OAuth2-Fluss](/images/oauth2.svg)

Ein interessanter Aspekt ist der Zeitpunkt der Authorization: die
Überprüfung der eigentlich Zugriffsberechtigung wird durch den
Authorization-Server zum Zeitpunkt der Ausstellung des Tokens
durchgeführt. Bei einem Zugriff auf den Resourcen Server werden die
eigentlichen Berechtigungen nicht mehr überprüft, sondern nur noch
getestet ob das übergebene Token Zugriff auf die angeforderten Resourcen
inkludiert und von einem validen Authorization Server signiert wurde.

Das Token besitzt eine Laufzeit und ist bis zum Ende der Laufzeit
gültig. Da der Resource Server nicht direkt mit dem Authorization Server
kommuniziert, gibt es keine Möglichkeit ein Token zuvorig zu
invalidieren. Dies ist problematisch, falls eine lange Laufzeit (z.B.
ein Jahr) gewählt wurde und ein Token abhanden gekommen ist. Ein
Angreifer mit dem entwendeten Token kann nun bis zum Ende der Laufzeit
dieses Token verwenden um auf die Resource zuzugreifen.

Um dieses Problem zu entschärfen werden zumeist zwei Tokens generiert:
ein Access-Token und ein Refresh-Token. Das Access-Token wird zum
Zugriff auf den Resource Server verwendet und besitzt eine sehr kurze
Laufzeit, zumeist im Minuten-Bereich. Falls ein Access-Token abgelaufen
ist, kann der Client das Refresh-Token verwenden um (ohne
Benutzerinteraktion) ein neues Access-Token zu erhalten. Dies verbessert
die Sicherheitssituation, da der Authorization-Server vor dem Ausstellen
eines Access-Tokens überprüft, ob das Subject/der User überhaupt noch
die notwendige Berechtigung besitzt. Dadurch wird das verwundbare
Zeitfenster zwar nicht entfernt, aber zumindest reduziert.

## OpenID Connect

OpenID Connect verwendet OAuth2 um eine Benutzerauthentication
durchzuführen. Es gibt verschiedene Subprotokolle (*flows* genannt). Im
Allgemeinen funktioniert das OpenID Connect Protokoll auf folgende
Weise:

1. Der Client schickt einen Request zu dem OpenID Provider.

2. Der OpenID Provider authentifiziert den Benutzer, der Benutzer
   bestätigt den Authentication Request.

3. Der OpenID Provider returniert einen ID Token (und zumeist auch
   einen Access Token).

4. Der Cient kann das Access Token verwenden um weitere Informationen
   über den User über den *UserInfo Endpoint* zu erhalten.

Das ID Token ist ein JSON Web Token (JWT, siehe auch Kapitel
<a href="#jwt" data-reference-type="ref" data-reference="jwt">[jwt]</a>,
Seite ), folgende Felder müssen in diesem ausgefüllt werden:

iss  
: der Aussteller des Tokens. Dieser muss ein https-Endpunkt sein.

sub  
: der subject identifier identifiziert den Benutzer.

aud  
: der Identifier für den Server, der die Authentification anforderte.

exp  
: Ablaufdateum des Tokens.

iat  
: Austellungsdatum des Tokens.

OpenID Connect definiert drei verschiedene flows (*code*, *implicit*
oder *hybrid*), ihre Unterschiede werden kurz in Tabelle
<a href="#tbl:oidc" data-reference-type="ref"
data-reference="tbl:oidc">1.1</a> aufgeführt. Für ,,normale”
Applikationen wird die Verwendung des *code* Flows empfohlen.

| Eigenschaft                                  | Code | Implicit | Hybrid    |
|:---------------------------------------------|:-----|:---------|:----------|
| Authorization Endpoint versendet alle Tokens | nein | ja       | nein      |
| Token Endpunkt versendet alle Tokens         | ja   | nein     | nein      |
| User Agent erhält Tokens                     | ja   | nein     | nein      |
| Client kann authenticated werden             | ja   | nein     | ja        |
| Refresh Tokens können verwendet werden       | ja   | nein     | ja        |
| Kommunikation geschieht in einem Roundtrip   | nein | ja       | nein      |
| Großteils Server-zu-Server Kommunikation     | ja   | nein     | teilweise |

Übersicht über die verschiedenen OpenID Connect Flüsse

## SAML2

Die Abkürzung SAML2 steht für Security Assertion Markup Language
(Version 2). Diese XML-basierte Sprache dient zum Austausch von
Authentication und Authorization Informationen zwischen mehreren
Parteien. Dabei will sich ein Benutzer mittels eines Clients (z.B.
Webbrowser) an einem Service Provider (z.B. Webserver) anmelden. Um dies
durchzuführen wird ein Identity Provider (IdP) bemüht dieser ist ein
Service welches für einen User gegenüber einem Service Provider
authentifiziert und autorisiert; ebenso kann dieser Service einen
synchronen Single Sign-Out durchführen.

Die jeweiligen Operationen werden im SAML2 Jargon häufig *Flows*
genannt. Es gibt Login- und Logout-Flows, beide können entweder vom
Service Provider oder vom Identity Provider gestartet werden. Diese
unterschiedliche Ausprägung ist durch unterschiedliche Use-Cases
bedingt. Falls ein Betrieb mehrere Websysteme betreibt, die eigenständig
sind (z.B. eine GitLab-Instanz, eine NextCloud-Instanz), diese aber mit
einem unified Sign-In versehen will, macht der SP-trigered flow Sinn.
Der Benutzer wird beim Login auf z.B. GitLab zu dem IdP weitergeleitet
und loggt sich auf diesem ein. Es wird eine Bestätigung für GitLab
generiert (Token) und der User wird automatisch mit diesem Token zu dem
GitLab-Server weitergeleitet (auf dem er nun eingeloggt ist). Den
IdP-triggered flow würde man eher in einem Portal-Umfeld verwenden: hier
gibt es eine initiale Login-Seite und dem Benutzer wird danach ein
typisches Portal mit mehreren eigenständigen aber integrierten
Applikationen angezeigt. Wenn er nun auf eine Subapplikation klickt,
wird das Token automatisch mit übertragen und der User ist in der
Subapplikation eingeloggt (ohne zuvor vom SP zum IdP umgeleitet zu
werden).

### SAML2 Assertions

Das Herzstück von SAML2 sind die Security Assertions die vom IdP
ausgestellt werden. Eine solche Assertion beschreibt die Rechte, welche
ein User auf einem SP besitzt. Die Assertion wird vom IdP mittels einer
public-key basierten Signatur unterschrieben.

Typische Elemente einer Assertion wären:

- *Issuer* identifiziert den IdP der diese Assertion ausgestellt hat.

- *Signature* beinhaltet die Signatur welche die Integrität der
  Security Assertion sichert.

- *Subject* beschreibt das identifizierte Objekt, in diesem Fall den
  identifizierten User. Der verwendete Identifier (*NameId*) kann
  verschiedene Typen besitzen, häufig wird *transient* verwendet.
  *transient* beschreibt einen kurzfristigen Identifier, ähnlich einer
  Session-Id, und besitzt den Vorteil, dass auf diese Weise der SP
  nicht die genaue Identität des Subjects erfährt.

- Conditions: beliebig viele Conditions welche den Anwendungsbereich
  der Assertion beschränken. Beispiel sind z. b. temporale
  Beschränkungen (*NotBefore*, *NotOnOrAfter*) oder eine Einschränkung
  der Service für welche die Assertion gültig sein soll.

- AttributeStatement: beliebig viele Attribute-Statements welche
  optionale Daten an die Assertion anhängen.

- *AuthnStatement* beschreibt die Assertion selbst und beinhaltet
  einen eindeutigen Identifier für die Assertion (*SessionIndex*).
  Dieser Identifier wird häufig im Zuge des Sign-Out zur
  Identifikation der betroffenen Session verwendet.

Bei einem realen Deployment kann die Situation auftreten, dass mehrere
Identity Provider verfügbar sind und der Service Provider den korrekten
IdP selektieren muss. Ein Beispiel wäre ein Unternehmen welches interne
User gegen einem Active Directory und externe User gegen einen
öffentlichen IdP authentifiziert.

Um die Selektion des IdPs zu vereinfachen, gibt es das IdP Discovery
Protokoll. Die beiden häufigen Arten des IdP Discoveries sind:

- IdP Discovery am SP: der SP selbst kann die User einem IdP zuordnen
  und weiß daher, welchen IdP er kontaktieren soll.

- Delegated IdP Discovery: der SP leitet die Anfrage an einen eigenen
  IdP Discovery Service weiter. Dieser identifiziert den zu wählenden
  IdP und retourniert diese Information an den SP. Bei diesem
  Protokoll muss erwähnt werden, dass die gesamte Kommunikation über
  den Client läuft: der SP teilt dem Client mit, dass dieser per HTTP
  Redirect den IdP Discovery Service kontaktieren soll (auf diese
  Weise erhält der IdP Discovery Service die IP des Clients).

### Protocol Bindings

SAML2 dient zur Vereinheitlichung bestehender SSO-Lösung, daher wurde
beim Entwurf des Standards auf vielfältige Integrationsmöglichkeiten in
bestehende Netzwerke geachtet. Dementsprechend definiert SAML2 multiple
Transportprotokolle, sogenannte Bindings:

- HTTP Redirect Binding
- HTTP POST Binding
- HTTP Artifact Binding
- SAML SOAP Binding
- Reverse SOAP Binding
- SAML URI Binding

Bei Webbrowser-basierten Flows wird meistens das HTTP Redirect oder das
HTTP POST Binding verwendet. Bei dem Redirect binding werden die
übertragenen SAML Dokumente mittels Base64 codiert und als HTTP
Parameter innerhalb von HTTP Redirects verwendet. Da die Länge der
Parameter durch die jeweiligen Webbrowser limitiert ist, wird dieses
Verfahren vor allem für kurze Nachrichten verwendet. HTTP POST basierte
Verfahren verpacken die Nachrichten innerhalb von HTML Formularen und
umgehen dadurch die Größenlimitierung. Um den Fluss zu automatisieren,
werden die Formular zumeist mittels JavaScript automatisch versendet.

### SAML2-Beispiel: Single Sign-On

Abbildung <a href="#saml2_sso" data-reference-type="ref"
data-reference="saml2_sso">1.1</a> zeigt ein Beispiel für ein
SP-initiated Single-Sign On welches durch einen Service Provider
gestartet und mittels HTTP POST Binding implementiert wurde.

![Beispiel für ein Single-Sign On welches durch einen Service-Provider angestossen wurde](/images/saml2.svg)

In diesem Beispiel will ein Client (*User Agent*) auf einen Service
Provider zugreifen und benötigt hierfür eine Autorisierung. Nach dem
initialen Client-Zugriff (Schritt 1) verwendet der SP zusammen mit dem
Client das *IdP Discovery* Protokoll um den zugehörigen IdP zu
identifizieren. Sobald dieser bekannt ist, erstellt der SP einen
Authorization Request und teilt diesen (samt der Adresse des IdPs) dem
Client mit. Der Client kontaktiert nun den IdP und übermittelt den
Request.

Der IdP authentisiert und autorisiert nun den Client. Falls dies
erfolgreich durchgeführt wurde, wird eine SAML2 Security Assertion
ausgestellt, vom IdP signiert und dem Client mitgeteilt. Da dies ein
HTTP POST basierter Flow ist, erstellt der IdP ein HTML Formular,
inkludiert in diesem HTML Formular das generierte SAML-Dokument (hidden
field) und submitted das Formular automatisch mittels Javascript
(Schritt 4). Der Client greift nun auf den SP zu und übermittelt die
SAML assertion. Der SP verifiziert die Signatur und erstellt eine
Session basierend auf den Daten innerhalb der Assertion. Bei Schritt 6
wird (wahrscheinlich, ist implementierungsabhängig) ein Session Cookie
gesetzt, dass der Client bei allen weiteren Anfragen an den SP
verwendet. Auf diese Weise sind nun alle folgenden Requests
authentifiziert und autorisiert.

## Reflektionsfragen

1. Wie funktioniert der Sign-On Fluss bei SAML2?

2. Wie funktioniert der Authorisierungsfluss bei OAuth2?

3. Wie ist ein JSON Web Token aufgebaut? Welches Problem kann im
   Zusammenhang mit Verwechslungen der Signatur und er MAC-Adresse
   passieren?

4. Welche Rolle übernimmt das IdP Discovery Protokoll innerhalb von
   SAML2?

5. Gegeben eine SAML2 Example Assertion, was sagt diese aus (wer ist
   issuer? wer ist subject, etc.)?

6. Wie ist das Verhältnis zwischen OIDC und OAuth2?
