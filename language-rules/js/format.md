# JavaScript 代码格式规范

## 核心原则

JavaScript 格式应让模块边界、数据流和 DOM 操作一眼可读。

格式规范只约束排版、导入顺序、命名和文件组织，不规定业务逻辑或框架选型。

## 基础格式

- 使用 2 个空格缩进，不使用 Tab。
- 文件末尾保留一个换行。
- 单行代码尽量保持在 100 个字符以内。
- 使用分号结束语句。
- 字符串默认使用双引号；同一文件中保持一致。
- 对象、数组和函数调用在跨多行时保留尾随逗号。
- 删除无用空行、无用导入、无用变量、废弃注释和被注释掉的旧代码。

## 格式化工具

优先使用项目已经配置的格式化和检查工具。

如果项目没有现成配置，推荐使用 Prettier，并尊重以下默认格式：

```json
{
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": false,
  "trailingComma": "all",
  "bracketSpacing": true,
  "arrowParens": "always"
}
```

提交前优先运行项目已有命令；没有命令时，可运行：

```bash
npx prettier --write "**/*.js"
```

## 导入和导出格式

导入按以下顺序分组，每组之间保留一个空行：

1. 标准运行时或平台 API
2. 第三方依赖
3. 项目内部模块
4. 同目录相对模块

要求：

- ES Module 项目优先使用 `import` / `export`，不要混用 `require`。
- 浏览器原生模块导入应保留明确扩展名，例如 `./api.js`。
- 不使用通配符导入，除非模块本身就是命名空间对象。
- 不保留未使用的导入。
- 默认导出和命名导出不要在同一模块中混乱混用；优先使用命名导出。

示例：

```js
import { checkHealth } from "./api.js";
import { API_BASE } from "./config.js";
import { qs } from "./utils/dom.js";
```

## 命名格式

- 变量、函数和方法使用 `camelCase`。
- 类和构造函数使用 `PascalCase`。
- 常量使用 `UPPER_SNAKE_CASE`。
- 文件名使用项目既有风格；没有约定时，普通模块使用 `camelCase.js` 或 `kebab-case.js`，同一目录必须一致。
- 布尔变量应表达判断含义，例如 `isActive`、`hasPermission`、`shouldRetry`。
- DOM 节点变量应能表达元素含义，例如 `statusElement`、`submitButton`。

## 函数和控制流排版

- 函数之间保留一个空行。
- 优先使用早返回减少嵌套。
- `if`、`for`、`while`、`switch` 始终使用花括号。
- `else` 与前一个 `}` 保持同一行；能早返回时避免不必要的 `else`。
- 回调函数较长时提取为具名函数，避免深层缩进。

推荐：

```js
function renderStatus(state) {
  if (state.loading) {
    return "运行中";
  }

  if (state.apiStatus === "ok") {
    return "API 正常";
  }

  return "未连接";
}
```

## 对象和数组格式

- 简短对象可以单行书写。
- 多字段对象每个字段独占一行，并保留尾随逗号。
- 对象字段顺序应稳定：标识字段、配置字段、状态字段、回调字段。
- 数组元素较长或包含对象时，每个元素独占一行。

示例：

```js
setState({
  apiStatus: health ? "ok" : "error",
  loading: false,
});
```

## DOM 和事件代码格式

- 选择器字符串集中、清晰，不使用难以维护的长链式选择器。
- DOM 查询结果变量命名应体现元素用途。
- 事件监听器回调使用具名函数或短箭头函数；复杂逻辑不要直接写在监听器内部。
- 链式调用超过一层时换行。

示例：

```js
const statusElement = qs("#connectionStatus");
statusElement.classList.remove("ok", "error");
```

## 注释格式

- 注释解释原因，不重复代码表面行为。
- 不保留临时代码、调试输出和被注释掉的旧实现。
- 公共函数行为不明显时，在函数上方使用简短 JSDoc；内部简单函数优先靠命名表达意图。

## 结构分隔注释

- 每个类、每个顶层函数、每组连续的类内函数之前必须添加分隔注释行。
- 分隔注释使用 `----` 加名称，并用 `-` 延伸到整行。
- 类内函数按职责分组，例如 lifecycle、accessors、events、render、helpers；组名应表达这一组方法的用途。
- JavaScript 使用 `//` 分隔行：

```js
// ---- RunController ----------------------------------------------------------
class RunController {
  // ---- lifecycle ------------------------------------------------------------
  constructor(client) {
    this.client = client;
  }

  // ---- actions --------------------------------------------------------------
  startRun(request) {
    return this.client.startRun(request);
  }
}

// ---- createRunController ----------------------------------------------------
export function createRunController(client) {
  return new RunController(client);
}
```

## 面向 LLM 的可读性

- 模块职责保持单一，导入列表保持清晰。
- 控制流尽量线性，减少深层嵌套和跨文件跳转。
- 相似代码使用一致的顺序、命名和换行方式。
- 不用格式技巧隐藏副作用；有副作用的函数名称应明确表达行为。
