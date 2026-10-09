# Skill Pack Teardown: Gstack vs Pstack vs Matt Pocock's Skills

## At a Glance

| Dimension | Gstack (Garry Tan) | Pstack (Lauren Tan / Poteto) | Matt Pocock's Skills |
|---|---|---|---|
| **Version** | 1.91.67.0 | Not versioned (mirror) | 1.3.1 |
| **Skill count** | 50+ | 55 (31 action + 24 principles) | 27 |
| **Philosophy** | "Boil the Ocean" -- do the complete thing, AI makes thoroughness free | "Go deep first" -- anti-slop, less code but higher quality | "Skills for Real Engineers, not vibe coding" -- composable, hackable, grounded in classic eng literature |
| **Author background** | YC President/CEO | React core team, Cursor engineer (ex-Meta, Netflix) | TypeScript educator, podcaster |
| **Primary agent** | Claude Code | Cursor (cross-harness) | Claude Code (cross-harness) |
| **Lifecycle coverage** | Brainstorm to post-deploy monitoring | Understanding to overnight autonomous runs | Grilling to retro |
| **Tech stack bias** | TypeScript/Bun, macOS, Aside browser | TypeScript, Cursor, multi-model | TypeScript, Git/GitHub |
| **Complexity** | Full software product (compiled binaries, browser engine, build system) | Mid-weight (SKILL.md + references + playbooks + subagents) | Lightest (hand-written SKILL.md files, no build step) |
| **Installation** | `git clone` + `./setup` (builds binaries, installs Chromium) | `/add-plugin pstack` (Cursor) or `npx skills add` | `claude plugin install` or `npx skills@latest` |
| **Agent support** | Claude Code, Codex, Cursor, Copilot, Kiro, OpenCode, +5 more | Cursor (primary), Claude Code, Codex, Pi, OpenCode | Claude Code, Codex, Copilot, Gemini, Cursor, Windsurf, Devin, Amp |

---

## Wait -- These Aren't "Just Markdown"

The backlash frames skill files as "people getting famous for writing markdown." That framing is wrong. These are invokable slash commands with routing, context loading, and cross-agent plugin systems. The three packs sit on a spectrum from "well-structured prompt templates" to "a full development platform."

### How They Actually Work

When you see `/tdd` or `/ship`, that's a literal slash command. You type it in your agent, and the agent loads that skill's SKILL.md into its context window on demand. Skills are not passive background instructions the agent reads at startup -- they are tools the agent activates when called.

**Two types of invocation exist across all three packs:**

- **User-invoked:** You type `/tdd`. The agent loads the SKILL.md and follows its instructions.
- **Model-invoked:** The agent autonomously decides to invoke a skill when the task matches trigger phrases in the skill's metadata. For example, if you start debugging, the agent might auto-fire `/diagnosing-bugs`. This can be disabled per-skill.

### How Users Discover Commands

| Pack | Discovery mechanism |
|---|---|
| **Gstack** | Root SKILL.md is a router that pattern-matches natural language to the right skill. Type a vague request, get routed. |
| **Pstack** | `/poteto-help` -- "stuck? ask me." Plus Cursor's `/` menu lists all registered skills. |
| **Matt's** | `/ask-matt` -- a hand-maintained router mapping situations to skills. "You don't remember every skill, so ask." |

### What's Actually In the Folder

This is where the "just markdown" framing breaks down completely.

**Matt's Skills (lightest):**
```
skills/
  engineering/
    tdd/
      SKILL.md              # The instructions (hand-written markdown)
      tests.md              # Reference doc the skill reads
      mocking.md            # Reference doc
      agents/openai.yaml    # Codex-compatible metadata
  productivity/
    ...
.claude-plugin/
  plugin.json               # Plugin manifest for Claude Code
  marketplace.json           # Auto-update registration
```
No build step. No binaries. No templates. The SKILL.md IS the source of truth. Distributed via Claude Code's plugin system with one-command install. The closest thing to "just markdown" -- but even here, the plugin manifest, per-agent metadata files, and the invocation system (user-invoked vs model-invoked, controlled by frontmatter flags) make it more than a folder of text files.

