# Kapitel 6: Datenschutz und die GMC-Schnittstelle

Dieses Kapitel beschreibt die zwei harten Einschränkungen des FayID-Systems: **Human IDs verlassen niemals das System und erscheinen niemals in Logs** sowie **die GMC-Schnittstelle ist eine streng nur lesbare Grenze**. Zusammen bilden sie die Sicherheitsvoraussetzungen, damit FayID als langfristige Identitätsschicht der Global Merit Chain dienen kann.

---

## Harte Datenschutzeinschränkungen

### Regeln für ausgehende Payloads

Wir definieren einen „ausgehenden Payload" als jeden Bytestrom, der von außerhalb des FayID-Systems beobachtbar ist – einschließlich Drittressourcen, Legacy-Authentifizierungsquellen, der Global Merit Chain und Observability-Backends.

| Entität | Darf im Klartext in einem ausgehenden Payload erscheinen? |
| --- | --- |
| **Human ID** | **Verboten**, es sei denn, die Kommunikation weist die Human ID selbst explizit nach |
| **Mnemonic** | **Verboten**, ausnahmslos |
| **Privater Schlüssel** | **Verboten**, ausnahmslos |
| iFay ID / coFay ID / Organization ID | Erlaubt |
| Dynamic Code | Erlaubt |
| Verification Code | Nur gepaart mit einer coFay ID; niemals allein verbreitet |
| Authorization Grant | Erlaubt |

### Logging und Observability

```
log_allow  := { dynamicCode, ifayID, cofayID, organizationID, grantID, errorCode }
log_deny   := { humanID(plaintext), mnemonic, privateKey, verificationCode(plaintext) }
```

Logger auf der Implementierungsseite müssen über eine **Allowlist** filtern: Jeder Serialisierungspfad muss vor dem Schreiben in ein Log einen Redact-Schritt durchlaufen.

> Kurz gesagt: Klartext-Human-IDs und Mnemonics erscheinen niemals in einem von außen beobachtbaren Bytestrom – weder in Netzwerkpaketen, Log-Dateien noch im Audit-Output.

---

## Unverknüpfbarkeit von Dynamic Codes

### Designabsicht

Wenn ein außenstehender Beobachter zwei literale Werte von Dynamic Codes erhält, **kann er nicht erkennen**, ob die Codes von derselben Human ID stammen. Dies ist die zentrale Datenschutzeigenschaft, die es einem Dynamic Code ermöglicht, als öffentlicher Stellvertreter einer Human ID zu dienen.

### Designgrundlage

Die Ableitung des Dynamic Codes verwendet:

- **Eine frische Nonce pro Ableitung**: Sie macht aufeinanderfolgende Ausgaben statistisch unabhängig
- **Ein für den Issuer privates ikm**: Es verhindert, dass außenstehende Beobachter die Ableitungsfunktion lokal reproduzieren
- **Typpräfix + Zeitfenster**: Sie hält Dynamic Codes literal von anderen Entitäten unterscheidbar, ohne zusätzliche korrelierbare Marker einzuführen

Die stärkste statistische Schlussfolgerung, die ein außenstehender Beobachter aus zwei literalen Werten von Dynamic Codes ziehen kann, ist daher nicht besser als zufälliges Raten.

> Dies ist die kryptografische Grundlage von Property P9 im Designdokument.

---

## Zugriffskontrolle bei Eigentumslisten-Abfragen

### Regeln

- `listIFayIDsOfHuman(proofOfHuman)` **muss** `proofOfHuman` verifizieren, bevor irgendwelche Ergebnisse zurückgegeben werden
- Ohne Verifizierung wird `HUMAN_ID_OWNERSHIP_NOT_PROVEN` zurückgegeben
- Der Resolver **stellt keine** anonyme „iFay IDs nach Human ID nachschlagen"-Schnittstelle bereit

### Designmotivation

Dies verhindert, dass außenstehende Beobachter über eine Rückwärts-Lookup-Schnittstelle die Verknüpfung „welche iFay-Personas besitzt diese Human ID" sammeln – was dem indirekten Offenlegen des Aktivitätsprofils einer Human ID entspricht.

---

## Grenze der GMC-Schnittstelle

### Rolle

Die GMC-Schnittstelle ist der **einzige Kanal** zwischen dem FayID-System und der Global Merit Chain. Ihre Designprinzipien:

> Read-only + unumkehrbar + niemals die Wurzelidentität offenlegen

### Methoden, die der GMC zur Verfügung gestellt werden

