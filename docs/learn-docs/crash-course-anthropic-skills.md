# Crash Course: Agent Skills, from the Anthropic sources

**BLUF:** Everything Anthropic says about skills fits in six ideas: a skill is a folder with one required file; it loads in three levels so installing many costs almost nothing; the description is the trigger and the body is sticky; code in the folder runs without entering context; evals come before documentation; and a skill does not travel between surfaces by itself. Each section below is a fact list with a self-check box. Tick what you already knew, and the unticked rows are the review. Written 2026-11-04 from the five primary sources in §9; mechanics Verified against those pages the same day. Facts specific to Claude Code are marked CC; to the open standard, Spec; to the API and claude.ai, API.

**Companions:** `map.md` (the five schools and the checklist) (skills as one tool surface next to MCP).

---

## 1. The One Idea

**A skill is an onboarding guide for a new hire, stored as a folder the agent reads with `bash`.** Anthropic's own phrase. The agent has filesystem access, so the guide can hold instructions, scripts and reference material, and the agent opens only what the task needs. Prompts repeat per conversation; skills load on demand, once, when relevant. Their design goal is composable specialists instead of one custom agent per use case (engineering blog, Reported).

## 2. Structure and the Open Standard (Spec)

| # | Fact | Knew it |
|---|---|---|
| 2.1 | A skill is a directory; `SKILL.md` is the only required file. Conventional subfolders: `scripts/`, `references/`, `assets/` | [ ] |
| 2.2 | `name`: 1 to 64 chars, lowercase letters, digits, hyphens; no leading, trailing or double hyphen; must match the folder name; no `anthropic` or `claude` | [ ] |
| 2.3 | `description`: 1 to 1,024 chars, non-empty, no XML tags; what it does and when to use it, with keywords | [ ] |
| 2.4 | Optional spec fields: `license`, `compatibility` (max 500 chars, only when the environment matters), `metadata` (string map), `allowed-tools` (experimental) | [ ] |
| 2.5 | Body has no format rule. Recommended: steps, input and output examples, edge cases. Keep under 500 lines, under 5,000 tokens | [ ] |
| 2.6 | File references are relative to the skill root and one level deep; nested chains make the agent preview with `head -100` and miss content | [ ] |
| 2.7 | `skills-ref validate ./my-skill` checks the frontmatter against the standard (agentskills.io); `claude plugin validate .claude/skills` does it inside Claude Code (v2.1.233+) | [ ] |
| 2.8 | The standard is open (agentskills.io); the same folder is meant to work in other agents, not only Claude | [ ] |

## 3. Progressive Disclosure: the Three Levels

| Level | What | When loaded | Cost |
|---|---|---|---|
| 1 Metadata | `name` + `description` | Startup, into the system prompt, for every installed skill | about 100 tokens per skill |
| 2 Instructions | `SKILL.md` body | When the skill triggers; the agent runs `cat SKILL.md` | under 5,000 tokens |
| 3 Resources | Bundled files and scripts | When referenced; scripts run and only their output enters context | zero until touched |

| # | Fact | Knew it |
|---|---|---|
| 3.1 | Installing many skills is nearly free; only level 1 is paid every turn | [ ] |
| 3.2 | A bundled script's code never enters context, only its output. A reference file enters in full when read | [ ] |
| 3.3 | "No practical limit on bundled content": full API docs, large datasets, many examples cost nothing until opened | [ ] |
| 3.4 | Organize by domain so one task reads one file (`reference/finance.md`, not `docs/file1.md`); give files over 100 lines a table of contents | [ ] |
| 3.5 | Say which it is: "Run `analyze_form.py`" (execute) versus "See `analyze_form.py` for the algorithm" (read) | [ ] |

## 4. Triggering and Lifetime (CC)

