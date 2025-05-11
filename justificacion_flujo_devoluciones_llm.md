# Justificación del Desarrollo y Diseño de Prompts

## Objetivo General

El objetivo de este flujo es procesar automáticamente emails de solicitudes de devolución de clientes, simulando una situación real en un entorno empresarial. El flujo está compuesto por tres pasos principales:

1. Extracción de información estructurada del email.  
2. Evaluación automatizada según políticas internas.  
3. Generación de una respuesta profesional al cliente.

Cada paso está diseñado como una función independiente que interactúa con un modelo LLM (*ChatOpenAI*) mediante *prompts* específicos. Esto permite modularidad, interpretabilidad y la posibilidad de reutilizar componentes en otros flujos.

---

## Paso 1: `extractor_info_devolucion(email_content)`

**Propósito:** Transformar un texto no estructurado (email del cliente) en un JSON bien definido con campos clave.

**Diseño del prompt:**
- Se solicita al modelo que devuelva exclusivamente un JSON válido.
- Se especifica la estructura deseada y se limita el tamaño del campo `motivo_solicitud` para forzar una síntesis.
- Se indica cómo deben representarse campos booleanos como `evidencia_adjunta` de forma expresiva ("sí: imágenes", "no").
- Se fuerza la respuesta en formato JSON con `model_kwargs={"response_format": {"type": "json_object"}}` para evitar errores de parsing.

Este diseño permite obtener datos limpios y estructurados para su posterior evaluación automática.

---

## Paso 2: `evaluador_devolucion(info_solicitud)`

**Propósito:** Evaluar si la solicitud cumple con las políticas internas de la empresa.

**Diseño del prompt:**
- Se repasan las políticas explícitamente dentro del prompt para que el modelo tenga contexto suficiente.
- Se solicitan campos booleanos (`motivo_aceptable`, `evidencia_suficiente`, etc.) y una `decision_preliminar` clara ("aceptar" o "rechazar").
- Se solicita una explicación resumida (`razon`) en máximo 50 palabras para que el equipo humano pueda auditar la decisión si es necesario.
- Nuevamente, se exige un JSON como salida para facilitar el procesamiento automatizado.

Esto transforma un juicio cualitativo del LLM en una decisión de negocio trazable.

---

## Paso 3: `generador_respuesta(info_solicitud, evaluacion)`

**Propósito:** Redactar una respuesta profesional y personalizada para el cliente.

**Diseño del prompt:**
- El cuerpo del mensaje está parcialmente fijado para mantener consistencia y tono corporativo.
- Se incorpora el resultado de la evaluación y la información original del cliente como contexto.
- El modelo puede variar el contenido en función de si se trata de una aceptación o rechazo, incluyendo alternativas si aplica.
- Se introduce una estructura de contacto empresarial (`EMPRESA_INFO`) para simular un entorno corporativo realista.

Esto permite mantener la humanización de la comunicación, pero con un ahorro significativo de tiempo operativo.

---

## Flujo Integrado: `procesar_devolucion(email_content)`

La función final orquesta los tres pasos anteriores, devolviendo un diccionario con:
- La información extraída (`info`)
- La evaluación (`evaluacion`)
- La respuesta generada (`respuesta`)

Además, se incluye manejo de errores básico para asegurar robustez mínima en entornos reales.

---

## Conclusión

Este flujo demuestra cómo los LLMs pueden integrarse en sistemas de gestión operativa para:
- Reducir tiempos de atención al cliente.
- Mejorar la trazabilidad de decisiones.
- Mantener un alto nivel de personalización y profesionalismo.

Además, el diseño modular permite reutilizar o extender el sistema fácilmente, por ejemplo, para solicitudes de cambios, soporte técnico u otras gestiones administrativas.