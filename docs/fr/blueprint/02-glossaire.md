# Chapitre 2 : Glossaire

Ce chapitre regroupe l'ensemble de la terminologie utilisée dans le système FayID. La première moitié est héritée du [Glossaire de requirements.md](../../../.kiro/specs/fayid-identity-system/requirements.md#glossary) ; la seconde moitié ajoute les termes étendus introduits durant la phase de conception.

> Convention : lorsque les chapitres ultérieurs de ce blueprint utilisent l'un des termes suivants, les définitions de ce chapitre font autorité. En cas d'ambiguïté, les définitions de requirements.md prévalent en dernier ressort.

---

## Entités principales

| Terme | Définition |
| --- | --- |
| **FayID System** | Le système d'identité global défini par cette spécification, comprenant les composants logiques Issuer, Resolver, Auth Exchange, GMC Interface, Serializer, Parser, et d'autres. |
| **Human ID** | L'identité racine d'une personne physique au sein du système FayID. Dérivée d'une paire de clés, associée à un Mnemonic, et détenue exclusivement par un seul Human Prototype. |
| **Human Prototype** | La personne physique réelle associée en correspondance un-à-un avec un Human ID. |
| **iFay ID** | L'identité d'un seul persona numérique iFay. Doit être lié à exactement un Human ID, bien qu'un même Human ID puisse lier plusieurs iFay IDs. |
| **coFay ID** | L'identité d'un rôle public coFay. Doit être détenu soit par un Human ID soit par un Organization ID. |
| **Organization ID** | L'identité d'une organisation au sein du système FayID. Publiée en clair et ne nécessitant pas de Dynamic Code dérivé. |

## Justificatifs et dérivés

| Terme | Définition |
| --- | --- |
| **Mnemonic** | La phrase mnémonique associée à un Human ID ; une sauvegarde lisible par l'humain de la clé privée du Human ID. Renvoyée au détenteur une seule fois au moment de la génération et jamais persistée en clair. |
| **Dynamic Code** | Une chaîne à durée limitée dérivée d'un Human ID qui peut être transmise en clair. Utilisée pour référer à un Human ID sans l'exposer. |
| **Verification Code** | Un code lié à un coFay ID, utilisé pour vérifier l'authenticité d'un détenteur lorsque le coFay ID est en usage. Peut faire l'objet d'une rotation par le propriétaire. |
| **Authorization Grant** | Un justificatif d'authentification à durée limitée émis à un iFay ID ou Human ID après passage par l'Auth Exchange. Porte un délai d'expiration explicite et prend en charge la révocation active. |

## Composants logiques

| Terme | Définition |
| --- | --- |
| **Issuer** | Le composant logique au sein du système FayID chargé de générer, faire tourner et révoquer les identifiants et les justificatifs. |
| **Resolver** | Le composant logique qui résout un justificatif en clair (Dynamic Code, Verification Code, ou chaîne d'ID) vers l'entité correspondante. |
| **Auth Exchange** | Le composant logique qui réalise l'échange entre FayID et les méthodes d'authentification traditionnelles (mot de passe, certificat, autorisation, access token, smart contract). |
| **GMC Interface** | Le composant logique de frontière par lequel le FayID System interagit avec le Global Merit Chain. N'expose que des méthodes en lecture seule ; les écritures inverses de Human IDs ou de matériel de clé privée sont interdites. |
| **Serializer** | Le composant qui encode les entités FayID en chaînes transmissibles. Chaque entité porte un préfixe de type reconnaissable. |
| **Parser** | Le composant qui décode une chaîne transmissible en une entité FayID. Distingue les types d'entités par le préfixe de type. |

## Systèmes externes

| Terme | Définition |
| --- | --- |
| **Global Merit Chain** | Le système de chaîne externe dans l'écosystème iFay qui porte les enregistrements d'identité et de réputation sur le long terme. FayID en est la couche d'identité. |
| **Legacy Auth Source** | Un système externe qui fournit des méthodes d'authentification traditionnelles telles que mot de passe, certificat, autorisation, access token, ou smart contract. |
| **Target Resource** | Une ressource externe protégée par un Authorization Grant, identifiée par un resourceRef. |

---

## Termes étendus (introduits durant la conception)

Les termes suivants apparaissent pour la première fois dans le document de conception et sont utilisés pour décrire le comportement du protocole de manière plus précise :

| Terme | Définition |
| --- | --- |
| **opaqueRef** | Une chaîne stable mais irréversible dérivée d'un Human ID par la GMC Interface. Utilisée pour associer la réputation d'une personne physique sur le Global Merit Chain sans exposer le Human ID. |
| **resourceRef** | Une chaîne hiérarchique dans un Authorization Grant qui identifie de manière unique la ressource cible. Forme recommandée : `<scheme>://<authority>/<path>`. |
| **proofOfHuman / proofOfOwner** | Des mécanismes abstraits de preuve de propriété (typiquement des challenges de signature dans les implémentations). La couche de protocole exige seulement qu'ils soient vérifiables et qu'ils n'exigent pas de Mnemonic en clair. |
| **OwnerKind** | Un enum prenant les valeurs `HUMAN` ou `ORGANIZATION`, identifiant la catégorie de propriétaire d'un coFay ID. |
| **LegacySourceKind** | Un enum prenant les valeurs `PASSWORD` / `CERTIFICATE` / `AUTHORIZATION` / `ACCESS_TOKEN` / `SMART_CONTRACT`, identifiant la source à partir de laquelle un Authorization Grant a été émis. |
| **GrantState** | Un enum prenant les valeurs `ACTIVE` / `EXPIRED` / `REVOKED`, identifiant l'état courant d'un Authorization Grant. |
| **EntityKind** | Un enum identifiant le type d'entité auquel appartient une chaîne FayID (HUMAN_ID / IFAY_ID / COFAY_ID / ORGANIZATION_ID / DYNAMIC_CODE / VERIFICATION_CODE / AUTHORIZATION_GRANT). |
| **normalize** | Une fonction qui convertit une chaîne FayID en forme canonique : minuscules + correspondance de préfixe de type + filtrage de caractères selon une liste blanche. |
| **derive_secret** | Du matériel de clé conservé en interne par l'Issuer et dérivé d'un Human ID ; utilisé pour générer les Dynamic Codes. Jamais exposé à l'extérieur. |
| **gmc_namespace_secret** | Une clé d'espace de noms détenue par le FayID System, utilisée pour dériver les opaqueRefs. La stratégie de rotation est une question ouverte. |

---

## Référence rapide des préfixes de type

| Préfixe | Entité | Peut apparaître en public |
| --- | --- | --- |
| `hid_` | Human ID | **Interdit** (contrainte de la couche de confidentialité) |
| `ifay_` | iFay ID | Autorisé |
| `cofay_` | coFay ID | Autorisé |
| `org_` | Organization ID | Autorisé |
| `dyn_` | Dynamic Code | Autorisé |
| `vrf_` | Verification Code | Uniquement en paire avec un coFay ID |
| `grt_` | Authorization Grant | Autorisé |

> Voir la section « Identifier Format & Encoding » dans design.md pour les jeux de caractères, bornes de longueur et règles de normalisation détaillés.