| # | Fact | Knew it |
|---|---|---|
| 4.1 | Trigger is the `description` plus optional `when_to_use`; combined text capped at 1,536 chars in the listing; key use case first. Write in third person: the text is injected into the system prompt, and "I can help you" breaks matching | [ ] |
| 4.2 | The listing of all skill names and descriptions has a budget of 1% of the context window. Over budget, Claude Code drops descriptions starting with the least-invoked skills. `/doctor` shows the cost; `/skill-doctor` finds unused skills; `skillListingBudgetFraction` raises it; `skillOverrides: "name-only"` trims one | [ ] |
| 4.3 | Three invocation modes: default (you and Claude), `disable-model-invocation: true` (only `/name`, description leaves context, also blocks preloading into subagents), `user-invocable: false` (only Claude, hidden from the menu) | [ ] |
| 4.4 | `/name` at the start of a message runs it. The name mid-sentence only gives Claude permission to run it | [ ] |
| 4.5 | Once invoked, the body stays in the conversation across turns and is not re-read. Write standing instructions ("run the tests after every edit"), not one-time steps | [ ] |
| 4.6 | After compaction, the latest invocation of each skill is re-attached, up to 5,000 tokens per skill and 25,000 combined. Put the most important instructions at the top. Re-invoke to restore the rest | [ ] |
| 4.7 | `paths:` limits auto-activation to matching files, and only inside the current working directory; a sibling worktree does not count even after `/add-dir` | [ ] |
| 4.8 | Malformed YAML still loads the body with empty metadata: `/name` works, auto-trigger does not. `--debug` shows the parse error | [ ] |
| 4.9 | Precedence for the same name: enterprise over personal over project. Plugin skills are namespaced `/plugin:skill` and never collide. `.claude/commands/*.md` is the legacy form: a skill without a folder | [ ] |
| 4.10 | Edits to an existing `SKILL.md` apply live in the session; a new top-level skill folder needs `/reload-skills` | [ ] |

## 5. Frontmatter Beyond the Standard (CC)

| # | Field | Does | Knew it |
|---|---|---|---|
| 5.1 | `argument-hint`, `arguments`, `$ARGUMENTS`, `$0`, `$name` | Named or positional arguments substituted into the body; without a placeholder, Claude Code appends `ARGUMENTS: <value>` | [ ] |
| 5.2 | `${CLAUDE_SKILL_DIR}`, `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_SESSION_ID}` | Paths that resolve wherever the skill is installed; use `${CLAUDE_SKILL_DIR}/scripts/x.py`, never a hard-coded path | [ ] |
| 5.3 | `` !`command` `` | Runs before Claude reads the skill; output replaces the placeholder. A failing command aborts the whole invocation (append `\|\| true`). Never runs for claude.ai-synced skills | [ ] |
| 5.4 | `allowed-tools`, `disallowed-tools` | Pre-approved tools for the invoking turn only; deny rules still win. The second removes tools while the skill is active | [ ] |
| 5.5 | `context: fork`, `agent:`, `background:` | Body becomes the prompt of a fresh subagent with no conversation history; default background. Only for skills with an explicit task; guidelines return nothing. Explore and Plan agents start without CLAUDE.md | [ ] |
| 5.6 | `model`, `effort` | Override for the rest of the turn; with `context: fork`, sets the subagent's | [ ] |
| 5.7 | `hooks:` | Same format as settings hooks; registered on invocation, kept for the session; `once: true` removes after the first successful run. In agent frontmatter, hooks live only while the subagent runs | [ ] |
| 5.8 | Spec-only surfaces (claude.ai upload, Skills API) accept only `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`. Any other field is a hard error at upload | [ ] |

## 6. Writing Rules (best-practices page)

