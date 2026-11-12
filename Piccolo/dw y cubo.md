# Proyecto Data Warehouse & Cubo OLAP - Supermercado

Este repositorio documenta el desarrollo, modelado, limpieza de datos y conexión de un Data Warehouse analítico para un supermercado, utilizando **Supabase (PostgreSQL)** como base de datos y **Cube** como plataforma de modelado OLAP.

---

## Tabla de Contenidos
1. [Modelo del Data Warehouse (Star Schema)](#1-modelo-del-data-warehouse-star-schema)
2. [Instalación de Node.js y Configuración de Cube](#2-instalación-de-nodejs-y-configuración-de-cube)
3. [Conexión de Cube con Supabase](#3-conexión-de-cube-con-supabase)
4. [Estructura y Limpieza de las Tablas de Hechos (Blindaje con Constraints)](#4-estructura-y-limpieza-de-las-tablas-de-hechos-blindaje-con-constraints)
5. [Limpieza y Sincronización de Dimensiones (Productos y Proveedores)](#5-limpieza-y-sincronización-de-dimensiones-productos-y-proveedores)
6. [Joins y Consultas Clave del DW](#6-joins-y-consultas-clave-del-dw)

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

---

## 2. Instalación de Node.js y Configuración de Cube

Para levantar el servidor de métricas y el cubo OLAP, se utilizó **Node.js** como entorno de ejecución y la herramienta de Cube CLI.

1. **Instalación de Node.js:** Se descargó e instaló la versión LTS oficial de Node.js en el entorno local (Windows con VS Code y PowerShell).
2. **Inicialización del proyecto Cube:**
   ```bash
   npx cubejs-cli init cubodatos -d postgres
