# Tarea: Mi prompt avanzado

## Tarea elegida
Generación de casos de prueba funcionales, negativos y de valores límite para un formulario de registro de usuarios en una aplicación web (campos: nombre completo, correo electrónico y contraseña).

## Version 1: prompt basico

```text
Genera casos de prueba para un formulario de registro de usuarios que tiene nombre completo, correo electrónico y contraseña.
```

* **Técnica agregada:** Ninguna (prompt inicial directo para establecer una línea base).
* **Por qué:** Necesitaba comprobar qué respuesta entrega el modelo sin contexto, sin rol definido y sin restricciones de formato.
* **Qué faltó / Qué mejoró en la respuesta:** Al enviar esta versión en el chat, el modelo devolvió una lista general con viñetas (por ejemplo, *"probar contraseña válida"*, *"probar correo inválido"*), pero **no incluyó datos de entrada concretos** (como cadenas exactas de texto o longitudes específicas), omitió los mensajes de error esperados y entregó un formato narrativo poco práctico para documentar pruebas de QA.

## Version 2

```text
### Rol
Actúa como un Ingeniero de Aseguramiento de Calidad (QA) especializado en pruebas funcionales de aplicaciones web y validación de formularios.

### Contexto
Estoy validando el módulo de registro de nuevos usuarios de una plataforma web antes de su despliegue a producción.

### Reglas de negocio del formulario
- Nombre completo: obligatorio, solo letras y espacios, longitud entre 3 y 50 caracteres.
- Correo electrónico: obligatorio, formato válido (usuario@dominio.com) y único en el sistema.
- Contraseña: obligatoria, longitud entre 8 y 20 caracteres, debe contener al menos una letra mayúscula, un número y un símbolo (@, #, $, %).

### Formato de salida
Responde únicamente con una tabla en Markdown que contenga estas columnas exactas:
| ID | Escenario de prueba | Tipo (Positivo / Negativo / Límite) | Datos de entrada | Resultado esperado |
```

* **Técnica agregada:** **Role Prompting** (asignación de un rol específico de QA sin usar la palabra "experto") y **Prompt Estructurado** (separación clara mediante encabezados `###` y definición estricta de tabla Markdown).
* **Por qué:** Después de evaluar la v1, vi que faltaba contexto técnico sobre las reglas del formulario y una estructura tabular reutilizable en un plan de pruebas.
* **Qué mejoró en la respuesta:** El modelo eliminó introducciones innecesarias, organizó los casos directamente en la tabla solicitada y aplicó las validaciones del formulario. Sin embargo, al revisar la salida noté que todavía faltaba precisión en los **valores límite exactos** (por ejemplo, escribía *"contraseña muy corta"* en lugar de probar exactamente 7 vs. 8 caracteres) y los mensajes esperados variaban de estilo entre una fila y otra.

## Version 3: prompt final

```text
### Rol
Actúa como un Ingeniero de Aseguramiento de Calidad (QA) especializado en pruebas funcionales de aplicaciones web y validación de formularios.

### Contexto
Estoy validando el módulo de registro de nuevos usuarios de una plataforma web antes de su despliegue a producción.

### Reglas de negocio del formulario
- Nombre completo: obligatorio, solo letras y espacios, longitud entre 3 y 50 caracteres.
- Correo electrónico: obligatorio, formato válido (usuario@dominio.com) y único en el sistema.
- Contraseña: obligatoria, longitud entre 8 y 20 caracteres, debe contener al menos una letra mayúscula, un número y un símbolo (@, #, $, %).

### Instrucciones de razonamiento (paso a paso)
Antes de generar la tabla final, ejecuta internamente los siguientes pasos:
1. Identifica las particiones de equivalencia válidas e inválidas de cada campo.
2. Determina los valores límite exactos en la frontera de cada regla (2, 3, 50 y 51 caracteres para nombre; 7, 8, 20 y 21 caracteres para contraseña).
3. Verifica que exista cobertura para campos vacíos, formato de correo inválido, correo duplicado y caracteres especiales no permitidos.

### Ejemplo de referencia (Few-Shot)
Usa exactamente este nivel de precisión en los datos de entrada y mensajes esperados:
| ID | Escenario de prueba | Tipo | Datos de entrada | Resultado esperado |
| :--- | :--- | :--- | :--- | :--- |
| TC-01 | Registro fallido por contraseña debajo del límite mínimo (7 caracteres) | Límite | Nombre: "Carlos Mendoza", Correo: "cmendoza@mail.com", Pass: "Abc@123" | El sistema bloquea el registro y muestra: "La contraseña debe tener entre 8 y 20 caracteres". |

### Formato de salida
Muestra primero un resumen de 3 viñetas con los valores límite calculados y luego entrega la tabla completa en Markdown con al menos 10 casos de prueba siguiendo el estilo del ejemplo.
```

