# Kapitel 7: Offene Fragen

Dieses Kapitel listet die offenen Probleme auf, die die aktuelle Protokollschicht unentschieden lässt und die durch Implementierungsspezifikationen, zukünftige Protokollversionen oder eigenständige ADRs gelöst werden sollen. Jeder Punkt entspricht demselben Eintrag in Abschnitt 10 von design.md und kann weiter verfeinert werden, wenn `open-questions.md` in der bevorstehenden Aufgabe 8 archiviert wird.

> Diese Punkte sind keine Schwachstellen des FayID-Protokolls; es sind absichtlich offen gelassene Erweiterungspunkte. Ihre gemeinsame Eigenschaft ist, dass eine vorzeitige Festlegung auf der Protokollschicht entweder die Implementierungsfreiheit einschränken oder im Zuge der Weiterentwicklung des Ökosystems eine Überarbeitung erfordern würde.

---

## 1. Konkrete Hash- / Signatur- / KDF-Algorithmen

**Problem**: Die Protokollschicht listet nur Kandidaten auf (Ed25519 / HKDF-SHA256 / BIP-39); die endgültige Wahl bleibt der Implementierungsspezifikation überlassen.

**Geltungsbereich**: implementierungsübergreifende Interoperabilität, kryptografisches Audit

**Richtung**: Algorithmen in der Implementierungsspezifikation festlegen, zusammen mit einer Versionierung (z. B. das Suffix `v1` in `fayid/dyn/v1`, `fayid/gmc/v1`). Künftige Algorithmus-Upgrades können über neue Versionsnummern koexistieren.

---

## 2. Länge des Zeitfensters für Dynamic Codes

**Problem**: Die Protokollschicht verlangt nur, dass ein Dynamic Code ein `expiresAt` besitzt; die konkrete Länge (Minuten, Stunden) ist eine Entscheidung der Implementierungsrichtlinie.

**Geltungsbereich**: Benutzererfahrung, Datenschutzstärke, Rotationshäufigkeit

**Richtung**: Implementierungen passen sie an das jeweilige Szenario an. Hochsicherheitsszenarien verwenden ein kurzes Fenster (z. B. 5 Minuten); gewöhnliche Szenarien können auf den Stundenbereich gelockert werden.

---

## 3. Schwellenwerte für die Ratenbegrenzung bei Verification-Code-Fehlern

**Problem**: Fensterbreite, Schwellenwert für die Anzahl der Fehlversuche und Back-off-Strategie für `VERIFICATION_RATE_LIMITED` sind allesamt Implementierungsentscheidungen.

**Geltungsbereich**: Brute-Force-Resistenz, Benutzererfahrung

**Richtung**: Branchenüblichen Normen folgen (z. B. exponentielles Back-off); die Implementierungsspezifikation definiert die Standardwerte.

---

## 4. Standard-TTL und Erneuerungsmodell für Authorization Grants

**Problem**: Die Protokollschicht verlangt nur, dass `expiresAt` explizit ist; konkrete Modelle wie Erneuerung, gleitender Ablauf oder kurze TTL plus Refresh-Token sind der Implementierungsspezifikation überlassen.

**Geltungsbereich**: Benutzererfahrung, Sicherheit, Interoperabilität mit Legacy-Authentifizierungsquellen

**Richtung**: Implementierungsspezifikationen können einen `RefreshGrant`-Typ einführen oder differenzierte TTLs pro `legacySourceKind` anwenden.

---

## 5. Widerrufssemantik für Human IDs

**Problem**: In der aktuellen Protokollschicht **gehen** Human IDs **nicht** in den REVOKED-Zustand über. Der Wiederherstellungspfad nach einer Mnemonic-Kompromittierung (Neuausstellung der Human ID, Reputationsmigration usw.) liegt außerhalb des Geltungsbereichs dieser Spezifikation.

**Geltungsbereich**: Disaster Recovery, Reputationskontinuität

**Richtung**: Mögliche Entwicklungen:

- Ein höherschichtiger Mechanismus zur „Reputationsmigration" (eine On-Chain-Erklärung, die alten opaqueRef → neuen opaqueRef abbildet)
- Oder ein eingeschränkter Widerruf der Human ID plus Nachfolgemechanismus in Protokoll v2

---

## 6. Spezifikation des resourceRef-Namespaces

