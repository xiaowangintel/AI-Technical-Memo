# check-return.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-return.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements semantic checking, symbol reasoning, and language-rule enforcement for check return.
- **Purpose (CN)**: 实现 check return 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14

````cpp
//===-- lib/Semantics/check-return.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "check-return.h"
#include "flang/Parser/message.h"
#include "flang/Parser/parse-tree.h"
#include "flang/Semantics/semantics.h"
#include "flang/Semantics/tools.h"
#include "flang/Support/Fortran-features.h"
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
- **L9 EN**: Includes "check-return.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "check-return.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "flang/Parser/message.h" to access parse-tree, token, or source representation support.
  **L10 CN**: 引入 "flang/Parser/message.h" 以使用语法树、词法单元或源码表示支持。
- **L11 EN**: Includes "flang/Parser/parse-tree.h" to access parse-tree, token, or source representation support.
  **L11 CN**: 引入 "flang/Parser/parse-tree.h" 以使用语法树、词法单元或源码表示支持。
- **L12 EN**: Includes "flang/Semantics/semantics.h" to access Fortran semantic analysis, symbol, and type information.
  **L12 CN**: 引入 "flang/Semantics/semantics.h" 以使用Fortran 语义分析、符号与类型信息。
- **L13 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L13 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L14 EN**: Includes "flang/Support/Fortran-features.h" to access shared Flang utility infrastructure.
  **L14 CN**: 引入 "flang/Support/Fortran-features.h" 以使用Flang 共享工具基础设施。

### Lines 15-28

````cpp

