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

Figura 1. Modelo Relacional práctica SQL propuesta.

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
