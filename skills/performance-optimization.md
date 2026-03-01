# React 性能优化指南

## Description

优化 React 应用的性能，提升加载速度和运行效率。

## When to Use
- 应用加载慢
- 渲染卡顿
- 内存占用高
- 需要提升用户体验

## Example Usage
```bash
# 在 Claude Code 中使用
claude performance-optimization src/App.js
```

## Benefits
- ✅ 更快的加载速度
- ✅ 更流畅的体验
- ✅ 更低的资源占用
- ✅ 更好的 SEO

## Tips

### 1. 代码分割和懒加载

```javascript
// ✅ 动态导入
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <HeavyComponent />
    </Suspense>
  );
}

// ✅ 路由懒加载
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route
          path="/dashboard"
          element={
            <Suspense fallback={<LoadingSpinner />}>
              <Dashboard />
            </Suspense>
          }
        />
      </Routes>
    </BrowserRouter>
  );
}
```

### 2. 列表渲染优化

```javascript
// ✅ 使用 React.memo
const UserItem = React.memo(({ user }) => {
  return <div>{user.name}</div>;
});

// ✅ 使用 key 属性
function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <UserItem key={user.id} user={user} />
      ))}
    </ul>
  );
}

// ✅ 使用 useMemo 缓存计算
function UserList({ users }) {
  const sortedUsers = useMemo(() => {
    return users.sort((a, b) => a.name.localeCompare(b.name));
  }, [users]);

  return <ul>{sortedUsers.map((user) => <li key={user.id}>{user.name}</li>)}</ul>;
}
```

### 3. 虚拟化长列表

```javascript
// ✅ 使用 react-window
import { FixedSizeList as List } from 'react-window';

function LargeList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>{items[index]}</div>
  );

  return (
    <List
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </List>
  );
}
```

### 4. 避免不必要的重渲染

```javascript
// ✅ 使用 React.memo
const ExpensiveComponent = React.memo(({ data }) => {
  return <div>{data.value}</div>;
});

// ✅ 使用 useMemo 缓存
function App() {
  const expensiveValue = useMemo(() => {
    return computeExpensiveValue(data);
  }, [data]);

  return <ExpensiveComponent data={{ value: expensiveValue }} />;
}

// ✅ 使用 useCallback 缓存函数
function Parent({ onClick }) {
  const handleClick = useCallback(() => {
    onClick();
  }, [onClick]);

  return <button onClick={handleClick}>Click</button>;
}
```

### 5. 图片优化

```javascript
// ✅ 使用懒加载
<img src={image} loading="lazy" alt="Description" />

// ✅ 使用 WebP 格式
<img src="image.webp" alt="Description" />

// ✅ 使用响应式图片
<img
  srcSet="
    image-small.jpg 320w,
    image-medium.jpg 640w,
    image-large.jpg 1024w
  "
  sizes="(max-width: 600px) 100vw, 600px"
  src="image-large.jpg"
  alt="Description"
/>

// ✅ 使用 next/image (Next.js)
import Image from 'next/image';

<Image
  src="/image.jpg"
  alt="Description"
  width={500}
  height={300}
  loading="lazy"
/>
```

### 6. 减少包体积

```javascript
// ✅ 使用 Tree Shaking
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          ui: ['@mui/material', '@mui/icons-material'],
          utils: ['lodash', 'date-fns'],
        },
      },
    },
  },
});

// ✅ 压缩和混淆
// vite.config.js
export default defineConfig({
  build: {
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true, // 生产环境移除 console
      },
    },
  },
});
```

### 7. 服务器端渲染（SSR）

```javascript
// ✅ 使用 Next.js
import Head from 'next/head';

function HomePage() {
  return (
    <>
      <Head>
        <title>My Page</title>
        <meta name="description" content="Description" />
      </Head>
      <h1>Hello World</h1>
    </>
  );
}

export async function getServerSideProps() {
  const data = await fetchData();
  return {
    props: { data },
  };
}
```

### 8. 性能监控

```javascript
// ✅ 使用 React DevTools Profiler
import { Profiler } from 'react';

function onRenderCallback(
  id,
  phase,
  actualDuration,
  baseDuration,
  startTime,
  commitTime,
  interactions
) {
  console.log(`${id} ${phase} took ${actualDuration}ms`);
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <MainApp />
    </Profiler>
  );
}

// ✅ 使用性能 API
function usePerformanceMonitor(componentName) {
  const startTime = useRef(0);

  useEffect(() => {
    startTime.current = performance.now();
  }, [componentName]);

  useEffect(() => {
    const endTime = performance.now();
    const duration = endTime - startTime.current;
    if (duration > 16) { // 超过一帧 (60fps)
      console.warn(`${componentName} took ${duration.toFixed(2)}ms`);
    }
  }, [componentName]);
}
```

### 9. 最佳实践清单

- [ ] 使用代码分割和懒加载
- [ ] 优化列表渲染（虚拟化、memo）
- [ ] 使用 useMemo 和 useCallback
- [ ] 优化图片资源
- [ ] 减少包体积（Tree Shaking、压缩）
- [ ] 考虑 SSR/SSG
- [ ] 监控性能指标
- [ ] 避免过度优化
- [ ] 使用 CDN 加速
- [ ] 启用 Gzip/Brotli 压缩
