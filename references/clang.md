# C/C++ 白盒测试指南

## 测试框架和工具

### Unity 测试框架

Unity 是 C 语言最流行的单元测试框架，专为嵌入式系统设计。

```c
// test_example.c
#include "unity.h"

void setUp(void) {
    // 每个测试前的初始化
}

void tearDown(void) {
    // 每个测试后的清理
}

void test_function_should_return_true(void) {
    TEST_ASSERT_TRUE(my_function());
}

void test_function_should_handle_null(void) {
    TEST_ASSERT_EQUAL(LSC_INVALID_PARAM, my_function(NULL));
}

int main(void) {
    UNITY_BEGIN();
    RUN_TEST(test_function_should_return_true);
    RUN_TEST(test_function_should_handle_null);
    return UNITY_END();
}
```

### 运行测试命令

```bash
# 编译测试
cmake --build build

# 运行测试
./build/test_binary

# 运行特定测试
./build/test_binary -n test_function_should_return_true
```

---

## CMake 构建配置

### 基础 CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.20)
project(my_whitebox_tests C)

# 编译选项
set(CMAKE_C_STANDARD 11)

# Coverage 编译标志
if(CMAKE_BUILD_TYPE STREQUAL "Coverage")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fprofile-arcs -ftest-coverage --coverage")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} --coverage")
endif()

# Unity 测试框架
include_directories(${UNITY_PATH})

# 被测源文件
add_library(under_test STATIC
    src/my_module.c
)

# 测试文件
add_executable(test_binary
    tests/test_my_module.c
)

target_link_libraries(test_binary
    under_test
    gcov  # 覆盖率库
)
```

### Coverage 构建和报告生成

```bash
# 使用 Coverage 模式编译
cmake -B build -DCMAKE_BUILD_TYPE=Coverage -S tests/whitebox_tests
cmake --build build

# 运行测试
./build/test_binary

# 生成覆盖率数据
cd build
lcov --capture --directory . --output-file coverage.info

# 过滤系统头文件
lcov --remove coverage.info '/usr/*' --output-file coverage.info

# 生成 HTML 报告
genhtml coverage.info --output-directory ../coverage_report

# 查看报告
firefox ../coverage_report/index.html
```

---

## Include 路径管理

### C 项目常见的目录结构

```
project/
├── include/           # 公开头文件
│   ├── lsc.h
│   ├── lsc_errno.h
├── src/               # 源文件
│   ├── conns/
│   │   └── lsc_conn.c
├── include/pri_include/  # 私有头文件（内部实现）
│   ├── lsc_common.h
│   ├── lsc_config.h
├── tests/
│   └── whitebox_tests/
│       ├── test_lsc_conn.c
│       ├── CMakeLists.txt
│       ├── prj.conf    # Kconfig 配置
│       └── Kconfig
```

### CMakeLists.txt Include 配置

```cmake
# 必须包含所有相关头文件目录
target_include_directories(test_binary PRIVATE
    # 公开头文件
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_SOURCE_DIR}/include/pri_include

    # 被测源文件的私有头文件
    ${PROJECT_SOURCE_DIR}/src/conns
    ${PROJECT_SOURCE_DIR}/src/ultis/base64

    # 外部依赖
    ${LISA_PATH}/include
    ${CJSON_PATH}/include

    # Unity 测试框架
    ${UNITY_PATH}/src

    # 配置生成的头文件目录
    ${CMAKE_BINARY_DIR}/include/generated
)
```

---

## Kconfig 配置系统

### Zephyr/RTOS 项目常见配置

```ini
# prj.conf
CONFIG_LS_CHAT=y            # 启用主模块
CONFIG_LISA_LINUX=y         # Linux 平台支持
CONFIG_LOG=y                # 日志功能
CONFIG_ASSERT=y             # 断言功能
```

### Kconfig 文件

```kconfig
# Kconfig
source "${PROJECT_SOURCE_DIR}/Kconfig"

config TEST_MODULE
    bool "Enable test module"
    default y
