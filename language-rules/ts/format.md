# TypeScript 代码格式规范

## 核心原则

TypeScript 格式应同时呈现运行时代码和类型边界，让调用方能快速理解输入、输出和约束。

格式规范只约束排版、类型书写、导入顺序、命名和文件组织，不规定业务逻辑或框架选型。

## 基础格式

- 使用 2 个空格缩进，不使用 Tab。
- 文件末尾保留一个换行。
- 单行代码尽量保持在 100 个字符以内。
- 使用分号结束语句。
- 字符串默认使用双引号；同一文件中保持一致。
- 对象、数组、类型字面量和函数调用在跨多行时保留尾随逗号。
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
npx prettier --write "**/*.{ts,tsx}"
```

## 导入和导出格式

导入按以下顺序分组，每组之间保留一个空行：

1. 标准运行时或平台 API
2. 第三方依赖
3. 项目内部模块
4. 同目录相对模块
5. 仅类型导入

要求：

- 使用 `import type` 导入只在类型位置使用的符号。
- 不保留未使用导入。
- 不使用通配符导入，除非模块本身就是命名空间对象。
- 优先使用命名导出，避免默认导出和命名导出在同一模块中混乱混用。
- 导出类型和运行时代码时，保持相邻或按“类型在前、实现随后”的稳定顺序。

示例：

```ts
import { createClient } from "@example/sdk";

import { API_BASE } from "./config";
import type { RunRequest, RunResult } from "./types";
```

## 命名格式

- 变量、函数和方法使用 `camelCase`。
- 类、接口、类型别名、枚举使用 `PascalCase`。
- 常量使用 `UPPER_SNAKE_CASE`。
- 泛型参数使用简短且有含义的 `PascalCase`，例如 `TValue`、`TResult`。
- 布尔变量应表达判断含义，例如 `isActive`、`hasPermission`、`shouldRetry`。
- 文件名使用项目既有风格；没有约定时，同一目录内统一使用 `camelCase.ts` 或 `kebab-case.ts`。

## 类型书写格式

- 公共函数、模块边界、异步函数和复杂返回值必须显式标注返回类型。
- 局部变量能由右侧清晰推断时，不重复标注类型。
- 优先使用 `type` 表达组合、联合、映射类型；需要可扩展对象契约时使用 `interface`。
- 不使用 `any` 逃避类型问题；必须使用时应限制作用域并说明原因。
- 可选字段使用 `?`，不要用 `field: T | undefined` 表达普通可选属性。
- 联合类型较长时每个分支独占一行。

示例：

```ts
type RunState =
  | { status: "idle" }
  | { status: "loading"; requestId: string }
  | { status: "done"; result: RunResult }
  | { status: "error"; message: string };
```

## 函数和控制流排版

- 函数之间保留一个空行。
- 优先使用早返回减少嵌套。
- `if`、`for`、`while`、`switch` 始终使用花括号。
- `else` 与前一个 `}` 保持同一行；能早返回时避免不必要的 `else`。
- 参数超过三个或包含复杂类型时，优先使用对象参数并单独定义类型。
- 回调函数较长时提取为具名函数，避免深层缩进。

示例：

```ts
function resolveStatusLabel(state: RunState): string {
  if (state.status === "loading") {
    return "运行中";
  }

  if (state.status === "done") {
    return "已完成";
  }

  return "未开始";
}
```

## 对象、数组和类型字面量格式

- 简短对象可以单行书写。
- 多字段对象每个字段独占一行，并保留尾随逗号。
- 对象字段顺序应稳定：标识字段、配置字段、状态字段、回调字段。
- 类型字面量字段较多时每个字段独占一行，并使用分号结束字段声明。
- 数组元素较长或包含对象时，每个元素独占一行。

示例：

```ts
type RunRequest = {
  runId: string;
  idea: string;
  goals: string[];
  metadata?: Record<string, unknown>;
};
```

## TSX 格式

- TSX 属性使用双引号。
- 组件属性较少且行长可控时保持单行。
- 属性过多或包含回调时，每个属性独占一行。
- 布尔属性省略值，例如 `<Button disabled />`。
- 事件处理器命名使用 `handle` 前缀，例如 `handleSubmit`。

示例：

```tsx
<RunForm
  initialValue={initialValue}
  disabled={state.status === "loading"}
  onSubmit={handleSubmit}
/>
```

## 注释格式

- 注释解释原因，不重复代码表面行为。
- 公共类型、公共函数和不明显的约束可使用简短 JSDoc。
- 不保留临时代码、调试输出和被注释掉的旧实现。

## 结构分隔注释

- 每个类、每个顶层函数、每组连续的类内函数之前必须添加分隔注释行。
- 分隔注释使用 `----` 加名称，并用 `-` 延伸到整行。
- 类内函数按职责分组，例如 lifecycle、accessors、events、render、helpers；组名应表达这一组方法的用途。
- TypeScript / TSX 使用 `//` 分隔行；如果公共 API 还需要 JSDoc，分隔行放在 JSDoc 前面。

```ts
// ---- RunController ----------------------------------------------------------
class RunController {
  // ---- lifecycle ------------------------------------------------------------
  constructor(private readonly client: RunClient) {}

  // ---- actions --------------------------------------------------------------
  startRun(request: RunRequest): Promise<RunResult> {
    return this.client.startRun(request);
  }
}

// ---- createRunController ----------------------------------------------------
export function createRunController(client: RunClient): RunController {
  return new RunController(client);
}
```

## 面向 LLM 的可读性

- 类型定义靠近使用方或模块边界，避免跨文件追踪简单类型。
- 公共类型名称表达业务含义，不只表达结构形状。
- 控制流尽量线性，减少深层嵌套和隐式类型魔法。
- 相似对象、类型和函数使用一致的字段顺序与换行方式。
