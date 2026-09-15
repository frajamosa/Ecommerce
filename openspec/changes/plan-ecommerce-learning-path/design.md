## Context

El repositorio esta vacio salvo por OpenSpec. La motivacion y el alcance funcional estan en [proposal.md](./proposal.md), y los contratos observables estan separados en ocho archivos bajo `specs/`.

La restriccion principal es educativa: la persona usuaria escribira la aplicacion con guia paso a paso. Por eso el diseño favorece limites visibles, herramientas comunes y ciclos cortos antes que una arquitectura distribuida. El navegador debe hablar solo con el BFF; el BFF puede resolver sesion y acceso a vistas, pero toda regla comercial y persistencia pertenece a la API.

## Goals / Non-Goals

**Goals:**

- Hacer evidente donde vive cada responsabilidad y permitir probar cada limite por separado.
- Enseñar TypeScript estricto, fundamentos de NestJS y PostgreSQL relacional usando un flujo real de ecommerce.
- Poder entregar valor vertical desde temprano: primero un producto leido desde PostgreSQL, luego catalogo, identidad, pedido, pago y administracion.
- Ejecutar todo el MVP localmente con costos externos iguales a cero.
- Conservar trazabilidad entre escenario OpenSpec, tarea, codigo y prueba.

**Non-Goals:**

- Diseñar una plataforma de escala global o anticipar microservicios.
- Ocultar PostgreSQL detras de abstracciones que impidan aprender SQL, indices y transacciones.
- Compartir entidades TypeORM o implementacion NestJS con el storefront o el BFF.
- Resolver despliegue productivo, entrega de correos, pagos o logistica reales durante el MVP.

## Decisions

### 1. Monorepo pnpm con tres aplicaciones desplegables

Estructura objetivo:

```text
ECommerce/
├── apps/
│   ├── storefront/       # React + Vite
│   ├── bff/              # NestJS, frontera del navegador
│   └── api/              # NestJS, dominio y PostgreSQL
├── packages/
│   ├── eslint-config/    # convenciones compartidas
│   ├── tsconfig/         # opciones estrictas compartidas
│   └── api-contract/     # tipos generados desde OpenAPI, cuando exista el contrato
├── infra/
│   └── compose.yaml      # PostgreSQL local
├── openspec/
├── package.json
└── pnpm-workspace.yaml
```

`pnpm` evita instalaciones duplicadas y permite ejecutar comprobaciones de todo el workspace. No se incorporara Turborepo al inicio: el ahorro de cache no compensa una herramienta adicional mientras se aprenden las bases.

Alternativa considerada: tres repositorios. Representa mejor despliegues independientes, pero dificulta cambios atomicos de contrato y añade trabajo de versionado prematuro.

### 2. Storefront con React, Vite y TypeScript

El front usara React Router para rutas, TanStack Query para estado remoto y CSS Modules con variables globales para el diseño responsive. El estado efimero del carrito se modelara con reducer y Context, validando su version persistida antes de hidratarla. Los criterios de busqueda y filtros viven en la URL; no se duplican como una segunda fuente de verdad.

La eleccion de Vite mantiene claro que el storefront es un cliente del BFF. Next.js agregaria un segundo servidor con responsabilidades parecidas al BFF y haria mas dificil aprender el limite solicitado.

### 3. BFF y API como aplicaciones NestJS independientes

El BFF y la API se crean como proyectos NestJS separados dentro del workspace. Ambos usan modulos, controladores, providers, pipes, guards, filtros e interceptores, pero con responsabilidades distintas.

| Responsabilidad | Storefront | BFF | API |
|---|---:|---:|---:|
| Presentar vistas y estado de UI | Si | No | No |
| Guardar carrito invitado | Si, local | No | No |
| Guardar tokens en cookies seguras | No | Si | Emite/valida |
| Autorizar acceso a vistas | Refleja | Si | Provee identidad/roles |
| Calcular precio, stock y estados | No | No | Si |
| Persistir dominio | No | No | Si |
| Reenviar HTTP | No | Si | No |
| Autorizar recursos y operaciones | No | No | Si |

La API sera un monolito modular con modulos `auth`, `users`, `catalog`, `orders`, `payments`, `admin` y `health`. Dentro de cada modulo se comienza con la estructura NestJS sencilla `controller -> service -> repository`; se extraen objetos de dominio solo donde una regla lo justifique.

Alternativa considerada: microservicios por dominio. Se descarta porque transacciones, observabilidad y consistencia distribuidas desviarian el aprendizaje de NestJS y PostgreSQL fundamentales.

### 4. Contrato HTTP explicito y versionado

