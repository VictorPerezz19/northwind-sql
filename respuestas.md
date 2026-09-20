## Pregunta 1 — Catálogo comercial activo

**Enunciado:** El equipo de ventas prepara la tarifa de la próxima campaña y necesita el catálogo depurado.

Obtén los productos que **no** están descatalogados y cuyo precio unitario esté entre 10 y 50 euros, ambos incluidos. Muestra el nombre del producto y su precio redondeado a dos decimales, ordenado de mayor a menor precio.
**Columnas esperadas:** `producto`, `precio`

**Técnicas:** `WHERE`, `BETWEEN`, `ROUND()`, alias de columna, `ORDER BY`

> **Pista:** la columna `discontinued` es de tipo `integer`, no booleana. Un producto activo tiene valor 0.
>

**Consulta:**

```sql
SELECT
	PRODUCT_NAME AS PRODUCTO,
	ROUND(UNIT_PRICE::NUMERIC, 2) AS PRECIO
FROM
	PRODUCTS
WHERE
	DISCONTINUED = 0
	AND UNIT_PRICE BETWEEN 10 AND 50
ORDER BY
	PRECIO DESC;
```

**Resultado:**

![imagen resultado](img\pregunta-1.png)

**Comentario:** He usado ROUND(::numeric, 2) para redondear a dos decimales y cambiar el tipo y discontinued = 0 para ver si esta descatalogado. Para el rango de precios usé BETWEEN y ORDER BY DESC para ordenarlo de mayor a menor.

## Pregunta 2 — Concentración geográfica de la cartera

**Enunciado:** Dirección quiere saber en qué mercados está realmente concentrada la base de clientes antes de decidir dónde abrir delegación.

Cuenta cuántos clientes hay en cada país y muestra únicamente aquellos países con **5 o más clientes**, ordenados de mayor a menor. Indica también cuántas ciudades distintas hay en cada uno de esos países.

**Columnas esperadas:** `pais`, `num_clientes`, `num_ciudades`

**Técnicas:** `GROUP BY`, `COUNT()`, `COUNT(DISTINCT ...)`, `HAVING`

> **Pista:** `HAVING` filtra después de agrupar; `WHERE` filtra antes. Aquí la condición se aplica sobre el resultado de un conteo, así que solo una de las dos cláusulas sirve.
>
**Consulta:**

```sql
SELECT
	COUNTRY AS PAIS,
	COUNT(CUSTOMER_ID) AS NUM_CLIENTES,
	COUNT(CITY) AS NUM_CIUDADES
FROM
	CUSTOMERS
GROUP BY
	COUNTRY
HAVING
	COUNT(CUSTOMER_ID) >= 5
ORDER BY
	NUM_CLIENTES DESC;
```

**Resultado:**

![alt text](img\pregunta-2.png)

**Comentario:** He agrupado por país con GROUP BY y contado los clientes y ciudades con COUNT. Por último, usé HAVING, ya que use GROUP BY para que despues de agruparlo por paises mostrara solo los que tengan mas de 5 client, ademas use ORDER BY DESC para ordenarlo de mayor a menor.

## Pregunta 3 — Alerta de reposición

**Enunciado:** Logística necesita detectar qué referencias están en riesgo de rotura de stock.

Localiza los productos activos cuyas unidades en stock sean **inferiores o iguales** a su nivel de reposición. Muestra el nombre, las unidades en stock, el nivel de reposición, las unidades ya pedidas al proveedor y una columna de texto que indique `'CRÍTICO'` cuando el stock sea 0 y `'AVISO'` en el resto de casos.

**Columnas esperadas:** `producto`, `stock`, `nivel_reposicion`, `pedido_a_proveedor`, `situacion`

**Técnicas:** `WHERE` con comparación entre columnas, `CASE WHEN`

> **Pista:** comprueba antes si alguna de estas columnas admite nulos. Un `NULL` en una comparación no devuelve ni verdadero ni falso, y la fila desaparece del resultado sin avisarte.
>

**Consulta:**

```sql
SELECT
	PRODUCT_NAME AS PRODUCTO,
	UNITS_IN_STOCK AS STOCK,
	REORDER_LEVEL AS NIVEL_REPOSICION,
	UNITS_ON_ORDER AS PEDIDO_A_PROVEEDOR,
	CASE
		WHEN UNITS_IN_STOCK = 0 THEN 'CRÍTICO'
		ELSE 'AVISO'
	END AS SITUACION
FROM
	PRODUCTS
WHERE
	DISCONTINUED = 0
	AND UNITS_IN_STOCK <= REORDER_LEVEL;
```

**Resultado:**

![alt text](img\pregunta-3.png)

**Comentario:** He usado discontinued = 0 para ver solo los activos y he comparado units_in_stock <= reorder_level para sacar los que tienen poco stock. Por último, añadí un CASE WHEN para ponerle la etiqueta 'CRÍTICO' a los que están a 0 y 'AVISO' a los demás.

## Pregunta 4 — Ficha completa de producto

**Enunciado:** Marketing va a rehacer el catálogo impreso y necesita cada producto con su categoría y los datos de contacto de quien lo suministra.

Para los productos suministrados por empresas de **Italia, Francia o España**, muestra el nombre del producto, el nombre de la categoría, el nombre del proveedor, su país y su ciudad. Ordena por país y, dentro de cada país, por nombre de producto.

**Columnas esperadas:** `producto`, `categoria`, `proveedor`, `pais`, `ciudad`

