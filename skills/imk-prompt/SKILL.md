---
name: imk-prompt
description: Rewrites a rough request into an optimized prompt for the AI model that will run it (Claude, or Codex/GPT through the Codex plugin), following that vendor's official prompting guidance, shows it for approval, then runs it. Use when the user invokes /imk-prompt or asks to "optimize this prompt" before executing it.
argument-hint: "[--target codex] [--run] <request>"
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

`--target` has **one value only: `codex`**, and exists only to use the Codex plugin from Claude
Code. There is no `--target claude`: Claude is simply the default. If the user passes any other
value, say that only `--target codex` exists and ask what they meant.

`--target codex` **requires the Codex plugin installed in Claude Code** (`codex@openai-codex`,
checked with `/codex:setup`). When set, after approval the execution is **delegated to that
plugin**, which runs Codex with its own default model. In a Codex session, `--target codex`
makes no sense (you already are Codex): ignore it and say so.

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
  for. If you fill a gap, list it under *Assumptions* so the user can reject it.
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

Translate the headings to the user's language. With `--run`, print the header and the prompt
briefly and proceed to run it without asking.
