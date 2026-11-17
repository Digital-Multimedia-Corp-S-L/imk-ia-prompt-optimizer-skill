# Prompting Codex / GPT-5 — reference

Condensed from OpenAI's official guidance and the `openai/codex-plugin-cc` prompting skill,
read on **2026-11-17**. It is a snapshot: the vendor pages are the living source, and OpenAI's
advice changes between model versions. When the task is big and the pages are reachable, fetch
them and prefer them over this file.

- Codex prompting guide: https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide
- GPT-5 prompting guide: https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide
- Operator-style prompt blocks: https://github.com/openai/codex-plugin-cc/blob/main/plugins/codex/skills/gpt-5-4-prompting/SKILL.md

## Principles

1. **Operator, not collaborator.** Compact, block-structured prompts with XML tags: state the
   task, the output contract, the follow-through default, and only the extra constraints that
   matter.
2. **One clear task per run.** Split unrelated asks (review, then fix, then docs) into
   separate runs.
3. **Say what done looks like.** Codex will not infer the end state.
4. **Four elements:** goal, context, constraints, done-when.
5. **Bias to action.** Default to implementing with reasonable assumptions; ask only when
   truly blocked, or when a missing detail changes correctness, safety or an irreversible
   action.
6. **Carry the work through** implementation, verification and a short explanation within
   the turn. Never end on a plan alone.
7. **Tighten the contract before raising reasoning.** Do not write "think harder"; add
   verification and grounding rules instead. Medium effort is the interactive default; high
   or xhigh only for the hardest, long tasks. Effort is a setting, not prompt prose.
8. **Ground claims.** For review, research and root-cause work: ground every claim in
   provided context or tool output; label hypotheses; do not guess repository facts, fetch
   them or state what is unknown.
9. **Keep changes narrow.** No unrelated refactors or renames; call out risky or
   irreversible actions first; never revert changes it did not make.
10. **Tool use.** Prefer dedicated tools over raw shell; `rg` over `grep`; batch independent
    reads in parallel; `apply_patch` for single-file edits.
11. **Output:** concise, friendly-teammate tone; lead with the change or the findings (by
    severity for reviews); reference files by path; no large file dumps; few, flat bullets.
12. **Avoid** upfront-plan/preamble/status-chatter instructions on older Codex models (they
    can stop the run early), long over-specified developer messages, mixed unrelated jobs, and
    shouting.

## Reusable blocks (use only the ones the task needs)

| Block | Add it when | Text |
|---|---|---|
| `<task>` | always | the concrete job, relevant repo or failure context, and expected end state |
| `<structured_output_contract>` | the response shape matters | Return exactly the requested output shape and nothing else. Keep it compact. Highest-value findings first. |
| `<compact_output_contract>` | you want concise prose | Keep the final answer compact and structured. No long scene-setting or repeated recap. |
| `<default_follow_through_policy>` | it should act without asking routine questions | Default to the most reasonable low-risk interpretation and keep going. Stop to ask only when a missing detail changes correctness, safety, or an irreversible action. |
| `<completeness_contract>` | debugging, implementation, multi-step | Resolve the task fully before stopping. Do not stop at the first plausible answer. Check follow-on fixes, edge cases and cleanup. |
| `<verification_loop>` | correctness matters | Before finalizing, verify the result against the task requirements and the changed files or tool outputs. If a check fails, revise instead of reporting the first draft. |
| `<missing_context_gating>` | it might otherwise guess | Do not guess missing repository facts. If required context is absent, retrieve it with tools or state exactly what remains unknown. |
| `<grounding_rules>` | review, research, root cause | Ground every claim in the provided context or your tool outputs. Do not present inferences as facts. Label hypotheses. |
| `<citation_rules>` | external research or quotes | Back important claims with citations or explicit references to the sources you inspected. Prefer primary sources. |
| `<action_safety>` | write-capable or broad tasks | Keep changes tightly scoped to the stated task. Avoid unrelated refactors, renames, or cleanup unless required for correctness. Call out any risky or irreversible action before taking it. |
| `<tool_persistence_rules>` | long, tool-heavy tasks | Keep using tools until you have enough evidence to finish confidently. Do not abandon the workflow after a partial read when another targeted check would change the answer. |
| `<research_mode>` | exploration, comparison, recommendation | Separate observed facts, reasoned inferences and open questions. Breadth first, then deeper only where the evidence changes the recommendation. |
| `<dig_deeper_nudge>` | review, adversarial inspection | After the first plausible issue, check second-order failures, empty-state behavior, retries, stale state and rollback paths before finalizing. |

Typical combinations: coding or debugging → `task` + `completeness_contract` +
`verification_loop` + `missing_context_gating`; review → `task` + `grounding_rules` +
`structured_output_contract` + `dig_deeper_nudge`; research → `task` + `research_mode` +
`citation_rules`; any write task → add `action_safety`.

## Example

```
<task>
Fix the failing `test_x` in backend/tests/Feature/Foo.php: the endpoint returns 500 when the
payload has no `name`. Expected end state: the test passes and no other test regresses.
</task>

<default_follow_through_policy>
Keep going until you have identified the root cause with evidence; ask only if blocked.
</default_follow_through_policy>

<verification_loop>
Run the failing test and the related suite before finalizing; if a check fails, revise.
</verification_loop>

<action_safety>
Touch only what the fix needs. No unrelated refactors.
</action_safety>

<structured_output_contract>
Return: 1. root cause, 2. what you changed (files), 3. test results.
</structured_output_contract>
```

## Sources

- Codex Prompting Guide — https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide
- GPT-5 prompting guide — https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide
- openai/codex-plugin-cc, `gpt-5-4-prompting` skill — https://github.com/openai/codex-plugin-cc/blob/main/plugins/codex/skills/gpt-5-4-prompting/SKILL.md