**Técnicas:** `INNER JOIN` de tres tablas, alias de tabla, `WHERE ... IN`

> **Pista:** `products` no se une directamente con nada geográfico. Mira el diagrama: la información de país está en `suppliers`.
>

**Consulta:**

```sql
SELECT
	P.PRODUCT_NAME AS PRODUCTO,
	C.CATEGORY_NAME AS CATEGORIA,
	S.COMPANY_NAME AS PROVEEDOR,
	S.COUNTRY AS PAIS,
	S.CITY AS CIUDAD
FROM
	PRODUCTS P
	INNER JOIN CATEGORIES C ON P.CATEGORY_ID = C.CATEGORY_ID
	INNER JOIN SUPPLIERS S ON P.SUPPLIER_ID = S.SUPPLIER_ID
WHERE
	S.COUNTRY IN ('Italy', 'France', 'Spain')
ORDER BY
	PAIS,
	PRODUCTO;
```

**Resultado:**

![alt text](img\pregunta-4.png)

**Comentario:** He usado INNER JOIN para unir tres tablas: products como tabla central, conectándola con categories (para sacar el nombre de la categoría) y con suppliers (para sacar los datos del proveedor y el país). Para filtrar los países de golpe usé WHERE, IN, y finalmente ordené por país y luego por producto con ORDER BY.

## Pregunta 5 — Detalle valorizado de un pedido

**Enunciado:** Atención al cliente recibe una reclamación sobre el pedido **10248** y necesita reconstruir la factura línea a línea.

Muestra, para ese pedido, el nombre del producto, el precio unitario aplicado, la cantidad, el descuento y el importe final de cada línea. Añade el nombre del cliente y la fecha del pedido.

**Columnas esperadas:** `cliente`, `fecha_pedido`, `producto`, `precio_unitario`, `cantidad`, `descuento`, `importe_linea`

**Técnicas:** `INNER JOIN` con `USING`, aritmética entre columnas, `ROUND()`

> **Pista:** `orders` y `order_details` comparten el nombre de columna `order_id`; `order_details` y `products` comparten `product_id`. Cuando los nombres coinciden a ambos lados, `USING(columna)` es más limpio que `ON a.col = b.col` y además evita que la columna aparezca duplicada en el resultado.
>

**Consulta:**

```sql
SELECT
	C.COMPANY_NAME AS CLIENTE,
	O.ORDER_DATE AS FECHA_PEDIDO,
	P.PRODUCT_NAME AS PRODUCTO,
	ROUND(OD.UNIT_PRICE::NUMERIC, 2) AS PRECIO_UNITARIO,
	OD.QUANTITY AS CANTIDAD,
	OD.DISCOUNT AS DESCUENTO,
	ROUND(
		(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT))::NUMERIC,
		2
	) AS IMPORTE_LINEA
FROM
	ORDERS O
	INNER JOIN CUSTOMERS C USING (CUSTOMER_ID)
	INNER JOIN ORDER_DETAILS OD USING (ORDER_ID)
	INNER JOIN PRODUCTS P USING (PRODUCT_ID)
WHERE
	ORDER_ID = 10248;
```

**Resultado:**

![alt text](img\pregunta-5.png)

**Comentario:** He unido 4 tablas usando INNER JOIN y USING para no usar el ON ya que las claves se llaman igual en ambas tablas. Para evitar errores, le he puesto el alias od a unit_price, porque esa columna existe tanto en productos como en detalles del pedido y SQL necesita saber cuál coger. El importe de la línea lo saqué multiplicando el precio por la cantidad y restándole el descuento (1 - descuento), redondeando el resultado final a dos decimales, utilice, a parte de esto he especificado od.unit_price en lugar de poner solo unit_price. Esto es vital porque la base de datos guarda el precio actual en la tabla products y el precio histórico en la tabla order_details. Si indico de qué tabla sacar el precio od me sale un error de duplicado de tablas, además, utilice od ya que se refiere a un precio de hace tiempo, no acctual sino habría utilizado p (el uso de numeric y de round lo explico en los ejercicios anteriores).

## Pregunta 6 — Ranking de categorías por facturación

**Enunciado:** Comité de dirección: ¿qué familias de producto sostienen realmente el negocio?

Calcula la facturación total de cada categoría durante toda la historia de la compañía. Muestra el nombre de la categoría, el número de líneas de pedido que ha generado, el número de productos distintos vendidos y la facturación total. Incluye únicamente las categorías que superen los **100.000 euros** de facturación, ordenadas de mayor a menor.

**Columnas esperadas:** `categoria`, `num_lineas`, `num_productos`, `facturacion`

**Técnicas:** `INNER JOIN` de tres tablas, `GROUP BY`, `SUM()`, `COUNT(DISTINCT ...)`, `HAVING`, `ROUND()`

> **Pista:** el `HAVING` se aplica sobre la expresión agregada completa, no sobre el alias. En PostgreSQL puedes repetir la expresión o envolver la consulta.
>

**Consulta:**

```sql
SELECT
	C.CATEGORY_NAME AS CATEGORIA,
	COUNT(OD.ORDER_ID) AS NUM_LINEAS,
	COUNT(DISTINCT OD.PRODUCT_ID) AS NUM_PRODUCTOS,
	ROUND(
		SUM(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT))::NUMERIC,
		2
	) AS FACTURACION
FROM
	CATEGORIES C
	INNER JOIN PRODUCTS P USING (CATEGORY_ID)
	INNER JOIN ORDER_DETAILS OD USING (PRODUCT_ID)
GROUP BY
	C.CATEGORY_NAME
HAVING
	SUM(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT)) > 100000
ORDER BY
	FACTURACION DESC;
```

