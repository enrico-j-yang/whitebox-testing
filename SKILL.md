---
name: whitebox-testing
description: >-
  Use when the task involves white-box testing, code-structure analysis,
  path coverage, or cyclomatic complexity.


  TRIGGER for: 白盒测试, white-box testing, whitebox testing, 覆盖率, 代码覆盖率,
  路径覆盖, 路径分析, 语句覆盖, 分支覆盖, 判定覆盖, 判定路径覆盖,
  圈复杂度, McCabe complexity,
  McCabe复杂度, 控制流图, 控制流分析, 代码路径分析.


  TRIGGER when user: analyzes code paths, designs comprehensive tests,
  calculates complexity, improves coverage, asks how many test cases are needed,
  wants all paths or branches, or mentions pytest/Jest/JUnit coverage.


  DO NOT trigger for: simple unit-test requests without coverage goals,
  boundary-only testing, debugging, refactoring, API questions,
  or algorithm implementation.
---

# whitebox-testing

白盒测试专家助手，帮助你进行系统化的代码结构分析和测试用例设计。

White-box testing skill for systematic code-structure analysis and coverage-driven test design.

本文档采用混合双语格式：核心说明使用中英双语，命令、模板和示例尽量保留单份，兼顾可读性与上下文效率。

This document uses a mixed bilingual format: core guidance is bilingual, while commands, templates, and examples stay single-sourced to keep the skill readable and efficient.

## 核心价值 / Core Value

使用此 skill 可获得：

Using this skill helps you produce:

1. **完整的路径分析 / Complete path analysis**：识别判定节点、列举独立路径、计算覆盖率需求
2. **圈复杂度评估 / Cyclomatic-complexity assessment**：McCabe 复杂度计算 + 风险评级 + 重构建议
3. **清晰的流程图 / Clear flow diagrams**：ASCII 格式控制流程图，直观展示代码结构
4. **专业测试用例 / Professional test cases**：符合 pytest/Jest/JUnit 规范的测试代码
5. **覆盖率优化建议 / Coverage-improvement guidance**：具体命令和迭代策略

## 触发条件 / When to Trigger

**必须使用此 skill 的场景 / Must use this skill when:**

- 用户提到 "白盒测试"、"white-box testing"、"覆盖率" / The user explicitly mentions white-box testing or coverage goals.
- 用户要求分析代码路径、判定节点、控制流 / The user asks to analyze code paths, decision points, or control flow.
- 用户需要计算圈复杂度（McCabe 复杂度） / The user wants cyclomatic complexity or McCabe complexity.
- 用户要求设计"全面的"、"系统的"、"完整的"测试用例 / The user asks for comprehensive or systematic structural test design.
- 用户提到 pytest、Jest、JUnit 等测试框架的覆盖率功能 / The user refers to framework-level coverage features such as pytest, Jest, or JUnit coverage.

**可选使用场景 / Optional use cases:**

- 用户要求对某个函数进行单元测试 / The user asks for unit tests for a specific function.
- 用户询问如何提高测试质量 / The user asks how to improve test quality.
- 用户提到边界值、边界条件测试 / The user mentions boundary-value or boundary-condition testing.

## 执行流程 / Workflow

### 步骤 1：确认测试目标 / Step 1: Confirm the Testing Target

使用 AskUserQuestion 或等效提问方式确认：

Use AskUserQuestion or an equivalent user prompt to confirm:

1. **编程语言 / Language**：Python / Java / JavaScript/TypeScript / C/C++ / Go / 其他
2. **被测代码文件 / Target source files**：读取源文件，按文件大小列出各个模块
3. **覆盖标准 / Coverage criterion**：语句覆盖 / 判定路径覆盖

### 步骤 2：加载语言指南 / Step 2: Load the Language Guide

根据语言选择对应参考文件：

Choose the matching reference file by language:

- Python -> `references/python.md`
- JavaScript/TypeScript -> `references/javascript.md`
- Java -> `references/java.md`
- C/C++ -> `references/clang.md`
- 其他 / Other -> 使用通用原则 / use the general principles in this skill

