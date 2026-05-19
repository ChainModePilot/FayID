# Chapitre 6 : Confidentialité et Interface GMC

Ce chapitre décrit les deux contraintes strictes du système FayID : **les Human IDs ne quittent jamais le système et n'apparaissent jamais dans les journaux**, et **la GMC Interface est une frontière strictement en lecture seule**. Ensemble, elles constituent les prérequis de sécurité pour que FayID puisse servir de couche d'identité à long terme du Global Merit Chain.

---

## Contraintes strictes de confidentialité

### Règles concernant les payloads sortants

Nous définissons un « payload sortant » comme tout flux d'octets observable depuis l'extérieur du FayID System — y compris les ressources tierces, les legacy auth sources, le Global Merit Chain et les backends d'observabilité.

| Entité | Peut apparaître en clair dans un payload sortant ? |
| --- | --- |
| **Human ID** | **Interdit**, sauf si la communication prouve explicitement le Human ID lui-même |
| **Mnemonic** | **Interdit**, sans exception |
| **Clé privée** | **Interdit**, sans exception |
| iFay ID / coFay ID / Organization ID | Autorisé |
| Dynamic Code | Autorisé |
| Verification Code | Uniquement apparié avec un coFay ID ; jamais diffusé seul |
| Authorization Grant | Autorisé |

### Journalisation et observabilité

```
log_allow  := { dynamicCode, ifayID, cofayID, organizationID, grantID, errorCode }
log_deny   := { humanID(plaintext), mnemonic, privateKey, verificationCode(plaintext) }
```

Les loggers côté implémentation doivent filtrer via une **liste blanche** : chaque chemin de sérialisation doit passer par une étape de redact avant l'écriture dans un journal.

> En bref : les Human IDs et Mnemonics en clair n'apparaissent jamais dans aucun flux d'octets observable de l'extérieur — ni dans les paquets réseau, ni dans les fichiers de journaux, ni dans les sorties d'audit.

---

## Non-corrélabilité des Dynamic Codes

### Intention de conception

Lorsqu'un observateur extérieur obtient deux valeurs littérales de Dynamic Code, il **ne peut pas dire** si ces codes proviennent du même Human ID. C'est la propriété de confidentialité fondamentale qui permet à un Dynamic Code de servir de proxy public d'un Human ID.

### Bases de la conception

La dérivation du Dynamic Code utilise :

- **Un nonce frais par dérivation** : rendant les sorties consécutives statistiquement indépendantes
- **Un ikm privé à l'Issuer** : empêchant les observateurs extérieurs de reproduire la fonction de dérivation localement
- **Préfixe de type + fenêtre temporelle** : maintenant les Dynamic Codes littéralement distincts des autres entités sans introduire de marqueurs corrélables supplémentaires

En conséquence, l'inférence statistique la plus forte qu'un observateur extérieur puisse faire à partir de deux valeurs littérales de Dynamic Code n'est pas meilleure qu'une supposition aléatoire.

> C'est la base cryptographique de la propriété P9 dans le document de conception.

---

## Contrôle d'accès aux requêtes de liste de propriété

### Règles

- `listIFayIDsOfHuman(proofOfHuman)` **doit** vérifier `proofOfHuman` avant de renvoyer le moindre résultat
- Sans vérification, renvoie `HUMAN_ID_OWNERSHIP_NOT_PROVEN`
- Le Resolver **ne fournit pas** d'interface anonyme « rechercher les iFay IDs par Human ID »

### Motivation de la conception

Cela empêche les observateurs extérieurs de collecter le lien « quels personas iFay ce Human ID possède-t-il » par une interface de recherche inverse — équivalent à exposer indirectement le profil d'activité d'un Human ID.

---

## Frontière de la GMC Interface

### Rôle

La GMC Interface est l'**unique canal** entre le FayID System et le Global Merit Chain. Ses principes de conception :

> Lecture seule + irréversible + ne jamais exposer l'identité racine

### Méthodes exposées à GMC

