# New Laravel Project

A Claude Code skill for starting new Laravel projects — from client discovery through implementation-ready documentation, coding standards, and into development using the full superpowers skill suite.

## What it does

Systematic methodology for the gap between "client says what they want" and "developer starts coding." Scales from a simple landing page (15 questions, 3 deliverables) to a complex multi-role platform (100 questions, 13 deliverables).

### Supported project types

| Tier | Types |
|------|-------|
| **T1 Presentation** | Landing page, company site, portfolio, blog |
| **T2 Application** | E-shop, booking system, directory, LMS, portal |
| **T3 Platform** | Marketplace, SaaS, complex multi-role platform |

### What it produces

1. **Discovery questionnaire** — domain-mapped questions (29 domains, pick what's relevant)
2. **Use cases** — actor-based flows with cross-cutting matrices
3. **Edge cases** — type-specific checklists (cart abandonment, double booking, tenant isolation...)
4. **Open questions** — tracked ambiguities with options, recommendations, and priority
5. **Tech recommendations** — stack, infrastructure, packages with trade-offs
6. **Notification catalog** — every system email/notification: trigger, recipient, channel
7. **Design requirements** — brand assets checklist, page complexity, accessibility
8. **Time estimation** — by module, optimistic/realistic, dependency map
9. **Maintenance plan** — post-launch support, data retention, handover
10. **Coding standards** — Laravel architecture rules, security headers, CORS, testing, deployment
11. **Client-facing documents** — non-technical project description + time estimate in client's language
12. **Final review** — cross-reference audit, blind spot analysis, consistency check

## Full Project Lifecycle

This skill integrates with 12 superpowers skills + Laravel Boost to cover the entire project from idea to production:

```
DISCOVERY & PLANNING
  1. superpowers:brainstorming           → Shape vague idea (if needed)
  2. new-laravel-project (this skill)    → Phases 1-7 discovery
  3. superpowers:writing-plans           → Implementation plan from spec

PROJECT SETUP (once)
  4. Laravel Boost                       → composer require laravel/boost --dev
                                           Gives AI agents deep codebase context via MCP

IMPLEMENTATION (per task)
  5. superpowers:using-git-worktrees     → Isolate feature work
  6. superpowers:test-driven-development → Write test FIRST
  7. superpowers:systematic-debugging    → If bug or test failure
  8. superpowers:verification-before-completion → Verify before claiming done
  9. superpowers:requesting-code-review  → Review the work
  10. superpowers:receiving-code-review  → Process review feedback
  11. superpowers:finishing-a-development-branch → Merge / PR / cleanup

PARALLELIZATION (when tasks are independent)
  12. superpowers:subagent-driven-development
  13. superpowers:dispatching-parallel-agents
```

**No code without a plan. No code without a test.**

## Installation

### Claude Code

```bash
claude skill install github:BabicM/claude-new-laravel-project
```

Or clone manually:

```bash
git clone https://github.com/BabicM/claude-new-laravel-project.git ~/.claude/skills/new-laravel-project
```

### Prerequisites

This skill works best with the [superpowers plugin](https://github.com/anthropics/claude-code-plugins) installed, which provides the implementation skills referenced in the lifecycle above.

### Manual usage

The skill files are plain Markdown — readable and usable by any AI assistant or as a human checklist. Start with `SKILL.md`.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill — project classification, 29 discovery domains, 7 phases, superpowers lifecycle, quick start recipes |
| `coding-standards.md` | Laravel architecture rules, security (CORS, headers, CSP), rate limiting, file upload security, session management, API versioning, error handling, logging, backup & disaster recovery, deployment checklist, testing rules, 47 common mistakes. **Also works as a standalone skill.** |
| `edge-cases.md` | Type-specific edge case checklists: universal (9 categories) + E-shop, Booking, Directory, LMS, Marketplace, SaaS, Portal |

## How it works

```
Client input → Classify (T1/T2/T3) → Pick relevant domains → Ask questions
    → Use cases → Edge cases → Open questions → Tech recommendations
    → Notifications → Design → Time estimate → Coding standards
    → Client docs → Final review → Writing plans → Implementation
```

The skill auto-scales: a T1 brochure site skips 80% of the process. A T3 marketplace gets the full treatment.

## Coding Standards Highlights

The `coding-standards.md` file covers 20 architecture categories with 48 common mistakes. Key sections:

- **Code Architecture** — strict typing, services, validation, authorization, events, enums, named routes
- **Security** — CORS policy (7 rules), security headers (7 + CSP), rate limiting, file upload security, session management
- **Data** — migrations (schema only), factories/seeders, indexing, soft delete policy
- **Operations** — deployment checklist (6 stages), backup & disaster recovery (RTO/RPO), logging, environment management
- **AI-Assisted Development** — Laravel Boost as required MCP server for AI agent codebase context
- **Quality** — test pyramid, TDD, CI enforcement, accessibility (EU 2025 legal requirement)
- **API** — versioning strategy, error response consistency, breaking change policy

## Key design decisions

- **Laravel-focused** — coding standards, architecture rules, and recommendations tailored for Laravel ecosystem
- **Superpowers-integrated** — full project lifecycle from brainstorming through implementation with 13 invocation points (12 skills + Laravel Boost)
- **Brainstorming gate** — if the client has a vague idea ("I want an app for X"), the skill redirects to brainstorming first, then returns for structured analysis
- **Open question tracking** — every ambiguity becomes a tracked OQ with options and recommendations, not a silent assumption
- **No code without a plan, no code without a test** — enforced via superpowers:writing-plans and superpowers:test-driven-development

## License

MIT
