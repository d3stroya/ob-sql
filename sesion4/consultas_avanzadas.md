```sql

/*
Obtener todos los datos o únicos
*/
-- 604 resultados (todos)
SELECT * FROM address;

-- 604 resultados (todos)
SELECT district from address;

-- DISTINCT: 378 resultados (quita los duplicados)
SELECT DISTINCT district FROM address;

/*
AND: se tienen que cumplir las condiciones
OR: se tiene que cumplir una condición o la otra
NOT: niega la condición
*/

--NOT o != 
SELECT * FROM address WHERE district != 'California';
SELECT * FROM address WHERE NOT district = 'California';
SELECT * FROM address WHERE address2 IS NOT null;

-- Ordenar alfabéticamente
SELECT * FROM address WHERE district != 'California' ORDER BY district;

-- OR
SELECT * FROM address WHERE district = 'California' OR district = 'Aceh';

-- AND
SELECT * FROM address WHERE district IS NOT null AND NOT district = '' ORDER BY district;

/*
Agrupar con GROUP BY
Hay que usar una función de agregación: COUNT(district),
es decir, contar cuántas veces se repite cada distrito en la tabla address.
AS permite poner un alias a la columna de COUNT (en lugar de llamarse
count, se llamará num).

SELECT columna, función de agregación(columna) AS alias FROM tabla GROUP BY columna ORDER BY columna.

*/
SELECT district, COUNT(district) AS num FROM address GROUP BY district ORDER BY district;

-- Ver si hay actores/actrices con el mismo apellido
SELECT * FROM actor;
SELECT last_name, count(last_name) FROM actor GROUP BY last_name;

-- COUNT no puede ir después de WHERE, así que usamos HAVING: filtra sobre el agregado count()
SELECT last_name, count(last_name) FROM actor GROUP BY last_name HAVING count(last_name) > 1 ORDER BY count DESC;

-- Obtener el número de películas en las que aparece cada actor/actriz
SELECT * FROM film_actor;
SELECT * FROM film;
SELECT * FROM actor;

SELECT f.title, count(fa.actor_id) AS appereances FROM film f
INNER JOIN film_actor fa ON f.film_id = fa.film_id
GROUP BY f.title

/*
Unir tablas con JOINS para hacer una petición compleja, es decir,
en lugar de hacer 4 peticiones a 4 tablas relacionadas entre sí,
unimos las tablas para hacer una petición y de ahí obtenerlo todo.
*/
-- Ver la estructura
SELECT * FROM customer;
SELECT * FROM address;
SELECT * FROM city;
SELECT * FROM country;

-- Consulta a 2 tablas: customer y address: SELECT columnas FROM tabla1 INNER JOIN tabla2 ON tabla1.columna_foreignkey = tabla2.columna_primarykey
SELECT first_name, last_name, address_id FROM customer
INNER JOIN address ON customer.address_id = address.address_id
/* Saca un error: ERROR:  la referencia a la columna «address_id» es ambigua. 
Se debe a que tenemos dos columnas con el mismo nombre en las tablas (address_id).
Para solucionarlo, añadimos el nombre de la tabla y un punto delante del atributo.
*/
SELECT first_name, last_name, customer.address_id FROM customer
INNER JOIN address ON customer.address_id = address.address_id

/* Para abreviar, podemos darle alias a las tablas */
SELECT first_name, last_name, c.address_id FROM customer c
INNER JOIN address a ON c.address_id = a.address_id

-- Sacamos la calle
SELECT first_name, last_name, c.address_id, address FROM customer c
INNER JOIN address a ON c.address_id = a.address_id

-- Sacamos email (de la tabla customer) y address (de la tabla address)
SELECT c.email, a.address FROM customer c
INNER JOIN address a ON c.address_id = a.address_id

-- Consulta a 3 tablas: customer, address, city
SELECT cu.email, a.address, ci.city FROM customer cu
INNER JOIN address a ON cu.address_id = a.address_id
INNER JOIN city ci ON a.city_id = ci.city_id

-- Consulta a 4 tablas: customer, address, city y country
/* Estamos cogiendo: 
 -> los emails de la tabla customer, 
 -> la dirección de la tabla address 
 -> la ciudad de la tabla city
 -> y el país de la tabla country
 */
SELECT cu.email, a.address, ci.city, co.country FROM customer cu
INNER JOIN address a ON cu.address_id = a.address_id
INNER JOIN city ci ON a.city_id = ci.city_id
INNER JOIN country co ON ci.country_id = co.country_id


/*
Concatenar columnas con CONCAT()
Permite crear, por ejemplo, un texto 
que una nombre y apellido separado por un espacio.
Crea una nueva columna con el alia que queramos.

SELECT CONCAT(columna1, separador, columna2) AS alias FROM tabla;
*/
SELECT * FROM actor;
SELECT first_name, last_name FROM actor;
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM actor;

/* 
LIKE obtiene campos que contienen una(s) palabra(s)
Los porcentajes indican que en ese lugar hay otro texto.
*/
SELECT * FROM film;
-- Tiene más palabras delante y detrás
SELECT * FROM film WHERE description LIKE '%Epic%';
-- Descripción que termina con Monastery
SELECT * FROM film WHERE description LIKE '%Monastery';

/* 
Combinación de LIKE y ORDER BY
*/
SELECT * FROM actor;
-- Orden ascendente
SELECT * FROM actor WHERE last_name LIKE '%LI%' ORDER BY last_name;
-- Orden descendente
SELECT * FROM actor WHERE last_name LIKE '%LI%' ORDER BY last_name DESC;

/*
IN() recupera los valores que indicamos entre paréntesis que hay dentro de la tabla indicada

SELECT columna FROM tabla WHERE columna IN(valores)
*/
SELECT * FROM country;
SELECT * FROM country WHERE country = 'Spain' OR country = 'Germany';
SELECT * FROM country WHERE country = 'Spain' OR country = 'Germany' OR country = 'France';

-- Para optimizar el código usamos IN
SELECT * FROM country WHERE country IN('Spain', 'Germany', 'France', 'Italy');

-- Seleccionar clientes con ciertos IDs
SELECT * FROM customer;
SELECT * FROM customer WHERE customer_id IN(15, 67, 34, 132);

-- stock de una película en base a su título
select * from inventory;

select f.title, count(i.inventory_id) as unidades from film f
inner join inventory i on i.film_id = f.film_id
GROUP BY title;

select f.title, count(i.inventory_id) as unidades from film f
inner join inventory i on i.film_id = f.film_id
WHERE title = 'FICTION CHRISTMAS'
GROUP BY title;

select f.title, count(i.inventory_id) as unidades from film f
inner join inventory i on i.film_id = f.film_id
GROUP BY title ORDER BY unidades;

select f.title, count(i.inventory_id) as unidades from film f
inner join inventory i on i.film_id = f.film_id
GROUP BY title ORDER BY unidades DESC;

/*
 SUM()
 */
 select * from customer;
select * from payment;

SELECT * FROM payment p
inner join customer c on p.customer_id = c.customer_id

SELECT c.email, count(p.payment_id) as num_pagos FROM payment p
inner join customer c on p.customer_id = c.customer_id
group by c.email

SELECT c.email, sum(p.amount) as num_pagos FROM payment p
inner join customer c on p.customer_id = c.customer_id
group by c.email

select * from staff;

select * from payment p
inner join staff s on p.staff_id = s.staff_id

select s.first_name, count(p.payment_id) as num_ventas, sum(p.amount) cantidad_ventas from payment p
inner join staff s on p.staff_id = s.staff_id
group by s.first_name

/*
 Sub queries
*/
select * from film;
select * from language;

select distinct language_id from film;

select * from film f
inner join language l on f.language_id = l.language_id 

select l.name, count(f.film_id) from film f
inner join language l on f.language_id = l.language_id 
group by l.name

-- Cambiar idioma a algunas películas
UPDATE film SET language_id = 2 WHERE film_id > 100 and film_id < 200;
UPDATE film SET language_id = 3 WHERE film_id >= 200 and film_id < 300;
UPDATE film SET language_id = 4 WHERE film_id >= 300 and film_id < 400;

-- Filtrar películas cuyo idioma sea English
SELECT title FROM film WHERE language_id = (SELECT language_id FROM language WHERE name = 'English');

-- Filtrar películas cuyo idioma sea English o Italian (se sustituye el = por IN)
SELECT title FROM film WHERE language_id IN (SELECT language_id FROM language WHERE name = 'English' OR name = 'Italian');

-- Películas más alquiladas
/* 
COMBINAR SUB QUERY CON INNER JOIN.
*/
SELECT * FROM rental;
SELECT * FROM film;
SELECT * FROM inventory;

SELECT f.title, COUNT(f.film_id) AS rental_times FROM film f
INNER JOIN (SELECT * FROM inventory i 
INNER JOIN rental r ON r.inventory_id = i.inventory_id) res ON res.film_id = f.film_id
GROUP BY f.title
ORDER BY rental_times DESC; 

```