```

### 配置头文件生成

CMake 通常使用 Python 脚本生成 `autoconf.h`：

```cmake
# 触发配置生成
find_package(Python3 REQUIRED)
add_custom_command(
    OUTPUT ${CMAKE_BINARY_DIR}/include/generated/autoconf.h
    COMMAND ${Python3_EXECUTABLE} ${PROJECT_SOURCE_DIR}/scripts/gen_config.py
    DEPENDS ${CMAKE_BINARY_DIR}/.config
)
```

---

## 静态函数测试策略

### 问题：C 静态函数无法直接测试

```c
// lsc_conn.c
static int conn_update_auth_header(char *token) {
    // 静态函数，外部无法直接调用
}
```

### 解决方案：通过公开 API 间接测试

```c
// test_lsc_conn.c
void test_conn_update_auth_header_indirect(void) {
    // 创建连接对象
    lsc_conn_t *conn = lsc_conn_create();
    TEST_ASSERT_NOT_NULL(conn);

    // 调用公开 API（内部会调用静态函数）
    int ret = conn->connect("test_token");

    // 验证结果
    TEST_ASSERT_EQUAL(LSC_OK, ret);
}
```

### 测试策略表

| 函数类型 | 测试方法 | 示例 |
|----------|----------|------|
| 公开函数 | 直接测试 | `lsc_conn_create()` |
| 静态函数 | 通过公开 API 间接测试 | `conn_disconnect()` 通过 `conn->disconnect()` |
| 回调函数 | 触发事件间接调用 | `_ws_data_cb` 需要 Mock WebSocket |

---

## Mock 策略

### 1. 手动 Mock（简单场景）

```c
// mock_lisa_ws.h
#ifndef MOCK_LISA_WS_H
#define MOCK_LISA_WS_H

// 定义 Mock 函数
int mock_lisa_ws_connect(void *hdl);
int mock_lisa_ws_disconnect(void *hdl);

// Mock 状态
typedef struct {
    bool connected;
    int connect_call_count;
    int disconnect_call_count;
} mock_ws_state_t;

mock_ws_state_t* get_mock_ws_state(void);

#endif
```

```c
// mock_lisa_ws.c
#include "mock_lisa_ws.h"

static mock_ws_state_t g_mock_state = {0};

mock_ws_state_t* get_mock_ws_state(void) {
    return &g_mock_state;
}

int mock_lisa_ws_connect(void *hdl) {
    g_mock_state.connect_call_count++;
    g_mock_state.connected = true;
    return 0;  // LISA_WS_OK
}

int mock_lisa_ws_disconnect(void *hdl) {
    g_mock_state.disconnect_call_count++;
    g_mock_state.connected = false;
    return 0;
}
```

### 2. FFF (Fake Function Framework)

```c
#include "fff.h"

DEFINE_FFF_GLOBALS;

// 定义 Mock 函数
FAKE_VALUE_FUNC(int, lisa_ws_connect, void*);
FAKE_VALUE_FUNC(int, lisa_ws_disconnect, void*);
FAKE_VALUE_FUNC(void*, lisa_ws_new);

void test_conn_connect_with_mock(void) {
    // 设置 Mock 返回值
    lisa_ws_connect_fake.return_val = 0;  // LISA_WS_OK
    lisa_ws_new_fake.return_val = (void*)0x1234;

    // 执行测试
    lsc_conn_t *conn = lsc_conn_create();
    int ret = conn->connect("token");

    // 验证 Mock 被调用
    TEST_ASSERT_EQUAL(1, lisa_ws_connect_fake.call_count);

    // 重置 Mock
    RESET_FAKE(lisa_ws_connect);
    RESET_FAKE(lisa_ws_disconnect);
    RESET_FAKE(lisa_ws_new);
}
```

### 3. 链接时替换

```cmake
# 创建 Mock 库替换真实依赖
add_library(mock_lisa_ws STATIC
    tests/mocks/mock_lisa_ws.c
)

# 测试链接 Mock 而非真实库
target_link_libraries(test_binary
    under_test
    mock_lisa_ws  # 替换真实 lisa 库
)
```

---

## 内存泄漏检测

### 内存统计测试

```c
void test_memory_leak_create_destroy(void) {
    size_t total_size, free_size_before, free_size_after;

    // 获取初始内存状态
    lisa_mem_get_info(&total_size, &free_size_before);

    // 执行操作
    lsc_conn_t *conn = lsc_conn_create();
    TEST_ASSERT_NOT_NULL(conn);

    int ret = lsc_conn_destroy(conn);
    TEST_ASSERT_EQUAL(LSC_OK, ret);

    // 获取结束内存状态
    lisa_mem_get_info(&total_size, &free_size_after);

    // 验证无内存泄漏
    TEST_ASSERT_EQUAL(free_size_before, free_size_after);
}
```

### 重复创建销毁测试

```c
void test_repeated_create_destroy(void) {
    size_t total_size, free_size_before, free_size_after;
    lisa_mem_get_info(&total_size, &free_size_before);

    for (int i = 0; i < 5; i++) {
        lsc_conn_t *conn = lsc_conn_create();
        TEST_ASSERT_NOT_NULL(conn);

        int ret = lsc_conn_destroy(conn);
        TEST_ASSERT_EQUAL(LSC_OK, ret);
    }

    lisa_mem_get_info(&total_size, &free_size_after);
    TEST_ASSERT_EQUAL(free_size_before, free_size_after);
}
```

---

## 圈复杂度分析

### 计算公式

```
V(G) = P + 1

