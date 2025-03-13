# Sicherheit als Prozess

Sicherheit kann nicht alleine stehen, man kann nicht “Sicherheit
programmieren” sondern nur “eine Applikation sicher programmieren”. Sie
ist keine One-Shot Operation die einmalig vor Projektende durchgeführt
wird, sondern muss während der gesamten Laufzeit der Softwareentwicklung
beachtet werden. Ein typisches (fiktives) Beispiel: ein Softwareprojekt
wurde als klassisches Wasserfall-Model geplant. Nach drei Jahren
Laufzeit sollte das Projekt abgeschlossen sein, ca. sechs Monate vor
Ende ist ein Penetration-Test der Software vorgesehen — die sechs Monate
sollten ausreichend sein um potentiell gefundene Schwachstellen auch zu
beheben. Es entwickelt sich leider eine typische
“Softwareprojekt”-Geschichte: die Fertigstellung verzögert sich,
schlussendlich kann der Penetration-Test erst eine Woche vor Go-Live
durchgeführt werden. Natürlich werden kritische Fehler gefunden —
aufgrund der Verzögerungen und der Mehrbelastung der Entwickler war zu
wenig Zeit für Sicherheits- bzw. Qualitätssicherheitsmaßnahmen
vorhanden. Der Launch-Zeitpunkt kann aufgrund zugekaufter Werbung nicht
mehr verschoben werden, was nun? Wie kann man diese Situation vermeiden?

Professionelle Softwareentwicklung verwendet meistens einen
(semi-)standardisierten Software Development Lifecycle (SDLC), es gibt
verschiedene Ausprägungen hier Security einzubringen. Zumeist werden in
den jeweiligen Phasen sicherheitsrelevante Inhalte hinzugefügt:

- Security Training des Personals

- Requirements and Risk Analysis

- Threat Modeling

- Secure Coding Guidelines, Secure Coding Checklists

- Security Testing Guides, Pen-Tests

- Vulnerability Management and Incident Response

Einige dieser Punkte werden in den Folgekapiteln etwas genauer
erläutert.

## Requirementsanalyse

In der *Requirementsanalyse* sollte bereits Security berücksichtigt
werden. Dies wird meistens unterlassen, da Security-Anforderungen
non-functional[1] requirements sind. Negative Auswirkungen dieses
Versäumnis sind fehlende Awareness für Security, nicht ausreichende
Ressourcen (Zeit, Personal, Budget) und schlussendlich fehlende
Sicherheit im resultierenden Softwareprodukt.

### Schützenswertes Gut

Eine zentrale Frage einer Sicherheitsdiskussion ist, was überhaupt
beschützt werden sollte. Diese Operationen oder Daten werden häufig
*Schützenswertes Gut* genannt. Beispiele für diese sind z.B. sensible
Benutzerdaten, ein essentieller Geschäftsprozess aber auch immaterielle
Werte wie die Reputation eines Unternehmens, dessen Aktienkurs oder
intellectual property.

Häufig wird die sog. CIA-Triade zur Klassifizierung verwendet. Hierbei
stehen die einzelnen Buchstaben für einen schützenswerten Bereich:
*Confidentiality*, *Integrity* und *Availability*. Diese werden in der folgenden
Tabelle genauer erläutert:

| Buchstabe | Name            | Beschreibung                                  |
|:----------|:----------------|:----------------------------------------------|
| C         | Confidentiality | no unauthorized access to data                |
| I         | Integrity       | no unauthorized or undetected[2] modification |
| A         | Availability    | Verfügbarkeit der Daten                       |

Die jeweiligen
Bereiche sind verwandt, Availability kann stark von der Integrität der
Daten abhängig sein. Beispiel: wenn eine Fahrzeitauskunft zwar als
Webservice verfügbar ist, aber den Daten nicht vertraut werden kann, ist
das Gesamtservice aus Usersicht wahrscheinlich nicht available.

