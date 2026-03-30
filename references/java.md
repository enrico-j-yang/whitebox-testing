# Java 白盒测试指南

## 概述

本指南总结了在 **Spring Boot + JPA** 项目中进行白盒测试的经验，主要针对Service层的业务逻辑测试，采用**判定路径覆盖(Decision Path Coverage)**标准。

---

## 测试框架和工具

### 核心测试框架

```xml
<!-- pom.xml dependencies -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.9.3</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.5.0</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.24.2</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 工具说明

| 工具 | 用途 | 推荐度 |
|------|------|--------|
| **JUnit 5** | 测试框架，提供@Test、@DisplayName等注解 | ⭐⭐⭐⭐⭐ |
| **Mockito** | Mock框架，模拟外部依赖 | ⭐⭐⭐⭐⭐ |
| **AssertJ** | 流式断言库，比JUnit Assertions更强大 | ⭐⭐⭐⭐⭐ |
| **Spring Test Utils** | ReflectionTestUtils，用于依赖注入 | ⭐⭐⭐⭐ |
| **MockMultipartFile** | Spring提供的文件上传测试工具 | ⭐⭐⭐⭐ |

---

## 测试类结构模板

### 标准测试类结构

```java
package com.example.service;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import static org.assertj.core.api.Assertions.*;
import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;

/**
 * ServiceName 白盒测试 - 判定路径覆盖
 *
 * 测试目标：覆盖所有独立路径，达到100%判定路径覆盖率
 */
@DisplayName("ServiceName 白盒测试")
class ServiceNameWhiteboxTest {

    @Mock
    private DependencyRepository dependencyRepository;

    @Mock
    private OtherService otherService;

    private ServiceName serviceName;

    private TestEntity testEntity;

    @BeforeEach
    void setUp() {
        // 初始化Mock
        MockitoAnnotations.openMocks(this);

        // 创建Service实例
        serviceName = new ServiceName();

        // 使用反射注入依赖（非Spring管理的Service）
        org.springframework.test.util.ReflectionTestUtils.setField(
            serviceName, "dependencyRepository", dependencyRepository
        );
        org.springframework.test.util.ReflectionTestUtils.setField(
            serviceName, "otherService", otherService
        );

        // 创建测试数据
        testEntity = new TestEntity();
        testEntity.setId("test-id");
        testEntity.setName("test-name");
    }

    // ========== 方法名() 测试 - V(G)=X ==========

    @Test
    @DisplayName("路径1: 测试场景描述")
    void testMethodName_Path1_ScenarioDescription() {
        // Given - 准备测试数据和Mock行为
        when(dependencyRepository.findById("test-id"))
            .thenReturn(Optional.of(testEntity));

        // When - 执行测试方法
        Result result = serviceName.methodName("test-id");

        // Then - 验证结果
        assertThat(result).isNotNull();
        assertThat(result.getName()).isEqualTo("test-name");

        // Verify - 验证Mock调用
        verify(dependencyRepository, times(1)).findById("test-id");
    }

    @Test
    @DisplayName("路径2: 异常场景")
    void testMethodName_Path2_ExceptionScenario() {
        // Given
        when(dependencyRepository.findById("non-existent"))
            .thenReturn(Optional.empty());

        // When & Then - 异常测试
        assertThatThrownBy(() -> serviceName.methodName("non-existent"))
            .isInstanceOf(BusinessException.class)
            .hasMessageContaining("不存在");

        // Verify
        verify(otherService, never()).doSomething();
    }
}
```

---

## Mock策略

### 1. Repository层Mock

```java
@Mock
private UserRepository userRepository;

// 查询方法
when(userRepository.findById("user-123"))
    .thenReturn(Optional.of(testUser));

when(userRepository.findByUsername("testuser"))
    .thenReturn(Optional.of(testUser));

// 存在性检查
when(userRepository.existsByUsername("existinguser"))
    .thenReturn(true);

when(userRepository.existsByEmail("existing@example.com"))
    .thenReturn(false);

// 保存方法
when(userRepository.save(any(User.class)))
    .thenReturn(testUser);

