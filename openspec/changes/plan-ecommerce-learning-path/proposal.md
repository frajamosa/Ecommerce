## Why

El proyecto necesita una ruta de aprendizaje practica y verificable para dominar TypeScript, NestJS y PostgreSQL sin intentar construir todo el ecommerce al mismo tiempo. Definir primero los limites entre storefront, BFF y API permite aprender cada concepto en contexto, evitar logica de negocio duplicada y comprobar el avance con resultados pequenos.

## What Changes

- Crear un recorrido guiado por incrementos verticales: preparar el entorno, entregar una funcion pequena, probarla y reflexionar antes de continuar.
- Crear un storefront responsive con catalogo, busqueda, filtros, detalle de producto, carrito y area administrativa.
- Crear un BFF NestJS como unico punto de entrada del navegador, encargado de sesion, proteccion de vistas y proxy hacia la API, sin reglas comerciales ni persistencia propia.
- Crear una API NestJS que sea la autoridad sobre usuarios, permisos de dominio, catalogo, inventario, pedidos y pagos simulados.
- Crear un esquema PostgreSQL versionado mediante migraciones TypeORM y datos semilla reproducibles.
- Incorporar autenticacion con credenciales, sesiones seguras mediadas por el BFF y roles `customer` y `admin`.
- Incorporar checkout autenticado y un adaptador de pagos simulados con resultados controlables para practicar casos aprobados y rechazados.
- Incorporar pruebas unitarias, de integracion, de contrato y end-to-end en la misma secuencia en que se aprende cada funcionalidad.
- Documentar decisiones, comandos, criterios de aceptacion y puntos de reflexion en espanol; mantener codigo e identificadores en ingles.

### Objetivos del MVP

- Aprender TypeScript aplicandolo de extremo a extremo, sin usar `any` como salida habitual.
- Comprender modulos, controladores, providers, guards, pipes, interceptors y pruebas en NestJS.
- Aprender modelado relacional, claves, restricciones, transacciones, indices, migraciones y consultas en PostgreSQL.
- Completar un flujo demostrable: explorar productos, agregarlos al carrito, autenticarse, crear un pedido, simular su pago y administrarlo segun el rol.

### Fuera del alcance inicial

- Pagos reales, envios reales, facturacion, impuestos por jurisdiccion y devoluciones monetarias.
- Microservicios, Kubernetes, colas distribuidas, Redis, busqueda externa o despliegue de produccion.
- Marketplace con multiples vendedores, variantes complejas, cupones, recomendaciones y multiples monedas.
- Inicio de sesion social, recuperacion de contrasena por correo y autenticacion multifactor.

### Supuestos

- El MVP usa una sola tienda, moneda configurable con valor inicial `CLP` y precios almacenados como enteros en la unidad minima.
- El catalogo es publico; crear pedidos, consultar pedidos propios y acceder a administracion requiere autenticacion.
- El carrito invitado vive en el navegador durante el MVP. Al confirmar la compra, la API vuelve a consultar precios y disponibilidad; nunca confia en totales calculados por el cliente.
- El panel administrativo forma parte del mismo storefront bajo rutas `/admin` y no es una aplicacion separada.
- La autorizacion del BFF protege la experiencia y las vistas; la API repite la autorizacion necesaria para proteger datos y operaciones.

## Capabilities

### New Capabilities

- `platform-foundation`: Entorno reproducible, limites de aplicaciones, configuracion, salud, errores y convenciones transversales.
- `storefront-catalog`: Catalogo y detalle de productos con busqueda, filtros, paginacion y presentacion responsive y accesible.
- `shopping-cart`: Carrito invitado administrado en el storefront, con cantidades, subtotales y persistencia local controlada.
- `identity-access`: Registro, inicio, renovacion y cierre de sesion, roles y autorizacion final en la API.
- `bff-access-proxy`: Sesion segura, permisos de vistas y reenvio transparente y acotado entre navegador y API.
- `checkout-orders`: Validacion autoritativa del carrito, creacion transaccional del pedido y consulta de pedidos propios.
- `simulated-payments`: Intentos de pago idempotentes y resultados simulados aprobados o rechazados, sin proveedores reales.
- `admin-operations`: Vistas y operaciones administrativas para catalogo, inventario, pedidos y usuarios, protegidas por rol.

### Modified Capabilities

Ninguna. El proyecto es nuevo y aun no existen especificaciones base.

## Impact

- El codigo y los artefactos OpenSpec se versionaran en `https://github.com/frajamosa/Ecommerce.git`, usando `origin` y la rama principal `main`.
- Nuevas aplicaciones: `apps/storefront`, `apps/bff` y `apps/api` dentro de un workspace `pnpm`.
- Nuevos paquetes compartidos limitados a configuracion de TypeScript, lint y contratos generados; no se compartiran entidades de persistencia con el front.
- Nuevas dependencias principales: React, Vite, NestJS, TypeORM, PostgreSQL y herramientas de pruebas.
- Nuevos contratos HTTP versionados entre storefront, BFF y API, documentados con OpenAPI.
- Nuevo entorno local con Docker Compose para PostgreSQL; ningun servicio externo ni pago real.
- Cada sesion verificada terminara en un commit pequeno y un push; archivos `.env`, credenciales, tokens, dependencias y salidas generadas permaneceran excluidos.
- El plan completo se implementara en sesiones guiadas. Esta propuesta no autoriza todavia la escritura del codigo de la aplicacion.
