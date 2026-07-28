# Backend Engineering Vault (Go)

A personal Obsidian vault — a self-contained, linked-note learning path through backend engineering, from junior to senior level, with **Go** as the primary language.

This isn't reference documentation meant to be read once. It's built to be re-read, annotated, and argued with over years, until every note's `## Catatan Saya` section is full of real questions and real examples pulled from production.

## Who this is for

The vault is written for one specific reader: a backend engineer with roughly a year of production experience, also acting as technical coordinator for 10+ developers across 13+ government legal-services applications. Daily stack is **PHP (Yii1/Yii2) + MariaDB**, surrounded by **Go, Kafka, Elasticsearch, Kubernetes, Jenkins, Next.js**. It assumes working knowledge of programming fundamentals, HTTP, SQL, and Git — it does not explain what a variable is.

Three goals sit behind it:

1. Master backend engineering conceptually up to senior level — not just terminology, but *why* each concept exists and when it's the wrong choice.
2. Real mastery in Go — the runtime, the memory model, the idioms, not just handlers that compile.
3. Build the judgment to make and defend architectural decisions under real constraints.

Behind all three is a longer-term goal: a master's degree in distributed systems. `60 Distributed Systems/` is treated as a serious track, not a survey chapter.

## Language

Explanations are written in **Bahasa Indonesia**. Technical terms stay in **English** and are never translated — a translated term (`tingkat isolasi` instead of `isolation level`) is harder to recognize later in official docs, error messages, and interviews, not easier. See `00 Start Here/Vault Conventions.md` for the full list of terms kept in English and the grammar rules around mixing the two languages. File names, tags, frontmatter, and section headings are always in English so the vault stays queryable and the graph stays legible.

## How it's organized

```
00 Start Here/     → onboarding, reading method, conventions, progress tracker
01 Maps/            → cross-domain reading orders, one per level (junior/intermediate/senior)
02 Templates/       → raw note templates (concept / tool / case study)
03 My Zettels/       → reader's own atomic ideas only — never written by anyone else
10–90               → domain folders, numbered by distance from the machine:
                       Foundations → Go Language → APIs and Web → Databases →
                       Concurrency and Performance → Distributed Systems →
                       Infrastructure and Delivery → Security → Architecture and Design
92 Tools/            → concrete products (PostgreSQL, Kafka, Kubernetes, ...) — perishable
                       knowledge, deliberately separated from the durable concepts above
94 Case Studies/     → real decisions made under real constraints
95 Projects/         → one capstone project per level, all in Go
99 Glossary/         → one term per note, for quick lookup and backlinks
```

Folders are grouped by **domain**, not by level — level (`junior`/`intermediate`/`senior`) lives in each note's frontmatter instead. This is deliberate: a single topic (say, database indexing) can be read shallow-to-deep in one place, rather than being scattered across three disconnected level-only folders. The numeric prefix on domain folders is a reading-order hint, not a hard rule — `10` is closest to the machine, `90` is closest to the organization.

## How this vault gets built

Content is drafted following a detailed build spec (`CLAUDE.md`, checked into this repo) that defines the writing principles, note templates, frontmatter schema, and quality bar for every note — narrative prose over bullet lists, a concrete "why does this exist" problem before any mechanism, an analogy with its breaking point stated explicitly, idiomatic and compilable Go code, honest trade-offs, and open-ended design exercises. Notes are drafted with LLM assistance against that spec, one domain at a time, taken to full depth (junior → intermediate → senior) before moving to the next domain — then reviewed and lived-in by hand, which is what the empty `## Catatan Saya` section at the end of every note is for.

`00 Start Here/Progress Tracker.md` and `00 Start Here/Backlog.md` track what's actually written versus what's still planned; `00 Start Here/Vault Manifest.md` is the source of truth for the full planned shape of the vault.

## Using this vault

1. Open the folder in [Obsidian](https://obsidian.md).
2. Install the **Dataview** community plugin — several notes (progress tracker, domain overviews, level maps) use Dataview queries to show status tables.
3. Start at `00 Start Here/Read Me First.md`, then `00 Start Here/How To Read This Vault.md`, then `01 Maps/Level 1 - Junior Path.md`.

Full reading method (linear vs. graph-driven vs. problem-driven reading, the status lifecycle, review rhythm) is in `00 Start Here/How To Read This Vault.md`.

## Status

This vault is a work in progress, written incrementally rather than all at once. Check `00 Start Here/Progress Tracker.md` for current completion status per level and domain, and `00 Start Here/Backlog.md` for what's queued next.

## License

This is a personal learning artifact, published for transparency and in case it's useful to someone else on a similar path. Feel free to fork it or borrow the structure for your own vault — the content itself is written for one specific reader's context and background, so treat it as a reference rather than a drop-in curriculum.
