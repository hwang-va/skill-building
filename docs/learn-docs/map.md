# 00. The map: what a skill is made of, and who it is for

**BLUF:** A skill is a Markdown file with a trigger (the `description`, always in context) and a body (loaded on invocation) plus optional files the agent reads or runs on demand. It has two readers. The human reads it as a desk procedure and maintains it. The agent reads it as code: every sentence changes behaviour or wastes attention. Five schools teach the craft; they agree that the description is a trigger not a summary, that the body carries only deltas from default behaviour, and that a skill is tested by running it, never by reading it. They disagree on tone (firm prohibition versus explained why) and on naming. The checklist in §5 is the sprint's working tool. Written 2026-11-04 from the sources in §3; mechanics Verified against the Claude Code docs the same day.

## 1. The Two Readers

| Reader | Reads the skill as | Cares about | Failure it sees |
|---|---|---|---|
| Human (author, teammate, reviewer) | A desk procedure: when it applies, what it produces, how to change it | Can I find it, trust it, edit it, see what it did | Skill nobody remembers exists; skill nobody dares edit |
| Agent (at run time) | A prompt appended to its context, sticky across turns | Does the trigger fire at the right moment; does each line change what I do | Undertrigger; sprawl thins attention; a negation that plants the banned idea |

**The Craft:** Write for the agent, package for the human. Every word of the description costs on every turn; every line of the body costs on every turn after invocation.

## 1b. the author's Three Elements (2026-11-04)

The craft is still human-owned. Three elements, and where the schools land on each:

| Element | What it means | School support | Test |
|---|---|---|---|
| Transform domain knowledge into a skill | Decide what the agent lacks: the firm's vocabulary, rules, data shapes, exceptions. Not the method it already knows | Under-covered. Pocock's no-op hunt applied to knowledge, not only instructions | Ask the agent cold. If it already knows, the line is a no-op |
| Anchor, never describe | Name the method (a spec name, a known process); the agent holds the definition. Why the spec-skill agent sections are short | All five: leading word (Pocock), "Claude is already smart" (Anthropic), anchor (in-house compression skill) | Each description word must name a misfire it prevents |
| Feed domain specifics | Static specifics inside the skill folder; live or shared specifics behind an interface | Anthropic: bundle comprehensive resources, use `Server:tool` names; Pocock: the environment is the source of truth, a doc that restates it is a cache | Does it travel with the skill? See the tiers |

**Where Specifics Live (the travel test):**

| Tier | Holds | Travels to claude.ai, Cowork, another machine? |
|---|---|---|
| Body | Anchor names, firm vocabulary, the rules | Yes |
| Skill folder: `references/`, `scripts/`, `assets/` | Static, small, owned by the skill: rubric, schema, template, validator | Yes. Bundled files upload with the skill. Verified 2026-11-04: the synced `skill-creator` carries `agents/`, `references/`, `scripts/`, `assets/` |
| Interface | Live, large, or shared: MCP tool as `Server:tool`, URL, ARN, database | Only where every surface authenticates it. MCP connectors (Notion, Atlassian) do; an AWS ARN needs AWS credentials on that surface |

**Rules:** a path outside the skill folder (`C:\Users\<me>\...`) breaks when the skill leaves this laptop. `` !`cmd` `` injection never runs for synced skills (Verified, Claude Code docs). So: bundle the static, point at an MCP tool for the live, never reference the local filesystem. Checklist line 13 follows from this.

## 2. Anatomy (Verified, Claude Code docs, 2026-11-04)

```
skill-name/
  SKILL.md            required: frontmatter + body
  reference.md        loaded only when SKILL.md points at it; one level deep
  examples.md         same
  scripts/helper.py   run, not loaded; only its output enters context
```

**Three Loading Tiers:** metadata (`name` + `description`, always in context, about 100 words), then body (on invocation, under 500 lines), then bundled files (on demand, unbounded).

**Frontmatter That Matters:**

