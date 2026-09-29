# Análisis de Ventas y Rendimiento - Maven Toys (Power BI)

## 📌 Descripción del Proyecto

Este proyecto consiste en el desarrollo de un panel de control interactivo en Power BI para evaluar el rendimiento comercial y operativo de la cadena de tiendas **Maven Toys**. El objetivo principal fue realizar el proceso completo de Inteligencia de Negocios (BI): desde la extracción, transformación y limpieza de datos (ETL), hasta el diseño de un modelo de datos en esquema de estrella, cálculo de métricas clave con DAX y visualización de resultados.

---

## 🛠️ Tecnologías y Herramientas Utilizadas

- **Power BI Desktop**
- **Power Query Editor** (ETL y Limpieza de datos)
- **DAX (Data Analysis Expressions)** (Creación de columnas y medidas iterativas)
- **Git / GitHub** (Control de versiones y documentación)

---

## 📊 Dataset y Exploración Inicial

El conjunto de datos de origen está compuesto por 4 archivos CSV principales:

- `sales`: 829,262 registros de transacciones.
- `stores`: 50 ubicaciones de tiendas a nivel nacional.
- `products`: Catálogo de productos con costo y precio unitario (Rango de precios: $1.99 - $34.99).
- `calendar`: Tabla de dimensión temporal para análisis dinámico.

---

## 🚀 Fases del Desarrollo

### 1. Extracción, Limpieza y Transformación de Datos (ETL)

- **Carga de fuentes:** Importación de archivos CSV a Power Query.
- **Validación de calidad:** Revisión de columnas para identificar valores nulos, vacíos y asignar tipos de datos correctos.
- **Ajuste de Configuración Regional (Locale):** Se identificó un conflicto en el formato de fechas en la tabla `calendar` (formato original `MM/DD/YYYY` vs. `DD/MM/YYYY`). Se resolvió cambiando la configuración regional de la columna a formato de EE. UU./Inglés antes de la conversión a tipo fecha.
- **Promoción de encabezados:** Asignación de la primera fila como cabecera en las tablas requeridas.
- **Enriquecimiento del Calendario:** Creación de columnas calculadas para `Inicio de Mes` e `Inicio de Semana` directamente desde las herramientas nativas de Power Query.

---

### 2. Modelado de Datos (Data Modeling)

Se diseñó un **Esquema en Estrella (Star Schema)** optimizado para asegurar un rendimiento eficiente en las consultas y un filtrado correcto:

- **Tabla de Hechos (Fact Table):** `sales`
- **Tablas de Dimensión (Dimension Tables):** `products`, `stores`, `calendar`
- **Relaciones:** Relaciones de 1 a Muchos ($1:*$) entre las dimensiones y la tabla de hechos con dirección de filtro único.
- **Jerarquías:** Se configuró la jerarquía de fecha (`Inicio de Mes` -> `Inicio de Semana` -> `Fecha`).
- **Buenas Prácticas:** Se ocultaron las llaves foráneas (`Product_ID`, `Store_ID`, `Date`) en la tabla de ventas dentro de la vista de informe para evitar su uso incorrecto en visualizaciones.

---

### 3. Lógica de Negocio y DAX

#### A. Columnas Calculadas (Vista de Tabla)

Para evaluar el modelo inicial, se crearon las siguientes columnas en la tabla `sales`:

```dax
Cost = RELATED(products[Product_Cost])
Price = RELATED(products[Product_Price])
Revenue = sales[Units] * sales[Price]
Profit = sales[Revenue] - (sales[Cost] * sales[Units])
```

#### B. Medidas Explícitas (Vista de Informe)

Métricas agregadas base:

```dax
Total Revenue = SUM(sales[Revenue])
Total Profit = SUM(sales[Profit])
Total Orders = DISTINCTCOUNT(sales[Sale_ID])
```

#### C. Optimización DAX con Iteradores (`SUMX`)

Para optimizar el tamaño del modelo en memoria RAM, se reemplazó la dependencia de las columnas calculadas mediante el uso de iteradores que evalúan fila por fila en tiempo de ejecución:

```dax
Revenue (M) = SUMX(sales, sales[Units] * RELATED(products[Product_Price]))
Profit (M) = SUMX(sales, [Revenue (M)] - (sales[Units] * RELATED(products[Product_Cost])))
```

---

## 📈 Dashboard y Visualización

El informe se estructuró de manera limpia e intuitiva con los siguientes componentes:

1. **KPI Cards & Sparklines:**
   - Órdenes Totales por Mes (Visualización general: 41,830 órdenes en contexto activo).
   - Ingresos Totales por Mes ($658,194).
   - Ganancias Totales por Mes ($180,445).
2. **Segmentador (Slicer):**
   - Filtrado interactivo por ubicación de la tienda (`Airport`, `Commercial`, `Downtown`, `Residential`).
3. **Gráfico de Barras Horizontal:**
   - Distribución de pedidos por categoría de producto (`Toys`, `Art & Crafts`, `Games`, `Sports & Outdoors`, `Electronics`).
4. **Gráfico de Líneas:**
   - Tendencia temporal de ingresos mensuales (2022 - 2023).

---

## 💡 Aprendizajes Clave

- Resolución de problemas de formato de fecha derivados de la localización/configuración regional.
- Importancia de ocultar claves primarias/foráneas para mantener un modelo limpio.
- Diferencia técnica y práctica entre usar columnas calculadas (almacenamiento en disco/RAM) e iteradores `SUMX` (cálculo al vuelo).
