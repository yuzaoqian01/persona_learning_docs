```ts
import { useEffect } from 'react';

/**
 * useScrollContainerLock
 * 作用：us
 * - 阻止非 scroll-container 区域的滚动（防止 iOS WebView 弹性回弹）
 * - 修复 scroll-container 顶部/底部卡住问题
 */
export const useScrollContainerLock = () => {
  useEffect(() => {
    // touchmove 全局处理
    const handleTouchMove = (e: TouchEvent) => {
      const scroller = (e.target as HTMLElement).closest('.scroll-container');

      if (scroller) {
        // 内容不够滚动，仍需 preventDefault 防止回弹
        if (scroller.scrollHeight <= scroller.clientHeight) {
          e.preventDefault();
        }
        return;
      }

      // 非滚动区域禁止滚动
      e.preventDefault();
    };

    // 处理顶部/底部卡住
    const handleTouchStart = (el: HTMLElement) => () => {
      const top = el.scrollTop;
      const height = el.clientHeight;
      const total = el.scrollHeight;

      if (top === 0) {
        el.scrollTop = 1; // 避免顶部卡住
      } else if (top + height === total) {
        el.scrollTop = top - 1; // 避免底部卡住
      }
    };

    document.addEventListener('touchmove', handleTouchMove, { passive: false });

    const containers = document.querySelectorAll<HTMLElement>('.scroll-container');
    containers.forEach((el) => el.addEventListener('touchstart', handleTouchStart(el)));

    // 清理事件
    return () => {
      document.removeEventListener('touchmove', handleTouchMove);
      containers.forEach((el) => el.removeEventListener('touchstart', handleTouchStart(el)));
    };
  }, []);
};

```

搭配使用

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
import { useEffect, useRef } from 'react';

/**
 * useScrollLock
 * 适用于移动端 H5 页面滚动管理
 * 功能：
 * 1. 阻止非 .scroll-container 区域滚动
 * 2. 修复 .scroll-container 顶部/底部卡住问题
 * 3. 可选锁定整个页面滚动（弹窗/Modal 时使用）
 */
export const useScrollLock = (lockPage = false) => {
  const scrollYRef = useRef(0);

  useEffect(() => {
    // touchmove 全局处理
    const handleTouchMove = (e: TouchEvent) => {
      const scroller = (e.target as HTMLElement).closest('.scroll-container');

      if (scroller) {
        // 内容不足滚动仍 preventDefault
        if (scroller.scrollHeight <= scroller.clientHeight) {
          e.preventDefault();
        }
        return;
      }

      // 非滚动区域禁止滚动
      e.preventDefault();
    };

    // 处理顶部/底部卡住
    const handleTouchStart = (el: HTMLElement) => () => {
      const top = el.scrollTop;
      const height = el.clientHeight;
      const total = el.scrollHeight;

      if (top === 0) {
        el.scrollTop = 1;
      } else if (top + height === total) {
        el.scrollTop = top - 1;
      }
    };

    // 锁定页面滚动（Modal 等场景）
    const lockPageScroll = () => {
      scrollYRef.current = window.scrollY;
      document.body.style.position = 'fixed';
      document.body.style.top = `-${scrollYRef.current}px`;
      document.body.style.width = '100%';
    };

    const unlockPageScroll = () => {
      document.body.style.position = '';
      document.body.style.top = '';
      document.body.style.width = '';
      window.scrollTo(0, scrollYRef.current);
    };

    document.addEventListener('touchmove', handleTouchMove, { passive: false });

    const containers = document.querySelectorAll<HTMLElement>('.scroll-container');
    containers.forEach((el) => el.addEventListener('touchstart', handleTouchStart(el)));

    if (lockPage) lockPageScroll();

    // 清理
    return () => {
      document.removeEventListener('touchmove', handleTouchMove);
      containers.forEach((el) => el.removeEventListener('touchstart', handleTouchStart(el)));
      if (lockPage) unlockPageScroll();
    };
  }, [lockPage]);
};

```

