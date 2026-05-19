# Chapter 2: Glossary

This chapter consolidates all terminology used in the FayID system. The first half is inherited from the [requirements.md Glossary](../../../.kiro/specs/fayid-identity-system/requirements.md#glossary); the second half adds extended terms introduced during the design phase.

> Convention: when later chapters in this blueprint use any of the following terms, the definitions in this chapter are authoritative. In case of ambiguity, the definitions in requirements.md take final precedence.

---

## Core Entities

| Term | Definition |
| --- | --- |
| **FayID System** | The overall identity system defined by this specification, comprising the logical components Issuer, Resolver, Auth Exchange, GMC Interface, Serializer, Parser, and others. |
| **Human ID** | A natural person's root identity within the FayID system. Derived from a key pair, paired with a Mnemonic, and held exclusively by a single Human Prototype. |
| **Human Prototype** | The real natural person paired one-to-one with a Human ID. |
| **iFay ID** | The identity of a single iFay digital persona. Must be bound to exactly one Human ID, although a single Human ID may bind multiple iFay IDs. |
| **coFay ID** | The identity of a coFay public role. Must be owned by either a Human ID or an Organization ID. |
| **Organization ID** | An organization's identity within the FayID system. Published in plaintext and does not require a derived Dynamic Code. |

## Credentials and Derivatives

| Term | Definition |
| --- | --- |
| **Mnemonic** | The mnemonic phrase associated with a Human ID; a human-readable backup of the Human ID's private key. Returned to the holder once at generation time only and never persisted in plaintext. |
| **Dynamic Code** | A time-limited string derived from a Human ID that may be transmitted in plaintext. Used to refer to a Human ID without exposing it. |
| **Verification Code** | A code bound to a coFay ID, used to verify the authenticity of a holder when the coFay ID is in use. May be rotated by the owner. |
| **Authorization Grant** | A time-limited authentication credential issued to an iFay ID or Human ID after going through Auth Exchange. Carries an explicit expiration time and supports active revocation. |

## Logical Components

| Term | Definition |
| --- | --- |
| **Issuer** | The logical component within the FayID system responsible for generating, rotating, and revoking identifiers and credentials. |
| **Resolver** | The logical component that resolves a plaintext credential (Dynamic Code, Verification Code, or ID string) back to the corresponding entity. |
| **Auth Exchange** | The logical component that exchanges between FayID and traditional authentication methods (password, certificate, authorization, access token, smart contract). |
| **GMC Interface** | The logical boundary component through which the FayID System interacts with the Global Merit Chain. Exposes only read-only methods; reverse writes of Human IDs or private-key material are forbidden. |
| **Serializer** | The component that encodes FayID entities into transmissible strings. Each entity carries a recognizable type prefix. |
| **Parser** | The component that decodes a transmissible string back into a FayID entity. Distinguishes entity types by the type prefix. |

## External Systems

| Term | Definition |
| --- | --- |
| **Global Merit Chain** | The external chain system in the iFay ecosystem that carries identity and reputation records over the long term. FayID is its identity layer. |
| **Legacy Auth Source** | An external system that provides traditional authentication methods such as password, certificate, authorization, access token, or smart contract. |
| **Target Resource** | An external resource protected by an Authorization Grant, identified by a resourceRef. |

---

## Extended Terms (Introduced During Design)

The following terms first appear in the design document and are used to describe protocol behavior more precisely:

| Term | Definition |
| --- | --- |
| **opaqueRef** | A stable but irreversible string derived from a Human ID by the GMC Interface. Used to associate a natural person's reputation on the Global Merit Chain without exposing the Human ID. |
| **resourceRef** | A hierarchical string in an Authorization Grant that uniquely identifies the target resource. Recommended form: `<scheme>://<authority>/<path>`. |
| **proofOfHuman / proofOfOwner** | Abstract ownership-proof mechanisms (typically signature challenges in implementations). The protocol layer requires only that they be verifiable and that they not require a plaintext Mnemonic. |
| **OwnerKind** | An enum taking values `HUMAN` or `ORGANIZATION`, identifying the kind of owner of a coFay ID. |
| **LegacySourceKind** | An enum taking values `PASSWORD` / `CERTIFICATE` / `AUTHORIZATION` / `ACCESS_TOKEN` / `SMART_CONTRACT`, identifying the source from which an Authorization Grant was minted. |
| **GrantState** | An enum taking values `ACTIVE` / `EXPIRED` / `REVOKED`, identifying the current state of an Authorization Grant. |
| **EntityKind** | An enum identifying the entity type that a FayID string belongs to (HUMAN_ID / IFAY_ID / COFAY_ID / ORGANIZATION_ID / DYNAMIC_CODE / VERIFICATION_CODE / AUTHORIZATION_GRANT). |
| **normalize** | A function that converts a FayID string into canonical form: lowercase + type-prefix match + whitelist character filter. |
| **derive_secret** | Key material held internally by the Issuer and derived from a Human ID; used to generate Dynamic Codes. Never exposed externally. |
| **gmc_namespace_secret** | A namespace key held by the FayID System, used to derive opaqueRefs. The rotation strategy is an Open Question. |

---

## Type Prefix Quick Reference

| Prefix | Entity | May Appear in Public |
| --- | --- | --- |
| `hid_` | Human ID | **Forbidden** (privacy-layer constraint) |
| `ifay_` | iFay ID | Allowed |
| `cofay_` | coFay ID | Allowed |
| `org_` | Organization ID | Allowed |
| `dyn_` | Dynamic Code | Allowed |
| `vrf_` | Verification Code | Only in pair with a coFay ID |
| `grt_` | Authorization Grant | Allowed |

> See the "Identifier Format & Encoding" section in design.md for detailed character sets, length bounds, and normalization rules.
