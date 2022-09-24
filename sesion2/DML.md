```sql

-- Sentencias DML (Data Manipulation Language)

-- 1. Consultas o recuperación de datos
-- 1.1. Recuperar toda la información
SELECT * FROM employees;

-- 1.2. Recuperar una columna
SELECT id FROM employees;

-- 1.3. Obtener varios campos
SELECT id, name FROM employees;
SELECT name, email, salary FROM employees;

-- 1.4. Filtrar filas: WHERE
SELECT * FROM employees WHERE id = 2;
SELECT * FROM employees WHERE name = 'Bridget';
SELECT * FROM employees WHERE married = False;
SELECT * FROM employees WHERE birth_date = '1987-05-02';
SELECT * FROM employees WHERE married = True AND salary > 2500;

-- 2. Inserción de datos: INSERT INTO tabla[(columnas)] VALUES (valores)
INSERT INTO employees(name, email) VALUES ('Bea', 'b.smith@ww.au');
INSERT INTO employees(name, married, genre, salary, birth_date) VALUES ('Allie', false, 'F', 1954.63, '1990-12-24');
INSERT INTO employees VALUES (7, 'Tess', 't.doyle@ww.au', false, 'F', 1849.05, '2000-4-5', '10:00:00', '16:00:00');

-- 3. Actualizar datos: UPDATE tabla SET columna operador valor WHERE columna operador valor
UPDATE employees SET salary = 3600;
UPDATE employees SET genre = 'F' WHERE name = 'Allie';
UPDATE employees SET married = true WHERE id = 4 RETURNING *;
UPDATE employees SET married = false, birth_date = '1990-12-24', start_at = '09:00:00', end_at = '15:00:00' WHERE id = 4;

-- 4. Borrar datos: DELETE FROM tabla WHERE columna operador valor
DELETE FROM employees WHERE id = 6;
DELETE FROM employees WHERE salary IS NULL;
```











