---
name: formik-patterns
description: Formik 表单处理与校验模式；在构建表单、实现验证或处理提交时使用。
---

# Formik 模式

## 基础表单

```tsx
import { useFormik } from 'formik';
import * as yup from 'yup';

const validationSchema = yup.object({
  email: yup.string().email('Invalid email').required('Email is required'),
  password: yup.string().min(8, 'Min 8 characters').required('Password is required'),
});

const LoginForm = () => {
  const formik = useFormik({
    initialValues: {
      email: '',
      password: '',
    },
    validationSchema,
    onSubmit: async (values) => {
      await loginMutation({ variables: { input: values } });
    },
  });

  return (
    <VStack gap="$4">
      <Input
        label="Email"
        value={formik.values.email}
        onChangeText={formik.handleChange('email')}
        onBlur={formik.handleBlur('email')}
        error={formik.touched.email ? formik.errors.email : undefined}
        keyboardType="email-address"
        autoCapitalize="none"
      />

      <Input
        label="Password"
        value={formik.values.password}
        onChangeText={formik.handleChange('password')}
        onBlur={formik.handleBlur('password')}
        error={formik.touched.password ? formik.errors.password : undefined}
        secureTextEntry
      />

      <Button
        onPress={formik.handleSubmit}
        isDisabled={!formik.isValid || formik.isSubmitting}
        isLoading={formik.isSubmitting}
      >
        Login
      </Button>
    </VStack>
  );
};
```

## 校验模式

### 常见示例

```typescript
import * as yup from 'yup';

// 邮箱
email: yup.string()
  .email('Invalid email address')
  .required('Email is required')

// 密码要求
password: yup.string()
  .min(8, 'Must be at least 8 characters')
  .matches(/[a-z]/, 'Must contain lowercase letter')
  .matches(/[A-Z]/, 'Must contain uppercase letter')
  .matches(/[0-9]/, 'Must contain number')
  .required('Password is required')

// 确认密码
confirmPassword: yup.string()
  .oneOf([yup.ref('password')], 'Passwords must match')
  .required('Please confirm password')

// 手机号
phone: yup.string()
  .matches(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number')
  .required('Phone is required')

// 可选字段但存在时需要校验
website: yup.string()
  .url('Must be a valid URL')
  .nullable()

// 数字区间
quantity: yup.number()
  .min(1, 'Minimum 1')
  .max(100, 'Maximum 100')
  .required('Quantity required')

// 数组至少一个条目
tags: yup.array()
  .of(yup.string())
  .min(1, 'Select at least one tag')
```

### 条件校验

```typescript
const schema = yup.object({
  hasCompany: yup.boolean(),
  companyName: yup.string().when('hasCompany', {
    is: true,
    then: (schema) => schema.required('Company name required'),
    otherwise: (schema) => schema.nullable(),
  }),
});
```

## 字段辅助

### 通用 Input Helper

```tsx
const getFieldProps = (name: keyof typeof formik.values) => ({
  value: formik.values[name],
  onChangeText: formik.handleChange(name),
  onBlur: formik.handleBlur(name),
  error: formik.touched[name] ? formik.errors[name] : undefined,
});

// 用法
<Input label="Email" {...getFieldProps('email')} />
```

### Select/Picker

```tsx
<Select
  label="Country"
  value={formik.values.country}
  onValueChange={(value) => formik.setFieldValue('country', value)}
  error={formik.touched.country ? formik.errors.country : undefined}
  options={countryOptions}
/>
```

## GraphQL 表单提交

```tsx
const CreateItemForm = () => {
  const [createItem] = useCreateItemMutation({
    onCompleted: () => {
      toast.success({ title: 'Item created' });
      navigation.goBack();
    },
    onError: (error) => {
      console.error('createItem failed:', error);
      toast.error({ title: 'Failed to create item' });
    },
  });

  const formik = useFormik({
    initialValues: { name: '', description: '' },
    validationSchema,
    onSubmit: async (values, { setSubmitting }) => {
      try {
        await createItem({ variables: { input: values } });
      } finally {
        setSubmitting(false);
      }
    },
  });

  return (
    <VStack gap="$4">
      {/* 表单字段 */}
      <Button
        onPress={formik.handleSubmit}
        isDisabled={!formik.isValid || formik.isSubmitting}
        isLoading={formik.isSubmitting}
      >
        Create
      </Button>
    </VStack>
  );
};
```

