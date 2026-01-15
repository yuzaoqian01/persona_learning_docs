**Open Graph（OG）协议** 是一套由 **Facebook 提出** 的网页元数据规范，用来告诉社交平台：

> “这个网页被分享出去时，应该怎么展示？”

简单说，它决定了**分享卡片的标题、描述、图片、类型等信息**。

------

### OG 协议是干嘛的

当你把一个链接发到：

- 微信
- Facebook
- Twitter（X）
- Telegram
- Discord

平台会去 **抓取网页 `<head>` 里的 OG 标签**，然后生成一张“分享卡片”。

没有 OG：

- 标题、图片可能乱
- 有的平台只显示链接

有 OG：

- 展示统一、可控
- 更专业，点击率更高

------

### OG 的基本写法

OG 使用 `<meta property="og:xxx" />` 的形式，必须写在 `<head>` 里。

最常用的一组是：

```
<meta property="og:title" content="页面标题" />
<meta property="og:description" content="页面简介" />
<meta property="og:image" content="https://example.com/og.png" />
<meta property="og:url" content="https://example.com/page" />
<meta property="og:type" content="website" />
```

------

### 每个字段的含义

- **og:title**
   分享时显示的标题
- **og:description**
   分享时显示的简介文本
- **og:image**
   分享卡片的大图（非常重要）
   建议比例 1.91:1，例如 1200×630
- **og:url**
   当前页面的标准 URL（防止重复内容）
- **og:type**
   页面类型
   常见值：
  - `website`
  - `article`（博客文章）
  - `video`
  - `profile`

------

### OG 和 SEO 的关系

OG **不是给搜索引擎用的**，而是给社交平台用的。

| 用途       | 标签                        |
| ---------- | --------------------------- |
| 浏览器标题 | `<title>`                   |
| SEO        | `title`、`meta description` |
| 社交分享   | `og:*`                      |

但在实际项目中，**OG 通常和 SEO 一起配置**。

------

### Twitter Card 和 OG 的关系

Twitter（X）在 OG 之上又加了一套：

```
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="标题" />
<meta name="twitter:description" content="描述" />
<meta name="twitter:image" content="https://example.com/og.png" />
```

不过：

- 如果没写 twitter 标签
- Twitter 会退化使用 OG

------

### 在 Astro / Web3 项目里的常见用途

- 官网 / Landing Page 分享
- 博客文章分享
- DApp 邀请链接
- 红包 / 邀请裂变链接（很常见）

------

一句话总结：

> **Open Graph 是“网页在社交平台上的外观说明书”**