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
- Sin SELECT dentro de loops
- Early return
- Métodos pequeños y claros
- No usar short form inválido (-> solo instancia, => solo estático)
- Código compilable

---

# ZCL_BP_READER

## Objetivo
Leer Business Partners desde tabla ZBP_BUT000_SIM según rango de partner.

## Prompt

Actúa como ABAP senior en S/4HANA (7.4+).  
Crea desde cero la clase ZCL_BP_READER (DEFINITION + IMPLEMENTATION).

Requerimiento:

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

Entrega:
Clase completa lista para activar.

---

# ZCL_BP_NITVALIDATOR

## Objetivo
Validar NIT Guatemala con algoritmo módulo 11.

## Prompt

Crea desde cero la clase ZCL_BP_NITVALIDATOR.

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

- Sin regex.
- Métodos pequeños:
  normalize,
  is_format_ok,
  calc_check_digit,
  build_issue.
- Conversión numérica segura (sin char * int implícito).

Entrega:
Clase completa compilable.

---

# ZCL_BP_DPIVALIDATOR

## Objetivo
Validar DPI/CUI Guatemala.

## Prompt

Crea desde cero la clase ZCL_BP_DPIVALIDATOR.

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

# Resultado Final Esperado

Arquitectura modular, desacoplada y limpia:

Reader → Validator → Processor → ALV

Aplicando:

- SRP
- Early return
- Lazy initialization
- Clean ABAP
- ABAP 7.4+
- Sin dependencias innecesarias