## 带初始值的编辑表单

```tsx
const EditItemForm = ({ item }: { item: Item }) => {
  const [updateItem] = useUpdateItemMutation({
    onCompleted: () => toast.success({ title: 'Saved' }),
    onError: (error) => {
      console.error('updateItem failed:', error);
      toast.error({ title: 'Save failed' });
    },
  });

  const formik = useFormik({
    initialValues: {
      name: item.name,
      description: item.description ?? '',
    },
    enableReinitialize: true, // props 变化时重置
    validationSchema,
    onSubmit: async (values) => {
      await updateItem({
        variables: { id: item.id, input: values },
      });
    },
  });

  const hasChanges = formik.dirty;

  return (
    <VStack gap="$4">
      {/* 字段 */}
      <Button
        onPress={formik.handleSubmit}
        isDisabled={!hasChanges || !formik.isValid || formik.isSubmitting}
        isLoading={formik.isSubmitting}
      >
        Save Changes
      </Button>
    </VStack>
  );
};
```

## Formik 状态助手

```tsx
const {
  values,
  errors,
  touched,
  isValid,
  isSubmitting,
  dirty,
  handleSubmit,
  handleChange,
  handleBlur,
  setFieldValue,
  setFieldTouched,
  resetForm,
  setSubmitting,
} = formik;
```

## 多步骤表单

```tsx
const MultiStepForm = () => {
  const [step, setStep] = useState(0);

  const formik = useFormik({
    initialValues: {
      name: '',
      email: '',
      address: '',
      city: '',
      cardNumber: '',
    },
    validationSchema: stepSchemas[step],
    onSubmit: async (values) => {
      if (step < steps.length - 1) {
        setStep(step + 1);
      } else {
        await submitOrder(values);
      }
    },
  });

  return (
    <VStack>
      {step === 0 && <PersonalInfoStep formik={formik} />}
      {step === 1 && <AddressStep formik={formik} />}
      {step === 2 && <PaymentStep formik={formik} />}

      <HStack gap="$4">
        {step > 0 && (
          <Button variant="outline" onPress={() => setStep(step - 1)}>
            Back
          </Button>
        )}
        <Button
          onPress={formik.handleSubmit}
          isDisabled={!formik.isValid}
          isLoading={formik.isSubmitting}
        >
          {step < steps.length - 1 ? 'Next' : 'Submit'}
        </Button>
      </HStack>
    </VStack>
  );
};
```

## 反模式

```tsx
// 错误：不显示校验
<Input
  value={formik.values.email}
  onChangeText={formik.handleChange('email')}
/>

// 正确：失焦后呈现错误
<Input
  value={formik.values.email}
  onChangeText={formik.handleChange('email')}
  onBlur={formik.handleBlur('email')}
  error={formik.touched.email ? formik.errors.email : undefined}
/>


// 错误：提交按钮始终可用
<Button onPress={formik.handleSubmit}>Submit</Button>

// 正确：无效或提交中禁用
<Button
  onPress={formik.handleSubmit}
  isDisabled={!formik.isValid || formik.isSubmitting}
  isLoading={formik.isSubmitting}
>
  Submit
</Button>


// 错误：Mutation 未处理错误
onSubmit: async (values) => {
  await createItem({ variables: { input: values } });
}

// 正确：捕获并提示
onSubmit: async (values, { setSubmitting }) => {
  try {
    await createItem({ variables: { input: values } });
  } catch (error) {
    toast.error({ title: 'Failed to save' });
  } finally {
    setSubmitting(false);
  }
}
```

## 与其他技能协作

- **graphql-schema**：Mutation 提交流程
- **react-ui-patterns**：加载/错误状态
- **testing-patterns**：测试表单验证与提交
