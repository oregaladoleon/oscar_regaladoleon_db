# Práctica SQL
  
> Autor: Óscar Regalado León.  
> Fecha: Abril 2026.  
> Módulo: BAE.
---
## Índice del informe.
1. Requisitos de la BBDD.
2. Diagrama de Entidad-Relación.
3. Diagrama Modelo Relacional.
4. Script SQL de la BBDD.
5. Resultados en psql de las entradas realizadas.
6. Introducción de datos en la BBDD.
7. Consultas realizadas a la BBDD.  
    7.1. Consulta 1. Segundo surfista por edad.   
    7.2. Consulta 2. Tiempo de experiencia del surfista.     
    7.3. Consulta 3. Tablas de los surfistas.  
    7.4. Consulta 4. Tablas suministradas por cada tienda.   
    7.5. Consulta 5. Surfista con la tabla de mayor length.   
8. Anexo: Comprobación de actualización y borrado.  
    8.1 Prueba 1. Eliminar registro surfista  
    8.2 Prueba 2. Eliminar registro tienda.  
    8.3 Prueba 3. Actualización de registro.
---
### 1. Requisitos de la BBDD.
En este informe se expone el proceso llevado a cabo para ejecutar una base de datos, en adelante BBDD, que trata sobre la colección de tablas de surf (Quiver) que posee un determinado surfista, las tablas son suministradas por tiendas locales y nacionales.

Se tendrá en cuenta que, un surfista es identificado por su **DNI** y además se conocerá su **name**, **surname**, **born** y **experience**. Un surfista puede poseer varias tablas, de las cuales, se conocerá: su **brand**, **model**, **liters**, **length** y **type**. Una tabla la puede tener uno o muchos surfistas.

De cada tabla que posee cada surfista, será interesante conocer su **status** y el **level** asociado a esa tabla y surfista.

Por otro lado, las tablas serán suministradas por una única tienda, interesa conocer la **date** de adquisición, una tienda podrá suministrar varias tablas. De las tiendas es necesario conocer, el **name** y el **address**.

### 2. Diagrama de Entidad-Relación.
El diagrama Entidad-Relación asociado a los requisitos establecidos es el siguiente.  

![diagrama_ER](ER_Ejercicio_1_BBDD.drawio.svg)

Figura 1. Modelo Entidad-Relación práctica SQL propuesta.

### 3. Diagrama Modelo Relacional.

Producto del diagrama Entidad-Relación descrito en el apartado anterior, el modelo Relacional es el siguiente.

![diagrama_MR](MR_Ejercicio_1_BBDD-2.drawio-1.svg)

Figura 2. Modelo Relacional práctica SQL propuesta.

### 4. Script SQL de la BBDD.
Se expone en el siguiente cuadro de comandos el script final de la BBDD.

~~~sql
-- 1.Tablas sin dependencias
CREATE TABLE surfista (
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    surname VARCHAR(250) NOT NULL,
    born DATE,
    start_experience DATE NOT NULL,
    dni VARCHAR(10) UNIQUE NOT NULL
);
CREATE TABLE quiver (
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    brand VARCHAR(50),
    model TEXT,
    length NUMERIC(4,2),
    liters NUMERIC(3,1),
    type VARCHAR(25),
CONSTRAINT chk_type CHECK (type IN ('shortboard', 'evolutiva', 'longboard', 'malibu', 'fish'))
);
CREATE TABLE tienda (
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(50),
    address TEXT
);
-- 2. Resto de tablas
CREATE TABLE posee (
    id_surfista INT,
    id_quiver INT,
    level VARCHAR(25),
    status VARCHAR(25),
CONSTRAINT pk_posee PRIMARY KEY (id_surfista, id_quiver),
CONSTRAINT chk_level CHECK (level IN ('beginner', 'intermediate', 'pro')),
CONSTRAINT chk_status CHECK (status IN ('new','second_hand')),
CONSTRAINT fk_id_surfista FOREIGN KEY (id_surfista)
REFERENCES surfista (id)
ON DELETE CASCADE,
CONSTRAINT fk_id_quiver FOREIGN KEY (id_quiver)
REFERENCES quiver (id)
ON DELETE CASCADE 
);
CREATE TABLE suministro (
    id_quiver INT,
    id_tienda INT,
    date DATE,
CONSTRAINT pk_suministro PRIMARY KEY (id_quiver, id_tienda),
CONSTRAINT fk_id_quiver FOREIGN KEY (id_quiver)
REFERENCES quiver (id)
ON DELETE CASCADE,
CONSTRAINT fk_id_tienda FOREIGN KEY (id_tienda)
REFERENCES tienda (id)
ON DELETE RESTRICT --No permitimos el borrado de las tablas vendidas por una tienda que queramos borrar.
);
~~~

