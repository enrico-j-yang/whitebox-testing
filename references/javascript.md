# JavaScript/TypeScript 白盒测试指南

## 测试框架和工具

### Jest 配置

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',  // 或 'node' 用于后端代码
  roots: ['<rootDir>/src'],
  testMatch: ['**/*.test.ts', '**/*.test.tsx'],
  moduleFileExtensions: ['ts', 'tsx', 'js', 'jsx'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/index.ts',  // 排除入口文件
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
};
```

### 运行测试命令

```bash
# 运行所有测试
npm test

# 运行特定文件测试
npm test -- --testPathPattern="AnimationManager"

# 运行并生成覆盖率报告
npm test -- --coverage

# 详细输出
npm test -- --verbose

# 监听模式
npm test -- --watch
```

---

## Mock 策略

### 1. 模块级别 Mock

适用于第三方库或外部依赖：

```typescript
// Mock three-stdlib
jest.mock('three-stdlib', () => ({
  MMDAnimationHelper: jest.fn().mockImplementation(() => ({
    add: jest.fn(),
    enable: jest.fn(),
    update: jest.fn(),
    objects: new Map(),
  })),
  MMDLoader: jest.fn().mockImplementation(() => ({
    loadAnimation: jest.fn(),
  })),
}));
```

### 2. 实例级别 Mock

多个测试共享同一个 mock 实例：

```typescript
// 在 describe 外部定义共享 mock
const mockHelper = {
  add: jest.fn(),
  enable: jest.fn(),
  update: jest.fn(),
  objects: new Map(),
};

jest.mock('three-stdlib', () => ({
  MMDAnimationHelper: jest.fn(() => mockHelper),
}));

describe('MyClass', () => {
  beforeEach(() => {
    // 重置 mock 状态
    mockHelper.add.mockClear();
    mockHelper.objects = new Map();
  });
});
```

### 3. 原型方法 Mock

适用于覆盖类的方法：

```typescript
it('应成功加载纹理', async () => {
  const mockTexture = new THREE.Texture();
  const loadSpy = jest.spyOn(THREE.TextureLoader.prototype, 'load')
    .mockImplementation(function(this: THREE.TextureLoader, _url, onLoad, _onProgress, _onError) {
      setTimeout(() => {
        if (onLoad) onLoad(mockTexture);
      }, 0);
      return mockTexture;
    });

  const texture = await loader.loadTexture('test.png');

  expect(texture).toBe(mockTexture);
  loadSpy.mockRestore();  // 重要：恢复原始方法
});
```

### 4. Console Mock

减少测试噪音并验证日志输出：

```typescript
let consoleLogSpy: jest.SpyInstance;
let consoleErrorSpy: jest.SpyInstance;

beforeEach(() => {
  consoleLogSpy = jest.spyOn(console, 'log').mockImplementation();
  consoleErrorSpy = jest.spyOn(console, 'error').mockImplementation();
});

afterEach(() => {
  consoleLogSpy.mockRestore();
  consoleErrorSpy.mockRestore();
});
```

---

## 异步代码测试

### Promise 测试

```typescript
// 成功场景
it('应成功加载数据', async () => {
  const data = await loader.loadModel({ modelPath: 'test.pmx' });
  expect(data).toBeDefined();
});

// 失败场景
it('应在失败时抛出错误', async () => {
  await expect(loader.loadModel({ modelPath: 'invalid.pmx' }))
    .rejects.toThrow('Load failed');
});
```

### 回调转 Promise

```typescript
// 将回调风格的 API 转换为 Promise
const result = await new Promise((resolve, reject) => {
  loader.load(
    path,
    (data) => resolve(data),
    (progress) => { /* 进度处理 */ },
    (error) => reject(error)
  );
});
```

### 模拟异步回调

```typescript
mockLoader.load.mockImplementation(
  (url: string, onLoad: any, onProgress: any, onError: any) => {
    // 模拟进度
    setTimeout(() => onProgress({ loaded: 50, total: 100 }), 0);

    // 模拟成功/失败
    setTimeout(() => {
      if (url.includes('fail')) {
        onError(new Error('Mock error'));
      } else {
        onLoad(mockData);
      }
    }, 10);
  }
);
```

---

## 覆盖率分析

### 覆盖率指标

| 指标 | 说明 | 公式 |
|------|------|------|
| 语句覆盖 | 每条语句至少执行一次 | 已执行语句 / 总语句 |
| 分支覆盖 | 每个分支至少执行一次 | 已执行分支 / 总分支 |
| 函数覆盖 | 每个函数至少调用一次 | 已调用函数 / 总函数 |
| 行覆盖 | 每行代码至少执行一次 | 已执行行 / 总行 |

### 覆盖率报告解读

```
File                     | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
-------------------------|---------|----------|---------|---------|-------------------
AnimationManager.ts      |   100   |   88.88  |   100   |   100   | 46
```

- **Uncovered Line #s**: 未覆盖的行号
- 分支覆盖低于语句覆盖是正常的（需要更多测试用例）

### 未覆盖代码原因分析

1. **条件分支未覆盖**：添加边界测试用例
2. **异常处理未覆盖**：模拟错误场景
3. **死代码**：分析是否可以删除
4. **不可达代码**：几何/算法限制导致某些分支无法执行

---

## 圈复杂度分析

### 计算公式

```
V(G) = P + 1

