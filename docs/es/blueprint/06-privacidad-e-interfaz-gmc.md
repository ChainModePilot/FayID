# Capítulo 6: Privacidad e Interfaz GMC

Este capítulo describe las dos restricciones estrictas del sistema FayID: **los Human IDs nunca salen del sistema y nunca aparecen en los logs**, y **la GMC Interface es una frontera estrictamente de solo lectura**. Juntas constituyen los requisitos de seguridad para que FayID sirva como capa de identidad a largo plazo del Global Merit Chain.

---

## Restricciones estrictas de privacidad

### Reglas para payloads salientes

Definimos un "payload saliente" como cualquier flujo de bytes observable desde fuera del sistema FayID, incluyendo recursos de terceros, legacy auth sources, el Global Merit Chain y backends de observabilidad.

| Entidad | ¿Puede aparecer en texto plano en un payload saliente? |
| --- | --- |
| **Human ID** | **Prohibido**, salvo que la comunicación pruebe explícitamente el propio Human ID |
| **Mnemonic** | **Prohibido**, sin excepciones |
| **Clave privada** | **Prohibido**, sin excepciones |
| iFay ID / coFay ID / Organization ID | Permitido |
| Dynamic Code | Permitido |
| Verification Code | Solo emparejado con un coFay ID; nunca difundido por sí solo |
| Authorization Grant | Permitido |

### Logging y observabilidad

```
log_allow  := { dynamicCode, ifayID, cofayID, organizationID, grantID, errorCode }
log_deny   := { humanID(plaintext), mnemonic, privateKey, verificationCode(plaintext) }
```

Los registradores del lado de la implementación deben filtrar mediante una **lista blanca**: cada vía de serialización debe pasar por un paso de redact antes de escribir en un log.

> En resumen: los Human IDs y Mnemonics en texto plano nunca aparecen en ningún flujo de bytes observable externamente: ni en paquetes de red, ni en archivos de log, ni en salida de auditoría.

---

## No vinculabilidad de Dynamic Codes

### Intención de diseño

Cuando un observador externo obtiene dos literales de Dynamic Code, **no puede determinar** si los códigos provienen del mismo Human ID. Esta es la propiedad de privacidad central que permite que un Dynamic Code sirva como sustituto público de un Human ID.

### Base del diseño

La derivación del Dynamic Code utiliza:

- **Un nonce nuevo por derivación**: hace que las salidas consecutivas sean estadísticamente independientes
- **Un ikm privado del Issuer**: impide que observadores externos reproduzcan localmente la función de derivación
- **Prefijo de tipo + ventana temporal**: mantiene los Dynamic Codes literalmente distintos de otras entidades sin introducir marcadores correlacionables adicionales

Como resultado, la inferencia estadística más fuerte que un observador externo puede hacer a partir de dos literales de Dynamic Code no es mejor que la suposición aleatoria.

> Esta es la base criptográfica de la propiedad P9 del documento de diseño.

---

## Control de acceso en consultas de listas de titularidad

### Reglas

- `listIFayIDsOfHuman(proofOfHuman)` **debe** verificar `proofOfHuman` antes de devolver cualquier resultado
- Sin verificación, devuelve `HUMAN_ID_OWNERSHIP_NOT_PROVEN`
- El Resolver **no provee** una interfaz anónima de "buscar iFay IDs por Human ID"

### Motivación del diseño

Esto evita que observadores externos recopilen el vínculo "qué personas iFay posee este Human ID" mediante una interfaz de búsqueda inversa, lo que equivaldría a exponer indirectamente el perfil de actividad de un Human ID.

---

## Frontera de la GMC Interface

### Rol

La GMC Interface es el **único canal** entre el sistema FayID y el Global Merit Chain. Sus principios de diseño:

> Solo lectura + irreversible + nunca exponer la identidad raíz

### Métodos expuestos a GMC

