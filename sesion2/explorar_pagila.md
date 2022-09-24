```sql
-- Explorar tablas
SELECT * FROM actor;
SELECT * FROM actor WHERE last_name = 'CHASE';

/*
La tabla address tiene una columna de city_id. 
El _id nos indica que es una clave foránea 
que apunta a otra tabla, en este caso, city.
*/
SELECT * FROM address;
SELECT * FROM city;

SELECT * FROM address WHERE city_id = 300;

SELECT * FROM address WHERE district = 'California' AND postal_code = '17886';
SELECT * FROM address WHERE postal_code = '1027' OR postal_code = '17886';


-- Seleccionar campos que empiezan por 'A'
SELECT * FROM city WHERE city LIKE 'A%';

/*
address apunta a city (citi_id), que, a su vez, apunta a country (country_id).
Para obtener datos de una tabla, tendremos que pasar por otra.
*/

/*
Normalmente las bases de datos tienen una tabla 
que es la más importante y el resto giran en torno a ella. 
Son bases de datos en estrella.
En este caso la tabla principal es payment.
Esta tabla tiene una P. Eso significa 
que está particionada para optimizar la consulta.
*/

/*
La tabla film_actor asocia las tablas film (film_id) y actor (actor_id)
*/
SELECT * FROM actor;
SELECT * FROM film;
SELECT * FROM film_actor;

-- Filtrar por texto que contiene
SELECT title, description FROM film WHERE description LIKE '%Drama%';

-- Insertar nuevos datos
SELECT * FROM actor;
INSERT INTO actor(first_name, last_name) VALUES('NICOLE', 'DA SILVA');

/*
Explorar bien todas las tablas.
Puede ser que para crear un registro (como customer)
tengamos que crear otros registros en otras tablas (address)
*/
SELECT * FROM customer;
SELECT * FROM address;
SELECT * FROM store;

INSERT INTO address(address, district, city_id, postal_code, phone)
VALUES ('28 MySQL Boulevard', 'QLD', 576, '6172235589', '645892137')

INSERT INTO customer (store_id, first_name, last_name, email, address_id, activebool)
VALUES (1, 'MARY', 'JONES', 'm.jones@sakilacustomer.com', 606, true)



```