### 5. Resultados en psql de las entradas realizadas.
Tras introducir las tablas del apartado anterior en la BBDD creada, se comprueba en consola, obteniendo los siguientes resultados.

~~~bash
surf_db=# \d
               List of relations
 Schema |      Name       |   Type   |  Owner   
--------+-----------------+----------+----------
 public | posee           | table    | postgres
 public | quiver          | table    | postgres
 public | quiver_id_seq   | sequence | postgres
 public | suministro      | table    | postgres
 public | surfista        | table    | postgres
 public | surfista_id_seq | sequence | postgres
 public | tienda          | table    | postgres
 public | tienda_id_seq   | sequence | postgres
(8 rows)
~~~

~~~bash
surf_db=# \d posee
                         Table "public.posee"
   Column    |         Type          | Collation | Nullable | Default 
-------------+-----------------------+-----------+----------+---------
 id_surfista | integer               |           | not null | 
 id_quiver   | integer               |           | not null | 
 level       | character varying(25) |           |          | 
 status      | character varying(25) |           |          | 
Indexes:
    "pk_posee" PRIMARY KEY, btree (id_surfista, id_quiver)
Check constraints:
    "chk_level" CHECK (level::text = ANY (ARRAY['beginner'::character varying, 'intermediate'::character varying, 'pro'::character varying]::text[]))
    "chk_status" CHECK (status::text = ANY (ARRAY['new'::character varying, 'second_hand'::character varying]::text[]))
Foreign-key constraints:
    "fk_id_quiver" FOREIGN KEY (id_quiver) REFERENCES quiver(id) ON DELETE CASCADE
    "fk_id_surfista" FOREIGN KEY (id_surfista) REFERENCES surfista(id) ON DELETE CASCADE
~~~

~~~bash
surf_db=# \d quiver
                                Table "public.quiver"
 Column |         Type          | Collation | Nullable |           Default            
--------+-----------------------+-----------+----------+------------------------------
 id     | integer               |           | not null | generated always as identity
 brand  | character varying(50) |           |          | 
 model  | text                  |           |          | 
 length | numeric(4,2)          |           |          | 
 liters | numeric(3,1)          |           |          | 
 type   | character varying(25) |           |          | 
Indexes:
    "quiver_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "chk_type" CHECK (type::text = ANY (ARRAY['shortboard'::character varying, 'evolutiva'::character varying, 'longboard'::character varying, 'malibu'::character varying, 'fish'::character varying]::text[]))
Referenced by:
    TABLE "posee" CONSTRAINT "fk_id_quiver" FOREIGN KEY (id_quiver) REFERENCES quiver(id) ON DELETE CASCADE
    TABLE "suministro" CONSTRAINT "fk_id_quiver" FOREIGN KEY (id_quiver) REFERENCES quiver(id) ON DELETE CASCADE
~~~

~~~bash
surf_db=# \d suministro
              Table "public.suministro"
  Column   |  Type   | Collation | Nullable | Default 
-----------+---------+-----------+----------+---------
 id_quiver | integer |           | not null | 
 id_tienda | integer |           | not null | 
 date      | date    |           |          | 
Indexes:
    "pk_suministro" PRIMARY KEY, btree (id_quiver, id_tienda)
Foreign-key constraints:
    "fk_id_quiver" FOREIGN KEY (id_quiver) REFERENCES quiver(id) ON DELETE CASCADE
    "fk_id_tienda" FOREIGN KEY (id_tienda) REFERENCES tienda(id) ON DELETE RESTRICT
~~~

~~~bash
urf_db=# \d surfista
                                     Table "public.surfista"
      Column      |          Type          | Collation | Nullable |           Default            
------------------+------------------------+-----------+----------+------------------------------
 id               | integer                |           | not null | generated always as identity
 name             | character varying(50)  |           | not null | 
 surname          | character varying(250) |           | not null | 
 born             | date                   |           |          | 
 start_experience | date                   |           | not null | 
 dni              | character varying(10)  |           | not null | 
