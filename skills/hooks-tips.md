# React Hooks 最佳实践

## Description

掌握 React Hooks 的最佳实践，写出更简洁、更高效的 React 代码。

## When to Use
- 使用 React 16.8+ 新特性
- 重构 Class 组件
- 优化性能
- 避免常见陷阱

## Example Usage
```bash
# 在 Claude Code 中使用
claude hooks-tips src/App.js
```

## Benefits
- ✅ 代码更简洁
- ✅ 更好的性能
- ✅ 更容易维护
- ✅ 减少代码重复

## Tips

### 1. Hooks 规则

```javascript
// ✅ 正确
function Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}

// ❌ 错误 - 在循环中调用
function Counter() {
  for (let i = 0; i < 10; i++) {
    useEffect(() => {}); // Bad!
  }
}

// ✅ 正确 - 使用 useCallback
function Counter() {
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);
  return <button onClick={handleClick}>Click</button>;
}
```

### 2. 性能优化

```javascript
// ✅ 使用 useMemo 缓存计算结果
function ExpensiveComponent({ items }) {
  const sortedItems = useMemo(() => {
    return items.sort((a, b) => a.value - b.value);
  }, [items]);

  return <List items={sortedItems} />;
}

// ✅ 使用 useCallback 缓存函数
function Parent({ items }) {
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);

  return <Child onClick={handleClick} />;
}
```

### 3. 自定义 Hooks

```javascript
// ✅ 创建自定义 Hook
function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });

  useEffect(() => {
    function handleResize() {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    }

    handleResize();
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// 使用
function App() {
  const { width, height } = useWindowSize();
  return <div>Window size: {width}x{height}</div>;
}
```

### 4. 依赖数组

```javascript
// ✅ 正确的依赖数组
useEffect(() => {
  fetchData();
}, [dependency1, dependency2]); // 明确列出依赖

// ❌ 错误 - 缺少依赖
useEffect(() => {
  fetchData();
}, []); // 可能导致闭包陷阱

// ✅ 使用函数式更新
setCount(prev => prev + 1); // 不需要依赖 count
```

### 5. 避免的常见陷阱

```javascript
// ❌ 错误 - 依赖项缺失
useEffect(() => {
  document.title = `Count: ${count}`;
}, []); // 缺少 count 依赖

// ✅ 正确
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);

// ❌ 错误 - 在条件语句中调用 Hook
if (condition) {
  useState(); // Bad!
}

// ✅ 正确
if (condition) {
  return <Component />;
}
useState(); // Good!
```

### 6. 性能监控

```javascript
// 使用 React DevTools Profiler
<Profiler id="ExpensiveComponent" onRender={handleRender}>
  <ExpensiveComponent data={items} />
</Profiler>

// 自定义性能监控
function usePerformanceMonitor(componentName) {
  const startTime = useRef(0);

  useEffect(() => {
    startTime.current = performance.now();
  }, [componentName]);

  useEffect(() => {
    const endTime = performance.now();
    const duration = endTime - startTime.current;
    console.log(`${componentName} rendered in ${duration.toFixed(2)}ms`);
  }, [componentName]);
}
```

### 7. 最佳实践清单

- [ ] 使用 `useCallback` 缓存函数
- [ ] 使用 `useMemo` 缓存计算结果
- [ ] 正确设置依赖数组
- [ ] 避免在条件语句中调用 Hook
- [ ] 使用 ESLint 插件检测问题
- [ ] 避免过度优化
- [ ] 使用自定义 Hooks 封装逻辑
- [ ] 遵循 React Hooks 规则
