剪切板限制必须在localhost 或者https环境才可以

### 方案一 

```js
async function readClipboard() {
  try {
    const text = await navigator.clipboard.readText();
    return text;
  } catch (err) {
    return null;
  }
}
```



### 方案二

```html
<input id="hiddenPaste" 
       style="opacity:0; position:fixed; top:-100px;" 
       type="text" />


```

```js
function readClipboardByManualPaste() {
  return new Promise((resolve) => {
    const input = document.getElementById("hiddenPaste");
    input.value = "";
    input.focus();

    // 必须等待用户手动粘贴
    const handler = () => {
      resolve(input.value);
      input.removeEventListener("input", handler);
    };

    input.addEventListener("input", handler);
  });
}
btn.onclick = async () => {
  const text = await readClipboardByManualPaste();
  console.log("剪切板内容:", text);
};

```

```js
export default {
  data() {
    return {
      manualInput: null
    };
  },

  mounted() {
    // 创建隐藏 input
    const input = document.createElement("input");
    input.type = "text";
    input.style.opacity = 0;
    input.style.position = "fixed";
    input.style.top = "-100px";
    input.style.left = "0";
    document.body.appendChild(input);
    this.manualInput = input;
  },

  methods: {
    async tryClipboardAPI() {
      if (!navigator.clipboard || !navigator.clipboard.readText) return null;

      try {
        const text = await navigator.clipboard.readText();
        if (text) return text;
      } catch {
        return null;
      }
    },

    manualPaste() {
      return new Promise((resolve) => {
        const input = this.manualInput;
        input.value = "";
        input.focus();

        const handler = () => {
          resolve(input.value);
          input.removeEventListener("input", handler);
        };

        input.addEventListener("input", handler);
      });
    },

    async readClipboard() {
      const text = await this.tryClipboardAPI();
      if (text) return text;
      return await this.manualPaste();
    }
  }
};

```

```vue
<template>
  <div>
    <button @click="paste">读取剪切板</button>
  </div>
</template>

<script>
import clipboardMixin from "./clipboardMixin";

export default {
  mixins: [clipboardMixin],

  methods: {
    async paste() {
      const text = await this.readClipboard();
      alert("剪切板内容：" + text);
    }
  }
};
</script>

```

```js
import { ref, onMounted } from "vue";

export function useClipboard() {
  const manualInputRef = ref(null);

  async function tryClipboardAPI() {
    if (!navigator.clipboard || !navigator.clipboard.readText) return null;

    try {
      const text = await navigator.clipboard.readText();
      if (text) return text;
    } catch (e) {
      return null;
    }
  }

  function manualPaste() {
    return new Promise((resolve) => {
      const input = manualInputRef.value;
      if (!input) return resolve(null);

      input.value = "";
      input.focus();

      const handler = () => {
        resolve(input.value);
        input.removeEventListener("input", handler);
      };

      input.addEventListener("input", handler);
    });
  }

  async function readClipboard() {
    const text = await tryClipboardAPI();
    if (text) return text;

    return await manualPaste();
  }

  // 必须渲染一个隐藏 input
  const manualPasteElement = () => (
    <input
      ref={manualInputRef}
      type="text"
      style="opacity:0; position:fixed; top:-100px; left:0;"
    />
  );

  return { readClipboard, manualInputRef, manualPasteElement };
}
vue3
```

```vue
<template>
  <div>
    <button @click="handlePaste">读取剪切板</button>

    <!-- 隐藏输入框：必须渲染 -->
    <component :is="manualPasteElement"></component>
  </div>
</template>

<script setup>
import { useClipboard } from "./useClipboard";

const { readClipboard, manualPasteElement } = useClipboard();

async function handlePaste() {
  const text = await readClipboard();
  alert("剪切板内容：" + text);
}
</script>

```

```js
import { useRef } from "react";

export function useClipboard() {
  const manualInputRef = useRef(null);

  // 自动尝试：Clipboard API
  async function tryClipboardAPI() {
    if (!navigator.clipboard || !navigator.clipboard.readText) return null;

    try {
      const text = await navigator.clipboard.readText();
      if (text) return text;
    } catch (e) {
      return null;
    }
  }

  // 最终兼容方案：隐藏 input + 手动粘贴
  function manualPaste() {
    return new Promise((resolve) => {
      const input = manualInputRef.current;
      if (!input) return resolve(null);

      input.value = "";
      input.focus();

      const handler = () => {
        resolve(input.value);
        input.removeEventListener("input", handler);
      };

      input.addEventListener("input", handler);
    });
  }

  // 主入口：自动选择最可靠方案
  async function readClipboard() {
    // Step 1: Clipboard API
    const text = await tryClipboardAPI();
    if (text) return text;

    // Step 2: manual paste fallback
    return await manualPaste();
  }

  // 在页面里需要渲染隐藏 input
  const manualPasteElement = (
    <input
      ref={manualInputRef}
      style={{
        opacity: 0,
        position: "fixed",
        top: "-100px",
        left: 0,
      }}
      type="text"
    />
  );

  return { readClipboard, manualPasteElement };
}

```

```jsx
import React from "react";
import { useClipboard } from "./useClipboard";

export default function App() {
  const { readClipboard, manualPasteElement } = useClipboard();

  async function handlePaste() {
    const text = await readClipboard();
    alert("剪切板内容: " + text);
  }

  return (
    <div style={{ padding: 20 }}>
      <button onClick={handlePaste}>读取剪切板</button>

      {/* 隐藏 input（必须渲染） */}
      {manualPasteElement}
    </div>
  );
}

```

