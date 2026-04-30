从模糊到精确

### 提示词遵循SMART原则

```yaml
这是一个[项目类型]项目，使用[技术栈]。
主要功能是[核心功能描述]。
我们的编码规范包括：
- [规范1]
- [规范2]
- [规范3]

当前我需要[具体需求]，请基于以上背景提供解决方案。
```

```yaml
刚才生成的代码有个问题：当用户输入为空时，应该显示友好的提示信息，而不是抛出异常。
现在的行为是：[描述当前行为]
期望的行为是：[描述期望行为]
请修改相关的验证逻辑。
```

```yaml
我正在开发一个基于Next.js和TypeScript的博客系统。
目前已有：
- 文章展示功能
- 用户认证系统
- 基于Tailwind CSS的UI组件库

我想添加评论功能，包括：
- 用户可以对文章发表评论
- 支持评论回复
- 实时更新评论列表
- 评论内容支持基础的Markdown格式

请先帮我分析一下实现方案。


# TypeScript开发规则

## 类型定义规则
1. 所有公共接口必须有明确的类型定义
2. 避免使用 `any` 类型，必要时使用 `unknown`
3. 使用严格的TSConfig配置
4. 自定义类型放在 `types/` 目录下

## 组件设计规则
1. React组件使用函数式写法
2. Props接口命名为 `{ComponentName}Props`
3. 使用 `forwardRef` 处理ref传递
4. 组件文件结构：导入 → 类型 → 组件 → 导出

## 状态管理规则
1. 本地状态优先使用 `useState`
2. 复杂状态使用 `useReducer`
3. 全局状态通过Context或状态管理库
4. 状态更新必须是不可变的

请在编写代码时严格遵循这些规则。

# API开发规则

## 接口设计规则
1. 遵循RESTful设计原则
2. 使用HTTP状态码表示结果
3. 响应格式统一为JSON
4. 支持分页的接口必须包含元数据

## 错误处理规则
1. 统一的错误响应格式
2. 错误信息要对用户友好
3. 记录详细的错误日志
4. 敏感信息不能暴露给客户端

## 安全规则
1. 所有接口都要有认证检查
2. 使用HTTPS传输敏感数据
3. 输入数据必须验证和清理
4. 实施适当的限流策略

在实现API时，请确保每个接口都符合这些规则。
```

```1.yaml
你是一个专注于React/TypeScript开发的前端专家智能体。

你的专业领域：
- React组件设计和优化
- TypeScript类型系统
- CSS-in-JS和Tailwind CSS
- 前端性能优化
- 用户体验设计

工作原则：
1. 始终考虑组件的可复用性
2. 遵循React最佳实践
3. 确保类型安全
4. 优先考虑用户体验
5. 代码要易于测试

当接到任务时，你会：
1. 分析需求的UI/UX影响
2. 设计组件架构
3. 实现功能代码
4. 提供使用示例
5. 建议测试策略



你是一个专注于Node.js/Python后端开发的专家智能体。

你的职责范围：
- RESTful API设计
- 数据库设计和优化
- 安全性和认证
- 性能监控和优化
- 微服务架构

核心原则：
1. API设计要符合RESTful规范
2. 数据安全是第一优先级
3. 代码要有充分的错误处理
4. 性能优化从设计开始
5. 可扩展性要考虑在内

工作流程：
1. 理解业务需求
2. 设计数据模型
3. 定义API接口
4. 实现核心逻辑
5. 添加监控和日志
```

### 1. 基础组件生成

```markdown


用【Vue3/React】+【TS】+【Tailwind CSS/Element Plus/Ant Design】生成【组件名称，如：登录表单/商品卡片/分页组件】，要求：
1. 包含【具体功能，如：表单校验/分页切换/hover 动效】；
2. 支持【自定义属性，如：自定义颜色/尺寸/回调函数】；
3. 带完整 TS 类型定义、详细注释，符合 ESLint 规范；
4. 适配移动端响应式，兼容主流浏览器；
5. 输出完整可运行代码，复制就能直接导入项目。

```

