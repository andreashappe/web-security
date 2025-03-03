# Kryptographische Grundlagen

Kryptographie beschreibt die Technik (und Kunst) über
nicht-vertrauenswürdige Kanäle bzw. Speicherorte Daten integritäts- und
vertraulichkeitsgesichert zu übertragen. Dadurch kann Kryptographie als
Mittel gegen *spoofing*, *tampering*, *repudiation* und *information
disclosure* dienen. Dieses Kapitel soll eine (extrem) kurze Einführung
in die, in diesem Dokument, verwendeten Konzepte geben.

Bei der Verschlüsselung wird der ursprüngliche Text (häufig *plaintext*
genannt) durch den Algorithmus in einen neuen, verschlüsselten, Text
(häufig *ciphertext* genannt) konvertiert. Dieser Ciphertext kann durch
die Entschlüsselung wieder in den Plaintext zurück verwandelt werden.
Die hierbei verwendeten Algorithmen werden häufig *Cipher* genannt. In
der Literatur werden die dabei beteiligten Partein meistens ident
benannt: *Alice* und *Bob* sind die beiden Parteien die miteinander
sicher kommunizieren wollen. *Eve* ist ein Angreifer, der diese
Nachrichten abhören, aber nicht modifizieren kann; *Mallory* ist ein
Angreifer, der auch aktiv angreifen darf.

Grundlegend sollten folgende Grundsätze bei der Verwendung von
Kryptographie beachtet werden:

-   Niemals selbst ein kryptographisches System entwerfen, sondern immer
    ein etabliertes (und getestetes) System verwenden.

-   Niemals selbst einen kryptographischen Algorithmus/Bibliothek
    implementieren, sondern immer etablierte und getestet Komponenten
    verwenden.

-   Die richtige kryptographische Methode wählen.

-   Immer davon ausgehen, dass der eigene Source Code früher oder später
    öffentlich wird. Aus diesem Grund darf ein kryptographischer
    Schlüssel (oder auch Credentials) niemals Teil des Source Codes
    werden.

-   Essentiell zur sicheren Verwendung der verschiedenen
    kryptographischen Methoden sind die dabei verwendeten Schüssel.
    Diese müssen sowohl sicher gespeichert als auch transportiert
    werden. Noch komplexer ist das Herstellen eines Vertrauensverhältnis
    (Trust) zwischen den jeweiligen Kommunikationspartnern: woher weiss
    ein Partner, dass ein vorhandener Schlüssel eines anderen
    Kommunikationsparters vertrauenswürdig ist? Key Management ist
    komplex und sollte nicht unterschätzt werden!

Jede implementierte und konfigurierbare kryptographische Methode erhöht
potentiell die Angriffsoberfläche. Ein Beispiel hierfür ist z. B. die
OpenSSL-Bibliothek die dutzende Algorithmen implementiert. Als
Alternative sind in den letzten Jahren kryptographische Bibliotheken wie
*NaCl* (“salt”) entstanden, die für jede kryptographische Methode genau
eine sichere Implementierung anbieten. Auf diese Weise sollen
Selektionsfehler durch Entwickler vermieden werden.

Ein häufiger verwendeter Begriff ist *Rubber Hose Cryptography*. Ein
noch so technisch sicheres kryptographisches System kann durch bezahlte
Schläger mit einem Gummischlauch und der Androhung von Gewalt, falls das
Opfer nicht den privaten Schlüssel mitteilt, günstig gebrochen werden.
Anstatt durch Androhung von Gewalt kann *Rubber Hose Cryptography* auch
auf andere Aspekte eines Schlüsselträgers abzielen: Geld, Ideologie,
Coersion oder Ego (Sex sells).

## Verschlüsselung

