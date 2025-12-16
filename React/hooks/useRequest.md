```ts
import { useCallback, useEffect, useRef, useState } from 'react';

const sleep = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

type Service<TData, TParams extends any[]> = (...args: TParams) => Promise<TData>;

interface UseRequestOptions<TData, TParams extends any[]> {
  manual?: boolean;
  defaultParams?: TParams;
  onBefore?: (params: TParams) => void;
  onSuccess?: (data: TData, params: TParams) => void;
  onError?: (e: Error, params: TParams) => void;
  onFinally?: (params: TParams, data?: TData, e?: Error | null) => void;
  retryCount?: number;
  retryInterval?: number;
  loadingDelay?: number;
  pollingInterval?: number;
  debounceInterval?: number;
  throttleInterval?: number;
  cacheKey?: string;
  cacheTime?: number; // ms to keep cache (stale => undefined)
  refreshOnWindowFocus?: boolean;
  throwOnError?: boolean; // whether runAsync should rethrow
}

const defaultOptions = {
  manual: false,
  retryCount: 0,
  retryInterval: 1000,
  loadingDelay: 0,
  throwOnError: false,
} as const;

const internalCache = new Map<
  string,
  { time: number; data: any; cacheTime?: number }
>();

export function useRequest<TData, TParams extends any[] = any[]>(
  service: Service<TData, TParams>,
  options?: UseRequestOptions<TData, TParams>
) {
  const opt = { ...defaultOptions, ...(options || {}) } as Required<
    UseRequestOptions<TData, TParams>
  > & { throwOnError: boolean };

  // state
  const [data, setData] = useState<TData | undefined>(() => {
    if (opt.cacheKey) {
      const entry = internalCache.get(opt.cacheKey);
      if (entry) {
        // if cacheTime provided and expired => ignore
        if (!entry.cacheTime || Date.now() - entry.time <= entry.cacheTime) {
          return entry.data as TData;
        }
        internalCache.delete(opt.cacheKey);
      }
    }
    return undefined;
  });
  const [error, setError] = useState<Error | undefined>();
  const [loading, setLoading] = useState(false);

  // refs
  const serviceRef = useRef(service);
  serviceRef.current = service;

  const paramsRef = useRef<TParams | undefined>(opt.defaultParams);
  const abortControllerRef = useRef<AbortController | null>(null);
  const isUnmountedRef = useRef(false);

  // concurrency control: allow concurrent requests but track lastRequestId to support "latestOnly" behavior
  const lastRequestIdRef = useRef(0);

  // timers
  const loadingTimerRef = useRef<number | null>(null);
  const debounceTimerRef = useRef<number | null>(null);
  const throttleLastTimeRef = useRef<number>(0);
  const pollingTimerRef = useRef<number | null>(null);

  // ---- helpers ----

  const setCache = useCallback(
    (key: string, val: TData) => {
      const entry = { time: Date.now(), data: val, cacheTime: opt.cacheTime };
      internalCache.set(key, entry);
    },
    [opt.cacheTime]
  );

  const clearTimers = useCallback(() => {
    if (loadingTimerRef.current) {
      clearTimeout(loadingTimerRef.current);
      loadingTimerRef.current = null;
    }
    if (debounceTimerRef.current) {
      clearTimeout(debounceTimerRef.current);
      debounceTimerRef.current = null;
    }
    if (pollingTimerRef.current) {
      clearInterval(pollingTimerRef.current);
      pollingTimerRef.current = null;
    }
  }, []);

  // Utility to decide whether we should append AbortSignal to service args.
  // This is a best-effort heuristic: if service.length > params.length, pass signal as last arg.
  function callServiceWithSignalIfNeeded(params: TParams, signal?: AbortSignal) {
    try {
      // If service expects more args than passed, append signal.
      // This is a non-breaking heuristic: if service doesn't accept it, JS will still call but extra args are ignored by many implementations.
      if (signal !== undefined && serviceRef.current.length >= (params?.length ?? 0) + 1) {
        // eslint-disable-next-line @typescript-eslint/ban-ts-comment
        // @ts-ignore - dynamic call
        return serviceRef.current(...params, signal);
      }
      // otherwise just call with params
      // eslint-disable-next-line @typescript-eslint/ban-ts-comment
      // @ts-ignore
      return serviceRef.current(...params);
    } catch (err) {
      return Promise.reject(err);
    }
  }

  // core fetch function
  const fetchData = useCallback(
    async (...args: TParams): Promise<TData> => {
      const requestId = ++lastRequestIdRef.current;
      const finalParams = (args && args.length ? args : paramsRef.current) as TParams;
      paramsRef.current = finalParams;

      opt.onBefore?.(finalParams);
      setError(undefined);

      // loading delay
      if (opt.loadingDelay > 0) {
        loadingTimerRef.current = window.setTimeout(() => {
          setLoading(true);
        }, opt.loadingDelay);
      } else {
        setLoading(true);
      }

      // abort previous
      abortControllerRef.current?.abort();
      const ac = new AbortController();
      abortControllerRef.current = ac;

      let lastError: Error | null = null;
      let result: TData | undefined;

      try {
        // retry loop
        let attempt = 0;
        while (true) {
          try {
            // call service (maybe with signal)
            // if service supports AbortSignal as last arg, above function will append it
            // eslint-disable-next-line @typescript-eslint/await-thenable
            const res = await callServiceWithSignalIfNeeded(finalParams, ac.signal);
            result = res as TData;
            break;
          } catch (e: any) {
            // If aborted, just throw immediately
            if (e && (e.name === 'AbortError' || ac.signal.aborted)) {
              throw e;
            }
            attempt++;
            lastError = e instanceof Error ? e : new Error(String(e));
            if (attempt > opt.retryCount) {
              throw lastError;
            }
            // wait before retrying
            // allow abort during sleep
            const waitPromise = sleep(opt.retryInterval);
            // race between abort and sleep
            await Promise.race([
              waitPromise,
              new Promise((_, rej) => {
                ac.signal.addEventListener(
                  'abort',
                  () => rej(new DOMException('Aborted', 'AbortError')),
                  { once: true }
                );
              }),
            ]);
            // continue to next attempt
          }
        }

        // Only update state if not unmounted and if this request is the latest one
        if (!isUnmountedRef.current && requestId === lastRequestIdRef.current) {
          if (opt.cacheKey && result !== undefined) {
            setCache(opt.cacheKey, result);
          }
          setData(result as TData);
          opt.onSuccess?.(result as TData, finalParams);
        }

        return result as TData;
      } catch (err: any) {
        if (!isUnmountedRef.current && requestId === lastRequestIdRef.current) {
          const thrown = err instanceof Error ? err : new Error(String(err));
          setError(thrown);
          opt.onError?.(thrown, finalParams);
        }
        // rethrow for runAsync depending on throwOnError
        if (opt.throwOnError) {
          throw err;
        }
        // otherwise return a rejected promise so callers can still await failure if they want
        return Promise.reject(err);
      } finally {
        // cleanup loading timer and state if still the latest
        if (loadingTimerRef.current) {
          clearTimeout(loadingTimerRef.current);
          loadingTimerRef.current = null;
        }
        if (!isUnmountedRef.current && requestId === lastRequestIdRef.current) {
          setLoading(false);
          opt.onFinally?.(finalParams, result, lastError);
        }
      }
    },
    [
      opt.loadingDelay,
      opt.onBefore,
      opt.onSuccess,
      opt.onError,
      opt.onFinally,
      opt.retryCount,
      opt.retryInterval,
      opt.cacheKey,
      opt.throwOnError,
      setCache,
    ]
  );

  // ---- wrappers: debounce / throttle ----

  const runRef = useRef<( ...args: TParams ) => Promise<TData>>();
  runRef.current = fetchData;

  const run = useCallback(
    (...args: TParams): Promise<TData | undefined> => {
      // debounce
      if (opt.debounceInterval && opt.debounceInterval > 0) {
        if (debounceTimerRef.current) clearTimeout(debounceTimerRef.current);
        return new Promise((resolve, reject) => {
          debounceTimerRef.current = window.setTimeout(() => {
            runRef.current!(...args)
              .then(d => resolve(d))
              .catch(e => reject(e));
            debounceTimerRef.current = null;
          }, opt.debounceInterval) as unknown as number;
        });
      }

      // throttle
      if (opt.throttleInterval && opt.throttleInterval > 0) {
        const now = Date.now();
        if (now - throttleLastTimeRef.current >= opt.throttleInterval) {
          throttleLastTimeRef.current = now;
          return runRef.current!(...args).catch(e => Promise.reject(e));
        } else {
          // return last promise? For simplicity return a resolved undefined to mimic ahooks behavior where calls during throttle window are ignored.
          // But better: return a promise that resolves when allowed — that complicates logic.
          return Promise.resolve(undefined as unknown as TData);
        }
      }

      // default immediate
      return runRef.current!(...args).catch(e => Promise.reject(e));
    },
    [opt.debounceInterval, opt.throttleInterval]
  );

  // runAsync: always calls fetchData directly (no debounce/throttle), returns promise and respects throwOnError option
  const runAsync = useCallback((...args: TParams) => {
    return fetchData(...args);
  }, [fetchData]);

  // mutate
  const mutate = useCallback(
    (newData: TData | ((oldData?: TData) => TData | undefined)) => {
      setData(prev => {
        const next = typeof newData === 'function' ? (newData as any)(prev) : newData;
        if (opt.cacheKey && next !== undefined) {
          setCache(opt.cacheKey, next as TData);
        }
        return next as TData | undefined;
      });
    },
    [opt.cacheKey, setCache]
  );

  // cancel
  const cancel = useCallback(() => {
    abortControllerRef.current?.abort();
    abortControllerRef.current = null;
    // mark this request as stale so its finally handlers won't overwrite state
    lastRequestIdRef.current++;
    setLoading(false);
  }, []);

  // refresh (use last params)
  const refresh = useCallback(() => {
    if (paramsRef.current) {
      // use debounce/throttle behavior in run
      // eslint-disable-next-line @typescript-eslint/no-floating-promises
      run(...(paramsRef.current as TParams));
    }
  }, [run]);

  const refreshAsync = useCallback(() => {
    if (paramsRef.current) {
      return fetchData(...(paramsRef.current as TParams));
    }
    return Promise.resolve(undefined as unknown as TData);
  }, [fetchData]);

  // initial auto run
  useEffect(() => {
    if (!opt.manual) {
      // if there's cacheKey and cache exists and not expired, we already set data via initial state
      // still may want to fetch again depending on requirements. ahooks usually fetches even if cache exists depending on staleTime.
      run(...(opt.defaultParams as TParams));
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  // polling
  useEffect(() => {
    if (opt.pollingInterval && opt.pollingInterval > 0) {
      pollingTimerRef.current = window.setInterval(() => {
        // use run so debounce/throttle behavior applies
        // eslint-disable-next-line @typescript-eslint/no-floating-promises
        run(...(paramsRef.current as TParams));
      }, opt.pollingInterval) as unknown as number;
      return () => {
        if (pollingTimerRef.current) {
          clearInterval(pollingTimerRef.current);
          pollingTimerRef.current = null;
        }
      };
    }
    return;
  }, [opt.pollingInterval, run]);

  // refresh on focus
  useEffect(() => {
    if (!opt.refreshOnWindowFocus) return;
    const onFocus = () => {
      if (document.visibilityState === 'visible') {
        run(...(paramsRef.current as TParams));
      }
    };
    window.addEventListener('focus', onFocus);
    document.addEventListener('visibilitychange', onFocus);
    return () => {
      window.removeEventListener('focus', onFocus);
      document.removeEventListener('visibilitychange', onFocus);
    };
  }, [opt.refreshOnWindowFocus, run]);

  // cleanup on unmount
  useEffect(() => {
    return () => {
      isUnmountedRef.current = true;
      abortControllerRef.current?.abort();
      abortControllerRef.current = null;
      clearTimers();
    };
  }, [clearTimers]);

  // expose params as readonly snapshot
  const paramsSnapshot = paramsRef.current ?? ([] as unknown as TParams);

  return {
    loading,
    data,
    error,
    params: paramsSnapshot,
    run,
    runAsync,
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

