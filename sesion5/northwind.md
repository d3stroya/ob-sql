# Trabajar con base de datos Northwind
1. Crear base de datos 'northwind'
2. Descargar los archivos desde https://github.com/pthom/northwind_psql
3. Abrir cmd desde la carpeta de northwind
4. Ejecutar el comando psql -U postgres -d northwind < northwind.sql
   1. -U: usuario
   2. -d: base de datos
   3. <: archivo

# Optimización de bases de datos y consultas
```sql
/*
Ver el tamaño de bases de datos y tablas para tener una idea
de cómo funciona la BD.
*/
-- Ver el espacio que ocupa la base de datos
SELECT pg_size_pretty (pg_database_size('northwind'))

-- Ver el peso de todas las bases de datos
SELECT pg_database.datname, pg_size_pretty(pg_database_size(pg_database.datname)) AS size FROM pg_database;

-- Ver tamaño de una tabla
SELECT pg_size_pretty(pg_relation_size('orders'))

-- Ver tamaño de las 10 tablas que más ocupan
SELECT
    relname AS "relation",
    pg_size_pretty (
        pg_total_relation_size (C .oid)
    ) AS "total_size"
FROM
    pg_class C
LEFT JOIN pg_namespace N ON (N.oid = C .relnamespace)
WHERE
    nspname NOT IN (
        'pg_catalog',
        'information_schema'
    )
AND C .relkind <> 'i'
AND nspname !~ '^pg_toast'
ORDER BY
    pg_total_relation_size (C .oid) DESC
LIMIT 10;

-- Ver esquema actual
SELECT current_schema()

-- Ver todas las vistas materializadas de la base de datos
SELECT * FROM pg_matviews;

-- Cargar extensiones
CREATE EXTENSION pgcrypto;

/*
1. CONSULTAS JOIN

INNER: trae todas las filas incluidas en las tablas unidas. 
Es decir, si representamos 2 tablas como 2 círculos, 
inner join coge la intersección de las 2. Es la más común.

LEFT: trae todas las filas de la tabla de la izquierda (la primera), 
incluyendo las filas que tiene en común con la segunda tabla. 
Es decir, trae la primera tabla completa.

RIGHT: opuesto a left.

Distinción entre INNER RIGHT/LEFT: 
En inner join obtenemos 830 resultados, pero en right join obtenemos 831, 
porque trae todos los datos de employees aunque no estén relacionados con orders.
*/
SELECT * FROM customers;
SELECT * FROM orders;
SELECT * FROM shippers;

-- 830 resultados
SELECT o.order_id, c.contact_name FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id

SELECT o.order_id, c.contact_name, s.company_name FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN shippers s ON o.ship_via = s.shipper_id

-- 832 resultados, porque trae clientes (customers) sin pedidos (orders)
SELECT o.order_id, c.contact_name FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.customer_id

-- Insertamos un empleado sin pedidos
SELECT * FROM employees

INSERT INTO employees (employee_id, last_name, first_name, title)
VALUES (10, 'Miller', 'Anne', 'Sales Representative')

-- Ofuscar datos con extensión
CREATE EXTENSION pgcrypto;

INSERT INTO employees (employee_id, last_name, first_name, title, notes)
VALUES (11, 'Miller', 'Anne', 'Sales Representative', pgp_sym_encrypt('Notes', 'password')

SELECT id, pfp_sym_decrypt(notes::bytea, 'password') AS email FROM users;

-- 830 resultados
SELECT o.order_id, e.first_name, e.last_name FROM orders o
INNER JOIN employees e ON o.employee_id = e.employee_id

-- 831 resultados, porque trae un empleado sin pedido asociado.
SELECT o.order_id, e.first_name, e.last_name FROM orders o
RIGHT JOIN employees e ON o.employee_id = e.employee_id

/*
2. GROUP BY
*/
SELECT * FROM customers
SELECT * FROM orders
SELECT * FROM employees

-- Países con más clientes
SELECT city, count(customer_id) AS num_customers FROM customers GROUP BY city ORDER BY num_customers DESC
SELECT city, count(customer_id) AS num_customers FROM customers GROUP BY city ORDER BY city 

SELECT country, count(customer_id) AS num_customers FROM customers GROUP BY country ORDER BY num_customers DESC

-- Qué puesto ha vendido más
SELECT e.title, count(o.order_id) as num_orders FROM orders o
INNER JOIN employees e ON o.employee_id = e.employee_id
GROUP BY e.first_name
ORDER BY num_orders DESC

-- Qué empleado ha vendido más
SELECT e.first_name, count(o.order_id) as num_orders FROM orders o
INNER JOIN employees e ON o.employee_id = e.employee_id
GROUP BY e.first_name
ORDER BY num_orders DESC


/* 
3. VISTAS: Guardar una consulta con un identificador 
para que el resultado quede grabado, 
ya que las consultas por sí solas no se guardan.

CREATE VIEW identificador AS consulta;

Queda guardado en Views.

Hecho esto, podemos acceder a esta misma consulta
múltiples veces con una sola línea.
*/
-- Crear vista
CREATE VIEW num_orders_by_employee AS 
SELECT e.first_name, count(o.order_id) as num_orders FROM orders o
INNER JOIN employees e ON o.employee_id = e.employee_id
GROUP BY e.first_name
ORDER BY num_orders DESC

-- Acceder a la vista
SELECT * FROM num_orders_by_employee

/*
4. VISTAS MATERIALIZADAS: Guarda los datos de una vista,
con lo que se acelera la consulta porque no hay que volver
a sacar los datos de la BD, sino que los guarda en una caché.
Actualiza los datos periódicamente, ya que permite refrescar los
datos cuando sea necesario.

CREATE MATERIALIZED VIEW [IF NOT EXISTS] identificador AS 
consulta 
WITH [NO] DATA;
*/
CREATE MATERIALIZED VIEW mv_num_orders_by_employee AS
SELECT e.first_name, count(o.order_id) as num_orders FROM orders o
INNER JOIN employees e ON o.employee_id = e.employee_id
GROUP BY e.first_name
ORDER BY num_orders DESC
WITH DATA

SELECT * FROM mv_num_orders_by_employee


/* 
5. GENERATE_SERIES (start, stop)
Generar datos para insertarlos en tablas
*/
CREATE TABLE example (
    id INT,
    name VARCHAR
)

SELECT * FROM example

INSERT into example(id) 
SELECT * FROM generate_series(1,5000)

CREATE MATERIALIZED VIEW mv_example AS
SELECT * FROM example
WITH DATA

SELECT * FROM mv_example

SELECT * FROM generate_series(
    '2022-01-01 00:00'::timestamp,
    '2022-12-31 23:59',
    '6 hours'
)

/*
6. MÉTODOS PARA OPTIMIZAR LA BD Y LAS CONSULTAS
*/
-- EXPLAIN ANALYZE: Da información sobre el "coste" de la consulta: cómo se realizan, cuánto tarda,...
EXPLAIN ANALYZE SELECT * FROM order_details;
EXPLAIN ANALYZE SELECT * FROM num_orders_by_employee;
EXPLAIN ANALYZE SELECT * FROM orders;

-- ÍNDICES: estructura de datos que permite optimizar las consultas en base a una columna o filtro en particular con el fin de evitar el escaneo secuencial.
CREATE INDEX idx_orders_pk ON orders(order_id)
EXPLAIN ANALYZE SELECT * FROM orders;

-- Sequence scan
EXPLAIN ANALYZE SELECT * FROM example;

-- Crear un índice para optimizar la consulta. Es útil en tablas con muchos datos.
CREATE INDEX idx_example_pk ON example(id)

-- Index scan
EXPLAIN ANALYZE SELECT * FROM example WHERE id = 3965

-- PARTICIONAMIENTO: dividir una tabla en particiones en base a ciertos datos de una columna (por ejemplo, por país). Es útil a partir de medio millón de consultas, apróx.
-- De tipo rango
-- De tipo lista
-- De tipo hash

-- Crear una tabla
CREATE TABLE users (
    -- BIGSERIAL ofrece un rango de números mayor que SERIAL
    id BIGSERIAL,
    birth_date DATE NOT NULL,
    first_name VARCHAR(20) NOT NULL,
    PRIMARY KEY(id, birth_date)
-- Disponible a partir de PostgreSQL 9
) PARTITION BY RANGE (birth_date)

-- Crear las particiones
CREATE TABLE users_2020 PARTITION OF users 
                -- Incluido         Excluido
FOR VALUES FROM ('2020-01-01') TO ('2021-01-01')

CREATE TABLE users_2021 PARTITION OF users 
FOR VALUES FROM ('2021-01-01') TO ('2022-01-01')

CREATE TABLE users_2022 PARTITION OF users 
FOR VALUES FROM ('2022-01-01') TO ('2023-01-01')

-- Insertar datos en la tabla padre. Esto ya los inserta directamente en la partición que corresponda.
INSERT INTO users(birth_date, first_name) VALUES
('2020-01-16', 'User1'),
('2021-08-16', 'User2'),
('2022-05-16', 'User3'),
('2020-12-16', 'User4'),
('2021-03-16', 'User5'),
('2021-01-16', 'User6'),
('2022-07-16', 'User7');

SELECT * FROM users_2020
SELECT * FROM users_2021
SELECT * FROM users_2022

EXPLAIN ANALYZE SELECT * FROM users

/* 
Hace automáticamente la búsqueda en la tabla users_2022, 
ignorando la tabla users, que tiene más datos.
*/
EXPLAIN ANALYZE SELECT * FROM users WHERE birth_date = '2022-05-16'

-- Extraer los usuarios que cumplen años en mayo
SELECT * FROM users WHERE EXTRACT (month FROM birth_date) = 5
EXPLAIN ANALYZE SELECT * FROM users WHERE EXTRACT (month FROM birth_date) = 5 AND EXTRACT (year FROM birth_date) = 2022
```