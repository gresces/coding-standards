# Rust 编码指南

## 项目约束

1. 必须通过 cargo fmt
2. 必须通过 cargo clippy --all-targets --all-features -- -D warnings
3. 库代码禁止 unwrap / expect / panic，除非有明确说明
4. 默认 forbid unsafe_code
5. 公共 API 必须有 rustdoc，错误行为必须写 # Errors
6. 参数优先用 &str、&[T]、&Path，而不是强迫传 String、Vec、PathBuf
7. 公开结构体字段默认私有
8. 错误类型用 thiserror；应用层可用 anyhow
9. 模块默认私有，只通过 lib.rs 暴露稳定 API
10. 每个 PR 至少跑 fmt、clippy、test

## 所有权与参数设计

优先级大概是：

```rust
&str      // 只读字符串参数，优先于 String
&[T]      // 只读数组/Vec 参数，优先于 Vec<T>
impl AsRef<Path>  // 路径参数常用
impl Into<String> // 确实要接收并拥有字符串时
```

常见规范：

```rust
// 好：调用方可以传 String 或 &str
fn parse_name(name: &str) -> Result<User, ParseUserError>;

// 不好：强迫调用方交出所有权
fn parse_name(name: String) -> Result<User, ParseUserError>;
```

API Guidelines 的核心思想之一是：参数应尽量通过类型表达语义，避免用 bool / Option 搞模糊接口；复杂构造用 builder，新类型用 newtype 区分语义。

例如：

```rust
// 不好：true/false 含义不清楚
fn connect(addr: &str, secure: bool);

// 好：类型表达语义
enum Transport {
    Tcp,
    Tls,
}

fn connect(addr: &str, transport: Transport);
```

## 错误处理规范

库代码不要随便 unwrap() / expect() / panic!()。可恢复错误应该返回 Result<T, E>，Rust Book 也强调当函数可能失败时，使用 Result 表达成功值和错误值。

推荐：

```rust
pub fn load_config(path: &Path) -> Result<Config, ConfigError> {
    let content = std::fs::read_to_string(path)?;
    let config = toml::from_str(&content)?;
    Ok(config)
}
```

不推荐：

```rust
pub fn load_config(path: &Path) -> Config {
    let content = std::fs::read_to_string(path).unwrap();
    toml::from_str(&content).unwrap()
}
```

简单规则：

- 库代码：返回 Result，少 panic
- 二进制入口 main：可以统一处理错误
- 测试代码：unwrap/expect 可以接受
- 真正不可能失败的分支：expect("说明为什么不可能")

## API 设计规范

比较好的 Rust API 应该做到：

- 输入类型宽松，输出类型明确
- 错误类型稳定
- 公开字段尽量少
- trait 边界尽量写在函数上，而不是结构体定义上
- 不要过早暴露内部实现

## unsafe 规范

默认建议：

- 能不用 unsafe 就不用
- 用了 unsafe，必须写 SAFETY 注释
- unsafe 尽量封装在很小的模块里
- 对外暴露安全 API
- CI 里默认 forbid unsafe_code，确实需要时局部 allow

