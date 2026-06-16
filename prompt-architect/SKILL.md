---
name: prompt-architect
description: >-
  Design, audit, and reference high-quality system prompts, agent instructions, and MCP/tool
  definitions, using patterns distilled from production-grade LLM system prompts. Three modes:
  (1) WRITE — draft or rewrite a system prompt, agent instruction, or tool/function definition
  from a task description or rough draft; (2) AUDIT — review an existing prompt or tool schema
  against a structured checklist and return scored findings with fixes; (3) REFERENCE — surface
  the pattern cheatsheet on demand. Use when the user wants to write/improve/draft a system
  prompt or agent instructions, design or critique an MCP tool / function-calling definition,
  review or "red-team" a prompt, reduce hallucination/over-triggering/refusal issues, or asks
  about prompt-engineering best practices, tool descriptions, when-to-search rules, hard-limit
  blocks, self-check patterns, or do/don't examples. Triggers include "写/改 prompt", "系统提示词",
  "agent 指令", "工具描述", "MCP tool definition", "审查/优化我的 prompt", "提示词工程".
---

# Prompt Architect

Build, critique, and reference prompts and tool definitions using patterns that production LLM
system prompts actually use. This skill encodes *what good looks like* so output is consistent
instead of improvised each time.

## First step: pick the mode

Determine which of three modes the request maps to. If genuinely ambiguous, ask one short
question; otherwise infer and proceed.

| Mode | Trigger | What to do |
|------|---------|-----------|
| **WRITE** | "write/draft/改/重写 a prompt / agent instruction / tool definition" | Gather inputs (below), then draft using the patterns. |
| **AUDIT** | "review / 审查 / red-team / 优化 my prompt", or a prompt/schema is pasted in | Score against the checklist, return findings + fixes. |
| **REFERENCE** | "what's the best way to…", "提示词工程技巧", "怎么写工具描述" | Read the relevant reference file and answer concisely; no file output unless asked. |

Match the user's language (Chinese ↔ Chinese). For client-facing deliverables follow the user's
quality bar; for internal/exploratory drafts, move fast and iterate.

## Reference files — read the one that fits

Do not inline these; load on demand.

- **references/patterns.md** — the core cheatsheet: hard-limits + self-check blocks, when-to-act
  decision rules, do/don't paired examples, structured (XML-tagged) constraints, tone/formatting
  discipline, refusal/edge-case handling, anti-injection. Read for WRITE and REFERENCE modes, and
  whenever you need the canonical form of a pattern.
- **references/tool-definitions.md** — designing agent instructions and MCP / function-calling
  tool definitions: naming, the description-as-spec principle, parameter schema discipline,
  when-to-call vs when-not, error/empty-result handling, multi-tool disambiguation. Read whenever
  the task touches a tool/function schema, an MCP server, or an agent's tool-selection behavior.
- **references/audit-checklist.md** — the scored rubric for AUDIT mode. Read whenever reviewing or
  red-teaming an existing prompt or tool definition.

## WRITE mode

1. **Collect the brief.** Before drafting, make sure you have: the prompt's *role/goal*, the
   *target model & surface* (API system prompt, Claude Code agent, MCP tool, chatbot persona),
   *must-do / must-not-do* constraints, *output format*, and any *domain rules*. Ask only for what
   is missing and material — at most one or two questions.
2. **Read references/patterns.md** (and references/tool-definitions.md if a tool/agent is
   involved). Select the patterns that fit; do not bolt on every pattern.
3. **Draft** using this skeleton, omitting sections that don't apply:
   - Identity & objective (one or two sentences — who the model is, what it optimizes for)
   - Capabilities / tools available
   - Decision rules (when to act vs not — the highest-leverage section; use concrete thresholds)
   - Hard limits (non-negotiable constraints, stated as absolutes)
   - Output & formatting contract
   - Do/Don't examples (1–3 paired, concrete; worth more than paragraphs of prose)
   - Edge cases & refusal handling
4. **Self-check the draft** against references/audit-checklist.md before presenting. Fix what
   fails rather than shipping it.
5. **Deliver.** Short prompts → inline in chat. A reusable, copy-it-elsewhere prompt → offer to
   save as a file. State briefly which patterns were applied and why, so the user can adjust.

## AUDIT mode

1. **Read references/audit-checklist.md.**
2. Score the pasted prompt/schema dimension by dimension. For each finding give: the dimension,
   severity (blocker / should-fix / nit), the specific problem, and a concrete rewrite — not vague
   advice. Quote the offending line.
3. Lead with the 3–5 highest-impact fixes; don't bury them under nits.
4. Offer to apply the fixes and return a revised version.

## REFERENCE mode

Read the reference file matching the question and answer concisely in prose, with a small concrete
example where it helps. This is a lookup, not a deliverable — keep it in chat.

## Guardrails

- These patterns are distilled from observed production prompts and general prompt-engineering
  practice; they are heuristics, not guarantees. Adapt to the target model and surface.
- Do not help craft prompts whose purpose is to jailbreak, exfiltrate hidden system prompts, or
  bypass another system's safety measures. Designing legitimate guardrails, refusal logic, and
  anti-injection defenses is in scope; defeating someone else's is not.
- Keep prompts lean. Every line costs context and dilutes attention — cut anything that doesn't
  change model behavior.