Bei realen Projekten ist die Einschätzung immer vom Kunden abhängig. Ein
IT-System ist immer in die Kundenlandschaft integriert und daher können
klassische IT-Fehler unterschiedliche Auswirkungen besitzen. Z. B. wird
einem reinen Online-Shop die Availability wichtiger sein, als einem
physikalischen Shop der nebenbei einen kleinen Onlineshop betreibt;
teilweise werden Fehler durch organisatorische Maßnahme (Buchhaltung)
abgefangen, etc.

### Sicheres Design

Bei der Erstellung der Software Architektur/des Software Designs sollte
auf Sicherheit geachtet werden. Um die Ziele der CIA-Triad zu erfüllen,
empfehlt OWASP folgende Elemente bei der Analyse eines sicheren Designs
zu beachten:

- Authentication

- Authorization

- Data Confidentiality and Integrity

- Availability

- Auditing and Non-Repudiation

Die ersten vier Punkte stellen die Anforderungen aus der CIA Triade dar.

Audit Logs dienen u.a. dazu, um im Fehlerfall die Schwachstelle zu
erkennen als auch den Schadfall einzugrenzen (z.B. welche User sind in
welchem Umfang betroffen?). Unter Non-Repudiation versteht man die
Nicht-Abstreitbarkeit: falls eine Operation von einem Benutzer
durchgeführt wurde, sollte nachträglich auch verifizierbar sein, dass
diese Operation auch wirklich von dem jeweiligen Benutzer in Auftrag
gegeben wurde.

## Threat Modeling

Threat Models dienen zur systematischen Analyse von Softwareprodukten
auf Risiken, Schwachstellen und Gegenmaßnahmen. Durch die Verwendung
eines formalisierten Ablaufs wird die gleich bleibende Qualität der
Analyse gewährleistet.

Bei der Analyse sollten vier Hauptfragen gestellt und beantwortet
werden[3]:

1. What are you building?

2. What could go wrong?

3. What should you do about those things that could go wrong?

4. Did you do a decent job of analysis?

Bevor auf diese einzelnen Bereiche kurz eingegangen wird sollte noch
kurz erwähnt werden, dass Threat Models im Laufe der Zeit sehr
umfangreich und daher schwer zu verstehen werden. Im Worst-Case wird es
so “aufgebauscht” dass es nicht mehr effektiv verwendbar ist und
schlussendlich nur noch “tote” Dokumentation darstellt. Ein guter
Mittelweg zwischen Detailiertheit und Lesbarkeit ist essentiell für ein
verwendbares Threat Model.

### What are you building?

Folgende Bereiche sollten durch das Threat Model abgedeckt werden:

- Threat Actors: wer sind die potentiellen Angreifer. Dies ist wichtig
  zu wissen, da dadurch eine bessere Ressourceneinschätzung (wie viel
  Zeit bzw. finanzielle Ressourcen kann ein Angreifer aufbringen?)
  möglich ist. Ebenso wird dadurch geklärt, ob auch Insider-Angriffe
  möglich sind.

- Schützenswertes Gut: vor welchen Angriffen hat ein Unternehmen Angst
  bzw. welche Daten sind schützenswert. Die Dokumentation
  schützenswerter Güter ergibt Synergie-Effekte zu der notwendigen
  DSGVO-Dokumentation.

- Grundlegende Sicherheitsannahmen: im Laufe eines Softwareprojektes
  werden Produktentscheidungen aufgrund des aktuellen Wissensstand
  getroffen. Hier sollten diese Entscheidungen dokumentiert[4] werden.
  Beispielsweise könnte für embedded systems eine schwächere
  Verschlüsselungstechnik gewählt worden sein, da die vorhandene
  Hardware nicht potent genug für ein besseres Verfahren war. Durch
  die Dokumentation der Annahmen können diese periodische auf ihre
  Haltbarkeit hin überprüft werden. Die Dokumentation dieser Annahmen
  ist auch essentiell im Falle des Ausfalls eines Entwicklungsteams.

- Scope: welche Bereiche unterliegen der Sicherheitsobacht des
  Entwicklers? Ist die Datenbank, der Webserver, etc. Teil des
  Projekts oder werden diese von externen Personen bereitgestellt?

