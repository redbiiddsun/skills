---
name: code-principles-review
description: Review code in any language or framework against Clean Code, DRY, KISS and SOLID plus the project's own conventions, and report ranked, actionable findings per layer (entry points/controllers, services, models, shared utils, config/wiring). Use this whenever the user asks for a code review, a quality pass, a refactor suggestion, or mentions clean code, DRY, KISS, SOLID, SRP, coupling, duplication, "is this well structured", or wants the codebase reviewed by an agent team. Also use it before merging a large change, even if the user only says "look over this".
---

# Code principles review

The goal of a review is a short, ranked list of changes that make the code easier to change safely, not a tour of everything that could theoretically be different. A finding earns its place only if a reader can act on it.

## Learn the project first

Before judging anything, find the rulebook and the layout:

- Read the project's `CLAUDE.md` / `AGENTS.md` / `CONTRIBUTING.md` / architecture docs. Where they exist, they are the rules this review enforces; they override the generic guidance below.
- Identify the language, framework, and its conventions (e.g. Spring, Django, Rails, Laravel, Express/Nest, Elysia, ASP.NET, Gin, Flutter, React). Judge code by the framework's idioms, not another framework's.
- Map the layers. Most projects have some version of:

| Layer | Job |
|-------|-----|
| Entry point / controller / handler / route / view | Routing, input validation, auth wiring, translating results to responses. No business rules, no data access. |
| Service / use case / domain | Business rules and orchestration. Takes plain typed values, never the framework's request/context object. |
| Model / schema / entity / DTO | Single source of truth for shapes. Types derived from schemas, not hand-duplicated. |
| Repository / data access (if present) | Queries and persistence. |
| Shared utils / lib | Generic helpers; never import feature modules. |
| Config / wiring / bootstrap | Reads and validates configuration once, composes modules, starts the app. |

Dependencies should point one way: wiring → entry points → services/models → utils/config. A service never imports a controller; utils never import a feature.

## Procedure

1. **Build, type-check, and test first.** Run the project's compile / type-check / lint / test commands before reading anything. Code that does not build is the wrong code to review, and the break is finding #1. If tests need infrastructure that isn't running (database, emulator), say so and continue with what did run.
2. **Scope.** Decide what is under review: the whole repo, one feature module, or a diff. For the whole repo, review one feature module per pass (plus a pass for wiring/config/utils). If an agent team is requested, run one read-only reviewer per module in parallel, each given this file, the project's rule docs, and the neighbouring files it needs; the lead applies fixes afterwards.
3. **Read before judging.** Read every file in scope fully. Trace one request or user action end to end (entry point → validation → auth → service → data access → response) so you know where validation, auth, and transactions actually happen.
4. **Apply the checklist** in the next sections. For each hit, confirm it is real by pointing at the exact lines.
5. **Rank and cap.** Most important first. Cap at roughly 12 findings per module. Drop nits when there are substantive issues.
6. **Report** in the format at the end. Then, if asked to fix, apply the high and medium findings, rerun the build/tests, and follow the project's review loop if it has one (e.g. tech-lead → developer agents).

## What each principle means

**Clean Code.** Names say what a thing is for, not how it is built. Functions do one thing at one level of abstraction. No magic strings or numbers: meaningful literals become named constants, user-facing error text lives in one place. Comments explain why (a lock, a timing-safe compare), never what. Dead code, unused exports, and unused parameters go. Errors are translated to the caller's language (HTTP status, UI message) at the layer that knows that meaning; unexpected errors are never swallowed silently.

**DRY.** One source of truth for each rule. Watch for decisions that will drift: the same field constraint written in two schemas; the same not-found / soft-delete / permission handling spread across services without a helper once it starts to diverge; entity-to-response mapping done inline in several places instead of one mapper; guard/auth logic copied instead of reusing the shared middleware/decorator; config parsing repeated outside the config module. Duplicated three lines is fine; duplicated decisions are not.

**KISS.** Prefer the plainest construct that meets the requirement. A function beats a class with state. A thin handler that calls one service method beats an elaborate controller hierarchy. Don't add a repository layer, DI container, generic CRUD base, or plugin system for a future that is not on the roadmap. Use what the language, framework, and installed libraries already provide (validation, auth middleware, ORM/query builder, crypto, password hashing) before writing your own.

**SOLID.**

- _SRP_: a file has one job (see the layer table). A controller that builds SQL, or a service that sets response status codes or reads headers, is mixing jobs.
- _OCP_: new behaviour should come from adding a route, handler, schema, or strategy, not editing a growing `switch` on a string.
- _LSP_: a middleware, plugin, or subclass must behave the same wherever it is used — same errors, same resulting shape.
- _ISP_: a service method takes the fields it uses (a typed input, an id), not a whole request/context or grab-bag object.
- _DIP_: features depend on shared utils and config, not on each other's internals. A feature importing another feature's private helpers is a finding; promote the helper to shared code instead.

## Framework-agnostic checks

Check these first; they are cheap to spot and common in every stack.

- **Request/context leaking out of the entry layer.** Passing the framework's request, response, context, or session object into services. Services take plain values so they can be called from tests, jobs, and other entry points.
- **Missing or loose validation at trust boundaries.** Input read without a schema/validator, `any`/untyped where the shape is known, ids without format checks, unbounded strings or arrays.
- **Types duplicating schemas.** A hand-written type/class next to the schema/entity that already defines the shape. Derive it (DB row types describing storage, not the wire, are fine).
- **Guard not applied.** A route/screen on a protected area that forgets the auth guard, or a guard re-implemented inline. Prefer one place that protects the whole group so a new route cannot forget it.
- **Inconsistent error shapes.** Some paths throw, some return error objects with success status, some use the framework's error helper. Pick one style per project.
- **Non-atomic writes.** Check-then-write sequences (uniqueness checks, multi-table updates) outside a transaction, or inside one without the lock/constraint that makes the check meaningful.
- **Injection safety.** String-concatenated SQL/shell/HTML instead of parameterized queries, safe builders, or escaping.
- **Config outside the config module.** Environment variables read all over the codebase, or used without validation; app doesn't fail fast on invalid config.
- **Side effects at import/load time.** Modules that open connections, start servers, or schedule jobs merely by being imported, outside the bootstrap file.
- **Framework idioms ignored.** Hand-rolled versions of what the framework provides (routing, DI, validation, serialization), or patterns that defeat its type inference or tooling.

## Things worth keeping

Not every bit of complexity is a smell. Timing-safe comparisons for secrets, row or advisory locks inside transactions, soft-delete filters, idempotency keys, and fail-fast config validation each exist for a correctness or security reason. If you think one is not pulling its weight, say what would have to be true for it to be safe to remove rather than recommending removal outright.

## Finding format

For every finding:

```text
N. [severity] file:line — Principle
   What is wrong (one sentence).
   Change: the concrete edit you recommend.
```

Severity: **high** = real correctness, security or maintainability risk, **medium** = worth doing this iteration, **low** = nit, mention only if the list is short.

End each module's review with two or three sentences on overall health: what is good, what the single most valuable change would be.

## Things that are not findings

- Style that the compiler, linter, or formatter already enforces.
- Missing tests, unless the structure makes testing impossible. Say "hard to test because X" instead (e.g. "service takes the request object, so it cannot be called directly from a test").
- Features the user has not asked for.
- Rewriting working code into a different pattern of equal merit.
