```ts
async function promisePool<T>(
  tasks: (() => Promise<T>)[],
  limit: number
): Promise<T[]> {
  const results: T[] = [];
  let index = 0;

  async function worker() {
    while (index < tasks.length) {
      const cur = index++;
      results[cur] = await tasks[cur]();
    }
  }

  const workers = Array.from({ length: limit }, worker);
  await Promise.all(workers);

  return results;
}

```