namespace Fortran::semantics {

static const Scope *FindContainingSubprogram(const Scope &start) {
  const Scope &scope{GetProgramUnitContaining(start)};
  return scope.kind() == Scope::Kind::MainProgram ||
          scope.kind() == Scope::Kind::Subprogram
      ? &scope
      : nullptr;
}

void ReturnStmtChecker::Leave(const parser::ReturnStmt &returnStmt) {
  // R1542 Expression analysis validates the scalar-int-expr
  // C1574 The return-stmt shall be in the inclusive scope of a function or
````
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L16 EN**: Opens namespace scope `Fortran::semantics`.
  **L16 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L18 EN**: Starts a function, method, lambda, or structured scope: `static const Scope *FindContainingSubprogram(const Scope &start) {`.
  **L18 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static const Scope *FindContainingSubprogram(const Scope &start) {`。
- **L19 EN**: Executes a call or declaration centered on `&scope{GetProgramUnitContaining`.
  **L19 CN**: 执行以 `&scope{GetProgramUnitContaining` 为核心的调用或声明。
- **L20 EN**: Returns from the current function with `scope.kind() == Scope::Kind::MainProgram ||`.
  **L20 CN**: 以 `scope.kind() == Scope::Kind::MainProgram ||` 从当前函数返回。
- **L21 EN**: Continues logic associated with callable symbol `kind`.
  **L21 CN**: 继续与可调用符号 `kind` 相关的逻辑。
- **L22 EN**: Continues the surrounding expression or declaration: `? &scope`.
  **L22 CN**: 继续构造周围的表达式或声明：`? &scope`。
- **L23 EN**: Executes a standalone statement or declaration: `: nullptr;`.
  **L23 CN**: 执行一条独立语句或声明：`: nullptr;`。
- **L24 EN**: Closes the current lexical scope or compound statement.
  **L24 CN**: 结束当前词法作用域或复合语句块。
- **L25 EN**: Blank line separating nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L26 EN**: Starts a function, method, lambda, or structured scope: `void ReturnStmtChecker::Leave(const parser::ReturnStmt &returnStmt) {`.
  **L26 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void ReturnStmtChecker::Leave(const parser::ReturnStmt &returnStmt) {`。
- **L27 EN**: Comment explains nearby logic, intent, or metadata: `R1542 Expression analysis validates the scalar-int-expr`.
  **L27 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1542 Expression analysis validates the scalar-int-expr`。
- **L28 EN**: Comment explains nearby logic, intent, or metadata: `C1574 The return-stmt shall be in the inclusive scope of a function or`.
  **L28 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1574 The return-stmt shall be in the inclusive scope of a function or`。

### Lines 29-42

````cpp
  // subroutine subprogram.
  // C1575 The scalar-int-expr is allowed only in the inclusive scope of a
  // subroutine subprogram.
  const auto &scope{context_.FindScope(context_.location().value())};
  if (const auto *subprogramScope{FindContainingSubprogram(scope)}) {
    if (returnStmt.v &&
        (subprogramScope->kind() == Scope::Kind::MainProgram ||
            IsFunction(*subprogramScope->GetSymbol()))) {
      context_.Say(
          "RETURN with expression is only allowed in SUBROUTINE subprogram"_err_en_US);
    } else if (subprogramScope->kind() == Scope::Kind::MainProgram) {
      context_.Warn(common::LanguageFeature::ProgramReturn,
          "RETURN should not appear in a main program"_port_en_US);
    }
````
- **L29 EN**: Comment explains nearby logic, intent, or metadata: `subroutine subprogram.`.
  **L29 CN**: 注释说明附近代码的逻辑、意图或元数据：`subroutine subprogram.`。
- **L30 EN**: Comment explains nearby logic, intent, or metadata: `C1575 The scalar-int-expr is allowed only in the inclusive scope of a`.
  **L30 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1575 The scalar-int-expr is allowed only in the inclusive scope of a`。
- **L31 EN**: Comment explains nearby logic, intent, or metadata: `subroutine subprogram.`.
  **L31 CN**: 注释说明附近代码的逻辑、意图或元数据：`subroutine subprogram.`。
- **L32 EN**: Executes a call or declaration centered on `&scope{context_.FindScope`.
  **L32 CN**: 执行以 `&scope{context_.FindScope` 为核心的调用或声明。
- **L33 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L33 CN**: 开始 `if` 控制流语句并计算其条件。
- **L34 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L34 CN**: 开始 `if` 控制流语句并计算其条件。
- **L35 EN**: Continues logic associated with callable symbol `kind`.
  **L35 CN**: 继续与可调用符号 `kind` 相关的逻辑。
- **L36 EN**: Starts a function, method, lambda, or structured scope: `IsFunction(*subprogramScope->GetSymbol()))) {`.
  **L36 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsFunction(*subprogramScope->GetSymbol()))) {`。
- **L37 EN**: Continues logic associated with callable symbol `Say`.
  **L37 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L38 EN**: Executes a standalone statement or declaration: `"RETURN with expression is only allowed in SUBROUTINE subprogram"_err_en_US);`.
  **L38 CN**: 执行一条独立语句或声明：`"RETURN with expression is only allowed in SUBROUTINE subprogram"_err_en_US);`。
- **L39 EN**: Transitions from the previous branch into an `else if` condition.
  **L39 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L40 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::LanguageFeature::ProgramReturn,`.
  **L40 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::LanguageFeature::ProgramReturn,`。
- **L41 EN**: Executes a standalone statement or declaration: `"RETURN should not appear in a main program"_port_en_US);`.
  **L41 CN**: 执行一条独立语句或声明：`"RETURN should not appear in a main program"_port_en_US);`。
- **L42 EN**: Closes the current lexical scope or compound statement.
  **L42 CN**: 结束当前词法作用域或复合语句块。

### Lines 43-46

````cpp
  }
}

} // namespace Fortran::semantics
````
- **L43 EN**: Closes the current lexical scope or compound statement.
  **L43 CN**: 结束当前词法作用域或复合语句块。
- **L44 EN**: Closes the current lexical scope or compound statement.
  **L44 CN**: 结束当前词法作用域或复合语句块。
- **L45 EN**: Blank line separating nearby declarations or logic blocks.
  **L45 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L46 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L46 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Fortran parse tree handling / Fortran 语法树处理**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Rule-checking workflow / 规则检查流程**

## Dependencies / 依赖关系

- `check-return.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Parser/message.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/parse-tree.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/semantics.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Support/Fortran-features.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
