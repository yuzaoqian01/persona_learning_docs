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
import { useReducer, useRef } from 'react';

function usePagingList(api, options = {}) {
  const { pageSize = 20, initParams = {} } = options;

  const initialState = {
    list: [],
    loading: false,
    refreshing: false,
    page: 1,
    hasMore: true,
    params: initParams,
  };

  function reducer(state, action) {
    switch (action.type) {
      case 'START_LOAD':
        return { ...state, loading: true };
      case 'END_LOAD':
        return { ...state, loading: false };
      case 'START_REFRESH':
        return { ...state, refreshing: true };
      case 'END_REFRESH':
        return { ...state, refreshing: false };
      case 'LOAD_SUCCESS':
        return {
          ...state,
          list: [...state.list, ...action.payload.items],
          page: state.page + 1,
          hasMore: action.payload.hasMore,
        };
      case 'REFRESH_SUCCESS':
        return {
          ...state,
          list: action.payload.items,
          page: 2,
          hasMore: action.payload.hasMore,
        };
      case 'UPDATE_PARAMS':
        return { ...state, params: { ...state.params, ...action.payload } };
      default:
        return state;
    }
  }

  const [state, dispatch] = useReducer(reducer, initialState);
  const loadLock = useRef(false);

  // 更新查询参数
  const updateParams = (newParams) => {
    dispatch({ type: 'UPDATE_PARAMS', payload: newParams });
    refresh(); // 条件变化自动刷新
  };

  const loadMore = async () => {
    if (loadLock.current || !state.hasMore) return;

    loadLock.current = true;
    dispatch({ type: 'START_LOAD' });

    try {
      const res = await api({ page: state.page, pageSize, ...state.params });
      const items = res.items ?? res.data ?? [];
      dispatch({
        type: 'LOAD_SUCCESS',
        payload: { items, hasMore: items.length >= pageSize },
      });
    } finally {
      loadLock.current = false;
      dispatch({ type: 'END_LOAD' });
    }
  };

  const refresh = async () => {
    if (loadLock.current) return;

    loadLock.current = true;
    dispatch({ type: 'START_REFRESH' });

    try {
      const res = await api({ page: 1, pageSize, ...state.params });
      const items = res.items ?? res.data ?? [];
      dispatch({
        type: 'REFRESH_SUCCESS',
        payload: { items, hasMore: items.length >= pageSize },
      });
    } finally {
      loadLock.current = false;
      dispatch({ type: 'END_REFRESH' });
    }
  };

  return {
    ...state,
    loadMore,
    refresh,
    updateParams,
  };
}

export default usePagingList;

```

