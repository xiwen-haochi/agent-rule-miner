# Rule Miner

从项目代码中挖掘真实的开发习惯，生成 ≤1000 字的 AI 编码规则，写入主流 IDE 配置文件。

让 AI 写出**像你团队写的代码**，而不是千篇一律的 AI 风格。

---

## 为什么需要它？

AI 编码助手默认使用语言/框架的"标准写法"。但每个项目都有自己的味道：

- 某些命名前缀是团队约定（`handle*`、`use*`）
- 某些常见做法从来不用（比如不用 `any`、不用 default export）
- 某些文件总是成对出现（service 和 types 永远一起创建）
- 某些架构约束没有写在文档里，只存在于代码中

Rule Miner 会**全量阅读**你的项目源码，提取这些隐藏的开发 DNA，浓缩为精炼的规则，让后续 AI 编码自动遵循。

---

## 核心特性

- **语言无关** — 自动识别项目语言栈，适用于任何编程语言
- **8 类模式提取** — 命名、组织、导入、错误处理、注释、测试、隐藏关联、反模式
- **智能筛选** — 只保留"偏离语言默认"的规则，不复述常识
- **≤1000 字** — 精炼到位，不会淹没 AI 上下文窗口
- **多 IDE 输出** — 一次生成，四种格式
- **安全合并** — 不覆盖已有配置，使用标记区间追加- **毒舌风格** — 生成的规则以刻薄、不容忍的语气写就，让 AI 不敢越雷池一步
---

## 支持的 IDE

| IDE / 工具 | 输出文件 |
|-----------|---------|
| GitHub Copilot | `.github/copilot-instructions.md` |
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursorrules` |
| Codex (OpenAI) | `.codex/instructions.md` |
| Trae | `.trae/instructions.md` |
| 其他 | 用户自定义路径 |

规则生成后会**先询问你要写入哪个 IDE**，不会一次性全部覆盖。

---

## 安装

将 `rule-miner/` 文件夹复制到你的项目根目录（或使用 skill 安装机制）：

```
your-project/
├── rule-miner/
│   └── SKILL.md
├── src/
│   └── ...
└── package.json
```

---

## 快速安装

推荐使用 skills CLI 一键安装：

```sh
npx skills add xiwen-haochi/agent-rule-miner
```

或手动复制 `rule-miner/` 文件夹到项目根目录。

---

## 使用方法

安装后，在 AI 对话中输入以下任一指令：

```
分析这个项目的编码风格，生成项目规则
```

```
Mine the coding conventions of this project
```

```
Generate .cursorrules for this project
```

```
学习项目规范，帮 AI 理解这个项目
```

AI 将自动触发 Rule Miner skill，执行以下流程：

1. **侦察** — 扫描目录结构，识别语言/框架/已有 linter 配置
2. **全量读取** — 按优先级读取所有源文件
3. **模式提取** — 跨文件统计 8 类编码模式
4. **规则蒸馏** — 按"频率 × 偏离度"排序，生成 ≤1000 字规则
5. **预览确认** — 展示规则草稿，等你确认
6. **写入文件** — 写入 4 种 IDE 配置文件（已有内容不会被覆盖）

---

## 设计原则

| 维度 | 说明 |
|------|------|
| **核心定位** | 挖掘项目的编码 DNA，而非复述语言文档 |
| **边界约束** | 只读不写源码，不执行项目代码，不跨项目携带规则 |
| **禁忌红线** | 无证据不编规则、不暴露敏感信息、不覆盖用户配置、不超 1000 字 |
| **异常处理** | 空项目提示先写代码、小项目降低阈值、大项目分层采样、混合语言分区输出 |

---

## 示例输出

```markdown
# Project Rules
> Stack: TypeScript · Express · PostgreSQL. REST API service.

## Naming
- Service files end with `Service.ts`, interfaces with `IService.ts`. Don't get creative — creativity belongs in the logic, not the filenames.
- Route handlers are `handle[Verb][Resource]`. If I see `processData` or `doStuff`, I'm reverting the PR.

## Code Organization
- Every service has a `[name].types.ts` beside it. No types file = the service doesn't exist yet.
- Import `pg` directly in a service and you just became the single point of failure. All DB access goes through `db/`.

