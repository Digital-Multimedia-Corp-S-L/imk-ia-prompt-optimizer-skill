# Prompting Claude — reference

Condensed from Anthropic's official guidance, read on **2026-11-17**. This is a snapshot: the
vendor pages are the living source. When the target model's page is reachable and the task is
big, fetch it and prefer it over this file.

- General: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- Per model: `.../prompt-engineering/prompting-claude-<model>` (for example
  `prompting-claude-opus-5-5`, `prompting-claude-sonnet-5-5`, `prompting-claude-haiku-5-5`)

## Principles for every current Claude model

1. **Be clear and direct.** Think of Claude as a brilliant new employee with no context on
   your norms. Test: would a colleague with minimal context be confused by this prompt? If
   you want more than the strict task, ask for it explicitly.
2. **Give the why.** Context or motivation behind an instruction lets Claude generalize
   ("read aloud by a text-to-speech engine, so no ellipses" beats "NEVER use ellipses").
3. **Examples steer format and tone.** 3-5 relevant, diverse examples inside `<example>` /
   `<examples>` tags.
4. **XML tags** to separate instructions, context, examples and inputs
   (`<instructions>`, `<context>`, `<input>`), with consistent names. Skip them for short
   prompts.
5. **Role** in one sentence when it focuses tone or behavior.
6. **Long inputs first, question last.** Put documents at the top, the instruction at the end.
7. **Say what to do, not what to avoid**; match the prompt's own formatting to the output
   you want (a prompt without markdown nudges output with less markdown).
8. **Sequential steps** as a numbered list when order or completeness matters.
9. **Define done and how to verify** (run the tests / build / the changed command).
10. **Don't over-prompt.** Blanket "always use X" / "if in doubt, use the tool" cause
    overtriggering on current models. Remove aggressive emphasis; keep one clear sentence.
11. **Tame overeagerness when coding**: only changes directly requested or clearly necessary;
    no extra abstractions, docs, defensive code or refactors for hypothetical needs.
12. **Don't ask the model to write its reasoning into the answer** (it can be declined);
    ask for a short explanation or a summary of actions instead.
13. **Subagents**: current models delegate natively and can overuse it. For simple or
    single-file work, say to work directly.
14. **Effort is the main thinking control**, not "think harder/less" prose.

## Per-model adjustments (check the exact target)

| Target | What to adapt in the prompt |
|---|---|
| **Opus 5.5** | Strongest at long, multistep, autonomous work; needs less hand-holding. Ask it to explore broadly *before* acting on loosely specified, multi-source tasks. For unattended runs, name the early stops you do not want (ending a turn with a summary that announces the next step instead of doing it) and the stops you do want (blocked on the user). Treat any "think carefully" line as unnecessary. |
| **Opus 5** | Verbose by default and effort does not shorten it: ask for conciseness explicitly. Delegates to subagents readily: say when not to. |
| **Sonnet 5.5** | At low/medium effort it may stop to check in: add "keep working until everything asked is done; only stop to ask when you can't go on or before a risky step". It adds tests/docs/files unasked: add "when done and checked, stop and report; don't add features, tests, files, docs or refactors that weren't asked for; mention them at the end". For open-ended asks ("show me what you can do"), say "give ideas/a plan and stop until told to go ahead". On code changes, require a real check (tests/type-check/build) before reporting done. For JSON answers needing multi-step working, add "Think the problem through before you answer." Prefer explicit, specific instructions over implied ones. |
| **Haiku 5.5** | Smaller: be more explicit and step-wise, keep prompts focused. In long agent prompts at low effort it can stop early: add the same keep-working / report-when-done lines as Sonnet. Require verification of code changes. Give today's date if search is involved. Keep reasoning out of user-facing text. |
| **Fable 5.x / Mythos 5.x** | Different behaviors by version (effort calibration, approvals, progress updates, final reports). Fetch the matching page before optimizing a large task. |

## Prompt shapes that work for a request typed in a coding agent

A user message, not an API system prompt, so fold the guidance into the message:

```
<task>
<one or two sentences: the concrete outcome, in the project's own terms>
</task>

<context>
<why it is needed; the files / modules / conventions that matter; what is already decided>
</context>

<constraints>
<scope limits, things not to touch, risky actions that need confirmation first>
</constraints>

<done_when>
<observable end state and the check to run: tests, build, command>
</done_when>
```

Drop any block that adds nothing. For a small task a plain paragraph is better.

## Sources

- Prompting best practices — https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- Prompting Claude Opus 5.5 — https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5
- Prompting Claude Sonnet 5.5 — https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5
- Prompting Claude Haiku 5.5 — https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5
