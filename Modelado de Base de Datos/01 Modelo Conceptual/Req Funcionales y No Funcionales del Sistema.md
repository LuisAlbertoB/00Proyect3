# Requerimientos Funcionales y No Funcionales del Sistema

**Versión:** 1.0
**Fecha:** 24 de Octubre de 2025

Este documento detalla los requerimientos que guían el diseño y la implementación del sistema "Reconexión Humana", sirviendo como base para el modelado de la base de datos y la arquitectura de microservicios.

---

## 1. Requerimientos Funcionales (RF)

Los requerimientos funcionales describen **qué debe hacer** el sistema.

*   **RF-AUTH-01: Registro de Usuario:** Los usuarios deben poder crear una cuenta con un nombre de usuario, email y contraseña.
*   **RF-AUTH-02: Autenticación:** Los usuarios deben poder iniciar sesión con su email y contraseña para obtener un token de acceso.
*   **RF-AUTH-03: Gestión de Perfil:** Los usuarios deben poder ver y editar su perfil público (nombre completo, biografía, foto de perfil).

*   **RF-CONTENT-01: Creación de Publicaciones:** Los usuarios deben poder crear publicaciones permanentes (Posts) con texto y contenido multimedia.
*   **RF-CONTENT-02: Creación de Contenido Efímero:** Los usuarios deben poder crear publicaciones temporales (Stories) que desaparecen después de 24 horas.

*   **RF-SOCIAL-01: Seguimiento de Usuarios:** Los usuarios deben poder seguir y dejar de seguir a otros usuarios.
*   **RF-SOCIAL-02: Bloqueo de Usuarios:** Los usuarios deben poder bloquear a otros usuarios para impedir cualquier tipo de interacción.

*   **RF-INTERACTION-01: Dar "Like":** Los usuarios deben poder indicar que les gusta una publicación.
*   **RF-INTERACTION-02: Comentar:** Los usuarios deben poder dejar comentarios de texto en las publicaciones.

*   **RF-MESSAGING-01: Mensajería Privada:** Los usuarios deben poder iniciar conversaciones uno a uno con otros usuarios.
*   **RF-MESSAGING-02: Mensajería Grupal:** Los usuarios deben poder crear conversaciones con múltiples participantes.

*   **RF-FEED-01: Feed Personalizado:** El sistema debe presentar a cada usuario un feed de contenido relevante de las personas a las que sigue.

*   **RF-RISK-01: Análisis de Comportamiento:** El sistema debe analizar de forma asíncrona y anónima los patrones de interacción para identificar perfiles de riesgo.
*   **RF-RISK-02: Sugerencia de Recursos:** El sistema debe poder sugerir recursos de salud (artículos, guías) a los usuarios identificados con perfiles de riesgo, sin revelar el motivo de la sugerencia.

---

## 2. Requerimientos No Funcionales (RNF)

Los requerimientos no funcionales describen **cómo debe ser** el sistema en términos de calidad y restricciones.

*   **RNF-SECURITY-01: Aislamiento de Datos:** Debe existir una separación estricta y total entre los datos del contexto social (`SocialConnect`) y los datos del contexto de análisis de riesgo (`RiskMitigation`). La única conexión permitida es el `user_id`.
*   **RNF-SECURITY-02: Contraseñas Seguras:** El sistema debe verificar que las contraseñas de los nuevos usuarios no estén en bases de datos de contraseñas comprometidas conocidas. Las contraseñas deben almacenarse siempre hasheadas con un algoritmo robusto (ej. Argon2, bcrypt).

*   **RNF-PERFORMANCE-01: Baja Latencia en Lecturas Críticas:** La carga del feed de un usuario y la entrega de mensajes en tiempo real deben tener una latencia mínima para garantizar una buena experiencia de usuario.
*   **RNF-PERFORMANCE-02: Escrituras Asíncronas:** Las operaciones de escritura que no requieren una respuesta inmediata (ej. registrar una vista, alimentar el motor de análisis) deben procesarse de forma asíncrona para no bloquear al usuario.

*   **RNF-SCALABILITY-01: Escalabilidad Horizontal:** Cada microservicio debe poder escalarse de forma independiente para manejar la carga. El sistema debe estar preparado para un alto volumen de lecturas (feeds) y escrituras (interacciones, mensajes).

*   **RNF-AVAILABILITY-01: Alta Disponibilidad:** Los servicios críticos de cara al usuario (`AuthIdentity`, `SocialConnect`) deben tener una alta disponibilidad.
*   **RNF-AVAILABILITY-02: Resiliencia y Tolerancia a Fallos:** La falla de un servicio no crítico (ej. `RiskMitigation`, `NotificationService`) no debe afectar la funcionalidad principal de la red social. El uso de un broker de mensajes ayuda a lograr este objetivo.

*   **RNF-MAINTAINABILITY-01: Bajo Acoplamiento:** Los microservicios deben estar débilmente acoplados para permitir que los equipos trabajen en ellos y los desplieguen de forma independiente.

*   **RNF-CONSISTENCY-01: Consistencia Fuerte para Identidad:** Las operaciones en el servicio `AuthIdentity` (registro, cambio de contraseña) deben ser transaccionales y fuertemente consistentes (ACID).
*   **RNF-CONSISTENCY-02: Consistencia Eventual para Datos Sociales:** Se acepta la consistencia eventual para datos no críticos. Por ejemplo, si un usuario da "like" a un post, el contador puede tardar unos segundos en actualizarse en todas las vistas.