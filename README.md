# Promt 1
Actúa como consultor ABAP senior.
Estoy iniciando el diseño de un programa Z para auditar y corregir datos de Business Partners en SAP.
El objetivo es identificar registros con datos inconsistentes o incompletos, validar la información según reglas definidas y proponer correcciones, manteniendo un enfoque orientado a objetos y principios SOLID.

## Validaciones requeridas:
NIT Empresarial: formato XXXXXXXX-X (8–9 dígitos + dígito verificador)
DPI Personal: 13 dígitos consecutivos
Email: formato estándar de dirección (RFC 5322), con @ y dominio válido
Teléfono: código país Guatemala +502 seguido de 8 dígitos
Funcionalidades esperadas:
Lectura selectiva de Business Partners desde la tabla estándar BUT000
Validación automática según tipo de BP (Persona u Organización)
Generación de resultados con estados visuales (Correcto / Advertencia / Error)
Actualización controlada de datos mediante BAPIs estándar
Registro de auditoría de los cambios realizados

El proyecto se desarrollará por fases (análisis, core, UI, calidad y documentación).
Por ahora solo necesito análisis y diseño, no implementación ni código.

Ayúdame a identificar los principales componentes del sistema, el flujo lógico general y los aspectos técnicos que debo considerar antes de iniciar el desarrollo.
