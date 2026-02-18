# Bloque 3: UI y Persitencia
## Prompting: Reporte OO UI Base

### Contexto (C)
* **Ambiente:** S/4HANA 2020.
* **Ubicación:** Bloque 3A-1 (Interfaz de Usuario base).
* **Dependencias:** Clases e interfaces del Bloque 2 (Processor + Contratos) ya existentes.
* **Fuente de Datos:** Tabla simulada `ZBP_BUT000_SIM`.
* **Alcance:** * Reporte orientado a objetos (OO).
    * Resolución de UI, ejecución y filtros en memoria.
    * **Exclusiones:** No se implementa SALV/ALV (Bloque 3A-2), ni procesos de actualización (*updates*) o logs.

---

### Objetivo (O)
Generar un reporte OO desde cero que cumpla con:

1.  **Pantalla de Selección:**
    * `SELECT-OPTIONS`: Rango de BP referenciando `ZBP_BUT000_SIM-PARTNER`.
    * `Listbox`: Tipo de BP (Valores: `ALL` - Todos, `PER` - Persona, `ORG` - Organización).
    * `Checkbox`: "Modo simulación" (Default: `'X'`).
    * `Checkbox`: "Solo con errores" (Default: `SPACE`).
2.  **Integración con Processor:**
    * Construir estructura de selección compatible con el contrato (`ty_sel`).
    * Extender el tipo del contrato y ajustar el processor si faltan campos (`bp_type`, `is_simulation`, `only_errors`).
3.  **Lógica de Filtros (En Memoria):**
    * **Sin sentencias `SELECT` adicionales** en el reporte.
    * **Solo con errores:** Filtrar sobre `tt_alv` donde `errors > 0` o `status = 'R'`.
    * **Tipo BP:** Filtrar por campo `bp_type` en la salida.
4.  **Salida:** Placeholder final con conteo de registros procesados.

---

### Sintaxis y Estándares (S)
* **Compatibilidad:** ABAP 7.4+ (S/4HANA 2020).
* **Estructura:** Clase local `lcl_app` definida como `FINAL`.
* **Modularización:** Prohibido el uso de `FORM` / `PERFORM`.
* **Eventos:**
    * `AT SELECTION-SCREEN OUTPUT`: Llamada a método estático para `VRM_SET_VALUES`.
    * `START-OF-SELECTION`: Única instrucción `lcl_app=>run( ).`.
* **Clean Code:** Métodos pequeños y uso de *Early Return*.

---

### Arquitectura y Tipos (T / A)

#### Definición de Tipos
* Uso de `ty_sel` del contrato del processor.
* `bp_type`: Tipo `C` de longitud 3.
* `tt_alv`: Debe contener al menos `partner`, `bp_type`, `status` y `errors`.

#### Estructura de lcl_app
La clase debe orquestar la UI y la ejecución mínima sin consultar la base de datos para filtros.

| Método | Tipo | Descripción |
| :--- | :--- | :--- |
| `run` | Static | Punto de entrada principal. |
| `setup_listbox` | Static | Configuración de valores del Listbox. |
| `constructor` | Instance | Importa `is_sel TYPE <ty_sel>`. |
| `execute` | Instance | Lógica de ejecución del processor. |
| `apply_filters` | Instance | Orquestador de filtros (CHANGING `ct_alv`). |
| `filter_errors_only` | Instance | Lógica para registros con error. |
| `filter_by_bp_type` | Instance | Lógica para filtrado por tipo. |

> **Nota de implementación:** Para filtrar, preferir expresiones `VALUE #( FOR ... )` o el operador `FILTER` si las llaves de tabla lo permiten.

---

### Restricciones (R)
* **No SELECT:** El reporte no debe realizar consultas SQL para filtrado.
* **Encapsulamiento:** No crear clases externas a `lcl_app`.
* **Simplicidad:** Mantener el código compilable y sin implementación de SALV en esta fase.
