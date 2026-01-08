---
name: testing-patterns
description: Jest 测试模式、工厂函数、mock 策略与 TDD 流程；在编写单元测试、创建测试工厂或遵循红-绿-重构周期时使用。
---

# 测试模式与工具

## 测试理念

**测试驱动开发（TDD）：**
- 先写失败的测试
- 仅实现能让其通过的最小代码
- 通过后再重构
- 没有失败测试时禁止写生产代码

**行为驱动测试：**
- 关注行为而非实现
- 聚焦公共 API 与业务需求
- 避免测试内部细节
- 使用描述行为的测试名称

**工厂模式：**
- 创建 `getMockX(overrides?: Partial<X>)`
- 提供合理默认值
- 允许按需覆盖字段
- 让测试保持 DRY 且易维护

## 测试工具

### 自定义 render 函数

创建一个包含项目 Provider 的 render：

```typescript
// src/utils/testUtils.tsx
import { render } from '@testing-library/react-native';
import { ThemeProvider } from './theme';

export const renderWithTheme = (ui: React.ReactElement) => {
  return render(
    <ThemeProvider>{ui}</ThemeProvider>
  );
};
```

**使用方式：**
```typescript
import { renderWithTheme } from 'utils/testUtils';
import { screen } from '@testing-library/react-native';

it('should render component', () => {
  renderWithTheme(<MyComponent />);
  expect(screen.getByText('Hello')).toBeTruthy();
});
```

## 工厂模式

### 组件 Props 工厂

```typescript
import { ComponentProps } from 'react';

const getMockMyComponentProps = (
  overrides?: Partial<ComponentProps<typeof MyComponent>>
) => {
  return {
    title: 'Default Title',
    count: 0,
    onPress: jest.fn(),
    isLoading: false,
    ...overrides,
  };
};

// 测试用例
it('should render with custom title', () => {
  const props = getMockMyComponentProps({ title: 'Custom Title' });
  renderWithTheme(<MyComponent {...props} />);
  expect(screen.getByText('Custom Title')).toBeTruthy();
});
```

### 数据工厂

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

const getMockUser = (overrides?: Partial<User>): User => {
  return {
    id: '123',
    name: 'John Doe',
    email: 'john@example.com',
    role: 'user',
    ...overrides,
  };
};

// 示例
it('should display admin badge for admin users', () => {
  const user = getMockUser({ role: 'admin' });
  renderWithTheme(<UserCard user={user} />);
  expect(screen.getByText('Admin')).toBeTruthy();
});
```

## Mock 模式

### Mock 模块

```typescript
// 整体 mock 模块
jest.mock('utils/analytics');

// 使用 factory 自定义
jest.mock('utils/analytics', () => ({
  Analytics: {
    logEvent: jest.fn(),
  },
}));

// 在测试中获取 mock
const mockLogEvent = jest.requireMock('utils/analytics').Analytics.logEvent;
```

### Mock GraphQL Hooks

```typescript
jest.mock('./GetItems.generated', () => ({
  useGetItemsQuery: jest.fn(),
}));

const mockUseGetItemsQuery = jest.requireMock(
  './GetItems.generated'
).useGetItemsQuery as jest.Mock;

// 测试中
mockUseGetItemsQuery.mockReturnValue({
  data: { items: [] },
  loading: false,
  error: undefined,
});
```

## 测试结构

```typescript
describe('ComponentName', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('Rendering', () => {
    it('should render component with default props', () => {});
    it('should render loading state when loading', () => {});
  });

  describe('User interactions', () => {
    it('should call onPress when button is clicked', async () => {});
  });

  describe('Edge cases', () => {
    it('should handle empty data gracefully', () => {});
  });
});
```

## 查询断言

```typescript
// 元素必须存在
expect(screen.getByText('Hello')).toBeTruthy();

// 元素不应存在
expect(screen.queryByText('Goodbye')).toBeNull();

// 异步出现的元素
await waitFor(() => {
  expect(screen.findByText('Loaded')).toBeTruthy();
});
```

## 用户交互模式

```typescript
import { fireEvent, screen } from '@testing-library/react-native';

it('should submit form on button click', async () => {
  const onSubmit = jest.fn();
  renderWithTheme(<LoginForm onSubmit={onSubmit} />);

  fireEvent.changeText(screen.getByLabelText('Email'), 'user@example.com');
  fireEvent.changeText(screen.getByLabelText('Password'), 'password123');
  fireEvent.press(screen.getByTestId('login-button'));

  await waitFor(() => {
    expect(onSubmit).toHaveBeenCalled();
  });
});
```

## 反模式

### 测试 mock 行为而非真实行为

```typescript
// 错误：只断言 mock 被调用
expect(mockFetchData).toHaveBeenCalled();

// 正确：断言真实输出
expect(screen.getByText('John Doe')).toBeTruthy();
```

### 不使用工厂

```typescript
// 错误：重复、不一致的数据
it('test 1', () => {
  const user = { id: '1', name: 'John', email: 'john@test.com', role: 'user' };
});
it('test 2', () => {
  const user = { id: '2', name: 'Jane', email: 'jane@test.com' }; // 缺少 role!
});

// 正确：复用工厂
const user = getMockUser({ name: 'Custom Name' });
```

## 最佳实践

1. **始终使用工厂函数** 生成 props/数据
2. **测试行为而非实现细节**
3. **使用描述性测试名称**
4. **用 describe 分组** 保持结构清晰
5. **在测试间清除 mock**
6. **单测聚焦**——每个测试只覆盖一个行为

## 执行测试

```bash
# 全量测试
npm test

# 覆盖率
npm run test:coverage

# 指定文件
npm test ComponentName.test.tsx
```

## 与其他技能协作

- **react-ui-patterns**：覆盖 Loading/Error/Empty/Success 全状态
- **systematic-debugging**：修复前先写能复现 bug 的测试