### 步骤 3：判定节点分析 / Step 3: Analyze Decision Points

**识别所有判定节点 / Identify all decision points:**

- `if/else` 条件 / conditions
- `for/while` 循环条件 / loop conditions
- `switch/case` 分支 / branches
- `try/catch/finally` 异常处理 / exception handling
- `?:` 三元运算符 / ternary operators
- 逻辑运算符组合 / logical combinations (`&&`, `||`, `!`)

**记录每个判定节点 / Record each decision point:**

| 编号 / ID | 代码位置 / Location | 条件表达式 / Condition | 分支数 / Branches |
|-----------|----------------------|------------------------|-------------------|

### 步骤 4：计算圈复杂度 / Step 4: Calculate Cyclomatic Complexity

**McCabe 公式 / McCabe formula:**

```
V(G) = P + 1
```

其中 `P` = 判定节点数量。 `P` is the number of decision points.

**风险评估表 / Risk table:**

| 复杂度 V(G) / Complexity | 风险级别 / Risk | 测试建议 / Testing guidance |
|--------------------------|-----------------|-----------------------------|
| 1-5 | 低 / Low | 正常测试流程 / Normal testing flow |
| 6-10 | 中 / Medium | 增加边界测试用例 / Add boundary-focused cases |
| 11-20 | 高 / High | 重点测试，考虑重构 / Intensive testing, consider refactoring |
| 21+ | 极高 / Very high | **必须重构拆分函数 / Refactor and split the function** |

**输出重构建议 / Output refactoring guidance**（当 `V(G) > 10`）：

列出可拆分的子函数建议。

List candidate helper functions or sub-functions that can reduce complexity.

### 步骤 5：绘制控制流程图 / Step 5: Draw the Control-Flow Diagram

**使用简化 ASCII 格式 / Use a simplified ASCII layout:**

```
        ┌───────────┐
        │   开始    │
        └─────┬─────┘
              │
        ┌─────▼─────┐
        │  条件D1?  │
        └─────┬─────┘
              │
    ┌─────────┴─────────┐
    │True               │False
    │                   │
┌───▼───┐         ┌─────▼─────┐
│处理A  │         │  处理B    │
└───┬───┘         └─────┬─────┘
    │                   │
    └─────────┬─────────┘
              │
        ┌─────▼─────┐
        │   结束    │
        └───────────┘
```

**流程图绘制规则 / Diagram rules:**

1. 开始/结束节点用方框 / Use boxes for start and end nodes.
2. 判定节点用菱形（或注明 `条件?`） / Use diamonds for decisions, or clearly mark them as conditions.
3. 处理节点用方框 / Use boxes for processing nodes.
4. 真分支在左/上，假分支在右/下 / Place the true branch on the left or top, false on the right or bottom.
5. 循环用回箭头表示 / Show loops with return arrows.
6. 每个判定标注编号（D1, D2...） / Label each decision point (D1, D2, ...).

### 步骤 6：列举独立路径 / Step 6: Enumerate Independent Paths

**路径格式 / Path format:**

```
路径 P1: 开始 -> D1-真 -> D2-真 -> 处理A -> 结束
路径 P2: 开始 -> D1-真 -> D2-假 -> 处理B -> 结束
路径 P3: 开始 -> D1-假 -> 处理C -> 结束
...
```

**循环路径规则 / Loop-path rules:**

- 循环只需覆盖 **2 次执行** 即可满足判定路径要求 / Two loop executions are usually enough for decision-path coverage.
- 测试：循环 0 次、循环 1 次、循环 2 次 / Test the loop with 0, 1, and 2 executions.

### 步骤 7：设计测试用例 / Step 7: Design Test Cases

**测试用例表格式 / Test-case table format:**

| 用例 ID / Case ID | 覆盖路径 / Covered Path | 输入数据 / Inputs | 预期输出 / Expected Output | 说明 / Notes |
|-------------------|-------------------------|-------------------|----------------------------|--------------|

