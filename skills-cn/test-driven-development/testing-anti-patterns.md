# 测试反模式

**在以下情况下加载此参考：** 编写或修改测试、添加 mock，或者被诱惑向生产代码添加仅测试用的方法。

## 概述

测试必须验证真实行为，而不是 mock 行为。Mock 是隔离的手段，不是被测试的对象。

**核心原则：** 测试代码做了什么，而不是 mock 做了什么。

**严格遵循 TDD 可以防止这些反模式。**

## 铁律

```
1. 永远不 (NEVER) 测试 mock 行为
2. 永远不 (NEVER) 向生产类添加仅测试用的方法
3. 永远不 (NEVER) 在不理解依赖的情况下进行 mock
```

## 反模式 1：测试 Mock 行为

**违规示例：**
```typescript
// ❌ BAD: Testing that the mock exists
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```

**为什么这是错的：**
- 你在验证 mock 能工作，而不是组件能工作
- 当 mock 存在时测试通过，不存在时失败
- 对真实行为没有任何说明

**你的人类伙伴 (your human partner) 的纠正：** "我们是在测试一个 mock 的行为吗？"

**修复方法：**
```typescript
// ✅ GOOD: Test real component or don't mock it
test('renders sidebar', () => {
  render(<Page />);  // Don't mock sidebar
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});

// OR if sidebar must be mocked for isolation:
// Don't assert on the mock - test Page's behavior with sidebar present
```

### 门控函数 (Gate Function)

```
在断言任何 mock 元素之前：
  问："我是在测试真实组件行为还是仅仅 mock 的存在？"

  如果是测试 mock 的存在：
    停止 - 删除断言或取消 mock 该组件

  转而测试真实行为
```

## 反模式 2：生产代码中的仅测试用方法

**违规示例：**
```typescript
// ❌ BAD: destroy() only used in tests
class Session {
  async destroy() {  // Looks like production API!
    await this._workspaceManager?.destroyWorkspace(this.id);
    // ... cleanup
  }
}

// In tests
afterEach(() => session.destroy());
```

**为什么这是错的：**
- 生产类被仅测试用的代码污染
- 如果在生产环境中意外调用会很危险
- 违反 YAGNI 和关注点分离
- 混淆了对象生命周期和实体生命周期

**修复方法：**
```typescript
// ✅ GOOD: Test utilities handle test cleanup
// Session has no destroy() - it's stateless in production

// In test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}

// In tests
afterEach(() => cleanupSession(session));
```

### 门控函数 (Gate Function)

```
在向生产类添加任何方法之前：
  问："这个方法只有测试在用吗？"

  如果是：
    停止 - 不要添加它
    把它放到测试工具中

  问："这个类拥有此资源的生命周期吗？"

  如果不是：
    停止 - 不是添加此方法的正确类
```

## 反模式 3：不理解就进行 Mock

**违规示例：**
```typescript
// ❌ BAD: Mock breaks test logic
test('detects duplicate server', () => {
  // Mock prevents config write that test depends on!
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));

  await addServer(config);
  await addServer(config);  // Should throw - but won't!
});
```

**为什么这是错的：**
- 被 mock 的方法有测试依赖的副作用（写入配置）
- "为了安全"过度 mock 破坏了实际行为
- 测试因错误原因通过或神秘失败

**修复方法：**
```typescript
// ✅ GOOD: Mock at correct level
test('detects duplicate server', () => {
  // Mock the slow part, preserve behavior test needs
  vi.mock('MCPServerManager'); // Just mock slow server startup

  await addServer(config);  // Config written
  await addServer(config);  // Duplicate detected ✓
});
```

### 门控函数 (Gate Function)

