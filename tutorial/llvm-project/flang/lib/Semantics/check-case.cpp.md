# check-case.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-case.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements semantic checking, symbol reasoning, and language-rule enforcement for check case.
- **Purpose (CN)**: 实现 check case 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===-- lib/Semantics/check-case.cpp --------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "check-case.h"
#include "flang/Common/idioms.h"
#include "flang/Common/reference.h"
#include "flang/Common/template.h"
#include "flang/Evaluate/fold.h"
#include "flang/Evaluate/type.h"
#include "flang/Parser/parse-tree.h"
#include "flang/Semantics/semantics.h"
#include "flang/Semantics/tools.h"
#include <tuple>
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
- **L9 EN**: Includes "check-case.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "check-case.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "flang/Common/idioms.h" to access shared Flang utility infrastructure.
  **L10 CN**: 引入 "flang/Common/idioms.h" 以使用Flang 共享工具基础设施。
- **L11 EN**: Includes "flang/Common/reference.h" to access shared Flang utility infrastructure.
  **L11 CN**: 引入 "flang/Common/reference.h" 以使用Flang 共享工具基础设施。
- **L12 EN**: Includes "flang/Common/template.h" to access shared Flang utility infrastructure.
  **L12 CN**: 引入 "flang/Common/template.h" 以使用Flang 共享工具基础设施。
- **L13 EN**: Includes "flang/Evaluate/fold.h" to access Fortran constant-folding and evaluation facilities.
  **L13 CN**: 引入 "flang/Evaluate/fold.h" 以使用Fortran 常量折叠与求值能力。
- **L14 EN**: Includes "flang/Evaluate/type.h" to access Fortran constant-folding and evaluation facilities.
  **L14 CN**: 引入 "flang/Evaluate/type.h" 以使用Fortran 常量折叠与求值能力。
- **L15 EN**: Includes "flang/Parser/parse-tree.h" to access parse-tree, token, or source representation support.
  **L15 CN**: 引入 "flang/Parser/parse-tree.h" 以使用语法树、词法单元或源码表示支持。
- **L16 EN**: Includes "flang/Semantics/semantics.h" to access Fortran semantic analysis, symbol, and type information.
  **L16 CN**: 引入 "flang/Semantics/semantics.h" 以使用Fortran 语义分析、符号与类型信息。
- **L17 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L17 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L18 EN**: Includes <tuple> to access supporting declarations used by this translation unit.
  **L18 CN**: 引入 <tuple> 以使用当前编译单元使用的辅助声明。

### Lines 19-36

````cpp

