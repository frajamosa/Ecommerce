## Purpose

Define el comportamiento transversal que hace que las tres aplicaciones puedan ejecutarse, comunicarse y fallar de forma predecible durante todo el aprendizaje.

## ADDED Requirements

### Requirement: Entorno local reproducible
El proyecto SHALL ofrecer una forma documentada de iniciar storefront, BFF, API y PostgreSQL con configuracion local validada y sin depender de servicios pagados.

#### Scenario: Inicio desde una copia limpia
- **GIVEN** una maquina con las versiones requeridas de Node, pnpm y Docker
- **WHEN** la persona sigue las instrucciones de preparacion
- **THEN** las tres aplicaciones y PostgreSQL quedan disponibles en puertos documentados

#### Scenario: Configuracion obligatoria ausente
- **WHEN** una aplicacion inicia sin una variable obligatoria
- **THEN** el proceso termina con un mensaje que identifica la variable faltante sin revelar secretos

### Requirement: Limites de comunicacion
El navegador MUST consumir las funciones de negocio unicamente mediante el BFF, y el BFF SHALL consumir la API mediante una URL interna configurada.

#### Scenario: Flujo normal de una solicitud
- **WHEN** el storefront solicita una operacion del ecommerce
- **THEN** la solicitud viaja al BFF y desde este a la API con un identificador de correlacion

### Requirement: Respuestas y errores consistentes
Los endpoints HTTP SHALL devolver codigos de estado correctos y un formato de error estable que incluya codigo, titulo, detalle seguro, estado e identificador de correlacion.

#### Scenario: Error de validacion
- **WHEN** una solicitud contiene datos invalidos
- **THEN** el cliente recibe estado 400 con los campos invalidos y no recibe trazas internas

#### Scenario: Recurso inexistente
- **WHEN** se solicita un identificador que no existe
- **THEN** el cliente recibe estado 404 en el formato de error comun

### Requirement: Estado de salud
El BFF y la API SHALL exponer endpoints de salud que distingan entre proceso activo y dependencias listas.

#### Scenario: Base de datos no disponible
- **WHEN** PostgreSQL no responde
- **THEN** la comprobacion de disponibilidad de la API falla sin exponer credenciales de conexion