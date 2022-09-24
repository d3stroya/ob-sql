# Diseño de base de datos
Crearemos una simulación de base de datos de lo que podría ser un concesionario.
Tenemos una tabla central de ventas, sale, y otras como:
* customer
* employee
* extra
* extra_version
* manufacturer
* model
* vehicle
* version


+----------------+
|      SALE      |
+----------------+
|  id            |
|  sale_date     |
|  description   |
|  vehicle_id    |
|  employee_id   |      
|  customer_id   |
+----------------+

+--------------------+
|      EMPLOYEE      |
+--------------------+

## Empezamos a crear tablas y a insertar datos
``` sql
-- 1. MANUFACTURER
-- Crear tabla con sus atributos 
/*
constraint crea claves: 
pk_manufacturer PRIMARY KEY (id) define el atributo id 
como clave primaria con el nombre pk_manufacturer
*/
CREATE TABLE manufacturer (
    id SERIAL,
    name VARCHAR(50) NOT NULL,
    num_employees INT,
    CONSTRAINT pk_manufacturer PRIMARY KEY (id)
);

-- Insertar datos
SELECT * FROM manufacturer;
INSERT INTO manufacturer (name, num_employees) VALUES ('Ford', 29000);
INSERT INTO manufacturer (name, num_employees) VALUES ('Toyota', 45000);
INSERT INTO manufacturer (name, num_employees) VALUES ('Volkswagen', 34000);
INSERT INTO manufacturer (name, num_employees) VALUES ('Seat', 18000);

-- 2. MODEL
/*
Llaves primarias: 
    PRIMARY KEY
Llaves foráneas (relacionan un atributo de la tabla A con un atributo de la tabla B).
En este caso, definimos id como llave primaria
y manufacturer_id como llave foránea para relacionarlo con 
el id de la tabla manufacturer.
    FOREIGN KEY(columna) REFERENCES tabla_a_la_que_apunta(columna_foránea)
    
Las relaciones pueden ser:
    De 1 a 1: UNIQUE
    De 1 a muchos: REFERENCES --> vincula columnas
    De muchos a muchos --> necesitamos una tabla nueva donde hacer esas relaciones
*/
CREATE TABLE model (
    id SERIAL,
    name VARCHAR(50) NOT NULL,
    manufacturer_id INT,
    CONSTRAINT pk_model PRIMARY KEY(id),
    CONSTRAINT fk_model_manufacturer FOREIGN KEY(manufacturer_id) REFERENCES manufacturer(id)
);

SELECT * FROM model;

INSERT INTO model (name, manufacturer_id) VALUES ('Mondeo', 2);
UPDATE model SET manufacturer_id = 1 WHERE name = 'Mondeo';

INSERT INTO model (name, manufacturer_id) VALUES ('Fiesta', 2);
UPDATE model SET name = 'Prius' WHERE manufacturer_id = 2;

INSERT INTO model (name, manufacturer_id) VALUES ('Polo', 3);

-- 3. VERSION
CREATE TABLE version (
    id SERIAL,
    name VARCHAR(50) NOT NULL,
    engine VARCHAR(50),
    price NUMERIC,
    cc NUMERIC(2,1),
    model_id INT,
    CONSTRAINT pk_version PRIMARY KEY (id),
    /*
    ON DELETE: Si borramos una entidad que esté relacionada con model_id, se borrarán todos los atributos de ese registro (CASCADE).
    ON UPDATE: Si actualizamos, todos los atributos pasan a null (SET NULL).
    */
    CONSTRAINT fk_version_model FOREIGN KEY (model_id) REFERENCES model (id) ON UPDATE set null ON DELETE cascade
);

SELECT * FROM version;

INSERT INTO version (name, engine, price, cc, model_id) VALUES ('Basic', 'Diesel', 30000, 1.9, 1);
INSERT INTO version (name, engine, price, cc, model_id) VALUES ('Medium', 'Diesel', 40000, 2.2, 1);
INSERT INTO version (name, engine, price, cc, model_id) VALUES ('Advanced', 'Diesel', 50000, 3.2, 1);

INSERT INTO version (name, engine, price, cc, model_id) VALUES ('City', 'Diesel', 30000, 1.9, 2);
INSERT INTO version (name, engine, price, cc, model_id) VALUES ('Sport', 'Diesel', 40000, 2.2, 2);
INSERT INTO version (name, engine, price, cc, model_id) VALUES ('Sport Advanced', 'Diesel', 50000, 3.2, 2);

INSERT INTO version (name, engine, price, cc, model_id) VALUES ('AllStar', 'Diesel', 30000, 1.9, 3);
INSERT INTO version (name, engine, price, cc, model_id) VALUES ('GTI', 'Diesel', 40000, 2.2, 3);

-- 4. EXTRA
/*
Va a tener relaciones de muchos a muchos
*/
CREATE TABLE extra (
    id SERIAL,
    name VARCHAR(50) NOT NULL,
    description VARCHAR(300),
    CONSTRAINT pk_extra PRIMARY KEY(id)
);

SELECT * FROM extra;

INSERT INTO extra (name, description) VALUES ('Techo solar', 'Descripción...');
INSERT INTO extra (name, description) VALUES ('Limitador de velocidad', 'Descripción...');
INSERT INTO extra (name, description) VALUES ('Climatizador por zonas', 'Descripción...');

-- 5. EXTRA_VERSION: relaciona las tablas extra y version
/*
En esta tabla no se crean nuevos registros,
sino que se relacionan registros que ya existen.

La clave primaria puede ser de una columna o de varias.
*/
CREATE TABLE extra_version(
    version_id INT,
    extra_id INT,
    price NUMERIC NOT NULL CHECK (price >= 0),
    CONSTRAINT pk_extra_version PRIMARY KEY(version_id, extra_id),
    CONSTRAINT fk_version_extra FOREIGN KEY(version_id) REFERENCES version(id) ON UPDATE cascade ON DELETE cascade,
    CONSTRAINT fk_extra_version FOREIGN KEY(extra_id) REFERENCES extra(id) ON UPDATE cascade ON DELETE cascade
);

SELECT * FROM extra_version;

-- 5.1. Insertar relaciones
-- Volkswagen Polo GTI - Techo solar
INSERT INTO extra_version VALUES (8, 1, 500.99);

-- Volkswagen Polo GTI - Limitador de velocidad
INSERT INTO extra_version VALUES (8, 2, 400.23);

-- Volkswagen Polo GTI - Climatizador
INSERT INTO extra_version VALUES (8, 3, 900.89);


-- Volkswagen Polo AllStar - Techo solar
INSERT INTO extra_version VALUES (7, 1, 500.99);

-- Volkswagen Polo AllStar - Limitador de velocidad
INSERT INTO extra_version VALUES (7, 2, 400.23);

-- Volkswagen Polo AllStar - Climatizador
INSERT INTO extra_version VALUES (7, 3, 900.89);

-- 6. EMPLOYEE
CREATE TABLE employee (
    id SERIAL,
    name VARCHAR(30),
    nif VARCHAR(9) NOT NULL UNIQUE,
    phone VARCHAR(9),
    CONSTRAINT pk_employee PRIMARY KEY(id)
);

INSERT INTO employee (name, nif, phone) VALUES ('Ann', '567439O', '6748593');
INSERT INTO employee (name, nif, phone) VALUES ('Jack', '54895612P', '6124578');

SELECT * FROM employee;

-- 7. CUSTOMER
CREATE TABLE customer (
    id SERIAL, 
    name VARCHAR(30),
    email VARCHAR(9) NOT NULL UNIQUE,
    CONSTRAINT pk_customer PRIMARY KEY(id)
);

INSERT INTO customer (name, email) VALUES ('James', 'james@com');
INSERT INTO customer (name, email) VALUES ('Lucy', 'lucy@com');

SELECT * FROM customer;

-- 8. VEHICULO
CREATE TABLE vehicle (
    id SERIAL,
    license_num VARCHAR(7),
    creation_date DATE,
    price_gross NUMERIC,
    price_net NUMERIC,
    type VARCHAR(30),
    
    manufacturer_id INT,
    model_id INT,
    version_id INT,
    extra_id INT,
    
    CONSTRAINT pk_vehicle PRIMARY KEY(id),
    CONSTRAINT fk_vehicle_manufacturer FOREIGN KEY(manufacturer_id) REFERENCES manufacturer(id),
    CONSTRAINT fk_vehicle_model FOREIGN KEY(model_id) REFERENCES model(id),
    -- Apuntar a la tabla extra_version, que tiene un id compuesto por 2 columnas
    CONSTRAINT fk_vehicle_extra_version FOREIGN KEY(version_id, extra_id) REFERENCES extra_version(version_id, extra_id)
);

INSERT INTO vehicle (license_num, creation_date, price_gross, manufacturer_id, model_id, version_id, extra_id)
VALUES ('5478LPG', '2022-01-12', 29354.56, 3, 3, 8, 1);

INSERT INTO vehicle (license_num, creation_date, price_gross, manufacturer_id, model_id, version_id, extra_id)
VALUES ('5346LTD', '2021-12-17', 32123.98, 3, 3, 8, 2);

SELECT * FROM vehicle;

-- 9. SALE
CREATE TABLE sale (
    id SERIAL,
    sale_date DATE,
    description VARCHAR(300),
    
    vehicle_id INT,
    employee_id INT,
    customer_id INT,
    
    CONSTRAINT pk_sale PRIMARY KEY(id),
    CONSTRAINT fk_sale_vehicle FOREIGN KEY(vehicle_id) REFERENCES vehicle(id),
    CONSTRAINT fk_sale_employee FOREIGN KEY(employee_id) REFERENCES employee(id),
    CONSTRAINT fk_sale_customer FOREIGN KEY(customer_id) REFERENCES customer(id)
);

INSERT INTO sale (sale_date, description, vehicle_id, employee_id, customer_id)
VALUES ('2022-01-20', 'Easy sale', 3, 1, 2);

SELECT * FROM sale;

/*
Con esto, ya podemos hacer consultas para saber aspectos como:
    - Cuántos cohces hemos vendido
    - Cuántos coches ha vendido cada empleado
    - Cuántos coches ha comprado cada cliente
    - Cuántos coches hemos vendido de cada marca/modelo
    - Cuántos extras hemos vendido
    - El dinero generado
    - La comisión que gana cada empleado
    ...
*/
```

