---
name: imk-prompt
description: Rewrites a rough request into an optimized prompt for the AI model that will run it (Claude, or Codex/GPT through the Codex plugin), following that vendor's official prompting guidance, shows it for approval, then runs it. Use when the user invokes /imk-prompt or asks to "optimize this prompt" before executing it.
argument-hint: "[--update | [--target codex] [--run] [--explain] <request>]"
disable-model-invocation: true
---

# imk-prompt

Turn the user's rough request into a prompt tuned for the model that will execute it, show it,
and only then run it. Never run the original request before the user has seen the optimized one
(unless `--run`).

The **optimizer is always the current session model**, with the full conversation context.
Without `--target`, the prompt is optimized for that same model and runs in this session.

## Arguments

Parse these from the start of the arguments; everything else is the **raw request**.

| Argument | Meaning | Default |
|---|---|---|
| `--target codex` | Optimize for Codex and run it through the Codex plugin installed in Claude Code | off: optimize for the session model and run it here |
| `--run` | Run the optimized prompt right away, no confirmation | off: show it and ask |
| `--explain` | Also show the explanations: model-specific adjustments, key changes, assumptions and the guidance used | off: show only the prompt |
| `--update` | Update this skill to the latest published version (no request, no optimizing) | off |

If `--update` is present, ignore the rest and do only **Updating** below.

`--target` has **one value only: `codex`**, and exists only to use the Codex plugin from Claude
Code. There is no `--target claude`: Claude is simply the default. If the user passes any other
value, say that only `--target codex` exists and ask what they meant.

`--target codex` **requires the Codex plugin installed in Claude Code** (`codex@openai-codex`,
checked with `/codex:setup`). When set, after approval the execution is **delegated to that
plugin**, which runs Codex with its own default model. In a Codex session, `--target codex`
makes no sense (you already are Codex): ignore it and say so.

## Updating (`--update`)

Updates this skill from its public GitHub repo (it needs `git` and network access). Do not
optimize anything. Decide the case from where this skill is installed (its base directory):

**Claude Code, installed as a plugin** (base directory under `~/.claude/plugins/cache/`):
1. List the version folders in
   `~/.claude/plugins/cache/imk-ia-prompt-optimizer/imk-ia-prompt-optimizer/` (the "before").
2. With the Bash tool, run in order:
   `claude plugin marketplace update imk-ia-prompt-optimizer` and then
   `claude plugin update imk-ia-prompt-optimizer@imk-ia-prompt-optimizer`.
3. List the folders again. If no new version folder appeared, output exactly
   `No hay actualizaciones de imk-ia-prompt-optimizer-skill`. If one appeared, output exactly
   `Se ha actualizado imk-ia-prompt-optimizer-skill a la versión vX.Y.Z` (X.Y.Z is the new folder's
   name).
4. Only after an update: you cannot run `/reload-plugins` (it is a built-in command, not a
   tool). On the line after the update message, output exactly
   `Escribe /reload-plugins para cargarla (si sigue la versión antigua, reinicia Claude Code)`.

