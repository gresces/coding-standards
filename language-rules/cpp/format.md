# C++ 代码格式规范

## 核心原则

C++ 格式应让资源所有权、调用边界、错误路径和数据结构一眼可见。

本规范从 `ASRS/llvm_service` 的 C++17 代码风格总结而来，覆盖排版、包含顺序、命名、函数布局、类和结构体组织、注释与提交前检查；不规定业务架构或算法选择。

## 基础格式

- 使用 4 个空格缩进，不使用 Tab。
- 文件末尾保留一个换行。
- 单行代码尽量保持在 100 个字符以内；复杂表达式优先拆成局部变量或多行参数。
- 左花括号与声明或控制语句保持同一行。
- 函数、类、结构体和较大的逻辑段之间保留一个空行。
- `if`、`for`、`while`、`switch` 的主体默认使用花括号；仅非常短的守卫返回可单行书写。
- 删除无用空行、无用包含、无用变量、废弃注释、调试输出和被注释掉的旧代码。

示例：

```cpp
std::string IrModule::getFunctionIR(const std::string& name) const {
    llvm::Function* function = findFunction(name);
    if (!function) return "";

    std::string ir;
    llvm::raw_string_ostream output(ir);
    function->print(output);
    return ir;
}
```

## 格式化工具

优先使用项目已有的 `.clang-format`、CMake 目标、脚本或 CI 命令。

如果项目没有格式化配置，推荐使用接近以下风格的 `clang-format` 配置，并根据现有代码微调：

```yaml
BasedOnStyle: LLVM
IndentWidth: 4
TabWidth: 4
UseTab: Never
ColumnLimit: 100
PointerAlignment: Left
ReferenceAlignment: Left
AllowShortIfStatementsOnASingleLine: WithoutElse
AllowShortFunctionsOnASingleLine: InlineOnly
BreakBeforeBraces: Attach
NamespaceIndentation: None
SortIncludes: false
```

没有项目命令时，可对修改过的文件运行：

```bash
clang-format -i path/to/file.cpp path/to/file.h
```

## 文件组织

- 头文件使用 `#pragma once`。
- `.h` 文件声明公共类型、轻量内联函数和接口；`.cpp` 文件放实现、内部 helper 和私有常量。
- 文件名沿用项目风格；没有约定时使用 `snake_case.cpp` / `snake_case.h`。
- 一个文件应围绕一个清晰模块组织，避免把无关 CLI、协议、算法和存储逻辑混在同一文件中。
- 测试或演示程序可放在单独的 `test_*.cpp` 文件中，避免污染库接口。

## 包含顺序

包含顺序以局部一致性优先。新文件推荐：

1. 当前 `.cpp` 对应的头文件
2. LLVM、第三方库或平台库头文件
3. C++ 标准库头文件
4. 当前项目内部头文件

每组之间保留一个空行。不要保留未使用的 `#include`。

示例：

```cpp
#include "ir_module.h"

#include "llvm/IR/CFG.h"
#include "llvm/IRReader/IRReader.h"
#include "llvm/Support/raw_ostream.h"

#include <filesystem>
#include <fstream>
#include <string>
#include <vector>
```

如果文件没有对应头文件，例如 CLI 入口文件，可按“标准库、系统头、第三方库、项目头”的既有顺序组织，并保持同文件一致。

## 命名格式

- 类型、类、结构体使用 `PascalCase`，例如 `IrModule`、`SessionManager`、`BasicBlockInfo`。
- 函数和方法使用 `camelCase`，例如 `load`、`moduleName`、`buildCallGraph`。
- 局部变量和结构体字段使用 `snake_case`，例如 `session_id`、`known_vars`、`type_str`。
- 私有成员变量使用尾随下划线，例如 `module_`、`loaded_path_`、`mutex_`。
- 文件作用域全局变量使用 `g_` 前缀，例如 `g_server_fd`、`g_sock_path`。
- 常量使用项目既有风格；局部 `constexpr` 常量优先使用 `snake_case`，例如 `alphabet_size`。
- 布尔变量应表达判断含义，例如 `is_loaded`、`has_value`、`show_value`。
- 避免 `data`、`obj`、`tmp`、`flag` 等宽泛名称，除非作用域极小且含义明确。

## 指针、引用和类型书写

- 指针和引用符号贴近类型：`llvm::Function* function`、`const std::string& name`。
- 不修改对象时使用 `const`；只读参数优先使用 `const T&`，小型标量按值传递。
- 表达所有权时优先使用标准库类型：`std::unique_ptr` 表示唯一所有权，`std::shared_ptr` 表示共享生命周期。
- 不拥有对象时使用裸指针或引用，并通过命名和空值检查表达可空性。
- 能由右侧清晰推断且不降低可读性时使用 `auto`；公共接口、复杂返回值和重要局部变量应写出具体类型。
- 需要可空返回时优先使用 `std::optional<T>`；不要用魔法值隐藏缺失状态，除非现有接口已经约定。

## 函数和控制流排版

