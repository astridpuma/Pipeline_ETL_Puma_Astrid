# Pipeline ETL en Power BI — RetailPro

Pipeline de extracción, limpieza y transformación de datos de ventas construido en **Power Query** (Power BI), con las decisiones de calidad documentadas en **lenguaje M**.

Proyecto del curso de Data Analytics de Coderhouse (Módulo 6). Este pipeline es la base del modelo analítico y los dashboards de los módulos siguientes.

## 🎯 Objetivo

Transformar datos crudos de ventas (con duplicados, nulos, inconsistencias de texto y tipos incorrectos) en un modelo limpio, con nomenclatura profesional y listo para el análisis, justificando cada decisión de limpieza.

## 🗂️ Dataset

`RetailPro_ETL_Dataset.xlsx`, provisto por el curso, con 6 tablas de datos y un diccionario de datos que define, para cada campo, su tipo esperado y si acepta valores nulos.

Se conectaron las **tablas de Excel** (`tbl...`) en lugar de las hojas, porque tienen un rango definido: si alguien agrega contenido suelto en la hoja, el pipeline no se rompe. El diccionario se usó como **criterio de limpieza** y no se cargó al modelo, porque describe los datos, no es un dato de negocio.

## 🔄 Resultado

| Consulta | Filas antes | Filas después | Cambio |
|---|---|---|---|
| `Dim_Clientes` | 244 | 240 | 4 duplicados eliminados |
| `Dim_Productos` | 47 | 45 | 2 duplicados eliminados |
| `Fact_Ventas` | 4.847 | 4.819 | 28 duplicados eliminados |
| `Dim_Categorias` | 7 | 7 | Datos limpios |
| `Dim_Territorios` | 8 | 8 | Datos limpios |

## 🧠 Decisiones de limpieza

El criterio general: **un nulo no es un cero, y borrar una fila borra todo lo que esa fila sabía.** Antes de decidir, se respondieron tres preguntas: ¿el campo es crítico para los cálculos?, ¿se puede recuperar el valor real?, ¿qué se rompe con cada opción?

| Problema | Decisión | Justificación |
|---|---|---|
| IDs duplicados | Quitar duplicados por la columna de ID | La clave primaria debe ser única; un duplicado en una dimensión multiplica las filas al combinar y infla los ingresos |
| Nulos en texto (`email`, `ciudad`, `segmento`, `canal`) | Reemplazar por "Sin dato" | Campos opcionales según el diccionario; se conserva el registro y sus ventas, y el valor explícito evita que desaparezcan de los filtros |
| Nulos en fechas (`fecha_registro`) | Mantener como nulo | "Sin dato" rompería el tipo Fecha, e inventar una fecha distorsionaría el análisis temporal |
| Nulos en `costo` y `stock` | Mantener como nulo | El diccionario los acepta; 0 significaría "sin costo" o "agotado", y null significa "sin información" |
| Nulos en `total_venta` y `hora_venta` | Mantener como nulo | Aceptados por el diccionario. `total_venta` es recuperable a partir de cantidad, precio unitario y descuento; su reconstrucción queda para la etapa de modelado |
| Mayúsculas inconsistentes (`segmento`, `canal`, `activo`) | Estandarizar valores | "ONLINE", "online" y "Online" se contarían como canales distintos en los gráficos |
| `activo` como texto ("Sí" / "SI" / "No") | Estandarizar → traducir a true/false → tipo lógico | Orden **limpiar → estandarizar → tipar**: cambiar el tipo antes de limpiar genera errores |
| Fechas en formato día/mes/año | Tipar con configuración regional Español | Sin la configuración regional, "4/1/2025" se leería como 1 de abril |

## 🔗 Enriquecimiento de `Fact_Ventas`

Dos **Merge** de tipo **LeftOuter** (externa izquierda):
1. Con `Dim_Productos`, por `id_producto` → `nombre_producto`
2. Con `Dim_Categorias`, por `id_categoria` → `nombre_categoria`

Se usó LeftOuter para **conservar todas las ventas**, aunque alguna referencie un producto que no esté en el catálogo. Con un join interno, esas ventas desaparecerían y los ingresos quedarían subestimados.

## 📝 Documentación en M

Las consultas `Fact_Ventas` y `Dim_Productos` incluyen comentarios (`//`) en el Editor Avanzado que explican el razonamiento detrás de cada transformación.

## ⚠️ Pendientes para el modelado

- **Márgenes:** en operaciones aritméticas como `precio − costo`, DAX trata el nulo como 0, lo que mostraría márgenes falsos del 100%. Los cálculos de margen deben excluir los productos sin costo registrado.
- **`total_venta`:** reconstruir los 9 valores nulos a partir de cantidad, precio unitario y descuento.

## ▶️ Cómo usarlo

1. Descarga el archivo `.pbix` y el dataset.
2. Abre el `.pbix` en Power BI Desktop.
3. Como Power BI guarda la ruta local del Excel, actualízala en **Inicio → Transformar datos → Configuración de origen de datos**.

## 🛠️ Herramientas

Power BI Desktop · Power Query · Lenguaje M