```
// Lecture seule
gmcLookupOwnership(ifayIDOrCofayID)
  -> { ownerKind: "HUMAN" | "ORGANIZATION",
       ownerOpaqueRef: string }

gmcResolvePublicEntity(idString)
  -> { kind: "IFAY" | "COFAY" | "ORGANIZATION",
       revoked: bool,
       displayMetadata: opaque }
```

### Directions explicitement interdites

```
// N'existe pas — les écritures inverses sont interdites à la couche de protocole
// gmcWriteHumanID(humanID)
// gmcWriteMnemonic(mnemonic)
// gmcWritePrivateKey(...)
```

> L'interdiction est imposée par « la méthode n'existe pas dans l'IDL », plutôt que par des vérifications à l'exécution. Cela rend les écritures inverses structurellement impossibles à la couche de protocole.

---

## opaqueRef

### Dérivation

```
opaqueRef := encode(prefix="gmcref_",
  payload = HKDF(
    ikm    = gmc_namespace_secret,         // clé d'espace de noms détenue par le FayID System
    salt   = humanID,                       // n'apparaît qu'à l'intérieur de FayID
    info   = "fayid/gmc/v1",
    length = 256 bit
  )
)
```

### Propriétés clés

| Propriété | Description |
| --- | --- |
| **Stabilité** | L'opaqueRef dérivé du même Human ID est stablement égal, permettant aux enregistrements de réputation sur GMC de s'accumuler sur le long terme |
| **Résistance aux collisions** | Les opaqueRefs dérivés de Human IDs différents sont presque partout distincts |
| **Irréversibilité** | Détenir uniquement un opaqueRef ne permet pas de récupérer le Human ID en temps polynomial |
| **Non-corrélabilité** | Les opaqueRefs ne participent pas à la dérivation des Dynamic Codes ; du point de vue d'un observateur extérieur, les deux ne sont pas corrélables |

### Signification

opaqueRef résout une tension fondamentale :

- Le Global Merit Chain a besoin d'une référence à long terme et stable vers une personne physique afin d'accumuler de la réputation
- Le Human ID d'une personne physique doit rester privé et ne doit pas apparaître on-chain

opaqueRef est le compromis — c'est la sortie d'une « fonction à sens unique » appliquée au Human ID : stablement comparable, et pourtant irréversible.

---

## Modes d'association de réputation

### Réputation iFay / coFay / Organisation

- Les iFay IDs, coFay IDs et Organization IDs servent de sujets en clair des enregistrements de réputation
- GMC peut indexer directement la réputation par ces IDs

### Réputation des personnes physiques

- Les Human IDs **n'apparaissent pas** directement on-chain
- Lorsqu'un enregistrement de réputation doit être associé à une personne physique, l'opaqueRef de ce Human ID est utilisé comme référence
- L'opaqueRef dérivé du même Human ID reste stable sur le long terme

### Requêtes à la frontière

Lorsque le Global Merit Chain interroge la propriété d'un iFay ID ou coFay ID via la GMC Interface :

- Renvoie l'OwnerKind (HUMAN / ORGANIZATION)
- Dans le cas HUMAN, renvoie un opaqueRef et **non** le Human ID en clair
- Dans le cas ORGANIZATION, renvoie l'Organization ID en clair (les Organisations sont publiques par nature)

---

## Récapitulatif de la frontière de confidentialité

| Frontière | Intérieur (au sein du FayID System) | Extérieur (sortant / GMC / journaux) |
| --- | --- | --- |
| Human ID en clair | N'apparaît qu'à l'intérieur de l'Issuer / Resolver | Interdit |
| Mnemonic | Renvoyé au détenteur une seule fois à la génération | Interdit |
| Clé privée | N'existe que le long du chemin de dérivation de clé | Interdit |
| Dynamic Code | Généré en interne | Peut être rendu public |
| opaqueRef | Dérivé en interne | Exposé uniquement à GMC |

> La confidentialité, ce n'est pas « il suffit de chiffrer » — par des contraintes strictes à la couche de protocole (pas de méthodes d'écriture dans l'IDL, une liste blanche pour les payloads sortants, une liste blanche pour la journalisation), FayID élimine structurellement la possibilité qu'un Human ID en clair apparaisse jamais à l'extérieur.
