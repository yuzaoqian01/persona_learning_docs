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