- El storefront usa rutas relativas `/api/*`; Vite las redirige al BFF en desarrollo.
- El BFF expone `/api/*` al navegador y solo define rutas permitidas.
- La API expone internamente `/api/v1/*`.
- La API genera OpenAPI desde DTOs validados. Cuando el primer contrato se estabilice, `packages/api-contract` se genera desde ese documento; no se escriben copias manuales de tipos.
- Los errores usan `application/problem+json` con extensiones `code`, `fieldErrors` y `correlationId`.

El BFF no sera un proxy abierto con URL controlada por el usuario. Cada grupo de rutas se declara o se incluye en una lista fija de metodo y path. Reenvia query, cuerpo, respuesta y algunos encabezados; elimina encabezados hop-by-hop, cookies internas y cualquier host proporcionado por el navegador.

Alternativa considerada: importar DTOs de la API en el BFF. Acopla el proceso de despliegue y puede arrastrar decoradores o entidades; OpenAPI generado conserva el contrato sin compartir implementacion.

### 5. PostgreSQL con TypeORM y migraciones explicitas

TypeORM se usara con `synchronize: false` desde el primer dia. Cada cambio de esquema se expresa como migracion reversible y revisable. Las entidades ayudan a aprender la integracion NestJS, mientras consultas importantes se inspeccionan con SQL y `EXPLAIN` durante las sesiones correspondientes.

Modelo inicial:

| Tabla | Datos principales | Restricciones e indices relevantes |
|---|---|---|
| `users` | id, email normalizado, password_hash, role, active, timestamps | email unico, role controlado |
| `refresh_sessions` | id, user_id, family_id, token_hash, expires_at, revoked_at | hash unico, indice por usuario/familia |
| `categories` | id, name, slug, active | slug unico |
| `products` | id, category_id, sku, slug, name, description, price_amount, currency, stock_quantity, active, version, timestamps | SKU y slug unicos; checks de precio/stock; indices por activo, categoria y precio |
| `orders` | id, user_id, status, payment_status, currency, total_amount, checkout_key, request_hash, stock_released_at, timestamps | checkout unico por usuario; checks de monto y estados |
| `order_items` | id, order_id, product_id, sku, product_name, unit_price, quantity, subtotal | cantidad positiva; instantanea historica |
| `payment_attempts` | id, order_id, user_id, idempotency_key, request_hash, status, safe_reason, amount, currency, timestamps | clave unica por usuario; monto no negativo |
| `audit_events` | id, actor_user_id, action, resource_type, resource_id, metadata segura, created_at | indices por recurso y fecha |

Los UUID identifican recursos publicos. Dinero se almacena como entero en unidad minima con moneda ISO; el MVP inicia en `CLP`. Las fechas se guardan con zona horaria y se serializan en UTC. Los productos se desactivan, no se borran fisicamente, para conservar historia.

Alternativa considerada: Prisma. Tiene una experiencia excelente, pero TypeORM expone de forma mas directa repositorios, QueryBuilder, bloqueos y migraciones dentro de NestJS, alineados con el objetivo de aprender PostgreSQL.

### 6. Flujo de autenticacion mediado por el BFF

1. El storefront envia credenciales y prueba CSRF al BFF.
2. El BFF reenvia las credenciales a la API mediante HTTPS en entornos no locales.
3. La API normaliza correo, verifica Argon2id y emite access token corto y refresh token rotatorio.
4. El BFF coloca ambos en cookies `HttpOnly`; `Secure` se exige fuera de desarrollo y `SameSite=Lax`. JavaScript nunca recibe los tokens.
5. `/api/session` pide la identidad a la API y devuelve solo datos seguros y permisos de vista.
6. Si el acceso expiro, el BFF intenta una renovacion y repite como maximo una vez la solicitud original. No reintenta automaticamente operaciones no idempotentes si no poseen clave.
7. El cierre de sesion revoca en la API y limpia cookies en el BFF incluso si la revocacion responde que ya estaba cerrada.

El refresh token se almacena solo como hash en PostgreSQL. La rotacion usa familias para detectar reutilizacion. Los endpoints de autenticacion tienen limitacion de intentos. Las operaciones mutantes del BFF requieren un token CSRF de doble envio mediante cookie legible mas encabezado.

El BFF traduce `anonymous`, `customer` y `admin` a un conjunto fijo de areas de UI. Esto es politica tecnica de navegacion, no permiso comercial. La API aplica de nuevo rol y propiedad con guards y consultas acotadas al usuario.

### 7. Catalogo consultado en PostgreSQL

La API valida `q`, `category`, `inStock`, `minPrice`, `maxPrice`, `sort`, `page` y `pageSize` antes de construir la consulta. TypeORM QueryBuilder usa parametros enlazados. La paginacion es por offset durante el MVP y siempre agrega `id` como desempate estable.