**必须包含的测试类型 / Required test types:**

1. **基本路径测试 / Basic path tests**：覆盖所有独立路径
2. **边界值测试 / Boundary-value tests**：判定条件的边界值（如 `x > 0` 测试 `x=0, x=-1, x=1`）
3. **异常路径测试 / Exception-path tests**：异常处理分支
4. **组合测试 / Combination tests**：多个条件组合的场景

### 步骤 8：生成测试代码 / Step 8: Generate Test Code

按照语言指南生成规范的测试代码：

Generate test code according to the language guide:

- Python: pytest 格式，`test_` 函数命名，`assert` 断言
- JavaScript: Jest 格式，`describe/it` 结构，`expect` 断言
- Java: JUnit 格式，`@Test` 注解

**测试代码模板 / Test-code template（Python 示例 / Python example）:**

```python
"""
白盒测试：{function_name} 函数判定路径覆盖测试
圈复杂度：V(G) = {complexity}
"""

import pytest
from {module} import {function_name}


class Test{FunctionName}:
    """{function_name} 白盒测试类"""

    # 路径 P1: {path_description}
    def test_path1_{scenario}():
        """{description}"""
        result = {function_name}({inputs})
        assert result == {expected}

    # ... 更多测试用例


@pytest.mark.parametrize("input,expected", [
    # 边界值和组合测试
])
def test_{function_name}_boundaries(input, expected):
    """参数化边界测试"""
    ...
```

### 步骤 9：运行测试并修复源代码问题 / Step 9: Run Tests and Fix Source Issues

**运行测试命令 / Test commands:**

| 语言 / Language | 命令 / Command |
|-----------------|----------------|
| Python | `pytest` |
| JavaScript | `npm test` |
| Java | `mvn test` |

### 步骤 10：收集覆盖率 / Step 10: Collect Coverage

**覆盖率命令 / Coverage commands:**

| 语言 / Language | 命令 / Command |
|-----------------|----------------|
| Python | `pytest --cov={module} --cov-report=term-missing` |
| JavaScript | `npm test -- --coverage` |
| Java | `mvn test jacoco:report` |
| C/C++ | `gcov {file.c}` |

**覆盖率报告解读 / Coverage report interpretation:**

```
Name                    Stmts   Miss  Cover   Missing
----------------------------------------------------
app/module.py             100     10    90%   25-30, 45
```

- **Stmts / Statements**：总语句数
- **Miss / Missing statements**：未覆盖语句数
- **Cover / Coverage**：覆盖率百分比
- **Missing / Missing lines**：未覆盖的行号

### 步骤 11：分析未覆盖代码 / Step 11: Analyze Uncovered Code

**未覆盖原因分类 / Categories of uncovered-code causes:**

| 原因类型 / Cause | 说明 / Description | 处理方式 / Action |
|------------------|--------------------|-------------------|
| 测试缺失 / Missing tests | 有路径但未设计测试 / The path exists but no test covers it | **补充测试用例 / Add tests** |
| 条件限制 / Condition limits | 某些条件不可达 / Some conditions are unreachable | 文档化说明 / Document the reason |
| 死代码 / Dead code | 永远不执行 / Never executes | **考虑删除 / Consider removing it** |
| Mock 限制 / Mock limits | 外部依赖被绕过 / External dependencies were bypassed | 集成测试覆盖 / Cover with integration tests |
| 异常场景 / Exceptional scenarios | 极端错误处理 / Extreme error-handling paths | 评估必要性 / Evaluate necessity |

### 步骤 12：确认目标并迭代优化 / Step 12: Confirm Target and Iterate

**先给出目标覆盖率建议 / First present the suggested target coverage:**

根据被测代码的类型，先向用户展示以下建议表：

Based on the type of the code under test, first show the user the following recommendation table:

| 项目类型 / Project type | 语句覆盖 / Statement | 分支覆盖 / Branch |
|-------------------------|----------------------|-------------------|
| 关键业务 / Critical business logic | 100% | 90%+ |
| 一般功能 / General features | 90%+ | 80%+ |
| 工具函数 / Utility functions | 100% | 100% |
| UI 组件 / UI components | 80%+ | 70%+ |

**然后追问用户需要达到的覆盖率 / Then ask the user what coverage they want to reach:**

使用 AskUserQuestion 或等效提问方式询问用户的目标覆盖率（语句覆盖率、分支覆盖率），并记录用户的确认值作为本次测试目标。

Use AskUserQuestion or an equivalent prompt to ask the user for the target coverage values (statement and branch coverage). Record the confirmed values as the goal for this round.

**迭代直至达到目标 / Iterate until the target is reached:**

如果当前覆盖率未达到用户确认的目标：

If the current coverage is below the user-confirmed target:

1. 分析 `Missing` 行号对应的代码 / Analyze the code behind the `Missing` lines.
2. 设计新测试用例覆盖缺失路径 / Design new tests for the uncovered paths.
3. **重复执行步骤 7 到步骤 11 / Repeat Steps 7 through 11**（重新设计用例 → 生成测试代码 → 运行测试 → 收集覆盖率 → 分析未覆盖代码）。
4. 持续迭代，直到覆盖率达到用户设定的目标为止 / Continue iterating until the coverage meets the user-defined target.

每一轮迭代完成后向用户汇报当前覆盖率与目标的差距，并说明本轮新增的测试用例与覆盖到的路径。

After each iteration, report the gap between current coverage and the target, and describe the new test cases added and the paths they cover.

---

## 输出格式要求 / Required Outputs

完成分析后，必须输出以下文件：

After the analysis, produce the following files:

1. **`path_analysis.md`** - 路径分析报告 / Path-analysis report
   - 判定节点表 / Decision-point table
   - 圈复杂度计算 / Cyclomatic-complexity calculation
   - 流程图 / Flow diagram
   - 独立路径列表 / Independent-path list
   - 未覆盖代码分析 / Uncovered-code analysis

2. **`test_{module}.py`** 或 **`{module}.test.js`** - 测试代码文件 / Test-code file
   - 完整的可执行测试用例 / Complete executable test cases
   - 参数化边界测试 / Parameterized boundary tests
   - 清晰的中文注释，必要时可补英文说明 / Clear Chinese comments, with English notes when useful

---

## 注意事项 / Notes

1. **判定路径覆盖是中等难度标准 / Decision-path coverage is a medium-depth standard**，比语句覆盖更全面
2. **圈复杂度 > 10 必须给出重构建议 / If cyclomatic complexity is above 10, provide refactoring advice**
3. **循环只需覆盖 2 次 / Two loop executions are usually enough**，但需测试 0 次和 1 次边界
4. **异常处理路径必须覆盖 / Exception-handling paths must be covered**
5. **不可达代码需要记录并报告 / Unreachable code must be documented and reported**
6. **测试数据必须与函数逻辑一致 / Test data must match the real function logic**，预期值要准确计算

---

## 语言特定指南 / Language-Specific Guides

已创建的语言指南：

Available language guides:

- `references/python.md` - pytest + pytest-cov + Mock
- `references/javascript.md` - Jest + 覆盖率 / coverage + Mock 策略 / mock strategy
- `references/java.md` - JUnit + Mockito + JaCoCo
- `references/clang.md` - Unity + FFF Mock + lcov

---

## 快速模式 / Fast Mode

当用户只需**快速生成测试用例**（不需要完整分析）时：

When the user only needs **quick test generation** instead of a full analysis:

1. 直接读取源代码 / Read the source code directly.
2. 识别主要分支（if/else） / Identify the main branches (`if/else`).
3. 为每个分支设计一个测试用例 / Design one test case for each branch.
4. 添加边界值测试 / Add boundary-value tests.
5. 输出测试代码文件 / Output the test file.

跳过流程图和复杂度计算，直接生成可用测试。

Skip the flowchart and complexity calculation, and go straight to generating practical tests.
