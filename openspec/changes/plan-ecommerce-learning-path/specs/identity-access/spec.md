## Purpose

Define identidad, sesion y autorizacion de dominio para que clientes y administradores accedan solamente a las operaciones y datos que les corresponden.

## ADDED Requirements

### Requirement: Registro de cliente
La API SHALL registrar usuarios con correo normalizado y contrasena que cumpla la politica publicada, asignando siempre el rol `customer` desde el registro publico.

#### Scenario: Registro valido
- **WHEN** una persona envia un correo no registrado y una contrasena valida
- **THEN** se crea el usuario sin devolver ni registrar la contrasena o su hash

#### Scenario: Correo duplicado
- **WHEN** una persona intenta registrar un correo ya utilizado ignorando mayusculas
- **THEN** el sistema rechaza la operacion sin crear un segundo usuario

### Requirement: Inicio de sesion seguro
La API SHALL verificar credenciales usando un hash de contrasena resistente y emitir una sesion renovable de corta duracion sin exponer si fallo el correo o la contrasena.

#### Scenario: Credenciales correctas
- **WHEN** un usuario activo presenta credenciales correctas
- **THEN** el BFF establece cookies seguras, HttpOnly y con una politica SameSite documentada

#### Scenario: Credenciales incorrectas
- **WHEN** el correo o la contrasena no coinciden
- **THEN** el sistema responde 401 con un mensaje generico y no crea sesion

### Requirement: Renovacion y cierre de sesion
La API SHALL rotar el token de renovacion en cada uso y SHALL permitir revocar la sesion actual al cerrar sesion.

#### Scenario: Renovacion valida
- **WHEN** vence el acceso y el token de renovacion sigue vigente
- **THEN** el BFF obtiene un nuevo par, reemplaza las cookies y completa una sola repeticion de la solicitud original

#### Scenario: Reutilizacion de token rotado
- **WHEN** se presenta nuevamente un token de renovacion ya reemplazado o revocado
- **THEN** la API rechaza la renovacion y revoca la familia de sesion afectada

#### Scenario: Cierre de sesion
- **WHEN** un usuario cierra sesion
- **THEN** la API revoca la sesion y el BFF elimina sus cookies aunque la API ya no la considere valida

### Requirement: Autorizacion de dominio
La API MUST verificar identidad, rol y propiedad del recurso en cada operacion protegida, independientemente de las decisiones de vista del BFF.

#### Scenario: Cliente consulta un pedido ajeno
- **WHEN** un cliente autenticado solicita un pedido de otro usuario
- **THEN** la API no entrega el pedido y responde sin confirmar su existencia

#### Scenario: Cliente intenta operacion administrativa
- **WHEN** un usuario `customer` invoca un endpoint administrativo
- **THEN** la API responde 403 y no modifica datos

### Requirement: Proteccion de credenciales y datos
El sistema MUST excluir contrasenas, hashes, tokens y secretos de respuestas, logs y repositorio, y SHALL aplicar limites de intentos a los endpoints de autenticacion.

#### Scenario: Multiples intentos fallidos
- **WHEN** un origen supera el limite documentado de intentos de inicio de sesion
- **THEN** recibe estado 429 sin bloquear permanentemente la cuenta