El indice inicial cubre estado activo, categoria y precio. La busqueda textual comienza con `ILIKE` sobre un conjunto pequeno. Solo despues de medir se considerara `pg_trgm` o busqueda de texto completo; asi la optimizacion nace de evidencia y permite practicar `EXPLAIN ANALYZE`.

### 8. Carrito local, checkout autoritativo y stock transaccional

El carrito invitado contiene `productId`, cantidad y una instantanea visible. Un esquema versionado valida `localStorage`; no se guardan datos personales.

`POST /checkout/preview` recibe unicamente IDs, cantidades y valores esperados. La API devuelve valores actuales y diferencias. `POST /orders` vuelve a validar dentro de una transaccion. Bloquea o actualiza condicionalmente filas de producto, crea pedido y lineas historicas, y descuenta el stock reservado de forma atomica.

Cada confirmacion lleva `Idempotency-Key`. En `orders` se conserva la clave junto con un hash canonico del contenido:

- misma clave + mismo hash: devuelve el pedido existente;
- misma clave + contenido distinto: responde 409;
- clave nueva: procesa normalmente.

No se confia en el total del cliente. Si precio o stock difiere de lo aceptado, la API responde 409 con una previsualizacion segura y no crea ningun registro parcial.

### 9. Pagos simulados como adaptador de dominio

`PaymentGateway` sera una interfaz de la API. `SimulatedPaymentGateway` acepta solo codigos constantes de prueba, por ejemplo `SIMULATE_APPROVED` y `SIMULATE_DECLINED`. No existe un campo de tarjeta.

El servicio de pagos valida propiedad, pedido, monto, moneda e idempotencia y luego invoca el adaptador. Aprobado cambia el pedido de `pending_payment` a `paid`. Rechazado lo cambia a `payment_failed` y libera stock en la misma transaccion, protegido por `stock_released_at` para hacerlo una sola vez.

Un pedido pendiente abandonado conserva la reserva en el MVP. El administrador puede cancelarlo y liberar stock. La expiracion automatica queda fuera del alcance porque requeriria un scheduler y decisiones de negocio adicionales.

### 10. Administracion y concurrencia optimista

El panel reutiliza el storefront bajo `/admin`. Sus loaders o guards consultan `/api/session`, pero las pantallas siempre manejan tambien respuestas 401 y 403 del servidor.

Los productos usan una columna `version`. Cada edicion envia la version leida; si ya cambio, la API responde 409 con datos actuales. Las desactivaciones preservan pedidos historicos. Los cambios sensibles crean `audit_events` sin secretos.

La cuenta inicial de administrador se crea mediante un comando de semilla local que exige valores por variables de entorno. La API impide quitar el rol al usuario actual y verifica transaccionalmente que permanezca al menos un administrador activo.

### 11. Validacion, errores, logs y configuracion

- TypeScript usa `strict`, `noUncheckedIndexedAccess` y limites explicitos para valores desconocidos.
- DTOs de entrada usan transformacion controlada, lista blanca y rechazo de propiedades no declaradas.
- Variables de entorno se validan al inicio.
- Un interceptor propaga o crea `X-Correlation-Id`.
- Los logs son estructurados y omiten credenciales, tokens, cuerpos de login y entradas con apariencia de tarjeta.
- Tiempos limite y tamano maximo de cuerpo se configuran en BFF y API.
- Secretos viven en `.env` ignorados; se versiona solamente `.env.example`.

### 12. Estrategia de pruebas en piramide

| Nivel | Herramienta propuesta | Que demuestra |
|---|---|---|
| Unitario front | Vitest + Testing Library | reducer del carrito, filtros, permisos visuales y componentes |
| Unitario NestJS | Jest | servicios, guards, validadores, estados e idempotencia |
| Integracion API | Jest + Supertest + PostgreSQL aislado | entidades, migraciones, indices, repositorios, transacciones y HTTP |
| Contrato | OpenAPI validado y cliente generado | BFF y API coinciden en metodos, datos y errores |
| Integracion BFF | Jest + Supertest + upstream falso | cookies, CSRF, allowlist, timeout y preservacion de respuestas |
| End-to-end | Playwright | catalogo, carrito, login, compra simulada y administracion responsive |

La base de integracion se recrea desde migraciones y semillas. Nunca se usa SQLite como sustituto, porque ocultaria comportamientos de PostgreSQL. Los escenarios OpenSpec nombran o enlazan las pruebas que los verifican.

### 13. Cadencia educativa de cada tarea

Cada sesion sigue el mismo ciclo:

1. **Objetivo observable:** que debe funcionar al terminar.
2. **Conceptos:** de dos a cuatro ideas nuevas como maximo.
3. **Tu implementacion:** la persona escribe un cambio pequeno con pistas, no con una solucion completa por defecto.
4. **Comprobacion:** comando, prueba automatica y una comprobacion manual breve.
5. **Explicacion:** la persona describe con sus palabras por que funciona.
6. **Registro:** solo entonces se marca la tarea y se pasa a la siguiente.

Cada incremento termina funcionando de extremo a extremo. Cuando una sesion introduzca un error deliberado, se restaurara el estado verde antes de cerrar.

### 14. Versionado continuo en GitHub

El repositorio local se inicializara con rama `main` y remoto `origin` apuntando a `https://github.com/frajamosa/Ecommerce.git`. El remoto estaba vacio al incorporarlo al proyecto, por lo que el primer push publicara los artefactos OpenSpec y la configuracion segura inicial sin reconciliar historias previas.

Cada sesion completada y verificada producira un commit pequeno con un mensaje descriptivo y luego un push a `origin/main`. Una tarea no se considera terminada solo por haber sido subida: primero debe cumplir su criterio de verificacion y despues se marca en `tasks.md`, se incluye esa evidencia documental en el commit y se publica.

Antes de cada commit se revisaran `git status` y el diff preparado. `.gitignore` excluira como minimo `.env`, secretos, tokens, `node_modules`, builds, cobertura, logs, archivos temporales y datos/volumenes locales de PostgreSQL. Se versionaran `.env.example`, el lockfile, migraciones, pruebas, documentacion, configuracion OpenSpec y los workflows generados necesarios para reproducir el proyecto.

Durante el MVP educativo se trabajara directamente en `main` porque hay una sola persona implementadora y cada incremento debe quedar verde. Si aparecen colaboradores o trabajo concurrente, la adopcion de ramas y pull requests se definira mediante una nueva decision OpenSpec. No se reescribira historial ya publicado con force-push.

## Risks / Trade-offs

- [Un plan completo puede resultar abrumador] -> Ejecutar una sola sesion de 45 a 120 minutos y mostrar siempre el siguiente objetivo, no toda la carga pendiente.
- [El BFF puede acumular logica de negocio] -> Mantener una matriz de responsabilidades, prohibir acceso a base de datos y probar que preserva decisiones y errores de la API.
- [La proteccion de vistas puede confundirse con seguridad de datos] -> Repetir autorizacion por rol y propiedad en la API y probar acceso directo no autorizado.
- [Tokens en cookies introducen CSRF] -> Aplicar SameSite, token de doble envio y pruebas de rechazo antes de agregar mutaciones sensibles.
- [Carrito local pierde sincronizacion entre dispositivos] -> Aceptarlo y comunicarlo como limite del MVP; la API siempre recalcula antes de crear el pedido.
- [Reservas de pedidos abandonados retienen stock] -> Permitir cancelacion administrativa; diseñar expiracion como cambio OpenSpec posterior si se necesita.
- [Offset pagination se degrada con muchos registros] -> Mantenerla por claridad en el MVP y migrar a cursor solo despues de medir.
- [TypeORM puede ocultar SQL] -> Revisar migraciones generadas, activar SQL de desarrollo de forma segura y practicar consultas y planes de ejecucion.
- [Pruebas con PostgreSQL son mas lentas] -> Separar unitarias rapidas de suites de integracion y ejecutar ambas en CI.
- [Codigos de pago simulados podrian parecer datos reales] -> Usar nombres imposibles de confundir con una tarjeta y rechazar cualquier otro formato.

## Migration Plan

Es un sistema nuevo; no hay datos que migrar ni usuarios que compatibilizar. La entrega se habilita por incrementos:

0. Inicializar Git, configurar `origin`, proteger secretos con `.gitignore` y publicar el plan OpenSpec en `main`.
1. Inicializar workspace, calidad automatica y PostgreSQL.
2. Entregar una ruta vertical de solo lectura producto PostgreSQL -> API -> BFF -> storefront.
3. Ampliar catalogo, filtros, detalle y responsive.
4. Incorporar carrito local.
5. Incorporar usuarios, sesion y permisos de vistas.
6. Incorporar checkout, pedidos e idempotencia.
7. Incorporar pago simulado y estados.
8. Incorporar panel administrativo, auditoria y concurrencia.
9. Endurecer seguridad, contratos, accesibilidad y pruebas end-to-end.

Cada incremento conserva migraciones reversibles y debe dejar todas las comprobaciones en verde. Si un incremento falla antes de compartir datos, se revierte codigo y migracion. Una migracion que ya contenga datos reales requerira una propuesta OpenSpec especifica de roll-forward; no se usara `synchronize` ni se borrara la base como solucion.