Zur Wahrung der Vertraulichkeit von Daten wird Verschlüsselung
eingesetzt. Bei dieser wird der Originaltext (engl. *plaintext*) in
einen verschlüsselten Text (engl. *ciphertext*) konvertiert. Dieser kann
wieder durch den Entschlüsselungs-Vorgang in den Originaltext zurück
verwandelt werden. Verschlüsselungsalgorithmen können in zwei Familien
eingeteilt werden: symmetrisch und asymmetrisch (auch public-key
encryption genannt). Bei symmetrischer Verschlüsselung wird zum ver- und
entschlüsseln der idente Schlüssel verwendet. Problematisch hierbei ist,
dass dieser geteilte geheime Schlüssel initial zwischen allen
Beteiligten verteilt werden muss. Bei der asymmetrischen Verschlüsselung
wird statt einem geteilten Schlüssel ein Schlüsselpaar[1] verwendet.
Dieses besteht aus einem öffentlichen Schlüssel der zur Verschlüsselung
dient und einem zugehörigen privaten Schlüssel der zum Entschlüsseln
verwendet wird. Dadurch wird die Problematik des initialen
Schlüsselverteilens entschärft, da nur öffentliche Schlüssel verteilt
werden müssen (diese dürfen veröffentlicht bzw. verloren werden). Ein
Nachteil asymmetrischer Verschlüsselung gegenüber symmetrischer
Verschlüsselung ist, dass sie langsamer als symmetrische Verschlüsselung
ist.

## Block- und Stream-Cipher

Eine weitere Unterscheidungsmöglichkeit für Verschlüsselungsalgorithmen
ist die in *block* und *stream* ciphers. Bei Blockciphern werden zuerst
Daten angehäuft (“ein Block” an plain-data) und dann dieser Block
verschlüsselt. Bei einem Streamcipher wird jedes Zeichen sofort
verschlüsselt, das Sammeln von Blocken wird so vermieden. Während
Stream-Ciphers teilweise einfacher für Programmierer in ihrer Verwendung
sind, werden aus Effizienzgründen fast ausschließlich Blockcipher
verwendet. Werden zwei idente Blöcke mit dem identen Schlüssel
verschlüsselt, würden idente verschlüsselte Blöcke entstehen. Dies
erlaubt es einem Angreifer, strukturelle Informationen aus
verschlüsselten Dokumenten zu extrahieren. Um dies zu vermeiden werden
so genannte *Block Modes* verwendet um sicherzustellen, dass idente
plain-text Blöcke unterschiedliche cipher-text Blöcke produzieren. Bei
Auswahl des Block Modes sollten GCM-Modes (bzw. AEAD-Varianten)
bevorzugt und ECB bzw. CBC Modes vermieden werden.

## Integritätsschutz

Verschlüsselung gewährleistet nicht automatisch die Integrität der
verschlüsselten Daten. Hierfür müssen eigene Algorithmen verwendet
werden. Häufig vorgefunden werden Hashes, Message Authentication Codes
(MACs) und Signaturen. Vereinfacht ausgedruckt berechnen Hashes
ausgehend von beliebig langen Eingangsdaten eine Checksumme konstanter
Größe. Wird ein Hash auf identen Eingangsdaten angewandt, wird auch ein
identer Hash berechnet. Ein Hash ist eine Einwegfunktion: während der
zugehörige Hash zu einem Eingangsdatum schnell berechnet werden kann
(gegeben den ursprünglichen Daten), ist das Berechnen der Eingangsdaten
ausgehend von einem Hash realistisch nicht möglich.

Bei einem Message Authentication Code (MAC) wird der Hash um ein
geheimes geteiltes Passwort erweitert. Zur Berechnung bzw. Validierung
eines MACs wird dieses Passwort benötigt. Analog zur symmetrischen
Verschlüsselung ergibt sich hier die Problematik der
Schlüsselverteilung. Signaturen lösen dieses Problem indem sie
asymmetrische (public-key) Verschlüsselung einsetzen. Bei ihnen kann die
Checksumme (Signature) mit Hilfe des privaten Schlüssels erstellt und
mit Hilfe des öffentlichen Schlüssels verifiziert werden. Dadurch
entfällt das Problem der Schlüsselverteilung, allerdings wird auch hier
der Vorteil durch geringere Geschwindigkeit erkauft.

