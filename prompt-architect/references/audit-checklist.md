# Prompt / Tool-Definition Audit Rubric

Use in AUDIT mode. Score the target across the dimensions below. For each issue, report:
**dimension · severity · the offending line (quoted) · concrete rewrite**. Severity =
blocker (will misbehave) / should-fix (degrades quality) / nit (polish). Lead with the highest-
impact 3–5, then the rest.

## How to score

For each dimension, mark Pass / Weak / Fail and justify in one line. Don't give vague advice
("be clearer") — always show the rewrite.

## Dimensions — system prompts & agent instructions

1. **Objective clarity** — Is there a one/two-sentence identity + what it optimizes for? Fail if the
   identity is adjective-only ("helpful, smart") with no behavioral steer.
2. **Decision rules** — Are when-to-act and when-NOT both specified with concrete, testable triggers
   and a tie-breaking default? This is usually where the biggest wins are. Flag any "when
   appropriate / as needed" with no criterion.
3. **Hard limits** — Are non-negotiables stated as absolutes, quantified where possible, and visually
   separated from soft guidance? Flag unenforceable rules ("keep it short" with no bound).
4. **Self-check** — For outputs with recurring checkable failure modes, is there a pre-send
   checklist? Flag its absence on high-stakes outputs (citations, safety, format contracts).
5. **Examples** — Are there 1–3 concrete do/don't pairs at the *boundary* cases, each with a
   rationale? Flag prose-only rules that would benefit from a paired example.
6. **Structure** — For long prompts, are rules grouped under named/XML tags so each block is
   referenceable? Flag a flat wall of text where a key rule is likely to get lost.
7. **Output & format contract** — Is the default format and its exceptions stated (prose vs bullets,
   length bound, language)? Flag prompts that will over-format by default.
8. **Refusal & edge cases** — Are missing-input, out-of-scope, partial-capability, and harmful-
   request behaviors pre-specified with a path forward? Flag improvised-refusal risk.
9. **Anti-injection / hierarchy** — If the prompt consumes external/untrusted text, is precedence
   stated and is that text marked as data-not-instructions? Blocker if it processes untrusted input
   with no defense.
10. **Effort scaling** — Are there anchors for matching effort to task complexity? Flag prompts that
    will over-work simple asks or under-work hard ones.
11. **Positive vs negative phrasing** — Is ordinary behavior stated as what *to do*, with negation
    reserved for genuine red lines (safety, destructive/irreversible, order-dependent)? Flag "don't
    skip / don't forget"-style rules for routine behavior — rewrite positively. Don't flag "never"
    on actual hard limits; that's correct.
12. **Uncertainty permission** — Is the model explicitly allowed to say "I don't know," mark low
    confidence, or ask for missing input? Flag prompts that demand an answer with no sanctioned
    fallback — a hallucination risk, especially alongside a no-fabrication limit with no escape hatch.
13. **Task decomposition** — For multi-part tasks, is the model told to break the work into ordered
    subtasks (or outline-then-execute)? Flag complex one-pass instructions that will drop steps.
14. **Context sufficiency** — Does the prompt give the model the background it needs (domain terms,
    conventions, what the inputs mean), or does it assume knowledge the model won't have at runtime?
    Flag missing context that will force guessing.
15. **Example bias** — If examples are present, do they vary in incidental features so the model
    learns the intended pattern, not a surface trait (same length/format/answer every time)? Flag
    example sets that will be over-imitated.
16. **Leanness** — Is every line earning its context cost? Flag redundancy, restated rules, and
    decorative text that doesn't change behavior.
17. **Contradictions** — Do any two rules conflict (e.g. "always cite" + "be extremely brief")?
    Blocker — the model will pick unpredictably. Flag and propose the resolution/precedence.

## Dimensions — tool / MCP definitions

(Apply when the target is a tool/function schema. See tool-definitions.md for the full standard.)

18. **Name** — `verb_noun`, unambiguous vs siblings.
19. **Description completeness** — purpose + when-to-use + when-NOT + return shape + gotchas.
20. **Parameter descriptions** — every parameter documented with units/format/allowed values.
21. **Schema discipline** — enums for closed sets, bounded ranges, honest required/optional, stated
    defaults, not over-nested.
22. **Failure semantics** — empty-result and error behavior documented so the model handles them in
    one turn.
23. **Routing** — overlap with sibling tools resolved by an explicit rule.

## Output template

```
## Audit: <name>

**Top fixes**
1. [BLOCKER] <dimension> — "<quoted line>" → <rewrite>
2. [SHOULD-FIX] <dimension> — "<quoted line>" → <rewrite>
...

**Dimension scores**
Objective: Pass/Weak/Fail — <one line>
Decision rules: ...
(...)

**Revised version available on request.**
```
