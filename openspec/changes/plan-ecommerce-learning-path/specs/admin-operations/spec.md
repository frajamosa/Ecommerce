## Purpose

Permite a administradores mantener el catalogo y supervisar pedidos y usuarios mediante operaciones auditables que siguen protegidas por la API.

## ADDED Requirements

### Requirement: Acceso administrativo por rol
Las rutas y endpoints administrativos SHALL requerir una sesion valida con rol `admin` tanto en el BFF como en la API.

#### Scenario: Administrador autorizado
- **WHEN** un administrador abre `/admin`
- **THEN** ve las funciones administrativas habilitadas por el MVP

#### Scenario: Rol alterado en el navegador
- **WHEN** un cliente modifica estado local para aparentar rol `admin`
- **THEN** BFF y API rechazan las operaciones y no exponen datos administrativos

### Requirement: Gestion de productos e inventario
Un administrador SHALL poder crear y editar nombre, SKU, descripcion, precio, categoria, stock y estado activo de productos con validacion y control de concurrencia.

#### Scenario: Crear producto valido
- **WHEN** un administrador envia datos validos y un SKU unico
- **THEN** la API crea el producto y este aparece en el catalogo si queda activo

#### Scenario: Edicion obsoleta
- **WHEN** dos administradores editan la misma version y el segundo envia una version antigua
- **THEN** la API responde conflicto en vez de sobrescribir silenciosamente el cambio reciente

#### Scenario: Retirar producto
- **WHEN** un administrador desactiva un producto
- **THEN** deja de aparecer en el catalogo publico sin borrar las lineas historicas de pedidos

### Requirement: Supervision de pedidos
Un administrador SHALL poder listar y consultar pedidos, filtrar por estado y cancelar solo los estados permitidos, registrando quien realizo el cambio.

#### Scenario: Cancelar pedido pendiente
- **WHEN** un administrador cancela un pedido `pending_payment`
- **THEN** el pedido queda `cancelled`, el stock reservado se libera una vez y se registra el actor

#### Scenario: Cancelar pedido pagado
- **WHEN** se intenta cancelar un pedido `paid` en el MVP
- **THEN** la API rechaza la operacion porque no existen reembolsos reales

### Requirement: Consulta y gestion limitada de usuarios
Un administrador SHALL poder listar usuarios sin secretos y cambiar roles con protecciones que impidan degradarse a si mismo o dejar al sistema sin administradores.

#### Scenario: Listado seguro
- **WHEN** un administrador lista usuarios
- **THEN** recibe identificador, correo, rol, estado y fechas, pero ningun hash, token o secreto

#### Scenario: Ultimo administrador
- **WHEN** una operacion dejaria al sistema sin usuarios `admin`
- **THEN** la API responde conflicto y conserva los roles existentes

### Requirement: Interfaz administrativa responsive y accesible
Las vistas administrativas SHALL ser operables con teclado y en pantallas pequenas, usando tablas adaptables o vistas equivalentes sin perder acciones ni etiquetas.

#### Scenario: Gestion desde pantalla pequena
- **WHEN** un administrador usa un viewport de 320 px
- **THEN** puede consultar y editar datos sin desplazamiento horizontal de toda la pagina