**Resultado:**

![alt text](img\pregunta-6.png)

**Comentario:** He unido las tablas de categorías, productos y detalles de pedidos usando USING. Agrupé por categoría para poder calcular la facturación con SUM, y usé COUNT(DISTINCT od.product_id) para contar solo los productos diferentes vendidos, sin repetir. Para filtrar las que superan los 100.000€, tuve que copiar lo mismo en el HAVING. Por ultimo hice un ORDER BY de mayor a menor facturación.  

## Pregunta 7 — Clientes sin actividad comercial

**Enunciado:** Dirección comercial sospecha que hay cuentas abiertas que nunca han llegado a comprar.

Lista **todos** los clientes con el número de pedidos que ha realizado cada uno y la fecha de su último pedido. Los clientes sin ningún pedido deben aparecer igualmente, con un 0 en el conteo y el texto `'SIN PEDIDOS'` en lugar de la fecha. Ordena de forma que los clientes inactivos aparezcan primero.

**Columnas esperadas:** `cliente`, `pais`, `num_pedidos`, `ultimo_pedido`

**Técnicas:** `LEFT JOIN`, `COUNT()` sobre columna de la tabla derecha, `COALESCE()`, `MAX()`

> **Pista:** `COUNT(*)` cuenta filas, incluidas las que el `LEFT JOIN` rellenó con nulos, y te dará 1 para los clientes sin pedidos. `COUNT(columna)` ignora los nulos. Esa diferencia es exactamente el objetivo del ejercicio.
>

**Consulta:**

```sql
SELECT
	C.COMPANY_NAME AS CLIENTE,
	C.COUNTRY AS PAIS,
	COUNT(O.ORDER_ID) AS NUM_PEDIDOS,
	COALESCE(MAX(O.ORDER_DATE)::TEXT, 'SIN PEDIDOS') AS ULTIMO_PEDIDO
FROM
	CUSTOMERS C
	LEFT JOIN ORDERS O USING (CUSTOMER_ID)
GROUP BY
	C.COMPANY_NAME,
	C.COUNTRY
ORDER BY
	NUM_PEDIDOS ASC;
```

**Resultado:**

![alt text](img\pregunta-7.png)

**Comentario:** He usado LEFT JOIN para traerme los clientes, hayan comprado o no. Aunque esto genera valores null, utilizando COUNT(o.order_id) hago que los nulos se conviertan en un 0. Para sacar el último pedido usé MAX(o.order_date) y le añadí ::text para convertir esa fecha en texto; así, COALESCE me deja ocultar el nulo sobrescribiéndolo con la frase 'SIN PEDIDOS' sin dar error de tipos. Por último, usé ORDER BY num_pedidos ASC para que los de 0 salgan arriba del todo, esto se puede hacer tambien con CASE WHEN pero con COALESCE es mas rapido.

## Pregunta 8 — Organigrama de la fuerza de ventas

**Enunciado:** Recursos Humanos necesita el organigrama del departamento comercial en formato tabla.

Muestra cada empleado con su nombre completo, su cargo, el nombre completo de la persona a la que reporta y el cargo de esa persona. El empleado que no reporta a nadie debe aparecer también, con el texto `'DIRECCIÓN GENERAL'` en el campo del responsable.

**Columnas esperadas:** `empleado`, `cargo`, `responsable`, `cargo_responsable`

**Técnicas:** `SELF JOIN` con `LEFT JOIN`, alias de tabla obligatorios, concatenación de texto, `COALESCE()`

> **Pista:** la misma tabla aparece dos veces en el `FROM`, así que los alias dejan de ser una comodidad y pasan a ser imprescindibles. Piensa en `emp` y `jefe` como si fueran dos tablas distintas.
>

**Consulta:**

```sql
SELECT
	E.FIRST_NAME || ' ' || E.LAST_NAME AS EMPLEADO,
	E.TITLE AS CARGO,
	COALESCE(
		J.FIRST_NAME || ' ' || J.LAST_NAME,
		'DIRECCIÓN GENERAL'
	) AS RESPONSABLE,
	COALESCE(J.TITLE, 'DIRECCIÓN GENERAL') AS CARGO_RESPONSABLE
FROM
	EMPLOYEES E
	LEFT JOIN EMPLOYEES J ON E.REPORTS_TO = J.EMPLOYEE_ID;
```

**Resultado:**

![alt text](img\pregunta-8.png)

**Comentario:** Para esta consulta he hecho un SELF JOIN, crucé la tabla employees consigo misma, las he unido igualando e.reports_to con j.employee_id. He usado LEFT JOIN para que el jefe supremo no desaparezca al no tener a nadie por encima. Para juntar el nombre y el apellido he usado el operador ||, y he envuelto los datos del jefe en un COALESCE para que, cuando devuelva nulo, escriba 'DIRECCIÓN GENERAL'.

## Pregunta 9 — Rejilla de cobertura categoría × año

**Enunciado:** Control de gestión quiere una rejilla completa de facturación por categoría y año, **sin huecos**: si una categoría no vendió nada en un año concreto, debe aparecer con un 0, no desaparecer de la tabla.

Genera todas las combinaciones posibles de las 8 categorías con los 3 años del histórico (24 filas) y asocia a cada combinación su facturación. Ordena por categoría y año.

