## Pregunta 1 — Catálogo comercial activo

**Enunciado:** El equipo de ventas prepara la tarifa de la próxima campaña y necesita el catálogo depurado.

Obtén los productos que **no** están descatalogados y cuyo precio unitario esté entre 10 y 50 euros, ambos incluidos. Muestra el nombre del producto y su precio redondeado a dos decimales, ordenado de mayor a menor precio.
**Columnas esperadas:** `producto`, `precio`

**Técnicas:** `WHERE`, `BETWEEN`, `ROUND()`, alias de columna, `ORDER BY`

> **Pista:** la columna `discontinued` es de tipo `integer`, no booleana. Un producto activo tiene valor 0.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 2 — Concentración geográfica de la cartera

**Enunciado:** Dirección quiere saber en qué mercados está realmente concentrada la base de clientes antes de decidir dónde abrir delegación.

Cuenta cuántos clientes hay en cada país y muestra únicamente aquellos países con **5 o más clientes**, ordenados de mayor a menor. Indica también cuántas ciudades distintas hay en cada uno de esos países.

**Columnas esperadas:** `pais`, `num_clientes`, `num_ciudades`

**Técnicas:** `GROUP BY`, `COUNT()`, `COUNT(DISTINCT ...)`, `HAVING`

> **Pista:** `HAVING` filtra después de agrupar; `WHERE` filtra antes. Aquí la condición se aplica sobre el resultado de un conteo, así que solo una de las dos cláusulas sirve.
>
**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 3 — Alerta de reposición

**Enunciado:** Logística necesita detectar qué referencias están en riesgo de rotura de stock.

Localiza los productos activos cuyas unidades en stock sean **inferiores o iguales** a su nivel de reposición. Muestra el nombre, las unidades en stock, el nivel de reposición, las unidades ya pedidas al proveedor y una columna de texto que indique `'CRÍTICO'` cuando el stock sea 0 y `'AVISO'` en el resto de casos.

**Columnas esperadas:** `producto`, `stock`, `nivel_reposicion`, `pedido_a_proveedor`, `situacion`

**Técnicas:** `WHERE` con comparación entre columnas, `CASE WHEN`

> **Pista:** comprueba antes si alguna de estas columnas admite nulos. Un `NULL` en una comparación no devuelve ni verdadero ni falso, y la fila desaparece del resultado sin avisarte.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 4 — Ficha completa de producto

**Enunciado:** Marketing va a rehacer el catálogo impreso y necesita cada producto con su categoría y los datos de contacto de quien lo suministra.

Para los productos suministrados por empresas de **Italia, Francia o España**, muestra el nombre del producto, el nombre de la categoría, el nombre del proveedor, su país y su ciudad. Ordena por país y, dentro de cada país, por nombre de producto.

**Columnas esperadas:** `producto`, `categoria`, `proveedor`, `pais`, `ciudad`

**Técnicas:** `INNER JOIN` de tres tablas, alias de tabla, `WHERE ... IN`

> **Pista:** `products` no se une directamente con nada geográfico. Mira el diagrama: la información de país está en `suppliers`.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 5 — Detalle valorizado de un pedido

**Enunciado:** Atención al cliente recibe una reclamación sobre el pedido **10248** y necesita reconstruir la factura línea a línea.

Muestra, para ese pedido, el nombre del producto, el precio unitario aplicado, la cantidad, el descuento y el importe final de cada línea. Añade el nombre del cliente y la fecha del pedido.

**Columnas esperadas:** `cliente`, `fecha_pedido`, `producto`, `precio_unitario`, `cantidad`, `descuento`, `importe_linea`

**Técnicas:** `INNER JOIN` con `USING`, aritmética entre columnas, `ROUND()`

> **Pista:** `orders` y `order_details` comparten el nombre de columna `order_id`; `order_details` y `products` comparten `product_id`. Cuando los nombres coinciden a ambos lados, `USING(columna)` es más limpio que `ON a.col = b.col` y además evita que la columna aparezca duplicada en el resultado.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 6 — Ranking de categorías por facturación

