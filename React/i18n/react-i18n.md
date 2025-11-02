## 1. 安装

```cmd
# npm
$ npm install react-i18next i18next --save
```

```ts
import i18n from "i18next";
import { initReactI18next } from "react-i18next";

i18n
  .use(initReactI18next) 
  .init({
    resources: {
      zh: {
        translation: {
          "Email Login": "邮箱登录"
        }
      },
      en: {
        translation: {
          "Email Login": "Email Login"
        }
      }
    },
    lng: "en", 
    fallbackLng: "en",
    interpolation: {
      escapeValue: false 
    }
  });

export default i18n;
```

```js
i18next-browser-languagedetector // 是用来检测浏览器语言的插件

// init
export default {
  name: 'customLanguageDetector',

  lookup(options) {
    // 优先从 localStorage 获取
    const stored = localStorage.getItem('lang');
    if (stored) return stored;

    // 其次从 URL ?lang=zh 获取
    const params = new URLSearchParams(window.location.search);
    const urlLang = params.get('lang');
    if (urlLang) return urlLang;

    // 否则使用浏览器语言
    return navigator.language.split('-')[0] || 'en';
  },

  cacheUserLanguage(lng) {
    localStorage.setItem('lang', lng);
  },
};
// 配置文件
{
  // 👇 检测语言的优先顺序（从上到下依次尝试）
  // 可以从 URL 参数、哈希、cookie、本地存储、浏览器语言、HTML 标签、路径或子域名中检测
  order: [
    'querystring',     // 从 URL 查询参数，例如 ?lng=zh-CN
    'hash',            // 从 URL 哈希值，例如 #lng=en 或 #/de
    'cookie',          // 从 cookie 读取
    'localStorage',    // 从 localStorage 读取
    'sessionStorage',  // 从 sessionStorage 读取
    'navigator',       // 从浏览器的首选语言 navigator.language 读取
    'htmlTag',         // 从 <html lang="..."> 属性读取
    'path',            // 从路径部分读取，例如 /en/home
    'subdomain'        // 从子域名读取，例如 en.mydomain.com
  ],

  // 👇 指定每种方式下用于读取语言的键名或位置
  lookupQuerystring: 'lng',          // URL 查询参数名 ?lng=zh-CN
  lookupCookie: 'i18next',           // cookie 名称
  lookupLocalStorage: 'i18nextLng',  // localStorage 键名
  lookupSessionStorage: 'i18nextLng',// sessionStorage 键名
  lookupFromPathIndex: 0,            // 从路径的第几段取语言（0 表示第一段）
  lookupFromSubdomainIndex: 0,       // 从子域名的第几部分取语言（0 表示第一个）
  lookupHash: 'lng',                 // 从 URL 哈希中读取语言，例如 #lng=en 或 #/de
  lookupFromHashIndex: 0,            // 从哈希路径中取语言位置，例如 #/de

  // 👇 缓存语言到哪里（检测成功后会存储）
  caches: ['localStorage', 'cookie'],  // 可选 localStorage / cookie / sessionStorage
  excludeCacheFor: ['cimode'],         // 不缓存的语言（cimode 是 i18next 的调试语言模式）

  // 👇 cookie 设置（仅在 caches 含 'cookie' 时有效）
  cookieMinutes: 10,            // cookie 有效期（分钟）
  cookieDomain: 'myDomain',     // cookie 作用域（域名）

  // 👇 从哪个 HTML 标签读取语言（默认 document.documentElement）
  htmlTag: document.documentElement,

  // 👇 额外的 cookie 设置，可参考：
  // https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
  cookieOptions: { 
    path: '/', 
    sameSite: 'strict'          // SameSite 策略，防止 CSRF
  },

  // 👇 转换检测到的语言代码（可选）
  // 例如将 zh-CN 转成 zh_CN
  convertDetectedLanguage: (lng) => lng.replace('-', '_')
}


```

### 1️⃣ 从 localStorage 检测语言

```js
lookup(options) {
  return localStorage.getItem('lang') || 'en';
},
```

### 2️⃣ 从浏览器语言检测

```js
lookup(options) {
  return navigator.language || 'en';
},
```

### 3️⃣ 从 URL 参数检测

```js
lookup(options) {
  const params = new URLSearchParams(window.location.search);
  return params.get('lang') || 'en';
},
```

### 4️⃣ 缓存语言

```js
cacheUserLanguage(lng) {
  localStorage.setItem('lang', lng);
},
```