- Komponenten und Datenflüsse: die Applikation wird in einzelne
  Komponenten dekonstruiert. Der Datenfluss (samt Klassifizierung der
  betroffenen Daten) zwischen den Komponenten wird meistens mittels
  Datenflussdiagrammen (data flow diagrams, DFDs) dargestellt.

### What could go wrong?

Basierend auf den Datenflussdiagrammen werden potentielle Risiken und
Schwachstellen identifiziert. Häufig wird hierfür STRIDE verwendet.
Jeder Buchstabe dieser Abkürzung steht für eine Angriffsart, durch das
Analysieren jedes Elements (des Datenflussdiagrammes) sollten möglichst
viele Gefährdungen identifiziert werden. Die folgende Tabelle
listet die jeweiligen Angriffsarten auf:

| Buchstabe | Name                   |
|:----------|:-----------------------|
| S         | Spoofing               |
| T         | Tampering              |
| R         | Repudiation            |
| I         | Information Disclosure |
| D         | Denial of Service      |
| E         | Elevation of Privilege |

Im Privacy Umfeld existiert mit LINDDUN eine ähnliche Methode, die
jeweiligen Angriffe zielen hier nun nicht auf die Sicherheit, sondern
auf die Privatsphäre der Benutzer ab. Die folgende Tabelle
listet die jeweiligen Gefährdungen für die Privatsphäre auf:

| Buchstabe | Name                             |
|:----------|:---------------------------------|
| L         | Linkability                      |
| I         | Identifiability                  |
| N         | Non-Repudiation                  |
| D         | Detectability                    |
| D         | Disclosure of Information        |
| U         | Content Unawareness              |
| N         | Policy and Consent Noncompliance |

Teilweise sind diese Methoden widersprüchlich. So wird im Zuge von
STRIDE auf die Repudiation hin geachtet, also auf die
Nicht-Abstreitbarkeit der Durchführung einer Operation, während LINDDUN
dies als Non-Repudation als negativ für die Privatsphäre des Benutzers
betrachtet wird.

### What should you do about those things that could go wrong?

Die identifizierten Gefährdungen können dann mittels DREAD quantifiziert
und sortiert. Diese Reihenfolge kann bei der Behebung der
identifizierten Gefährdungen durch das Entwicklungsteam berücksichtigt
werden.

Prinzipiell gibt es mehrere Möglichkeiten mit einer Schwachstelle
umzugehen:

- Elemination: die Schwachstelle wird entfernt — dies ist effektiv nur
  durch Entfernen von Features möglich.

- Mitigation: es werden Maßnahmen implementiert die das Ausnutzen der
  Schwachstelle vermeiden bzw. erschweren sollen. Die meisten
  implementierten Sicherheitsmaßnahmen fallen in diesen Bereich.

- Transfer: durch Versicherungen und Verträge kann das Risiko an
  Andere übertragen werden.

- Accept: ein Risiko kann auch (durch die Geschäftsführung) akzeptiert
  werden. In diesem Fall ist die Dokumentation der Zuständigkeiten
  wichtig.

### Did we do a decent job of analysis?

Die Ausarbeitung eines Threat Models macht Sinn wenn das Model mit der
realen Applikation übereinstimmt und durch die sorgfältige Analyse der
Elemente des Models Verwundbarkeiten identifiziert wurden. Die
gefundenen Gefährdungen sollten in das Bug-Tracking System der Software
einfließen um ein Tracking des Fortschritts zu ermöglichen.

Wird im Zuge des Softwareprojekts automatisiert getestet wird empfohlen,
mittels Unit Tests die implementierten Mitigations zu verifizieren.
Dadurch wird der Security Test Teil der Continues-Integration Pipeline
und damit Teil der Qualitätssicherung der Software.

