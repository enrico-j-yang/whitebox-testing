# Python白盒测试指南

本文档包含Python项目��盒测试的特定实现细节和技巧。

## 测试框架

### pytest + pytest-cov
推荐使用pytest作为测试框架，pytest-cov作为覆盖率工具：

```bash
# 安装
pip install pytest pytest-cov pytest-asyncio

# 运行测试并生成覆盖率报告
pytest --cov=app --cov-report=term-missing

# 运行指定测试文件
pytest tests/test_module.py -v --cov=app --cov-report=term-missing
```

### 覆盖率报告解读
```
Name                    Stmts   Miss  Cover   Missing
----------------------------------------------------
app/module.py             100     10    90%   25-30, 45, 67
```
- **Stmts**: 总语句数
- **Miss**: 未覆盖语句数
- **Cover**: 覆盖率
- **Missing**: 未覆盖的行号

## 常见Mock技巧

### 1. 文件读取Mock
使用 `MagicMock` 需要正确设置上下文管理器：

```python
mock_file = MagicMock()
mock_file.__enter__ = Mock(return_value=mock_file)
mock_file.__exit__ = Mock(return_value=False)
mock_file.read = Mock(return_value=test_data)

with patch('builtins.open', return_value=mock_file):
    # 测试代码
```

### 2. AsyncMock陷阱
`AsyncMock()` 对象不能直接用于 `await`，需要包装在异步函数中：

```python
# 错误方式 - 会导致 "object AsyncMock can't be used in 'await' expression"
mock_ws.recv = AsyncMock(return_value="data")

# 正确方式 - 使用异步函数
async def mock_recv():
    return "data"
mock_ws.recv = mock_recv
```

### 3. 时间函数Mock陷阱
使用 `itertools.cycle` 模拟时间可能导致无限循环：

```python
# 错误方式 - 可能导致无限循环
# elapsed 永远是 6， >= 5，但 sleep_time=0，不触发 sleep
with patch('time.time', side_effect=itertools.cycle([1000, 1006])):
    await func()

# 正确方式 - 使用递增序列
time_values = [1000, 1001, 1002, 1003, 1004, 1005, 1006]
call_count = [0]

def mock_time():
    val = time_values[call_count[0] % len(time_values)]
    call_count[0] += 1
    return val

with patch('time.time', side_effect=mock_time):
    await func()
```

### 4. WebSocket Mock
```python
mock_ws = Mock()
mock_ws.send = AsyncMock()
mock_ws.close = AsyncMock()
mock_ws.recv = AsyncMock()

# 或者使用异步函数
async def mock_recv():
    return '{"action": "result", "data": {...}}'
mock_ws.recv = mock_recv
```

### 5. Pydantic配置对象
Pydantic模型可能要求必填字段，确保测试中提供所有必要字段：

```python
# 检查模型必填字段
from pydantic import BaseModel

class ASRConfig(BaseModel):
    websocket_url: str
    app_id: str
    api_key: str      # 必填
    api_secret: str   # 必填
    sample_rate: int = 16000  # 可选，有默认值

# 测试时必须提供所有必填字段
config = ASRConfig(
    websocket_url="ws://test.example.com/ws",
    app_id="test_app",
    api_key="test_key",      # 必填
    api_secret="test_secret" # 必填
)
```

## 测试用例模板

### 异步测试模板
```python
@pytest.mark.asyncio
async def test_{method_name}_{scenario}():
    """测试{方法名}-{场景描述}"""
    from app.module import ClassName

    # 准备配置
    config = Config(
        field1="value1",
        field2="value2",
    )

    # 创建实例
    instance = ClassName(config)

    # 设置Mock
    mock_dep = Mock()
    mock_dep.method = AsyncMock(return_value="result")
    instance.dependency = mock_dep

    # 执行测试
    result = await instance.method_under_test()

    # 验证结果
    assert result == expected_value
```

### 同步测试模板
```python
def test_{function_name}_{scenario}():
    """测试{函数名}-{场景描述}"""
    from app.module import function_name

    # 准备输入
    input_data = {"key": "value"}

    # 执行测试
    result = function_name(input_data)

    # 验证结果
    assert result == expected_value
```

## 路径覆盖示例

### 示例代码
```python
def calculate(a, b, c):
    if a > 0:           # 行1
        x = b + c       # 行2
    else:               # 行3
        x = b - c       # 行4

    if x > 0:          # 行5
        result = x * 2  # 行6
    else:               # 行7
        result = x + 1 # 行8

    return result       # 行9
```

### 独立路径分析
```
路径1: a>0为真 → x>0为真 (覆盖行1,2,5,6,9)
路径2: a>0为真 → x>0为假 (覆盖行1,2,5,7,8,9)
路径3: a>0为假 → x>0为真 (覆盖行1,3,4,5,6,9)
路径4: a>0为假 → x>0为假 (覆盖行1,3,4,5,7,8,9)
```

### 测试用例设计
```python
@pytest.mark.parametrize("a,b,c,expected,path", [
    (1, 1, 1, 4, "路径1"),    # a>0, x=2>0, result=4
    (1, -2, 1, -1, "路径2"),  # a>0, x=-1<=0, result=-1+1=0
    (-1, 3, 1, 4, "路径3"),   # a<=0, x=2>0, result=4
    (-1, 1, 3, -1, "路径4"),  # a<=0, x=-2<=0, result=-2+1=-1
])
def test_calculate_paths(a, b, c, expected, path):
    """测试calculate函数的所有路径"""
    result = calculate(a, b, c)
    assert result == expected, f"{path}测试失败"
```

## 运行测试

```bash
# 运行所有测试
pytest

# 运行指定测试文件
pytest tests/test_module.py -v

# 运行指定测试类
pytest tests/test_module.py::TestClass -v

# 运行指定测试方法
pytest tests/test_module.py::TestClass::test_method -v

# 生成覆盖率报告
pytest --cov=app --cov-report=html
```
