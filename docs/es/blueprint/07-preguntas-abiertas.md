# Capítulo 7: Preguntas Abiertas

Este capítulo enumera los problemas abiertos que la capa de protocolo actual deja sin decidir, para ser resueltos por especificaciones de implementación, futuras versiones del protocolo o ADRs independientes. Cada elemento se corresponde con la misma entrada de la sección 10 de design.md y puede refinarse aún más cuando `open-questions.md` se archive en la próxima Tarea 8.

> Estos elementos no son fallos del protocolo FayID; son puntos de extensión que se han dejado abiertos intencionalmente. Su rasgo común es que decidirlos prematuramente en la capa de protocolo restringiría la libertad de implementación o requeriría revisión a medida que el ecosistema evolucione.

---

## 1. Algoritmos específicos de hash / firma / KDF

**Problema**: la capa de protocolo solo enumera candidatos (Ed25519 / HKDF-SHA256 / BIP-39); la elección final queda en manos de la especificación de implementación.

**Alcance**: interoperabilidad entre implementaciones, auditoría criptográfica

**Dirección**: fijar los algoritmos en la especificación de implementación, junto con el versionado (por ejemplo, el sufijo `v1` en `fayid/dyn/v1`, `fayid/gmc/v1`). Las futuras actualizaciones de algoritmos pueden coexistir mediante nuevos números de versión.

---

## 2. Longitud de la ventana temporal del Dynamic Code

**Problema**: la capa de protocolo solo requiere que un Dynamic Code tenga un `expiresAt`; la longitud específica (minutos, horas) es una decisión de política de implementación.

**Alcance**: experiencia de usuario, fortaleza de privacidad, frecuencia de rotación

**Dirección**: las implementaciones la ajustan al escenario. Los escenarios de alta seguridad usan una ventana corta (por ejemplo, 5 minutos); los escenarios ordinarios pueden relajarla a la escala de horas.

---

## 3. Umbrales de límite de tasa por fallo de Verification Code

**Problema**: el ancho de la ventana, el umbral de número de fallos y la estrategia de back-off para `VERIFICATION_RATE_LIMITED` son todas decisiones de implementación.

**Alcance**: resistencia a fuerza bruta, experiencia de usuario

**Dirección**: seguir las normas de la industria (por ejemplo, back-off exponencial); la especificación de implementación define los valores por defecto.

---

## 4. TTL por defecto y modelo de renovación para los Authorization Grants

**Problema**: la capa de protocolo solo requiere que `expiresAt` sea explícito; modelos concretos como renovación, expiración deslizante o TTL corto más refresh tokens quedan en manos de la especificación de implementación.

**Alcance**: experiencia de usuario, seguridad, interoperabilidad con legacy auth sources

**Dirección**: las especificaciones de implementación pueden introducir un tipo `RefreshGrant` o aplicar TTLs diferenciados por `legacySourceKind`.

---

## 5. Semántica de la revocación del Human ID

**Problema**: en la capa de protocolo actual, los Human IDs **no entran** en el estado REVOKED. La vía de recuperación tras el compromiso de un Mnemonic (re-emisión de Human ID, migración de reputación, etc.) queda fuera del alcance de esta especificación.

**Alcance**: recuperación ante desastres, continuidad de la reputación

**Dirección**: puede evolucionar hacia:

- Un mecanismo de "migración de reputación" en una capa superior (una declaración on-chain que mapee opaqueRef antiguo → opaqueRef nuevo)
- O una revocación restringida del Human ID más un mecanismo sucesor en la versión 2 del protocolo

---

## 6. Especificación del espacio de nombres de resourceRef

**Problema**: esta especificación solo sugiere la forma `<scheme>://<authority>/<path>`; la gramática formal puede evolucionar hacia una especificación independiente o alinearse con un estándar URI existente.

**Alcance**: interoperabilidad entre implementaciones

**Dirección**: puede formar una especificación separada de "FayID Resource Identifier" o adoptar directamente el estándar URI RFC 3986.

---

## 7. Interoperabilidad entre instancias del sistema FayID

**Problema**: cuando coexisten múltiples despliegues del sistema FayID (operados por diferentes operadores), la unicidad global de Human ID / iFay ID / Organization ID requiere un espacio de nombres global o un mecanismo de partición por prefijo.

**Alcance**: ecosistemas con múltiples operadores, interoperabilidad entre cadenas

**Dirección**: puede convertirse en un tema central de FayID v2. Los esquemas candidatos incluyen:

- Prefijos de espacio de nombres (por ejemplo, `hid_us_xxx` vs `hid_eu_xxx`)
- Una autoridad de nombres raíz al estilo DNSSEC
- Un registro on-chain

---

## 8. Rotación de claves para el opaqueRef de GMC

**Problema**: rotar `gmc_namespace_secret` rompe la continuidad de la reputación a largo plazo: el mismo Human ID deriva opaqueRefs distintos bajo las claves antigua y nueva.

**Alcance**: continuidad de la reputación, seguridad de las claves

**Dirección**:

- Diseñar una ventana de coexistencia para los namespace secrets antiguo y nuevo
- O establecer la equivalencia entre las referencias antiguas y nuevas mediante una "declaración de migración de opaqueRef" on-chain
- Este elemento requiere un ADR independiente

---

## 9. Mecanismo de aplicación de la propiedad P9

**Problema**: la capa de protocolo especifica el comportamiento "los Human IDs nunca salen del sistema y nunca aparecen en los logs"; los mecanismos de aplicación del lado de la implementación (etiquetas en el sistema de tipos, redact en la serialización, escaneos estáticos en CI) los elige la especificación de implementación.

**Alcance**: calidad de implementación, auditoría de seguridad

**Dirección**:

- A nivel del sistema de tipos: usar newtypes / branded types para distinguir los Human IDs de otras entidades
- A nivel de serialización: redact por defecto, con interruptores explícitos de lista blanca
- A nivel de CI: escaneos estáticos de las vías de payload saliente y de las vías de log

---

## 10. Clasificación por niveles de confianza de las Legacy Auth Sources

**Problema**: ¿debería intercambiarse cada credencial tradicional por igual por un Authorization Grant? ¿Deberían `SMART_CONTRACT` y `PASSWORD` aplicar topes de TTL distintos?

**Alcance**: política de seguridad, control de riesgo

**Dirección**:

- Introducir un mapeo de `legacySourceKind` a tope de TTL
- Aplicar TTLs más cortos a fuentes de menor confianza (por ejemplo, PASSWORD simple)
- Relajar los límites para fuentes de mayor confianza (por ejemplo, SMART_CONTRACT)

---

## Archivado y seguimiento

Los 10 elementos anteriores serán archivados en `.kiro/specs/fayid-identity-system/open-questions.md` en la próxima Tarea 8, con cada entrada etiquetada por:

- **owner**: el responsable del problema
- **scope**: los subsistemas afectados
- **resolution milestone**: v1 / v2 / especificación de implementación / ADR

> Este capítulo sirve como índice de "preguntas abiertas" del blueprint y solo replantea los problemas en forma de resumen; para el archivado y seguimiento detallado, consulta el próximo `open-questions.md`.