P = 判定节点数量（if/else/for/while/switch/catch）
```

### 风险评估

| 复杂度 | 风险 | 建议 |
|--------|------|------|
| 1-10 | 低 | 正常测试 |
| 11-20 | 中 | 考虑重构或���加测试 |
| 21+ | 高 | 必须重构，拆分函数 |

### 示例分析

```typescript
// V(G) = 3 (2个 if-else + 1)
function processValue(value: number): string {
  if (value > 100) {           // 判定1
    return 'large';
  } else if (value > 50) {     // 判定2
    return 'medium';
  } else {
    return 'small';
  }
}

// 需要测试用例覆盖 3 条独立路径:
// P1: value > 100 → 'large'
// P2: 50 < value <= 100 → 'medium'
// P3: value <= 50 → 'small'
```

---

## 测试用例设计模式

### 1. 状态转换测试

```typescript
describe('播放状态管理', () => {
  it('初始状态应为停止', () => {
    expect(manager.getIsPlaying()).toBe(false);
  });

  it('play() 应将状态改为播放', () => {
    manager.play();
    expect(manager.getIsPlaying()).toBe(true);
  });

  it('pause() 应将状态改为暂停', () => {
    manager.play();
    manager.pause();
    expect(manager.getIsPlaying()).toBe(false);
  });
});
```

### 2. 边界值测试

```typescript
describe('权重分配', () => {
  it('y > 0.5 应分配上半身权重', () => { /* ... */ });
  it('y = 0.5 边界', () => { /* ... */ });
  it('y > -0.5 且 y <= 0.5 应分配中间权重', () => { /* ... */ });
  it('y = -0.5 边界', () => { /* ... */ });
  it('y <= -0.5 应分配下半身权重', () => { /* ... */ });
});
```

### 3. 异常路径测试

```typescript
describe('错误处理', () => {
  it('应处理网络错误', async () => {
    mockLoader.load.mockImplementation((_, __, ___, onError) => {
      onError(new Error('Network error'));
    });
    await expect(loader.loadModel(options)).rejects.toThrow();
  });

  it('应处理空值输入', () => {
    expect(() => processor.process(null)).toThrow();
  });
});
```

### 4. 集成测试

```typescript
describe('集成测试', () => {
  it('完整工作流', async () => {
    // 1. 初始化
    const manager = new Manager();

    // 2. 执行操作
    const result = await manager.doSomething();

    // 3. 验证结果
    expect(result).toBeDefined();

    // 4. 清理
    manager.dispose();
  });
});
```

---

## 测试文件组织

### 测试文件模板

```typescript
/**
 * AnimationManager 白盒测试
 * 测试覆盖目标: 语句覆盖 100%
 */

import { AnimationManager } from '../AnimationManager';

// Mock 外部依赖
jest.mock('three-stdlib', () => ({
  // ... mock 配置
}));

// 共享测试变量
let manager: AnimationManager;
let consoleLogSpy: jest.SpyInstance;

describe('AnimationManager', () => {
  beforeEach(() => {
    jest.clearAllMocks();
    consoleLogSpy = jest.spyOn(console, 'log').mockImplementation();
    manager = new AnimationManager();
  });

  afterEach(() => {
    consoleLogSpy.mockRestore();
  });

  // ============================================
  // 测试用例分组
  // ============================================
  describe('constructor()', () => {
    it('应正确初始化', () => {
      // ...
    });
  });

  // ... 更多测试组
});
```

---

## 常见问题解决

### 1. 测试超时

```typescript
// 增加超时时间
it('长时间操作', async () => {
  // ...
}, 60000);  // 60秒

// 或在 jest.config.js 中全局配置
// testTimeout: 60000
```

### 2. Mock 不生效

确保：
- `jest.mock()` 在文件顶部，在任何 import 之前
- 使用 `jest.clearAllMocks()` 在 `beforeEach` 中清理状态
- 检查 mock 是否返回了正确的实例

### 3. 异步测试不等待

```typescript
// 错误：忘记 await
it('测试', () => {
  loader.loadModel(options);  // 缺少 await
  expect(mockFn).toHaveBeenCalled();  // 可能过早断言
});

// 正确
it('测试', async () => {
  await loader.loadModel(options);
  expect(mockFn).toHaveBeenCalled();
});
```

### 4. 私有属性访问

```typescript
// 使用类型断言访问私有成员
(manager as any).privateProperty = 'test';
expect((manager as any).internalState).toBe('expected');
```

---

## 最佳实践

1. **测试命名清晰**：使用中文或英文描述测试意图
2. **一个测试一个断言**：便于定位问题
3. **保持测试独立**：避免测试之间共享状态
4. **覆盖边界条件**：空值、极限值、错误输入
5. **Mock 外部依赖**：单元测试不应依赖外部系统
6. **及时清理资源**：在 `afterEach` 中恢复 mock 和清理资源
7. **记录不可达代码**：分析并文档化无法覆盖的代码分支

---

## 覆盖率目标

| 项目类型 | 语句覆盖 | 分支覆盖 |
|----------|----------|----------|
| 关键业务逻辑 | 100% | 90%+ |
| 一般功能模块 | 90%+ | 80%+ |
| 工具函数 | 100% | 100% |
| UI 组件 | 80%+ | 70%+ |

---

## 参考资料

- [Jest 官方文档](https://jestjs.io/docs/getting-started)
- [Testing Library](https://testing-library.com/)
- [ts-jest 配置](https://kulshekhar.github.io/ts-jest/)