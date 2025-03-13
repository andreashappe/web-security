# LDAP-Injections

Das Lightweight Directory Access Protocol (LDAP) ist ein
standardisiertes Protokoll welches aktuell häufig für den Zugriff auf
Identifikations und Authentikationsdaten verwendet wird. Ein Angreifer
kann hier, ähnlich zu Datenbank-Injections, das Verketten von Strings
als Angriffsvektor verwenden.

LDAP verwendet Key-Value Pairs um Daten zu speichern, bzw. zu
identifizieren. Ein Beispiel:

```ldap
(cn=Andreas Happe, ou=IT Security, dc=technikum-wien, ec=at)
```

Abfragen werden mit Hilfe einiger Sonderzeichen gebildet, diese müssen
innerhalb von Datenfeldern nicht maskiert werden:

```text
* ( ) . & - _ [ ] ` ~ | @ $ % ^ ? : { } ! '
```

Abfragen werden in prefix Notation geschrieben, folgende Abfrage sucht
alle Namen, welche mit Andreas beginnen:

```ldap
(cn=Andreas*)
```

Mehrere Abfragen können mit logsischen Operatoren verknüpft werden, z.B.
sucht folgendes nach einem Namen der mit ‘Andreas’’ beginnt und mit
‘Happe’’ endet:

```ldap
(&(cn=Andreas*)(cn=*Happe))
```

Verwendet ein Entwickler eine ungesicherte String-Concatination zur
Erstellung einer Abfrage können, analog zu SQL-Injections, Fehler
geschehen.

Beispiel: wird ein Login über Benutzername und Passwort überprüft könnte
die dabei entstehende Abfrage folgend aussehen:

```ldap
(&(userID=happe)(password=trustno1))
```

Was passsiert, wenn der Angreifer *\*)(userID=\*))(\|(userID=\** als
Benutzername eingibt? Die resultierende Abfrage wäre:

```ldap
(&(userID=*)(userID=*))(|(userID=*)(password=anything))
```

Es entsteht eine Abfrage mit zwei Teilen die Und-verknüpft werden. Der
erste Part ist immer war (Tautologie). Aufgrund der Oder-Verknüpfung ist
auch der zweite Part immer war, in Summe ist der entstehende Ausdruck
immer wahr und somit kann der Login umgangen werden.

Weiter Informationen können dem Blog der Netsparker-Homepage[4]
entnommen werden.
