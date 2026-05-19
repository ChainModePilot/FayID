# Chapitre 1 : Introduction

## Qu'est-ce que FayID

FayID est l'**infrastructure d'identité unifiée** de l'écosystème iFay. Elle fournit un ensemble cohérent de mécanismes d'identification, de liaison et d'échange d'authentification pour chaque participant de l'écosystème — personnes physiques réelles, personas numériques qu'elles créent, rôles publics partagés et entités organisationnelles.

> En une phrase : FayID donne à chaque participant de l'écosystème iFay une identité vérifiable, traçable et préservant la confidentialité.

## Quatre catégories de sujets

Le système FayID s'organise autour de quatre catégories de sujets fondamentaux :

| Sujet | Description |
| --- | --- |
| **Human Prototype** | Une personne réelle dans le monde physique. Chaque human prototype détient un Human ID unique comme identité racine, sauvegardée par un Mnemonic. |
| **Persona numérique (iFay)** | Un persona IA qu'une personne physique crée dans l'écosystème iFay. Une personne peut posséder plusieurs iFay ; chaque iFay porte son propre iFay ID, mais chaque iFay est rattaché au même Human ID. |
| **Rôle public (coFay)** | Un rôle public partagé qui peut être créé et possédé par un individu ou une organisation. Chaque coFay possède son propre coFay ID et son Verification Code. |
| **Organisation** | Une entreprise, équipe ou autre entité juridique. Un Organization ID est publié en clair et ne nécessite pas de protection par Dynamic Code. |

Les relations de propriété et de liaison entre ces quatre catégories de sujets forment l'ossature du système FayID — voir [Chapitre 3 Entités et Relations](./03-entites-et-relations.md) pour les détails.

## Pourquoi FayID est nécessaire

Sur l'internet traditionnel, une même personne doit souvent maintenir un grand nombre de comptes, mots de passe, certificats et tokens indépendants pour différents systèmes. La conception de FayID est motivée par trois besoins fondamentaux :

1. **Une personne, une identité, plusieurs tickets agrégés** : une personne physique détient un seul Human ID et peut l'échanger contre un nombre quelconque de tickets d'authentification traditionnels (mots de passe, certificats, access tokens, smart contracts, etc.) sans avoir à se les rappeler un par un.

2. **Confidentialité de l'identité racine** : le Human ID est l'identité racine et ne doit jamais apparaître dans une communication publique. Les interactions externes utilisent à la place un Dynamic Code à durée limitée, qui se renouvelle automatiquement à expiration ; les Dynamic Codes générés à différents moments ne peuvent pas être corrélés entre eux.

3. **Une fondation pour le Global Merit Chain** : FayID est la couche d'identité du système de réputation à long terme de l'écosystème iFay, le Global Merit Chain. Les enregistrements de réputation doivent s'accumuler dans le temps sans jamais exposer l'identité racine d'une personne physique — FayID résout cette tension grâce à une référence irréversible, l'opaqueRef.

## La vision à long terme : Global Merit Chain

Le Global Merit Chain est la vision à long terme de l'écosystème iFay : un système d'enregistrements de réputation décentralisé qui s'accumule dans le temps. Dans ce système :

- Les iFay IDs, coFay IDs et Organization IDs servent d'identifiants de sujet en clair pour les enregistrements de réputation et sont visibles on-chain.
- La réputation d'une personne physique est associée indirectement par une référence irréversible, préservant la continuité de la réputation tout en protégeant l'identité racine.
- Les primitives d'identification, de propriété et de révocation fournies par FayID sont des prérequis au fonctionnement du Global Merit Chain.

> FayID n'est pas le Global Merit Chain lui-même ; c'est la couche d'identité fondamentale qui se trouve en dessous. Sans un système d'identité stable, vérifiable et préservant la confidentialité, les enregistrements de réputation n'ont aucun point d'ancrage.

## Comment lire ce blueprint

Ce blueprint est organisé en fichiers de chapitre séparés. Pour une première lecture, nous recommandons l'ordre suivant :

1. **Glossaire** ([Chapitre 2](./02-glossaire.md)) : établir un vocabulaire commun
2. **Entités et Relations** ([Chapitre 3](./03-entites-et-relations.md)) : comprendre la propriété et la liaison entre les quatre catégories de sujets
3. **Justificatifs et Cycle de Vie** ([Chapitre 4](./04-justificatifs-et-cycle-de-vie.md)) : apprendre comment les Dynamic Codes, Verification Codes et Authorization Grants sont créés et expirent
4. **Auth Exchange** ([Chapitre 5](./05-echange-d-authentification.md)) : voir comment FayID remplace l'authentification traditionnelle
5. **Confidentialité et Interface GMC** ([Chapitre 6](./06-confidentialite-et-interface-gmc.md)) : comprendre les contraintes strictes de confidentialité et la frontière de l'interface on-chain
6. **Questions ouvertes** ([Chapitre 7](./07-questions-ouvertes.md)) : passer en revue les problèmes ouverts que la couche de protocole actuelle laisse en suspens

Chaque chapitre peut être lu de façon autonome, mais une première lecture de bout en bout aide à construire un modèle mental complet.