**Columnas esperadas:** `categoria`, `anio`, `facturacion`

**Técnicas:** `CROSS JOIN` para generar la rejilla, `LEFT JOIN` contra los datos reales, `COALESCE()`, `EXTRACT()`

> **Pista:** este es el patrón clásico para informes con huecos. Primero construyes el "esqueleto" de todas las combinaciones posibles con un `CROSS JOIN`, y solo después cuelgas los datos reales con un `LEFT JOIN`. Si lo haces al revés, las combinaciones sin datos nunca aparecerán.
>

**Consulta:**

```sql
SELECT
	C.CATEGORY_NAME AS CATEGORIA,
	Y.ANIO,
	COALESCE(VENTAS.FACTURACION, 0) AS FACTURACION
FROM
	CATEGORIES C
	CROSS JOIN (
		SELECT DISTINCT
			EXTRACT(
				YEAR
				FROM
					ORDER_DATE
			) AS ANIO
		FROM
			ORDERS
	) Y
	LEFT JOIN (
		SELECT
			P.CATEGORY_ID,
			EXTRACT(
				YEAR
				FROM
					O.ORDER_DATE
			) AS ANIO,
			ROUND(
				SUM(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT))::NUMERIC,
				2
			) AS FACTURACION
		FROM
			ORDERS O
			INNER JOIN ORDER_DETAILS OD USING (ORDER_ID)
			INNER JOIN PRODUCTS P USING (PRODUCT_ID)
		GROUP BY
			P.CATEGORY_ID,
			EXTRACT(
				YEAR
				FROM
					O.ORDER_DATE
			)
	) VENTAS ON C.CATEGORY_ID = VENTAS.CATEGORY_ID
	AND Y.ANIO = VENTAS.ANIO
ORDER BY
	CATEGORIA,
	ANIO;
```

**Resultado:**

![alt text](img\pregunta-9.png)

**Comentario:** Primero, he creado usado CROSS JOIN en la tabla de categorías y consulta que saca los 3 años distintos usando EXTRACT(YEAR FROM) Esto genera 8 categorías × 3 años. Después, he calculado la facturación haciendo una subconsulta llamada ventas. Finalmente luego he usado el LEFT JOIN. Al hacerlo en este orden, las combinaciones que no tuvieron ventas se quedan con la celda vacía (NULL), por eso hago el COALESCE para que no desaparezcan los que tengan null.

## Pregunta 10 — Mapa de países: clientes frente a proveedores

**Enunciado:** Expansión internacional quiere una única tabla que muestre, para cada país en el que la compañía tiene presencia, cuántos clientes y cuántos proveedores hay. Deben aparecer los países que solo tienen clientes, los que solo tienen proveedores y los que tienen ambos.

**Columnas esperadas:** `pais`, `num_clientes`, `num_proveedores`, `tipo_presencia`

Donde `tipo_presencia` toma los valores `'SOLO CLIENTES'`, `'SOLO PROVEEDORES'` o `'AMBOS'`.

**Técnicas:** `FULL JOIN` entre dos subconsultas agregadas, `COALESCE()`, `CASE WHEN`

> **Pista:** en un `FULL JOIN` la columna de unión puede venir nula por cualquiera de los dos lados. Si escribes `SELECT a.country`, perderás el nombre de los países que solo existen en la tabla `b`.
>

**Consulta:**

```sql
SELECT 
    COALESCE(c.country, p.country) AS pais,
    COALESCE(c.num_clientes, 0) AS num_clientes,
    COALESCE(p.num_proveedores, 0) AS num_proveedores,
    CASE 
        WHEN c.country IS NULL THEN 'SOLO PROVEEDORES'
        WHEN p.country IS NULL THEN 'SOLO CLIENTES'
        ELSE 'AMBOS'
    END AS tipo_presencia
FROM 
    (SELECT country, COUNT(customer_id) AS num_clientes 
     FROM customers 
     GROUP BY country) c
FULL JOIN 
    (SELECT country, COUNT(supplier_id) AS num_proveedores 
     FROM suppliers 
     GROUP BY country) p 
ON c.country = p.country
ORDER BY 
    pais;
```

**Resultado:**

![alt text](img\pregunta-10.png)

**Comentario:** Hicé un FULL JOIN para conservar todos los países. use COALESCE(c.country, p.country) para coger el nombre del país sin importar de qué subconsulta venga, evitando que el campo quede vacío, y usé la misma función para convertir los conteos nulos en ceros. Por último, utilicé un CASE WHEN que dice de qué lado falta el país (IS NULL) para etiquetar automáticamente el tipo de presencia como 'SOLO CLIENTES', 'SOLO PROVEEDORES' o 'AMBOS'. Se podria haber hecho sin el FULL JOIN pero habría tenido que usar COUNT DISTINCT.

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
SELECT
	'CLIENTE' AS ORIGEN,
	UPPER(CONTACT_NAME) AS CONTACTO,
	COMPANY_NAME AS ORGANIZACION,
	CITY AS CIUDAD,
	COUNTRY AS PAIS
FROM
	CUSTOMERS
UNION ALL
SELECT
	'PROVEEDOR' AS ORIGEN,
	UPPER(CONTACT_NAME) AS CONTACTO,
	COMPANY_NAME AS ORGANIZACION,
	CITY AS CIUDAD,
	COUNTRY AS PAIS
FROM
	SUPPLIERS
