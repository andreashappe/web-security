# Type-Juggling Angriffe

Type Juggling Angriffe können in mehreren Programmiersprachen auftreten,
besonders “bekannt” ist dieser Angriffsvektor in PHP. Diese Angriffe
sind möglich, wenn durch eine automatische, implizite Typkonvertierung
das erwartete Resultat einer Operation verfälscht wird. In PHP liegt das
Grundproblem in den beiden Vergleichsoperatoren *==* (loose) und *===*
(strict), ersterer führt automatisch Typkonvertierungen durch und wird
leider häufig anstatt des sicheren zweiten Operators verwendet.

Wird z.B. server-seitig in PHP ein String mit einer Zahl verglichen,
wird der String automatisch in eine Zahl konvertiert, dies inkludiert
Hex- und Octal-Darstellungen von Zahlen:

| Operant A | Operant B | Ergebnis |
|:----------|:----------|:---------|
| "0000"    | int(0)    | true     |
| "0e42"    | int(0)    | true     |
| "1abc"    | int(1)    | true     |
| "abc"     | int(0)    | true     |
| "0xF"     | "15"      | true     |
| "0e1234"  | "0e5678"  | true     |

Dies kann verwendet werden, um Vergleiche “kurzzuschließen”, wie
folgendes Beispiel (aus einer älteren WordPress-Version) zeigen soll.
Hier wird die Benutzerauthorisierung über einen berechneten MAC
durchgeführt. Der Anwender setzt mehrere Werte über Cookies, die
Integrität dieser Werte wird durch einen berechneten MAC verifiziert.
Zur Berechnung des MACs wird ein geheimer Schlüssel (*key* in dem
Beispiel) verwendet, der nie den Server verlässt. Dies wird vereinfacht
durch folgenden server-seitigen Code umgesetzt:

```php
$hash = hash_mac('md5', $username . '|' . $expiration, $key);
if ($hmac != $hash) {
    // bad cookie, give error
} else {
    // accept operation
}
```

*username*, *expiration* und *hmac* werden aus dem Cookie gelesen und
können dadurch durch den Angreifer bestimmt werden. Ein Angreifer kann
nun *username* auf *Administrator*, und *hmac* auf den Wert *0* setzen.
Nun kann er einen Brute-Force Angriff ausführen, bei dem das Ablaufdatem
(*expiration*) au feinen zufälligen Wert gesetzt wird. Der Angreifer
hofft, dass bei einem der Zugriffe zufällig als Hash ein Wert generiert
wird, der mit "0e…" beginnt, da hier automatisch eine Konvertierung des
Wertes auf *0* passieren würde. Dies entspricht dem übergebenen *hmac*
(der ebenso *0*) ist und eine server-seitige Administratoren-Identität
ist übernommen.
