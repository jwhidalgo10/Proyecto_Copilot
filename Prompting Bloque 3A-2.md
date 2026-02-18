# Bloque 3: UI y Persistencia
## Prompting: Refactor Clase Global ALV

### Contexto (C)
----------------------------------------
* **Situación:** En la entrega previa, la lógica de UI y `SALV` se implementó dentro del reporte `ZBP_AUDIT_REPORT` (clase `lcl_app`). Esto genera acoplamiento y dificulta la reutilización.
* **Estado del Processor:** El reporte ya obtiene `mt_alv` y `mt_issues` desde el processor; los filtros de usuario ya se aplican en memoria.
* **Requerimiento Visual:** El ALV debe mostrar la relación entre el Business Partner y sus hallazgos con las columnas: `BP | Tipo | Campo | Valor Actual | Valor Corregido | Estado` (con semáforos).
* **Alcance:** No se implementa persistencia ni gestión de logs en este bloque (3A-2).

### Objetivo (O)
----------------------------------------
Rehacer la solución para externalizar la visualización:
1.  **Limpieza de lcl_app:** El reporte y su clase local no deben contener lógica de `SALV/ALV`, actuando solo como orquestador.
2.  **Clase Global Dedicada:** Crear una clase global (ej. `ZCL_BP_ALV_BUILDER`) responsable de:
    * Transformar `tt_issue` + metadatos a "UI rows".
    * Construir y mostrar `CL_SALV_TABLE`.
    * Configurar columnas, cabeceras y semáforos.
3.  **Flujo de Control:** `lcl_app->display_results` solo debe invocar a esta nueva clase y gestionar el mensaje de "No hay datos" si las tablas están vacías.

### Sintaxis / Estándar (S)
----------------------------------------
* **Versión:** ABAP OO 7.4+ compatible con S/4HANA 2020.
* **Definición:** Clase global `FINAL`, `PUBLIC`, sin acceso a base de datos.
* **Visualización:** Uso exclusivo de la API de `CL_SALV_TABLE`.
* **Clean Code:** Métodos pequeños, legibles y sin uso de `FORM/PERFORM`.
* **Excepciones:** Manejo robusto de `CX_SALV_MSG` (ya sea capturado internamente o propagado según diseño).

### Tipos / Contratos (T)
----------------------------------------
* **Single Source of Truth:** No inventar tipos duplicados; usar los contratos del processor.
* **Estructura de Fila UI:** Definir un tipo de fila específico para el ALV que incluya:
    * `partner`, `bp_type`, `field_name`, `current_value`, `proposed_value`, `status_icon`.
* **Consumo de Parámetros:** La clase ALV recibirá:
    * `it_alv_summary`: Tabla de resumen post-filtros.
    * `it_issues`: Tabla de hallazgos completa.
* **Performance:** La clase debe filtrar los `issues` para procesar únicamente los `partners` presentes en `it_alv_summary`.

### Arquitectura (A)
----------------------------------------
Crear una clase global **"ALV Presenter/Builder"** con las siguientes responsabilidades:

| Método | Responsabilidad |
| :--- | :--- |
| `display` | Método principal (entrada de tablas). |
| `build_ui_rows` | Mapeo de `issues` a filas de visualización. |
| `map_severity_to_status` | Lógica para determinar el color del semáforo. |
| `map_status_to_icon` | Conversión de estados técnicos a iconos SAP. |
| `configure_salv_columns` | Definición de textos, visibilidad y formato de celdas. |

**Refactor en `lcl_app`:**
* Eliminar tipos `ty_ui_row/tt_ui_row`.
* Eliminar métodos `build_ui_rows`, `display_salv` y mapeos de iconos.
* Mantener únicamente: Llamada al processor, filtros en memoria y disparo del ALV global.

### Restricciones (R)
----------------------------------------
* **No DB:** Prohibido realizar `SELECT` en la clase ALV o el reporte para construir la UI.
* **UI Estricta:** El ALV debe coincidir exactamente con las columnas requeridas.
* **Semáforos:** El estado debe ser visual (icono verde/amarillo/rojo).
* **Simplicidad:** Todo el componente visual debe residir en una única clase global nueva.
