# React 异步处理最佳实践

## Description

处理 React 中的异步操作，包括 API 调用、数据获取、状态管理等。

## When to Use
- 调用 API 获取数据
- 处理异步操作
- 管理加载和错误状态
- 实现数据缓存

## Example Usage
```bash
# 在 Claude Code 中使用
claude async-handling src/api.js
```

## Benefits
- ✅ 更好的用户体验
- ✅ 更清晰的代码结构
- ✅ 更容易调试
- ✅ 更好的错误处理

## Tips

### 1. 使用自定义 Hook 封装 API 调用

```javascript
// ✅ 创建自定义 Hook
function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchData() {
      try {
        setLoading(true);
        const response = await fetch(url, options);
        const json = await response.json();

        if (!cancelled) {
          setData(json);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
          setData(null);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }

    fetchData();

    return () => {
      cancelled = true;
    };
  }, [url, options]);

  return { data, loading, error };
}

// 使用
function UserProfile({ userId }) {
  const { data, loading, error } = useFetch(
    `https://api.example.com/users/${userId}`
  );

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!data) return null;

  return <UserProfileCard user={data} />;
}
```

### 2. 数据缓存策略

```javascript
// ✅ 使用 React Query 或 SWR
import { useQuery } from 'react-query';

function UserProfile({ userId }) {
  const { data, isLoading, error } = useQuery(
    ['user', userId],
    () => fetchUser(userId),
    {
      staleTime: 5 * 60 * 1000, // 5 分钟内不重新请求
      cacheTime: 10 * 60 * 1000, // 10 分钟后清除缓存
      refetchOnWindowFocus: false, // 切换窗口时不自动刷新
    }
  );

  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;

  return <UserProfileCard user={data} />;
}

// ✅ 本地缓存
function useCachedData(key, fetchFn, ttl = 5 * 60 * 1000) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const cacheExpiry = useRef(null);

  useEffect(() => {
    const cachedData = localStorage.getItem(key);
    const now = Date.now();

    if (cachedData && cacheExpiry.current && now < cacheExpiry.current) {
      setData(JSON.parse(cachedData));
      setLoading(false);
      return;
    }

    fetchFn()
      .then((result) => {
        setData(result);
        localStorage.setItem(key, JSON.stringify(result));
        cacheExpiry.current = now + ttl;
        setError(null);
      })
      .catch((err) => {
        setError(err.message);
        setData(null);
      })
      .finally(() => {
        setLoading(false);
      });
  }, [key, fetchFn, ttl]);

  return { data, loading, error };
}
```

### 3. 错误处理

```javascript
// ✅ 统一错误处理
function useErrorHandler() {
  const [error, setError] = useState(null);

  const handleError = useCallback((error, context = {}) => {
    console.error('Error:', error, context);
    setError({
      message: error.message,
      code: error.code,
      context,
    });
  }, []);

  const clearError = useCallback(() => {
    setError(null);
  }, []);

  return { error, handleError, clearError };
}

// 使用
function UserProfile({ userId }) {
  const { data, loading, error, handleError } = useFetch(
    `https://api.example.com/users/${userId}`
  );

  return (
    <div>
      {error && (
        <ErrorMessage
          error={error}
          onRetry={() => fetchData()}
          onError={handleError}
        />
      )}
      {loading && <LoadingSpinner />}
      {data && <UserProfileCard user={data} />}
    </div>
  );
}
```

### 4. 并发请求

```javascript
// ✅ 并发请求多个 API
function UserProfile({ userIds }) {
  const { data: users, loading, error } = useQuery(
    ['users', userIds],
    () => Promise.all(userIds.map((id) => fetchUser(id))),
    {
      combine: (results) => results.map((r) => r.json()),
    }
  );

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <div>
      {users.map((user) => (
        <UserProfileCard key={user.id} user={user} />
      ))}
    </div>
  );
}
```

### 5. 重试机制

```javascript
// ✅ 自动重试
function useFetchWithRetry(url, options = {}, maxRetries = 3) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let retries = 0;

    async function fetchData() {
      try {
        const response = await fetch(url, options);
        const json = await response.json();
        setData(json);
        setError(null);
      } catch (err) {
        if (retries < maxRetries) {
          retries++;
          const delay = Math.pow(2, retries) * 1000; // 指数退避
          setTimeout(fetchData, delay);
        } else {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchData();
  }, [url, options, maxRetries]);

  return { data, loading, error };
}
```

### 6. 乐观更新

```javascript
// ✅ 乐观更新
function useOptimisticUpdate(key, updateFn) {
  const [state, setState] = useState(null);

  const optimisticUpdate = useCallback(
    async (params) => {
      // 乐观更新 UI
      setState((prev) => ({
        ...prev,
        ...updateFn(prev, params),
      }));

      try {
        // 实际请求
        const result = await actualUpdate(params);
        setState(result);
        return result;
      } catch (error) {
        // 回滚
        setState(null);
        throw error;
      }
    },
    [key, updateFn]
  );

  return { state, optimisticUpdate };
}
```

### 7. 最佳实践清单

- [ ] 使用自定义 Hook 封装异步逻辑
- [ ] 提供加载和错误状态
- [ ] 实现取消请求（AbortController）
- [ ] 使用数据缓存
- [ ] 添加重试机制
- [ ] 实现乐观更新
- [ ] 处理并发请求
- [ ] 提供错误恢复机制
- [ ] 监控和日志记录
- [ ] 考虑离线支持
