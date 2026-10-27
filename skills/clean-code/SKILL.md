---
name: clean-code
description: Clean code baseline for all generated code. Use automatically whenever a prompt results in writing, generating, or modifying code — any language, any framework, any layer. Applies naming, function-size, structure, and simplicity rules before the code is returned.
---

# Clean Code

Apply these rules to every piece of code you write or modify, in any language or framework. They are a baseline, not a refactoring pass — new code follows them from the first draft; edited code follows them for the lines you touch.

Examples use generic pseudo-names; translate them into the idioms of the language and framework in front of you.

## Naming

- Names say what a thing is or does — no abbreviations (`customerRepository`, not `custRepo`), no type suffixes that lie, no `data`/`temp`/`result2`.
- Variables are named for the value they hold, not how they're used mechanically: `unpaidInvoices`, not `list` or `items`. If you can't name it meaningfully, the code around it is probably doing too much.
- Single letters only for trivial loop indices (`i`, `j`) or established math/domain notation — never `e`, `d`, `x` for real values.
- Booleans read as questions: `isActive`, `hasAccess`, `canRetry`.
- Functions/methods are verbs (`calculateTotal`), classes/types/structs are nouns (`InvoiceCalculator`).
- One name per concept per codebase — if the project says `fetch`, don't introduce `get`/`retrieve` for the same thing.
- Use the language's casing convention (`snake_case` in Python/Rust/Ruby, `camelCase` in JS/TS/Java/Kotlin/Swift, exported `PascalCase` in Go/C#) — the rule is about meaning, not case.

## Functions

- One function, one job. If describing it needs "and", split it.
- Keep them short enough to read without scrolling; extract when a block needs a comment to explain what it does — the extracted function's name becomes the comment.
- Max 3–4 parameters; more means the parameters want to be an object/struct/record/data class.
- No boolean flag parameters that switch behavior — two functions instead.
- Return early on guard conditions; avoid deep nesting (max ~2 levels).

## Code style

- Use the language's idiomatic function and module style, and whatever form the surrounding code already uses (e.g. arrow functions if a JS/TS codebase uses them, free functions vs. methods as the codebase does).
- When writing new logic, separate it into its own function (or service, per the rules below) so it can be reused — don't bury it inline where only one caller can reach it. Before writing it, check if it already exists somewhere reusable.

## Services

- If a page, view, controller, handler, or screen contains isolated functionality (logic that is self-contained and not tied to rendering or request wiring), extract it into a service.
- Put services where the framework expects them; if it has no convention, use a dedicated `services` module/package/folder (e.g. `services/` in JS/TS, a `services` package in Python/Go/Java, `app/services` in Rails, `app/Services` in Laravel).
- One service per file, named after what it does (`auth`, `invoice_export`, `InvoiceExportService`).
- UI and entry-point code keep only rendering and wiring; they call the service, they don't own the logic.
- Don't extract prematurely: logic used by one caller and under ~20 lines can stay inline until it grows or a second caller needs it.

## Constants & Enums

- No magic strings or magic numbers: every literal with meaning gets a named constant (`MAX_RETRIES = 3`, not a bare `3`).
- Declare constants with the language's idiom (`const`, `static final`, module-level `UPPER_SNAKE_CASE`, `const` block, etc.).
- If a constant is used globally — or could plausibly be — put it in a shared `constants` module/package/folder, grouped by domain (e.g. `api_routes`, `limits`).
- Enums go in a shared `enums` module/package/folder, one enum per file, named after the enum. If the language has no native enum, use its idiomatic substitute (union of literals, `Enum` class, `iota` constants, sealed class).
- If a value is only used within one scope (function, component, file), declare it as a constant in that scope instead of the shared location.

## Models

- Every time a model is created — or a value is model-worthy (a structured object with named fields that represents a domain thing) — define it as a named type in its own file in a `models` module/package/folder.
- One model per file, named after the model (`user`, `invoice`).
- Never define model shapes inline in pages, controllers, services, or components; import them from the models location.
- Where the framework owns models (ORM entities, schema files such as Django `models.py`, Rails `app/models`, Prisma `schema.prisma`, JPA entities), follow its convention instead.

## Simplicity

- Match the existing codebase: reuse its helpers, patterns, and naming before writing anything new.
- No speculative abstractions — no interface with one implementation, no config for a constant, no "for later" scaffolding.
- Delete dead code you replace; never leave commented-out code behind.
- Prefer stdlib/platform/framework features over custom code, custom code over new dependencies.

## Comments & errors

- Comments explain *why*, never *what* — code that needs a what-comment needs a better name or an extraction instead.
- No noise comments (`// constructor`, `# loop over items`).
- Handle errors where you can act on them; never swallow one silently (empty `catch`/`except`/`rescue`, ignored error return). Fail with a message that names what failed and with what input.
- Validate input at trust boundaries (API, user input, file/network reads) — not on every internal call.

## Formatting & consistency

- Follow the project's existing formatter/linter config and style; never introduce a competing style.
- Keep related code together: declare variables next to first use, keep small private helpers near their caller.

## Rules

- These apply silently — don't narrate that you followed them.
- When a rule conflicts with the language's idioms, the framework's conventions, or the surrounding codebase's established style, those win; consistency beats the checklist.
- Explicit user instructions override any rule here.
