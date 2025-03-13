# XML-basierte Angriffe

Werden von einem Webserver XML-Daten entgegengenommen und serverseitig
bearbeitet, entstehen mehrere potentielle Angriffsvektoren. Zwei davon,
XML External Entities und XML-basierte DoS-Angriffe, werden in diesem
Kapitel betrachtet. Beide basieren darauf, dass XML ein komplexes
Datenformat besitzt welches durch einen ebenso komplexen Parser
serverseitig analysiert werden muss.

## XML External Entities

XML besitzt die Möglichkeit direkt innerhalb des XML-Dokuments
Typdefinitionen zu inkludieren. Diese DTD (Document Type Definition)
beginnt mit dem DOCTYPE Tag und kann auch External Entities definieren.
Diese External Entities sind Verweise auf externe Datenquellen, diese
werden durch den Parser automatisch in das XML-Dokument eingefügt.

Ein Beispiel für einen XML External Entities Angriff der auf die
Extraktion lokaler Daten zielt:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [  
  <!ELEMENT foo ANY>
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<foo>&xxe;</foo>
```

Bei diesem Beispiel wird ein neues Element (*foo*), und als möglicher
Wert für dieses Element die externe Datenquelle /etc/passwd als Referenz
*&xxe* definiert. Anschließend wird dieser Elementtyp auch sofort samt
der Referenz verwendet. Erlaubt ein Server das Parsen dieses
XML-Dokumentes würde er nun diese Datei auslesen, deren Inhalt in das
XML Dokument einfügen und ggf. das ausgefüllte Dokument an den Client
zurückgeben. Somit kann der Angreifer auf eine Datei, auf die er
eigentlich keinen Zugriff besitzen sollte mit den Rechten des
Applikationsservers zugreifen.

Ebenso kann auf eine Netzwerkadresse verwiesen werden:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "http://www.attacker.com/text.txt" >
]>
<foo>&xxe;</foo>
```

In diesem Beispiel kann der Angreifer den XML-verarbeitenden Server dazu
bringen, mittels HTTP GET auf die übergebene URL
(<http://www.attacker.com/text.txt>) zuzugreifen. Dadurch ergeben sich
mehrere Angriffsmöglichkeiten:

- Der Angreifer kann den Webserver zum “Besuch” einer Webseite
  bringen, bei diesem Besuch wird zumeist auch die öffentliche
  IP-Adresse des Webservers auf dem besuchten Webserver vermerkt. Bei
  Inhalten die z.B. gegen das Verbotsgesetz verstoßen kann dies
  negative Publicity für den Betreiber des XML-verarbeitenden
  Webservers bewirken.

- Da der Zugriff vom XML-verarbeitenden Server ausgeht, kann der
  Angreifer einen HTTP GET Request auf interne Server absetzen, die
  ansonsten durch eine initiale Firewall blockiert gewesen wären.

- Der Angreifer kann ebenso auf *localhost*, sprich dem eigenen
  Server, zugreifen. Häufig werden interne Administrationsprogramme so
  konfiguriert, dass diese nur auf Localhost lauschen (als
  Sicherheitsmassname um remote Angreifern den Zugriff zu
  unterbinden). Im Zuge eines XML External Entity basierten Angriffs
  kann ein Angreifer diesen Schutz aushebeln und direkt auf localhost
  zugreifen.

- Bei einigen Protokollen (http, smb, cifs) werden automatisch Tokens
  und Credentials vom XML-verarbeitenden Server aus zum Zielserver
  verschickt. Ein Angreifer kann dies z.B. missbrauchen um bei einem
  Windows-basierten Server via SMB NTLM-Hashes zu extrahieren und
  gegen diese offline einen Brute-Force Angriff durchzuführen.

Ein XML External Entity kann auch auf virtuelle Adressen verweisen. So
wird z.B. vom PHP XML Parser als Schema *expect* angeboten. Bei diesem
Schema wird die übergebene URL als Systemkommando ausgeführt und dessen
Ergebnis in das XML-Dokument eingefügt. Ein Angreifer kann dies
missbrauchen um Systemkommands (Command Injection) auszuführen:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
  <!DOCTYPE foo [ <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "expect://id" >
]>
<creds>
  <user>&xxe;</user>
  <pass>mypass</pass>
</creds>
```

In diesem Fall wird als Benutzername die Ausgabe des
UNIX-Systemkommandos *id* eingefügt.

## Gegenmaßnahmen

Die bevorzugte Gegenmaßnahme ist es, den verwendeten Parser so zu
konfigurieren, dass er keinen Zugriff auf XML External Entities zulässt.
Häufig wird auch die Verwendung “einfacherer” Dokumentenformate als
Gegenmaßnahme vorgeschlagen: dies ist allerdings IMHO nicht der beste
Weg, da auch die Parser einfacher Dokumentenformate (wie z.B. CSV und
JSON) ebenso Schwachstellen besitzen.

## Denial-of-Service Attacks

Ein weiteres Problem von External Entities ist es, dass hierdurch
schnell tiefe und breite Datenstrukturen aufgebaut werden können.
Versucht ein Parser nun diese Datenstruktur in-memory zu bauen, kann ein
Parser sehr schnell out-of-memory gehen und dadurch einen
Speicher-basierten DoS-Angriff durchführen.

Ein bekanntes Beispiel sind Million-Laugh Angriffe:

```xml
<!DOCTYPE root [
 <!ELEMENT root ANY>
 <!ENTITY LOL "LOL">
 <!ENTITY LOL1 "&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;&LOL;">
 <!ENTITY LOL2 "&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;&LOL1;">
 <!ENTITY LOL3 "&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;&LOL2;">
 <!ENTITY LOL4 "&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;&LOL3;">
 <!ENTITY LOL5 "&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;&LOL4;">
 <!ENTITY LOL6 "&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;&LOL5;">
 <!ENTITY LOL7 "&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;&LOL6;">
 <!ENTITY LOL8 "&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;&LOL7;">
 <!ENTITY LOL9 "&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;&LOL8;"> 
]>
<root>&LOL9;</root>
```

Bei dieser Angriffsart wird LOL9 durch 10 Elemente des Types LOL8
ersetzt. Jedes dieser 10 LOL8 Elemente wird mit mit 10 LOL7 Elementen
gebaut, etc. In Summe erzeugt dieses DTD rund drei Milliarden LOL
Elemente. Falls ein Parser versucht diese im Arbeitsspeicher zu
erstellen, wird dieser mit hoher Wahrscheinlichkeit nicht ausreichend
sein.