// 分页查询
Page<User> userPage = new PageImpl<>(Arrays.asList(testUser));
when(userRepository.findAll(any(Pageable.class)))
    .thenReturn(userPage);
```

### 2. Service层Mock

```java
@Mock
private NotificationService notificationService;

// 同步方法
when(notificationService.getNotifications(anyString(), anyInt(), anyInt()))
    .thenReturn(notificationResponse);

// 异步方法 - 使用doNothing()
doNothing().when(notificationService)
    .createLikeNotification(anyString(), anyString());
```

### 3. Security组件Mock

```java
@Mock
private AuthenticationManager authenticationManager;

@Mock
private JwtTokenProvider tokenProvider;

@Mock
private PasswordEncoder passwordEncoder;

// 认证流程
Authentication testAuth = new UsernamePasswordAuthenticationToken(
    userPrincipal, null, userPrincipal.getAuthorities()
);
when(authenticationManager.authenticate(any(UsernamePasswordAuthenticationToken.class)))
    .thenReturn(testAuth);

// JWT令牌生成
when(tokenProvider.generateAccessToken(any(Authentication.class)))
    .thenReturn("access-token");

when(tokenProvider.generateRefreshToken(anyString()))
    .thenReturn("refresh-token");

// 令牌验证
when(tokenProvider.validateToken("valid-token"))
    .thenReturn(true);

// 密码编码
when(passwordEncoder.encode("password123"))
    .thenReturn("hashedPassword");
```

---

## 依赖注入方法

### 方法1: ReflectionTestUtils（推荐）

适用于**非Spring管理的Service实例**（手动new创建）：

```java
@BeforeEach
void setUp() {
    MockitoAnnotations.openMocks(this);
    serviceName = new ServiceName();

    // 使用反射注入依赖
    ReflectionTestUtils.setField(serviceName, "repository", repository);
    ReflectionTestUtils.setField(serviceName, "otherService", otherService);
}
```

### 方法2: Spring @Autowired（集成测试）

适用于需要完整Spring上下文的集成测试：

```java
@SpringBootTest
class ServiceIntegrationTest {

    @Autowired
    private ServiceName serviceName;

    @MockBean
    private DependencyRepository repository;
}
```

### 方法3: 构造函数注入

如果Service使用构造函数注入：

```java
@BeforeEach
void setUp() {
    MockitoAnnotations.openMocks(this);
    serviceName = new ServiceName(repository, otherService);
}
```

---

## 特殊测试场景

### 1. 异步方法测试

**场景**: Service调用异步方法（@Async）

```java
@Test
@DisplayName("异步方法调用测试")
void testAsyncMethodInvocation() {
    // Given
    @Mock
    private NotificationService notificationService;

    // Mock异步方法 - 使用doNothing()
    doNothing().when(notificationService)
        .createLikeNotification(anyString(), anyString());

    // When
    likeService.likePost("user-123", "post-456");

    // Then - 验证异步方法被调用
    verify(notificationService, times(1))
        .createLikeNotification("post-456", "user-123");
}
```

**注意**:
- 异步方法测试不需要等待执行完成
- 只验证异步方法被正确调用即可
- 使用 `doNothing().when()` Mock异步方法

---

### 2. SecurityContext测试

**场景**: 测试依赖SecurityContext的方法

```java
@Test
@DisplayName("获取当前用户成功")
void testGetCurrentUser_Success() {
    // Given - 设置SecurityContext
    UserPrincipal userPrincipal = UserPrincipal.create(testUser);
    Authentication auth = new UsernamePasswordAuthenticationToken(
        userPrincipal, null, userPrincipal.getAuthorities()
    );
    SecurityContextHolder.getContext().setAuthentication(auth);

    when(userRepository.findById("user-123"))
        .thenReturn(Optional.of(testUser));

    // When
    User currentUser = authService.getCurrentUser();

    // Then
    assertThat(currentUser).isNotNull();
    assertThat(currentUser.getUsername()).isEqualTo("testuser");

    // Cleanup - 清空SecurityContext
    SecurityContextHolder.clearContext();
}

