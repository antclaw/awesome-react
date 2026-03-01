# React 路由指南

## Description
React 路由配置和最佳实践，包括 React Router 使用、动态路由、路由守卫等。

## When to Use
- 创建单页应用（SPA）
- 实现页面导航
- 处理 URL 参数
- 实现路由守卫
- 多页面应用

## Example Usage
```bash
# 在 Claude Code 中使用
claude routing-guide src/App.jsx
```

## Benefits
- ✅ 清晰的页面结构
- ✅ 用户体验流畅
- ✅ SEO 友好
- ✅ 易于维护

## 推荐工具

### 1. React Router（推荐）

```bash
# 安装
npm install react-router-dom

# 基础使用
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">首页</Link>
        <Link to="/about">关于</Link>
        <Link to="/users">用户</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users" element={<Users />} />
        <Route path="/users/:id" element={<UserProfile />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### 2. 动态路由

```javascript
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();
  const user = useUser(id);

  return <div>用户 ID: {id}</div>;
}

// 配置
<Route path="/users/:id" element={<UserProfile />} />
```

### 3. 路由守卫

```javascript
import { Navigate, useLocation } from 'react-router-dom';

function ProtectedRoute({ children }) {
  const location = useLocation();
  const isAuthenticated = useAuth();

  if (!isAuthenticated) {
    // 保存当前位置，登录后跳转
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}

// 使用
<Route
  path="/dashboard"
  element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  }
/>
```

### 4. 嵌套路由

```javascript
function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="about" element={<About />} />
        <Route path="users" element={<UsersLayout />}>
          <Route index element={<UsersList />} />
          <Route path=":id" element={<UserProfile />} />
        </Route>
      </Route>
    </Routes>
  );
}

// 嵌套布局
function UsersLayout() {
  return (
    <div>
      <UsersSidebar />
      <UsersContent />
    </div>
  );
}
```

### 5. 查询参数

```javascript
import { useSearchParams } from 'react-router-dom';

function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get('q');

  const handleSearch = (newQuery) => {
    setSearchParams({ q: newQuery });
  };

  return (
    <div>
      <input
        value={query || ''}
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="搜索..."
      />
    </div>
  );
}

// 配置
<Route path="/search" element={<SearchPage />} />
```

### 6. 编程式导航

```javascript
import { useNavigate } from 'react-router-dom';

function UserProfile({ userId }) {
  const navigate = useNavigate();

  const handleEdit = () => {
    navigate(`/users/${userId}/edit`);
  };

  return <button onClick={handleEdit}>编辑</button>;
}

// 返回上一页
const navigate = useNavigate();
navigate(-1); // 返回上一页
navigate('/home'); // 跳转到指定页面
```

### 7. 路由懒加载

```javascript
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// 懒加载组件
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Users = lazy(() => import('./pages/Users'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/users" element={<Users />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

### 8. 路由元信息

```javascript
// 配置路由
const routes = [
  {
    path: '/',
    element: <Home />,
    meta: { title: '首页', requiresAuth: false }
  },
  {
    path: '/dashboard',
    element: <Dashboard />,
    meta: { title: '仪表盘', requiresAuth: true }
  },
  {
    path: '/admin',
    element: <Admin />,
    meta: { title: '管理', requiresAuth: true, requiresAdmin: true }
  }
];

// 创建路由守卫
function ProtectedRoute({ children, meta }) {
  const isAuthenticated = useAuth();
  const isAdmin = useAdmin();

  if (meta.requiresAuth && !isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  if (meta.requiresAdmin && !isAdmin) {
    return <Navigate to="/dashboard" replace />;
  }

  return children;
}
```

### 9. 404 页面

```javascript
function NotFound() {
  return (
    <div>
      <h1>404 - 页面不存在</h1>
      <Link to="/">返回首页</Link>
    </div>
  );
}

// 配置
<Route path="*" element={<NotFound />} />
```

### 10. 路由懒加载 + 预加载

```javascript
import { lazy, Suspense } from 'react';

// 预加载
const loadAbout = () => import('./pages/About');

// 懒加载
const About = lazy(loadAbout);

function App() {
  return <Suspense fallback={<LoadingSpinner />}><About /></Suspense>;
}
```

## 最佳实践

### 1. 路由命名规范

```javascript
// ✅ 推荐
<Route path="/users/:userId" element={<UserProfile />} />
<Route path="/products/:productId" element={<ProductDetail />} />

// ❌ 不推荐
<Route path="/users/:id" element={<UserProfile />} />
<Route path="/p/:id" element={<ProductDetail />} />
```

### 2. 路由顺序

```javascript
// ✅ 具体/动态路由在前
<Routes>
  <Route path="/users/:id" element={<UserProfile />} />
  <Route path="/users" element={<UsersList />} />
  <Route path="/" element={<Home />} />
</Routes>

// ❌ 动态路由在后面
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/users" element={<UsersList />} />
  <Route path="/users/:id" element={<UserProfile />} />
</Routes>
```

### 3. 路由嵌套结构

```javascript
// ✅ 清晰的嵌套结构
<Route path="/app">
  <Route path="dashboard" element={<Dashboard />} />
  <Route path="settings" element={<Settings />} />
  <Route path="profile" element={<Profile />} />
</Route>

// ❌ 过深的嵌套
<Route path="/app">
  <Route path="dashboard">
    <Route path="overview">
      <Route path="overview">
        <Route path="overview">
          <Route path="..." />
        </Route>
      </Route>
    </Route>
  </Route>
</Route>
```

### 4. 路由配置分离

```javascript
// 路由配置文件
const routes = [
  {
    path: '/',
    element: <Home />,
    meta: { title: '首页' }
  },
  {
    path: '/about',
    element: <About />,
    meta: { title: '关于' }
  }
];

// 渲染路由
function App() {
  return (
    <Routes>
      {routes.map((route) => (
        <Route
          key={route.path}
          path={route.path}
          element={<route.element />}
        />
      ))}
    </Routes>
  );
}
```

## 常用命令速查

| 操作 | 命令 |
|------|------|
| 安装 | `npm install react-router-dom` |
| 基础路由 | `<Route path="/" element={<Home />} />` |
| 动态路由 | `<Route path="/users/:id" element={<UserProfile />} />` |
| 路由守卫 | `<ProtectedRoute><Dashboard /></ProtectedRoute>` |
| 查询参数 | `const [params] = useSearchParams()` |
| 编程式导航 | `navigate('/home')` |
| 懒加载 | `const Home = lazy(() => import('./Home'))` |

## 学习资源

- [React Router 文档](https://reactrouter.com/)
- [React Router 中文文档](https://reactrouter.com/zh-CN)