**Pstack (mid-weight):**
```
skills/
  poteto-mode/
    SKILL.md                # The central router
    playbooks/              # 23 playbook files it dispatches to
    scripts/                # Scripts the skill uses
  interrogate/
    SKILL.md
    references/             # Supporting docs (rubric, reviewer prompts)
      rubric.md
      reviewer-prompt.md
      code-quality-review.md
agents/
  poteto-agent.md           # Reusable subagent definition
  comment-sicko.md          # Anti-comment reviewer persona
automations/benny/          # Dormant Slack-to-PR bug pipeline
.cursor-plugin/
  plugin.json               # Cursor plugin manifest
```
No compiled code, but the playbook routing system, subagent definitions, and the Benny automation pack make this a structured system, not a flat collection.

**Gstack (heaviest -- a full software product):**
```
ship/
  SKILL.md                  # GENERATED from template (never edit directly)
  SKILL.md.tmpl             # The actual source (with template variables)
  sections/                 # Modular sections composed into the template
  bin/                      # TypeScript executables
bin/                        # 60+ CLI utilities
  gstack-skill-start        # Preamble script every skill runs
  gstack-skill-end          # Telemetry logging
  gstack-learnings-log      # Cross-session learning
  gstack-decision-log       # Append-only decision store
  gstack-next-version       # Version queue for concurrent branches
browse/                     # Headless Chromium browser (~62MB compiled binary)
design/                     # Design detection binary
make-pdf/                   # PDF generation binary
lib/                        # Shared TypeScript libraries
  redact-patterns.ts        # PII/secrets guard
  aside-render.ts           # Browser integration
scripts/
  gen-skill-docs.ts         # Renders .tmpl -> SKILL.md
test/                       # E2E evals and unit tests
extension/                  # Chrome extension
hosts/                      # Per-agent configurations
agents/openai.yaml          # Codex metadata
.github/workflows/          # CI pipelines
```
SKILL.md files are **generated** from `.tmpl` templates using a build system. Template variables like `{{ASIDE_SETUP}}` and `{{BROWSE_FALLBACK}}` get resolved per-host. Every skill session starts by running a preamble script that outputs machine-readable status lines. Telemetry logs outcomes. A cross-session decision store persists in `~/.gstack/projects/`. This is a development platform that happens to output markdown as its user-facing interface.

### Context Loading: Not "Read Everything at Once"

None of the packs dump all their skills into the agent's context window at startup. That would blow the token budget immediately. Instead:

- **Gstack:** The root router SKILL.md is always loaded (small). When you type a command or describe a task, the router invokes the specific skill via Claude Code's Skill tool, which loads that one SKILL.md on demand. Skills have token budgets (warn above 160KB / ~40K tokens).
- **Matt's:** Skills are registered through the plugin manifest. The agent's slash-command system loads the right SKILL.md when you type its name. Model-invoked skills load when trigger phrases in the frontmatter match.
- **Pstack:** `/poteto-mode` loads first, matches your task to a playbook, then invokes other skills as the playbook steps require them. In Cursor, "custom mode" (option+enter) keeps `/poteto-mode` persistent across turns.

### Agent Harness Compatibility

| Pack | Primary | Installation | Other agents |
|---|---|---|---|
| **Gstack** | Claude Code | `git clone` + `./setup` (builds binaries) | `./setup --host codex`, `--host cursor`, `--host copilot`, etc. Renders host-specific SKILL.md files. |
| **Pstack** | Cursor | `/add-plugin pstack` | `npx skills add backnotprop/pstack` for Claude Code, Codex, Pi, OpenCode |
| **Matt's** | Claude Code | `claude plugin install mattpocock-skills` | `codex plugin marketplace add`, `copilot plugin install`, `gemini skills install`, `npx skills@latest add` |

