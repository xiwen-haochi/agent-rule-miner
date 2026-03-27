---
name: rule-miner
description: |
  ALWAYS use this skill when the user asks to "analyze the project", "learn the coding style", "generate project rules", "mine coding conventions", "create .cursorrules", "generate copilot instructions", "分析项目风格", "学习代码规范", "生成项目规则", "挖掘编码习惯", or wants the AI to follow the project's existing coding habits instead of generic AI defaults. Also use when the user says "help AI understand this project", "make AI code like the team", or any request about making AI-generated code consistent with existing project patterns.
  
  This skill reads the entire project codebase, extracts real coding patterns (including hidden conventions and anti-patterns), and writes ≤1000-word rules to IDE config files so future AI coding stays consistent with the project's established style — not generic best practices.
---

# Rule Miner

Mine a project's real coding DNA and write it as AI-ready rules (≤1000 words) into standard IDE config files.

The core insight: most projects have drifted from language defaults in specific, intentional ways. The goal is to find exactly those deviations — the things the team always does, never does, or does together — not to restate what the language docs already say.

---

## Boundaries — What This Skill Does NOT Do

- **Does NOT modify source code** — only reads it. The sole output is rule config files.
- **Does NOT execute or run project code** — no builds, no test runs, no scripts.
- **Does NOT refactor or suggest refactors** — it describes what IS, not what SHOULD BE.
- **Does NOT enforce rules** — it writes them for other AI sessions to follow; enforcement is the IDE's job.
- **Does NOT replace linter/formatter config** — if a rule is already enforced by tooling, skip it.
- **Does NOT apply across projects** — rules are project-specific; never carry rules from one project to another.

---

## Red Lines — Absolute Prohibitions

1. **Never invent rules without evidence.** Every rule must trace back to a pattern observed in ≥2 files. If you can't point to the files, don't write the rule.
2. **Never include sensitive information** in generated rules — no API keys, passwords, internal URLs, private package names, or security implementation details.
3. **Never silently overwrite** existing IDE config files. Always read first, merge if content exists, and ask the user before writing.
4. **Never exceed the 1000-word limit.** If you have more rules than fit, cut the lowest-priority ones (low frequency × low deviation).
5. **Never generate rules that contradict actual code.** If the codebase is inconsistent on a point, either note the inconsistency or skip the rule — don't pick a side.
6. **Never state language/framework defaults as rules.** "Use camelCase for variables in JavaScript" is not a rule — it's a default. Only codify deviations.

---

## Edge Cases & Fallback Strategies

| Scenario | Strategy |
|----------|----------|
| **Empty project** (no source files) | Inform the user: "No source files found to analyze. Add code first, then run rule-miner again." Do not generate empty rules. |
| **Very small project** (<5 source files) | Proceed but warn: "Small sample size — rules may change as the project grows. Re-run after more code is written." Lower the frequency threshold to ≥2 occurrences. |
| **Very large project** (>500 source files or token limit hit) | Use stratified sampling: read 100% of core/shared modules, then sample 30% of remaining files proportionally from each directory. Note in output: "Rules based on sampled analysis of large codebase." |
| **Mixed languages** (e.g., Python backend + TypeScript frontend) | Generate separate rule sections per language, clearly labeled. Share cross-language rules (e.g., directory naming) in a common section. |
| **Generated/scaffolded code dominates** | Exclude generated files (already skipped in Phase 2). If >80% is generated, warn the user. |
| **Existing IDE config files** | Read existing content. Append mined rules in a clearly marked `<!-- rule-miner -->` section. Never delete the user's existing rules. |
| **Contradictory patterns** (50/50 split) | Do not write a rule for that pattern. Optionally mention it to the user: "Inconsistent pattern detected for X — no rule generated." |

---

## Workflow

Work through these phases in order. Don't skip ahead.

### Phase 1 — Project Reconnaissance

**Goal**: Understand the project's shape before reading any code.

1. List the root directory to see top-level structure
2. Identify the primary language(s) — look for `package.json`, `go.mod`, `Cargo.toml`, `pyproject.toml`, `pom.xml`, `*.csproj`, `Gemfile`, etc.
3. Note the directory layout — monorepo? single package? flat or layered?
4. Find config files: `tsconfig.json`, `.eslintrc*`, `.prettierrc*`, `black.toml`, `rustfmt.toml`, etc. These reveal enforced conventions and skip manual inference for things already configured.
5. List all source files (skip: `node_modules/`, `dist/`, `build/`, `.git/`, `__pycache__/`, `target/`, `vendor/`, `*.min.js`, `*.lock`, `*.sum`). Count them to calibrate analysis effort.

