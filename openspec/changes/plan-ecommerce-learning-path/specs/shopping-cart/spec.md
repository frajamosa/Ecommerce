## Purpose

Permite practicar estado de interfaz y persistencia local mientras la persona prepara una compra sin atribuir al cliente precios ni disponibilidad autoritativos.

## ADDED Requirements

### Requirement: Carrito invitado local
El storefront SHALL permitir agregar productos activos al carrito sin iniciar sesion y conservarlos localmente entre recargas en el mismo navegador.

#### Scenario: Agregar un producto
- **GIVEN** un producto activo con stock
- **WHEN** la persona lo agrega con una cantidad valida
- **THEN** el contador, la linea y el subtotal visible se actualizan

#### Scenario: Recargar el navegador
- **GIVEN** existe un carrito local valido
- **WHEN** la persona recarga la aplicacion
- **THEN** el carrito se restaura sin contener datos personales ni credenciales

### Requirement: Edicion segura de cantidades
El storefront SHALL permitir aumentar, disminuir y eliminar lineas, y SHALL impedir cantidades menores que uno o mayores que el limite visible conocido.

#### Scenario: Eliminar la ultima unidad
- **WHEN** la persona elimina una linea del carrito
- **THEN** la linea desaparece y los totales visibles se recalculan

#### Scenario: Persistencia local corrupta
- **WHEN** los datos guardados del carrito no cumplen el formato esperado
- **THEN** el storefront descarta esos datos, inicia un carrito vacio e informa de manera no bloqueante

### Requirement: Totales del cliente son informativos
El carrito SHALL indicar que sus precios y disponibilidad se confirmaran durante el checkout; la API MUST ignorar precios o totales enviados por el navegador.

#### Scenario: Precio cambia antes del checkout
- **GIVEN** el precio visible fue actualizado en la API
- **WHEN** la persona inicia el checkout con datos locales antiguos
- **THEN** la API calcula el total vigente y el sistema informa la diferencia antes de cobrar