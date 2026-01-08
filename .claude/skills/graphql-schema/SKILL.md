---
name: graphql-schema
description: GraphQL 查询、Mutation 与代码生成模式；在创建 GraphQL 操作、使用 Apollo Client 或生成类型时使用。
---

# GraphQL Schema 模式

## 核心规则

1. **绝不要内联 `gql` 字面量**——创建独立 `.gql` 文件
2. **每次修改 `.gql` 后都要运行 codegen**
3. **所有 Mutation 必须提供 `onError`**
4. **只使用生成的 Hook**——不要直接写 Apollo 原生 Hook

## 文件结构

```
src/
├── components/
│   └── ItemList/
│       ├── ItemList.tsx
│       ├── GetItems.gql           # 查询定义
│       └── GetItems.generated.ts  # 自动生成（勿改）
└── graphql/
    └── mutations/
        └── CreateItem.gql         # 共享 Mutation
```

## 创建查询

### 第一步：编写 .gql

```graphql
# src/components/ItemList/GetItems.gql
query GetItems($limit: Int, $offset: Int) {
  items(limit: $limit, offset: $offset) {
    id
    name
    description
    createdAt
  }
}
```

### 第二步：运行 codegen

```bash
npm run gql:typegen
```

### 第三步：导入并使用生成的 Hook

```typescript
import { useGetItemsQuery } from './GetItems.generated';

const ItemList = () => {
  const { data, loading, error, refetch } = useGetItemsQuery({
    variables: { limit: 20, offset: 0 },
  });

  if (error) return <ErrorState error={error} onRetry={refetch} />;
  if (loading && !data) return <LoadingSkeleton />;
  if (!data?.items.length) return <EmptyState />;

  return <List items={data.items} />;
};
```

## 创建 Mutation

### 第一步：编写 .gql

```graphql
# src/graphql/mutations/CreateItem.gql
mutation CreateItem($input: CreateItemInput!) {
  createItem(input: $input) {
    id
    name
    description
  }
}
```

### 第二步：运行 codegen

```bash
npm run gql:typegen
```

### 第三步：强制加入错误处理

```typescript
import { useCreateItemMutation } from 'graphql/mutations/CreateItem.generated';

const CreateItemForm = () => {
  const [createItem, { loading }] = useCreateItemMutation({
    // 成功处理
    onCompleted: (data) => {
      toast.success({ title: 'Item created' });
      navigation.goBack();
    },
    // 必填：错误处理
    onError: (error) => {
      console.error('createItem failed:', error);
      toast.error({ title: 'Failed to create item' });
    },
    // 缓存更新
    update: (cache, { data }) => {
      if (data?.createItem) {
        cache.modify({
          fields: {
            items: (existing = []) => [...existing, data.createItem],
          },
        });
      }
    },
  });

  return (
    <Button
      onPress={() => createItem({ variables: { input: formValues } })}
      isDisabled={!isValid || loading}
      isLoading={loading}
    >
      Create
    </Button>
  );
};
```

## Mutation UI 要求

**关键：触发 Mutation 的控件必须满足：**

1. **执行过程中禁用**——防止重复点击
2. **展示加载状态**——提供视觉反馈
3. **具备 onError**——失败时提示用户
4. **成功反馈**——让用户知道操作完成

```typescript
// 正确：完整 Mutation 模式
const [submit, { loading }] = useSubmitMutation({
  onError: (error) => {
    console.error('submit failed:', error);
    toast.error({ title: 'Save failed' });
  },
  onCompleted: () => {
    toast.success({ title: 'Saved' });
  },
});

<Button
  onPress={handleSubmit}
  isDisabled={!isValid || loading}
  isLoading={loading}
>
  Submit
</Button>
```

## 查询选项

### Fetch Policy

| 策略 | 适用场景 |
|------|----------|
| `cache-first` | 数据很少变化 |
| `cache-and-network` | 需要速度也需要最新（默认） |
| `network-only` | 必须拿最新数据 |
| `no-cache` | 不缓存（极少） |

### 常用配置

```typescript
useGetItemsQuery({
  variables: { id: itemId },

  // 拉取策略
  fetchPolicy: 'cache-and-network',

  // 网络状态变化时重新渲染
  notifyOnNetworkStatusChange: true,

  // 条件不满足时跳过
  skip: !itemId,

  // 轮询更新
  pollInterval: 30000,
});
```

## 乐观更新

用于即时 UI 反馈：

```typescript
const [toggleFavorite] = useToggleFavoriteMutation({
  optimisticResponse: {
    toggleFavorite: {
      __typename: 'Item',
      id: itemId,
      isFavorite: !currentState,
    },
  },
  onError: (error) => {
    // 回滚自动发生
    console.error('toggleFavorite failed:', error);
    toast.error({ title: 'Failed to update' });
  },
});
```

### 何时不要使用乐观更新

- 可能因校验失败的操作
- 依赖服务端生成值
- 破坏性操作（删除）
- 会影响其他用户的操作

## 片段（Fragments）

```graphql
# src/graphql/fragments/ItemFields.gql
fragment ItemFields on Item {
  id
  name
  description
  createdAt
  updatedAt
}
```

在查询中使用：

```graphql
query GetItems {
  items {
    ...ItemFields
  }
}
```

## 反模式

```typescript
// 错误：内联 gql
const GET_ITEMS = gql`
  query GetItems { items { id } }
`;

// 正确：.gql + 生成的 Hook
import { useGetItemsQuery } from './GetItems.generated';


// 错误：缺少错误处理
const [mutate] = useMutation(MUTATION);

// 正确：始终处理错误
const [mutate] = useMutation(MUTATION, {
  onError: (error) => {
    console.error('mutation failed:', error);
    toast.error({ title: 'Operation failed' });
  },
});


// 错误：Mutation 期间按钮未禁用
<Button onPress={submit}>Submit</Button>

// 正确：禁用并显示加载
<Button onPress={submit} isDisabled={loading} isLoading={loading}>
  Submit
</Button>
```

## Codegen 命令

```bash
# 从 .gql 生成类型
npm run gql:typegen

# 下载 schema 并生成类型
npm run sync-types
```

## 与其他技能协作

- **react-ui-patterns**：查询的加载/错误/空态
- **testing-patterns**：在测试中 mock 生成的 Hook
- **formik-patterns**：Mutation 表单提交流程
