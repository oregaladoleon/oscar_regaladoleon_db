# Práctica SQL
  
> Autor: Óscar Regalado León.  
> Fecha: Abril 2026.  
> Módulo: BAE.
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

![diagrama_MR](MR_Ejercicio_1_BBDD.drawio.svg)

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

