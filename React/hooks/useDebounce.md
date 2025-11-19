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

