# Coding Standards & Architecture Rules

Universal rules for the coding standards deliverable (Phase 5). Referenced from SKILL.md.

**Applies to:** All custom-built T2-T3 projects, regardless of tech stack.

---

## Code Architecture (decide before writing code)

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Strict typing** | Enable strictest type checking the language supports. | Catches bugs at analysis time, not production. PHP: `declare(strict_types=1)`. TypeScript: `strict: true`. Python: mypy strict. |
| **Business logic location** | Service classes — group related operations by domain. Controllers call services, services call models/events. | Scattered logic (in controllers, middleware, closures) = untestable, undiscoverable. Single-method "Action" classes fragment related logic across too many files. |
| **Input validation** | Dedicated validation layer, separate from controllers. | Reusable across endpoints, testable independently, keeps controllers thin. |
| **Authorization** | Policy/permission classes — centralized, auditable. | `if (user.role === 'admin')` scattered in controllers = security holes when you miss one. |
| **Side effects** | Event-driven — decouple secondary effects (email, logging, notifications) from core logic. | Queueable independently, testable in isolation, easy to add/remove without touching core logic. User registered → fire event → listeners handle email, log, notification separately. |
| **Mass assignment protection** | Explicitly whitelist assignable fields on every model/entity. Never allow-all. | One missing guard on a sensitive field = security vulnerability. |
| **N+1 query prevention** | Enable lazy-loading detection/exception in development. | N+1 is the #1 silent performance killer in ORMs. A page with 50 items makes 51 queries instead of 2. Detection in dev catches it before production. |
| **Enums for fixed value sets** | Use language-level enum types for statuses, types, categories — not magic strings, not integer constants, not database-only values. If multiple enums share behavior (labels, colors, serialization), define a base/abstract enum or interface once. | Magic strings (`'active'`, `'pending'`) scatter across codebase, typos cause silent bugs, IDE can't autocomplete. Enums are type-safe, refactorable, and self-documenting. |
| **Traits/mixins only for shared behavior** | Use traits (PHP), mixins (JS/Python), or composition only when multiple unrelated classes genuinely need the same behavior. Never as code organization ("let me move these methods to a trait to make the class shorter"). | Traits used for organization = hidden dependencies, unclear interfaces, method name collisions, untestable. If only one class uses it, it's not shared behavior — keep it in the class. If you need to reduce class size, extract a service instead. |
| **Named routes used everywhere** | Every route must have a name. All code (controllers, templates, tests, redirects, API responses) must reference routes by name, never by hardcoded URL path. | Hardcoded `/admin/users/5/edit` breaks when URL structure changes. Named routes (`route('admin.users.edit', $user)`) update automatically. Tests with hardcoded URLs pass even when the route is broken. Named routes are the single source of truth for URL generation. |
| **Clean namespace/module structure** | Namespaces must mirror directory structure, follow a consistent convention, and group by domain/feature — not by technical layer alone. No deep nesting without reason, no orphaned classes, no "Utils" or "Helpers" dumping grounds. | Messy namespaces = developers can't find things. `App\Services\Admin\User\Profile\UpdateService` is too deep. `App\Services\UserProfileService` is findable. Organize by what the code DOES, not by abstract technical taxonomy. |

---

## Date, Time & Timezone

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Store in UTC** | All timestamps in the database must be stored in UTC. No exceptions. Convert to user's timezone only at display time (frontend). | Mixing timezones in the database = impossible to compare, sort, or aggregate dates. Daylight saving transitions create phantom duplicates or gaps. UTC is the only sane storage format. |
| **Database timezone config** | Verify the database server timezone is set to UTC. Verify the application's DB connection timezone is UTC. Test this explicitly — don't assume. | A database defaulting to a local timezone (e.g., `US/Eastern`, `Europe/Berlin`) silently converts your UTC timestamps. You discover this 6 months in when reports show wrong times. Check `SHOW timezone;` (PostgreSQL) or `SELECT @@global.time_zone;` (MySQL) during setup. |
| **Frontend display** | Convert UTC → user's local timezone in the frontend, not the backend. Use the browser's `Intl.DateTimeFormat` or a library. Always show the timezone to avoid ambiguity. | Backend doesn't reliably know the user's timezone. The browser does. Displaying "14:00" without timezone context is ambiguous — "14:00 CET" is not. |
| **Scheduled content & events** | For content with scheduled publish times, booking times, event times — decide: stored in which timezone? Displayed in which timezone? What happens during DST transitions? | A booking at "14:00 on March 30" during spring-forward: does it shift to 15:00? Does it stay at 14:00? A "daily 02:00" cron job during DST: does it run twice, or skip? Define this per feature. |
| **Date formats per locale** | Define date display format per language (DD.MM.YYYY for SK/CZ, MM/DD/YYYY for US, YYYY-MM-DD for ISO). Use locale-aware formatting, never hardcode format strings. | "03/04/2026" is March 4th or April 3rd depending on who reads it. Locale-aware formatting eliminates ambiguity. |

