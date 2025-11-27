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
  pageSize: number;
  [key: string]: any;
}

export interface UsePagingListOptions<P extends object = {}> {
  pageSize?: number;
  initParams?: P;

  // ⭐ 错误回调（可选）
  onError?: (error: unknown, type: "loadMore" | "refresh") => void;
}

export function usePagingList<T, P extends object = {}>(
  api: (params: PagingApiParams & P) => Promise<PagingResult<T>>,
  options: UsePagingListOptions<P> = {}
) {
  const { pageSize = 20, initParams = {} as P, onError } = options;

  const [list, setList] = useState<T[]>([]);
  const [loading, setLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);
  const [error, setError] = useState<unknown>(null);

  const loadLock = useRef(false);
  const pageRef = useRef(1);
  const hasMoreRef = useRef(true);
  const paramsRef = useRef<P>(initParams);

  const getItems = (res: PagingResult<T>): T[] => {
    return res.items || res.data || res.list || [];
  };

  const updateParams = (newParams: Partial<P>) => {
    paramsRef.current = { ...paramsRef.current, ...newParams };
    refresh();
  };

  const loadMore = async () => {
    if (loadLock.current) return;
    if (!hasMoreRef.current) return;

    loadLock.current = true;
    setLoading(true);
    setError(null);

    try {
      const res = await api({
        page: pageRef.current,
        pageSize,
        ...paramsRef.current,
      });

      const items = getItems(res);
      setList((prev) => [...prev, ...items]);

      pageRef.current += 1;
      hasMoreRef.current = items.length >= pageSize;
    } catch (err) {
      setError(err);
      onError?.(err, "loadMore");
    } finally {
      loadLock.current = false;
      setLoading(false);
    }
  };

  const refresh = async () => {
    if (loadLock.current) return;

    loadLock.current = true;
    setRefreshing(true);
    setError(null);

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
    } catch (err) {
      setError(err);
      onError?.(err, "refresh");
    } finally {
      loadLock.current = false;
      setRefreshing(false);
    }
  };

  return {
    list,
    loading,
    refreshing,
    error, // ⭐ 对外暴露错误信息，UI 可做提示
    loadMore,
    refresh,
    updateParams,
    params: paramsRef.current,
    hasMore: hasMoreRef.current,
  };
}

```

