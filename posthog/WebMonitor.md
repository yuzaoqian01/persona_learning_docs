```js
function report(data) {
  const url = '/api/log';
  // sendBeacon 只能发 POST，且无法自定义 Header（ content-type 默认为 text/plain ）
  // 通常用 Blob 包装一下
  const blob = new Blob([JSON.stringify(data)], { type: 'application/json' });

  if (navigator.sendBeacon) {
    navigator.sendBeacon(url, blob);
  } else {
    // 降级方案：创建一个 1x1 的图片
    const img = new Image();
    img.src = `${url}?data=${encodeURIComponent(JSON.stringify(data))}`;
  }
}



class WebMonitor {
  constructor(options = {}) {
    this.url = options.url;
    this.appId = options.appId;
    this.init();
  }

  init() {
    this.listenJSError();
    this.listenPromiseError();
    // 这里的白屏检测可以做成可配置开启
  }

  listenJSError() {
    window.addEventListener('error', (e) => {
      // 过滤资源错误
      if (e.target instanceof HTMLScriptElement || e.target instanceof HTMLImageElement) return; 
      this.report({ type: 'js', msg: e.message });
    });
  }

  listenPromiseError() {
    window.addEventListener('unhandledrejection', (e) => {
      this.report({ type: 'promise', msg: e.reason });
    });
  }

  report(data) {
    const payload = { ...data, appId: this.appId, time: Date.now() };
    navigator.sendBeacon(this.url, JSON.stringify(payload));
  }
}

// 使用
new WebMonitor({ url: 'https://monitor.com/api', appId: 'my-app' });
```

