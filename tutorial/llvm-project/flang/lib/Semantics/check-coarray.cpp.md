# check-coarray.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-coarray.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements semantic checking, symbol reasoning, and language-rule enforcement for check coarray.
- **Purpose (CN)**: 实现 check coarray 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===-- lib/Semantics/check-coarray.cpp -----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "check-coarray.h"
#include "definable.h"
#include "flang/Common/indirection.h"
#include "flang/Evaluate/check-expression.h"
#include "flang/Evaluate/expression.h"
#include "flang/Parser/message.h"
#include "flang/Parser/parse-tree.h"
#include "flang/Parser/tools.h"
#include "flang/Semantics/expression.h"
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
- **L9 EN**: Includes "check-coarray.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "check-coarray.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "definable.h" to access local declarations paired with this implementation.
  **L10 CN**: 引入 "definable.h" 以使用与该实现配套的本地声明。
- **L11 EN**: Includes "flang/Common/indirection.h" to access shared Flang utility infrastructure.
  **L11 CN**: 引入 "flang/Common/indirection.h" 以使用Flang 共享工具基础设施。
- **L12 EN**: Includes "flang/Evaluate/check-expression.h" to access Fortran constant-folding and evaluation facilities.
  **L12 CN**: 引入 "flang/Evaluate/check-expression.h" 以使用Fortran 常量折叠与求值能力。
- **L13 EN**: Includes "flang/Evaluate/expression.h" to access Fortran constant-folding and evaluation facilities.
  **L13 CN**: 引入 "flang/Evaluate/expression.h" 以使用Fortran 常量折叠与求值能力。
- **L14 EN**: Includes "flang/Parser/message.h" to access parse-tree, token, or source representation support.
  **L14 CN**: 引入 "flang/Parser/message.h" 以使用语法树、词法单元或源码表示支持。
- **L15 EN**: Includes "flang/Parser/parse-tree.h" to access parse-tree, token, or source representation support.
  **L15 CN**: 引入 "flang/Parser/parse-tree.h" 以使用语法树、词法单元或源码表示支持。
- **L16 EN**: Includes "flang/Parser/tools.h" to access parse-tree, token, or source representation support.
  **L16 CN**: 引入 "flang/Parser/tools.h" 以使用语法树、词法单元或源码表示支持。
- **L17 EN**: Includes "flang/Semantics/expression.h" to access Fortran semantic analysis, symbol, and type information.
  **L17 CN**: 引入 "flang/Semantics/expression.h" 以使用Fortran 语义分析、符号与类型信息。
- **L18 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L18 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Opens namespace scope `Fortran::semantics`.
  **L20 CN**: 打开命名空间作用域 `Fortran::semantics`。

### Lines 21-40

````cpp

class CriticalBodyEnforce {
public:
  CriticalBodyEnforce(
      SemanticsContext &context, parser::CharBlock criticalSourcePosition)
      : context_{context}, criticalSourcePosition_{criticalSourcePosition} {}
  std::set<parser::Label> labels() { return labels_; }
  template <typename T> bool Pre(const T &) { return true; }
  template <typename T> void Post(const T &) {}

  template <typename T> bool Pre(const parser::Statement<T> &statement) {
    currentStatementSourcePosition_ = statement.source;
    if (statement.label.has_value()) {
      labels_.insert(*statement.label);
    }
    return true;
  }