@Test
@DisplayName("未授权访问异常")
void testGetCurrentUser_Unauthorized() {
    // Given - 清空SecurityContext
    SecurityContextHolder.clearContext();

    // When & Then
    assertThatThrownBy(() -> authService.getCurrentUser())
        .isInstanceOf(BusinessException.class)
        .hasMessageContaining("未授权访问");
}
```

---

### 3. 文件上传测试

**场景**: 测试文件上传方法

```java
@Test
@DisplayName("文件上传成功")
void testFileUpload_Success() throws Exception {
    // Given - 使用MockMultipartFile
    MockMultipartFile file = new MockMultipartFile(
        "file",
        "test.jpg",
        "image/jpeg",
        new byte[1024] // 1KB文件
    );

    when(fileStorageService.saveFile(any(MultipartFile.class)))
        .thenReturn("/uploads/test.jpg");
    when(postRepository.save(any(Post.class)))
        .thenReturn(testPost);

    // When
    Post post = mediaService.uploadAndCreatePost(
        "user-123", file, "Test caption"
    );

    // Then
    assertThat(post).isNotNull();
    assertThat(post.getMediaUrl()).isEqualTo("/uploads/test.jpg");

    // Verify
    verify(fileStorageService, times(1)).saveFile(any(MultipartFile.class));
}
```

---

### 4. 分页查询测试

**场景**: 测试分页查询方法

```java
@Test
@DisplayName("分页查询成功")
void testPaginationQuery_Success() {
    // Given - 创建分页结果
    List<Post> posts = Arrays.asList(post1, post2, post3);
    Page<Post> postPage = new PageImpl<>(posts);

    when(postRepository.findByUserIdOrderByCreatedAtDesc(
        eq("user-123"),
        any(Pageable.class)
    )).thenReturn(postPage);

    // When
    Page<Post> result = postService.getUserPosts("user-123", 1, 10);

    // Then
    assertThat(result.getContent()).hasSize(3);
    assertThat(result.getTotalElements()).isEqualTo(3);
    assertThat(result.getNumber()).isEqualTo(0); // 页码从0开始

    // Verify - 验证Pageable参数
    ArgumentCaptor<Pageable> pageableCaptor = ArgumentCaptor.forClass(Pageable.class);
    verify(postRepository).findByUserIdOrderByCreatedAtDesc(
        eq("user-123"),
        pageableCaptor.capture()
    );

    Pageable capturedPageable = pageableCaptor.getValue();
    assertThat(capturedPageable.getPageNumber()).isEqualTo(0);
    assertThat(capturedPageable.getPageSize()).isEqualTo(10);
}
```

---

### 5. 边界值测试

**场景**: 测试数值边界情况

```java
@Test
@DisplayName("边界值: 点赞计数从0增加到1")
void testLikeCount_ZeroToOne() {
    // Given
    testPost.setLikeCount(0); // 边界值

    when(likeRepository.existsByUserAndPost(testUser, testPost))
        .thenReturn(false);
    when(likeRepository.save(any(Like.class)))
        .thenReturn(testLike);

    // When
    LikeResponse response = likeService.likePost("user-123", "post-456");

    // Then
    assertThat(response.getLikeCount()).isEqualTo(1); // 0 + 1 = 1
}

@Test
@DisplayName("边界值: 点赞计数从1减少到0")
void testLikeCount_OneToZero() {
    // Given
    testPost.setLikeCount(1); // 边界值

    when(likeRepository.findByUserAndPost(testUser, testPost))
        .thenReturn(Optional.of(testLike));

    // When
    LikeResponse response = likeService.unlikePost("user-123", "post-456");

    // Then
    assertThat(response.getLikeCount()).isEqualTo(0); // 1 - 1 = 0
}

@Test
@DisplayName("边界值: 计数为0时不执行减法")
void testLikeCount_NoDecrementWhenZero() {
    // Given
    testPost.setLikeCount(0); // 已为0，不应再减

    when(likeRepository.findByUserAndPost(testUser, testPost))
        .thenReturn(Optional.of(testLike));

    // When
    LikeResponse response = likeService.unlikePost("user-123", "post-456");

    // Then - 计数保持为0，不出现负数
    assertThat(response.getLikeCount()).isEqualTo(0);

    // Verify - 不保存帖子（计数未改变）
    verify(postRepository, never()).save(any(Post.class));
}
```

---

## 流程图绘制方法

### 圈复杂度计算

使用McCabe公式计算圈复杂度 V(G):

```
V(G) = E - N + 2P