---

## Philosophy Comparison

### Gstack: "Boil the Ocean"
The old advice "don't boil the ocean" is legacy thinking. With AI, the marginal cost of completeness is near-zero. Always write the tests. Always handle the edge cases. Always do the full thing. Tan claims 810x improvement in logical code output (11,417 lines/day in 2026 vs 14 in 2013). The system embodies this by covering every phase from "is this idea worth building?" (`/office-hours`) to "is production healthy after deploy?" (`/canary`).

### Pstack: "Go Deep First"
The opposite instinct, same goal. If you want to go fast, go deep first. Pstack is anti-slop at its core -- it opposes the common complaint that AI writes too much low-quality code. The answer isn't more code, it's less code that's better. The laziness protocol biases toward deletion. "Minimize the diff." Multi-model adversarial review catches what a single model misses.

### Matt's Skills: "Composable Engineering Fundamentals"
Positioned against monolithic AI workflow frameworks that "own the process." Each skill does one thing. They compose into chains but don't require each other. The intellectual foundation is explicit: Ousterhout's deep modules, Feathers' seams, Beck's red-green, Fowler's code smells, Evans' ubiquitous language, Hunt & Thomas' tracer bullets.

---

## The Overlap Map

These are the capabilities that appear across multiple packs -- the things the community has converged on as essential.

### Present in All Three

| Capability | Gstack | Pstack | Matt's Skills |
|---|---|---|---|
| **TDD / test-first** | Embedded in QA skills, test-audit | `/tdd` skill | `/tdd` skill with strict red-green |
| **Code review** | `/review` with SQL/LLM trust checks | `/interrogate` multi-model panel | `/code-review` two-axis (standards + spec) |
| **Anti-slop / voice rules** | `slop-scan`, banned vocabulary list | `/unslop` with 33 cataloged patterns | `wait-what` (3-line anti-verbosity) |
| **Git/PR discipline** | `/ship` with version queue, atomic commits | PR playbook, conventional commits | `/pr` with summary/evidence/merge-danger |
| **Planning / spec** | `/spec`, `/office-hours`, `/autoplan` | `/poteto-mode` with 23 playbooks | `/grill-with-docs` -> `/to-spec` -> `/to-tickets` |
| **Bug investigation** | `/investigate` (no fix without investigation) | `/why` + `/blast-radius` | `/diagnosing-bugs` (6-phase loop) |
| **Context handoff** | `/context-save` + `/context-restore` | `/recall` + `/show-me-your-work` | `/handoff` |
| **Retrospective / learning** | `/retro` + `/learn` | `/reflect` + `/correct` | `/retro` |
| **Documentation** | `/document-release`, `/document-generate` (Diataxis) | `/technical-writing` (Diataxis + Google + STE) | `/writing-for-agents` |

### Present in Two of Three

| Capability | Who Has It | Who Doesn't |
|---|---|---|
| **Multi-model review** | Gstack (`/codex`), Pstack (`/interrogate`, `/arena`) | Matt's |
| **Design system / visual review** | Gstack (4 design skills), Pstack (`/architect`) | Matt's |
| **Browser QA** | Gstack (Aside + headless Chromium), Pstack (verification skills) | Matt's |
| **Prototype / exploration** | Gstack (`/design-shotgun`), Matt's (`/prototype`) | Pstack |
| **Issue triage** | Pstack (Benny automation), Matt's (`/triage`) | Gstack |
| **Domain modeling / glossary** | Pstack (`principle-model-the-domain`), Matt's (`/domain-modeling` + GLOSSARY.md) | Gstack |
| **Safety hooks** | Gstack (`/careful`, `/freeze`, `/guard`), Pstack (principles) | Matt's |
| **Overnight / autonomous runs** | Gstack (context save/restore), Pstack (autopilot playbooks + decision logs) | Matt's |

### Unique to Each Pack

