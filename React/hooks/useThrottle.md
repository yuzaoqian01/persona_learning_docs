```ts
import { useRef } from 'react';

export const useThrottle = (fn: Function, delay = 300) => {
  const lastCall = useRef(0);

  return (...args: any[]) => {
    const now = Date.now();
    if (now - lastCall.current > delay) {
      lastCall.current = now;
      fn(...args);
    }
  };
};

```