Je nach Einsatzbereich muss nun ein geeignetes Verfahren zur
Integritätssicherung und Verschlüsselung gewählt werden. Werden Daten
über ein öffentliches bzw. feindliches Netzwerk transferiert ist z. B.
der Einsatz eines Hashes problematisch. Falls ein Angreifer einen
Datensatz abfangen und modifizieren kann, kann er ebenso einen neuen
Hash berechnen und so den Integritätsschutz umgehen. Bei diesem Beispiel
wäre der Einsatz eines MACs oder von Signaturen sinnvoller.

## Zufallszahlen

Bei der korrekten Verwendung von kryptographischen Methoden ist der
Einsatz guter Zufallszahlengenerator essentiell. Dieser sollte
Zufallszahlen mit hoher Entropie generieren. Dies kann z. B. durch
Einsatz eines Hardware-Zufallsgenerators sichergestellt werden. Ist ein
solcher nicht verfügbar, muss ein kryptographisch sicherer
Pseudo-Zufallszahlengenerator (PRNG) verwendet werden. Moderne
Betriebssysteme bieten zumeist hybride Lösungen an: hierbei werden zwar
PRNGs verwendet, diese allerdings mit Entropie aus weiteren Quelle[2]
angereichert.

Die Qualität der generierten Zufallszahlen kann über deren Entropie
bestimmt werden. Hierbei wird über statistische Methoden die Qualität
der Zufälligkeit der generierten Karten ermittelt.

## Weitere Informationsquellen

Entwickler benötigen Guidance zur Selektion der jeweiligen
kryptographischen Algorithmen, hier eine kleine Auswahl öffentlich
verfügbarer Dokumente:

1.  Das amerikanische NIST gibt Empfehlungen für Cryptographical
    Standards ab, z. B. [SP-800-175B](https://csrc.nist.gov/publications/detail/sp/800-175b/final). Aufgrund der Zusammenarbeit des
    NIST mit der amerikanischen NSA bei zu vorigen Crypto-Standards
    (Vermutung der Platzierung einer Backdoor in einen
    Random-Number-Generator) wird mittlerweile gerne von den
    NIST-Empfehlungen abgesehen.

2.  Die europäische ENISA gibt regelmäßig Empfehlungen zu verwendeten
    kryptographischen Standards und Schlüssellängen ab ([Algorithms, key
    size and parameter report 2014](https://www.enisa.europa.eu/publications/algorithms-key-size-and-parameters-report-2014)). Während diese relativ gut sind,
    ist die Frequenz der Veröffentlichung für IT-Verhältnisse etwas
    behäbig (4-5 Jahre).

3.  Das deutsche Bundesamt für Sicherheit in der Informationstechnik
    (BSI) bietet häufig überarbeitete Empfehlungen zum Einsatz
    kryptographischer Methoden an ([BSI TR-02102](https://www.bsi.bund.de/DE/Publikationen/TechnischeRichtlinien/tr02102/index_htm.html)). Diese sind relativ
    aktuell und klassifizieren Algorithmen in sichere Algorithmen die
    bei aktuellen Neuentwicklungen verwendet werden sollen und in
    legacy-Algorithmen, die zwar nicht mehr bei Neuentwicklungen
    verwendet werden sollten, die aber bei bestehender Software durchaus
    weiterverwendet werden können.

4.  das BetterCrypto.org[6] bietet regelmäßig upgedatete
    Beispielskonfigurationen für geläufige Webserver. Diese sollten dazu
    dienen, dass ein Administrator diese Snippets direkt in die
    Konfiguration eines Webservers kopieren können und dadurch eine
    sichere Konfiguration erreicht wird.

## Reflektionsfragen

1.  Welche Grundideen sollten bei dem Entwurf und Einsatz
    kryptographischer Methoden angewandt werden?

2.  Wann sollte ein MAC verwendet werden? Stelle diesen einem Hash oder
    einer kryptographsichen Signatur gegenüber.

[1] Das Schlüsselpaar ist mathematisch “verwandt”.

[2] Z. B. aus CPU-Zufallszahlengeneratoren, etc.

[6] <https://www.bettercrypto.org>