```
在 mock 任何方法之前：
  停止 - 先不要 mock

  1. 问："真实方法有哪些副作用？"
  2. 问："这个测试依赖于哪些副作用？"
  3. 问："我完全理解这个测试需要什么吗？"

  如果依赖于副作用：
    在更低层 mock（实际的慢速/外部操作）
    或使用保留必要行为的测试替身 (test doubles)
    而不是 mock 测试依赖的高层方法

  如果不确定测试依赖什么：
    先用真实实现运行测试
    观察实际需要发生什么
    然后在正确的层级添加最小化的 mock

  红旗信号：
    - "我 mock 一下这个以确保安全"
    - "这个可能很慢，最好 mock 它"
    - 在不理解依赖链的情况下进行 mock
```

## 反模式 4：不完整的 Mock

**违规示例：**
```typescript
// ❌ BAD: Partial mock - only fields you think you need
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // Missing: metadata that downstream code uses
};

// Later: breaks when code accesses response.metadata.requestId
```

**为什么这是错的：**
- **不完整的 mock 隐藏结构假设** - 你只 mock 了你所知道的字段
- **下游代码可能依赖你没有包含的字段** - 静默失败
- **测试通过但集成失败** - Mock 不完整，真实 API 完整
- **虚假的信心** - 测试对真实行为没有证明任何东西

**铁律：** Mock 完整的真实数据结构，而不仅仅是当前测试使用的字段。

**修复方法：**
```typescript
// ✅ GOOD: Mirror real API completeness
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
  // All fields real API returns
};
```

### 门控函数 (Gate Function)

```
在创建 mock 响应之前：
  检查："真实 API 响应包含哪些字段？"

  行动：
    1. 从文档/示例中检查实际的 API 响应
    2. 包含系统可能在下游使用的所有字段
    3. 验证 mock 完全匹配真实响应结构

  关键：
    如果你要创建 mock，你必须理解完整的结构
    当代码依赖省略的字段时，不完整的 mock 会静默失败

  如果不确定：包含所有已记录字段
```

## 反模式 5：集成测试作为事后想法

**违规示例：**
```
✅ 实现完成
❌ 没有写测试
"准备测试"
```

**为什么这是错的：**
- 测试是实现的一部分，不是可选的后续工作
- TDD 本可以捕获这一点
- 没有测试就不能声称完成

**修复方法：**
```
TDD 循环：
1. 写失败测试
2. 实现以通过
3. 重构
4. 然后声称完成
```

## 当 Mock 变得过于复杂

**警告信号：**
- Mock 设置比测试逻辑更长
- Mock 所有东西以让测试通过
- Mock 缺少真实组件有的方法
- 当 mock 变更时测试就坏掉

**你的人类伙伴 (your human partner) 的问题：** "我们真的需要在这里使用 mock 吗？"

**考虑：** 用真实组件的集成测试通常比复杂的 mock 更简单

## TDD 防止这些反模式

**为什么 TDD 有帮助：**
1. **先写测试** → 迫使你思考你实际在测试什么
2. **看着它失败** → 确认测试测试的是真实行为，而不是 mock
3. **最简实现** → 不会悄悄混入仅测试用的方法
4. **真实依赖** → 在 mock 之前你看到测试实际需要什么

**如果你在测试 mock 行为，你违反了 TDD** - 你在没有先看着测试在真实代码上失败的情况下添加了 mock。

## 快速参考

| 反模式 | 修复 |
|--------------|-----|
| 断言 mock 元素 | 测试真实组件或取消 mock |
| 生产中的仅测试用方法 | 移到测试工具中 |
| 不理解就 mock | 先理解依赖，最小化 mock |
| 不完整的 mock | 完整镜像真实 API |
| 测试作为事后想法 | TDD - 测试先行 |
| 过度复杂的 mock | 考虑集成测试 |

## 红旗信号

- 断言检查 `*-mock` 测试 ID
- 只在测试文件中调用的方法
- Mock 设置占测试的 >50%
- 移除 mock 后测试失败
- 无法解释为什么需要 mock
- "为了安全"而 mock

## 底线

**Mock 是隔离的工具，不是测试的对象。**

如果 TDD 揭示你在测试 mock 行为，说明你走错了。

修复：测试真实行为，或质疑你为什么一开始就要 mock。
