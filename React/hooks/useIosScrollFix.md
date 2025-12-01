```ts
import { useEffect } from 'react';

export function useIosScrollFix(containerSelector = '.scroll-container') {
  useEffect(() => {
    const handleTouchMove = (e: TouchEvent) => {
      const path = e.composedPath();
      const element = path[0] as HTMLElement | undefined;

      if (!element) {
        e.preventDefault();
        return;
      }

      const scroller = element.closest(containerSelector) as HTMLElement | null;

      if (scroller) {
        // 内容不足一屏时禁止触发回弹
        if (scroller.scrollHeight <= scroller.clientHeight) {
          e.preventDefault();
        }
        return; // 允许滚动事件在 scroller 内继续
      }

      // 非可滚动区域 → 禁止滚动
      e.preventDefault();
    };

    const fixBoundaryBounce = (el: HTMLElement) => {
      const onTouchStart = () => {
        const top = el.scrollTop;
        const height = el.clientHeight;
        const total = el.scrollHeight;

        // 顶部和底部进行修正，避免 iOS 回弹
        if (top === 0) {
          el.scrollTop = 1;
        } else if (top + height === total) {
          el.scrollTop = top - 1;
        }
      };

      el.addEventListener('touchstart', onTouchStart);
      return () => el.removeEventListener('touchstart', onTouchStart);
    };

    // 监听 touchmove
    document.addEventListener('touchmove', handleTouchMove, { passive: false });

    // 处理所有滚动容器边界回弹
    const scrollers = document.querySelectorAll(containerSelector);
    const cleanups = Array.from(scrollers).map((el) =>
      fixBoundaryBounce(el as HTMLElement)
    );

    // 清理函数
    return () => {
      document.removeEventListener('touchmove', handleTouchMove);
      cleanups.forEach((fn) => fn());
    };
  }, [containerSelector]);
}

```



```ts
import { useEffect } from 'react';

export function useIosScrollFix(containerSelector = '.scroll-container') {
  useEffect(() => {
    const containers = Array.from(
      document.querySelectorAll<HTMLElement>(containerSelector)
    );

    // 阻止非滚动区域下拉
    const handleTouchMove = (e: TouchEvent) => {
      const target = e.target as HTMLElement | null;

      // 判断 target 是否在任意滚动容器内部
      const inScroller = containers.some((el) => el.contains(target!));

      if (!inScroller) {
        e.preventDefault(); // 非滚动区域禁止滚动
      } else {
        // 如果内容不足一屏，也阻止回弹
        const scroller = containers.find((el) => el.contains(target!));
        if (scroller && scroller.scrollHeight <= scroller.clientHeight) {
          e.preventDefault();
        }
      }
    };

    // 处理滚动容器顶部/底部回弹
    const fixBoundaryBounce = (el: HTMLElement) => {
      const onTouchStart = () => {
        const top = el.scrollTop;
        const height = el.clientHeight;
        const total = el.scrollHeight;

        if (top === 0) el.scrollTop = 1;
        else if (top + height === total) el.scrollTop = top - 1;
      };
      el.addEventListener('touchstart', onTouchStart);
      return () => el.removeEventListener('touchstart', onTouchStart);
    };

    // 绑定事件
    document.addEventListener('touchmove', handleTouchMove, { passive: false });
    const cleanups = containers.map(fixBoundaryBounce);

    // 清理
    return () => {
      document.removeEventListener('touchmove', handleTouchMove);
      cleanups.forEach((fn) => fn());
    };
  }, [containerSelector]);
}

```





```css
html, body {
  overflow: hidden;
  height: 100%;
}

.scroll-container {
  height: 100%;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch; /* 惯性滚动 */
}

```