Before moving on, mentally answer:
- What language(s) and what version/dialect?
- Roughly how many source files?
- Is there a clear architectural pattern (MVC, layered, hexagonal, etc.)?

---

### Phase 2 — Full Source Reading

**Goal**: Read all source files that contain human-written code logic.

Reading priority order (read all of them, but prioritize if tokens are scarce):
1. **Core modules** — main entry points, primary business logic, shared utilities
2. **Helper / shared code** — utility functions, base classes, common types
3. **Configuration files** — tooling config, env templates, build files
4. **Tests** — at least a sample to understand test conventions

While reading, actively track patterns across files using a mental tally. You care about frequency and consistency. A pattern seen in 80%+ of files is a real convention. A pattern seen once might be noise.

**Skip** generated files, lockfiles, minified code, binary assets, and vendor copies.

---

### Phase 3 — Pattern Extraction

Extract evidence across these 9 categories. For each pattern, mentally note: "Would a new developer writing fresh code in this language naturally do this?" If yes, it's not worth a rule. If no, it IS worth a rule.

**1. Function & Code Body Patterns** *(this is the #1 category — AI writes function bodies 90% of the time)*

This category determines whether AI-generated code *reads* like the rest of the project line-by-line.

- **Function length**: short (5-15 lines) or long (30+)? If most functions are short, the team values decomposition — AI should never generate 50-line functions.
- **Function declarations vs arrow functions**: `function foo()` vs `const foo = () =>`? Mixed usage or strict preference?
- **Parameter style**: individual params `(name, age, email)` vs options object `(options: CreateUserOpts)`? At what param count does the project switch?
- **Destructuring**: heavy destructuring in params `({ name, age })`, in assignments `const { x, y } = obj`, or minimal?
- **Return style**: early returns with guard clauses, or single return at the end? Ternary expressions or if/else blocks?
- **Async patterns**: `async/await` everywhere, or `.then()` chains? Are `try/catch` blocks around every await, or is there a global error boundary?
- **Variable declarations**: `const` by default, or `let`? Any `var` usage? How often are variables reassigned?
- **Immutability preference**: spread operators / `Object.assign` / `.map()` for new arrays, or in-place mutation?
- **Type system patterns** (typed languages only): `interface` vs `type`? Enums vs union types vs const objects? `null` vs `undefined`? How heavy is generic usage?
- **String style**: template literals vs concatenation? Single vs double quotes (if not linter-enforced)? Multi-line strings: template literals or array `.join()`?
- **Chaining / piping**: does the project chain method calls (`arr.filter().map().reduce()`) or use intermediate variables?

These micro-patterns are invisible in naming or organization rules, but they're what make code look "native" to a project.

**2. Naming Conventions**
- File names: `camelCase.ts` vs `kebab-case.ts` vs `snake_case.py`?
- Classes/interfaces/types: PascalCase fine, but any suffix conventions? (`Service`, `Manager`, `Handler`, `Repo`, `Impl`)?
- Functions/methods: any prefix patterns (`handle*`, `get*`, `on*`, `use*` for hooks)?
- Variables: any notable patterns (`_private`, `SCREAMING_SNAKE` for constants, `I` prefix for interfaces)?
- One deviating convention is worth 50 generic ones.

**3. Code Organization**
- Directory naming: plural or singular? (`utils/` vs `util/`, `services/` vs `service/`)
- Index file usage: barrel exports (`index.ts`) everywhere, or sparse?
- File-per-class vs multi-export files?
- Any circular dependency avoidance strategies visible in the structure?

**4. Import / Dependency Style**
- Import ordering: stdlib → third-party → local? Or mixed? Enforced by tooling or by hand?
- Aliasing: `import * as X`, destructured only, or both?
- Relative vs absolute paths (`../../utils` vs `@/utils`)?
- Any imports that always appear together?

**5. Error Handling**
- Exceptions vs Result/Either types vs error codes?
- Are errors wrapped or re-thrown raw?
- Is there a shared error type/class hierarchy?
- Logging style: `console.error` / `logger.error` / structured JSON?

**6. Comment & Documentation Style** *(treat this as equally important as naming — comments reveal how the team thinks)*

Read at least 30 comment instances across different file types before drawing conclusions. Look for:

**Density & placement**
- Ratio of commented vs uncommented functions — is every function annotated, or only complex ones, or almost none?
- Where do comments appear: above the function, inside the body, or both?
- Inline end-of-line comments: common, rare, or never?
- Block comments (`/* */`) vs line comments (`//` / `#`): which do they prefer for multi-line blocks?

**Content philosophy**
- Do comments explain *what* the code does, or *why* the decision was made? (why-only = strong convention to encode)
- Are there "context comments" that reference tickets, links, or business rules? (e.g., `// See JIRA-1234`, `// Business rule: ...`)
- Are there "warning comments" that flag traps or non-obvious behavior? (e.g., `// Don't change the order — side effect`, `// Intentionally not using X here because...`)
- Are commented-out code blocks present — tolerated or never seen?

**Format & language**
- Comment language: English, Chinese, mixed — and does it follow the code language or differ?
- Sentence style: full sentences with periods, or lowercase fragments?
- JSDoc / TSDoc / docstrings: used on all public APIs, only on exported ones, only on complex ones, or barely used?
- When JSDoc IS used: which tags appear (`@param`, `@returns`, `@throws`, `@example`, `@deprecated`)? Which are conspicuously absent?
- Are `@param` descriptions just restating the type (useless) or adding semantics (useful)?

**TODO / FIXME conventions**
- Format: `// TODO(author): message`, `// TODO: message`, `# FIXME`, freeform?
- Are TODOs ever resolved, or is the codebase littered with them (implies they're noise)?
- Any custom tags like `// HACK:`, `// TEMP:`, `// NOTE:`?

**Section dividers**
- Do they use visual separators to chunk long files? (e.g., `// ─── Helpers ───`, `// === Public API ===`)
- Any file-level header comments (author, purpose, license)?

A project that writes almost no comments has a rule: *don't add noise comments*. A project that writes why-comments on every non-trivial block has a rule: *always explain the why*. Both are equally valid and must be captured faithfully.

**7. Test Style**
- Framework: Jest/Vitest/pytest/go test/RSpec?
- Test file co-location or separate `__tests__` / `tests/` directory?
- Describe/it nesting depth convention?
- Mock philosophy: heavy mocks, minimal mocks, or integration-preferred?
- Test data: factory helpers, fixtures files, or inline literals?

**8. Hidden Associations (Coupling Patterns)**
Look for things that always appear together across the codebase:
- "Every service file always has a corresponding types file"
- "Every exported function always has a matching test"
- "Every API handler always validates input then calls exactly one service method"
- "Every database call is always wrapped in a transaction helper"
These are the most valuable rules — they encode implicit architecture contracts.

**9. Anti-Patterns (What This Project Never Does)**
What common language/framework idioms are conspicuously absent?
- No `any` types in TypeScript?
- No raw SQL strings (always uses query builder)?
- No direct `process.env` access (always uses a config module)?
- No `class` keyword (pure functional)?
- No default exports?
These are as important as positive rules — they prevent AI from introducing patterns the team actively avoids.

---

### Phase 3.5 — Domain-Specific Pattern Check (Conditional)

If Phase 1 identified the project as a **frontend app**, **backend service**, or **full-stack project**, run these additional checks. Skip entirely for libraries, CLIs, or infrastructure tools.

#### Frontend Projects (React / Vue / Angular / Svelte / etc.)

**Component patterns**
- Component structure: one file per component, or folder-per-component (`Button/index.tsx` + `Button.module.css` + `Button.test.tsx`)?
- Functional components only, or class components still present? Mixed?
- Component size: small single-responsibility or large "page" components?
- Props: destructured in parameter `({ name, onClick })` or accessed via `props.name`?
- Default props: default parameter values, or separate `defaultProps` / `withDefaults`?

**State management**
- Which tool: Redux / Zustand / Pinia / MobX / Context / Jotai / signals / none?
- Where does state live: global store, component-local, URL params, or a mix?
- Data fetching: custom hooks, React Query / SWR / TanStack, or in components directly?
- Loading/error states: how are they represented? Discriminated unions, boolean flags, or enums?

**Styling approach**
- Method: CSS Modules, Tailwind utility classes, styled-components / Emotion, SCSS/LESS, inline styles?
- Class naming: BEM, utility-first, or freeform?
- Theme/design tokens: CSS variables, JS theme objects, or Tailwind config?
- Responsive strategy: mobile-first or desktop-first breakpoints?

**Routing & pages**
- File-based routing (Next.js / Nuxt convention) or config-based?
- Route guards / auth wrappers: HOC, middleware, or layout-level?
- Page loading patterns: suspense, skeleton, spinner, or none?

**Event handling**
- Handler naming: `onClick` / `handleClick` / `onClickHandler`?
- Are handlers defined inline in JSX, or extracted as named functions above the return?
- Form handling: controlled vs uncontrolled? Which form library (react-hook-form, formik, native)?

#### Backend Projects (Express / NestJS / FastAPI / Django / Spring / Go net/http / etc.)

**API design**
- REST naming style: plural resources (`/users`) or singular (`/user`)? Nested routes (`/users/:id/posts`) or flat?
- Response envelope: `{ data, error, meta }` wrapper, or raw payload?
- Pagination: cursor-based, offset/limit, or page/size? Where are defaults?
- API versioning: URL path (`/v1/`), header, or none?
- HTTP status codes: precise (201, 204, 409) or loose (200 for everything, 500 for errors)?

**Request lifecycle**
- Middleware ordering: auth → validation → handler → error handler? Any custom pattern?
- Input validation: where does it happen — controller, service, or dedicated middleware? Which library (Joi, Zod, class-validator, Pydantic)?
- DTO / request types: separate type per endpoint, or shared types?

**Database patterns**
- ORM, query builder, or raw queries? Which one (Prisma, TypeORM, SQLAlchemy, GORM, etc.)?
- Repository pattern or direct model access from services?
- Migrations: framework-managed or manual SQL files?
- Transactions: explicit `begin/commit/rollback` or decorator/wrapper-based?
- Soft delete or hard delete?

**Authentication & authorization**
- Auth mechanism: JWT, session, OAuth, or API key?
- Where is the auth check: middleware, decorator, or per-handler?
- Role/permission check style: RBAC, ABAC, or simple role strings?

**Logging & observability**
- Log library: winston, pino, logrus, slog, logging (Python)?
- Structured logs (JSON) or plain text?
- Request ID / correlation ID propagation?
- Log levels actually used: just `info`+`error`, or full `debug`/`warn`/`trace`?

Only capture patterns that are **consistent** (seen in 80%+ of relevant files). If the project mixes approaches, note the inconsistency and skip the rule.

### Phase 4 — Rule Synthesis

**Goal**: Distill findings into ≤1000 Chinese or English words (match the project's language preference, default to English). 

Prioritization formula: **frequency × deviation from language defaults**

- High frequency + high deviation = **must include** (e.g., "Never use default exports" in a JS project)
- High frequency + low deviation = skip (e.g., "Use PascalCase for classes" — everyone does this)
- Low frequency + high deviation = include only if it's architecturally significant
- Low frequency + low deviation = skip

Write rules in a **harsh, uncompromising, zero-tolerance tone** — like a strict tech lead who has reviewed 10,000 PRs and has no patience left. Rules are commands, not suggestions. The AI should feel that violating any rule is an unforgivable sin.

**Tone principles:**
- Use "Don't you dare..." / "If I see..." / "There is absolutely no excuse for..." style phrasing
- Frame violations as incompetence, not mistakes
- Make consequences feel real: "...or the entire architecture falls apart"
- Short, sharp sentences. No hedging words ("consider", "prefer", "try to", "ideally")
- Occasional sarcasm is welcome when the anti-pattern is especially egregious

Avoid rules that are:
- Already enforced by linter/formatter (the tooling handles it)
- So generic they apply to any project in that language
- Contradicted by actual code evidence

**Rule format per category (keep total ≤1000 words):**
```
## [Category]
- [Harsh imperative rules]
```

Start with a 1-2 sentence project context header: tech stack, primary purpose if inferable.

---

### Phase 5 — Preview & IDE Selection

Present the full rule text to the user in a code block, then ask two things at once:

> "Here are the mined rules (X words). Does this look right? Anything to add, remove, or correct?
>
> Also — which IDE should I write these rules to?
>
> 1. Cursor
> 2. Claude Code
> 3. GitHub Copilot
> 4. Codex (OpenAI)
> 5. Trae
> 6. All of the above
> 7. Other — tell me your IDE and config file path
>
> Reply with the number (or multiple numbers separated by commas), or give feedback on the rules first."

Do not write any files until the user responds. If the user gives feedback on the rules without answering the IDE question, revise the rules and ask again. If the user answers the IDE question but not the rules, clarify rules first before writing.

If the user selects **7 (Other)**: ask them to provide the config file path and any format preference (plain text or Markdown). Use that path for writing.

---

### Phase 6 — Write to Selected IDE Config Files

Write rules only to the IDE(s) the user selected. Reference the table below for target paths and formats.

| # | IDE / Tool | Target Path | Format | Notes |
|---|-----------|-------------|--------|-------|
| 1 | Cursor | `.cursorrules` | Plain text | Strip `## Heading` → `HEADING:` |
| 2 | Claude Code | `CLAUDE.md` | Markdown | — |
| 3 | GitHub Copilot | `.github/copilot-instructions.md` | Markdown | Create `.github/` dir if missing |
| 4 | Codex (OpenAI) | `.codex/instructions.md` | Markdown | Create `.codex/` dir if missing |
| 5 | Trae | `.trae/instructions.md` | Markdown | Create `.trae/` dir if missing |
| 7 | Other | *(user-provided path)* | *(user preference)* | — |

**For every target file:**
1. Check if the file already exists
2. If it exists: read it, look for `<!-- rule-miner-start -->` / `<!-- rule-miner-end -->` markers
   - Markers present → replace content between them
   - No markers → append the mined rules wrapped in markers at the end
3. If it does not exist: create the file with the rules wrapped in markers
4. For `.cursorrules` (plain text), use `# --- rule-miner-start ---` / `# --- rule-miner-end ---` instead

After writing, confirm each file:
```
✓ .cursorrules (843 words)
✓ CLAUDE.md (843 words)
```
If the user selected multiple IDEs, write all of them and show a combined summary.

---

## Rules Quality Checklist

Before presenting rules to the user, verify:
- [ ] Total word count ≤ 1000
- [ ] Every rule has evidence from actual files (not assumed)
- [ ] At least one "hidden association" rule (the most unique insight)
- [ ] At least one anti-pattern rule (what NOT to do)
- [ ] No rules already covered by `.eslintrc` / `black.toml` / formatter configs
- [ ] Rules read as AI instructions, not as human documentation

---

## Example Rule Output Shape

```markdown
# Project Rules
> Stack: TypeScript · Express · PostgreSQL. REST API service.

## Naming
- Service files end with `Service.ts`, interfaces with `IService.ts`. No exceptions. Don't get creative with naming — creativity belongs in the logic, not the filenames.
- Route handlers are `handle[Verb][Resource]`. If I see `processData` or `doStuff` one more time, I'm mass-reverting the PR.
- Constants are SCREAMING_SNAKE_CASE at module level. Put a SCREAMING_SNAKE inside a function and you've proven you don't understand scope.

## Code Organization
- Every service has a `[name].types.ts` right beside it. A service without its types file is a service that doesn't exist yet.
- Barrel exports (`index.ts`) live at the top of each feature folder. Nested barrels are a dependency graph nightmare — don't even think about it.
- All database access goes through the `db/` layer. If you import `pg` directly in a service, you just became the single point of failure. Congratulations.

## Imports
- Order: Node stdlib → third-party → `@/` aliases → relative. Blank line between groups. Scrambled imports tell me you don't read your own code.
- Named exports everywhere. Default exports are for Next.js pages and literally nothing else.

## Error Handling
- Every service error gets wrapped in `AppError` with an HTTP status. Raw `Error` objects reaching the response layer is amateur hour — there is absolutely no excuse.

## Comments
- Comment the **why**, never the what. If your comment says `// increment counter` above `counter++`, just delete both.
- Public service methods get a one-line JSDoc. No `@param`, no `@return` — the types already say that. Don't repeat yourself.

## Testing
- Tests in `__tests__/` beside source, named `[file].test.ts`. Tests in a distant `test/` folder means nobody runs them.
- Use `createMockService()` factories. Inline `jest.fn()` spaghetti is unreadable and you know it.

## Hidden Associations
- Every route file registers in `routes/index.ts`. Orphan route files are dead code with extra steps.
- Validation middleware ALWAYS comes before the handler. No validation = no handler. Non-negotiable.

## Anti-Patterns
- Use `any` and I will mass-revert your entire PR. `unknown` exists for a reason — learn to narrow.
- Touch `process.env` outside `config/env.ts` and you just scattered secrets across the codebase. Well done.
- Raw SQL strings? In this project? Absolutely not. The query builder is right there.
```

This example is ~280 words. Real output will be longer but must stay under 1000.


