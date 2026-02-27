# Prompts de Calidad y Documentación ABAP (Bloque 4)

Este documento contiene las plantillas maestras para automatizar la documentación técnica y la generación de pruebas unitarias en el entorno de desarrollo.

---

## 1. Prompt para Generación de ABAPDoc
**Uso:** Copiar este prompt junto con el código de la definición de la clase para generar los comentarios técnicos estándar.

### Prompt:
```text
" PROMPT (ABAPDoc) – Plantilla breve para documentar definición de clase (solo PUBLIC SECTION)
"
" Genera ABAPDoc SOLO en la sección DEFINITION de esta clase (no tocar IMPLEMENTATION).
" Reglas:
" - Documenta: descripción de clase + cada método público (incluye los de interfaces implementadas).
" - Mantén el texto breve, explícito y técnico (máx 3–5 líneas por método).
" - Usa el formato ABAPDoc estándar:
"   "! <p class="shorttext synchronized">...</p>
"   "! ...
"   "! @parameter ...
"   "! @returning ...
"   "! @raising ...
" - Solo documenta parámetros reales de la firma (si no hay, no inventar).
" - Si el método retorna tabla/estructura, documenta qué contiene.
" - Si aplica, menciona regla de negocio en 1 línea (ej. formato, normalización, severidades E/W, etc.).
" - No agregar comentarios '//' ni texto fuera de ABAPDoc.
"
" Entregable:
" - Devuelve la sección CLASS ... DEFINITION completa, con ABAPDoc insertado arriba de la clase y de cada método público.

```
---

## 2. Prompt para ABAP Unit (ZIF_BP_VALIDATOR)

### Prompt:
```text
" PROMPT (ABAP Unit) – Plantilla genérica para generar tests de clases validadoras (ZIF_BP_VALIDATOR)
"
" Genera una clase ABAP Unit completa para probar ESTA clase validadora que implementa ZIF_BP_VALIDATOR.
" Objetivo: validar comportamiento observable de validate( ) sin asumir fórmulas ni reglas externas.
"
" Reglas:
" - Analiza el código de la clase (métodos privados llamados desde validate y mensajes/severidades usadas).
" - Construye tests en base a:
"   * Reglas de formato/longitud
"   * Normalización (si proposed_value difiere del original)
"   * Mensajes exactos usados en ls_issue-message
"   * Severidades usadas ('E' / 'W')
" - NO inventes nuevas reglas. Solo prueba lo que está implementado.
" - Usa los mensajes EXACTOS del validador (copy-paste) para asserts.
" - Donde haya normalización:
"   * Caso “ya normalizado”: no debe existir warning de normalización.
"   * Caso “normalizable”: debe existir warning de normalización y proposed_value esperado.
" - Donde haya validación de formato:
"   * Caso válido: NO debe existir issue con severity 'E' para el field_name correspondiente.
"   * Caso inválido: DEBE existir issue con severity 'E' y message no inicial.
" - Donde el campo sea opcional (si validate retorna sin issues cuando está vacío): crear test que confirme “sin issues”.
"
" Estructura del test:
" - Clase: ZCL_<VALIDADOR>_TEST  FINAL FOR TESTING RISK LEVEL HARMLESS DURATION SHORT.
" - setup: instancia NEW <validador>( ).
" - build_bp: arma zif_bp_validator=>ty_bp con partner fijo, bp_type fijo y setea SOLO los campos usados por el validador.
" - Helpers:
"   * assert_no_error( it_issues, iv_field, iv_context )
"   * assert_has_error( it_issues, iv_field, iv_context )
"   * assert_has_warning_msg( it_issues, iv_field, iv_message, iv_context )
"   * assert_no_warning_msg( it_issues, iv_field, iv_message, iv_context )
"   * opcional: assert_proposed_value( it_issues, iv_field, iv_severity, iv_message, iv_expected, iv_context )
"
" Casos mínimos que debes generar (si aplican al código):
" 1) valid_<field>_already_normalized:
"    - Entrada que cumple formato y no cambia con normalización -> sin E y sin warning de normalización.
" 2) valid_<field>_normalizes:
"    - Entrada que se pueda normalizar (espacios, guiones, puntos, mayúsculas/minúsculas) -> sin E y con warning de normalización.
" 3) invalid_<field>_empty (si el validador emite W de vacío) o empty_is_ok (si no emite nada):
"    - Verifica el comportamiento exacto del código.
" 4) invalid_<field>_format:
"    - Al menos 2 variantes que disparen E (longitud incorrecta, caracteres inválidos, faltan separadores, etc.)
"
" Entregable:
" - Devuelve SOLO código ABAP (DEFINITION + IMPLEMENTATION) de la clase de test, listo para pegar.
" - No incluyas explicación fuera del código.
" - Mantén nombres de tests claros y consistentes.
" - Si el validador produce issues para múltiples fields, crea tests separados por field.
" - Si un método privado existe pero no afecta validate, no lo pruebes.
```
