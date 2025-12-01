```js
function usePagingList(api, options = {}) {
  const {
    pageSize = 20,
    initParams = {},   // 外部传入的各种条件
  } = options;

  const [list, setList] = useState([]);
  const [loading, setLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);

  const loadLock = useRef(false);
  const pageRef = useRef(1);
  const hasMoreRef = useRef(true);
  const paramsRef = useRef(initParams); // 动态查询参数

  // ⭐ 更新过滤条件（例如搜索、筛选、排序）
  const updateParams = (newParams) => {
    paramsRef.current = {
      ...paramsRef.current,
      ...newParams,
    };
    refresh(); // 条件变化 → 自动刷新
  };

  const loadMore = async () => {
    if (loadLock.current) return;
    if (!hasMoreRef.current) return;

    loadLock.current = true;
    setLoading(true);

    try {
      const res = await api({
        page: pageRef.current,
        pageSize,
        ...paramsRef.current,
      });

      const items = res.items ?? res.data ?? [];

      setList(prev => [...prev, ...items]);

      pageRef.current += 1;
      hasMoreRef.current = items.length >= pageSize;
    } finally {
      loadLock.current = false;
      setLoading(false);
    }
  };

  const refresh = async () => {
    if (loadLock.current) return;

    loadLock.current = true;
    setRefreshing(true);

    try {
      pageRef.current = 1;
      hasMoreRef.current = true;

      const res = await api({
        page: 1,
        pageSize,
        ...paramsRef.current,
      });

      const items = res.items ?? res.data ?? [];
      setList(items);

      pageRef.current += 1;
    } finally {
      loadLock.current = false;
      setRefreshing(false);
    }
  };

  return {
    list,
    loading,
    refreshing,
    loadMore,
    refresh,
    updateParams, // ⭐ 对外暴露用于更新搜索条件/筛选条件
    params: paramsRef.current,
    hasMore: hasMoreRef.current,
  };
}

```





```ts
import { useRef, useState } from "react";

export interface PagingResult<T> {
  items?: T[];
  data?: T[];
  list?: T[];
  total?: number;
  [key: string]: any;
}

export interface PagingApiParams {
  page: number;
  pageSize: number;
  [key: string]: any;
}

export interface UsePagingListOptions<P extends object = {}> {
  pageSize?: number;
  initParams?: P;
}

export function usePagingList<T, P extends object = {}>(
  api: (params: PagingApiParams & P) => Promise<PagingResult<T>>,
  options: UsePagingListOptions<P> = {}
) {
  const { pageSize = 20, initParams = {} as P } = options;

  const [list, setList] = useState<T[]>([]);
  const [loading, setLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);

  const loadLock = useRef(false);
  const pageRef = useRef(1);
  const hasMoreRef = useRef(true);
  const paramsRef = useRef<P>(initParams);

  // 统一取 items/data/list
  const getItems = (res: PagingResult<T>): T[] => {
    return (
      res.items ||
      res.data ||
      res.list ||
      []
    );
  };

  /** 更新查询参数（搜索、筛选、排序） */
  const updateParams = (newParams: Partial<P>) => {
    paramsRef.current = {
      ...paramsRef.current,
      ...newParams,
    };
    refresh();
  };

  /** 加载下一页 */
  const loadMore = async () => {
    if (loadLock.current) return;
    if (!hasMoreRef.current) return;

    loadLock.current = true;
    setLoading(true);

    try {
      const res = await api({
        page: pageRef.current,
        pageSize,
        ...paramsRef.current,
      });

      const items = getItems(res);
      setList(prev => [...prev, ...items]);

      pageRef.current += 1;

      // 是否还有更多数据（通用逻辑）
      hasMoreRef.current = items.length >= pageSize;
    } finally {
      loadLock.current = false;
      setLoading(false);
    }
  };

  /** 刷新（重置参数重新加载第一页） */
  const refresh = async () => {
    if (loadLock.current) return;

    loadLock.current = true;
    setRefreshing(true);

    try {
      pageRef.current = 1;
      hasMoreRef.current = true;

      const res = await api({
        page: 1,
        pageSize,
        ...paramsRef.current,
      });

      const items = getItems(res);
      setList(items);

      pageRef.current += 1;
    } finally {
      loadLock.current = false;
      setRefreshing(false);
    }
  };

  return {
    list,
    loading,
    refreshing,
    loadMore,
    refresh,
    updateParams,
    params: paramsRef.current,
    hasMore: hasMoreRef.current,
  };
}

```

