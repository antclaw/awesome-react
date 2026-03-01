# React 状态管理指南

## Description
React 状态管理方案对比和最佳实践，包括 Context、Redux、Zustand 等。

## When to Use
- 管理全局状态
- 跨组件共享数据
- 复杂状态逻辑
- 服务端状态管理

## Example Usage
```bash
# 在 Claude Code 中使用
claude state-management src/App.jsx
```

## Benefits
- ✅ 统一状态管理
- ✅ 代码可维护性
- ✅ 调试便利
- ✅ 性能优化

## 状态管理方案对比

| 方案 | 适用场景 | 学习成本 | 性能 | 复杂度 |
|------|---------|---------|------|--------|
| **Context API** | 简单全局状态 | 低 | 好 | 低 |
| **Zustand** | 中等复杂度 | 低 | 优秀 | 低 |
| **Jotai** | 原子化状态 | 中 | 优秀 | 中 |
| **Redux Toolkit** | 复杂应用 | 高 | 好 | 高 |
| **React Query** | 服务端状态 | 中 | 优秀 | 中 |

## 1. Context API（简单场景）

```javascript
import { createContext, useContext, useState } from 'react';

// 1. 创建 Context
const UserContext = createContext(null);

// 2. 创建 Provider
function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
}

// 3. 使用 Context
function UserProfile() {
  const { user, setUser } = useContext(UserContext);
  return <div>{user?.name}</div>;
}

// 使用
function App() {
  return (
    <UserProvider>
      <UserProfile />
    </UserProvider>
  );
}
```

## 2. Zustand（推荐）

```bash
# 安装
npm install zustand
```

```javascript
import { create } from 'zustand';

// 创建 Store
const useUserStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  logout: () => set({ user: null })
}));

// 使用
function UserProfile() {
  const user = useUserStore((state) => state.user);
  const setUser = useUserStore((state) => state.setUser);

  return (
    <div>
      {user?.name}
      <button onClick={() => setUser({ name: 'Alice' })}>设置用户</button>
    </div>
  );
}

// 选择器优化
function UserName() {
  const name = useUserStore((state) => state.user?.name);
  return <div>{name}</div>;
}
```

## 3. Jotai（原子化）

```bash
# 安装
npm install jotai
```

```javascript
import { atom, useAtom } from 'jotai';

// 创建原子
const countAtom = atom(0);
const userAtom = atom({ name: 'Alice' });

// 使用
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <span>{count}</span>
    </div>
  );
}

function UserInfo() {
  const [user, setUser] = useAtom(userAtom);
  return <div>{user?.name}</div>;
}
```

## 4. Redux Toolkit（复杂场景）

```bash
# 安装
npm install @reduxjs/toolkit react-redux
```

```javascript
import { createSlice, configureStore } from '@reduxjs/toolkit';
import { Provider, useSelector, useDispatch } from 'react-redux';

// 创建 Slice
const userSlice = createSlice({
  name: 'user',
  initialState: {
    user: null,
    isAuthenticated: false
  },
  reducers: {
    setUser: (state, action) => {
      state.user = action.payload;
      state.isAuthenticated = true;
    },
    logout: (state) => {
      state.user = null;
      state.isAuthenticated = false;
    }
  }
});

export const { setUser, logout } = userSlice.actions;

// 配置 Store
const store = configureStore({
  reducer: {
    user: userSlice.reducer
  }
});

// 使用
function UserProfile() {
  const user = useSelector((state) => state.user.user);
  const dispatch = useDispatch();

  return (
    <div>
      {user?.name}
      <button onClick={() => dispatch(setUser({ name: 'Alice' }))}>
        设置用户
      </button>
      <button onClick={() => dispatch(logout())}>退出</button>
    </div>
  );
}

// Provider
function App() {
  return (
    <Provider store={store}>
      <UserProfile />
    </Provider>
  );
}
```

## 5. React Query（服务端状态）

```bash
# 安装
npm install @tanstack/react-query
```

```javascript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// 查询用户
function UserProfile({ userId }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId)
  });

  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!data) return null;

  return <UserProfileCard user={data} />;
}

// 更新用户
function UserUpdateForm({ userId }) {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (updates) => updateUser(userId, updates),
    onSuccess: () => {
      // 重新获取数据
      queryClient.invalidateQueries(['user', userId]);
    }
  });

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      mutation.mutate({ name: 'New Name' });
    }}>
      <button type="submit">更新</button>
    </form>
  );
}
```

