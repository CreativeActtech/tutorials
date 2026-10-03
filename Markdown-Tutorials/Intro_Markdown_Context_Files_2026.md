# Introduction to Contextual Markdown for LLMs and Agents 2026

A production guide to writing **contextual documents** — persistent Markdown files that teach LLMs and AI agents how to work with your codebase, docs, and website. Covers `AGENTS.md`, `llms.txt` (v2), `CLAUDE.md`, Copilot instructions, Cursor rules, Windsurf rules, and more.

## Table of Contents

1. [What Are Contextual Documents?](#1-what-are-contextual-documents)
2. [The Agent Context File Ecosystem](#2-the-agent-context-file-ecosystem)
3. [Why Markdown Wins for Agent Context](#3-why-markdown-wins-for-agent-context)
4. [AGENTS.md: The Open Standard](#4-agentsmd-the-open-standard)
5. [llms.txt and llms-full.txt: Making Websites Agent-Readable](#5-llmstxt-and-llms-fulltxt-making-websites-agent-readable)
6. [Tool-Specific Context Files](#6-tool-specific-context-files)
7. [Markdown Techniques for Agent-Facing Docs](#7-markdown-techniques-for-agent-facing-docs)
8. [Multi-File Strategy: Nesting, Splitting, Imports](#8-multi-file-strategy-nesting-splitting-imports)
9. [Anti-Patterns to Avoid](#9-anti-patterns-to-avoid)
10. [Testing and Maintenance](#10-testing-and-maintenance)
11. [Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)
12. [Practice Exercises](#12-practice-exercises)

---

## 1. What Are Contextual Documents?

A **contextual document** is a Markdown file that lives in your repository or on your website and is automatically loaded by an LLM or agent as standing instructions. Instead of repeating yourself in every prompt, you write the rules once:

- How to build and test the project
- Coding conventions and style rules
- Architecture decisions and gotchas
- Hard constraints (security, secrets, things never to touch)
- Where to find key resources

> [!NOTE]
> Think of prompts as *ephemeral* instructions and contextual documents as *persistent* instructions. Prompt engineering gets a task done once; context engineering shapes every future task.

**The two main audiences:**

| Audience | Reads files from | Goal |
| :--- | :--- | :--- |
| **Coding agents** (Codex, Claude Code, Copilot, Cursor, Windsurf, Gemini CLI) | Your repository | Follow your conventions while writing code |
| **General LLMs / RAG crawlers** | Your website root | Understand and cite your product or docs accurately |

---

## 2. The Agent Context File Ecosystem

Different tools read different filenames. Here is the 2026 landscape:

| File | Tool(s) | Scope | Notes |
| :--- | :--- | :--- | :--- |
| `AGENTS.md` | OpenAI Codex, Cursor, Google Jules, Amp, Zed, and many others | Repo + subdirectories | The emerging open standard; stewarded by the Agentic AI Foundation. |
| `CLAUDE.md` | Claude Code | Repo root + subdirectories | Supports `@path/to/import` syntax for file imports; generated via `/init`. |
| `.github/copilot-instructions.md` | GitHub Copilot | Whole repo | Official Copilot location. |
| `.github/instructions/*.instructions.md` | GitHub Copilot | File-pattern scoped | Uses `applyTo` globs in YAML frontmatter. |
| `.cursor/rules/*.mdc` | Cursor | Glob-scoped or global | Markdown + YAML frontmatter. |
| `.cursorrules` | Cursor (legacy) | Whole repo | Still read, but superseded by the `.mdc` rules system. |
| `.windsurf/rules/*.md` | Windsurf | Trigger-scoped | Uses frontmatter triggers (`always_on`, `glob`, `model_decision`, `manual`). |
| `GEMINI.md` | Gemini CLI | Repo root | Mirrors `CLAUDE.md` conventions. |
| `llms.txt` | Crawlers, LLMs, RAG pipelines | Website root | LLM-friendly site map (updated to v2 spec in Aug 2026). |
| `llms-full.txt` | Crawlers, LLMs, RAG pipelines | Website root | Full docs, inline. |

> [!IMPORTANT]
> **Convergence trend:** By 2026, several tools that originally required their own file also read `AGENTS.md`. If you can only maintain one file, make it `AGENTS.md` and use import wrappers (like `@AGENTS.md` in `CLAUDE.md`) for the others.

---

## 3. Why Markdown Wins for Agent Context

Everything from the companion prompt guide applies, but three properties matter even more here:

1. **Token budget is permanent.** Context files are injected into *every* agent session. A bloated file taxes every request. Markdown's compact syntax keeps overhead minimal — recall that Markdown can use ~70% fewer tokens than HTML and ~15% fewer than JSON.
2. **Headings are retrieval anchors.** Agents scan and grep for sections like `## Testing` before running commands. Predictable headings make agents faster and cheaper.
3. **Code fences are executable truth.** Agents copy commands directly out of fenced blocks. A wrong or missing fence means failed builds.

> [!CAUTION]
> These files are trusted input to autonomous agents. Never include secrets, credentials, or internal URLs in them. They are frequently committed publicly.

---

## 4. AGENTS.md: The Open Standard

`AGENTS.md` is a simple, open format: one Markdown file per directory, plain text, no required frontmatter. Agents read the file nearest to the files they are working on, walking up the tree when needed.

### 4.1 Recommended Sections

| Section | Purpose |
| :--- | :--- |
| Project overview | One paragraph: what this is, stack, entry points |
| Setup commands | Exact commands to install and build |
| Code style | Language, formatter, naming rules |
| Testing instructions | How to run the full suite and a single test |
| Debugging | Logs, common errors, how to reproduce |
| PR / commit instructions | Commit format, branch naming, checklist |
| Boundaries | Files and actions the agent must never touch |

### 4.2 Full Production Template

````markdown
# AGENTS.md

## Project Overview
Event-streaming API in Go 1.22. Entry point: `cmd/server/main.go`.
PostgreSQL 16 via `pgx`. No ORMs.

## Setup Commands
```bash
make setup        # install tools, run migrations
make dev          # start local server on :8080
```

## Code Style
- Run `gofmt` and `golangci-lint run` before committing.
- Errors: wrap with `fmt.Errorf("...: %w", err)`.
- No `init()` functions. No global state outside `internal/config`.

## Testing
```bash
make test                    # full suite
go test ./internal/orders/ -run TestCheckout -v   # single test
```
Tests use `testify`. Table-driven tests only for handlers.

## Boundaries
> [!CAUTION]
> Never modify `migrations/` files that already shipped.
> Never read from `.env*` files directly; use `internal/config`.

> [!IMPORTANT]
> All new endpoints require an auth middleware test.

## PR Instructions
- Commits: imperative mood, ≤72 chars ("Add checkout retry", not "added stuff").
- Keep PRs under 400 changed lines; split larger work.

<!-- AGENTS.md v1.3.0 | Owner: @platform-team -->
````

### 4.3 What Makes It Work

1. **Imperative voice.** "Run `gofmt`" beats "you might want to run gofmt".
2. **Copy-pasteable fences.** Every command is in a fenced block with `bash` tagged.
3. **Negative space defined.** The `Boundaries` section prevents the most expensive agent mistakes.
4. **Short.** This file is ~200 tokens. Agents read it on every session — brevity is a feature.

---

## 5. llms.txt and llms-full.txt: Making Websites Agent-Readable

`llms.txt` is a standardized Markdown file at the **root of your domain** that tells LLMs what your site is and where the important content lives. It exists because `robots.txt` was built for search crawlers, not language models.

### 5.1 The Format

The spec is deliberately minimal:

1. A level-1 heading: **the site name**.
2. A blockquote: **a one-paragraph description**.
3. Zero or more `##` sections, each containing a **list of Markdown links**.
4. A section named **`## Optional`** is treated as skippable when context is tight.

### 5.2 Template

```markdown
# Acme Stream

> Acme Stream is an event-streaming platform for real-time data pipelines.
> These docs cover the API, SDKs, and self-hosted deployment.

## Docs

- [Quickstart](https://docs.acme.dev/quickstart): From zero to first event in 10 minutes
- [API Reference](https://docs.acme.dev/api): REST and WebSocket endpoints
- [Go SDK](https://docs.acme.dev/sdk/go)

## Blog

- [Exactly-Once Semantics Explained](https://acme.dev/blog/exactly-once)

## Optional

- [Changelog](https://docs.acme.dev/changelog)
- [Legacy v1 API](https://docs.acme.dev/api/v1)
```

> [!TIP]
> **The v2 Spec Update (August 2026):** The v2 update formalized standard link relations (`rel="alternate"`) so AI agents can easily find Markdown versions of your HTML pages and your `llms.txt` file directly from HTTP headers.

### 5.3 llms-full.txt

`llms-full.txt` puts the **actual documentation content** inline rather than linking out — designed to fit inside a single model context window.

| File | Contains | Best for |
| :--- | :--- | :--- |
| `llms.txt` | Links + descriptions | Discovery, RAG seeding, chat citations |
| `llms-full.txt` | Full doc text, concatenated | One-shot ingestion, offline fine-tuning context |

> [!TIP]
> Serve these as `text/markdown` with a long cache TTL. Strip navigation chrome, navbars, and JavaScript — pure Markdown content only.

---

## 6. Tool-Specific Context Files

### 6.1 CLAUDE.md (Claude Code)

Loaded automatically at session start. Supports nesting (files in subdirectories apply to work there) and `@path/to/file` imports.

```markdown
# CLAUDE.md

## Commands
- `pnpm test` — full suite
- `pnpm test -- --watch path/to/file.test.ts` — single test

## Architecture
@docs/architecture.md

## Conventions
- All API responses go through `src/lib/respond.ts`
- Never import from `src/internal/` outside that directory

<!-- .claude/settings.json controls permissions -->
```

> [!NOTE]
> Claude Code's `/init` command generates a solid first draft of `CLAUDE.md` from your repo. Review and trim it before committing.

### 6.2 GitHub Copilot

Repository-wide instructions live in `.github/copilot-instructions.md`. For file-pattern-scoped rules, use `.github/instructions/<name>.instructions.md` with frontmatter:

````markdown
---
applyTo: "**/*.test.ts"
---

# Test File Rules
- Use `vitest`, never `jest`.
- Every handler test includes an auth-failure case.
````

### 6.3 Cursor Rules

Modern Cursor uses `.cursor/rules/*.mdc` — Markdown with required YAML frontmatter:

````markdown
---
description: TypeScript and React conventions
globs: ["src/**/*.ts", "src/**/*.tsx"]
alwaysApply: false
---

# React Rules
- Function components only; no class components.
- Colocate styles with `.module.css`.
- Prefer `useQuery` over manual `useEffect` fetches.
````

Frontmatter keys: `description` (when the rule is relevant), `globs` (which files trigger it), `alwaysApply` (force into every context).

### 6.4 Windsurf Rules

````markdown
---
trigger: model_decision
globs: ["*.py"]
description: Python conventions for the data pipeline
---

# Python Rules
- Type hints required on public functions.
- Use `pathlib`, not `os.path`.
````

Trigger types: `always_on`, `glob`, `model_decision` (agent decides based on `description`), and `manual`.

### 6.5 Gemini CLI

Uses `GEMINI.md` at the repo root with the same content strategy as `CLAUDE.md`. A practical pattern:

```markdown
# GEMINI.md
@AGENTS.md
```

---

## 7. Markdown Techniques for Agent-Facing Docs

### 7.1 Optimize for Scanning, Not Reading

Agents locate sections by heading text. Use a **stable, predictable vocabulary**:

```markdown
## Setup Commands
## Code Style
## Testing
## Boundaries
```

> [!IMPORTANT]
> Never rename headings casually. Agents, cached prompts, and team habits all key off the exact heading text.

### 7.2 Imperative Tables for Rules

Rules compress beautifully into two-column tables:

```markdown
| Do | Don't |
| :--- | :--- |
| `fmt.Errorf("...: %w", err)` | `errors.New` for wrapped errors |
| Table-driven tests | One giant test function |
| `slices.Contains` | Hand-rolled loops for membership |
```

### 7.3 Fences Are Contracts

- Always tag the language: ` ```bash `, ` ```go `, ` ```json `.
- One command per fence when agents will execute them.
- Show **expected output** in a separate fence when it helps verification:

````markdown
```bash
make test
```

Expected tail of output:

```text
ok   github.com/acme/api/internal/orders   2.41s
```
````

### 7.4 Alerts for Hard Constraints Only

Reserve `> [!CAUTION]` and `> [!IMPORTANT]` for rules whose violation costs real money or incidents. If everything is a warning, nothing is.

### 7.5 No Images, Few Links

Agents do not render images and often cannot fetch links at edit time. Describe diagrams in text or ASCII. Keep links to things that matter; every dead link is noise.

### 7.6 Paths, Not Prose

```markdown
# Bad
The database code is in the folder inside the internal directory.

# Good
DB access layer: `internal/db/`. Migrations: `migrations/`.
```

---

## 8. Multi-File Strategy: Nesting, Splitting, Imports

A single root file does not scale to a monorepo. Use locality:

```text
repo/
├── AGENTS.md                    # global: setup, style, boundaries
├── frontend/
│   └── AGENTS.md                # React rules, component conventions
├── services/
│   ├── billing/
│   │   └── AGENTS.md            # billing invariants, ledger rules
│   └── emails/
│       └── AGENTS.md
└── docs/
    └── llms/
        ├── llms.txt
        └── llms-full.txt
```

**Rules of thumb:**

1. Root file = global contracts (formatter, test runner, boundaries).
2. Directory files = local contracts (domain invariants, module APIs).
3. Keep each file under ~300 lines; split before that.
4. Where imports exist (`CLAUDE.md` `@docs/x.md`, Cursor rule files), reference — do not duplicate.

> [!TIP]
> Token math: a 2,000-token context file loaded into 50 agent sessions a day is 100,000 tokens of standing cost daily. Trimming it 30% pays for itself immediately.

---

## 9. Anti-Patterns to Avoid

| Anti-Pattern | Failure Mode | Fix |
| :--- | :--- | :--- |
| Copy-pasting the whole README | Bloat, stale duplication | Write agent-specific content; link the README |
| Secrets or internal URLs | Leak via public repos/logs | Audit with a secret scanner; use placeholders |
| Vague advice ("write clean code") | Agent ignores it | Concrete, checkable rules with examples |
| Walls of prose | Agent misses key rules | Headings, tables, fences |
| 10+ alerts in one file | Warning fatigue | Max 2–3 `CAUTION`/`IMPORTANT` per file |
| Untagged code fences | Mis-parsed, wrong syntax highlighting | Always tag: ` ```bash `, ` ```json ` |
| Images for architecture | Invisible to agents | Text/ASCII descriptions |
| Never updating the file | Drift, wrong commands | Version footer + review on CI changes |

---

## 10. Testing and Maintenance

Context files are code. Treat them like code:

1. **Version footer.** `<!-- AGENTS.md v1.3.0 | Owner: @team -->`
2. **CI smoke test.** Lint that every fenced `bash` block in `AGENTS.md` runs on a fresh runner.
3. **Agent dry-runs.** Monthly: give a fresh agent session a task ("add a health endpoint") and check it follows the file without prompting.
4. **Review together.** Change `Makefile`? The same PR updates `AGENTS.md`.
5. **Track drift.** If agents keep making the same mistake twice, the fix is a line in the context file — not another prompt.

> [!IMPORTANT]
> The best debugging question for agent misbehavior: "Which rule is missing from the context file?" Add it once, and it is fixed for every future session.

---

## 11. Quick Reference Cheat Sheet

````markdown
# AGENTS.md skeleton

## Project Overview
One paragraph: what, stack, entry point.

## Setup Commands
```bash
make setup && make dev
```

## Code Style
| Do | Don't |
| :--- | :--- |
| Formatter X | Hand-formatting |

## Testing
```bash
make test
```

## Boundaries
> [!CAUTION]
> Never touch: migrations/ (shipped), .env*

<!-- AGENTS.md vX.Y.Z | Owner: @who -->
````

````markdown
# llms.txt skeleton

# Site Name

> One-paragraph description.

## Docs
- [Page](https://example.com/page): description

## Optional
- [Changelog](https://example.com/changelog)
````

---

## 12. Practice Exercises

### Exercise 1: Write an AGENTS.md

Pick a repo you maintain. Write a 200-token `AGENTS.md` covering: overview, setup, testing, and one `> [!CAUTION]` boundary.

### Exercise 2: Publish llms.txt

Draft an `llms.txt` for your product docs with at least three sections and one `Optional` section. Validate that every link resolves. Add the v2 link relations to your HTTP headers.

### Exercise 3: Split a Giant File

Take any context file over 300 lines and split it into a root file plus two directory-scoped files. Verify an agent session in a subdirectory still gets the local rules.

---

## Next Steps

1. **Adopt one standard first:** `AGENTS.md` + symlinks/copies for your tools.
2. **Add `llms.txt`** if you maintain public docs.
3. **Level up:** context-file evaluation harnesses, MCP resource servers, and RAG pipelines that ingest `llms-full.txt` automatically.

---

**Author:** CreativeAct
**License:** MIT
**Version:** 1.1.0

<!-- Contextual Markdown Guide v1.1.0 | 2026-10-04 | Author: CreativeAct | License: MIT -->