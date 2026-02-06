# Prompt 1 Analis del requerimiento
Actúa como consultor ABAP senior.
Estoy iniciando el diseño de un programa Z para auditar y corregir datos de Business Partners en SAP.
El objetivo es identificar registros con datos inconsistentes o incompletos, validar la información según reglas definidas y proponer correcciones, manteniendo un enfoque orientado a objetos y principios SOLID.

**Validaciones requeridas:**
- NIT Empresarial: formato XXXXXXXX-X (8–9 dígitos + dígito verificador)
- DPI Personal: 13 dígitos consecutivos
- Email: formato estándar de dirección (RFC 5322), con @ y dominio válido
- Teléfono: código país Guatemala +502 seguido de 8 dígitos
- Funcionalidades esperadas:
- Lectura selectiva de Business Partners desde la tabla estándar BUT000
- Validación automática según tipo de BP (Persona u Organización)
- Generación de resultados con estados visuales (Correcto / Advertencia / Error)
- Actualización controlada de datos mediante BAPIs estándar
- Registro de auditoría de los cambios realizados

El proyecto se desarrollará por fases (análisis, core, UI, calidad y documentación).
Por ahora solo necesito análisis y diseño, no implementación ni código.

Ayúdame a identificar los principales componentes del sistema, el flujo lógico general y los aspectos técnicos que debo considerar antes de iniciar el desarrollo.

# Prompt 2 Flujo General End-to-End del Sistema

Con base en el contexto del proyecto de auditoría y corrección de Business Partners que se proporcionó previamente, se define el flujo general del sistema de forma end-to-end.

El objetivo es describir de manera lógica y secuencial cómo debe comportarse el programa desde la selección de los Business Partners hasta la obtención de los resultados finales de validación.

**Alcance del Flujo**

El flujo considera los siguientes puntos clave del proceso:

- Lectura selectiva de datos desde `BUT000`
- Determinación del tipo de Business Partner (Persona u Organización)
- Aplicación de validaciones según el tipo de Business Partner
- Clasificación de resultados:
  - Correcto
  - Advertencia
  - Error
- Escenario de ejecución en modo simulación
- Escenario de ejecución con actualización real
- Registro de auditoría y mensajes

**Consideraciones Generales**

- No se describe código ni implementación técnica.
- No se utilizan nombres técnicos definitivos.
- El enfoque del documento es **funcional**, orientado a:
  - Comprender el flujo del proceso
  - Identificar decisiones clave del sistema
  - Documentar el comportamiento esperado del programa

# Prompt 3 – Diseño de arquitectura orientada a objetos (SOLID)

Basado en el contexto y en el flujo end-to-end definidos para el programa de auditoría y corrección de Business Partners, propón una arquitectura orientada a objetos aplicando principios SOLID.

Necesito que lo presentes como diseño conceptual, no implementación. Incluye:

- Componentes principales del sistema (por responsabilidad) y su propósito.
- Lectura de datos
- Validación (por tipo de BP y por tipo de dato)
- Orquestación / coordinación del proceso
- Actualización controlada vía BAPI
- Logging / auditoría

Explica brevemente cómo se aplica SOLID en este caso:

- SRP (una clase por responsabilidad)
- OCP (agregar nuevas validaciones sin tocar el core)
- DIP (depender de interfaces, no de implementaciones)

Propón un patrón o enfoque para ejecutar validaciones según tipo (Persona/Organización) sin duplicar lógica (por ejemplo estrategia/fábrica u otra alternativa).

Define el flujo de comunicación entre componentes (quién llama a quién) y muestra un diagrama simple en texto (ASCII).

No escribas código ABAP ni firmes métodos exactos.