| # | Rule | Knew it |
|---|---|---|
| 6.1 | "Claude is already very smart." Add only what it does not have. Challenge each paragraph: does it justify its token cost | [ ] |
| 6.2 | Degrees of freedom match fragility: high (prose heuristics) for open fields, medium (pseudocode with parameters) for a preferred pattern, low ("run exactly this script, do not add flags") for narrow bridges | [ ] |
| 6.3 | Test on every model you will run: Haiku asks "is there enough guidance", Opus asks "does it over-explain" | [ ] |
| 6.4 | Naming: gerund (`processing-pdfs`) recommended; noun phrase or verb acceptable; never `helper`, `utils`, `tools`; stay consistent in one collection | [ ] |
| 6.5 | Workflows as a copyable checklist the agent ticks; feedback loops as "run validator, fix, repeat, proceed only when it passes" | [ ] |
| 6.6 | No time-sensitive text ("before August use v1"); an "Old patterns" `<details>` block instead | [ ] |
| 6.7 | One term per thing throughout (always "field", never "field, box, element") | [ ] |
| 6.8 | Templates: strict ("ALWAYS use this exact structure") for data shapes, flexible ("sensible default, use judgment") for prose | [ ] |
| 6.9 | Provide a default with an escape hatch, not a menu of libraries | [ ] |
| 6.10 | Forward slashes always, even on Windows. MCP tools by full name `Server:tool`. Say which packages to install; never assume | [ ] |
| 6.11 | Scripts solve, they do not defer: handle the missing file, name the reason for every constant ("30 s because slow connections"), print verbose errors that list the valid options | [ ] |
| 6.12 | Plan, validate, execute: for batch or destructive work, write a plan file, validate it with a script, then apply | [ ] |

## 7. Evaluation (best practices, Claude Code, skill-creator)

| # | Fact | Knew it |
|---|---|---|
| 7.1 | "Create evaluations BEFORE writing extensive documentation." Run tasks without a skill, record the gaps, write three scenarios, measure the baseline, write the minimum that passes, iterate | [ ] |
| 7.2 | Claude A writes the skill with you; Claude B, a fresh instance, runs it. Observe B, bring findings back to A. Claude handles the format natively; no special "writing skills" skill is needed to get valid frontmatter | [ ] |
| 7.3 | Watch how Claude navigates: files read in an odd order, references never followed, one file read every time (move it into `SKILL.md`), a file never read (cut it or signal it) | [ ] |
| 7.4 | `skill-creator` plugin: `evals/<case>/` folders with `prompt.md`, `case.yaml`, `graders/`; with-skill versus without-skill runs; `benchmark.json` with pass rate, time, tokens; blind A/B between versions; description tuning with should-trigger and should-not-trigger prompts; an HTML review viewer | [ ] |
| 7.5 | `claude plugin eval` runs the same case folders from a terminal or CI; a `tool_used: Skill` grader measures trigger rate after each description change | [ ] |
| 7.6 | Troubleshooting ladder: not triggering (keywords, `What skills are available?`, `/name`, `paths` inside cwd); too often (sharpen, or `disable-model-invocation`); stops following (a must-hold rule goes to a hook, judgment gets "every edit" wording, compaction needs re-invoke) | [ ] |

## 8. Where Skills Run, and What Does Not Travel (API)

| Surface | How skills get there | Sharing | Runtime |
|---|---|---|---|
| Claude Code | Folders in `~/.claude/skills/`, `.claude/skills/`, plugins, managed settings | Personal, project (commit it), plugin, org | Full network; install packages locally only |
| claude.ai and Cowork | Zip upload in Settings > Features; synced down to `~/.claude/skills/synced/` | Individual user only; no admin distribution | Network varies by admin setting |
| Claude API | `POST /v1/skills`; `skill_ids` in the `container` parameter with the code execution tool | Workspace-wide | No network, no package install, pre-installed packages only |
| Managed Agents | `skills` array on the agent, or a mounted GitHub repo | Per agent | Container |

