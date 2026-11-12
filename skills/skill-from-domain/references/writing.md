# Writing rules for a skill the agent will read

Bundled so the skill travels without any other skill installed. Read before step 5 (Encode).

These rules follow Anthropic's `skill-creator`, the official guidance, as read on 2026-11-11. Where this file and `skill-creator` disagree, `skill-creator` wins and this file gets corrected.

## Description

The description decides whether the agent loads the skill, so every when-to-use fact goes here and none goes in the body.

- Form: what the skill does, then when to use it, third person, no procedure. "Use when …" names the contexts and the words a user says, including requests that never name the skill's subject outright.
- Limits: at most 1024 characters and no angle brackets. `skill-creator`'s validator rejects either, and a skill that fails it cannot be packaged.
- Lean toward firing. Claude tends to under-trigger: it skips a skill that would have helped. Name more contexts rather than fewer, a little pushy, within about 100 words.
- Audit by misfire. A word stays when it prevents one of three misfires: the skill stays silent on a request it should take, it fires on a request it should leave alone, or it promises a result the body does not deliver. A word that prevents none is cut.
- Measure, do not count. A description is right when the skill fires on the right prompts, not when it is short. Write about twenty queries a real user would type, half that should trigger and half near-misses that share the skill's words but need something else, and run them. `skill-creator`'s description optimizer does this and keeps the version that wins on queries it was not tuned on. Make the queries detailed and multi-step, because a one-step request may trigger no skill at all, however good the description.

## Body

- Give the reason with the rule. Write each rule as a plain sentence and say why it matters: "Give no value for a price older than 30 days, because a stale mark reads as today's." An agent that knows the reason can handle the case the rule did not name; an agent holding a bare rule can only apply it literally. When a rule seems to need capital letters to hold, its reason is what is missing.
- Anchor or spell out. Ask once per skill: is there a named method, standard or tool the model already knows, and is it still the one the expert uses? Yes: anchor, never describe. Name the method, the tool, the field, the standard, and whose it is; writing out how the anchor works fights what the model knows and overweights whichever sub-step got written. No (frontier work, or the known method is stale): the expert's walk from the process map is the anchor, so write the steps in full and in order, because there is nothing to point at.
- Only the diff. Every line names the misfire it prevents in the cold run. A line the agent would obey anyway is a no-op: delete it whole. Cutting whole lines keeps the skill lean; clipping words out of the lines that stay removes their reasons.
- General, not fitted. Write each rule for the kind of request, not for one test prompt. A rule that only makes sense for a single eval case is a patch: find what the failing cases share and write that, with its reason. A skill that passes its own examples and fails the next real question has not done its job.
- Positive targets. Say what to do; keep a prohibition only where no positive form exists, and pair it with the words the agent should say instead.
- Plain sentences, imperative form. Terse fragments are fine in lookup material (a field table, a list of defaults), where nothing needs a reason. Wherever text is shortened, every path, number, name, list item and negation survives verbatim: a version missing a fact the longer one stated is a summary, not compression.
- BLUF first. One term per thing; no synonyms for the same field or state. Use headers and labels where they help the agent find a section; force no single shape on every line.
- One real example of the output, filled, from the source material. Add a fixed template only when the format must be exact.
- Paths, tool names, field names rooted and in backticks.
- Lean, not short. `SKILL.md` keeps only what the agent needs on every run. There is no line target, because a count rewards cutting reasons. The ceiling is 500 lines; near it, move detail into `references/` and say in `SKILL.md` when to read each file. The full field map is reviewer evidence: it goes beside the process map, outside the skill folder. `references/` holds only what the agent itself reads.

## Process map

Tag every cell: `V` verified in code or data (cite the path), `T` from the ticket or the owner's words, `?` open with a named question. A guess gets no tag; it does not go in.

## Evals

Each test case is a folder under `evals/` in the skill directory. This is the layout `skill-creator` writes and `claude plugin eval` reads, so one set of cases runs in both. The format is strict: copy the three blocks as they are and change only the names and the body text.

```
<skill>/
  SKILL.md
  evals/
    asks-which-on-ambiguous-name/     case name: says what is checked
      prompt.md                       settings block, then the prompt
      case.yaml                       the name, and what a right answer does
      graders/                        one check per file
        01-lists-every-match.md
        02-gives-no-figure.md
```

Case name: lowercase letters, digits and hyphens, starting with a letter. `results`, `mocks`, `files`, `node_modules`, `true`, `false` and `null` are taken.

`prompt.md`: the settings block, then the prompt exactly as the user would type it. An unknown key in the block makes the case fail to load, so free text goes in `case.yaml`.

```markdown
---
tags: [skill-creator]
max_turns: 20
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Write, Edit, Bash]
---

User's task prompt
```

These tools cover files and the shell. A case that must call the agent's MCP tools needs three more things: the tools added by full name (`mcp__<server>__<tool>`), a server the run can reach, and the runner's grant. `claude plugin eval --help` lists the switches: `--allow-tools`, `--mocks`, `--allow-real-servers`.

`case.yaml`: the version stays quoted, or the case fails to load; the name equals the folder name.

```yaml
schema_version: "1.1"
name: asks-which-on-ambiguous-name
expected_outcome: |
  One line: what a right answer does
```

`graders/NN-few-words.md`: one check per file, numbered to keep their order. The top block is always the same; `focus: trace` lets the judge see the tool calls as well as the final message.

```markdown
---
type: llm
focus: trace
weight: 1
---

The answer lists every matching issuer and asks which one is meant
```

Which cases to write:

- The source's cases, prompt verbatim. They are the requirements, so the skill is written against them.
- Held-out cases: requests the source never listed (the same intent in other words, another entity, a neighbouring edge), worded as a real user would type them. Default: one for every three source cases, at least three. Get them from the expert, or write them from the requirement rows before drafting the skill body, and add no rule that names one of them. They show whether the skill is general: passing the source cases and failing these means the skill is fitted to its tests.
- One neighbouring request the skill should leave alone, with a check that its rules were not applied.

Checks: two to four per case, each a statement someone can verify by reading the transcript ("tool A is called before tool B", "no figure is given for X", "the answer asks which one"). State each about the result itself, not about where a run saves its files, so it holds in both runners. A check that passes with and without the skill says nothing about the skill: replace it. A quality only a person can judge gets no forced check: the case keeps one placeholder grader with the body `The result matches this description: <the expected outcome>`, because a case with no grader file fails to load.

Runs: every case with and without the skill. `skill-creator` runs both and opens a viewer for the expert; `claude plugin eval .` from the skill directory runs the same folders from a terminal (Claude Code 2.1.269 or later) and writes a report under `evals/results/`. These are billed model runs, so the person who pays decides when. Put the outputs in front of the expert before grading them yourself, because a real answer shows them what no check anticipated. Write the delta in the process map.

Older format: an existing `evals/evals.json` still runs in `skill-creator` and does not run under `claude plugin eval`. Leave the file as it is and write new cases as folders. A `skill-creator` copy that still writes `evals/evals.json` is an older one: update it, or run the folders with `claude plugin eval`.

## Before publishing

- No secrets, tokens, account ids or internal hostnames.
- No client or person names; keep the register sample generic or from public material.
- Nothing referenced outside the skill folder except an interface every target surface can reach.
- `skill-creator`'s validator passes: `python -m scripts.quick_validate <skill folder>`, run from the `skill-creator` directory.
- `claude plugin validate` passes where the skill is installed as a plugin.
