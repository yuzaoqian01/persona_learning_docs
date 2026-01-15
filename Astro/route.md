### 静态 （SSG） 模式



因为必须在构建时确定所有路由，所以动态路由必须导出一个 `getStaticPaths()` ，它返回一个具有 `params` 属性的对象数组。数组中的每一个对象都会生成相应的路由。

`[dog].astro` 在其文件名中定义一个 `dog` 参数，则 `getStaticPaths()` 返回的这些对象的 `params` 中必须包含 `dog` 。 然后页面可以使用 `Astro.params` 访问该参数

中必须包含 `dog` 。 然后页面可以使用 `Astro.params` 访问该参数。



```astro

export function getStaticPaths() {
  return [
    { params: { dog: "clifford" }},

    { params: { dog: "rover" }},

    { params: { dog: "spot" }},

  ];

}




const { dog } = Astro.params;

---

<div>Good dog, {dog}!</div>
```

这将生成三个页面： `/dogs/clifford`、`/dogs/rover` 和 `/dogs/spot` ，每个页面显示相应的狗名。

文件名可以包含多个参数，这些参数必须都包含在 `getStaticPaths()` 的 `params` 对象中：

src/pages/[lang]-[version]/info.astro

```
---

export function getStaticPaths() {

  return [

    { params: { lang: "en", version: "v1" }},

    { params: { lang: "fr", version: "v2" }},

  ];

}




const { lang, version } = Astro.params;

---

...
```

这将生成 `/en-v1/info` 和 `/fr-v2/info` 路由。

参数也可以包含在路径的单独部分中。例如， `src/pages/[lang]/[version]/info.astro` 文件与上面相同的 `getStaticPaths()` 将生成路由 `/en/v1/info` 和 `/fr/v2/info`。



#### 解码 `params`

提供给 `getStaticPaths()` 函数的 `params` 不会被解码。当你需要解码参数值时，请使用 [`decodeURI`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/decodeURI) 函数。

src/pages/[slug].astro

```
---

export function getStaticPaths() {

  return [

    { params: { slug: decodeURI("%5Bpage%5D") }}, // 解码成 "[page]"
    
  ]

}

---
```

#### 剩余参数



如果你的URL路由需要更加灵活，你可以在 `.astro` 文件名中使用一个 [剩余参数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)（`[...path]`），去匹配任何深度的文件路径。

src/pages/sequences/[...path].astro

```
---

export function getStaticPaths() {

  return [

    { params: { path: "one/two/three" }},

    { params: { path: "four" }},

    { params: { path: undefined }}

  ]

}




const { path } = Astro.params;

---
```

这将生成 `/sequences/one/two/three` 、 `/sequences/four` 和 `/sequences` 。（将剩余参数设置为 `undefined` 允许它匹配顶级页面。）

剩余参数可以与 **其他命名参数** 一起使用。例如， GitHub 的文件查看器可以用以下动态路由表示：

```
/[org]/[repo]/tree/[branch]/[...file]
```

在这个示例中，对 `/withastro/astro/tree/main/docs/public/favicon.svg` 的请求将拆分为以下命名参数：

```
{

  org: "withastro",

  repo: "astro",

  branch: "main",

  file: "docs/public/favicon.svg"

}
```

多级动态路由

```html
---
export function getStaticPaths() {
  const pages = [
    {
      slug: undefined,
      title: "Astro Store",
      text: "Welcome to the Astro store!",
    },
    {
      slug: "products",
      title: "Astro products",
      text: "We have lots of products for you",
    },
    {
      slug: "products/astro-handbook",
      title: "The ultimate Astro handbook",
      text: "If you want to learn Astro, you must read this book.",
    },
  ];

  return pages.map(({ slug, title, text }) => {
    return {
      params: { slug },
      props: { title, text },
    };
  });
}

const { title, text } = Astro.props;
---
<html>
  <head>
    <title>{title}</title>
  </head>
  <body>
    <h1>{title}</h1>
    <p>{text}</p>
  </body>
</html>
```

### 按需动态路由



对于使用适配器进行 [按需渲染](https://www.astrojs.cn/zh-cn/guides/on-demand-rendering/) 的情况下，动态路由以相同的方式定义：在文件名中包含 `[param]` 或 `[...path]` 以匹配任意字符串或路径。但是由于不再提前构建路由，页面将提供给任何匹配的路由。由于这些路由不是“静态”的，因此不应使用 `getStaticPaths`。

对于按需渲染路由，文件名中应该只使用一次对剩余参数的展开语法（例如应该使用 `src/pages/[locale]/[...slug].astro` 或 `src/pages/[...locale]/[slug].astro`，而不是 `src/pages/[...locale]/[...slug].astro`）

src/pages/resources/[resource]/[id].astro

```
---

export const prerender = false; // 'server' 模式下，无需配置

const { resource, id } = Astro.params;

---

<h1>{resource}: {id}</h1>
```

该页面将为任意的 `resource` 和 `id` 提供服务： `resources/users/1`、 `resources/colors/blue` 等。



### 配置重定向



**添加于：** `astro@2.9.0`

在你的 Astro 配置中，你可以使用 [`redirects`](https://www.astrojs.cn/zh-cn/reference/configuration-reference/#redirects) 值来指定永久重定向的映射关系。

对于内部重定向，这是旧路由路径到新路由的映射。从 Astro v5.2.0 开始，还可以重定向到以 `http` 或 `https` 开头的外部 URL，并且 [可以被解析](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/canParse_static)：

astro.config.mjs

```
import { defineConfig } from "astro/config";



export default defineConfig({

  redirects: {

    "/old-page": "/new-page",

    "/blog": "https://example.com/blog"

  }

});
```

这些重定向遵循 [与基于文件路由相同的优先级规则](https://www.astrojs.cn/zh-cn/guides/routing/#路由优先级顺序)，并且优先级将会始终低于现有项目中同名的页面文件。例如：如果你的项目包含 `src/pages/old-page.astro` 文件，那么 `/old-page` 将不会重定向到 `/new-page`。



### 动态重定向



在全局的 `Astro` 对象上，`Astro.redirect` 方法允许你动态地重定向到另一个页面。例如，你可以在检查用户是否已登录（通过从 Cookie 中获取其会话）之后执行此操作。

src/pages/account.astro

```
---

import { isLoggedIn } from "../utils";




const cookie = Astro.request.headers.get("cookie");




// 如果用户未登录，将其重定向到登录页面。

if (!isLoggedIn(cookie)) {

  return Astro.redirect("/login");

}

---
```