**Enunciado:** Comité de dirección: ¿qué familias de producto sostienen realmente el negocio?

Calcula la facturación total de cada categoría durante toda la historia de la compañía. Muestra el nombre de la categoría, el número de líneas de pedido que ha generado, el número de productos distintos vendidos y la facturación total. Incluye únicamente las categorías que superen los **100.000 euros** de facturación, ordenadas de mayor a menor.

**Columnas esperadas:** `categoria`, `num_lineas`, `num_productos`, `facturacion`

**Técnicas:** `INNER JOIN` de tres tablas, `GROUP BY`, `SUM()`, `COUNT(DISTINCT ...)`, `HAVING`, `ROUND()`

> **Pista:** el `HAVING` se aplica sobre la expresión agregada completa, no sobre el alias. En PostgreSQL puedes repetir la expresión o envolver la consulta.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 7 — Clientes sin actividad comercial

**Enunciado:** Dirección comercial sospecha que hay cuentas abiertas que nunca han llegado a comprar.

Lista **todos** los clientes con el número de pedidos que ha realizado cada uno y la fecha de su último pedido. Los clientes sin ningún pedido deben aparecer igualmente, con un 0 en el conteo y el texto `'SIN PEDIDOS'` en lugar de la fecha. Ordena de forma que los clientes inactivos aparezcan primero.

**Columnas esperadas:** `cliente`, `pais`, `num_pedidos`, `ultimo_pedido`

**Técnicas:** `LEFT JOIN`, `COUNT()` sobre columna de la tabla derecha, `COALESCE()`, `MAX()`

> **Pista:** `COUNT(*)` cuenta filas, incluidas las que el `LEFT JOIN` rellenó con nulos, y te dará 1 para los clientes sin pedidos. `COUNT(columna)` ignora los nulos. Esa diferencia es exactamente el objetivo del ejercicio.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 8 — Organigrama de la fuerza de ventas

**Enunciado:** Recursos Humanos necesita el organigrama del departamento comercial en formato tabla.

Muestra cada empleado con su nombre completo, su cargo, el nombre completo de la persona a la que reporta y el cargo de esa persona. El empleado que no reporta a nadie debe aparecer también, con el texto `'DIRECCIÓN GENERAL'` en el campo del responsable.

**Columnas esperadas:** `empleado`, `cargo`, `responsable`, `cargo_responsable`

**Técnicas:** `SELF JOIN` con `LEFT JOIN`, alias de tabla obligatorios, concatenación de texto, `COALESCE()`

> **Pista:** la misma tabla aparece dos veces en el `FROM`, así que los alias dejan de ser una comodidad y pasan a ser imprescindibles. Piensa en `emp` y `jefe` como si fueran dos tablas distintas.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 9 — Rejilla de cobertura categoría × año

**Enunciado:** Control de gestión quiere una rejilla completa de facturación por categoría y año, **sin huecos**: si una categoría no vendió nada en un año concreto, debe aparecer con un 0, no desaparecer de la tabla.

Genera todas las combinaciones posibles de las 8 categorías con los 3 años del histórico (24 filas) y asocia a cada combinación su facturación. Ordena por categoría y año.

**Columnas esperadas:** `categoria`, `anio`, `facturacion`

**Técnicas:** `CROSS JOIN` para generar la rejilla, `LEFT JOIN` contra los datos reales, `COALESCE()`, `EXTRACT()`

> **Pista:** este es el patrón clásico para informes con huecos. Primero construyes el "esqueleto" de todas las combinaciones posibles con un `CROSS JOIN`, y solo después cuelgas los datos reales con un `LEFT JOIN`. Si lo haces al revés, las combinaciones sin datos nunca aparecerán.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 10 — Mapa de países: clientes frente a proveedores

**Enunciado:** Expansión internacional quiere una única tabla que muestre, para cada país en el que la compañía tiene presencia, cuántos clientes y cuántos proveedores hay. Deben aparecer los países que solo tienen clientes, los que solo tienen proveedores y los que tienen ambos.

