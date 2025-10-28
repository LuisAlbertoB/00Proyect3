# Entidades Principales, Relaciones y Reglas de Negocio

**Versión:** 1.0
**Fecha:** 24 de Octubre de 2025

Este documento traduce los requerimientos del sistema en un conjunto de entidades de datos, las relaciones que las conectan y las reglas de negocio que gobiernan su comportamiento.

---

## 1. Entidades Principales

Estas son las entidades fundamentales que componen el dominio del sistema "Reconexión Humana".

*   **User:** La entidad central que representa a un individuo en la plataforma. Contiene los datos de autenticación y la información esencial.
*   **UserProfile:** Datos públicos y editables del usuario, como su biografía y foto de perfil. Está directamente asociada a un `User`.
*   **Publication:** Una **generalización** para agrupar todo el contenido generado por el usuario (`Post`, `Story`). Esto simplifica las interacciones como `Likes` y `Comments`, ya que pueden apuntar a una única entidad `Publication` en lugar de a múltiples tipos de contenido.
*   **Media:** Representa un archivo multimedia individual (imagen, video) que puede estar asociado a una `Publication` o a un `Message`.
*   **Comment:** Un comentario de texto realizado por un `User` en una `Publication` específica.
*   **Conversation:** Representa una sala de chat, que puede ser una conversación privada (dos participantes) o un grupo (múltiples participantes).
*   **Message:** Un mensaje individual (texto o multimedia) enviado por un `User` dentro de una `Conversation`.
*   **Resource:** Contenido educativo y de apoyo gestionado por el microservicio `ResourcesDocs`.
*   **RiskProfile:** Un perfil de datos interno y aislado, asociado a un `User`, que contiene la puntuación de riesgo calculada por el sistema.

---

## 2. Relaciones Clave

Las relaciones definen cómo interactúan las entidades entre sí.

*   Un `User` **tiene un** `UserProfile` (Relación 1 a 1).
*   Un `User` **crea muchas** `Publications` (Relación 1 a N).
*   Un `User` **escribe muchos** `Comments` (Relación 1 a N).
*   Un `User` **envía muchos** `Messages` (Relación 1 a N).

*   Una `Publication` **puede tener muchos** `Comments` (Relación 1 a N).
*   Una `Publication` **puede contener muchos** `Media` (Relación 1 a N).

*   Una `Conversation` **contiene muchos** `Messages` (Relación 1 a N).

*   La relación de **seguimiento** entre `Users` es de muchos a muchos (N a M).
*   La relación de **bloqueo** entre `Users` es de muchos a muchos (N a M).
*   La relación de **"like"** entre `User` y `Publication` es de muchos a muchos (N a M).
*   La relación de **participación** entre `User` y `Conversation` es de muchos a muchos (N a M).

---

## 3. Reglas de Negocio Críticas

Estas reglas imponen restricciones y lógica sobre el modelo de datos.

1.  **Unicidad de Identidad:** Los campos `username` y `email` en la entidad `User` deben ser únicos en todo el sistema (Regla derivada de RF-AUTH-01).
2.  **Propiedad de Contenido:** Solo el `User` que creó una `Publication` o un `Comment` puede eliminarlo.
3.  **Interacción Única:** Un `User` solo puede dar "like" una vez a una misma `Publication`. Un segundo intento debe deshacer el "like".
4.  **Lógica de Bloqueo:** Si `UserA` bloquea a `UserB`, el sistema debe impedir que `UserB` vea el `UserProfile` o las `Publications` de `UserA`, y que pueda iniciar una `Conversation` con `UserA` (Regla derivada de RF-SOCIAL-02).
5.  **Temporalidad de Contenido:** Las `Publications` de tipo "Story" deben tener una fecha de expiración. El sistema no debe mostrarlas en los feeds después de esa fecha (Regla derivada de RF-CONTENT-02).
6.  **Aislamiento de Riesgo:** El `user_id` es la única clave foránea permitida para conectar el mundo social con la entidad `RiskProfile`. No debe existir ninguna otra relación directa o indirecta entre las entidades del `SocialConnect` y las del `RiskMitigation` para cumplir con RNF-SECURITY-01.