Indexes:
    "surfista_pkey" PRIMARY KEY, btree (id)
    "surfista_dni_key" UNIQUE CONSTRAINT, btree (dni)
Referenced by:
    TABLE "posee" CONSTRAINT "fk_id_surfista" FOREIGN KEY (id_surfista) REFERENCES surfista(id) ON DELETE CASCADE
~~~

~~~bash
surf_db=# \d tienda
                                 Table "public.tienda"
 Column  |         Type          | Collation | Nullable |           Default            
---------+-----------------------+-----------+----------+------------------------------
 id      | integer               |           | not null | generated always as identity
 name    | character varying(50) |           |          | 
 address | text                  |           |          | 
Indexes:
    "tienda_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "suministro" CONSTRAINT "fk_id_tienda" FOREIGN KEY (id_tienda) REFERENCES tienda(id) ON DELETE RESTRICT
~~~

### 6. Introducción de datos en la BBDD.
Para poder realizar las consultas, se procede a introducir datos de **surfistas**, **quiver**, **tienda** y **tablas intermedias** en la BBDD.

~~~sql
INSERT INTO surfista (name, surname, born, start_experience, dni) VALUES
('Kelly', 'Slater', '1972-02-11', '1992-01-01', '123456789'),
('Robert', 'Machado', '1973-10-16', '1994-01-01', '123456711'),
('Gabriel', 'Medina', '1993-12-22', '2014-01-01', '123456722');

INSERT INTO quiver (brand, model, length, liters, type) VALUES
('Pukas', 'Dark', 6.05, 28.5, 'shortboard'),
('Lost', 'RNF', 5, 32, 'fish'),
('Torq', 'Mod Fun', 7.02, 48.0, 'evolutiva');

INSERT INTO tienda (name, address) VALUES 
('Pukas Surf Shop', 'Calle Mayor 5, San Sebastián'),
('Mundaka Surf Shop', 'Txorrokopunta Kalea 2, Mundaka'),
('Watsay Surf', 'Playa de Berria, Santoña');

INSERT INTO posee (id_surfista, id_quiver, level, status) VALUES 
(1, 1, 'pro', 'new'),
(1, 2, 'pro', 'second_hand'),
(2, 3, 'intermediate', 'second_hand');

INSERT INTO suministro (id_quiver, id_tienda, date) VALUES 
(1, 1, '2024-01-10'),
(2, 1, '2024-02-15'),
(3, 3, '2023-11-20');
~~~
> Leyenda de los datos introducidos:

| Término | Descripción |
|---------|-------------|
| Quiver | Es la colección de tablas que posee un surfista. Cada tabla está diseñada para unas condiciones de olas específicas.|
| Length | La longitud de la tabla, medida en pies y pulgadas. En la base de datos se usa formato decimal (ej: 6.05 son 6 pies y media pulgada).|
| Liters | El volumen de la tabla. A más litros, más flota la tabla y más fácil es remar.|
|Type | Categoría de la tabla. Por ejemplo: Shortboard (giros rápidos), Longboard (estabilidad/olas pequeñas) o Fish (olas con poca fuerza). |
| Experience | Fecha en la que el surfista comenzó a practicar |
| Status | Estado físico de la tabla (nueva, usada).|

Como ejemplo de muestra de los datos introducimos, podemos observar la tabla quiver:

~~~bash
urf_db=# SELECT * FROM quiver
surf_db-# ;
 id | brand |  model  | length | liters |    type    
----+-------+---------+--------+--------+------------
  1 | Pukas | Dark    |   6.05 |   28.5 | shortboard
  2 | Lost  | RNF     |   5.00 |   32.0 | fish
  3 | Torq  | Mod Fun |   7.02 |   48.0 | evolutiva
(3 rows)
~~~
### 7. Consultas realizadas a la BBDD.

#### 7.1 Consulta 1. Segundo surfista por edad.
Se quiere conocer al segundo surfista por edad, expresando el nombre completo con apellidos en una columna.

~~~sql
SELECT id, surname || ', ' || name AS "Nombre completo", born, start_experience
FROM surfista
ORDER BY  --Ordena la consulta según la columna definida born.
born
OFFSET --No muestra la primera columna.
1 ROW
FETCH  -- Muestra la 1 columna siguiente.
NEXT 1 ROW ONLY;
~~~
Como resultado hemos obtenido:
~~~bash
surf_db=# SELECT id, surname || ', ' || name AS "Nombre completo", born, start_experience
FROM 
surfista
ORDER BY
born
OFFSET
1 ROW
FETCH
NEXT 1 ROW ONLY;
 id | Nombre completo |    born    | start_experience 
