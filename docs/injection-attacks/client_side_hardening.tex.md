# Clientseitige Schutzmaßnahmen

## Integration externer Komponenten

Werden externe Inhalte innerhalb der eigenen Seite inkludiert, erhöht
sich die Angriffsfläche: ein Angreifer mit Zugriff auf die externe Seite
kann über diese Schadcode in die, eigentlich sichere, eigene Webseite
einschleusen. Falls die Einbindung externer Inhalte zwingend benötigt
wird, kann das Gefahrenpotential durch Einsatz folgender Techniken
reduziert werden:

### IFrame: sandbox-Flag

Wird eine externe Resource über das HTML *iframe* Tag eingebunden kann
durch Verwendung des *sandbox*-Attributes die Sicherheit erhöht werden.
Bei Verwendung dieses Attributes werden folgende Einschränkungen
aktiviert:

-   JavaScript wird für die eingebundene Resource deaktiviert.

-   Die eingebundene Seite bekommt einen eigenen Origin; dadurch kann
    dieses nicht mehr auf die einbindende Seite zugreifen, auch wenn
    diese auf dem identen Server abgelegt waren.

-   Die eingebundene Seite kann keine neuen Fenster bzw. Dialoge öffnen.

-   Es können keine Formulare abgeschickt werden.

-   Plugins werden für die eingebettete Seite deaktiviert.

-   Autoplay wird deaktiviert.

Die Einschränkungen können durch mehrere Optionen aufgeweicht werden,
der Namen dieser Optionen beginnt mit *allow-*. Ein abschließendes
Beispiel für einen per Iframe eingebundenen Twitter-Button:

    <iframe sandbox="allow-same-origin allow-scripts allow-popups allow-forms"
    src="https://platform.twitter.com/widgets/tweet_button.html"
    style="border: 0; width:130px; height:20px;"
    </iframe>

### Subresource Integrity (SRI)

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

    <script src="https://example.com/example-framework.js"
    integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
    crossorigin="anonymous">
    </script>

Mittels CSP kann die Verwendung von Subresource Integrity erzwungen
werden, folgendes Beispiel erzwingt die Angabe von Hashsummen für alle
inkludierte JavaScript- und CSS-Dateien.

    Content-Security-Policy: require-sri-for script style;

## Referrer-Policy

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

no-referrer  
: der Referer-Header wird nicht übertragen.

no-referrer-when-downgrad  
: die URL wird als Referer übertragen sofern die Folgeseite nicht ein
unsichereres Protokoll verwendet. Dadurch wird z.B. eine Übertragung der
URL beim Übergang von HTTPS zu HTTP verboten. Dies ist häufig das
Default-Verhalten der Webbrowser.

origin  
: es wird immer nur der Origin übertragen.

origin-when-cross-origin  
: es wird die volle URL übertragen, sofern man sich innerhalb des
identen Origins befindet. Falls es zu einem Origin-Wechsel kommt (z.B.
durch Navigation auf eine externe Seite) wird nur der Origin übertragen.

same-origin  
: solange man sich innerhalb des Origins befindet, wird die URL gesetzt,
ansonsten wird kein Referer-Header versendet.

strict-origin  
: es wird immer nur der Origin als Referer versendet, und dies auch nur
falls das Sicherheitslevel ident ist (es wird also beim Übergang von
einer HTTPS auf eine HTTP Seite kein Referer gesetzt).

strict-origin-when-cross-origin  
: es wird die volle URL innerhalb des identen Origin verwendet, wird auf
Webseiten mit unterschiedlichen Origin zugegriffen (also z.B. beim
Übergang auf externe Webseiten) wird nur der Origin als Referer
verwendet. Falls die Sicherheit der Kommunikation schlechter wird (also
z.B. beim Übergang von HTTPS auf HTTP) wird überhaupt kein
Referer-Header versendet.

unsafe-url  
: die gesamte URL wird im Referrer-Header immer übertragen.

Es wird empfohlen, eine *Referrer-Policy* zu wählen, die nicht
*no-referrer-when-downgrade* bzw. *unsafe-url* ist.

## Content-Security-Policy

<span id="csp" label="csp"></span>

Die Content-Security-Policy (CSP) erlaubt es, eine umfangreiche Policy
von Webservern an Browser zu übertragen. Prinzipiell sind mittels einer
CSP viele der bereits erwähnten Security-Header abbildbar.

Die Content-Security-Policy kann entweder als HTTP-Header oder als Teil
des HTML-Dokuments übertragen werden. Der verwendete HTTP Header heißt
*Content-Security-Policy*, bei Verwendung eines Meta-Tags würde der Code
beispielsweise folgenderweise aussehen:

    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; img-src https://*; child-src 'none';">

Die grundlegende Funktionalität von CSP ist:

-   Definition von vertrauenswürdigem Javascript bzw. Javascript-Sourcen

