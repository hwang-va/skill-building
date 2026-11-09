# Elicit, Diff, Encode: how domain knowledge becomes a skill

**BLUF:** A skill builder does not need to be the domain expert. The job is to pull the expert's real process out (Elicitation), run the agent cold on the same task and diff the two (Delta analysis), and write only the difference in a form the agent follows every time (Encoding). The three moves are the method; the examples below fill in as I build skills with them. Found 2026-11-04 while asking what an equity-research skill should contain once the model already knows how to research a company. Principles now, worked examples later, so this can be told to someone else with one real case per move.

**One-line version for others:** ask the expert what they actually do, watch the agent do it without help, write down only the difference, then prove it with three runs.

---

## The Three Moves

| Move | Question it answers | Output | Tools I have |
|---|---|---|---|
| 1. Elicitation | What does the expert actually do, including the parts they never say? | A process map with the tacit checks and the scars | an interview skill, discovery-interview craft, a written interview script |
| 2. Delta analysis | What does the agent do differently when it runs cold? | A diff: lines the agent got wrong, skipped, or did in a different order | A baseline run (no skill), `skill-creator` without-skill mode, the Superpowers rationalization log |
| 3. Encoding | How do I write the diff so the agent follows it every run? | `SKILL.md` plus bundled files, three evals, hooks for the non-negotiables | `map.md` §5 checklist, the in-house compression and hook-authoring skills, `skill-creator` evals |

The order matters. Encoding before elicitation writes the method the model already knows. Encoding before the diff writes no-ops. Elicitation without the diff produces a procedure for humans, not a skill.

## Move 1: Elicitation

**Principle:** The expert's written process is not their real process. The real one lives in what they check first, what they ignore, what they stopped doing after it burned them, and what they would never let a junior do alone.

**Actions:**

1. Pick one workflow with one named owner and one real recent case. Not "equity research"; "how the author decided on last Thursday's name".
2. Walk the real case end to end. At each step ask: what did you look at, what did you skip, what would have stopped you here.
3. Pull the tacit layer with four question families:
   - **First look:** "What do you check before anything else, and what would make you drop the name in two minutes?"
   - **Ignored:** "What does the textbook say to do that you never do, and why?"
   - **Scars:** "When did this go wrong, and what do you now do differently because of it?"
   - **Hand-off:** "What would you never let a new analyst decide alone?"
4. Write the process map: steps, the check at each step, the data each check needs and where it comes from, the output the next step consumes.
5. Read it back in one minute. The expert corrects; the corrections are usually the most valuable lines.

**Done when:** the map names every data source by tool or system, every check has a pass condition, and the expert says "yes, that is what I do" without adding a step.

**Where I am the expert:** my investing loop, my spec process, my AWS lab. Play both roles: write the map first, then interview myself with the four families.

## Move 2: Delta Analysis

**Principle:** The skill is the difference between the expert and the cold agent. Frontier models already hold the method (DCF, 10-K sections, Porter five forces, UML). Writing it again costs tokens and buys nothing. Only the diff earns its place.

**Actions:**

1. Give the agent the same real case with no skill. Same inputs, same ask, fresh session.
2. Record the transcript, not only the output. Where it went, what it read, what it invented, what it skipped.
3. Diff against the process map, line by line. Four kinds of difference:
   - **Wrong:** the agent did something the expert would not (made up a number, used the wrong source).
   - **Missing:** the expert's check is absent (no cash-conversion check, no citation).
   - **Order:** right steps, wrong sequence (recommendation before the gate).
   - **Shape:** right content, wrong output form (prose where the journal needs a row).
4. Collect the agent's rationalizations verbatim when it skips a check. They become the excuse table or the hook.
5. Everything the agent did the same as the expert is crossed out. It will not be in the skill.

**Done when:** every line that survives names one concrete misfire it prevents. A line that cannot name its misfire is a no-op.

**Repeat:** after the skill exists, the diff runs again, with the skill. The new diff is the next version.

## Move 3: Encoding

**Principle:** Write for the agent, package for the human. The agent reads the skill as code; every sentence changes behaviour or wastes attention. The human must find it, trust it, and edit it in one place.

**Actions:**

1. **Anchor the known.** Name the method, never describe it: "run a Porter five forces", "EARS requirement form". One leading word per region of behaviour, repeated.
2. **State the focus points** as positive targets, one line each: "rate cash conversion before growth".
3. **Name the data interfaces** by tool: `Server:tool` for MCP, the script in `scripts/` for deterministic work, the schema in `references/`. Numbers come from tools, never from the model.
4. **Give the output shape** as one real filled example in `assets/`, not a blank template.
5. **Write the gates** as checkable completion criteria: "every number cites a tool call", "no recommendation before the gate passes".
6. **Add the vocabulary** the model cannot guess: universe names, scoring levels, what a defined term means here.
7. **Keep the scar list** from Move 1 and the rationalizations from Move 2 as the excuse table.
8. **Move non-negotiables to hooks.** A rule the agent must never argue with becomes a `PreToolUse` hook, carried in the skill's `hooks:` frontmatter or the agent's.
9. **Write the description last:** trigger plus outcome, third person, no procedure. Decide invocation on purpose (model-invoked only if the agent must reach it alone).
10. **Prove it:** three eval cases, with and without, the delta written down. Then the `map.md` §5 checklist, line by line.

**Done when:** the checklist scores no zeros, the three evals pass with the skill and at least one fails without it, and nothing outside the skill folder is referenced except an interface every target surface can reach.

## Worked Examples (fill as they happen)

One real case per move, dated. A reader should be able to follow the case without me in the room.

| Date | Workflow | Move 1 output | Move 2 diff (top three lines) | Move 3 result | Where |
|---|---|---|---|---|---|
| 2026-11-06 | a skill-set project: sort, roles, collect skills | Six concrete starts (a scheduler dashboard, the paper investing loop, memo sign-off, the fills nobody mentioned) lifted to one-line rules | Agent defaulted to `alone` where authority was unsourced; closed rows without a raw file; missed the asks with no past instance | Contract rules 2, 12, 13; `sort-level1` and `roles-level1` rebuilt, `collect-level1` new; one run on a real source each | a companion handbook on building skill sets (not in this repo) §5 |

**Companion:** a companion handbook on building skill sets (not in this repo) is the same method one level up, for a whole set: its loop (purpose, mental model, contract, skill, run, judge, publish) is Elicit, Diff, Encode with the contract as the shared process map and "built versus written" as the diff's evidence rule.

## Telling It to Others

Three sentences, then one example per move from the table above:

> The agent already knows the method. What it does not know is how we do it here: what we check first, where our numbers come from, what we never do. So we interview the person who does it, watch the agent try it cold, and write down only the gap. Then we prove the gap closed with three runs.
