---
name: whitebox-testing
description: "White-box testing expert for code structure analysis, path coverage, cyclomatic complexity, and graph-driven dataflow path coverage.

TRIGGER for: \u767d\u76d2\u6d4b\u8bd5, white-box testing, whitebox testing, code coverage, test coverage, path coverage, statement coverage, branch coverage, decision coverage, dataflow path coverage, def-use coverage, variable dependency coverage, cyclomatic complexity, McCabe complexity, control flow graph, code path analysis.

TRIGGER when user: analyzes code paths, designs comprehensive tests, calculates complexity, improves coverage, asks how many test cases are needed, wants all paths or branches, requests def-use paths, variable dependency graphs, or test-to-dataflow-path mapping, or mentions pytest/Jest/JUnit coverage.

DO NOT trigger for: simple unit-test requests without coverage goals, boundary-only testing, debugging, refactoring, API questions, or algorithm implementation."
---

# whitebox-testing

白盒测试专家助手，帮助你进行系统化的代码结构分析和测试用例设计。

## 核心价值

使用此 skill 可获得：
1. **完整的路径分析**：识别判定节点、列举独立路径、计算覆盖率需求
2. **圈复杂度评估**：McCabe 复杂度计算 + 风险评级 + 重构建议
3. **清晰的流程图**：ASCII 格式控制流程图，直观展示代码结构
4. **专业测试用例**：符合 pytest/Jest/JUnit 规范的测试代码
5. **覆盖率优化建议**：具体命令和迭代策略
6. **Dataflow path coverage**: use def-use paths, variable-dependency graphs, and DOT/SVG/HTML artifacts as explicit coverage targets

## 触发条件

**必须使用此 skill 的场景**：
- 用户提到 "白盒测试"、"white-box testing"、"覆盖率"
- 用户要求分析代码路径、判定节点、控制流
- 用户需要计算圈复杂度（McCabe 复杂度）
- 用户要求设计"全面的"、"系统的"、"完整的"测试用例
- 用户提到 pytest、Jest、JUnit 等测试框架的覆盖率功能

**可选使用场景**：
- 用户要求对某个函数进行单元测试
- 用户询问如何提高测试质量
- 用户提到边界值、边界条件测试

## 执行流程

### 步骤 1: 确认测试目标

使用 AskUserQuestion 确认：
1. **编程语言**：Python / Java / JavaScript/TypeScript / C/C++ / Go / 其他
2. **被测代码文件**：读取源文件,按文件大小列出各个模块
3. **覆盖标准**：语句覆盖 / 判定路径覆盖 / dataflow path coverage

### 步骤 2: 加载语言指南

根据语言选择，加载对应参考文件：
- Python → `references/python.md`
- JavaScript/TypeScript → `references/javascript.md`
- Java → `references/java.md`
- C/C++ → `references/clang.md`
- 其他 → 使用通用原则

When the user asks for dataflow-path coverage, def-use paths, variable-dependency graphs, or DOT/SVG/HTML reports, also read `references/dataflow-path-coverage.md` and use the bundled analyzer documented there. Do not look for `rust-dataflow-analyzer.exe` in the current workspace.

### 步骤 3: 判定节点分析

**识别所有判定节点**：
- `if/else` 条件
- `for/while` 循环条件
- `switch/case` 分支
- `try/catch/finally` 异常处理
- `?:` 三元运算符
- 逻辑运算符组合 (`&&`, `||`, `!`)

**记录每个判定节点**：
| 编号 | 代码位置 | 条件表达式 | 分支数 |
|------|----------|------------|--------|

### 步骤 4: 计算圈复杂度

**McCabe 公式**：
```
V(G) = P + 1
```
其中 P = 判定节点数量

**风险评估表**：
| 复杂度 V(G) | 风险级别 | 测试建议 |
|-------------|----------|----------|
| 1-5 | 低 | 正常测试流程 |
| 6-10 | 中 | 增加边界测试用例 |
| 11-20 | 高 | 重点测试，考虑重构 |
| 21+ | 极高 | **必须重构拆分函数** |

**输出重构建议**（当 V(G) > 10）：
列出可拆分的子函数建议。

### 步骤 5: 绘制控制流程图

**使用简化 ASCII 格式**：

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

**流程图绘制规则**：
1. 开始/结束节点用方框
2. 判定节点用菱形（或注明 `条件?`）
3. 处理节点用方框
4. 真分支在左/上，假分支在右/下
5. 循环用回箭头表示
6. 每个判定标注编号（D1, D2...）

### 步骤 6: 列举独立路径

**路径格式**：
```
路径 P1: 开始 → D1-真 → D2-真 → 处理A → 结束
路径 P2: 开始 → D1-真 → D2-假 → 处理B → 结束
路径 P3: 开始 → D1-假 → 处理C → 结束
...
```

**循环路径规则**：
- 循环只需覆盖 **2次执行** 即可满足判定路径要求
- 测试：循环0次、循环1次、循环2次

### 步骤 7: 设计测试用例

**测试用例表格式**：

| 用例ID | 覆盖路径 | 输入数据 | 预期输出 | 说明 |
|--------|----------|----------|----------|------|

**必须包含的测试类型**：
1. **基本路径测试**：覆盖所有独立路径
2. **边界值测试**：判定条件的边界值（如 `x > 0` 测试 x=0, x=-1, x=1）
3. **异常路径测试**：异常处理分支
4. **组合测试**：多个条件组合的场景