P = 判定节点数量（if/else/for/while/switch/case）
```

### C 语言典型复杂度示例

```c
// V(G) = 11 (高复杂度，建议重构)
static int conn_connect(char *token) {
    int ret;
    lsc_conn_t *conn = g_lsc_conn_obj;

    if (!conn) return LSC_INVALID_STATE;              // 判定1

    ret = conn_update_auth_header(token);
    if (ret != LSC_OK) return LSC_INVALID_STATE;      // 判定2

    char *ws_url = conn_generate_url_from_malloc();
    if (!ws_url) goto _err;                           // 判定3

    // ... 配置代码

    if (!host_status) {                               // 判定4
        req.host = LSC_HOST;
    } else {
        req.host = LSC_HOST_STAGING;
    }

    ret = lisa_ws_cfg_set(conn->ws_hdl, &req);
    if (ret != LISA_WS_OK) goto _err;                 // 判定5

    if (ws_url) {                                     // 判定6
        lisa_mem_free(ws_url);
        ws_url = NULL;
    }

    ret = lisa_ws_connect(conn->ws_hdl);
    if (ret != LISA_WS_OK) goto _err;                 // 判定7

    // ... 错误处理分支
}
```

### 流程图绘制（Mermaid）

步骤5默认在 `path_analysis.md` 中直接输出 Mermaid `flowchart TD` 代码块，不需要安装 Graphviz、Python 包或生成独立图片文件。使用 `([开始])`/`([结束])` 表示开始与结束，`{"D1: 条件?"}` 表示判定，`[处理]` 表示处理步骤；所有判定分支必须标注真/假，循环必须使用回边表示。图后附节点/边的文字说明，供不支持 Mermaid 的阅读器使用。

---

## 测试用例设计模式

### 1. 初始化/销毁测试

```c
void test_create_success(void) {
    g_test_conn = lsc_conn_create();
    TEST_ASSERT_NOT_NULL(g_test_conn);
    TEST_ASSERT_NOT_NULL(g_test_conn->evt_cb_list);
    TEST_ASSERT_NOT_NULL(g_test_conn->ws_hdl);
}

void test_destroy_null_handle(void) {
    int ret = lsc_conn_destroy(NULL);
    TEST_ASSERT_EQUAL(LSC_INVALID_PARAM, ret);
}

void test_destroy_mismatch_handle(void) {
    lsc_conn_t fake_hdl;
    memset(&fake_hdl, 0, sizeof(lsc_conn_t));

    int ret = lsc_conn_destroy(&fake_hdl);
    TEST_ASSERT_EQUAL(LSC_INVALID_PARAM, ret);
}
```

### 2. 回调注册测试

```c
static int g_event_count = 0;
static conn_event_e g_last_event = 0;

static void test_event_callback(conn_event_e evt, void *data,
                                 uint32_t size, void *usr) {
    g_event_count++;
    g_last_event = evt;
}

void test_add_evt_callback_success(void) {
    g_test_conn = lsc_conn_create();

    int ret = g_test_conn->add_evt_callback(test_event_callback,
                                            CONN_CONNECTED | CONN_DISCONNECTED,
                                            NULL);
    TEST_ASSERT_EQUAL(LSC_OK, ret);
}

void test_add_evt_callback_multiple(void) {
    g_test_conn = lsc_conn_create();

    // 添加多个事件类型
    TEST_ASSERT_EQUAL(LSC_OK,
        g_test_conn->add_evt_callback(test_event_callback, CONN_CONNECTED, NULL));
    TEST_ASSERT_EQUAL(LSC_OK,
        g_test_conn->add_evt_callback(test_event_callback, CONN_DISCONNECTED, NULL));
    TEST_ASSERT_EQUAL(LSC_OK,
        g_test_conn->add_evt_callback(test_event_callback, CONN_AUTH_SUCESS, NULL));
}
```

### 3. 状态切换测试

```c
void test_host_status_toggle(void) {
    g_test_conn = lsc_conn_create();

    // 多次切换状态
    TEST_ASSERT_EQUAL(LSC_OK, lsc_set_host_status(true));
    TEST_ASSERT_EQUAL(LSC_OK, lsc_set_host_status(false));
    TEST_ASSERT_EQUAL(LSC_OK, lsc_set_host_status(true));
    TEST_ASSERT_EQUAL(LSC_OK, lsc_set_host_status(false));
}