UNION ALL
SELECT
	'EMPLEADO' AS ORIGEN,
	UPPER(FIRST_NAME || ' ' || LAST_NAME) AS CONTACTO,
	'NORTHWIND TRADERS' AS ORGANIZACION,
	CITY AS CIUDAD,
	COUNTRY AS PAIS
FROM
	EMPLOYEES
ORDER BY
	ORIGEN,
	PAIS;
```

**Resultado:**

![alt text](img\pregunta-11.png)

**Comentario:** he hecho tres consultas independientes utilizando UNION ALL. Entonces puse como textos ('CLIENTE', 'PROVEEDOR' o 'NORTHWIND TRADERS') directamente en el SELECT para generar las columnas que faltaban, y he usado UPPER() junto con la concatenación || el formato del contacto en los empleados. Es necesario usar el UNION ALL en lugar del UNION estándar porque este último rastrea y elimina filas duplicadas de forma automática. Si dos personas distintas se llamaran igual, estuvieran en la misma ciudad y compartieran empresa, un UNION normal asumiría que es un error y borraría a una de ellas, haciendo que perdamos datos.

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

**Consulta A:**

```sql
SELECT
	COUNTRY AS PAIS
FROM
	CUSTOMERS
EXCEPT
SELECT
	COUNTRY
FROM
	SUPPLIERS
ORDER BY
	PAIS;
```

**Resultado:**

![alt text](img\pregunta-12A.png)

**Consulta B:**

```sql
SELECT
	COUNTRY AS PAIS
FROM
	CUSTOMERS
INTERSECT
SELECT
	COUNTRY
FROM
	SUPPLIERS
ORDER BY
	PAIS;
```

**Resultado:**

![alt text](img\pregunta-12B.png)

**Comentario:** Para resolver el apartado A he usado EXCEPT, que extrae la lista de países de los clientes y le elimina cualquier país que aparezca en la lista de proveedores, dejando solo los que no coinciden. Para el apartado B he usado INTERSECT, que compara ambas listas pero devuelve únicamente los países que están presentes en las dos a la vez. El A se podria haber realizado con LEFT JOIN, intentando buscarle un proveedor a cada cliente. Si no lo encuentra, deja la parte del proveedor en NULL. El WHERE filtra para quedarse solo con esos clientes huérfanos de proveedor, y finalmente el DISTINCT agrupa los países para que no salgan repetidos, el code habría sido mas largo que usando solo EXCEPT.

## Pregunta 13 — Clientes que nunca han comprado pescado

**Enunciado:** El responsable de la categoría Seafood quiere una lista de cuentas sobre las que hacer campaña de captación.

Localiza los clientes que **nunca** han incluido un producto de la categoría `'Seafood'` en ninguno de sus pedidos. Muestra el nombre del cliente, su país y el número total de pedidos que sí ha realizado, de mayor a menor.

**Columnas esperadas:** `cliente`, `pais`, `pedidos_realizados`

**Técnicas:** anti join con `NOT EXISTS`, subconsulta correlacionada, `INNER JOIN` en la subconsulta

> **Pista:** hay tres formas de escribir un anti join: `NOT EXISTS`, `NOT IN` y `LEFT JOIN ... WHERE ... IS NULL`. Escribe la versión con `NOT EXISTS` y después prueba con `NOT IN`. Si la subconsulta de `NOT IN` puede devolver algún `NULL`, el resultado será una tabla vacía sin ningún mensaje de error. Es uno de los fallos más difíciles de detectar en SQL.
> 

**Consulta:**

```sql
SELECT
	C.COMPANY_NAME AS CLIENTE,
	C.COUNTRY AS PAIS,
	COUNT(O.ORDER_ID) AS PEDIDOS_REALIZADOS
FROM
	CUSTOMERS C
	LEFT JOIN ORDERS O ON C.CUSTOMER_ID = O.CUSTOMER_ID
WHERE
	NOT EXISTS (
		SELECT
			1
		FROM
			ORDERS ORD
			INNER JOIN ORDER_DETAILS OD ON ORD.ORDER_ID = OD.ORDER_ID
			INNER JOIN PRODUCTS P ON OD.PRODUCT_ID = P.PRODUCT_ID
			INNER JOIN CATEGORIES CAT ON P.CATEGORY_ID = CAT.CATEGORY_ID
		WHERE
			CAT.CATEGORY_NAME = 'Seafood'
			AND ORD.CUSTOMER_ID = C.CUSTOMER_ID
	)
GROUP BY
	C.COMPANY_NAME,
	C.COUNTRY
ORDER BY
	PEDIDOS_REALIZADOS DESC;
