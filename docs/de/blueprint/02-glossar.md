# Kapitel 2: Glossar

Dieses Kapitel führt die gesamte im FayID-System verwendete Terminologie zusammen. Die erste Hälfte ist aus dem [Glossar in requirements.md](../../../.kiro/specs/fayid-identity-system/requirements.md#glossary) übernommen; die zweite Hälfte ergänzt erweiterte Begriffe, die während der Entwurfsphase eingeführt wurden.

> Konvention: Wenn spätere Kapitel dieses Blueprints einen der folgenden Begriffe verwenden, sind die Definitionen in diesem Kapitel maßgeblich. Bei Mehrdeutigkeit haben die Definitionen in requirements.md endgültigen Vorrang.

---

## Kernentitäten

| Begriff | Definition |
| --- | --- |
| **FayID System** | Das in dieser Spezifikation definierte Gesamtidentitätssystem, bestehend aus den logischen Komponenten Issuer, Resolver, Auth Exchange, GMC Interface, Serializer, Parser und weiteren. |
| **Human ID** | Die Wurzelidentität einer natürlichen Person innerhalb des FayID-Systems. Aus einem Schlüsselpaar abgeleitet, mit einem Mnemonic gepaart und ausschließlich von einem einzelnen Human Prototype gehalten. |
| **Human Prototype** | Die reale natürliche Person, die eins-zu-eins mit einer Human ID gepaart ist. |
| **iFay ID** | Die Identität einer einzelnen digitalen iFay-Persona. Muss an genau eine Human ID gebunden sein, wobei eine einzelne Human ID mehrere iFay IDs binden kann. |
| **coFay ID** | Die Identität einer öffentlichen coFay-Rolle. Muss entweder von einer Human ID oder einer Organization ID besessen werden. |
| **Organization ID** | Die Identität einer Organisation innerhalb des FayID-Systems. Wird im Klartext veröffentlicht und benötigt keinen abgeleiteten Dynamic Code. |

## Anmeldedaten und abgeleitete Werte

| Begriff | Definition |
| --- | --- |
| **Mnemonic** | Die einer Human ID zugeordnete Mnemonik-Phrase; ein menschenlesbares Backup des privaten Schlüssels der Human ID. Wird dem Inhaber zum Zeitpunkt der Erzeugung nur einmal zurückgegeben und niemals im Klartext persistiert. |
| **Dynamic Code** | Eine zeitlich begrenzte Zeichenkette, die von einer Human ID abgeleitet ist und im Klartext übertragen werden darf. Wird verwendet, um auf eine Human ID zu verweisen, ohne sie offenzulegen. |
| **Verification Code** | Ein an eine coFay ID gebundener Code, der die Authentizität eines Inhabers prüft, wenn die coFay ID verwendet wird. Kann vom Eigentümer rotiert werden. |
| **Authorization Grant** | Ein zeitlich begrenztes Authentifizierungs-Anmeldedatum, das einer iFay ID oder Human ID nach Durchlaufen des Auth Exchange ausgestellt wird. Trägt einen expliziten Ablaufzeitpunkt und unterstützt aktiven Widerruf. |

## Logische Komponenten

| Begriff | Definition |
| --- | --- |
| **Issuer** | Die logische Komponente innerhalb des FayID-Systems, die für das Erzeugen, Rotieren und Widerrufen von Bezeichnern und Anmeldedaten zuständig ist. |
| **Resolver** | Die logische Komponente, die ein Klartext-Anmeldedatum (Dynamic Code, Verification Code oder ID-String) zurück zur entsprechenden Entität auflöst. |
| **Auth Exchange** | Die logische Komponente, die zwischen FayID und traditionellen Authentifizierungsverfahren (Passwort, Zertifikat, Autorisierung, Access Token, Smart Contract) austauscht. |
| **GMC Interface** | Die logische Grenzkomponente, über die das FayID System mit der Global Merit Chain interagiert. Stellt nur Read-Only-Methoden bereit; umgekehrte Schreibzugriffe von Human IDs oder Material privater Schlüssel sind verboten. |
| **Serializer** | Die Komponente, die FayID-Entitäten in übertragbare Zeichenketten kodiert. Jede Entität trägt ein erkennbares Typpräfix. |
| **Parser** | Die Komponente, die eine übertragbare Zeichenkette zurück in eine FayID-Entität dekodiert. Unterscheidet Entitätstypen anhand des Typpräfixes. |

## Externe Systeme

| Begriff | Definition |
| --- | --- |
| **Global Merit Chain** | Das externe Chain-System im iFay-Ökosystem, das Identitäts- und Reputationsdaten langfristig trägt. FayID ist seine Identitätsschicht. |
| **Legacy Auth Source** | Ein externes System, das traditionelle Authentifizierungsverfahren wie Passwort, Zertifikat, Autorisierung, Access Token oder Smart Contract bereitstellt. |
| **Target Resource** | Eine externe Ressource, die durch einen Authorization Grant geschützt ist und durch einen resourceRef identifiziert wird. |

---

## Erweiterte Begriffe (während des Entwurfs eingeführt)

Die folgenden Begriffe erscheinen zuerst im Designdokument und werden verwendet, um das Protokollverhalten präziser zu beschreiben:

| Begriff | Definition |
| --- | --- |
| **opaqueRef** | Eine stabile, aber unumkehrbare Zeichenkette, die vom GMC Interface aus einer Human ID abgeleitet wird. Wird verwendet, um die Reputation einer natürlichen Person auf der Global Merit Chain zuzuordnen, ohne die Human ID offenzulegen. |
| **resourceRef** | Eine hierarchische Zeichenkette in einem Authorization Grant, die die Zielressource eindeutig identifiziert. Empfohlene Form: `<scheme>://<authority>/<path>`. |
| **proofOfHuman / proofOfOwner** | Abstrakte Mechanismen zum Eigentumsnachweis (in Implementierungen typischerweise Signatur-Challenges). Die Protokollschicht verlangt nur, dass sie verifizierbar sind und kein Klartext-Mnemonic erfordern. |
| **OwnerKind** | Ein Enum mit den Werten `HUMAN` oder `ORGANIZATION`, das die Art des Eigentümers einer coFay ID kennzeichnet. |
| **LegacySourceKind** | Ein Enum mit den Werten `PASSWORD` / `CERTIFICATE` / `AUTHORIZATION` / `ACCESS_TOKEN` / `SMART_CONTRACT`, das die Quelle kennzeichnet, aus der ein Authorization Grant erzeugt wurde. |
| **GrantState** | Ein Enum mit den Werten `ACTIVE` / `EXPIRED` / `REVOKED`, das den aktuellen Zustand eines Authorization Grants kennzeichnet. |
| **EntityKind** | Ein Enum, das den Entitätstyp kennzeichnet, zu dem eine FayID-Zeichenkette gehört (HUMAN_ID / IFAY_ID / COFAY_ID / ORGANIZATION_ID / DYNAMIC_CODE / VERIFICATION_CODE / AUTHORIZATION_GRANT). |
| **normalize** | Eine Funktion, die eine FayID-Zeichenkette in kanonische Form überführt: Kleinbuchstaben + Typpräfix-Abgleich + Allowlist-Zeichenfilter. |
| **derive_secret** | Schlüsselmaterial, das intern vom Issuer gehalten und von einer Human ID abgeleitet wird; wird zur Erzeugung von Dynamic Codes verwendet. Wird niemals nach außen offengelegt. |
| **gmc_namespace_secret** | Ein vom FayID System gehaltener Namespace-Schlüssel, der zur Ableitung von opaqueRefs verwendet wird. Die Rotationsstrategie ist eine offene Frage. |

---

## Schnellreferenz für Typpräfixe

| Präfix | Entität | Darf öffentlich erscheinen |
| --- | --- | --- |
| `hid_` | Human ID | **Verboten** (Einschränkung der Datenschutzschicht) |
| `ifay_` | iFay ID | Erlaubt |
| `cofay_` | coFay ID | Erlaubt |
| `org_` | Organization ID | Erlaubt |
| `dyn_` | Dynamic Code | Erlaubt |
| `vrf_` | Verification Code | Nur gepaart mit einer coFay ID |
| `grt_` | Authorization Grant | Erlaubt |

> Detaillierte Zeichensätze, Längenbeschränkungen und Normalisierungsregeln finden sich im Abschnitt „Identifier Format & Encoding" in design.md.
