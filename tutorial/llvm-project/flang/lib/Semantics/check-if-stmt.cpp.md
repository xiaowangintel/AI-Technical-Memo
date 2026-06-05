# check-if-stmt.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-if-stmt.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements semantic checking, symbol reasoning, and language-rule enforcement for check if stmt.
- **Purpose (CN)**: 实现 check if stmt 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14

````cpp
//===-- lib/Semantics/check-if-stmt.cpp -----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "check-if-stmt.h"
#include "flang/Parser/message.h"
#include "flang/Parser/parse-tree.h"
#include "flang/Semantics/tools.h"

namespace Fortran::semantics {
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
- **L9 EN**: Includes "check-if-stmt.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "check-if-stmt.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "flang/Parser/message.h" to access parse-tree, token, or source representation support.
  **L10 CN**: 引入 "flang/Parser/message.h" 以使用语法树、词法单元或源码表示支持。
- **L11 EN**: Includes "flang/Parser/parse-tree.h" to access parse-tree, token, or source representation support.
  **L11 CN**: 引入 "flang/Parser/parse-tree.h" 以使用语法树、词法单元或源码表示支持。
- **L12 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L12 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L14 EN**: Opens namespace scope `Fortran::semantics`.
  **L14 CN**: 打开命名空间作用域 `Fortran::semantics`。

### Lines 15-27

````cpp

void IfStmtChecker::Leave(const parser::IfStmt &ifStmt) {
  // C1143 Check that the action stmt is not an if stmt
  const auto &body{
      std::get<parser::UnlabeledStatement<parser::ActionStmt>>(ifStmt.t)};
  if (std::holds_alternative<common::Indirection<parser::IfStmt>>(
          body.statement.u)) {
    context_.Say(
        body.source, "IF statement is not allowed in IF statement"_err_en_US);
  }
}

} // namespace Fortran::semantics
````
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L16 EN**: Starts a function, method, lambda, or structured scope: `void IfStmtChecker::Leave(const parser::IfStmt &ifStmt) {`.
  **L16 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IfStmtChecker::Leave(const parser::IfStmt &ifStmt) {`。
- **L17 EN**: Comment explains nearby logic, intent, or metadata: `C1143 Check that the action stmt is not an if stmt`.
  **L17 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1143 Check that the action stmt is not an if stmt`。
- **L18 EN**: Continues the surrounding expression or declaration: `const auto &body{`.
  **L18 CN**: 继续构造周围的表达式或声明：`const auto &body{`。
- **L19 EN**: Executes a call or declaration centered on `std::get<parser::UnlabeledStatement<parser::ActionStmt>>`.
  **L19 CN**: 执行以 `std::get<parser::UnlabeledStatement<parser::ActionStmt>>` 为核心的调用或声明。
- **L20 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L20 CN**: 开始 `if` 控制流语句并计算其条件。
- **L21 EN**: Continues the surrounding expression or declaration: `body.statement.u)) {`.
  **L21 CN**: 继续构造周围的表达式或声明：`body.statement.u)) {`。
- **L22 EN**: Continues logic associated with callable symbol `Say`.
  **L22 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L23 EN**: Executes a standalone statement or declaration: `body.source, "IF statement is not allowed in IF statement"_err_en_US);`.
  **L23 CN**: 执行一条独立语句或声明：`body.source, "IF statement is not allowed in IF statement"_err_en_US);`。
- **L24 EN**: Closes the current lexical scope or compound statement.
  **L24 CN**: 结束当前词法作用域或复合语句块。
- **L25 EN**: Closes the current lexical scope or compound statement.
  **L25 CN**: 结束当前词法作用域或复合语句块。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L27 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Fortran parse tree handling / Fortran 语法树处理**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Rule-checking workflow / 规则检查流程**

## Dependencies / 依赖关系

- `check-if-stmt.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Parser/message.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/parse-tree.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