  // C1118
  void Post(const parser::ReturnStmt &) {
````
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Declares class `CriticalBodyEnforce`.
  **L22 CN**: 声明 class `CriticalBodyEnforce`。
- **L23 EN**: Sets the following members to `public` access.
  **L23 CN**: 将后续成员的访问级别设为 `public`。
- **L24 EN**: Continues logic associated with callable symbol `CriticalBodyEnforce`.
  **L24 CN**: 继续与可调用符号 `CriticalBodyEnforce` 相关的逻辑。
- **L25 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, parser::CharBlock criticalSourcePosition)`.
  **L25 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, parser::CharBlock criticalSourcePosition)`。
- **L26 EN**: Continues the surrounding expression or declaration: `: context_{context}, criticalSourcePosition_{criticalSourcePosition} {}`.
  **L26 CN**: 继续构造周围的表达式或声明：`: context_{context}, criticalSourcePosition_{criticalSourcePosition} {}`。
- **L27 EN**: Continues logic associated with callable symbol `labels`.
  **L27 CN**: 继续与可调用符号 `labels` 相关的逻辑。
- **L28 EN**: Introduces template parameters or specialization context: `template <typename T> bool Pre(const T &) { return true; }`.
  **L28 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> bool Pre(const T &) { return true; }`。
- **L29 EN**: Introduces template parameters or specialization context: `template <typename T> void Post(const T &) {}`.
  **L29 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> void Post(const T &) {}`。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L31 EN**: Introduces template parameters or specialization context: `template <typename T> bool Pre(const parser::Statement<T> &statement) {`.
  **L31 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> bool Pre(const parser::Statement<T> &statement) {`。
- **L32 EN**: Executes a standalone statement or declaration: `currentStatementSourcePosition_ = statement.source;`.
  **L32 CN**: 执行一条独立语句或声明：`currentStatementSourcePosition_ = statement.source;`。
- **L33 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L33 CN**: 开始 `if` 控制流语句并计算其条件。
- **L34 EN**: Executes a call or declaration centered on `labels_.insert`.
  **L34 CN**: 执行以 `labels_.insert` 为核心的调用或声明。
- **L35 EN**: Closes the current lexical scope or compound statement.
  **L35 CN**: 结束当前词法作用域或复合语句块。
- **L36 EN**: Returns from the current function with `true`.
  **L36 CN**: 以 `true` 从当前函数返回。
- **L37 EN**: Closes the current lexical scope or compound statement.
  **L37 CN**: 结束当前词法作用域或复合语句块。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L39 EN**: Comment explains nearby logic, intent, or metadata: `C1118`.
  **L39 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1118`。
- **L40 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::ReturnStmt &) {`.
  **L40 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::ReturnStmt &) {`。

### Lines 41-60

````cpp
    context_
        .Say(currentStatementSourcePosition_,
            "RETURN statement is not allowed in a CRITICAL construct"_err_en_US)
        .Attach(criticalSourcePosition_, GetEnclosingMsg());
  }
  void Post(const parser::ExecutableConstruct &construct) {
    if (IsImageControlStmt(construct)) {
      context_
          .Say(currentStatementSourcePosition_,
              "An image control statement is not allowed in a CRITICAL"
              " construct"_err_en_US)
          .Attach(criticalSourcePosition_, GetEnclosingMsg());
    }
  }

private:
  parser::MessageFixedText GetEnclosingMsg() {
    return "Enclosing CRITICAL statement"_en_US;
  }

````
- **L41 EN**: Continues the surrounding expression or declaration: `context_`.
  **L41 CN**: 继续构造周围的表达式或声明：`context_`。
- **L42 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(currentStatementSourcePosition_,`.
  **L42 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(currentStatementSourcePosition_,`。
- **L43 EN**: Continues the surrounding expression or declaration: `"RETURN statement is not allowed in a CRITICAL construct"_err_en_US)`.
  **L43 CN**: 继续构造周围的表达式或声明：`"RETURN statement is not allowed in a CRITICAL construct"_err_en_US)`。
- **L44 EN**: Executes a call or declaration centered on `.Attach`.
  **L44 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L45 EN**: Closes the current lexical scope or compound statement.
  **L45 CN**: 结束当前词法作用域或复合语句块。
- **L46 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::ExecutableConstruct &construct) {`.
  **L46 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::ExecutableConstruct &construct) {`。
- **L47 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L47 CN**: 开始 `if` 控制流语句并计算其条件。
- **L48 EN**: Continues the surrounding expression or declaration: `context_`.
  **L48 CN**: 继续构造周围的表达式或声明：`context_`。
- **L49 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(currentStatementSourcePosition_,`.
  **L49 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(currentStatementSourcePosition_,`。
- **L50 EN**: Continues the surrounding expression or declaration: `"An image control statement is not allowed in a CRITICAL"`.
  **L50 CN**: 继续构造周围的表达式或声明：`"An image control statement is not allowed in a CRITICAL"`。
- **L51 EN**: Continues the surrounding expression or declaration: `" construct"_err_en_US)`.
  **L51 CN**: 继续构造周围的表达式或声明：`" construct"_err_en_US)`。
- **L52 EN**: Executes a call or declaration centered on `.Attach`.
  **L52 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L53 EN**: Closes the current lexical scope or compound statement.
  **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Closes the current lexical scope or compound statement.
  **L54 CN**: 结束当前词法作用域或复合语句块。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L56 EN**: Sets the following members to `private` access.
  **L56 CN**: 将后续成员的访问级别设为 `private`。
- **L57 EN**: Starts a function, method, lambda, or structured scope: `parser::MessageFixedText GetEnclosingMsg() {`.
  **L57 CN**: 开始一个函数、方法、lambda 或结构化作用域：`parser::MessageFixedText GetEnclosingMsg() {`。
- **L58 EN**: Returns from the current function with `"Enclosing CRITICAL statement"_en_US`.
  **L58 CN**: 以 `"Enclosing CRITICAL statement"_en_US` 从当前函数返回。
- **L59 EN**: Closes the current lexical scope or compound statement.
  **L59 CN**: 结束当前词法作用域或复合语句块。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 61-80

````cpp
  SemanticsContext &context_;
  std::set<parser::Label> labels_;
  parser::CharBlock currentStatementSourcePosition_;
  parser::CharBlock criticalSourcePosition_;
};

class ChangeTeamBodyEnforce {
public:
  ChangeTeamBodyEnforce(
      SemanticsContext &context, parser::CharBlock changeTeamSourcePosition)
      : context_{context}, changeTeamSourcePosition_{changeTeamSourcePosition} {
  }
  std::set<parser::Label> labels() { return labels_; }
  template <typename T> bool Pre(const T &) { return true; }
  template <typename T> void Post(const T &) {}

  template <typename T> bool Pre(const parser::Statement<T> &statement) {
    currentStatementSourcePosition_ = statement.source;
    if (statement.label.has_value()) {
      labels_.insert(*statement.label);
````
- **L61 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L61 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。
- **L62 EN**: Executes a standalone statement or declaration: `std::set<parser::Label> labels_;`.
  **L62 CN**: 执行一条独立语句或声明：`std::set<parser::Label> labels_;`。
- **L63 EN**: Executes a standalone statement or declaration: `parser::CharBlock currentStatementSourcePosition_;`.
  **L63 CN**: 执行一条独立语句或声明：`parser::CharBlock currentStatementSourcePosition_;`。
- **L64 EN**: Executes a standalone statement or declaration: `parser::CharBlock criticalSourcePosition_;`.
  **L64 CN**: 执行一条独立语句或声明：`parser::CharBlock criticalSourcePosition_;`。
- **L65 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L65 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L67 EN**: Declares class `ChangeTeamBodyEnforce`.
  **L67 CN**: 声明 class `ChangeTeamBodyEnforce`。
- **L68 EN**: Sets the following members to `public` access.
  **L68 CN**: 将后续成员的访问级别设为 `public`。
- **L69 EN**: Continues logic associated with callable symbol `ChangeTeamBodyEnforce`.
  **L69 CN**: 继续与可调用符号 `ChangeTeamBodyEnforce` 相关的逻辑。
- **L70 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, parser::CharBlock changeTeamSourcePosition)`.
  **L70 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, parser::CharBlock changeTeamSourcePosition)`。
- **L71 EN**: Continues the surrounding expression or declaration: `: context_{context}, changeTeamSourcePosition_{changeTeamSourcePosition} {`.
  **L71 CN**: 继续构造周围的表达式或声明：`: context_{context}, changeTeamSourcePosition_{changeTeamSourcePosition} {`。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。
- **L73 EN**: Continues logic associated with callable symbol `labels`.
  **L73 CN**: 继续与可调用符号 `labels` 相关的逻辑。
- **L74 EN**: Introduces template parameters or specialization context: `template <typename T> bool Pre(const T &) { return true; }`.
  **L74 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> bool Pre(const T &) { return true; }`。
- **L75 EN**: Introduces template parameters or specialization context: `template <typename T> void Post(const T &) {}`.
  **L75 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> void Post(const T &) {}`。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L77 EN**: Introduces template parameters or specialization context: `template <typename T> bool Pre(const parser::Statement<T> &statement) {`.
  **L77 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> bool Pre(const parser::Statement<T> &statement) {`。
- **L78 EN**: Executes a standalone statement or declaration: `currentStatementSourcePosition_ = statement.source;`.
  **L78 CN**: 执行一条独立语句或声明：`currentStatementSourcePosition_ = statement.source;`。
- **L79 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L79 CN**: 开始 `if` 控制流语句并计算其条件。
- **L80 EN**: Executes a call or declaration centered on `labels_.insert`.
  **L80 CN**: 执行以 `labels_.insert` 为核心的调用或声明。

### Lines 81-100

````cpp
    }
    return true;
  }

  void Post(const parser::ReturnStmt &) {
    context_
        .Say(currentStatementSourcePosition_,
            "RETURN statement is not allowed in a CHANGE TEAM construct"_err_en_US)
        .Attach(
            changeTeamSourcePosition_, "Enclosing CHANGE TEAM construct"_en_US);
  }

private:
  SemanticsContext &context_;
  std::set<parser::Label> labels_;
  parser::CharBlock currentStatementSourcePosition_;
  parser::CharBlock changeTeamSourcePosition_;
};

template <typename T>
````
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Returns from the current function with `true`.
  **L82 CN**: 以 `true` 从当前函数返回。
- **L83 EN**: Closes the current lexical scope or compound statement.
  **L83 CN**: 结束当前词法作用域或复合语句块。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L85 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::ReturnStmt &) {`.
  **L85 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::ReturnStmt &) {`。
- **L86 EN**: Continues the surrounding expression or declaration: `context_`.
  **L86 CN**: 继续构造周围的表达式或声明：`context_`。
- **L87 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(currentStatementSourcePosition_,`.
  **L87 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(currentStatementSourcePosition_,`。
- **L88 EN**: Continues the surrounding expression or declaration: `"RETURN statement is not allowed in a CHANGE TEAM construct"_err_en_US)`.
  **L88 CN**: 继续构造周围的表达式或声明：`"RETURN statement is not allowed in a CHANGE TEAM construct"_err_en_US)`。
- **L89 EN**: Continues logic associated with callable symbol `Attach`.
  **L89 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L90 EN**: Executes a standalone statement or declaration: `changeTeamSourcePosition_, "Enclosing CHANGE TEAM construct"_en_US);`.
  **L90 CN**: 执行一条独立语句或声明：`changeTeamSourcePosition_, "Enclosing CHANGE TEAM construct"_en_US);`。
- **L91 EN**: Closes the current lexical scope or compound statement.
  **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L93 EN**: Sets the following members to `private` access.
  **L93 CN**: 将后续成员的访问级别设为 `private`。
- **L94 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L94 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。
- **L95 EN**: Executes a standalone statement or declaration: `std::set<parser::Label> labels_;`.
  **L95 CN**: 执行一条独立语句或声明：`std::set<parser::Label> labels_;`。
- **L96 EN**: Executes a standalone statement or declaration: `parser::CharBlock currentStatementSourcePosition_;`.
  **L96 CN**: 执行一条独立语句或声明：`parser::CharBlock currentStatementSourcePosition_;`。
- **L97 EN**: Executes a standalone statement or declaration: `parser::CharBlock changeTeamSourcePosition_;`.
  **L97 CN**: 执行一条独立语句或声明：`parser::CharBlock changeTeamSourcePosition_;`。
- **L98 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L98 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L99 EN**: Blank line separating nearby declarations or logic blocks.
  **L99 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L100 EN**: Introduces template parameters or specialization context: `template <typename T>`.
  **L100 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>`。

### Lines 101-120

````cpp
static void CheckTeamType(
    SemanticsContext &context, const T &x, bool mustBeVariable = false) {
  if (const auto *expr{GetExpr(context, x)}) {
    NoteUsedSymbols(context, *expr);
    if (!IsTeamType(evaluate::GetDerivedTypeSpec(expr->GetType()))) {
      context.Say(parser::FindSourceLocation(x), // C1114
          "Team value must be of type TEAM_TYPE from module ISO_FORTRAN_ENV"_err_en_US);
    } else if (mustBeVariable && !IsVariable(*expr)) {
      context.Say(parser::FindSourceLocation(x),
          "Team must be a variable in this context"_err_en_US);
    }
  }
}

static void CheckTeamStat(
    SemanticsContext &context, const parser::ImageSelectorSpec::Stat &stat) {
  const auto &var{parser::UnwrapRef<parser::Variable>(stat)};
  if (parser::GetCoindexedNamedObject(var)) {
    context.Say(parser::FindSourceLocation(var), // C931
        "Image selector STAT variable must not be a coindexed "
````
- **L101 EN**: Continues logic associated with callable symbol `CheckTeamType`.
  **L101 CN**: 继续与可调用符号 `CheckTeamType` 相关的逻辑。
- **L102 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const T &x, bool mustBeVariable = false) {`.
  **L102 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const T &x, bool mustBeVariable = false) {`。
- **L103 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L103 CN**: 开始 `if` 控制流语句并计算其条件。
- **L104 EN**: Executes a call or declaration centered on `NoteUsedSymbols`.
  **L104 CN**: 执行以 `NoteUsedSymbols` 为核心的调用或声明。
- **L105 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L105 CN**: 开始 `if` 控制流语句并计算其条件。
- **L106 EN**: Continues logic associated with callable symbol `Say`.
  **L106 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L107 EN**: Executes a standalone statement or declaration: `"Team value must be of type TEAM_TYPE from module ISO_FORTRAN_ENV"_err_en_US);`.
  **L107 CN**: 执行一条独立语句或声明：`"Team value must be of type TEAM_TYPE from module ISO_FORTRAN_ENV"_err_en_US);`。
- **L108 EN**: Transitions from the previous branch into an `else if` condition.
  **L108 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(parser::FindSourceLocation(x),`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(parser::FindSourceLocation(x),`。
- **L110 EN**: Executes a standalone statement or declaration: `"Team must be a variable in this context"_err_en_US);`.
  **L110 CN**: 执行一条独立语句或声明：`"Team must be a variable in this context"_err_en_US);`。
- **L111 EN**: Closes the current lexical scope or compound statement.
  **L111 CN**: 结束当前词法作用域或复合语句块。
- **L112 EN**: Closes the current lexical scope or compound statement.
  **L112 CN**: 结束当前词法作用域或复合语句块。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L115 EN**: Continues logic associated with callable symbol `CheckTeamStat`.
  **L115 CN**: 继续与可调用符号 `CheckTeamStat` 相关的逻辑。
- **L116 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::ImageSelectorSpec::Stat &stat) {`.
  **L116 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::ImageSelectorSpec::Stat &stat) {`。
- **L117 EN**: Executes a call or declaration centered on `&var{parser::UnwrapRef<parser::Variable>`.
  **L117 CN**: 执行以 `&var{parser::UnwrapRef<parser::Variable>` 为核心的调用或声明。
- **L118 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L118 CN**: 开始 `if` 控制流语句并计算其条件。
- **L119 EN**: Continues logic associated with callable symbol `Say`.
  **L119 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L120 EN**: Continues the surrounding expression or declaration: `"Image selector STAT variable must not be a coindexed "`.
  **L120 CN**: 继续构造周围的表达式或声明：`"Image selector STAT variable must not be a coindexed "`。

### Lines 121-140

````cpp
        "object"_err_en_US);
  }
}

static void CheckCoindexedStatOrErrmsg(SemanticsContext &context,
    const parser::StatOrErrmsg &statOrErrmsg, const std::string &listName) {
  auto CoindexedCheck{[&](const auto &statOrErrmsg) {
    if (const auto *expr{GetExpr(context, statOrErrmsg)}) {
      if (ExtractCoarrayRef(expr)) {
        context.Say(parser::FindSourceLocation(statOrErrmsg), // C1173
            "The stat-variable or errmsg-variable in a %s may not be a coindexed object"_err_en_US,
            listName);
      }
    }
  }};
  Fortran::common::visit(CoindexedCheck, statOrErrmsg.u);
}

static void CheckSyncStat(SemanticsContext &context,
    const parser::StatOrErrmsg &statOrErrmsg, bool &gotStat, bool &gotMsg) {
````
- **L121 EN**: Executes a standalone statement or declaration: `"object"_err_en_US);`.
  **L121 CN**: 执行一条独立语句或声明：`"object"_err_en_US);`。
- **L122 EN**: Closes the current lexical scope or compound statement.
  **L122 CN**: 结束当前词法作用域或复合语句块。
- **L123 EN**: Closes the current lexical scope or compound statement.
  **L123 CN**: 结束当前词法作用域或复合语句块。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckCoindexedStatOrErrmsg(SemanticsContext &context,`.
  **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckCoindexedStatOrErrmsg(SemanticsContext &context,`。
- **L126 EN**: Continues the surrounding expression or declaration: `const parser::StatOrErrmsg &statOrErrmsg, const std::string &listName) {`.
  **L126 CN**: 继续构造周围的表达式或声明：`const parser::StatOrErrmsg &statOrErrmsg, const std::string &listName) {`。
- **L127 EN**: Starts a function, method, lambda, or structured scope: `auto CoindexedCheck{[&](const auto &statOrErrmsg) {`.
  **L127 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto CoindexedCheck{[&](const auto &statOrErrmsg) {`。
- **L128 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L128 CN**: 开始 `if` 控制流语句并计算其条件。
- **L129 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L129 CN**: 开始 `if` 控制流语句并计算其条件。
- **L130 EN**: Continues logic associated with callable symbol `Say`.
  **L130 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The stat-variable or errmsg-variable in a %s may not be a coindexed object"_err_en_US,`.
  **L131 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The stat-variable or errmsg-variable in a %s may not be a coindexed object"_err_en_US,`。
- **L132 EN**: Executes a standalone statement or declaration: `listName);`.
  **L132 CN**: 执行一条独立语句或声明：`listName);`。
- **L133 EN**: Closes the current lexical scope or compound statement.
  **L133 CN**: 结束当前词法作用域或复合语句块。
- **L134 EN**: Closes the current lexical scope or compound statement.
  **L134 CN**: 结束当前词法作用域或复合语句块。
- **L135 EN**: Executes a standalone statement or declaration: `}};`.
  **L135 CN**: 执行一条独立语句或声明：`}};`。
- **L136 EN**: Executes a call or declaration centered on `Fortran::common::visit`.
  **L136 CN**: 执行以 `Fortran::common::visit` 为核心的调用或声明。
- **L137 EN**: Closes the current lexical scope or compound statement.
  **L137 CN**: 结束当前词法作用域或复合语句块。
- **L138 EN**: Blank line separating nearby declarations or logic blocks.
  **L138 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckSyncStat(SemanticsContext &context,`.
  **L139 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckSyncStat(SemanticsContext &context,`。
- **L140 EN**: Continues the surrounding expression or declaration: `const parser::StatOrErrmsg &statOrErrmsg, bool &gotStat, bool &gotMsg) {`.
  **L140 CN**: 继续构造周围的表达式或声明：`const parser::StatOrErrmsg &statOrErrmsg, bool &gotStat, bool &gotMsg) {`。

### Lines 141-160

````cpp
  common::visit(
      common::visitors{
          [&](const parser::StatVariable &stat) {
            if (gotStat) {
              context.Say( // C1172
                  "The stat-variable in a sync-stat-list may not be repeated"_err_en_US);
            }
            gotStat = true;
          },
          [&](const parser::MsgVariable &var) {
            WarnOnDeferredLengthCharacterScalar(context, GetExpr(context, var),
                parser::UnwrapRef<parser::Variable>(var).GetSource(),
                "ERRMSG=");
            if (gotMsg) {
              context.Say( // C1172
                  "The errmsg-variable in a sync-stat-list may not be repeated"_err_en_US);
            }
            gotMsg = true;
          },
      },
````
- **L141 EN**: Continues logic associated with callable symbol `visit`.
  **L141 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L142 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L142 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L143 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::StatVariable &stat) {`.
  **L143 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::StatVariable &stat) {`。
- **L144 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L144 CN**: 开始 `if` 控制流语句并计算其条件。
- **L145 EN**: Continues logic associated with callable symbol `Say`.
  **L145 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L146 EN**: Executes a standalone statement or declaration: `"The stat-variable in a sync-stat-list may not be repeated"_err_en_US);`.
  **L146 CN**: 执行一条独立语句或声明：`"The stat-variable in a sync-stat-list may not be repeated"_err_en_US);`。
- **L147 EN**: Closes the current lexical scope or compound statement.
  **L147 CN**: 结束当前词法作用域或复合语句块。
- **L148 EN**: Executes a standalone statement or declaration: `gotStat = true;`.
  **L148 CN**: 执行一条独立语句或声明：`gotStat = true;`。
- **L149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L149 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L150 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::MsgVariable &var) {`.
  **L150 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::MsgVariable &var) {`。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WarnOnDeferredLengthCharacterScalar(context, GetExpr(context, var),`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`WarnOnDeferredLengthCharacterScalar(context, GetExpr(context, var),`。
- **L152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::UnwrapRef<parser::Variable>(var).GetSource(),`.
  **L152 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::UnwrapRef<parser::Variable>(var).GetSource(),`。
- **L153 EN**: Executes a standalone statement or declaration: `"ERRMSG=");`.
  **L153 CN**: 执行一条独立语句或声明：`"ERRMSG=");`。
- **L154 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L154 CN**: 开始 `if` 控制流语句并计算其条件。
- **L155 EN**: Continues logic associated with callable symbol `Say`.
  **L155 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L156 EN**: Executes a standalone statement or declaration: `"The errmsg-variable in a sync-stat-list may not be repeated"_err_en_US);`.
  **L156 CN**: 执行一条独立语句或声明：`"The errmsg-variable in a sync-stat-list may not be repeated"_err_en_US);`。
- **L157 EN**: Closes the current lexical scope or compound statement.
  **L157 CN**: 结束当前词法作用域或复合语句块。
- **L158 EN**: Executes a standalone statement or declaration: `gotMsg = true;`.
  **L158 CN**: 执行一条独立语句或声明：`gotMsg = true;`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。

### Lines 161-180

````cpp
      statOrErrmsg.u);

  CheckCoindexedStatOrErrmsg(context, statOrErrmsg, "sync-stat-list");
}

static void CheckSyncStatList(
    SemanticsContext &context, const std::list<parser::StatOrErrmsg> &list) {
  bool gotStat{false}, gotMsg{false};
  for (const parser::StatOrErrmsg &statOrErrmsg : list) {
    CheckSyncStat(context, statOrErrmsg, gotStat, gotMsg);
  }
}

static void CheckEventVariable(
    SemanticsContext &context, const parser::EventVariable &eventVar) {
  if (const auto *expr{GetExpr(context, eventVar)}) {
    if (!IsEventType(evaluate::GetDerivedTypeSpec(expr->GetType()))) { // C1176
      context.Say(parser::FindSourceLocation(eventVar),
          "The event-variable must be of type EVENT_TYPE from module ISO_FORTRAN_ENV"_err_en_US);
    }
````
- **L161 EN**: Executes a standalone statement or declaration: `statOrErrmsg.u);`.
  **L161 CN**: 执行一条独立语句或声明：`statOrErrmsg.u);`。
- **L162 EN**: Blank line separating nearby declarations or logic blocks.
  **L162 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L163 EN**: Executes a call or declaration centered on `CheckCoindexedStatOrErrmsg`.
  **L163 CN**: 执行以 `CheckCoindexedStatOrErrmsg` 为核心的调用或声明。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L166 EN**: Continues logic associated with callable symbol `CheckSyncStatList`.
  **L166 CN**: 继续与可调用符号 `CheckSyncStatList` 相关的逻辑。
- **L167 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const std::list<parser::StatOrErrmsg> &list) {`.
  **L167 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const std::list<parser::StatOrErrmsg> &list) {`。
- **L168 EN**: Executes a standalone statement or declaration: `bool gotStat{false}, gotMsg{false};`.
  **L168 CN**: 执行一条独立语句或声明：`bool gotStat{false}, gotMsg{false};`。
- **L169 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L169 CN**: 开始 `for` 控制流语句并计算其条件。
- **L170 EN**: Executes a call or declaration centered on `CheckSyncStat`.
  **L170 CN**: 执行以 `CheckSyncStat` 为核心的调用或声明。
- **L171 EN**: Closes the current lexical scope or compound statement.
  **L171 CN**: 结束当前词法作用域或复合语句块。
- **L172 EN**: Closes the current lexical scope or compound statement.
  **L172 CN**: 结束当前词法作用域或复合语句块。
- **L173 EN**: Blank line separating nearby declarations or logic blocks.
  **L173 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L174 EN**: Continues logic associated with callable symbol `CheckEventVariable`.
  **L174 CN**: 继续与可调用符号 `CheckEventVariable` 相关的逻辑。
- **L175 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::EventVariable &eventVar) {`.
  **L175 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::EventVariable &eventVar) {`。
- **L176 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L176 CN**: 开始 `if` 控制流语句并计算其条件。
- **L177 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L177 CN**: 开始 `if` 控制流语句并计算其条件。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(parser::FindSourceLocation(eventVar),`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(parser::FindSourceLocation(eventVar),`。
- **L179 EN**: Executes a standalone statement or declaration: `"The event-variable must be of type EVENT_TYPE from module ISO_FORTRAN_ENV"_err_en_US);`.
  **L179 CN**: 执行一条独立语句或声明：`"The event-variable must be of type EVENT_TYPE from module ISO_FORTRAN_ENV"_err_en_US);`。
- **L180 EN**: Closes the current lexical scope or compound statement.
  **L180 CN**: 结束当前词法作用域或复合语句块。

### Lines 181-200

````cpp
  }
}

void CoarrayChecker::Leave(const parser::ChangeTeamStmt &x) {
  CheckNamesAreDistinct(std::get<std::list<parser::CoarrayAssociation>>(x.t));
  CheckTeamType(context_, std::get<parser::TeamValue>(x.t));
  CheckSyncStatList(context_, std::get<std::list<parser::StatOrErrmsg>>(x.t));
}

void CoarrayChecker::Leave(const parser::EndChangeTeamStmt &x) {
  CheckSyncStatList(context_, std::get<std::list<parser::StatOrErrmsg>>(x.t));
}

void CoarrayChecker::Leave(const parser::SyncAllStmt &x) {
  CheckSyncStatList(context_, x.v);
}

void CoarrayChecker::Leave(const parser::SyncImagesStmt &x) {
  CheckSyncStatList(context_, std::get<std::list<parser::StatOrErrmsg>>(x.t));
  const auto &imageSet{std::get<parser::SyncImagesStmt::ImageSet>(x.t)};
````
- **L181 EN**: Closes the current lexical scope or compound statement.
  **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Closes the current lexical scope or compound statement.
  **L182 CN**: 结束当前词法作用域或复合语句块。
- **L183 EN**: Blank line separating nearby declarations or logic blocks.
  **L183 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L184 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::ChangeTeamStmt &x) {`.
  **L184 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::ChangeTeamStmt &x) {`。
- **L185 EN**: Executes a call or declaration centered on `CheckNamesAreDistinct`.
  **L185 CN**: 执行以 `CheckNamesAreDistinct` 为核心的调用或声明。
- **L186 EN**: Executes a call or declaration centered on `CheckTeamType`.
  **L186 CN**: 执行以 `CheckTeamType` 为核心的调用或声明。
- **L187 EN**: Executes a call or declaration centered on `CheckSyncStatList`.
  **L187 CN**: 执行以 `CheckSyncStatList` 为核心的调用或声明。
- **L188 EN**: Closes the current lexical scope or compound statement.
  **L188 CN**: 结束当前词法作用域或复合语句块。
- **L189 EN**: Blank line separating nearby declarations or logic blocks.
  **L189 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L190 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::EndChangeTeamStmt &x) {`.
  **L190 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::EndChangeTeamStmt &x) {`。
- **L191 EN**: Executes a call or declaration centered on `CheckSyncStatList`.
  **L191 CN**: 执行以 `CheckSyncStatList` 为核心的调用或声明。
- **L192 EN**: Closes the current lexical scope or compound statement.
  **L192 CN**: 结束当前词法作用域或复合语句块。
- **L193 EN**: Blank line separating nearby declarations or logic blocks.
  **L193 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L194 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::SyncAllStmt &x) {`.
  **L194 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::SyncAllStmt &x) {`。
- **L195 EN**: Executes a call or declaration centered on `CheckSyncStatList`.
  **L195 CN**: 执行以 `CheckSyncStatList` 为核心的调用或声明。
- **L196 EN**: Closes the current lexical scope or compound statement.
  **L196 CN**: 结束当前词法作用域或复合语句块。
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L198 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::SyncImagesStmt &x) {`.
  **L198 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::SyncImagesStmt &x) {`。
- **L199 EN**: Executes a call or declaration centered on `CheckSyncStatList`.
  **L199 CN**: 执行以 `CheckSyncStatList` 为核心的调用或声明。
- **L200 EN**: Executes a call or declaration centered on `&imageSet{std::get<parser::SyncImagesStmt::ImageSet>`.
  **L200 CN**: 执行以 `&imageSet{std::get<parser::SyncImagesStmt::ImageSet>` 为核心的调用或声明。

### Lines 201-220

````cpp
  if (const auto *intExpr{std::get_if<parser::IntExpr>(&imageSet.u)}) {
    if (const auto *expr{GetExpr(context_, *intExpr)}) {
      if (expr->Rank() > 1) {
        context_.Say(parser::FindSourceLocation(imageSet), // C1174
            "An image-set that is an int-expr must be a scalar or a rank-one array"_err_en_US);
      }
      if (const auto *someInt{
              std::get_if<evaluate::Expr<evaluate::SomeInteger>>(&expr->u)};
          someInt && evaluate::IsActuallyConstant(*someInt)) {
        auto converted{evaluate::Fold(context_.foldingContext(),
            evaluate::ConvertToType<evaluate::SubscriptInteger>(
                common::Clone(*someInt)))};
        if (const auto *cst{
                evaluate::UnwrapConstantValue<evaluate::SubscriptInteger>(
                    converted)}) {
          for (auto elt : cst->values()) {
            auto n{elt.ToInt64()};
            if (n < 1) {
              context_.Say(parser::FindSourceLocation(imageSet),
                  "Image number %jd in the image-set is not valid"_err_en_US,
````
- **L201 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L201 CN**: 开始 `if` 控制流语句并计算其条件。
- **L202 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L202 CN**: 开始 `if` 控制流语句并计算其条件。
- **L203 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L203 CN**: 开始 `if` 控制流语句并计算其条件。
- **L204 EN**: Continues logic associated with callable symbol `Say`.
  **L204 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L205 EN**: Executes a standalone statement or declaration: `"An image-set that is an int-expr must be a scalar or a rank-one array"_err_en_US);`.
  **L205 CN**: 执行一条独立语句或声明：`"An image-set that is an int-expr must be a scalar or a rank-one array"_err_en_US);`。
- **L206 EN**: Closes the current lexical scope or compound statement.
  **L206 CN**: 结束当前词法作用域或复合语句块。
- **L207 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L207 CN**: 开始 `if` 控制流语句并计算其条件。
- **L208 EN**: Executes a call or declaration centered on `std::get_if<evaluate::Expr<evaluate::SomeInteger>>`.
  **L208 CN**: 执行以 `std::get_if<evaluate::Expr<evaluate::SomeInteger>>` 为核心的调用或声明。
- **L209 EN**: Starts a function, method, lambda, or structured scope: `someInt && evaluate::IsActuallyConstant(*someInt)) {`.
  **L209 CN**: 开始一个函数、方法、lambda 或结构化作用域：`someInt && evaluate::IsActuallyConstant(*someInt)) {`。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto converted{evaluate::Fold(context_.foldingContext(),`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto converted{evaluate::Fold(context_.foldingContext(),`。
- **L211 EN**: Continues logic associated with callable symbol `SubscriptInteger>`.
  **L211 CN**: 继续与可调用符号 `SubscriptInteger>` 相关的逻辑。
- **L212 EN**: Executes a call or declaration centered on `common::Clone`.
  **L212 CN**: 执行以 `common::Clone` 为核心的调用或声明。
- **L213 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L213 CN**: 开始 `if` 控制流语句并计算其条件。
- **L214 EN**: Continues logic associated with callable symbol `SubscriptInteger>`.
  **L214 CN**: 继续与可调用符号 `SubscriptInteger>` 相关的逻辑。
- **L215 EN**: Continues the surrounding expression or declaration: `converted)}) {`.
  **L215 CN**: 继续构造周围的表达式或声明：`converted)}) {`。
- **L216 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L216 CN**: 开始 `for` 控制流语句并计算其条件。
- **L217 EN**: Executes a call or declaration centered on `n{elt.ToInt64`.
  **L217 CN**: 执行以 `n{elt.ToInt64` 为核心的调用或声明。
- **L218 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L218 CN**: 开始 `if` 控制流语句并计算其条件。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(imageSet),`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(imageSet),`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Image number %jd in the image-set is not valid"_err_en_US,`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Image number %jd in the image-set is not valid"_err_en_US,`。

### Lines 221-240

````cpp
                  std::intmax_t{n});
              break;
            }
          }
        }
      }
    }
  }
}

