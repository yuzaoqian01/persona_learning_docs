1. 对象 / Props / API → interface
2. 联合 / 工具 / 状态 → type
3. 能用 interface 就不用 type（除非 type 更合适）

```ts
type Status = 'loading'|'success'|'error'
```

```ts
//扩展

interface User{
  name: string;
  sex: number
}

interface Man extends User{
  age: number
}

interface User {
  id: number;
  name: string;
}

type UserWithAge = User & {
  age: number;
};


```

