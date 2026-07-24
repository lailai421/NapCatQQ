# Frontend Component & Form Guidelines

> Real conventions from `packages/napcat-webui-frontend/src/`.

---

## Tech Stack

- **React** with TypeScript (functional components only)
- **UI Library**: `@heroui` (HeroUI) — Button, Input, Card, Image, etc.
- **Form Management**: `react-hook-form` with `Controller`
- **Notifications**: `react-hot-toast`
- **Routing**: `react-router-dom` (`useNavigate`)
- **HTTP Client**: `axios` (via `src/utils/request.ts`)
- **Animations**: `motion/react` (Framer Motion)
- **Local Storage**: `@uidotdev/usehooks` (`useLocalStorage`)

---

## Component File Conventions

- One component per file, default export
- Located under `src/components/` or `src/pages/`
- File extension: `.tsx`
- Component name: `PascalCase`

Basic structure:
```tsx
import { Button } from '@heroui/button';
import { Input } from '@heroui/input';
import { Controller, useForm } from 'react-hook-form';
import toast from 'react-hot-toast';

const MyComponent = () => {
  const {
    control,
    handleSubmit,
    formState: { isSubmitting, errors },
    reset,
    watch,
  } = useForm<MyFormData>({ defaultValues: { ... } });

  const onSubmit = handleSubmit(async (data) => {
    try {
      await SomeApi.call(data);
      toast.success('成功');
    } catch (error) {
      toast.error(`失败: ${(error as Error).message}`);
    }
  });

  return (
    <>
      <title>页面标题 - NapCat WebUI</title>
      <Controller
        control={control}
        name='fieldName'
        rules={{ required: '不能为空' }}
        render={({ field }) => (
          <Input {...field} label='标签' isInvalid={!!errors.fieldName} errorMessage={errors.fieldName?.message} />
        )}
      />
      <Button color='primary' onPress={onSubmit} isLoading={isSubmitting}>提交</Button>
    </>
  );
};

export default MyComponent;
```

---

## Form Patterns

### react-hook-form + Controller

Every form uses `useForm` + `Controller` from react-hook-form. Never uncontrolled inputs.

```tsx
// packages/napcat-webui-frontend/src/pages/dashboard/config/change_password.tsx
const { control, handleSubmit, formState: { isSubmitting, errors }, reset, watch } = useForm<{
  oldToken: string;
  newToken: string;
}>({
  defaultValues: { oldToken: '', newToken: '' },
});
```

### Validation Rules

Validation is inline via `rules` prop on `Controller`:

```tsx
<Controller
  control={control}
  name='newToken'
  rules={{
    required: '新密码不能为空',
    minLength: { value: 6, message: '新密码至少需要6个字符' },
    validate: (value) => {
      if (!/[a-zA-Z]/.test(value)) return '新密码必须包含字母';
      if (!/[0-9]/.test(value)) return '新密码必须包含数字';
      return true;
    },
  }}
  render={({ field }) => (
    <Input {...field} isInvalid={!!errors.newToken} errorMessage={errors.newToken?.message} />
  )}
/>
```

### Submit Handler

Always `async` with try/catch + toast:

```tsx
const onSubmit = handleSubmit(async (data) => {
  try {
    await WebUIManager.changePassword(data.oldToken, data.newToken);
    toast.success('修改成功');
  } catch (error) {
    toast.error(`修改失败: ${(error as Error).message}`);
  }
});
```

### Buttons

Use `onPress` (HeroUI convention), not `onClick`:
```tsx
<Button color='primary' onPress={onSubmit} isLoading={isSubmitting}>确认</Button>
```

---

## API Call Patterns

### Controller Class Pattern

Backend APIs are called through static methods on controller classes (`src/controllers/webui_manager.ts`):

```ts
export default class WebUIManager {
  public static async getWebUIConfig() {
    const { data } = await serverRequest.get<ServerResponse<WebUIConfig>>('/WebUIConfig/GetConfig');
    return data.data;
  }

  public static async updateWebUIConfig(config: Partial<WebUIConfig>) {
    const { data } = await serverRequest.post<ServerResponse<boolean>>('/WebUIConfig/UpdateConfig', config);
    return data.data;
  }
}
```

### axios Configuration

`src/utils/request.ts` creates a pre-configured axios instance:

```ts
export const serverRequest = axios.create({ timeout: 30000 });
```

**Request interceptor** auto-injects:
- `baseURL: '/api'`
- `Authorization: Bearer <token>` from localStorage

**Response interceptor** auto-handles:
- `code !== 0` → throws Error with `message`
- `message === 'Unauthorized'` → clears token, reloads page
- `Content-Type: application/octet-stream` → passes through for file downloads

### Response Type

```ts
interface ServerResponse<T> {
  code: number;   // 0 = success, -1 = error
  message: string;
  data?: T;
}
```

---

## Toast Notifications

Always use `react-hot-toast` for user feedback:

```ts
toast.success('保存成功');
toast.error(`保存失败: ${msg}`);
```

---

## Styling

Uses HeroUI's Tailwind-based class system. Common patterns:

```html
className='flex flex-col gap-4'
className='text-sm text-default-600 dark:text-default-400'
className='bg-default-50 border border-default-200 rounded-lg p-4'
```

Theme-aware colors use `default-*` tokens that auto-switch with dark mode.
