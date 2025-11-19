```ts
import { useState, useCallback } from 'react';

export const useToggle = (initialValue = false) => {
  const [state, setState] = useState(initialValue);
  const toggle = useCallback(() => setState(prev => !prev), []);
  return [state, toggle] as const;
};

```

