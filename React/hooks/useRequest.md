```ts
import { useState, useRef, useEffect, useCallback } from 'react';

const sleep = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

interface UseRequestOptions<TData, TParams extends any[]> {
  manual?: boolean;
  defaultParams?: TParams;
  onBefore?: (params: TParams) => void;
  onSuccess?: (data: TData, params: TParams) => void;
  onError?: (e: Error, params: TParams) => void;
  onFinally?: (params: TParams, data?: TData, e?: Error) => void;
  retryCount?: number;
  retryInterval?: number;
  loadingDelay?: number;
  pollingInterval?: number;
  debounceInterval?: number;
  throttleInterval?: number;
  cacheKey?: string;
  refreshOnWindowFocus?: boolean;
}

const cacheMap = new Map<string, any>();

export function useRequest<Service extends (...args: any[]) => Promise<any>>(
  service: Service,
  options: UseRequestOptions<
    Service extends (...args: any[]) => Promise<infer R> ? R : any,
    Service extends (...args: infer P) => Promise<any> ? P : any[]
  > = {}
) {
  type TData = Service extends (...args: any[]) => Promise<infer R> ? R : any;
  type TParams = Service extends (...args: infer P) => Promise<any> ? P : any[];

  const {
    manual = false,
    defaultParams,
    onBefore,
    onSuccess,
    onError,
    onFinally,
    retryCount = 0,
    retryInterval = 1000,
    loadingDelay = 0,
    pollingInterval,
    debounceInterval,
    throttleInterval,
    cacheKey,
    refreshOnWindowFocus = false,
  } = options;

  const [data, setData] = useState<TData | undefined>(cacheKey ? cacheMap.get(cacheKey) : undefined);
  const [error, setError] = useState<Error | undefined>();
  const [loading, setLoading] = useState(false);

  const paramsRef = useRef<TParams | undefined>(defaultParams);
  const isRequestingRef = useRef(false);
  const abortControllerRef = useRef<AbortController | null>(null);
  const timerRef = useRef<NodeJS.Timeout | null>(null);
  const loadingTimerRef = useRef<NodeJS.Timeout | null>(null);
  const debounceTimerRef = useRef<NodeJS.Timeout | null>(null);
  const throttleRef = useRef(false);

  const serviceRef = useRef(service);
  serviceRef.current = service;

  const fetchData = useCallback(
    async (...args: TParams): Promise<TData | undefined> => {
      if (isRequestingRef.current) return;
      isRequestingRef.current = true;

      const finalParams = args.length ? args : (paramsRef.current || ([] as unknown as TParams));
      paramsRef.current = finalParams;

      onBefore?.(finalParams);
      setError(undefined);

      if (loadingDelay > 0) {
        loadingTimerRef.current = setTimeout(() => setLoading(true), loadingDelay);
      } else {
        setLoading(true);
      }

      let result: TData | undefined;
      let finalError: Error | undefined;

      try {
        let attempt = 0;
        while (attempt <= retryCount) {
          try {
            abortControllerRef.current = new AbortController();
            result = await serviceRef.current(...finalParams, abortControllerRef.current.signal);
            break;
          } catch (e: any) {
            if (e.name === 'AbortError') throw e;
            attempt++;
            if (attempt > retryCount) throw e;
            await sleep(retryInterval);
          }
        }

        if (result !== undefined) {
          if (cacheKey) cacheMap.set(cacheKey, result);
          setData(result);
          onSuccess?.(result, finalParams);
        }

        return result;
      } catch (e: any) {
        finalError = e;
        setError(e);
        onError?.(e, finalParams);
      } finally {
        if (loadingTimerRef.current) {
          clearTimeout(loadingTimerRef.current);
          loadingTimerRef.current = null;
        }
        setLoading(false);
        isRequestingRef.current = false;
        onFinally?.(finalParams, result, finalError);
      }
    },
    [retryCount, retryInterval, loadingDelay, cacheKey, onBefore, onSuccess, onError, onFinally]
  );

  let run: (...args: TParams) => Promise<TData | undefined> = fetchData;
  if (debounceInterval) {
    run = ((...args: TParams) => {
      return new Promise<TData | undefined>(resolve => {
        if (debounceTimerRef.current) clearTimeout(debounceTimerRef.current);
        debounceTimerRef.current = setTimeout(() => resolve(fetchData(...args)), debounceInterval);
      });
    }) as typeof fetchData;
  } else if (throttleInterval) {
    run = ((...args: TParams) => {
      if (!throttleRef.current) {
        throttleRef.current = true;
        fetchData(...args).finally(() => {
          setTimeout(() => (throttleRef.current = false), throttleInterval);
        });
      }
      return Promise.resolve(undefined);
    }) as typeof fetchData;
  }

  useEffect(() => {
    if (!manual) run(...(defaultParams || ([] as unknown as TParams)));
    return () => {
      if (debounceTimerRef.current) clearTimeout(debounceTimerRef.current);
      if (loadingTimerRef.current) clearTimeout(loadingTimerRef.current);
      if (timerRef.current) clearInterval(timerRef.current);
    };
  }, [manual]);

  useEffect(() => {
    if (!pollingInterval) return;
    timerRef.current = setInterval(() => run(...(paramsRef.current || ([] as unknown as TParams))), pollingInterval);
    return () => {
      if (timerRef.current) clearInterval(timerRef.current);
    };
  }, [pollingInterval]);

  useEffect(() => {
    if (!refreshOnWindowFocus) return;
    const handleRefresh = () => run(...(paramsRef.current || ([] as unknown as TParams)));
    window.addEventListener('focus', handleRefresh);
    document.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'visible') handleRefresh();
    });
    return () => {
      window.removeEventListener('focus', handleRefresh);
      document.removeEventListener('visibilitychange', handleRefresh);
    };
  }, [refreshOnWindowFocus]);

  const mutate = useCallback(
    (newData?: TData | ((oldData?: TData) => TData | undefined)) => {
      setData(prev => {
        const value = typeof newData === 'function' ? (newData as any)(prev) : newData;
        if (cacheKey && value !== undefined) cacheMap.set(cacheKey, value);
        return value;
      });
    },
    [cacheKey]
  );

  const cancel = useCallback(() => {
    abortControllerRef.current?.abort();
    isRequestingRef.current = false;
    setLoading(false);
  }, []);

  const refresh = useCallback(() => {
    if (paramsRef.current) run(...paramsRef.current);
  }, []);

  const refreshAsync = useCallback(() => {
    if (paramsRef.current) return run(...paramsRef.current);
    return Promise.resolve(undefined as unknown as TData);
  }, []);

  return {
    loading,
    data,
    error,
    params: paramsRef.current || ([] as unknown as TParams),
    run,
    runAsync: fetchData,
    refresh,
    refreshAsync,
    mutate,
    cancel,
  };
}

```



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

