# Writing rules for a skill the agent will read

Bundled so the skill travels without any house skill installed. Read before step 5 (Encode).

## Description

Form: trigger plus outcome, third person, no procedure. "Use when …" names the words a user says; the outcome names what exists after the run.
Per-word audit: for every word, state the misfire it prevents (the agent fires on the wrong request, or expects the wrong result). A word with no misfire is cut. Count words before and after.

## Body

- Anchor or spell out. Ask once per skill: is there a named method, standard or tool the model already knows, and is it still the one the expert uses? Yes: anchor, never describe. Name the method, the tool, the field, the standard, and whose it is; writing out how the anchor works fights what the model knows and overweights whichever sub-step got written. No (frontier work, or the known method is stale): the expert's walk from the process map is the anchor, so write the steps in full and in order, because there is nothing to point at.
- Only the diff. Every line names the misfire it prevents in the cold run. A line the agent would obey anyway is a no-op: delete it whole, do not trim it.
- Positive targets. Say what to do; keep a prohibition only where no positive form exists, and pair it with the words the agent should say instead.
- Telegraphic allowed, lossless content. Drop articles, copulas and filler; keep every path, number, name, list item and negation verbatim. A version missing a fact the longer one stated is a summary, not compression.
- Shape: label, colon, sentence. BLUF first. One term per thing; no synonyms for the same field or state.
- One real example of the output, filled, from the source material; never a blank template.
- Paths, tool names, field names rooted and in backticks.
- Under 60 lines. `SKILL.md` keeps only what the agent needs on every run. The full field map is reviewer evidence: it goes beside the process map, outside the skill folder. `references/` holds only what the agent itself reads.

## Process map

Tag every cell: `V` verified in code or data (cite the path), `T` from the ticket or the owner's words, `?` open with a named question. A guess gets no tag; it does not go in.

## Evals

`evals/evals.json`, compatible with Anthropic's `skill-creator`:

```json
{
  "skill_name": "<name>",
  "evals": [
    {
      "id": 1,
      "prompt": "verbatim from the source",
      "expected_output": "one line: what a right answer does",
      "expectations": ["a verifiable statement about the output or the tool calls"]
    }
  ]
}
```

Two to four expectations per case, each checkable by reading the transcript ("tool A called before tool B", "no figure given for X", "the answer asks which"). One near-miss prompt that must not trigger. Run every case with and without the skill; write the delta in the process map.

## Before publishing

- No secrets, tokens, account ids or internal hostnames.
- No client or person names; keep the register sample generic or from public material.
- Nothing referenced outside the skill folder except an interface every target surface can reach.
- `claude plugin validate` passes where the skill is installed as a plugin.