其中:
E = 流图中的边数
N = 流图中的节点数
P = 连接组件数（通常为1）

简化公式:
V(G) = 判定节点数 + 1

判定节点:
- if语句
- while/for循环
- case语句
- try-catch块
- 条件表达式 (?:)
```

### 流程图绘制规范

使用ASCII字符绘制流程图：

```
            ┌──────────────────┐
            │      开始        │
            └��───────┬─────────┘
                     │
            ┌────────▼─────────┐
            │   判定节点?      │
            └────────┬─────────┘
               ┌─────┴─────┐
              是            否
               │            │
        ┌──────▼─────┐  ┌──▼───────┐
        │ 处理步骤A  │  │ 处理步骤B│
        └──────┬─────┘  └──┬───────┘
               │           │
               └───────┬───┘
                       │
               ┌───────▼───────┐
               │    结束       │
               └───────────────┘
```

**符号说明**:
- `┌─┐` - 方框（节点）
- `─` - 横线
- `│` - 竖线
- `▼` - 向下箭头
- `◄` - 向左箭头
- `├─` - 左分支
- `┤─` - 右分支
- `┬─` - 向下分支
- `┴─` - 向上合并

---

## 独立路径枚举方法

### 判定路径覆盖原则

每个判定节点产生2条路径（真/假），总路径数为：

```
独立路径数 = 所有判定节点的组合路径

示例：
if (A) {       // 判定1
    if (B) {   // 判定2
        // 路径1: A真 → B真
    } else {
        // 路径2: A真 → B假
    }
} else {
    // 路径3: A假
}

总路径数 = 3条
```

### 路径枚举模板

```markdown
### 独立路径（共X条）

#### 路径1: 场景描述
- **路径**: 开始 → 判定1(真) → 判定2(真) → 处理 → 返回 → 结束
- **测试数据**: `param1="value1", param2="value2"`
- **预期**: 返回结果，状态变更

#### 路径2: 异常场景
- **路径**: 开始 → 判定1(假) → 抛异常 → 结束
- **测试数据**: `param1="invalid"`
- **预期**: 抛出BusinessException("错误信息")
```

---

## 常见问题和解决方案

### 问题1: Mock方法参数模糊

**错误**: `reference to method is ambiguous`

```java
// 错误示例
verify(tokenProvider, never()).generateAccessToken(any());
// 编译器不知道any()应该匹配哪个方法
```

**解决方案**: 明确指定参数类型

```java
// 正确示例
verify(tokenProvider, never()).generateAccessToken(any(Authentication.class));
verify(tokenProvider, never()).generateAccessToken(anyString());
```

---

### 问题2: 参数匹配失败

**错误**: Mock未生效，实际参数与预期不符

**解决方案**: 使用ArgumentCaptor验证参数

```java
ArgumentCaptor<User> userCaptor = ArgumentCaptor.forClass(User.class);
verify(userRepository).save(userCaptor.capture());

User capturedUser = userCaptor.getValue();
assertThat(capturedUser.getUsername()).isEqualTo("testuser");
assertThat(capturedUser.getEmail()).isEqualTo("test@example.com");
```

---

### 问题3: 字符编码问题

**错误**: 中文字符长度计算不一致

```java
// 错误示例 - 中文字符长度判断
String comment = "这是一个很长的评论"; // 实际长度可能不符合预期
```

**解决方案**: 使用英文字符测试长度限制

```java
// 正确示例 - 使用ASCII字符
String longComment = "This is a very long comment that exceeds fifty characters limit for testing"; // 60字符
```

---

### 问题4: 异常消息不匹配

**错误**: `hasMessageContaining()` 消息不匹配

**解决方案**: 先读取ErrorCode定义

```java
// 1. 读取ErrorCode.java或Service代码
// 2. 找到确切的错误消息
// 3. 使用精确的消息片段