-   Sicherstellen, dass Seitenelemente (CSS, Bilder, etc.) nur aus
    vertrauenswürden Quellen bezogen werden.

-   Definition der Interaktion mit externen Seiten (z.B. mittels
    iFrames)

-   Sonstiges: Erhöhung der Verbindungssicherheit, etc.

### Trennung von HTML und JavaScript-Code

Mittels CSP wird zumeist definiert aus welchen Quellen Javascript-Code
geladen werden darf. Damit dadurch ein gutes Sicherheitsniveau erreicht
werden kann, ist eine Trennung vom JavaScript-Code von den verwendeten
HTML-Dateien notwendig. Falls Javascript innerhalb von HTML Dateien
erlaubt ist, kann schwer unterschieden werden ob vorgefundener
JavaScript-Code von der Applikation vorgesehen oder durch einen
Angreifer eingeschleust worden ist.

Diese Trennung wird erreicht, wenn der gesamte JavaScript-Code in
getrennten JS-Dateien bereitgestellt wird. Dieser wird in die HTML-Seite
mittels einem *script*-Tag eingebunden:

    <script src="externalfile.js"></script>

In *externalfile.js* wird nun der JavaScript-Code hinterlegt. Da
a-priori keine Verbindung zwischen dem JavaScript-Code und dem HTML-File
besteht, wird zumeist der *$(document).ready*-Callback verwendet. Code
in diesem Callback wird ausgeführt, sobald das DOM fertig geladen wurde:

    $(document).ready(function()  {
        // binden des JavaScript-Codes an etwaige Elemente
        document.getElementById("btn").addEventListener('click', doSomething);

        // other Javascript code
    });

Bei diesem Beispiel wird nun durch die Methode *addEventListener* die
JavaScript-Methode *doSomething* beim Klicken auf den Button mit der Id
*btn* aufgerufen.

### Verfügbare CSP-Elemente

CSP besteht aus mehreren Direktiven, die Tabelle
<a href="#tbl:csp_elements" data-reference-type="ref"
data-reference="tbl:csp_elements">1.1</a> gibt eine kurze Übersicht der
Möglichkeiten. Alle Direktiven die mit *-src* enden erlauben die
Verwendung ähnlicher Source-Werte, Tabelle
<a href="#tbl:csp_values" data-reference-type="ref"
data-reference="tbl:csp_values">1.2</a> gibt ein paar Beispiele. Durch
die Verwendung von “unsafe-inline” wird die Trennung zwischen HTML und
JavaScript nicht mehr erzwungen und daher der Großteils des XSS-Schutzes
neutralisiert. Diese Direktive sollte daher soweit wie möglich vermieden
werden.

| Name                         | Beschreibung                                                                                                                                              |
|:-----------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------|
| default-src                  | definiert die default Policy zum Laden von remote Elementen für die meisten Elemente.                                                                     |
| script-src                   | definiert vertrauenswürde Quellen für geladene JavaScript Dateien.                                                                                        |
| style-src                    | definiert vertrauenswürde Quellen für geladene CSS-Dateien.                                                                                               |
| plugin-types                 | object-src: definiert erlaubte Plugin Typen und deren vertrauenswürde Sourcen                                                                             |
| img-src, media-src, font-src | definiert vertrauenswürdige Quellen für die jeweiligen Dateitypen                                                                                         |
| child-src (ehem. frame-src)  | definiert erlaubte Quellen für den Inhalt verwendeter Iframes.                                                                                            |
| sandbox                      | aktiviert eine Sandbox für die aktuelle Resource ähnlich wie die Sandbox eines eingebetteten Iframas. Durch Optionen kann die Sandbox aufgeweicht werden. |
| connect-src                  | Wird von JSONP, WebSockets und EventSource verwendet.                                                                                                     |
| form-action                  | welche URIs dürfen als Ziel eines Formulars dienen                                                                                                        |
| reflected-xss                | entspricht X-XSS-Protection                                                                                                                               |
| frame-ancestors              | definiert, welche externen Seiten die Resource im Zuge eines Iframes verwenden dürfen, entspricht ca. einem *X-Frame-Option*s.                            |
| referrer                     | ähnlich wie Referrer-Header                                                                                                                               |
| report-uri                   | CSP erlaubt die Angabe einer Reporting-URL. Im Fehlerfall wird an diese URL eine detaillierte Fehlermeldung reported.                                     |
| block-all-mixed-content      |                                                                                                                                                           |
| upgrade-insecure-requests    |                                                                                                                                                           |

Häufig verwendete CSP-Direktiven

