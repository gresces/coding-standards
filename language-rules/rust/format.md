# Rust 代码格式规范

## 核心原则

Rust 代码应让模块边界、导入顺序、命名、控制流形状、文档位置和测试布局一眼可见。

本规范覆盖排版、模块组织、导入、命名、函数布局、注释、结构分隔、测试文件布局和格式化检查；API 设计、错误处理、`unsafe` 边界和完整验证要求见 `language-rules/rust/logic.md`。

## 基础格式

- 使用 `rustfmt` 默认风格，避免手工对齐制造格式噪声。
- 文件末尾保留一个换行。
- 优先保持表达式直接、局部变量命名清楚；复杂链式调用应拆成命名中间值。
- 控制流主体使用花括号；短守卫返回可保持单行，但不能牺牲可读性。
- 删除无用空行、无用 `use`、无用变量、废弃注释、调试输出和被注释掉的旧代码。

示例：

```rust
fn load_user(id: UserId, store: &UserStore) -> Result<User, LoadError> {
    let record = store.fetch(id)?;
    if record.is_disabled {
        return Err(LoadError::Disabled { id });
    }

    User::try_from(record)
}
```

## 格式化工具

优先使用项目已有的 `rustfmt.toml`、Cargo alias、Makefile、CI 或脚本。

没有项目命令时，对修改过的 Rust 代码运行：

```bash
cargo fmt
```

如果仓库不是单一 Cargo workspace，应在包含目标 crate 的目录运行项目指定命令。

## 文件和模块组织

- 文件名和模块名使用 `snake_case`。
- `lib.rs` 作为库 crate 的入口，`main.rs` 只保留进程启动、参数解析和顶层错误处理的布局。
- 模块文件围绕一个清晰责任组织，避免把 CLI、协议、存储、网络和业务规则混在同一文件中。
- 导出列表集中放在 `lib.rs` 或模块顶部，避免实现内部散落 `pub use`。
- 测试优先放在靠近被测代码的 `#[cfg(test)] mod tests` 中；跨 crate 行为测试放在 `tests/`。

## `use` 顺序

导入顺序以局部一致性优先。新文件推荐：

1. 标准库 `std` / `core` / `alloc`
2. 第三方 crate
3. 当前 crate 的 `crate::...` 模块
4. 相对模块 `super::...` / `self::...`

每组之间保留一个空行。不要保留未使用的 `use`。

示例：

```rust
use std::collections::HashMap;
use std::time::Duration;

use anyhow::{Context, Result};
use tokio::time::timeout;

use crate::auth::SessionId;
use crate::store::UserStore;
```

## 命名格式

- 类型、trait、enum 和 enum variant 使用 `PascalCase`，例如 `UserStore`、`RequestState`。
- 函数、方法、模块、局部变量和字段使用 `snake_case`，例如 `load_user`、`session_id`。
- 常量和 `static` 使用 `SCREAMING_SNAKE_CASE`。
- 类型参数使用简短但有含义的 `PascalCase`，例如 `T`、`Item`、`Error`。
- 布尔变量应表达判断含义，例如 `is_ready`、`has_token`、`should_retry`。
- 避免 `data`、`value`、`tmp`、`flag` 等宽泛名称，除非作用域极小且含义明确。

## 类型书写格式

- 所有权、参数选择和 API 语义规则见 `language-rules/rust/logic.md`；本节只约束类型的可读排版。
- 公共接口中的复杂类型应显式写出；局部变量只在右侧类型不明显时标注。
- 泛型参数、trait bound 和 `where` 子句交给 rustfmt 换行；不要手工压缩到一行。
- 长 `Result<...>`、`Option<...>`、迭代器适配器类型等优先用命名类型或 helper 函数提升可读性。
- 避免用注释解释类型含义；需要表达语义时在逻辑层使用清晰类型名或 newtype。

## 函数布局和控制流排版

- 函数签名、参数列表、返回类型和 `where` 子句按 rustfmt 结果保留，不做手工对齐。
- 守卫返回、`let ... else`、错误传播和主路径之间用空行分隔，让成功路径连续可读。
- `match` 或闭包分支超过一个表达式时使用块，并把副作用语句与返回表达式分开。
- 复杂链式调用拆成命名中间值，避免一行同时承载解析、转换、错误映射和返回。

示例：

```rust
fn parse_limit(raw: Option<&str>) -> Result<usize, ConfigError> {
    let Some(raw_limit) = raw else {
        return Ok(DEFAULT_LIMIT);
    };

    raw_limit
        .parse::<usize>()
        .map_err(|source| ConfigError::InvalidLimit { source })
}
```

## 注释和文档

- 注释解释原因、约束、协议含义或非显然行为，不重复代码表面动作。
- 公共 API 的必需 rustdoc、`# Errors` 和 `unsafe` 安全约束见 `language-rules/rust/logic.md`；本节只规定注释位置和写法。
- `///` 用于公共 API 文档；普通实现细节使用 `//`。
- 大文件可使用分隔注释组织章节，但不要用装饰性注释掩盖混乱结构。
- 不保留临时代码、调试语句和被注释掉的旧实现。

## 结构分隔注释

- 每个顶层类型、trait、impl 组、顶层函数之前必须添加分隔注释行。
- 分隔注释使用 `----` 加名称，并用 `-` 延伸到整行。
- `impl` 内方法按职责分组，例如 lifecycle、accessors、queries、mutation、helpers；组名应表达这一组方法的用途。
- Rust 使用 `//` 分隔行：

```rust
// ---- Session ---------------------------------------------------------------
pub struct Session {
    id: SessionId,
}

// ---- Session lifecycle -----------------------------------------------------
impl Session {
    pub fn new(id: SessionId) -> Self {
        Self { id }
    }
}

// ---- load_session ----------------------------------------------------------
pub fn load_session(id: SessionId, store: &SessionStore) -> Result<Session, LoadError> {
    ...
}
```

## 测试规范

- 新行为应补充覆盖正常路径、边界条件和有意义的错误路径。
- 测试名称描述行为，例如 `returns_default_limit_when_missing`。
- 单元测试靠近实现；跨 crate 或 CLI 行为使用集成测试。
- 不为了测试暴露无关公共 API；优先测试安全公共接口和可观察行为。

## 格式检查

提交前格式检查优先使用项目命令；Rust 逻辑、lint 和测试验证要求见 `language-rules/rust/logic.md`。

## 面向 LLM 的可读性

- 公共类型字段顺序保持稳定：标识字段、配置字段、状态字段、结果字段。
- 分支条件复杂时提取为命名布尔值或小 helper。
- 相似请求、响应、错误和状态结构使用一致字段顺序和换行方式。
- 避免隐藏副作用：文件 I/O、网络 I/O、全局状态修改、线程创建和任务派发应从函数名或调用位置可见。
