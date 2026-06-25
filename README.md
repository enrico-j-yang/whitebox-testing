# whitebox-testing

[![npx skills add](https://img.shields.io/badge/npx%20skills%20add-enrico--j--yang%2Fwhitebox--testing-blue?logo=npm)](https://skills.sh/b/enrico-j-yang/whitebox-testing)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-orange?logo=anthropic)](https://github.com/enrico-j-yang/whitebox-testing)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

`whitebox-testing` 是一个面向白盒测试分析的 AI skill，用来帮助代理在阅读源代码后，围绕控制流、数据流和覆盖标准系统化地设计测试。

`whitebox-testing` is an AI skill for white-box testing analysis. It helps an agent read source code and design tests systematically around control flow, data flow, and coverage criteria.

## 安装 / Installation

选择任一方式安装（推荐第一种，跨 agent 通用）：

Pick any of the methods below (the first is recommended — works across agents):

### 1. 通过 `npx skills add` 安装 / Install via `npx skills add`

```bash
npx skills add enrico-j-yang/whitebox-testing
```

支持 Claude Code、Cursor、Cline、Codex、Continue、Gemini CLI 等 70+ agent —— CLI 会自动检测当前环境并装到正确路径。

Supports Claude Code, Cursor, Cline, Codex, Continue, Gemini CLI, and 70+ other agents — the CLI auto-detects your environment and installs to the right location.

### 2. 通过 Claude Code 插件市场安装 / Install via Claude Code plugin marketplace

```text
/plugin marketplace add enrico-j-yang/whitebox-testing
/plugin install whitebox-testing@enrico-skills
```

### 3. 直接 git clone / Plain git clone

```bash
# Linux / macOS
git clone https://github.com/enrico-j-yang/whitebox-testing \
  ~/.claude/skills/whitebox-testing

# Windows PowerShell
git clone https://github.com/enrico-j-yang/whitebox-testing `
  "$HOME\.claude\skills\whitebox-testing"
```

安装完成后，在 Claude Code 中执行 `/reload-skills` 即可激活。

After installation, run `/reload-skills` in Claude Code to activate.

它的核心目标不是“多写几个测试”，而是把下面这些工作串成一套明确流程：

Its goal is not merely to "write more tests," but to connect the following activities into one clear workflow:

- 识别判定节点、分支和循环；
- 计算圈复杂度（McCabe Complexity）；
- 绘制控制流程图并列举独立路径；
- 按覆盖标准设计测试用例；
- 在需要时进一步分析 def-use 路径和变量依赖路径。

- Identify decision points, branches, and loops.
- Calculate cyclomatic complexity (McCabe Complexity).
- Draw control-flow diagrams and enumerate independent paths.
- Design test cases against explicit coverage criteria.
- Analyze def-use paths and variable-dependency paths when needed.

目前该 skill 内置了 Python、JavaScript/TypeScript、Java、C/C++ 的参考指南，并额外提供了一个用于 dataflow path coverage 的静态分析工具。

The skill currently includes reference guides for Python, JavaScript/TypeScript, Java, and C/C++, plus a static-analysis tool for dataflow path coverage.

## 白盒测试的基本概念 / White-Box Testing Basics

白盒测试（White-box Testing）是基于程序内部结构开展的测试方法。测试者不仅关心“输入后输出是否正确”，还关心代码内部究竟经过了哪些语句、哪些分支、哪些路径，以及变量在不同路径上的定义和使用是否合理。

White-box testing is a testing approach based on the internal structure of a program. The tester cares not only about whether the output is correct for a given input, but also about which statements, branches, and paths are executed, and whether variables are defined and used correctly across those paths.

和黑盒测试相比，白盒测试更强调：

Compared with black-box testing, white-box testing places more emphasis on:

- **控制流**：代码是如何一步步执行的，条件、循环、异常分支是否都被走到；
- **数据流**：变量在哪里被定义、如何被传递、最终在哪里被使用；
- **结构复杂度**：函数或模块是否因为分支过多而难以测试、难以维护；
- **覆盖度量**：测试是否真正覆盖了关键逻辑，而不是只运行到了“表面代码”。

- **Control flow**: how the code executes step by step, and whether conditions, loops, and exception branches are actually exercised.
- **Data flow**: where variables are defined, how they propagate, and where they are eventually used.
- **Structural complexity**: whether a function or module is hard to test and maintain because it contains too many branches.
- **Coverage measurement**: whether tests truly exercise critical logic instead of only touching surface-level code.

白盒测试特别适合以下场景：

White-box testing is especially useful in the following situations:

- 需要提高语句、分支或路径覆盖率；
- 需要分析复杂函数的执行路径；
- 需要找出遗漏分支、异常路径或不可达代码；
- 需要验证关键变量的定义-使用链是否被测试触达。

- You need to improve statement, branch, or path coverage.
- You need to analyze the execution paths of complex functions.
- You need to find missing branches, exception paths, or unreachable code.
- You need to verify that critical definition-use chains are exercised by tests.

需要注意的是，白盒测试并不替代黑盒测试。白盒测试擅长验证“代码结构是否被充分检查”，黑盒测试则更擅长验证“需求和业务行为是否正确”。

White-box testing does not replace black-box testing. White-box testing is good at verifying whether code structure has been exercised thoroughly, while black-box testing is better at validating requirements and business behavior.

## 本 skill 使用的三种覆盖标准 / The Three Coverage Criteria Used by This Skill

这个 skill 的工作流围绕三种覆盖标准展开，它们从浅到深形成一个递进关系：先看语句，再看控制路径，最后看数据传播。

This skill is organized around three coverage criteria that build on each other: first statements, then control paths, and finally data propagation.

### 1. 语句覆盖 / Statement Coverage

语句覆盖（Statement Coverage）要求每条可执行语句至少被执行一次。

Statement coverage requires every executable statement to run at least once.

- **它回答的问题**：这段代码里的每一行核心逻辑，是否至少运行过一次？
- **优点**：门槛最低、最直观，适合快速发现完全未被触达的代码；
- **局限**：即使覆盖率很高，也可能遗漏条件真假分支、异常路径和变量传播问题。

- **Question it answers**: Has every important executable line in this code run at least once?
- **Strength**: It is the most basic and intuitive metric, so it quickly reveals code that was never reached at all.
- **Limitation**: Even a high statement-coverage number can still miss true/false condition outcomes, exception paths, and data-propagation issues.

例子：如果一个 `if` 语句只走了 `true` 分支，那么语句覆盖可能已经不低，但 `false` 分支仍然没有真正被验证。

Example: if an `if` statement only takes the `true` branch, statement coverage may still look decent, but the `false` branch has not actually been validated.

### 2. 判定路径覆盖 / Decision-Path Coverage

本 skill 中的“判定路径覆盖”是以判定节点和独立执行路径为中心的覆盖方式。它不只看某一行代码是否被执行，还关注条件判断的真假分支是否出现，以及这些判定组合出来的代表性路径是否被覆盖。

In this skill, "decision-path coverage" is centered on decision points and independent execution paths. It does not only ask whether a line was executed; it also checks whether true and false outcomes of conditions appear, and whether representative paths formed by those decisions are covered.

- **它回答的问题**：关键条件的不同结果，是否都形成了被测试的执行路径？
- **关注重点**：`if/else`、循环条件、`switch/case`、异常分支、复合逻辑表达式；
- **优点**：比语句覆盖更能暴露遗漏分支、短路逻辑和边界路径；
- **局限**：路径数量会随着复杂度快速增长，因此通常需要结合圈复杂度和独立路径分析来控制测试规模。

- **Question it answers**: Do different outcomes of key conditions lead to execution paths that are actually tested?
- **Focus areas**: `if/else`, loop conditions, `switch/case`, exception branches, and compound logical expressions.
- **Strength**: It exposes missing branches, short-circuit logic, and edge paths better than statement coverage.
- **Limitation**: The number of paths grows rapidly with complexity, so you usually need cyclomatic complexity and independent-path analysis to keep the test scope manageable.

这个 skill 会围绕判定路径覆盖做几件事：

For decision-path coverage, this skill helps with the following activities:

- 识别所有判定节点；
- 计算圈复杂度；
- 用 ASCII 流程图表达控制流；
- 列举独立路径并为每条路径设计测试用例。

- Identify all decision points.
- Calculate cyclomatic complexity.
- Express control flow with ASCII diagrams.
- Enumerate independent paths and design a test case for each path.

### 3. 数据流路径覆盖 / Dataflow Path Coverage

Dataflow path coverage（数据流路径覆盖）把重点放在变量“定义后如何流向使用点”这件事上。它关注的不只是代码走没走到，还关注变量值是否沿着重要路径正确传播。

Dataflow path coverage focuses on how a variable flows from its definition to its use sites. It is not only about whether code was executed, but also about whether values propagate correctly along important paths.

- **它回答的问题**：某个变量的定义，是否沿着有效路径到达了对应使用点？
- **核心对象**：def-use 路径、变量依赖路径、调用参数传播、同一行内的定义/使用传播；
- **优点**：适合发现“控制流看似覆盖了，但关键数据其实没有被真正验证”的问题；
- **局限**：需要更强的静态分析支持，对动态语言特性也要保守解读。

- **Question it answers**: Does a variable definition reach the corresponding use site through a valid path?
- **Core targets**: def-use paths, variable-dependency paths, call-argument propagation, and same-line def/use propagation.
- **Strength**: It is useful for finding cases where control flow looks covered but critical data has not actually been validated.
- **Limitation**: It requires stronger static-analysis support and must be interpreted conservatively for dynamic-language features.

在这个 skill 里，dataflow path coverage 通过内置分析器完成，相关入口位于：

In this skill, dataflow path coverage is supported by a bundled analyzer located at:

`scripts/rust-dataflow-analyzer.exe`

当用户要求 def-use 路径、变量依赖图、DOT/SVG/HTML 报告或测试到路径的映射时，应同时参考：

When the user asks for def-use paths, variable-dependency graphs, DOT/SVG/HTML reports, or test-to-path mappings, also consult:

`references/dataflow-path-coverage.md`

## 三种覆盖标准的关系 / How the Three Coverage Criteria Relate

可以把这三种覆盖标准理解为三个递进层次：

You can think of these three coverage criteria as three increasing levels of depth:

| 覆盖标准 | 关注点 | 适合回答的问题 |
| --- | --- | --- |
| 语句覆盖 | 语句是否执行过 | 有没有完全没跑到的代码？ |
| 判定路径覆盖 | 分支和路径是否被走到 | 条件真假、循环和异常路径是否都被验证？ |
| Dataflow path coverage | 变量定义和使用之间的传播 | 关键数据是否沿着正确路径流向真正的使用点？ |

| Coverage Criterion | Focus | Best Question to Answer |
| --- | --- | --- |
| Statement coverage | Whether statements executed | Is there code that was never reached at all? |
| Decision-path coverage | Whether branches and paths were exercised | Were true/false outcomes, loops, and exception paths actually validated? |
| Dataflow path coverage | Propagation between variable definitions and uses | Does critical data reach real use sites along the correct paths? |

一个常见误区是只看语句覆盖率数字。高语句覆盖率并不自动意味着高质量测试；很多与条件组合、异常处理、变量依赖相关的问题，只有进入更深层的路径覆盖和数据流覆盖后才会显现。

A common mistake is to look only at the statement-coverage number. High statement coverage does not automatically mean high-quality testing; many issues involving condition combinations, exception handling, and variable dependencies only become visible with deeper path and dataflow coverage.

## 这个 skill 的典型输出 / Typical Outputs of This Skill

根据任务类型不同，这个 skill 通常会生成或建议生成以下产物：

Depending on the task, this skill typically generates or recommends the following outputs:

- `path_analysis.md`：路径分析报告，包含判定节点、圈复杂度、流程图、独立路径和遗漏路径分析；
- `test_{module}.py` 或 `{module}.test.js`：按路径设计的测试代码；
- `dataflow_path_coverage.md`：数据流路径覆盖报告；
- `path_mapping.csv` 或 `path_mapping.json`：测试与路径之间的映射；
- HTML、DOT、SVG、JSON 等分析产物：用于可视化 def-use 和变量依赖关系。

- `path_analysis.md`: a path-analysis report containing decision points, cyclomatic complexity, flow diagrams, independent paths, and uncovered-path analysis.
- `test_{module}.py` or `{module}.test.js`: test code designed against explicit paths.
- `dataflow_path_coverage.md`: a report for dataflow path coverage.
- `path_mapping.csv` or `path_mapping.json`: mappings between tests and paths.
- HTML, DOT, SVG, and JSON artifacts for visualizing def-use and variable-dependency relationships.

## 这个 skill 的目录结构 / Directory Structure of This Skill

当前目录结构如下：

The current directory structure is:

```text
whitebox-testing/
|-- README.md
|-- SKILL.md
|-- LICENSE
|-- assets/
|   |-- chroma_db/
|   |   `-- .gitkeep
|   |-- logs/
|   |   `-- .gitkeep
|   `-- references/
|       |-- context7/
|       |   `-- .gitkeep
|       `-- user/
|           `-- .gitkeep
|-- references/
|   |-- python.md
|   |-- javascript.md
|   |-- java.md
|   |-- clang.md
|   `-- dataflow-path-coverage.md
`-- scripts/
    `-- rust-dataflow-analyzer.exe
```

各部分职责可以这样理解：

The main responsibilities of each part are:

- `SKILL.md`：主技能说明，定义触发条件、执行流程、输出要求和注意事项；
- `README.md`：面向维护者和使用者的总览文档，帮助快速理解这个 skill 在做什么；
- `references/`：按语言和专题拆分的参考资料；
- `references/dataflow-path-coverage.md`：数据流路径覆盖的专项工作流说明；
- `scripts/`：可执行辅助工具，目前包含 Rust 编写的数据流分析器；
- `assets/`：为缓存、日志和外部参考资料预留的资源目录。

- `SKILL.md`: the main skill definition, including trigger conditions, workflow, output requirements, and notes.
- `README.md`: an overview document for maintainers and users, helping them understand what this skill does at a glance.
- `references/`: reference materials split by language and topic.
- `references/dataflow-path-coverage.md`: the dedicated workflow guide for dataflow path coverage.
- `scripts/`: helper executables, currently including the Rust-based dataflow analyzer.
- `assets/`: reserved directories for caches, logs, and external reference material.

## 推荐使用方式 / Recommended Way to Use This Skill

如果你准备使用这个 skill 处理真实代码，通常可以按下面的顺序理解它：

If you plan to use this skill on real code, a practical reading order is:

1. 先在 `SKILL.md` 中确认适用场景与标准流程；
2. 根据语言选择 `references/` 下对应指南；
3. 如果目标是提升结构覆盖，先从语句覆盖和判定路径覆盖入手；
4. 如果目标是验证变量传播、def-use 路径或生成依赖图，再进入 dataflow path coverage；
5. 根据输出产物补齐测试文件、路径报告和覆盖缺口说明。

1. Start with `SKILL.md` to confirm the applicable scenarios and standard workflow.
2. Choose the matching guide under `references/` for the target language.
3. If the goal is structural coverage, begin with statement coverage and decision-path coverage.
4. If the goal is validating value propagation, def-use paths, or dependency graphs, move on to dataflow path coverage.
5. Use the expected outputs to complete test files, path reports, and gap explanations.

## 适用边界 / Scope and Limits

这个 skill 更适合“结构化分析 + 覆盖驱动的测试设计”，不适合把所有测试问题都简化成覆盖率问题。使用时建议记住两点：

This skill is best suited for structured analysis and coverage-driven test design. It is not meant to reduce every testing problem to a coverage metric. Two reminders are especially important:

- 覆盖率是信号，不是最终目标；
- dataflow path coverage 也不能替代业务层面的黑盒验证。

- Coverage is a signal, not the final goal.
- Dataflow path coverage does not replace business-level black-box validation.

如果把三种覆盖标准配合起来使用，这个 skill 最擅长的事情就是：把“我大概写了一些测试”提升成“我知道这些测试具体覆盖了哪些语句、哪些路径、哪些变量传播关系”。

When the three coverage criteria are used together, this skill is most effective at turning "I wrote some tests" into "I know exactly which statements, paths, and variable-propagation relationships those tests cover."