| Field | Does | Use it for |
|---|---|---|
| `description` | The trigger. Claude matches the request against it. Capped at 1,536 chars with `when_to_use` | Key use case first, third person, what plus when |
| `disable-model-invocation: true` | Only `/name` fires it; description leaves the agent's context | Side-effecting workflows (`/deploy`, `/commit`), hand-only tools |
| `user-invocable: false` | Hidden from the `/` menu; only Claude fires it | Background knowledge, shared reference |
| `context: fork` + `agent:` | Body becomes the prompt of a fresh subagent; no conversation history | A skill with an explicit task, not guidelines |
| `allowed-tools` | Tools pre-approved for the invoking turn | `Bash(gh *)` style, side effects the skill owns |
| `paths:` | Auto-activation limited to matching files | Repo-area skills |
| `hooks:` | Hooks registered on invocation, kept for the session | Guardrails the skill brings with it |
| `$ARGUMENTS`, `` !`cmd` `` | Argument substitution; shell output injected before Claude reads | Dynamic context (PR diff, today's log) |

**Where It Lives:** enterprise, then personal (`~/.claude/skills/`), then project (`.claude/skills/`), then nested; plugins namespaced as `/plugin:skill`, never collide. The body stays in context across turns and is not re-read; after compaction the last invocation is re-attached (5,000 tokens per skill, 25,000 combined).

## 3. The Five Schools

| School | Source | Thesis in one line | On the description | On the body | On testing |
|---|---|---|---|---|---|
| Anthropic docs | `platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices`; `code.claude.com/docs/en/skills` | Concise; Claude is already smart; match freedom to fragility | What plus when, third person, key terms; gerund names | Under 500 lines; one level deep; workflows as checklists; validator loops; explain why | Build evals before docs; Claude A writes, Claude B runs; test on Haiku, Sonnet, Opus |
| `skill-creator` (Anthropic plugin) | `~/.claude/skills/synced/<id>/skill-creator/SKILL.md` | Draft, run with and without, grade, rewrite, repeat | Make it a little pushy: Claude undertriggers | Explain why, not musty MUSTs; caps ALWAYS/NEVER is a yellow flag; bundle a script when every run writes the same one | `evals/<case>/` folders, baseline vs with-skill, `benchmark.md`, viewer, description optimizer with 20 near-miss queries |
| Superpowers `writing-skills` | `repos/superpowers/skills/writing-skills/SKILL.md` | TDD on process docs: watch the agent fail first | A trigger, never a summary; "Use when" plus symptoms | Discipline skills get firm prohibition, excuse table, red flags; shape errors get a positive recipe; under 200 words if loaded often | No skill without a failing baseline; pressure scenarios with subagents; collect rationalizations verbatim |
| Pocock `writing-for-agents` | `repos/mattpocock-skills/skills/productivity/writing-for-agents/` | Two loads (context, cognitive); information hierarchy; leading words | A context pointer: front-load the leading word, one trigger per branch; user-invoked skills carry no trigger at all | Steps end on checkable completion criteria; disclose what only some branches reach; prompt the positive; hunt no-ops; cache only what the environment cannot tell | Settle no-op disputes by running the document, not debating |
| In-house (two compression skills) | an in-house skills repo | Anchor to a known concept, compress to the delta, audit every word | `Trigger: <minimum>. Outcome: <one deliverable>.`; per-word audit naming the misfire each word prevents | Telegraphic; label, colon, sentence; length tracks state governed; quoted sample utterances teach tone | a contribute gate: branch, then the eval run, then PR with an eval section |

## 4. Where They Agree and Where They Split

**Agreed (five of five):**

- **Description is the trigger.** It says when and what exists after, never how. Procedure in the description makes the agent skip the body (Superpowers, Verified by their baseline).
- **Delta only.** Write what differs from default behaviour. A sentence the model already obeys is a no-op and a cost.
- **Run it to judge it.** A skill is graded by an agent's run with and without it, never by its text.
- **One level deep, under 500 lines, scripts for the deterministic parts.**

**Split (decide per skill):**

| Question | Side A | Side B | My default for the sprint |
|---|---|---|---|
| Tone of rules | Superpowers: discipline skills need firm prohibition plus an excuse table, because agents rationalize under pressure | `skill-creator`, Pocock: explain why; prohibition plants the banned idea; caps are a yellow flag | Positive target first; prohibition only as a hard guardrail, paired with the positive, and only after a baseline showed the rationalization |
| Naming | Anthropic: gerund (`processing-pdfs`) | Pocock and an in-house skill: anchor plus action (`json-core`); Superpowers: action verbs | Plugin convention wins (a plugin marketplace: `/plugin:verb-noun`) |
| Pushy or minimal description | `skill-creator`: a little pushy, undertrigger is the common failure | in-house compression skill: ruthless word audit, more is context rot | Minimal words, maximal discrimination: a near-miss eval set decides |
| Who may invoke | Anthropic docs: model-invoked by default | Pocock: user-invoked unless the agent must reach it alone; router skill when hand-only skills pile up | Ask "must the agent or another skill reach this on its own?" If no, `disable-model-invocation: true` |

## 5. Checklist v0 (the sprint's working tool)

Score each line 0 (absent), 1 (present), 2 (present and tested by a run).

| # | Line | Reader served |
|---|---|---|
| 1 | Description states when (trigger) and what exists after (outcome); no procedure | Agent |
| 2 | Invocation chosen on purpose: model-invoked, user-invoked, or forked with an explicit task | Both |
| 3 | Every body sentence is a delta from default; no-ops deleted | Agent |
| 4 | Steps end on a checkable completion criterion | Agent |
| 5 | Rules stated as the positive target; prohibitions only as paired guardrails | Agent |
| 6 | One leading word carries each region of behaviour, repeated, not restated | Agent |
| 7 | Reference pushed one level down; SKILL.md under 500 lines; scripts run, not read | Both |
| 8 | One excellent real example, not a template | Both |
| 9 | A baseline run exists (without the skill) and the skill answers its specific failures | Both |
| 10 | Three eval cases with graders; a run with and without, and the delta written down | Both |
| 11 | A human can find it (router, README row, plugin listing) and edit it in one place | Human |
| 12 | Why is explained where the agent must judge; mechanics are exact where it must not | Both |
| 13 | No reference outside the skill folder except an interface every target surface can reach (MCP tool, URL) | Both |

## 6. The Leap: Skills for an Agent Team (Inferred, to test in week 3)

**Claim:** On an agent team the skill is the job description and the agent definition is the hire. Claude Code gives four pieces that compose:

| Piece | Role on the team | Skill-side lever |
|---|---|---|
| Skill (`SKILL.md`) | The procedure one role follows | Body, trigger, bundled files |
| Agent (`.claude/agents/*.md`, `skills:` field) | The role: its system prompt, its tools, the skills it is born with | Preload the role's skills; `context: fork` + `agent:` hands a skill a fresh worker |
| Hook (`hooks:` in frontmatter or settings) | The guardrail the role cannot argue with | What a prohibition should become when the baseline shows rationalization; see §6b |
| Eval (`evals/<case>/`, `claude plugin eval`) | The performance review | Gate before the role ships; the evidence someone else can see |

**Open questions for week 3:** how a router skill maps a team; how handoff between two forked skills carries state; whether a team-skill template is one file or a skill plus an agent file.

## 6b. Hooks: the Deterministic Layer a Skill Can Carry (Verified, hooks doc, 2026-11-04)

**Where It Sits:** a hook is harness code, run before or after a tool call, outside the model. A skill line is advice the agent can rationalize away; a hook with `e.cancel()` on a cancelable event (`PreToolUse`, `Stop`, `UserPromptSubmit` and 10 more) blocks the action. Judgment goes in the skill body; non-negotiables go in a hook. The skill keeps the rule as text so the agent knows why; the hook makes it hold.

**Three Ways Skills and Hooks Meet:**

| Way | Mechanism | Lifetime | Human decision point |
|---|---|---|---|
| Settings hook | `settings.json`, `.claude/settings.local.json`, plugin `hooks.json` | Every session, every matching call | Author time |
| Skill that authors a hook | A skill whose output is a hook script plus its settings entry. An in-house example bundles a hook kit, classifies by event and type, names the 13 cancelable events); also `update-config`, `plugin-authoring` | Whatever the written hook says | While the skill runs |
| Skill that carries a hook | `hooks:` block in skill frontmatter, same format as settings hooks | Registered on invocation, kept for the rest of the session; `once: true` removes it after the first successful run (skill frontmatter only). Agent frontmatter hooks live only while that subagent runs; `Stop` becomes `SubagentStop` | The invocation itself |

**Pattern: the Armed Guardrail.** A skill that is nothing but a hook block, `disable-model-invocation: true`. Typing its name arms the guardrail for this session. No settings edit, no file watcher, one slash command.

```yaml
---
name: guard-aws
description: Arm the AWS guardrail for this session
disable-model-invocation: true
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "node ~/.claude/hooks/aws-guard.cjs"
---
Guardrail armed: any `aws` command naming prod or billing is blocked until the session ends.
```

**Dynamic Alternative (Reported, not run here):** a skill asks the human, then writes the hook entry into `.claude/settings.local.json`. The doc says direct edits "are normally picked up automatically by the file watcher"; a `ConfigChange` event fires and can block the change. The `/hooks` menu is read-only.

**For the Team (§6):** each role's non-negotiables become its agent-frontmatter hooks, scoped to the role's run. First candidate: a utility skill's never-touch list (`prod-*`, `billing*`) as a `PreToolUse` hook via a hook-authoring skill.

## 7. PM Transfer Line

Replace "agent" with "new analyst" and "skill" with "desk procedure": the trigger is the situation on the procedure's cover sheet, the body is the steps with a done-check on each, the baseline is watching the analyst work without it, and the eval is the first three cases they run while you watch.
