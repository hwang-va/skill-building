# Vocabulary: the terms used in this folder

**BLUF:** One row per term, one meaning each, grouped by where the term lives: the method, the skill file, loading and triggering, writing, testing, the harness and the team. Use these words and no synonyms when writing a skill or explaining one; one term per thing is itself a rule from the Anthropic best-practices page. Source column says where the term comes from so a reader can go deeper. Started 2026-11-06.

## 1. The Method (`method-elicit-diff-encode.md`)

| Term | Meaning | Source |
|---|---|---|
| Elicitation | Pulling the expert's real process out of them, including the parts they never write down. Move 1 | the author, 2026-11-04; discovery-interview craft |
| Delta analysis | Running the agent cold on the same task and diffing it against the expert's process. The diff is the skill. Move 2 | the author, 2026-11-04; Superpowers baseline |
| Encoding | Writing the diff in a form the agent follows every run: anchors, gates, interfaces, hooks, evals. Move 3 | the author, 2026-11-04 |
| Process map | The written output of Elicitation: steps, the check at each step, the data each check needs and its source, the output the next step consumes | This folder |
| Tacit layer | The part of the expert's process they do without saying: first look, what they ignore, scars, what they never delegate | Elicitation question families |
| Scar list | Mistakes the expert made once and now guards against. Goes into the skill as the excuse table | Elicitation, question family 3 |
| Cold run | The agent doing the task with no skill, fresh session, same inputs. Also called the baseline | Superpowers RED step; `skill-creator` without-skill run |
| Diff | The line-by-line difference between the process map and the cold run. Four kinds: wrong, missing, order, shape | Delta analysis |
| No-op | A line the agent would obey anyway. Costs tokens, changes nothing. Deleted whole, never trimmed | Pocock, in-house compression skill |
| Misfire | The concrete wrong behaviour a line prevents. A line that cannot name its misfire is a no-op | in-house per-word audit |
| Delta (in a skill) | What differs from default model behaviour. The only thing a skill body should contain | Pocock, in-house compression skill |
| Focus point | One of the expert's own emphases, stated as a positive target in one line ("rate cash conversion before growth") | the author, equity-research example |
| Concrete to abstract | Start from one real case in one domain, lift it one level at a time until the essence is a sentence that holds in another domain. The sentence goes in the skill, the case stays as the register sample | skill-set handbook, not in this repo |
| Contract | The shared reference every skill in a set reads: parts, tests, columns, rules. Changed first, skills second | skill-set handbook, not in this repo |
| Register sample | The one concrete example kept beside the abstract rule, so a reader sees both | skill-set handbook, not in this repo |
| Built versus written | Written is a validated `SKILL.md`; built is a folder that skill produced from a real source. Only built counts as done | skill-set handbook, not in this repo |
| Bin, line, detector | Skill-set terms from a skill-set project: where a fact lands; one action with every slot filled; a rule that turns a folder into a collection row | skill-set handbook, not in this repo |

## 2. The Skill File

| Term | Meaning | Source |
|---|---|---|
| Skill | A folder with a `SKILL.md` that gives an agent a procedure, loaded on demand. Anthropic's phrase: an onboarding guide for a new hire | Anthropic overview |
| `SKILL.md` | The one required file: YAML frontmatter plus a Markdown body | Agent Skills spec |
| Frontmatter | The YAML block between `---` lines at the top. Holds `name`, `description`, and the Claude Code fields | Spec; Claude Code docs |
| `name` | The skill's identifier and slash command. 1 to 64 chars, lowercase, digits, hyphens; must match the folder | Spec |
| `description` | The trigger text, always in context. What the skill does and when to use it, third person, no procedure | Spec; best practices |
| Body | Everything after the frontmatter. Loaded on invocation, stays in context for the session | Claude Code docs |
| Bundled files | `references/`, `scripts/`, `assets/` and any other file in the folder. Loaded or run only when the body points at them | Spec |
| Reference | A Markdown file in the folder that the body links to, read on demand. One level deep from `SKILL.md` | Best practices |
| Script | Executable code in the folder. Runs through bash; only its output enters context | Overview |
| Asset | A static file the skill uses in its output: a template, a schema, a filled example | Spec |
| Anatomy | The layout of a skill folder and which file loads when | `map.md` §2 |

## 3. Loading and Triggering