### 2. 复杂组件封装

```markdown
帮我封装一个【复杂组件名称，如：树形表格/弹窗表单/下拉搜索选择器】，技术栈【Vue3/React + TS】，要求：
1. 核心功能：【详细描述功能，如：树形表格支持勾选、展开/折叠、搜索筛选；弹窗表单支持表单联动、提交校验】；
2. 性能优化：【如：懒加载、防抖节流、避免重复渲染】；
3. 可扩展性：支持插槽、自定义事件、Props 传参，方便后续二次开发；
4. 附带使用示例、TS 类型说明、常见问题备注；
5. 代码结构清晰，分模块编写，便于维护。

```

### 3. 报错快速修复

```markdown
帮我分析以下前端报错和对应代码，要求：
1. 报错信息：【粘贴完整报错信息，如：Uncaught TypeError: Cannot read properties of undefined (reading 'value')】；
2. 对应代码：【粘贴报错相关的完整代码片段】；
3. 请找出报错根因，给出详细解释，然后提供完整的修复代码；
4. 补充优化建议，避免以后再出现类似问题；
5. 修复后的代码要符合项目技术栈【Vue3/React + TS】规范，可直接替换使用。

```

### 4. 兼容性/Bug 排查

```markdown
我遇到一个前端问题：【详细描述问题，如：iOS 微信浏览器样式错乱、页面滚动卡顿、接口请求跨域失败、组件渲染异常】；
项目技术栈：【Vue3/React + TS + 具体框架/工具】；
请帮我：
1. 分析可能的问题原因，列出所有可能性；
2. 给出每一种原因的解决方案和完整代码；
3. 提供预防措施，避免后续出现类似兼容性/性能问题；
4. 方案要简单易操作，不用复杂配置，直接能落地。

```

### 5. 代码优化/重构

```markdown
帮我重构以下前端代码，项目技术栈【Vue3/React + TS】，要求：
1. 原始代码：【粘贴需要重构的代码片段】；
2. 重构目标：优化代码结构、移除冗余代码、修复潜在 Bug、提升代码可读性和可维护性；
3. 保留原有的所有功能，不改变业务逻辑；
4. 加入 TS 类型定义（如果没有），补充必要注释，符合 ESLint 规范；
5. 给出重构前后的对比说明，解释优化的原因和好处。

```

### 6. 版本升级迁移

```markdown
帮我将【旧版本技术，如：Vue2 组件/Vue3 旧语法/jQuery 代码】迁移到【新版本技术，如：Vue3 组合式 API/TS/React 函数组件】，要求：
1. 原始代码：【粘贴需要迁移的代码片段/文件】；
2. 迁移要求：完全保留原业务功能，兼容原有项目配置，不引入新的依赖；
3. 遵循新版本的最佳实践，如：Vue3 组合式 API 规范、React Hooks 规范；
4. 补充迁移说明，列出需要注意的细节和可能出现的问题及解决方案；
5. 输出完整的迁移后代码，可直接替换使用。

```

### 7. 样式快速生成/优化

```markdown
帮我写/优化【元素/组件】的样式，技术栈【Tailwind CSS/CSS3/SCSS】，要求：
1. 样式需求：【详细描述，如：居中显示、圆角、阴影、hover 动效、响应式适配（375px/768px/1200px）、深色模式兼容】；
2. 样式规范：符合项目设计规范，避免样式冲突，代码简洁可复用；
3. 优化要求：减少冗余样式，提升样式加载速度，兼容主流浏览器；
4. 输出完整的样式代码，可直接复制到项目中使用，并给出使用说明。

```

### 8. 交互效果实现

```markdown
帮我实现【交互效果，如：下拉菜单动画、弹窗淡入淡出、滚动加载、拖拽排序、表单联动】，技术栈【Vue3/React + JS/TS】，要求：
1. 交互细节：【详细描述，如：弹窗点击遮罩关闭、下拉菜单hover展开、拖拽时显示提示、滚动加载到底部自动请求数据】；
2. 性能要求：避免卡顿、防抖节流处理，不影响页面其他功能；
3. 兼容性：适配移动端和PC端，兼容主流浏览器；
4. 输出完整的代码（HTML/CSS/JS/TS），复制就能用，附带使用说明和注意事项。

```