## 6. 组合状态管理

```javascript
// 使用多个 Store
import { create } from 'zustand';
import { atom } from 'jotai';

// Zustand Store
const useUserStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user })
}));

// Jotai Atoms
const countAtom = atom(0);
const themeAtom = atom('light');

// 组合使用
function App() {
  const user = useUserStore((state) => state.user);
  const [count, setCount] = useAtom(countAtom);
  const [theme, setTheme] = useAtom(themeAtom);

  return (
    <div>
      <h1>{user?.name}</h1>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Theme: {theme}
      </button>
    </div>
  );
}
```

## 最佳实践

### 1. 选择合适的方案

```javascript
// ✅ 简单场景 - Context API
const ThemeContext = createContext(null);

// ✅ 中等场景 - Zustand
const useUserStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user })
}));

// ✅ 复杂场景 - Redux Toolkit
const userSlice = createSlice({ ... });

// ✅ 服务端状态 - React Query
const { data } = useQuery({ ... });
```

### 2. 避免过度优化

```javascript
// ❌ 不推荐 - 过度使用 Redux
const store = configureStore({
  reducer: {
    user: userSlice.reducer,
    theme: themeSlice.reducer,
    notifications: notificationsSlice.reducer,
    // ... 太多 reducers
  }
});

// ✅ 推荐 - 简单场景用 Context
const ThemeContext = createContext(null);
```

### 3. 选择器优化

```javascript
// ✅ 推荐 - 使用选择器
const userName = useUserStore((state) => state.user?.name);

// ❌ 不推荐 - 直接解构
const { user, setUser } = useUserStore();
```

### 4. 组件拆分

```javascript
// ✅ 推荐 - 拆分组件
function UserList() {
  const users = useUserStore((state) => state.users);
  return (
    <ul>
      {users.map(user => <UserItem key={user.id} user={user} />)}
    </ul>
  );
}

function UserItem({ user }) {
  return <li>{user.name}</li>;
}

// ❌ 不推荐 - 组件过大
function UserList() {
  const users = useUserStore((state) => state.users);
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name}
          {user.email}
          {user.avatar}
          {user.bio}
          {/* 太多逻辑 */}
        </li>
      ))}
    </ul>
  );
}
```

## 常见模式

### 1. 路由状态

```javascript
import { useSearchParams } from 'react-router-dom';

function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get('q');

  return (
    <div>
      <input
        value={query || ''}
        onChange={(e) => setSearchParams({ q: e.target.value })}
      />
    </div>
  );
}
```

### 2. 表单状态

```javascript
import { useForm } from 'react-hook-form';

function LoginForm() {
  const { register, handleSubmit } = useForm();

  const onSubmit = handleSubmit((data) => {
    console.log(data);
  });

  return (
    <form onSubmit={onSubmit}>
      <input {...register('username')} />
      <input {...register('password', { required: true })} />
      <button type="submit">提交</button>
    </form>
  );
}
```

### 3. 模态框状态

```javascript
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;

  return (
    <div className="modal">
      <button onClick={onClose}>关闭</button>
      {children}
    </div>
  );
}

function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsModalOpen(true)}>打开</button>
      <Modal isOpen={isModalOpen} onClose={() => setIsModalOpen(false)}>
        <p>模态框内容</p>
      </Modal>
    </div>
  );
}
```

## 常用命令速查

| 操作 | 命令 |
|------|------|
| Context API | `createContext()` |
| Zustand | `create((set) => ({ ... }))` |
| Jotai | `atom(初始值)` |
| Redux Toolkit | `createSlice({ ... })` |
| React Query | `useQuery({ ... })` |
| 选择器 | `useStore((state) => state.xxx)` |

## 学习资源

- [React Context 文档](https://react.dev/reference/react/Context)
- [Zustand 文档](https://docs.pmnd.rs/zustand/getting-started/introduction)
- [Jotai 文档](https://jotai.org/)
- [Redux Toolkit 文档](https://redux-toolkit.js.org/)
- [React Query 文档](https://tanstack.com/query/latest)