```
// Solo lectura
gmcLookupOwnership(ifayIDOrCofayID)
  -> { ownerKind: "HUMAN" | "ORGANIZATION",
       ownerOpaqueRef: string }

gmcResolvePublicEntity(idString)
  -> { kind: "IFAY" | "COFAY" | "ORGANIZATION",
       revoked: bool,
       displayMetadata: opaque }
```

### Direcciones explícitamente prohibidas

```
// No existe — las escrituras inversas están prohibidas en la capa de protocolo
// gmcWriteHumanID(humanID)
// gmcWriteMnemonic(mnemonic)
// gmcWritePrivateKey(...)
```

> La prohibición se aplica mediante "el método no existe en el IDL", en lugar de mediante comprobaciones en tiempo de ejecución. Esto hace que las escrituras inversas sean estructuralmente imposibles en la capa de protocolo.

---

## opaqueRef

### Derivación

```
opaqueRef := encode(prefix="gmcref_",
  payload = HKDF(
    ikm    = gmc_namespace_secret,         // clave de espacio de nombres del sistema FayID
    salt   = humanID,                       // aparece solo dentro de FayID
    info   = "fayid/gmc/v1",
    length = 256 bit
  )
)
```

### Propiedades clave

| Propiedad | Descripción |
| --- | --- |
| **Estabilidad** | El opaqueRef derivado del mismo Human ID es establemente igual, lo que permite que los registros de reputación en GMC se acumulen a largo plazo |
| **Resistencia a colisiones** | Los opaqueRefs derivados de Human IDs distintos son casi en todas partes distintos |
| **Irreversibilidad** | Mantener únicamente un opaqueRef no permite recuperar el Human ID en tiempo polinómico |
| **No vinculabilidad** | Los opaqueRefs no participan en la derivación del Dynamic Code; desde la perspectiva de un observador externo ambos son no vinculables |

### Significado

opaqueRef resuelve una tensión central:

- El Global Merit Chain necesita una referencia estable y a largo plazo a una persona natural para acumular reputación
- El Human ID de una persona natural debe permanecer privado y no debe aparecer on-chain

opaqueRef es el compromiso: es la salida de una "función unidireccional" aplicada al Human ID: comparable de forma estable, pero irreversible.

---

## Modos de asociación de reputación

### Reputación de iFay / coFay / Organization

- Los iFay IDs, coFay IDs y Organization IDs sirven como sujetos en texto plano de los registros de reputación
- GMC puede indexar la reputación directamente por estos IDs

### Reputación humana

- Los Human IDs **no aparecen** on-chain directamente
- Cuando un registro de reputación necesita asociarse a una persona natural, se utiliza el opaqueRef de ese Human ID como referencia
- El opaqueRef derivado del mismo Human ID se mantiene estable a largo plazo

### Consultas en la frontera

Cuando el Global Merit Chain consulta la titularidad de un iFay ID o coFay ID a través de la GMC Interface:

- Devuelve el OwnerKind (HUMAN / ORGANIZATION)
- En el caso HUMAN, devuelve un opaqueRef y **no** el Human ID en texto plano
- En el caso ORGANIZATION, devuelve el Organization ID en texto plano (las organizaciones son públicas por naturaleza)

---

## Resumen de la frontera de privacidad

| Frontera | Interior (dentro del sistema FayID) | Exterior (saliente / GMC / logs) |
| --- | --- | --- |
| Human ID en texto plano | Aparece solo dentro del Issuer / Resolver | Prohibido |
| Mnemonic | Se devuelve al titular una sola vez en la generación | Prohibido |
| Clave privada | Existe solo a lo largo de la vía de derivación de claves | Prohibido |
| Dynamic Code | Generado internamente | Puede hacerse público |
| opaqueRef | Derivado internamente | Expuesto solo a GMC |

> La privacidad no consiste en "simplemente cifrarlo": mediante restricciones estrictas en la capa de protocolo (ningún método de escritura en el IDL, una lista blanca para payloads salientes, una lista blanca para logs), FayID elimina estructuralmente la posibilidad de que un Human ID en texto plano aparezca alguna vez en el exterior.
