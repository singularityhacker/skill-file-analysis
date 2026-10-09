# Do Skill Files Actually Work?

**The Markdown Phenomenon: A Teardown of Gstack, Pstack, and Matt Pocock's Skills**

---

## The Backlash

Last week, Markus Eisele dropped a post that hit 140K views in a day:

> "I still can't wrap my head around the fact that people get famous for writing markdown files, calling them engineering skills and putting them on GitHub."

Mo followed up with a video calling the whole thing out. "The Markdown thing is getting out of hand."

And I get it. On the surface, the skill file phenomenon looks absurd. People sharing markdown files on GitHub like they just invented software engineering. Influencers getting famous for writing prompt templates. The whole thing has a whiff of snake oil.

But here's the thing -- I've been building a production banking app in Elixir with AI agents on both sides of the product, and I have opinions about this. The backlash is wrong, but not for the reason the skill file creators would tell you. It's wrong because the people making fun of it haven't actually looked at what's inside these packs, and the people defending it haven't proven that they work.

So I did both. I downloaded the three most talked-about skill packs, tore them apart, ran controlled tests, and came to a conclusion. This is that analysis.

---

## The Context

This didn't come out of nowhere.

Garry Tan's **Gstack** has been circulating for months. The YC president building a skill pack for AI-assisted development was already a conversation starter. Then things escalated. Lauren Tan (Poteto), a React core team member and engineer at Cursor, went on **Matt Pocock's podcast**. During the conversation, they mentioned that they use each other's skill packs. Matt uses Pstack. Poteto uses Matt's skills. Cross-pollination between people who are supposedly just "writing markdown."

That podcast appearance is what triggered the backlash wave. Mo's response, Eisele's post -- the timing isn't a coincidence. The skill file community went from niche curiosity to public target in about a week.

My response isn't to defend anyone's pack. It's to do what nobody else has done: **run evals**.

---

## What a Skill File Actually Is

Before the teardown, let's kill the biggest misconception.

**Skill files are not passive markdown that sits in your project.** They are invokable slash commands. When you see `/tdd` or `/ship`, that is a literal command you type into your AI agent. The agent loads that skill's instructions into its context window on demand and follows them as a structured workflow.

This matters because the "just markdown" framing is factually wrong at every level of complexity.

### The Complexity Spectrum

The three packs I analyzed sit on a spectrum:

**Matt Pocock's Skills** is the lightest. Hand-written SKILL.md files, reference docs, no build step, no binaries. Distributed as a Claude Code plugin you install with one command. This is the closest thing to "just markdown" -- but even here, the plugin manifest, per-agent metadata files, and the invocation system (user-invoked vs model-invoked, controlled by YAML frontmatter) make it a structured system, not a text file.

**Pstack** adds playbooks, subagents, and routing. The central `/poteto-mode` skill matches your task description to one of 23 playbooks and sequences other skills as the steps require. Subagent definitions ship as separate files. An automation pack called Benny can triage Slack bug reports, reproduce them through real UI, and open PRs. No compiled code, but this is clearly a system.

**Gstack** is a full software product. 60+ CLI utilities written in TypeScript. Compiled Bun binaries (~62MB each). A headless Chromium browser. A Chrome extension. Template rendering that generates SKILL.md files from `.tmpl` sources with per-host configuration. A build system. A test suite. An event-sourced decision store. Version 1.91.67.0.

Calling Gstack "a markdown file" is like calling a car "a steering wheel." The markdown is the interface. What's behind it is an entire development platform.

### Two Types of Invocation

Across all three packs, skills work in two modes:

**User-invoked:** You type `/ship`. The agent loads the skill and follows it. This is for high-stakes workflows where human intent matters.

**Model-invoked:** The agent autonomously decides to invoke a skill when the task matches trigger phrases in the skill's metadata. You start debugging, and the agent fires `/diagnosing-bugs` on its own. This can be disabled per-skill.

### How Users Find the Right Command

Nobody memorizes 50+ slash commands. Each pack solves this differently:

- **Gstack** has a root router that pattern-matches natural language. Describe what you want; it routes you.
- **Pstack** has `/poteto-help`. "Stuck? Ask me."
- **Matt's** has `/ask-matt`. A hand-maintained router mapping situations to skills.

### Context Loading

None of these packs dump all their skills into the agent's context at once. That would blow the token budget immediately. Skills load on demand when invoked, one at a time, with token budgets enforced (Gstack warns above 160KB per skill). The router or plugin system handles dispatch.

### Agent Compatibility

These aren't locked to one tool:

