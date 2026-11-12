# Skill-Creator Versus `writing.md`: Four Conflicts

**BLUF:** The writing rules in `skills/skill-from-domain/references/writing.md` agreed with Anthropic's `skill-creator` on most points and conflicted on four: reasons, descriptions, generality and eval file format. All four are decided: `skill-creator` is the official guidance, so it wins, and `writing.md` is aligned. No eval case has run under the new rules yet.

- **Date:** 2026-11-11.
- **Status:** Decided and applied in the commit that adds this note.
- **Standing Rule:** where a rule in this repo and `skill-creator` disagree, `skill-creator` wins and the rule here gets corrected. `writing.md` and `README.md` both state it.

## What Was Compared

- **Ours:** `skills/skill-from-domain/references/writing.md` at commit `74bce08`, with steps 1 and 6 of `skills/skill-from-domain/SKILL.md`.
- **Theirs:** Anthropic's `skill-creator` (Apache 2.0): `SKILL.md`, `references/schemas.md`, `agents/grader.md`, `agents/analyzer.md` and `scripts/quick_validate.py`.
- **Two Copies Read:** the plugin from the `claude-plugins-official` marketplace, and a newer copy synced from the Claude account. The two differ on test-case storage and file delivery, not on writing guidance (Verified by diff).

Every quote below is Verified against those files on the date above.

## The Four Conflicts

### 1. Reasons

- **`skill-creator`:** "Try hard to explain the **why** behind everything you're asking the model to do." Also: "If you find yourself writing ALWAYS or NEVER in all caps, or using super rigid structures, that's a yellow flag". Body limit: under 500 lines.
- **`writing.md` Before:** No rule asked for the reason. "Telegraphic allowed", "Shape: label, colon, sentence" and "Under 60 lines" all pressed the text shorter. "Every line names the misfire it prevents" said what goes wrong, not why it matters.
- **Effect:** The two skills built so far (kept local, see `README.md`) state rules and facts with almost no reasons; neither file contains `because`, `so that` or `since` (Verified). `skill-creator` argues that a model given the reason can "go beyond rote instructions" on a case the rule does not name (Reported, not tested here).
- **Decision:** Align.
- **Before → After:** no reason rule → "Give the reason with the rule" opens the body rules. "Telegraphic allowed" → plain sentences in imperative form, terse fragments in lookup material only. Fixed label-colon shape → headers and labels where they help. "Under 60 lines" → "Lean, not short": no line target, 500-line ceiling.

### 2. Descriptions

- **`skill-creator`:** Claude has a tendency to "undertrigger" skills, so a description should be a little bit "pushy" and name more contexts for use.
- **`writing.md` Before:** The per-word audit counted two misfires: "the agent fires on the wrong request, or expects the wrong result". A word with no misfire was cut.
- **Effect:** The audit did not count a third misfire, the skill failing to fire on a request it should take, so it cut the words that prevent it (Inferred from the two texts; no trigger eval has run).
- **Decision:** Align.
- **Before → After:** two misfires → three, with staying silent first. "Count words before and after" → "Measure, do not count": about twenty queries, half should-trigger and half near-miss, run by the description optimizer. New limits from the validator: at most 1024 characters, no angle brackets.

### 3. Generality

- **`skill-creator`:** "make the skill general and not super-narrow to specific examples". A skill that "works only for those examples" is "useless". Its description optimizer picks the winner on a held-out 40% of queries "to avoid overfitting".
- **`writing.md` Before:** Eval prompts were "verbatim from the source", and every skill line was written against the misfires in those same cases. No prompt was held out.
- **Effect:** The skill is tuned on its own test set, so a pass rate on the source cases overstates how it handles a new question (Inferred; no run exists yet).
- **Decision:** Align.
- **Before → After:** no generality rule → "General, not fitted". Source cases only → source cases plus held-out cases, one for every three and at least three, set aside in step 1 of `skills/skill-from-domain/SKILL.md` before any drafting.

### 4. Eval File Format