## Añadimos alguna información más:
```sql
-- Añadir más datos
INSERT INTO sale (sale_date, description, vehicle_id, employee_id, customer_id)
VALUES ('2022-03-23', 'Easy sale', 3, 2, 2);

INSERT INTO sale (sale_date, description, vehicle_id, employee_id, customer_id)
VALUES ('2021-01-20', 'Easy sale', 3, 1, 2);

INSERT INTO sale (sale_date, description, vehicle_id, employee_id, customer_id)
VALUES ('2022-08-12', 'Easy sale', 3, 2, 2);

INSERT INTO sale (sale_date, description, vehicle_id, employee_id, customer_id)
VALUES ('2022-08-12', 'Easy sale', 3, 2, 1);

INSERT INTO sale (sale_date, description, vehicle_id, employee_id, customer_id)
VALUES ('2022-08-12', 'Easy sale', 3, 1, 1);

INSERT INTO sale (sale_date, description, vehicle_id, employee_id, customer_id)
VALUES ('2022-06-25', 'Easy sale', 4, 1, 1);

INSERT INTO sale (sale_date, description, vehicle_id, employee_id, customer_id)
VALUES ('2022-01-31', 'Easy sale', 4, 1, 1);

-- Ventas por empleado
SELECT * FROM sale
SELECT * FROM employee

SELECT e.name, COUNT(s.id) FROM sale s
INNER JOIN employee e ON s.employee_id = e.id
GROUP BY e.name

-- Compras por cliente
SELECT c.email, COUNT(s.id) FROM sale s
INNER JOIN customer c ON s.customer_id = c.id
GROUP BY c.email;

-- Ventas por fecha
SELECT * FROM sale WHERE sale_date >= '2022-03-01' AND sale_date <= '2022-03-31';

SELECT e.name, SUM(v.price_gross) AS total_gross FROM sale s 
INNER JOIN vehicle v ON s.vehicle_id = v.id
INNER JOIN employee e ON s.employee_id = e.id
WHERE sale_date >= '2022-01-01' AND sale_date <= '2022-01-31'
GROUP BY e.name;

-- Fabricante más vendido
SELECT * FROM vehicle;
SELECT * FROM sale;
SELECT * FROM manufacturer;

SELECT m.name, COUNT(s.id) FROM sale s
INNER JOIN vehicle v ON s.vehicle_id = v.id
INNER JOIN manufacturer m ON v.manufacturer_id = m.id
GROUP BY m.name;

-- Modelo más vendido
SELECT * FROM model;
SELECT * FROM sale;
SELECT * FROM vehicle;

SELECT m.name, COUNT(s.id) FROM sale s
INNER JOIN vehicle v ON s.vehicle_id = v.id
INNER JOIN model m ON v.model_id = m.id
GROUP BY m.name;

-- Versión más vendida
SELECT * FROM version;
SELECT * FROM sale;
SELECT * FROM vehicle;

SELECT ve.name, COUNT(s.id) num_sells FROM sale s 
INNER JOIN vehicle v ON s.vehicle_id = v.id
INNER JOIN version ve ON v.version_id = ve.id
GROUP BY ve.name
ORDER BY num_sells;

-- Extra más vendido
SELECT * FROM extra;
SELECT * FROM sale;
SELECT * FROM vehicle;

SELECT e.name, COUNT(s.id) num_sells FROM sale s 
INNER JOIN vehicle v ON s.vehicle_id = v.id
INNER JOIN extra e ON v.version_id = e.id
GROUP BY e.name
ORDER BY num_sells;
```