| Pack | Primary Agent | Also Works With |
|---|---|---|
| Gstack | Claude Code | Codex, Cursor, Copilot CLI, Kiro, OpenCode, +5 more |
| Pstack | Cursor | Claude Code, Codex, Pi, OpenCode |
| Matt's Skills | Claude Code | Codex, Copilot, Gemini CLI, Cursor, Windsurf, Devin, Amp |

---

## The Teardown

I cloned all three repos and had agents read every SKILL.md, every reference doc, every config file. Here's what's inside.

### Gstack: 50+ Skills, Brainstorm to Canary

**Author:** Garry Tan, President & CEO of Y Combinator
**Philosophy:** "Boil the Ocean. The marginal cost of completeness is near-zero with AI."

Gstack is the maximalist approach. It covers every phase of software development from "is this idea worth building?" to "is production healthy after deploy?"

The headline skills:

- `/office-hours` -- YC-style product brainstorming. Reframes your idea before you write code.
- `/autoplan` -- Runs CEO, design, DX, and engineering reviews sequentially with auto-decisions. One command, fully reviewed plan.
- `/spec` -- Turns vague intent into a precise spec in five phases. Files a GitHub issue.
- `/review` -- Pre-landing PR review. Checks for SQL injection, LLM trust boundary violations, conditional side effects.
- `/qa` -- Full QA across browser, API, CLI, job, worker, and webhook surfaces. Three tiers: Quick, Standard, Exhaustive.
- `/ship` -- Version bump, changelog, PR creation with a workspace-aware version queue.
- `/land-and-deploy` -- Merges, waits for CI, deploys, verifies production health.
- `/canary` -- Post-deploy monitoring loop with periodic screenshots and anomaly detection.
- `/ios-qa` -- Live-device iOS QA via USB. Vision-driven agent loop on a real iPhone.
- `/cso` -- OWASP-scoped security audit with PII/secrets redaction.

**What makes it unique:** Real browser testing (drives your actual browser with real cookies via Aside on macOS), iOS QA on physical hardware, a cross-session decision store that remembers what you decided and resurfaces it before re-deciding, developer psychographic profiling that learns which questions to stop asking you, and post-deploy canary monitoring. Nobody else has any of this.

**The build chain:**
```
/office-hours -> /autoplan -> /spec -> build -> /review -> /qa -> /ship -> /land-and-deploy -> /canary
```

### Pstack: 55 Skills, Anti-Slop Philosophy

**Author:** Lauren Tan (Poteto), React core team, Cursor engineer
**Philosophy:** "If you want to go fast, go deep first."

Pstack is the anti-slop pack. Where Gstack says "do more," Pstack says "do less, but better." The laziness protocol biases toward deletion. "Minimize the diff." "If a human developer would find the code exhausting to maintain, it is a bad solution."

The 55 skills break into 31 action skills and 24 codified engineering principles. The principles are first-class skills that agents must cite when they apply them. This is the most philosophically rigorous pack.

Headline skills:

- `/poteto-mode` -- The central router. Matches your task to one of 23 playbooks.
- `/interrogate` -- Multi-model adversarial code review. Spawns reviewers on different models, synthesizes into Act / Consider / Noted / Dismissed buckets.
- `/arena` -- Fans out N parallel candidates at the same task, picks a base, grafts the best parts from losers.
- `/unslop` -- Removes AI tells from writing. 33 cataloged patterns: em dashes, hedging, filler, abstract metaphors, chatbot phrases.
- `/tdd` -- Test-driven bug fix. Failing test first, then the fix. Pragmatically skips when tests would be impractical.
- `/correct` -- Mines history for mistakes agents keep repeating and fixes each at the highest level that works (architecture > types > lint > tests > docs).
- `/automate-me` -- Mines your chat history to generate a personal workflow skill.
- **Comment Sicko** -- A deliberately extreme anti-comment reviewer persona shipped as its own subagent.

The **Benny automation pack** deserves special mention: a dormant pipeline that takes Slack bug reports, triages them, reproduces bugs through real UI interaction twice, and opens bounded draft PRs with before/after proof. Production-grade automation with serious safety engineering.

**What makes it unique:** Multi-model adversarial review (using model diversity as a signal), 33 cataloged AI slop patterns with fixes, 24 codified principles as first-class skills, the arena competitive design system, and overnight autonomous runs with decision logs and cross-model trail review.

**The build chain:**
```
Describe goal -> /poteto-mode (picks playbook) -> /how + /why -> /architect -> /arena -> /tdd -> /interrogate -> PR -> /reflect
```

### Matt Pocock's Skills: 27 Skills, Composable Fundamentals

