```ts
import { useState, useEffect, useCallback, useRef } from 'react';

// 简单防抖函数
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

// 简单节流函数
function throttle(fn, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}

/**
 * 高级 useRequest
 * @param {Function} service - 返回 Promise 的请求函数
 * @param {Object} options
 *  - manual: boolean 是否手动触发
 *  - defaultParams: array 默认参数
 *  - pollingInterval: number 轮询间隔(ms)
 *  - debounceInterval: number 防抖间隔(ms)
 *  - throttleInterval: number 节流间隔(ms)
 *  - cacheKey: string 数据缓存key
 */
function useRequest(service, options = {}) {
  const {
    manual = false,
    defaultParams = [],
    pollingInterval = 0,
    debounceInterval = 0,
    throttleInterval = 0,
    cacheKey,
  } = options;

  const [data, setData] = useState(cacheKey ? JSON.parse(localStorage.getItem(cacheKey)) || null : null);
  const [loading, setLoading] = useState(!manual);
  const [error, setError] = useState(null);

  const paramsRef = useRef(defaultParams);
  const pollingRef = useRef(null);

  const fetchData = useCallback(
    async (...params) => {
      const finalParams = params.length ? params : paramsRef.current;
      setLoading(true);
      setError(null);

      try {
        const result = await service(...finalParams);
        setData(result);
        if (cacheKey) localStorage.setItem(cacheKey, JSON.stringify(result));
        return result;
      } catch (err) {
        setError(err);
        throw err;
      } finally {
        setLoading(false);
      }
    },
    [service, cacheKey]
  );

  // 包装防抖和节流
  const run = useCallback(() => {
    let fn = fetchData;
    if (debounceInterval > 0) fn = debounce(fn, debounceInterval);
    if (throttleInterval > 0) fn = throttle(fn, throttleInterval);
    return fn(...arguments);
  }, [fetchData, debounceInterval, throttleInterval]);

  // 自动请求
  useEffect(() => {
    if (!manual) {
      run(...defaultParams);
    }
  }, [manual, run, ...defaultParams]);

  // 轮询功能
  useEffect(() => {
    if (pollingInterval > 0) {
      pollingRef.current = setInterval(() => {
        run(...paramsRef.current);
      }, pollingInterval);
      return () => clearInterval(pollingRef.current);
    }
  }, [pollingInterval, run]);

  // 手动刷新
  const refresh = useCallback(() => run(...paramsRef.current), [run]);

  return { data, loading, error, run, refresh };
}

export default useRequest;

```

