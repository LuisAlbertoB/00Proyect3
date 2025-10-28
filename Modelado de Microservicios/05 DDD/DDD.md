# Descripción del Diagrama de Arquitectura DDD

**Versión:** 1.0
**Fecha:** 24 de Octubre de 2025

## 1. Introducción

Este documento proporciona una descripción detallada de los componentes y atributos del diagrama de arquitectura basado en Domain-Driven Design (DDD) para el sistema "Reconexión Humana". El diagrama es un mapa visual que articula la estructura de microservicios, los límites de los dominios, los patrones de comunicación y la estrategia de datos.

---

## 2. Atributos y Componentes del Diagrama

A continuación, se desglosan los elementos clave representados en el diagrama.

### 2.1. Bounded Contexts (Contextos Delimitados)

Cada subgrafo rectangular de color representa un **Bounded Context**, un límite explícito donde un modelo de dominio y su lenguaje ubicuo son consistentes.

*   **`AuthIdentity` (Contexto de Identidad):**
    *   **Atributos:** Gestiona la identidad central del usuario.
    *   **Categoría:** Subdominio de Soporte.

*   **`SocialConnect` (Core Domain):**
    *   **Atributos:** Orquesta la creación de contenido y las interacciones sociales. Es el corazón del negocio.
    *   **Categoría:** Dominio Principal (Core Domain), resaltado en rojo.

*   **`MessagingService` (Contexto de Mensajería):**
    *   **Atributos:** Maneja la comunicación en tiempo real entre usuarios.
    *   **Categoría:** Subdominio de Soporte.

*   **`RiskMitigation` (Contexto de Riesgo):**
    *   **Atributos:** Analiza el comportamiento de forma asíncrona para identificar perfiles de riesgo. Está funcionalmente aislado.
    *   **Categoría:** Parte del Dominio Principal, pero con un modelo de datos separado.

*   **`ResourcesDocs` (Contexto de Recursos):**
    *   **Atributos:** Gestiona el contenido de ayuda y recursos de salud.
    *   **Categoría:** Subdominio de Soporte.

*   **`PasswordGuardian` y `NotificationService`:**
    *   **Atributos:** Realizan tareas auxiliares y bien definidas.
    *   **Categoría:** Subdominios Genéricos.

### 2.2. Aggregate Roots (Raíces de Agregado)

Dentro de cada contexto, se identifica el **Aggregate Root**, que es la entidad principal que garantiza la consistencia de las operaciones.

*   **`User` (en AuthIdentity):** La raíz para todas las operaciones de identidad. Asegura que un usuario y su perfil se creen de forma atómica.
*   **`Publication` (en SocialConnect):** La raíz para las interacciones. Un `like` o un `comment` deben aplicarse a un agregado `Publication` válido.
*   **`Conversation` (en MessagingService):** La raíz para los mensajes. Un `message` solo puede ser enviado dentro de una `Conversation` existente y por un participante válido.
*   **`RiskProfile` (en RiskMitigation):** La unidad de análisis para un usuario.
*   **`Resource` (en ResourcesDocs):** La unidad de contenido de ayuda.

### 2.3. Estrategia de Persistencia (Bases de Datos)

El icono de base de datos en cada contexto refleja el patrón **Database per Service** y la **persistencia políglota**.

*   **`users_db (PostgreSQL)`:** Elegido para `AuthIdentity` por su necesidad de transacciones ACID y consistencia fuerte.
*   **`Bases de Datos Políglotas (MongoDB, Neo4j, Redis)`:** `SocialConnect` utiliza la mejor herramienta para cada trabajo: MongoDB para documentos flexibles (publicaciones), Neo4j para el grafo social y Redis para contadores rápidos.
*   **`messaging_db (Cassandra)`:** Ideal para `MessagingService` por su alta capacidad de escritura y escalabilidad horizontal, perfecta para chats.
*   **`risk_analysis_db (Data Warehouse)`:** `RiskMitigation` utiliza una base de datos optimizada para análisis y agregaciones complejas.

### 2.4. Patrones de Comunicación

Las flechas y líneas del diagrama representan los dos patrones de comunicación principales.

*   **Comunicación Síncrona (Líneas continuas):**
    *   **Atributos:** Llamadas API REST/HTTP que requieren una respuesta inmediata.
    *   **Componentes:**
        *   **`API Gateway`:** Punto de entrada único que enruta las peticiones del `Cliente` a los servicios internos.
        *   **Llamadas Servicio a Servicio:** Acoplamiento controlado para obtener datos necesarios en el momento (ej. `AuthSvc` -> `GuardianSvc`).

*   **Comunicación Asíncrona (Líneas hacia/desde el Broker):**
    *   **Atributos:** Patrón orientado a eventos que desacopla los servicios.
    *   **Componentes:**
        *   **`Message Broker (RabbitMQ)`:** Intermediario que recibe eventos de los "publicadores" y los entrega a los "suscriptores".
        *   **Eventos (ej. `UserRegistered`):** Notificaciones de negocio que se publican en el broker. Este flujo garantiza la resiliencia del sistema.

### 2.5. Actores y Sistemas Externos

Son componentes fuera de nuestro sistema principal con los que interactuamos.

*   **`Cliente`:** La aplicación de usuario final (web o móvil) que consume la API a través del Gateway.
*   **`API Externa (Have I Been Pwned)`:** Un servicio de terceros que `PasswordGuardian` consulta para verificar la seguridad de las contraseñas.
*   **`Servicios Push (APNS / FCM)`:** Plataformas de Apple y Google que `NotificationService` utiliza para enviar notificaciones a los dispositivos móviles.

---

Este desglose de atributos y componentes proporciona una comprensión profunda de cómo los principios de DDD y los patrones de arquitectura de microservicios se han materializado en el diseño del sistema "Reconexión Humana".