namespace Fortran::semantics {

template <typename T> class CaseValues {
public:
  CaseValues(SemanticsContext &c, const evaluate::DynamicType &t)
      : context_{c}, caseExprType_{t} {}

  void Check(const std::list<parser::CaseConstruct::Case> &cases) {
    for (const parser::CaseConstruct::Case &c : cases) {
      AddCase(c);
    }
    if (!hasErrors_) {
      cases_.sort(Comparator{});
      if (!AreCasesDisjoint()) { // C1149
        ReportConflictingCases();
      }
    }
````
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Opens namespace scope `Fortran::semantics`.
  **L20 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Introduces template parameters or specialization context: `template <typename T> class CaseValues {`.
  **L22 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> class CaseValues {`。
- **L23 EN**: Sets the following members to `public` access.
  **L23 CN**: 将后续成员的访问级别设为 `public`。
- **L24 EN**: Continues logic associated with callable symbol `CaseValues`.
  **L24 CN**: 继续与可调用符号 `CaseValues` 相关的逻辑。
- **L25 EN**: Continues the surrounding expression or declaration: `: context_{c}, caseExprType_{t} {}`.
  **L25 CN**: 继续构造周围的表达式或声明：`: context_{c}, caseExprType_{t} {}`。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Starts a function, method, lambda, or structured scope: `void Check(const std::list<parser::CaseConstruct::Case> &cases) {`.
  **L27 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const std::list<parser::CaseConstruct::Case> &cases) {`。
- **L28 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L28 CN**: 开始 `for` 控制流语句并计算其条件。
- **L29 EN**: Executes a call or declaration centered on `AddCase`.
  **L29 CN**: 执行以 `AddCase` 为核心的调用或声明。
- **L30 EN**: Closes the current lexical scope or compound statement.
  **L30 CN**: 结束当前词法作用域或复合语句块。
- **L31 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L31 CN**: 开始 `if` 控制流语句并计算其条件。
- **L32 EN**: Executes a call or declaration centered on `cases_.sort`.
  **L32 CN**: 执行以 `cases_.sort` 为核心的调用或声明。
- **L33 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L33 CN**: 开始 `if` 控制流语句并计算其条件。
- **L34 EN**: Executes a call or declaration centered on `ReportConflictingCases`.
  **L34 CN**: 执行以 `ReportConflictingCases` 为核心的调用或声明。
- **L35 EN**: Closes the current lexical scope or compound statement.
  **L35 CN**: 结束当前词法作用域或复合语句块。
- **L36 EN**: Closes the current lexical scope or compound statement.
  **L36 CN**: 结束当前词法作用域或复合语句块。

### Lines 37-54

````cpp
  }

private:
  using Value = evaluate::Scalar<T>;

  void AddCase(const parser::CaseConstruct::Case &c) {
    const auto &stmt{std::get<parser::Statement<parser::CaseStmt>>(c.t)};
    const parser::CaseStmt &caseStmt{stmt.statement};
    const auto &selector{std::get<parser::CaseSelector>(caseStmt.t)};
    common::visit(
        common::visitors{
            [&](const std::list<parser::CaseValueRange> &ranges) {
              for (const auto &range : ranges) {
                auto pair{ComputeBounds(range)};
                if (pair.first && pair.second && *pair.first > *pair.second) {
                  context_.Warn(common::UsageWarning::EmptyCase, stmt.source,
                      "CASE has lower bound greater than upper bound"_warn_en_US);
                } else {
````
- **L37 EN**: Closes the current lexical scope or compound statement.
  **L37 CN**: 结束当前词法作用域或复合语句块。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L39 EN**: Sets the following members to `private` access.
  **L39 CN**: 将后续成员的访问级别设为 `private`。
- **L40 EN**: Defines alias `Value` to simplify later code.
  **L40 CN**: 定义别名 `Value` 以简化后续代码。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L42 EN**: Starts a function, method, lambda, or structured scope: `void AddCase(const parser::CaseConstruct::Case &c) {`.
  **L42 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AddCase(const parser::CaseConstruct::Case &c) {`。
- **L43 EN**: Executes a call or declaration centered on `&stmt{std::get<parser::Statement<parser::CaseStmt>>`.
  **L43 CN**: 执行以 `&stmt{std::get<parser::Statement<parser::CaseStmt>>` 为核心的调用或声明。
- **L44 EN**: Executes a standalone statement or declaration: `const parser::CaseStmt &caseStmt{stmt.statement};`.
  **L44 CN**: 执行一条独立语句或声明：`const parser::CaseStmt &caseStmt{stmt.statement};`。
- **L45 EN**: Executes a call or declaration centered on `&selector{std::get<parser::CaseSelector>`.
  **L45 CN**: 执行以 `&selector{std::get<parser::CaseSelector>` 为核心的调用或声明。
- **L46 EN**: Continues logic associated with callable symbol `visit`.
  **L46 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L47 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L47 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L48 EN**: Starts a function, method, lambda, or structured scope: `[&](const std::list<parser::CaseValueRange> &ranges) {`.
  **L48 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const std::list<parser::CaseValueRange> &ranges) {`。
- **L49 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L49 CN**: 开始 `for` 控制流语句并计算其条件。
- **L50 EN**: Executes a call or declaration centered on `pair{ComputeBounds`.
  **L50 CN**: 执行以 `pair{ComputeBounds` 为核心的调用或声明。
- **L51 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L51 CN**: 开始 `if` 控制流语句并计算其条件。
- **L52 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::UsageWarning::EmptyCase, stmt.source,`.
  **L52 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::UsageWarning::EmptyCase, stmt.source,`。
- **L53 EN**: Executes a standalone statement or declaration: `"CASE has lower bound greater than upper bound"_warn_en_US);`.
  **L53 CN**: 执行一条独立语句或声明：`"CASE has lower bound greater than upper bound"_warn_en_US);`。
- **L54 EN**: Transitions from the previous branch into the alternative path.
  **L54 CN**: 从前一个分支过渡到备选路径。

### Lines 55-72

````cpp
                  if constexpr (T::category == TypeCategory::Logical) { // C1148
                    if ((pair.first || pair.second) &&
                        (!pair.first || !pair.second ||
                            *pair.first != *pair.second)) {
                      context_.Say(stmt.source,
                          "CASE range is not allowed for LOGICAL"_err_en_US);
                    }
                  }
                  cases_.emplace_back(stmt);
                  cases_.back().lower = std::move(pair.first);
                  cases_.back().upper = std::move(pair.second);
                }
              }
            },
            [&](const parser::Default &) { cases_.emplace_front(stmt); },
        },
        selector.u);
  }
````
- **L55 EN**: Continues logic associated with callable symbol `constexpr`.
  **L55 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L56 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L56 CN**: 开始 `if` 控制流语句并计算其条件。
- **L57 EN**: Continues the surrounding expression or declaration: `(!pair.first || !pair.second ||`.
  **L57 CN**: 继续构造周围的表达式或声明：`(!pair.first || !pair.second ||`。
- **L58 EN**: Comment explains nearby logic, intent, or metadata: `pair.first != *pair.second)) {`.
  **L58 CN**: 注释说明附近代码的逻辑、意图或元数据：`pair.first != *pair.second)) {`。
- **L59 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(stmt.source,`.
  **L59 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(stmt.source,`。
- **L60 EN**: Executes a standalone statement or declaration: `"CASE range is not allowed for LOGICAL"_err_en_US);`.
  **L60 CN**: 执行一条独立语句或声明：`"CASE range is not allowed for LOGICAL"_err_en_US);`。
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Closes the current lexical scope or compound statement.
  **L62 CN**: 结束当前词法作用域或复合语句块。
- **L63 EN**: Executes a call or declaration centered on `cases_.emplace_back`.
  **L63 CN**: 执行以 `cases_.emplace_back` 为核心的调用或声明。
- **L64 EN**: Executes a call or declaration centered on `cases_.back`.
  **L64 CN**: 执行以 `cases_.back` 为核心的调用或声明。
- **L65 EN**: Executes a call or declaration centered on `cases_.back`.
  **L65 CN**: 执行以 `cases_.back` 为核心的调用或声明。
- **L66 EN**: Closes the current lexical scope or compound statement.
  **L66 CN**: 结束当前词法作用域或复合语句块。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L68 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L69 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const parser::Default &) { cases_.emplace_front(stmt); },`.
  **L69 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const parser::Default &) { cases_.emplace_front(stmt); },`。
- **L70 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L70 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L71 EN**: Executes a standalone statement or declaration: `selector.u);`.
  **L71 CN**: 执行一条独立语句或声明：`selector.u);`。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。

### Lines 73-90

````cpp

  std::optional<Value> GetValue(const parser::CaseValue &caseValue) {
    const auto &expr{parser::UnwrapRef<parser::Expr>(caseValue)};
    auto *x{expr.typedExpr.get()};
    if (x && x->v) { // C1147
      auto type{x->v->GetType()};
      if (type && type->category() == caseExprType_.category() &&
          (type->category() != TypeCategory::Character ||
              type->kind() == caseExprType_.kind())) {
        parser::Messages buffer; // discarded folding messages
        parser::ContextualMessages foldingMessages{expr.source, &buffer};
        evaluate::FoldingContext foldingContext{
            context_.foldingContext(), foldingMessages};
        auto folded{evaluate::Fold(foldingContext, SomeExpr{*x->v})};
        if (auto converted{evaluate::Fold(foldingContext,
                evaluate::ConvertToType(T::GetType(), SomeExpr{folded}))}) {
          if (auto value{evaluate::GetScalarConstantValue<T>(*converted)}) {
            auto back{evaluate::Fold(foldingContext,
````
- **L73 EN**: Blank line separating nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L74 EN**: Starts a function, method, lambda, or structured scope: `std::optional<Value> GetValue(const parser::CaseValue &caseValue) {`.
  **L74 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::optional<Value> GetValue(const parser::CaseValue &caseValue) {`。
- **L75 EN**: Executes a call or declaration centered on `&expr{parser::UnwrapRef<parser::Expr>`.
  **L75 CN**: 执行以 `&expr{parser::UnwrapRef<parser::Expr>` 为核心的调用或声明。
- **L76 EN**: Executes a call or declaration centered on `*x{expr.typedExpr.get`.
  **L76 CN**: 执行以 `*x{expr.typedExpr.get` 为核心的调用或声明。
- **L77 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L77 CN**: 开始 `if` 控制流语句并计算其条件。
- **L78 EN**: Executes a call or declaration centered on `type{x->v->GetType`.
  **L78 CN**: 执行以 `type{x->v->GetType` 为核心的调用或声明。
- **L79 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L79 CN**: 开始 `if` 控制流语句并计算其条件。
- **L80 EN**: Continues logic associated with callable symbol `category`.
  **L80 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L81 EN**: Starts a function, method, lambda, or structured scope: `type->kind() == caseExprType_.kind())) {`.
  **L81 CN**: 开始一个函数、方法、lambda 或结构化作用域：`type->kind() == caseExprType_.kind())) {`。
- **L82 EN**: Continues the surrounding expression or declaration: `parser::Messages buffer; // discarded folding messages`.
  **L82 CN**: 继续构造周围的表达式或声明：`parser::Messages buffer; // discarded folding messages`。
- **L83 EN**: Executes a standalone statement or declaration: `parser::ContextualMessages foldingMessages{expr.source, &buffer};`.
  **L83 CN**: 执行一条独立语句或声明：`parser::ContextualMessages foldingMessages{expr.source, &buffer};`。
- **L84 EN**: Continues the surrounding expression or declaration: `evaluate::FoldingContext foldingContext{`.
  **L84 CN**: 继续构造周围的表达式或声明：`evaluate::FoldingContext foldingContext{`。
- **L85 EN**: Executes a call or declaration centered on `context_.foldingContext`.
  **L85 CN**: 执行以 `context_.foldingContext` 为核心的调用或声明。
- **L86 EN**: Executes a call or declaration centered on `folded{evaluate::Fold`.
  **L86 CN**: 执行以 `folded{evaluate::Fold` 为核心的调用或声明。
- **L87 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L87 CN**: 开始 `if` 控制流语句并计算其条件。
- **L88 EN**: Starts a function, method, lambda, or structured scope: `evaluate::ConvertToType(T::GetType(), SomeExpr{folded}))}) {`.
  **L88 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::ConvertToType(T::GetType(), SomeExpr{folded}))}) {`。
- **L89 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L89 CN**: 开始 `if` 控制流语句并计算其条件。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto back{evaluate::Fold(foldingContext,`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto back{evaluate::Fold(foldingContext,`。

### Lines 91-108

````cpp
                evaluate::ConvertToType(*type, SomeExpr{*converted}))};
            if (back == folded) {
              x->v = converted;
              return value;
            } else {
              context_.Warn(common::UsageWarning::CaseOverflow, expr.source,
                  "CASE value (%s) overflows type (%s) of SELECT CASE expression"_warn_en_US,
                  folded.AsFortran(), caseExprType_.AsFortran());
              hasErrors_ = true;
              return std::nullopt;
            }
          }
        }
        context_.Say(expr.source,
            "CASE value (%s) must be a constant scalar"_err_en_US,
            x->v->AsFortran());
      } else {
        std::string typeStr{type ? type->AsFortran() : "typeless"s};
````
- **L91 EN**: Executes a call or declaration centered on `evaluate::ConvertToType`.
  **L91 CN**: 执行以 `evaluate::ConvertToType` 为核心的调用或声明。
- **L92 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L92 CN**: 开始 `if` 控制流语句并计算其条件。
- **L93 EN**: Executes a standalone statement or declaration: `x->v = converted;`.
  **L93 CN**: 执行一条独立语句或声明：`x->v = converted;`。
- **L94 EN**: Returns from the current function with `value`.
  **L94 CN**: 以 `value` 从当前函数返回。
- **L95 EN**: Transitions from the previous branch into the alternative path.
  **L95 CN**: 从前一个分支过渡到备选路径。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::UsageWarning::CaseOverflow, expr.source,`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::UsageWarning::CaseOverflow, expr.source,`。
- **L97 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"CASE value (%s) overflows type (%s) of SELECT CASE expression"_warn_en_US,`.
  **L97 CN**: 继续一个多行参数列表、初始化器或聚合项：`"CASE value (%s) overflows type (%s) of SELECT CASE expression"_warn_en_US,`。
- **L98 EN**: Executes a call or declaration centered on `folded.AsFortran`.
  **L98 CN**: 执行以 `folded.AsFortran` 为核心的调用或声明。
- **L99 EN**: Executes a standalone statement or declaration: `hasErrors_ = true;`.
  **L99 CN**: 执行一条独立语句或声明：`hasErrors_ = true;`。
- **L100 EN**: Returns from the current function with `std::nullopt`.
  **L100 CN**: 以 `std::nullopt` 从当前函数返回。
- **L101 EN**: Closes the current lexical scope or compound statement.
  **L101 CN**: 结束当前词法作用域或复合语句块。
- **L102 EN**: Closes the current lexical scope or compound statement.
  **L102 CN**: 结束当前词法作用域或复合语句块。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(expr.source,`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(expr.source,`。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"CASE value (%s) must be a constant scalar"_err_en_US,`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`"CASE value (%s) must be a constant scalar"_err_en_US,`。
- **L106 EN**: Executes a call or declaration centered on `x->v->AsFortran`.
  **L106 CN**: 执行以 `x->v->AsFortran` 为核心的调用或声明。
- **L107 EN**: Transitions from the previous branch into the alternative path.
  **L107 CN**: 从前一个分支过渡到备选路径。
- **L108 EN**: Executes a call or declaration centered on `type->AsFortran`.
  **L108 CN**: 执行以 `type->AsFortran` 为核心的调用或声明。

### Lines 109-126

````cpp
        context_.Say(expr.source,
            "CASE value has type '%s' which is not compatible with the SELECT CASE expression's type '%s'"_err_en_US,
            typeStr, caseExprType_.AsFortran());
      }
      hasErrors_ = true;
    }
    return std::nullopt;
  }

  using PairOfValues = std::pair<std::optional<Value>, std::optional<Value>>;
  PairOfValues ComputeBounds(const parser::CaseValueRange &range) {
    return common::visit(common::visitors{
                             [&](const parser::CaseValue &x) {
                               auto value{GetValue(x)};
                               return PairOfValues{value, value};
                             },
                             [&](const parser::CaseValueRange::Range &x) {
                               const auto &[lower, upper]{x.t};
````
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(expr.source,`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(expr.source,`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"CASE value has type '%s' which is not compatible with the SELECT CASE expression's type '%s'"_err_en_US,`.
  **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`"CASE value has type '%s' which is not compatible with the SELECT CASE expression's type '%s'"_err_en_US,`。
- **L111 EN**: Executes a call or declaration centered on `caseExprType_.AsFortran`.
  **L111 CN**: 执行以 `caseExprType_.AsFortran` 为核心的调用或声明。
- **L112 EN**: Closes the current lexical scope or compound statement.
  **L112 CN**: 结束当前词法作用域或复合语句块。
- **L113 EN**: Executes a standalone statement or declaration: `hasErrors_ = true;`.
  **L113 CN**: 执行一条独立语句或声明：`hasErrors_ = true;`。
- **L114 EN**: Closes the current lexical scope or compound statement.
  **L114 CN**: 结束当前词法作用域或复合语句块。
- **L115 EN**: Returns from the current function with `std::nullopt`.
  **L115 CN**: 以 `std::nullopt` 从当前函数返回。
- **L116 EN**: Closes the current lexical scope or compound statement.
  **L116 CN**: 结束当前词法作用域或复合语句块。
- **L117 EN**: Blank line separating nearby declarations or logic blocks.
  **L117 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L118 EN**: Defines alias `PairOfValues` to simplify later code.
  **L118 CN**: 定义别名 `PairOfValues` 以简化后续代码。
- **L119 EN**: Starts a function, method, lambda, or structured scope: `PairOfValues ComputeBounds(const parser::CaseValueRange &range) {`.
  **L119 CN**: 开始一个函数、方法、lambda 或结构化作用域：`PairOfValues ComputeBounds(const parser::CaseValueRange &range) {`。
- **L120 EN**: Returns from the current function with `common::visit(common::visitors{`.
  **L120 CN**: 以 `common::visit(common::visitors{` 从当前函数返回。
- **L121 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::CaseValue &x) {`.
  **L121 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::CaseValue &x) {`。
- **L122 EN**: Executes a call or declaration centered on `value{GetValue`.
  **L122 CN**: 执行以 `value{GetValue` 为核心的调用或声明。
- **L123 EN**: Returns from the current function with `PairOfValues{value, value}`.
  **L123 CN**: 以 `PairOfValues{value, value}` 从当前函数返回。
- **L124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L124 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L125 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::CaseValueRange::Range &x) {`.
  **L125 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::CaseValueRange::Range &x) {`。
- **L126 EN**: Executes a standalone statement or declaration: `const auto &[lower, upper]{x.t};`.
  **L126 CN**: 执行一条独立语句或声明：`const auto &[lower, upper]{x.t};`。

### Lines 127-144

````cpp
                               std::optional<Value> lo, hi;
                               if (lower) {
                                 lo = GetValue(*lower);
                               }
                               if (upper) {
                                 hi = GetValue(*upper);
                               }
                               if ((lower && !lo) || (upper && !hi)) {
                                 return PairOfValues{}; // error case
                               }
                               return PairOfValues{
                                   std::move(lo), std::move(hi)};
                             },
                         },
        range.u);
  }

  struct Case {
````
- **L127 EN**: Executes a standalone statement or declaration: `std::optional<Value> lo, hi;`.
  **L127 CN**: 执行一条独立语句或声明：`std::optional<Value> lo, hi;`。
- **L128 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L128 CN**: 开始 `if` 控制流语句并计算其条件。
- **L129 EN**: Executes a call or declaration centered on `GetValue`.
  **L129 CN**: 执行以 `GetValue` 为核心的调用或声明。
- **L130 EN**: Closes the current lexical scope or compound statement.
  **L130 CN**: 结束当前词法作用域或复合语句块。
- **L131 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L131 CN**: 开始 `if` 控制流语句并计算其条件。
- **L132 EN**: Executes a call or declaration centered on `GetValue`.
  **L132 CN**: 执行以 `GetValue` 为核心的调用或声明。
- **L133 EN**: Closes the current lexical scope or compound statement.
  **L133 CN**: 结束当前词法作用域或复合语句块。
- **L134 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L134 CN**: 开始 `if` 控制流语句并计算其条件。
- **L135 EN**: Returns from the current function with `PairOfValues{}; // error case`.
  **L135 CN**: 以 `PairOfValues{}; // error case` 从当前函数返回。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Returns from the current function with `PairOfValues{`.
  **L137 CN**: 以 `PairOfValues{` 从当前函数返回。
- **L138 EN**: Executes a call or declaration centered on `std::move`.
  **L138 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L139 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L141 EN**: Executes a standalone statement or declaration: `range.u);`.
  **L141 CN**: 执行一条独立语句或声明：`range.u);`。
- **L142 EN**: Closes the current lexical scope or compound statement.
  **L142 CN**: 结束当前词法作用域或复合语句块。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L144 EN**: Declares struct `Case`.
  **L144 CN**: 声明 struct `Case`。

### Lines 145-162

````cpp
    explicit Case(const parser::Statement<parser::CaseStmt> &s) : stmt{s} {}
    bool IsDefault() const { return !lower && !upper; }
    std::string AsFortran() const {
      std::string result;
      {
        llvm::raw_string_ostream bs{result};
        if (lower) {
          evaluate::Constant<T>{*lower}.AsFortran(bs << '(');
          if (!upper) {
            bs << ':';
          } else if (*lower != *upper) {
            evaluate::Constant<T>{*upper}.AsFortran(bs << ':');
          }
          bs << ')';
        } else if (upper) {
          evaluate::Constant<T>{*upper}.AsFortran(bs << "(:") << ')';
        } else {
          bs << "DEFAULT";
````
- **L145 EN**: Continues logic associated with callable symbol `Case`.
  **L145 CN**: 继续与可调用符号 `Case` 相关的逻辑。
- **L146 EN**: Continues logic associated with callable symbol `IsDefault`.
  **L146 CN**: 继续与可调用符号 `IsDefault` 相关的逻辑。
- **L147 EN**: Starts a function, method, lambda, or structured scope: `std::string AsFortran() const {`.
  **L147 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::string AsFortran() const {`。
- **L148 EN**: Executes a standalone statement or declaration: `std::string result;`.
  **L148 CN**: 执行一条独立语句或声明：`std::string result;`。
- **L149 EN**: Opens a new lexical scope or compound statement.
  **L149 CN**: 打开一个新的词法作用域或复合语句块。
- **L150 EN**: Executes a standalone statement or declaration: `llvm::raw_string_ostream bs{result};`.
  **L150 CN**: 执行一条独立语句或声明：`llvm::raw_string_ostream bs{result};`。
- **L151 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L151 CN**: 开始 `if` 控制流语句并计算其条件。
- **L152 EN**: Executes a call or declaration centered on `evaluate::Constant<T>{*lower}.AsFortran`.
  **L152 CN**: 执行以 `evaluate::Constant<T>{*lower}.AsFortran` 为核心的调用或声明。
- **L153 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L153 CN**: 开始 `if` 控制流语句并计算其条件。
- **L154 EN**: Executes a standalone statement or declaration: `bs << ':';`.
  **L154 CN**: 执行一条独立语句或声明：`bs << ':';`。
- **L155 EN**: Transitions from the previous branch into an `else if` condition.
  **L155 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L156 EN**: Executes a call or declaration centered on `evaluate::Constant<T>{*upper}.AsFortran`.
  **L156 CN**: 执行以 `evaluate::Constant<T>{*upper}.AsFortran` 为核心的调用或声明。
- **L157 EN**: Closes the current lexical scope or compound statement.
  **L157 CN**: 结束当前词法作用域或复合语句块。
- **L158 EN**: Executes a standalone statement or declaration: `bs << ')';`.
  **L158 CN**: 执行一条独立语句或声明：`bs << ')';`。
- **L159 EN**: Transitions from the previous branch into an `else if` condition.
  **L159 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L160 EN**: Executes a call or declaration centered on `evaluate::Constant<T>{*upper}.AsFortran`.
  **L160 CN**: 执行以 `evaluate::Constant<T>{*upper}.AsFortran` 为核心的调用或声明。
- **L161 EN**: Transitions from the previous branch into the alternative path.
  **L161 CN**: 从前一个分支过渡到备选路径。
- **L162 EN**: Executes a standalone statement or declaration: `bs << "DEFAULT";`.
  **L162 CN**: 执行一条独立语句或声明：`bs << "DEFAULT";`。

### Lines 163-180

````cpp
        }
      }
      return result;
    }

    const parser::Statement<parser::CaseStmt> &stmt;
    std::optional<Value> lower, upper;
  };

  // Defines a comparator for use with std::list<>::sort().
  // Returns true if and only if the highest value in range x is less
  // than the least value in range y.  The DEFAULT case is arbitrarily
  // defined to be less than all others.  When two ranges overlap,
  // neither is less than the other.
  struct Comparator {
    bool operator()(const Case &x, const Case &y) const {
      if (x.IsDefault()) {
        return !y.IsDefault();
````
- **L163 EN**: Closes the current lexical scope or compound statement.
  **L163 CN**: 结束当前词法作用域或复合语句块。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Returns from the current function with `result`.
  **L165 CN**: 以 `result` 从当前函数返回。
- **L166 EN**: Closes the current lexical scope or compound statement.
  **L166 CN**: 结束当前词法作用域或复合语句块。
- **L167 EN**: Blank line separating nearby declarations or logic blocks.
  **L167 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L168 EN**: Executes a standalone statement or declaration: `const parser::Statement<parser::CaseStmt> &stmt;`.
  **L168 CN**: 执行一条独立语句或声明：`const parser::Statement<parser::CaseStmt> &stmt;`。
- **L169 EN**: Executes a standalone statement or declaration: `std::optional<Value> lower, upper;`.
  **L169 CN**: 执行一条独立语句或声明：`std::optional<Value> lower, upper;`。
- **L170 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L170 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L172 EN**: Comment explains nearby logic, intent, or metadata: `Defines a comparator for use with std::list<>::sort().`.
  **L172 CN**: 注释说明附近代码的逻辑、意图或元数据：`Defines a comparator for use with std::list<>::sort().`。
- **L173 EN**: Comment explains nearby logic, intent, or metadata: `Returns true if and only if the highest value in range x is less`.
  **L173 CN**: 注释说明附近代码的逻辑、意图或元数据：`Returns true if and only if the highest value in range x is less`。
- **L174 EN**: Comment explains nearby logic, intent, or metadata: `than the least value in range y.  The DEFAULT case is arbitrarily`.
  **L174 CN**: 注释说明附近代码的逻辑、意图或元数据：`than the least value in range y.  The DEFAULT case is arbitrarily`。
- **L175 EN**: Comment explains nearby logic, intent, or metadata: `defined to be less than all others.  When two ranges overlap,`.
  **L175 CN**: 注释说明附近代码的逻辑、意图或元数据：`defined to be less than all others.  When two ranges overlap,`。
- **L176 EN**: Comment explains nearby logic, intent, or metadata: `neither is less than the other.`.
  **L176 CN**: 注释说明附近代码的逻辑、意图或元数据：`neither is less than the other.`。
- **L177 EN**: Declares struct `Comparator`.
  **L177 CN**: 声明 struct `Comparator`。
- **L178 EN**: Starts a function, method, lambda, or structured scope: `bool operator()(const Case &x, const Case &y) const {`.
  **L178 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool operator()(const Case &x, const Case &y) const {`。
- **L179 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L179 CN**: 开始 `if` 控制流语句并计算其条件。
- **L180 EN**: Returns from the current function with `!y.IsDefault()`.
  **L180 CN**: 以 `!y.IsDefault()` 从当前函数返回。

### Lines 181-198

````cpp
      } else {
        return x.upper && y.lower && *x.upper < *y.lower;
      }
    }
  };

  bool AreCasesDisjoint() const {
    auto endIter{cases_.end()};
    for (auto iter{cases_.begin()}; iter != endIter; ++iter) {
      auto next{iter};
      if (++next != endIter && !Comparator{}(*iter, *next)) {
        return false;
      }
    }
    return true;
  }

  // This has quadratic time, but only runs in error cases
````
- **L181 EN**: Transitions from the previous branch into the alternative path.
  **L181 CN**: 从前一个分支过渡到备选路径。
- **L182 EN**: Returns from the current function with `x.upper && y.lower && *x.upper < *y.lower`.
  **L182 CN**: 以 `x.upper && y.lower && *x.upper < *y.lower` 从当前函数返回。
- **L183 EN**: Closes the current lexical scope or compound statement.
  **L183 CN**: 结束当前词法作用域或复合语句块。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L185 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L186 EN**: Blank line separating nearby declarations or logic blocks.
  **L186 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L187 EN**: Starts a function, method, lambda, or structured scope: `bool AreCasesDisjoint() const {`.
  **L187 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool AreCasesDisjoint() const {`。
- **L188 EN**: Executes a call or declaration centered on `endIter{cases_.end`.
  **L188 CN**: 执行以 `endIter{cases_.end` 为核心的调用或声明。
- **L189 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L189 CN**: 开始 `for` 控制流语句并计算其条件。
- **L190 EN**: Executes a standalone statement or declaration: `auto next{iter};`.
  **L190 CN**: 执行一条独立语句或声明：`auto next{iter};`。
- **L191 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L191 CN**: 开始 `if` 控制流语句并计算其条件。
- **L192 EN**: Returns from the current function with `false`.
  **L192 CN**: 以 `false` 从当前函数返回。
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Closes the current lexical scope or compound statement.
  **L194 CN**: 结束当前词法作用域或复合语句块。
- **L195 EN**: Returns from the current function with `true`.
  **L195 CN**: 以 `true` 从当前函数返回。
- **L196 EN**: Closes the current lexical scope or compound statement.
  **L196 CN**: 结束当前词法作用域或复合语句块。
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L198 EN**: Comment explains nearby logic, intent, or metadata: `This has quadratic time, but only runs in error cases`.
  **L198 CN**: 注释说明附近代码的逻辑、意图或元数据：`This has quadratic time, but only runs in error cases`。

### Lines 199-216

````cpp
  void ReportConflictingCases() {
    for (auto iter{cases_.begin()}; iter != cases_.end(); ++iter) {
      parser::Message *msg{nullptr};
      for (auto p{cases_.begin()}; p != cases_.end(); ++p) {
        if (p->stmt.source.begin() < iter->stmt.source.begin() &&
            !Comparator{}(*p, *iter) && !Comparator{}(*iter, *p)) {
          if (!msg) {
            msg = &context_.Say(iter->stmt.source,
                "CASE %s conflicts with previous cases"_err_en_US,
                iter->AsFortran());
          }
          msg->Attach(
              p->stmt.source, "Conflicting CASE %s"_en_US, p->AsFortran());
        }
      }
    }
  }

````
- **L199 EN**: Starts a function, method, lambda, or structured scope: `void ReportConflictingCases() {`.
  **L199 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void ReportConflictingCases() {`。
- **L200 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L200 CN**: 开始 `for` 控制流语句并计算其条件。
- **L201 EN**: Executes a standalone statement or declaration: `parser::Message *msg{nullptr};`.
  **L201 CN**: 执行一条独立语句或声明：`parser::Message *msg{nullptr};`。
- **L202 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L202 CN**: 开始 `for` 控制流语句并计算其条件。
- **L203 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L203 CN**: 开始 `if` 控制流语句并计算其条件。
- **L204 EN**: Starts a function, method, lambda, or structured scope: `!Comparator{}(*p, *iter) && !Comparator{}(*iter, *p)) {`.
  **L204 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!Comparator{}(*p, *iter) && !Comparator{}(*iter, *p)) {`。
- **L205 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L205 CN**: 开始 `if` 控制流语句并计算其条件。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg = &context_.Say(iter->stmt.source,`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg = &context_.Say(iter->stmt.source,`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"CASE %s conflicts with previous cases"_err_en_US,`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`"CASE %s conflicts with previous cases"_err_en_US,`。
- **L208 EN**: Executes a call or declaration centered on `iter->AsFortran`.
  **L208 CN**: 执行以 `iter->AsFortran` 为核心的调用或声明。
- **L209 EN**: Closes the current lexical scope or compound statement.
  **L209 CN**: 结束当前词法作用域或复合语句块。
- **L210 EN**: Continues logic associated with callable symbol `Attach`.
  **L210 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L211 EN**: Executes a call or declaration centered on `p->AsFortran`.
  **L211 CN**: 执行以 `p->AsFortran` 为核心的调用或声明。
- **L212 EN**: Closes the current lexical scope or compound statement.
  **L212 CN**: 结束当前词法作用域或复合语句块。
- **L213 EN**: Closes the current lexical scope or compound statement.
  **L213 CN**: 结束当前词法作用域或复合语句块。
- **L214 EN**: Closes the current lexical scope or compound statement.
  **L214 CN**: 结束当前词法作用域或复合语句块。
- **L215 EN**: Closes the current lexical scope or compound statement.
  **L215 CN**: 结束当前词法作用域或复合语句块。
- **L216 EN**: Blank line separating nearby declarations or logic blocks.
  **L216 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 217-234

````cpp
  SemanticsContext &context_;
  const evaluate::DynamicType &caseExprType_;
  std::list<Case> cases_;
  bool hasErrors_{false};
};

template <TypeCategory CAT> struct TypeVisitor {
  using Result = bool;
  using Types = evaluate::CategoryTypes<CAT>;
  template <typename T> Result Test() {
    if (T::kind == exprType.kind()) {
      CaseValues<T>(context, exprType).Check(caseList);
      return true;
    } else {
      return false;
    }
  }
  SemanticsContext &context;
````
- **L217 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L217 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。
- **L218 EN**: Executes a standalone statement or declaration: `const evaluate::DynamicType &caseExprType_;`.
  **L218 CN**: 执行一条独立语句或声明：`const evaluate::DynamicType &caseExprType_;`。
- **L219 EN**: Executes a standalone statement or declaration: `std::list<Case> cases_;`.
  **L219 CN**: 执行一条独立语句或声明：`std::list<Case> cases_;`。
- **L220 EN**: Executes a standalone statement or declaration: `bool hasErrors_{false};`.
  **L220 CN**: 执行一条独立语句或声明：`bool hasErrors_{false};`。
- **L221 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L221 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L222 EN**: Blank line separating nearby declarations or logic blocks.
  **L222 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L223 EN**: Introduces template parameters or specialization context: `template <TypeCategory CAT> struct TypeVisitor {`.
  **L223 CN**: 为后续声明引入模板参数或特化上下文：`template <TypeCategory CAT> struct TypeVisitor {`。
- **L224 EN**: Defines alias `Result` to simplify later code.
  **L224 CN**: 定义别名 `Result` 以简化后续代码。
- **L225 EN**: Defines alias `Types` to simplify later code.
  **L225 CN**: 定义别名 `Types` 以简化后续代码。
- **L226 EN**: Introduces template parameters or specialization context: `template <typename T> Result Test() {`.
  **L226 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> Result Test() {`。
- **L227 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L227 CN**: 开始 `if` 控制流语句并计算其条件。
- **L228 EN**: Executes a call or declaration centered on `CaseValues<T>`.
  **L228 CN**: 执行以 `CaseValues<T>` 为核心的调用或声明。
- **L229 EN**: Returns from the current function with `true`.
  **L229 CN**: 以 `true` 从当前函数返回。
- **L230 EN**: Transitions from the previous branch into the alternative path.
  **L230 CN**: 从前一个分支过渡到备选路径。
- **L231 EN**: Returns from the current function with `false`.
  **L231 CN**: 以 `false` 从当前函数返回。
- **L232 EN**: Closes the current lexical scope or compound statement.
  **L232 CN**: 结束当前词法作用域或复合语句块。
- **L233 EN**: Closes the current lexical scope or compound statement.
  **L233 CN**: 结束当前词法作用域或复合语句块。
- **L234 EN**: Executes a standalone statement or declaration: `SemanticsContext &context;`.
  **L234 CN**: 执行一条独立语句或声明：`SemanticsContext &context;`。

### Lines 235-252

````cpp
  const evaluate::DynamicType &exprType;
  const std::list<parser::CaseConstruct::Case> &caseList;
};

void CaseChecker::Enter(const parser::CaseConstruct &construct) {
  const auto &selectCaseStmt{
      std::get<parser::Statement<parser::SelectCaseStmt>>(construct.t)};
  const auto &selectCase{selectCaseStmt.statement};
  const auto &selectExpr{
      std::get<parser::Scalar<parser::Expr>>(selectCase.t).thing};
  const auto *x{GetExpr(context_, selectExpr)};
  if (!x) {
    return; // expression semantics failed
  }
  if (auto exprType{x->GetType()}) {
    const auto &caseList{
        std::get<std::list<parser::CaseConstruct::Case>>(construct.t)};
    switch (exprType->category()) {
````
- **L235 EN**: Executes a standalone statement or declaration: `const evaluate::DynamicType &exprType;`.
  **L235 CN**: 执行一条独立语句或声明：`const evaluate::DynamicType &exprType;`。
- **L236 EN**: Executes a standalone statement or declaration: `const std::list<parser::CaseConstruct::Case> &caseList;`.
  **L236 CN**: 执行一条独立语句或声明：`const std::list<parser::CaseConstruct::Case> &caseList;`。
- **L237 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L237 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L238 EN**: Blank line separating nearby declarations or logic blocks.
  **L238 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L239 EN**: Starts a function, method, lambda, or structured scope: `void CaseChecker::Enter(const parser::CaseConstruct &construct) {`.
  **L239 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CaseChecker::Enter(const parser::CaseConstruct &construct) {`。
- **L240 EN**: Continues the surrounding expression or declaration: `const auto &selectCaseStmt{`.
  **L240 CN**: 继续构造周围的表达式或声明：`const auto &selectCaseStmt{`。
- **L241 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::SelectCaseStmt>>`.
  **L241 CN**: 执行以 `std::get<parser::Statement<parser::SelectCaseStmt>>` 为核心的调用或声明。
- **L242 EN**: Executes a standalone statement or declaration: `const auto &selectCase{selectCaseStmt.statement};`.
  **L242 CN**: 执行一条独立语句或声明：`const auto &selectCase{selectCaseStmt.statement};`。
- **L243 EN**: Continues the surrounding expression or declaration: `const auto &selectExpr{`.
  **L243 CN**: 继续构造周围的表达式或声明：`const auto &selectExpr{`。
- **L244 EN**: Executes a call or declaration centered on `std::get<parser::Scalar<parser::Expr>>`.
  **L244 CN**: 执行以 `std::get<parser::Scalar<parser::Expr>>` 为核心的调用或声明。
- **L245 EN**: Executes a call or declaration centered on `*x{GetExpr`.
  **L245 CN**: 执行以 `*x{GetExpr` 为核心的调用或声明。
- **L246 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L246 CN**: 开始 `if` 控制流语句并计算其条件。
- **L247 EN**: Returns from the current function with `; // expression semantics failed`.
  **L247 CN**: 以 `; // expression semantics failed` 从当前函数返回。
- **L248 EN**: Closes the current lexical scope or compound statement.
  **L248 CN**: 结束当前词法作用域或复合语句块。
- **L249 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L249 CN**: 开始 `if` 控制流语句并计算其条件。
- **L250 EN**: Continues the surrounding expression or declaration: `const auto &caseList{`.
  **L250 CN**: 继续构造周围的表达式或声明：`const auto &caseList{`。
- **L251 EN**: Executes a call or declaration centered on `std::get<std::list<parser::CaseConstruct::Case>>`.
  **L251 CN**: 执行以 `std::get<std::list<parser::CaseConstruct::Case>>` 为核心的调用或声明。
- **L252 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L252 CN**: 开始 `switch` 控制流语句并计算其条件。

### Lines 253-270

````cpp
    case TypeCategory::Integer:
      common::SearchTypes(
          TypeVisitor<TypeCategory::Integer>{context_, *exprType, caseList});
      return;
    case TypeCategory::Unsigned:
      common::SearchTypes(
          TypeVisitor<TypeCategory::Unsigned>{context_, *exprType, caseList});
      return;
    case TypeCategory::Logical:
      CaseValues<evaluate::Type<TypeCategory::Logical, 1>>{context_, *exprType}
          .Check(caseList);
      return;
    case TypeCategory::Character:
      common::SearchTypes(
          TypeVisitor<TypeCategory::Character>{context_, *exprType, caseList});
      return;
    default:
      break;
````
- **L253 EN**: Introduces a switch dispatch label: `case TypeCategory::Integer:`.
  **L253 CN**: 引入一个 switch 分发标签：`case TypeCategory::Integer:`。
- **L254 EN**: Continues logic associated with callable symbol `SearchTypes`.
  **L254 CN**: 继续与可调用符号 `SearchTypes` 相关的逻辑。
- **L255 EN**: Executes a standalone statement or declaration: `TypeVisitor<TypeCategory::Integer>{context_, *exprType, caseList});`.
  **L255 CN**: 执行一条独立语句或声明：`TypeVisitor<TypeCategory::Integer>{context_, *exprType, caseList});`。
- **L256 EN**: Returns from the current function with `void`.
  **L256 CN**: 以 `void` 从当前函数返回。
- **L257 EN**: Introduces a switch dispatch label: `case TypeCategory::Unsigned:`.
  **L257 CN**: 引入一个 switch 分发标签：`case TypeCategory::Unsigned:`。
- **L258 EN**: Continues logic associated with callable symbol `SearchTypes`.
  **L258 CN**: 继续与可调用符号 `SearchTypes` 相关的逻辑。
- **L259 EN**: Executes a standalone statement or declaration: `TypeVisitor<TypeCategory::Unsigned>{context_, *exprType, caseList});`.
  **L259 CN**: 执行一条独立语句或声明：`TypeVisitor<TypeCategory::Unsigned>{context_, *exprType, caseList});`。
- **L260 EN**: Returns from the current function with `void`.
  **L260 CN**: 以 `void` 从当前函数返回。
- **L261 EN**: Introduces a switch dispatch label: `case TypeCategory::Logical:`.
  **L261 CN**: 引入一个 switch 分发标签：`case TypeCategory::Logical:`。
- **L262 EN**: Continues the surrounding expression or declaration: `CaseValues<evaluate::Type<TypeCategory::Logical, 1>>{context_, *exprType}`.
  **L262 CN**: 继续构造周围的表达式或声明：`CaseValues<evaluate::Type<TypeCategory::Logical, 1>>{context_, *exprType}`。
- **L263 EN**: Executes a call or declaration centered on `.Check`.
  **L263 CN**: 执行以 `.Check` 为核心的调用或声明。
- **L264 EN**: Returns from the current function with `void`.
  **L264 CN**: 以 `void` 从当前函数返回。
- **L265 EN**: Introduces a switch dispatch label: `case TypeCategory::Character:`.
  **L265 CN**: 引入一个 switch 分发标签：`case TypeCategory::Character:`。
- **L266 EN**: Continues logic associated with callable symbol `SearchTypes`.
  **L266 CN**: 继续与可调用符号 `SearchTypes` 相关的逻辑。
- **L267 EN**: Executes a standalone statement or declaration: `TypeVisitor<TypeCategory::Character>{context_, *exprType, caseList});`.
  **L267 CN**: 执行一条独立语句或声明：`TypeVisitor<TypeCategory::Character>{context_, *exprType, caseList});`。
- **L268 EN**: Returns from the current function with `void`.
  **L268 CN**: 以 `void` 从当前函数返回。
- **L269 EN**: Introduces a switch dispatch label: `default:`.
  **L269 CN**: 引入一个 switch 分发标签：`default:`。
- **L270 EN**: Exits the nearest loop or switch statement.
  **L270 CN**: 退出最近的循环或 switch 语句。

### Lines 271-278

````cpp
    }
  }
  context_.Say(selectExpr.source,
      context_.IsEnabled(common::LanguageFeature::Unsigned)
          ? "SELECT CASE expression must be integer, unsigned, logical, or character"_err_en_US
          : "SELECT CASE expression must be integer, logical, or character"_err_en_US);
}
} // namespace Fortran::semantics
````
- **L271 EN**: Closes the current lexical scope or compound statement.
  **L271 CN**: 结束当前词法作用域或复合语句块。
- **L272 EN**: Closes the current lexical scope or compound statement.
  **L272 CN**: 结束当前词法作用域或复合语句块。
- **L273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(selectExpr.source,`.
  **L273 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(selectExpr.source,`。
- **L274 EN**: Continues logic associated with callable symbol `IsEnabled`.
  **L274 CN**: 继续与可调用符号 `IsEnabled` 相关的逻辑。
- **L275 EN**: Continues the surrounding expression or declaration: `? "SELECT CASE expression must be integer, unsigned, logical, or character"_err_en_US`.
  **L275 CN**: 继续构造周围的表达式或声明：`? "SELECT CASE expression must be integer, unsigned, logical, or character"_err_en_US`。
- **L276 EN**: Executes a standalone statement or declaration: `: "SELECT CASE expression must be integer, logical, or character"_err_en_US);`.
  **L276 CN**: 执行一条独立语句或声明：`: "SELECT CASE expression must be integer, logical, or character"_err_en_US);`。
- **L277 EN**: Closes the current lexical scope or compound statement.
  **L277 CN**: 结束当前词法作用域或复合语句块。
- **L278 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L278 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Fortran parse tree handling / Fortran 语法树处理**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Semantic-context management / 语义上下文管理**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **Constant folding / 常量折叠**
- **Evaluation context management / 求值上下文管理**
- **Rule-checking workflow / 规则检查流程**

## Dependencies / 依赖关系

- `check-case.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Common/idioms.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Common/reference.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Common/template.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Evaluate/fold.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/type.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Parser/parse-tree.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/semantics.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `tuple`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
