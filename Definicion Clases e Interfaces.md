# Propuesta de Diseño: Clases e Interfaces

### Convención de Nombres
- **Prefijo común:** `ZCL_BP_` para clases y `ZIF_BP_` para interfaces.
- **Justificación:** Sigue la convención estándar de ABAP para clases (`ZCL_`) e interfaces (`ZIF_`), manteniendo claridad y consistencia.
- **Excepciones de dominio:** `ZCX_BP_*` para excepciones específicas del programa.
- **Paquete sugerido:** `ZFI_BP_AUDIT` (FI: Finanzas, BP: Business Partner, AUDIT: Auditoría).

---

### Lista de Clases e Interfaces

#### 1. ZCL_BP_DATA_READER
- **Responsabilidad:** Leer registros de la tabla `BUT000` según filtros definidos.
- **Entradas:** Parámetros de selección (tipo de BP, rango de fechas, etc.).
- **Salidas:** Lista de Business Partners (estructura interna).
- **Colaboradores:** Ninguno.
- **Métodos Públicos:**
  - `READ_DATA`: Leer datos desde `BUT000` aplicando filtros.
  - `GET_RESULTS`: Retornar los datos leídos.

#### 2. ZCL_BP_VALIDATOR
- **Responsabilidad:** Orquestar la validación de datos de Business Partners.
- **Entradas:** Lista de Business Partners.
- **Salidas:** Resultados de validación (Correcto/Advertencia/Error).
- **Colaboradores:** `ZIF_BP_VALIDATION_RULE` (interfaces de reglas de validación).
- **Métodos Públicos:**
  - `VALIDATE_BP`: Ejecutar validaciones para un BP.
  - `GET_RESULTS`: Retornar resultados de validación.

#### 3. ZIF_BP_VALIDATION_RULE
- **Responsabilidad:** Definir la interfaz para reglas de validación.
- **Entradas:** Datos de un BP.
- **Salidas:** Resultado de la validación (estado y mensaje).
- **Colaboradores:** Implementaciones concretas.
- **Métodos Públicos:**
  - `VALIDATE`: Validar un campo o conjunto de campos.

#### 4. ZCL_BP_VALIDATION_NIT
- **Responsabilidad:** Validar el NIT de organizaciones.
- **Entradas:** NIT de un BP.
- **Salidas:** Resultado de validación (Correcto/Error).
- **Colaboradores:** `ZIF_BP_VALIDATION_RULE`.
- **Métodos Públicos:**
  - `VALIDATE`: Validar el formato y dígito verificador del NIT.

#### 5. ZCL_BP_VALIDATION_DPI
- **Responsabilidad:** Validar el DPI de personas.
- **Entradas:** DPI de un BP.
- **Salidas:** Resultado de validación (Correcto/Error).
- **Colaboradores:** `ZIF_BP_VALIDATION_RULE`.
- **Métodos Públicos:**
  - `VALIDATE`: Validar la longitud y formato del DPI.

#### 6. ZCL_BP_VALIDATION_CONTACT
- **Responsabilidad:** Validar datos de contacto (email y teléfono).
- **Entradas:** Email y teléfono de un BP.
- **Salidas:** Resultado de validación (Correcto/Advertencia/Error).
- **Colaboradores:** `ZIF_BP_VALIDATION_RULE`.
- **Métodos Públicos:**
  - `VALIDATE_EMAIL`: Validar el formato del email.
  - `VALIDATE_PHONE`: Validar el formato del teléfono.

#### 7. ZCL_BP_PROCESS_COORDINATOR
- **Responsabilidad:** Coordinar el flujo completo del programa.
- **Entradas:** Parámetros iniciales del programa.
- **Salidas:** Resultados finales (ALV, logs, etc.).
- **Colaboradores:** `ZCL_BP_DATA_READER`, `ZCL_BP_VALIDATOR`, `ZCL_BP_RESULT_BUILDER`, `ZCL_BP_DATA_UPDATER`, `ZCL_BP_AUDIT_LOGGER`.
- **Métodos Públicos:**
  - `EXECUTE`: Ejecutar el flujo completo (lectura, validación, actualización).
  - `SET_MODE`: Configurar el modo (simulación/actualización).

#### 8. ZCL_BP_RESULT_BUILDER
- **Responsabilidad:** Construir resultados con semáforos (Correcto/Advertencia/Error).
- **Entradas:** Resultados de validación.
- **Salidas:** Estructura lista para ALV.
- **Colaboradores:** Ninguno.
- **Métodos Públicos:**
  - `BUILD_RESULTS`: Generar estructura de resultados.
  - `GET_RESULTS`: Retornar resultados construidos.

#### 9. ZCL_BP_DATA_UPDATER
- **Responsabilidad:** Actualizar datos de BP utilizando BAPIs estándar.
- **Entradas:** Correcciones aceptadas por el usuario.
- **Salidas:** Mensajes de éxito o error.
- **Colaboradores:** BAPIs estándar de SAP.
- **Métodos Públicos:**
  - `UPDATE_BP`: Aplicar correcciones a un BP.
  - `GET_MESSAGES`: Retornar mensajes de actualización.

#### 10. ZCL_BP_AUDIT_LOGGER
- **Responsabilidad:** Registrar cambios realizados en el sistema.
- **Entradas:** Datos modificados (BP, campo, valor anterior/nuevo, usuario, fecha).
- **Salidas:** Logs en SLG1 o tabla Z.
- **Colaboradores:** Ninguno.
- **Métodos Públicos:**
  - `LOG_CHANGE`: Registrar un cambio.
  - `GET_LOGS`: Retornar logs registrados.

#### 11. ZCX_BP_EXCEPTION
- **Responsabilidad:** Manejar excepciones específicas del programa.
- **Entradas:** Mensaje de error.
- **Salidas:** Excepción lanzada.
- **Colaboradores:** Ninguno.
- **Métodos Públicos:**
  - `RAISE`: Lanzar una excepción específica.

---

### Estructura del Paquete
- **Paquete:** `ZFI_BP_AUDIT`
- **Subcarpetas:**
  - **CLASSES:** Contiene todas las clases (`ZCL_BP_*`).
  - **INTERFACES:** Contiene todas las interfaces (`ZIF_BP_*`).
  - **EXCEPTIONS:** Contiene excepciones (`ZCX_BP_*`).
  - **UTILITIES:** Clases utilitarias (si aplica).