----+-----------------+------------+------------------
  2 | Machado, Robert | 1973-10-16 | 1994-01-01
(1 row)
~~~
#### 7.2 Consulta 2. Tiempo de experiencia del surfista.
Calculamos el tiempo de experiencia de cada surfista, para ello emplearemos la función AGE(columna) que nos calculará el tiempo transcurrido desde la fecha marcada en la columna hasta el día de hoy:
~~~sql
SELECT 
    surname || ', ' || name AS "Surfista", -- Aplicamos alias y concatenación.
    born AS "Fecha de nacimiento", --Aplicamos Alias a la columna born
    AGE(start_experience) AS "Tiempo surfeando"
FROM surfista
ORDER BY
start_experience ASC; --Ordenamos de manera ascendente.
~~~
El resultado obtenido ha sido:
~~~bash
surf_db=# SELECT
surname || ', ' || name AS "Surfista",
born AS "Fecha de nacimiento",
AGE(start_experience) AS "Tiempo surfeando"
FROM surfista
ORDER BY
start_experience ASC;
    Surfista     | Fecha de nacimiento |    Tiempo surfeando     
-----------------+---------------------+-------------------------
 Slater, Kelly   | 1972-02-11          | 34 years 3 mons 24 days
 Machado, Robert | 1973-10-16          | 32 years 3 mons 24 days
 Medina, Gabriel | 1993-12-22          | 12 years 3 mons 24 days
(3 rows)
~~~

#### 7.3 Consulta 3. Tablas de los surfistas.
El objetivo de esta consulta es mostrar un listado donde aparezca el nombre del surfista, la marca y el modelo de las tablas que posee, ordenado por el nombre del surfista.

~~~sql
SELECT
    s.name AS "Surfista",
    q.brand AS "Marca",
    q.model AS "Modelo",
    p.level AS "Nivel de uso"
FROM
surfista s
INNER JOIN posee p ON s.id = p.id_surfista
INNER JOIN quiver q ON q.id = p.id_quiver
ORDER BY s.name ASC;
~~~
El resultado obtenido ha sido:
~~~bash
urf_db=# SELECT
    s.name AS "Surfista",
    q.brand AS "Marca",
    q.model AS "Modelo",
    p.level AS "Nivel de uso"
FROM
surfista s
INNER JOIN posee p ON s.id = p.id_surfista
INNER JOIN quiver q ON q.id = p.id_quiver
ORDER BY s.name ASC;
 Surfista | Marca | Modelo  | Nivel de uso 
----------+-------+---------+--------------
 Kelly    | Pukas | Dark    | pro
 Kelly    | Lost  | RNF     | pro
 Robert   | Torq  | Mod Fun | intermediate
(3 rows)
~~~

#### 7.4 Consulta 4. Tablas suministradas por cada tienda.
En esta consulta, pretendemos saber cuántas tablas ha suministrado cada tienda a nuestro sistema.

~~~sql
SELECT
    t.name AS "Tienda",
    COUNT(sum.id_quiver) AS "Número de tablas"
FROM tienda t
LEFT JOIN suministro sum ON t.id = sum.id_tienda
GROUP BY
t.name;
~~~
El resultado obtenido ha sido:
~~~bash
surf_db=# SELECT
    t.name AS "Tienda",
    COUNT(sum.id_quiver) AS "Número de tablas"
FROM tienda t
LEFT JOIN suministro sum ON t.id = sum.id_tienda
GROUP BY
t.name;
      Tienda       | Número de tablas 
-------------------+------------------
 Pukas Surf Shop   |                2
 Mundaka Surf Shop |                0
 Watsay Surf       |                1
(3 rows)
~~~

#### 7.5 Consulta 5. Surfista con la tabla de mayor length.
Por último, en esta consulta pretendemos encontrar el nombre del surfista que posee la tabla más larga de toda la base de datos.

~~~sql
SELECT
    s.name || ', ' || s.surname AS "Surfista",
    q.brand AS "Marca",
    q.model AS "Model",
    q.length AS "Longitud"
