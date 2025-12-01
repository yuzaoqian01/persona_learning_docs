### 产生原因

在 iOS 中，手指按住屏幕上下拖动，会触发 `touchmove` 事件。这个事件触发的对象是整个 `webview` 容器，容器自然会被拖动，剩下的部分会成空白。

在 W3C 文档中说

**`touchmove` 事件的速度是可以实现定义的，取决于硬件性能和其他实现细节**

**`preventDefault` 方法，阻止同一触点上所有默认行为，比如滚动。**

### 解决方案

#### 监听事件禁止滑动，通过监听 `touchmove`，让需要滑动的地方滑动，不需要滑动的地方禁止滑动。

```js
document.addEventListener(
  'touchmove',
  (e) => {
    const target = e.target as HTMLElement;
    if (target.closest('.scroll-container')) return; // 允许内部滚动
    e.preventDefault(); // 禁止外部滚动
  },
  { passive: false }
);

```

```css
html, body { overflow: hidden; height: 100%; }
.scroll-container { overflow-y: auto; -webkit-overflow-scrolling: touch; }

```

```ts
// 允许 .scroll-container 内滚动（增强版）
document.addEventListener(
  'touchmove',
  function (e) {
    const scroller = e.target.closest('.scroll-container');

    if (scroller) {
      // 若内容不够滚动，还是要 preventDefault，否则仍会触发回弹
      if (scroller.scrollHeight <= scroller.clientHeight) {
        e.preventDefault();
      }
      // 否则允许正常滚动
      return;
    }

    // 非滚动区域禁止
    e.preventDefault();
  },
  { passive: false }
);

// 处理顶部/底部卡住问题
document.querySelectorAll('.scroll-container').forEach((el) => {
  el.addEventListener('touchstart', function () {
    let top = el.scrollTop;
    let height = el.clientHeight;
    let total = el.scrollHeight;

    if (top === 0) {
      el.scrollTop = 1; // 关键：避免顶部卡住
    } else if (top + height === total) {
      el.scrollTop = top - 1; // 避免底部卡住
    }
  });
});

```

```css
html, body {
  overflow: hidden;
  height: 100%;
} /* 这样会导致不可滚动 需要注意使用 */

.scroll-container {
  overflow-y: auto;
  -webkit-overflow-scrolling: touch; /* 惯性滚动 */
}

```

```ts
document.body.addEventListener(
  "touchmove",
  (e) => {
    //阻止默认的处理方式(阻止下拉滑动的效果)
    e.preventDefault();
    // 这里可以允许内部元素滚动
    document.querySelector(".content").scrollTop = XXX;
    return false;
  },
  /**
   如果我们是为了阻止页面滚动添加了上述代码，默认行为就是滚动页
   面，但是如果我们阻止了这一默认行为，浏览器是无法预先知道的，
   必须等待事件监听器执行完成后，才知道要去阻止默认行为。等待监
   听器的执行是耗时的，，有些甚至耗时很明显，这样就会导致页面卡
   顿。即便监听器是个空函数，也会产生一定的卡顿，毕竟空函数的执
   行也会耗时。所以就有了passive属性，如果要阻止默认事件可以设
   置passive：false。
  */
  { passive: false }
);

```

