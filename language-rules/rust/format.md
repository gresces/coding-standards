# Rust 代码格式规范

## 核心原则

Rust 代码应让所有权、生命周期、错误路径、线程边界和不安全边界一眼可见。

本规范覆盖排版、模块组织、导入、命名、函数布局、错误处理、`unsafe` 使用、测试和提交前检查；不规定业务架构或算法选择。

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
- `lib.rs` 暴露库边界，`main.rs` 只保留进程启动、参数解析和顶层错误处理。
- 模块应围绕一个清晰责任组织，避免把 CLI、协议、存储、网络和业务规则混在同一文件中。
- 公共 API 通过 `pub use` 有意导出；不要为了调用方便扩大可见性。
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

## 所有权、借用和类型书写

- 优先借用而不是克隆；只有当所有权转移、生命周期简化或性能证据支持时才 `clone`。
- 函数参数不修改数据时使用共享引用；需要修改时使用 `&mut`，并让可变借用范围尽量短。
- 公共接口应写出清晰类型；局部变量可在右侧类型明显时使用推断。
- 字符串参数根据需求选择 `&str`、`String`、`Cow<'_, str>`，不要无故分配新 `String`。
- 集合参数优先接受 slice，例如 `&[Item]`，除非必须依赖具体容器能力。
- 返回缺失状态使用 `Option<T>`；返回失败原因使用 `Result<T, E>`，不要用魔法值隐藏状态。

## 函数和控制流

- 函数保持单一职责；长函数应拆成表达业务含义的小 helper。
- 优先使用早返回处理错误、空值和边界条件，减少嵌套。
- 用 `?` 传播错误时保留上下文；跨模块边界不要丢失失败原因。
- `match` 分支应覆盖业务语义；避免把复杂逻辑塞进单个分支表达式。
- 闭包只用于局部转换或短生命周期回调；复杂逻辑应命名为函数。

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

## 错误处理

- 库代码返回具体错误类型或项目既有错误抽象；二进制入口可使用 `anyhow` 等项目已有方案。
- 错误应包含定位问题所需上下文，但不能泄露密码、token、cookie、密钥或个人敏感信息。
- 不要 `unwrap` / `expect` 处理可恢复错误；测试、常量初始化或已被不变量证明的路径除外。
- `panic!` 只用于不可恢复的内部不变量破坏，不用于普通输入校验或外部系统失败。
- 异步代码中不要持有阻塞锁跨 `await`；需要时缩小作用域或使用异步同步原语。

## `unsafe` 规范

- 默认禁止引入 `unsafe`；只有 FFI、裸指针、性能关键路径或封装底层不变量确实需要时才使用。
- 代码中任何 `unsafe` 部分都必须有相邻注释说明安全原因，包括 `unsafe` 块、`unsafe fn`、`unsafe impl`、FFI 边界和裸指针解引用。
- `unsafe` 块前使用 `// SAFETY:` 注释，说明调用满足了哪些前置条件、哪些不变量由谁维护、为什么不会造成未定义行为。
- `unsafe fn` 的文档必须包含 `# Safety` 段落，列出调用方必须满足的条件。
- `unsafe impl Send` / `unsafe impl Sync` 必须说明内部状态、别名规则和线程安全不变量。
- 尽量把 `unsafe` 封装在小范围私有 helper 中，对外暴露安全接口；不要让不安全前置条件扩散到普通业务代码。

示例：

```rust
/// # Safety
/// `ptr` must be valid for reads of `len` bytes and must remain alive for the returned slice lifetime.
unsafe fn bytes_from_raw<'a>(ptr: *const u8, len: usize) -> &'a [u8] {
    // SAFETY: The caller guarantees that `ptr` is readable for `len` bytes and lives for `'a`.
    unsafe { std::slice::from_raw_parts(ptr, len) }
}
```

## 注释和文档

- 注释解释原因、约束、协议含义或非显然行为，不重复代码表面动作。
- 公共类型、公共函数、错误约定、跨模块协议和安全不变量应有简短文档。
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

## 提交前检查

修改 Rust 代码后，优先运行项目已有的格式化、构建和测试命令。

没有项目命令时，通常至少运行：

```bash
cargo fmt
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all-targets --all-features
```

如果项目没有启用所有 feature，按项目 README、CI 或 Cargo workspace 约定选择对应命令，并说明无法运行的检查。

## 面向 LLM 的可读性

- 公共类型字段顺序保持稳定：标识字段、配置字段、状态字段、结果字段。
- 分支条件复杂时提取为命名布尔值或小 helper。
- 相似请求、响应、错误和状态结构使用一致字段顺序和换行方式。
- 避免隐藏副作用：文件 I/O、网络 I/O、全局状态修改、线程创建和任务派发应从函数名或调用位置可见。