Zusätzlich können Penetration Tests zur Überprüfung der Sicherheit
durchgeführt werden. Penetration Tests können Sicherheitsmängel
aufdecken, sie sind allerdings nicht zur gezielten Erhöhung der
Softwarequalität dienlich, da diese vor dem Testen bereits gewährleistet
werden sollte (*You can’t test quality in*). Auch hier gibt es eine
Interaktion mit dem Threat Model: während ein Threat Model im
Gegensatz zu Penetration-Tests weniger direkte Sicherheitslücken findet,
richtet es den Fokus der Penetration-Tests auf die wichtigsten bzw.
gefährdetsten Komponenten der zu testenden Applikation.

## Secure Coding

Während der Entwicklung sollte durch die Verwendung von Secure Code
Guidelines und der Einhaltung von Best-Practises die Sicherheit der
erstellten Software gewährleistet werden. Diese Maßnahmen zielen darauf
ab, das Rad nicht neu zu erfinden. Durch Verwendung etablierter
Methodiken und Frameworks kann auf den Erfahrungsschatz dieser
zugegriffen werden und potentielle Fehler vermieden werden.

Bei der Wahl von Bibliotheken und Frameworks sollte man auf deren
Security-Historie Rücksicht nehmen. Regelmäßige Bugfix-Releases mit
dezidierten Security-Releases sind ein gutes Zeichen. Ebenso sind dies
regelmäßige Security-Audits. Falls keine Sicherheitsinformationen
verfügbar sind oder die Bibliothek/das Framework keinen langfristigen
Support gewährleistet, ist dies ein Grund ggf. dieses Framework nicht zu
verwenden.

Das Sicherheitslevel kann durch Verwendung von Security-Checklists
überprüft werden. Ein Beispiel hierfür ist der OWASP Application
Security Verfication Standard (ASVS) welcher aus einem Fragenkatalog zur
Selbstbeantwortung durch Softwareentwickler besteht.

## Secure Testing

Es sollte so früh wie möglich und regelmäßig wie möglich getestet
werden. Zumindest vor größeren Releases sollte ein Security-Check
durchgeführt werden.

Hierbei gibt es eine Interaktion mit Threat Modeling: aufgrund des
Threatmodels können besonders gefährdete Bereiche identifiziert, und
diese Bereiche gezielt getestet werden. Dadurch werden die Kosten des
Testens reduziert.

## Maintenance

Auch nach dem Abschluss der Entwicklungsphase eines Projektes gibt es
Security-Anforderungen. Es sollte dokumentiert werden, wie im Falle
eines Security-Vorfalls (Security Incident) vorgegangen wird. Dieser
Prozess kann u.a. die Notifizierung von Kunden, das Deaktivieren von
Servern, Bereitstellung eines Patch-Plans, etc. beinhalten.

Diese Vorkehrungen müssen nicht nur den eigenen Code, sondern auch
Schwachstellen in verwendeten Fremdbibliotheken und Frameworks
beinhalten. Angriffe gegen verwendete Bibliotheken/Frameworks (eine Form
der supply-chain attacks) nahmen in letzter Zeit zu.

## Reflektionsfragen

1. Was versteht man unter einem Threat Model, welche Elemente sollten
   vorhanden sein (1-2 Sätz.B.schreibung pro Element)

2. Welche Maßnahmen sollten im Zuge des Secure Development Lifecycles
   betrachtet werden? Erläutere einige der Maßnahmen.

[1] Functional Requirements beschreiben die Funktionsweise einer
Applikation und sind z.B. mittels use-cases abgebildet. Non-Functional
Requirements beschreiben eher die Qualität der erstellen Applikation wie
Sicherheit und Performance.

[2] Mittels Hashes, MACs, Verschlüsselung, etc. können prinzipiell
Veränderungen nicht verhindert werden, allerdings können Veränderungen
im Nachhinein detektiert werden.

[3] Quelle: Adam Shostack — Threat Modeling

[4] Bonuspunkte wenn nicht nur die Annahme, sondern zusätzlich auch die
Auswirkungen im Falle einer gebrochenen Annahme, wer für die Überprüfung
der Annahme zuständig ist, und wer fachlich die Annahme überprüfen kann,
dokumentiert ist.
