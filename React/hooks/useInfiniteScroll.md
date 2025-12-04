**监听滑动自动触发加载**

```ts
import { useState, useEffect, useRef } from 'react';

export const useInfiniteScroll = (callback: () => void) => {
  const [loading, setLoading] = useState(false);
  const callbackRef = useRef(callback);

  // 保持 callback 引用最新
  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  const handleScroll = () => {
    if (loading) return;
    const scrollTop = window.scrollY;
    const windowHeight = window.innerHeight;
    const fullHeight = document.documentElement.scrollHeight;

    if (scrollTop + windowHeight >= fullHeight - 50) { // 距底部 50px
      setLoading(true);
      callbackRef.current();
    }
  };

  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [loading]);

  const done = () => setLoading(false);

  return { loading, done };
};

```

```ts
import { useState, useEffect, useRef } from 'react';

export const useInfiniteScroll = (callback: () => Promise<void> | void) => {
  const [loading, setLoading] = useState(false);
  const callbackRef = useRef(callback);

  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  const handleScroll = () => {
    if (loading) return;
    const scrollTop = window.scrollY;
    const windowHeight = window.innerHeight;
    const fullHeight = document.documentElement.scrollHeight;

    if (scrollTop + windowHeight >= fullHeight - 50) {
      setLoading(true);
      const result = callbackRef.current();
      if (result && typeof (result as Promise<void>).then === 'function') {
        (result as Promise<void>).finally(() => setLoading(false));
      } else {
        setLoading(false);
      }
    }
  };

  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [loading]);

  const done = () => setLoading(false);

  return { loading, done };
};

```

```ts
import { useState, useEffect, useRef } from 'react';

export const useInfiniteScroll = (
  containerRef: React.RefObject<HTMLElement>,
  callback: () => Promise<void> | void
) => {
  const [loading, setLoading] = useState(false);
  const callbackRef = useRef(callback);

  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  const handleScroll = () => {
    if (!containerRef.current || loading) return;

    const el = containerRef.current;

    const scrollTop = el.scrollTop;
    const scrollHeight = el.scrollHeight;
    const clientHeight = el.clientHeight;

    if (scrollTop + clientHeight >= scrollHeight - 50) {
      setLoading(true);

      const r = callbackRef.current();
      if (r && typeof (r as Promise<void>).then === "function") {
        (r as Promise<void>).finally(() => setLoading(false));
      } else {
        setLoading(false);
      }
    }
  };

  useEffect(() => {
    const el = containerRef.current;
    if (!el) return;

    el.addEventListener("scroll", handleScroll);
    return () => el.removeEventListener("scroll", handleScroll);
  }, [containerRef.current, loading]);

  const done = () => setLoading(false);

  return { loading, done };
};

```

