# check-purity.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-purity.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements semantic checking, symbol reasoning, and language-rule enforcement for check purity.
- **Purpose (CN)**: 实现 check purity 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

````cpp
//===-- lib/Semantics/check-purity.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "check-purity.h"
#include "flang/Parser/parse-tree.h"
#include "flang/Semantics/tools.h"

namespace Fortran::semantics {
void PurityChecker::Enter(const parser::ExecutableConstruct &exec) {
  if (InPureSubprogram() && IsImageControlStmt(exec)) {
    context_.Say(GetImageControlStmtLocation(exec),
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, intent, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明附近代码的逻辑、意图或元数据：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, intent, or metadata: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明附近代码的逻辑、意图或元数据：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, intent, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明附近代码的逻辑、意图或元数据：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes "check-purity.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "check-purity.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "flang/Parser/parse-tree.h" to access parse-tree, token, or source representation support.
  **L10 CN**: 引入 "flang/Parser/parse-tree.h" 以使用语法树、词法单元或源码表示支持。
- **L11 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L11 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Opens namespace scope `Fortran::semantics`.
  **L13 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L14 EN**: Starts a function, method, lambda, or structured scope: `void PurityChecker::Enter(const parser::ExecutableConstruct &exec) {`.
  **L14 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void PurityChecker::Enter(const parser::ExecutableConstruct &exec) {`。
- **L15 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L15 CN**: 开始 `if` 控制流语句并计算其条件。
- **L16 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetImageControlStmtLocation(exec),`.
  **L16 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetImageControlStmtLocation(exec),`。

### Lines 17-32

````cpp
        "An image control statement may not appear in a pure subprogram"_err_en_US);
  }
}

void PurityChecker::Enter(const parser::SubroutineSubprogram &subr) {
  const auto &stmt{std::get<parser::Statement<parser::SubroutineStmt>>(subr.t)};
  Entered(
      stmt.source, std::get<std::list<parser::PrefixSpec>>(stmt.statement.t));
}

void PurityChecker::Leave(const parser::SubroutineSubprogram &) { Left(); }

void PurityChecker::Enter(const parser::FunctionSubprogram &func) {
  const auto &stmt{std::get<parser::Statement<parser::FunctionStmt>>(func.t)};
  Entered(
      stmt.source, std::get<std::list<parser::PrefixSpec>>(stmt.statement.t));
````
- **L17 EN**: Executes a standalone statement or declaration: `"An image control statement may not appear in a pure subprogram"_err_en_US);`.
  **L17 CN**: 执行一条独立语句或声明：`"An image control statement may not appear in a pure subprogram"_err_en_US);`。
- **L18 EN**: Closes the current lexical scope or compound statement.
  **L18 CN**: 结束当前词法作用域或复合语句块。
- **L19 EN**: Closes the current lexical scope or compound statement.
  **L19 CN**: 结束当前词法作用域或复合语句块。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L21 EN**: Starts a function, method, lambda, or structured scope: `void PurityChecker::Enter(const parser::SubroutineSubprogram &subr) {`.
  **L21 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void PurityChecker::Enter(const parser::SubroutineSubprogram &subr) {`。
- **L22 EN**: Executes a call or declaration centered on `&stmt{std::get<parser::Statement<parser::SubroutineStmt>>`.
  **L22 CN**: 执行以 `&stmt{std::get<parser::Statement<parser::SubroutineStmt>>` 为核心的调用或声明。
- **L23 EN**: Continues logic associated with callable symbol `Entered`.
  **L23 CN**: 继续与可调用符号 `Entered` 相关的逻辑。
- **L24 EN**: Executes a call or declaration centered on `std::get<std::list<parser::PrefixSpec>>`.
  **L24 CN**: 执行以 `std::get<std::list<parser::PrefixSpec>>` 为核心的调用或声明。
- **L25 EN**: Closes the current lexical scope or compound statement.
  **L25 CN**: 结束当前词法作用域或复合语句块。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Continues logic associated with callable symbol `Leave`.
  **L27 CN**: 继续与可调用符号 `Leave` 相关的逻辑。
- **L28 EN**: Blank line separating nearby declarations or logic blocks.
  **L28 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L29 EN**: Starts a function, method, lambda, or structured scope: `void PurityChecker::Enter(const parser::FunctionSubprogram &func) {`.
  **L29 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void PurityChecker::Enter(const parser::FunctionSubprogram &func) {`。
- **L30 EN**: Executes a call or declaration centered on `&stmt{std::get<parser::Statement<parser::FunctionStmt>>`.
  **L30 CN**: 执行以 `&stmt{std::get<parser::Statement<parser::FunctionStmt>>` 为核心的调用或声明。
- **L31 EN**: Continues logic associated with callable symbol `Entered`.
  **L31 CN**: 继续与可调用符号 `Entered` 相关的逻辑。
- **L32 EN**: Executes a call or declaration centered on `std::get<std::list<parser::PrefixSpec>>`.
  **L32 CN**: 执行以 `std::get<std::list<parser::PrefixSpec>>` 为核心的调用或声明。

### Lines 33-48

````cpp
}

void PurityChecker::Leave(const parser::FunctionSubprogram &) { Left(); }

void PurityChecker::Enter(const parser::MainProgram &) { ++depth_; }
void PurityChecker::Leave(const parser::MainProgram &) { --depth_; }

bool PurityChecker::InPureSubprogram() const {
  return pureDepth_ >= 0 && depth_ >= pureDepth_;
}

bool PurityChecker::HasPurePrefix(
    const std::list<parser::PrefixSpec> &prefixes) const {
  bool result{false};
  for (const parser::PrefixSpec &prefix : prefixes) {
    if (std::holds_alternative<parser::PrefixSpec::Impure>(prefix.u)) {
````
- **L33 EN**: Closes the current lexical scope or compound statement.
  **L33 CN**: 结束当前词法作用域或复合语句块。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L35 EN**: Continues logic associated with callable symbol `Leave`.
  **L35 CN**: 继续与可调用符号 `Leave` 相关的逻辑。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L37 EN**: Continues logic associated with callable symbol `Enter`.
  **L37 CN**: 继续与可调用符号 `Enter` 相关的逻辑。
- **L38 EN**: Continues logic associated with callable symbol `Leave`.
  **L38 CN**: 继续与可调用符号 `Leave` 相关的逻辑。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L40 EN**: Starts a function, method, lambda, or structured scope: `bool PurityChecker::InPureSubprogram() const {`.
  **L40 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool PurityChecker::InPureSubprogram() const {`。
- **L41 EN**: Returns from the current function with `pureDepth_ >= 0 && depth_ >= pureDepth_`.
  **L41 CN**: 以 `pureDepth_ >= 0 && depth_ >= pureDepth_` 从当前函数返回。
- **L42 EN**: Closes the current lexical scope or compound statement.
  **L42 CN**: 结束当前词法作用域或复合语句块。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Continues logic associated with callable symbol `HasPurePrefix`.
  **L44 CN**: 继续与可调用符号 `HasPurePrefix` 相关的逻辑。
- **L45 EN**: Continues the surrounding expression or declaration: `const std::list<parser::PrefixSpec> &prefixes) const {`.
  **L45 CN**: 继续构造周围的表达式或声明：`const std::list<parser::PrefixSpec> &prefixes) const {`。
- **L46 EN**: Executes a standalone statement or declaration: `bool result{false};`.
  **L46 CN**: 执行一条独立语句或声明：`bool result{false};`。
- **L47 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L47 CN**: 开始 `for` 控制流语句并计算其条件。
- **L48 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L48 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 49-64

````cpp
      return false;
    } else if (std::holds_alternative<parser::PrefixSpec::Pure>(prefix.u) ||
        std::holds_alternative<parser::PrefixSpec::Elemental>(prefix.u)) {
      result = true;
    }
  }
  return result;
}

void PurityChecker::Entered(
    parser::CharBlock source, const std::list<parser::PrefixSpec> &prefixes) {
  if (depth_ == 2) {
    context_.messages().Say(source,
        "An internal subprogram may not contain an internal subprogram"_err_en_US);
  }
  if (HasPurePrefix(prefixes)) {
````
- **L49 EN**: Returns from the current function with `false`.
  **L49 CN**: 以 `false` 从当前函数返回。
- **L50 EN**: Transitions from the previous branch into an `else if` condition.
  **L50 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L51 EN**: Starts a function, method, lambda, or structured scope: `std::holds_alternative<parser::PrefixSpec::Elemental>(prefix.u)) {`.
  **L51 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::holds_alternative<parser::PrefixSpec::Elemental>(prefix.u)) {`。
- **L52 EN**: Executes a standalone statement or declaration: `result = true;`.
  **L52 CN**: 执行一条独立语句或声明：`result = true;`。
- **L53 EN**: Closes the current lexical scope or compound statement.
  **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Closes the current lexical scope or compound statement.
  **L54 CN**: 结束当前词法作用域或复合语句块。
- **L55 EN**: Returns from the current function with `result`.
  **L55 CN**: 以 `result` 从当前函数返回。
- **L56 EN**: Closes the current lexical scope or compound statement.
  **L56 CN**: 结束当前词法作用域或复合语句块。
- **L57 EN**: Blank line separating nearby declarations or logic blocks.
  **L57 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L58 EN**: Continues logic associated with callable symbol `Entered`.
  **L58 CN**: 继续与可调用符号 `Entered` 相关的逻辑。
- **L59 EN**: Continues the surrounding expression or declaration: `parser::CharBlock source, const std::list<parser::PrefixSpec> &prefixes) {`.
  **L59 CN**: 继续构造周围的表达式或声明：`parser::CharBlock source, const std::list<parser::PrefixSpec> &prefixes) {`。
- **L60 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L60 CN**: 开始 `if` 控制流语句并计算其条件。
- **L61 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.messages().Say(source,`.
  **L61 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.messages().Say(source,`。
- **L62 EN**: Executes a standalone statement or declaration: `"An internal subprogram may not contain an internal subprogram"_err_en_US);`.
  **L62 CN**: 执行一条独立语句或声明：`"An internal subprogram may not contain an internal subprogram"_err_en_US);`。
- **L63 EN**: Closes the current lexical scope or compound statement.
  **L63 CN**: 结束当前词法作用域或复合语句块。
- **L64 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L64 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 65-80

````cpp
    if (pureDepth_ < 0) {
      pureDepth_ = depth_;
    }
  } else if (InPureSubprogram()) {
    context_.messages().Say(source,
        "An internal subprogram of a pure subprogram must also be pure"_err_en_US);
  }
  ++depth_;
}

void PurityChecker::Left() {
  if (pureDepth_ == --depth_) {
    pureDepth_ = -1;
  }
}

````
- **L65 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L65 CN**: 开始 `if` 控制流语句并计算其条件。
- **L66 EN**: Executes a standalone statement or declaration: `pureDepth_ = depth_;`.
  **L66 CN**: 执行一条独立语句或声明：`pureDepth_ = depth_;`。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Transitions from the previous branch into an `else if` condition.
  **L68 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L69 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.messages().Say(source,`.
  **L69 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.messages().Say(source,`。
- **L70 EN**: Executes a standalone statement or declaration: `"An internal subprogram of a pure subprogram must also be pure"_err_en_US);`.
  **L70 CN**: 执行一条独立语句或声明：`"An internal subprogram of a pure subprogram must also be pure"_err_en_US);`。
- **L71 EN**: Closes the current lexical scope or compound statement.
  **L71 CN**: 结束当前词法作用域或复合语句块。
- **L72 EN**: Executes a standalone statement or declaration: `++depth_;`.
  **L72 CN**: 执行一条独立语句或声明：`++depth_;`。
- **L73 EN**: Closes the current lexical scope or compound statement.
  **L73 CN**: 结束当前词法作用域或复合语句块。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L75 EN**: Starts a function, method, lambda, or structured scope: `void PurityChecker::Left() {`.
  **L75 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void PurityChecker::Left() {`。
- **L76 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L76 CN**: 开始 `if` 控制流语句并计算其条件。
- **L77 EN**: Executes a standalone statement or declaration: `pureDepth_ = -1;`.
  **L77 CN**: 执行一条独立语句或声明：`pureDepth_ = -1;`。
- **L78 EN**: Closes the current lexical scope or compound statement.
  **L78 CN**: 结束当前词法作用域或复合语句块。
- **L79 EN**: Closes the current lexical scope or compound statement.
  **L79 CN**: 结束当前词法作用域或复合语句块。
- **L80 EN**: Blank line separating nearby declarations or logic blocks.
  **L80 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 81-81

````cpp
} // namespace Fortran::semantics
````
- **L81 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L81 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Fortran parse tree handling / Fortran 语法树处理**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Rule-checking workflow / 规则检查流程**

## Dependencies / 依赖关系

- `check-purity.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Parser/parse-tree.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