**Gstack only:**
- iOS QA on real hardware (USB CoreDevice tunnel, vision-driven agent loop)
- Post-deploy canary monitoring
- Scrape-to-skill pipeline (`/scrape` -> `/skillify`)
- Developer psychographic profiling (`/plan-tune`)
- Security audit (`/cso` with OWASP scope)
- Cross-session decision store (append-only, event-sourced)
- PDF generation, diagram generation
- Health dashboard (weighted 0-10 composite score)
- YC-style office hours product brainstorming

**Pstack only:**
- 24 codified engineering principles as standalone skills
- Arena (fan out N candidates, pick best, graft)
- Swarm (parallel workers across slices)
- Comment Sicko (aggressive anti-comment reviewer persona)
- Automate-me (mines chat history to generate personal workflow)
- Benchmark-checklist (7 questions for vetting performance numbers)
- Unslop with 33 specific AI writing patterns cataloged
- Benny automation pack (Slack -> triage -> reproduce -> fix -> PR)
- Verification skill generator (project-local harnesses)

**Matt's Skills only:**
- Grilling primitive (frontier-based interview rounds)
- Teach skill (stateful learning workspace for any subject, not just code)
- Wizard (generates interactive bash scripts for manual procedures)
- To-questionnaire (turns blocked decisions into structured questionnaires)
- Wait-what (3-line anti-verbosity, triggers re-explanation)
- Tracer-bullet ticket decomposition with blocking edges
- Prototype preservation on named branches (never deleted, never merged)
- Improve-codebase-architecture (periodic deepening survey with HTML report)

---

## The Main Build Chains Compared

Each pack defines a primary workflow -- how you go from idea to shipped code.

### Gstack: The Review Gauntlet
```
/office-hours (brainstorm)
    -> /autoplan (CEO + design + DX + eng reviews, auto-decided)
        -> /spec (precise spec + GitHub issue)
            -> implementation (manual or agent in worktree)
                -> /review (pre-landing PR review)
                    -> /ship (version bump, changelog, PR)
                        -> /land-and-deploy (merge, CI, deploy)
                            -> /canary (post-deploy monitoring)
```

### Pstack: The Playbook Router
```
User describes goal
    -> /poteto-mode (matches to 1 of 23 playbooks)
        -> playbook sequences skills:
            /how + /why (understand)
                -> /architect (design with arena)
                    -> /tdd (test-first build)
                        -> /interrogate (multi-model review)
                            -> PR playbook (ship)
                                -> /reflect (capture learnings)
```

### Matt's Skills: The Composable Chain
```
/grill-with-docs (interview, resolve terms, write ADRs)
    -> /to-spec (synthesize spec, file issue)
        -> /to-tickets (vertical-slice tickets with blocking edges)
            -> /implement or /implement-spec (TDD at pre-agreed seams)
                -> /code-review (standards + spec axes)
                    -> /pr (summary, evidence, merge danger)
                        -> /retro (propose environment improvements)
```

---

## Strengths Summary

### Where Gstack Wins
- **Lifecycle completeness.** Nothing else covers brainstorm-to-canary. The post-deploy monitoring and iOS real-device QA are unique.
- **Real browser testing.** Driving the user's actual browser with real cookies and logins is qualitatively different from headless testing.
- **Safety infrastructure.** PreToolUse hooks that intercept destructive commands, freeze boundaries, egress receipts, PII redaction -- production-grade safety.
- **Self-tuning.** The question tuning and developer psychographic mean it adapts to you over time.

### Where Pstack Wins
- **Multi-model adversarial review.** Using model diversity as a review signal is genuinely novel and catches blind spots a single model has.
- **Anti-slop depth.** 33 cataloged AI writing patterns, Comment Sicko, and the laziness protocol make this the most thorough anti-slop system.
- **Playbook routing.** 23 playbooks remove the "which skill do I use?" decision burden. The user describes the goal; the system picks the path.
- **Principle-driven.** 24 principles as first-class skills that agents must cite when applied. This is the most philosophically rigorous pack.
- **Overnight autonomy.** Decision logs with cross-model review, finish conditions, and escape hatches make unattended runs auditable.