**Columnas esperadas:** `pais`, `num_clientes`, `num_proveedores`, `tipo_presencia`

Donde `tipo_presencia` toma los valores `'SOLO CLIENTES'`, `'SOLO PROVEEDORES'` o `'AMBOS'`.

**Técnicas:** `FULL JOIN` entre dos subconsultas agregadas, `COALESCE()`, `CASE WHEN`

> **Pista:** en un `FULL JOIN` la columna de unión puede venir nula por cualquiera de los dos lados. Si escribes `SELECT a.country`, perderás el nombre de los países que solo existen en la tabla `b`.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 11 — Directorio unificado de contactos

**Enunciado:** Sistemas va a migrar el CRM y necesita una exportación única con todos los contactos de la compañía, vengan de donde vengan.

Construye una sola tabla que reúna los contactos de clientes, los de proveedores y los empleados. Cada fila debe indicar el origen (`'CLIENTE'`, `'PROVEEDOR'`, `'EMPLEADO'`), el nombre de la persona de contacto **en mayúsculas**, la organización a la que pertenece, la ciudad y el país. Para los empleados, la organización es el literal `'NORTHWIND TRADERS'` y el nombre de contacto se forma concatenando nombre y apellidos.

Ordena por origen y luego por país.

**Columnas esperadas:** `origen`, `contacto`, `organizacion`, `ciudad`, `pais`

**Técnicas:** `UNION ALL`, `UPPER()`, concatenación con `||` o `CONCAT()`, literales como columna

> **Pista:** las tres consultas deben devolver el mismo número de columnas, en el mismo orden y con tipos compatibles. Razona por qué aquí conviene `UNION ALL` y no `UNION`: ¿qué pasaría si un cliente y un proveedor compartieran nombre de contacto y ciudad?
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 12 — Mercados con desequilibrio

**Enunciado:** Compras y Ventas mantienen una discusión recurrente: ¿en qué países vendemos sin tener proveedor local, y en cuáles coincidimos?

Resuelve las dos preguntas en dos consultas independientes:

**a)** Países donde hay clientes pero **ningún** proveedor.
**b)** Países donde hay **a la vez** clientes y proveedores.

Ordena ambos resultados alfabéticamente.

**Columnas esperadas:** `pais`

**Técnicas:** `EXCEPT`, `INTERSECT`

> **Pista:** los operadores de conjunto eliminan duplicados automáticamente, a diferencia de `UNION ALL`. Compara el resultado del apartado (a) con el que obtendrías usando un `LEFT JOIN ... WHERE ... IS NULL`: llegan al mismo sitio por caminos distintos, y conviene que sepas escribir los dos.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 13 — Clientes que nunca han comprado pescado

**Enunciado:** El responsable de la categoría Seafood quiere una lista de cuentas sobre las que hacer campaña de captación.

Localiza los clientes que **nunca** han incluido un producto de la categoría `'Seafood'` en ninguno de sus pedidos. Muestra el nombre del cliente, su país y el número total de pedidos que sí ha realizado, de mayor a menor.

**Columnas esperadas:** `cliente`, `pais`, `pedidos_realizados`

**Técnicas:** anti join con `NOT EXISTS`, subconsulta correlacionada, `INNER JOIN` en la subconsulta

> **Pista:** hay tres formas de escribir un anti join: `NOT EXISTS`, `NOT IN` y `LEFT JOIN ... WHERE ... IS NULL`. Escribe la versión con `NOT EXISTS` y después prueba con `NOT IN`. Si la subconsulta de `NOT IN` puede devolver algún `NULL`, el resultado será una tabla vacía sin ningún mensaje de error. Es uno de los fallos más difíciles de detectar en SQL.
> 

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 14 — Productos por encima de la media

**Enunciado:** El comité de precios quiere identificar el segmento premium del catálogo.

