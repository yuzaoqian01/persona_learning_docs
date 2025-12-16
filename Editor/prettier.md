# 安装

```cmd
npm install -D prettier prettier-plugin-tailwindcss
```

### 格式化tailwind3

```cmd
npm install -D prettier prettier-plugin-tailwindcss
```

```js
// .prettierrc
{
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

编辑器默认格式化工具 format 改为prettier 打开onSave 时自动格式化

```json
// .editorconfig

# 表示这是项目根配置
root = true

# 所有文件通用规则
[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 2
insert_final_newline = true
trim_trailing_whitespace = true

# Markdown：不强制去掉行尾空格（用于换行）
[*.md]
trim_trailing_whitespace = false

# JSON / YAML
[*.{json,yml,yaml}]
indent_size = 2

# Shell
[*.sh]
indent_size = 2

# Flutter / Dart
[*.dart]
indent_size = 2

# Solidity（如果有 Web3）
[*.sol]
indent_size = 2


```

```json
// prettier

{
  "semi": false,
  "singleQuote": true,
  "printWidth": 100,
  "trailingComma": "none"
}

```

