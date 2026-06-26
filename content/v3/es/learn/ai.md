# IA y Experiencia de Desarrollador con Flight

## Resumen

Flight facilita la mejora de tus proyectos PHP con herramientas impulsadas por IA y flujos de trabajo modernos para desarrolladores. Con comandos integrados para conectarse a proveedores de LLM (Modelo de Lenguaje Grande) y generar instrucciones de codificación con IA específicas para el proyecto, Flight te ayuda a ti y a tu equipo a sacar el máximo provecho de asistentes de IA como GitHub Copilot, Cursor, Windsurf y Antigravity (Gemini).

## Entendimiento

Los asistentes de codificación con IA son más útiles cuando comprenden el contexto, las convenciones y los objetivos de tu proyecto. Los ayudantes de IA de Flight te permiten:
- Conectar tu proyecto a proveedores de LLM populares (OpenAI, Grok, Claude, etc.)
- Generar y actualizar instrucciones específicas del proyecto para herramientas de IA, para que todos reciban ayuda consistente y relevante
- Mantener a tu equipo alineado y productivo, con menos tiempo dedicado a explicar el contexto

Estas características están integradas en la CLI principal de Flight y en el proyecto inicial oficial [flightphp/skeleton](https://github.com/flightphp/skeleton).

## Uso Básico

### Configurando Credenciales de LLM

El comando `ai:init` te guía para conectar tu proyecto a un proveedor de LLM.

```bash
php runway ai:init
```

Se te pedirá que:
- Eligas tu proveedor (OpenAI, Grok, Claude, etc.)
- Ingreses tu clave API
- Establezcas la URL base y el nombre del modelo

Esto crea las credenciales necesarias para que puedas realizar solicitudes LLM futuras.

**Ejemplo:**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### Generando Instrucciones de IA Específicas del Proyecto

El comando `ai:generate-instructions` te ayuda a crear o actualizar instrucciones para asistentes de codificación con IA, adaptadas a tu proyecto.

```bash
php runway ai:generate-instructions
```

Responderás algunas preguntas sobre tu proyecto (descripción, base de datos, plantillas, seguridad, tamaño del equipo, etc.). Flight usa tu proveedor de LLM para generar instrucciones, luego escribe el mismo contenido en:
- `.github/copilot-instructions.md` (para GitHub Copilot)
- `.cursor/rules/project-overview.mdc` (para Cursor)
- `.windsurfrules` (para Windsurf)
- `.gemini/GEMINI.md` (para Antigravity)
- `AGENTS.md` (en la raíz del proyecto, para asistentes de IA independientes de herramientas)

**Ejemplo:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? latte
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

Ahora, tus herramientas de IA darán sugerencias más inteligentes y relevantes basadas en las necesidades reales de tu proyecto.

## Uso Avanzado

- Puedes personalizar la ubicación de tus archivos de credenciales o instrucciones usando opciones de comandos (consulta `--help` para cada comando).
- Los ayudantes de IA están diseñados para funcionar con cualquier proveedor de LLM que admita APIs compatibles con OpenAI.
- Si quieres actualizar tus instrucciones a medida que evoluciona tu proyecto, simplemente vuelve a ejecutar `ai:generate-instructions` y responde los prompts nuevamente.

## Ver También

- [Flight Skeleton](https://github.com/flightphp/skeleton) – El proyecto inicial oficial con integración de IA
- [Runway CLI](/awesome-plugins/runway) – Más información sobre la herramienta CLI que impulsa estos comandos

## Solución de Problemas

- Si ves "Missing .runway-creds.json", ejecuta primero `php runway ai:init`.
- Asegúrate de que tu clave API sea válida y tenga acceso al modelo seleccionado.
- Si las instrucciones no se están actualizando, verifica los permisos de archivo en el directorio de tu proyecto.

## Registro de Cambios

- v3.18.4 – `ai:generate-instructions` ahora también escribe las instrucciones del proyecto en `AGENTS.md` en la raíz del proyecto.
- v3.16.0 – Se añadieron los comandos CLI `ai:init` y `ai:generate-instructions` para la integración con IA.