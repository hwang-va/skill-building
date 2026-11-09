# skill-building: the skill factory

**BLUF:** One craft, many skills. `skills/skill-from-domain/` is the method (Elicit, Diff, Encode) as a runnable skill for Claude Code: it takes a ticket or an expert's workflow and produces a skill folder with its process map, field map and evals. Domain bindings and the skills the craft produced live beside it, local to each machine. Started 2026-11-09.

## Why a Skill

A skill does not make the agent smarter. The model already holds more methods than any one expert, and for most tasks several of them work. Left alone, the agent picks one, and picks differently next time or on the next model. A skill fixes the choice: the agent does the task the user's way, with the user's checks, data sources and output shape, every run.

So the two things a skill buys are consistency and alignment with the person who owns the work, not the best method. The requirements come from the domain expert; the skill keeps only the steps the agent gets wrong without them; the harness knowledge (triggering, loading, hooks, evals) turns those steps into a form the agent follows. The builder needs all three: the expert for the process, a cold run for the gap, the harness for the encoding.

Two caveats keep this honest. State the chosen method as the house way, so a later expert can see it and upgrade it on purpose rather than by accident. Run the evals again whenever the underlying model changes: a line that landed on one model may be ignored by the next, and the eval set is the only thing that tells you.

## Layout

| Path | What | Published |
|---|---|---|
| `skills/skill-from-domain/` | The core: seven steps from a written process or an expert to a skill folder. `references/writing.md` holds the writing, compression and eval rules, so it needs no other skill installed | yes |
| `skills/<domain>-skill-builder/` | A binding: the core plus one domain's "where the truth is" table, tool list and landing spot. One file, under 40 lines | no, local |
| `built/<name>/` | Every skill the craft produced: `01-process-map.md`, `skill/<name>/SKILL.md`, `references/fields.md`, `evals/evals.json` | no, local |
| `docs/learn-docs/` | The learning behind the method: the three moves, a map of skill anatomy and the five schools, a vocabulary, a crash course on the Anthropic sources. For the human builder | yes |

Bindings and `built/` are ignored by git because they carry employer-specific paths and data. They are the proof; the method is the product.

## Use

- You hold the written process, or you are the expert: `/skill-from-domain <ticket, document or workflow>`.
- A domain with a binding: `/<domain>-skill-builder <ticket>`.
- A new domain: copy an existing binding, replace the "where the truth is" table, the tool list and the landing spot. Nothing else changes.

## Install

Run the skills from this repo through junctions (Windows) or symlinks, so an edit here is live at once:

```powershell
New-Item -ItemType Junction -Path "$HOME\.claude\skills\skill-from-domain" -Target "$HOME\projects\skill-building\skills\skill-from-domain"
```

## Rules

- Built beats written. A skill counts when it ran on a real source, with and without, and the delta is in its process map.
- Core first, binding second. A step that helps every domain goes into `skill-from-domain`; a path or a tool name goes into a binding.
- Only the diff. Every skill line names the misfire it prevents; the model's own method stays out.
- Validate before commit: `claude plugin validate` once a skill sits in its plugin; Anthropic's `skill-creator` for evals when installed.
- No secrets in any file. No client, employer or person names in anything tracked here.

## Log

| Date | What |
|---|---|
| 2026-11-09 | Repo created. Core skill written from two real runs (one by hand, one through a binding). Both built skills wait on a reachable tool server for their cold runs. |