```js
// i18n.js（React 项目）
// 安装依赖：
// npm i i18next react-i18next i18next-http-backend i18next-browser-languagedetector

import i18next from 'i18next';
import { initReactI18next } from 'react-i18next';
import HttpBackend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';

/*
  下面是一个比较全面的初始化配置，注释里解释每一项的作用和适用场景。
  修改：根据你的项目路径、语言列表和是否 SSR 调整选项。
*/
i18next
  .use(HttpBackend) // 从服务器/静态文件加载语言文件（/locales/{{lng}}/{{ns}}.json）
  .use(LanguageDetector) // 浏览器端自动探测语言（querystring > localStorage > cookie > navigator > htmlTag）
  .use(initReactI18next) // 让 react-i18next 工作在 React 环境中
  .init({
    // —— 语言相关 —— //
    lng: 'zh-CN', // 可选：强制默认语言；通常不写由检测器决定
    fallbackLng: 'zh-CN', // 当某个键在当前语言缺失时回退的语言
    supportedLngs: ['zh-CN', 'en'], // 明确支持的语言列表（用于检测器白名单）
    load: 'all', // 加载策略：'all' 会加载 'zh-CN' 与 'zh' 这样的父语言

    // —— 资源 / 命名空间 —— //
    ns: ['common'], // 命名空间数组，可拆分大型应用的语言文件
    defaultNS: 'common',
    fallbackNS: 'common', // 找不到键时回落到哪个命名空间

    // 如果你愿意也可以直接内联资源（适用于小应用）
    // resources: {
    //   'zh-CN': { common: { welcome: '欢迎' } },
    //   en: { common: { welcome: 'Welcome' } }
    // },

    // —— 插值（模板）设置 —— //
    interpolation: {
      escapeValue: false, // React 已经做 XSS 防护，设为 false
      prefix: '{{', // 插值前缀
      suffix: '}}', // 插值后缀
      // 自定义格式化函数，例如用于货币/日期/大写
      format: (value, format, lng) => {
        if (!format) return value;
        if (format === 'uppercase') return String(value).toUpperCase();
        if (format === 'date') {
          // 简单示例，真实项目可接入 dayjs/moment 并考虑时区
          return new Date(value).toLocaleDateString(lng);
        }
        return value;
      },
    },

    // —— 自动检测（browser language detector） —— //
    detection: {
      // 检测顺序：优先 URL query -> localStorage -> cookie -> 浏览器设置 -> <html lang>
      order: ['querystring', 'localStorage', 'cookie', 'navigator', 'htmlTag'],
      // querystring 参数名，例如 ?lang=zh-CN
      lookupQuerystring: 'lang',
      // 如果使用 cookie，设置 cookie 名称
      lookupCookie: 'i18next',
      // localStorage 键名（当使用 caches 时会写入）
      lookupLocalStorage: 'lang',
      // 将检测到的语言缓存到 localStorage 或 cookie
      caches: ['localStorage'], // ['localStorage', 'cookie']
      // cookie 额外设置（只有在 caches 包含 'cookie' 时有效）
      cookieMinutes: 60 * 24 * 30, // 30 天
      cookieDomain: window && window.location ? window.location.hostname : undefined,
      // 从 <html lang="zh-CN"> 中读取语言
      htmlTag: document && document.documentElement,
      // 如果你已经设置 supportedLngs，启用 checkWhitelist 则只接受白名单语言
      checkWhitelist: true,
    },

    // —— 后端 / 动态加载 —— //
    backend: {
      // 语言文件放置路径，常见文件结构：/public/locales/{lng}/{ns}.json
      loadPath: '/locales/{{lng}}/{{ns}}.json',
      // addPath: '/locales/add/{{lng}}/{{ns}}' // 如果使用 saveMissing，上报缺失键的接口
      // allowMultiLoading: false,
    },

    // —— 缺失键（missing）处理 —— //
    saveMissing: false, // 是否把缺失的键发送到后端（需要 backend.addPath）
    missingKeyHandler: function(lng, ns, key, fallbackValue) {
      // 可自定义缺失键处理逻辑（例如上报日志）
      // console.warn('missing key', lng, ns, key, fallbackValue);
    },
    parseMissingKeyHandler: (key) => `[missing translation: ${key}]`, // 前端显示的占位文本

    // —— 键解析（如果你的 key 里包含 . 或 :，可以设置为 false） —— //
    keySeparator: '.', // 若 key 本身含 '.'，请设为 false
    nsSeparator: ':', // 命名空间分隔符 ('ns:key')，若含 ':'，设为 false

    // —— 返回值控制 —— //
    returnNull: false, // 当键不存在时返回 null？ false 会返回 key 或 fallback
    returnEmptyString: false, // 若翻译是空字符串，是否返回空
    returnObjects: false, // 是否允许返回对象（而非字符串）

    // —— React 适配选项 —— //
    react: {
      useSuspense: true, // 是否启用 React Suspense（若为 true 且文件异步加载，组件会挂起）
      wait: false, // 旧版本选项，现代版本用 useSuspense
      bindI18n: 'languageChanged loaded', // 监听事件触发组件重渲染
    },

    // —— 其它实用选项 —— //
    debug: false, // 开发时设 true 可查看加载和回退细节
    initImmediate: true, // 是否异步初始化（SSR 环境通常设为 false）
    saveMissingTo: 'all', // 将缺失键保存到 backend 或 all

    // —— 预加载（可选）—— //
    // preload: ['en', 'zh-CN'] // 如果你想初始化时就加载多个语言文件
  });

export default i18next;

```

