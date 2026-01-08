---
name: core-components
description: 核心组件库与设计系统模式；在构建 UI、使用设计令牌或核心组件时启用。
---

# Core Components

## 设计系统概览

优先使用核心组件库，而不是原生平台组件，以保持统一的样式与交互。

## 设计令牌（Design Tokens）

**绝不要硬编码值，必须使用 design token。**

### 间距令牌

```tsx
// 正确：使用 token
<Box padding="$4" marginBottom="$2" />

// 错误：硬编码
<Box padding={16} marginBottom={8} />
```

| Token | 值 |
|-------|----|
| `$1` | 4px |
| `$2` | 8px |
| `$3` | 12px |
| `$4` | 16px |
| `$6` | 24px |
| `$8` | 32px |

### 颜色令牌

```tsx
// 正确：语义颜色
<Text color="$textPrimary" />
<Box backgroundColor="$backgroundSecondary" />

// 错误：硬编码颜色
<Text color="#333333" />
<Box backgroundColor="rgb(245, 245, 245)" />
```

| 语义 Token | 用途 |
|------------|------|
| `$textPrimary` | 主文本 |
| `$textSecondary` | 辅助文本 |
| `$textTertiary` | 禁用/提示文本 |
| `$primary500` | 品牌/强调色 |
| `$statusError` | 错误状态 |
| `$statusSuccess` | 成功状态 |

### 排版令牌

```tsx
<Text fontSize="$lg" fontWeight="$semibold" />
```

| Token | Size |
|-------|------|
| `$xs` | 12px |
| `$sm` | 14px |
| `$md` | 16px |
| `$lg` | 18px |
| `$xl` | 20px |
| `$2xl` | 24px |

## 核心组件

### Box

支持 token 的基础布局组件：

```tsx
<Box
  padding="$4"
  backgroundColor="$backgroundPrimary"
  borderRadius="$lg"
>
  {children}
</Box>
```

### HStack / VStack

水平与垂直布局：

```tsx
<HStack gap="$3" alignItems="center">
  <Icon name="user" />
  <Text>Username</Text>
</HStack>

<VStack gap="$4" padding="$4">
  <Heading>Title</Heading>
  <Text>Content</Text>
</VStack>
```

### Text

支持 token 的排版组件：

```tsx
<Text
  fontSize="$lg"
  fontWeight="$semibold"
  color="$textPrimary"
>
  Hello World
</Text>
```

### Button

带多种变体的交互按钮：

```tsx
<Button
  onPress={handlePress}
  variant="solid"
  size="md"
  isLoading={loading}
  isDisabled={disabled}
>
  Click Me
</Button>
```

| Variant | 用途 |
|---------|------|
| `solid` | 主要操作 |
| `outline` | 次要操作 |
| `ghost` | 第三梯队/弱提示 |
| `link` | 行内操作 |

### Input

带校验信息的表单输入：

```tsx
<Input
  value={value}
  onChangeText={setValue}
  placeholder="Enter text"
  error={touched ? errors.field : undefined}
  label="Field Name"
/>
```

### Card

内容卡片容器：

```tsx
<Card padding="$4" gap="$3">
  <CardHeader>
    <Heading size="sm">Card Title</Heading>
  </CardHeader>
  <CardBody>
    <Text>Card content</Text>
  </CardBody>
</Card>
```

## 布局模式

### Screen 布局

```tsx
const MyScreen = () => (
  <Screen>
    <ScreenHeader title="Page Title" />
    <ScreenContent padding="$4">
      {/* Content */}
    </ScreenContent>
  </Screen>
);
```

### 表单布局

```tsx
<VStack gap="$4" padding="$4">
  <Input label="Name" {...nameProps} />
  <Input label="Email" {...emailProps} />
  <Button isLoading={loading}>Submit</Button>
</VStack>
```

### 列表项布局

```tsx
<HStack
  padding="$4"
  gap="$3"
  alignItems="center"
  borderBottomWidth={1}
  borderColor="$borderLight"
>
  <Avatar source={{ uri: imageUrl }} size="md" />
  <VStack flex={1}>
    <Text fontWeight="$semibold">{title}</Text>
    <Text color="$textSecondary" fontSize="$sm">{subtitle}</Text>
  </VStack>
  <Icon name="chevron-right" color="$textTertiary" />
</HStack>
```

## 反模式

```tsx
// 错误：硬编码样式
<View style={{ padding: 16, backgroundColor: '#fff' }}>

// 正确：使用 token
<Box padding="$4" backgroundColor="$backgroundPrimary">


// 错误：直接使用原生组件
import { View, Text } from 'react-native';

// 正确：使用核心组件
import { Box, Text } from 'components/core';


// 错误：内联样式
<Text style={{ fontSize: 18, fontWeight: '600' }}>

// 正确：token 属性
<Text fontSize="$lg" fontWeight="$semibold">
```

## 组件 Props 模式

创建新组件时使用 token 化的 props：

```tsx
interface CardProps {
  padding?: '$2' | '$4' | '$6';
  variant?: 'elevated' | 'outlined' | 'filled';
  children: React.ReactNode;
}

const Card = ({ padding = '$4', variant = 'elevated', children }: CardProps) => (
  <Box
    padding={padding}
    backgroundColor="$backgroundPrimary"
    borderRadius="$lg"
    {...variantStyles[variant]}
  >
    {children}
  </Box>
);
```

## 与其他技能协作

- **react-ui-patterns**：使用核心组件实现 UI 状态
- **testing-patterns**：在测试中 mock 核心组件
- **storybook**：记录组件变体
