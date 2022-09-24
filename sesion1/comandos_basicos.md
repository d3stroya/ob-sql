## ¿Qué es una base de datos?
Mecanismo de persistencia que permite almacenar la información en el tiempo.

Pueden ser SQL o no SQL. Nos centraremos en MySQL y PostgreSQL.

## PostgreSQL
1. Instalar: https://www.postgresql.org/download/
2. El instalador instala postgresql en el puerto 5432
3. Incorpora pgAdmin.


## MySQL
1. Descargar instalador: https://dev.mysql.com/downloads/windows/installer/8.0.html
2. Instalar Server y Workbench

## Herramientas GUI
* pgAdmin
* MySQL Workbench
* DBeaver
* DataGrip (de pago)

# Sentencias DDL (Data Definition Language)
Sentencias para crear bases de datos. Permiten crear la estructura de la BD.

## Cómo crear una base de datos
```sql
-- Tipos de datos:
/*
int: números enteros
numeric: números decimales
boolean: verdadero o falso
char
varchar(número de caracteres)
text
date: fechas
time: tiempo
*/

-- Identificador
/* 
id SERIAL: crea series de números incrementales (1, 2, 3,...).
*/

-- Primary key
/*
PRIMARY KEY

Primary key impide crear id duplicados
INSERT INTO employees (id, name, married, genre, salary, birth_date, start_at, end_at) 
VALUES (2, 'Bridget', True, 'F', 2743.21, '1977-04-23', '09:00:00', '15:00:00'); 
*/

-- Hacer que un campo sea obligatorio 
/*
NOT NULL

Impide que un empleado no tenga nombre
INSERT INTO employees (married, genre, salary, birth_date, start_at, end_at) 
VALUES (True, 'F', 2486.93, '1987-05-02', '09:00:00', '15:00:00');
*/

-- Hacer que un campo sea único
/*
UNIQUE

Impide que haya un campo duplicado
INSERT INTO employees (name, email, married, genre, salary, birth_date, start_at, end_at) 
VALUES ('Franky', 'f.doyle@ww.au', True, 'F', 2486.93, '1987-05-02', '09:00:00', '15:00:00');

INSERT INTO employees (name, email, married, genre, salary, birth_date, start_at, end_at) 
VALUES ('Bridget', 'f.doyle@ww.au', True, 'F', 2743.21, '1977-04-23', '09:00:00', '15:00:00');
*/

-- Restricción en rangos de datos
/*
CHECK (condición: columna - operador relacional - valor)

Verificar que el salario es mayor que 0
salary NUMERIC(9,2) CHECK (salary > 0)

Impide crear salarios menores al valor indicado
INSERT INTO employees (name, email, married, genre, salary, birth_date, start_at, end_at) 
VALUES ('Franky', 'f.d@ww.au', True, 'F', 926.93, '1987-05-02', '09:00:00', '15:00:00');
*/

-- Creación de tablas: CREATE TABLE [IF NOT EXISTS] tabla (columnas)
CREATE TABLE IF NOT EXISTS employees(
    id SERIAL PRIMARY KEY, 
    name VARCHAR(250) NOT NULL,
    email VARCHAR(100) UNIQUE,
    married BOOLEAN,
    genre CHAR(1),
    salary NUMERIC(9,2) CHECK (salary > 1100),
    birth_date DATE,
    start_at TIME,
    end_at TIME
);

-- Insertar datos: INSERT INTO table (columnas) VALUES (valores)
INSERT INTO employees (name, email, married, genre, salary, birth_date, start_at, end_at) 
VALUES ('Franky', 'f.doyle@ww.au', True, 'F', 2486.93, '1987-05-02', '09:00:00', '15:00:00');

INSERT INTO employees (name, email, married, genre, salary, birth_date, start_at, end_at) 
VALUES ('Bridget', 'b.westfall@ww.au', True, 'F', 2743.21, '1977-04-23', '09:00:00', '15:00:00');

-- Renombrar tabla: ALTER TABLE tabla operación nuevo_nombre
ALTER TABLE IF EXISTS employees RENAME TO employees_2021

-- Agregar columnas a las tablas: ALTER TABLE tabla ADD COLUMN columna
ALTER TABLE employees ADD COLUMN email VARCHAR(100);

-- Ver datos de una tabla: SELECT columna FROM tabla --> * indica todas las columnas
SELECT * FROM employees;
SELECT name FROM employees_2021;

-- Borrar columna: ALTER TABLE [IF EXISTS] tabla DROP COLUMN columna
ALTER TABLE employees DROP COLUMN IF EXISTS email

-- Borrar tabla: DROP TABLE IF EXISTS tabla
DROP TABLE IF EXISTS employees;
```

## Cargar base de datos pagila (PostgreSQL)
1. Crear base de datos de prueba: pagila
   
```sql 
CREATE DATABASE pagila; 
```

2. Agregar path a variables del entorno si no funciona psql en cmd

3. En la cmd ejecutar el sql para crear el esquema:
```
psql -h localhost -p 5432 -U postgres -d pagila < pagila-schema.sql
```
4. En la cmd ejecutar el sql para poblar el esquema:
```
psql -h localhost -p 5432 -U postgres -d pagila < pagila-data.sql
```