FROM surfista s
INNER JOIN posee p ON s.id = p.id_surfista
INNER JOIN quiver q ON q.id = p.id_quiver
WHERE q.length = (SELECT MAX(q2.length) FROM quiver q2); --Dentro de la subconsulta creamos otro alias, de no ser así genera error.
~~~
Se ha obtenido:
~~~bash
surf_db=# SELECT
    s.name || ', ' || s.surname AS "Surfista",
    q.brand AS "Marca",
    q.model AS "Model",
    q.length AS "Longitud"
FROM surfista s
INNER JOIN posee p ON s.id = p.id_surfista
INNER JOIN quiver q ON q.id = p.id_quiver
WHERE q.length = (SELECT MAX(q2.length) FROM quiver q2);
    Surfista     | Marca |  Model  | Longitud 
-----------------+-------+---------+----------
 Robert, Machado | Torq  | Mod Fun |     7.02
~~~
### 8. Anexo: Comprobación de actualización y borrado.
Para la comprobación del correcto funcionamiento de las restricciones de borrado y actualización especificadas en la creación de cada tabla de relación, se realizan las siguientes actuaciones.
#### 8.1 Prueba 1. Eliminar registro surfista
1. Seleccionamos un surfista y comprobamos sus registros.
~~~sql
SELECT * FROM surfista
WHERE id = 1;
~~~
~~~bash
surf_db=# SELECT * FROM surfista
WHERE id = 1;
 id | name  | surname |    born    | start_experience |    dni    
----+-------+---------+------------+------------------+-----------
  1 | Kelly | Slater  | 1972-02-11 | 1992-01-01       | 123456789
(1 row)
~~~
~~~sql
SELECT * FROM posee
WHERE id_surfista = 1;
~~~
~~~bash
surf_db=# SELECT * FROM posee
WHERE id_surfista = 1;
 id_surfista | id_quiver | level |   status    
-------------+-----------+-------+-------------
           1 |         1 | pro   | new
           1 |         2 | pro   | second_hand
(2 rows)
~~~
2. Realizamos el borrado.
~~~sql
DELETE FROM surfista
WHERE id = 1;
~~~
3. Comprobamos que la acción se ha ejecutado.
~~~sql
SELECT * FROM posee
WHERE id_surfista = 1;
~~~
~~~bash
surf_db=# DELETE FROM surfista
surf_db-# WHERE id = 1;
DELETE 1
surf_db=# SELECT * FROM posee
WHERE id_surfista = 1;
 id_surfista | id_quiver | level | status 
-------------+-----------+-------+--------
(0 rows)
~~~
#### 8.2 Prueba 2. Eliminar registro tienda.
En este caso, en la tabla intermedia de relación habíamos definido ON DELETE RESTRICT, por lo tanto, no nos debe permitir el borrado del registro.
1. Seleccionamos una tienda a borrar.
~~~sql
SELECT * FROM tienda
WHERE id = 1;
~~~
~~~bash
surf_db=# SELECT * FROM tienda
surf_db-# WHERE id= 1;
 id |      name       |           address            
----+-----------------+------------------------------
  1 | Pukas Surf Shop | Calle Mayor 5, San Sebastián
(1 row)
~~~
2. Realizamos la acción de borrar.
~~~sql
DELETE FROM tienda
WHERE id = 1;
~~~
~~~bash
surf_db=# DELETE FROM tienda
surf_db-# WHERE id =1;
ERROR:  update or delete on table "tienda" violates foreign key constraint "fk_id_tienda" on table "suministro"
DETAIL:  Key (id)=(1) is still referenced from table "suministro".
~~~
#### 8.3 Prueba 3. Actualización de registro.
Para esta prueba de actualización de registro, vamos a cambiar el **status** de la tabla **posee**. 
~~~sql
SELECT * FROM posee;
~~~
~~~bash
surf_db=# SELECT * FROM posee;
 id_surfista | id_quiver |    level     |   status    
-------------+-----------+--------------+-------------
           2 |         3 | intermediate | second_hand
(1 row)
~~~
~~~sql
UPDATE posee
SET status = 'new'
WHERE id_surfista = 2 AND id_quiver = 3;
~~~
~~~bash
surf_db=# UPDATE posee
surf_db-# SET status = 'new'
surf_db-# WHERE id_surfista = 2 AND id_quiver = 3;
UPDATE 1
surf_db=# SELECT * FROM posee;
 id_surfista | id_quiver |    level     | status 
-------------+-----------+--------------+--------
           2 |         3 | intermediate | new
(1 row)
~~~