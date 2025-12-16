```ts
import { useRef, useCallback } from "react";

export function useDebouncedFn<T extends (...args: any[]) => void>(
  fn: T,
  delay: number
) {
  const timer = useRef<NodeJS.Timeout | null>(null);

  return useCallback((...args: Parameters<T>) => {
    if (timer.current) clearTimeout(timer.current);
    timer.current = setTimeout(() => fn(...args), delay);
  }, [fn, delay]);
}

```