**Problem**: Diese Spezifikation schlägt nur die Form `<scheme>://<authority>/<path>` vor; die formale Grammatik kann sich zu einer eigenständigen Spezifikation entwickeln oder sich an einem bestehenden URI-Standard orientieren.

**Geltungsbereich**: implementierungsübergreifende Interoperabilität

**Richtung**: Kann eine eigenständige Spezifikation „FayID Resource Identifier" bilden oder direkt den URI-Standard RFC 3986 übernehmen.

---

## 7. Interoperabilität zwischen Instanzen des FayID-Systems

**Problem**: Wenn mehrere FayID-System-Deployments koexistieren (von unterschiedlichen Betreibern betrieben), erfordert die globale Eindeutigkeit von Human ID / iFay ID / Organization ID einen globalen Namespace oder einen Mechanismus zur Präfix-Partitionierung.

**Geltungsbereich**: Mehrbetreiber-Ökosysteme, Cross-Chain-Interoperabilität

**Richtung**: Kann ein zentrales Thema von FayID v2 werden. Mögliche Schemata:

- Namespace-Präfixe (z. B. `hid_us_xxx` vs. `hid_eu_xxx`)
- Eine Root-Naming-Authority im Stil von DNSSEC
- Eine On-Chain-Registry

---

## 8. Schlüsselrotation für den GMC-opaqueRef

**Problem**: Die Rotation von `gmc_namespace_secret` durchbricht die langfristige Reputationskontinuität – dieselbe Human ID leitet unter dem alten und dem neuen Schlüssel unterschiedliche opaqueRefs ab.

**Geltungsbereich**: Reputationskontinuität, Schlüsselsicherheit

**Richtung**:

- Ein Koexistenzfenster für alte und neue Namespace-Schlüssel entwerfen
- Oder Äquivalenz zwischen alten und neuen Referenzen über eine On-Chain-„opaqueRef-Migrationserklärung" herstellen
- Dieser Punkt erfordert ein eigenständiges ADR

---

## 9. Durchsetzungsmechanismus für Property P9

**Problem**: Die Protokollschicht spezifiziert das Verhalten „Human IDs verlassen niemals das System und erscheinen niemals in Logs"; die Durchsetzungsmechanismen auf der Implementierungsseite (Tags des Typsystems, Serialisierungs-Redact, statische CI-Scans) werden von der Implementierungsspezifikation gewählt.

**Geltungsbereich**: Implementierungsqualität, Sicherheitsaudit

**Richtung**:

- Auf Ebene des Typsystems: Newtypes / Branded Types verwenden, um Human IDs von anderen Entitäten zu unterscheiden
- Auf Serialisierungsebene: standardmäßig redacten, mit expliziten Allowlist-Schaltern
- Auf CI-Ebene: statische Scans der Pfade ausgehender Payloads und Logpfade

---

## 10. Klassifizierung der Vertrauensstufen von Legacy-Authentifizierungsquellen

**Problem**: Sollte jedes traditionelle Anmeldedatum gleichberechtigt gegen einen Authorization Grant getauscht werden? Sollten `SMART_CONTRACT` und `PASSWORD` unterschiedliche TTL-Obergrenzen anwenden?

**Geltungsbereich**: Sicherheitsrichtlinie, Risikokontrolle

**Richtung**:

- Eine Zuordnung von `legacySourceKind` zu TTL-Obergrenzen einführen
- Auf Quellen mit geringerem Vertrauen kürzere TTLs anwenden (z. B. einfaches PASSWORD)
- Bei Quellen mit höherem Vertrauen die Beschränkungen lockern (z. B. SMART_CONTRACT)

---

## Archivierung und Nachverfolgung

Die obigen 10 Punkte werden in der bevorstehenden Aufgabe 8 nach `.kiro/specs/fayid-identity-system/open-questions.md` archiviert, wobei jeder Eintrag folgendermaßen markiert wird:

- **owner**: der Verantwortliche für den Punkt
- **scope**: die betroffenen Subsysteme
- **resolution milestone**: v1 / v2 / Implementierungsspezifikation / ADR

> Dieses Kapitel dient als Index der „offenen Fragen" des Blueprints und führt die Punkte nur in Kurzform auf; für die ausführliche Archivierung und Nachverfolgung siehe die kommende `open-questions.md`.
