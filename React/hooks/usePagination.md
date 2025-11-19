```ts
import { useState, useMemo } from 'react';

export const usePagination = (totalItems: number, initialPage = 1, pageSize = 10) => {
  const [currentPage, setCurrentPage] = useState(initialPage);

  const totalPages = useMemo(() => Math.ceil(totalItems / pageSize), [totalItems, pageSize]);

  const next = () => setCurrentPage(prev => Math.min(prev + 1, totalPages));
  const prev = () => setCurrentPage(prev => Math.max(prev - 1, 1));
  const jump = (page: number) => setCurrentPage(Math.max(1, Math.min(page, totalPages)));

  return {
    currentPage,
    pageSize,
    totalPages,
    next,
    prev,
    jump,
  };
};

```

