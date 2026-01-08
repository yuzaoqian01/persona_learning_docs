在 CSS 中针对链接常用伪类的顺序是 **LVHA**（Link, Visited, Hover, Active）：

```css
a:link { color: blue; }    /* 未访问过的链接 */
a:visited { color: purple; } /* 已访问过的链接 */
a:hover { color: orange; }   /* 鼠标悬停 */
a:active { color: red; }     /* 点击时 */
```