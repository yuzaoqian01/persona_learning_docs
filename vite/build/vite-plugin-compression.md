## 1. 安装

```cmd
npm i vite-plugin-compression -D
```

## 2. 使用

```ts
// vite.config.ts 
import viteCompression from 'vite-plugin-compression';

export default () => {
  return {
    plugins: [viteCompression()],
  };
};
```

### 3.配置说明

| 参数               | 类型                                  | 默认值          | 说明                                                         |
| ------------------ | ------------------------------------- | --------------- | ------------------------------------------------------------ |
| verbose            | `boolean`                             | `true`          | 是否在控制台输出压缩结果                                     |
| filter             | `RegExp or (file: string) => boolean` | `DefaultFilter` | 指定哪些资源不压缩                                           |
| disable            | `boolean`                             | `false`         | 是否禁用                                                     |
| threshold          | `number`                              | -               | 体积大于 threshold 才会被压缩,单位 b                         |
| algorithm          | `string`                              | `gzip`          | 压缩算法,可选 [ 'gzip' , 'brotliCompress' ,'deflate' , 'deflateRaw'] |
| ext                | `string`                              | `.gz`           | 生成的压缩包后缀                                             |
| compressionOptions | `object`                              | -               | 对应的压缩算法的参数                                         |
| deleteOriginFile   | `boolean`                             | -               | 压缩后是否删除源文件                                         |

**DefaultFilter**

```
/\.(js|mjs|json|css|html)$/i
```