Muestra los productos activos cuyo precio unitario supere el precio medio de **todo** el catálogo. Incluye en cada fila el precio del producto, el precio medio general y la diferencia entre ambos, todo redondeado a dos decimales. Ordena por diferencia descendente.

**Columnas esperadas:** `producto`, `precio`, `precio_medio_catalogo`, `diferencia`

**Técnicas:** subconsulta escalar en `WHERE`, subconsulta escalar en `SELECT`, aritmética

> **Pista:** una subconsulta escalar es aquella que devuelve exactamente una fila y una columna, y por eso se puede usar donde iría un valor. Fíjate en que la misma subconsulta aparece en dos sitios; más adelante verás cómo evitar esa repetición con un CTE.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 15 — Ticket medio por cliente

**Enunciado:** Dirección comercial quiere segmentar la cartera por valor medio de pedido, no por volumen total.

Calcula, para cada cliente que haya comprado alguna vez, el número de pedidos, el importe total acumulado y el importe medio por pedido. Muestra los 15 clientes con mayor ticket medio.

El cálculo tiene dos niveles: primero hay que obtener el importe de cada pedido sumando sus líneas, y solo después promediar esos importes por cliente. **Promediar directamente las líneas daría un resultado distinto y equivocado.**

**Columnas esperadas:** `cliente`, `pais`, `num_pedidos`, `importe_total`, `ticket_medio`

**Técnicas:** subconsulta en `FROM` (tabla derivada), agregación en dos niveles, `LIMIT`

> **Pista:** toda subconsulta en `FROM` necesita un alias en PostgreSQL, aunque no lo uses. Si lo olvidas, el error que verás es `subquery in FROM must have an alias`.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 16 — El producto más caro de cada categoría

**Enunciado:** El equipo de compras quiere revisar el posicionamiento de precio en cada familia.

Para cada categoría, muestra el producto con el precio unitario más alto. Incluye el nombre de la categoría, el nombre del producto, su precio y el precio medio de su categoría.

Resuélvelo con una **subconsulta correlacionada**: para cada producto, comprueba si su precio coincide con el máximo de su propia categoría.

**Columnas esperadas:** `categoria`, `producto`, `precio`, `precio_medio_categoria`

**Técnicas:** subconsulta correlacionada en `WHERE`, subconsulta correlacionada en `SELECT`, `INNER JOIN`

> **Pista:** una subconsulta correlacionada se ejecuta conceptualmente una vez por cada fila de la consulta externa, porque hace referencia a una columna de esa fila. Eso la hace potente pero costosa. Cuando termines, plantéate cuál sería el coste sobre una tabla de diez millones de filas.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 17 — Segmentación ABC de la cartera de clientes

**Enunciado:** Dirección quiere clasificar a los clientes en tres tramos de valor para asignar recursos comerciales.

Usando expresiones de tabla común (CTE), construye una consulta que:

1. Calcule la facturación total de cada cliente.
2. Divida los clientes en **cuartiles** según esa facturación.
3. Asigne una etiqueta de segmento: `'A - Estratégico'` al cuartil superior, `'B - Consolidado'` al segundo, `'C - Ocasional'` al tercero y `'D - Marginal'` al cuarto.
4. Devuelva, por segmento, el número de clientes, la facturación total del segmento y el porcentaje que representa sobre el total de la compañía.

**Columnas esperadas:** `segmento`, `num_clientes`, `facturacion_segmento`, `porcentaje_sobre_total`

**Técnicas:** `WITH` con varias CTE encadenadas, `NTILE()`, `CASE WHEN`, agregación sobre el resultado de una CTE, cálculo de porcentaje

> **Pista:** encadenar CTE permite leer la consulta de arriba abajo como una receta, en lugar de descifrarla de dentro hacia fuera como ocurre con las subconsultas anidadas. Una CTE puede referirse a las declaradas antes que ella.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 18 — Los tres productos más vendidos de cada categoría

**Enunciado:** El equipo de categoría necesita el podio de cada familia para negociar con proveedores.

