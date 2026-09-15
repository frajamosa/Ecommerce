## Purpose

Permite que cualquier visitante encuentre y comprenda los productos disponibles mediante una experiencia rapida, accesible y adaptable a distintos tamanos de pantalla.

## ADDED Requirements

### Requirement: Catalogo publico paginado
El sistema SHALL mostrar solo productos activos mediante una lista paginada con nombre, imagen alternativa, precio, categoria y disponibilidad.

#### Scenario: Primera visita al catalogo
- **GIVEN** existen productos activos
- **WHEN** una persona abre el catalogo sin filtros
- **THEN** ve la primera pagina y controles que indican la pagina actual y permiten avanzar cuando corresponde

#### Scenario: Catalogo vacio
- **GIVEN** no existen productos activos
- **WHEN** una persona abre el catalogo
- **THEN** ve un estado vacio comprensible y no una pantalla de error

### Requirement: Busqueda y filtros combinables
El sistema SHALL permitir buscar por texto y filtrar por categoria, disponibilidad y rango de precio, conservando los criterios en la URL.

#### Scenario: Combinacion de criterios
- **GIVEN** el catalogo contiene productos de distintas categorias y precios
- **WHEN** la persona busca un texto y aplica categoria y rango de precio
- **THEN** recibe solo coincidencias que cumplen todos los criterios y vuelve a la primera pagina

#### Scenario: Criterios invalidos
- **WHEN** la URL contiene un rango de precio o pagina invalida
- **THEN** el sistema informa el problema de forma recuperable y no ejecuta una consulta insegura

### Requirement: Orden estable
El sistema SHALL permitir ordenar por relevancia o nombre y por precio ascendente o descendente, usando un segundo criterio estable para evitar elementos repetidos entre paginas.

#### Scenario: Cambio de orden
- **WHEN** la persona selecciona precio de menor a mayor
- **THEN** la lista y la URL reflejan el orden elegido de forma determinista

### Requirement: Detalle de producto
El sistema SHALL ofrecer una vista publica para cada producto activo con descripcion, precio, categoria y stock disponible para compra.

#### Scenario: Producto inactivo o inexistente
- **WHEN** una persona abre el detalle de un producto inexistente o inactivo
- **THEN** recibe una vista de no encontrado sin datos administrativos

### Requirement: Interfaz responsive y accesible
El catalogo y el detalle MUST poder utilizarse desde 320 px de ancho sin desplazamiento horizontal y mediante teclado, con foco visible, etiquetas, contraste y jerarquia semantica.

#### Scenario: Pantalla pequena
- **WHEN** el viewport mide 320 px de ancho
- **THEN** productos, filtros y acciones siguen siendo legibles y operables sin desplazamiento horizontal

#### Scenario: Navegacion por teclado
- **WHEN** una persona recorre busqueda, filtros, productos y paginacion usando solo teclado
- **THEN** el foco sigue un orden comprensible y siempre es visible

### Requirement: Estados de carga y fallo
El storefront SHALL distinguir carga, ausencia de resultados y fallo de red, y ofrecer reintento cuando la operacion sea segura.

#### Scenario: Fallo al cargar productos
- **WHEN** el BFF no puede responder al listado
- **THEN** se conserva la pagina utilizable y se muestra una accion para reintentar