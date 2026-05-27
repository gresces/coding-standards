# Python 包管理器规范

## 核心原则

默认情况下，所有 Python 项目都使用 `uv` 进行开发管理。

除非项目根目录的 `README.md` 明确声明使用其他包管理器，否则不得使用 `pip`、`poetry`、`pdm`、`conda` 或其他工具来管理项目依赖、虚拟环境和运行环境。

## 适用范围

- 新建 Python 项目
- 现有 Python 项目的依赖安装、升级、移除和同步
- 本地开发环境搭建
- 测试、脚本、命令行工具和 CI/CD 中的 Python 命令执行
- `pyproject.toml`、`uv.lock` 等项目依赖文件的维护

## 默认要求

### 初始化项目

新建项目时优先使用：

```bash
uv init
```

如果项目已经存在，应优先检查是否已有 `pyproject.toml` 和 `uv.lock`。不要因为缺少传统的 `requirements.txt` 就改用 `pip`。

### 创建和同步环境

安装或恢复依赖时使用：

```bash
uv sync
```

需要同步开发依赖时，仍应优先使用 `uv sync`，并根据项目配置选择对应的 dependency group 或 extra。

### 添加依赖

添加运行时依赖：

```bash
uv add <package-name>
```

添加开发依赖：

```bash
uv add --dev <package-name>
```

除非已有明确理由，不要手动编辑 `pyproject.toml` 的依赖列表来替代 `uv add`。

### 移除依赖

```bash
uv remove <package-name>
```

移除依赖后，应确认相关导入、测试配置和工具配置也被同步清理。

### 运行 Python 命令

所有需要项目环境的命令，都通过 `uv run` 执行：

```bash
uv run python <script.py>
uv run pytest
uv run ruff check .
uv run mypy .
```

不要直接假设当前 shell 已经激活了正确的虚拟环境。

### 安装命令行工具

项目内工具优先作为开发依赖添加，再通过 `uv run` 使用：

```bash
uv add --dev ruff pytest mypy
uv run ruff check .
```

临时运行不属于项目依赖的工具时，可以使用 `uvx`：

```bash
uvx <tool-name>
```

但如果该工具是项目开发、测试、格式化或构建流程的一部分，应写入项目依赖，而不是长期依赖 `uvx`。

## 文件约定

- `pyproject.toml` 是 Python 项目的主要配置入口。
- `uv.lock` 应提交到版本控制，用于保证依赖解析结果可复现。
- `.venv/`、`__pycache__/`、`.pytest_cache/`、`.ruff_cache/` 等本地生成目录不得提交。
- 不要同时维护多套依赖来源，例如 `uv.lock`、`poetry.lock`、`Pipfile.lock`、手写 `requirements.txt` 并存。

## README 例外

只有当项目根目录 `README.md` 明确说明使用其他包管理器时，才允许按 README 执行。

推荐声明格式：

```markdown
## Python 包管理器

本项目使用 poetry 作为包管理器。

原因：本项目需要兼容既有发布流程和内部工具链。
常用命令：
- poetry install
- poetry run pytest
```

例外声明必须至少包含：

- 指定的包管理器名称
- 使用该包管理器的原因
- 常用安装、运行和测试命令

如果 README 没有清晰声明，则回到默认规则：使用 `uv`。

## 禁止事项

- 不要使用 `pip install <package>` 直接修改项目环境。
- 不要使用 `python -m pip install <package>` 管理项目依赖。
- 不要在未声明例外的项目中引入 `poetry.lock`、`Pipfile.lock`、`environment.yml` 等其他包管理文件。
- 不要提交 `.venv/` 或其他虚拟环境目录。
- 不要在 CI/CD 中绕过 `uv sync` 手动安装依赖。
- 不要依赖全局 Python 环境中已经安装的包。

## 推荐工作流

新项目：

```bash
uv init
uv add <runtime-dependency>
uv add --dev pytest ruff
uv run pytest
```

已有项目：

```bash
uv sync
uv run pytest
```

添加新依赖：

```bash
uv add <package-name>
uv run pytest
```

更新依赖后，应提交 `pyproject.toml` 和 `uv.lock` 的相关变化。
