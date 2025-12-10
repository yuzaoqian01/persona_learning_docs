```ts
// useDebounce.ts
import { useState, useEffect } from "react";

export function useDebounce<T>(value: T, delay: number) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debounced;
}

```

使用示例

```tsx
const [amout, setAmout] = useState('');
const debouncedAmount = useDebounce(amout, 500);

  useEffect(()=>{
        console.log('debouncedAmount',debouncedAmount)
    },[debouncedAmount])
```

```ts
import { useCallback, useRef } from 'react';

export function useDebounceFn<T extends (...args: any[]) => any>(
  fn: T,
  wait: number = 300
) {
  const timer = useRef<NodeJS.Timeout | null>(null);

  const run = useCallback(
    (...args: Parameters<T>) => {
      if (timer.current) {
        clearTimeout(timer.current);
      }

      timer.current = setTimeout(() => {
        fn(...args);
      }, wait);
    },
    [fn, wait]
  );

  const cancel = useCallback(() => {
    if (timer.current) {
      clearTimeout(timer.current);
      timer.current = null;
    }
  }, []);

  const flush = useCallback(
    (...args: Parameters<T>) => {
      if (timer.current) {
        clearTimeout(timer.current);
        timer.current = null;
        fn(...args);
      }
    },
    [fn]
  );

  return { run, cancel, flush };
}

```