**Author:** Matt Pocock, TypeScript educator and podcaster
**Philosophy:** "Skills for Real Engineers, not vibe coding."

Matt's pack is positioned against monolithic AI workflow frameworks that "own the process." Each skill does one thing. They compose into chains but don't require each other. The intellectual foundation is explicit and named: Ousterhout's deep modules, Feathers' seams, Beck's red-green, Fowler's code smells, Evans' ubiquitous language, Hunt & Thomas' tracer bullets.

Headline skills:

- `/grill-with-docs` -- Interviews you about a plan while writing resolved terms to `GLOSSARY.md` and decisions to ADRs in real time. The head of the main build chain.
- `/to-tickets` -- Breaks a plan into tracer-bullet vertical-slice tickets with explicit blocking edges. No horizontal "build all the DB first."
- `/implement` -- TDD at pre-agreed seams, typecheck, full suite, code review, commit. One ticket per invocation.
- `/code-review` -- Two-axis review: standards (Fowler's 12 code smells) and spec compliance. Each axis runs in its own sub-agent to prevent cross-contamination.
- `/pr` -- PR body structure: Summary (a visual, not prose), Evidence (before/after), Merge Danger (one-way vs two-way door + blast radius).
- `/diagnosing-bugs` -- Six-phase disciplined diagnosis loop. Will not let the agent theorize without a test that goes red on the bug.
- `/teach` -- Stateful learning workspace for any subject, not just code. Users learn Korean, piano, guitar, board game design.
- `/wait-what` -- Three lines. The entire skill says "wait, you lost me" to trigger re-explanation. A 400-line concision skill still leaves the model verbose.

**What makes it unique:** The grilling primitive (frontier-based interview rounds), GLOSSARY.md with \_Avoid\_ synonyms as the ubiquitous language artifact, tracer-bullet ticket decomposition, prototype preservation on named branches, and exceptional transparency about known bugs.

**The build chain:**
```
/grill-with-docs -> /to-spec -> /to-tickets -> /implement -> /code-review -> /pr -> /retro
```

---

## Where They Converge

Nine capabilities appear across all three packs. This is the industry's emerging consensus on what AI agents need to be told:

1. **TDD / test-first development**
2. **Code review as a defined process** (not "look at the code")
3. **Anti-slop rules** (banned vocabulary, style enforcement)
4. **Git/PR discipline** (atomic commits, structured PR bodies)
5. **Planning before code** (spec, interview, playbook)
6. **Bug investigation before fixing** (reproduce first, no guessing)
7. **Context handoff** between sessions
8. **Retrospective / learning loops**
9. **Documentation standards**

The fact that three independently developed packs all converge on the same nine capabilities is itself an argument that skill files work. These aren't arbitrary choices. They're the things a raw model consistently skips.

---

## The Contrast Test

Here's the proof.

I ran the same prompt through the same model (Claude Opus 4.6) four times: once cold, once with each skill pack's philosophy and key files pre-loaded. The prompt: **"I want to build a to-do app. What would you set up for me?"**

### The Cold Model

A competent, generic senior engineer answer. React + Express + Postgres + Prisma monorepo. Working CI/CD YAML. Sensible architecture decisions. Reasonable "what to leave out" section.

But it:
- **Assumed features without asking.** Auth, priorities, drag-and-drop, monorepo -- all decided for me.
- **Listed testing frameworks without testing philosophy.** "Use Vitest" is not a testing strategy.
- **Had no code review process.** Not mentioned.
- **Had no PR structure.** Not mentioned.
- **Stopped at deploy.** The lifecycle ended there. Nothing about what happens after the code is running.
- **Chose Postgres without justification.** Just the default.

### What Changed With Skill Packs

Every skill-informed response changed the shape of the answer in the same three ways:

**1. Requirements discovery.** All three questioned the premise instead of assuming features. Matt's agent listed six unresolved branches: "Do items have due dates? Is this multi-user? What does 'complete' mean?" Pstack pushed back entirely: "The question is backwards. Plan after the design settles." Gstack structured a process for resolving ambiguity through its spec and autoplan pipeline.

**2. Principles over tools.** The cold model said "use Vitest." The skill-informed models said "test behavior at pre-agreed seams, never mock your own modules, ban three anti-patterns." The cold model listed frameworks. The skill packs gave philosophy.

**3. The full lifecycle.** The cold model's lifecycle was scaffold -> implement -> deploy. Every skill pack extended this in both directions: grilling/interviewing and spec writing before code, and code review, PR structure, retrospectives, and (in Gstack's case) canary monitoring after deploy.