### 步骤 8: 生成测试代码

按照语言指南生成规范的测试代码：
- Python: pytest 格式，`test_` 函数命名，`assert` 断言
- JavaScript: Jest 格式，`describe/it` 结构，`expect` 断言
- Java: JUnit 格式，`@Test` 注解

**测试代码模板**（Python 示例）：
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


### 步骤 9: 运行测试并修复源代码问题

**运行测试命令**：

| 语言 | 命令 |
|------|------|
| Python | `pytest` |
| JavaScript | `npm test` |
| Java | `mvn test` |

### 步骤 10: 运行测试并收集覆盖率

**覆盖率命令**：

| 语言 | 命令 |
|------|------|
| Python | `pytest --cov={module} --cov-report=term-missing` |
| JavaScript | `npm test -- --coverage` |
| Java | `mvn test jacoco:report` |
| C/C++ | `gcov {file.c}` |

**覆盖率报告解读**：
```
Name                    Stmts   Miss  Cover   Missing
----------------------------------------------------
app/module.py             100     10    90%   25-30, 45
```

- **Stmts**: 总语句数
- **Miss**: 未覆盖语句数
- **Cover**: 覆盖率百分比
- **Missing**: 未覆盖的行号

**Dataflow-path coverage standard**:
1. Set `$analyzer = "$HOME\.agents\skills\whitebox-testing\scripts\rust-dataflow-analyzer.exe"` and run `& $analyzer analyze --lang python --input <src_dir> --out <report_dir>`.
2. Treat def-use paths and variable-dependency paths as coverage targets.
3. Build `test -> paths` and `path -> tests` mappings, and prioritize uncovered plus split paths.
4. Document dynamic behavior, external dependencies, unreachable code, and conservative-analysis false positives.

### 步骤 11: 分析未覆盖代码

**未覆盖原因分类**：

| 原因类型 | 说明 | 处理方式 |
|----------|------|----------|
| 测试缺失 | 有路径但未设计测试 | **补充测试用例** |
| 条件限制 | 某些条件不可达 | 文档化说明 |
| 死代码 | 永远不执行 | **考虑删除** |
| Mock限制 | 外部依赖被绕过 | 集成测试覆盖 |
| 异常场景 | 极端错误处理 | 评估必要性 |

### 步骤 12: 迭代优化

如果覆盖率未达 100%：
1. 分析 Missing 行号对应的代码
2. 设计新测试用例覆盖缺失路径
3. 返回步骤 7

**目标覆盖率建议**：

| 项目类型 | 语句覆盖 | 分支覆盖 |
|----------|----------|----------|
| 关键业务 | 100% | 90%+ |
| 一般功能 | 90%+ | 80%+ |
| 工具函数 | 100% | 100% |
| UI组件 | 80%+ | 70%+ |

---

## 输出格式要求

完成分析后，必须输出以下文件：

1. **`path_analysis.md`** - 路径分析报告
   - 判定节点表
   - 圈复杂度计算
   - 流程图
   - 独立路径列表
   - 未覆盖代码分析

2. **`test_{module}.py`** 或 **`{module}.test.js`** - 测试代码文件
   - 完整的可执行测试用例
   - 参数化边界测试
   - 清晰的中文注释

3. **`dataflow_path_coverage.md`** and **`path_mapping.csv`/`path_mapping.json`** - required when using dataflow-path coverage
   - analyzer command, source directory, and output directory
   - DOT/SVG/HTML report locations
   - total path counts and covered/uncovered/split statistics
   - per-path `path_id`, root function, gap reason, and linked tests

---

## 注意事项

1. **判定路径覆盖是中等难度标准**，比语句覆盖更全面
2. **圈复杂度 > 10 必须给出重构建议**
3. **循环只需覆盖 2 次**，但需测试 0 次和 1 次边界
4. **异常处理路径必须覆盖**
5. **不可达代码需要记录并报告**
7. **Dataflow path coverage does not replace statement/branch coverage**; use it to verify def-use and dependency propagation.
8. **Prefer the bundled analyzer documented in `references/dataflow-path-coverage.md` for Python dataflow work** instead of rebuilding ad-hoc def-use extractors.
6. **测试数据必须与函数逻辑一致**，预期值要准确计算

---

## 语言特定指南

已创建的语言指南：
- `references/python.md` - pytest + pytest-cov + Mock
- `references/javascript.md` - Jest + 覆盖率 + Mock策略
- `references/java.md` - JUnit + Mockito + JaCoCo
- `references/dataflow-path-coverage.md` - dataflow-path coverage workflow with the bundled Rust analyzer
- `references/clang.md` - Unity + FFF Mock + lcov

---


## Additional resources

- `references/dataflow-path-coverage.md` - workflow for dataflow path coverage, including the bundled analyzer location and invocation pattern
- `scripts/rust-dataflow-analyzer.exe` - bundled Windows static-analysis binary under `$HOME\.agents\skills\whitebox-testing\scripts\`
## 快速模式

当用户只需**快速生成测试用例**（不需要完整分析）时：

1. 直接读取源代码
2. 识别主要分支（if/else）
3. 为每个分支设计一个测试用例
4. 添加边界值测试
5. 输出测试代码文件

跳过流程图和复杂度计算，直接生成可用测试。