```ts
import { useRef, useState } from "react";

export interface PagingResult<T> {
  items?: T[];
  data?: T[];
  list?: T[];
  total?: number;
  [key: string]: any;
}

export interface PagingApiParams {
  page: number;
  page_size: number;
  [key: string]: any;
}

export interface UsePagingListOptions<P extends object = {}> {
  page_size?: number;
  initParams?: P;
  onError?: (error: unknown, type: "loadMore" | "refresh") => void;
}

export function usePagingList<T extends Record<string, any>, P extends object = {}>(
  api: (params: PagingApiParams & P) => Promise<PagingResult<T>>,
  options: UsePagingListOptions<P> = {}
) {
  const { page_size = 10, initParams = {} as P, onError } = options;

  const [list, setList] = useState<T[]>([]);
  const [loading, setLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);
  const [error, setError] = useState<unknown>(null);
  const [noData, setNoData] = useState(false);
  const [isEnd, setIsEnd] = useState(false);

  const pageRef = useRef(1);
  const hasMoreRef = useRef(true);
  const paramsRef = useRef<P>(initParams);

  const refreshLock = useRef(false);
  const loadMoreLock = useRef(false);

  // 支持多个失败请求重试
  const failedRequestsRef = useRef<{ type: "loadMore" | "refresh"; params: PagingApiParams & P }[]>([]);

  const getItems = (res: PagingResult<T>): T[] => res.items || res.data || res.list || [];

  const updateParams = (newParams: Partial<P>, autoRefresh = true) => {
    paramsRef.current = { ...paramsRef.current, ...newParams };
    if (autoRefresh) refresh();
  };

  const execute = async (type: "loadMore" | "refresh", params: PagingApiParams & P) => {
    const lock = type === "refresh" ? refreshLock : loadMoreLock;
    const setStateLoading = type === "refresh" ? setRefreshing : setLoading;

    if (lock.current) return;
    lock.current = true;
    setError(null);
    setStateLoading(true);

    try {
      // 成功时清除对应失败请求
      failedRequestsRef.current = failedRequestsRef.current.filter(fr => fr.params !== params || fr.type !== type);

      const res = await api(params);
      const items = getItems(res);
      const total = Number(res.total ?? Infinity);

      const isNoData = total <= 0;
      const hasMore = pageRef.current * page_size < total;

      if (type === "refresh") {
        setList(items);
        pageRef.current = 2;
      } else {
        setList(prev => [...prev, ...items]);
        if (hasMore) pageRef.current += 1;
      }

      hasMoreRef.current = hasMore;
      setNoData(isNoData);
      setIsEnd(isNoData || !hasMore);

    } catch (err) {
      setError(err);
      onError?.(err, type);
      failedRequestsRef.current.push({ type, params });
    } finally {
      lock.current = false;
      setLoading(false);
      setRefreshing(false);
    }
  };

  const loadMore = async () => {
    if (!hasMoreRef.current) return;
    await execute("loadMore", {
      page: pageRef.current,
      page_size,
      ...paramsRef.current,
    });
  };

  const refresh = async () => {
    pageRef.current = 1;
    hasMoreRef.current = true;
    await execute("refresh", {
      page: 1,
      page_size,
      ...paramsRef.current,
    });
  };

  const retry = async () => {
    const queue = [...failedRequestsRef.current];
    failedRequestsRef.current = [];
    for (const { type, params } of queue) {
      await execute(type, params);
    }
  };

  /** 更新 list 中某一项，按 key/value 匹配 */
  const updateItem = (key: keyof T, value: any, updater: (item: T) => T) => {
    setList(prev => prev.map(item => (item[key] === value ? updater(item) : item)));
  };

  /** 乐观更新：先更新 UI，再执行异步操作，失败回滚（不会覆盖其他操作） */
  const updateItemOptimistic = async (
    key: keyof T,
    value: any,
    updater: (item: T) => T,
    apiCall?: () => Promise<any>
  ) => {
    const snapshot = list.map(item => ({ ...item }));
    updateItem(key, value, updater);

    try {
      if (apiCall) await apiCall();
    } catch {
      setList(prev => prev.map(item => {
        const original = snapshot.find(s => s[key] === item[key]);
        return original ? original : item;
      }));
    }
  };

  return {
    list,
    loading,
    refreshing,
    error,
    noData,
    isEnd,
    hasMore: hasMoreRef.current,
    params: paramsRef.current,

    loadMore,
    refresh,
    retry,
    updateParams,
    updateItem,
    updateItemOptimistic,
  };
}

```