One more thing: **all three skill-informed responses argued for SQLite over Postgres**, each with different reasoning. Gstack cited "search before building" (Bun has a built-in SQLite driver). Pstack cited the laziness protocol (graduate to Postgres when you need to, not before). Matt's cited simplicity. The cold model defaulted to Postgres because that's what senior engineers default to.

The raw model knows all of this. It was trained on the same engineering literature these packs cite. But without the skill file, it defaults to "here's a standard stack, go build." The skill file says: **wait. Think about this differently.**

---

## What Nobody Covers

All three packs share the same gaps:

- **No database/migration skills.** Schema design, data migrations, database-specific tooling -- absent everywhere.
- **No accessibility audit.** No dedicated WCAG/a11y skill in any pack.
- **No CI/CD authoring.** Deploy detection exists but writing CI pipelines is not covered.
- **No infrastructure-as-code.** Terraform, Kubernetes, Docker -- none of them.
- **No non-TypeScript language skills.** Process skills are language-agnostic. Code-level skills assume TypeScript. Nothing for Elixir, Python, Go, Rust, or Java.
- **No cost monitoring.** Cloud spend tracking is absent everywhere.

That last gap -- non-TypeScript languages -- is the one I'm filling.

---

## The Verdict

The backlash is wrong on two levels.

**First, the "just markdown" framing is factually incorrect.** Even the lightest pack is an invokable slash command system with routing, model-invocation triggers, and cross-agent plugin manifests. The heaviest is a full software product with compiled binaries, a browser engine, and a build system. Calling these "markdown files" is like calling a recipe "just ink on paper."

**Second, the substance works.** The contrast test proves it. A raw model gives you a competent but generic answer. A skill-informed model questions your requirements, gives you engineering principles instead of tool recommendations, and extends the development lifecycle in both directions. The delta is consistent across all three packs.

The skill file is not making the model smarter. It's making the model **disciplined**. And discipline is the thing AI-assisted development is missing.

Are they worth the hype? Yes, with caveats:

1. **They work best when you use the process, not just install the files.** A skill pack sitting in your repo doing nothing is, in fact, just markdown.
2. **The value scales with complexity.** A toy app won't show the difference. A production app with tests, deploys, and multiple contributors will.
3. **Pick the philosophy that matches how you think.** Gstack if you want to boil the ocean. Pstack if you want to go deep first. Matt's if you want composable fundamentals.
4. **None of them cover your stack.** If you're not writing TypeScript, the code-level skills don't apply. The process skills transfer, but you're on your own for language-specific patterns.

That last point is why I'm building [Estack](link-tbd) -- a skill pack for Elixir and Phoenix, extracted from a production banking app where AI agents are on both sides of the product. The process ideas come from the three packs above. The Elixir patterns come from building on the BEAM where getting it wrong costs money. There's some irony in concluding a skill pack evaluation by shilling my own, but that's the honest outcome: I did the analysis, saw what was missing, and started filling the gap. More on that in a follow-up piece on why Phoenix might be the best framework for agent-assisted development.

---

## Footnote: Phoenix Is Already Ahead

One thing I noticed while doing this analysis: the Elixir ecosystem may be further along on the agent-readiness front than anyone in the TypeScript world realizes.

Phoenix 1.8, launched in August 2025, ships with an `AGENTS.md` file by default. Every new Phoenix project generates agent instructions that correct common LLM mistakes with Elixir syntax, idioms, and framework APIs. This is baked into the generator. You get it for free.

On top of that, [UsageRules](https://github.com/ash-project/usage_rules) -- from the Ash Framework team -- auto-generates agent instructions from your project's installed dependencies. Library authors can ship machine-readable rules alongside their packages. And [Tidewave](https://tidewave.ai/), built by a team including Elixir creator Jose Valim, gives agents runtime access to the running Phoenix app through MCP -- evaluating code, querying the database, inspecting logs, and browsing the rendered UI.

Skills tell agents what a framework is supposed to do. Tidewave lets them see what the application is actually doing. Phoenix is building both sides of that equation into the ecosystem itself.

None of the TypeScript skill packs have anything like this. The framework doesn't ship agent instructions. The libraries don't ship machine-readable rules. The runtime doesn't expose itself to agents. The TypeScript community is building agent discipline on top. The Elixir community is building it into the foundation.

That's a different article. But it's worth noting that while everyone is arguing about whether markdown files work, one ecosystem quietly made the framework itself agent-native.

---

*This analysis was performed by downloading and reading every file in all three repos, running controlled contrast tests with the same model and prompt, and building the Estack patterns from a production codebase. The full teardown data, contrast test transcripts, and diagrams are in the [companion repo](link-tbd).*