```
// Read-only
gmcLookupOwnership(ifayIDOrCofayID)
  -> { ownerKind: "HUMAN" | "ORGANIZATION",
       ownerOpaqueRef: string }

gmcResolvePublicEntity(idString)
  -> { kind: "IFAY" | "COFAY" | "ORGANIZATION",
       revoked: bool,
       displayMetadata: opaque }
```

### Ausdrücklich verbotene Richtungen

```
// Existiert nicht — umgekehrte Schreibvorgänge sind auf der Protokollschicht verboten
// gmcWriteHumanID(humanID)
// gmcWriteMnemonic(mnemonic)
// gmcWritePrivateKey(...)
```

> Das Verbot wird durch „die Methode existiert nicht in der IDL" durchgesetzt, nicht durch Laufzeitprüfungen. Damit werden umgekehrte Schreibvorgänge auf der Protokollschicht strukturell unmöglich.

---

## opaqueRef

### Ableitung

```
opaqueRef := encode(prefix="gmcref_",
  payload = HKDF(
    ikm    = gmc_namespace_secret,         // vom FayID-System gehaltener Namespace-Schlüssel
    salt   = humanID,                       // erscheint nur innerhalb von FayID
    info   = "fayid/gmc/v1",
    length = 256 bit
  )
)
```

### Wesentliche Eigenschaften

| Eigenschaft | Beschreibung |
| --- | --- |
| **Stabilität** | Der aus derselben Human ID abgeleitete opaqueRef ist stabil gleich, sodass sich Reputationsdaten auf der GMC langfristig ansammeln können |
| **Kollisionsresistenz** | Aus unterschiedlichen Human IDs abgeleitete opaqueRefs sind fast überall verschieden |
| **Unumkehrbarkeit** | Mit dem alleinigen Besitz eines opaqueRef lässt sich die Human ID nicht in polynomieller Zeit rekonstruieren |
| **Unverknüpfbarkeit** | opaqueRefs nehmen nicht an der Ableitung von Dynamic Codes teil; aus Sicht eines außenstehenden Beobachters sind beide nicht miteinander verknüpfbar |

### Bedeutung

opaqueRef löst eine zentrale Spannung:

- Die Global Merit Chain benötigt eine langfristige, stabile Referenz auf eine natürliche Person, um Reputation anzusammeln
- Die Human ID einer natürlichen Person muss privat bleiben und darf nicht On-Chain erscheinen

opaqueRef ist der Kompromiss – er ist die Ausgabe einer „Einwegfunktion", die auf die Human ID angewendet wird: stabil vergleichbar und dennoch unumkehrbar.

---

## Modi der Reputationszuordnung

### iFay- / coFay- / Organization-Reputation

- iFay IDs, coFay IDs und Organization IDs dienen als Klartext-Subjekte von Reputationsdaten
- Die GMC kann Reputation direkt über diese IDs indizieren

### Reputation einer natürlichen Person

- Human IDs **erscheinen nicht** direkt On-Chain
- Wenn ein Reputationsdatensatz mit einer natürlichen Person verknüpft werden muss, wird der opaqueRef dieser Human ID als Referenz verwendet
- Der aus derselben Human ID abgeleitete opaqueRef bleibt langfristig stabil

### Grenzabfragen

Wenn die Global Merit Chain das Eigentum einer iFay ID oder coFay ID über die GMC-Schnittstelle abfragt:

- Wird der OwnerKind (HUMAN / ORGANIZATION) zurückgegeben
- Im Fall HUMAN wird ein opaqueRef zurückgegeben und **nicht** die Klartext-Human-ID
- Im Fall ORGANIZATION wird die Klartext-Organization-ID zurückgegeben (Organisationen sind ihrer Natur nach öffentlich)

---

## Zusammenfassung der Datenschutzgrenze

| Grenze | Innen (innerhalb des FayID-Systems) | Außen (ausgehend / GMC / Logs) |
| --- | --- | --- |
| Klartext-Human-ID | Erscheint nur innerhalb von Issuer / Resolver | Verboten |
| Mnemonic | Wird dem Inhaber bei der Erzeugung einmalig zurückgegeben | Verboten |
| Privater Schlüssel | Existiert nur entlang des Schlüsselableitungspfads | Verboten |
| Dynamic Code | Intern erzeugt | Darf veröffentlicht werden |
| opaqueRef | Intern abgeleitet | Wird nur gegenüber der GMC offengelegt |

> Datenschutz heißt nicht „einfach verschlüsseln" – durch harte Einschränkungen auf der Protokollschicht (keine Schreibmethoden in der IDL, eine Allowlist für ausgehende Payloads, eine Logging-Allowlist) eliminiert FayID strukturell die Möglichkeit, dass eine Klartext-Human-ID jemals nach außen erscheint.
