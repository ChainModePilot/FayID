# Capítulo 1: Introducción

## Qué es FayID

FayID es la **infraestructura de identidad unificada** del ecosistema iFay. Proporciona un conjunto coherente de mecanismos de identificación, vinculación e intercambio de autenticación para cada participante del ecosistema: personas naturales reales, las personas digitales que estas crean, los roles públicos compartidos y las entidades organizativas.

> En una línea: FayID otorga a cada participante del ecosistema iFay una identidad verificable, trazable y respetuosa con la privacidad.

## Cuatro tipos de sujetos

El sistema FayID se organiza en torno a cuatro tipos de sujetos centrales:

| Sujeto | Descripción |
| --- | --- |
| **Human Prototype** | Una persona real del mundo físico. Cada Human Prototype posee un Human ID único como identidad raíz, respaldado por un Mnemonic. |
| **Persona digital (iFay)** | Una persona basada en IA que una persona natural crea dentro del ecosistema iFay. Una persona puede poseer múltiples iFays; cada iFay tiene su propio iFay ID, pero todo iFay queda vinculado al mismo Human ID. |
| **Rol público (coFay)** | Un rol compartido y público que puede ser creado y poseído por una persona individual o una organización. Cada coFay tiene su propio coFay ID y Verification Code. |
| **Organización** | Una empresa, equipo u otra entidad jurídica. Un Organization ID se publica en texto plano y no requiere protección mediante Dynamic Code. |

Las relaciones de titularidad y vinculación entre estos cuatro tipos de sujetos forman el esqueleto del sistema FayID; consulta el [Capítulo 3 Entidades y Relaciones](./03-entidades-y-relaciones.md) para más detalles.

## Por qué FayID es necesario

En la internet tradicional, una sola persona suele tener que mantener un gran número de cuentas, contraseñas, certificados y tokens independientes para diferentes sistemas. El diseño de FayID está motivado por tres necesidades centrales:

1. **Una persona, una identidad, muchos tickets agregados**: una persona natural posee un único Human ID y puede intercambiarlo por cualquier número de tickets de autenticación tradicionales (contraseñas, certificados, tokens de acceso, contratos inteligentes, etc.) sin tener que recordar cada uno de ellos.

2. **Privacidad de la identidad raíz**: el Human ID es la identidad raíz y nunca debe aparecer en la comunicación pública. Las interacciones externas usan en su lugar un Dynamic Code de tiempo limitado, que rota automáticamente al expirar; los Dynamic Codes generados en momentos distintos no pueden correlacionarse entre sí.

3. **Una base para el Global Merit Chain**: FayID es la capa de identidad del sistema de reputación a largo plazo del ecosistema iFay, el Global Merit Chain. Los registros de reputación deben acumularse a lo largo del tiempo sin exponer jamás la identidad raíz de una persona natural; FayID resuelve esta tensión mediante una referencia irreversible, el opaqueRef.

## La visión a largo plazo: Global Merit Chain

El Global Merit Chain es la visión a largo plazo del ecosistema iFay: un sistema descentralizado de registro de reputación que se acumula con el tiempo. En este sistema:

- Los iFay IDs, coFay IDs y Organization IDs sirven como identificadores públicos de los sujetos de los registros de reputación y son visibles on-chain.
- La reputación de una persona natural se asocia indirectamente mediante una referencia irreversible, preservando la continuidad de la reputación a la vez que se protege la identidad raíz.
- Las primitivas de identificación, titularidad y revocación que proporciona FayID son requisitos previos para que el Global Merit Chain funcione.

> FayID no es el Global Merit Chain en sí mismo; es la capa de identidad fundamental que lo sustenta. Sin un sistema de identidad estable, verificable y respetuoso con la privacidad, los registros de reputación no tendrían a qué anclarse.

## Cómo leer este blueprint

Este blueprint está organizado en archivos de capítulos separados. Para una primera lectura, recomendamos el siguiente orden:

1. **Glosario** ([Capítulo 2](./02-glosario.md)): establecer un vocabulario común
2. **Entidades y Relaciones** ([Capítulo 3](./03-entidades-y-relaciones.md)): comprender la titularidad y la vinculación entre los cuatro tipos de sujetos
3. **Credenciales y Ciclo de Vida** ([Capítulo 4](./04-credenciales-y-ciclo-de-vida.md)): aprender cómo se crean y expiran los Dynamic Codes, Verification Codes y Authorization Grants
4. **Auth Exchange** ([Capítulo 5](./05-intercambio-de-autenticacion.md)): ver cómo FayID reemplaza la autenticación tradicional
5. **Privacidad e Interfaz GMC** ([Capítulo 6](./06-privacidad-e-interfaz-gmc.md)): comprender las restricciones estrictas de privacidad y la frontera de la interfaz on-chain
6. **Preguntas Abiertas** ([Capítulo 7](./07-preguntas-abiertas.md)): revisar los problemas abiertos que la capa de protocolo actual deja sin decidir

Cada capítulo puede leerse de forma independiente, pero una primera lectura completa de principio a fin ayuda a construir un modelo mental integral.