```

**Resultado:**

![alt text](img\pregunta-13.png)

**Comentario:** para seafood he utlizado el NOT EXISTS. Primero agrupo a los clientes y cuento todos sus pedidos usando un LEFT JOIN para no perder a los que tienen 0 pedidos. Luego, el WHERE NOT EXISTS filtra la lista descartando a cualquiera que haya comprado pescado.
El NOT EXISTS hace que por cada cliente que intenta entrar en tu reporte final, el portero ejecuta la subconsulta interna y pregunta: "¿Hay al menos un registro de la categoría Seafood asociado a este cliente?".
Esta linea: AND o2.customer_id = c.customer_id. Enlaza al cliente que está siendo evaluado en la consulta principal con los pedidos de la subconsulta. Si omites esa línea, la subconsulta simplemente miraría si alguien en el mundo ha comprado pescado; como la respuesta sería afirmativa, no habria ningún cliente y saldria vacio todo.

## Pregunta 14 — Productos por encima de la media

**Enunciado:** El comité de precios quiere identificar el segmento premium del catálogo.

Muestra los productos activos cuyo precio unitario supere el precio medio de **todo** el catálogo. Incluye en cada fila el precio del producto, el precio medio general y la diferencia entre ambos, todo redondeado a dos decimales. Ordena por diferencia descendente.

**Columnas esperadas:** `producto`, `precio`, `precio_medio_catalogo`, `diferencia`

**Técnicas:** subconsulta escalar en `WHERE`, subconsulta escalar en `SELECT`, aritmética

> **Pista:** una subconsulta escalar es aquella que devuelve exactamente una fila y una columna, y por eso se puede usar donde iría un valor. Fíjate en que la misma subconsulta aparece en dos sitios; más adelante verás cómo evitar esa repetición con un CTE.
>

**Consulta:**

```sql
SELECT
	PRODUCT_NAME AS PRODUCTO,
	ROUND(UNIT_PRICE::NUMERIC, 2) AS PRECIO,
	ROUND(
		(
			SELECT
				AVG(UNIT_PRICE)
			FROM
				PRODUCTS
		)::NUMERIC,
		2
	) AS PRECIO_MEDIO_CATALOGO,
	ROUND(
		(
			UNIT_PRICE - (
				SELECT
					AVG(UNIT_PRICE)
				FROM
					PRODUCTS
			)
		)::NUMERIC,
		2
	) AS DIFERENCIA
FROM
	PRODUCTS
WHERE
	DISCONTINUED = 0
	AND UNIT_PRICE > (
		SELECT
			AVG(UNIT_PRICE)
		FROM
			PRODUCTS
	)
ORDER BY
	DIFERENCIA DESC;
```

**Resultado:**

![alt text](img\pregunta-14.png)

**Comentario:** He resuelto la consulta haciendo esta subconsulta (SELECT AVG(unit_price) FROM products), que calcula la media de todo el catálogo sin importar si están descatalogados o no. Como esta subconsulta devuelve un único valor matemático, se puede tratar como un número y colocar directamente en el WHERE para filtrar qué productos superan el corte, y en el SELECT para mostrar el dato estático y calcular la resta de la diferencia.

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
SELECT
	C.COMPANY_NAME AS CLIENTE,
	C.COUNTRY AS PAIS,
	COUNT(TOTALES.ORDER_ID) AS NUM_PEDIDOS,
	ROUND(SUM(TOTALES.IMPORTE_PEDIDO)::NUMERIC, 2) AS IMPORTE_TOTAL,
	ROUND(AVG(TOTALES.IMPORTE_PEDIDO)::NUMERIC, 2) AS TICKET_MEDIO
FROM
	CUSTOMERS C
	INNER JOIN (
		SELECT
			O.CUSTOMER_ID,
			O.ORDER_ID,
			SUM(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT)) AS IMPORTE_PEDIDO
		FROM
			ORDERS O
			INNER JOIN ORDER_DETAILS OD ON O.ORDER_ID = OD.ORDER_ID
		GROUP BY
			O.CUSTOMER_ID,
			O.ORDER_ID
	) TOTALES ON C.CUSTOMER_ID = TOTALES.CUSTOMER_ID
GROUP BY
	C.COMPANY_NAME,
	C.COUNTRY
ORDER BY
	TICKET_MEDIO DESC
LIMIT
	15;
```

**Resultado:**

![alt text](img\pregunta-15.png)

**Comentario:** Primero, calculo el coste total exacto de cada pedido sumando sus líneas y agrupando por order_id. Le he asignado  el alias totales a este bloque para que PostgreSQL no devuelva error. Luego, cruzo estos tickets ya cerrados con la tabla de clientes. Al tener el importe del pedido como un solo bloque, ahora es correcto aplicar COUNT para el volumen, SUM para el total y AVG para el ticket medio real. Por último, un ORDER BY DESC seguido de LIMIT 15 recorta el resultado para entregar únicamente el segmento premium de la cartera.

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
SELECT
	C.CATEGORY_NAME AS CATEGORIA,
	P.PRODUCT_NAME AS PRODUCTO,
	ROUND(P.UNIT_PRICE::NUMERIC, 2) AS PRECIO,
	ROUND(
		(
			SELECT
				AVG(UNIT_PRICE)
			FROM
				PRODUCTS P_AVG
			WHERE
				P_AVG.CATEGORY_ID = P.CATEGORY_ID
		)::NUMERIC,
		2
	) AS PRECIO_MEDIO_CATEGORIA
FROM
	PRODUCTS P
	INNER JOIN CATEGORIES C ON P.CATEGORY_ID = C.CATEGORY_ID
WHERE
	P.UNIT_PRICE = (
		SELECT
			MAX(UNIT_PRICE)
		FROM
			PRODUCTS P_MAX
		WHERE
			P_MAX.CATEGORY_ID = P.CATEGORY_ID
	)
ORDER BY
	CATEGORIA;