| Term | Meaning | Source |
|---|---|---|
| Progressive disclosure | Loading in stages: metadata always, body on trigger, bundled files on demand. Also the act of moving material down that ladder so the top stays short | Overview; Pocock |
| Three levels | Level 1 metadata (about 100 tokens, always), Level 2 body (under 5,000 tokens, on trigger), Level 3 resources (zero until read) | Overview |
| Trigger | The moment the agent decides a skill applies, by matching the request against `description` | Claude Code docs |
| Outcome | What exists after the skill runs. The second half of a `Trigger: ... Outcome: ...` description | the in-house compression skill |
| Invocation | How a skill gets loaded: model-invoked (Claude decides), user-invoked (only `/name`), or forked (runs as a fresh subagent) | Claude Code docs; Pocock mechanics |
| Model-invoked | Default. Claude or the user can fire it; the description sits in context every turn | Pocock |
| User-invoked | `disable-model-invocation: true`. Only a typed `/name` fires it; description leaves the agent's context | Claude Code docs; Pocock |
| Forked skill | `context: fork`. The body becomes the prompt of a new subagent with no conversation history | Claude Code docs |
| Router skill | A user-invoked skill that names other user-invoked skills and when to reach each, so the human remembers one name | Pocock mechanics |
| Context load | The cost of always-loaded text on the agent's window, every turn: descriptions, CLAUDE.md lines | Pocock |
| Cognitive load | The cost on the human of remembering which skills exist and when to use them | Pocock |
| Context pointer | A line in context that names material outside context and the condition for reaching it. A description is one | Pocock |
| Listing budget | The share of the context window all skill descriptions may use, 1% by default. Over it, Claude Code drops descriptions from least-used skills | Claude Code docs |
| Sticky body | The body stays in the conversation after invocation and is not re-read. Hence standing instructions, not one-time steps | Claude Code docs |
| Compaction | Context summarisation when the window fills. The latest invocation of each skill is re-attached, up to 5,000 tokens per skill | Claude Code docs |
| Surface | Where a skill runs: Claude Code, claude.ai, Cowork, the API, Managed Agents. Skills do not sync across surfaces | Overview |
| Travel test | Does this reference still resolve when the skill leaves this laptop? Body and bundled files travel; local paths do not; an interface travels only where every surface authenticates it | `map.md` §1b |
| Interface | A live or shared data source reached by name: an MCP tool as `Server:tool`, a URL, a database | `map.md` §1b |

## 4. Writing

| Term | Meaning | Source |
|---|---|---|
| Anchor | A name the model already holds a definition for ("Porter five forces", "EARS"). Named, never described | in-house compression skill; Pocock's leading word |
| Leading word | A pretrained word that carries a whole region of behaviour, repeated through the body so its meaning accumulates (*tight*, *red*, *relentless*) | Pocock |
| Positive target | A rule stated as what to do, not what to avoid. Prohibition plants the banned idea | Pocock; `skill-creator` |
| Guardrail | A prohibition kept because it cannot be phrased positively; paired with the positive target. Candidates for a hook | Pocock; `map.md` §6b |
| Completion criterion | The condition that tells the agent a step is done. Checkable and demanding ("every number cites a tool call") | Pocock; in-house compression skill |
| Gate | A completion criterion the agent may not pass without: a yes from the human, a validator pass, a login check | This folder |
| Degrees of freedom | How much the skill constrains: high (heuristics), medium (pattern with parameters), low (exact script, no flags). Match to fragility | Best practices |
| Excuse table | Rationalizations the agent used to skip a rule, each answered. For discipline skills | Superpowers |
| Red flags | Phrases that signal a corner is about to be cut, listed so the agent notices itself | Superpowers |
| Feedback loop | Run validator, fix, repeat, proceed only on pass | Best practices |
| Plan, validate, execute | For batch or destructive work: write a plan file, validate it with a script, then apply | Best practices |
| Telegraphic | Lossy grammar, lossless content: drop articles and filler, keep every fact, path and negation | in-house telegraphic and compression skills |
| Sprawl | A body too long even when every line is live; attention thins. Cured by disclosure and splitting | Pocock |
| Sediment | Stale lines that stay because removing feels risky. Cured by the relevance check per line | Pocock |

## 5. Testing

| Term | Meaning | Source |
|---|---|---|
| Baseline | The cold run's result, kept as the comparison point | Superpowers; `skill-creator` |
| Eval case | One folder under `evals/`: a prompt, a `case.yaml`, graders, optional input files | `skill-creator`; `claude plugin eval` |
| Grader | One assertion about the result, stated about the output itself, pass or fail | `skill-creator` |
| With and without | The same case run with the skill and with no skill (or the old version), side by side | `skill-creator` |
| Benchmark | Pass rate, time and tokens for each configuration, with the delta | `skill-creator` |
| Near-miss query | A should-not-trigger prompt that shares keywords with the skill. The valuable negative test | `skill-creator` description optimizer |
| Claude A, Claude B | The instance that writes the skill with you, and the fresh instance that runs it. Observe B, bring findings to A | Best practices |
| Checklist | `map.md` §5: thirteen lines scored 0, 1 or 2 per skill | This folder |

## 6. Harness and Team

| Term | Meaning | Source |
|---|---|---|
| Harness | The program around the model: Claude Code itself. Runs tools, hooks, permissions, compaction. Deterministic where the model is not | the author, 2026-11-04 |
| Hook | Harness code that runs on an event, before or after a tool call. Can block on cancelable events. The model cannot argue with it | Claude Code hooks doc |
| Cancelable event | One of 13 events where a hook may veto the action: `PreToolUse`, `Stop`, `UserPromptSubmit` and others | an in-house hook-authoring skill |
| Carried hook | A hook declared in a skill's `hooks:` frontmatter, registered on invocation, kept for the session; `once: true` drops it after the first run | Claude Code hooks doc |
| Armed guardrail | A skill that is nothing but a carried hook, user-invoked. Typing its name arms the rule for the session | `map.md` §6b |
| Agent definition | A `.claude/agents/*.md` file: a role's system prompt, tools and preloaded skills. The hire, where the skill is the job description | Claude Code docs; `map.md` §6 |
| Agent team | Several agent definitions, each running its skills, composed through forks, handoffs and hooks. What the author is building | the author, 2026-11-04 |
| Plugin | A folder that ships skills, agents and hooks together, namespaced `/plugin:skill` | Claude Code docs |
| MCP tool | A capability exposed by an MCP server, named `Server:tool` inside a skill. The skill is the process on top; the MCP is the capability underneath | Best practices; `mcp-builder` |
