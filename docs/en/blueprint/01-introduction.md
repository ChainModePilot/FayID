# Chapter 1: Introduction

## What Is FayID

FayID is the **unified identity infrastructure** of the iFay ecosystem. It provides a consistent set of identification, binding, and authentication-exchange mechanisms for every participant in the ecosystem — real natural persons, digital personas they create, public-facing shared roles, and organizational entities alike.

> In one line: FayID gives every participant in the iFay ecosystem an identity that is verifiable, traceable, and privacy-preserving.

## Four Kinds of Subjects

The FayID system is organized around four kinds of core subjects:

| Subject | Description |
| --- | --- |
| **Human Prototype** | A real person in the physical world. Each human prototype holds a unique Human ID as their root identity, backed up via a Mnemonic. |
| **Digital Persona (iFay)** | An AI persona that a natural person creates within the iFay ecosystem. One person can own multiple iFays; each iFay carries its own iFay ID, but every iFay is bound back to the same Human ID. |
| **Public Role (coFay)** | A public-facing shared role that may be created and owned by an individual or an organization. Each coFay has its own coFay ID and Verification Code. |
| **Organization** | A company, team, or other legal entity. An Organization ID is published in plaintext and does not require Dynamic Code protection. |

The ownership and binding relationships among these four kinds of subjects form the skeleton of the FayID system — see [Chapter 3 Entities and Relationships](./03-entities-and-relationships.md) for details.

## Why FayID Is Needed

On the traditional internet, a single person often has to maintain a large number of independent accounts, passwords, certificates, and tokens for different systems. FayID's design is motivated by three core needs:

1. **One person, one identity, many tickets aggregated**: a natural person holds a single Human ID and can exchange it for any number of traditional authentication tickets (passwords, certificates, access tokens, smart contracts, etc.) without having to remember each one.

2. **Root identity privacy**: the Human ID is the root identity and must never appear in public communication. External interactions use a time-limited Dynamic Code instead, which rotates automatically when it expires; Dynamic Codes generated at different times cannot be correlated with each other.

3. **A foundation for the Global Merit Chain**: FayID is the identity layer of the iFay ecosystem's long-term reputation system, the Global Merit Chain. Reputation records must accumulate over time without ever exposing a natural person's root identity — FayID resolves this tension through an irreversible reference, the opaqueRef.

## The Long-Term Vision: Global Merit Chain

The Global Merit Chain is the long-term vision of the iFay ecosystem: a decentralized reputation-record system that accumulates over time. In this system:

- iFay IDs, coFay IDs, and Organization IDs serve as plaintext subject identifiers for reputation records and are visible on-chain.
- A natural person's reputation is associated indirectly through an irreversible reference, preserving reputation continuity while protecting the root identity.
- The identification, ownership, and revocation primitives provided by FayID are prerequisites for the Global Merit Chain to function.

> FayID is not the Global Merit Chain itself; it is the foundational identity layer beneath it. Without a stable, verifiable, privacy-preserving identity system, reputation records have nothing to anchor to.

## How to Read This Blueprint

This blueprint is organized into separate chapter files. For a first read, we recommend the following order:

1. **Glossary** ([Chapter 2](./02-glossary.md)): establish a shared vocabulary
2. **Entities and Relationships** ([Chapter 3](./03-entities-and-relationships.md)): understand ownership and binding among the four kinds of subjects
3. **Credentials and Lifecycle** ([Chapter 4](./04-credentials-and-lifecycle.md)): learn how Dynamic Codes, Verification Codes, and Authorization Grants are created and expire
4. **Auth Exchange** ([Chapter 5](./05-auth-exchange.md)): see how FayID replaces traditional authentication
5. **Privacy and the GMC Interface** ([Chapter 6](./06-privacy-and-gmc-interface.md)): understand the privacy hard constraints and the on-chain interface boundary
6. **Open Questions** ([Chapter 7](./07-open-questions.md)): review the open problems that the current protocol layer leaves undecided

Each chapter can be read on its own, but a first end-to-end read helps build a complete mental model.
