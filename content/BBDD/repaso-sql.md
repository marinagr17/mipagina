+++
title = "Repaso SQL"
date = "2025-10-02"
+++

En esta entrada dejo un ejercicio práctico de repaso en SQL.

## Tabla PRODUCTOS


| Campo         | Tipo de dato       | Restricciones                         |
|---------------|-------------------|--------------------------------------|
| CodProducto   | NUMBER            | —                                    |
| Nombre        | VARCHAR2 (20)     | —                                    |
| Tipo          | VARCHAR2 (15)     | Menaje, Informática o Telefonía      |
| PrecioUnitario| NUMBER            | Entre 0 y 5000 

## Tabla CLIENTES

| Campo       | Tipo de dato       | Restricciones                                     |
|-------------|--------------------|--------------------------------------------------|
| DNI Cliente | VARCHAR2 (10)      | 8 números, un guión y una letra mayúscula        |
| Nombre      | VARCHAR2 (20)      | —                                                |
| FechaAlta   | Fecha              | Debe ser posterior a 2019                        |
| País        | VARCHAR2 (20)      | Solo España, Italia o Francia                    |

---

## Tabla VENTAS

| Campo       | Tipo de dato       | Restricciones |
|-------------|--------------------|---------------|
| CodProducto | NUMBER             | —             |
| DNI Cliente | VARCHAR2 (10)      | —             |
| FechaVenta  | Fecha              | —             |
| NumUnidades | NUMBER             | —             |

## Ejercicios de SQL

1. Realizar una consulta que muestre el nombre del último producto que compró cada cliente que ha realizado alguna compra en los últimos diez días. (0,5 puntos)

**ORACLE**

```sql
select sum(v.numunidades*p.PrecioUnitario) as imtotal
from ventas v, productos p 
where v.codproducto=p.codproducto
AND tipo='menaje';	
```

**POSTGRES**


```sql
select nombre
from productos
where codproducto in (
    select codproducto
    from ventas
    where fechaventa BETWEEN current_timestamp- INTERVAL '10 days' AND current_timestamp
);
```



2. Realizar una consulta que muestre el importe total de las compras de productos de Tipo ‘Menaje’ para cada uno de los clientes junto con el nombre de dicho cliente incluyendo aquellos que no han comprado productos de ese tipo. (0,5 puntos)

 **ORACLE**

```sql
select sum(v.numunidades*p.PrecioUnitario) as imtotal
from ventas v, productos p 
where v.codproducto=p.codproducto
AND tipo='menaje';
```

**POSTGRES**


```sql
select sum(v.numunidades*p.PrecioUnitario) as imtotal
from ventas v, productos p 
where v.codproducto=p.codproducto
AND tipo='menaje';
```


3. Realizar una vista llamada **‘Productos de Telefonía’** con los siguientes datos: Código del Producto, Nombre del Producto, Importe Total de las Ventas del Producto, Fecha de la última venta del producto y país del primer cliente que lo compró. En la vista solo deben aparecer los artículos de tipo ‘Telefonía’. (1 punto)

4. Muestra los distintos tipos de productos junto al nombre del cliente que ha comprado más unidades de ese tipo de producto en los últimos diez años. (1 punto)

5. Realiza una consulta con operadores de conjuntos que nos diga qué artículos se han vendido tanto en enero como en febrero como en marzo. (0,5 puntos)

**ORACLE y POSTGRES**

```sql
SELECT nombre 
from productos 
where codproducto in (select codproducto
                      from ventas 
                      where TO_CHAR(fechaventa,'MM')='01')
INTERSECT
SELECT nombre 
from productos 
where codproducto in (select codproducto
                      from ventas 
                      where TO_CHAR(fechaventa,'MM')='02')
INTERSECT
SELECT nombre 
from productos 
where codproducto in (select codproducto
                      from ventas 
                      where TO_CHAR(fechaventa,'MM')='03');
```
