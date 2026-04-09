# Project Discovery to Spec

A Claude Code skill that transforms raw client requirements into implementation-ready documentation for any web project type.

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
10. **Coding standards** — architecture rules, security headers, CORS, testing, deployment
11. **Client-facing documents** — non-technical project description + time estimate in client's language
12. **Final review** — cross-reference audit, blind spot analysis, consistency check

## Installation

### Claude Code

Add to your project's `.claude/settings.json` or install as a personal skill:

```bash
# Personal skill (available in all projects)
claude skill install github:BabicM/claude-new-php-project
```

Or clone manually:

```bash
git clone https://github.com/BabicM/claude-new-php-project.git ~/.claude/skills/project-discovery-to-spec
```

### Manual usage

The skill files are plain Markdown — readable and usable by any AI assistant or as a human checklist. Start with `SKILL.md`.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill — classification, 29 discovery domains, 7 phases, quick start recipes |
| `coding-standards.md` | Universal architecture rules, security, testing, deployment checklist |
| `edge-cases.md` | Type-specific edge case checklists for systematic review |

## How it works

```
Client input → Classify (T1/T2/T3) → Pick relevant domains → Ask questions
    → Use cases → Edge cases → Open questions → Tech recommendations
    → Notifications → Design → Time estimate → Coding standards
    → Client docs → Final review → Ready for implementation planning
```

The skill auto-scales: a T1 brochure site skips 80% of the process. A T3 marketplace gets the full treatment.

## Key design decisions

- **Technology-agnostic** — works with any stack (Laravel, Django, Rails, Next.js, etc.)
- **Region-agnostic** — no hardcoded services, carriers, or locale assumptions
- **Brainstorming gate** — if the client has a vague idea ("I want an app for X"), the skill redirects to brainstorming first, then returns for structured analysis
- **Open question tracking** — every ambiguity becomes a tracked OQ with options and recommendations, not a silent assumption

## License

MIT