| Name            | Beschreibung                                                                                                                                                                                       |
|:----------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| \*              | erlaubt alle URIs ausgenommen *data:*, *blob:* und *filesystem:*                                                                                                                                   |
| ’none’          | verbietet das Laden von Ressourcen.                                                                                                                                                                |
| ’self’          | erlaubt das Laden von Ressourcen vom eigenen Origin                                                                                                                                                |
| data:           | erlaubt das Bereitstellen von Ressourcen über data (base64-codierte Daten).                                                                                                                        |
| domain          | erlaubt das Laden von Daten von der entsprechenden Domain. Wildcards für Subdomains dürfen verwendet werden. Wird der Domainname mit *https://* begonnen, muss HTTPS beim Zugriff verwendet werden |
| https           | erzwingt das Laden von Ressourcen über HTTPS, alle Domains sind erlaubt.                                                                                                                           |
| ’unsafe-inline’ | erlaubt inline Javascript als auch *javascript:* URIs                                                                                                                                              |
| ’unsafe-eval’   | erlaubt unsichere dynamische Code-Exekution mittels eval.                                                                                                                                          |
| ’nonce-(nonce)  | *script*- oder *style*-Tags werden exekutiert sofern diese ein nonce-Attribut besitzen welches ident zu dem Wert innerhalb des CSP ist.                                                            |
| sha256-(hash)   | Erlaubt die Exekution von Skripts sofern ihr Hash dem in der CSP angegeben Hash entsprechen.                                                                                                       |

Häufig verwendete CSP Source-Werte

### CSP-Nonces

CSP-Nonces erlauben die Integration von CSP ohne die gesamte
Web-Applikation nach dem Grundsatz der strikten Trennung von JavaScript
und HTML entwickelt zu haben. Dieses System basiert darauf, dass
innerhalb des CSP-Headers eine Nonce (zufällige einmalige Zahl)
definiert wird und innerhalb der HTML Datei nur *Script*- und
*Style*-Elemente nur exekutiert werden, wenn diese einen identen Wert
als *nonce*-Attribut gesetzt haben.

Wird zum Beispiel folgender CSP-Header verwendet:

    Content-Security-Policy: script-src 'nonce-2726c7f26c'

würde das Skript nur exekutiert werden, wenn es die idente nonce
(*2726c7f26c*) innerhalb des *script*-Tags verwendet:

    <script nonce="2726c7f26c">
      var inline = 1;
    </script>

Die Sicherheit dieses Verfahrens ist von zwei Annahmen abhängig:

1.  Der Angreifer darf die nonce nicht vorherbestimmten können. Ebenso
    muss bei jedem Seitenaufruf eine neue nonce generiert werden.

2.  Der Angreifer darf nicht die Möglichkeit besitzen, innerhalb eines
    sicheren Skript-Aufrufs (bei dem die richtige nonce gesetzt ist)
    bösartigen JavaScript-Code einzufügen.

### CSP-Beispiele

Ein einfaches Beispiel, welches das Laden von Ressourcen (Javascript,
CSS, Images) nur vom eigenen Server erlaubt:

    Content-Security-Policy: default-src 'self'

Folgendes Beispiel schränkt mögliche Angriffsvektoren bereits stark ein:

    Content-Security-Policy:
      object-src 'none';
      script-src 'nonce-{random}' 'unsafe-inline' 'strict-dynamic' https: http:;
      base-uri 'none';

Folgende Einstellungen werden dadurch an den Browser übermittelt:

-   *object-src: none* verhindert das Laden von Plugins wie z.B. Flash.

-   Die verwendete script-src Line verwendet “neues” als auch “altes”
    CSP damit unterschiedliche Browserversionen sichere CSP
    Einstellungen bekommen. Die Kombination von *nonce* und
    *unsafe-inline* bewirkt bei neueren Browsern, dass *script*-Tags nur
    verwendet werden, wenn die angegebene Nonce bei dem Skript-Tag als
    Attribut hinterlegt ist. Neuere Browser ignorieren in dem Fall
    *unsafe-inline*. Ältere Browser ignorieren die Nonce, über
    *unsafe-inline* werden allerdings “normale” script-Tags erlaubt.
    *strict-dynamic* erlaubt das Laden von remote JavaScript-Dateien
    ausgehend von trusted Scripts. Moderne Browser ignorieren die
    zusätzlichen Schemas (http und https), während ältere Browser die
    kein *strict-dynamic* erkennen durch die Schemas externe
    JavaScript-Dateien laden können.

-   *base-uri: none* deaktiviert das HTML *base*-Element welches im
    Zusammenhang mit relativen Imports verwendet werden kann, um
    Injection-Angriffe durchzuführen.

## Reflektionsfragen

1.  Was ist HTML Subresource Integrity (SRI), wie wird diese verwendet
    und gegen welche Angriffe schützt diese Maßnahme?

2.  Wie werden CSP verwendet? Gib ein Beispiel für CSPs? Wie
    funktionieren script-src nonces?

3.  Was ist die Referrer-Policy und warum sollte diese verwendet werden?

[1] Achtung: während der Header aufgrund eines Rechtschreibfehlers
*Referer* heißt, heißt der Policy Header *Referrer-Policy*.
