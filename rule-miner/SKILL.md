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
| **Generated/scaffolded code dominates** | Detect generated markers (auto-generated comments, identical structures). Exclude generated files from pattern extraction. If >80% is generated, warn the user. |
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

Extract evidence across these 8 categories. For each pattern, mentally note: "Would a new developer writing fresh code in this language naturally do this?" If yes, it's not worth a rule. If no, it IS worth a rule.

**1. Naming Conventions**
- File names: `camelCase.ts` vs `kebab-case.ts` vs `snake_case.py`?
- Classes/interfaces/types: PascalCase fine, but any suffix conventions? (`Service`, `Manager`, `Handler`, `Repo`, `Impl`)?
- Functions/methods: any prefix patterns (`handle*`, `get*`, `on*`, `use*` for hooks)?
- Variables: any notable patterns (`_private`, `SCREAMING_SNAKE` for constants, `I` prefix for interfaces)?
- One deviating convention is worth 50 generic ones.

**2. Code Organization**
- Directory naming: plural or singular? (`utils/` vs `util/`, `services/` vs `service/`)
- Index file usage: barrel exports (`index.ts`) everywhere, or sparse?
- File-per-class vs multi-export files?
- Any circular dependency avoidance strategies visible in the structure?

**3. Import / Dependency Style**
- Import ordering: stdlib → third-party → local? Or mixed? Enforced by tooling or by hand?
- Aliasing: `import * as X`, destructured only, or both?
- Relative vs absolute paths (`../../utils` vs `@/utils`)?
- Any imports that always appear together?

**4. Error Handling**
- Exceptions vs Result/Either types vs error codes?
- Are errors wrapped or re-thrown raw?
- Is there a shared error type/class hierarchy?
- Logging style: `console.error` / `logger.error` / structured JSON?

**5. Comment & Documentation Style**
- JSDoc / TSDoc / docstrings: used on all public functions, or only on complex ones, or barely used?
- Comment language: English, Chinese, mixed?
- TODO format: `// TODO(author): message` or `# TODO: message` or freeform?
- Are comments explaining *what* or *why*? (why-only is a meaningful convention)

**6. Test Style**
- Framework: Jest/Vitest/pytest/go test/RSpec?
- Test file co-location or separate `__tests__` / `tests/` directory?
- Describe/it nesting depth convention?
- Mock philosophy: heavy mocks, minimal mocks, or integration-preferred?
- Test data: factory helpers, fixtures files, or inline literals?

**7. Hidden Associations (Coupling Patterns)**
Look for things that always appear together across the codebase:
- "Every service file always has a corresponding types file"
- "Every exported function always has a matching test"
- "Every API handler always validates input then calls exactly one service method"
- "Every database call is always wrapped in a transaction helper"
These are the most valuable rules — they encode implicit architecture contracts.

**8. Anti-Patterns (What This Project Never Does)**
What common language/framework idioms are conspicuously absent?
- No `any` types in TypeScript?
- No raw SQL strings (always uses query builder)?
- No direct `process.env` access (always uses a config module)?
- No `class` keyword (pure functional)?
- No default exports?
These are as important as positive rules — they prevent AI from introducing patterns the team actively avoids.

---

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

### Phase 5 — Preview & Confirmation

Before writing any files, present the full rule text to the user in a code block and ask:

> "Here are the mined rules (X words). Does this look right? Anything to add, remove, or correct? Reply 'ok' to write to files, or give feedback."

Do not write files until the user confirms.

---

### Phase 6 — Write to IDE Config Files

After user confirmation, write rules to all applicable files. For each target:

**Check if file exists first.** If it exists, read it and merge: append a clearly marked section rather than overwriting existing content.

| IDE / Tool | Target Path | Format |
|-----------|-------------|--------|
| GitHub Copilot | `.github/copilot-instructions.md` | Markdown |
| Claude Code | `CLAUDE.md` | Markdown |
| Cursor | `.cursorrules` | Plain text |
| Trae | `.trae/instructions.md` | Markdown |

For `.cursorrules`, strip Markdown heading syntax (replace `## Heading` with `HEADING:`) since Cursor treats it as plain text.

For each file written, confirm the path and word count. Example:
```
✓ .github/copilot-instructions.md (843 words)
✓ CLAUDE.md (843 words)  
✓ .cursorrules (843 words)
✓ .trae/instructions.md (843 words)
```

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

---

## Merge Strategy for Existing Config Files

When a target config file already exists:

1. Read the entire existing content
2. Look for a `<!-- rule-miner-start -->` / `<!-- rule-miner-end -->` marker pair
3. If markers exist: **replace** only the content between them (this is a re-run)
4. If no markers exist: **append** at the end with markers wrapping the new content
5. For `.cursorrules` (plain text), use `# --- rule-miner-start ---` / `# --- rule-miner-end ---` comments instead

This ensures user-written rules are never lost, and re-runs cleanly update only the mined section.