void CoarrayChecker::Leave(const parser::SyncMemoryStmt &x) {
  CheckSyncStatList(context_, x.v);
}

void CoarrayChecker::Leave(const parser::SyncTeamStmt &x) {
  CheckTeamType(context_, std::get<parser::TeamValue>(x.t));
  CheckSyncStatList(context_, std::get<std::list<parser::StatOrErrmsg>>(x.t));
}

static void CheckEventWaitSpecList(SemanticsContext &context,
````
- **L221 EN**: Executes a standalone statement or declaration: `std::intmax_t{n});`.
  **L221 CN**: 执行一条独立语句或声明：`std::intmax_t{n});`。
- **L222 EN**: Exits the nearest loop or switch statement.
  **L222 CN**: 退出最近的循环或 switch 语句。
- **L223 EN**: Closes the current lexical scope or compound statement.
  **L223 CN**: 结束当前词法作用域或复合语句块。
- **L224 EN**: Closes the current lexical scope or compound statement.
  **L224 CN**: 结束当前词法作用域或复合语句块。
- **L225 EN**: Closes the current lexical scope or compound statement.
  **L225 CN**: 结束当前词法作用域或复合语句块。
- **L226 EN**: Closes the current lexical scope or compound statement.
  **L226 CN**: 结束当前词法作用域或复合语句块。
- **L227 EN**: Closes the current lexical scope or compound statement.
  **L227 CN**: 结束当前词法作用域或复合语句块。
- **L228 EN**: Closes the current lexical scope or compound statement.
  **L228 CN**: 结束当前词法作用域或复合语句块。
- **L229 EN**: Closes the current lexical scope or compound statement.
  **L229 CN**: 结束当前词法作用域或复合语句块。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L231 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::SyncMemoryStmt &x) {`.
  **L231 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::SyncMemoryStmt &x) {`。
- **L232 EN**: Executes a call or declaration centered on `CheckSyncStatList`.
  **L232 CN**: 执行以 `CheckSyncStatList` 为核心的调用或声明。
- **L233 EN**: Closes the current lexical scope or compound statement.
  **L233 CN**: 结束当前词法作用域或复合语句块。
- **L234 EN**: Blank line separating nearby declarations or logic blocks.
  **L234 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L235 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::SyncTeamStmt &x) {`.
  **L235 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::SyncTeamStmt &x) {`。
- **L236 EN**: Executes a call or declaration centered on `CheckTeamType`.
  **L236 CN**: 执行以 `CheckTeamType` 为核心的调用或声明。
- **L237 EN**: Executes a call or declaration centered on `CheckSyncStatList`.
  **L237 CN**: 执行以 `CheckSyncStatList` 为核心的调用或声明。
- **L238 EN**: Closes the current lexical scope or compound statement.
  **L238 CN**: 结束当前词法作用域或复合语句块。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckEventWaitSpecList(SemanticsContext &context,`.
  **L240 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckEventWaitSpecList(SemanticsContext &context,`。

### Lines 241-260

````cpp
    const std::list<parser::EventWaitSpec> &eventWaitSpecList) {
  bool gotStat{false}, gotMsg{false}, gotUntil{false};
  for (const parser::EventWaitSpec &eventWaitSpec : eventWaitSpecList) {
    common::visit(
        common::visitors{
            [&](const parser::ScalarIntExpr &untilCount) {
              if (gotUntil) {
                context.Say( // C1178
                    "Until-spec in a event-wait-spec-list may not be repeated"_err_en_US);
              }
              gotUntil = true;
            },
            [&](const parser::StatOrErrmsg &statOrErrmsg) {
              common::visit(
                  common::visitors{
                      [&](const parser::StatVariable &stat) {
                        if (gotStat) {
                          context.Say( // C1178
                              "A stat-variable in a event-wait-spec-list may not be repeated"_err_en_US);
                        }
````
- **L241 EN**: Continues the surrounding expression or declaration: `const std::list<parser::EventWaitSpec> &eventWaitSpecList) {`.
  **L241 CN**: 继续构造周围的表达式或声明：`const std::list<parser::EventWaitSpec> &eventWaitSpecList) {`。
- **L242 EN**: Executes a standalone statement or declaration: `bool gotStat{false}, gotMsg{false}, gotUntil{false};`.
  **L242 CN**: 执行一条独立语句或声明：`bool gotStat{false}, gotMsg{false}, gotUntil{false};`。
- **L243 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L243 CN**: 开始 `for` 控制流语句并计算其条件。
- **L244 EN**: Continues logic associated with callable symbol `visit`.
  **L244 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L245 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L245 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L246 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::ScalarIntExpr &untilCount) {`.
  **L246 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::ScalarIntExpr &untilCount) {`。
- **L247 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L247 CN**: 开始 `if` 控制流语句并计算其条件。
- **L248 EN**: Continues logic associated with callable symbol `Say`.
  **L248 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L249 EN**: Executes a standalone statement or declaration: `"Until-spec in a event-wait-spec-list may not be repeated"_err_en_US);`.
  **L249 CN**: 执行一条独立语句或声明：`"Until-spec in a event-wait-spec-list may not be repeated"_err_en_US);`。
- **L250 EN**: Closes the current lexical scope or compound statement.
  **L250 CN**: 结束当前词法作用域或复合语句块。
- **L251 EN**: Executes a standalone statement or declaration: `gotUntil = true;`.
  **L251 CN**: 执行一条独立语句或声明：`gotUntil = true;`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L253 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::StatOrErrmsg &statOrErrmsg) {`.
  **L253 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::StatOrErrmsg &statOrErrmsg) {`。
- **L254 EN**: Continues logic associated with callable symbol `visit`.
  **L254 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L255 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L255 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L256 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::StatVariable &stat) {`.
  **L256 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::StatVariable &stat) {`。
- **L257 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L257 CN**: 开始 `if` 控制流语句并计算其条件。
- **L258 EN**: Continues logic associated with callable symbol `Say`.
  **L258 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L259 EN**: Executes a standalone statement or declaration: `"A stat-variable in a event-wait-spec-list may not be repeated"_err_en_US);`.
  **L259 CN**: 执行一条独立语句或声明：`"A stat-variable in a event-wait-spec-list may not be repeated"_err_en_US);`。
- **L260 EN**: Closes the current lexical scope or compound statement.
  **L260 CN**: 结束当前词法作用域或复合语句块。

### Lines 261-280

````cpp
                        gotStat = true;
                      },
                      [&](const parser::MsgVariable &var) {
                        WarnOnDeferredLengthCharacterScalar(context,
                            GetExpr(context, var),
                            parser::UnwrapRef<parser::Variable>(var)
                                .GetSource(),
                            "ERRMSG=");
                        if (gotMsg) {
                          context.Say( // C1178
                              "A errmsg-variable in a event-wait-spec-list may not be repeated"_err_en_US);
                        }
                        gotMsg = true;
                      },
                  },
                  statOrErrmsg.u);
              CheckCoindexedStatOrErrmsg(
                  context, statOrErrmsg, "event-wait-spec-list");
            },

````
- **L261 EN**: Executes a standalone statement or declaration: `gotStat = true;`.
  **L261 CN**: 执行一条独立语句或声明：`gotStat = true;`。
- **L262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L262 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L263 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::MsgVariable &var) {`.
  **L263 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::MsgVariable &var) {`。
- **L264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WarnOnDeferredLengthCharacterScalar(context,`.
  **L264 CN**: 继续一个多行参数列表、初始化器或聚合项：`WarnOnDeferredLengthCharacterScalar(context,`。
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `GetExpr(context, var),`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`GetExpr(context, var),`。
- **L266 EN**: Continues logic associated with callable symbol `Variable>`.
  **L266 CN**: 继续与可调用符号 `Variable>` 相关的逻辑。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.GetSource(),`.
  **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`.GetSource(),`。
- **L268 EN**: Executes a standalone statement or declaration: `"ERRMSG=");`.
  **L268 CN**: 执行一条独立语句或声明：`"ERRMSG=");`。
- **L269 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L269 CN**: 开始 `if` 控制流语句并计算其条件。
- **L270 EN**: Continues logic associated with callable symbol `Say`.
  **L270 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L271 EN**: Executes a standalone statement or declaration: `"A errmsg-variable in a event-wait-spec-list may not be repeated"_err_en_US);`.
  **L271 CN**: 执行一条独立语句或声明：`"A errmsg-variable in a event-wait-spec-list may not be repeated"_err_en_US);`。
- **L272 EN**: Closes the current lexical scope or compound statement.
  **L272 CN**: 结束当前词法作用域或复合语句块。
- **L273 EN**: Executes a standalone statement or declaration: `gotMsg = true;`.
  **L273 CN**: 执行一条独立语句或声明：`gotMsg = true;`。
- **L274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L274 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L275 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L276 EN**: Executes a standalone statement or declaration: `statOrErrmsg.u);`.
  **L276 CN**: 执行一条独立语句或声明：`statOrErrmsg.u);`。
- **L277 EN**: Continues logic associated with callable symbol `CheckCoindexedStatOrErrmsg`.
  **L277 CN**: 继续与可调用符号 `CheckCoindexedStatOrErrmsg` 相关的逻辑。
- **L278 EN**: Executes a standalone statement or declaration: `context, statOrErrmsg, "event-wait-spec-list");`.
  **L278 CN**: 执行一条独立语句或声明：`context, statOrErrmsg, "event-wait-spec-list");`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L280 EN**: Blank line separating nearby declarations or logic blocks.
  **L280 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 281-300

````cpp
        },
        eventWaitSpec.u);
  }
}

void CoarrayChecker::Leave(const parser::NotifyWaitStmt &x) {
  const auto &notifyVar{std::get<parser::Scalar<parser::Variable>>(x.t)};

  if (const auto *expr{GetExpr(context_, notifyVar)}) {
    if (ExtractCoarrayRef(expr)) {
      context_.Say(parser::FindSourceLocation(notifyVar), // F2023 - C1178
          "A notify-variable in a NOTIFY WAIT statement may not be a coindexed object"_err_en_US);
    } else if (!IsNotifyType(evaluate::GetDerivedTypeSpec(
                   expr->GetType()))) { // F2023 - C1177
      context_.Say(parser::FindSourceLocation(notifyVar),
          "The notify-variable must be of type NOTIFY_TYPE from module ISO_FORTRAN_ENV"_err_en_US);
    } else if (!evaluate::IsCoarray(*expr)) { // F2023 - C1612
      context_.Say(parser::FindSourceLocation(notifyVar),
          "The notify-variable must be a coarray"_err_en_US);
    }
````
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L282 EN**: Executes a standalone statement or declaration: `eventWaitSpec.u);`.
  **L282 CN**: 执行一条独立语句或声明：`eventWaitSpec.u);`。
- **L283 EN**: Closes the current lexical scope or compound statement.
  **L283 CN**: 结束当前词法作用域或复合语句块。
- **L284 EN**: Closes the current lexical scope or compound statement.
  **L284 CN**: 结束当前词法作用域或复合语句块。
- **L285 EN**: Blank line separating nearby declarations or logic blocks.
  **L285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L286 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::NotifyWaitStmt &x) {`.
  **L286 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::NotifyWaitStmt &x) {`。
- **L287 EN**: Executes a call or declaration centered on `&notifyVar{std::get<parser::Scalar<parser::Variable>>`.
  **L287 CN**: 执行以 `&notifyVar{std::get<parser::Scalar<parser::Variable>>` 为核心的调用或声明。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L289 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L289 CN**: 开始 `if` 控制流语句并计算其条件。
- **L290 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L290 CN**: 开始 `if` 控制流语句并计算其条件。
- **L291 EN**: Continues logic associated with callable symbol `Say`.
  **L291 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L292 EN**: Executes a standalone statement or declaration: `"A notify-variable in a NOTIFY WAIT statement may not be a coindexed object"_err_en_US);`.
  **L292 CN**: 执行一条独立语句或声明：`"A notify-variable in a NOTIFY WAIT statement may not be a coindexed object"_err_en_US);`。
- **L293 EN**: Transitions from the previous branch into an `else if` condition.
  **L293 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L294 EN**: Continues logic associated with callable symbol `GetType`.
  **L294 CN**: 继续与可调用符号 `GetType` 相关的逻辑。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(notifyVar),`.
  **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(notifyVar),`。
- **L296 EN**: Executes a standalone statement or declaration: `"The notify-variable must be of type NOTIFY_TYPE from module ISO_FORTRAN_ENV"_err_en_US);`.
  **L296 CN**: 执行一条独立语句或声明：`"The notify-variable must be of type NOTIFY_TYPE from module ISO_FORTRAN_ENV"_err_en_US);`。
- **L297 EN**: Transitions from the previous branch into an `else if` condition.
  **L297 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(notifyVar),`.
  **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(notifyVar),`。
- **L299 EN**: Executes a standalone statement or declaration: `"The notify-variable must be a coarray"_err_en_US);`.
  **L299 CN**: 执行一条独立语句或声明：`"The notify-variable must be a coarray"_err_en_US);`。
- **L300 EN**: Closes the current lexical scope or compound statement.
  **L300 CN**: 结束当前词法作用域或复合语句块。

### Lines 301-320

````cpp
  }

  CheckEventWaitSpecList(
      context_, std::get<std::list<parser::EventWaitSpec>>(x.t));
}

void CoarrayChecker::Leave(const parser::EventPostStmt &x) {
  CheckSyncStatList(context_, std::get<std::list<parser::StatOrErrmsg>>(x.t));
  CheckEventVariable(context_, std::get<parser::EventVariable>(x.t));
}

void CoarrayChecker::Leave(const parser::EventWaitStmt &x) {
  const auto &eventVar{std::get<parser::EventVariable>(x.t)};

  if (const auto *expr{GetExpr(context_, eventVar)}) {
    if (ExtractCoarrayRef(expr)) {
      context_.Say(parser::FindSourceLocation(eventVar), // C1177
          "A event-variable in a EVENT WAIT statement may not be a coindexed object"_err_en_US);
    } else {
      CheckEventVariable(context_, eventVar);
````
- **L301 EN**: Closes the current lexical scope or compound statement.
  **L301 CN**: 结束当前词法作用域或复合语句块。
- **L302 EN**: Blank line separating nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L303 EN**: Continues logic associated with callable symbol `CheckEventWaitSpecList`.
  **L303 CN**: 继续与可调用符号 `CheckEventWaitSpecList` 相关的逻辑。
- **L304 EN**: Executes a call or declaration centered on `std::get<std::list<parser::EventWaitSpec>>`.
  **L304 CN**: 执行以 `std::get<std::list<parser::EventWaitSpec>>` 为核心的调用或声明。
- **L305 EN**: Closes the current lexical scope or compound statement.
  **L305 CN**: 结束当前词法作用域或复合语句块。
- **L306 EN**: Blank line separating nearby declarations or logic blocks.
  **L306 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L307 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::EventPostStmt &x) {`.
  **L307 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::EventPostStmt &x) {`。
- **L308 EN**: Executes a call or declaration centered on `CheckSyncStatList`.
  **L308 CN**: 执行以 `CheckSyncStatList` 为核心的调用或声明。
- **L309 EN**: Executes a call or declaration centered on `CheckEventVariable`.
  **L309 CN**: 执行以 `CheckEventVariable` 为核心的调用或声明。
- **L310 EN**: Closes the current lexical scope or compound statement.
  **L310 CN**: 结束当前词法作用域或复合语句块。
- **L311 EN**: Blank line separating nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L312 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::EventWaitStmt &x) {`.
  **L312 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::EventWaitStmt &x) {`。
- **L313 EN**: Executes a call or declaration centered on `&eventVar{std::get<parser::EventVariable>`.
  **L313 CN**: 执行以 `&eventVar{std::get<parser::EventVariable>` 为核心的调用或声明。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L315 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L315 CN**: 开始 `if` 控制流语句并计算其条件。
- **L316 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L316 CN**: 开始 `if` 控制流语句并计算其条件。
- **L317 EN**: Continues logic associated with callable symbol `Say`.
  **L317 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L318 EN**: Executes a standalone statement or declaration: `"A event-variable in a EVENT WAIT statement may not be a coindexed object"_err_en_US);`.
  **L318 CN**: 执行一条独立语句或声明：`"A event-variable in a EVENT WAIT statement may not be a coindexed object"_err_en_US);`。
- **L319 EN**: Transitions from the previous branch into the alternative path.
  **L319 CN**: 从前一个分支过渡到备选路径。
- **L320 EN**: Executes a call or declaration centered on `CheckEventVariable`.
  **L320 CN**: 执行以 `CheckEventVariable` 为核心的调用或声明。

### Lines 321-340

````cpp
    }
  }

  CheckEventWaitSpecList(
      context_, std::get<std::list<parser::EventWaitSpec>>(x.t));
}

static void CheckLockVariable(
    SemanticsContext &context, const parser::LockVariable &lockVar) {
  if (const SomeExpr * expr{GetExpr(lockVar)}) {
    if (auto dyType{expr->GetType()}) {
      auto at{parser::FindSourceLocation(lockVar)};
      if (dyType->category() != TypeCategory::Derived ||
          dyType->IsUnlimitedPolymorphic() ||
          !IsLockType(&dyType->GetDerivedTypeSpec())) {
        context.Say(at,
            "Lock variable must have type LOCK_TYPE from ISO_FORTRAN_ENV"_err_en_US);
      } else if (auto whyNot{WhyNotDefinable(at, context.FindScope(at),
                     {DefinabilityFlag::DoNotNoteDefinition,
                         DefinabilityFlag::AllowEventLockOrNotifyType},
````
- **L321 EN**: Closes the current lexical scope or compound statement.
  **L321 CN**: 结束当前词法作用域或复合语句块。
- **L322 EN**: Closes the current lexical scope or compound statement.
  **L322 CN**: 结束当前词法作用域或复合语句块。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L324 EN**: Continues logic associated with callable symbol `CheckEventWaitSpecList`.
  **L324 CN**: 继续与可调用符号 `CheckEventWaitSpecList` 相关的逻辑。
- **L325 EN**: Executes a call or declaration centered on `std::get<std::list<parser::EventWaitSpec>>`.
  **L325 CN**: 执行以 `std::get<std::list<parser::EventWaitSpec>>` 为核心的调用或声明。
- **L326 EN**: Closes the current lexical scope or compound statement.
  **L326 CN**: 结束当前词法作用域或复合语句块。
- **L327 EN**: Blank line separating nearby declarations or logic blocks.
  **L327 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L328 EN**: Continues logic associated with callable symbol `CheckLockVariable`.
  **L328 CN**: 继续与可调用符号 `CheckLockVariable` 相关的逻辑。
- **L329 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::LockVariable &lockVar) {`.
  **L329 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::LockVariable &lockVar) {`。
- **L330 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L330 CN**: 开始 `if` 控制流语句并计算其条件。
- **L331 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L331 CN**: 开始 `if` 控制流语句并计算其条件。
- **L332 EN**: Executes a call or declaration centered on `at{parser::FindSourceLocation`.
  **L332 CN**: 执行以 `at{parser::FindSourceLocation` 为核心的调用或声明。
- **L333 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L333 CN**: 开始 `if` 控制流语句并计算其条件。
- **L334 EN**: Continues logic associated with callable symbol `IsUnlimitedPolymorphic`.
  **L334 CN**: 继续与可调用符号 `IsUnlimitedPolymorphic` 相关的逻辑。
- **L335 EN**: Starts a function, method, lambda, or structured scope: `!IsLockType(&dyType->GetDerivedTypeSpec())) {`.
  **L335 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!IsLockType(&dyType->GetDerivedTypeSpec())) {`。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(at,`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(at,`。
- **L337 EN**: Executes a standalone statement or declaration: `"Lock variable must have type LOCK_TYPE from ISO_FORTRAN_ENV"_err_en_US);`.
  **L337 CN**: 执行一条独立语句或声明：`"Lock variable must have type LOCK_TYPE from ISO_FORTRAN_ENV"_err_en_US);`。
- **L338 EN**: Transitions from the previous branch into an `else if` condition.
  **L338 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{DefinabilityFlag::DoNotNoteDefinition,`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`{DefinabilityFlag::DoNotNoteDefinition,`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefinabilityFlag::AllowEventLockOrNotifyType},`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefinabilityFlag::AllowEventLockOrNotifyType},`。

### Lines 341-360

````cpp
                     *expr)}) {
        whyNot->set_severity(parser::Severity::Because);
        context.Say(at, "Lock variable is not definable"_err_en_US)
            .Attach(std::move(*whyNot));
      }
    }
  }
}

void CoarrayChecker::Leave(const parser::LockStmt &x) {
  CheckLockVariable(context_, std::get<parser::LockVariable>(x.t));
  bool gotAcquired{false}, gotStat{false}, gotMsg{false};
  for (const parser::LockStmt::LockStat &lockStat :
      std::get<std::list<parser::LockStmt::LockStat>>(x.t)) {
    if (const auto *statOrErrmsg{
            std::get_if<parser::StatOrErrmsg>(&lockStat.u)}) {
      CheckSyncStat(context_, *statOrErrmsg, gotStat, gotMsg);
    } else {
      CHECK(std::holds_alternative<
          parser::Scalar<parser::Logical<parser::Variable>>>(lockStat.u));
````
- **L341 EN**: Comment explains nearby logic, intent, or metadata: `expr)}) {`.
  **L341 CN**: 注释说明附近代码的逻辑、意图或元数据：`expr)}) {`。
- **L342 EN**: Executes a call or declaration centered on `whyNot->set_severity`.
  **L342 CN**: 执行以 `whyNot->set_severity` 为核心的调用或声明。
- **L343 EN**: Continues logic associated with callable symbol `Say`.
  **L343 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L344 EN**: Executes a call or declaration centered on `.Attach`.
  **L344 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L345 EN**: Closes the current lexical scope or compound statement.
  **L345 CN**: 结束当前词法作用域或复合语句块。
- **L346 EN**: Closes the current lexical scope or compound statement.
  **L346 CN**: 结束当前词法作用域或复合语句块。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L350 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::LockStmt &x) {`.
  **L350 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::LockStmt &x) {`。
- **L351 EN**: Executes a call or declaration centered on `CheckLockVariable`.
  **L351 CN**: 执行以 `CheckLockVariable` 为核心的调用或声明。
- **L352 EN**: Executes a standalone statement or declaration: `bool gotAcquired{false}, gotStat{false}, gotMsg{false};`.
  **L352 CN**: 执行一条独立语句或声明：`bool gotAcquired{false}, gotStat{false}, gotMsg{false};`。
- **L353 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L353 CN**: 开始 `for` 控制流语句并计算其条件。
- **L354 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::LockStmt::LockStat>>(x.t)) {`.
  **L354 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::LockStmt::LockStat>>(x.t)) {`。
- **L355 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L355 CN**: 开始 `if` 控制流语句并计算其条件。
- **L356 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<parser::StatOrErrmsg>(&lockStat.u)}) {`.
  **L356 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<parser::StatOrErrmsg>(&lockStat.u)}) {`。
- **L357 EN**: Executes a call or declaration centered on `CheckSyncStat`.
  **L357 CN**: 执行以 `CheckSyncStat` 为核心的调用或声明。
- **L358 EN**: Transitions from the previous branch into the alternative path.
  **L358 CN**: 从前一个分支过渡到备选路径。
- **L359 EN**: Continues logic associated with callable symbol `CHECK`.
  **L359 CN**: 继续与可调用符号 `CHECK` 相关的逻辑。
- **L360 EN**: Executes a call or declaration centered on `parser::Scalar<parser::Logical<parser::Variable>>>`.
  **L360 CN**: 执行以 `parser::Scalar<parser::Logical<parser::Variable>>>` 为核心的调用或声明。

### Lines 361-380

````cpp
      if (gotAcquired) {
        context_.Say(parser::FindSourceLocation(lockStat),
            "Multiple ACQUIRED_LOCK specifiers"_err_en_US);
      } else {
        gotAcquired = true;
      }
    }
  }
}

