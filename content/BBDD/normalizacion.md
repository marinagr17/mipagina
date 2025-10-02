+++
title = "Normalización de Bases de Datos"
date = "2025-10-02"
+++

La normalización es un proceso para organizar las tablas y relaciones en una base de datos con el objetivo de:

- Evitar redundancia de datos.
- Mejorar la integridad de la información.
- Facilitar las consultas y el mantenimiento.

## Formas normales principales

1. **Primera Forma Normal (1FN)**  
   - Cada columna contiene valores atómicos.  
   - No se permiten listas o conjuntos de valores en una sola celda.

2. **Segunda Forma Normal (2FN)**  
   - Cumple 1FN.  
   - Todas las columnas dependen completamente de la clave primaria.

3. **Tercera Forma Normal (3FN)**  
   - Cumple 2FN.  
   - No existen dependencias transitivas entre columnas.

```sql
-- Ejemplo simple de 1FN
CREATE TABLE empleados (
    id INT PRIMARY KEY,
    nombre VARCHAR(25),
    apellido1 VARCHAR(25),
    apellido2 VARCHAR(25),
    telefono VARCHAR(20)
);