### 9. 接口请求/类型生成

```markdown
根据以下接口文档，生成【Vue3/React】项目的接口请求代码，要求：
1. 接口信息：【粘贴接口文档，包含请求地址、请求方式、参数、返回值】；
2. 技术栈：【Axios + TS】；
3. 输出内容：
   - 完整的接口请求函数封装（包含请求拦截、响应拦截、错误处理）；
   - 所有接口参数和返回值的 TS 类型定义；
   - Mock 数据生成（用于本地调试）；
   - 接口调用示例；
4. 代码符合项目规范，可直接导入项目使用。

```

### 10. 测试用例/工程化配置

```markdown
帮我生成【组件/函数】的测试用例，或【工程化配置文件】，要求：
1. 目标：【如：为登录组件写单元测试、生成 ESLint 配置、生成 Vitest 配置、生成 Dockerfile】；
2. 技术栈：【Vitest/Jest/ESLint/Docker】；
3. 具体要求：【如：测试用例覆盖渲染、交互、边界情况；配置文件适配 Vue3/React + TS 项目，包含常用配置】；
4. 输出完整的代码/配置文件，可直接复制到项目中使用，并给出配置说明和使用方法。

```

```js
Always respond in Chinese-simplified

All methods must always include clear comments, including:
- Purpose of the method
- Description of parameters
- Description of return value
- Explanation of key logic (if complex)

请始终使用中文进行文档生成、注释编写、字段说明、函数描述、类型说明、提交说明和规则描述。

具体要求：
1. 代码逻辑可以保持英文命名，但所有注释必须使用中文。
2. README、开发文档、接口文档、使用说明、设计说明全部使用中文。
3. TypeScript / JavaScript / Dart / Flutter / React Native 等代码中的注释、JSDoc、TSDoc、说明性文本统一使用中文。
4. 对复杂逻辑、业务流程、边界条件、异常处理必须补充中文注释。
5. 如果生成配置文件、规则文件或规范文档，描述内容也必须使用中文。
6. 不要把变量名、函数名、类名强行翻译成中文，保持工程代码命名规范。
7. 回答技术方案时，优先用中文解释，并给出中文标题和中文步骤。


JSDoc
```

```md
You must strictly follow all rules below:

Language Rules:
- Always respond in Chinese (Simplified)
- All documentation, comments, field descriptions, function descriptions, type definitions, commit messages, and rule descriptions must be written in Chinese

Code Conventions:
- Variable names, function names, and class names must remain in English (do NOT translate them into Chinese)
- Every method must include detailed Chinese comments. Missing comments are NOT allowed

Comment Requirements (Mandatory):
All methods (functions) must include complete Chinese comments, including:
- Purpose (what the method does)
- Parameter descriptions (meaning of each parameter)
- Return value description
- Key logic explanation (especially for complex logic)
- Edge cases / error handling (if applicable)

Documentation Rules:
- README, development docs, API docs, usage guides, and design docs must all be written in Chinese
- Comments, JSDoc, TSDoc, and explanatory text in TypeScript / JavaScript / Dart / Flutter / React Native must be in Chinese
- Any generated config files, rule files, or specification documents must use Chinese for all descriptive content

Engineering Requirements:
- Complex logic, business flows, edge cases, and error handling must be clearly explained in Chinese comments
- Do NOT translate variable names, function names, or class names into Chinese
- Prioritize readability and completeness of Chinese explanations over minimal code

Response Guidelines:
- When answering technical questions, explanations must be in Chinese
- Prefer structured responses (e.g., headings, steps, bullet points)
```



```md

Always respond in Chinese (Simplified).
All methods must include detailed Chinese comments (purpose, parameters, return values, and key logic). Missing comments are not allowed.
Keep all code naming in English. All documentation and explanations must be in Chinese.
```