void CoarrayChecker::Leave(const parser::UnlockStmt &x) {
  CheckLockVariable(context_, std::get<parser::LockVariable>(x.t));
  CheckSyncStatList(context_, std::get<std::list<parser::StatOrErrmsg>>(x.t));
}

void CoarrayChecker::Leave(const parser::CriticalStmt &x) {
  CheckSyncStatList(context_, std::get<std::list<parser::StatOrErrmsg>>(x.t));
}

void CoarrayChecker::Leave(const parser::ImageSelector &imageSelector) {
````
- **L361 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L361 CN**: 开始 `if` 控制流语句并计算其条件。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(lockStat),`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(lockStat),`。
- **L363 EN**: Executes a standalone statement or declaration: `"Multiple ACQUIRED_LOCK specifiers"_err_en_US);`.
  **L363 CN**: 执行一条独立语句或声明：`"Multiple ACQUIRED_LOCK specifiers"_err_en_US);`。
- **L364 EN**: Transitions from the previous branch into the alternative path.
  **L364 CN**: 从前一个分支过渡到备选路径。
- **L365 EN**: Executes a standalone statement or declaration: `gotAcquired = true;`.
  **L365 CN**: 执行一条独立语句或声明：`gotAcquired = true;`。
- **L366 EN**: Closes the current lexical scope or compound statement.
  **L366 CN**: 结束当前词法作用域或复合语句块。
- **L367 EN**: Closes the current lexical scope or compound statement.
  **L367 CN**: 结束当前词法作用域或复合语句块。
- **L368 EN**: Closes the current lexical scope or compound statement.
  **L368 CN**: 结束当前词法作用域或复合语句块。
- **L369 EN**: Closes the current lexical scope or compound statement.
  **L369 CN**: 结束当前词法作用域或复合语句块。
- **L370 EN**: Blank line separating nearby declarations or logic blocks.
  **L370 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L371 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::UnlockStmt &x) {`.
  **L371 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::UnlockStmt &x) {`。
- **L372 EN**: Executes a call or declaration centered on `CheckLockVariable`.
  **L372 CN**: 执行以 `CheckLockVariable` 为核心的调用或声明。
- **L373 EN**: Executes a call or declaration centered on `CheckSyncStatList`.
  **L373 CN**: 执行以 `CheckSyncStatList` 为核心的调用或声明。
- **L374 EN**: Closes the current lexical scope or compound statement.
  **L374 CN**: 结束当前词法作用域或复合语句块。
- **L375 EN**: Blank line separating nearby declarations or logic blocks.
  **L375 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L376 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::CriticalStmt &x) {`.
  **L376 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::CriticalStmt &x) {`。
- **L377 EN**: Executes a call or declaration centered on `CheckSyncStatList`.
  **L377 CN**: 执行以 `CheckSyncStatList` 为核心的调用或声明。
- **L378 EN**: Closes the current lexical scope or compound statement.
  **L378 CN**: 结束当前词法作用域或复合语句块。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L380 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::ImageSelector &imageSelector) {`.
  **L380 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::ImageSelector &imageSelector) {`。

### Lines 381-400

````cpp
  for (const auto &imageSelectorSpec :
      std::get<std::list<parser::ImageSelectorSpec>>(imageSelector.t)) {
    if (const auto *stat{std::get_if<parser::ImageSelectorSpec::Stat>(
            &imageSelectorSpec.u)}) {
      CheckTeamStat(context_, *stat);
    }
  }
}