## Anti-Patterns
- Use `any` and I will mass-revert your entire PR. `unknown` exists for a reason.
- Touch `process.env` outside `config/env.ts` and you just scattered secrets across the codebase.
```

---

## 许可证

MIT

---

---

# Rule Miner (English)

Mine real coding conventions from your project source code, generate ≤1000-word AI coding rules, and write them to mainstream IDE config files.

Make AI write code **the way your team writes it** — not generic AI-style code.

---

## Why?

AI coding assistants default to "standard" language/framework patterns. But every project has its own flavor:

- Certain naming prefixes are team conventions (`handle*`, `use*`)
- Some common patterns are never used (no `any`, no default exports)
- Certain files always appear in pairs (service + types always created together)
- Architectural constraints live only in the code, not in docs

Rule Miner **reads your entire codebase**, extracts these hidden development DNA patterns, and distills them into precise rules that future AI coding sessions automatically follow.

---

## Key Features

- **Language-agnostic** — auto-detects your tech stack, works with any language
- **8 pattern categories** — naming, organization, imports, error handling, comments, tests, hidden associations, anti-patterns
- **Smart filtering** — only keeps rules that deviate from language defaults, skips obvious conventions
- **≤1000 words** — concise enough to not flood an AI context window
- **Multi-IDE output** — one analysis, four config formats
- **Safe merging** — never overwrites existing config; appends with markers
- **Harsh tone** — generated rules use a strict, no-nonsense, zero-tolerance voice that makes AI afraid to deviate

---

## Supported IDEs

| IDE / Tool | Output File |
|-----------|-------------|
| GitHub Copilot | `.github/copilot-instructions.md` |
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursorrules` |
| Codex (OpenAI) | `.codex/instructions.md` |
| Trae | `.trae/instructions.md` |
| Other | User-defined path |

After generating rules, Rule Miner **asks which IDE to write to** before touching any files — no silent mass-overwrite.

---

## Installation

Copy the `rule-miner/` folder into your project root (or use your IDE's skill install mechanism):

```
your-project/
├── rule-miner/
│   └── SKILL.md
├── src/
│   └── ...
└── package.json
```

---

## Usage

After installation, type any of these in your AI chat:

```
Mine the coding conventions of this project
```

```
Analyze this project's coding style and generate rules
```

```
Generate .cursorrules for this project
```

```
Help AI understand this project's coding patterns
```

The AI will automatically trigger Rule Miner and execute this workflow:

1. **Recon** — scan directory structure, identify languages/frameworks/existing linter configs
2. **Full read** — read all source files by priority
3. **Pattern extraction** — tally 8 categories of coding patterns across files
4. **Rule synthesis** — rank by "frequency × deviation from defaults", output ≤1000 words
5. **Preview** — show draft rules for your confirmation
6. **Write** — write to 4 IDE config files (existing content is preserved)

---

## Design Principles

| Dimension | Description |
|-----------|-------------|
| **Core purpose** | Extract project coding DNA, not restate language documentation |
| **Boundaries** | Read-only; never modifies source code, never runs project code, never carries rules across projects |
| **Red lines** | No rules without evidence, no sensitive data exposure, no overwriting user config, strict ≤1000 word limit |
| **Edge cases** | Empty projects get a prompt to add code first; small projects lower thresholds; large projects use stratified sampling; mixed languages get separate sections |

---

## Example Output

```markdown
# Project Rules
> Stack: TypeScript · Express · PostgreSQL. REST API service.

## Naming
- Service files end with `Service.ts`, interfaces with `IService.ts`. Don't get creative — creativity belongs in the logic, not the filenames.
- Route handlers are `handle[Verb][Resource]`. If I see `processData` or `doStuff`, I'm reverting the PR.

## Code Organization
- Every service has a `[name].types.ts` beside it. No types file = the service doesn't exist yet.
- Import `pg` directly in a service and you just became the single point of failure. All DB access goes through `db/`.

## Anti-Patterns
- Use `any` and I will mass-revert your entire PR. `unknown` exists for a reason.
- Touch `process.env` outside `config/env.ts` and you just scattered secrets across the codebase.
```

---

## License

MIT