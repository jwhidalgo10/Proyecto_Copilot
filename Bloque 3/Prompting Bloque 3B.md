# Bloque 3: UI y Persistencia
## Prompting: Persistencia Transaccional (Update Z)

### Contexto (C)
----------------------------------------
* **Estado del Proyecto:** El Bloque 3A ya permite ejecutar el processor, filtrar resultados y visualizar el ALV mediante `ZCL_BP_ALV_PRESENTER`. 
* **Fuente de Datos:** Tabla `ZBP_BUT000_SIM` (simulación de `BUT000`).
* **Regla de Negocio:** No se utilizarán BAPIs; la actualización se realiza mediante sentencias `UPDATE/MODIFY` directas a la tabla Z.
* **Condición:** El parámetro `is_simulation` ya está disponible en `zif_bp_processor=>ty_sel`. Se requiere implementar la ejecución real de cambios (Bloque 3B).

### Objetivo (O)
----------------------------------------
Implementar una capa de persistencia y su integración en el flujo transaccional:
1.  **Control de Simulación:** Si `is_simulation = abap_true`, no se debe ejecutar ninguna persistencia.
2.  **Actualización Real:** Si es `abap_false`, aplicar cambios en `ZBP_BUT000_SIM` basados en los `issues` que contengan un `proposed_value`.
3.  **Gestión de LUW:** * Éxito por BP: `COMMIT WORK AND WAIT`.
    * Error Crítico: `ROLLBACK WORK`.
4.  **Feedback:** Capturar y devolver mensajes de retorno por campo/BP y actualizar el estado en `mt_alv` para que el usuario vea el resultado final en el reporte.

### Sintaxis / Estándar (S)
----------------------------------------
* **Estilo:** ABAP OO 7.4+ compatible con S/4HANA 2020 (Clean ABAP).
* **Eficiencia:** Prohibido `SELECT *` y `SELECT` dentro de loops (salvo `SELECT ... FOR UPDATE` justificado por BP).
* **Robustez:** Uso de bloques `TRY...CATCH` para el manejo de excepciones.
* **Diseño:** Orientado a interfaces (Separación de contrato y clase concreta).

### Tipos / Contratos (T)
----------------------------------------
* **Reutilización:** Consumir tipos de `zif_bp_processor` (`tt_alv`, `tt_issue`, `ty_sel`).
* **Nuevo Contrato:** Definir estructura de retorno para persistencia con:
    * `partner`, `field_name`, `result` (S/W/E), `message`, `old_value`, `new_value`.
* **Lógica de Aplicación:** Solo procesar registros donde `proposed_value` sea diferente de `original_value` y no esté inicial.

### Arquitectura (A)
----------------------------------------
Crear un componente de persistencia desacoplado:

**1. Definición de Componentes:**
* **Interfaz:** `ZIF_BP_PERSISTENCE`.
* **Clase:** `ZCL_BP_TABLE_UPDATER` (implementa la interfaz).

**2. Responsabilidades del Updater:**
* **Método `apply_changes`:** Recibe la selección, el resumen y los issues.
* **Procesamiento:** Agrupar por BP, aplicar bloqueo (`SELECT FOR UPDATE`), ejecutar `UPDATE` con campos explícitos y acumular mensajes.
* **Estrategia:** Transaccional por BP (el fallo de uno no debe revertir a los demás).

**3. Integración:**
* Modificar el punto de orquestación (Processor o `lcl_app`) para decidir la llamada al updater según el flag de simulación y refrescar las tablas internas tras la persistencia.

### Restricciones (R)
----------------------------------------
* **Sin UI:** No incluir lógica de SALV o visualización en esta capa.
* **No Log:** No implementar registro de log persistente (correspondiente al Bloque 3C).
* **Integridad:** Validar la existencia del BP antes de intentar el `UPDATE`.
* **Alcance de Campos:** Solo actualizar campos soportados (`NIT`, `DPI`, `EMAIL`, `PHONE`).

### Criterios de Aceptación
----------------------------------------
* **Simulación:** Confirmar 0 actualizaciones y 0 commits en base de datos.
* **Modo Real:** Updates aplicados correctamente con manejo de `COMMIT/ROLLBACK` por registro.
* **Visibilidad:** El ALV final debe reflejar el estatus real (ej. "Actualizado con éxito" o "Error en base de datos").
