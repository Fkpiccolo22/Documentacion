# Proyecto Data Warehouse & Cubo OLAP - Supermercado

Este repositorio documenta el desarrollo, modelado, limpieza de datos y conexión de un Data Warehouse analítico para un supermercado, utilizando **Supabase (PostgreSQL)** como base de datos y **Cube** como plataforma de modelado OLAP.

---

## Tabla de Contenidos
1. [Modelo del Data Warehouse (Star Schema)](#1-modelo-del-data-warehouse-star-schema)
2. [Instalación de Node.js y Configuración de Cube](#2-instalación-de-nodejs-y-configuración-de-cube)
3. [Conexión de Cube con Supabase](#3-conexión-de-cube-con-supabase)
4. [Configuración de Relaciones y Joins del Modelo (Cubo OLAP)](#4-configuración-de-relaciones-y-joins-del-modelo-cubo-olap)
5. [Estructura y Limpieza de las Tablas de Hechos (Blindaje con Constraints)](#5-estructura-y-limpieza-de-las-tablas-de-hechos-blindaje-con-constraints)

---

## 1. Modelo del Data Warehouse (Star Schema)

El proyecto está diseñado bajo un esquema de estrella (*Star Schema*) adaptado para la lógica de un **supermercado**, optimizado para consultas analíticas y procesamiento OLAP.

* **Tablas de Hechos (Fact Tables):**
  * `dw.fact_ventas`: Registra las transacciones de venta detalladas por ticket, producto, tiempo, sucursal, empleado y cliente.
  * `dw.fact_inventario`: Almacena la fotografía diaria del stock (actual, mínimo y máximo) por producto y sucursal.
* **Tablas de Dimensiones (Dimension Tables):**
  * `dw.dim_producto`: Contiene los productos, sus SKUs, precios unitarios, categorías y su relación con proveedores.
  * `dw.dim_proveedor`: Catálogo de proveedores del supermercado.
  * `dw.dim_sucursal`: Sucursales de la cadena.
  * `dw.dim_cliente`: Información de los clientes.
  * `dw.dim_empleado`: Personal de caja y ventas.
  * `dw.dim_tiempo`: Dimensión de tiempo estructurada para análisis por fecha.

<img width="1360" height="676" alt="image" src="https://github.com/user-attachments/assets/768a6430-1b0d-4bd5-a3b9-12b4c5bd1b02" />

---

## 2. Instalación de Node.js y configuración de Cube

Para levantar el servidor de métricas y el cubo OLAP, se utilizó **Node.js** como entorno de ejecución y la herramienta de Cube CLI.

1. **Instalación de Node.js:** Se descargó e instaló la versión LTS oficial de Node.js en el entorno local (Windows con VS Code y PowerShell).
2. **Inicialización del proyecto Cube:**
   ```bash
   npx cubejs-cli init cubodatos -d postgres

Esto generó la estructura base del proyecto Cube conectada a un driver de PostgreSQL.

<img width="1222" height="452" alt="image" src="https://github.com/user-attachments/assets/cd911259-771f-404d-85aa-4706a0221a07" />

---

## 3. Conexión de Cube con Supabase

Para conectar Cube con la base de datos alojada en la nube de Supabase, se configuraron las variables de entorno en el archivo .env:

CUBEJS_DB_TYPE=postgres
CUBEJS_DB_HOST=db.xxxxxx.supabase.co
CUBEJS_DB_NAME=postgres
CUBEJS_DB_USER=postgres
CUBEJS_DB_PASS=tu_contraseña_de_supabase
CUBEJS_DB_PORT=5432
CUBEJS_DB_SSL=true
CUBEJS_API_SECRET=tu_api_secret_generado

<img width="718" height="339" alt="image" src="https://github.com/user-attachments/assets/b45f0598-9c5c-47c2-9d3f-07c5b87ea6a2" />

<img width="1366" height="687" alt="image" src="https://github.com/user-attachments/assets/7d4684e3-6f77-48fc-9332-af8ab170fed8" />

---

## 4. Configuración de relaciones y joins del modelo cubo OLAP 

Para que Cube pudiera interpretar correctamente cómo se conectan las tablas de hechos con las dimensiones en el esquema de estrella, se configuraron los joins en Visual Studio Code. Esto permite que las métricas de ventas e inventario se crucen de forma automática con las dimensiones de productos, proveedores, sucursales y tiempo.

Ejemplo de la estructura de relaciones (joins) configurada en los cubos:
cube(`FactVentas`, {
  sql: `SELECT * FROM dw.fact_ventas`,

  joins: {
    DimProducto: {
      relationship: `belongsTo`,
      sql: `${CUBE}.id_producto = ${DimProducto}.id_producto`
    },
    DimSucursal: {
      relationship: `belongsTo`,
      sql: `${CUBE}.id_sucursal = ${DimSucursal}.id_sucursal`
    },
    DimTiempo: {
      relationship: `belongsTo`,
      sql: `${CUBE}.id_tiempo = ${DimTiempo}.id_tiempo`
    },
    DimCliente: {
      relationship: `belongsTo`,
      sql: `${CUBE}.id_cliente = ${DimCliente}.id_cliente`
    },
    DimEmpleado: {
      relationship: `belongsTo`,
      sql: `${CUBE}.id_empleado = ${DimEmpleado}.id_empleado`
    }
  },

  measures: {
    totalVentas: {
      sql: `monto_total`,
      type: `sum`
    },
    cantidadVendida: {
      sql: `cantidad_vendida`,
      type: `sum`
    }
  }
});

<img width="975" height="723" alt="image" src="https://github.com/user-attachments/assets/c858eb50-4cfe-49ab-908a-75159628d237" />

<img width="975" height="693" alt="image" src="https://github.com/user-attachments/assets/1a97f2c5-c1ab-4e73-8689-927aae1fb9a1" />

<img width="895" height="703" alt="image" src="https://github.com/user-attachments/assets/6e41c8e9-4ab1-427a-86a4-d75fd416f2c0" />


---


## 5. Estructura y limpieza de las tablas de hechos (Blindaje con Constraints)

Para evitar duplicidad de registros y asegurar la integridad de la granularidad en las tablas de hechos, se implementaron restricciones únicas (UNIQUE CONSTRAINTS) directamente en Supabase.

-- Verificar duplicados previos
SELECT id_producto, id_tiempo, id_sucursal, COUNT(*) as total
FROM dw.fact_inventario
GROUP BY id_producto, id_tiempo, id_sucursal
HAVING COUNT(*) > 1;

-- Aplicar restricción única
ALTER TABLE dw.fact_inventario
ADD CONSTRAINT unique_producto_tiempo_sucursal UNIQUE (id_producto, id_tiempo, id_sucursal);

<img width="1046" height="417" alt="image" src="https://github.com/user-attachments/assets/e7ae3f16-bcac-4e61-9d37-d5ff59eabc30" />

-- Verificar duplicados previos
SELECT id_venta, id_producto, COUNT(*) as total
FROM dw.fact_ventas
GROUP BY id_venta, id_producto
HAVING COUNT(*) > 1;

-- Aplicar restricción única
ALTER TABLE dw.fact_ventas
ADD CONSTRAINT unique_venta_producto UNIQUE (id_venta, id_producto);

<img width="1059" height="392" alt="image" src="https://github.com/user-attachments/assets/36c97c68-df3a-4202-a96c-2f4d83db20fc" />

---




