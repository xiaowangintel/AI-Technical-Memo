# check-directive-structure.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-directive-structure.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Directive structure validity checks common to OpenMP, OpenACC and other directive language.
- **Purpose (CN)**: 实现 check directive structure 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===-- lib/Semantics/check-directive-structure.h ---------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// Directive structure validity checks common to OpenMP, OpenACC and other
// directive language.

#ifndef FORTRAN_SEMANTICS_CHECK_DIRECTIVE_STRUCTURE_H_
#define FORTRAN_SEMANTICS_CHECK_DIRECTIVE_STRUCTURE_H_

#include "flang/Common/enum-set.h"
#include "flang/Semantics/semantics.h"
#include "flang/Semantics/tools.h"
#include "llvm/ADT/iterator_range.h"

#include <set>
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
- **L9 EN**: Comment explains nearby logic, intent, or metadata: `Directive structure validity checks common to OpenMP, OpenACC and other`.
  **L9 CN**: 注释说明附近代码的逻辑、意图或元数据：`Directive structure validity checks common to OpenMP, OpenACC and other`。
- **L10 EN**: Comment explains nearby logic, intent, or metadata: `directive language.`.
  **L10 CN**: 注释说明附近代码的逻辑、意图或元数据：`directive language.`。
- **L11 EN**: Blank line separating nearby declarations or logic blocks.
  **L11 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L12 EN**: Starts a preprocessor conditional block: `#ifndef FORTRAN_SEMANTICS_CHECK_DIRECTIVE_STRUCTURE_H_`.
  **L12 CN**: 开始一个预处理条件块：`#ifndef FORTRAN_SEMANTICS_CHECK_DIRECTIVE_STRUCTURE_H_`。
- **L13 EN**: Defines macro `FORTRAN_SEMANTICS_CHECK_DIRECTIVE_STRUCTURE_H_` for conditional compilation or local shorthand.
  **L13 CN**: 定义宏 `FORTRAN_SEMANTICS_CHECK_DIRECTIVE_STRUCTURE_H_`，用于条件编译或本地简写。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Includes "flang/Common/enum-set.h" to access shared Flang utility infrastructure.
  **L15 CN**: 引入 "flang/Common/enum-set.h" 以使用Flang 共享工具基础设施。
- **L16 EN**: Includes "flang/Semantics/semantics.h" to access Fortran semantic analysis, symbol, and type information.
  **L16 CN**: 引入 "flang/Semantics/semantics.h" 以使用Fortran 语义分析、符号与类型信息。
- **L17 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L17 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L18 EN**: Includes "llvm/ADT/iterator_range.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L18 CN**: 引入 "llvm/ADT/iterator_range.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Includes <set> to access supporting declarations used by this translation unit.
  **L20 CN**: 引入 <set> 以使用当前编译单元使用的辅助声明。

### Lines 21-40

````cpp
#include <unordered_map>

namespace Fortran::semantics {

template <typename C, std::size_t ClauseEnumSize> struct DirectiveClauses {
  const common::EnumSet<C, ClauseEnumSize> allowed;
  const common::EnumSet<C, ClauseEnumSize> allowedOnce;
  const common::EnumSet<C, ClauseEnumSize> allowedExclusive;
  const common::EnumSet<C, ClauseEnumSize> requiredOneOf;
};

// Generic branching checker for invalid branching out of OpenMP/OpenACC
// directive.
// typename D is the directive enumeration.
template <typename D> class NoBranchingEnforce {
public:
  NoBranchingEnforce(SemanticsContext &context,
      parser::CharBlock sourcePosition, D directive,
      std::string &&upperCaseDirName)
      : context_{context}, sourcePosition_{sourcePosition},
````
- **L21 EN**: Includes <unordered_map> to access supporting declarations used by this translation unit.
  **L21 CN**: 引入 <unordered_map> 以使用当前编译单元使用的辅助声明。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L23 EN**: Opens namespace scope `Fortran::semantics`.
  **L23 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L25 EN**: Introduces template parameters or specialization context: `template <typename C, std::size_t ClauseEnumSize> struct DirectiveClauses {`.
  **L25 CN**: 为后续声明引入模板参数或特化上下文：`template <typename C, std::size_t ClauseEnumSize> struct DirectiveClauses {`。
- **L26 EN**: Executes a standalone statement or declaration: `const common::EnumSet<C, ClauseEnumSize> allowed;`.
  **L26 CN**: 执行一条独立语句或声明：`const common::EnumSet<C, ClauseEnumSize> allowed;`。
- **L27 EN**: Executes a standalone statement or declaration: `const common::EnumSet<C, ClauseEnumSize> allowedOnce;`.
  **L27 CN**: 执行一条独立语句或声明：`const common::EnumSet<C, ClauseEnumSize> allowedOnce;`。
- **L28 EN**: Executes a standalone statement or declaration: `const common::EnumSet<C, ClauseEnumSize> allowedExclusive;`.
  **L28 CN**: 执行一条独立语句或声明：`const common::EnumSet<C, ClauseEnumSize> allowedExclusive;`。
- **L29 EN**: Executes a standalone statement or declaration: `const common::EnumSet<C, ClauseEnumSize> requiredOneOf;`.
  **L29 CN**: 执行一条独立语句或声明：`const common::EnumSet<C, ClauseEnumSize> requiredOneOf;`。
- **L30 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L30 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L32 EN**: Comment explains nearby logic, intent, or metadata: `Generic branching checker for invalid branching out of OpenMP/OpenACC`.
  **L32 CN**: 注释说明附近代码的逻辑、意图或元数据：`Generic branching checker for invalid branching out of OpenMP/OpenACC`。
- **L33 EN**: Comment explains nearby logic, intent, or metadata: `directive.`.
  **L33 CN**: 注释说明附近代码的逻辑、意图或元数据：`directive.`。
- **L34 EN**: Comment explains nearby logic, intent, or metadata: `typename D is the directive enumeration.`.
  **L34 CN**: 注释说明附近代码的逻辑、意图或元数据：`typename D is the directive enumeration.`。
- **L35 EN**: Introduces template parameters or specialization context: `template <typename D> class NoBranchingEnforce {`.
  **L35 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D> class NoBranchingEnforce {`。
- **L36 EN**: Sets the following members to `public` access.
  **L36 CN**: 将后续成员的访问级别设为 `public`。
- **L37 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NoBranchingEnforce(SemanticsContext &context,`.
  **L37 CN**: 继续一个多行参数列表、初始化器或聚合项：`NoBranchingEnforce(SemanticsContext &context,`。
- **L38 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::CharBlock sourcePosition, D directive,`.
  **L38 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::CharBlock sourcePosition, D directive,`。
- **L39 EN**: Continues the surrounding expression or declaration: `std::string &&upperCaseDirName)`.
  **L39 CN**: 继续构造周围的表达式或声明：`std::string &&upperCaseDirName)`。
- **L40 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: context_{context}, sourcePosition_{sourcePosition},`.
  **L40 CN**: 继续一个多行参数列表、初始化器或聚合项：`: context_{context}, sourcePosition_{sourcePosition},`。

### Lines 41-60

````cpp
        upperCaseDirName_{std::move(upperCaseDirName)},
        currentDirective_{directive}, numDoConstruct_{0} {}
  template <typename T> bool Pre(const T &) { return true; }
  template <typename T> void Post(const T &) {}

  template <typename T> bool Pre(const parser::Statement<T> &statement) {
    currentStatementSourcePosition_ = statement.source;
    return true;
  }

  bool Pre(const parser::DoConstruct &) {
    numDoConstruct_++;
    return true;
  }
  void Post(const parser::DoConstruct &) { numDoConstruct_--; }
  void Post(const parser::ReturnStmt &) { EmitBranchOutError("RETURN"); }
  void Post(const parser::GotoStmt &gotoStmt) {
    if constexpr (std::is_same_v<D, llvm::acc::Directive>) {
      switch ((llvm::acc::Directive)currentDirective_) {
      case llvm::acc::Directive::ACCD_parallel:
````
- **L41 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `upperCaseDirName_{std::move(upperCaseDirName)},`.
  **L41 CN**: 继续一个多行参数列表、初始化器或聚合项：`upperCaseDirName_{std::move(upperCaseDirName)},`。
- **L42 EN**: Continues the surrounding expression or declaration: `currentDirective_{directive}, numDoConstruct_{0} {}`.
  **L42 CN**: 继续构造周围的表达式或声明：`currentDirective_{directive}, numDoConstruct_{0} {}`。
- **L43 EN**: Introduces template parameters or specialization context: `template <typename T> bool Pre(const T &) { return true; }`.
  **L43 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> bool Pre(const T &) { return true; }`。
- **L44 EN**: Introduces template parameters or specialization context: `template <typename T> void Post(const T &) {}`.
  **L44 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> void Post(const T &) {}`。
- **L45 EN**: Blank line separating nearby declarations or logic blocks.
  **L45 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L46 EN**: Introduces template parameters or specialization context: `template <typename T> bool Pre(const parser::Statement<T> &statement) {`.
  **L46 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> bool Pre(const parser::Statement<T> &statement) {`。
- **L47 EN**: Executes a standalone statement or declaration: `currentStatementSourcePosition_ = statement.source;`.
  **L47 CN**: 执行一条独立语句或声明：`currentStatementSourcePosition_ = statement.source;`。
- **L48 EN**: Returns from the current function with `true`.
  **L48 CN**: 以 `true` 从当前函数返回。
- **L49 EN**: Closes the current lexical scope or compound statement.
  **L49 CN**: 结束当前词法作用域或复合语句块。
- **L50 EN**: Blank line separating nearby declarations or logic blocks.
  **L50 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L51 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::DoConstruct &) {`.
  **L51 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::DoConstruct &) {`。
- **L52 EN**: Executes a standalone statement or declaration: `numDoConstruct_++;`.
  **L52 CN**: 执行一条独立语句或声明：`numDoConstruct_++;`。
- **L53 EN**: Returns from the current function with `true`.
  **L53 CN**: 以 `true` 从当前函数返回。
- **L54 EN**: Closes the current lexical scope or compound statement.
  **L54 CN**: 结束当前词法作用域或复合语句块。
- **L55 EN**: Continues logic associated with callable symbol `Post`.
  **L55 CN**: 继续与可调用符号 `Post` 相关的逻辑。
- **L56 EN**: Continues logic associated with callable symbol `Post`.
  **L56 CN**: 继续与可调用符号 `Post` 相关的逻辑。
- **L57 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::GotoStmt &gotoStmt) {`.
  **L57 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::GotoStmt &gotoStmt) {`。
- **L58 EN**: Continues logic associated with callable symbol `constexpr`.
  **L58 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L59 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L59 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L60 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_parallel:`.
  **L60 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_parallel:`。

### Lines 61-80

````cpp
      case llvm::acc::Directive::ACCD_serial:
      case llvm::acc::Directive::ACCD_kernels:
        if (labelsInBlock_.count(gotoStmt.v) == 0)
          EmitBranchOutOfComputeConstructError("GOTO");
        break;
      default:
        break;
      }
    }
  }
  void CollectLabel(parser::Label label) { labelsInBlock_.insert(label); }
  void Post(const parser::ExitStmt &exitStmt) {
    if (const auto &exitName{exitStmt.v}) {
      CheckConstructNameBranching("EXIT", exitName.value());
    } else {
      CheckConstructNameBranching("EXIT");
    }
  }
  void Post(const parser::CycleStmt &cycleStmt) {
    if (const auto &cycleName{cycleStmt.v}) {
````
- **L61 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_serial:`.
  **L61 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_serial:`。
- **L62 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_kernels:`.
  **L62 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_kernels:`。
- **L63 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L63 CN**: 开始 `if` 控制流语句并计算其条件。
- **L64 EN**: Executes a call or declaration centered on `EmitBranchOutOfComputeConstructError`.
  **L64 CN**: 执行以 `EmitBranchOutOfComputeConstructError` 为核心的调用或声明。
- **L65 EN**: Exits the nearest loop or switch statement.
  **L65 CN**: 退出最近的循环或 switch 语句。
- **L66 EN**: Introduces a switch dispatch label: `default:`.
  **L66 CN**: 引入一个 switch 分发标签：`default:`。
- **L67 EN**: Exits the nearest loop or switch statement.
  **L67 CN**: 退出最近的循环或 switch 语句。
- **L68 EN**: Closes the current lexical scope or compound statement.
  **L68 CN**: 结束当前词法作用域或复合语句块。
- **L69 EN**: Closes the current lexical scope or compound statement.
  **L69 CN**: 结束当前词法作用域或复合语句块。
- **L70 EN**: Closes the current lexical scope or compound statement.
  **L70 CN**: 结束当前词法作用域或复合语句块。
- **L71 EN**: Continues logic associated with callable symbol `CollectLabel`.
  **L71 CN**: 继续与可调用符号 `CollectLabel` 相关的逻辑。
- **L72 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::ExitStmt &exitStmt) {`.
  **L72 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::ExitStmt &exitStmt) {`。
- **L73 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L73 CN**: 开始 `if` 控制流语句并计算其条件。
- **L74 EN**: Executes a call or declaration centered on `CheckConstructNameBranching`.
  **L74 CN**: 执行以 `CheckConstructNameBranching` 为核心的调用或声明。
- **L75 EN**: Transitions from the previous branch into the alternative path.
  **L75 CN**: 从前一个分支过渡到备选路径。
- **L76 EN**: Executes a call or declaration centered on `CheckConstructNameBranching`.
  **L76 CN**: 执行以 `CheckConstructNameBranching` 为核心的调用或声明。
- **L77 EN**: Closes the current lexical scope or compound statement.
  **L77 CN**: 结束当前词法作用域或复合语句块。
- **L78 EN**: Closes the current lexical scope or compound statement.
  **L78 CN**: 结束当前词法作用域或复合语句块。
- **L79 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::CycleStmt &cycleStmt) {`.
  **L79 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::CycleStmt &cycleStmt) {`。
- **L80 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L80 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 81-100

````cpp
      CheckConstructNameBranching("CYCLE", cycleName.value());
    } else {
      if constexpr (std::is_same_v<D, llvm::omp::Directive>) {
        switch ((llvm::omp::Directive)currentDirective_) {
        // exclude directives which do not need a check for unlabelled CYCLES
        case llvm::omp::Directive::OMPD_do:
        case llvm::omp::Directive::OMPD_simd:
        case llvm::omp::Directive::OMPD_parallel_do:
        case llvm::omp::Directive::OMPD_parallel_do_simd:
        case llvm::omp::Directive::OMPD_distribute_parallel_do:
        case llvm::omp::Directive::OMPD_distribute_parallel_do_simd:
        case llvm::omp::Directive::OMPD_distribute_parallel_for:
        case llvm::omp::Directive::OMPD_distribute_simd:
        case llvm::omp::Directive::OMPD_distribute_parallel_for_simd:
        case llvm::omp::Directive::OMPD_target_teams_distribute:
        case llvm::omp::Directive::OMPD_target_teams_distribute_simd:
        case llvm::omp::Directive::OMPD_target_teams_distribute_parallel_do:
        case llvm::omp::Directive::
            OMPD_target_teams_distribute_parallel_do_simd:
          return;
````
- **L81 EN**: Executes a call or declaration centered on `CheckConstructNameBranching`.
  **L81 CN**: 执行以 `CheckConstructNameBranching` 为核心的调用或声明。
- **L82 EN**: Transitions from the previous branch into the alternative path.
  **L82 CN**: 从前一个分支过渡到备选路径。
- **L83 EN**: Continues logic associated with callable symbol `constexpr`.
  **L83 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L84 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L84 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L85 EN**: Comment explains nearby logic, intent, or metadata: `exclude directives which do not need a check for unlabelled CYCLES`.
  **L85 CN**: 注释说明附近代码的逻辑、意图或元数据：`exclude directives which do not need a check for unlabelled CYCLES`。
- **L86 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_do:`.
  **L86 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_do:`。
- **L87 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_simd:`.
  **L87 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_simd:`。
- **L88 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_parallel_do:`.
  **L88 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_parallel_do:`。
- **L89 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_parallel_do_simd:`.
  **L89 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_parallel_do_simd:`。
- **L90 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_distribute_parallel_do:`.
  **L90 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_distribute_parallel_do:`。
- **L91 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_distribute_parallel_do_simd:`.
  **L91 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_distribute_parallel_do_simd:`。
- **L92 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_distribute_parallel_for:`.
  **L92 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_distribute_parallel_for:`。
- **L93 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_distribute_simd:`.
  **L93 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_distribute_simd:`。
- **L94 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_distribute_parallel_for_simd:`.
  **L94 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_distribute_parallel_for_simd:`。
- **L95 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_target_teams_distribute:`.
  **L95 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_target_teams_distribute:`。
- **L96 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_target_teams_distribute_simd:`.
  **L96 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_target_teams_distribute_simd:`。
- **L97 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::OMPD_target_teams_distribute_parallel_do:`.
  **L97 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::OMPD_target_teams_distribute_parallel_do:`。
- **L98 EN**: Introduces a switch dispatch label: `case llvm::omp::Directive::`.
  **L98 CN**: 引入一个 switch 分发标签：`case llvm::omp::Directive::`。
- **L99 EN**: Continues the surrounding expression or declaration: `OMPD_target_teams_distribute_parallel_do_simd:`.
  **L99 CN**: 继续构造周围的表达式或声明：`OMPD_target_teams_distribute_parallel_do_simd:`。
- **L100 EN**: Returns from the current function with `void`.
  **L100 CN**: 以 `void` 从当前函数返回。

### Lines 101-120

````cpp
        default:
          break;
        }
      } else if constexpr (std::is_same_v<D, llvm::acc::Directive>) {
        switch ((llvm::acc::Directive)currentDirective_) {
        // exclude loop directives which do not need a check for unlabelled
        // CYCLES
        case llvm::acc::Directive::ACCD_loop:
        case llvm::acc::Directive::ACCD_kernels_loop:
        case llvm::acc::Directive::ACCD_parallel_loop:
        case llvm::acc::Directive::ACCD_serial_loop:
          return;
        default:
          break;
        }
      }
      CheckConstructNameBranching("CYCLE");
    }
  }

````
- **L101 EN**: Introduces a switch dispatch label: `default:`.
  **L101 CN**: 引入一个 switch 分发标签：`default:`。
- **L102 EN**: Exits the nearest loop or switch statement.
  **L102 CN**: 退出最近的循环或 switch 语句。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Transitions from the previous branch into an `else if` condition.
  **L104 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L105 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L105 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L106 EN**: Comment explains nearby logic, intent, or metadata: `exclude loop directives which do not need a check for unlabelled`.
  **L106 CN**: 注释说明附近代码的逻辑、意图或元数据：`exclude loop directives which do not need a check for unlabelled`。
- **L107 EN**: Comment explains nearby logic, intent, or metadata: `CYCLES`.
  **L107 CN**: 注释说明附近代码的逻辑、意图或元数据：`CYCLES`。
- **L108 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_loop:`.
  **L108 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_loop:`。
- **L109 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_kernels_loop:`.
  **L109 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_kernels_loop:`。
- **L110 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_parallel_loop:`.
  **L110 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_parallel_loop:`。
- **L111 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_serial_loop:`.
  **L111 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_serial_loop:`。
- **L112 EN**: Returns from the current function with `void`.
  **L112 CN**: 以 `void` 从当前函数返回。
- **L113 EN**: Introduces a switch dispatch label: `default:`.
  **L113 CN**: 引入一个 switch 分发标签：`default:`。
- **L114 EN**: Exits the nearest loop or switch statement.
  **L114 CN**: 退出最近的循环或 switch 语句。
- **L115 EN**: Closes the current lexical scope or compound statement.
  **L115 CN**: 结束当前词法作用域或复合语句块。
- **L116 EN**: Closes the current lexical scope or compound statement.
  **L116 CN**: 结束当前词法作用域或复合语句块。
- **L117 EN**: Executes a call or declaration centered on `CheckConstructNameBranching`.
  **L117 CN**: 执行以 `CheckConstructNameBranching` 为核心的调用或声明。
- **L118 EN**: Closes the current lexical scope or compound statement.
  **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Closes the current lexical scope or compound statement.
  **L119 CN**: 结束当前词法作用域或复合语句块。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 121-140

````cpp
private:
  parser::MessageFormattedText GetEnclosingMsg() const {
    return {"Enclosing %s construct"_en_US, upperCaseDirName_};
  }