void CoarrayChecker::Leave(const parser::FormTeamStmt &x) {
  CheckTeamType(
      context_, std::get<parser::TeamVariable>(x.t), /*mustBeVariable=*/true);
  for (const auto &spec :
      std::get<std::list<parser::FormTeamStmt::FormTeamSpec>>(x.t)) {
    if (const auto *statOrErrmsg{std::get_if<parser::StatOrErrmsg>(&spec.u)}) {
      CheckCoindexedStatOrErrmsg(
          context_, *statOrErrmsg, "form-team-spec-list");
    }
  }
}
````
- **L381 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L381 CN**: 开始 `for` 控制流语句并计算其条件。
- **L382 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::ImageSelectorSpec>>(imageSelector.t)) {`.
  **L382 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::ImageSelectorSpec>>(imageSelector.t)) {`。
- **L383 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L383 CN**: 开始 `if` 控制流语句并计算其条件。
- **L384 EN**: Continues the surrounding expression or declaration: `&imageSelectorSpec.u)}) {`.
  **L384 CN**: 继续构造周围的表达式或声明：`&imageSelectorSpec.u)}) {`。
- **L385 EN**: Executes a call or declaration centered on `CheckTeamStat`.
  **L385 CN**: 执行以 `CheckTeamStat` 为核心的调用或声明。
- **L386 EN**: Closes the current lexical scope or compound statement.
  **L386 CN**: 结束当前词法作用域或复合语句块。
- **L387 EN**: Closes the current lexical scope or compound statement.
  **L387 CN**: 结束当前词法作用域或复合语句块。
- **L388 EN**: Closes the current lexical scope or compound statement.
  **L388 CN**: 结束当前词法作用域或复合语句块。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L390 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Leave(const parser::FormTeamStmt &x) {`.
  **L390 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Leave(const parser::FormTeamStmt &x) {`。
- **L391 EN**: Continues logic associated with callable symbol `CheckTeamType`.
  **L391 CN**: 继续与可调用符号 `CheckTeamType` 相关的逻辑。
- **L392 EN**: Executes a call or declaration centered on `std::get<parser::TeamVariable>`.
  **L392 CN**: 执行以 `std::get<parser::TeamVariable>` 为核心的调用或声明。
- **L393 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L393 CN**: 开始 `for` 控制流语句并计算其条件。
- **L394 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::FormTeamStmt::FormTeamSpec>>(x.t)) {`.
  **L394 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::FormTeamStmt::FormTeamSpec>>(x.t)) {`。
- **L395 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L395 CN**: 开始 `if` 控制流语句并计算其条件。
- **L396 EN**: Continues logic associated with callable symbol `CheckCoindexedStatOrErrmsg`.
  **L396 CN**: 继续与可调用符号 `CheckCoindexedStatOrErrmsg` 相关的逻辑。
- **L397 EN**: Executes a standalone statement or declaration: `context_, *statOrErrmsg, "form-team-spec-list");`.
  **L397 CN**: 执行一条独立语句或声明：`context_, *statOrErrmsg, "form-team-spec-list");`。
- **L398 EN**: Closes the current lexical scope or compound statement.
  **L398 CN**: 结束当前词法作用域或复合语句块。
- **L399 EN**: Closes the current lexical scope or compound statement.
  **L399 CN**: 结束当前词法作用域或复合语句块。
- **L400 EN**: Closes the current lexical scope or compound statement.
  **L400 CN**: 结束当前词法作用域或复合语句块。

### Lines 401-420

````cpp

void CoarrayChecker::Enter(const parser::CriticalConstruct &x) {
  auto &criticalStmt{std::get<parser::Statement<parser::CriticalStmt>>(x.t)};
  const parser::Block &block{std::get<parser::Block>(x.t)};
  CriticalBodyEnforce criticalBodyEnforce{context_, criticalStmt.source};
  parser::Walk(block, criticalBodyEnforce);
  parser::Walk(std::get<parser::Statement<parser::EndCriticalStmt>>(x.t),
      criticalBodyEnforce);
  LabelEnforce criticalLabelEnforce{
      context_, criticalBodyEnforce.labels(), criticalStmt.source, "CRITICAL"};
  parser::Walk(block, criticalLabelEnforce);
}

void CoarrayChecker::Enter(const parser::ChangeTeamConstruct &x) {
  auto &changeTeamStmt{
      std::get<parser::Statement<parser::ChangeTeamStmt>>(x.t)};
  const parser::Block &block{std::get<parser::Block>(x.t)};
  ChangeTeamBodyEnforce changeTeamBodyEnforce{context_, changeTeamStmt.source};
  parser::Walk(block, changeTeamBodyEnforce);
  parser::Walk(std::get<parser::Statement<parser::EndChangeTeamStmt>>(x.t),
````
- **L401 EN**: Blank line separating nearby declarations or logic blocks.
  **L401 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L402 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Enter(const parser::CriticalConstruct &x) {`.
  **L402 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Enter(const parser::CriticalConstruct &x) {`。
- **L403 EN**: Executes a call or declaration centered on `&criticalStmt{std::get<parser::Statement<parser::CriticalStmt>>`.
  **L403 CN**: 执行以 `&criticalStmt{std::get<parser::Statement<parser::CriticalStmt>>` 为核心的调用或声明。
- **L404 EN**: Executes a call or declaration centered on `&block{std::get<parser::Block>`.
  **L404 CN**: 执行以 `&block{std::get<parser::Block>` 为核心的调用或声明。
- **L405 EN**: Executes a standalone statement or declaration: `CriticalBodyEnforce criticalBodyEnforce{context_, criticalStmt.source};`.
  **L405 CN**: 执行一条独立语句或声明：`CriticalBodyEnforce criticalBodyEnforce{context_, criticalStmt.source};`。
- **L406 EN**: Executes a call or declaration centered on `parser::Walk`.
  **L406 CN**: 执行以 `parser::Walk` 为核心的调用或声明。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::Walk(std::get<parser::Statement<parser::EndCriticalStmt>>(x.t),`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::Walk(std::get<parser::Statement<parser::EndCriticalStmt>>(x.t),`。
- **L408 EN**: Executes a standalone statement or declaration: `criticalBodyEnforce);`.
  **L408 CN**: 执行一条独立语句或声明：`criticalBodyEnforce);`。
- **L409 EN**: Continues the surrounding expression or declaration: `LabelEnforce criticalLabelEnforce{`.
  **L409 CN**: 继续构造周围的表达式或声明：`LabelEnforce criticalLabelEnforce{`。
- **L410 EN**: Executes a call or declaration centered on `criticalBodyEnforce.labels`.
  **L410 CN**: 执行以 `criticalBodyEnforce.labels` 为核心的调用或声明。
- **L411 EN**: Executes a call or declaration centered on `parser::Walk`.
  **L411 CN**: 执行以 `parser::Walk` 为核心的调用或声明。
- **L412 EN**: Closes the current lexical scope or compound statement.
  **L412 CN**: 结束当前词法作用域或复合语句块。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L414 EN**: Starts a function, method, lambda, or structured scope: `void CoarrayChecker::Enter(const parser::ChangeTeamConstruct &x) {`.
  **L414 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CoarrayChecker::Enter(const parser::ChangeTeamConstruct &x) {`。
- **L415 EN**: Continues the surrounding expression or declaration: `auto &changeTeamStmt{`.
  **L415 CN**: 继续构造周围的表达式或声明：`auto &changeTeamStmt{`。
- **L416 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::ChangeTeamStmt>>`.
  **L416 CN**: 执行以 `std::get<parser::Statement<parser::ChangeTeamStmt>>` 为核心的调用或声明。
- **L417 EN**: Executes a call or declaration centered on `&block{std::get<parser::Block>`.
  **L417 CN**: 执行以 `&block{std::get<parser::Block>` 为核心的调用或声明。
- **L418 EN**: Executes a standalone statement or declaration: `ChangeTeamBodyEnforce changeTeamBodyEnforce{context_, changeTeamStmt.source};`.
  **L418 CN**: 执行一条独立语句或声明：`ChangeTeamBodyEnforce changeTeamBodyEnforce{context_, changeTeamStmt.source};`。
- **L419 EN**: Executes a call or declaration centered on `parser::Walk`.
  **L419 CN**: 执行以 `parser::Walk` 为核心的调用或声明。
- **L420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::Walk(std::get<parser::Statement<parser::EndChangeTeamStmt>>(x.t),`.
  **L420 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::Walk(std::get<parser::Statement<parser::EndChangeTeamStmt>>(x.t),`。

### Lines 421-440

````cpp
      changeTeamBodyEnforce);
  LabelEnforce changeTeamLabelEnforce{context_, changeTeamBodyEnforce.labels(),
      changeTeamStmt.source, "CHANGE TEAM"};
  parser::Walk(block, changeTeamLabelEnforce);
}

// Check that coarray names and selector names are all distinct.
void CoarrayChecker::CheckNamesAreDistinct(
    const std::list<parser::CoarrayAssociation> &list) {
  std::set<parser::CharBlock> names;
  auto getPreviousUse{
      [&](const parser::Name &name) -> const parser::CharBlock * {
        auto pair{names.insert(name.source)};
        return !pair.second ? &*pair.first : nullptr;
      }};
  for (const auto &assoc : list) {
    const auto &decl{std::get<parser::CodimensionDecl>(assoc.t)};
    const auto &selector{std::get<parser::Selector>(assoc.t)};
    const auto &declName{std::get<parser::Name>(decl.t)};
    if (context_.HasError(declName)) {
````
- **L421 EN**: Executes a standalone statement or declaration: `changeTeamBodyEnforce);`.
  **L421 CN**: 执行一条独立语句或声明：`changeTeamBodyEnforce);`。
- **L422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LabelEnforce changeTeamLabelEnforce{context_, changeTeamBodyEnforce.labels(),`.
  **L422 CN**: 继续一个多行参数列表、初始化器或聚合项：`LabelEnforce changeTeamLabelEnforce{context_, changeTeamBodyEnforce.labels(),`。
- **L423 EN**: Executes a standalone statement or declaration: `changeTeamStmt.source, "CHANGE TEAM"};`.
  **L423 CN**: 执行一条独立语句或声明：`changeTeamStmt.source, "CHANGE TEAM"};`。
- **L424 EN**: Executes a call or declaration centered on `parser::Walk`.
  **L424 CN**: 执行以 `parser::Walk` 为核心的调用或声明。
- **L425 EN**: Closes the current lexical scope or compound statement.
  **L425 CN**: 结束当前词法作用域或复合语句块。
- **L426 EN**: Blank line separating nearby declarations or logic blocks.
  **L426 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L427 EN**: Comment explains nearby logic, intent, or metadata: `Check that coarray names and selector names are all distinct.`.
  **L427 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check that coarray names and selector names are all distinct.`。
- **L428 EN**: Continues logic associated with callable symbol `CheckNamesAreDistinct`.
  **L428 CN**: 继续与可调用符号 `CheckNamesAreDistinct` 相关的逻辑。
- **L429 EN**: Continues the surrounding expression or declaration: `const std::list<parser::CoarrayAssociation> &list) {`.
  **L429 CN**: 继续构造周围的表达式或声明：`const std::list<parser::CoarrayAssociation> &list) {`。
- **L430 EN**: Executes a standalone statement or declaration: `std::set<parser::CharBlock> names;`.
  **L430 CN**: 执行一条独立语句或声明：`std::set<parser::CharBlock> names;`。
- **L431 EN**: Continues the surrounding expression or declaration: `auto getPreviousUse{`.
  **L431 CN**: 继续构造周围的表达式或声明：`auto getPreviousUse{`。
- **L432 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::Name &name) -> const parser::CharBlock * {`.
  **L432 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::Name &name) -> const parser::CharBlock * {`。
- **L433 EN**: Executes a call or declaration centered on `pair{names.insert`.
  **L433 CN**: 执行以 `pair{names.insert` 为核心的调用或声明。
- **L434 EN**: Returns from the current function with `!pair.second ? &*pair.first : nullptr`.
  **L434 CN**: 以 `!pair.second ? &*pair.first : nullptr` 从当前函数返回。
- **L435 EN**: Executes a standalone statement or declaration: `}};`.
  **L435 CN**: 执行一条独立语句或声明：`}};`。
- **L436 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L436 CN**: 开始 `for` 控制流语句并计算其条件。
- **L437 EN**: Executes a call or declaration centered on `&decl{std::get<parser::CodimensionDecl>`.
  **L437 CN**: 执行以 `&decl{std::get<parser::CodimensionDecl>` 为核心的调用或声明。
- **L438 EN**: Executes a call or declaration centered on `&selector{std::get<parser::Selector>`.
  **L438 CN**: 执行以 `&selector{std::get<parser::Selector>` 为核心的调用或声明。
- **L439 EN**: Executes a call or declaration centered on `&declName{std::get<parser::Name>`.
  **L439 CN**: 执行以 `&declName{std::get<parser::Name>` 为核心的调用或声明。
- **L440 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L440 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 441-460

````cpp
      continue; // already reported an error about this name
    }
    if (auto *prev{getPreviousUse(declName)}) {
      Say2(declName.source, // C1113
          "Coarray '%s' was already used as a selector or coarray in this statement"_err_en_US,
          *prev, "Previous use of '%s'"_en_US);
    }
    // ResolveNames verified the selector is a simple name
    const parser::Name *name{parser::Unwrap<parser::Name>(selector)};
    if (name) {
      if (auto *prev{getPreviousUse(*name)}) {
        Say2(name->source, // C1113, C1115
            "Selector '%s' was already used as a selector or coarray in this statement"_err_en_US,
            *prev, "Previous use of '%s'"_en_US);
      }
    }
  }
}

void CoarrayChecker::Say2(const parser::CharBlock &name1,
````
- **L441 EN**: Skips to the next loop iteration.
  **L441 CN**: 跳到下一次循环迭代。
- **L442 EN**: Closes the current lexical scope or compound statement.
  **L442 CN**: 结束当前词法作用域或复合语句块。
- **L443 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L443 CN**: 开始 `if` 控制流语句并计算其条件。
- **L444 EN**: Continues logic associated with callable symbol `Say2`.
  **L444 CN**: 继续与可调用符号 `Say2` 相关的逻辑。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Coarray '%s' was already used as a selector or coarray in this statement"_err_en_US,`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Coarray '%s' was already used as a selector or coarray in this statement"_err_en_US,`。
- **L446 EN**: Comment explains nearby logic, intent, or metadata: `prev, "Previous use of '%s'"_en_US);`.
  **L446 CN**: 注释说明附近代码的逻辑、意图或元数据：`prev, "Previous use of '%s'"_en_US);`。
- **L447 EN**: Closes the current lexical scope or compound statement.
  **L447 CN**: 结束当前词法作用域或复合语句块。
- **L448 EN**: Comment explains nearby logic, intent, or metadata: `ResolveNames verified the selector is a simple name`.
  **L448 CN**: 注释说明附近代码的逻辑、意图或元数据：`ResolveNames verified the selector is a simple name`。
- **L449 EN**: Executes a call or declaration centered on `*name{parser::Unwrap<parser::Name>`.
  **L449 CN**: 执行以 `*name{parser::Unwrap<parser::Name>` 为核心的调用或声明。
- **L450 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L450 CN**: 开始 `if` 控制流语句并计算其条件。
- **L451 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L451 CN**: 开始 `if` 控制流语句并计算其条件。
- **L452 EN**: Continues logic associated with callable symbol `Say2`.
  **L452 CN**: 继续与可调用符号 `Say2` 相关的逻辑。
- **L453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Selector '%s' was already used as a selector or coarray in this statement"_err_en_US,`.
  **L453 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Selector '%s' was already used as a selector or coarray in this statement"_err_en_US,`。
- **L454 EN**: Comment explains nearby logic, intent, or metadata: `prev, "Previous use of '%s'"_en_US);`.
  **L454 CN**: 注释说明附近代码的逻辑、意图或元数据：`prev, "Previous use of '%s'"_en_US);`。
- **L455 EN**: Closes the current lexical scope or compound statement.
  **L455 CN**: 结束当前词法作用域或复合语句块。
- **L456 EN**: Closes the current lexical scope or compound statement.
  **L456 CN**: 结束当前词法作用域或复合语句块。
- **L457 EN**: Closes the current lexical scope or compound statement.
  **L457 CN**: 结束当前词法作用域或复合语句块。
- **L458 EN**: Closes the current lexical scope or compound statement.
  **L458 CN**: 结束当前词法作用域或复合语句块。
- **L459 EN**: Blank line separating nearby declarations or logic blocks.
  **L459 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CoarrayChecker::Say2(const parser::CharBlock &name1,`.
  **L460 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CoarrayChecker::Say2(const parser::CharBlock &name1,`。

### Lines 461-466

````cpp
    parser::MessageFixedText &&msg1, const parser::CharBlock &name2,
    parser::MessageFixedText &&msg2) {
  context_.Say(name1, std::move(msg1), name1)
      .Attach(name2, std::move(msg2), name2);
}
} // namespace Fortran::semantics
````
- **L461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::MessageFixedText &&msg1, const parser::CharBlock &name2,`.
  **L461 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::MessageFixedText &&msg1, const parser::CharBlock &name2,`。
- **L462 EN**: Continues the surrounding expression or declaration: `parser::MessageFixedText &&msg2) {`.
  **L462 CN**: 继续构造周围的表达式或声明：`parser::MessageFixedText &&msg2) {`。
- **L463 EN**: Continues logic associated with callable symbol `Say`.
  **L463 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L464 EN**: Executes a call or declaration centered on `.Attach`.
  **L464 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L465 EN**: Closes the current lexical scope or compound statement.
  **L465 CN**: 结束当前词法作用域或复合语句块。
- **L466 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L466 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Fortran parse tree handling / Fortran 语法树处理**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Semantic-context management / 语义上下文管理**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **Constant folding / 常量折叠**
- **Rule-checking workflow / 规则检查流程**

## Dependencies / 依赖关系

- `check-coarray.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `definable.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Common/indirection.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Evaluate/check-expression.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/expression.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Parser/message.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/parse-tree.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/tools.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/expression.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