---

## Data & Migrations

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Schema vs data migrations** | Migration files = schema changes ONLY (create table, add column, add index). Never put data manipulation (insert/update/delete rows) in migrations. | Data migrations break rollback, fail across environments, mix schema with business logic. Use seeders for initial data, CLI commands for one-time fixes, jobs for bulk transformations. |
| **Test data** | Factories + seeders for every entity. Realistic dev data. Non-negotiable. | Without factories: tests are painful, onboarding new devs is slow, demo environments are empty. Every model/entity must have a factory from day 1. |
| **Database choice** | Deliberate decision — evaluate data shape, query patterns, scale. Document WHY. | "We always use X" is not a decision. PostgreSQL for complex queries/fulltext/JSON. MySQL for simpler CRUD. SQLite for embedded. The choice affects indexing, search, JSON support, and scaling options. |
| **Indexing strategy** | Index: all foreign keys, columns used in WHERE/ORDER BY frequently, unique constraints. Don't over-index (slows writes). Review slow query log monthly. Plan composite indexes for common multi-column query patterns. | Missing index on a foreign key → full table scan on every JOIN. Over-indexed table → every INSERT takes 5x longer. Neither is visible until production traffic hits. |
| **Soft delete policy** | Decide per entity: hard delete or soft delete? Soft-deleted records leak into queries if not handled — deleted users showing in search, deleted products in reports. Define: which entities get soft delete, default query scope excluding deleted, cascade rules (delete parent → children?), purge schedule (permanently remove after 90 days). | "We soft delete everything" → database grows forever, queries slow down, GDPR compliance complicated ("right to be forgotten" vs "soft deleted"). Decide deliberately per entity. |

---

## Internationalization

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Translations** | Every UI-visible string through the translation system. Check after every change: new strings in all language files, no broken keys, no hardcoded strings in templates. | Missing translations = broken UI in other languages. Hardcoded strings are invisible until a client opens the Czech version. Enforce via CI lint if possible. |

---

## Async & Background Processing

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Queue strategy** | Define upfront: what runs async (emails, PDFs, image processing, imports/exports, notifications), retry policy (attempts, backoff), failed job handling (log, alert, retry), timeout per job type. | Deciding "later" = emails sent synchronously blocking requests, PDF generation timing out, image processing crashing the web server. Include failed job monitoring in alerting. |

---

## Domain & Routing

**This is the FIRST architectural decision. Make it before writing any route.**

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Domain structure** | Single domain, subdomains (`admin.`, `api.`, `app.`), separate domains, locale domains (`site.sk`, `site.cz`), or tenant custom domains. | Affects: route organization, middleware groups, session/cookie scope, CORS configuration, SSL certificates (wildcard vs per-domain), asset URLs, trusted proxies. Changing after launch = rewriting routes, breaking sessions, breaking assets. |

---

## CORS Policy

**Configure day 1. Misconfigured CORS = broken API calls or security holes.**

| Rule | Detail |
|------|--------|
| **Never `*` origins in production** | List exact domains (frontend app, mobile app, partner integrations). Wildcard is only acceptable for truly public read-only APIs with no credentials. |
| **Credentials + wildcard = silent failure** | Browsers silently reject `Access-Control-Allow-Origin: *` when the request includes cookies/auth headers. No error in console, just broken requests. |
| **Restrict methods per route group** | Public API: GET only. Authenticated API: GET, POST, PUT, DELETE. Admin: full CRUD. Don't allow all methods globally. |
| **Restrict headers** | Allow only what's needed: `Authorization`, `Content-Type`, `Accept`, `X-Requested-With`. Don't allow all. |
| **Per-domain CORS** | If you have subdomains (`api.app.com` called from `app.com`), configure CORS per route group, not one global policy. |
| **Preflight caching** | Set `max-age` to 86400 (24h) to reduce OPTIONS preflight requests. |
| **Exposed headers** | If frontend needs custom response headers (pagination info, rate limit remaining), explicitly expose them. |

