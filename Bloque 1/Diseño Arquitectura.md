# Diseño Conceptual de la Arquitectura Orientada a Objetos

### Componentes Principales del Sistema

#### 1. Componente de Lectura de Datos
- **Responsabilidad:** Leer registros de la tabla `BUT000` según filtros definidos.
- **Clase:** `BPDataReader`
- **Propósito:** Proveer datos limpios y estructurados al resto del sistema.

#### 2. Componente de Validación
- **Responsabilidad:** Validar los datos de los Business Partners según reglas específicas.
- **Clases:**
  - `BPValidator` (orquestador de validaciones).
  - `ValidationRule` (interfaz para reglas de validación).
  - Implementaciones concretas como `NITValidation`, `DPIValidation`, `EmailValidation`, etc.
- **Propósito:** Aplicar reglas de validación específicas según el tipo de BP y tipo de dato.

#### 3. Componente de Orquestación
- **Responsabilidad:** Coordinar el flujo del programa (lectura, validación, clasificación, corrección, actualización).
- **Clase:** `BPProcessCoordinator`
- **Propósito:** Actuar como el controlador principal del sistema.

#### 4. Componente de Actualización
- **Responsabilidad:** Aplicar correcciones a los datos utilizando BAPIs estándar.
- **Clase:** `BPDataUpdater`
- **Propósito:** Garantizar actualizaciones seguras y controladas.

#### 5. Componente de Logging/Auditoría
- **Responsabilidad:** Registrar cambios realizados y generar logs de auditoría.
- **Clase:** `BPAuditLogger`
- **Propósito:** Mantener un historial de modificaciones para trazabilidad.

---

### Aplicación de Principios SOLID

#### 1. SRP (Single Responsibility Principle)
- Cada clase tiene una única responsabilidad (lectura, validación, orquestación, etc.).
- Ejemplo: `BPDataReader` solo se encarga de leer datos, no de validarlos.

#### 2. OCP (Open/Closed Principle)
- Nuevas validaciones pueden agregarse creando nuevas implementaciones de `ValidationRule` sin modificar el core (`BPValidator`).
- Ejemplo: Si se requiere validar un nuevo campo, se crea una nueva clase de validación.

#### 3. DIP (Dependency Inversion Principle)
- Las clases dependen de interfaces, no de implementaciones concretas.
- Ejemplo: `BPValidator` utiliza la interfaz `ValidationRule` para ejecutar validaciones, sin conocer las clases específicas.

---

### Patrón para Validaciones

- **Patrón Estrategia:**
  - Cada regla de validación es una estrategia concreta que implementa la interfaz `ValidationRule`.
  - `BPValidator` selecciona y aplica las estrategias según el tipo de BP (Persona u Organización).

- **Fábrica:**
  - Una fábrica (`ValidationFactory`) se encarga de instanciar las reglas de validación necesarias según el tipo de BP.

---

### Flujo de Comunicación entre Componentes

1. `BPProcessCoordinator` inicia el proceso.
2. `BPDataReader` lee los datos de `BUT000`.
3. `BPValidator` aplica las validaciones utilizando reglas específicas.
4. Los resultados se clasifican (Correcto, Advertencia, Error).
5. Si se ejecuta en modo actualización:
   1. `BPDataUpdater` aplica las correcciones aceptadas.
   2. `BPAuditLogger` registra los cambios realizados.
6. Se genera un reporte final con los resultados.

---

### Diagrama Simple en Texto (ASCII)

```txt
+----------------------+
| BPProcessCoordinator |
+----------------------+
           |
           v
+-------------------+
|   BPDataReader    |
+-------------------+
           |
           v
+-------------------+
|    BPValidator    |
+-------------------+
           |
           v
+-------------------+
| BPDataUpdater (op)|
+-------------------+
           |
           v
+-------------------+
|  BPAuditLogger    |
+-------------------+


