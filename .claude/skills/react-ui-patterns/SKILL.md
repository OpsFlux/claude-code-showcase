---
name: react-ui-patterns
description: 现代 React UI 模式，涵盖加载状态、错误处理和数据获取。在构建组件、处理异步数据或管理界面状态时使用。
---

# React UI 模式

## 核心原则

1. **不要展示过期 UI**——只有在真正加载时才显示转圈
2. **始终暴露错误**——用户必须知道失败
3. **乐观更新**——让界面感觉即时
4. **渐进披露**——数据到达就展示
5. **优雅降级**——部分数据优于无数据

## Loading 状态模式

### 黄金法则

**只有在没有可展示数据时才显示加载指示。**

```typescript
// 正确：仅在无数据时展示 Loading
const { data, loading, error, refetch } = useGetItemsQuery();

if (error) return <ErrorState error={error} onRetry={refetch} />;
if (loading && !data) return <LoadingState />;
if (!data?.items.length) return <EmptyState />;

return <ItemList items={data.items} />;
```

```typescript
// 错误：即使有缓存数据也显示 Spinner
if (loading) return <LoadingState />; // 重新获取时闪烁！
```

### Loading 状态决策树

```
是否有错误？
  → 是：展示可重试的错误状态
  → 否：继续

是否正在加载且没有任何数据？
  → 是：显示加载指示（Spinner/Skeleton）
  → 否：继续

是否有数据？
  → 有且非空：展示数据
  → 有但为空：展示空状态
  → 没有：回退到加载状态
```

### Skeleton vs Spinner

| 使用 Skeleton 的场景 | 使用 Spinner 的场景 |
|---------------------|--------------------|
| 已知内容结构 | 内容形态未知 |
| 列表/卡片布局 | 模态操作 |
| 首屏加载 | 按钮提交流程 |
| 占位内容 | 内联小操作 |

## 错误处理模式

### 错误处理层级

```
1. 行内错误（字段级）→ 表单校验
2. Toast → 可恢复错误，用户可重试
3. 错误横幅 → 页面级错误但仍可用
4. 错误整屏 → 不可恢复，需要操作
```

### 一定要展示错误

**关键：绝不能静默吞掉错误。**

```typescript
// 正确：始终呈现错误
define const [createItem, { loading }] = useCreateItemMutation({
  onCompleted: () => {
    toast.success({ title: 'Item created' });
  },
  onError: (error) => {
    console.error('createItem failed:', error);
    toast.error({ title: 'Failed to create item' });
  },
});

// 错误：用户毫无感知
const [createItem] = useCreateItemMutation({
  onError: (error) => {
    console.error(error); // 用户看不到任何提示
  },
});
```

### 错误状态组件

```typescript
interface ErrorStateProps {
  error: Error;
  onRetry?: () => void;
  title?: string;
}

const ErrorState = ({ error, onRetry, title }: ErrorStateProps) => (
  <div className="error-state">
    <Icon name="exclamation-circle" />
    <h3>{title ?? 'Something went wrong'}</h3>
    <p>{error.message}</p>
    {onRetry && (
      <Button onClick={onRetry}>Try Again</Button>
    )}
  </div>
);
```

## 按钮状态模式

### 按钮加载状态

```tsx
<Button
  onClick={handleSubmit}
  isLoading={isSubmitting}
  disabled={!isValid || isSubmitting}
>
  Submit
</Button>
```

### 操作期间禁用

**关键：异步过程中必须禁用触发控件。**

```tsx
// 正确：加载期间禁用按钮
<Button
  disabled={isSubmitting}
  isLoading={isSubmitting}
  onClick={handleSubmit}
>
  Submit
</Button>

// 错误：可以点多次
<Button onClick={handleSubmit}>
  {isSubmitting ? 'Submitting...' : 'Submit'}
</Button>
```

## 空状态

### 空状态要求

所有列表/集合都必须提供空状态：

```tsx
// 错误：无空状态
return <FlatList data={items} />;

// 正确：明确的空状态
return (
  <FlatList
    data={items}
    ListEmptyComponent={<EmptyState />}
  />
);
```

### 语境化的空状态

```tsx
// 搜索无结果
<EmptyState
  icon="search"
  title="No results found"
  description="Try different search terms"
/>

// 尚无数据
<EmptyState
  icon="plus-circle"
  title="No items yet"
  description="Create your first item"
  action={{ label: 'Create Item', onClick: handleCreate }}
/>
```

## 表单提交模式

```tsx
const MyForm = () => {
  const [submit, { loading }] = useSubmitMutation({
    onCompleted: handleSuccess,
    onError: handleError,
  });

  const handleSubmit = async () => {
    if (!isValid) {
      toast.error({ title: 'Please fix errors' });
      return;
    }
    await submit({ variables: { input: values } });
  };

  return (
    <form>
      <Input
        value={values.name}
        onChange={handleChange('name')}
        error={touched.name ? errors.name : undefined}
      />
      <Button
        type="submit"
        onClick={handleSubmit}
        disabled={!isValid || loading}
        isLoading={loading}
      >
        Submit
      </Button>
    </form>
  );
};
```

## 反模式

### Loading 状态

```typescript
// 错误：即使有数据也转圈
if (loading) return <Spinner />;

// 正确：无数据时才转圈
if (loading && !data) return <Spinner />;
```

### 错误处理

```typescript
// 错误：吞掉异常
try {
  await mutation();
} catch (e) {
  console.log(e); // 用户毫不知情
}

// 正确：抛给用户
onError: (error) => {
  console.error('operation failed:', error);
  toast.error({ title: 'Operation failed' });
}
```

### 按钮状态

```typescript
// 错误：提交时未禁用
<Button onClick={submit}>Submit</Button>

// 正确：禁用+加载
<Button onClick={submit} disabled={loading} isLoading={loading}>
  Submit
</Button>
```

## 检查清单

在完成任何 UI 组件之前：

**界面状态：**
- [ ] 错误状态已处理并展示
- [ ] Loading 仅在无数据时出现
- [ ] 集合具备空状态
- [ ] 异步期间按钮禁用
- [ ] 按钮根据需要展示加载

**数据与 Mutation：**
- [ ] Mutation 具备 onError
- [ ] 所有用户操作都有反馈（toast/视觉）

## 与其他技能协作

- **graphql-schema**：结合 Mutation 错误处理模式
- **testing-patterns**：为 Loading/Error/Empty/Success 编写测试
- **formik-patterns**：落实表单提交流程
