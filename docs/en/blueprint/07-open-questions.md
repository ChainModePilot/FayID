# Chapter 7: Open Questions

This chapter lists the open problems that the current protocol layer leaves undecided, to be resolved by implementation specs, future protocol versions, or standalone ADRs. Each item maps to the same entry in Section 10 of design.md and may be refined further when `open-questions.md` is archived in the upcoming Task 8.

> These items are not flaws in the FayID protocol; they are extension points that have been intentionally left open. Their common trait is that deciding them prematurely at the protocol layer would either constrain implementation freedom or require revision as the ecosystem evolves.

---

## 1. Specific Hash / Signature / KDF Algorithms

**Issue**: the protocol layer only lists candidates (Ed25519 / HKDF-SHA256 / BIP-39); the final choice is left to the implementation spec.

**Scope**: cross-implementation interoperability, cryptographic audit

**Direction**: pin the algorithms in the implementation spec, alongside versioning (e.g., the `v1` suffix in `fayid/dyn/v1`, `fayid/gmc/v1`). Future algorithm upgrades can coexist via new version numbers.

---

## 2. Dynamic Code Time Window Length

**Issue**: the protocol layer requires only that a Dynamic Code have an `expiresAt`; the specific length (minutes, hours) is an implementation policy decision.

**Scope**: user experience, privacy strength, rotation frequency

**Direction**: implementations tailor it to the scenario. High-security scenarios use a short window (e.g., 5 minutes); ordinary scenarios may relax to hour-scale.

---

## 3. Verification Code Failure Rate-Limit Thresholds

**Issue**: the window width, failure-count threshold, and back-off strategy for `VERIFICATION_RATE_LIMITED` are all implementation decisions.

**Scope**: brute-force resistance, user experience

**Direction**: follow industry norms (e.g., exponential back-off); the implementation spec defines defaults.

---

## 4. Default TTL and Renewal Model for Authorization Grants

**Issue**: the protocol layer requires only that `expiresAt` be explicit; concrete models such as renewal, sliding expiration, or short TTL plus refresh tokens are left to the implementation spec.

**Scope**: user experience, security, interoperability with legacy auth sources

**Direction**: implementation specs may introduce a `RefreshGrant` type or apply differentiated TTLs per `legacySourceKind`.

---

## 5. Human ID Revocation Semantics

**Issue**: at the current protocol layer, Human IDs **do not enter** the REVOKED state. The recovery path after Mnemonic compromise (Human ID re-issue, reputation migration, etc.) is out of scope for this specification.

**Scope**: disaster recovery, reputation continuity

**Direction**: may evolve toward:

- A higher-layer "reputation migration" mechanism (an on-chain declaration mapping old opaqueRef → new opaqueRef)
- Or a constrained Human ID revocation plus successor mechanism in protocol v2

---

## 6. resourceRef Namespace Specification

**Issue**: this specification only suggests the `<scheme>://<authority>/<path>` form; the formal grammar may evolve into a standalone spec or align with an existing URI standard.

**Scope**: cross-implementation interoperability

**Direction**: may form a separate "FayID Resource Identifier" specification or directly adopt the RFC 3986 URI standard.

---

## 7. Interoperability Across FayID System Instances

**Issue**: when multiple FayID System deployments coexist (run by different operators), the global uniqueness of Human ID / iFay ID / Organization ID requires a global namespace or prefix-partitioning mechanism.

**Scope**: multi-operator ecosystems, cross-chain interoperability

**Direction**: may become a core topic of FayID v2. Candidate schemes include:

- Namespace prefixes (e.g., `hid_us_xxx` vs `hid_eu_xxx`)
- A DNSSEC-style root naming authority
- An on-chain registry

---

## 8. Key Rotation for the GMC opaqueRef

**Issue**: rotating `gmc_namespace_secret` breaks long-term reputation continuity — the same Human ID derives different opaqueRefs under the old and new keys.

**Scope**: reputation continuity, key security

**Direction**:

- Design a coexistence window for old and new namespace secrets
- Or establish equivalence between old and new references through an on-chain "opaqueRef migration declaration"
- This item requires a standalone ADR

---

## 9. Enforcement Mechanism for Property P9

**Issue**: the protocol layer specifies the behavior "Human IDs never leave the system and never appear in logs"; the enforcement mechanisms on the implementation side (type-system tags, serialization redact, CI static scans) are chosen by the implementation spec.

**Scope**: implementation quality, security audit

**Direction**:

- At the type-system level: use newtypes / branded types to distinguish Human IDs from other entities
- At the serialization level: redact by default, with explicit whitelist toggles
- At the CI level: static scans of outbound payload paths and log paths

---

## 10. Trust-Tier Classification of Legacy Auth Sources

**Issue**: should every traditional credential exchange equally for an Authorization Grant? Should `SMART_CONTRACT` and `PASSWORD` apply different TTL caps?

**Scope**: security policy, risk control

**Direction**:

- Introduce a `legacySourceKind`-to-TTL-cap mapping
- Apply shorter TTLs to lower-trust sources (e.g., simple PASSWORD)
- Relax limits for higher-trust sources (e.g., SMART_CONTRACT)

---

## Archival and Tracking

The 10 items above will be archived to `.kiro/specs/fayid-identity-system/open-questions.md` in the upcoming Task 8, with each entry tagged by:

- **owner**: the steward of the issue
- **scope**: the affected subsystems
- **resolution milestone**: v1 / v2 / implementation spec / ADR

> This chapter serves as the blueprint's "open questions" index and only restates the issues in summary; for detailed archival and tracking, see the forthcoming `open-questions.md`.