- 函数应短小、单一职责；较长函数用清晰的局部 helper 和段落注释分隔。
- 优先使用早返回处理错误、空值和边界条件，减少嵌套。
- 短守卫可以单行书写：`if (!module_) return {};`。
- `else` 与前一个 `}` 保持同一行；能早返回时避免不必要的 `else`。
- 多行函数声明或调用时，后续参数与首个参数对齐，或在换行后使用 8 个空格缩进；同一文件中保持一致。
- 遍历容器时使用范围 `for`；只读遍历使用 `const auto&`，需要修改元素时使用 `auto&`。

示例：

```cpp
std::vector<BasicBlockInfo> IrModule::getCFG(const std::string& func,
                                             const std::vector<std::string>& labels) const {
    std::vector<BasicBlockInfo> blocks;
    llvm::Function* function = findFunction(func);
    if (!function) return blocks;

    for (auto& block : *function) {
        BasicBlockInfo info;
        info.name = bbName(&block);
        blocks.push_back(std::move(info));
    }

    return blocks;
}
```

## 类和结构体组织

- `public` 接口在前，`private` 数据和 helper 在后；访问控制标签顶格书写。
- 构造函数初始化列表每个成员独立成行，按成员声明顺序初始化。
- 结构体用于简单数据聚合，字段直接公开；类用于维护不变量或隐藏状态。
- 相关的数据结构靠近使用方或模块边界定义，避免调用方跨多个文件追踪简单返回类型。
- 对外接口应明确返回值含义，尤其是错误、空结果和部分成功状态。

示例：

```cpp
class Session {
public:
    explicit Session(std::string session_id)
        : session_id_(std::move(session_id)),
          last_access_(std::chrono::steady_clock::now()) {}

    const std::string& id() const { return session_id_; }

private:
    std::string session_id_;
    std::chrono::steady_clock::time_point last_access_;
};
```

## 注释格式

- 注释解释原因、约束、协议含义或非显然行为，不重复代码表面动作。
- 公共接口、返回结构、错误约定和跨模块协议应有简短注释。
- 大文件可使用分隔注释组织章节，但不要用装饰性注释掩盖混乱结构。
- 不保留临时代码、调试语句和被注释掉的旧实现。

## 结构分隔注释

- 每个类、每个顶层函数、每组连续的类内函数之前必须添加分隔注释行。
- 分隔注释使用 `----` 加名称，并用 `-` 延伸到整行。
- 类内函数按职责分组，例如 lifecycle、accessors、queries、mutation、helpers；组名应表达这一组方法的用途。
- C++ 使用 `//` 分隔行：

```cpp
// ---- IrModule ---------------------------------------------------------------
class IrModule {
public:
    // ---- lifecycle ----------------------------------------------------------
    bool load(const std::string& path, std::string& error);
    void unload();

    // ---- queries ------------------------------------------------------------
    std::string moduleName() const;
    std::vector<std::string> listFunctions() const;
};

// ---- printValue -------------------------------------------------------------
static std::string printValue(const llvm::Value* value) {
    ...
}
```

示例：

```cpp
// Starting from the given set of functions, BFS through the call graph.
// Returns all external declaration-only functions encountered.
std::vector<std::string> reachableApis(const std::vector<std::string>& starts) const;
```

## 错误处理和资源释放格式

- 错误路径应靠近触发条件，并返回带上下文的信息。
- 不吞掉异常或错误码；如果必须捕获异常，应限制捕获范围并保留原因。
- 文件描述符、socket、内存和锁应由 RAII 管理；必须使用 C API 时，关闭和清理逻辑要集中且可验证。
- 加锁范围尽量小，使用 `std::lock_guard` / `std::unique_lock` 表达生命周期。
- 不在普通业务函数中直接 `std::exit`，除非该函数是进程边界或信号处理路径。

## LLVM 代码约定

- LLVM 对象命名应表达 IR 层级：`module`、`function`、`block`、`inst`、`callee`。
- 使用 `llvm::dyn_cast` / `llvm::isa` / `llvm::cast` 表达类型判断，不使用 C 风格强转。
- 打印 LLVM IR 或类型时使用 `llvm::raw_string_ostream`。
- 遍历 IR 时按 Module → Function → BasicBlock → Instruction 的层级缩进，避免在深层循环中混入无关逻辑。
- 对可能不存在的 `Function*`、`BasicBlock*`、`DIFile*` 等指针先检查再访问。

## 提交前检查

修改 C++ 代码后，优先运行项目已有的格式化、构建和测试命令。

对 CMake 项目，通常至少运行：

```bash
cmake --build build
```

如果修改了测试或公共行为，运行受影响的测试可执行文件或项目定义的测试命令。没有可运行测试时，应至少完成编译验证，并说明缺失原因。

## 面向 LLM 的可读性

- 公共类型和成员顺序保持稳定：标识字段、类型/配置字段、状态字段、结果字段。
- 分支条件复杂时提取为命名布尔值或小 helper。
- 相似 JSON、IR、CFG、session 等结构使用一致字段顺序和换行方式。
- 避免隐藏副作用：加载、清理、网络 I/O、文件 I/O 和全局状态修改应从函数名或调用位置可见。
