## Purpose

Convierte un carrito autenticado en un pedido consistente, preservando precios historicos y evitando que datos manipulados por el navegador determinen el cobro.

## ADDED Requirements

### Requirement: Previsualizacion autoritativa del checkout
La API SHALL recibir solo identificadores y cantidades, volver a consultar productos activos, precios y stock, y devolver una previsualizacion con total, moneda y diferencias respecto del carrito.

#### Scenario: Carrito vigente
- **WHEN** un cliente autenticado envia productos y cantidades disponibles
- **THEN** recibe una previsualizacion calculada por la API apta para confirmacion

#### Scenario: Precio o disponibilidad cambio
- **WHEN** al menos una linea ya no coincide con el catalogo actual
- **THEN** la respuesta identifica las lineas afectadas y exige aceptar la nueva previsualizacion

### Requirement: Creacion transaccional e idempotente
La API SHALL crear el pedido, sus lineas historicas y la reserva de stock en una sola transaccion, asociando una clave de idempotencia al intento de checkout.

#### Scenario: Pedido creado
- **GIVEN** una previsualizacion vigente y stock suficiente
- **WHEN** el cliente confirma con una clave nueva
- **THEN** se crea exactamente un pedido `pending_payment`, se descuenta el stock reservado y se guardan nombre, SKU, precio y cantidad de cada linea

#### Scenario: Repeticion por problema de red
- **WHEN** el cliente repite la misma confirmacion con la misma clave e igual contenido
- **THEN** recibe el pedido original y no se descuenta stock nuevamente

#### Scenario: Stock concurrente insuficiente
- **WHEN** dos confirmaciones compiten por la ultima unidad
- **THEN** como maximo una confirma la reserva y la otra recibe un conflicto sin pedido parcial

### Requirement: Consulta de pedidos propios
Un cliente SHALL poder listar y abrir solamente sus pedidos con lineas, totales y estados, en orden del mas reciente al mas antiguo.

#### Scenario: Historial del cliente
- **WHEN** un cliente autenticado abre su historial
- **THEN** solo recibe pedidos asociados a su identidad

### Requirement: Maquina de estados de pedido
El sistema MUST aceptar solo transiciones declaradas y SHALL devolver conflicto cuando el estado actual ya no permite la operacion.

#### Scenario: Pago aprobado
- **WHEN** un pago valido se aprueba para un pedido `pending_payment`
- **THEN** el pedido cambia a `paid` una sola vez

#### Scenario: Pago rechazado
- **WHEN** el pago se rechaza definitivamente
- **THEN** el pedido cambia a `payment_failed` y la reserva de stock se libera una sola vez