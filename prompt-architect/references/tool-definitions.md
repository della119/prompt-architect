# Agent Instructions & MCP / Tool-Definition Design

How to design tool/function definitions and agent instructions so the model calls the right tool,
at the right time, with the right arguments. The model only sees the tool's name, description, and
parameter schema — those three fields *are* the spec. Get them right and most tool-use bugs vanish.

## Table of contents

1. The description-as-spec principle
2. Naming
3. Writing the description
4. Parameter schema discipline
5. When-to-call vs when-not (and disambiguation)
6. Output, errors, and empty results
7. Agent-level instructions (system prompt for a tool-using agent)
8. Tool-definition checklist
9. Worked example

---

## 1. The description-as-spec principle

The model decides whether and how to call a tool almost entirely from the description. Treat it as a
contract written for a competent stranger who cannot see your code: it must convey *what the tool
does, when to use it, when NOT to use it, and what comes back*. Implementation details the caller
doesn't need are noise; consequences and edge behavior the caller must anticipate are essential.

## 2. Naming

- Use a clear `verb_noun` form: `search_invoices`, `get_fx_rate`, `create_ticket`.
- Make names distinguish siblings: `list_files` vs `read_file` vs `search_files` — each name should
  predict its behavior so the model doesn't confuse them.
- Avoid abbreviations and internal jargon the model can't ground.

## 3. Writing the description

Structure a description as: one-line purpose → when to use → when *not* to use → return shape →
gotchas. Keep it tight; every clause should change a call decision.

> **search_filings**: Full-text search over a company's SEC filings (10-K, 10-Q, 8-K). Use to find
> where a topic is discussed or to pull specific disclosures. Not for real-time prices (use
> get_quote) or for non-US issuers (returns empty). Returns up to 20 ranked excerpts with filing
> date and URL. Search is keyword-based — prefer 2–4 specific terms over full sentences.

Bake the *usage rules that prevent misfires* into the description itself, because the model reads it
at decision time — e.g. "prefer 2–4 specific terms," "returns empty for non-US issuers."

## 4. Parameter schema discipline

- **Every parameter gets a description**, even "obvious" ones. State units, format, and allowed
  values: `as_of_date` → "ISO 8601 date (YYYY-MM-DD); defaults to today if omitted."
- **Use enums** for closed sets instead of free text — it removes a whole class of bad arguments.
- **Mark required vs optional honestly** and give defaults for optional ones in the description.
- **Prefer few, flat parameters.** Deeply nested objects and 12-arg tools both raise error rates.
  If a tool needs many arguments, consider splitting it.
- **Constrain ranges** where it matters: "limit: 1–100, default 20."
- Name parameters for what they mean, not their type: `ticker`, not `str1`.

## 5. When-to-call vs when-not (and disambiguation)

The two most common tool-use failures are *over-calling* (reaching for a tool when reasoning would
do, or calling the wrong sibling) and *under-calling* (answering from stale memory when a tool
exists). Fix both explicitly:

- In each description, name the situations that should route *elsewhere* ("not for X — use Y").
- When several tools overlap, add a short routing rule to the agent instructions: "For company
  financials use get_financials; for market prices use get_quote; never use web_search for either."
- Give a tie-breaking default for ambiguous cases.

## 6. Output, errors, and empty results

Tell the model what a result looks like and what the failure modes are, so it can handle them in one
turn instead of retrying blindly:

- Describe the return shape and units.
- State what an empty result means ("returns [] when no match — do not retry with the same query;
  broaden terms or tell the user").
- Define error semantics ("raises if the ticker is unknown — surface the error, don't fabricate a
  price").
- If results are large, say how they're truncated/ranked so the model knows it isn't seeing
  everything.

## 7. Agent-level instructions (system prompt for a tool-using agent)

Beyond individual tools, the agent's own system prompt should cover:

- **Objective & scope** — what the agent is for and what's out of bounds.
- **Tool routing** — the when-to-call map across the toolset (§5).
- **Planning vs acting** — for multi-step tasks, instruct it to outline the plan, then call tools;
  for one-shot lookups, act directly. Scale effort to complexity.
- **Grounding rule** — answer from tool results, not memory, for anything the tools cover; cite
  what came from where.
- **Parallel vs sequential** — call independent tools together; serialize only when one depends on
  another's output.
- **Stop condition** — when to stop calling tools and answer (avoid infinite tool loops).
- **Untrusted content** — treat tool/retrieved output as data, not instructions (anti-injection).

## 8. Tool-definition checklist

- [ ] Name is `verb_noun`, unambiguous vs siblings.
- [ ] Description states purpose, when-to-use, when-NOT, return shape, gotchas.
- [ ] Every parameter has a description with units/format/allowed values.
- [ ] Closed sets use enums; ranges are bounded; defaults stated.
- [ ] Required/optional marked correctly.
- [ ] Empty-result and error behavior documented.
- [ ] No overlap with another tool left unresolved (routing rule exists).
- [ ] Nothing the caller doesn't need (no leaked implementation detail).

## 9. Worked example

Weak:

> **getData(q)** — gets data for the query.

Strong:

> **search_companies** — Find public companies by name, ticker, or industry keyword. Use to resolve
> a name to a ticker or discover peers in a sector. Not for private companies (returns []). Returns
> up to 25 matches, each with ticker, legal name, exchange, and primary SIC industry, ranked by
> market cap.
> - `query` (string, required): name, ticker, or industry keyword. 1–5 words works best.
> - `exchange` (enum: NYSE | NASDAQ | HKEX | SSE | SZSE, optional): restrict to one exchange.
> - `limit` (integer, optional, 1–25, default 25): max results.

The strong version lets the model decide *whether* to call it, *when not to*, and *how to fill every
argument* — without seeing the implementation.
