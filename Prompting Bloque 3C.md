# Bloque 3: UI y Persistencia
## Prompting: Log Persistente de Operaciones (Auditoría Técnica)

### Contexto (C)
----------------------------------------
* **Componentes Existentes:** Processor de reglas (`tt_alv/tt_issue`), persistencia transaccional (3B) con gestión de LUW, y visualización desacoplada vía `ZCL_BP_ALV_PRESENTER`.
* **Fuente de Datos:** Tabla `ZBP_BUT000_SIM` con actualizaciones directas y resultados estructurados por campo (`tt_update_result`).
* **Necesidad:** Implementar un rastro de auditoría técnica que registre el "quién, cuándo y qué" de cada ejecución, incluyendo cambios aplicados y errores, tanto en modo real como en simulación.

### Objetivo (O)
----------------------------------------
Implementar una capa de logging persistente con las siguientes capacidades:
1.  **Trazabilidad Completa:** Registrar cada ejecución, cada Business Partner procesado y cada campo evaluado.
2.  **Detalle Técnico:** Almacenar valor anterior, valor propuesto, severidad (S/E/W) y el mensaje técnico resultante.
3.  **Independencia:** El log debe registrarse sin afectar la transacción principal y ser lo suficientemente genérico para ser reutilizable.
4.  **Soporte de Simulación:** Diferenciar ejecuciones de prueba mediante el flag `IS_SIMULATION`.

### Sintaxis / Estándar (S)
----------------------------------------
* **Plataforma:** ABAP OO para S/4HANA 2020.
* **Simplificación:** No se utilizará el estándar `SLG1` (Application Log); se opta por tablas Z a medida para reportabilidad directa.
* **Calidad:** Adherencia a *Clean ABAP*, evitando `SELECT` en bucles y manteniendo métodos de responsabilidad única.
* **Persistencia:** Inserción limpia en tablas de auditoría.

### Modelo de Datos (Tablas Z) (T)
----------------------------------------
Se requiere la creación de dos tablas de base de datos:

1.  **ZBP_AUDIT_HDR (Cabecera):**
    * `LOG_ID` (GUID/RAW16), `EXECUTION_DATE`, `EXECUTION_TIME`, `USERNAME`.
    * `IS_SIMULATION` (Flag), `TOTAL_BP`, `TOTAL_ERRORS`, `TOTAL_SUCCESS`, `TOTAL_WARNINGS`.

2.  **ZBP_AUDIT_ITM (Detalle):**
    * `LOG_ID` (FK), `PARTNER`, `FIELD_NAME`.
    * `OLD_VALUE`, `NEW_VALUE`, `RESULT` (S/W/E), `MESSAGE`, `CREATED_AT`.

### Arquitectura (A)
----------------------------------------
El diseño se basa en un servicio de auditoría inyectable:

**1. Definición de Interfaz: `ZIF_BP_AUDIT_LOGGER`**
* `start_log( is_sel )`: Inicializa la sesión de log y devuelve el GUID generado.
* `log_bp_result( log_id, it_update_results )`: Registra los resultados detallados por BP.
* `finalize_log( log_id, summary_data )`: Cierra el log calculando los totales de éxito/error.

**2. Clase Implementadora: `ZCL_BP_AUDIT_LOGGER`**
* Encargada de la generación de GUIDs y las operaciones `INSERT` en las tablas Z.
* **Importante:** No debe ejecutar `COMMIT` (la capa de persistencia 3B o el reporte principal controlan la LUW).

**3. Integración del Flujo:**
1. Iniciar Log al comenzar el proceso.
2. Ejecutar lógica de negocio/persistencia.
3. Capturar resultados y enviarlos al logger por cada BP.
4. Finalizar log tras completar el procesamiento de la lista.

### Restricciones (R)
----------------------------------------
* **Neutralidad:** El logger nunca debe ejecutar `ROLLBACK`.
* **Desacoplamiento:** No debe existir dependencia entre el Logger y el ALV (`PRESENTER`).
* **Responsabilidad:** La lógica de actualización de tablas maestras y la lógica de guardado de logs deben estar separadas.
* **Limpieza:** No usar sentencias `WRITE` o `MESSAGE` para persistir datos técnicos.
* **Límite de Objetos:** La solución debe limitarse a 1 interfaz, 1 clase y 2 tablas Z.