void test_host_status_null_conn(void) {
    // 未初始化时设置状态应失败
    int ret = lsc_set_host_status(true);
    TEST_ASSERT_EQUAL(LSC_ERR, ret);
}
```

---

## 常见问题解决

### 1. autoconf.h 未生成

```
错误: fatal error: autoconf.h: No such file or directory
```

**解决方案**：

```cmake
# 确保 prj.conf 存在且正确配置
# 确保 Kconfig 正确引用主项目配置
# 触发 CMake 配置阶段生成头文件
cmake -B build -DDEFCONF_FILE=prj.conf
```

### 2. 静态函数无法测试

```
错误: undefined reference to `conn_update_auth_header'
```

**解决方案**：

- 通过公开 API 间接测试
- 或在测试编译时 `#include "lsc_conn.c"` 直接编译源文件（不推荐）

### 3. 链接错误：库未找到

```
错误: cannot find -lxxx
```

**解决方案**：

```cmake
# 添加库路径
link_directories(${LIB_PATH})

# 或使用 target_link_libraries 指定完整路径
target_link_libraries(test_binary ${LIB_PATH}/libxxx.a)
```

### 4. Segmentation Fault

```
错误: Segmentation fault (core dumped)
```

**排查步骤**：

1. 检查空指针访问
2. 检查内存越界
3. 使用 gdb 调试：

```bash
gdb ./build/test_binary
(gdb) run
(gdb) bt  # 查看调用栈
```

---

## 最佳实践

### 测试文件模板

```c
/**
 * @file test_lsc_conn_whitebox.c
 * @brief lsc_conn.c 白盒测试用例
 *
 * 测试策略：
 * 1. 判定路径覆盖 - 测试所有独立路径
 * 2. 边界值测试 - 测试空指针、内存失败等边界情况
 * 3. 异常处理测试 - 测试所有错误处理分支
 *
 * 注意：通过公开API间接测试静态函数
 */

#include "unity.h"
#include "lsc_conn.h"
#include "lsc_errno.h"
#include "lisa_mem.h"
#include <string.h>

/* ==================== 全局变量 ==================== */
static lsc_conn_t *g_test_conn = NULL;

/* ==================== 测试辅助函数 ==================== */

void setUp(void) {
    g_test_conn = NULL;
}

void tearDown(void) {
    if (g_test_conn != NULL) {
        lsc_conn_destroy(g_test_conn);
        g_test_conn = NULL;
    }
}

/* ==================== 测试用例 ==================== */

void test_lsc_conn_create_success(void) {
    g_test_conn = lsc_conn_create();
    TEST_ASSERT_NOT_NULL(g_test_conn);
}

/* ==================== 测试主程序 ==================== */

int main(void) {
    UNITY_BEGIN();

    RUN_TEST(test_lsc_conn_create_success);
    // ... 更多测试

    return UNITY_END();
}
```

### 测试组织原则

1. **分层组织**：setUp/tearDown 管理资源生命周期
2. **全局变量**：使用 static 全局变量跟踪测试状态
3. **命名规范**：`test_<函数名>_<场景描述>`
4. **文档注释**：说明测试目标和覆盖路径
5. **内存检测**：重要测试后验证内存无泄漏

---

## 覆盖率目标

| 项目类型 | 行覆盖率 | 函数覆盖率 |
|----------|----------|------------|
| 关键业务逻辑 | 80%+ | 90%+ |
| 通信模块 | 70%+ | 80%+ |
| 工具函数 | 100% | 100% |
| 嵌入式驱动 | 60%+ | 70%+ |

---

## 参考资料

- [Unity 测试框架](https://github.com/ThrowTheSwitch/Unity)
- [FFF Mock 框架](https://github.com/meekrosoft/fff)
- [lcov 覆盖率工具](https://github.com/linux-test-project/lcov)
- [CMake 官方文档](https://cmake.org/documentation/)
- [GCC Coverage 选项](https://gcc.gnu.org/onlinedocs/gcc/Gcov.html)