```

**Resultado:**

![alt text](img\pregunta-16.png)

**Comentario:** He resuelto la consulta utilizando dos subconsultas correlacionadas, enlazando el category_id de la consulta principal con el de las internas. En el WHERE, he comprobado fila a fila si el precio del producto evaluado coincide con el precio máximo exclusivo de su propia categoría, filtrando así al más caro. Simultáneamente, en el SELECT, introduzco otra subconsulta correlacionada para calcular en tiempo real la media de esa misma categoría. Aplicar esta técnica sobre una tabla de diez millones de filas hundiría el rendimiento por completo, el motor de la base de datos se vería obligado a ejecutar un monton de micro-consultas internas, por lo que en escenarios de Big Data reales esto se resolvería utilizando funciones de ventana para hacer el cálculo en una sola pasada.

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
WITH
	FACTURACIONCLIENTE AS (
		SELECT
			O.CUSTOMER_ID,
			SUM(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT)) AS FACTURACION
		FROM
			ORDERS O
			INNER JOIN ORDER_DETAILS OD ON O.ORDER_ID = OD.ORDER_ID
		GROUP BY
			O.CUSTOMER_ID
	),
	CUARTILES AS (
		SELECT
			CUSTOMER_ID,
			FACTURACION,
			NTILE(4) OVER (
				ORDER BY
					FACTURACION DESC
			) AS CUARTIL
		FROM
			FACTURACIONCLIENTE
	),
	GRANTOTAL AS (
		SELECT
			SUM(FACTURACION) AS TOTAL_ABSOLUTO
		FROM
			FACTURACIONCLIENTE
	)
SELECT
	CASE
		WHEN C.CUARTIL = 1 THEN 'A - Estratégico'
		WHEN C.CUARTIL = 2 THEN 'B - Consolidado'
		WHEN C.CUARTIL = 3 THEN 'C - Ocasional'
		WHEN C.CUARTIL = 4 THEN 'D - Marginal'
	END AS SEGMENTO,
	COUNT(C.CUSTOMER_ID) AS NUM_CLIENTES,
	ROUND(SUM(C.FACTURACION)::NUMERIC, 2) AS FACTURACION_SEGMENTO,
	ROUND(
		(SUM(C.FACTURACION) / MAX(G.TOTAL_ABSOLUTO) * 100)::NUMERIC,
		2
	) AS PORCENTAJE_SOBRE_TOTAL
FROM
	CUARTILES C
	CROSS JOIN GRANTOTAL G
GROUP BY
	C.CUARTIL,
	SEGMENTO
ORDER BY
	C.CUARTIL;
```

**Resultado:**

![alt text](img\pregunta-17.png)

**Comentario:** He utilizado para encadenar el proceso paso a paso de forma secuencial. Primero calculo la facturación de cada cliente despues, utilizo la función de ventana NTILE(4) OVER para ordenar a esos clientes de mayor a menor ingreso y cortarlos automáticamente en cuatro grupos exactos. A continuación, he cruzado este resultado con la facturación global de la compañía para tener una base sobre la que calcular los porcentajes. Finalmente, en la consulta principal, el CASE WHEN transforma los números del 1 al 4 en las etiquetas de segmento requeridas, agrupando el resultado final mediante COUNT y SUM para obtener las métricas resumidas.

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
WITH
	VENTASPRODUCTO AS (
		SELECT
			C.CATEGORY_NAME AS CATEGORIA,
			P.PRODUCT_NAME AS PRODUCTO,
			SUM(OD.QUANTITY) AS UNIDADES,
			SUM(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT)) AS FACTURACION
		FROM
			CATEGORIES C
			INNER JOIN PRODUCTS P ON C.CATEGORY_ID = P.CATEGORY_ID
			INNER JOIN ORDER_DETAILS OD ON P.PRODUCT_ID = OD.PRODUCT_ID
		GROUP BY
			C.CATEGORY_NAME,
			P.PRODUCT_NAME
	),
	RANKINGPRODUCTOS AS (
		SELECT
			CATEGORIA,
			PRODUCTO,
			UNIDADES,
			ROUND(FACTURACION::NUMERIC, 2) AS FACTURACION,
			DENSE_RANK() OVER (
				PARTITION BY
					CATEGORIA
				ORDER BY
					FACTURACION DESC
			) AS POSICION_EN_CATEGORIA,
			DENSE_RANK() OVER (
				ORDER BY
					FACTURACION DESC
			) AS POSICION_GLOBAL
		FROM
			VENTASPRODUCTO
	)
SELECT
	CATEGORIA,
	POSICION_EN_CATEGORIA,
	PRODUCTO,
	UNIDADES,
	FACTURACION,
	POSICION_GLOBAL
FROM
	RANKINGPRODUCTOS
WHERE
	POSICION_EN_CATEGORIA <= 3
ORDER BY
	CATEGORIA,
	POSICION_EN_CATEGORIA;