* **Técnica agregada:** **Chain of Thought** (instrucciones paso a paso para calcular particiones de equivalencia y valores límite antes de redactar) y **Few-Shot** (un ejemplo concreto de fila `TC-01` como plantilla de calidad).
* **Por qué:** Para corregir la falta de precisión numérica en los bordes y estandarizar cómo se redactan los datos de prueba y los mensajes de validación en toda la matriz.
* **Qué mejoró en la respuesta:** La salida final presentó un análisis previo de fronteras exactas (7 vs. 8 y 20 vs. 21 caracteres), mantuvo un formato homogéneo en todos los casos `TC-XX` y generó datos de entrada listos para copiar y ejecutar directamente sobre el formulario.

## Tecnicas usadas en el prompt final

| Técnica aplicada | Parte del prompt final donde se ubica | Función dentro del prompt |
| :--- | :--- | :--- |
| **Role Prompting** | Sección `### Rol` (*"Ingeniero de Aseguramiento de Calidad (QA)..."*) | Orienta el criterio técnico hacia pruebas funcionales y detección de fallos en bordes. |
| **Prompt Estructurado** | Encabezados `### Rol`, `### Contexto`, `### Reglas de negocio...`, `### Formato de salida` | Delimita reglas, contexto y formato para evitar que el modelo omita restricciones. |
| **Chain of Thought** | Sección `### Instrucciones de razonamiento (paso a paso)` | Obliga a calcular particiones de equivalencia y límites numéricos antes de construir la tabla. |
| **Few-Shot** | Sección `### Ejemplo de referencia (Few-Shot)` con el caso `TC-01` | Fija el nivel exacto de detalle en los datos de entrada y mensajes de error esperados. |

## Evaluacion del resultado

| Criterio de evaluación | Cumple (Sí / No) | Justificación |
| :--- | :---: | :--- |
| ¿Mantiene un rol específico sin usar la palabra "experto" y un formato definido? | Sí | Define el rol de Ingeniero de QA y exige salida en viñetas breves + tabla Markdown. |
| ¿Cubre casos positivos, negativos y valores límite exactos? | Sí | Incluye registro válido, campos vacíos, formatos inválidos y longitudes frontera (7, 8, 20, 21). |
| ¿Los datos de entrada son concretos y reproducibles? | Sí | Cada fila contiene cadenas exactas para nombre, correo y contraseña listas para probar. |
| ¿Respeta todas las reglas de negocio definidas? | Sí | Valida longitud, complejidad de contraseña, caracteres permitidos y duplicidad de correo. |
| ¿Sigue de forma consistente el ejemplo *Few-Shot*? | Sí | Todos los casos conservan la codificación `TC-XX` y detallan el mensaje esperado del sistema. |

## Por que elegi estas tecnicas

Elegí combinar **Role Prompting**, **Prompt Estructurado**, **Chain of Thought** y **Few-Shot** porque la generación de casos de prueba exige exactitud lógica en los valores frontera, cumplimiento estricto de reglas de negocio y un formato tabular estandarizado que pueda ejecutarse sin ambigüedades: el razonamiento paso a paso garantiza que no se omitan límites críticos (como probar 7 y 8 caracteres en la contraseña) y el ejemplo *few-shot* evita que el modelo escriba descripciones genéricas en lugar de datos de entrada concretos. Descarté la **descomposición en múltiples prompts** porque un formulario de tres campos tiene un alcance acotado que conviene mantener unificado en una sola matriz, y no agregué **autocrítica** al final porque la verificación previa mediante *Chain of Thought* ya valida la cobertura completa sin duplicar la longitud de la respuesta.

