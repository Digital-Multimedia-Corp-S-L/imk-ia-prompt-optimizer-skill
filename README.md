# imk-ia-prompt-optimizer-skill

Skill de Claude Code (empaquetada como plugin) que convierte una petición en bruto en un
**prompt optimizado para el modelo que la va a ejecutar**, siguiendo la guía oficial de cada
proveedor, te lo enseña para que lo apruebes y entonces lo lanza.

Por ahora soporta **Claude** y **Codex/GPT**. Copilot y otros quedan para más adelante.

## Instalación

Repo privado de la organización: hace falta acceso de lectura a
`Digital-Multimedia-Corp-S-L` y credenciales de git en tu máquina (`gh auth login` o clave
SSH).

```
/plugin marketplace add Digital-Multimedia-Corp-S-L/imk-ia-prompt-optimizer-skill
/plugin install imk-ia-prompt-optimizer
```

Para actualizar: `/plugin update`. Sin plugins, también sirve copiar
`skills/imk-prompt/` a `~/.claude/skills/imk-prompt/`.

## Uso

```
/imk-prompt <petición>
/imk-prompt --target codex <petición>
/imk-prompt --model opus <petición>
/imk-prompt --run <petición>
```

Si hay otra skill llamada `imk-prompt`, usa el nombre completo:
`/imk-ia-prompt-optimizer:imk-prompt`.

| Argumento | Qué hace | Por defecto |
|---|---|---|
| `--target claude\|codex` | Guía del proveedor que se aplica | se detecta por el modelo de la sesión |
| `--model <alias>` | Modelo que **optimiza** (p. ej. `opus`) | el modelo de la sesión |
| `--run` | Lanza el prompt sin pedir confirmación | desactivado: lo enseña y pregunta |

## Qué te enseña

Antes de ejecutar nada, la skill muestra:

- **Optimizado para**: proveedor, modelo y versión exactos (y si se detectó o lo fijaste tú).
- **Optimizado por**: el modelo que hizo la optimización.
- **Guía aplicada**: qué referencia se usó y de qué fecha es.
- El **prompt**, los **ajustes específicos del modelo**, los **cambios clave** y las
  **suposiciones** que ha tenido que hacer.

## Cómo funciona

1. Detecta el modelo destino (o usa `--target`).
2. Carga solo la referencia de ese proveedor (`references/claude.md` o `references/codex.md`).
3. Si la petición es trivial, no la optimiza y la ejecuta tal cual.
4. Optimiza con el modelo de la sesión (contexto completo) o, con `--model`, con un agente de
   otro modelo al que se le pasa un **resumen** del contexto relevante (no ve la conversación).
5. Te enseña el resultado y espera tu OK, salvo con `--run`.

La skill está escrita en inglés (menos tokens y alineada con las fuentes). El prompt
optimizado y las explicaciones salen en el idioma de tu petición.

## Referencias

Las guías son una **instantánea** con fecha y enlaces a la fuente; los consejos de los
proveedores cambian con cada modelo. Al actualizarlas, vuelve a leer las páginas oficiales y
cambia la fecha:

- `skills/imk-prompt/references/claude.md` — Anthropic (instantánea 2026-11-17).
- `skills/imk-prompt/references/codex.md` — OpenAI / Codex (instantánea 2026-11-17).

## Estructura

```
.claude-plugin/        plugin.json y marketplace.json
skills/imk-prompt/
├── SKILL.md           flujo, argumentos, reglas y formato de salida
└── references/        guía por proveedor (claude.md, codex.md)
```

## Limitaciones

- `--model` depende de la herramienta de agentes de Claude Code; en otras herramientas se usa
  el modelo de la sesión.
- La autodetección del modelo se basa en lo que el propio modelo sabe de sí mismo; si dudas,
  fija `--target`.
