# The Contrast Test: Raw Model vs. Skill Packs

**Prompt:** "I want to build a to-do app. What would you set up for me?"
**Model:** Claude Opus 4.6 (same model, same prompt, four sessions)
**Variables:** One cold session (no skills), three sessions each pre-loaded with one skill pack's philosophy and key files.

---

## The Scoreboard

| Dimension | Cold (No Skills) | Gstack | Pstack | Matt's Skills |
|---|---|---|---|---|
| **Led with philosophy?** | No -- jumped straight to tech stack | Yes -- "Boil the Ocean" framing | Yes -- pushed back on the premise itself | Yes -- "the process chain IS the architecture" |
| **Questioned the requirements?** | No -- assumed features (auth, priorities, drag-and-drop) | Partially -- mentioned presenting decisions as "AskUserQuestion" | Yes -- "the question is backwards, plan after the design settles" | Yes -- listed 6 unresolved branches (due dates? multi-user? what does 'complete' mean?) |
| **Domain modeling?** | Generic types file | Mentioned but didn't demonstrate | Discriminated unions, branded types, state machine | Full GLOSSARY.md with \_Avoid\_ synonyms, ADRs |
| **Project structure** | Monorepo (Turborepo) with apps/api + apps/web + packages/shared | Single app, src/ with routes/db/models/lib | Single app, domain/ + boundary/ separation | Deep modules: items/, lists/, filtering/ with narrow interfaces |
| **Testing approach** | Listed 3 frameworks, priority-ordered what to test first | "Tests are the cheapest lake to boil" -- 100% coverage, real SQLite in tests | Pure function tests (no mocks), real DB in integration, TDD for bugs | Red-green at pre-agreed seams, 3 anti-patterns to avoid, mocks only at system boundaries |
| **Git strategy** | Trunk-based, conventional commits, branch protection | Trunk-based, bisect commits, `/ship` workflow | Trunk-based, small PRs in dependency order | One branch per ticket (vertical slice), prototype branches preserved |
| **Code review** | Not mentioned | `/review` checks SQL safety, LLM trust boundaries | `/interrogate` multi-model adversarial review | Two-axis: standards (Fowler smells) + spec compliance, separate sub-agents |
| **PR structure** | Not mentioned | `/ship` handles version bump, changelog, PR creation | Conventional commits, ordered commits | Summary (visual), Evidence (before/after), Merge Danger (one-way/two-way door) |
| **Deployment** | Docker + Railway/Fly.io, full CI/CD YAML | Fly.io with LiteFS, full CI/CD YAML | Docker or Fly.io, `check.sh` as single source of truth | Vercel/Fly.io, minimal -- "no staging for a greenfield this size" |
| **Post-deploy?** | No | Yes -- `/canary` monitoring loop | No | No |
| **Retrospective?** | No | Yes -- `/retro` | No (mentioned `/reflect` in passing) | Yes -- `/retro` for environment improvements |
| **Database choice** | PostgreSQL + Prisma ORM | SQLite (Bun built-in), argued against Postgres | SQLite (better-sqlite3), no ORM | SQLite (Drizzle ORM or better-sqlite3) |
| **Auth included?** | Yes -- JWT + bcrypt, full auth module | Yes -- Better Auth or Lucia | No -- "build it when you need it" | Depends on grilling session |
| **Effort estimate** | "A weekend for an experienced developer" | 6 hours AI-assisted (with table comparing human vs AI) | Not given | Not given |
| **What they'd leave out** | Listed 5 things (GraphQL, Redis, microservices, SSR, feature flags) | Implicit in "search before building" | Listed 6 things with principle citations (no ORM, no auth, no monorepo, no state mgmt, no Docker Compose, no README) | Implicit in grilling -- only build what survives the interview |

---

## The Delta: What Changes When You Add a Skill Pack

### The Cold Model Response

A competent, generic senior engineer answer. Picks a standard stack (React + Express + Postgres + Prisma), scaffolds a thorough monorepo structure, and gives you working CI/CD YAML. It's the kind of answer you'd get from a good blog post or a senior dev's first-pass recommendation.

**What it does well:**
- Complete project structure with every file named
- Working code samples (Prisma schema, CI YAML, Docker Compose, bootstrap commands)
- Sensible architecture decisions (module-based backend, service layer, Zod validation)
- Practical "what to leave out" section

**What it skips entirely:**
- Never questions whether you actually need all this (auth? priorities? drag-and-drop?)
- No domain modeling -- just a generic `types.ts`
- No code review process
- No PR structure
- No post-deploy strategy
- No retrospective or learning loop
- No discussion of testing philosophy beyond "use these frameworks"
- Assumes the most complex reasonable stack (Postgres, monorepo, full auth) without justification

### What Each Skill Pack Added

**Gstack added:** A philosophy ("boil the ocean"), a workflow chain (spec -> autoplan -> review -> qa -> ship -> land-and-deploy -> canary), cross-session memory, post-deploy monitoring, and the argument that SQLite beats Postgres for this use case. It also added an effort comparison table (3 weeks human vs 6 hours AI-assisted) -- the most sales-oriented response.

**Pstack added:** Pushback on the premise ("the question is backwards"), rigorous type modeling (discriminated unions, branded types, state machine), a clean domain/boundary architecture, a principle citation table mapping every decision to a named principle, and the most aggressive "leave things out" list. The most opinionated about code quality and the least about process ceremony.

**Matt's Skills added:** A complete process chain (grill -> spec -> tickets -> implement -> review -> pr -> retro), a GLOSSARY.md with \_Avoid\_ synonyms, vertical-slice ticket decomposition with blocking edges, deep module architecture, two-axis code review, and the most sophisticated PR structure (summary as visual, evidence as before/after, merge danger as one-way/two-way door). The most process-oriented response.

---

## The Three Biggest Differences

### 1. Requirements Discovery

The cold model **assumed** features. It gave you auth, priorities, drag-and-drop, and a full monorepo without asking if you needed any of it. All three skill packs either questioned the requirements (Matt's, Pstack) or structured a process for resolving them (Gstack's `/spec` and `/autoplan`).

This is the single biggest delta. A raw model builds what it guesses you want. A skill-informed model asks what you actually need.

### 2. Testing Philosophy vs. Testing Tooling

The cold model listed testing frameworks and told you what to test first. The skill-informed responses described **how to think about testing**: pre-agreed seams (Matt's), test behavior not implementation (Pstack), 100% coverage because "tests are the cheapest lake to boil" (Gstack). The cold model gave you tools. The skill packs gave you principles.

### 3. The Lifecycle Beyond Code

The cold model's lifecycle was: scaffold -> implement -> CI -> deploy. Done.

Every skill pack extended this:
- **Before code:** Grilling/interviewing, spec writing, ticket decomposition
- **During code:** TDD loops, code review as a defined process
- **After code:** PR structure, post-deploy monitoring, retrospectives

The raw model treats software development as "write code and ship it." The skill packs treat it as a discipline with phases before and after the code.

---

## Verdict

The skill files work. The proof is not that the skill-informed responses are longer or more detailed -- the cold response was already thorough. The proof is that they are **differently shaped**:

1. They question requirements instead of assuming them
2. They model the domain before picking a framework
3. They define testing philosophy, not just testing tools
4. They include code review, PR structure, and post-deploy monitoring
5. They cite principles for their decisions instead of defaulting to convention

The markdown is not the magic. The discipline encoded in the markdown is the magic. A raw model knows all of this -- it was trained on the same engineering literature. But without the skill file, it defaults to "here's a standard stack, go build." The skill file says: "wait. Think about this differently."

That's the delta. That's the story.
