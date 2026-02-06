### Flujo General del Sistema de Auditoría y Corrección de Business Partners

### 1. Inicio del Programa
- El usuario ejecuta el programa y selecciona los parámetros iniciales:
  - Filtros para la selección de Business Partners (por ejemplo, rango de fechas, tipo de BP, etc.).
  - Modo de ejecución: **Simulación** o **Actualización Real**.

### 2. Lectura Selectiva de Datos
- El sistema consulta la tabla estándar `BUT000` para obtener los registros de Business Partners que cumplan con los filtros definidos.
- Se cargan únicamente los campos relevantes para las validaciones (NIT, DPI, Email, Teléfono, etc.).

### 3. Determinación del Tipo de BP
- Para cada registro leído:
  - Se identifica si el BP es una **Persona** o una **Organización** según el campo correspondiente.
  - Esta clasificación determina las reglas de validación que se aplicarán.

### 4. Aplicación de Validaciones
- Se ejecutan las validaciones específicas para cada tipo de BP:
  - **Persona**: Validar DPI, Email, Teléfono.
  - **Organización**: Validar NIT, Email, Teléfono.
- Cada validación genera un estado asociado al registro:
  - **Correcto**: Todos los datos cumplen con las reglas.
  - **Advertencia**: Datos incompletos o dudosos (por ejemplo, formato válido pero dominio de email no confiable).
  - **Error**: Datos inconsistentes o inválidos.

### 5. Clasificación de Resultados
- Los registros se agrupan en tres categorías:
  - **Correctos**: No requieren corrección.
  - **Advertencias**: Se sugieren correcciones, pero no son obligatorias.
  - **Errores**: Requieren corrección obligatoria.

### 6. Generación de Propuestas de Corrección
- Para los registros con **Advertencias** o **Errores**:
  - El sistema genera valores sugeridos basados en reglas predefinidas (por ejemplo, calcular el dígito verificador del NIT o corregir el formato del teléfono).
  - Estas propuestas se presentan al usuario para revisión.

### 7. Ejecución en Modo Simulación
- Si el programa se ejecuta en modo **Simulación**:
  - No se realizan cambios en los datos.
  - Se genera un reporte con los resultados de las validaciones y las propuestas de corrección.

### 8. Ejecución con Actualización Real
- Si el programa se ejecuta en modo **Actualización Real**:
  - El usuario puede aceptar o rechazar las correcciones propuestas.
  - Para los registros aceptados:
    - Se actualizan los datos utilizando BAPIs estándar de SAP.
    - Se registra cada cambio en la tabla de auditoría, incluyendo:
      - BP modificado.
      - Campo actualizado.
      - Valor anterior y nuevo.
      - Usuario y fecha.

### 9. Registro de Auditoría y Mensajes
- Cada cambio realizado se registra en una tabla de auditoría personalizada.
- Se generan mensajes informativos para el usuario:
  - Cambios realizados exitosamente.
  - Errores en la actualización (si los hubiera).

### 10. Generación de Resultados Finales
- Se presenta un reporte ALV con los resultados de la validación:
  - Estados visuales (**Correcto**, **Advertencia**, **Error**).
  - Detalles de las validaciones y correcciones aplicadas.
- Opcionalmente, el reporte puede exportarse a formatos como Excel o PDF.

### 11. Cierre del Programa
- El programa finaliza mostrando un resumen general:
  - Total de registros procesados.
  - Total de registros correctos, con advertencias y con errores.
  - Total de cambios realizados (en caso de actualización real).
