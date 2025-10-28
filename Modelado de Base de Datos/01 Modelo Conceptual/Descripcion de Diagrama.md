# Descripción del Diagrama del Modelo Conceptual

**Versión:** 1.0
**Fecha:** 24 de Octubre de 2025

Este documento proporciona una explicación detallada del diagrama Entidad-Relación (ER) que representa el modelo conceptual del sistema "Reconexión Humana".

---

## 1. Propósito del Diagrama

El diagrama del modelo conceptual es una representación visual de alto nivel de las principales entidades de datos del sistema y las relaciones que existen entre ellas. Su objetivo es:

*   **Comunicar la estructura:** Ofrecer una visión clara y unificada de la arquitectura de datos a todos los miembros del equipo (desarrolladores, analistas, stakeholders).
*   **Validar el entendimiento:** Asegurar que el modelo de datos refleja correctamente los requerimientos funcionales y las reglas de negocio.
*   **Servir como base:** Actuar como el punto de partida para el diseño de los modelos lógicos y físicos de las diferentes bases de datos que componen el sistema.

---

## 2. Descripción de Componentes

El diagrama está organizado en torno a los contextos de dominio clave del sistema.

### 2.1. Contexto de Identidad y Perfil

*   **`User`:** Es la entidad raíz. Contiene los datos críticos para la autenticación (`email`, `password_hash`) y la identificación única (`user_id`, `username`). Las restricciones `UNIQUE` en `username` y `email` son fundamentales para la integridad del sistema.
*   **`UserProfile`:** Contiene datos públicos y descriptivos del usuario. La relación uno a uno (`||--||`) con `User` indica que cada usuario tiene exactamente un perfil.

### 2.2. Contexto de Contenido y Social

*   **`Publication`:** Es una entidad generalizada que representa cualquier contenido creado por un usuario (un post, una story, etc.). El campo `type` distingue entre ellos. Esta decisión de diseño simplifica enormemente las interacciones, ya que `Comment` y las relaciones de `likes` solo necesitan apuntar a `Publication`.
*   **`Media`:** Representa un archivo multimedia. Su relación uno a muchos (`||--o{`) con `Publication` permite que una publicación contenga múltiples imágenes o videos.
*   **`Comment`:** Un comentario hecho por un `User` en una `Publication`.

### 2.3. Contexto de Mensajería

*   **`Conversation`:** Representa una sala de chat. El campo `type` diferencia entre conversaciones privadas y grupales.
*   **`Message`:** Un mensaje individual enviado por un `User` dentro de una `Conversation`.

### 2.4. Contexto de Salud (Aislado)

*   **`RiskProfile` y `Resource`:** Estas entidades están visualmente separadas para reforzar la regla de negocio de **aislamiento de datos (RNF-SECURITY-01)**. La única conexión con el resto del sistema es a través del `user_id` en `RiskProfile`, que debe ser manejado con extremo cuidado.

---

## 3. Interpretación de las Relaciones

El diagrama utiliza la notación de "patas de gallo" para representar la cardinalidad:

*   **Uno a Uno (`||--||`):** Un `User` tiene un `UserProfile`.
*   **Uno a Muchos (`||--o{`):** Un `User` puede crear muchas `Publications`. Una `Publication` puede tener muchos `Comments`.
*   **Muchos a Muchos (`}o--o{`):**
    *   `User }o--o{ User`: Representa relaciones como "follows" y "blocks". Un usuario puede seguir a muchos otros, y ser seguido por muchos.
    *   `User }o--o{ Publication`: Representa los "likes". Un usuario puede dar like a muchas publicaciones, y una publicación puede recibir likes de muchos usuarios.
    *   `User }o--o{ Conversation`: Representa la participación en chats. Un usuario puede estar en muchas conversaciones, y una conversación (de grupo) puede tener muchos usuarios.

En un modelo lógico, las relaciones de muchos a muchos se implementarán como tablas intermedias (ej. `user_follows`, `publication_likes`, `conversation_participants`).

---

## 4. Decisiones de Diseño Reflejadas

*   **Generalización:** La entidad `Publication` es el ejemplo más claro de generalización para reducir la complejidad y evitar la duplicación de tablas de interacciones.
*   **Anotaciones:** El uso de etiquetas como `UNIQUE`, `Nullable` y `Enum` directamente en el diagrama conceptual ayuda a capturar reglas de negocio importantes desde una etapa temprana.
*   **Aislamiento de Contextos:** La separación visual de los contextos (Identidad, Social, Mensajería, Salud) se alinea con la arquitectura de microservicios definida, donde cada contexto será gestionado por un servicio diferente.