Para cada categoría, obtén los **tres productos con mayor facturación**. Muestra la categoría, la posición dentro de la categoría, el nombre del producto, las unidades vendidas y la facturación.

Incluye además una columna con la posición global del producto en el conjunto de la compañía, para que se vea qué productos son líderes de su nicho pero irrelevantes en el total.

**Columnas esperadas:** `categoria`, `posicion_en_categoria`, `producto`, `unidades`, `facturacion`, `posicion_global`

**Técnicas:** `RANK()` o `ROW_NUMBER()` con `OVER (PARTITION BY ... ORDER BY ...)`, CTE para poder filtrar por la posición, función de ventana sin `PARTITION BY`

> **Pista:** no se puede filtrar por una función de ventana en el `WHERE`, porque las funciones de ventana se evalúan después del filtrado. Necesitas calcularla en una CTE o subconsulta y filtrar fuera.
> 
> 
> Piensa también qué ocurriría con `RANK()` frente a `DENSE_RANK()` frente a `ROW_NUMBER()` si dos productos empatasen exactamente en facturación.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 19 — Evolución mensual con acumulado y media móvil

**Enunciado:** Control de gestión prepara el cuadro de mando de la evolución del negocio durante 1997.

Para cada mes de 1997, calcula:

- La facturación del mes.
- El total acumulado desde enero.
- La media móvil de los tres últimos meses (el mes actual y los dos anteriores).
- La facturación del mes anterior.
- La variación porcentual respecto al mes anterior.

**Columnas esperadas:** `mes`, `facturacion`, `acumulado`, `media_movil_3m`, `mes_anterior`, `variacion_pct`

**Técnicas:** `DATE_TRUNC()`, `SUM() OVER (ORDER BY ...)` como total acumulado, definición explícita de marco con `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`, `LAG()`, CTE

> **Pista:** cuando una función de ventana agregada lleva `ORDER BY` pero no especificas marco, PostgreSQL aplica por defecto `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, que es justo lo que quieres para el acumulado. Para la media móvil ese comportamiento por defecto no sirve: ahí tienes que declarar el marco tú.
> 
> 
> La primera fila no tiene mes anterior. Decide qué mostrar en ese caso.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...

## Pregunta 20 — Cuadro de mando anual por categoría

**Enunciado:** Última petición, y la más ambiciosa: el informe anual que se presenta al consejo.

Construye una tabla donde cada fila sea una categoría y las columnas muestren la facturación de 1996, 1997 y 1998 en columnas separadas, más el total de los tres años. Añade al final una fila de totales generales.

Incluye además una columna que indique el peso de cada categoría sobre la facturación total de la compañía, y otra que muestre si la categoría creció o decreció entre 1997 y 1998.

**Columnas esperadas:** `categoria`, `f_1996`, `f_1997`, `f_1998`, `total`, `peso_pct`, `tendencia`

**Técnicas:** pivotado manual con `CASE WHEN` dentro de `SUM()` (o `FILTER`), `ROLLUP` para la fila de totales, `COALESCE()`, `CASE WHEN` para la tendencia, funciones de ventana para el peso

> **Pista:** el pivotado en SQL estándar consiste en convertir filas en columnas mediante una función de agregación que solo suma cuando se cumple una condición. PostgreSQL ofrece dos sintaxis equivalentes: `SUM(CASE WHEN anio = 1997 THEN importe ELSE 0 END)` y la más moderna `SUM(importe) FILTER (WHERE anio = 1997)`. Escribe la versión con `FILTER`, que es específica de PostgreSQL y mucho más legible.
> 
> 
> Ten en cuenta que 1996 solo tiene medio año de datos (desde julio) y 1998 llega solo hasta mayo. La tendencia entre 1997 y 1998 no es comparable sin normalizar. Menciónalo en un comentario dentro de tu consulta: detectar que una comparación no es válida vale más que calcularla bien.
>

**Consulta:**

```sql

```

**Resultado:**

!Resultado pregunta 7

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...