| # | Fact | Knew it |
|---|---|---|
| 8.1 | "Custom Skills do not sync across surfaces." Upload separately to each | [ ] |
| 8.2 | Pre-built `pptx`, `xlsx`, `docx`, `pdf` skills exist on claude.ai and the API, not in Claude Code | [ ] |
| 8.3 | Security: a skill is software. Audit `SKILL.md`, scripts, images; skills that fetch external URLs can be turned by what they fetch. Enterprise can enable Skill content scanning for claude.ai and Cowork uploads, not for API uploads | [ ] |
| 8.4 | Not covered by zero data retention | [ ] |

## 9. Maturity Ladder

| Rung | You can | Evidence |
|---|---|---|
| 1 | Write a valid `SKILL.md` that triggers on the right phrase | `claude plugin validate` passes; `/name` and a natural request both fire it |
| 2 | Keep the body a delta and push depth one level down | Body under 100 lines for a judgment skill; references read only when needed |
| 3 | Choose invocation and scope on purpose | `disable-model-invocation`, `allowed-tools`, `paths`, `context: fork` each used once for a reason you can state |
| 4 | Bundle a script and a validator loop | A `scripts/` file the agent runs; a loop that proceeds only on pass |
| 5 | Prove it with a baseline | Three eval cases; with and without runs; a written delta |
| 6 | Carry the non-negotiables as hooks | A `hooks:` block or an agent-frontmatter hook; a blocked attempt in the log |
| 7 | Ship across surfaces | The same folder on Claude Code and uploaded to claude.ai or the API, with no path outside the folder |

## 10. Failure Modes

| Failure | Cause | Fix |
|---|---|---|
| Fires in a learning chat about AWS | Trigger phrase too broad ("on aws") | Sharpen, or make it user-invoked |
| Followed on turn 1, ignored on turn 9 | One-time wording; body not re-read | Standing wording; hook for must-hold rules |
| Works on this laptop, breaks on claude.ai | `C:\Users\...` paths, `` !`cmd` `` injection | `${CLAUDE_SKILL_DIR}`, bundle the file, point at an MCP tool |
| Agent reads `head -100` of a reference and misses the rule | Nested references | One level deep; table of contents over 100 lines |
| Description silently truncated | Listing over the 1% budget | `/doctor`, `/skill-doctor`, name-only overrides, shorter descriptions |
| Upload rejected | `argument-hint` or `context` in a skill sent to claude.ai or the API | Strip to the six spec fields for that copy |
| Cannot tell if the skill helps | No baseline | Three cases, run both ways, write the delta |

## 11. Exercises (one evening each)

1. Run `claude plugin validate ~/.claude/skills` and `/doctor`. Record the listing cost and the top three contributors.
2. Take one of your utility skills: add `allowed-tools` for the read commands, set `disable-model-invocation: true`, replace any absolute path with `${CLAUDE_SKILL_DIR}`. Confirm it still works and "explain S3 lifecycle on aws" no longer fires it.
3. Write three eval cases for one spec skill in `evals/<case>/`, run with and without via `skill-creator`, and keep `benchmark.md`.

## 12. Sources (all read 2026-11-04)

| Source | What it owns |
|---|---|
| https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview | Levels, architecture, surfaces, sharing, runtime limits, security |
| https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices | Writing rules, patterns, evaluation method, checklist |
| https://code.claude.com/docs/en/skills | Claude Code frontmatter, invocation, lifetime, budget, troubleshooting, evals |
| https://code.claude.com/docs/en/hooks | `hooks:` in skill and agent frontmatter, `once`, events |
| https://agentskills.io/specification | The open standard: fields, constraints, layout, validation |
| https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills | Why skills, the onboarding-guide framing, lessons, direction |

## 13. Self-Check

- Name the three levels and what each costs.
- Which six frontmatter fields survive an upload to claude.ai?
- What happens to a skill's body after compaction, and where should the important lines sit?
- When does `context: fork` return nothing useful?
- What is the first step of evaluation-driven development, before any writing?
- Which of your current skills would break on another machine, and why?
