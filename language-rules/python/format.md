# Python 代码格式规范

## 核心原则

Python 代码应同时面向人类可读和 LLM 可读。

代码格式的目标不是展示技巧，而是降低理解成本：结构清晰、命名准确、依赖明确、控制流直接、文件组织稳定。

## 基础格式

- 遵循 PEP 8 的常规风格。
- 使用 4 个空格缩进，不使用 Tab。
- 单行代码应尽量保持在 88 到 100 个字符以内。
- 文件末尾保留一个换行。
- 删除无用空行、无用导入、无用变量和已废弃注释。
- 同一文件中保持一致的字符串风格；没有项目约定时，优先使用双引号。

## 格式化工具

优先使用项目已经配置的格式化和检查工具。

如果项目没有现成配置，推荐使用：

```bash
uv add --dev ruff
uv run ruff format .
uv run ruff check . --fix
```

如果项目已明确使用 `black`、`isort`、`flake8`、`pylint` 或其他工具，应尊重项目现有配置，不要擅自迁移工具链。

## 导入规范

导入按以下顺序分组，每组之间保留一个空行：

1. Python 标准库
2. 第三方库
3. 项目内部模块

示例：

```python
from pathlib import Path

import httpx
from pydantic import BaseModel

from app.config import Settings
from app.users.models import User
```

导入要求：

- 优先使用绝对导入。
- 不要使用通配符导入，例如 `from module import *`。
- 不要保留未使用的导入。
- 避免在函数内部导入，除非用于解决循环依赖、降低启动成本或隔离可选依赖。
- 如果导入顺序由工具管理，以工具结果为准。

## 命名规范

- 变量、函数、方法使用 `snake_case`。
- 类名使用 `PascalCase`。
- 常量使用 `UPPER_SNAKE_CASE`。
- 私有实现细节使用单下划线前缀，例如 `_normalize_path`。
- 布尔变量应表达判断含义，例如 `is_active`、`has_permission`、`should_retry`。
- 避免使用 `data`、`obj`、`item`、`temp` 这类含义过宽的名称，除非作用域极小且上下文明确。

命名应表达业务含义，而不只是表达类型。

```python
# 推荐
active_users = filter_active_users(users)

# 不推荐
list_data = filter_data(data)
```

## 类型标注

新增代码应优先添加类型标注，尤其是公共函数、模块边界、复杂数据结构和外部输入输出。

```python
def calculate_total(items: list[OrderItem]) -> Decimal:
    ...
```

类型标注要求：

- 函数参数和返回值尽量显式标注。
- 返回 `None` 的函数应写明 `-> None`。
- 对复杂字典、嵌套结构和跨模块传递的数据，优先使用 `dataclass`、`TypedDict`、`NamedTuple` 或 Pydantic model。
- 不要为了逃避类型问题滥用 `Any`。
- 如果必须使用 `Any`，应限制作用域，并让边界处尽快转换为明确类型。

## 函数格式

函数应短小、聚焦，并通过名称说明意图。

推荐结构：

1. 校验输入
2. 准备数据
3. 执行核心逻辑
4. 返回结果

```python
def build_invoice(customer: Customer, orders: list[Order]) -> Invoice:
    if not orders:
        raise ValueError("orders must not be empty")

    line_items = [build_line_item(order) for order in orders]
    total = calculate_total(line_items)

    return Invoice(customer=customer, line_items=line_items, total=total)
```

函数要求：

- 一个函数只承担一个清晰职责。
- 优先使用早返回减少嵌套。
- 避免超过三层嵌套。
- 避免在函数中混合解析、校验、业务处理、持久化和展示逻辑。
- 不要用过长的参数列表；参数过多时考虑引入配置对象、数据类或请求模型。

## 类和模块

类应表达稳定的概念或状态，不应只是函数集合的容器。

适合使用类的情况：

- 需要维护状态。
- 需要表达领域对象。
- 需要实现清晰的接口或协议。
- 需要把一组强相关行为绑定到同一实体上。

不适合使用类的情况：

- 只是为了放置静态方法。
- 只是为了模拟命名空间。
- 简单函数已经能清楚表达逻辑。

模块组织要求：

- 一个文件应围绕一个主题组织。
- 公共接口放在文件上方或明确位置。
- 内部辅助函数使用 `_` 前缀，并靠近调用方。
- 避免创建过大的 `utils.py`。工具函数应按领域或用途拆分，例如 `path_utils.py`、`date_parser.py`、`validation.py`。

## 注释和文档字符串

注释解释原因，不重复代码表面行为。

```python
# 推荐：解释为什么需要这个限制
# The upstream API rejects payloads larger than 1 MB.
if payload_size > MAX_PAYLOAD_SIZE:
    ...

# 不推荐：重复代码正在做什么
# Check if payload size is too large.
if payload_size > MAX_PAYLOAD_SIZE:
    ...
```

文档字符串适用于：

- 公共模块、公共类和公共函数。
- 行为不明显的业务规则。
- 有重要副作用、异常或边界条件的函数。

简单内部函数如果名称和类型标注已经足够清楚，可以不写文档字符串。

### 结构分隔注释

每个类、每个顶层函数、每组连续的类内函数之前必须添加分隔注释行。
分隔注释使用 `----` 加名称，并用 `-` 延伸到整行。类内函数按职责分组，
例如 lifecycle、accessors、queries、mutation、helpers；组名应表达这一组方法的用途。

```python
# ---- UserService ------------------------------------------------------------
class UserService:
    # ---- lifecycle ----------------------------------------------------------
    def __init__(self, repository: UserRepository) -> None:
        self.repository = repository

    # ---- queries ------------------------------------------------------------
    def get_user(self, user_id: str) -> User:
        ...

# ---- load_user --------------------------------------------------------------
def load_user(user_id: str) -> User:
    ...
```

## 错误处理

- 不要裸捕获 `except:`。
- 捕获异常时应指定具体异常类型。
- 不要静默吞掉异常，除非这是明确的业务要求。
- 重新抛出异常时保留上下文。
- 错误信息应包含足够定位问题的信息，但不要泄露密码、令牌、密钥或敏感数据。
- 有些函数需要内部处理所有的异常，这样的函数要保证不会造成额外的异常

```python
try:
    user = load_user(user_id)
except UserNotFoundError as exc:
    raise PermissionDeniedError(f"user {user_id} does not exist") from exc
```

## 日志规范

- 库代码不要直接使用 `print()` 输出运行信息。
- 应使用 `logging` 或项目已有日志系统。
- 日志文本应说明事件和关键上下文。
- 不要记录密钥、密码、token、cookie、个人敏感信息或完整凭证。
- 高频路径中的日志应控制级别和数量。

## 面向 LLM 的可读性

为了让 LLM 更稳定地理解、修改和扩展代码，应遵循以下约定：

- 代码结构保持显式，不依赖隐晦的元编程。
- 重要数据结构使用清晰类型，而不是任意嵌套字典。
- 函数名、变量名和文件名表达业务含义。
- 控制流尽量线性，减少深层嵌套和跨文件跳转。
- 边界条件集中处理，不把特殊情况散落在多个分支中。
- 不保留过期注释、临时代码和未使用的抽象。
- 测试名称描述行为，例如 `test_create_user_rejects_duplicate_email`。

## 提交前检查

提交代码前，优先运行：

```bash
uv run ruff format .
uv run ruff check .
uv run pytest
```

如果项目使用其他检查命令，以项目 README 或 CI 配置为准。
