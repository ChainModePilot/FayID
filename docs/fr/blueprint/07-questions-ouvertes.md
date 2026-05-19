# Chapitre 7 : Questions ouvertes

Ce chapitre liste les problèmes ouverts que la couche de protocole actuelle laisse en suspens, à résoudre par les spécifications d'implémentation, les futures versions du protocole ou des ADRs autonomes. Chaque entrée correspond à la même entrée de la section 10 de design.md et pourra être affinée davantage lorsque `open-questions.md` sera archivé dans la prochaine Tâche 8.

> Ces éléments ne sont pas des défauts du protocole FayID ; ce sont des points d'extension intentionnellement laissés ouverts. Leur trait commun est qu'une décision prématurée à la couche de protocole contraindrait soit la liberté d'implémentation, soit nécessiterait une révision à mesure que l'écosystème évolue.

---

## 1. Algorithmes spécifiques de hachage / signature / KDF

**Problème** : la couche de protocole ne liste que des candidats (Ed25519 / HKDF-SHA256 / BIP-39) ; le choix final est laissé à la spécification d'implémentation.

**Portée** : interopérabilité entre implémentations, audit cryptographique

**Direction** : épingler les algorithmes dans la spécification d'implémentation, aux côtés du versionnage (par ex. le suffixe `v1` dans `fayid/dyn/v1`, `fayid/gmc/v1`). Les futures mises à niveau d'algorithmes peuvent coexister via de nouveaux numéros de version.

---

## 2. Longueur de la fenêtre temporelle des Dynamic Codes

**Problème** : la couche de protocole exige seulement qu'un Dynamic Code possède un `expiresAt` ; la longueur spécifique (minutes, heures) est une décision de politique d'implémentation.

**Portée** : expérience utilisateur, force de la confidentialité, fréquence de rotation

**Direction** : les implémentations l'adaptent au scénario. Les scénarios à haute sécurité utilisent une fenêtre courte (par ex. 5 minutes) ; les scénarios ordinaires peuvent l'assouplir à l'échelle horaire.

---

## 3. Seuils de limitation de débit pour les échecs de Verification Code

**Problème** : la largeur de fenêtre, le seuil du nombre d'échecs et la stratégie de back-off pour `VERIFICATION_RATE_LIMITED` sont tous des décisions d'implémentation.

**Portée** : résistance aux attaques par force brute, expérience utilisateur

**Direction** : suivre les normes de l'industrie (par ex. back-off exponentiel) ; la spécification d'implémentation définit les valeurs par défaut.

---

## 4. TTL par défaut et modèle de renouvellement pour les Authorization Grants

**Problème** : la couche de protocole exige seulement que `expiresAt` soit explicite ; les modèles concrets tels que renouvellement, expiration glissante ou TTL court avec refresh tokens sont laissés à la spécification d'implémentation.

**Portée** : expérience utilisateur, sécurité, interopérabilité avec les legacy auth sources

**Direction** : les spécifications d'implémentation peuvent introduire un type `RefreshGrant` ou appliquer des TTLs différenciés par `legacySourceKind`.

---

## 5. Sémantique de révocation des Human IDs

**Problème** : à la couche de protocole actuelle, les Human IDs **n'entrent pas** dans l'état REVOKED. Le chemin de récupération après compromission du Mnemonic (réémission de Human ID, migration de réputation, etc.) est hors du périmètre de cette spécification.

**Portée** : récupération après sinistre, continuité de la réputation

**Direction** : peut évoluer vers :

- Un mécanisme de « migration de réputation » de couche supérieure (une déclaration on-chain mappant ancien opaqueRef → nouvel opaqueRef)
- Ou un mécanisme contraint de révocation de Human ID plus succession dans le protocole v2

---

## 6. Spécification de l'espace de noms resourceRef

**Problème** : cette spécification ne suggère que la forme `<scheme>://<authority>/<path>` ; la grammaire formelle peut évoluer en spécification autonome ou s'aligner sur un standard URI existant.

**Portée** : interopérabilité entre implémentations

**Direction** : peut former une spécification distincte « FayID Resource Identifier » ou adopter directement le standard URI RFC 3986.

---

## 7. Interopérabilité entre instances du FayID System

**Problème** : lorsque plusieurs déploiements de FayID System coexistent (gérés par différents opérateurs), l'unicité globale des Human ID / iFay ID / Organization ID exige un espace de noms global ou un mécanisme de partitionnement par préfixe.

**Portée** : écosystèmes multi-opérateurs, interopérabilité inter-chaînes

**Direction** : peut devenir un sujet central de FayID v2. Les schémas candidats incluent :

- Préfixes d'espace de noms (par ex. `hid_us_xxx` vs `hid_eu_xxx`)
- Une autorité racine de nommage à la DNSSEC
- Un registre on-chain

---

## 8. Rotation de clé pour l'opaqueRef GMC

**Problème** : la rotation de `gmc_namespace_secret` brise la continuité de la réputation à long terme — le même Human ID dérive des opaqueRefs différents sous l'ancienne et la nouvelle clé.

**Portée** : continuité de la réputation, sécurité de la clé

**Direction** :

- Concevoir une fenêtre de coexistence pour les anciens et nouveaux secrets d'espace de noms
- Ou établir une équivalence entre anciennes et nouvelles références par une « déclaration de migration d'opaqueRef » on-chain
- Cet élément nécessite un ADR autonome

---

## 9. Mécanisme d'application de la propriété P9

**Problème** : la couche de protocole spécifie le comportement « les Human IDs ne quittent jamais le système et n'apparaissent jamais dans les journaux » ; les mécanismes d'application côté implémentation (étiquettes du système de types, redact à la sérialisation, scans statiques en CI) sont choisis par la spécification d'implémentation.

**Portée** : qualité d'implémentation, audit de sécurité

**Direction** :

- Au niveau du système de types : utiliser des newtypes / branded types pour distinguer les Human IDs des autres entités
- Au niveau de la sérialisation : redact par défaut, avec des bascules de liste blanche explicites
- Au niveau de la CI : scans statiques des chemins de payloads sortants et des chemins de journaux

---

## 10. Classification par niveau de confiance des Legacy Auth Sources

**Problème** : chaque échange de justificatif traditionnel doit-il donner droit de manière équivalente à un Authorization Grant ? `SMART_CONTRACT` et `PASSWORD` doivent-ils appliquer des plafonds de TTL différents ?

**Portée** : politique de sécurité, contrôle des risques

**Direction** :

- Introduire un mappage de `legacySourceKind` vers un plafond de TTL
- Appliquer des TTLs plus courts aux sources de plus faible confiance (par ex. PASSWORD simple)
- Assouplir les limites pour les sources de plus haute confiance (par ex. SMART_CONTRACT)

---

## Archivage et suivi

Les 10 éléments ci-dessus seront archivés dans `.kiro/specs/fayid-identity-system/open-questions.md` lors de la prochaine Tâche 8, chaque entrée étant étiquetée par :

- **owner** : le responsable de la question
- **scope** : les sous-systèmes affectés
- **resolution milestone** : v1 / v2 / spécification d'implémentation / ADR

> Ce chapitre sert d'index des « questions ouvertes » du blueprint et ne reformule les problèmes qu'en résumé ; pour l'archivage et le suivi détaillés, voir le futur `open-questions.md`.