```ts
import { useEffect, useRef, useState } from "react";

/**
 * 通用请求 Hook
 * @param request 请求函数，返回 Promise<T>
 * @param options 配置选项
 *  - manual: 是否手动触发，默认 false（自动执行）
 *  - deps: 依赖数组，request 函数或相关参数变化时重新执行
 * @returns { data: T | null, loading: boolean, error: Error | null, run: () => Promise<T> } 请求状态和触发函数
 */
export const useRequest = <T>(
    request: () => Promise<T>,
    options: { manual?: boolean; deps?: any[] } = {}
) => {
    const { manual = false, deps = [] } = options;

    // 请求结果状态
    const [data, setData] = useState<T | null>(null);
    // 请求 loading 状态
    const [loading, setLoading] = useState(false);
    // 请求错误状态
    const [error, setError] = useState<Error | null>(null);

    // 标记请求是否正在运行，防止重复请求
    const isRunningRef = useRef(false);
    // 标记组件是否已卸载，避免更新卸载组件的状态
    const isMountedRef = useRef(true);

    /**
     * 手动触发请求函数
     */
    const run = async () => {
        setLoading(true);
        isRunningRef.current = true;
        setError(null);

        try {
            const result = await request();
            // 组件已卸载则不更新状态
            if (!isMountedRef.current) return;
            setData(result);
            return result;
        } catch (err) {
            if (!isMountedRef.current) return;
            setError(err as Error);
            throw err;
        } finally {
            if (!isMountedRef.current) return;
            setLoading(false);
            isRunningRef.current = false;
        }
    };

    // 自动执行请求（如果 manual 为 false）或监听依赖变化
    useEffect(() => {
        isMountedRef.current = true;

        if (!manual) {
            run();
        }

        // 组件卸载时，标记已卸载，避免状态更新
        return () => {
            isMountedRef.current = false;
        };
    }, deps);

    // 返回请求状态和手动触发函数
    return { data, loading, error, run };
};

```

