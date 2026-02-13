# Bloque 2 - Desarrollo core
---

# Contexto General

Se desarrolla una solución orientada a objetos para validar Business Partners aplicando buenas prácticas de Clean ABAP y principios de diseño.

La arquitectura está compuesta por 6 clases:

1. ZCL_BP_READER  
2. ZCL_BP_NITVALIDATOR  
3. ZCL_BP_DPIVALIDATOR  
4. ZCL_BP_CONTACTVALIDATOR  
5. ZCL_BP_VALIDATOR  
6. ZCL_BP_PROCESSOR  

Restricciones obligatorias del diseño:

- ABAP 7.4+
- Sin SELECT *
- Early return
- Métodos pequeños y claros
- Código compilable

Reutilizar los tipos ya definidos en el dueño del modelo (clase lectora o clase central de tipos).
No redefinir ty_bp o ty_issue en cada clase.

Cada clase debe respetar SRP.

No inventar objetos nuevos.
No SELECT dentro de LOOP.
No cambiar firmas públicas existentes

---

# ZCL_BP_READER

## Objetivo
Leer Business Partners desde tabla ZBP_BUT000_SIM según rango de partner.

## Prompt

Actúa como ABAP senior en S/4HANA (7.4+).  
Crea desde cero la clase ZCL_BP_READER (DEFINITION + IMPLEMENTATION).

Requerimiento:
Existe una clase responsable únicamente de obtener datos desde la tabla simulada de BUT000.
- Entrada: ty_sel con partner_range TYPE RANGE OF bu_partner.
- Salida: tt_bp con estructura:
  - partner
  - bp_type
  - nit
  - dpi
  - email
  - phone
- Fuente: ZBP_BUT000_SIM.
- Método público:
  CLASS-METHODS read_data
    IMPORTING is_sel TYPE ty_sel
    RETURNING VALUE(rt_bp) TYPE tt_bp.

Reglas:

- Early return si rango vacío.
- Un solo SELECT INTO TABLE @rt_bp.
- WHERE partner IN @is_sel-partner_range.
- No loops innecesarios.
- Sintaxis ABAP 7.4+.
- Responsabilidad única: acceso y mapeo.

Entrega:
Clase completa lista para activar.

---

# ZCL_BP_NITVALIDATOR

## Objetivo
Validar NIT Guatemala con algoritmo módulo 11.

## Prompt

Crea desde cero la clase ZCL_BP_NITVALIDATOR.

Implementar una validación coherente y mantenible que distinga entre:
- Valor vacío
- Formato incorrecto
- Valor corregible por normalización

Interfaz:

- ty_bp: partner, nit.
- ty_issue: partner, field, severity ('E'/'W'), message.
- tt_issue: tabla estándar.
- METHODS validate
    IMPORTING is_bp TYPE ty_bp
    RETURNING VALUE(rt_issues) TYPE tt_issue.

Reglas:

- NIT vacío → WARNING.
- Normalizar: quitar espacios, puntos y guiones.
- Si cambia al normalizar → WARNING.
- Validar formato:
  - Parte principal 8 o 9 dígitos.
  - Verificador 0-9 o K.
- Validar dígito verificador (módulo 11).
- Si verificador incorrecto → ERROR.

Restricciones:

- No redefinir tipos.
- No mezclar validación con persistencia.

Entrega:
Clase completa compilable.

---

# ZCL_BP_DPIVALIDATOR

## Objetivo
Validar DPI/CUI Guatemala.

## Prompt

Crea desde cero la clase ZCL_BP_DPIVALIDATOR.

- Debe diferenciar entre:
- Vacío
- Formato inválido

Valor normalizado válido
Interfaz:

- ty_bp: partner, dpi.
- ty_issue: partner, field 'DPI', severity, message.
- tt_issue.
- METHODS validate
    IMPORTING is_bp TYPE ty_bp
    RETURNING VALUE(rt_issues) TYPE tt_issue.

Reglas:

- DPI vacío → WARNING.
- Normalizar: quitar espacios y guiones.
- Si cambia → WARNING.
- Debe tener exactamente 13 dígitos.
- Si contiene caracteres no numéricos → ERROR.

Restricciones:

- Sin regex.
- Early return.
- Métodos privados pequeños.

Entrega:
Clase completa lista para activar.

---

# ZCL_BP_CONTACTVALIDATOR

## Objetivo
Validar Email y Teléfono Guatemala.

## Prompt

Crea desde cero la clase ZCL_BP_CONTACTVALIDATOR.

Interfaz:

- ty_bp: partner, email, phone.
- ty_issue: partner, field ('EMAIL'/'PHONE'), severity, message.
- tt_issue.
- METHODS validate
    IMPORTING is_bp TYPE ty_bp
    RETURNING VALUE(rt_issues) TYPE tt_issue.

Email:

- Vacío → WARNING (definir política).
- Normalizar: trim + to_lower.
- Si cambió → WARNING.
- Exactamente un '@'.
- Dominio con al menos un '.'.
- No iniciar ni terminar con punto.
- Si falla → ERROR.

Teléfono:

- Vacío → WARNING.
- Normalizar: quitar espacios y guiones.
- Formato válido: +502 seguido de 8 dígitos.
- Si falla → ERROR.

Restricciones:

- Sin regex.
- No usar sy-tabix.
- Métodos privados claros y reutilizables.

Entrega:
Clase completa compilable.

---

# ZCL_BP_VALIDATOR (Orquestador)

## Objetivo
Orquestar validaciones según bp_type.

## Prompt

Crea desde cero la clase ZCL_BP_VALIDATOR.

Interfaz pública:

- ty_bp: partner, bp_type, nit, dpi, email, phone.
- tt_issue compatible con validadores.
- METHODS validate (instancia)
    IMPORTING is_bp TYPE ty_bp
    RETURNING VALUE(rt_issues) TYPE tt_issue.

Reglas:

- Si bp_type = 'ORG':
    validar NIT y luego contacto.
- Si bp_type = 'PER':
    validar DPI y luego contacto.
- Tipo desconocido → early return vacío.

Lazy init:

- mo_nit_validator
- mo_dpi_validator
- mo_contact_validator
- Métodos get_* que crean instancia si INITIAL.

Restricciones:

- validate NO debe ser CLASS-METHOD.
- Usar me-> correctamente.
- Usar CASE en vez de IF duplicado.
- Factorizar validación de contacto.

Entrega:
Clase completa compilable.

---

# ZCL_BP_PROCESSOR

## Objetivo
Orquestar lectura, validación y construcción de salida ALV.

## Prompt

Crea desde cero la clase ZCL_BP_PROCESSOR.

Interfaz:

- ty_sel: partner_range TYPE RANGE OF bu_partner.
- ty_alv: partner, status (1 char), errors (i), warnings (i).
- tt_alv.
- tt_issue.
- METHODS process
    IMPORTING is_sel TYPE ty_sel
    EXPORTING rt_alv TYPE tt_alv
              rt_issues TYPE tt_issue.

Implementación:

- lt_bp = ZCL_BP_READER=>read_data( is_sel ).
- Early return si vacío.
- DATA(lo_validator) = NEW zcl_bp_validator( ).
- LOOP:
    convertir usando CORRESPONDING.
    validar.
    acumular issues.
- Agrupar issues por partner usando HASHED TABLE.
- Calcular status:
    'R' si hay error.
    'Y' si solo warnings.
    'G' si limpio.
- Llenar rt_alv y rt_issues.

Restricciones:

- No SELECT en loops.
- No reusar mismo field-symbol en loops distintos.
- No line_exists con field-symbol; usar IS ASSIGNED.
- Código limpio y legible.

Entrega:
Clase completa compilable.

---

## Resultado Final Esperado

Arquitectura modular, desacoplada y limpia:

Reader → Validator → Processor → ALV

Aplicando:

- SRP
- Early return
- Lazy initialization
- Clean ABAP
- ABAP 7.4+
- Sin dependencias innecesarias


# Interfaz de Validación de Business Partners

**Contexto**

El proyecto de validación de Business Partners ya tiene varias clases que implementan reglas específicas (NIT, DPI, Contacto). Actualmente existen definiciones de tipos dispersas en clases, lo cual genera acoplamiento y errores de compatibilidad.

Se requiere centralizar el modelo de datos del bloque en una interfaz común.


**Objetivo**

Crear una interfaz pública que:

- Declare los tipos canónicos del bloque.
- Defina el contrato común de validación.


**Sintaxis / Restricciones Técnicas**

- ABAP OO, interfaz pura.
- Los tipos deben declararse en la sección pública de la interfaz.
- No declarar atributos.
- No implementar lógica.


**Tipos Requeridos**

La interfaz debe definir:

**1. Tipo estructura BP**  
Debe incluir los siguientes campos:
- partner
- tipo BP
- nit
- dpi
- email
- phone

**2. Tabla de BPs**

**3. Tipo estructura Issue**  
Debe incluir los siguientes campos:
- partner
- field_name
- severity
- message
- original_value
- proposed_value

**4. Tabla de Issues**


**Método Requerido**

Debe declarar un método de validación que:

- Reciba un BP.
- Devuelva tabla de issues.



**Arquitectura**

Todas las clases validadoras deben implementar esta interfaz y usar exactamente estos tipos.


**Restricciones**

- No redefinir tipos en las clases.
- No agregar constantes innecesarias.
- No incluir lógica en la interfaz.

---

# Interfaz del Procesador Principal

**Contexto**

Existe una clase que coordina lectura, validación y consolidación. Para mejorar el desacoplamiento, se requiere definir su contrato formal.


**Objetivo**

Crear una interfaz para el componente procesador principal.



**Sintaxis / Restricciones Técnicas**

- Reutilizar los tipos definidos en la interfaz de validación.
- No redefinir estructuras.



**Método Requerido**

Debe declarar un método que:

- Reciba estructura de selección.
- Devuelva tabla ALV.
- Devuelva tabla de issues.



**Arquitectura**

La clase procesadora existente debe implementar esta interfaz.



**Restricciones**

- No lógica en la interfaz.
- No tipos duplicados.