---

## Security Headers

**Configure as middleware, applied globally. Day 1, not "before launch".**

| Header | Recommended Value | Purpose |
|--------|-------------------|---------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` | Forces HTTPS for all future visits. Include subdomains. Submit to browser preload list for maximum protection. **Caution:** test HTTPS thoroughly first — HSTS is hard to undo. |
| `X-Content-Type-Options` | `nosniff` | Prevents browsers from MIME-type sniffing (executing a disguised .js as text/html). Simple, no downside. |
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` | Prevents clickjacking (embedding your site in attacker's iframe). Use `SAMEORIGIN` only if you embed your own pages in iframes. |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Controls referrer info sent to other sites. Balances analytics needs vs user privacy. |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` | Disables browser features you don't use. Add back only what's needed (e.g., `camera=(self)` for photo capture, `geolocation=(self)` for map features). |
| `X-XSS-Protection` | `0` | Set to 0. Old browser XSS filters are deprecated and can cause more harm than good. Rely on CSP instead. |
| `Content-Security-Policy` | **Project-specific — plan carefully** | See CSP section below. |

### Content Security Policy (CSP)

The most complex header. Requires per-project planning:

- Define allowed sources for: scripts, styles, images, fonts, frames, connect (API calls), media
- Start strict (`default-src 'self'`), loosen only what's needed
- **Common pitfalls:** inline scripts (required by many JS frameworks), Google Analytics/GTM, payment scripts (Stripe.js), CDN-hosted fonts/icons, embedded maps, social widgets
- Use `nonce` or `hash` for inline scripts instead of `'unsafe-inline'`
- **Test with `Content-Security-Policy-Report-Only` first** — logs violations without breaking anything. Fix all violations, then switch to enforcing mode.
- CSP breaks things silently in production if you forget a third-party domain. Always test the full site after enabling.

### Common Security Traps

- CORS `*` with `credentials: true` → browser rejects silently, no error visible
- Missing CSP → any injected script runs freely (XSS)
- HSTS enabled without testing → locked out if HTTPS setup breaks
- Forgot third-party in CSP → broken payments/analytics/maps in production
- Security headers added "before launch" → never actually added

---

## Error Handling

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Custom exception classes** | Define domain-specific exceptions (`InsufficientStockException`, `BookingConflictException`, `PaymentFailedException`) — not generic "something went wrong". Group by domain. | Catchable at the right level, loggable with context, translatable for users. Generic exceptions hide what actually failed and make debugging production issues a guessing game. |
| **Global error handler** | One place that catches all unhandled exceptions. Logs with full context (user, request, stack trace). Returns appropriate response based on request type. | API request → JSON error `{message, errors, code}`. Web request → custom error page. Never expose stack traces, SQL queries, or file paths in production. |
| **Error pages** | Custom 404, 403, 500, 503 pages matching the design. | Users hitting an error should stay in your brand. Framework default error pages signal "amateur" to users and leak technical info to attackers. |
| **Error response consistency** | If any API: define error format once, use everywhere. Include: HTTP status code, machine-readable error code, human-readable message, validation errors (field-level). | Inconsistent error responses (sometimes `{error: "..."}`, sometimes `{message: "..."}`, sometimes plain text) = frontend can't handle errors reliably. |

---

## Logging & Observability

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Structured logging** | Use structured format (JSON) — not plain text. Every log entry: timestamp, level, message, context (user ID, request ID, relevant entity). | Plain text logs (`"User 5 failed to login"`) are impossible to search, filter, or aggregate at scale. Structured logs feed into log management tools and enable alerting. |
| **Log levels used correctly** | **ERROR:** something broke and needs attention (payment failed, service down). **WARNING:** something unexpected but handled (retry succeeded, fallback used). **INFO:** significant business events (user registered, order placed). **DEBUG:** development details, off in production. | If everything is ERROR, nothing is error. If payment failures are WARNING, they get ignored. Correct levels enable meaningful alerts — page on ERROR, review WARNING weekly, ignore INFO. |
| **Request/correlation ID** | Generate a unique ID per request. Pass through all services, include in all log entries and error responses. | User reports "something broke" → they give you the error ID → you find the exact request across all logs in seconds. Without it: "sometime around 2pm, someone got an error." |
| **Never log sensitive data** | Passwords, tokens, API keys, credit card numbers, national IDs, personal health data — NEVER in logs. Sanitize/mask before logging. | Logs are often less protected than the database. Logging a password "for debugging" = plain text credential in your log management tool, accessible to every developer. |

---

## Environment & Secrets

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **All config via environment variables** | Database credentials, API keys, app secrets, service URLs — all from environment, never hardcoded. | Hardcoded secrets in code = secrets in git history forever. Environment variables change per deployment without code changes. |
| **No secrets in repository** | `.env` in `.gitignore` from day 1. No API keys, passwords, or tokens anywhere in committed code. Scan for accidentally committed secrets in CI. | One committed AWS key = $50,000 crypto mining bill overnight. This happens regularly. Git history is permanent — force-push doesn't erase it from forks. |
| **`.env.example` always current** | Every time a new env variable is added, update `.env.example` with a placeholder and comment. | New developer clones repo, copies `.env.example`, app crashes because `STRIPE_KEY` isn't documented. Stale `.env.example` = broken onboarding. |
| **Per-environment configuration** | Define what differs: debug mode, log level, cache driver, mail driver (real vs captured), error reporting (verbose vs minimal). | Sending real emails from staging. Showing stack traces in production. Debug mode leaking database credentials. All caused by missing per-environment config. |
| **Secret rotation plan** | Define how to rotate API keys, encryption keys, JWT secrets without downtime. | If a secret leaks, you need to rotate in minutes, not figure out the process while panicking. |

---

## Git & Workflow Conventions

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Branch protection** | Main/master branch: no direct commits. All changes via PR. At least 1 review required. CI must pass before merge. | One bad commit to main without review → broken production. Branch protection is the safety net. |
| **Commit message convention** | Choose a format and enforce it. Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`) is widely used. Include ticket/issue reference. | Readable git history, auto-generated changelogs, bisecting bugs is faster when you can read what each commit intended. |
| **Branch naming** | Convention: `feature/`, `fix/`, `chore/` + ticket number + short description. Example: `feature/PROJ-42-user-registration`. | Consistent naming → readable PR lists, automatic linking to tickets, easy cleanup of stale branches. |
| **PR template** | Checklist: what changed, why, how to test, screenshots (if UI), migration notes, breaking changes. | Forces authors to explain their changes. Reviewers know what to look for. "Fix stuff" PRs get caught. |

---

## Dependency Management

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Lock files committed** | `composer.lock`, `package-lock.json`, `yarn.lock`, `Gemfile.lock` — always committed to git. | Lock files ensure every developer and every deployment uses identical dependency versions. Without them: "works on my machine" but breaks in production because a sub-dependency updated. |
| **Regular update schedule** | Define cadence: security updates immediately, minor updates monthly, major updates quarterly. Assign responsibility. | Ignoring updates for a year → 200 outdated packages, security vulnerabilities, impossible upgrade path. Regular small updates are painless; annual big-bang updates are projects. |
| **Security scanning in CI** | Run dependency vulnerability scanner in CI pipeline. Fail the build on known critical vulnerabilities. | Known vulnerabilities in published packages are the #1 attack vector for web apps. Automated scanning catches them before deployment. |
| **Version ranges, not exact pins** | Use version ranges (`^8.0`, `~3.2`) unless you have a specific reason to pin exact versions. | Exact pins prevent receiving security patches automatically. Ranges allow compatible updates. Pin only when a specific version has a known compatibility issue. |

---

## Database Design Rules

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Indexing strategy** | Index: all foreign keys, columns used in WHERE/ORDER BY frequently, unique constraints. Don't over-index (slows writes). Review slow query log monthly. Plan composite indexes for common multi-column query patterns. | Missing index on a foreign key → full table scan on every JOIN. Over-indexed table → every INSERT takes 5x longer. Neither is visible until production traffic hits. |
| **Soft delete policy** | Decide per entity: hard delete or soft delete? Soft-deleted records leak into queries if not handled — deleted users showing in search, deleted products in reports. Define: which entities get soft delete, default query scope excluding deleted, cascade rules (delete parent → children?), purge schedule (permanently remove after 90 days). | "We soft delete everything" → database grows forever, queries slow down, GDPR compliance complicated ("right to be forgotten" vs "soft deleted"). Decide deliberately per entity. |

---

## Accessibility

| Decision | What to define | Why it matters |
|----------|---------------|----------------|
| **Minimum standard** | Define target level: WCAG 2.1 AA minimum (EU legal requirement since 2025 — European Accessibility Act). | Not optional in the EU. Fines for non-compliance. Also: 15-20% of users have some form of disability. Inaccessible = excluding customers. |
| **Baseline requirements** | Semantic HTML (headings, landmarks, lists — not just `div` soup). Alt text on all images. Keyboard navigation for all interactive elements. Sufficient color contrast (4.5:1 minimum for text). Visible focus indicators. Form labels on every input. Error messages linked to fields. | These basics cover 80% of accessibility with minimal effort. They also improve SEO (semantic HTML), mobile usability (proper form labels), and code quality (structured markup). |
| **Testing approach** | Automated scan (axe, Lighthouse) in CI catches ~30% of issues. Manual keyboard-only navigation test for critical flows. Screen reader test (VoiceOver, NVDA) for key pages at minimum. | Automated tools miss context (alt text exists but says "image123.jpg") and interaction patterns (modal trap, focus management). Combine automated + manual for coverage. |

---

## Other Architecture Decisions

| Decision | When to include | What to decide |
|----------|----------------|----------------|
| **Cache strategy** | Public pages or heavy queries | What to cache: framework config, routes, views (deploy-time). Application cache: expensive queries, computed values (runtime). HTTP cache headers for public pages. **Always define invalidation rules.** |
| **API response format** | Any API endpoint | Dedicated response/resource classes — never expose raw database models (leaks fields, breaks when schema changes). Consistent error format: `{message, errors, code}`. |
| **Additional packages/tools** | Always discuss | List options with trade-offs in tech recommendations. Ask the team before adding. Don't auto-include packages the project doesn't need. |

---

## Deployment Checklist

**Every T2-T3 project needs an automated deployment process.** Specific commands depend on the stack — the stages are universal.

| Stage | What happens | Why |
|-------|-------------|-----|
| **1. Build assets** | Compile frontend (CSS, JS, images) | Production builds are optimized, minified, versioned |
| **2. Run migrations** | Apply pending database schema changes | Schema must match the code being deployed |
| **3. Cache/optimize** | Pre-cache configuration, routes, views, compiled assets | Eliminates runtime file reads, speeds up every request |
| **4. Restart workers** | Restart queue workers, websocket servers, schedulers | Workers run old code in memory until restarted |
| **5. Health check** | Hit a health endpoint to verify app is responding | Catches deployment failures before users do |
| **6. Verify scheduler** | Confirm cron/scheduler is running and processing | Scheduled jobs silently stop if cron entry is missing |

**Requirements:**
- All steps automated (no manual SSH + commands)
- Zero-downtime deployment (no maintenance mode for routine deploys)
- Rollback plan documented and tested
- Same pipeline for staging and production

---

## Testing & Quality Rules

1. **Static analysis first** — Use the strictest analysis tool for the chosen language. Run before tests in CI. Zero tolerance for baseline violations in new code.
2. **Test after every change** — Every new feature, bugfix, or refactor must include tests. No PR without tests. "We'll add tests later" = no tests.
3. **Test pyramid:**
   - **Unit tests** — Business logic, services, value objects. Fast, many.
   - **Feature/integration tests** — HTTP endpoints, database queries, queue jobs. Medium speed, cover critical paths.
   - **Browser/E2E tests** — Critical user flows only. Slow, few. Registration, checkout, search, core CRUD.
4. **Frontend changes = E2E tests** — If you change what the user sees in a browser, add or update a browser test for that flow.
5. **CI enforces everything** — Lint, analyze, test, format check. PR cannot merge if CI fails. No exceptions, no "skip CI" commits.
6. **Best practices for the stack** — Always follow official best practices for the chosen framework/platform. Use built-in features before third-party packages. Prefer first-party/official packages where available. Don't fight the framework — custom solutions for solved problems = tech debt.

---

## Common Coding Mistakes

| Mistake | Fix |
|---------|-----|
| No static analysis configured | Use the strictest tool for your language. Run in CI. Non-negotiable. |
| Frontend changes without E2E tests | If users see it in a browser, browser tests must cover the flow. |
| Tests written "later" | Tests accompany every change. "We'll add tests after" = no tests. |
| Ignoring framework best practices | Don't fight the framework. Use built-in features before packages. |
| No test data factories/seeders | Every entity needs a factory. Empty dev DB = slow development + untestable code. |
| Data manipulation in schema migrations | Migrations = schema only. Data fixes via seeders, commands, or jobs. |
| Hardcoded strings in templates | Every user-visible string through translation system. Check after every change. |
| Database chosen by default, not by need | "We always use X" is not a decision. Evaluate data shape, query needs, scale. |
| Auto-including packages without asking | Great when needed, bloat when not. List options with trade-offs, ask before adding. |
| Domain/routing structure decided "later" | Decide before first route. Affects sessions, cookies, CORS, SSL, assets. Retrofitting = rewrite. |
| No CORS policy defined | Never allow all origins in production. Wildcard + credentials = silent failure. |
| No security headers configured | HSTS, CSP, X-Frame-Options, nosniff — configure day 1. Missing = clickjacking, XSS, MIME attacks. |
| CSP added last minute | CSP breaks external scripts (analytics, payments, CDN). Plan per project. Test with report-only first. |
| Queue strategy decided "later" | Emails sent synchronously, PDFs blocking requests. Decide what's async during planning. |
| No deployment checklist | Build, migrate, cache, restart workers, health check — must be automated, not manual. |
| Business logic in controllers | Controllers should only receive input, call a service, return output. |
| Side effects chained inline | User created → email + log + notify in one method = untestable, unqueueable. Use events + listeners. |
| No mass assignment protection | Whitelist assignable fields on every entity. One missing guard = security vulnerability. |
| Manual deployments via SSH | "I'll just SSH in and pull" = human error, downtime, no rollback. Automate everything. |
| N+1 queries undetected | Enable lazy-loading detection in development. Silent N+1 = 50 queries where 2 would do. |
| No strict typing enabled | Enable the strictest type checking your language supports. Catches entire classes of bugs before runtime. |
| Magic strings instead of enums | Statuses, types, categories must be enums — not scattered strings. Typos cause silent bugs. |
| Traits used for code organization | Traits are for shared behavior across unrelated classes, not for making a class "look smaller". |
| Hardcoded URL paths in code/tests | Every route must be named. All code references routes by name. |
| Messy namespaces / "Helpers" dumping ground | Namespaces mirror directories, group by domain. No `Utils`, no 6-level nesting. |
| Timestamps not stored in UTC | Store UTC, display local. Mixing timezones in DB = wrong sorts, wrong aggregations, DST bugs. |
| Database timezone not verified | Check DB server timezone is UTC at setup. A default local timezone silently converts your values. |
| Date format not locale-aware | "03/04/2026" is ambiguous. Use locale-aware formatting per language. |
| Generic exceptions everywhere | `throw new Exception("error")` hides what actually failed. Domain exceptions are catchable, loggable. |
| Stack traces visible in production | Global error handler must catch all. API → JSON error. Web → custom error page. Never expose internals. |
| Sensitive data in logs | Passwords, tokens, card numbers in log files = plain text credentials accessible to all devs. |
| No request/correlation ID | User reports error, you grep logs for "around 2pm." With request ID: find exact request in seconds. |
| Secrets committed to git | `.env` in git history = permanent. One leaked AWS key = $50k bill overnight. Scan for secrets in CI. |
| Stale `.env.example` | New dev clones repo, app crashes on missing env var. Update `.env.example` every time. |
| No branch protection on main | One direct push with a bug = broken production. All changes via PR + review + CI pass. |
| Lock files not committed | "Works on my machine" because you have a different sub-dependency version. Commit lock files. |
| Dependencies never updated | Ignoring updates for a year = security holes + impossible upgrade. Small monthly updates win. |
| No database indexing strategy | Missing index on FK = full table scan on every JOIN. Review slow query log monthly. |
| "Soft delete everything" | Soft deletes leak into queries, bloat database, complicate GDPR. Decide per entity. |
| Accessibility ignored until launch | EU legal requirement since 2025. Semantic HTML + keyboard nav + contrast = 80% of compliance. Retrofit is 10x harder. |
