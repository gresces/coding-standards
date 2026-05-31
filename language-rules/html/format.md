# HTML 代码格式规范

## 核心原则

HTML 格式应优先保证结构可读、语义清晰、属性稳定、模板易于维护。

格式规范只约束排版、组织和命名，不规定业务逻辑、交互实现或组件架构。

## 基础格式

- 使用 2 个空格缩进，不使用 Tab。
- 文件末尾保留一个换行。
- 标签名、属性名使用小写。
- 使用双引号包裹属性值。
- 不省略可选闭合标签；显式写出 `</html>`、`</body>`、`</li>` 等闭合标签。
- 空元素使用自闭合写法，例如 `<meta charset="UTF-8" />`、`<input name="email" />`。
- 删除无用空行、废弃注释、未使用模板和重复属性。
- 同一层级的兄弟节点保持一致的缩进和换行方式。

## 文档结构

HTML 文档推荐保持稳定顺序：

1. `<!doctype html>`
2. `<html lang="...">`
3. `<head>`
4. 元信息 `meta`
5. 标题 `title`
6. 样式 `link`
7. `<body>`
8. 页面主体
9. 模板 `template`
10. 脚本 `script`

示例：

```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Example</title>
    <link rel="stylesheet" href="./styles/main.css" />
  </head>
  <body>
    <main class="app-shell">
      <h1>Example</h1>
    </main>

    <script type="module" src="./src/main.js"></script>
  </body>
</html>
```

## 标签排版

- 简短文本节点可以与标签同一行。
- 含有多个子节点、长文本、表单控件或嵌套结构时，每个子节点独占一行。
- 空行用于分隔明显的结构区域，不用于制造视觉间距。
- 避免把多个兄弟元素压在同一行。

推荐：

```html
<nav class="nav" aria-label="主导航">
  <button class="nav-item active" data-view="run">创建运行</button>
  <button class="nav-item" data-view="result">运行结果</button>
</nav>
```

不推荐：

```html
<nav class="nav"><button>创建运行</button><button>运行结果</button></nav>
```

## 属性排版

- 属性顺序保持稳定，优先级如下：
  1. 结构和语义属性：`id`、`class`、`role`、`aria-*`
  2. 行为和数据属性：`type`、`name`、`value`、`data-*`
  3. 资源属性：`href`、`src`、`alt`
  4. 状态属性：`disabled`、`checked`、`selected`、`required`
  5. 展示辅助属性：`placeholder`、`title`、`rows`、`cols`
- 属性较少且行长可控时，保持单行。
- 属性过长或影响阅读时，每个属性独占一行，并让闭合尖括号单独成行。

示例：

```html
<textarea
  class="full"
  name="idea"
  rows="4"
  required
  placeholder="描述你的想法或优化策略"
></textarea>
```

## class 和 id 命名格式

- `class` 使用小写短横线命名，例如 `app-shell`、`status-pill`。
- `id` 使用小驼峰或项目既有风格；同一文件内必须一致。
- `data-*` 使用小写短横线命名，例如 `data-view`、`data-run-id`。
- 避免只表达颜色或位置的类名，例如 `red-box`、`left-area`；优先表达结构或语义。

## 表单格式

- `label` 应包裹或显式关联对应控件。
- 表单控件属性顺序保持稳定：`type` / `name` / `value` / `required` / `placeholder`。
- 同一表单内相似字段使用一致的标签、控件和换行方式。
- 多行 `textarea` 保持开始标签、文本内容和结束标签结构清晰；无默认内容时不要插入空白文本。

## 模板和脚本

- `<template>` 放在主体结构之后、脚本之前。
- `template` 的 `id` 应清楚表达用途，例如 `run-form-template`。
- 脚本入口放在 `body` 末尾。
- ES Module 脚本显式写 `type="module"`。

## 注释格式

- 注释只用于解释结构原因、浏览器兼容性或非显而易见的约束。
- 不保留过期注释、临时注释和被注释掉的旧代码。
- 注释与相关结构相邻，不放在文件顶部堆积。

## 结构分隔注释

- 每个顶层结构块、顶层脚本函数、类，以及每组连续的类内函数之前必须添加分隔注释行。
- 分隔注释使用 `----` 加名称，并用 `-` 延伸到整行。
- HTML 使用 `<!-- ... -->`；`script` 中的 JavaScript / TypeScript 使用 `//`。

```html
<!-- ---- MainContent ------------------------------------------------------- -->
<main>
  ...
</main>

<script>
  // ---- AppController --------------------------------------------------------
  class AppController {
    // ---- events -------------------------------------------------------------
    handleSubmit(event) {
      ...
    }
  }
</script>
```

## 格式化工具

优先使用项目已经配置的格式化工具。

如果项目没有现成配置，推荐使用 Prettier，并尊重以下默认格式：

```json
{
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "singleQuote": false,
  "bracketSameLine": false
}
```

提交前优先运行项目已有的格式化命令；没有命令时，可运行：

```bash
npx prettier --write "**/*.html"
```
