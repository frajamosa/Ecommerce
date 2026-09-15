## Purpose

Permite aprender integracion de pagos, idempotencia y estados sin transmitir datos financieros reales ni depender de un proveedor externo.

## ADDED Requirements

### Requirement: Pago exclusivamente simulado
El sistema SHALL aceptar un codigo de simulacion documentado y MUST NOT solicitar, almacenar ni registrar numeros de tarjeta, CVV o credenciales financieras reales.

#### Scenario: Codigo aprobado
- **GIVEN** un pedido propio en `pending_payment`
- **WHEN** el cliente usa el codigo de prueba aprobado
- **THEN** se registra un intento `approved` por el total autoritativo del pedido

#### Scenario: Codigo rechazado
- **GIVEN** un pedido propio en `pending_payment`
- **WHEN** el cliente usa el codigo de prueba rechazado
- **THEN** se registra un intento `declined` con un motivo seguro y controlado

#### Scenario: Dato con apariencia de tarjeta
- **WHEN** la entrada contiene un numero de tarjeta o un codigo no permitido
- **THEN** la API rechaza la solicitud y no persiste el valor recibido

### Requirement: Intentos de pago idempotentes
La API SHALL exigir una clave de idempotencia y asociarla al usuario, pedido, monto y contenido del intento.

#### Scenario: Repeticion identica
- **WHEN** se repite un intento con la misma clave y el mismo contenido
- **THEN** se devuelve el resultado original sin aplicar nuevamente efectos al pedido o inventario

#### Scenario: Clave reutilizada con otro contenido
- **WHEN** una clave existente se usa con un pedido o codigo diferente
- **THEN** la API responde conflicto y no procesa el nuevo intento

### Requirement: Validacion del pedido
La API MUST verificar propiedad, estado pendiente, moneda y monto del pedido antes de ejecutar el simulador.

#### Scenario: Pedido ya pagado
- **WHEN** se intenta pagar un pedido que ya esta `paid`
- **THEN** la API rechaza la transicion sin crear un segundo cobro simulado