- **`skill-creator`, Newer Copy:** Each test is a folder, `evals/<case-name>/`, holding `prompt.md`, `case.yaml` and `graders/`. `evals/evals.json` is the "older format: read it, don't write it". `claude plugin eval` "does not read `evals.json`".
- **`writing.md` Before:** "`evals/evals.json`, compatible with Anthropic's `skill-creator`", followed by the JSON schema.
- **Effect:** An existing `evals.json` still runs inside both copies of `skill-creator`. It does not run under `claude plugin eval`, and the newer copy writes every new test as a folder (Verified in `references/schemas.md`).
- **Decision:** Align.
- **Before → After:** one `evals/evals.json` → one folder per case, with the three blocks copied from `references/schemas.md`. An existing `evals.json` is left as it is.

## Where the Two Agree

- **Description as Trigger:** all when-to-use text sits in the description, never in the body.
- **Lean Body:** "Remove things that aren't pulling their weight" there; "Only the diff" here.
- **Trust in the Model:** "Today's LLMs are *smart*" there; anchor, never describe, here.
- **Few Prohibitions:** capital-letter rules are a yellow flag there; positive targets here.
- **Examples:** both ask for an example of the output.
- **Progressive Disclosure:** `references/` holds what the agent reads on demand.
- **Baseline Runs:** both run every case with and without the skill and read the difference.
- **Verifiable Checks:** "objectively verifiable" assertions there; checks verifiable from the transcript here.
- **Near-Miss Negatives:** `skill-creator` calls near-misses the most valuable should-not-trigger cases; `writing.md` requires them.

## Smaller Differences

- **Output Template:** `skill-creator` allows a blank output template; `writing.md` banned it. **Decision:** Align. A fixed template is allowed when the format must be exact.
- **Human Review:** `skill-creator` puts the outputs in front of a person before the author grades them; `writing.md` had no review step. **Decision:** Align. The expert sees the outputs first.
- **When Checks Are Written:** `skill-creator` drafts assertions after the first runs start; `writing.md` writes checks with the case. **Decision:** Keep. The source's acceptance tests already state the expected result.

## What Changed

- **`skills/skill-from-domain/references/writing.md`:** sections Description, Body and Evals rewritten; one line added to Before Publishing.
- **`skills/skill-from-domain/SKILL.md`:** the intro and steps 1, 5, 6 and 7 follow the new rules. The description lost its angle brackets.
- **`README.md`:** Layout, Rules and Log.
- **`docs/learn-docs/README.md`:** one line saying `writing.md` is current where the learning notes differ.

## Verification

- **Validator:** `scripts/quick_validate.py` failed on `skills/skill-from-domain/` before the change with `Description cannot contain angle brackets (< or >)`. It now prints `Skill is valid!` from both copies (Verified).
- **Settings Block:** `claude plugin eval` on Claude Code 2.1.322 loads a case built from the three blocks in `writing.md` without error, and rejects a control case with `unknown frontmatter key "bogus_key"` (Verified, no model run).
- **`case.yaml` and Grader Blocks:** identical to the text in `references/schemas.md` (Verified by comparison). The load check does not read them: control cases with a bad `case.yaml` key and a bad grader key passed it. Their behaviour in a run is Reported by `skill-creator`, not tested here.
- **No Model Run:** no eval case has run under the new format, so the effect of the new rules on a real skill is Unknown.

## Not Done

- **Built Skills:** the two skills built before this change still follow the earlier rules: rules without reasons, `evals/evals.json`, no held-out cases.
- **Trigger Sets:** no skill has one yet.
- **Core Skill Description:** fixed for the validator only. It has not been tested for triggering.

## Found Along the Way

- **MCP Mocks:** `claude plugin eval --help` lists `--mocks`: "Mock stand-ins for MCP servers, from <eval dir>/mocks/". Not tried; the mock file format is Unknown. It may stand in for a tool server during cold runs.
- **Blank Case:** `claude plugin eval init --bare <name>` writes a blank case, `prompt.md` plus `graders/criteria.md`, with no model run (Verified).
- **Frontmatter Keys:** the validator accepts six keys only: `name`, `description`, `license`, `allowed-tools`, `metadata`, `compatibility`. A skill that uses Claude Code keys such as `argument-hint` or `disable-model-invocation` fails it and cannot be packaged for upload (Verified in `scripts/quick_validate.py`).