  void EmitBranchOutError(const char *stmt) const {
    context_
        .Say(currentStatementSourcePosition_,
            "%s statement is not allowed in a %s construct"_err_en_US, stmt,
            upperCaseDirName_)
        .Attach(sourcePosition_, GetEnclosingMsg());
  }

  void EmitBranchOutOfComputeConstructError(const char *stmt) const {
    context_
        .Say(currentStatementSourcePosition_,
            "%s to a label outside of a %s construct is not allowed"_err_en_US,
            stmt, upperCaseDirName_)
        .Attach(sourcePosition_, GetEnclosingMsg());
  }
````
- **L121 EN**: Sets the following members to `private` access.
  **L121 CN**: 将后续成员的访问级别设为 `private`。
- **L122 EN**: Starts a function, method, lambda, or structured scope: `parser::MessageFormattedText GetEnclosingMsg() const {`.
  **L122 CN**: 开始一个函数、方法、lambda 或结构化作用域：`parser::MessageFormattedText GetEnclosingMsg() const {`。
- **L123 EN**: Returns from the current function with `{"Enclosing %s construct"_en_US, upperCaseDirName_}`.
  **L123 CN**: 以 `{"Enclosing %s construct"_en_US, upperCaseDirName_}` 从当前函数返回。
- **L124 EN**: Closes the current lexical scope or compound statement.
  **L124 CN**: 结束当前词法作用域或复合语句块。
- **L125 EN**: Blank line separating nearby declarations or logic blocks.
  **L125 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L126 EN**: Starts a function, method, lambda, or structured scope: `void EmitBranchOutError(const char *stmt) const {`.
  **L126 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void EmitBranchOutError(const char *stmt) const {`。
- **L127 EN**: Continues the surrounding expression or declaration: `context_`.
  **L127 CN**: 继续构造周围的表达式或声明：`context_`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(currentStatementSourcePosition_,`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(currentStatementSourcePosition_,`。
- **L129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s statement is not allowed in a %s construct"_err_en_US, stmt,`.
  **L129 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s statement is not allowed in a %s construct"_err_en_US, stmt,`。
- **L130 EN**: Continues the surrounding expression or declaration: `upperCaseDirName_)`.
  **L130 CN**: 继续构造周围的表达式或声明：`upperCaseDirName_)`。
- **L131 EN**: Executes a call or declaration centered on `.Attach`.
  **L131 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L132 EN**: Closes the current lexical scope or compound statement.
  **L132 CN**: 结束当前词法作用域或复合语句块。
- **L133 EN**: Blank line separating nearby declarations or logic blocks.
  **L133 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L134 EN**: Starts a function, method, lambda, or structured scope: `void EmitBranchOutOfComputeConstructError(const char *stmt) const {`.
  **L134 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void EmitBranchOutOfComputeConstructError(const char *stmt) const {`。
- **L135 EN**: Continues the surrounding expression or declaration: `context_`.
  **L135 CN**: 继续构造周围的表达式或声明：`context_`。
- **L136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(currentStatementSourcePosition_,`.
  **L136 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(currentStatementSourcePosition_,`。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s to a label outside of a %s construct is not allowed"_err_en_US,`.
  **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s to a label outside of a %s construct is not allowed"_err_en_US,`。
- **L138 EN**: Continues the surrounding expression or declaration: `stmt, upperCaseDirName_)`.
  **L138 CN**: 继续构造周围的表达式或声明：`stmt, upperCaseDirName_)`。
- **L139 EN**: Executes a call or declaration centered on `.Attach`.
  **L139 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L140 EN**: Closes the current lexical scope or compound statement.
  **L140 CN**: 结束当前词法作用域或复合语句块。

### Lines 141-160

````cpp

  inline void EmitUnlabelledBranchOutError(const char *stmt) {
    context_
        .Say(currentStatementSourcePosition_,
            "%s to construct outside of %s construct is not allowed"_err_en_US,
            stmt, upperCaseDirName_)
        .Attach(sourcePosition_, GetEnclosingMsg());
  }

  void EmitBranchOutErrorWithName(
      const char *stmt, const parser::Name &toName) const {
    const std::string branchingToName{toName.ToString()};
    context_
        .Say(currentStatementSourcePosition_,
            "%s to construct '%s' outside of %s construct is not allowed"_err_en_US,
            stmt, branchingToName, upperCaseDirName_)
        .Attach(sourcePosition_, GetEnclosingMsg());
  }

  // Current semantic checker is not following OpenACC/OpenMP constructs as they
````
- **L141 EN**: Blank line separating nearby declarations or logic blocks.
  **L141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L142 EN**: Starts a function, method, lambda, or structured scope: `inline void EmitUnlabelledBranchOutError(const char *stmt) {`.
  **L142 CN**: 开始一个函数、方法、lambda 或结构化作用域：`inline void EmitUnlabelledBranchOutError(const char *stmt) {`。
- **L143 EN**: Continues the surrounding expression or declaration: `context_`.
  **L143 CN**: 继续构造周围的表达式或声明：`context_`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(currentStatementSourcePosition_,`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(currentStatementSourcePosition_,`。
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s to construct outside of %s construct is not allowed"_err_en_US,`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s to construct outside of %s construct is not allowed"_err_en_US,`。
- **L146 EN**: Continues the surrounding expression or declaration: `stmt, upperCaseDirName_)`.
  **L146 CN**: 继续构造周围的表达式或声明：`stmt, upperCaseDirName_)`。
- **L147 EN**: Executes a call or declaration centered on `.Attach`.
  **L147 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L148 EN**: Closes the current lexical scope or compound statement.
  **L148 CN**: 结束当前词法作用域或复合语句块。
- **L149 EN**: Blank line separating nearby declarations or logic blocks.
  **L149 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L150 EN**: Continues logic associated with callable symbol `EmitBranchOutErrorWithName`.
  **L150 CN**: 继续与可调用符号 `EmitBranchOutErrorWithName` 相关的逻辑。
- **L151 EN**: Continues the surrounding expression or declaration: `const char *stmt, const parser::Name &toName) const {`.
  **L151 CN**: 继续构造周围的表达式或声明：`const char *stmt, const parser::Name &toName) const {`。
- **L152 EN**: Executes a call or declaration centered on `branchingToName{toName.ToString`.
  **L152 CN**: 执行以 `branchingToName{toName.ToString` 为核心的调用或声明。
- **L153 EN**: Continues the surrounding expression or declaration: `context_`.
  **L153 CN**: 继续构造周围的表达式或声明：`context_`。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(currentStatementSourcePosition_,`.
  **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(currentStatementSourcePosition_,`。
- **L155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s to construct '%s' outside of %s construct is not allowed"_err_en_US,`.
  **L155 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s to construct '%s' outside of %s construct is not allowed"_err_en_US,`。
- **L156 EN**: Continues the surrounding expression or declaration: `stmt, branchingToName, upperCaseDirName_)`.
  **L156 CN**: 继续构造周围的表达式或声明：`stmt, branchingToName, upperCaseDirName_)`。
- **L157 EN**: Executes a call or declaration centered on `.Attach`.
  **L157 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L158 EN**: Closes the current lexical scope or compound statement.
  **L158 CN**: 结束当前词法作用域或复合语句块。
- **L159 EN**: Blank line separating nearby declarations or logic blocks.
  **L159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L160 EN**: Comment explains nearby logic, intent, or metadata: `Current semantic checker is not following OpenACC/OpenMP constructs as they`.
  **L160 CN**: 注释说明附近代码的逻辑、意图或元数据：`Current semantic checker is not following OpenACC/OpenMP constructs as they`。

### Lines 161-180

````cpp
  // are not Fortran constructs. Hence the ConstructStack doesn't capture
  // OpenACC/OpenMP constructs. Apply an inverse way to figure out if a
  // construct-name is branching out of an OpenACC/OpenMP construct. The control
  // flow goes out of an OpenACC/OpenMP construct, if a construct-name from
  // statement is found in ConstructStack.
  void CheckConstructNameBranching(
      const char *stmt, const parser::Name &stmtName) {
    const ConstructStack &stack{context_.constructStack()};
    for (auto iter{stack.cend()}; iter-- != stack.cbegin();) {
      const ConstructNode &construct{*iter};
      const auto &constructName{MaybeGetNodeName(construct)};
      if (constructName) {
        if (stmtName.source == constructName->source) {
          EmitBranchOutErrorWithName(stmt, stmtName);
          return;
        }
      }
    }
  }

````
- **L161 EN**: Comment explains nearby logic, intent, or metadata: `are not Fortran constructs. Hence the ConstructStack doesn't capture`.
  **L161 CN**: 注释说明附近代码的逻辑、意图或元数据：`are not Fortran constructs. Hence the ConstructStack doesn't capture`。
- **L162 EN**: Comment explains nearby logic, intent, or metadata: `OpenACC/OpenMP constructs. Apply an inverse way to figure out if a`.
  **L162 CN**: 注释说明附近代码的逻辑、意图或元数据：`OpenACC/OpenMP constructs. Apply an inverse way to figure out if a`。
- **L163 EN**: Comment explains nearby logic, intent, or metadata: `construct-name is branching out of an OpenACC/OpenMP construct. The control`.
  **L163 CN**: 注释说明附近代码的逻辑、意图或元数据：`construct-name is branching out of an OpenACC/OpenMP construct. The control`。
- **L164 EN**: Comment explains nearby logic, intent, or metadata: `flow goes out of an OpenACC/OpenMP construct, if a construct-name from`.
  **L164 CN**: 注释说明附近代码的逻辑、意图或元数据：`flow goes out of an OpenACC/OpenMP construct, if a construct-name from`。
- **L165 EN**: Comment explains nearby logic, intent, or metadata: `statement is found in ConstructStack.`.
  **L165 CN**: 注释说明附近代码的逻辑、意图或元数据：`statement is found in ConstructStack.`。
- **L166 EN**: Continues logic associated with callable symbol `CheckConstructNameBranching`.
  **L166 CN**: 继续与可调用符号 `CheckConstructNameBranching` 相关的逻辑。
- **L167 EN**: Continues the surrounding expression or declaration: `const char *stmt, const parser::Name &stmtName) {`.
  **L167 CN**: 继续构造周围的表达式或声明：`const char *stmt, const parser::Name &stmtName) {`。
- **L168 EN**: Executes a call or declaration centered on `&stack{context_.constructStack`.
  **L168 CN**: 执行以 `&stack{context_.constructStack` 为核心的调用或声明。
- **L169 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L169 CN**: 开始 `for` 控制流语句并计算其条件。
- **L170 EN**: Executes a standalone statement or declaration: `const ConstructNode &construct{*iter};`.
  **L170 CN**: 执行一条独立语句或声明：`const ConstructNode &construct{*iter};`。
- **L171 EN**: Executes a call or declaration centered on `&constructName{MaybeGetNodeName`.
  **L171 CN**: 执行以 `&constructName{MaybeGetNodeName` 为核心的调用或声明。
- **L172 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L172 CN**: 开始 `if` 控制流语句并计算其条件。
- **L173 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L173 CN**: 开始 `if` 控制流语句并计算其条件。
- **L174 EN**: Executes a call or declaration centered on `EmitBranchOutErrorWithName`.
  **L174 CN**: 执行以 `EmitBranchOutErrorWithName` 为核心的调用或声明。
- **L175 EN**: Returns from the current function with `void`.
  **L175 CN**: 以 `void` 从当前函数返回。
- **L176 EN**: Closes the current lexical scope or compound statement.
  **L176 CN**: 结束当前词法作用域或复合语句块。
- **L177 EN**: Closes the current lexical scope or compound statement.
  **L177 CN**: 结束当前词法作用域或复合语句块。
- **L178 EN**: Closes the current lexical scope or compound statement.
  **L178 CN**: 结束当前词法作用域或复合语句块。
- **L179 EN**: Closes the current lexical scope or compound statement.
  **L179 CN**: 结束当前词法作用域或复合语句块。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 181-200

````cpp
  // Check branching for unlabelled CYCLES and EXITs
  void CheckConstructNameBranching(const char *stmt) {
    // found an enclosing looping construct for the unlabelled EXIT/CYCLE
    if (numDoConstruct_ > 0) {
      return;
    }
    // did not found an enclosing looping construct within the OpenMP/OpenACC
    // directive
    EmitUnlabelledBranchOutError(stmt);
  }

  SemanticsContext &context_;
  parser::CharBlock currentStatementSourcePosition_;
  parser::CharBlock sourcePosition_;
  std::string upperCaseDirName_;
  D currentDirective_;
  int numDoConstruct_; // tracks number of DoConstruct found AFTER encountering
                       // an OpenMP/OpenACC directive
  std::set<parser::Label> labelsInBlock_;
};
````
- **L181 EN**: Comment explains nearby logic, intent, or metadata: `Check branching for unlabelled CYCLES and EXITs`.
  **L181 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check branching for unlabelled CYCLES and EXITs`。
- **L182 EN**: Starts a function, method, lambda, or structured scope: `void CheckConstructNameBranching(const char *stmt) {`.
  **L182 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckConstructNameBranching(const char *stmt) {`。
- **L183 EN**: Comment explains nearby logic, intent, or metadata: `found an enclosing looping construct for the unlabelled EXIT/CYCLE`.
  **L183 CN**: 注释说明附近代码的逻辑、意图或元数据：`found an enclosing looping construct for the unlabelled EXIT/CYCLE`。
- **L184 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L184 CN**: 开始 `if` 控制流语句并计算其条件。
- **L185 EN**: Returns from the current function with `void`.
  **L185 CN**: 以 `void` 从当前函数返回。
- **L186 EN**: Closes the current lexical scope or compound statement.
  **L186 CN**: 结束当前词法作用域或复合语句块。
- **L187 EN**: Comment explains nearby logic, intent, or metadata: `did not found an enclosing looping construct within the OpenMP/OpenACC`.
  **L187 CN**: 注释说明附近代码的逻辑、意图或元数据：`did not found an enclosing looping construct within the OpenMP/OpenACC`。
- **L188 EN**: Comment explains nearby logic, intent, or metadata: `directive`.
  **L188 CN**: 注释说明附近代码的逻辑、意图或元数据：`directive`。
- **L189 EN**: Executes a call or declaration centered on `EmitUnlabelledBranchOutError`.
  **L189 CN**: 执行以 `EmitUnlabelledBranchOutError` 为核心的调用或声明。
- **L190 EN**: Closes the current lexical scope or compound statement.
  **L190 CN**: 结束当前词法作用域或复合语句块。
- **L191 EN**: Blank line separating nearby declarations or logic blocks.
  **L191 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L192 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L192 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。
- **L193 EN**: Executes a standalone statement or declaration: `parser::CharBlock currentStatementSourcePosition_;`.
  **L193 CN**: 执行一条独立语句或声明：`parser::CharBlock currentStatementSourcePosition_;`。
- **L194 EN**: Executes a standalone statement or declaration: `parser::CharBlock sourcePosition_;`.
  **L194 CN**: 执行一条独立语句或声明：`parser::CharBlock sourcePosition_;`。
- **L195 EN**: Executes a standalone statement or declaration: `std::string upperCaseDirName_;`.
  **L195 CN**: 执行一条独立语句或声明：`std::string upperCaseDirName_;`。
- **L196 EN**: Executes a standalone statement or declaration: `D currentDirective_;`.
  **L196 CN**: 执行一条独立语句或声明：`D currentDirective_;`。
- **L197 EN**: Continues the surrounding expression or declaration: `int numDoConstruct_; // tracks number of DoConstruct found AFTER encountering`.
  **L197 CN**: 继续构造周围的表达式或声明：`int numDoConstruct_; // tracks number of DoConstruct found AFTER encountering`。
- **L198 EN**: Comment explains nearby logic, intent, or metadata: `an OpenMP/OpenACC directive`.
  **L198 CN**: 注释说明附近代码的逻辑、意图或元数据：`an OpenMP/OpenACC directive`。
- **L199 EN**: Executes a standalone statement or declaration: `std::set<parser::Label> labelsInBlock_;`.
  **L199 CN**: 执行一条独立语句或声明：`std::set<parser::Label> labelsInBlock_;`。
- **L200 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L200 CN**: 结束当前声明作用域，例如类、结构体或枚举。

### Lines 201-220

````cpp

// Generic structure checker for directives/clauses language such as OpenMP
// and OpenACC.
// typename D is the directive enumeration.
// typename C is the clause enumeration.
// typename PC is the parser class defined in parse-tree.h for the clauses.
template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
class DirectiveStructureChecker : public virtual BaseChecker {
protected:
  DirectiveStructureChecker(SemanticsContext &context,
      const std::unordered_map<D, DirectiveClauses<C, ClauseEnumSize>>
          &directiveClausesMap)
      : context_{context}, directiveClausesMap_(directiveClausesMap) {}
  virtual ~DirectiveStructureChecker() {}

  using ClauseMapTy = std::multimap<C, const PC *>;
  struct DirectiveContext {
    DirectiveContext(parser::CharBlock source, D d)
        : directiveSource{source}, directive{d} {}

````
- **L201 EN**: Blank line separating nearby declarations or logic blocks.
  **L201 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L202 EN**: Comment explains nearby logic, intent, or metadata: `Generic structure checker for directives/clauses language such as OpenMP`.
  **L202 CN**: 注释说明附近代码的逻辑、意图或元数据：`Generic structure checker for directives/clauses language such as OpenMP`。
- **L203 EN**: Comment explains nearby logic, intent, or metadata: `and OpenACC.`.
  **L203 CN**: 注释说明附近代码的逻辑、意图或元数据：`and OpenACC.`。
- **L204 EN**: Comment explains nearby logic, intent, or metadata: `typename D is the directive enumeration.`.
  **L204 CN**: 注释说明附近代码的逻辑、意图或元数据：`typename D is the directive enumeration.`。
- **L205 EN**: Comment explains nearby logic, intent, or metadata: `typename C is the clause enumeration.`.
  **L205 CN**: 注释说明附近代码的逻辑、意图或元数据：`typename C is the clause enumeration.`。
- **L206 EN**: Comment explains nearby logic, intent, or metadata: `typename PC is the parser class defined in parse-tree.h for the clauses.`.
  **L206 CN**: 注释说明附近代码的逻辑、意图或元数据：`typename PC is the parser class defined in parse-tree.h for the clauses.`。
- **L207 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L207 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L208 EN**: Declares class `DirectiveStructureChecker`.
  **L208 CN**: 声明 class `DirectiveStructureChecker`。
- **L209 EN**: Sets the following members to `protected` access.
  **L209 CN**: 将后续成员的访问级别设为 `protected`。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DirectiveStructureChecker(SemanticsContext &context,`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`DirectiveStructureChecker(SemanticsContext &context,`。
- **L211 EN**: Continues the surrounding expression or declaration: `const std::unordered_map<D, DirectiveClauses<C, ClauseEnumSize>>`.
  **L211 CN**: 继续构造周围的表达式或声明：`const std::unordered_map<D, DirectiveClauses<C, ClauseEnumSize>>`。
- **L212 EN**: Continues the surrounding expression or declaration: `&directiveClausesMap)`.
  **L212 CN**: 继续构造周围的表达式或声明：`&directiveClausesMap)`。
- **L213 EN**: Continues logic associated with callable symbol `directiveClausesMap_`.
  **L213 CN**: 继续与可调用符号 `directiveClausesMap_` 相关的逻辑。
- **L214 EN**: Continues logic associated with callable symbol `~DirectiveStructureChecker`.
  **L214 CN**: 继续与可调用符号 `~DirectiveStructureChecker` 相关的逻辑。
- **L215 EN**: Blank line separating nearby declarations or logic blocks.
  **L215 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L216 EN**: Defines alias `ClauseMapTy` to simplify later code.
  **L216 CN**: 定义别名 `ClauseMapTy` 以简化后续代码。
- **L217 EN**: Declares struct `DirectiveContext`.
  **L217 CN**: 声明 struct `DirectiveContext`。
- **L218 EN**: Continues logic associated with callable symbol `DirectiveContext`.
  **L218 CN**: 继续与可调用符号 `DirectiveContext` 相关的逻辑。
- **L219 EN**: Continues the surrounding expression or declaration: `: directiveSource{source}, directive{d} {}`.
  **L219 CN**: 继续构造周围的表达式或声明：`: directiveSource{source}, directive{d} {}`。
- **L220 EN**: Blank line separating nearby declarations or logic blocks.
  **L220 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 221-240

````cpp
    parser::CharBlock directiveSource{nullptr};
    parser::CharBlock clauseSource{nullptr};
    D directive;
    common::EnumSet<C, ClauseEnumSize> allowedClauses{};
    common::EnumSet<C, ClauseEnumSize> allowedOnceClauses{};
    common::EnumSet<C, ClauseEnumSize> allowedExclusiveClauses{};
    common::EnumSet<C, ClauseEnumSize> requiredClauses{};

    const PC *clause{nullptr};
    ClauseMapTy clauseInfo;
    std::list<C> actualClauses;
    std::list<C> endDirectiveClauses;
    std::list<C> crtGroup;
  };

  // back() is the top of the stack
  DirectiveContext &GetContext() {
    CHECK(!dirContext_.empty());
    return dirContext_.back();
  }
````
- **L221 EN**: Executes a standalone statement or declaration: `parser::CharBlock directiveSource{nullptr};`.
  **L221 CN**: 执行一条独立语句或声明：`parser::CharBlock directiveSource{nullptr};`。
- **L222 EN**: Executes a standalone statement or declaration: `parser::CharBlock clauseSource{nullptr};`.
  **L222 CN**: 执行一条独立语句或声明：`parser::CharBlock clauseSource{nullptr};`。
- **L223 EN**: Executes a standalone statement or declaration: `D directive;`.
  **L223 CN**: 执行一条独立语句或声明：`D directive;`。
- **L224 EN**: Executes a standalone statement or declaration: `common::EnumSet<C, ClauseEnumSize> allowedClauses{};`.
  **L224 CN**: 执行一条独立语句或声明：`common::EnumSet<C, ClauseEnumSize> allowedClauses{};`。
- **L225 EN**: Executes a standalone statement or declaration: `common::EnumSet<C, ClauseEnumSize> allowedOnceClauses{};`.
  **L225 CN**: 执行一条独立语句或声明：`common::EnumSet<C, ClauseEnumSize> allowedOnceClauses{};`。
- **L226 EN**: Executes a standalone statement or declaration: `common::EnumSet<C, ClauseEnumSize> allowedExclusiveClauses{};`.
  **L226 CN**: 执行一条独立语句或声明：`common::EnumSet<C, ClauseEnumSize> allowedExclusiveClauses{};`。
- **L227 EN**: Executes a standalone statement or declaration: `common::EnumSet<C, ClauseEnumSize> requiredClauses{};`.
  **L227 CN**: 执行一条独立语句或声明：`common::EnumSet<C, ClauseEnumSize> requiredClauses{};`。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L229 EN**: Executes a standalone statement or declaration: `const PC *clause{nullptr};`.
  **L229 CN**: 执行一条独立语句或声明：`const PC *clause{nullptr};`。
- **L230 EN**: Executes a standalone statement or declaration: `ClauseMapTy clauseInfo;`.
  **L230 CN**: 执行一条独立语句或声明：`ClauseMapTy clauseInfo;`。
- **L231 EN**: Executes a standalone statement or declaration: `std::list<C> actualClauses;`.
  **L231 CN**: 执行一条独立语句或声明：`std::list<C> actualClauses;`。
- **L232 EN**: Executes a standalone statement or declaration: `std::list<C> endDirectiveClauses;`.
  **L232 CN**: 执行一条独立语句或声明：`std::list<C> endDirectiveClauses;`。
- **L233 EN**: Executes a standalone statement or declaration: `std::list<C> crtGroup;`.
  **L233 CN**: 执行一条独立语句或声明：`std::list<C> crtGroup;`。
- **L234 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L234 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L235 EN**: Blank line separating nearby declarations or logic blocks.
  **L235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L236 EN**: Comment explains nearby logic, intent, or metadata: `back() is the top of the stack`.
  **L236 CN**: 注释说明附近代码的逻辑、意图或元数据：`back() is the top of the stack`。
- **L237 EN**: Starts a function, method, lambda, or structured scope: `DirectiveContext &GetContext() {`.
  **L237 CN**: 开始一个函数、方法、lambda 或结构化作用域：`DirectiveContext &GetContext() {`。
- **L238 EN**: Executes a call or declaration centered on `CHECK`.
  **L238 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L239 EN**: Returns from the current function with `dirContext_.back()`.
  **L239 CN**: 以 `dirContext_.back()` 从当前函数返回。
- **L240 EN**: Closes the current lexical scope or compound statement.
  **L240 CN**: 结束当前词法作用域或复合语句块。

### Lines 241-260

````cpp

  DirectiveContext &GetContextParent() {
    CHECK(dirContext_.size() >= 2);
    return dirContext_[dirContext_.size() - 2];
  }

  void SetContextClause(const PC &clause) {
    GetContext().clauseSource = clause.source;
    GetContext().clause = &clause;
  }

  void ResetPartialContext(const parser::CharBlock &source) {
    CHECK(!dirContext_.empty());
    SetContextDirectiveSource(source);
    GetContext().allowedClauses = {};
    GetContext().allowedOnceClauses = {};
    GetContext().allowedExclusiveClauses = {};
    GetContext().requiredClauses = {};
    GetContext().clauseInfo = {};
  }
````
- **L241 EN**: Blank line separating nearby declarations or logic blocks.
  **L241 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L242 EN**: Starts a function, method, lambda, or structured scope: `DirectiveContext &GetContextParent() {`.
  **L242 CN**: 开始一个函数、方法、lambda 或结构化作用域：`DirectiveContext &GetContextParent() {`。
- **L243 EN**: Executes a call or declaration centered on `CHECK`.
  **L243 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L244 EN**: Returns from the current function with `dirContext_[dirContext_.size() - 2]`.
  **L244 CN**: 以 `dirContext_[dirContext_.size() - 2]` 从当前函数返回。
- **L245 EN**: Closes the current lexical scope or compound statement.
  **L245 CN**: 结束当前词法作用域或复合语句块。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L247 EN**: Starts a function, method, lambda, or structured scope: `void SetContextClause(const PC &clause) {`.
  **L247 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void SetContextClause(const PC &clause) {`。
- **L248 EN**: Executes a call or declaration centered on `GetContext`.
  **L248 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L249 EN**: Executes a call or declaration centered on `GetContext`.
  **L249 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L250 EN**: Closes the current lexical scope or compound statement.
  **L250 CN**: 结束当前词法作用域或复合语句块。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L252 EN**: Starts a function, method, lambda, or structured scope: `void ResetPartialContext(const parser::CharBlock &source) {`.
  **L252 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void ResetPartialContext(const parser::CharBlock &source) {`。
- **L253 EN**: Executes a call or declaration centered on `CHECK`.
  **L253 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L254 EN**: Executes a call or declaration centered on `SetContextDirectiveSource`.
  **L254 CN**: 执行以 `SetContextDirectiveSource` 为核心的调用或声明。
- **L255 EN**: Executes a call or declaration centered on `GetContext`.
  **L255 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L256 EN**: Executes a call or declaration centered on `GetContext`.
  **L256 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L257 EN**: Executes a call or declaration centered on `GetContext`.
  **L257 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L258 EN**: Executes a call or declaration centered on `GetContext`.
  **L258 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L259 EN**: Executes a call or declaration centered on `GetContext`.
  **L259 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L260 EN**: Closes the current lexical scope or compound statement.
  **L260 CN**: 结束当前词法作用域或复合语句块。

### Lines 261-280

````cpp

  void SetContextDirectiveSource(const parser::CharBlock &directive) {
    GetContext().directiveSource = directive;
  }

  void SetContextDirectiveEnum(D dir) { GetContext().directive = dir; }

  void SetContextAllowed(const common::EnumSet<C, ClauseEnumSize> &allowed) {
    GetContext().allowedClauses = allowed;
  }

  void SetContextAllowedOnce(
      const common::EnumSet<C, ClauseEnumSize> &allowedOnce) {
    GetContext().allowedOnceClauses = allowedOnce;
  }

  void SetContextAllowedExclusive(
      const common::EnumSet<C, ClauseEnumSize> &allowedExclusive) {
    GetContext().allowedExclusiveClauses = allowedExclusive;
  }
````
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L262 EN**: Starts a function, method, lambda, or structured scope: `void SetContextDirectiveSource(const parser::CharBlock &directive) {`.
  **L262 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void SetContextDirectiveSource(const parser::CharBlock &directive) {`。
- **L263 EN**: Executes a call or declaration centered on `GetContext`.
  **L263 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L264 EN**: Closes the current lexical scope or compound statement.
  **L264 CN**: 结束当前词法作用域或复合语句块。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L266 EN**: Continues logic associated with callable symbol `SetContextDirectiveEnum`.
  **L266 CN**: 继续与可调用符号 `SetContextDirectiveEnum` 相关的逻辑。
- **L267 EN**: Blank line separating nearby declarations or logic blocks.
  **L267 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L268 EN**: Starts a function, method, lambda, or structured scope: `void SetContextAllowed(const common::EnumSet<C, ClauseEnumSize> &allowed) {`.
  **L268 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void SetContextAllowed(const common::EnumSet<C, ClauseEnumSize> &allowed) {`。
- **L269 EN**: Executes a call or declaration centered on `GetContext`.
  **L269 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L270 EN**: Closes the current lexical scope or compound statement.
  **L270 CN**: 结束当前词法作用域或复合语句块。
- **L271 EN**: Blank line separating nearby declarations or logic blocks.
  **L271 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L272 EN**: Continues logic associated with callable symbol `SetContextAllowedOnce`.
  **L272 CN**: 继续与可调用符号 `SetContextAllowedOnce` 相关的逻辑。
- **L273 EN**: Continues the surrounding expression or declaration: `const common::EnumSet<C, ClauseEnumSize> &allowedOnce) {`.
  **L273 CN**: 继续构造周围的表达式或声明：`const common::EnumSet<C, ClauseEnumSize> &allowedOnce) {`。
- **L274 EN**: Executes a call or declaration centered on `GetContext`.
  **L274 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L275 EN**: Closes the current lexical scope or compound statement.
  **L275 CN**: 结束当前词法作用域或复合语句块。
- **L276 EN**: Blank line separating nearby declarations or logic blocks.
  **L276 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L277 EN**: Continues logic associated with callable symbol `SetContextAllowedExclusive`.
  **L277 CN**: 继续与可调用符号 `SetContextAllowedExclusive` 相关的逻辑。
- **L278 EN**: Continues the surrounding expression or declaration: `const common::EnumSet<C, ClauseEnumSize> &allowedExclusive) {`.
  **L278 CN**: 继续构造周围的表达式或声明：`const common::EnumSet<C, ClauseEnumSize> &allowedExclusive) {`。
- **L279 EN**: Executes a call or declaration centered on `GetContext`.
  **L279 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L280 EN**: Closes the current lexical scope or compound statement.
  **L280 CN**: 结束当前词法作用域或复合语句块。

### Lines 281-300

````cpp

  void SetContextRequired(const common::EnumSet<C, ClauseEnumSize> &required) {
    GetContext().requiredClauses = required;
  }

  void SetContextClauseInfo(C type) {
    GetContext().clauseInfo.emplace(type, GetContext().clause);
  }

  void AddClauseToCrtContext(C type) {
    GetContext().actualClauses.push_back(type);
  }

  void AddClauseToCrtGroupInContext(C type) {
    GetContext().crtGroup.push_back(type);
  }

  void ResetCrtGroup() { GetContext().crtGroup.clear(); }

  // Check if the given clause is present in the current context
````
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L282 EN**: Starts a function, method, lambda, or structured scope: `void SetContextRequired(const common::EnumSet<C, ClauseEnumSize> &required) {`.
  **L282 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void SetContextRequired(const common::EnumSet<C, ClauseEnumSize> &required) {`。
- **L283 EN**: Executes a call or declaration centered on `GetContext`.
  **L283 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L284 EN**: Closes the current lexical scope or compound statement.
  **L284 CN**: 结束当前词法作用域或复合语句块。
- **L285 EN**: Blank line separating nearby declarations or logic blocks.
  **L285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L286 EN**: Starts a function, method, lambda, or structured scope: `void SetContextClauseInfo(C type) {`.
  **L286 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void SetContextClauseInfo(C type) {`。
- **L287 EN**: Executes a call or declaration centered on `GetContext`.
  **L287 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L288 EN**: Closes the current lexical scope or compound statement.
  **L288 CN**: 结束当前词法作用域或复合语句块。
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L290 EN**: Starts a function, method, lambda, or structured scope: `void AddClauseToCrtContext(C type) {`.
  **L290 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AddClauseToCrtContext(C type) {`。
- **L291 EN**: Executes a call or declaration centered on `GetContext`.
  **L291 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L292 EN**: Closes the current lexical scope or compound statement.
  **L292 CN**: 结束当前词法作用域或复合语句块。
- **L293 EN**: Blank line separating nearby declarations or logic blocks.
  **L293 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L294 EN**: Starts a function, method, lambda, or structured scope: `void AddClauseToCrtGroupInContext(C type) {`.
  **L294 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AddClauseToCrtGroupInContext(C type) {`。
- **L295 EN**: Executes a call or declaration centered on `GetContext`.
  **L295 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L296 EN**: Closes the current lexical scope or compound statement.
  **L296 CN**: 结束当前词法作用域或复合语句块。
- **L297 EN**: Blank line separating nearby declarations or logic blocks.
  **L297 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L298 EN**: Continues logic associated with callable symbol `ResetCrtGroup`.
  **L298 CN**: 继续与可调用符号 `ResetCrtGroup` 相关的逻辑。
- **L299 EN**: Blank line separating nearby declarations or logic blocks.
  **L299 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L300 EN**: Comment explains nearby logic, intent, or metadata: `Check if the given clause is present in the current context`.
  **L300 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check if the given clause is present in the current context`。

### Lines 301-320

````cpp
  const PC *FindClause(C type) { return FindClause(GetContext(), type); }

  // Check if the given clause is present in the given context
  const PC *FindClause(DirectiveContext &context, C type) {
    auto it{context.clauseInfo.find(type)};
    if (it != context.clauseInfo.end()) {
      return it->second;
    }
    return nullptr;
  }

  // Check if the given clause is present in the parent context
  const PC *FindClauseParent(C type) {
    auto it{GetContextParent().clauseInfo.find(type)};
    if (it != GetContextParent().clauseInfo.end()) {
      return it->second;
    }
    return nullptr;
  }

````
- **L301 EN**: Continues logic associated with callable symbol `FindClause`.
  **L301 CN**: 继续与可调用符号 `FindClause` 相关的逻辑。
- **L302 EN**: Blank line separating nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L303 EN**: Comment explains nearby logic, intent, or metadata: `Check if the given clause is present in the given context`.
  **L303 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check if the given clause is present in the given context`。
- **L304 EN**: Starts a function, method, lambda, or structured scope: `const PC *FindClause(DirectiveContext &context, C type) {`.
  **L304 CN**: 开始一个函数、方法、lambda 或结构化作用域：`const PC *FindClause(DirectiveContext &context, C type) {`。
- **L305 EN**: Executes a call or declaration centered on `it{context.clauseInfo.find`.
  **L305 CN**: 执行以 `it{context.clauseInfo.find` 为核心的调用或声明。
- **L306 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L306 CN**: 开始 `if` 控制流语句并计算其条件。
- **L307 EN**: Returns from the current function with `it->second`.
  **L307 CN**: 以 `it->second` 从当前函数返回。
- **L308 EN**: Closes the current lexical scope or compound statement.
  **L308 CN**: 结束当前词法作用域或复合语句块。
- **L309 EN**: Returns from the current function with `nullptr`.
  **L309 CN**: 以 `nullptr` 从当前函数返回。
- **L310 EN**: Closes the current lexical scope or compound statement.
  **L310 CN**: 结束当前词法作用域或复合语句块。
- **L311 EN**: Blank line separating nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L312 EN**: Comment explains nearby logic, intent, or metadata: `Check if the given clause is present in the parent context`.
  **L312 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check if the given clause is present in the parent context`。
- **L313 EN**: Starts a function, method, lambda, or structured scope: `const PC *FindClauseParent(C type) {`.
  **L313 CN**: 开始一个函数、方法、lambda 或结构化作用域：`const PC *FindClauseParent(C type) {`。
- **L314 EN**: Executes a call or declaration centered on `it{GetContextParent`.
  **L314 CN**: 执行以 `it{GetContextParent` 为核心的调用或声明。
- **L315 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L315 CN**: 开始 `if` 控制流语句并计算其条件。
- **L316 EN**: Returns from the current function with `it->second`.
  **L316 CN**: 以 `it->second` 从当前函数返回。
- **L317 EN**: Closes the current lexical scope or compound statement.
  **L317 CN**: 结束当前词法作用域或复合语句块。
- **L318 EN**: Returns from the current function with `nullptr`.
  **L318 CN**: 以 `nullptr` 从当前函数返回。
- **L319 EN**: Closes the current lexical scope or compound statement.
  **L319 CN**: 结束当前词法作用域或复合语句块。
- **L320 EN**: Blank line separating nearby declarations or logic blocks.
  **L320 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 321-340

````cpp
  llvm::iterator_range<typename ClauseMapTy::iterator> FindClauses(C type) {
    auto it{GetContext().clauseInfo.equal_range(type)};
    return llvm::make_range(it);
  }

  DirectiveContext *GetEnclosingDirContext() {
    CHECK(!dirContext_.empty());
    auto it{dirContext_.rbegin()};
    if (++it != dirContext_.rend()) {
      return &(*it);
    }
    return nullptr;
  }

  void PushContext(const parser::CharBlock &source, D dir) {
    dirContext_.emplace_back(source, dir);
  }

  DirectiveContext *GetEnclosingContextWithDir(D dir) {
    CHECK(!dirContext_.empty());
````
- **L321 EN**: Starts a function, method, lambda, or structured scope: `llvm::iterator_range<typename ClauseMapTy::iterator> FindClauses(C type) {`.
  **L321 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::iterator_range<typename ClauseMapTy::iterator> FindClauses(C type) {`。
- **L322 EN**: Executes a call or declaration centered on `it{GetContext`.
  **L322 CN**: 执行以 `it{GetContext` 为核心的调用或声明。
- **L323 EN**: Returns from the current function with `llvm::make_range(it)`.
  **L323 CN**: 以 `llvm::make_range(it)` 从当前函数返回。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L326 EN**: Starts a function, method, lambda, or structured scope: `DirectiveContext *GetEnclosingDirContext() {`.
  **L326 CN**: 开始一个函数、方法、lambda 或结构化作用域：`DirectiveContext *GetEnclosingDirContext() {`。
- **L327 EN**: Executes a call or declaration centered on `CHECK`.
  **L327 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L328 EN**: Executes a call or declaration centered on `it{dirContext_.rbegin`.
  **L328 CN**: 执行以 `it{dirContext_.rbegin` 为核心的调用或声明。
- **L329 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L329 CN**: 开始 `if` 控制流语句并计算其条件。
- **L330 EN**: Returns from the current function with `&(*it)`.
  **L330 CN**: 以 `&(*it)` 从当前函数返回。
- **L331 EN**: Closes the current lexical scope or compound statement.
  **L331 CN**: 结束当前词法作用域或复合语句块。
- **L332 EN**: Returns from the current function with `nullptr`.
  **L332 CN**: 以 `nullptr` 从当前函数返回。
- **L333 EN**: Closes the current lexical scope or compound statement.
  **L333 CN**: 结束当前词法作用域或复合语句块。
- **L334 EN**: Blank line separating nearby declarations or logic blocks.
  **L334 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L335 EN**: Starts a function, method, lambda, or structured scope: `void PushContext(const parser::CharBlock &source, D dir) {`.
  **L335 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void PushContext(const parser::CharBlock &source, D dir) {`。
- **L336 EN**: Executes a call or declaration centered on `dirContext_.emplace_back`.
  **L336 CN**: 执行以 `dirContext_.emplace_back` 为核心的调用或声明。
- **L337 EN**: Closes the current lexical scope or compound statement.
  **L337 CN**: 结束当前词法作用域或复合语句块。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L339 EN**: Starts a function, method, lambda, or structured scope: `DirectiveContext *GetEnclosingContextWithDir(D dir) {`.
  **L339 CN**: 开始一个函数、方法、lambda 或结构化作用域：`DirectiveContext *GetEnclosingContextWithDir(D dir) {`。
- **L340 EN**: Executes a call or declaration centered on `CHECK`.
  **L340 CN**: 执行以 `CHECK` 为核心的调用或声明。

### Lines 341-360

````cpp
    auto it{dirContext_.rbegin()};
    while (++it != dirContext_.rend()) {
      if (it->directive == dir) {
        return &(*it);
      }
    }
    return nullptr;
  }

  bool CurrentDirectiveIsNested() { return dirContext_.size() > 1; };

  void SetClauseSets(D dir) {
    dirContext_.back().allowedClauses = directiveClausesMap_[dir].allowed;
    dirContext_.back().allowedOnceClauses =
        directiveClausesMap_[dir].allowedOnce;
    dirContext_.back().allowedExclusiveClauses =
        directiveClausesMap_[dir].allowedExclusive;
    dirContext_.back().requiredClauses =
        directiveClausesMap_[dir].requiredOneOf;
  }
````
- **L341 EN**: Executes a call or declaration centered on `it{dirContext_.rbegin`.
  **L341 CN**: 执行以 `it{dirContext_.rbegin` 为核心的调用或声明。
- **L342 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L342 CN**: 开始 `while` 控制流语句并计算其条件。
- **L343 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L343 CN**: 开始 `if` 控制流语句并计算其条件。
- **L344 EN**: Returns from the current function with `&(*it)`.
  **L344 CN**: 以 `&(*it)` 从当前函数返回。
- **L345 EN**: Closes the current lexical scope or compound statement.
  **L345 CN**: 结束当前词法作用域或复合语句块。
- **L346 EN**: Closes the current lexical scope or compound statement.
  **L346 CN**: 结束当前词法作用域或复合语句块。
- **L347 EN**: Returns from the current function with `nullptr`.
  **L347 CN**: 以 `nullptr` 从当前函数返回。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L350 EN**: Executes a call or declaration centered on `CurrentDirectiveIsNested`.
  **L350 CN**: 执行以 `CurrentDirectiveIsNested` 为核心的调用或声明。
- **L351 EN**: Blank line separating nearby declarations or logic blocks.
  **L351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L352 EN**: Starts a function, method, lambda, or structured scope: `void SetClauseSets(D dir) {`.
  **L352 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void SetClauseSets(D dir) {`。
- **L353 EN**: Executes a call or declaration centered on `dirContext_.back`.
  **L353 CN**: 执行以 `dirContext_.back` 为核心的调用或声明。
- **L354 EN**: Continues logic associated with callable symbol `back`.
  **L354 CN**: 继续与可调用符号 `back` 相关的逻辑。
- **L355 EN**: Executes a standalone statement or declaration: `directiveClausesMap_[dir].allowedOnce;`.
  **L355 CN**: 执行一条独立语句或声明：`directiveClausesMap_[dir].allowedOnce;`。
- **L356 EN**: Continues logic associated with callable symbol `back`.
  **L356 CN**: 继续与可调用符号 `back` 相关的逻辑。
- **L357 EN**: Executes a standalone statement or declaration: `directiveClausesMap_[dir].allowedExclusive;`.
  **L357 CN**: 执行一条独立语句或声明：`directiveClausesMap_[dir].allowedExclusive;`。
- **L358 EN**: Continues logic associated with callable symbol `back`.
  **L358 CN**: 继续与可调用符号 `back` 相关的逻辑。
- **L359 EN**: Executes a standalone statement or declaration: `directiveClausesMap_[dir].requiredOneOf;`.
  **L359 CN**: 执行一条独立语句或声明：`directiveClausesMap_[dir].requiredOneOf;`。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-380

````cpp
  void PushContextAndClauseSets(const parser::CharBlock &source, D dir) {
    PushContext(source, dir);
    SetClauseSets(dir);
  }

  void SayNotMatching(const parser::CharBlock &, const parser::CharBlock &);

  template <typename B> void CheckMatching(const B &beginDir, const B &endDir) {
    const auto &begin{beginDir.v};
    const auto &end{endDir.v};
    if (begin != end) {
      SayNotMatching(beginDir.source, endDir.source);
    }
  }
  // Check illegal branching out of `Parser::Block` for `Parser::Name` based
  // nodes (example `Parser::ExitStmt`)
  void CheckNoBranching(const parser::Block &block, D directive,
      const parser::CharBlock &directiveSource);

  // Check that only clauses in set are after the specific clauses.
````
- **L361 EN**: Starts a function, method, lambda, or structured scope: `void PushContextAndClauseSets(const parser::CharBlock &source, D dir) {`.
  **L361 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void PushContextAndClauseSets(const parser::CharBlock &source, D dir) {`。
- **L362 EN**: Executes a call or declaration centered on `PushContext`.
  **L362 CN**: 执行以 `PushContext` 为核心的调用或声明。
- **L363 EN**: Executes a call or declaration centered on `SetClauseSets`.
  **L363 CN**: 执行以 `SetClauseSets` 为核心的调用或声明。
- **L364 EN**: Closes the current lexical scope or compound statement.
  **L364 CN**: 结束当前词法作用域或复合语句块。
- **L365 EN**: Blank line separating nearby declarations or logic blocks.
  **L365 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L366 EN**: Executes a call or declaration centered on `SayNotMatching`.
  **L366 CN**: 执行以 `SayNotMatching` 为核心的调用或声明。
- **L367 EN**: Blank line separating nearby declarations or logic blocks.
  **L367 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L368 EN**: Introduces template parameters or specialization context: `template <typename B> void CheckMatching(const B &beginDir, const B &endDir) {`.
  **L368 CN**: 为后续声明引入模板参数或特化上下文：`template <typename B> void CheckMatching(const B &beginDir, const B &endDir) {`。
- **L369 EN**: Executes a standalone statement or declaration: `const auto &begin{beginDir.v};`.
  **L369 CN**: 执行一条独立语句或声明：`const auto &begin{beginDir.v};`。
- **L370 EN**: Executes a standalone statement or declaration: `const auto &end{endDir.v};`.
  **L370 CN**: 执行一条独立语句或声明：`const auto &end{endDir.v};`。
- **L371 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L371 CN**: 开始 `if` 控制流语句并计算其条件。
- **L372 EN**: Executes a call or declaration centered on `SayNotMatching`.
  **L372 CN**: 执行以 `SayNotMatching` 为核心的调用或声明。
- **L373 EN**: Closes the current lexical scope or compound statement.
  **L373 CN**: 结束当前词法作用域或复合语句块。
- **L374 EN**: Closes the current lexical scope or compound statement.
  **L374 CN**: 结束当前词法作用域或复合语句块。
- **L375 EN**: Comment explains nearby logic, intent, or metadata: `Check illegal branching out of `Parser::Block` for `Parser::Name` based`.
  **L375 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check illegal branching out of `Parser::Block` for `Parser::Name` based`。
- **L376 EN**: Comment explains nearby logic, intent, or metadata: `nodes (example `Parser::ExitStmt`)`.
  **L376 CN**: 注释说明附近代码的逻辑、意图或元数据：`nodes (example `Parser::ExitStmt`)`。
- **L377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckNoBranching(const parser::Block &block, D directive,`.
  **L377 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckNoBranching(const parser::Block &block, D directive,`。
- **L378 EN**: Executes a standalone statement or declaration: `const parser::CharBlock &directiveSource);`.
  **L378 CN**: 执行一条独立语句或声明：`const parser::CharBlock &directiveSource);`。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L380 EN**: Comment explains nearby logic, intent, or metadata: `Check that only clauses in set are after the specific clauses.`.
  **L380 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check that only clauses in set are after the specific clauses.`。

### Lines 381-400

````cpp
  void CheckOnlyAllowedAfter(C clause, common::EnumSet<C, ClauseEnumSize> set);

  void CheckRequireAtLeastOneOf(bool warnInsteadOfError = false);

  // Check if a clause is allowed on a directive. Returns true if is and
  // false otherwise.
  bool CheckAllowed(C clause, bool warnInsteadOfError = false);

  // Check that the clause appears only once. The counter is reset when the
  // separator clause appears.
  void CheckAllowedOncePerGroup(C clause, C separator);

  void CheckMutuallyExclusivePerGroup(
      C clause, C separator, common::EnumSet<C, ClauseEnumSize> set);

  void CheckAtLeastOneClause();

  void CheckNotAllowedIfClause(
      C clause, common::EnumSet<C, ClauseEnumSize> set);

````
- **L381 EN**: Executes a call or declaration centered on `CheckOnlyAllowedAfter`.
  **L381 CN**: 执行以 `CheckOnlyAllowedAfter` 为核心的调用或声明。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L383 EN**: Executes a call or declaration centered on `CheckRequireAtLeastOneOf`.
  **L383 CN**: 执行以 `CheckRequireAtLeastOneOf` 为核心的调用或声明。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L385 EN**: Comment explains nearby logic, intent, or metadata: `Check if a clause is allowed on a directive. Returns true if is and`.
  **L385 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check if a clause is allowed on a directive. Returns true if is and`。
- **L386 EN**: Comment explains nearby logic, intent, or metadata: `false otherwise.`.
  **L386 CN**: 注释说明附近代码的逻辑、意图或元数据：`false otherwise.`。
- **L387 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L387 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L388 EN**: Blank line separating nearby declarations or logic blocks.
  **L388 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L389 EN**: Comment explains nearby logic, intent, or metadata: `Check that the clause appears only once. The counter is reset when the`.
  **L389 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check that the clause appears only once. The counter is reset when the`。
- **L390 EN**: Comment explains nearby logic, intent, or metadata: `separator clause appears.`.
  **L390 CN**: 注释说明附近代码的逻辑、意图或元数据：`separator clause appears.`。
- **L391 EN**: Executes a call or declaration centered on `CheckAllowedOncePerGroup`.
  **L391 CN**: 执行以 `CheckAllowedOncePerGroup` 为核心的调用或声明。
- **L392 EN**: Blank line separating nearby declarations or logic blocks.
  **L392 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L393 EN**: Continues logic associated with callable symbol `CheckMutuallyExclusivePerGroup`.
  **L393 CN**: 继续与可调用符号 `CheckMutuallyExclusivePerGroup` 相关的逻辑。
- **L394 EN**: Executes a standalone statement or declaration: `C clause, C separator, common::EnumSet<C, ClauseEnumSize> set);`.
  **L394 CN**: 执行一条独立语句或声明：`C clause, C separator, common::EnumSet<C, ClauseEnumSize> set);`。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L396 EN**: Executes a call or declaration centered on `CheckAtLeastOneClause`.
  **L396 CN**: 执行以 `CheckAtLeastOneClause` 为核心的调用或声明。
- **L397 EN**: Blank line separating nearby declarations or logic blocks.
  **L397 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L398 EN**: Continues logic associated with callable symbol `CheckNotAllowedIfClause`.
  **L398 CN**: 继续与可调用符号 `CheckNotAllowedIfClause` 相关的逻辑。
- **L399 EN**: Executes a standalone statement or declaration: `C clause, common::EnumSet<C, ClauseEnumSize> set);`.
  **L399 CN**: 执行一条独立语句或声明：`C clause, common::EnumSet<C, ClauseEnumSize> set);`。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 401-420

````cpp
  std::string ContextDirectiveAsFortran();

  void RequiresConstantPositiveParameter(
      const C &clause, const parser::ScalarIntConstantExpr &i);

  void RequiresPositiveParameter(const C &clause,
      const parser::ScalarIntExpr &i, llvm::StringRef paramName = "parameter",
      bool allowZero = true);

  void OptionalConstantPositiveParameter(
      const C &clause, const std::optional<parser::ScalarIntConstantExpr> &o);

  virtual llvm::StringRef getClauseName(C clause) { return ""; };

  virtual llvm::StringRef getDirectiveName(D directive) { return ""; };

  SemanticsContext &context_;
  std::vector<DirectiveContext> dirContext_; // used as a stack
  std::unordered_map<D, DirectiveClauses<C, ClauseEnumSize>>
      directiveClausesMap_;
````
- **L401 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L401 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L403 EN**: Continues logic associated with callable symbol `RequiresConstantPositiveParameter`.
  **L403 CN**: 继续与可调用符号 `RequiresConstantPositiveParameter` 相关的逻辑。
- **L404 EN**: Executes a standalone statement or declaration: `const C &clause, const parser::ScalarIntConstantExpr &i);`.
  **L404 CN**: 执行一条独立语句或声明：`const C &clause, const parser::ScalarIntConstantExpr &i);`。
- **L405 EN**: Blank line separating nearby declarations or logic blocks.
  **L405 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void RequiresPositiveParameter(const C &clause,`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`void RequiresPositiveParameter(const C &clause,`。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const parser::ScalarIntExpr &i, llvm::StringRef paramName = "parameter",`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`const parser::ScalarIntExpr &i, llvm::StringRef paramName = "parameter",`。
- **L408 EN**: Initializes variable `allowZero` from the right-hand expression.
  **L408 CN**: 使用右侧表达式初始化变量 `allowZero`。
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L410 EN**: Continues logic associated with callable symbol `OptionalConstantPositiveParameter`.
  **L410 CN**: 继续与可调用符号 `OptionalConstantPositiveParameter` 相关的逻辑。
- **L411 EN**: Executes a standalone statement or declaration: `const C &clause, const std::optional<parser::ScalarIntConstantExpr> &o);`.
  **L411 CN**: 执行一条独立语句或声明：`const C &clause, const std::optional<parser::ScalarIntConstantExpr> &o);`。
- **L412 EN**: Blank line separating nearby declarations or logic blocks.
  **L412 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L413 EN**: Executes a call or declaration centered on `getClauseName`.
  **L413 CN**: 执行以 `getClauseName` 为核心的调用或声明。
- **L414 EN**: Blank line separating nearby declarations or logic blocks.
  **L414 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L415 EN**: Executes a call or declaration centered on `getDirectiveName`.
  **L415 CN**: 执行以 `getDirectiveName` 为核心的调用或声明。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L417 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L417 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。
- **L418 EN**: Continues the surrounding expression or declaration: `std::vector<DirectiveContext> dirContext_; // used as a stack`.
  **L418 CN**: 继续构造周围的表达式或声明：`std::vector<DirectiveContext> dirContext_; // used as a stack`。
- **L419 EN**: Continues the surrounding expression or declaration: `std::unordered_map<D, DirectiveClauses<C, ClauseEnumSize>>`.
  **L419 CN**: 继续构造周围的表达式或声明：`std::unordered_map<D, DirectiveClauses<C, ClauseEnumSize>>`。
- **L420 EN**: Executes a standalone statement or declaration: `directiveClausesMap_;`.
  **L420 CN**: 执行一条独立语句或声明：`directiveClausesMap_;`。

### Lines 421-440

````cpp

  std::string ClauseSetToString(const common::EnumSet<C, ClauseEnumSize> set);
};

// Collect all labels defined in a block.
struct LabelCollector {
  std::set<parser::Label> labels;
  template <typename T> bool Pre(const T &) { return true; }
  template <typename T> void Post(const T &) {}
  template <typename T> bool Pre(const parser::Statement<T> &stmt) {
    if (stmt.label)
      labels.insert(*stmt.label);
    return true;
  }
};

template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
void DirectiveStructureChecker<D, C, PC, ClauseEnumSize>::CheckNoBranching(
    const parser::Block &block, D directive,
    const parser::CharBlock &directiveSource) {
````
- **L421 EN**: Blank line separating nearby declarations or logic blocks.
  **L421 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L422 EN**: Executes a call or declaration centered on `ClauseSetToString`.
  **L422 CN**: 执行以 `ClauseSetToString` 为核心的调用或声明。
- **L423 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L423 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L424 EN**: Blank line separating nearby declarations or logic blocks.
  **L424 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L425 EN**: Comment explains nearby logic, intent, or metadata: `Collect all labels defined in a block.`.
  **L425 CN**: 注释说明附近代码的逻辑、意图或元数据：`Collect all labels defined in a block.`。
- **L426 EN**: Declares struct `LabelCollector`.
  **L426 CN**: 声明 struct `LabelCollector`。
- **L427 EN**: Executes a standalone statement or declaration: `std::set<parser::Label> labels;`.
  **L427 CN**: 执行一条独立语句或声明：`std::set<parser::Label> labels;`。
- **L428 EN**: Introduces template parameters or specialization context: `template <typename T> bool Pre(const T &) { return true; }`.
  **L428 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> bool Pre(const T &) { return true; }`。
- **L429 EN**: Introduces template parameters or specialization context: `template <typename T> void Post(const T &) {}`.
  **L429 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> void Post(const T &) {}`。
- **L430 EN**: Introduces template parameters or specialization context: `template <typename T> bool Pre(const parser::Statement<T> &stmt) {`.
  **L430 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> bool Pre(const parser::Statement<T> &stmt) {`。
- **L431 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L431 CN**: 开始 `if` 控制流语句并计算其条件。
- **L432 EN**: Executes a call or declaration centered on `labels.insert`.
  **L432 CN**: 执行以 `labels.insert` 为核心的调用或声明。
- **L433 EN**: Returns from the current function with `true`.
  **L433 CN**: 以 `true` 从当前函数返回。
- **L434 EN**: Closes the current lexical scope or compound statement.
  **L434 CN**: 结束当前词法作用域或复合语句块。
- **L435 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L435 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L436 EN**: Blank line separating nearby declarations or logic blocks.
  **L436 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L437 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L437 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L438 EN**: Continues logic associated with callable symbol `CheckNoBranching`.
  **L438 CN**: 继续与可调用符号 `CheckNoBranching` 相关的逻辑。
- **L439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const parser::Block &block, D directive,`.
  **L439 CN**: 继续一个多行参数列表、初始化器或聚合项：`const parser::Block &block, D directive,`。
- **L440 EN**: Continues the surrounding expression or declaration: `const parser::CharBlock &directiveSource) {`.
  **L440 CN**: 继续构造周围的表达式或声明：`const parser::CharBlock &directiveSource) {`。

### Lines 441-460

````cpp
  LabelCollector labelCollector;
  parser::Walk(block, labelCollector);
  NoBranchingEnforce<D> noBranchingEnforce{
      context_, directiveSource, directive, ContextDirectiveAsFortran()};
  for (auto label : labelCollector.labels)
    noBranchingEnforce.CollectLabel(label);
  parser::Walk(block, noBranchingEnforce);
}

// Check that only clauses included in the given set are present after the given
// clause.
template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
void DirectiveStructureChecker<D, C, PC, ClauseEnumSize>::CheckOnlyAllowedAfter(
    C clause, common::EnumSet<C, ClauseEnumSize> set) {
  bool enforceCheck = false;
  for (auto cl : GetContext().actualClauses) {
    if (cl == clause) {
      enforceCheck = true;
      continue;
    } else if (enforceCheck && !set.test(cl)) {
````
- **L441 EN**: Executes a standalone statement or declaration: `LabelCollector labelCollector;`.
  **L441 CN**: 执行一条独立语句或声明：`LabelCollector labelCollector;`。
- **L442 EN**: Executes a call or declaration centered on `parser::Walk`.
  **L442 CN**: 执行以 `parser::Walk` 为核心的调用或声明。
- **L443 EN**: Continues the surrounding expression or declaration: `NoBranchingEnforce<D> noBranchingEnforce{`.
  **L443 CN**: 继续构造周围的表达式或声明：`NoBranchingEnforce<D> noBranchingEnforce{`。
- **L444 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L444 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L445 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L445 CN**: 开始 `for` 控制流语句并计算其条件。
- **L446 EN**: Executes a call or declaration centered on `noBranchingEnforce.CollectLabel`.
  **L446 CN**: 执行以 `noBranchingEnforce.CollectLabel` 为核心的调用或声明。
- **L447 EN**: Executes a call or declaration centered on `parser::Walk`.
  **L447 CN**: 执行以 `parser::Walk` 为核心的调用或声明。
- **L448 EN**: Closes the current lexical scope or compound statement.
  **L448 CN**: 结束当前词法作用域或复合语句块。
- **L449 EN**: Blank line separating nearby declarations or logic blocks.
  **L449 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L450 EN**: Comment explains nearby logic, intent, or metadata: `Check that only clauses included in the given set are present after the given`.
  **L450 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check that only clauses included in the given set are present after the given`。
- **L451 EN**: Comment explains nearby logic, intent, or metadata: `clause.`.
  **L451 CN**: 注释说明附近代码的逻辑、意图或元数据：`clause.`。
- **L452 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L452 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L453 EN**: Continues logic associated with callable symbol `CheckOnlyAllowedAfter`.
  **L453 CN**: 继续与可调用符号 `CheckOnlyAllowedAfter` 相关的逻辑。
- **L454 EN**: Continues the surrounding expression or declaration: `C clause, common::EnumSet<C, ClauseEnumSize> set) {`.
  **L454 CN**: 继续构造周围的表达式或声明：`C clause, common::EnumSet<C, ClauseEnumSize> set) {`。
- **L455 EN**: Initializes variable `enforceCheck` from the right-hand expression.
  **L455 CN**: 使用右侧表达式初始化变量 `enforceCheck`。
- **L456 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L456 CN**: 开始 `for` 控制流语句并计算其条件。
- **L457 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L457 CN**: 开始 `if` 控制流语句并计算其条件。
- **L458 EN**: Executes a standalone statement or declaration: `enforceCheck = true;`.
  **L458 CN**: 执行一条独立语句或声明：`enforceCheck = true;`。
- **L459 EN**: Skips to the next loop iteration.
  **L459 CN**: 跳到下一次循环迭代。
- **L460 EN**: Transitions from the previous branch into an `else if` condition.
  **L460 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 461-480

````cpp
      auto parserClause = GetContext().clauseInfo.find(cl);
      context_.Say(parserClause->second->source,
          "Clause %s is not allowed after clause %s on the %s "
          "directive"_err_en_US,
          parser::ToUpperCaseLetters(getClauseName(cl).str()),
          parser::ToUpperCaseLetters(getClauseName(clause).str()),
          ContextDirectiveAsFortran());
    }
  }
}

// Check that at least one clause is attached to the directive.
template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
void DirectiveStructureChecker<D, C, PC,
    ClauseEnumSize>::CheckAtLeastOneClause() {
  if (GetContext().actualClauses.empty()) {
    context_.Say(GetContext().directiveSource,
        "At least one clause is required on the %s directive"_err_en_US,
        ContextDirectiveAsFortran());
  }
````
- **L461 EN**: Initializes variable `parserClause` from the right-hand expression.
  **L461 CN**: 使用右侧表达式初始化变量 `parserClause`。
- **L462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parserClause->second->source,`.
  **L462 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parserClause->second->source,`。
- **L463 EN**: Continues the surrounding expression or declaration: `"Clause %s is not allowed after clause %s on the %s "`.
  **L463 CN**: 继续构造周围的表达式或声明：`"Clause %s is not allowed after clause %s on the %s "`。
- **L464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"directive"_err_en_US,`.
  **L464 CN**: 继续一个多行参数列表、初始化器或聚合项：`"directive"_err_en_US,`。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(cl).str()),`.
  **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(cl).str()),`。
- **L466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(clause).str()),`.
  **L466 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(clause).str()),`。
- **L467 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L467 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L468 EN**: Closes the current lexical scope or compound statement.
  **L468 CN**: 结束当前词法作用域或复合语句块。
- **L469 EN**: Closes the current lexical scope or compound statement.
  **L469 CN**: 结束当前词法作用域或复合语句块。
- **L470 EN**: Closes the current lexical scope or compound statement.
  **L470 CN**: 结束当前词法作用域或复合语句块。
- **L471 EN**: Blank line separating nearby declarations or logic blocks.
  **L471 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L472 EN**: Comment explains nearby logic, intent, or metadata: `Check that at least one clause is attached to the directive.`.
  **L472 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check that at least one clause is attached to the directive.`。
- **L473 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L473 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void DirectiveStructureChecker<D, C, PC,`.
  **L474 CN**: 继续一个多行参数列表、初始化器或聚合项：`void DirectiveStructureChecker<D, C, PC,`。
- **L475 EN**: Starts a function, method, lambda, or structured scope: `ClauseEnumSize>::CheckAtLeastOneClause() {`.
  **L475 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ClauseEnumSize>::CheckAtLeastOneClause() {`。
- **L476 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L476 CN**: 开始 `if` 控制流语句并计算其条件。
- **L477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().directiveSource,`.
  **L477 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().directiveSource,`。
- **L478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"At least one clause is required on the %s directive"_err_en_US,`.
  **L478 CN**: 继续一个多行参数列表、初始化器或聚合项：`"At least one clause is required on the %s directive"_err_en_US,`。
- **L479 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L479 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L480 EN**: Closes the current lexical scope or compound statement.
  **L480 CN**: 结束当前词法作用域或复合语句块。

### Lines 481-500

````cpp
}

template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
std::string
DirectiveStructureChecker<D, C, PC, ClauseEnumSize>::ClauseSetToString(
    const common::EnumSet<C, ClauseEnumSize> set) {
  std::string list;
  set.IterateOverMembers([&](C o) {
    if (!list.empty())
      list.append(", ");
    list.append(parser::ToUpperCaseLetters(getClauseName(o).str()));
  });
  return list;
}

// Check that at least one clause in the required set is present on the
// directive.
template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
void DirectiveStructureChecker<D, C, PC,
    ClauseEnumSize>::CheckRequireAtLeastOneOf(bool warnInsteadOfError) {
````
- **L481 EN**: Closes the current lexical scope or compound statement.
  **L481 CN**: 结束当前词法作用域或复合语句块。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L483 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L483 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L484 EN**: Continues the surrounding expression or declaration: `std::string`.
  **L484 CN**: 继续构造周围的表达式或声明：`std::string`。
- **L485 EN**: Continues logic associated with callable symbol `ClauseSetToString`.
  **L485 CN**: 继续与可调用符号 `ClauseSetToString` 相关的逻辑。
- **L486 EN**: Continues the surrounding expression or declaration: `const common::EnumSet<C, ClauseEnumSize> set) {`.
  **L486 CN**: 继续构造周围的表达式或声明：`const common::EnumSet<C, ClauseEnumSize> set) {`。
- **L487 EN**: Executes a standalone statement or declaration: `std::string list;`.
  **L487 CN**: 执行一条独立语句或声明：`std::string list;`。
- **L488 EN**: Starts a function, method, lambda, or structured scope: `set.IterateOverMembers([&](C o) {`.
  **L488 CN**: 开始一个函数、方法、lambda 或结构化作用域：`set.IterateOverMembers([&](C o) {`。
- **L489 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L489 CN**: 开始 `if` 控制流语句并计算其条件。
- **L490 EN**: Executes a call or declaration centered on `list.append`.
  **L490 CN**: 执行以 `list.append` 为核心的调用或声明。
- **L491 EN**: Executes a call or declaration centered on `list.append`.
  **L491 CN**: 执行以 `list.append` 为核心的调用或声明。
- **L492 EN**: Executes a standalone statement or declaration: `});`.
  **L492 CN**: 执行一条独立语句或声明：`});`。
- **L493 EN**: Returns from the current function with `list`.
  **L493 CN**: 以 `list` 从当前函数返回。
- **L494 EN**: Closes the current lexical scope or compound statement.
  **L494 CN**: 结束当前词法作用域或复合语句块。
- **L495 EN**: Blank line separating nearby declarations or logic blocks.
  **L495 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L496 EN**: Comment explains nearby logic, intent, or metadata: `Check that at least one clause in the required set is present on the`.
  **L496 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check that at least one clause in the required set is present on the`。
- **L497 EN**: Comment explains nearby logic, intent, or metadata: `directive.`.
  **L497 CN**: 注释说明附近代码的逻辑、意图或元数据：`directive.`。
- **L498 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L498 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void DirectiveStructureChecker<D, C, PC,`.
  **L499 CN**: 继续一个多行参数列表、初始化器或聚合项：`void DirectiveStructureChecker<D, C, PC,`。
- **L500 EN**: Starts a function, method, lambda, or structured scope: `ClauseEnumSize>::CheckRequireAtLeastOneOf(bool warnInsteadOfError) {`.
  **L500 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ClauseEnumSize>::CheckRequireAtLeastOneOf(bool warnInsteadOfError) {`。

### Lines 501-520

````cpp
  if (GetContext().requiredClauses.empty()) {
    return;
  }
  for (auto cl : GetContext().actualClauses) {
    if (GetContext().requiredClauses.test(cl)) {
      return;
    }
  }
  // No clause matched in the actual clauses list
  if (warnInsteadOfError) {
    context_.Warn(common::UsageWarning::Portability,
        GetContext().directiveSource,
        "At least one of %s clause should appear on the %s directive"_port_en_US,
        ClauseSetToString(GetContext().requiredClauses),
        ContextDirectiveAsFortran());
  } else {
    context_.Say(GetContext().directiveSource,
        "At least one of %s clause must appear on the %s directive"_err_en_US,
        ClauseSetToString(GetContext().requiredClauses),
        ContextDirectiveAsFortran());
````
- **L501 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L501 CN**: 开始 `if` 控制流语句并计算其条件。
- **L502 EN**: Returns from the current function with `void`.
  **L502 CN**: 以 `void` 从当前函数返回。
- **L503 EN**: Closes the current lexical scope or compound statement.
  **L503 CN**: 结束当前词法作用域或复合语句块。
- **L504 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L504 CN**: 开始 `for` 控制流语句并计算其条件。
- **L505 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L505 CN**: 开始 `if` 控制流语句并计算其条件。
- **L506 EN**: Returns from the current function with `void`.
  **L506 CN**: 以 `void` 从当前函数返回。
- **L507 EN**: Closes the current lexical scope or compound statement.
  **L507 CN**: 结束当前词法作用域或复合语句块。
- **L508 EN**: Closes the current lexical scope or compound statement.
  **L508 CN**: 结束当前词法作用域或复合语句块。
- **L509 EN**: Comment explains nearby logic, intent, or metadata: `No clause matched in the actual clauses list`.
  **L509 CN**: 注释说明附近代码的逻辑、意图或元数据：`No clause matched in the actual clauses list`。
- **L510 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L510 CN**: 开始 `if` 控制流语句并计算其条件。
- **L511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::UsageWarning::Portability,`.
  **L511 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::UsageWarning::Portability,`。
- **L512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `GetContext().directiveSource,`.
  **L512 CN**: 继续一个多行参数列表、初始化器或聚合项：`GetContext().directiveSource,`。
- **L513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"At least one of %s clause should appear on the %s directive"_port_en_US,`.
  **L513 CN**: 继续一个多行参数列表、初始化器或聚合项：`"At least one of %s clause should appear on the %s directive"_port_en_US,`。
- **L514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClauseSetToString(GetContext().requiredClauses),`.
  **L514 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClauseSetToString(GetContext().requiredClauses),`。
- **L515 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L515 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L516 EN**: Transitions from the previous branch into the alternative path.
  **L516 CN**: 从前一个分支过渡到备选路径。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().directiveSource,`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().directiveSource,`。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"At least one of %s clause must appear on the %s directive"_err_en_US,`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`"At least one of %s clause must appear on the %s directive"_err_en_US,`。
- **L519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClauseSetToString(GetContext().requiredClauses),`.
  **L519 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClauseSetToString(GetContext().requiredClauses),`。
- **L520 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L520 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。

### Lines 521-540

````cpp
  }
}

template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
std::string DirectiveStructureChecker<D, C, PC,
    ClauseEnumSize>::ContextDirectiveAsFortran() {
  return parser::ToUpperCaseLetters(
      getDirectiveName(GetContext().directive).str());
}

// Check that clauses present on the directive are allowed clauses.
template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
bool DirectiveStructureChecker<D, C, PC, ClauseEnumSize>::CheckAllowed(
    C clause, bool warnInsteadOfError) {
  if (!GetContext().allowedClauses.test(clause) &&
      !GetContext().allowedOnceClauses.test(clause) &&
      !GetContext().allowedExclusiveClauses.test(clause) &&
      !GetContext().requiredClauses.test(clause)) {
    if (warnInsteadOfError) {
      context_.Warn(common::UsageWarning::Portability,
````
- **L521 EN**: Closes the current lexical scope or compound statement.
  **L521 CN**: 结束当前词法作用域或复合语句块。
- **L522 EN**: Closes the current lexical scope or compound statement.
  **L522 CN**: 结束当前词法作用域或复合语句块。
- **L523 EN**: Blank line separating nearby declarations or logic blocks.
  **L523 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L524 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L524 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::string DirectiveStructureChecker<D, C, PC,`.
  **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::string DirectiveStructureChecker<D, C, PC,`。
- **L526 EN**: Starts a function, method, lambda, or structured scope: `ClauseEnumSize>::ContextDirectiveAsFortran() {`.
  **L526 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ClauseEnumSize>::ContextDirectiveAsFortran() {`。
- **L527 EN**: Returns from the current function with `parser::ToUpperCaseLetters(`.
  **L527 CN**: 以 `parser::ToUpperCaseLetters(` 从当前函数返回。
- **L528 EN**: Executes a call or declaration centered on `getDirectiveName`.
  **L528 CN**: 执行以 `getDirectiveName` 为核心的调用或声明。
- **L529 EN**: Closes the current lexical scope or compound statement.
  **L529 CN**: 结束当前词法作用域或复合语句块。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L531 EN**: Comment explains nearby logic, intent, or metadata: `Check that clauses present on the directive are allowed clauses.`.
  **L531 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check that clauses present on the directive are allowed clauses.`。
- **L532 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L532 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L533 EN**: Continues logic associated with callable symbol `CheckAllowed`.
  **L533 CN**: 继续与可调用符号 `CheckAllowed` 相关的逻辑。
- **L534 EN**: Continues the surrounding expression or declaration: `C clause, bool warnInsteadOfError) {`.
  **L534 CN**: 继续构造周围的表达式或声明：`C clause, bool warnInsteadOfError) {`。
- **L535 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L535 CN**: 开始 `if` 控制流语句并计算其条件。
- **L536 EN**: Continues logic associated with callable symbol `GetContext`.
  **L536 CN**: 继续与可调用符号 `GetContext` 相关的逻辑。
- **L537 EN**: Continues logic associated with callable symbol `GetContext`.
  **L537 CN**: 继续与可调用符号 `GetContext` 相关的逻辑。
- **L538 EN**: Starts a function, method, lambda, or structured scope: `!GetContext().requiredClauses.test(clause)) {`.
  **L538 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!GetContext().requiredClauses.test(clause)) {`。
- **L539 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L539 CN**: 开始 `if` 控制流语句并计算其条件。
- **L540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::UsageWarning::Portability,`.
  **L540 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::UsageWarning::Portability,`。

### Lines 541-560

````cpp
          GetContext().clauseSource,
          "%s clause is not allowed on the %s directive and will be ignored"_port_en_US,
          parser::ToUpperCaseLetters(getClauseName(clause).str()),
          parser::ToUpperCaseLetters(GetContext().directiveSource.ToString()));
    } else {
      context_.Say(GetContext().clauseSource,
          "%s clause is not allowed on the %s directive"_err_en_US,
          parser::ToUpperCaseLetters(getClauseName(clause).str()),
          parser::ToUpperCaseLetters(GetContext().directiveSource.ToString()));
    }
    return false;
  }
  if ((GetContext().allowedOnceClauses.test(clause) ||
          GetContext().allowedExclusiveClauses.test(clause)) &&
      FindClause(clause)) {
    context_.Say(GetContext().clauseSource,
        "At most one %s clause can appear on the %s directive"_err_en_US,
        parser::ToUpperCaseLetters(getClauseName(clause).str()),
        parser::ToUpperCaseLetters(GetContext().directiveSource.ToString()));
    return false;
````
- **L541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `GetContext().clauseSource,`.
  **L541 CN**: 继续一个多行参数列表、初始化器或聚合项：`GetContext().clauseSource,`。
- **L542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s clause is not allowed on the %s directive and will be ignored"_port_en_US,`.
  **L542 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s clause is not allowed on the %s directive and will be ignored"_port_en_US,`。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(clause).str()),`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(clause).str()),`。
- **L544 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L544 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L545 EN**: Transitions from the previous branch into the alternative path.
  **L545 CN**: 从前一个分支过渡到备选路径。
- **L546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L546 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s clause is not allowed on the %s directive"_err_en_US,`.
  **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s clause is not allowed on the %s directive"_err_en_US,`。
- **L548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(clause).str()),`.
  **L548 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(clause).str()),`。
- **L549 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L549 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L550 EN**: Closes the current lexical scope or compound statement.
  **L550 CN**: 结束当前词法作用域或复合语句块。
- **L551 EN**: Returns from the current function with `false`.
  **L551 CN**: 以 `false` 从当前函数返回。
- **L552 EN**: Closes the current lexical scope or compound statement.
  **L552 CN**: 结束当前词法作用域或复合语句块。
- **L553 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L553 CN**: 开始 `if` 控制流语句并计算其条件。
- **L554 EN**: Continues logic associated with callable symbol `GetContext`.
  **L554 CN**: 继续与可调用符号 `GetContext` 相关的逻辑。
- **L555 EN**: Starts a function, method, lambda, or structured scope: `FindClause(clause)) {`.
  **L555 CN**: 开始一个函数、方法、lambda 或结构化作用域：`FindClause(clause)) {`。
- **L556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L556 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"At most one %s clause can appear on the %s directive"_err_en_US,`.
  **L557 CN**: 继续一个多行参数列表、初始化器或聚合项：`"At most one %s clause can appear on the %s directive"_err_en_US,`。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(clause).str()),`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(clause).str()),`。
- **L559 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L559 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L560 EN**: Returns from the current function with `false`.
  **L560 CN**: 以 `false` 从当前函数返回。

### Lines 561-580

````cpp
  }
  if (GetContext().allowedExclusiveClauses.test(clause)) {
    std::vector<C> others;
    GetContext().allowedExclusiveClauses.IterateOverMembers([&](C o) {
      if (FindClause(o)) {
        others.emplace_back(o);
      }
    });
    for (const auto &e : others) {
      context_.Say(GetContext().clauseSource,
          "%s and %s clauses are mutually exclusive and may not appear on the "
          "same %s directive"_err_en_US,
          parser::ToUpperCaseLetters(getClauseName(clause).str()),
          parser::ToUpperCaseLetters(getClauseName(e).str()),
          parser::ToUpperCaseLetters(GetContext().directiveSource.ToString()));
    }
    if (!others.empty()) {
      return false;
    }
  }
````
- **L561 EN**: Closes the current lexical scope or compound statement.
  **L561 CN**: 结束当前词法作用域或复合语句块。
- **L562 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L562 CN**: 开始 `if` 控制流语句并计算其条件。
- **L563 EN**: Executes a standalone statement or declaration: `std::vector<C> others;`.
  **L563 CN**: 执行一条独立语句或声明：`std::vector<C> others;`。
- **L564 EN**: Starts a function, method, lambda, or structured scope: `GetContext().allowedExclusiveClauses.IterateOverMembers([&](C o) {`.
  **L564 CN**: 开始一个函数、方法、lambda 或结构化作用域：`GetContext().allowedExclusiveClauses.IterateOverMembers([&](C o) {`。
- **L565 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L565 CN**: 开始 `if` 控制流语句并计算其条件。
- **L566 EN**: Executes a call or declaration centered on `others.emplace_back`.
  **L566 CN**: 执行以 `others.emplace_back` 为核心的调用或声明。
- **L567 EN**: Closes the current lexical scope or compound statement.
  **L567 CN**: 结束当前词法作用域或复合语句块。
- **L568 EN**: Executes a standalone statement or declaration: `});`.
  **L568 CN**: 执行一条独立语句或声明：`});`。
- **L569 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L569 CN**: 开始 `for` 控制流语句并计算其条件。
- **L570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L570 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L571 EN**: Continues the surrounding expression or declaration: `"%s and %s clauses are mutually exclusive and may not appear on the "`.
  **L571 CN**: 继续构造周围的表达式或声明：`"%s and %s clauses are mutually exclusive and may not appear on the "`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"same %s directive"_err_en_US,`.
  **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`"same %s directive"_err_en_US,`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(clause).str()),`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(clause).str()),`。
- **L574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(e).str()),`.
  **L574 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(e).str()),`。
- **L575 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L575 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L576 EN**: Closes the current lexical scope or compound statement.
  **L576 CN**: 结束当前词法作用域或复合语句块。
- **L577 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L577 CN**: 开始 `if` 控制流语句并计算其条件。
- **L578 EN**: Returns from the current function with `false`.
  **L578 CN**: 以 `false` 从当前函数返回。
- **L579 EN**: Closes the current lexical scope or compound statement.
  **L579 CN**: 结束当前词法作用域或复合语句块。
- **L580 EN**: Closes the current lexical scope or compound statement.
  **L580 CN**: 结束当前词法作用域或复合语句块。

### Lines 581-600

````cpp
  SetContextClauseInfo(clause);
  AddClauseToCrtContext(clause);
  AddClauseToCrtGroupInContext(clause);
  return true;
}

// Enforce restriction where clauses in the given set are not allowed if the
// given clause appears.
template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
void DirectiveStructureChecker<D, C, PC,
    ClauseEnumSize>::CheckNotAllowedIfClause(C clause,
    common::EnumSet<C, ClauseEnumSize> set) {
  if (!llvm::is_contained(GetContext().actualClauses, clause)) {
    return; // Clause is not present
  }

  for (auto cl : GetContext().actualClauses) {
    if (set.test(cl)) {
      context_.Say(GetContext().directiveSource,
          "Clause %s is not allowed if clause %s appears on the %s directive"_err_en_US,
````
- **L581 EN**: Executes a call or declaration centered on `SetContextClauseInfo`.
  **L581 CN**: 执行以 `SetContextClauseInfo` 为核心的调用或声明。
- **L582 EN**: Executes a call or declaration centered on `AddClauseToCrtContext`.
  **L582 CN**: 执行以 `AddClauseToCrtContext` 为核心的调用或声明。
- **L583 EN**: Executes a call or declaration centered on `AddClauseToCrtGroupInContext`.
  **L583 CN**: 执行以 `AddClauseToCrtGroupInContext` 为核心的调用或声明。
- **L584 EN**: Returns from the current function with `true`.
  **L584 CN**: 以 `true` 从当前函数返回。
- **L585 EN**: Closes the current lexical scope or compound statement.
  **L585 CN**: 结束当前词法作用域或复合语句块。
- **L586 EN**: Blank line separating nearby declarations or logic blocks.
  **L586 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L587 EN**: Comment explains nearby logic, intent, or metadata: `Enforce restriction where clauses in the given set are not allowed if the`.
  **L587 CN**: 注释说明附近代码的逻辑、意图或元数据：`Enforce restriction where clauses in the given set are not allowed if the`。
- **L588 EN**: Comment explains nearby logic, intent, or metadata: `given clause appears.`.
  **L588 CN**: 注释说明附近代码的逻辑、意图或元数据：`given clause appears.`。
- **L589 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L589 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void DirectiveStructureChecker<D, C, PC,`.
  **L590 CN**: 继续一个多行参数列表、初始化器或聚合项：`void DirectiveStructureChecker<D, C, PC,`。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClauseEnumSize>::CheckNotAllowedIfClause(C clause,`.
  **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClauseEnumSize>::CheckNotAllowedIfClause(C clause,`。
- **L592 EN**: Continues the surrounding expression or declaration: `common::EnumSet<C, ClauseEnumSize> set) {`.
  **L592 CN**: 继续构造周围的表达式或声明：`common::EnumSet<C, ClauseEnumSize> set) {`。
- **L593 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L593 CN**: 开始 `if` 控制流语句并计算其条件。
- **L594 EN**: Returns from the current function with `; // Clause is not present`.
  **L594 CN**: 以 `; // Clause is not present` 从当前函数返回。
- **L595 EN**: Closes the current lexical scope or compound statement.
  **L595 CN**: 结束当前词法作用域或复合语句块。
- **L596 EN**: Blank line separating nearby declarations or logic blocks.
  **L596 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L597 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L597 CN**: 开始 `for` 控制流语句并计算其条件。
- **L598 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L598 CN**: 开始 `if` 控制流语句并计算其条件。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().directiveSource,`.
  **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().directiveSource,`。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Clause %s is not allowed if clause %s appears on the %s directive"_err_en_US,`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Clause %s is not allowed if clause %s appears on the %s directive"_err_en_US,`。

### Lines 601-620

````cpp
          parser::ToUpperCaseLetters(getClauseName(cl).str()),
          parser::ToUpperCaseLetters(getClauseName(clause).str()),
          ContextDirectiveAsFortran());
    }
  }
}

template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
void DirectiveStructureChecker<D, C, PC,
    ClauseEnumSize>::CheckAllowedOncePerGroup(C clause, C separator) {
  bool clauseIsPresent = false;
  for (auto cl : GetContext().actualClauses) {
    if (cl == clause) {
      if (clauseIsPresent) {
        context_.Say(GetContext().clauseSource,
            "At most one %s clause can appear on the %s directive or in group separated by the %s clause"_err_en_US,
            parser::ToUpperCaseLetters(getClauseName(clause).str()),
            parser::ToUpperCaseLetters(GetContext().directiveSource.ToString()),
            parser::ToUpperCaseLetters(getClauseName(separator).str()));
      } else {
````
- **L601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(cl).str()),`.
  **L601 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(cl).str()),`。
- **L602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(clause).str()),`.
  **L602 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(clause).str()),`。
- **L603 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L603 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L604 EN**: Closes the current lexical scope or compound statement.
  **L604 CN**: 结束当前词法作用域或复合语句块。
- **L605 EN**: Closes the current lexical scope or compound statement.
  **L605 CN**: 结束当前词法作用域或复合语句块。
- **L606 EN**: Closes the current lexical scope or compound statement.
  **L606 CN**: 结束当前词法作用域或复合语句块。
- **L607 EN**: Blank line separating nearby declarations or logic blocks.
  **L607 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L608 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L608 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void DirectiveStructureChecker<D, C, PC,`.
  **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`void DirectiveStructureChecker<D, C, PC,`。
- **L610 EN**: Starts a function, method, lambda, or structured scope: `ClauseEnumSize>::CheckAllowedOncePerGroup(C clause, C separator) {`.
  **L610 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ClauseEnumSize>::CheckAllowedOncePerGroup(C clause, C separator) {`。
- **L611 EN**: Initializes variable `clauseIsPresent` from the right-hand expression.
  **L611 CN**: 使用右侧表达式初始化变量 `clauseIsPresent`。
- **L612 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L612 CN**: 开始 `for` 控制流语句并计算其条件。
- **L613 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L613 CN**: 开始 `if` 控制流语句并计算其条件。
- **L614 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L614 CN**: 开始 `if` 控制流语句并计算其条件。
- **L615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L615 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"At most one %s clause can appear on the %s directive or in group separated by the %s clause"_err_en_US,`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`"At most one %s clause can appear on the %s directive or in group separated by the %s clause"_err_en_US,`。
- **L617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(clause).str()),`.
  **L617 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(clause).str()),`。
- **L618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(GetContext().directiveSource.ToString()),`.
  **L618 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(GetContext().directiveSource.ToString()),`。
- **L619 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L619 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L620 EN**: Transitions from the previous branch into the alternative path.
  **L620 CN**: 从前一个分支过渡到备选路径。

### Lines 621-640

````cpp
        clauseIsPresent = true;
      }
    }
    if (cl == separator)
      clauseIsPresent = false;
  }
}

template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
void DirectiveStructureChecker<D, C, PC,
    ClauseEnumSize>::CheckMutuallyExclusivePerGroup(C clause, C separator,
    common::EnumSet<C, ClauseEnumSize> set) {

  // Checking of there is any offending clauses before the first separator.
  for (auto cl : GetContext().actualClauses) {
    if (cl == separator) {
      break;
    }
    if (set.test(cl)) {
      context_.Say(GetContext().directiveSource,
````
- **L621 EN**: Executes a standalone statement or declaration: `clauseIsPresent = true;`.
  **L621 CN**: 执行一条独立语句或声明：`clauseIsPresent = true;`。
- **L622 EN**: Closes the current lexical scope or compound statement.
  **L622 CN**: 结束当前词法作用域或复合语句块。
- **L623 EN**: Closes the current lexical scope or compound statement.
  **L623 CN**: 结束当前词法作用域或复合语句块。
- **L624 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L624 CN**: 开始 `if` 控制流语句并计算其条件。
- **L625 EN**: Executes a standalone statement or declaration: `clauseIsPresent = false;`.
  **L625 CN**: 执行一条独立语句或声明：`clauseIsPresent = false;`。
- **L626 EN**: Closes the current lexical scope or compound statement.
  **L626 CN**: 结束当前词法作用域或复合语句块。
- **L627 EN**: Closes the current lexical scope or compound statement.
  **L627 CN**: 结束当前词法作用域或复合语句块。
- **L628 EN**: Blank line separating nearby declarations or logic blocks.
  **L628 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L629 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L629 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void DirectiveStructureChecker<D, C, PC,`.
  **L630 CN**: 继续一个多行参数列表、初始化器或聚合项：`void DirectiveStructureChecker<D, C, PC,`。
- **L631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClauseEnumSize>::CheckMutuallyExclusivePerGroup(C clause, C separator,`.
  **L631 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClauseEnumSize>::CheckMutuallyExclusivePerGroup(C clause, C separator,`。
- **L632 EN**: Continues the surrounding expression or declaration: `common::EnumSet<C, ClauseEnumSize> set) {`.
  **L632 CN**: 继续构造周围的表达式或声明：`common::EnumSet<C, ClauseEnumSize> set) {`。
- **L633 EN**: Blank line separating nearby declarations or logic blocks.
  **L633 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L634 EN**: Comment explains nearby logic, intent, or metadata: `Checking of there is any offending clauses before the first separator.`.
  **L634 CN**: 注释说明附近代码的逻辑、意图或元数据：`Checking of there is any offending clauses before the first separator.`。
- **L635 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L635 CN**: 开始 `for` 控制流语句并计算其条件。
- **L636 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L636 CN**: 开始 `if` 控制流语句并计算其条件。
- **L637 EN**: Exits the nearest loop or switch statement.
  **L637 CN**: 退出最近的循环或 switch 语句。
- **L638 EN**: Closes the current lexical scope or compound statement.
  **L638 CN**: 结束当前词法作用域或复合语句块。
- **L639 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L639 CN**: 开始 `if` 控制流语句并计算其条件。
- **L640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().directiveSource,`.
  **L640 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().directiveSource,`。

### Lines 641-660

````cpp
          "Clause %s is not allowed if clause %s appears on the %s directive"_err_en_US,
          parser::ToUpperCaseLetters(getClauseName(clause).str()),
          parser::ToUpperCaseLetters(getClauseName(cl).str()),
          ContextDirectiveAsFortran());
    }
  }

  // Checking for mutually exclusive clauses in the current group.
  for (auto cl : GetContext().crtGroup) {
    if (set.test(cl)) {
      context_.Say(GetContext().directiveSource,
          "Clause %s is not allowed if clause %s appears on the %s directive"_err_en_US,
          parser::ToUpperCaseLetters(getClauseName(clause).str()),
          parser::ToUpperCaseLetters(getClauseName(cl).str()),
          ContextDirectiveAsFortran());
    }
  }
}

// Check the value of the clause is a constant positive integer.
````
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Clause %s is not allowed if clause %s appears on the %s directive"_err_en_US,`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Clause %s is not allowed if clause %s appears on the %s directive"_err_en_US,`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(clause).str()),`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(clause).str()),`。
- **L643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(cl).str()),`.
  **L643 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(cl).str()),`。
- **L644 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L644 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L645 EN**: Closes the current lexical scope or compound statement.
  **L645 CN**: 结束当前词法作用域或复合语句块。
- **L646 EN**: Closes the current lexical scope or compound statement.
  **L646 CN**: 结束当前词法作用域或复合语句块。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L648 EN**: Comment explains nearby logic, intent, or metadata: `Checking for mutually exclusive clauses in the current group.`.
  **L648 CN**: 注释说明附近代码的逻辑、意图或元数据：`Checking for mutually exclusive clauses in the current group.`。
- **L649 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L649 CN**: 开始 `for` 控制流语句并计算其条件。
- **L650 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L650 CN**: 开始 `if` 控制流语句并计算其条件。
- **L651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().directiveSource,`.
  **L651 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().directiveSource,`。
- **L652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Clause %s is not allowed if clause %s appears on the %s directive"_err_en_US,`.
  **L652 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Clause %s is not allowed if clause %s appears on the %s directive"_err_en_US,`。
- **L653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(clause).str()),`.
  **L653 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(clause).str()),`。
- **L654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(cl).str()),`.
  **L654 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(cl).str()),`。
- **L655 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L655 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L656 EN**: Closes the current lexical scope or compound statement.
  **L656 CN**: 结束当前词法作用域或复合语句块。
- **L657 EN**: Closes the current lexical scope or compound statement.
  **L657 CN**: 结束当前词法作用域或复合语句块。
- **L658 EN**: Closes the current lexical scope or compound statement.
  **L658 CN**: 结束当前词法作用域或复合语句块。
- **L659 EN**: Blank line separating nearby declarations or logic blocks.
  **L659 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L660 EN**: Comment explains nearby logic, intent, or metadata: `Check the value of the clause is a constant positive integer.`.
  **L660 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check the value of the clause is a constant positive integer.`。

### Lines 661-680

````cpp
template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
void DirectiveStructureChecker<D, C, PC,
    ClauseEnumSize>::RequiresConstantPositiveParameter(const C &clause,
    const parser::ScalarIntConstantExpr &i) {
  if (const auto v{GetIntValue(i)}) {
    if (*v <= 0) {
      context_.Say(GetContext().clauseSource,
          "The parameter of the %s clause must be "
          "a constant positive integer expression"_err_en_US,
          parser::ToUpperCaseLetters(getClauseName(clause).str()));
    }
  }
}

// Check the value of the clause is a constant positive parameter.
template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
void DirectiveStructureChecker<D, C, PC,
    ClauseEnumSize>::OptionalConstantPositiveParameter(const C &clause,
    const std::optional<parser::ScalarIntConstantExpr> &o) {
  if (o != std::nullopt) {
````
- **L661 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L661 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void DirectiveStructureChecker<D, C, PC,`.
  **L662 CN**: 继续一个多行参数列表、初始化器或聚合项：`void DirectiveStructureChecker<D, C, PC,`。
- **L663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClauseEnumSize>::RequiresConstantPositiveParameter(const C &clause,`.
  **L663 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClauseEnumSize>::RequiresConstantPositiveParameter(const C &clause,`。
- **L664 EN**: Continues the surrounding expression or declaration: `const parser::ScalarIntConstantExpr &i) {`.
  **L664 CN**: 继续构造周围的表达式或声明：`const parser::ScalarIntConstantExpr &i) {`。
- **L665 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L665 CN**: 开始 `if` 控制流语句并计算其条件。
- **L666 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L666 CN**: 开始 `if` 控制流语句并计算其条件。
- **L667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L667 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L668 EN**: Continues the surrounding expression or declaration: `"The parameter of the %s clause must be "`.
  **L668 CN**: 继续构造周围的表达式或声明：`"The parameter of the %s clause must be "`。
- **L669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"a constant positive integer expression"_err_en_US,`.
  **L669 CN**: 继续一个多行参数列表、初始化器或聚合项：`"a constant positive integer expression"_err_en_US,`。
- **L670 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L670 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L671 EN**: Closes the current lexical scope or compound statement.
  **L671 CN**: 结束当前词法作用域或复合语句块。
- **L672 EN**: Closes the current lexical scope or compound statement.
  **L672 CN**: 结束当前词法作用域或复合语句块。
- **L673 EN**: Closes the current lexical scope or compound statement.
  **L673 CN**: 结束当前词法作用域或复合语句块。
- **L674 EN**: Blank line separating nearby declarations or logic blocks.
  **L674 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L675 EN**: Comment explains nearby logic, intent, or metadata: `Check the value of the clause is a constant positive parameter.`.
  **L675 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check the value of the clause is a constant positive parameter.`。
- **L676 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L676 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void DirectiveStructureChecker<D, C, PC,`.
  **L677 CN**: 继续一个多行参数列表、初始化器或聚合项：`void DirectiveStructureChecker<D, C, PC,`。
- **L678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClauseEnumSize>::OptionalConstantPositiveParameter(const C &clause,`.
  **L678 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClauseEnumSize>::OptionalConstantPositiveParameter(const C &clause,`。
- **L679 EN**: Continues the surrounding expression or declaration: `const std::optional<parser::ScalarIntConstantExpr> &o) {`.
  **L679 CN**: 继续构造周围的表达式或声明：`const std::optional<parser::ScalarIntConstantExpr> &o) {`。
- **L680 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L680 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 681-700

````cpp
    RequiresConstantPositiveParameter(clause, o.value());
  }
}

template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
void DirectiveStructureChecker<D, C, PC, ClauseEnumSize>::SayNotMatching(
    const parser::CharBlock &beginSource, const parser::CharBlock &endSource) {
  context_
      .Say(endSource, "Unmatched %s directive"_err_en_US,
          parser::ToUpperCaseLetters(endSource.ToString()))
      .Attach(beginSource, "Does not match directive"_en_US);
}

// Check the value of the clause is a positive parameter.
template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>
void DirectiveStructureChecker<D, C, PC,
    ClauseEnumSize>::RequiresPositiveParameter(const C &clause,
    const parser::ScalarIntExpr &i, llvm::StringRef paramName, bool allowZero) {
  if (const auto v{GetIntValue(i)}) {
    if (*v < (allowZero ? 0 : 1)) {
````
- **L681 EN**: Executes a call or declaration centered on `RequiresConstantPositiveParameter`.
  **L681 CN**: 执行以 `RequiresConstantPositiveParameter` 为核心的调用或声明。
- **L682 EN**: Closes the current lexical scope or compound statement.
  **L682 CN**: 结束当前词法作用域或复合语句块。
- **L683 EN**: Closes the current lexical scope or compound statement.
  **L683 CN**: 结束当前词法作用域或复合语句块。
- **L684 EN**: Blank line separating nearby declarations or logic blocks.
  **L684 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L685 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L685 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L686 EN**: Continues logic associated with callable symbol `SayNotMatching`.
  **L686 CN**: 继续与可调用符号 `SayNotMatching` 相关的逻辑。
- **L687 EN**: Continues the surrounding expression or declaration: `const parser::CharBlock &beginSource, const parser::CharBlock &endSource) {`.
  **L687 CN**: 继续构造周围的表达式或声明：`const parser::CharBlock &beginSource, const parser::CharBlock &endSource) {`。
- **L688 EN**: Continues the surrounding expression or declaration: `context_`.
  **L688 CN**: 继续构造周围的表达式或声明：`context_`。
- **L689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(endSource, "Unmatched %s directive"_err_en_US,`.
  **L689 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(endSource, "Unmatched %s directive"_err_en_US,`。
- **L690 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L690 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L691 EN**: Executes a call or declaration centered on `.Attach`.
  **L691 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L692 EN**: Closes the current lexical scope or compound statement.
  **L692 CN**: 结束当前词法作用域或复合语句块。
- **L693 EN**: Blank line separating nearby declarations or logic blocks.
  **L693 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L694 EN**: Comment explains nearby logic, intent, or metadata: `Check the value of the clause is a positive parameter.`.
  **L694 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check the value of the clause is a positive parameter.`。
- **L695 EN**: Introduces template parameters or specialization context: `template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`.
  **L695 CN**: 为后续声明引入模板参数或特化上下文：`template <typename D, typename C, typename PC, std::size_t ClauseEnumSize>`。
- **L696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void DirectiveStructureChecker<D, C, PC,`.
  **L696 CN**: 继续一个多行参数列表、初始化器或聚合项：`void DirectiveStructureChecker<D, C, PC,`。
- **L697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClauseEnumSize>::RequiresPositiveParameter(const C &clause,`.
  **L697 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClauseEnumSize>::RequiresPositiveParameter(const C &clause,`。
- **L698 EN**: Continues the surrounding expression or declaration: `const parser::ScalarIntExpr &i, llvm::StringRef paramName, bool allowZero) {`.
  **L698 CN**: 继续构造周围的表达式或声明：`const parser::ScalarIntExpr &i, llvm::StringRef paramName, bool allowZero) {`。
- **L699 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L699 CN**: 开始 `if` 控制流语句并计算其条件。
- **L700 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L700 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 701-712

````cpp
      context_.Say(GetContext().clauseSource,
          "The %s of the %s clause must be "
          "a positive integer expression"_err_en_US,
          paramName.str(),
          parser::ToUpperCaseLetters(getClauseName(clause).str()));
    }
  }
}

} // namespace Fortran::semantics

#endif // FORTRAN_SEMANTICS_CHECK_DIRECTIVE_STRUCTURE_H_
````
- **L701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L701 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L702 EN**: Continues the surrounding expression or declaration: `"The %s of the %s clause must be "`.
  **L702 CN**: 继续构造周围的表达式或声明：`"The %s of the %s clause must be "`。
- **L703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"a positive integer expression"_err_en_US,`.
  **L703 CN**: 继续一个多行参数列表、初始化器或聚合项：`"a positive integer expression"_err_en_US,`。
- **L704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `paramName.str(),`.
  **L704 CN**: 继续一个多行参数列表、初始化器或聚合项：`paramName.str(),`。
- **L705 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L705 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L706 EN**: Closes the current lexical scope or compound statement.
  **L706 CN**: 结束当前词法作用域或复合语句块。
- **L707 EN**: Closes the current lexical scope or compound statement.
  **L707 CN**: 结束当前词法作用域或复合语句块。
- **L708 EN**: Closes the current lexical scope or compound statement.
  **L708 CN**: 结束当前词法作用域或复合语句块。
- **L709 EN**: Blank line separating nearby declarations or logic blocks.
  **L709 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L710 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L710 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。
- **L711 EN**: Blank line separating nearby declarations or logic blocks.
  **L711 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L712 EN**: Closes the current preprocessor conditional block.
  **L712 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Fortran parse tree handling / Fortran 语法树处理**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Semantic-context management / 语义上下文管理**
- **OpenMP handling / OpenMP 处理**
- **OpenACC handling / OpenACC 处理**
- **Rule-checking workflow / 规则检查流程**

## Dependencies / 依赖关系

- `flang/Common/enum-set.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Semantics/semantics.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `llvm/ADT/iterator_range.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
- `set`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
- `unordered_map`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