### Where Matt's Skills Win
- **Composability.** Each skill genuinely stands alone. No framework lock-in, no mandatory setup, no routing layer. Pick what you need.
- **Intellectual foundations.** Grounded in specific, named ideas from Ousterhout, Feathers, Beck, Fowler, Evans, and Hunt/Thomas. Not invented methodology.
- **The grilling primitive.** The frontier-based round system is a genuinely novel approach to structured interviews with AI.
- **Human-in-the-loop design.** The cleanest separation between "facts the agent finds" and "decisions the human makes."
- **Cross-agent compatibility.** Works on Claude Code, Codex, Copilot, Gemini, Cursor, Windsurf, Devin, Amp, and others.
- **Honesty.** Exhaustively catalogs known bugs with user quotes and issue numbers. Unusually transparent.

---

## Shared Gaps (Missing from All Three)

- **No database/migration skills.** Schema design, data migrations, and database-specific tooling are absent across the board.
- **No accessibility audit.** None has a dedicated WCAG/a11y skill.
- **No CI/CD authoring.** Deploy detection exists (Gstack) but writing/debugging CI configs is not covered.
- **No security beyond Gstack's CSO.** Pstack and Matt's have no security-focused skills.
- **No infrastructure-as-code.** Terraform, Kubernetes, Docker review -- none of them cover this.
- **No cost monitoring.** Cloud spend tracking is absent everywhere.

---

## Verdict: Do They Work?

The backlash is wrong on two levels.

**First, the "just markdown" framing is factually incorrect.** Even the simplest pack (Matt's) isn't passive markdown -- it's invokable slash commands with routing, model-invocation triggers, and cross-agent plugin manifests. The most complex (Gstack) is a full software product with compiled binaries, a browser engine, a build system, and a cross-session decision store. The markdown is the interface. What's behind it ranges from well-structured prompt templates to an entire development platform.

**Second, the substance works.** We ran the same prompt ("build me a to-do app") through the same model four times: once cold, once with each skill pack loaded. The results are in `contrast-test.md`. The cold model gave a competent, generic answer -- standard stack, working code samples, reasonable architecture. But it:
- Assumed features without asking (auth, priorities, drag-and-drop)
- Listed testing frameworks without testing philosophy
- Had no code review process, no PR structure, no post-deploy strategy
- Stopped at "deploy" -- the lifecycle ended there

Every skill-informed response changed the shape of the answer in three consistent ways:
1. **Questioned requirements** instead of assuming them
2. **Gave principles, not just tools** (test behavior at pre-agreed seams, not "use Vitest")
3. **Extended the lifecycle** before and after the code (grilling, spec, tickets, review, PR structure, canary, retro)

The delta is not about the model being smarter. It's about the model being **disciplined**. A raw model knows all of this -- it was trained on the same engineering literature. But without the skill file, it defaults to "here's a standard stack, go build." The skill file says: "wait. Think about this differently."

The skill file is the discipline. And the discipline works.

**Are they worth the hype?** Yes, with caveats:
1. They work best when you actually use the process, not just install the files
2. The value scales with project complexity -- a toy app won't show the difference
3. Each pack has a different philosophy -- pick the one that matches how you think
4. They are not "just markdown" -- the best ones are structured systems with routing, invocation rules, and cross-agent compatibility

**Which one should you use?**
- **Gstack** if you want maximum lifecycle coverage and don't mind complexity. It's a development platform, not a prompt collection. Best on macOS with Claude Code. (50+ skills, compiled binaries, browser integration)
- **Pstack** if you want multi-model review, anti-slop discipline, and playbook-driven routing. Best on Cursor. (55 skills, 23 playbooks, 24 codified principles)
- **Matt's Skills** if you want composable fundamentals grounded in classic engineering literature. The lightest, the most portable, the easiest to understand. Works on everything. (27 skills, no build step, plugin-based install)
