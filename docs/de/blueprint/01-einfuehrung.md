# Kapitel 1: Einführung

## Was ist FayID

FayID ist die **einheitliche Identitätsinfrastruktur** des iFay-Ökosystems. Sie stellt für jeden Teilnehmer des Ökosystems – reale natürliche Personen, von ihnen erstellte digitale Personas, öffentlich auftretende geteilte Rollen sowie organisatorische Entitäten – einen konsistenten Satz von Identifikations-, Bindungs- und Authentifizierungsaustauschmechanismen bereit.

> In einem Satz: FayID verleiht jedem Teilnehmer im iFay-Ökosystem eine Identität, die verifizierbar, nachvollziehbar und datenschutzwahrend ist.

## Vier Arten von Subjekten

Das FayID-System ist um vier Arten von Kernsubjekten herum organisiert:

| Subjekt | Beschreibung |
| --- | --- |
| **Human Prototype** | Eine reale Person in der physischen Welt. Jeder Human Prototype hält eine eindeutige Human ID als Wurzelidentität, gesichert über ein Mnemonic. |
| **Digitale Persona (iFay)** | Eine KI-Persona, die eine natürliche Person innerhalb des iFay-Ökosystems erstellt. Eine Person kann mehrere iFays besitzen; jede iFay trägt ihre eigene iFay ID, doch jede iFay ist auf dieselbe Human ID zurückgebunden. |
| **Öffentliche Rolle (coFay)** | Eine öffentlich auftretende geteilte Rolle, die von einer Einzelperson oder einer Organisation erstellt und besessen werden kann. Jede coFay verfügt über ihre eigene coFay ID und einen Verification Code. |
| **Organization** | Ein Unternehmen, ein Team oder eine andere juristische Person. Eine Organization ID wird im Klartext veröffentlicht und benötigt keinen Schutz durch einen Dynamic Code. |

Die Eigentums- und Bindungsbeziehungen zwischen diesen vier Arten von Subjekten bilden das Skelett des FayID-Systems – Details siehe [Kapitel 3 Entitäten und Beziehungen](./03-entitaeten-und-beziehungen.md).

## Warum FayID benötigt wird

Im traditionellen Internet muss eine einzelne Person häufig eine große Anzahl unabhängiger Konten, Passwörter, Zertifikate und Tokens für verschiedene Systeme verwalten. Der Entwurf von FayID ist von drei Kernanforderungen getrieben:

1. **Eine Person, eine Identität, viele aggregierte Tickets**: Eine natürliche Person hält eine einzige Human ID und kann sie gegen beliebig viele traditionelle Authentifizierungstickets (Passwörter, Zertifikate, Access Tokens, Smart Contracts usw.) eintauschen, ohne sich jedes einzelne merken zu müssen.

2. **Datenschutz der Wurzelidentität**: Die Human ID ist die Wurzelidentität und darf in der öffentlichen Kommunikation niemals erscheinen. Externe Interaktionen verwenden stattdessen einen zeitlich begrenzten Dynamic Code, der nach Ablauf automatisch rotiert; zu unterschiedlichen Zeitpunkten erzeugte Dynamic Codes lassen sich nicht miteinander korrelieren.

3. **Eine Grundlage für die Global Merit Chain**: FayID ist die Identitätsschicht des langfristigen Reputationssystems des iFay-Ökosystems, der Global Merit Chain. Reputationsdaten müssen sich über die Zeit ansammeln, ohne jemals die Wurzelidentität einer natürlichen Person preiszugeben – FayID löst diese Spannung über eine unumkehrbare Referenz, den opaqueRef.

## Die langfristige Vision: Global Merit Chain

Die Global Merit Chain ist die langfristige Vision des iFay-Ökosystems: ein dezentrales Reputationsdatensystem, das über die Zeit anwächst. In diesem System gilt:

- iFay IDs, coFay IDs und Organization IDs dienen als Klartext-Subjektbezeichner für Reputationsdaten und sind On-Chain sichtbar.
- Die Reputation einer natürlichen Person wird indirekt über eine unumkehrbare Referenz zugeordnet, wodurch die Reputationskontinuität gewahrt und die Wurzelidentität geschützt bleibt.
- Die von FayID bereitgestellten Identifikations-, Eigentums- und Widerrufsprimitiven sind Voraussetzungen dafür, dass die Global Merit Chain funktionieren kann.

> FayID ist nicht die Global Merit Chain selbst; sie ist die fundamentale Identitätsschicht darunter. Ohne ein stabiles, verifizierbares, datenschutzwahrendes Identitätssystem haben Reputationsdaten keinen Anker.

## Wie dieser Blueprint zu lesen ist

Dieser Blueprint ist in einzelne Kapiteldateien gegliedert. Für eine erste Lektüre empfehlen wir folgende Reihenfolge:

1. **Glossar** ([Kapitel 2](./02-glossar.md)): einen gemeinsamen Sprachgebrauch etablieren
2. **Entitäten und Beziehungen** ([Kapitel 3](./03-entitaeten-und-beziehungen.md)): Eigentum und Bindung zwischen den vier Arten von Subjekten verstehen
3. **Anmeldedaten und Lebenszyklus** ([Kapitel 4](./04-anmeldedaten-und-lebenszyklus.md)): erfahren, wie Dynamic Codes, Verification Codes und Authorization Grants entstehen und ablaufen
4. **Authentifizierungsaustausch** ([Kapitel 5](./05-authentifizierungsaustausch.md)): sehen, wie FayID die traditionelle Authentifizierung ersetzt
5. **Datenschutz und die GMC-Schnittstelle** ([Kapitel 6](./06-datenschutz-und-gmc-schnittstelle.md)): die harten Datenschutzeinschränkungen und die On-Chain-Schnittstellengrenze verstehen
6. **Offene Fragen** ([Kapitel 7](./07-offene-fragen.md)): die offenen Probleme durchgehen, die die aktuelle Protokollschicht unentschieden lässt

Jedes Kapitel kann für sich gelesen werden, aber eine erste durchgehende Lektüre hilft, ein vollständiges mentales Modell aufzubauen.
