---
name: skill-from-domain
description: Build a domain skill for an agent that works over MCP tools, from a ticket or an expert's workflow, by the three moves Elicit, Diff, Encode. Reads the written process at its source, maps every requirement to the real tool fields in code, crosses out what the harness already says, writes only the difference, and turns the acceptance tests into evals. Use when asked to "draft the skill for <ticket>", "turn this workflow into a skill", or build a skill whose data comes from named tools.
---

# Skill from domain

The model already knows the domain method. The skill is the difference between the expert's process and what the agent does cold. Produce four files: `SKILL.md` under one screen, `references/fields.md`, `evals/evals.json`, and a process map with the owner's open questions. Writing, compression and eval rules: `references/writing.md`. Anthropic's `skill-creator`, when installed, runs the evals; nothing else is required.

## 1. Read the written process at its source

Fetch the ticket or document itself, by tool, not from a paste. Keep Goal, Scope out, Requirements, Acceptance Tests and Dependencies verbatim in the process map. The acceptance tests are the evals. The failure and edge-case rows are the scar list: the skill exists for them; the harness mostly passes the easy rows already.

## 2. Find the agent's real data path in code

Locate the tool definitions the ticket names and the schema their response comes from: the tool description text, the response message, its nested types. Read the schema, not the ticket's wish list. Build the requirement-to-field table: ticket field, response field by its JSON name, status `present`, `partial` or `none`. Every `none` becomes a refuse line, never an estimate. Record what the tools cannot resolve (an entity kind the name resolver lacks) and the enum values the user's words map to.

## 3. Cross out what the harness already says

Read the agent's system prompt and every tool description. A rule already there is not a skill line. List them in the process map under "harness already does", so a reviewer sees why they are absent from the skill.

## 4. Write the expert's walk

One row per step: what the expert does, the check, the field that answers it, what passes to the next step. Tag each cell `V` verified in code, `T` from the ticket, `?` open. Below it, the four tacit-layer question families for the owner: first look, ignored, scars, hand-off. Then one engineering question per `?`. When the user is the expert (their own AWS account, their own weekly review), ask the four families now, one at a time, and write the answers into the walk before drafting. Otherwise draft without waiting; the answers become the next version.

## 5. Encode only the diff

Sections, in this order, each kept only if it names a misfire it prevents:

- Calls: the defaults and gaps the tool text omits (default date, default basis, what "all" means, where a second figure sits on the same row).
- Where the fields are: the table from step 2, short; the full map in `references/fields.md`.
- Arithmetic: what the agent may compute from returned rows, labelled as computed, and what it must refuse. Decide this line explicitly or the agent decides per question.
- Refuse, and say why: one line per `none` and per edge case, with the words the agent should say.
- Data quality: stale, missing, ambiguous; flag, give no value, never substitute.
- Large results: count and totals first, offer a filter or export, never a silent cut.
- Say a number: one real example from the ticket; every number carries entity, field, currency, date.
- Read only, when the tools are.

Anchor, never instruct: name the tool and the field, not the JSON mechanics the tool text already gives. Positive targets; a prohibition only where it cannot be phrased positively. Description last: trigger plus outcome, third person, no procedure.

## 6. Evals

`evals/evals.json` in the schema given in `references/writing.md`: `id`, `prompt` verbatim from the ticket, `expected_output` in one line, two to four `expectations` each stated about the output or the tool calls ("ResolveNames called before GetHoldings", "no figure given for unfunded par", "the answer asks which"). Add one near-miss prompt that should not trigger the skill.

## 7. Done when

The field table has no `?` without an owner question. `SKILL.md` is under 60 lines and every line names its misfire. The evals ran with and without the skill, on a mock server when no live one is reachable, and the delta is written in the process map. `claude plugin validate` passes once the skill sits in its plugin. One named person ran it.

## Scars from the first run (a holdings skill over read-only MCP tools, 2026-11-09)

- The name resolver had no issuer kind. Issuer questions match on the row's issuer name; a cold agent calls the resolver and gets null.
- Revolver and delayed draw were booleans on the instrument. The name "TLB" says nothing; the cold agent infers from names.
- Half the ticket's fields had no field in the response (ratings, lien, cov-lite, unfunded par, defaulted). The skill refuses them; the ticket owner gets the list as engineering questions.
- The arithmetic line was the hard decision: sums and shares yes, weighted averages never. Unwritten, the agent decides differently each time.
- No live MCP on the laptop. The cold run waited on the mock server; the draft did not.