// 错误消息: "用户名已被使用"
assertThatThrownBy(() -> service.register(request))
    .hasMessageContaining("用户名")
    .hasMessageContaining("已被使用");
```

---

### 问题5: Repository.save()验证失败

**问题**: 无法验证save()的参数对象属性

**解决方案**: 使用argThat()自定义匹配器

```java
verify(userRepository, times(1)).save(
    argThat(user -> {
        return user.getUsername().equals("testuser") &&
               user.getEmail().equals("test@example.com") &&
               user.getChildBirthday() == null; // 验证特定属性为null
    })
);
```

---

### 问题6: 分页参数验证

**问题**: 无法验证Pageable参数的具体值

**解决方案**: 使用ArgumentCaptor捕获Pageable

```java
ArgumentCaptor<Pageable> pageableCaptor = ArgumentCaptor.forClass(Pageable.class);
verify(postRepository).findByUserId(anyString(), pageableCaptor.capture());

Pageable pageable = pageableCaptor.getValue();
assertThat(pageable.getPageNumber()).isEqualTo(0); // 页码从0开始
assertThat(pageable.getPageSize()).isEqualTo(10);
assertThat(pageable.getSort()).isEqualTo(Sort.by("createdAt").descending());
```

---

## 最佳实践

### 1. 测试命名规范

```java
@Test
@DisplayName("路径X: 测试场景详细描述")
void testMethodName_PathX_ScenarioDescription() {
    // 测试代码
}
```

**规范**:
- 方法名使用下划线分隔
- DisplayName包含路径编号和场景描述
- 中文描述更清晰（推荐）

---

### 2. 测试代码组织

```java
@Test
void testSomething() {
    // ========== Given - 准备数据 ==========
    when(repository.findById("id")).thenReturn(Optional.of(entity));

    // ========== When - 执行方法 ==========
    Result result = service.doSomething("id");

    // ========== Then - 验证结果 ==========
    assertThat(result).isNotNull();
    assertThat(result.getName()).isEqualTo("expected");

    // ========== Verify - 验证调用 ==========
    verify(repository, times(1)).findById("id");
    verify(otherService, never()).doSomething();
}
```

---

### 3. 复用测试数据

```java
class ServiceWhiteboxTest {
    private User testUser;
    private Post testPost;
    private Like testLike;

    @BeforeEach
    void setUp() {
        // 创建可复用的测试数据
        testUser = createTestUser();
        testPost = createTestPost();
        testLike = createTestLike();
    }

    private User createTestUser() {
        User user = new User();
        user.setId("user-123");
        user.setUsername("testuser");
        user.setEmail("test@example.com");
        user.setCreatedAt(LocalDateTime.now());
        return user;
    }
}
```

---

### 4. 测试分组

```java
// ========== 方法A() 测试 - V(G)=X ==========

@Test
@DisplayName("路径1: ...")
void testMethodA_Path1() { }

@Test
@DisplayName("路径2: ...")
void testMethodA_Path2() { }

// ========== 方法B() 测试 - V(G)=Y ==========

@Test
@DisplayName("路径1: ...")
void testMethodB_Path1() { }

// ========== 边界值测试 ==========

@Test
@DisplayName("边界值: ...")
void testBoundary() { }
```

---

### 5. 异常测试模式

```java
// 模式1: 异常抛出测试
assertThatThrownBy(() -> service.method(invalidParam))
    .isInstanceOf(BusinessException.class)
    .hasMessageContaining("错误关键词")
    .hasMessageNotContaining("不应出现的消息");

// 模式2: 验证异常后的状态
verify(repository, never()).save(any());
verify(otherService, never()).doSomething();
```

---

### 6. Mock行为重置

```java
@BeforeEach
void setUp() {
    MockitoAnnotations.openMocks(this); // 重置所有Mock
    // 每个测试方法前都会执行setUp，Mock状态被清空
}
```

**注意**: 避免在测试方法之间共享Mock状态

---

## 测试报告模板

### 测试概览

```markdown
## 测试概览

