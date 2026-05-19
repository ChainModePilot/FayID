# Capítulo 2: Glosario

Este capítulo consolida toda la terminología utilizada en el sistema FayID. La primera mitad se hereda del [Glosario de requirements.md](../../../.kiro/specs/fayid-identity-system/requirements.md#glossary); la segunda mitad añade términos extendidos introducidos durante la fase de diseño.

> Convención: cuando los capítulos posteriores de este blueprint utilicen cualquiera de los términos siguientes, las definiciones de este capítulo son las autoritativas. En caso de ambigüedad, las definiciones de requirements.md tienen precedencia final.

---

## Entidades centrales

| Término | Definición |
| --- | --- |
| **Sistema FayID** | El sistema de identidad global definido por esta especificación, que comprende los componentes lógicos Issuer, Resolver, Auth Exchange, GMC Interface, Serializer, Parser, entre otros. |
| **Human ID** | La identidad raíz de una persona natural dentro del sistema FayID. Se deriva de un par de claves, va acompañada de un Mnemonic y la posee exclusivamente un único Human Prototype. |
| **Human Prototype** | La persona natural real emparejada uno a uno con un Human ID. |
| **iFay ID** | La identidad de una persona digital iFay individual. Debe estar vinculada a exactamente un Human ID, aunque un mismo Human ID puede vincular múltiples iFay IDs. |
| **coFay ID** | La identidad de un rol público coFay. Debe ser propiedad de un Human ID o de un Organization ID. |
| **Organization ID** | La identidad de una organización dentro del sistema FayID. Se publica en texto plano y no requiere un Dynamic Code derivado. |

## Credenciales y derivados

| Término | Definición |
| --- | --- |
| **Mnemonic** | La frase mnemónica asociada a un Human ID; un respaldo legible por humanos de la clave privada del Human ID. Se devuelve al titular una sola vez en el momento de su generación y nunca se persiste en texto plano. |
| **Dynamic Code** | Una cadena de tiempo limitado derivada de un Human ID que puede transmitirse en texto plano. Se usa para referirse a un Human ID sin exponerlo. |
| **Verification Code** | Un código vinculado a un coFay ID, usado para verificar la autenticidad de un titular cuando se utiliza el coFay ID. Puede ser rotado por el propietario. |
| **Authorization Grant** | Una credencial de autenticación de tiempo limitado emitida a un iFay ID o Human ID tras pasar por el Auth Exchange. Lleva un tiempo de expiración explícito y soporta revocación activa. |

## Componentes lógicos

| Término | Definición |
| --- | --- |
| **Issuer** | El componente lógico dentro del sistema FayID responsable de generar, rotar y revocar identificadores y credenciales. |
| **Resolver** | El componente lógico que resuelve una credencial en texto plano (Dynamic Code, Verification Code o cadena de ID) hacia la entidad correspondiente. |
| **Auth Exchange** | El componente lógico que intercambia entre FayID y los métodos de autenticación tradicionales (contraseña, certificado, autorización, token de acceso, contrato inteligente). |
| **GMC Interface** | El componente lógico de frontera a través del cual el sistema FayID interactúa con el Global Merit Chain. Expone únicamente métodos de solo lectura; las escrituras inversas de Human IDs o material de clave privada están prohibidas. |
| **Serializer** | El componente que codifica entidades FayID en cadenas transmisibles. Cada entidad lleva un prefijo de tipo reconocible. |
| **Parser** | El componente que decodifica una cadena transmisible de vuelta en una entidad FayID. Distingue los tipos de entidad por el prefijo de tipo. |

## Sistemas externos

| Término | Definición |
| --- | --- |
| **Global Merit Chain** | El sistema de cadena externo del ecosistema iFay que registra la identidad y la reputación a largo plazo. FayID es su capa de identidad. |
| **Legacy Auth Source** | Un sistema externo que provee métodos de autenticación tradicionales como contraseña, certificado, autorización, token de acceso o contrato inteligente. |
| **Target Resource** | Un recurso externo protegido por un Authorization Grant, identificado por un resourceRef. |

---

## Términos extendidos (introducidos durante el diseño)

Los siguientes términos aparecen por primera vez en el documento de diseño y se utilizan para describir el comportamiento del protocolo con mayor precisión:

| Término | Definición |
| --- | --- |
| **opaqueRef** | Una cadena estable pero irreversible derivada de un Human ID por la GMC Interface. Se usa para asociar la reputación de una persona natural en el Global Merit Chain sin exponer el Human ID. |
| **resourceRef** | Una cadena jerárquica en un Authorization Grant que identifica de forma única el recurso objetivo. Forma recomendada: `<scheme>://<authority>/<path>`. |
| **proofOfHuman / proofOfOwner** | Mecanismos abstractos de prueba de titularidad (típicamente desafíos de firma en las implementaciones). La capa de protocolo solo requiere que sean verificables y que no requieran un Mnemonic en texto plano. |
| **OwnerKind** | Un enum con valores `HUMAN` u `ORGANIZATION`, que identifica el tipo de propietario de un coFay ID. |
| **LegacySourceKind** | Un enum con valores `PASSWORD` / `CERTIFICATE` / `AUTHORIZATION` / `ACCESS_TOKEN` / `SMART_CONTRACT`, que identifica la fuente desde la que se acuñó un Authorization Grant. |
| **GrantState** | Un enum con valores `ACTIVE` / `EXPIRED` / `REVOKED`, que identifica el estado actual de un Authorization Grant. |
| **EntityKind** | Un enum que identifica el tipo de entidad al que pertenece una cadena FayID (HUMAN_ID / IFAY_ID / COFAY_ID / ORGANIZATION_ID / DYNAMIC_CODE / VERIFICATION_CODE / AUTHORIZATION_GRANT). |
| **normalize** | Una función que convierte una cadena FayID a su forma canónica: minúsculas + coincidencia de prefijo de tipo + filtro de caracteres por lista blanca. |
| **derive_secret** | Material de clave que el Issuer mantiene internamente y que se deriva de un Human ID; se usa para generar Dynamic Codes. Nunca se expone externamente. |
| **gmc_namespace_secret** | Una clave de espacio de nombres que mantiene el sistema FayID, usada para derivar opaqueRefs. La estrategia de rotación es una pregunta abierta. |

---

## Referencia rápida de prefijos de tipo

| Prefijo | Entidad | ¿Puede aparecer en público? |
| --- | --- | --- |
| `hid_` | Human ID | **Prohibido** (restricción de la capa de privacidad) |
| `ifay_` | iFay ID | Permitido |
| `cofay_` | coFay ID | Permitido |
| `org_` | Organization ID | Permitido |
| `dyn_` | Dynamic Code | Permitido |
| `vrf_` | Verification Code | Solo emparejado con un coFay ID |
| `grt_` | Authorization Grant | Permitido |

> Consulta la sección "Identifier Format & Encoding" en design.md para los conjuntos de caracteres detallados, los límites de longitud y las reglas de normalización.
