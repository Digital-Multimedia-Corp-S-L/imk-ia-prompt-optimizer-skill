# imk-ia-prompt-optimizer-skill

Skill que convierte una petición en bruto en un **prompt optimizado para el modelo que la va a
ejecutar**, siguiendo la guía oficial de cada proveedor. Te enseña el prompt (con el modelo y la
versión para los que está optimizado) y, solo cuando lo apruebas, lo lanza.

Funciona en **Claude Code** (empaquetada como plugin) y en **Codex** (como skill). Soporta
Claude y Codex/GPT como destino; Copilot y otros quedan para más adelante.

> Repo privado de la organización: hace falta acceso de lectura a `Digital-Multimedia-Corp-S-L`
> y credenciales de git en tu máquina (`gh auth login` o clave SSH).

## Instalación

### Claude Code

```
/plugin marketplace add Digital-Multimedia-Corp-S-L/imk-ia-prompt-optimizer-skill
/plugin install imk-ia-prompt-optimizer@imk-ia-prompt-optimizer
/reload-plugins
```

Comprueba que está cargada escribiendo `/imk-prompt`. Para actualizar: `/plugin update`.

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

Reinicia Codex si no la detecta, y comprueba con `/skills` o escribiendo `$imk-prompt`. Para
actualizar: `git pull` y vuelve a copiar la carpeta (en Linux/macOS puedes usar un enlace
simbólico en lugar de copiar).

> La instalación en Codex sigue su documentación oficial para skills; la ruta en Windows no
> aparece en ella y conviene comprobarla en tu máquina.

## Ayuda de uso

```
/imk-prompt <petición>                    # en Claude Code
$imk-prompt <petición>                    # en Codex
```

Con argumentos opcionales al principio (en Codex se escriben igual):

| Argumento | Qué hace | Por defecto |
|---|---|---|
| `--target claude\|codex` | Quién ejecuta el prompt y qué guía se aplica | el proveedor del modelo de la sesión |
| `--run` | Lo lanza sin pedir confirmación | desactivado: lo enseña y pregunta |

### Ejemplos

```
/imk-prompt arregla el test que falla en el módulo de usuarios
/imk-prompt --target codex revisa el último commit
/imk-prompt --run añade un botón de exportar al listado
```

### Qué verás

Antes de ejecutar nada, la skill muestra:

- **Optimizado para**: proveedor, modelo y versión exactos (y si se detectó o lo fijaste tú).
- **Lo ejecutará**: esta sesión, o el plugin de Codex con su modelo y esfuerzo.
- **Optimizado por**: el modelo de la sesión.
- **Guía aplicada**: qué referencia se usó y de qué fecha es.
- El **prompt**, los **ajustes para ese modelo**, los **cambios clave** y las **suposiciones**.

Después te pregunta: lo lanza tal cual, lo ajustas tú o usa tu versión original.

### Cómo se comporta

- **Optimiza siempre el modelo de la sesión**, con todo el contexto de la conversación.
- **Peticiones triviales** (una pregunta corta, un saludo): no las optimiza y las ejecuta tal cual.
- **No añade requisitos** que no pediste; lo que completa lo lista como suposición.
- **Mismo idioma** que tu petición; la skill está escrita en inglés (menos tokens).
- **`--target codex` desde Claude Code**: optimiza para Codex y, al aprobar, **delega la
  ejecución en el plugin de Codex**, que usa su modelo por defecto. La skill lo lee de
  `~/.codex/config.toml` y lo muestra en la cabecera. El prompt sale autocontenido (Codex no ve
  la conversación) y, si la tarea es de solo lectura, se lo indica al plugin, que escribe por
  defecto. Requiere el plugin `codex` instalado y listo (`/codex:setup`).
- **Otras combinaciones** (por ejemplo `--target claude` desde Codex): no puede delegar, así que
  te enseña el prompt para pegarlo en la otra herramienta.

### Si algo no va

| Síntoma | Qué hacer |
|---|---|
| No aparece `/imk-prompt` | Ejecuta `/reload-plugins`; comprueba que el plugin está instalado con `/plugin`. |
| No aparece `$imk-prompt` en Codex | Revisa que la carpeta está en `~/.agents/skills/imk-prompt` con su `SKILL.md`; reinicia Codex. |
| La cabecera dice "versión no determinada" | El modelo no sabe cuál es; usó la guía general. Fija `--target` o revisa la guía del proveedor. |
| `--target codex` no ejecuta, solo enseña el prompt | El plugin de Codex no está instalado o listo: `/codex:setup`. |
| Recomienda revisar la página del proveedor | La guía interna tiene más de ~3 meses o el modelo es más nuevo que ella; ver Referencias. |

## Cómo funciona

1. Detecta el modelo destino (o usa `--target`).
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

- La delegación en Codex solo funciona desde Claude Code con el plugin `codex` instalado.
- La autodetección del modelo se basa en lo que el propio modelo sabe de sí mismo; si dudas,
  fija `--target`.
- En Codex, la skill no se ha probado todavía: `SKILL.md` lleva campos propios de Claude Code
  (`argument-hint`, `disable-model-invocation`) y la documentación de Codex no dice si los
  ignora.
