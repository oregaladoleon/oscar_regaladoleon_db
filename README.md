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

### 2. Diagrama de Entidad-Relación:
El diagrama Entidad-Relación asociado a los requisitos establecidos es el siguiente.  

![ALT diagrama_ER](ER_Ejercicio_1_BBDD.drawio.svg)

Figura 1. Modelo Entidad-Relación práctica SQL propuesta.

