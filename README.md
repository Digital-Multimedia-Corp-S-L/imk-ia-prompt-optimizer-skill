# imk-ia-prompt-optimizer-skill

Skill que convierte una petición en bruto en un **prompt optimizado para el modelo que la va a
ejecutar**, siguiendo la guía oficial de cada proveedor. Te enseña el prompt (con el modelo y la
versión para los que está optimizado) y, solo cuando lo apruebas, lo lanza.

Funciona en **Claude Code** (empaquetada como plugin) y en **Codex** (como skill). Soporta
Claude y Codex/GPT como destino; Copilot y otros quedan para más adelante.

> Licencia MIT. Repo público de la organización `Digital-Multimedia-Corp-S-L`: para instalarla y actualizarla
> no hace falta ninguna credencial. Solo los miembros con permiso de escritura pueden subir
> cambios.

## Instalación

### Claude Code

```
/plugin marketplace add Digital-Multimedia-Corp-S-L/imk-ia-prompt-optimizer-skill
/plugin install imk-ia-prompt-optimizer@imk-ia-prompt-optimizer
/reload-plugins
```

Comprueba que está cargada escribiendo `/imk-prompt`.

**Para actualizar**: `/imk-prompt --update` (ver [Actualizar con `--update`](#actualizar-con---update)).
Si tienes una versión anterior a la 0.1.5, que aún no incluye `--update`, actualiza esa primera
vez a mano:

```
/plugin marketplace update imk-ia-prompt-optimizer
/plugin update imk-ia-prompt-optimizer@imk-ia-prompt-optimizer
/reload-plugins
```

Claude Code decide si hay algo nuevo comparando el `version` del `plugin.json`, así que cada
cambio publicado sube la versión. Si tras `/reload-plugins` sigue saliendo la versión antigua,
reinicia Claude Code.

**Solo si vas a usar `--target codex`**: necesitas además el **plugin de Codex instalado en
Claude Code** (y Codex con sesión iniciada en tu máquina). Sin él, `--target codex` no funciona.

```
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/reload-plugins
/codex:setup
```

`/codex:setup` comprueba que Codex está listo. Sin `--target`, la skill no necesita el plugin
de Codex.

Desde una copia local del repo (para probar cambios), cambia la primera línea por la ruta de la
carpeta, por ejemplo `/plugin marketplace add S:\proyectos\imk-dev-toolkit\imk-ia-prompt-optimizer-skill`,
o abre Claude Code con `claude --plugin-dir <ruta-del-repo>`.

### Codex

Codex carga las skills desde `~/.agents/skills` (usuario) o `.agents/skills` (dentro de un repo).
Clona el repo y copia la carpeta de la skill:

```bash
git clone https://github.com/Digital-Multimedia-Corp-S-L/imk-ia-prompt-optimizer-skill
mkdir -p ~/.agents/skills
cp -r imk-ia-prompt-optimizer-skill/skills/imk-prompt ~/.agents/skills/
```

En Windows (PowerShell):

```powershell
git clone https://github.com/Digital-Multimedia-Corp-S-L/imk-ia-prompt-optimizer-skill
New-Item -ItemType Directory -Force $HOME\.agents\skills | Out-Null
Copy-Item -Recurse imk-ia-prompt-optimizer-skill\skills\imk-prompt $HOME\.agents\skills\
```

Reinicia Codex si no la detecta, y comprueba con `/skills` o escribiendo `$imk-prompt`.

**Para actualizar**: `$imk-prompt --update` (ver [Actualizar con `--update`](#actualizar-con---update)).
Si tienes una versión anterior a la 0.1.5, actualiza esa primera vez a mano: `git pull` en tu
clon y vuelve a copiar la carpeta `skills/imk-prompt` (en Linux/macOS puedes usar un enlace
simbólico en lugar de copiar).

> La instalación en Codex sigue su documentación oficial para skills; la ruta en Windows no
> aparece en ella y conviene comprobarla en tu máquina. Dentro de Codex no se usa `--target`
> (ya estás en Codex): la skill optimiza para el modelo de esa sesión.

## Ayuda de uso

```
/imk-prompt <petición>                    # en Claude Code
$imk-prompt <petición>                    # en Codex

/imk-prompt --update                      # actualizar la skill (en Codex: $imk-prompt --update)
```

Con argumentos opcionales al principio:

| Argumento | Qué hace | Por defecto |
|---|---|---|
| `--target codex` | Optimiza para Codex y lo ejecuta con el **plugin de Codex instalado en Claude Code** | desactivado: optimiza para el modelo de la sesión y lo ejecuta ahí |
| `--run` | Lo lanza sin pedir confirmación | desactivado: lo enseña y pregunta |
| `--explain` | Añade las explicaciones: ajustes para el modelo, cambios clave, suposiciones y guía aplicada | desactivado: solo enseña el prompt |
| `--update` | Actualiza la skill a la última versión publicada (sin petición) | desactivado |

`--target` tiene **un único valor, `codex`**, y solo existe para usar el plugin de Codex desde
Claude Code. No hay `--target claude`: Claude es lo que se usa por defecto, sin argumento. Solo
funciona en Claude Code y **exige tener el plugin de Codex instalado** (ver Instalación).

### Ejemplos

```
/imk-prompt arregla el test que falla en el módulo de usuarios
/imk-prompt --target codex revisa el último commit
/imk-prompt --run añade un botón de exportar al listado
/imk-prompt --explain mejora la paginación del listado
```

### Qué verás

Antes de ejecutar nada, la skill muestra **solo el prompt**, con una línea encima que indica
**para qué proveedor, modelo y versión** está optimizado (el de la sesión o, con `--target codex`,
el que use el plugin de Codex). Sin explicaciones.

Con `--explain` muestra además el informe completo:

- **Lo ejecutará**: esta sesión, o el plugin de Codex con su modelo y esfuerzo.
- **Optimizado por**: el modelo de la sesión.
- **Guía aplicada**: qué referencia se usó y de qué fecha es.
- Los **ajustes para ese modelo**, los **cambios clave** y las **suposiciones**.

Sin `--explain` no hay sección de suposiciones: si falta algo crítico, la skill te hace una
pregunta en lugar de suponerlo.

Después te ofrece tres opciones con **«Lanzarlo así» preseleccionada**: basta con pulsar Intro.
Las otras dos son «Ajustarlo» (me dices qué cambiar) y «Usar mi versión original». En Codex, que
no tiene ese selector, te lo pregunta en texto y respondes «sí» para aceptar.

### Cómo se comporta

- **Optimiza siempre el modelo de la sesión**, con todo el contexto de la conversación.
- **Peticiones triviales** (una pregunta corta, un saludo): no las optimiza y las ejecuta tal cual.
- **No añade requisitos** que no pediste; lo que completa lo lista como suposición.
- **Mismo idioma** que tu petición; la skill está escrita en inglés (menos tokens).
- **`--target codex`**: optimiza para Codex y, al aprobar, **delega la ejecución en el plugin de
  Codex**, que usa su modelo por defecto. La skill lo lee de `~/.codex/config.toml` y lo muestra
  en la cabecera. El prompt sale autocontenido (Codex no ve la conversación) y, si la tarea es de
  solo lectura, se lo indica al plugin, que escribe por defecto. Si el plugin no está instalado
  o listo, no optimiza: te avisa para que lo instales (`/codex:setup`).

### Actualizar con `--update`

```
/imk-prompt --update          # en Claude Code
$imk-prompt --update          # en Codex
```

Al terminar, la skill imprime siempre uno de estos dos mensajes:

```
No hay actualizaciones de imk-ia-prompt-optimizer-skill
Se ha actualizado imk-ia-prompt-optimizer-skill a la versión vX.Y.Z
```

- **Claude Code**: ejecuta `claude plugin marketplace update` y `claude plugin update` y compara
  las versiones instaladas antes y después. Una skill no puede lanzar `/reload-plugins`, así que **ese
  último paso lo escribes tú**; tras actualizar, imprime siempre `Escribe /reload-plugins para
  cargarla (si sigue la versión antigua, reinicia Claude Code)`.
  Si cargaste el plugin con `--plugin-dir`, no hay nada que actualizar: haz `git pull`.
- **Codex**: clona el repo en una carpeta temporal, compara con la carpeta de la skill y, si hay
  diferencias, la sustituye por la nueva (cualquier cambio local en esa carpeta se pierde).
  Reinicia Codex si no ves el cambio.
- En ambos casos hace falta `git` instalado y conexión a GitHub; el repo es público, no se
  piden credenciales.

### Si algo no va

| Síntoma | Qué hacer |
|---|---|
| No aparece `/imk-prompt` | Ejecuta `/reload-plugins`; comprueba que el plugin está instalado con `/plugin`. |
| No aparece `$imk-prompt` en Codex | Revisa que la carpeta está en `~/.agents/skills/imk-prompt` con su `SKILL.md`; reinicia Codex. |
| La cabecera dice "versión no determinada" | El modelo no sabe cuál es; usó la guía general. Revisa la guía del proveedor. |
| `--target codex` avisa de que falta el plugin | Instala el plugin de Codex en Claude Code y ejecuta `/codex:setup` (ver Instalación). |
| `--target claude` da error | No existe: Claude es el valor por defecto, usa la skill sin `--target`. |
| Recomienda revisar la página del proveedor | La guía interna tiene más de ~3 meses o el modelo es más nuevo que ella; ver Referencias. |

## Cómo funciona

1. Detecta el modelo destino: el de la sesión o, con `--target codex`, el del plugin de Codex.
2. Carga solo la referencia de ese proveedor (`references/claude.md` o `references/codex.md`).
3. Si la petición es trivial, no la optimiza y la ejecuta tal cual.
4. Optimiza con el modelo de la sesión (contexto completo).
5. Te enseña el resultado y espera tu OK, salvo con `--run`.
6. Lo ejecuta en la sesión, o con `--target codex` lo delega en el plugin de Codex.

## Referencias

Las guías son una **instantánea** con fecha y enlaces a la fuente; los consejos de los
proveedores cambian con cada modelo. Al actualizarlas, vuelve a leer las páginas oficiales y
cambia la fecha:

- `skills/imk-prompt/references/claude.md` — Anthropic (instantánea 2026-11-17).
- `skills/imk-prompt/references/codex.md` — OpenAI / Codex (instantánea 2026-11-17).

## Estructura

```
.claude-plugin/        plugin.json y marketplace.json (Claude Code)
skills/imk-prompt/
├── SKILL.md           flujo, argumentos, reglas y formato de salida
├── agents/openai.yaml sin invocación implícita en Codex
└── references/        guía por proveedor (claude.md, codex.md)
```

## Limitaciones

- `--target codex` solo funciona desde Claude Code con el plugin `codex` instalado.
- El modelo y la versión de la sesión los da el propio modelo; si no los sabe, la cabecera dice
  "versión no determinada" y se usa la guía general.
- En Codex, la skill no se ha probado todavía: `SKILL.md` lleva campos propios de Claude Code
  (`argument-hint`, `disable-model-invocation`) y la documentación de Codex no dice si los
  ignora.
