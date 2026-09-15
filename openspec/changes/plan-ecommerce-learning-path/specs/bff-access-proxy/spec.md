## Purpose

Establece al BFF como frontera tecnica del navegador para manejar sesion, acceso a vistas y transporte hacia la API sin duplicar decisiones comerciales.

## ADDED Requirements

### Requirement: Proteccion de vistas
El BFF SHALL informar la identidad y las vistas permitidas a partir de una sesion valida y de los roles emitidos por la API.

#### Scenario: Administrador abre administracion
- **GIVEN** una sesion valida con rol `admin`
- **WHEN** el storefront consulta sus permisos
- **THEN** el BFF permite las vistas administrativas

#### Scenario: Cliente abre administracion
- **GIVEN** una sesion valida con rol `customer`
- **WHEN** solicita una vista administrativa
- **THEN** el BFF responde 403 y el storefront muestra una vista de acceso denegado

#### Scenario: Visitante abre una vista autenticada
- **WHEN** no existe una sesion valida
- **THEN** el BFF responde 401 y el storefront dirige al inicio de sesion conservando un retorno seguro

### Requirement: Proxy acotado y transparente
El BFF SHALL reenviar solo rutas, metodos y encabezados incluidos explicitamente, preservando cuerpo y estado relevantes, y MUST impedir que el cliente seleccione el destino upstream.

#### Scenario: Ruta permitida
- **WHEN** el navegador solicita una ruta declarada
- **THEN** el BFF la reenvia a la API configurada con identidad y correlacion, y devuelve su resultado sin reinterpretar reglas comerciales

#### Scenario: Destino o ruta no permitida
- **WHEN** una solicitud intenta definir otra URL, host o ruta no declarada
- **THEN** el BFF la rechaza sin realizar una llamada saliente

### Requirement: BFF sin logica de negocio ni persistencia
El BFF MUST NOT calcular precios, decidir stock, cambiar estados de pedidos, acceder a PostgreSQL ni mantener una copia autoritativa de datos del dominio.

#### Scenario: API rechaza una compra
- **WHEN** la API rechaza el checkout por stock insuficiente
- **THEN** el BFF conserva el codigo y el problema recibido sin reemplazar la decision

### Requirement: Sesion adaptada al navegador
El BFF SHALL mantener los tokens fuera del JavaScript del navegador y SHALL aplicar proteccion CSRF a solicitudes que cambian estado.

#### Scenario: Solicitud mutante sin prueba CSRF
- **WHEN** una sesion por cookies envia una operacion mutante sin el encabezado o token CSRF valido
- **THEN** el BFF rechaza la solicitud antes de reenviarla

### Requirement: Fallos upstream controlados
El BFF SHALL aplicar tiempos limite y convertir indisponibilidad de la API en un error estable, sin reintentar automaticamente operaciones no idempotentes.

#### Scenario: API excede el tiempo limite
- **WHEN** la API no responde dentro del tiempo configurado
- **THEN** el BFF responde 504 con el identificador de correlacion