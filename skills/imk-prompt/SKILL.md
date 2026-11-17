---
name: imk-prompt
description: Rewrites a rough request into an optimized prompt for the AI model that will run it (Claude or Codex/GPT), following that vendor's official prompting guidance, shows it for approval, then runs it. Use when the user invokes /imk-prompt or asks to "optimize this prompt" before executing it.
argument-hint: "[--target claude|codex] [--model <optimizer-model>] [--run] <request>"
disable-model-invocation: true
---

# imk-prompt

Turn the user's rough request into a prompt tuned for the model that will execute it, show it,
and only then run it. Never run the original request before the user has seen the optimized one
(unless `--run`).

## Arguments

Parse these from the start of the arguments; everything else is the **raw request**.

| Argument | Meaning | Default |
|---|---|---|
| `--target claude\|codex` | Vendor guidance to apply | auto-detect (below) |
| `--model <alias>` | Model that does the *optimizing* (e.g. `opus`) | the current session model |
| `--run` | Run the optimized prompt right away, no confirmation | off: show it and ask |

## Procedure

1. **Detect the target.** Unless `--target` is given, decide from your own identity: Claude
   models (running in Claude Code or any Claude-based tool) → `claude`; OpenAI/GPT/Codex models
   → `codex`. Also note the exact target model (for example "Sonnet 5.5") when you know it.
   If you are not sure, ask the user in one short question. Copilot and others are not
   supported yet: say so and fall back to the general principles in `references/claude.md`.
2. **Load only that reference**: `references/claude.md` or `references/codex.md`, read with
   the file tool relative to this skill's folder. Do not load the other one.
3. **Decide whether to optimize at all.** If the request is a one-line question, a greeting
   or a trivial command, say "no hace falta optimizarlo" in the user's language and just run
   it. Optimizing short requests adds friction, not quality.
4. **Optimize** following the rules below, either:
   - **Same model** (no `--model`, or it equals the session model): do it yourself, with the
     conversation context you already have.
   - **Different model** (`--model`): spawn one general-purpose subagent with that `model`.
     It cannot see this conversation, so brief it fully: the raw request, the target vendor and
     model, the full text of the loaded reference, and a short summary of only the context
     that matters (files, conventions, constraints, what was already decided). Tell it to
     return the optimized prompt plus a list of assumptions, nothing else. If your environment
     has no subagent tool with a model option, say so and optimize with the session model.
5. **Present** the result (format below). Unless `--run`, stop and wait for approval or edits.
6. **Run** the approved prompt as if the user had typed it, in the session model.

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

- Optimizado para: <vendor> · <exact model and version, e.g. "Claude Sonnet 5.5"> (<detectado | --target>)
- Optimizado por: <model that did the optimizing> (<sesión | --model>)
- Guía aplicada: <reference file> (instantánea de <date in the file>) + <model page, only if fetched>

<the prompt, in one fenced block>

**Ajustes para este modelo**
- <the model-specific adjustments you applied, e.g. "stop-condition para Sonnet a esfuerzo bajo">

**Cambios clave**
- <what you changed and why, 2-5 bullets>

**Suposiciones**
- <only if any; otherwise omit the section>

¿Lo lanzo así, lo ajustas o prefieres tu versión original?
```

Rules for the header:
- Always state the exact model and version. If you could not determine the version, write
  "versión no determinada" and say the general guidance was used instead of model-specific
  adjustments.
- If the optimizer model differs from the target, say so plainly; the user may have chosen it
  on purpose.
- If the reference snapshot is old (more than about three months) or a newer model than the
  reference covers is the target, add one line recommending to check the vendor page.

Translate the headings to the user's language. With `--run`, print the header and the prompt
briefly and proceed to run it without asking.