- **测试方法**: ServiceName.java
- **测试标准**: 判定路径覆盖 (Decision Path Coverage)
- **总测试用例数**: X个
- **测试通过率**: 100% ✅
- **执行时间**: X.XXX秒
```

### 方法复杂度分析表

```markdown
| 方法名 | 圈复杂度 V(G) | 风险级别 | 独立路径数 | 测试用例数 |
|--------|--------------|----------|-----------|-----------|
| `methodA()` | 7 | 中 | 7 | 7 |
| `methodB()` | 4 | 低 | 4 | 4 |
| `methodC()` | 2 | 低 | 2 | 2 |
```

### 风险级别标准

| 复杂度 V(G) | 风险级别 | 测试建议 |
|-------------|----------|----------|
| 1-5 | 低 ✅ | 正常测试流程 |
| 6-10 | 中 ⚠️ | 增加边界测试用例 |
| 11-20 | 高 ❌ | 重点测试，考虑重构 |
| 21+ | 极高 ❌ | 必须重构拆分函数 |

---

## 测试执行命令

### 运行单个测试类

```bash
mvn test -Dtest="ServiceNameWhiteboxTest"
```

### 运行单个测试方法

```bash
mvn test -Dtest="ServiceNameWhiteboxTest#testMethodName"
```

### 运行所有测试

```bash
mvn test
```

### 生成测试报告

```bash
mvn test -Dtest="ServiceNameWhiteboxTest" > test-results.txt
```

---

## 经验总结

### 1. Mock依赖优先级

**优先Mock的依赖**:
- Repository层（数据访问）
- 外部Service调用
- 安全组件（认证/授权）
- 文件系统操作
- 网络请求

**不需要Mock的依赖**:
- 简单的工具类
- 纯计算逻辑
- 同Service内的私有方法

---

### 2. 测试用例设计顺序

**推荐顺序**:
1. 正常路径（成功场景）
2. 异常路径（失败场景）
3. 边界值测试
4. 并发/性能测试（可选）

---

### 3. 测试数据设计原则

- 使用固定的ID（如"user-123", "post-456"）
- 时间使用固定值（`LocalDateTime.of(2020, 1, 15, 0, 0)`）
- 避免使用当前时间（`LocalDateTime.now()`）导致测试不稳定
- 字符串长度测试使用ASCII字符

---

### 4. 测试代码可维护性

- 使用@BeforeEach统一初始化测试数据
- 使用DisplayName清晰描述测试场景
- 使用注释分隔测试代码段落（Given/When/Then/Verify）
- 将相似的测试分组组织

---

### 5. 避免过度Mock

**问题**: Mock太多导致测试变成Mock测试，而非逻辑测试

**建议**:
- 只Mock外部依赖，不Mock被测Service内部方法
- 简单的计算逻辑直接测试，不Mock
- 保持Mock行为简单明确

---

## 参考资源

### 官方文档

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
- [AssertJ Documentation](https://assertj.github.io/doc/)
- [Spring Boot Testing](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing)

### 最佳实践文章

- [Unit Testing with Mockito](https://www.baeldung.com/mockito-series)
- [Spring Boot Service Layer Unit Testing](https://www.baeldung.com/spring-boot-testing)
- [AssertJ Assertions Guide](https://www.baeldung.com/assertj)

---

## 附录：完整测试案例

参考以下已完成的白盒测试案例：

1. **MediaServiceWhiteboxTest.java** - 文件上传、图片处理
2. **NotificationServiceWhiteboxTest.java** - 异步通知、分页查询
3. **LikeServiceWhiteboxTest.java** - 状态切换、计数管理
4. **AuthServiceWhiteboxTest.java** - 认证授权、SecurityContext

所有测试代码位于：`src/test/java/com/kidsphotosharing/service/`

所有分析文档位于：`whitebox-analysis/`

---

**更新时间**: 2026-03-28
**测试框架版本**: JUnit 5.9.3 + Mockito 5.5.0 + AssertJ 3.24.2
**Spring Boot版本**: 2.7.18
**Java版本**: 17