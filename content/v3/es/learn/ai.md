# IA y Experiencia de Desarrollo con Flight

## Resumen

Flight está diseñado para trabajar *con* las herramientas de codificación de IA, no para luchar contra ellas. Una API pequeña y predecible, una disposición clara de la aplicación en el [skeleton oficial](https://github.com/flightphp/skeleton) y archivos de instrucciones específicos del proyecto significan que asistentes como GitHub Copilot, Cursor, Windsurf, Claude Code y Gemini pueden seguir los mismos patrones que escribirías a mano.

Con los comandos integrados de Runway para conectarse a proveedores de LLM y generar instrucciones de proyecto, Flight te ayuda a ti y a tu equipo a obtener ayuda consistente y relevante sin tener que pegar el mismo contexto en cada chat.

## Comprensión

Los asistentes de codificación con IA son más útiles cuando comprenden el contexto, las convenciones y los objetivos de tu proyecto. Los ayudantes de IA de Flight te permiten:

- Conectar tu proyecto a proveedores de LLM populares (OpenAI, Grok, Claude, etc.)
- Generar y actualizar instrucciones específicas del proyecto para que todos reciban la misma guía
- Mantener el código escrito a mano y el generado por IA en un solo diseño (especialmente con el skeleton)

Estas características vienen con el CLI principal de Flight (a través de [Runway](/awesome-plugins/runway)) y están preconfiguradas en el iniciador oficial [flightphp/skeleton](https://github.com/flightphp/skeleton).

### Lo que el skeleton incluye para IA

El iniciador oficial trata **`AGENTS.md` como la fuente de verdad** para las herramientas de IA:

| Archivo | Rol |
|------|------|
| **`AGENTS.md`** (raíz del proyecto) | Reglas globales, flujo de arranque, espacios de nombres, DI, "qué no hacer" |
| **`AGENTS.md` con ámbito** en `app/`, `migrations/`, `tests/`, etc. | Consejos ligeros y específicos de carpeta cuando trabajas en ese árbol |
| **`SECURITY.md`** | Secretos, cabeceras, XSS/SQL, reportes—la seguridad se mantiene deliberada y separada |

**No** hay un archivo de estilo de casa separado para Copilot / Cursor / Gemini / Windsurf en el skeleton. Señala a tu asistente al `AGENTS.md` raíz (y deja que siga los enlaces a los archivos con ámbito). Los humanos pueden ignorar estos archivos por completo y usar el [README](https://github.com/flightphp/skeleton); el diseño es el mismo de cualquier manera.

> **Los documentos enseñan APIs; el skeleton enseña el diseño.** Los ejemplos cortos de `Flight::` en estos documentos son excelentes para aprender. En una aplicación skeleton, prefiere las clases `App\…`, la inyección de constructor y `$this->app` sobre la fachada estática dentro de los controladores. Consulta [Instalación](/install) y [Autocarga](/learn/autoloading).

## Uso Básico

### Configuración de Credenciales de LLM

El comando `ai:init` te guía a través de la conexión de tu proyecto a un proveedor de LLM.

```bash
php runway ai:init
```

Se te pedirá que:

- Elijas tu proveedor (OpenAI, Grok, Claude, etc.)
- Ingreses tu clave de API
- Establezcas la URL base y el nombre del modelo

Esto crea las credenciales utilizadas para solicitudes posteriores de LLM (por ejemplo, para generar instrucciones).

**Ejemplo:**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### Generación de Instrucciones de IA Específicas del Proyecto

El comando `ai:generate-instructions` crea o actualiza instrucciones para asistentes de codificación con IA, adaptadas a *tu* proyecto.

```bash
php runway ai:generate-instructions
```

Responderás algunas preguntas (descripción, base de datos, plantillas, seguridad, tamaño del equipo, etc.). Flight utiliza tu proveedor de LLM para generar las instrucciones y las escribe principalmente en:

- **`AGENTS.md`** en la raíz del proyecto (independiente de la herramienta; lo que el skeleton oficial y la mayoría de los agentes modernos esperan)

Dependiendo de la versión del CLI y las opciones, el comando también puede escribir copias específicas de herramientas para flujos de trabajo más antiguos (por ejemplo, archivos de reglas de Copilot, Cursor, Windsurf o Gemini). Para **proyectos nuevos desde el skeleton**, trata **`AGENTS.md`** (más cualquier archivo `AGENTS.md` con ámbito que mantengas bajo `app/`) como la única fuente de verdad—no mantengas cinco archivos de instrucciones divergentes a mano.

**Ejemplo:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? twig
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

Ahora las herramientas de IA pueden sugerir código que coincida con tu pila y diseño reales, no un tutorial genérico de PHP.

## Uso Avanzado

- Personaliza credenciales o rutas de salida con las opciones de los comandos (consulta `--help` en cada comando).
- Los ayudantes funcionan con cualquier proveedor de LLM que hable una API compatible con OpenAI.
- Vuelve a ejecutar `ai:generate-instructions` a medida que el proyecto evolucione para que los agentes se mantengan sincronizados.
- En el skeleton, mantén la política de seguridad en **`SECURITY.md`** y el diseño de codificación en **`AGENTS.md`** para que ningún documento se convierta en una caja de todo.
- Prefiere [docs.flightphp.com](https://docs.flightphp.com) y el servidor MCP de Flight cuando los agentes necesiten detalles de la API; verifica los métodos inventados contra `vendor/flightphp/core`.

## Ver También

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Iniciador oficial con `AGENTS.md`, Twig, SimplePdo y Dice configurados para una estructura amigable con IA
- [Instalación](/install) – Diseño recomendado de `create-project`
- [Autocarga](/learn/autoloading) – Las **mayúsculas** de las carpetas coinciden con los espacios de nombres (`App\Controller` ↔ `app/Controller/`)
- [CLI de Runway](/awesome-plugins/runway) – CLI que impulsa los comandos `ai:*` y de scaffolding
- [Seguridad](/learn/security) – Valores predeterminados seguros que los agentes (y los humanos) no deberían debilitar

## Solución de Problemas

- Si ves "Missing .runway-creds.json", ejecuta `php runway ai:init` primero.
- Asegúrate de que tu clave de API sea válida y tenga acceso al modelo seleccionado.
- Si las instrucciones no se actualizan, verifica los permisos de archivo en el directorio de tu proyecto.
- Si un agente inventa APIs de Flight o el diseño de carpetas incorrecto, apúntalo al **`AGENTS.md`** raíz y a este sitio de documentación; el diseño del skeleton tiene prioridad para el código bajo `app/`.

## Registro de Cambios

- v3.18.4 – `ai:generate-instructions` escribe las instrucciones del proyecto en `AGENTS.md` en la raíz del proyecto.
- v3.16.0 – Se agregaron los comandos CLI `ai:init` y `ai:generate-instructions` para la integración de IA.