```

**Resultado:**

![alt text](img\pregunta-18.png)

**Comentario:** He estructurado la solución usando dos CTE encadenadas para poder filtrar el podio, ya que, como indica la pista, SQL evalúa el WHERE antes que las funciones de ventana y daría error si intentamos filtrar ahí directamente. En la primera CTE calculo el total de ventas por producto, y en la segunda aplico las ventanas: uso DENSE_RANK() OVER (PARTITION BY categoria ORDER BY facturacion DESC) para generar un ranking que se reinicia al cambiar de familia, y el mismo comando sin PARTITION BY para calcular el puesto global de la compañía. Respecto a la duda de la pista, he elegido DENSE_RANK() porque si dos productos empatan en facturación, ambos compartirán medalla y el siguiente producto será el 3; si usara RANK() el siguiente saltaría al 4, y si usara ROW_NUMBER() el empate se desempataría de forma aleatoria, lo cual sería injusto para un reporte de compras.

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
WITH
	FACTURACIONMENSUAL AS (
		SELECT
			DATE_TRUNC('month', O.ORDER_DATE)::DATE AS MES,
			SUM(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT)) AS FACTURACION
		FROM
			ORDERS O
			INNER JOIN ORDER_DETAILS OD ON O.ORDER_ID = OD.ORDER_ID
		WHERE
			EXTRACT(
				YEAR
				FROM
					O.ORDER_DATE
			) = 1997
		GROUP BY
			DATE_TRUNC('month', O.ORDER_DATE)
	),
	EVOLUCION AS (
		SELECT
			MES,
			FACTURACION,
			SUM(FACTURACION) OVER (
				ORDER BY
					MES
			) AS ACUMULADO,
			AVG(FACTURACION) OVER (
				ORDER BY
					MES ROWS BETWEEN 2 PRECEDING
					AND CURRENT ROW
			) AS MEDIA_MOVIL_3M,
			LAG(FACTURACION) OVER (
				ORDER BY
					MES
			) AS MES_ANTERIOR
		FROM
			FACTURACIONMENSUAL
	)
SELECT
	MES,
	ROUND(FACTURACION::NUMERIC, 2) AS FACTURACION,
	ROUND(ACUMULADO::NUMERIC, 2) AS ACUMULADO,
	ROUND(MEDIA_MOVIL_3M::NUMERIC, 2) AS MEDIA_MOVIL_3M,
	ROUND(MES_ANTERIOR::NUMERIC, 2) AS MES_ANTERIOR,
	ROUND(
		((FACTURACION - MES_ANTERIOR) / MES_ANTERIOR * 100)::NUMERIC,
		2
	) AS VARIACION_PCT
FROM
	EVOLUCION
ORDER BY
	MES;
```

**Resultado:**

![alt text](img\pregunta-19.png)

**Comentario:** He resuelto el ejercicio utilizando dos CTE encadenadas para separar el cálculo de la facturación base de las operaciones analíticas. En la segunda CTE, el total acumulado funciona perfectamente solo con SUM() OVER (ORDER BY mes) porque asume por defecto todo el histórico previo, pero para la media móvil he tenido que definir el marco estricto con ROWS BETWEEN 2 PRECEDING AND CURRENT ROW para limitar el alcance a tres meses. Para la comparativa, he empleado la función LAG() que saca el dato de la fila anterior, lo que permite calcular la variación porcentual en el SELECT final; he optado por dejar el mes anterior y la variación de enero en NULL, ya que inventar un dato sin tener el cierre de diciembre de 1996 rompería la lógica matemática del cuadro de mando.

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
WITH
	CUADROMANDO AS (
		SELECT
			COALESCE(C.CATEGORY_NAME, 'TOTAL') AS CATEGORIA,
			SUM(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT)) FILTER (
				WHERE
					EXTRACT(
						YEAR
						FROM
							O.ORDER_DATE
					) = 1996
			) AS F_1996,
			SUM(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT)) FILTER (
				WHERE
					EXTRACT(
						YEAR
						FROM
							O.ORDER_DATE
					) = 1997
			) AS F_1997,
			SUM(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT)) FILTER (
				WHERE
					EXTRACT(
						YEAR
						FROM
							O.ORDER_DATE
					) = 1998
			) AS F_1998,
			SUM(OD.UNIT_PRICE * OD.QUANTITY * (1 - OD.DISCOUNT)) AS TOTAL
		FROM
			CATEGORIES C
			INNER JOIN PRODUCTS P ON C.CATEGORY_ID = P.CATEGORY_ID
			INNER JOIN ORDER_DETAILS OD ON P.PRODUCT_ID = OD.PRODUCT_ID
			INNER JOIN ORDERS O ON OD.ORDER_ID = O.ORDER_ID
		GROUP BY
			ROLLUP (C.CATEGORY_NAME)
	)
SELECT
	CATEGORIA,
	COALESCE(ROUND(F_1996::NUMERIC, 2), 0) AS F_1996,
	COALESCE(ROUND(F_1997::NUMERIC, 2), 0) AS F_1997,
	COALESCE(ROUND(F_1998::NUMERIC, 2), 0) AS F_1998,
	ROUND(TOTAL::NUMERIC, 2) AS TOTAL,
	ROUND((TOTAL / MAX(TOTAL) OVER () * 100)::NUMERIC, 2) AS PESO_PCT,
	 
	CASE
		WHEN COALESCE(F_1998, 0) > COALESCE(F_1997, 0) THEN 'CRECE'
		WHEN COALESCE(F_1998, 0) < COALESCE(F_1997, 0) THEN 'DECRECE'
		ELSE 'ESTABLE'
	END AS TENDENCIA
FROM
	CUADROMANDO
ORDER BY
	CATEGORIA = 'TOTAL',
	TOTAL DESC;
```

**Resultado:**

![alt text](img\pregunta-20.png)

**Comentario:** En la CTE aplico la cláusula ROLLUP sobre las categorías para generar una fila extra al final con la suma de todo, cogiendo su nombre nulo con un COALESCE para etiquetarla como 'TOTAL'. Despues use FILTER (WHERE...) para calcular el peso de cada año de forma independiente. En la consulta principal calculo el porcentaje de peso utilizando una función de ventana (MAX(total) OVER()) que coge el número de la fila 'TOTAL' generada por el ROLLUP, y añado un bloque CASE WHEN para la tendencia.