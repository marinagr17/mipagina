+++
date = '2025-10-04T11:03:10+02:00'
draft = true
title = 'Repaso PL/SQL'
+++

** Nota importante: Es imprescindible la correcta división del ** 
** código en los módulos correspondientes y se valorará la legibilidad ** 
** del código y el uso del menor número posible de consultas al servidor. **

## Ejercicio 1

Realizar un procedimiento que reciba un tipo de producto, un mes y un año y muestre un listado de todas las compras que se han realizado de productos de dicho tipo en dicho mes agrupadas por país y cliente con el siguiente formato:
## Compras de Artículos de Tipo `TipodeProducto`
**Mes:** nn  **Año:** n.nnn

### País: `NombrePais`

- **Cliente:** `NombreCliente`
  - `CodProducto` | `FechaVenta` | `ImporteCompra`
  - `CodProducto` | `FechaVenta` | `ImporteCompra`
  - ...
  - `CodProducto` | `FechaVenta` | `ImporteCompra`

  **Importe Total Cliente `NombreCliente`**

- **Cliente:** `NombreCliente`
  - `CodProducto` | `FechaVenta` | `ImporteCompra`
  - ...
  
  **Importe Total Cliente `NombreCliente`**

**Total Compras `NombrePais`: nnn**

### País: `NombrePais`

- **Cliente:** `NombreCliente`
  - `CodProducto` | `FechaVenta` | `ImporteCompra`
  - ...
  
  **Importe Total Cliente `NombreCliente`**

- **Cliente:** `NombreCliente`
  - `CodProducto` | `FechaVenta` | `ImporteCompra`
  - ...
  
  **Importe Total Cliente `NombreCliente`**

**Total Compras `NombrePais`: nnn**

---

**Total Compras de Artículos de Tipo `TipodeProducto`: nn**

---

Si no existe ningún artículo de ese tipo debe generarse una excepción, así como si no existen ventas de artículos de ese tipo. Si desde un país no se ha hecho ninguna compra debe aparecer un mensaje indicándolo.

## Ejercicio 2.

Realizar un *trigger* que cuando se inserte una venta mantenga correctamente actualizada la columna ProductoMasCaroComprado (que previamente has de crear y rellenar mediante un procedimiento) en la tabla Clientes.

## Ejercicio 3.

Realiza los módulos de programación necesarios para asegurar que no se venden más de cinco productos de un mismo tipo en un mismo mes natural.
