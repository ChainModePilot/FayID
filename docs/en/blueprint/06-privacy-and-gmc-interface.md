# Chapter 6: Privacy and the GMC Interface

This chapter describes the two hard constraints of the FayID system: **Human IDs never leave the system and never appear in logs**, and **the GMC Interface is a strictly read-only boundary**. Together they are the security prerequisites for FayID to serve as the long-term identity layer of the Global Merit Chain.

---

## Privacy Hard Constraints

### Outbound Payload Rules

We define an "outbound payload" as any byte stream observable from outside the FayID System — including third-party resources, legacy auth sources, the Global Merit Chain, and observability backends.

| Entity | May Appear in Plaintext in an Outbound Payload? |
| --- | --- |
| **Human ID** | **Forbidden**, unless the communication explicitly proves the Human ID itself |
| **Mnemonic** | **Forbidden**, no exceptions |
| **Private Key** | **Forbidden**, no exceptions |
| iFay ID / coFay ID / Organization ID | Allowed |
| Dynamic Code | Allowed |
| Verification Code | Only paired with a coFay ID; never broadcast on its own |
| Authorization Grant | Allowed |

### Logging and Observability

```
log_allow  := { dynamicCode, ifayID, cofayID, organizationID, grantID, errorCode }
log_deny   := { humanID(plaintext), mnemonic, privateKey, verificationCode(plaintext) }
```

Loggers on the implementation side must filter via a **whitelist**: every serialization path must go through a redact step before writing to a log.

> In short: plaintext Human IDs and Mnemonics never appear in any externally observable byte stream — not in network packets, log files, or audit output.

---

## Dynamic Code Unlinkability

### Design Intent

When an outside observer obtains two Dynamic Code literals, they **cannot tell** whether the codes came from the same Human ID. This is the core privacy property that lets a Dynamic Code serve as the public proxy for a Human ID.

### Design Basis

The Dynamic Code derivation uses:

- **A fresh nonce per derivation**: making consecutive outputs statistically independent
- **An Issuer-private ikm**: preventing outside observers from reproducing the derivation function locally
- **Type prefix + time window**: keeping Dynamic Codes literally distinct from other entities without introducing additional correlatable markers

As a result, the strongest statistical inference an outside observer can make from two Dynamic Code literals is no better than random guessing.

> This is the cryptographic basis of Property P9 in the design document.

---

## Access Control on Ownership-List Queries

### Rules

- `listIFayIDsOfHuman(proofOfHuman)` **must** verify `proofOfHuman` before returning any results
- Without verification, returns `HUMAN_ID_OWNERSHIP_NOT_PROVEN`
- The Resolver **does not provide** an anonymous "look up iFay IDs by Human ID" interface

### Design Motivation

This prevents outside observers from collecting the linkage "which iFay personas does this Human ID own" through a reverse-lookup interface — equivalent to indirectly exposing the activity profile of a Human ID.

---

## GMC Interface Boundary

### Role

The GMC Interface is the **only channel** between the FayID System and the Global Merit Chain. Its design principles:

> Read-only + irreversible + never expose the root identity

### Methods Exposed to GMC

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

### Explicitly Forbidden Directions

```
// Does not exist — reverse writes are forbidden at the protocol layer
// gmcWriteHumanID(humanID)
// gmcWriteMnemonic(mnemonic)
// gmcWritePrivateKey(...)
```

> The prohibition is enforced by "the method does not exist in the IDL," rather than by runtime checks. This makes reverse writes structurally impossible at the protocol layer.

---

## opaqueRef

### Derivation

```
opaqueRef := encode(prefix="gmcref_",
  payload = HKDF(
    ikm    = gmc_namespace_secret,         // namespace key held by the FayID System
    salt   = humanID,                       // appears only inside FayID
    info   = "fayid/gmc/v1",
    length = 256 bit
  )
)
```

### Key Properties

| Property | Description |
| --- | --- |
| **Stability** | The opaqueRef derived from the same Human ID is stably equal, allowing reputation records on GMC to accumulate over the long term |
| **Collision resistance** | opaqueRefs derived from different Human IDs are almost everywhere distinct |
| **Irreversibility** | Holding only an opaqueRef does not allow recovery of the Human ID in polynomial time |
| **Unlinkability** | opaqueRefs do not participate in Dynamic Code derivation; from an outside observer's perspective the two are unlinkable |

### Significance

opaqueRef resolves a core tension:

- The Global Merit Chain needs a long-term, stable reference to a natural person in order to accumulate reputation
- A natural person's Human ID must remain private and must not appear on-chain

opaqueRef is the compromise — it is the output of a "one-way function" applied to the Human ID: stably comparable, yet irreversible.

---

## Reputation Association Modes

### iFay / coFay / Organization Reputation

- iFay IDs, coFay IDs, and Organization IDs serve as plaintext subjects of reputation records
- GMC may directly index reputation by these IDs

### Human Reputation

- Human IDs **do not appear** on-chain directly
- When a reputation record needs to be associated with a natural person, the opaqueRef of that Human ID is used as the reference
- The opaqueRef derived from the same Human ID stays stable over the long term

### Boundary Queries

When the Global Merit Chain queries the ownership of an iFay ID or coFay ID through the GMC Interface:

- Returns the OwnerKind (HUMAN / ORGANIZATION)
- In the HUMAN case, returns an opaqueRef and **not** the plaintext Human ID
- In the ORGANIZATION case, returns the plaintext Organization ID (Organizations are public by nature)

---

## Privacy Boundary Summary

| Boundary | Inside (within the FayID System) | Outside (outbound / GMC / logs) |
| --- | --- | --- |
| Plaintext Human ID | Appears only inside the Issuer / Resolver | Forbidden |
| Mnemonic | Returned to the holder once at generation time | Forbidden |
| Private Key | Exists only along the key-derivation path | Forbidden |
| Dynamic Code | Generated internally | May be made public |
| opaqueRef | Derived internally | Exposed only to GMC |

> Privacy is not "just encrypt it" — through hard constraints at the protocol layer (no write methods in the IDL, an outbound-payload whitelist, a logging whitelist), FayID structurally eliminates the possibility that a plaintext Human ID ever appears externally.
