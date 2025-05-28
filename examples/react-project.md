---
name: "React 现代化项目模板"
description: "基于 React 18 + TypeScript + Vite 的现代化前端项目开发指南，包含完整的开发规范和最佳实践"
category: "前端框架"
author: "Agents.md Collection"
authorUrl: "https://github.com/gakeez/agents_md_collection"
tags: ["react", "typescript", "vite", "frontend", "spa"]
lastUpdated: "2024-12-19"
---

# React 现代化项目开发指南

## 项目概述

这是一个基于 React 18、TypeScript 和 Vite 的现代化前端项目模板。适用于构建高性能的单页应用程序（SPA），集成了现代化的开发工具链和最佳实践。

## 技术栈

- **前端框架**: React 18 + TypeScript
- **构建工具**: Vite
- **状态管理**: Zustand / Redux Toolkit
- **路由**: React Router v6
- **UI 组件**: Ant Design / Material-UI
- **样式方案**: Tailwind CSS / Styled-components
- **HTTP 客户端**: Axios
- **测试框架**: Vitest + React Testing Library
- **代码质量**: ESLint + Prettier + Husky

## 项目结构

```
react-project/
├── public/                 # 静态资源
│   ├── favicon.ico
│   └── index.html
├── src/
│   ├── components/         # 可复用组件
│   │   ├── common/        # 通用组件
│   │   └── ui/            # UI 组件
│   ├── pages/             # 页面组件
│   ├── hooks/             # 自定义 Hooks
│   ├── store/             # 状态管理
│   ├── services/          # API 服务
│   ├── utils/             # 工具函数
│   ├── types/             # TypeScript 类型定义
│   ├── styles/            # 全局样式
│   ├── constants/         # 常量定义
│   ├── App.tsx
│   └── main.tsx
├── tests/                 # 测试文件
├── docs/                  # 项目文档
├── .env.example          # 环境变量示例
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

## 开发规范

### 组件开发规范

1. **函数组件优先**: 使用函数组件和 Hooks
2. **TypeScript 类型**: 为所有 props 定义接口
3. **组件命名**: 使用 PascalCase，文件名与组件名一致
4. **单一职责**: 每个组件只负责一个功能

```tsx
// 示例：Button 组件
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
}

export const Button: React.FC<ButtonProps> = ({
  variant,
  size = 'medium',
  disabled = false,
  onClick,
  children
}) => {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
};
```

### 状态管理规范

使用 Zustand 进行状态管理：

```tsx
// store/userStore.ts
import { create } from 'zustand';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserState {
  user: User | null;
  isLoading: boolean;
  setUser: (user: User) => void;
  clearUser: () => void;
  setLoading: (loading: boolean) => void;
}

export const useUserStore = create<UserState>((set) => ({
  user: null,
  isLoading: false,
  setUser: (user) => set({ user }),
  clearUser: () => set({ user: null }),
  setLoading: (isLoading) => set({ isLoading }),
}));
```

### API 服务规范

```tsx
// services/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
});

// 请求拦截器
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// 响应拦截器
api.interceptors.response.use(
  (response) => response.data,
  (error) => {
    console.error('API Error:', error);
    return Promise.reject(error);
  }
);

export default api;
```

## 环境配置

### 开发环境要求
- Node.js >= 18.0.0
- npm >= 8.0.0 或 yarn >= 1.22.0

### 安装步骤
```bash
# 1. 创建项目
npm create vite@latest my-react-app -- --template react-ts

# 2. 进入项目目录
cd my-react-app

# 3. 安装依赖
npm install

# 4. 安装额外依赖
npm install zustand react-router-dom axios
npm install -D @types/node

# 5. 启动开发服务器
npm run dev
```

### 环境变量配置
```env
# .env.local
VITE_API_URL=http://localhost:3001/api
VITE_APP_TITLE=My React App
VITE_ENABLE_MOCK=false
```

## 路由配置

```tsx
// App.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { HomePage } from './pages/HomePage';
import { AboutPage } from './pages/AboutPage';
import { NotFoundPage } from './pages/NotFoundPage';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        <Route path="*" element={<NotFoundPage />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

## 测试策略

### 单元测试示例
```tsx
// tests/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '../src/components/Button';

describe('Button Component', () => {
  test('renders button with text', () => {
    render(<Button variant="primary">Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  test('calls onClick when clicked', () => {
    const handleClick = vi.fn();
    render(
      <Button variant="primary" onClick={handleClick}>
        Click me
      </Button>
    );
    
    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

## 性能优化

### 代码分割
```tsx
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

### 内存优化
```tsx
import { memo, useMemo, useCallback } from 'react';

const ExpensiveComponent = memo(({ data, onUpdate }) => {
  const processedData = useMemo(() => {
    return data.map(item => ({ ...item, processed: true }));
  }, [data]);

  const handleUpdate = useCallback((id) => {
    onUpdate(id);
  }, [onUpdate]);

  return (
    <div>
      {processedData.map(item => (
        <div key={item.id} onClick={() => handleUpdate(item.id)}>
          {item.name}
        </div>
      ))}
    </div>
  );
});
```

## 部署配置

### 构建生产版本
```bash
npm run build
```

### Vite 配置优化
```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
      },
    },
  },
  server: {
    port: 3000,
    open: true,
  },
});
```

## 常见问题

### 问题1：Vite 开发服务器启动慢
**解决方案**：
- 检查依赖预构建缓存
- 使用 `npm run dev -- --force` 强制重新构建
- 优化 vite.config.ts 中的 optimizeDeps 配置

### 问题2：TypeScript 类型错误
**解决方案**：
- 确保安装了正确的类型定义包
- 检查 tsconfig.json 配置
- 使用 `npm run type-check` 进行类型检查

## 参考资源

- [React 官方文档](https://react.dev/)
- [Vite 官方文档](https://vitejs.dev/)
- [TypeScript 官方文档](https://www.typescriptlang.org/)
- [React Router 文档](https://reactrouter.com/)
- [Zustand 文档](https://github.com/pmndrs/zustand)