**Claude Code, loaded with `--plugin-dir`** (base directory is the user's repo checkout): there is
nothing to update; say that edits there apply with `/reload-plugins` and suggest `git pull`.

**Codex** (no `claude` command; the skill is a copied folder under `~/.agents/skills/`):
1. `tmp=$(mktemp -d)` and `git clone --depth 1
   https://github.com/Digital-Multimedia-Corp-S-L/imk-ia-prompt-optimizer-skill "$tmp/repo"`.
   If the clone fails, report the git error (likely no network or `git` missing) and stop.
2. Read the new version from `"$tmp/repo/.claude-plugin/plugin.json"`.
3. Compare with `diff -rq "$tmp/repo/skills/imk-prompt" <this skill's base directory>`. If there
   are no differences, remove `"$tmp"` and output exactly
   `No hay actualizaciones de imk-ia-prompt-optimizer-skill`; stop.
4. Otherwise replace the contents of this skill's base directory with
   `"$tmp/repo/skills/imk-prompt/"`, touching nothing outside that directory, and remove
   `"$tmp"`. Output exactly
   `Se ha actualizado imk-ia-prompt-optimizer-skill a la versión vX.Y.Z` and tell the user to
   restart Codex if the skill does not change.

The messages above (the two results and the reload reminder) are fixed text: print them exactly,
without translating or embellishing them. Only error reports and the Codex restart reminder are
free text, brief and in the user's language.

## Procedure

1. **Detect the target model.**
   - **With `--target codex`** (from Claude Code): first check that the Codex plugin is
     available (the `codex:codex-rescue` agent is listed). If it is not, stop and tell the user
     to install it and run `/codex:setup`; do not optimize. Otherwise the target is the model
     the plugin will run: read `model` and `model_reasoning_effort` from
     `$CODEX_HOME/config.toml` (default `~/.codex/config.toml`). If it cannot be read, write
     "modelo por defecto del plugin (no determinado)" and use the general guidance only.
   - **Without `--target`**: the target is your own model and version (for example
     "Sonnet 5.5"), and your own vendor decides the guidance: a Claude model → Claude guidance;
     an OpenAI/GPT/Codex model → Codex guidance.

   If you are not sure of your own model, say "versión no determinada". Copilot and others are
   not supported yet: say so and fall back to the general principles in `references/claude.md`.
2. **Load only the matching reference**: `references/codex.md` for Codex, otherwise
   `references/claude.md`, read with the file tool relative to this skill's folder. Do not load
   the other one.
3. **Decide whether to optimize at all.** If the request is a one-line question, a greeting
   or a trivial command, say "no hace falta optimizarlo" in the user's language and just run
   it. Optimizing short requests adds friction, not quality.
4. **Optimize it yourself** following the rules below, with the conversation context you
   already have.
5. **Present** the result (format below). Unless `--run`, end by asking for confirmation with the
   `AskUserQuestion` tool, so the user only has to press Enter to accept the default:
   - One question, in the user's language, e.g. "¿Lo lanzo así?", header "Lanzar".
   - Options, in this order: **"Lanzarlo así (Recommended)"** (first, so Enter selects it),
     **"Ajustarlo"** (the user then says what to change), **"Usar mi versión original"**.
   - Then act on the answer: run the optimized prompt, revise it and ask again, or run the
     user's original request.

   If `AskUserQuestion` is not available (for example in Codex), ask the same thing in plain
   text, make "lanzarlo así" the suggested default, and tell the user to answer "sí" to accept.
6. **Run** the approved prompt:
   - No `--target` → run it in this session as if the user had typed it.
   - `--target codex` → delegate with the Agent tool, `subagent_type: "codex:codex-rescue"`.
     Pass the optimized prompt **verbatim** as the task. Do not add `--model` or `--effort`
     (the plugin uses its defaults). If the task is read-only (review, diagnosis, research), say
     so in the forwarded text, because the plugin writes by default. When it returns, show
     Codex's output as it came and state that it was Codex. If the delegation fails because the
     plugin is not ready, say so (`/codex:setup`) and show the prompt for the user to paste into
     Codex instead.

## Optimization rules

- **Preserve intent.** Do not add requirements, scope or deliverables the user did not ask
  for. If you fill a gap, list it under *Assumptions* (with `--explain`) so the user can reject
  it; without `--explain` there is no such section, so ask instead of assuming.
- **Proportionate.** The optimized prompt should be about as long as the task deserves. A
  small request becomes a clear paragraph, not a template with ten XML blocks.
- **Same language as the request** for the prompt and for your notes; keep code, paths and
  identifiers untouched.
- **Ground it in the project.** Use real file names, commands and conventions you already
  know from the conversation or the repo instructions; never invent them. If something
  critical is unknown, say so under *Assumptions* or ask one question instead of guessing.
- **Self-contained for Codex.** Codex does not see this conversation, so a `codex` prompt must
  carry the context it needs (files, conventions, constraints, what is already decided).
- **Write for the target.** Apply the loaded reference: its structure, tone, autonomy and
  verification conventions, and the per-model adjustments for the exact target model.
- **Tell it what to do, not what to avoid**; give the reason behind each constraint; state
  what "done" looks like and how to verify it; avoid shouting (CAPS, "NEVER", "CRITICAL")
  unless it is a real invariant.
- **Safety stays.** Do not weaken risky or irreversible-action confirmations that the user's
  or project's instructions require.

## Output format

**Default (without `--explain`): only the prompt.** Print one line with the target, then the
prompt in one fenced block, and nothing else: no adjustments, no key changes, no assumptions, no
commentary.

```
**Optimizado para:** <vendor> · <exact model and version> (<modelo de la sesión | --target codex>)

<the prompt, in one fenced block>
```

Without the *Assumptions* section, do not fill gaps silently: if something critical is missing,
ask one question before optimizing instead of guessing. The staleness line (see the header
rules) is the only extra allowed, and only when it applies.

**With `--explain`**: print the full report below.

```
**Prompt optimizado**

- Optimizado para: <vendor> · <exact model and version, e.g. "Claude Sonnet 5.5"> (<modelo de la sesión | --target codex>)
- Lo ejecutará: <this session | Codex plugin · model and effort>
- Optimizado por: <session model>
- Guía aplicada: <reference file> (instantánea de <date in the file>)

<the prompt, in one fenced block>

**Ajustes para este modelo**
- <the model-specific adjustments you applied, e.g. "stop-condition para Sonnet a esfuerzo bajo">

**Cambios clave**
- <what you changed and why, 2-5 bullets>

**Suposiciones**
- <only if any; otherwise omit the section>
```

Do not write the confirmation question in the text: it goes in the `AskUserQuestion` call (or,
without that tool, as the last line: "¿Lo lanzo así? (sí / ajustar / usar mi original)").

Rules for the header:
- Always state the exact model and version. If you could not determine the version, write
  "versión no determinada" and say the general guidance was used instead of model-specific
  adjustments.
- If the reference snapshot is old (more than about three months), or the target model is newer
  than the reference covers, add one line recommending to check the vendor page.

Translate the headings to the user's language. With `--run`, print the target line and the prompt
briefly and proceed to run it without asking.
