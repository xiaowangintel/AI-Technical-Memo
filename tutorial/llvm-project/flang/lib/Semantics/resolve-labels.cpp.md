# resolve-labels.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/resolve-labels.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements semantic checking, symbol reasoning, and language-rule enforcement for resolve labels.
- **Purpose (CN)**: 实现 resolve labels 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/Semantics/resolve-labels.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "resolve-labels.h"
#include "flang/Common/enum-set.h"
#include "flang/Common/template.h"
#include "flang/Parser/parse-tree-visitor.h"
#include "flang/Semantics/semantics.h"
#include <cstdarg>
#include <type_traits>

namespace Fortran::semantics {

using namespace parser::literals;

ENUM_CLASS(
    TargetStatementEnum, Do, Branch, Format, CompatibleDo, CompatibleBranch)
using LabeledStmtClassificationSet =
    common::EnumSet<TargetStatementEnum, TargetStatementEnum_enumSize>;
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
- **L9 EN**: Includes "resolve-labels.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "resolve-labels.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "flang/Common/enum-set.h" to access shared Flang utility infrastructure.
  **L10 CN**: 引入 "flang/Common/enum-set.h" 以使用Flang 共享工具基础设施。
- **L11 EN**: Includes "flang/Common/template.h" to access shared Flang utility infrastructure.
  **L11 CN**: 引入 "flang/Common/template.h" 以使用Flang 共享工具基础设施。
- **L12 EN**: Includes "flang/Parser/parse-tree-visitor.h" to access parse-tree, token, or source representation support.
  **L12 CN**: 引入 "flang/Parser/parse-tree-visitor.h" 以使用语法树、词法单元或源码表示支持。
- **L13 EN**: Includes "flang/Semantics/semantics.h" to access Fortran semantic analysis, symbol, and type information.
  **L13 CN**: 引入 "flang/Semantics/semantics.h" 以使用Fortran 语义分析、符号与类型信息。
- **L14 EN**: Includes <cstdarg> to access supporting declarations used by this translation unit.
  **L14 CN**: 引入 <cstdarg> 以使用当前编译单元使用的辅助声明。
- **L15 EN**: Includes <type_traits> to access supporting declarations used by this translation unit.
  **L15 CN**: 引入 <type_traits> 以使用当前编译单元使用的辅助声明。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L17 EN**: Opens namespace scope `Fortran::semantics`.
  **L17 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Brings namespace `parser::literals` into the local scope.
  **L19 CN**: 将命名空间 `parser::literals` 引入当前作用域。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L21 EN**: Continues logic associated with callable symbol `ENUM_CLASS`.
  **L21 CN**: 继续与可调用符号 `ENUM_CLASS` 相关的逻辑。
- **L22 EN**: Continues the surrounding expression or declaration: `TargetStatementEnum, Do, Branch, Format, CompatibleDo, CompatibleBranch)`.
  **L22 CN**: 继续构造周围的表达式或声明：`TargetStatementEnum, Do, Branch, Format, CompatibleDo, CompatibleBranch)`。
- **L23 EN**: Defines alias `LabeledStmtClassificationSet` to simplify later code.
  **L23 CN**: 定义别名 `LabeledStmtClassificationSet` 以简化后续代码。
- **L24 EN**: Executes a standalone statement or declaration: `common::EnumSet<TargetStatementEnum, TargetStatementEnum_enumSize>;`.
  **L24 CN**: 执行一条独立语句或声明：`common::EnumSet<TargetStatementEnum, TargetStatementEnum_enumSize>;`。

### Lines 25-48

````cpp

using IndexList = std::vector<std::pair<parser::CharBlock, parser::CharBlock>>;
// A ProxyForScope is an integral proxy for a Fortran scope. This is required
// because the parse tree does not actually have the scopes required.
using ProxyForScope = unsigned;
// Minimal scope information
struct ScopeInfo {
  ProxyForScope parent{};
  bool isExteriorGotoFatal{false};
  int depth{0};
};
struct LabeledStatementInfoTuplePOD {
  ProxyForScope proxyForScope;
  parser::CharBlock parserCharBlock;
  LabeledStmtClassificationSet labeledStmtClassificationSet;
  bool isExecutableConstructEndStmt;
};
using TargetStmtMap = std::map<parser::Label, LabeledStatementInfoTuplePOD>;
struct SourceStatementInfoTuplePOD {
  SourceStatementInfoTuplePOD(const parser::Label &parserLabel,
      const ProxyForScope &proxyForScope,
      const parser::CharBlock &parserCharBlock)
      : parserLabel{parserLabel}, proxyForScope{proxyForScope},
        parserCharBlock{parserCharBlock} {}
````
- **L25 EN**: Blank line separating nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L26 EN**: Defines alias `IndexList` to simplify later code.
  **L26 CN**: 定义别名 `IndexList` 以简化后续代码。
- **L27 EN**: Comment explains nearby logic, intent, or metadata: `A ProxyForScope is an integral proxy for a Fortran scope. This is required`.
  **L27 CN**: 注释说明附近代码的逻辑、意图或元数据：`A ProxyForScope is an integral proxy for a Fortran scope. This is required`。
- **L28 EN**: Comment explains nearby logic, intent, or metadata: `because the parse tree does not actually have the scopes required.`.
  **L28 CN**: 注释说明附近代码的逻辑、意图或元数据：`because the parse tree does not actually have the scopes required.`。
- **L29 EN**: Defines alias `ProxyForScope` to simplify later code.
  **L29 CN**: 定义别名 `ProxyForScope` 以简化后续代码。
- **L30 EN**: Comment explains nearby logic, intent, or metadata: `Minimal scope information`.
  **L30 CN**: 注释说明附近代码的逻辑、意图或元数据：`Minimal scope information`。
- **L31 EN**: Declares struct `ScopeInfo`.
  **L31 CN**: 声明 struct `ScopeInfo`。
- **L32 EN**: Executes a standalone statement or declaration: `ProxyForScope parent{};`.
  **L32 CN**: 执行一条独立语句或声明：`ProxyForScope parent{};`。
- **L33 EN**: Executes a standalone statement or declaration: `bool isExteriorGotoFatal{false};`.
  **L33 CN**: 执行一条独立语句或声明：`bool isExteriorGotoFatal{false};`。
- **L34 EN**: Executes a standalone statement or declaration: `int depth{0};`.
  **L34 CN**: 执行一条独立语句或声明：`int depth{0};`。
- **L35 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L35 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L36 EN**: Declares struct `LabeledStatementInfoTuplePOD`.
  **L36 CN**: 声明 struct `LabeledStatementInfoTuplePOD`。
- **L37 EN**: Executes a standalone statement or declaration: `ProxyForScope proxyForScope;`.
  **L37 CN**: 执行一条独立语句或声明：`ProxyForScope proxyForScope;`。
- **L38 EN**: Executes a standalone statement or declaration: `parser::CharBlock parserCharBlock;`.
  **L38 CN**: 执行一条独立语句或声明：`parser::CharBlock parserCharBlock;`。
- **L39 EN**: Executes a standalone statement or declaration: `LabeledStmtClassificationSet labeledStmtClassificationSet;`.
  **L39 CN**: 执行一条独立语句或声明：`LabeledStmtClassificationSet labeledStmtClassificationSet;`。
- **L40 EN**: Executes a standalone statement or declaration: `bool isExecutableConstructEndStmt;`.
  **L40 CN**: 执行一条独立语句或声明：`bool isExecutableConstructEndStmt;`。
- **L41 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L41 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L42 EN**: Defines alias `TargetStmtMap` to simplify later code.
  **L42 CN**: 定义别名 `TargetStmtMap` 以简化后续代码。
- **L43 EN**: Declares struct `SourceStatementInfoTuplePOD`.
  **L43 CN**: 声明 struct `SourceStatementInfoTuplePOD`。
- **L44 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SourceStatementInfoTuplePOD(const parser::Label &parserLabel,`.
  **L44 CN**: 继续一个多行参数列表、初始化器或聚合项：`SourceStatementInfoTuplePOD(const parser::Label &parserLabel,`。
- **L45 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const ProxyForScope &proxyForScope,`.
  **L45 CN**: 继续一个多行参数列表、初始化器或聚合项：`const ProxyForScope &proxyForScope,`。
- **L46 EN**: Continues the surrounding expression or declaration: `const parser::CharBlock &parserCharBlock)`.
  **L46 CN**: 继续构造周围的表达式或声明：`const parser::CharBlock &parserCharBlock)`。
- **L47 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: parserLabel{parserLabel}, proxyForScope{proxyForScope},`.
  **L47 CN**: 继续一个多行参数列表、初始化器或聚合项：`: parserLabel{parserLabel}, proxyForScope{proxyForScope},`。
- **L48 EN**: Continues the surrounding expression or declaration: `parserCharBlock{parserCharBlock} {}`.
  **L48 CN**: 继续构造周围的表达式或声明：`parserCharBlock{parserCharBlock} {}`。

### Lines 49-72

````cpp
  parser::Label parserLabel;
  ProxyForScope proxyForScope;
  parser::CharBlock parserCharBlock;
};
using SourceStmtList = std::vector<SourceStatementInfoTuplePOD>;
enum class Legality { never, always, formerly };

bool HasScope(ProxyForScope scope) { return scope != ProxyForScope{0u}; }

// F18:R1131
template <typename A>
constexpr Legality IsLegalDoTerm(const parser::Statement<A> &) {
  if (std::is_same_v<A, common::Indirection<parser::EndDoStmt>> ||
      std::is_same_v<A, parser::EndDoStmt>) {
    return Legality::always;
  } else if (std::is_same_v<A, parser::EndForallStmt> ||
      std::is_same_v<A, parser::EndWhereStmt>) {
    // Executable construct end statements are also supported as
    // an extension but they need special care because the associated
    // construct create their own scope.
    return Legality::formerly;
  } else {
    return Legality::never;
  }
````
- **L49 EN**: Executes a standalone statement or declaration: `parser::Label parserLabel;`.
  **L49 CN**: 执行一条独立语句或声明：`parser::Label parserLabel;`。
- **L50 EN**: Executes a standalone statement or declaration: `ProxyForScope proxyForScope;`.
  **L50 CN**: 执行一条独立语句或声明：`ProxyForScope proxyForScope;`。
- **L51 EN**: Executes a standalone statement or declaration: `parser::CharBlock parserCharBlock;`.
  **L51 CN**: 执行一条独立语句或声明：`parser::CharBlock parserCharBlock;`。
- **L52 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L52 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L53 EN**: Defines alias `SourceStmtList` to simplify later code.
  **L53 CN**: 定义别名 `SourceStmtList` 以简化后续代码。
- **L54 EN**: Declares enum `class`.
  **L54 CN**: 声明 enum `class`。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L56 EN**: Continues logic associated with callable symbol `HasScope`.
  **L56 CN**: 继续与可调用符号 `HasScope` 相关的逻辑。
- **L57 EN**: Blank line separating nearby declarations or logic blocks.
  **L57 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L58 EN**: Comment explains nearby logic, intent, or metadata: `F18:R1131`.
  **L58 CN**: 注释说明附近代码的逻辑、意图或元数据：`F18:R1131`。
- **L59 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L59 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L60 EN**: Starts a function, method, lambda, or structured scope: `constexpr Legality IsLegalDoTerm(const parser::Statement<A> &) {`.
  **L60 CN**: 开始一个函数、方法、lambda 或结构化作用域：`constexpr Legality IsLegalDoTerm(const parser::Statement<A> &) {`。
- **L61 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L61 CN**: 开始 `if` 控制流语句并计算其条件。
- **L62 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndDoStmt>) {`.
  **L62 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndDoStmt>) {`。
- **L63 EN**: Returns from the current function with `Legality::always`.
  **L63 CN**: 以 `Legality::always` 从当前函数返回。
- **L64 EN**: Transitions from the previous branch into an `else if` condition.
  **L64 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L65 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndWhereStmt>) {`.
  **L65 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndWhereStmt>) {`。
- **L66 EN**: Comment explains nearby logic, intent, or metadata: `Executable construct end statements are also supported as`.
  **L66 CN**: 注释说明附近代码的逻辑、意图或元数据：`Executable construct end statements are also supported as`。
- **L67 EN**: Comment explains nearby logic, intent, or metadata: `an extension but they need special care because the associated`.
  **L67 CN**: 注释说明附近代码的逻辑、意图或元数据：`an extension but they need special care because the associated`。
- **L68 EN**: Comment explains nearby logic, intent, or metadata: `construct create their own scope.`.
  **L68 CN**: 注释说明附近代码的逻辑、意图或元数据：`construct create their own scope.`。
- **L69 EN**: Returns from the current function with `Legality::formerly`.
  **L69 CN**: 以 `Legality::formerly` 从当前函数返回。
- **L70 EN**: Transitions from the previous branch into the alternative path.
  **L70 CN**: 从前一个分支过渡到备选路径。
- **L71 EN**: Returns from the current function with `Legality::never`.
  **L71 CN**: 以 `Legality::never` 从当前函数返回。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。

### Lines 73-96

````cpp
}

constexpr Legality IsLegalDoTerm(
    const parser::Statement<parser::ActionStmt> &actionStmt) {
  if (std::holds_alternative<parser::ContinueStmt>(actionStmt.statement.u)) {
    // See F08:C816
    return Legality::always;
  } else if (!(std::holds_alternative<
                   common::Indirection<parser::ArithmeticIfStmt>>(
                   actionStmt.statement.u) ||
                 std::holds_alternative<common::Indirection<parser::CycleStmt>>(
                     actionStmt.statement.u) ||
                 std::holds_alternative<common::Indirection<parser::ExitStmt>>(
                     actionStmt.statement.u) ||
                 std::holds_alternative<common::Indirection<parser::StopStmt>>(
                     actionStmt.statement.u) ||
                 std::holds_alternative<common::Indirection<parser::GotoStmt>>(
                     actionStmt.statement.u) ||
                 std::holds_alternative<
                     common::Indirection<parser::ReturnStmt>>(
                     actionStmt.statement.u))) {
    return Legality::formerly;
  } else {
    return Legality::never;
````
- **L73 EN**: Closes the current lexical scope or compound statement.
  **L73 CN**: 结束当前词法作用域或复合语句块。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L75 EN**: Continues logic associated with callable symbol `IsLegalDoTerm`.
  **L75 CN**: 继续与可调用符号 `IsLegalDoTerm` 相关的逻辑。
- **L76 EN**: Continues the surrounding expression or declaration: `const parser::Statement<parser::ActionStmt> &actionStmt) {`.
  **L76 CN**: 继续构造周围的表达式或声明：`const parser::Statement<parser::ActionStmt> &actionStmt) {`。
- **L77 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L77 CN**: 开始 `if` 控制流语句并计算其条件。
- **L78 EN**: Comment explains nearby logic, intent, or metadata: `See F08:C816`.
  **L78 CN**: 注释说明附近代码的逻辑、意图或元数据：`See F08:C816`。
- **L79 EN**: Returns from the current function with `Legality::always`.
  **L79 CN**: 以 `Legality::always` 从当前函数返回。
- **L80 EN**: Transitions from the previous branch into an `else if` condition.
  **L80 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L81 EN**: Continues logic associated with callable symbol `ArithmeticIfStmt>>`.
  **L81 CN**: 继续与可调用符号 `ArithmeticIfStmt>>` 相关的逻辑。
- **L82 EN**: Continues the surrounding expression or declaration: `actionStmt.statement.u) ||`.
  **L82 CN**: 继续构造周围的表达式或声明：`actionStmt.statement.u) ||`。
- **L83 EN**: Continues logic associated with callable symbol `CycleStmt>>`.
  **L83 CN**: 继续与可调用符号 `CycleStmt>>` 相关的逻辑。
- **L84 EN**: Continues the surrounding expression or declaration: `actionStmt.statement.u) ||`.
  **L84 CN**: 继续构造周围的表达式或声明：`actionStmt.statement.u) ||`。
- **L85 EN**: Continues logic associated with callable symbol `ExitStmt>>`.
  **L85 CN**: 继续与可调用符号 `ExitStmt>>` 相关的逻辑。
- **L86 EN**: Continues the surrounding expression or declaration: `actionStmt.statement.u) ||`.
  **L86 CN**: 继续构造周围的表达式或声明：`actionStmt.statement.u) ||`。
- **L87 EN**: Continues logic associated with callable symbol `StopStmt>>`.
  **L87 CN**: 继续与可调用符号 `StopStmt>>` 相关的逻辑。
- **L88 EN**: Continues the surrounding expression or declaration: `actionStmt.statement.u) ||`.
  **L88 CN**: 继续构造周围的表达式或声明：`actionStmt.statement.u) ||`。
- **L89 EN**: Continues logic associated with callable symbol `GotoStmt>>`.
  **L89 CN**: 继续与可调用符号 `GotoStmt>>` 相关的逻辑。
- **L90 EN**: Continues the surrounding expression or declaration: `actionStmt.statement.u) ||`.
  **L90 CN**: 继续构造周围的表达式或声明：`actionStmt.statement.u) ||`。
- **L91 EN**: Continues the surrounding expression or declaration: `std::holds_alternative<`.
  **L91 CN**: 继续构造周围的表达式或声明：`std::holds_alternative<`。
- **L92 EN**: Continues logic associated with callable symbol `ReturnStmt>>`.
  **L92 CN**: 继续与可调用符号 `ReturnStmt>>` 相关的逻辑。
- **L93 EN**: Continues the surrounding expression or declaration: `actionStmt.statement.u))) {`.
  **L93 CN**: 继续构造周围的表达式或声明：`actionStmt.statement.u))) {`。
- **L94 EN**: Returns from the current function with `Legality::formerly`.
  **L94 CN**: 以 `Legality::formerly` 从当前函数返回。
- **L95 EN**: Transitions from the previous branch into the alternative path.
  **L95 CN**: 从前一个分支过渡到备选路径。
- **L96 EN**: Returns from the current function with `Legality::never`.
  **L96 CN**: 以 `Legality::never` 从当前函数返回。

### Lines 97-120

````cpp
  }
}

// Handles an assignment statement that might be wrapped by a construct
// other than an ActionStmt.
constexpr Legality IsLegalDoTerm(
    const parser::Statement<parser::AssignmentStmt> &) {
  return Legality::formerly;
}

template <typename A> constexpr bool IsFormat(const parser::Statement<A> &) {
  return std::is_same_v<A, common::Indirection<parser::FormatStmt>>;
}

template <typename A>
constexpr Legality IsLegalBranchTarget(const parser::Statement<A> &) {
  if (std::is_same_v<A, parser::ActionStmt> ||
      std::is_same_v<A, parser::AssociateStmt> ||
      std::is_same_v<A, parser::EndAssociateStmt> ||
      std::is_same_v<A, parser::IfThenStmt> ||
      std::is_same_v<A, parser::EndIfStmt> ||
      std::is_same_v<A, parser::SelectCaseStmt> ||
      std::is_same_v<A, parser::EndSelectStmt> ||
      std::is_same_v<A, parser::SelectRankStmt> ||
````
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。
- **L98 EN**: Closes the current lexical scope or compound statement.
  **L98 CN**: 结束当前词法作用域或复合语句块。
- **L99 EN**: Blank line separating nearby declarations or logic blocks.
  **L99 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L100 EN**: Comment explains nearby logic, intent, or metadata: `Handles an assignment statement that might be wrapped by a construct`.
  **L100 CN**: 注释说明附近代码的逻辑、意图或元数据：`Handles an assignment statement that might be wrapped by a construct`。
- **L101 EN**: Comment explains nearby logic, intent, or metadata: `other than an ActionStmt.`.
  **L101 CN**: 注释说明附近代码的逻辑、意图或元数据：`other than an ActionStmt.`。
- **L102 EN**: Continues logic associated with callable symbol `IsLegalDoTerm`.
  **L102 CN**: 继续与可调用符号 `IsLegalDoTerm` 相关的逻辑。
- **L103 EN**: Continues the surrounding expression or declaration: `const parser::Statement<parser::AssignmentStmt> &) {`.
  **L103 CN**: 继续构造周围的表达式或声明：`const parser::Statement<parser::AssignmentStmt> &) {`。
- **L104 EN**: Returns from the current function with `Legality::formerly`.
  **L104 CN**: 以 `Legality::formerly` 从当前函数返回。
- **L105 EN**: Closes the current lexical scope or compound statement.
  **L105 CN**: 结束当前词法作用域或复合语句块。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L107 EN**: Introduces template parameters or specialization context: `template <typename A> constexpr bool IsFormat(const parser::Statement<A> &) {`.
  **L107 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A> constexpr bool IsFormat(const parser::Statement<A> &) {`。
- **L108 EN**: Returns from the current function with `std::is_same_v<A, common::Indirection<parser::FormatStmt>>`.
  **L108 CN**: 以 `std::is_same_v<A, common::Indirection<parser::FormatStmt>>` 从当前函数返回。
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L111 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L111 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L112 EN**: Starts a function, method, lambda, or structured scope: `constexpr Legality IsLegalBranchTarget(const parser::Statement<A> &) {`.
  **L112 CN**: 开始一个函数、方法、lambda 或结构化作用域：`constexpr Legality IsLegalBranchTarget(const parser::Statement<A> &) {`。
- **L113 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L113 CN**: 开始 `if` 控制流语句并计算其条件。
- **L114 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::AssociateStmt> ||`.
  **L114 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::AssociateStmt> ||`。
- **L115 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndAssociateStmt> ||`.
  **L115 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndAssociateStmt> ||`。
- **L116 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::IfThenStmt> ||`.
  **L116 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::IfThenStmt> ||`。
- **L117 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndIfStmt> ||`.
  **L117 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndIfStmt> ||`。
- **L118 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::SelectCaseStmt> ||`.
  **L118 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::SelectCaseStmt> ||`。
- **L119 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndSelectStmt> ||`.
  **L119 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndSelectStmt> ||`。
- **L120 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::SelectRankStmt> ||`.
  **L120 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::SelectRankStmt> ||`。

### Lines 121-144

````cpp
      std::is_same_v<A, parser::SelectTypeStmt> ||
      std::is_same_v<A, common::Indirection<parser::LabelDoStmt>> ||
      std::is_same_v<A, parser::NonLabelDoStmt> ||
      std::is_same_v<A, parser::EndDoStmt> ||
      std::is_same_v<A, common::Indirection<parser::EndDoStmt>> ||
      std::is_same_v<A, parser::BlockStmt> ||
      std::is_same_v<A, parser::EndBlockStmt> ||
      std::is_same_v<A, parser::CriticalStmt> ||
      std::is_same_v<A, parser::EndCriticalStmt> ||
      std::is_same_v<A, parser::ForallConstructStmt> ||
      std::is_same_v<A, parser::WhereConstructStmt> ||
      std::is_same_v<A, parser::ChangeTeamStmt> ||
      std::is_same_v<A, parser::EndChangeTeamStmt> ||
      std::is_same_v<A, parser::EndFunctionStmt> ||
      std::is_same_v<A, parser::EndMpSubprogramStmt> ||
      std::is_same_v<A, parser::EndProgramStmt> ||
      std::is_same_v<A, parser::EndSubroutineStmt>) {
    return Legality::always;
  } else {
    return Legality::never;
  }
}

template <typename A>
````
- **L121 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::SelectTypeStmt> ||`.
  **L121 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::SelectTypeStmt> ||`。
- **L122 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, common::Indirection<parser::LabelDoStmt>> ||`.
  **L122 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, common::Indirection<parser::LabelDoStmt>> ||`。
- **L123 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::NonLabelDoStmt> ||`.
  **L123 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::NonLabelDoStmt> ||`。
- **L124 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndDoStmt> ||`.
  **L124 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndDoStmt> ||`。
- **L125 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, common::Indirection<parser::EndDoStmt>> ||`.
  **L125 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, common::Indirection<parser::EndDoStmt>> ||`。
- **L126 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::BlockStmt> ||`.
  **L126 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::BlockStmt> ||`。
- **L127 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndBlockStmt> ||`.
  **L127 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndBlockStmt> ||`。
- **L128 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::CriticalStmt> ||`.
  **L128 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::CriticalStmt> ||`。
- **L129 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndCriticalStmt> ||`.
  **L129 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndCriticalStmt> ||`。
- **L130 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::ForallConstructStmt> ||`.
  **L130 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::ForallConstructStmt> ||`。
- **L131 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::WhereConstructStmt> ||`.
  **L131 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::WhereConstructStmt> ||`。
- **L132 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::ChangeTeamStmt> ||`.
  **L132 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::ChangeTeamStmt> ||`。
- **L133 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndChangeTeamStmt> ||`.
  **L133 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndChangeTeamStmt> ||`。
- **L134 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndFunctionStmt> ||`.
  **L134 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndFunctionStmt> ||`。
- **L135 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndMpSubprogramStmt> ||`.
  **L135 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndMpSubprogramStmt> ||`。
- **L136 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndProgramStmt> ||`.
  **L136 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndProgramStmt> ||`。
- **L137 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndSubroutineStmt>) {`.
  **L137 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndSubroutineStmt>) {`。
- **L138 EN**: Returns from the current function with `Legality::always`.
  **L138 CN**: 以 `Legality::always` 从当前函数返回。
- **L139 EN**: Transitions from the previous branch into the alternative path.
  **L139 CN**: 从前一个分支过渡到备选路径。
- **L140 EN**: Returns from the current function with `Legality::never`.
  **L140 CN**: 以 `Legality::never` 从当前函数返回。
- **L141 EN**: Closes the current lexical scope or compound statement.
  **L141 CN**: 结束当前词法作用域或复合语句块。
- **L142 EN**: Closes the current lexical scope or compound statement.
  **L142 CN**: 结束当前词法作用域或复合语句块。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L144 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L144 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。

### Lines 145-168

````cpp
constexpr LabeledStmtClassificationSet ConstructBranchTargetFlags(
    const parser::Statement<A> &statement) {
  LabeledStmtClassificationSet labeledStmtClassificationSet{};
  if (IsLegalDoTerm(statement) == Legality::always) {
    labeledStmtClassificationSet.set(TargetStatementEnum::Do);
  } else if (IsLegalDoTerm(statement) == Legality::formerly) {
    labeledStmtClassificationSet.set(TargetStatementEnum::CompatibleDo);
  }
  if (IsLegalBranchTarget(statement) == Legality::always) {
    labeledStmtClassificationSet.set(TargetStatementEnum::Branch);
  } else if (IsLegalBranchTarget(statement) == Legality::formerly) {
    labeledStmtClassificationSet.set(TargetStatementEnum::CompatibleBranch);
  }
  if (IsFormat(statement)) {
    labeledStmtClassificationSet.set(TargetStatementEnum::Format);
  }
  return labeledStmtClassificationSet;
}

static unsigned SayLabel(parser::Label label) {
  return static_cast<unsigned>(label);
}

struct UnitAnalysis {
````
- **L145 EN**: Continues logic associated with callable symbol `ConstructBranchTargetFlags`.
  **L145 CN**: 继续与可调用符号 `ConstructBranchTargetFlags` 相关的逻辑。
- **L146 EN**: Continues the surrounding expression or declaration: `const parser::Statement<A> &statement) {`.
  **L146 CN**: 继续构造周围的表达式或声明：`const parser::Statement<A> &statement) {`。
- **L147 EN**: Executes a standalone statement or declaration: `LabeledStmtClassificationSet labeledStmtClassificationSet{};`.
  **L147 CN**: 执行一条独立语句或声明：`LabeledStmtClassificationSet labeledStmtClassificationSet{};`。
- **L148 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L148 CN**: 开始 `if` 控制流语句并计算其条件。
- **L149 EN**: Executes a call or declaration centered on `labeledStmtClassificationSet.set`.
  **L149 CN**: 执行以 `labeledStmtClassificationSet.set` 为核心的调用或声明。
- **L150 EN**: Transitions from the previous branch into an `else if` condition.
  **L150 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L151 EN**: Executes a call or declaration centered on `labeledStmtClassificationSet.set`.
  **L151 CN**: 执行以 `labeledStmtClassificationSet.set` 为核心的调用或声明。
- **L152 EN**: Closes the current lexical scope or compound statement.
  **L152 CN**: 结束当前词法作用域或复合语句块。
- **L153 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L153 CN**: 开始 `if` 控制流语句并计算其条件。
- **L154 EN**: Executes a call or declaration centered on `labeledStmtClassificationSet.set`.
  **L154 CN**: 执行以 `labeledStmtClassificationSet.set` 为核心的调用或声明。
- **L155 EN**: Transitions from the previous branch into an `else if` condition.
  **L155 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L156 EN**: Executes a call or declaration centered on `labeledStmtClassificationSet.set`.
  **L156 CN**: 执行以 `labeledStmtClassificationSet.set` 为核心的调用或声明。
- **L157 EN**: Closes the current lexical scope or compound statement.
  **L157 CN**: 结束当前词法作用域或复合语句块。
- **L158 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L158 CN**: 开始 `if` 控制流语句并计算其条件。
- **L159 EN**: Executes a call or declaration centered on `labeledStmtClassificationSet.set`.
  **L159 CN**: 执行以 `labeledStmtClassificationSet.set` 为核心的调用或声明。
- **L160 EN**: Closes the current lexical scope or compound statement.
  **L160 CN**: 结束当前词法作用域或复合语句块。
- **L161 EN**: Returns from the current function with `labeledStmtClassificationSet`.
  **L161 CN**: 以 `labeledStmtClassificationSet` 从当前函数返回。
- **L162 EN**: Closes the current lexical scope or compound statement.
  **L162 CN**: 结束当前词法作用域或复合语句块。
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L164 EN**: Starts a function, method, lambda, or structured scope: `static unsigned SayLabel(parser::Label label) {`.
  **L164 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static unsigned SayLabel(parser::Label label) {`。
- **L165 EN**: Returns from the current function with `static_cast<unsigned>(label)`.
  **L165 CN**: 以 `static_cast<unsigned>(label)` 从当前函数返回。
- **L166 EN**: Closes the current lexical scope or compound statement.
  **L166 CN**: 结束当前词法作用域或复合语句块。
- **L167 EN**: Blank line separating nearby declarations or logic blocks.
  **L167 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L168 EN**: Declares struct `UnitAnalysis`.
  **L168 CN**: 声明 struct `UnitAnalysis`。

### Lines 169-192

````cpp
  UnitAnalysis() { scopeModel.emplace_back(); }

  SourceStmtList doStmtSources;
  SourceStmtList formatStmtSources;
  SourceStmtList otherStmtSources;
  SourceStmtList assignStmtSources;
  TargetStmtMap targetStmts;
  std::vector<ScopeInfo> scopeModel;
};

// Some parse tree record for statements simply wrap construct names;
// others include them as tuple components.  Given a statement,
// return a pointer to its name if it has one.
template <typename A>
const parser::CharBlock *GetStmtName(const parser::Statement<A> &stmt) {
  const std::optional<parser::Name> *name{nullptr};
  if constexpr (WrapperTrait<A>) {
    if constexpr (std::is_same_v<decltype(A::v), parser::Name>) {
      return &stmt.statement.v.source;
    } else {
      name = &stmt.statement.v;
    }
  } else if constexpr (std::is_same_v<A, parser::SelectRankStmt> ||
      std::is_same_v<A, parser::SelectTypeStmt>) {
````
- **L169 EN**: Continues logic associated with callable symbol `UnitAnalysis`.
  **L169 CN**: 继续与可调用符号 `UnitAnalysis` 相关的逻辑。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L171 EN**: Executes a standalone statement or declaration: `SourceStmtList doStmtSources;`.
  **L171 CN**: 执行一条独立语句或声明：`SourceStmtList doStmtSources;`。
- **L172 EN**: Executes a standalone statement or declaration: `SourceStmtList formatStmtSources;`.
  **L172 CN**: 执行一条独立语句或声明：`SourceStmtList formatStmtSources;`。
- **L173 EN**: Executes a standalone statement or declaration: `SourceStmtList otherStmtSources;`.
  **L173 CN**: 执行一条独立语句或声明：`SourceStmtList otherStmtSources;`。
- **L174 EN**: Executes a standalone statement or declaration: `SourceStmtList assignStmtSources;`.
  **L174 CN**: 执行一条独立语句或声明：`SourceStmtList assignStmtSources;`。
- **L175 EN**: Executes a standalone statement or declaration: `TargetStmtMap targetStmts;`.
  **L175 CN**: 执行一条独立语句或声明：`TargetStmtMap targetStmts;`。
- **L176 EN**: Executes a standalone statement or declaration: `std::vector<ScopeInfo> scopeModel;`.
  **L176 CN**: 执行一条独立语句或声明：`std::vector<ScopeInfo> scopeModel;`。
- **L177 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L177 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L179 EN**: Comment explains nearby logic, intent, or metadata: `Some parse tree record for statements simply wrap construct names;`.
  **L179 CN**: 注释说明附近代码的逻辑、意图或元数据：`Some parse tree record for statements simply wrap construct names;`。
- **L180 EN**: Comment explains nearby logic, intent, or metadata: `others include them as tuple components.  Given a statement,`.
  **L180 CN**: 注释说明附近代码的逻辑、意图或元数据：`others include them as tuple components.  Given a statement,`。
- **L181 EN**: Comment explains nearby logic, intent, or metadata: `return a pointer to its name if it has one.`.
  **L181 CN**: 注释说明附近代码的逻辑、意图或元数据：`return a pointer to its name if it has one.`。
- **L182 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L182 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L183 EN**: Starts a function, method, lambda, or structured scope: `const parser::CharBlock *GetStmtName(const parser::Statement<A> &stmt) {`.
  **L183 CN**: 开始一个函数、方法、lambda 或结构化作用域：`const parser::CharBlock *GetStmtName(const parser::Statement<A> &stmt) {`。
- **L184 EN**: Executes a standalone statement or declaration: `const std::optional<parser::Name> *name{nullptr};`.
  **L184 CN**: 执行一条独立语句或声明：`const std::optional<parser::Name> *name{nullptr};`。
- **L185 EN**: Continues logic associated with callable symbol `constexpr`.
  **L185 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L186 EN**: Continues logic associated with callable symbol `constexpr`.
  **L186 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L187 EN**: Returns from the current function with `&stmt.statement.v.source`.
  **L187 CN**: 以 `&stmt.statement.v.source` 从当前函数返回。
- **L188 EN**: Transitions from the previous branch into the alternative path.
  **L188 CN**: 从前一个分支过渡到备选路径。
- **L189 EN**: Executes a standalone statement or declaration: `name = &stmt.statement.v;`.
  **L189 CN**: 执行一条独立语句或声明：`name = &stmt.statement.v;`。
- **L190 EN**: Closes the current lexical scope or compound statement.
  **L190 CN**: 结束当前词法作用域或复合语句块。
- **L191 EN**: Transitions from the previous branch into an `else if` condition.
  **L191 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L192 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::SelectTypeStmt>) {`.
  **L192 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::SelectTypeStmt>) {`。

### Lines 193-216

````cpp
    name = &std::get<0>(stmt.statement.t);
  } else if constexpr (common::HasMember<parser::Name,
                           decltype(stmt.statement.t)>) {
    return &std::get<parser::Name>(stmt.statement.t).source;
  } else {
    name = &std::get<std::optional<parser::Name>>(stmt.statement.t);
  }
  if (name && *name) {
    return &(*name)->source;
  }
  return nullptr;
}

class ParseTreeAnalyzer {
public:
  ParseTreeAnalyzer(ParseTreeAnalyzer &&that) = default;
  ParseTreeAnalyzer(SemanticsContext &context) : context_{context} {}

  template <typename A> constexpr bool Pre(const A &x) {
    using LabeledProgramUnitStmts =
        std::tuple<parser::MainProgram, parser::FunctionSubprogram,
            parser::SubroutineSubprogram, parser::SeparateModuleSubprogram>;
    if constexpr (common::HasMember<A, LabeledProgramUnitStmts>) {
      const auto &endStmt{std::get<std::tuple_size_v<decltype(x.t)> - 1>(x.t)};
````
- **L193 EN**: Executes a call or declaration centered on `&std::get<0>`.
  **L193 CN**: 执行以 `&std::get<0>` 为核心的调用或声明。
- **L194 EN**: Transitions from the previous branch into an `else if` condition.
  **L194 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L195 EN**: Starts a function, method, lambda, or structured scope: `decltype(stmt.statement.t)>) {`.
  **L195 CN**: 开始一个函数、方法、lambda 或结构化作用域：`decltype(stmt.statement.t)>) {`。
- **L196 EN**: Returns from the current function with `&std::get<parser::Name>(stmt.statement.t).source`.
  **L196 CN**: 以 `&std::get<parser::Name>(stmt.statement.t).source` 从当前函数返回。
- **L197 EN**: Transitions from the previous branch into the alternative path.
  **L197 CN**: 从前一个分支过渡到备选路径。
- **L198 EN**: Executes a call or declaration centered on `&std::get<std::optional<parser::Name>>`.
  **L198 CN**: 执行以 `&std::get<std::optional<parser::Name>>` 为核心的调用或声明。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L200 CN**: 开始 `if` 控制流语句并计算其条件。
- **L201 EN**: Returns from the current function with `&(*name)->source`.
  **L201 CN**: 以 `&(*name)->source` 从当前函数返回。
- **L202 EN**: Closes the current lexical scope or compound statement.
  **L202 CN**: 结束当前词法作用域或复合语句块。
- **L203 EN**: Returns from the current function with `nullptr`.
  **L203 CN**: 以 `nullptr` 从当前函数返回。
- **L204 EN**: Closes the current lexical scope or compound statement.
  **L204 CN**: 结束当前词法作用域或复合语句块。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L206 EN**: Declares class `ParseTreeAnalyzer`.
  **L206 CN**: 声明 class `ParseTreeAnalyzer`。
- **L207 EN**: Sets the following members to `public` access.
  **L207 CN**: 将后续成员的访问级别设为 `public`。
- **L208 EN**: Executes a call or declaration centered on `ParseTreeAnalyzer`.
  **L208 CN**: 执行以 `ParseTreeAnalyzer` 为核心的调用或声明。
- **L209 EN**: Continues logic associated with callable symbol `ParseTreeAnalyzer`.
  **L209 CN**: 继续与可调用符号 `ParseTreeAnalyzer` 相关的逻辑。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L211 EN**: Introduces template parameters or specialization context: `template <typename A> constexpr bool Pre(const A &x) {`.
  **L211 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A> constexpr bool Pre(const A &x) {`。
- **L212 EN**: Defines alias `LabeledProgramUnitStmts` to simplify later code.
  **L212 CN**: 定义别名 `LabeledProgramUnitStmts` 以简化后续代码。
- **L213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::tuple<parser::MainProgram, parser::FunctionSubprogram,`.
  **L213 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::tuple<parser::MainProgram, parser::FunctionSubprogram,`。
- **L214 EN**: Executes a standalone statement or declaration: `parser::SubroutineSubprogram, parser::SeparateModuleSubprogram>;`.
  **L214 CN**: 执行一条独立语句或声明：`parser::SubroutineSubprogram, parser::SeparateModuleSubprogram>;`。
- **L215 EN**: Continues logic associated with callable symbol `constexpr`.
  **L215 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L216 EN**: Executes a call or declaration centered on `&endStmt{std::get<std::tuple_size_v<decltype`.
  **L216 CN**: 执行以 `&endStmt{std::get<std::tuple_size_v<decltype` 为核心的调用或声明。

### Lines 217-240

````cpp
      if (endStmt.label) {
        // The END statement for a subprogram appears after any internal
        // subprograms.  Visit that statement in advance so that results
        // are placed in the correct programUnits_ slot.
        auto targetFlags{ConstructBranchTargetFlags(endStmt)};
        AddTargetLabelDefinition(endStmt.label.value(), targetFlags,
            currentScope_,
            /*isExecutableConstructEndStmt=*/false);
      }
    }
    return true;
  }
  template <typename A> constexpr void Post(const A &) {}

  template <typename A> bool Pre(const parser::Statement<A> &statement) {
    currentPosition_ = statement.source;
    const auto &label = statement.label;
    if (!label) {
      return true;
    }
    using LabeledConstructStmts = std::tuple<parser::AssociateStmt,
        parser::BlockStmt, parser::ChangeTeamStmt, parser::CriticalStmt,
        parser::IfThenStmt, parser::NonLabelDoStmt, parser::SelectCaseStmt,
        parser::SelectRankStmt, parser::SelectTypeStmt,
````
- **L217 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L217 CN**: 开始 `if` 控制流语句并计算其条件。
- **L218 EN**: Comment explains nearby logic, intent, or metadata: `The END statement for a subprogram appears after any internal`.
  **L218 CN**: 注释说明附近代码的逻辑、意图或元数据：`The END statement for a subprogram appears after any internal`。
- **L219 EN**: Comment explains nearby logic, intent, or metadata: `subprograms.  Visit that statement in advance so that results`.
  **L219 CN**: 注释说明附近代码的逻辑、意图或元数据：`subprograms.  Visit that statement in advance so that results`。
- **L220 EN**: Comment explains nearby logic, intent, or metadata: `are placed in the correct programUnits_ slot.`.
  **L220 CN**: 注释说明附近代码的逻辑、意图或元数据：`are placed in the correct programUnits_ slot.`。
- **L221 EN**: Executes a call or declaration centered on `targetFlags{ConstructBranchTargetFlags`.
  **L221 CN**: 执行以 `targetFlags{ConstructBranchTargetFlags` 为核心的调用或声明。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AddTargetLabelDefinition(endStmt.label.value(), targetFlags,`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`AddTargetLabelDefinition(endStmt.label.value(), targetFlags,`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `currentScope_,`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`currentScope_,`。
- **L224 EN**: Comment explains nearby logic, intent, or metadata: `isExecutableConstructEndStmt=*/false);`.
  **L224 CN**: 注释说明附近代码的逻辑、意图或元数据：`isExecutableConstructEndStmt=*/false);`。
- **L225 EN**: Closes the current lexical scope or compound statement.
  **L225 CN**: 结束当前词法作用域或复合语句块。
- **L226 EN**: Closes the current lexical scope or compound statement.
  **L226 CN**: 结束当前词法作用域或复合语句块。
- **L227 EN**: Returns from the current function with `true`.
  **L227 CN**: 以 `true` 从当前函数返回。
- **L228 EN**: Closes the current lexical scope or compound statement.
  **L228 CN**: 结束当前词法作用域或复合语句块。
- **L229 EN**: Introduces template parameters or specialization context: `template <typename A> constexpr void Post(const A &) {}`.
  **L229 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A> constexpr void Post(const A &) {}`。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L231 EN**: Introduces template parameters or specialization context: `template <typename A> bool Pre(const parser::Statement<A> &statement) {`.
  **L231 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A> bool Pre(const parser::Statement<A> &statement) {`。
- **L232 EN**: Executes a standalone statement or declaration: `currentPosition_ = statement.source;`.
  **L232 CN**: 执行一条独立语句或声明：`currentPosition_ = statement.source;`。
- **L233 EN**: Executes a standalone statement or declaration: `const auto &label = statement.label;`.
  **L233 CN**: 执行一条独立语句或声明：`const auto &label = statement.label;`。
- **L234 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L234 CN**: 开始 `if` 控制流语句并计算其条件。
- **L235 EN**: Returns from the current function with `true`.
  **L235 CN**: 以 `true` 从当前函数返回。
- **L236 EN**: Closes the current lexical scope or compound statement.
  **L236 CN**: 结束当前词法作用域或复合语句块。
- **L237 EN**: Defines alias `LabeledConstructStmts` to simplify later code.
  **L237 CN**: 定义别名 `LabeledConstructStmts` 以简化后续代码。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::BlockStmt, parser::ChangeTeamStmt, parser::CriticalStmt,`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::BlockStmt, parser::ChangeTeamStmt, parser::CriticalStmt,`。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::IfThenStmt, parser::NonLabelDoStmt, parser::SelectCaseStmt,`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::IfThenStmt, parser::NonLabelDoStmt, parser::SelectCaseStmt,`。
- **L240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::SelectRankStmt, parser::SelectTypeStmt,`.
  **L240 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::SelectRankStmt, parser::SelectTypeStmt,`。

### Lines 241-264

````cpp
        parser::ForallConstructStmt, parser::WhereConstructStmt>;
    using LabeledConstructEndStmts = std::tuple<parser::EndAssociateStmt,
        parser::EndBlockStmt, parser::EndChangeTeamStmt,
        parser::EndCriticalStmt, parser::EndDoStmt, parser::EndForallStmt,
        parser::EndIfStmt, parser::EndWhereStmt>;
    using LabeledProgramUnitEndStmts =
        std::tuple<parser::EndFunctionStmt, parser::EndMpSubprogramStmt,
            parser::EndProgramStmt, parser::EndSubroutineStmt>;
    auto targetFlags{ConstructBranchTargetFlags(statement)};
    if constexpr (common::HasMember<A, LabeledConstructStmts>) {
      AddTargetLabelDefinition(label.value(), targetFlags, ParentScope(),
          /*isExecutableConstructEndStmt=*/false);
    } else if constexpr (std::is_same_v<A, parser::EndIfStmt> ||
        std::is_same_v<A, parser::EndSelectStmt>) {
      // the label on an END IF/SELECT is not in the last part/case
      AddTargetLabelDefinition(label.value(), targetFlags, ParentScope(),
          /*isExecutableConstructEndStmt=*/true);
    } else if constexpr (common::HasMember<A, LabeledConstructEndStmts>) {
      AddTargetLabelDefinition(label.value(), targetFlags, currentScope_,
          /*isExecutableConstructEndStmt=*/true);
    } else if constexpr (!common::HasMember<A, LabeledProgramUnitEndStmts>) {
      // Program unit END statements have already been processed.
      AddTargetLabelDefinition(label.value(), targetFlags, currentScope_,
          /*isExecutableConstructEndStmt=*/false);
````
- **L241 EN**: Executes a standalone statement or declaration: `parser::ForallConstructStmt, parser::WhereConstructStmt>;`.
  **L241 CN**: 执行一条独立语句或声明：`parser::ForallConstructStmt, parser::WhereConstructStmt>;`。
- **L242 EN**: Defines alias `LabeledConstructEndStmts` to simplify later code.
  **L242 CN**: 定义别名 `LabeledConstructEndStmts` 以简化后续代码。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::EndBlockStmt, parser::EndChangeTeamStmt,`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::EndBlockStmt, parser::EndChangeTeamStmt,`。
- **L244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::EndCriticalStmt, parser::EndDoStmt, parser::EndForallStmt,`.
  **L244 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::EndCriticalStmt, parser::EndDoStmt, parser::EndForallStmt,`。
- **L245 EN**: Executes a standalone statement or declaration: `parser::EndIfStmt, parser::EndWhereStmt>;`.
  **L245 CN**: 执行一条独立语句或声明：`parser::EndIfStmt, parser::EndWhereStmt>;`。
- **L246 EN**: Defines alias `LabeledProgramUnitEndStmts` to simplify later code.
  **L246 CN**: 定义别名 `LabeledProgramUnitEndStmts` 以简化后续代码。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::tuple<parser::EndFunctionStmt, parser::EndMpSubprogramStmt,`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::tuple<parser::EndFunctionStmt, parser::EndMpSubprogramStmt,`。
- **L248 EN**: Executes a standalone statement or declaration: `parser::EndProgramStmt, parser::EndSubroutineStmt>;`.
  **L248 CN**: 执行一条独立语句或声明：`parser::EndProgramStmt, parser::EndSubroutineStmt>;`。
- **L249 EN**: Executes a call or declaration centered on `targetFlags{ConstructBranchTargetFlags`.
  **L249 CN**: 执行以 `targetFlags{ConstructBranchTargetFlags` 为核心的调用或声明。
- **L250 EN**: Continues logic associated with callable symbol `constexpr`.
  **L250 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AddTargetLabelDefinition(label.value(), targetFlags, ParentScope(),`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`AddTargetLabelDefinition(label.value(), targetFlags, ParentScope(),`。
- **L252 EN**: Comment explains nearby logic, intent, or metadata: `isExecutableConstructEndStmt=*/false);`.
  **L252 CN**: 注释说明附近代码的逻辑、意图或元数据：`isExecutableConstructEndStmt=*/false);`。
- **L253 EN**: Transitions from the previous branch into an `else if` condition.
  **L253 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L254 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::EndSelectStmt>) {`.
  **L254 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::EndSelectStmt>) {`。
- **L255 EN**: Comment explains nearby logic, intent, or metadata: `the label on an END IF/SELECT is not in the last part/case`.
  **L255 CN**: 注释说明附近代码的逻辑、意图或元数据：`the label on an END IF/SELECT is not in the last part/case`。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AddTargetLabelDefinition(label.value(), targetFlags, ParentScope(),`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`AddTargetLabelDefinition(label.value(), targetFlags, ParentScope(),`。
- **L257 EN**: Comment explains nearby logic, intent, or metadata: `isExecutableConstructEndStmt=*/true);`.
  **L257 CN**: 注释说明附近代码的逻辑、意图或元数据：`isExecutableConstructEndStmt=*/true);`。
- **L258 EN**: Transitions from the previous branch into an `else if` condition.
  **L258 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AddTargetLabelDefinition(label.value(), targetFlags, currentScope_,`.
  **L259 CN**: 继续一个多行参数列表、初始化器或聚合项：`AddTargetLabelDefinition(label.value(), targetFlags, currentScope_,`。
- **L260 EN**: Comment explains nearby logic, intent, or metadata: `isExecutableConstructEndStmt=*/true);`.
  **L260 CN**: 注释说明附近代码的逻辑、意图或元数据：`isExecutableConstructEndStmt=*/true);`。
- **L261 EN**: Transitions from the previous branch into an `else if` condition.
  **L261 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L262 EN**: Comment explains nearby logic, intent, or metadata: `Program unit END statements have already been processed.`.
  **L262 CN**: 注释说明附近代码的逻辑、意图或元数据：`Program unit END statements have already been processed.`。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AddTargetLabelDefinition(label.value(), targetFlags, currentScope_,`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`AddTargetLabelDefinition(label.value(), targetFlags, currentScope_,`。
- **L264 EN**: Comment explains nearby logic, intent, or metadata: `isExecutableConstructEndStmt=*/false);`.
  **L264 CN**: 注释说明附近代码的逻辑、意图或元数据：`isExecutableConstructEndStmt=*/false);`。

### Lines 265-288

````cpp
    }
    return true;
  }

  // see 11.1.1
  bool Pre(const parser::ProgramUnit &) { return InitializeNewScopeContext(); }
  bool Pre(const parser::InternalSubprogram &) {
    return InitializeNewScopeContext();
  }
  bool Pre(const parser::ModuleSubprogram &) {
    return InitializeNewScopeContext();
  }
  bool Pre(const parser::AssociateConstruct &associateConstruct) {
    return PushConstructName(associateConstruct);
  }
  bool Pre(const parser::BlockConstruct &blockConstruct) {
    return PushConstructName(blockConstruct);
  }
  bool Pre(const parser::ChangeTeamConstruct &changeTeamConstruct) {
    return PushConstructName(changeTeamConstruct);
  }
  bool Pre(const parser::CriticalConstruct &criticalConstruct) {
    return PushConstructName(criticalConstruct);
  }
````
- **L265 EN**: Closes the current lexical scope or compound statement.
  **L265 CN**: 结束当前词法作用域或复合语句块。
- **L266 EN**: Returns from the current function with `true`.
  **L266 CN**: 以 `true` 从当前函数返回。
- **L267 EN**: Closes the current lexical scope or compound statement.
  **L267 CN**: 结束当前词法作用域或复合语句块。
- **L268 EN**: Blank line separating nearby declarations or logic blocks.
  **L268 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L269 EN**: Comment explains nearby logic, intent, or metadata: `see 11.1.1`.
  **L269 CN**: 注释说明附近代码的逻辑、意图或元数据：`see 11.1.1`。
- **L270 EN**: Continues logic associated with callable symbol `Pre`.
  **L270 CN**: 继续与可调用符号 `Pre` 相关的逻辑。
- **L271 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::InternalSubprogram &) {`.
  **L271 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::InternalSubprogram &) {`。
- **L272 EN**: Returns from the current function with `InitializeNewScopeContext()`.
  **L272 CN**: 以 `InitializeNewScopeContext()` 从当前函数返回。
- **L273 EN**: Closes the current lexical scope or compound statement.
  **L273 CN**: 结束当前词法作用域或复合语句块。
- **L274 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::ModuleSubprogram &) {`.
  **L274 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::ModuleSubprogram &) {`。
- **L275 EN**: Returns from the current function with `InitializeNewScopeContext()`.
  **L275 CN**: 以 `InitializeNewScopeContext()` 从当前函数返回。
- **L276 EN**: Closes the current lexical scope or compound statement.
  **L276 CN**: 结束当前词法作用域或复合语句块。
- **L277 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::AssociateConstruct &associateConstruct) {`.
  **L277 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::AssociateConstruct &associateConstruct) {`。
- **L278 EN**: Returns from the current function with `PushConstructName(associateConstruct)`.
  **L278 CN**: 以 `PushConstructName(associateConstruct)` 从当前函数返回。
- **L279 EN**: Closes the current lexical scope or compound statement.
  **L279 CN**: 结束当前词法作用域或复合语句块。
- **L280 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::BlockConstruct &blockConstruct) {`.
  **L280 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::BlockConstruct &blockConstruct) {`。
- **L281 EN**: Returns from the current function with `PushConstructName(blockConstruct)`.
  **L281 CN**: 以 `PushConstructName(blockConstruct)` 从当前函数返回。
- **L282 EN**: Closes the current lexical scope or compound statement.
  **L282 CN**: 结束当前词法作用域或复合语句块。
- **L283 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::ChangeTeamConstruct &changeTeamConstruct) {`.
  **L283 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::ChangeTeamConstruct &changeTeamConstruct) {`。
- **L284 EN**: Returns from the current function with `PushConstructName(changeTeamConstruct)`.
  **L284 CN**: 以 `PushConstructName(changeTeamConstruct)` 从当前函数返回。
- **L285 EN**: Closes the current lexical scope or compound statement.
  **L285 CN**: 结束当前词法作用域或复合语句块。
- **L286 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::CriticalConstruct &criticalConstruct) {`.
  **L286 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::CriticalConstruct &criticalConstruct) {`。
- **L287 EN**: Returns from the current function with `PushConstructName(criticalConstruct)`.
  **L287 CN**: 以 `PushConstructName(criticalConstruct)` 从当前函数返回。
- **L288 EN**: Closes the current lexical scope or compound statement.
  **L288 CN**: 结束当前词法作用域或复合语句块。

### Lines 289-312

````cpp
  bool Pre(const parser::DoConstruct &doConstruct) {
    const auto &optionalName{std::get<std::optional<parser::Name>>(
        std::get<parser::Statement<parser::NonLabelDoStmt>>(doConstruct.t)
            .statement.t)};
    if (optionalName) {
      constructNames_.emplace_back(optionalName->ToString());
    }
    // Allow FORTRAN '66 extended DO ranges
    PushScope(false);
    // Process labels of the DO and END DO statements, but not the
    // statements themselves, so that a non-construct END DO
    // can be distinguished (below).
    Pre(std::get<parser::Statement<parser::NonLabelDoStmt>>(doConstruct.t));
    Walk(std::get<parser::Block>(doConstruct.t), *this);
    Pre(std::get<parser::Statement<parser::EndDoStmt>>(doConstruct.t));
    PopConstructName(doConstruct);
    return false;
  }
  void Post(const parser::EndDoStmt &endDoStmt) {
    // Visited only for non-construct labeled DO termination
    if (const auto &name{endDoStmt.v}) {
      context_.Say(name->source, "Unexpected DO construct name '%s'"_err_en_US,
          name->source);
    }
````
- **L289 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::DoConstruct &doConstruct) {`.
  **L289 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::DoConstruct &doConstruct) {`。
- **L290 EN**: Continues logic associated with callable symbol `Name>>`.
  **L290 CN**: 继续与可调用符号 `Name>>` 相关的逻辑。
- **L291 EN**: Continues logic associated with callable symbol `NonLabelDoStmt>>`.
  **L291 CN**: 继续与可调用符号 `NonLabelDoStmt>>` 相关的逻辑。
- **L292 EN**: Executes a standalone statement or declaration: `.statement.t)};`.
  **L292 CN**: 执行一条独立语句或声明：`.statement.t)};`。
- **L293 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L293 CN**: 开始 `if` 控制流语句并计算其条件。
- **L294 EN**: Executes a call or declaration centered on `constructNames_.emplace_back`.
  **L294 CN**: 执行以 `constructNames_.emplace_back` 为核心的调用或声明。
- **L295 EN**: Closes the current lexical scope or compound statement.
  **L295 CN**: 结束当前词法作用域或复合语句块。
- **L296 EN**: Comment explains nearby logic, intent, or metadata: `Allow FORTRAN '66 extended DO ranges`.
  **L296 CN**: 注释说明附近代码的逻辑、意图或元数据：`Allow FORTRAN '66 extended DO ranges`。
- **L297 EN**: Executes a call or declaration centered on `PushScope`.
  **L297 CN**: 执行以 `PushScope` 为核心的调用或声明。
- **L298 EN**: Comment explains nearby logic, intent, or metadata: `Process labels of the DO and END DO statements, but not the`.
  **L298 CN**: 注释说明附近代码的逻辑、意图或元数据：`Process labels of the DO and END DO statements, but not the`。
- **L299 EN**: Comment explains nearby logic, intent, or metadata: `statements themselves, so that a non-construct END DO`.
  **L299 CN**: 注释说明附近代码的逻辑、意图或元数据：`statements themselves, so that a non-construct END DO`。
- **L300 EN**: Comment explains nearby logic, intent, or metadata: `can be distinguished (below).`.
  **L300 CN**: 注释说明附近代码的逻辑、意图或元数据：`can be distinguished (below).`。
- **L301 EN**: Executes a call or declaration centered on `Pre`.
  **L301 CN**: 执行以 `Pre` 为核心的调用或声明。
- **L302 EN**: Executes a call or declaration centered on `Walk`.
  **L302 CN**: 执行以 `Walk` 为核心的调用或声明。
- **L303 EN**: Executes a call or declaration centered on `Pre`.
  **L303 CN**: 执行以 `Pre` 为核心的调用或声明。
- **L304 EN**: Executes a call or declaration centered on `PopConstructName`.
  **L304 CN**: 执行以 `PopConstructName` 为核心的调用或声明。
- **L305 EN**: Returns from the current function with `false`.
  **L305 CN**: 以 `false` 从当前函数返回。
- **L306 EN**: Closes the current lexical scope or compound statement.
  **L306 CN**: 结束当前词法作用域或复合语句块。
- **L307 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::EndDoStmt &endDoStmt) {`.
  **L307 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::EndDoStmt &endDoStmt) {`。
- **L308 EN**: Comment explains nearby logic, intent, or metadata: `Visited only for non-construct labeled DO termination`.
  **L308 CN**: 注释说明附近代码的逻辑、意图或元数据：`Visited only for non-construct labeled DO termination`。
- **L309 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L309 CN**: 开始 `if` 控制流语句并计算其条件。
- **L310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(name->source, "Unexpected DO construct name '%s'"_err_en_US,`.
  **L310 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(name->source, "Unexpected DO construct name '%s'"_err_en_US,`。
- **L311 EN**: Executes a standalone statement or declaration: `name->source);`.
  **L311 CN**: 执行一条独立语句或声明：`name->source);`。
- **L312 EN**: Closes the current lexical scope or compound statement.
  **L312 CN**: 结束当前词法作用域或复合语句块。

### Lines 313-336

````cpp
  }
  bool Pre(const parser::IfConstruct &ifConstruct) {
    return PushConstructName(ifConstruct);
  }
  void Post(const parser::IfThenStmt &) { PushScope(false); }
  bool Pre(const parser::IfConstruct::ElseIfBlock &) {
    return SwitchToNewScope();
  }
  bool Pre(const parser::IfConstruct::ElseBlock &) {
    return SwitchToNewScope();
  }
  bool Pre(const parser::EndIfStmt &) {
    PopScope();
    return true;
  }
  bool Pre(const parser::CaseConstruct &caseConstruct) {
    return PushConstructName(caseConstruct);
  }
  void Post(const parser::SelectCaseStmt &) { PushScope(false); }
  bool Pre(const parser::CaseConstruct::Case &) { return SwitchToNewScope(); }
  bool Pre(const parser::SelectRankConstruct &selectRankConstruct) {
    return PushConstructName(selectRankConstruct);
  }
  void Post(const parser::SelectRankStmt &) { PushScope(true); }
````
- **L313 EN**: Closes the current lexical scope or compound statement.
  **L313 CN**: 结束当前词法作用域或复合语句块。
- **L314 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::IfConstruct &ifConstruct) {`.
  **L314 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::IfConstruct &ifConstruct) {`。
- **L315 EN**: Returns from the current function with `PushConstructName(ifConstruct)`.
  **L315 CN**: 以 `PushConstructName(ifConstruct)` 从当前函数返回。
- **L316 EN**: Closes the current lexical scope or compound statement.
  **L316 CN**: 结束当前词法作用域或复合语句块。
- **L317 EN**: Continues logic associated with callable symbol `Post`.
  **L317 CN**: 继续与可调用符号 `Post` 相关的逻辑。
- **L318 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::IfConstruct::ElseIfBlock &) {`.
  **L318 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::IfConstruct::ElseIfBlock &) {`。
- **L319 EN**: Returns from the current function with `SwitchToNewScope()`.
  **L319 CN**: 以 `SwitchToNewScope()` 从当前函数返回。
- **L320 EN**: Closes the current lexical scope or compound statement.
  **L320 CN**: 结束当前词法作用域或复合语句块。
- **L321 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::IfConstruct::ElseBlock &) {`.
  **L321 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::IfConstruct::ElseBlock &) {`。
- **L322 EN**: Returns from the current function with `SwitchToNewScope()`.
  **L322 CN**: 以 `SwitchToNewScope()` 从当前函数返回。
- **L323 EN**: Closes the current lexical scope or compound statement.
  **L323 CN**: 结束当前词法作用域或复合语句块。
- **L324 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::EndIfStmt &) {`.
  **L324 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::EndIfStmt &) {`。
- **L325 EN**: Executes a call or declaration centered on `PopScope`.
  **L325 CN**: 执行以 `PopScope` 为核心的调用或声明。
- **L326 EN**: Returns from the current function with `true`.
  **L326 CN**: 以 `true` 从当前函数返回。
- **L327 EN**: Closes the current lexical scope or compound statement.
  **L327 CN**: 结束当前词法作用域或复合语句块。
- **L328 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::CaseConstruct &caseConstruct) {`.
  **L328 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::CaseConstruct &caseConstruct) {`。
- **L329 EN**: Returns from the current function with `PushConstructName(caseConstruct)`.
  **L329 CN**: 以 `PushConstructName(caseConstruct)` 从当前函数返回。
- **L330 EN**: Closes the current lexical scope or compound statement.
  **L330 CN**: 结束当前词法作用域或复合语句块。
- **L331 EN**: Continues logic associated with callable symbol `Post`.
  **L331 CN**: 继续与可调用符号 `Post` 相关的逻辑。
- **L332 EN**: Continues logic associated with callable symbol `Pre`.
  **L332 CN**: 继续与可调用符号 `Pre` 相关的逻辑。
- **L333 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::SelectRankConstruct &selectRankConstruct) {`.
  **L333 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::SelectRankConstruct &selectRankConstruct) {`。
- **L334 EN**: Returns from the current function with `PushConstructName(selectRankConstruct)`.
  **L334 CN**: 以 `PushConstructName(selectRankConstruct)` 从当前函数返回。
- **L335 EN**: Closes the current lexical scope or compound statement.
  **L335 CN**: 结束当前词法作用域或复合语句块。
- **L336 EN**: Continues logic associated with callable symbol `Post`.
  **L336 CN**: 继续与可调用符号 `Post` 相关的逻辑。

### Lines 337-360

````cpp
  bool Pre(const parser::SelectRankConstruct::RankCase &) {
    return SwitchToNewScope();
  }
  bool Pre(const parser::SelectTypeConstruct &selectTypeConstruct) {
    return PushConstructName(selectTypeConstruct);
  }
  void Post(const parser::SelectTypeStmt &) { PushScope(true); }
  bool Pre(const parser::SelectTypeConstruct::TypeCase &) {
    return SwitchToNewScope();
  }
  void Post(const parser::EndSelectStmt &) { PopScope(); }
  bool Pre(const parser::WhereConstruct &whereConstruct) {
    return PushConstructName(whereConstruct);
  }
  bool Pre(const parser::ForallConstruct &forallConstruct) {
    return PushConstructName(forallConstruct);
  }

  void Post(const parser::AssociateConstruct &associateConstruct) {
    PopConstructName(associateConstruct);
  }
  void Post(const parser::BlockConstruct &blockConstruct) {
    PopConstructName(blockConstruct);
  }
````
- **L337 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::SelectRankConstruct::RankCase &) {`.
  **L337 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::SelectRankConstruct::RankCase &) {`。
- **L338 EN**: Returns from the current function with `SwitchToNewScope()`.
  **L338 CN**: 以 `SwitchToNewScope()` 从当前函数返回。
- **L339 EN**: Closes the current lexical scope or compound statement.
  **L339 CN**: 结束当前词法作用域或复合语句块。
- **L340 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::SelectTypeConstruct &selectTypeConstruct) {`.
  **L340 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::SelectTypeConstruct &selectTypeConstruct) {`。
- **L341 EN**: Returns from the current function with `PushConstructName(selectTypeConstruct)`.
  **L341 CN**: 以 `PushConstructName(selectTypeConstruct)` 从当前函数返回。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Continues logic associated with callable symbol `Post`.
  **L343 CN**: 继续与可调用符号 `Post` 相关的逻辑。
- **L344 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::SelectTypeConstruct::TypeCase &) {`.
  **L344 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::SelectTypeConstruct::TypeCase &) {`。
- **L345 EN**: Returns from the current function with `SwitchToNewScope()`.
  **L345 CN**: 以 `SwitchToNewScope()` 从当前函数返回。
- **L346 EN**: Closes the current lexical scope or compound statement.
  **L346 CN**: 结束当前词法作用域或复合语句块。
- **L347 EN**: Continues logic associated with callable symbol `Post`.
  **L347 CN**: 继续与可调用符号 `Post` 相关的逻辑。
- **L348 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::WhereConstruct &whereConstruct) {`.
  **L348 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::WhereConstruct &whereConstruct) {`。
- **L349 EN**: Returns from the current function with `PushConstructName(whereConstruct)`.
  **L349 CN**: 以 `PushConstructName(whereConstruct)` 从当前函数返回。
- **L350 EN**: Closes the current lexical scope or compound statement.
  **L350 CN**: 结束当前词法作用域或复合语句块。
- **L351 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::ForallConstruct &forallConstruct) {`.
  **L351 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::ForallConstruct &forallConstruct) {`。
- **L352 EN**: Returns from the current function with `PushConstructName(forallConstruct)`.
  **L352 CN**: 以 `PushConstructName(forallConstruct)` 从当前函数返回。
- **L353 EN**: Closes the current lexical scope or compound statement.
  **L353 CN**: 结束当前词法作用域或复合语句块。
- **L354 EN**: Blank line separating nearby declarations or logic blocks.
  **L354 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L355 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::AssociateConstruct &associateConstruct) {`.
  **L355 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::AssociateConstruct &associateConstruct) {`。
- **L356 EN**: Executes a call or declaration centered on `PopConstructName`.
  **L356 CN**: 执行以 `PopConstructName` 为核心的调用或声明。
- **L357 EN**: Closes the current lexical scope or compound statement.
  **L357 CN**: 结束当前词法作用域或复合语句块。
- **L358 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::BlockConstruct &blockConstruct) {`.
  **L358 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::BlockConstruct &blockConstruct) {`。
- **L359 EN**: Executes a call or declaration centered on `PopConstructName`.
  **L359 CN**: 执行以 `PopConstructName` 为核心的调用或声明。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-384

````cpp
  void Post(const parser::ChangeTeamConstruct &changeTeamConstruct) {
    PopConstructName(changeTeamConstruct);
  }
  void Post(const parser::CriticalConstruct &criticalConstruct) {
    PopConstructName(criticalConstruct);
  }
  void Post(const parser::IfConstruct &ifConstruct) {
    PopConstructName(ifConstruct);
  }
  void Post(const parser::CaseConstruct &caseConstruct) {
    PopConstructName(caseConstruct);
  }
  void Post(const parser::SelectRankConstruct &selectRankConstruct) {
    PopConstructName(selectRankConstruct);
  }
  void Post(const parser::SelectTypeConstruct &selectTypeConstruct) {
    PopConstructName(selectTypeConstruct);
  }
  void Post(const parser::WhereConstruct &whereConstruct) {
    PopConstructName(whereConstruct);
  }
  void Post(const parser::ForallConstruct &forallConstruct) {
    PopConstructName(forallConstruct);
  }
````
- **L361 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::ChangeTeamConstruct &changeTeamConstruct) {`.
  **L361 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::ChangeTeamConstruct &changeTeamConstruct) {`。
- **L362 EN**: Executes a call or declaration centered on `PopConstructName`.
  **L362 CN**: 执行以 `PopConstructName` 为核心的调用或声明。
- **L363 EN**: Closes the current lexical scope or compound statement.
  **L363 CN**: 结束当前词法作用域或复合语句块。
- **L364 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::CriticalConstruct &criticalConstruct) {`.
  **L364 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::CriticalConstruct &criticalConstruct) {`。
- **L365 EN**: Executes a call or declaration centered on `PopConstructName`.
  **L365 CN**: 执行以 `PopConstructName` 为核心的调用或声明。
- **L366 EN**: Closes the current lexical scope or compound statement.
  **L366 CN**: 结束当前词法作用域或复合语句块。
- **L367 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::IfConstruct &ifConstruct) {`.
  **L367 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::IfConstruct &ifConstruct) {`。
- **L368 EN**: Executes a call or declaration centered on `PopConstructName`.
  **L368 CN**: 执行以 `PopConstructName` 为核心的调用或声明。
- **L369 EN**: Closes the current lexical scope or compound statement.
  **L369 CN**: 结束当前词法作用域或复合语句块。
- **L370 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::CaseConstruct &caseConstruct) {`.
  **L370 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::CaseConstruct &caseConstruct) {`。
- **L371 EN**: Executes a call or declaration centered on `PopConstructName`.
  **L371 CN**: 执行以 `PopConstructName` 为核心的调用或声明。
- **L372 EN**: Closes the current lexical scope or compound statement.
  **L372 CN**: 结束当前词法作用域或复合语句块。
- **L373 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::SelectRankConstruct &selectRankConstruct) {`.
  **L373 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::SelectRankConstruct &selectRankConstruct) {`。
- **L374 EN**: Executes a call or declaration centered on `PopConstructName`.
  **L374 CN**: 执行以 `PopConstructName` 为核心的调用或声明。
- **L375 EN**: Closes the current lexical scope or compound statement.
  **L375 CN**: 结束当前词法作用域或复合语句块。
- **L376 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::SelectTypeConstruct &selectTypeConstruct) {`.
  **L376 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::SelectTypeConstruct &selectTypeConstruct) {`。
- **L377 EN**: Executes a call or declaration centered on `PopConstructName`.
  **L377 CN**: 执行以 `PopConstructName` 为核心的调用或声明。
- **L378 EN**: Closes the current lexical scope or compound statement.
  **L378 CN**: 结束当前词法作用域或复合语句块。
- **L379 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::WhereConstruct &whereConstruct) {`.
  **L379 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::WhereConstruct &whereConstruct) {`。
- **L380 EN**: Executes a call or declaration centered on `PopConstructName`.
  **L380 CN**: 执行以 `PopConstructName` 为核心的调用或声明。
- **L381 EN**: Closes the current lexical scope or compound statement.
  **L381 CN**: 结束当前词法作用域或复合语句块。
- **L382 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::ForallConstruct &forallConstruct) {`.
  **L382 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::ForallConstruct &forallConstruct) {`。
- **L383 EN**: Executes a call or declaration centered on `PopConstructName`.
  **L383 CN**: 执行以 `PopConstructName` 为核心的调用或声明。
- **L384 EN**: Closes the current lexical scope or compound statement.
  **L384 CN**: 结束当前词法作用域或复合语句块。

### Lines 385-408

````cpp

  // Checks for missing or mismatching names on various constructs (e.g., IF)
  // and their intermediate or terminal statements that allow optional
  // construct names(e.g., ELSE).  When an optional construct name is present,
  // the construct as a whole must have a name that matches.
  template <typename FIRST, typename CONSTRUCT, typename STMT>
  void CheckOptionalName(const char *constructTag, const CONSTRUCT &a,
      const parser::Statement<STMT> &stmt) {
    if (const parser::CharBlock * name{GetStmtName(stmt)}) {
      const auto &firstStmt{std::get<parser::Statement<FIRST>>(a.t)};
      if (const parser::CharBlock * firstName{GetStmtName(firstStmt)}) {
        if (*firstName != *name) {
          context_.Say(*name, "%s name mismatch"_err_en_US, constructTag)
              .Attach(*firstName, "should be"_en_US);
        }
      } else {
        context_.Say(*name, "%s name not allowed"_err_en_US, constructTag)
            .Attach(firstStmt.source, "in unnamed %s"_en_US, constructTag);
      }
    }
  }

  // C1414
  void Post(const parser::BlockData &blockData) {
````
- **L385 EN**: Blank line separating nearby declarations or logic blocks.
  **L385 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L386 EN**: Comment explains nearby logic, intent, or metadata: `Checks for missing or mismatching names on various constructs (e.g., IF)`.
  **L386 CN**: 注释说明附近代码的逻辑、意图或元数据：`Checks for missing or mismatching names on various constructs (e.g., IF)`。
- **L387 EN**: Comment explains nearby logic, intent, or metadata: `and their intermediate or terminal statements that allow optional`.
  **L387 CN**: 注释说明附近代码的逻辑、意图或元数据：`and their intermediate or terminal statements that allow optional`。
- **L388 EN**: Comment explains nearby logic, intent, or metadata: `construct names(e.g., ELSE).  When an optional construct name is present,`.
  **L388 CN**: 注释说明附近代码的逻辑、意图或元数据：`construct names(e.g., ELSE).  When an optional construct name is present,`。
- **L389 EN**: Comment explains nearby logic, intent, or metadata: `the construct as a whole must have a name that matches.`.
  **L389 CN**: 注释说明附近代码的逻辑、意图或元数据：`the construct as a whole must have a name that matches.`。
- **L390 EN**: Introduces template parameters or specialization context: `template <typename FIRST, typename CONSTRUCT, typename STMT>`.
  **L390 CN**: 为后续声明引入模板参数或特化上下文：`template <typename FIRST, typename CONSTRUCT, typename STMT>`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckOptionalName(const char *constructTag, const CONSTRUCT &a,`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckOptionalName(const char *constructTag, const CONSTRUCT &a,`。
- **L392 EN**: Continues the surrounding expression or declaration: `const parser::Statement<STMT> &stmt) {`.
  **L392 CN**: 继续构造周围的表达式或声明：`const parser::Statement<STMT> &stmt) {`。
- **L393 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L393 CN**: 开始 `if` 控制流语句并计算其条件。
- **L394 EN**: Executes a call or declaration centered on `&firstStmt{std::get<parser::Statement<FIRST>>`.
  **L394 CN**: 执行以 `&firstStmt{std::get<parser::Statement<FIRST>>` 为核心的调用或声明。
- **L395 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L395 CN**: 开始 `if` 控制流语句并计算其条件。
- **L396 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L396 CN**: 开始 `if` 控制流语句并计算其条件。
- **L397 EN**: Continues logic associated with callable symbol `Say`.
  **L397 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L398 EN**: Executes a call or declaration centered on `.Attach`.
  **L398 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L399 EN**: Closes the current lexical scope or compound statement.
  **L399 CN**: 结束当前词法作用域或复合语句块。
- **L400 EN**: Transitions from the previous branch into the alternative path.
  **L400 CN**: 从前一个分支过渡到备选路径。
- **L401 EN**: Continues logic associated with callable symbol `Say`.
  **L401 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L402 EN**: Executes a call or declaration centered on `.Attach`.
  **L402 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L403 EN**: Closes the current lexical scope or compound statement.
  **L403 CN**: 结束当前词法作用域或复合语句块。
- **L404 EN**: Closes the current lexical scope or compound statement.
  **L404 CN**: 结束当前词法作用域或复合语句块。
- **L405 EN**: Closes the current lexical scope or compound statement.
  **L405 CN**: 结束当前词法作用域或复合语句块。
- **L406 EN**: Blank line separating nearby declarations or logic blocks.
  **L406 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L407 EN**: Comment explains nearby logic, intent, or metadata: `C1414`.
  **L407 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1414`。
- **L408 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::BlockData &blockData) {`.
  **L408 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::BlockData &blockData) {`。

### Lines 409-432

````cpp
    CheckOptionalName<parser::BlockDataStmt>("BLOCK DATA subprogram", blockData,
        std::get<parser::Statement<parser::EndBlockDataStmt>>(blockData.t));
  }

  bool Pre(const parser::InterfaceBody &) {
    PushDisposableMap();
    return true;
  }
  void Post(const parser::InterfaceBody &) { PopDisposableMap(); }

  // C1564
  void Post(const parser::InterfaceBody::Function &func) {
    CheckOptionalName<parser::FunctionStmt>("FUNCTION", func,
        std::get<parser::Statement<parser::EndFunctionStmt>>(func.t));
  }

  // C1564
  void Post(const parser::FunctionSubprogram &functionSubprogram) {
    CheckOptionalName<parser::FunctionStmt>("FUNCTION", functionSubprogram,
        std::get<parser::Statement<parser::EndFunctionStmt>>(
            functionSubprogram.t));
  }

  // C1502
````
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::BlockDataStmt>("BLOCK DATA subprogram", blockData,`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::BlockDataStmt>("BLOCK DATA subprogram", blockData,`。
- **L410 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::EndBlockDataStmt>>`.
  **L410 CN**: 执行以 `std::get<parser::Statement<parser::EndBlockDataStmt>>` 为核心的调用或声明。
- **L411 EN**: Closes the current lexical scope or compound statement.
  **L411 CN**: 结束当前词法作用域或复合语句块。
- **L412 EN**: Blank line separating nearby declarations or logic blocks.
  **L412 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L413 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::InterfaceBody &) {`.
  **L413 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::InterfaceBody &) {`。
- **L414 EN**: Executes a call or declaration centered on `PushDisposableMap`.
  **L414 CN**: 执行以 `PushDisposableMap` 为核心的调用或声明。
- **L415 EN**: Returns from the current function with `true`.
  **L415 CN**: 以 `true` 从当前函数返回。
- **L416 EN**: Closes the current lexical scope or compound statement.
  **L416 CN**: 结束当前词法作用域或复合语句块。
- **L417 EN**: Continues logic associated with callable symbol `Post`.
  **L417 CN**: 继续与可调用符号 `Post` 相关的逻辑。
- **L418 EN**: Blank line separating nearby declarations or logic blocks.
  **L418 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L419 EN**: Comment explains nearby logic, intent, or metadata: `C1564`.
  **L419 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1564`。
- **L420 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::InterfaceBody::Function &func) {`.
  **L420 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::InterfaceBody::Function &func) {`。
- **L421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::FunctionStmt>("FUNCTION", func,`.
  **L421 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::FunctionStmt>("FUNCTION", func,`。
- **L422 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::EndFunctionStmt>>`.
  **L422 CN**: 执行以 `std::get<parser::Statement<parser::EndFunctionStmt>>` 为核心的调用或声明。
- **L423 EN**: Closes the current lexical scope or compound statement.
  **L423 CN**: 结束当前词法作用域或复合语句块。
- **L424 EN**: Blank line separating nearby declarations or logic blocks.
  **L424 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L425 EN**: Comment explains nearby logic, intent, or metadata: `C1564`.
  **L425 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1564`。
- **L426 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::FunctionSubprogram &functionSubprogram) {`.
  **L426 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::FunctionSubprogram &functionSubprogram) {`。
- **L427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::FunctionStmt>("FUNCTION", functionSubprogram,`.
  **L427 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::FunctionStmt>("FUNCTION", functionSubprogram,`。
- **L428 EN**: Continues logic associated with callable symbol `EndFunctionStmt>>`.
  **L428 CN**: 继续与可调用符号 `EndFunctionStmt>>` 相关的逻辑。
- **L429 EN**: Executes a standalone statement or declaration: `functionSubprogram.t));`.
  **L429 CN**: 执行一条独立语句或声明：`functionSubprogram.t));`。
- **L430 EN**: Closes the current lexical scope or compound statement.
  **L430 CN**: 结束当前词法作用域或复合语句块。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L432 EN**: Comment explains nearby logic, intent, or metadata: `C1502`.
  **L432 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1502`。

### Lines 433-456

````cpp
  void Post(const parser::InterfaceBlock &interfaceBlock) {
    if (const auto &endGenericSpec{
            std::get<parser::Statement<parser::EndInterfaceStmt>>(
                interfaceBlock.t)
                .statement.v}) {
      const auto &interfaceStmt{
          std::get<parser::Statement<parser::InterfaceStmt>>(interfaceBlock.t)};
      if (std::holds_alternative<parser::Abstract>(interfaceStmt.statement.u)) {
        context_
            .Say(endGenericSpec->source,
                "END INTERFACE generic name (%s) may not appear for ABSTRACT INTERFACE"_err_en_US,
                endGenericSpec->source)
            .Attach(
                interfaceStmt.source, "corresponding ABSTRACT INTERFACE"_en_US);
      } else if (const auto &genericSpec{
                     std::get<std::optional<parser::GenericSpec>>(
                         interfaceStmt.statement.u)}) {
        bool ok{genericSpec->source == endGenericSpec->source};
        if (!ok) {
          // Accept variant spellings of .LT. &c.
          const auto *endOp{
              std::get_if<parser::DefinedOperator>(&endGenericSpec->u)};
          const auto *op{std::get_if<parser::DefinedOperator>(&genericSpec->u)};
          if (endOp && op) {
````
- **L433 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::InterfaceBlock &interfaceBlock) {`.
  **L433 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::InterfaceBlock &interfaceBlock) {`。
- **L434 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L434 CN**: 开始 `if` 控制流语句并计算其条件。
- **L435 EN**: Continues logic associated with callable symbol `EndInterfaceStmt>>`.
  **L435 CN**: 继续与可调用符号 `EndInterfaceStmt>>` 相关的逻辑。
- **L436 EN**: Continues the surrounding expression or declaration: `interfaceBlock.t)`.
  **L436 CN**: 继续构造周围的表达式或声明：`interfaceBlock.t)`。
- **L437 EN**: Continues the surrounding expression or declaration: `.statement.v}) {`.
  **L437 CN**: 继续构造周围的表达式或声明：`.statement.v}) {`。
- **L438 EN**: Continues the surrounding expression or declaration: `const auto &interfaceStmt{`.
  **L438 CN**: 继续构造周围的表达式或声明：`const auto &interfaceStmt{`。
- **L439 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::InterfaceStmt>>`.
  **L439 CN**: 执行以 `std::get<parser::Statement<parser::InterfaceStmt>>` 为核心的调用或声明。
- **L440 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L440 CN**: 开始 `if` 控制流语句并计算其条件。
- **L441 EN**: Continues the surrounding expression or declaration: `context_`.
  **L441 CN**: 继续构造周围的表达式或声明：`context_`。
- **L442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(endGenericSpec->source,`.
  **L442 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(endGenericSpec->source,`。
- **L443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"END INTERFACE generic name (%s) may not appear for ABSTRACT INTERFACE"_err_en_US,`.
  **L443 CN**: 继续一个多行参数列表、初始化器或聚合项：`"END INTERFACE generic name (%s) may not appear for ABSTRACT INTERFACE"_err_en_US,`。
- **L444 EN**: Continues the surrounding expression or declaration: `endGenericSpec->source)`.
  **L444 CN**: 继续构造周围的表达式或声明：`endGenericSpec->source)`。
- **L445 EN**: Continues logic associated with callable symbol `Attach`.
  **L445 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L446 EN**: Executes a standalone statement or declaration: `interfaceStmt.source, "corresponding ABSTRACT INTERFACE"_en_US);`.
  **L446 CN**: 执行一条独立语句或声明：`interfaceStmt.source, "corresponding ABSTRACT INTERFACE"_en_US);`。
- **L447 EN**: Transitions from the previous branch into an `else if` condition.
  **L447 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L448 EN**: Continues logic associated with callable symbol `GenericSpec>>`.
  **L448 CN**: 继续与可调用符号 `GenericSpec>>` 相关的逻辑。
- **L449 EN**: Continues the surrounding expression or declaration: `interfaceStmt.statement.u)}) {`.
  **L449 CN**: 继续构造周围的表达式或声明：`interfaceStmt.statement.u)}) {`。
- **L450 EN**: Executes a standalone statement or declaration: `bool ok{genericSpec->source == endGenericSpec->source};`.
  **L450 CN**: 执行一条独立语句或声明：`bool ok{genericSpec->source == endGenericSpec->source};`。
- **L451 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L451 CN**: 开始 `if` 控制流语句并计算其条件。
- **L452 EN**: Comment explains nearby logic, intent, or metadata: `Accept variant spellings of .LT. &c.`.
  **L452 CN**: 注释说明附近代码的逻辑、意图或元数据：`Accept variant spellings of .LT. &c.`。
- **L453 EN**: Continues the surrounding expression or declaration: `const auto *endOp{`.
  **L453 CN**: 继续构造周围的表达式或声明：`const auto *endOp{`。
- **L454 EN**: Executes a call or declaration centered on `std::get_if<parser::DefinedOperator>`.
  **L454 CN**: 执行以 `std::get_if<parser::DefinedOperator>` 为核心的调用或声明。
- **L455 EN**: Executes a call or declaration centered on `*op{std::get_if<parser::DefinedOperator>`.
  **L455 CN**: 执行以 `*op{std::get_if<parser::DefinedOperator>` 为核心的调用或声明。
- **L456 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L456 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 457-480

````cpp
            const auto *endIntrin{
                std::get_if<parser::DefinedOperator::IntrinsicOperator>(
                    &endOp->u)};
            const auto *intrin{
                std::get_if<parser::DefinedOperator::IntrinsicOperator>(
                    &op->u)};
            ok = endIntrin && intrin && *endIntrin == *intrin;
          }
        }
        if (!ok) {
          context_
              .Say(endGenericSpec->source,
                  "END INTERFACE generic name (%s) does not match generic INTERFACE (%s)"_err_en_US,
                  endGenericSpec->source, genericSpec->source)
              .Attach(genericSpec->source, "corresponding INTERFACE"_en_US);
        }
      } else {
        context_
            .Say(endGenericSpec->source,
                "END INTERFACE generic name (%s) may not appear for non-generic INTERFACE"_err_en_US,
                endGenericSpec->source)
            .Attach(interfaceStmt.source, "corresponding INTERFACE"_en_US);
      }
    }
````
- **L457 EN**: Continues the surrounding expression or declaration: `const auto *endIntrin{`.
  **L457 CN**: 继续构造周围的表达式或声明：`const auto *endIntrin{`。
- **L458 EN**: Continues logic associated with callable symbol `IntrinsicOperator>`.
  **L458 CN**: 继续与可调用符号 `IntrinsicOperator>` 相关的逻辑。
- **L459 EN**: Executes a standalone statement or declaration: `&endOp->u)};`.
  **L459 CN**: 执行一条独立语句或声明：`&endOp->u)};`。
- **L460 EN**: Continues the surrounding expression or declaration: `const auto *intrin{`.
  **L460 CN**: 继续构造周围的表达式或声明：`const auto *intrin{`。
- **L461 EN**: Continues logic associated with callable symbol `IntrinsicOperator>`.
  **L461 CN**: 继续与可调用符号 `IntrinsicOperator>` 相关的逻辑。
- **L462 EN**: Executes a standalone statement or declaration: `&op->u)};`.
  **L462 CN**: 执行一条独立语句或声明：`&op->u)};`。
- **L463 EN**: Executes a standalone statement or declaration: `ok = endIntrin && intrin && *endIntrin == *intrin;`.
  **L463 CN**: 执行一条独立语句或声明：`ok = endIntrin && intrin && *endIntrin == *intrin;`。
- **L464 EN**: Closes the current lexical scope or compound statement.
  **L464 CN**: 结束当前词法作用域或复合语句块。
- **L465 EN**: Closes the current lexical scope or compound statement.
  **L465 CN**: 结束当前词法作用域或复合语句块。
- **L466 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L466 CN**: 开始 `if` 控制流语句并计算其条件。
- **L467 EN**: Continues the surrounding expression or declaration: `context_`.
  **L467 CN**: 继续构造周围的表达式或声明：`context_`。
- **L468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(endGenericSpec->source,`.
  **L468 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(endGenericSpec->source,`。
- **L469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"END INTERFACE generic name (%s) does not match generic INTERFACE (%s)"_err_en_US,`.
  **L469 CN**: 继续一个多行参数列表、初始化器或聚合项：`"END INTERFACE generic name (%s) does not match generic INTERFACE (%s)"_err_en_US,`。
- **L470 EN**: Continues the surrounding expression or declaration: `endGenericSpec->source, genericSpec->source)`.
  **L470 CN**: 继续构造周围的表达式或声明：`endGenericSpec->source, genericSpec->source)`。
- **L471 EN**: Executes a call or declaration centered on `.Attach`.
  **L471 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L472 EN**: Closes the current lexical scope or compound statement.
  **L472 CN**: 结束当前词法作用域或复合语句块。
- **L473 EN**: Transitions from the previous branch into the alternative path.
  **L473 CN**: 从前一个分支过渡到备选路径。
- **L474 EN**: Continues the surrounding expression or declaration: `context_`.
  **L474 CN**: 继续构造周围的表达式或声明：`context_`。
- **L475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(endGenericSpec->source,`.
  **L475 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(endGenericSpec->source,`。
- **L476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"END INTERFACE generic name (%s) may not appear for non-generic INTERFACE"_err_en_US,`.
  **L476 CN**: 继续一个多行参数列表、初始化器或聚合项：`"END INTERFACE generic name (%s) may not appear for non-generic INTERFACE"_err_en_US,`。
- **L477 EN**: Continues the surrounding expression or declaration: `endGenericSpec->source)`.
  **L477 CN**: 继续构造周围的表达式或声明：`endGenericSpec->source)`。
- **L478 EN**: Executes a call or declaration centered on `.Attach`.
  **L478 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L479 EN**: Closes the current lexical scope or compound statement.
  **L479 CN**: 结束当前词法作用域或复合语句块。
- **L480 EN**: Closes the current lexical scope or compound statement.
  **L480 CN**: 结束当前词法作用域或复合语句块。

### Lines 481-504

````cpp
  }

  // C1402
  void Post(const parser::Module &module) {
    CheckOptionalName<parser::ModuleStmt>("MODULE", module,
        std::get<parser::Statement<parser::EndModuleStmt>>(module.t));
  }

  // C1569
  void Post(const parser::SeparateModuleSubprogram &separateModuleSubprogram) {
    CheckOptionalName<parser::MpSubprogramStmt>("MODULE PROCEDURE",
        separateModuleSubprogram,
        std::get<parser::Statement<parser::EndMpSubprogramStmt>>(
            separateModuleSubprogram.t));
  }

  // C1401
  void Post(const parser::MainProgram &mainProgram) {
    // Uppercase the name of the main program, so that its symbol name
    // would be unique from similarly named non-main-program symbols.
    auto upperCaseCharBlock = [](const parser::CharBlock &cb) {
      auto ch{const_cast<char *>(cb.begin())};
      for (char *endCh{ch + cb.size()}; ch != endCh; ++ch) {
        *ch = parser::ToUpperCaseLetter(*ch);
````
- **L481 EN**: Closes the current lexical scope or compound statement.
  **L481 CN**: 结束当前词法作用域或复合语句块。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L483 EN**: Comment explains nearby logic, intent, or metadata: `C1402`.
  **L483 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1402`。
- **L484 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::Module &module) {`.
  **L484 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::Module &module) {`。
- **L485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::ModuleStmt>("MODULE", module,`.
  **L485 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::ModuleStmt>("MODULE", module,`。
- **L486 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::EndModuleStmt>>`.
  **L486 CN**: 执行以 `std::get<parser::Statement<parser::EndModuleStmt>>` 为核心的调用或声明。
- **L487 EN**: Closes the current lexical scope or compound statement.
  **L487 CN**: 结束当前词法作用域或复合语句块。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L489 EN**: Comment explains nearby logic, intent, or metadata: `C1569`.
  **L489 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1569`。
- **L490 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::SeparateModuleSubprogram &separateModuleSubprogram) {`.
  **L490 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::SeparateModuleSubprogram &separateModuleSubprogram) {`。
- **L491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::MpSubprogramStmt>("MODULE PROCEDURE",`.
  **L491 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::MpSubprogramStmt>("MODULE PROCEDURE",`。
- **L492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `separateModuleSubprogram,`.
  **L492 CN**: 继续一个多行参数列表、初始化器或聚合项：`separateModuleSubprogram,`。
- **L493 EN**: Continues logic associated with callable symbol `EndMpSubprogramStmt>>`.
  **L493 CN**: 继续与可调用符号 `EndMpSubprogramStmt>>` 相关的逻辑。
- **L494 EN**: Executes a standalone statement or declaration: `separateModuleSubprogram.t));`.
  **L494 CN**: 执行一条独立语句或声明：`separateModuleSubprogram.t));`。
- **L495 EN**: Closes the current lexical scope or compound statement.
  **L495 CN**: 结束当前词法作用域或复合语句块。
- **L496 EN**: Blank line separating nearby declarations or logic blocks.
  **L496 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L497 EN**: Comment explains nearby logic, intent, or metadata: `C1401`.
  **L497 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1401`。
- **L498 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::MainProgram &mainProgram) {`.
  **L498 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::MainProgram &mainProgram) {`。
- **L499 EN**: Comment explains nearby logic, intent, or metadata: `Uppercase the name of the main program, so that its symbol name`.
  **L499 CN**: 注释说明附近代码的逻辑、意图或元数据：`Uppercase the name of the main program, so that its symbol name`。
- **L500 EN**: Comment explains nearby logic, intent, or metadata: `would be unique from similarly named non-main-program symbols.`.
  **L500 CN**: 注释说明附近代码的逻辑、意图或元数据：`would be unique from similarly named non-main-program symbols.`。
- **L501 EN**: Starts a function, method, lambda, or structured scope: `auto upperCaseCharBlock = [](const parser::CharBlock &cb) {`.
  **L501 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto upperCaseCharBlock = [](const parser::CharBlock &cb) {`。
- **L502 EN**: Executes a call or declaration centered on `*>`.
  **L502 CN**: 执行以 `*>` 为核心的调用或声明。
- **L503 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L503 CN**: 开始 `for` 控制流语句并计算其条件。
- **L504 EN**: Comment explains nearby logic, intent, or metadata: `ch = parser::ToUpperCaseLetter(*ch);`.
  **L504 CN**: 注释说明附近代码的逻辑、意图或元数据：`ch = parser::ToUpperCaseLetter(*ch);`。

### Lines 505-528

````cpp
      }
    };
    const parser::CharBlock *progName{nullptr};
    if (const auto &program{
            std::get<std::optional<parser::Statement<parser::ProgramStmt>>>(
                mainProgram.t)}) {
      progName = &program->statement.v.source;
      upperCaseCharBlock(*progName);
    }
    if (const parser::CharBlock *
        endName{GetStmtName(std::get<parser::Statement<parser::EndProgramStmt>>(
            mainProgram.t))}) {
      upperCaseCharBlock(*endName);
      if (progName) {
        if (*endName != *progName) {
          context_.Say(*endName, "END PROGRAM name mismatch"_err_en_US)
              .Attach(*progName, "should be"_en_US);
        }
      } else {
        context_.Say(*endName,
            "END PROGRAM has name without PROGRAM statement"_err_en_US);
      }
    }
  }
````
- **L505 EN**: Closes the current lexical scope or compound statement.
  **L505 CN**: 结束当前词法作用域或复合语句块。
- **L506 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L506 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L507 EN**: Executes a standalone statement or declaration: `const parser::CharBlock *progName{nullptr};`.
  **L507 CN**: 执行一条独立语句或声明：`const parser::CharBlock *progName{nullptr};`。
- **L508 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L508 CN**: 开始 `if` 控制流语句并计算其条件。
- **L509 EN**: Continues logic associated with callable symbol `ProgramStmt>>>`.
  **L509 CN**: 继续与可调用符号 `ProgramStmt>>>` 相关的逻辑。
- **L510 EN**: Continues the surrounding expression or declaration: `mainProgram.t)}) {`.
  **L510 CN**: 继续构造周围的表达式或声明：`mainProgram.t)}) {`。
- **L511 EN**: Executes a standalone statement or declaration: `progName = &program->statement.v.source;`.
  **L511 CN**: 执行一条独立语句或声明：`progName = &program->statement.v.source;`。
- **L512 EN**: Executes a call or declaration centered on `upperCaseCharBlock`.
  **L512 CN**: 执行以 `upperCaseCharBlock` 为核心的调用或声明。
- **L513 EN**: Closes the current lexical scope or compound statement.
  **L513 CN**: 结束当前词法作用域或复合语句块。
- **L514 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L514 CN**: 开始 `if` 控制流语句并计算其条件。
- **L515 EN**: Continues logic associated with callable symbol `GetStmtName`.
  **L515 CN**: 继续与可调用符号 `GetStmtName` 相关的逻辑。
- **L516 EN**: Continues the surrounding expression or declaration: `mainProgram.t))}) {`.
  **L516 CN**: 继续构造周围的表达式或声明：`mainProgram.t))}) {`。
- **L517 EN**: Executes a call or declaration centered on `upperCaseCharBlock`.
  **L517 CN**: 执行以 `upperCaseCharBlock` 为核心的调用或声明。
- **L518 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L518 CN**: 开始 `if` 控制流语句并计算其条件。
- **L519 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L519 CN**: 开始 `if` 控制流语句并计算其条件。
- **L520 EN**: Continues logic associated with callable symbol `Say`.
  **L520 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L521 EN**: Executes a call or declaration centered on `.Attach`.
  **L521 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L522 EN**: Closes the current lexical scope or compound statement.
  **L522 CN**: 结束当前词法作用域或复合语句块。
- **L523 EN**: Transitions from the previous branch into the alternative path.
  **L523 CN**: 从前一个分支过渡到备选路径。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(*endName,`.
  **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(*endName,`。
- **L525 EN**: Executes a standalone statement or declaration: `"END PROGRAM has name without PROGRAM statement"_err_en_US);`.
  **L525 CN**: 执行一条独立语句或声明：`"END PROGRAM has name without PROGRAM statement"_err_en_US);`。
- **L526 EN**: Closes the current lexical scope or compound statement.
  **L526 CN**: 结束当前词法作用域或复合语句块。
- **L527 EN**: Closes the current lexical scope or compound statement.
  **L527 CN**: 结束当前词法作用域或复合语句块。
- **L528 EN**: Closes the current lexical scope or compound statement.
  **L528 CN**: 结束当前词法作用域或复合语句块。

### Lines 529-552

````cpp

  // C1413
  void Post(const parser::Submodule &submodule) {
    CheckOptionalName<parser::SubmoduleStmt>("SUBMODULE", submodule,
        std::get<parser::Statement<parser::EndSubmoduleStmt>>(submodule.t));
  }

  // C1567
  void Post(const parser::InterfaceBody::Subroutine &sub) {
    CheckOptionalName<parser::SubroutineStmt>("SUBROUTINE", sub,
        std::get<parser::Statement<parser::EndSubroutineStmt>>(sub.t));
  }

  // C1567
  void Post(const parser::SubroutineSubprogram &subroutineSubprogram) {
    CheckOptionalName<parser::SubroutineStmt>("SUBROUTINE",
        subroutineSubprogram,
        std::get<parser::Statement<parser::EndSubroutineStmt>>(
            subroutineSubprogram.t));
  }

  // C739
  bool Pre(const parser::DerivedTypeDef &) {
    PushDisposableMap();
````
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L530 EN**: Comment explains nearby logic, intent, or metadata: `C1413`.
  **L530 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1413`。
- **L531 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::Submodule &submodule) {`.
  **L531 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::Submodule &submodule) {`。
- **L532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::SubmoduleStmt>("SUBMODULE", submodule,`.
  **L532 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::SubmoduleStmt>("SUBMODULE", submodule,`。
- **L533 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::EndSubmoduleStmt>>`.
  **L533 CN**: 执行以 `std::get<parser::Statement<parser::EndSubmoduleStmt>>` 为核心的调用或声明。
- **L534 EN**: Closes the current lexical scope or compound statement.
  **L534 CN**: 结束当前词法作用域或复合语句块。
- **L535 EN**: Blank line separating nearby declarations or logic blocks.
  **L535 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L536 EN**: Comment explains nearby logic, intent, or metadata: `C1567`.
  **L536 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1567`。
- **L537 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::InterfaceBody::Subroutine &sub) {`.
  **L537 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::InterfaceBody::Subroutine &sub) {`。
- **L538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::SubroutineStmt>("SUBROUTINE", sub,`.
  **L538 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::SubroutineStmt>("SUBROUTINE", sub,`。
- **L539 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::EndSubroutineStmt>>`.
  **L539 CN**: 执行以 `std::get<parser::Statement<parser::EndSubroutineStmt>>` 为核心的调用或声明。
- **L540 EN**: Closes the current lexical scope or compound statement.
  **L540 CN**: 结束当前词法作用域或复合语句块。
- **L541 EN**: Blank line separating nearby declarations or logic blocks.
  **L541 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L542 EN**: Comment explains nearby logic, intent, or metadata: `C1567`.
  **L542 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1567`。
- **L543 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::SubroutineSubprogram &subroutineSubprogram) {`.
  **L543 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::SubroutineSubprogram &subroutineSubprogram) {`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::SubroutineStmt>("SUBROUTINE",`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::SubroutineStmt>("SUBROUTINE",`。
- **L545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `subroutineSubprogram,`.
  **L545 CN**: 继续一个多行参数列表、初始化器或聚合项：`subroutineSubprogram,`。
- **L546 EN**: Continues logic associated with callable symbol `EndSubroutineStmt>>`.
  **L546 CN**: 继续与可调用符号 `EndSubroutineStmt>>` 相关的逻辑。
- **L547 EN**: Executes a standalone statement or declaration: `subroutineSubprogram.t));`.
  **L547 CN**: 执行一条独立语句或声明：`subroutineSubprogram.t));`。
- **L548 EN**: Closes the current lexical scope or compound statement.
  **L548 CN**: 结束当前词法作用域或复合语句块。
- **L549 EN**: Blank line separating nearby declarations or logic blocks.
  **L549 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L550 EN**: Comment explains nearby logic, intent, or metadata: `C739`.
  **L550 CN**: 注释说明附近代码的逻辑、意图或元数据：`C739`。
- **L551 EN**: Starts a function, method, lambda, or structured scope: `bool Pre(const parser::DerivedTypeDef &) {`.
  **L551 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Pre(const parser::DerivedTypeDef &) {`。
- **L552 EN**: Executes a call or declaration centered on `PushDisposableMap`.
  **L552 CN**: 执行以 `PushDisposableMap` 为核心的调用或声明。

### Lines 553-576

````cpp
    return true;
  }
  void Post(const parser::DerivedTypeDef &derivedTypeDef) {
    CheckOptionalName<parser::DerivedTypeStmt>("derived type definition",
        derivedTypeDef,
        std::get<parser::Statement<parser::EndTypeStmt>>(derivedTypeDef.t));
    PopDisposableMap();
  }

  void Post(const parser::LabelDoStmt &labelDoStmt) {
    AddLabelReferenceFromDoStmt(std::get<parser::Label>(labelDoStmt.t));
  }
  void Post(const parser::GotoStmt &gotoStmt) { AddLabelReference(gotoStmt.v); }
  void Post(const parser::ComputedGotoStmt &computedGotoStmt) {
    AddLabelReference(std::get<std::list<parser::Label>>(computedGotoStmt.t));
  }
  void Post(const parser::ArithmeticIfStmt &arithmeticIfStmt) {
    AddLabelReference(std::get<1>(arithmeticIfStmt.t));
    AddLabelReference(std::get<2>(arithmeticIfStmt.t));
    AddLabelReference(std::get<3>(arithmeticIfStmt.t));
  }
  void Post(const parser::AssignStmt &assignStmt) {
    AddLabelReferenceFromAssignStmt(std::get<parser::Label>(assignStmt.t));
  }
````
- **L553 EN**: Returns from the current function with `true`.
  **L553 CN**: 以 `true` 从当前函数返回。
- **L554 EN**: Closes the current lexical scope or compound statement.
  **L554 CN**: 结束当前词法作用域或复合语句块。
- **L555 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::DerivedTypeDef &derivedTypeDef) {`.
  **L555 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::DerivedTypeDef &derivedTypeDef) {`。
- **L556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::DerivedTypeStmt>("derived type definition",`.
  **L556 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::DerivedTypeStmt>("derived type definition",`。
- **L557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `derivedTypeDef,`.
  **L557 CN**: 继续一个多行参数列表、初始化器或聚合项：`derivedTypeDef,`。
- **L558 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::EndTypeStmt>>`.
  **L558 CN**: 执行以 `std::get<parser::Statement<parser::EndTypeStmt>>` 为核心的调用或声明。
- **L559 EN**: Executes a call or declaration centered on `PopDisposableMap`.
  **L559 CN**: 执行以 `PopDisposableMap` 为核心的调用或声明。
- **L560 EN**: Closes the current lexical scope or compound statement.
  **L560 CN**: 结束当前词法作用域或复合语句块。
- **L561 EN**: Blank line separating nearby declarations or logic blocks.
  **L561 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L562 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::LabelDoStmt &labelDoStmt) {`.
  **L562 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::LabelDoStmt &labelDoStmt) {`。
- **L563 EN**: Executes a call or declaration centered on `AddLabelReferenceFromDoStmt`.
  **L563 CN**: 执行以 `AddLabelReferenceFromDoStmt` 为核心的调用或声明。
- **L564 EN**: Closes the current lexical scope or compound statement.
  **L564 CN**: 结束当前词法作用域或复合语句块。
- **L565 EN**: Continues logic associated with callable symbol `Post`.
  **L565 CN**: 继续与可调用符号 `Post` 相关的逻辑。
- **L566 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::ComputedGotoStmt &computedGotoStmt) {`.
  **L566 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::ComputedGotoStmt &computedGotoStmt) {`。
- **L567 EN**: Executes a call or declaration centered on `AddLabelReference`.
  **L567 CN**: 执行以 `AddLabelReference` 为核心的调用或声明。
- **L568 EN**: Closes the current lexical scope or compound statement.
  **L568 CN**: 结束当前词法作用域或复合语句块。
- **L569 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::ArithmeticIfStmt &arithmeticIfStmt) {`.
  **L569 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::ArithmeticIfStmt &arithmeticIfStmt) {`。
- **L570 EN**: Executes a call or declaration centered on `AddLabelReference`.
  **L570 CN**: 执行以 `AddLabelReference` 为核心的调用或声明。
- **L571 EN**: Executes a call or declaration centered on `AddLabelReference`.
  **L571 CN**: 执行以 `AddLabelReference` 为核心的调用或声明。
- **L572 EN**: Executes a call or declaration centered on `AddLabelReference`.
  **L572 CN**: 执行以 `AddLabelReference` 为核心的调用或声明。
- **L573 EN**: Closes the current lexical scope or compound statement.
  **L573 CN**: 结束当前词法作用域或复合语句块。
- **L574 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::AssignStmt &assignStmt) {`.
  **L574 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::AssignStmt &assignStmt) {`。
- **L575 EN**: Executes a call or declaration centered on `AddLabelReferenceFromAssignStmt`.
  **L575 CN**: 执行以 `AddLabelReferenceFromAssignStmt` 为核心的调用或声明。
- **L576 EN**: Closes the current lexical scope or compound statement.
  **L576 CN**: 结束当前词法作用域或复合语句块。

### Lines 577-600

````cpp
  void Post(const parser::AssignedGotoStmt &assignedGotoStmt) {
    AddLabelReference(std::get<std::list<parser::Label>>(assignedGotoStmt.t));
  }
  void Post(const parser::AltReturnSpec &altReturnSpec) {
    AddLabelReference(altReturnSpec.v);
  }

  void Post(const parser::ErrLabel &errLabel) { AddLabelReference(errLabel.v); }
  void Post(const parser::EndLabel &endLabel) { AddLabelReference(endLabel.v); }
  void Post(const parser::EorLabel &eorLabel) { AddLabelReference(eorLabel.v); }
  void Post(const parser::Format &format) {
    if (const auto *labelPointer{std::get_if<parser::Label>(&format.u)}) {
      AddLabelReferenceToFormatStmt(*labelPointer);
    }
  }
  void Post(const parser::CycleStmt &cycleStmt) {
    if (cycleStmt.v) {
      CheckLabelContext("CYCLE", cycleStmt.v->source);
    }
  }
  void Post(const parser::ExitStmt &exitStmt) {
    if (exitStmt.v) {
      CheckLabelContext("EXIT", exitStmt.v->source);
    }
````
- **L577 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::AssignedGotoStmt &assignedGotoStmt) {`.
  **L577 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::AssignedGotoStmt &assignedGotoStmt) {`。
- **L578 EN**: Executes a call or declaration centered on `AddLabelReference`.
  **L578 CN**: 执行以 `AddLabelReference` 为核心的调用或声明。
- **L579 EN**: Closes the current lexical scope or compound statement.
  **L579 CN**: 结束当前词法作用域或复合语句块。
- **L580 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::AltReturnSpec &altReturnSpec) {`.
  **L580 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::AltReturnSpec &altReturnSpec) {`。
- **L581 EN**: Executes a call or declaration centered on `AddLabelReference`.
  **L581 CN**: 执行以 `AddLabelReference` 为核心的调用或声明。
- **L582 EN**: Closes the current lexical scope or compound statement.
  **L582 CN**: 结束当前词法作用域或复合语句块。
- **L583 EN**: Blank line separating nearby declarations or logic blocks.
  **L583 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L584 EN**: Continues logic associated with callable symbol `Post`.
  **L584 CN**: 继续与可调用符号 `Post` 相关的逻辑。
- **L585 EN**: Continues logic associated with callable symbol `Post`.
  **L585 CN**: 继续与可调用符号 `Post` 相关的逻辑。
- **L586 EN**: Continues logic associated with callable symbol `Post`.
  **L586 CN**: 继续与可调用符号 `Post` 相关的逻辑。
- **L587 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::Format &format) {`.
  **L587 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::Format &format) {`。
- **L588 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L588 CN**: 开始 `if` 控制流语句并计算其条件。
- **L589 EN**: Executes a call or declaration centered on `AddLabelReferenceToFormatStmt`.
  **L589 CN**: 执行以 `AddLabelReferenceToFormatStmt` 为核心的调用或声明。
- **L590 EN**: Closes the current lexical scope or compound statement.
  **L590 CN**: 结束当前词法作用域或复合语句块。
- **L591 EN**: Closes the current lexical scope or compound statement.
  **L591 CN**: 结束当前词法作用域或复合语句块。
- **L592 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::CycleStmt &cycleStmt) {`.
  **L592 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::CycleStmt &cycleStmt) {`。
- **L593 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L593 CN**: 开始 `if` 控制流语句并计算其条件。
- **L594 EN**: Executes a call or declaration centered on `CheckLabelContext`.
  **L594 CN**: 执行以 `CheckLabelContext` 为核心的调用或声明。
- **L595 EN**: Closes the current lexical scope or compound statement.
  **L595 CN**: 结束当前词法作用域或复合语句块。
- **L596 EN**: Closes the current lexical scope or compound statement.
  **L596 CN**: 结束当前词法作用域或复合语句块。
- **L597 EN**: Starts a function, method, lambda, or structured scope: `void Post(const parser::ExitStmt &exitStmt) {`.
  **L597 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Post(const parser::ExitStmt &exitStmt) {`。
- **L598 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L598 CN**: 开始 `if` 控制流语句并计算其条件。
- **L599 EN**: Executes a call or declaration centered on `CheckLabelContext`.
  **L599 CN**: 执行以 `CheckLabelContext` 为核心的调用或声明。
- **L600 EN**: Closes the current lexical scope or compound statement.
  **L600 CN**: 结束当前词法作用域或复合语句块。

### Lines 601-624

````cpp
  }

  const std::vector<UnitAnalysis> &ProgramUnits() const {
    return programUnits_;
  }
  SemanticsContext &ErrorHandler() { return context_; }

private:
  ScopeInfo &PushScope(bool isExteriorGotoFatal) {
    auto &model{programUnits_.back().scopeModel};
    int newDepth{model.empty() ? 1 : model[currentScope_].depth + 1};
    ScopeInfo &result{model.emplace_back()};
    result.parent = currentScope_;
    result.depth = newDepth;
    result.isExteriorGotoFatal = isExteriorGotoFatal;
    currentScope_ = model.size() - 1;
    return result;
  }
  bool InitializeNewScopeContext() {
    programUnits_.emplace_back(UnitAnalysis{});
    currentScope_ = 0u;
    PushScope(false);
    return true;
  }
````
- **L601 EN**: Closes the current lexical scope or compound statement.
  **L601 CN**: 结束当前词法作用域或复合语句块。
- **L602 EN**: Blank line separating nearby declarations or logic blocks.
  **L602 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L603 EN**: Starts a function, method, lambda, or structured scope: `const std::vector<UnitAnalysis> &ProgramUnits() const {`.
  **L603 CN**: 开始一个函数、方法、lambda 或结构化作用域：`const std::vector<UnitAnalysis> &ProgramUnits() const {`。
- **L604 EN**: Returns from the current function with `programUnits_`.
  **L604 CN**: 以 `programUnits_` 从当前函数返回。
- **L605 EN**: Closes the current lexical scope or compound statement.
  **L605 CN**: 结束当前词法作用域或复合语句块。
- **L606 EN**: Continues logic associated with callable symbol `ErrorHandler`.
  **L606 CN**: 继续与可调用符号 `ErrorHandler` 相关的逻辑。
- **L607 EN**: Blank line separating nearby declarations or logic blocks.
  **L607 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L608 EN**: Sets the following members to `private` access.
  **L608 CN**: 将后续成员的访问级别设为 `private`。
- **L609 EN**: Starts a function, method, lambda, or structured scope: `ScopeInfo &PushScope(bool isExteriorGotoFatal) {`.
  **L609 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ScopeInfo &PushScope(bool isExteriorGotoFatal) {`。
- **L610 EN**: Executes a call or declaration centered on `&model{programUnits_.back`.
  **L610 CN**: 执行以 `&model{programUnits_.back` 为核心的调用或声明。
- **L611 EN**: Executes a call or declaration centered on `newDepth{model.empty`.
  **L611 CN**: 执行以 `newDepth{model.empty` 为核心的调用或声明。
- **L612 EN**: Executes a call or declaration centered on `&result{model.emplace_back`.
  **L612 CN**: 执行以 `&result{model.emplace_back` 为核心的调用或声明。
- **L613 EN**: Executes a standalone statement or declaration: `result.parent = currentScope_;`.
  **L613 CN**: 执行一条独立语句或声明：`result.parent = currentScope_;`。
- **L614 EN**: Executes a standalone statement or declaration: `result.depth = newDepth;`.
  **L614 CN**: 执行一条独立语句或声明：`result.depth = newDepth;`。
- **L615 EN**: Executes a standalone statement or declaration: `result.isExteriorGotoFatal = isExteriorGotoFatal;`.
  **L615 CN**: 执行一条独立语句或声明：`result.isExteriorGotoFatal = isExteriorGotoFatal;`。
- **L616 EN**: Executes a call or declaration centered on `model.size`.
  **L616 CN**: 执行以 `model.size` 为核心的调用或声明。
- **L617 EN**: Returns from the current function with `result`.
  **L617 CN**: 以 `result` 从当前函数返回。
- **L618 EN**: Closes the current lexical scope or compound statement.
  **L618 CN**: 结束当前词法作用域或复合语句块。
- **L619 EN**: Starts a function, method, lambda, or structured scope: `bool InitializeNewScopeContext() {`.
  **L619 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool InitializeNewScopeContext() {`。
- **L620 EN**: Executes a call or declaration centered on `programUnits_.emplace_back`.
  **L620 CN**: 执行以 `programUnits_.emplace_back` 为核心的调用或声明。
- **L621 EN**: Executes a standalone statement or declaration: `currentScope_ = 0u;`.
  **L621 CN**: 执行一条独立语句或声明：`currentScope_ = 0u;`。
- **L622 EN**: Executes a call or declaration centered on `PushScope`.
  **L622 CN**: 执行以 `PushScope` 为核心的调用或声明。
- **L623 EN**: Returns from the current function with `true`.
  **L623 CN**: 以 `true` 从当前函数返回。
- **L624 EN**: Closes the current lexical scope or compound statement.
  **L624 CN**: 结束当前词法作用域或复合语句块。

### Lines 625-648

````cpp
  ScopeInfo &PopScope() {
    ScopeInfo &result{programUnits_.back().scopeModel[currentScope_]};
    currentScope_ = result.parent;
    return result;
  }
  ProxyForScope ParentScope() {
    return programUnits_.back().scopeModel[currentScope_].parent;
  }
  bool SwitchToNewScope() {
    PushScope(PopScope().isExteriorGotoFatal);
    return true;
  }

  template <typename A> bool PushConstructName(const A &a) {
    const auto &optionalName{std::get<0>(std::get<0>(a.t).statement.t)};
    if (optionalName) {
      constructNames_.emplace_back(optionalName->ToString());
    }
    // Gotos into this construct from outside it are diagnosed, and
    // are fatal unless the construct is a DO, IF, or SELECT CASE.
    PushScope(!(std::is_same_v<A, parser::DoConstruct> ||
        std::is_same_v<A, parser::IfConstruct> ||
        std::is_same_v<A, parser::CaseConstruct>));
    return true;
````
- **L625 EN**: Starts a function, method, lambda, or structured scope: `ScopeInfo &PopScope() {`.
  **L625 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ScopeInfo &PopScope() {`。
- **L626 EN**: Executes a call or declaration centered on `&result{programUnits_.back`.
  **L626 CN**: 执行以 `&result{programUnits_.back` 为核心的调用或声明。
- **L627 EN**: Executes a standalone statement or declaration: `currentScope_ = result.parent;`.
  **L627 CN**: 执行一条独立语句或声明：`currentScope_ = result.parent;`。
- **L628 EN**: Returns from the current function with `result`.
  **L628 CN**: 以 `result` 从当前函数返回。
- **L629 EN**: Closes the current lexical scope or compound statement.
  **L629 CN**: 结束当前词法作用域或复合语句块。
- **L630 EN**: Starts a function, method, lambda, or structured scope: `ProxyForScope ParentScope() {`.
  **L630 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ProxyForScope ParentScope() {`。
- **L631 EN**: Returns from the current function with `programUnits_.back().scopeModel[currentScope_].parent`.
  **L631 CN**: 以 `programUnits_.back().scopeModel[currentScope_].parent` 从当前函数返回。
- **L632 EN**: Closes the current lexical scope or compound statement.
  **L632 CN**: 结束当前词法作用域或复合语句块。
- **L633 EN**: Starts a function, method, lambda, or structured scope: `bool SwitchToNewScope() {`.
  **L633 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool SwitchToNewScope() {`。
- **L634 EN**: Executes a call or declaration centered on `PushScope`.
  **L634 CN**: 执行以 `PushScope` 为核心的调用或声明。
- **L635 EN**: Returns from the current function with `true`.
  **L635 CN**: 以 `true` 从当前函数返回。
- **L636 EN**: Closes the current lexical scope or compound statement.
  **L636 CN**: 结束当前词法作用域或复合语句块。
- **L637 EN**: Blank line separating nearby declarations or logic blocks.
  **L637 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L638 EN**: Introduces template parameters or specialization context: `template <typename A> bool PushConstructName(const A &a) {`.
  **L638 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A> bool PushConstructName(const A &a) {`。
- **L639 EN**: Executes a call or declaration centered on `&optionalName{std::get<0>`.
  **L639 CN**: 执行以 `&optionalName{std::get<0>` 为核心的调用或声明。
- **L640 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L640 CN**: 开始 `if` 控制流语句并计算其条件。
- **L641 EN**: Executes a call or declaration centered on `constructNames_.emplace_back`.
  **L641 CN**: 执行以 `constructNames_.emplace_back` 为核心的调用或声明。
- **L642 EN**: Closes the current lexical scope or compound statement.
  **L642 CN**: 结束当前词法作用域或复合语句块。
- **L643 EN**: Comment explains nearby logic, intent, or metadata: `Gotos into this construct from outside it are diagnosed, and`.
  **L643 CN**: 注释说明附近代码的逻辑、意图或元数据：`Gotos into this construct from outside it are diagnosed, and`。
- **L644 EN**: Comment explains nearby logic, intent, or metadata: `are fatal unless the construct is a DO, IF, or SELECT CASE.`.
  **L644 CN**: 注释说明附近代码的逻辑、意图或元数据：`are fatal unless the construct is a DO, IF, or SELECT CASE.`。
- **L645 EN**: Continues logic associated with callable symbol `PushScope`.
  **L645 CN**: 继续与可调用符号 `PushScope` 相关的逻辑。
- **L646 EN**: Continues the surrounding expression or declaration: `std::is_same_v<A, parser::IfConstruct> ||`.
  **L646 CN**: 继续构造周围的表达式或声明：`std::is_same_v<A, parser::IfConstruct> ||`。
- **L647 EN**: Executes a standalone statement or declaration: `std::is_same_v<A, parser::CaseConstruct>));`.
  **L647 CN**: 执行一条独立语句或声明：`std::is_same_v<A, parser::CaseConstruct>));`。
- **L648 EN**: Returns from the current function with `true`.
  **L648 CN**: 以 `true` 从当前函数返回。

### Lines 649-672

````cpp
  }
  bool PushConstructName(const parser::BlockConstruct &blockConstruct) {
    const auto &optionalName{
        std::get<parser::Statement<parser::BlockStmt>>(blockConstruct.t)
            .statement.v};
    if (optionalName) {
      constructNames_.emplace_back(optionalName->ToString());
    }
    PushScope(true);
    return true;
  }
  template <typename A> void PopConstructNameIfPresent(const A &a) {
    const auto &optionalName{std::get<0>(std::get<0>(a.t).statement.t)};
    if (optionalName) {
      constructNames_.pop_back();
    }
  }
  void PopConstructNameIfPresent(const parser::BlockConstruct &blockConstruct) {
    const auto &optionalName{
        std::get<parser::Statement<parser::BlockStmt>>(blockConstruct.t)
            .statement.v};
    if (optionalName) {
      constructNames_.pop_back();
    }
````
- **L649 EN**: Closes the current lexical scope or compound statement.
  **L649 CN**: 结束当前词法作用域或复合语句块。
- **L650 EN**: Starts a function, method, lambda, or structured scope: `bool PushConstructName(const parser::BlockConstruct &blockConstruct) {`.
  **L650 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool PushConstructName(const parser::BlockConstruct &blockConstruct) {`。
- **L651 EN**: Continues the surrounding expression or declaration: `const auto &optionalName{`.
  **L651 CN**: 继续构造周围的表达式或声明：`const auto &optionalName{`。
- **L652 EN**: Continues logic associated with callable symbol `BlockStmt>>`.
  **L652 CN**: 继续与可调用符号 `BlockStmt>>` 相关的逻辑。
- **L653 EN**: Executes a standalone statement or declaration: `.statement.v};`.
  **L653 CN**: 执行一条独立语句或声明：`.statement.v};`。
- **L654 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L654 CN**: 开始 `if` 控制流语句并计算其条件。
- **L655 EN**: Executes a call or declaration centered on `constructNames_.emplace_back`.
  **L655 CN**: 执行以 `constructNames_.emplace_back` 为核心的调用或声明。
- **L656 EN**: Closes the current lexical scope or compound statement.
  **L656 CN**: 结束当前词法作用域或复合语句块。
- **L657 EN**: Executes a call or declaration centered on `PushScope`.
  **L657 CN**: 执行以 `PushScope` 为核心的调用或声明。
- **L658 EN**: Returns from the current function with `true`.
  **L658 CN**: 以 `true` 从当前函数返回。
- **L659 EN**: Closes the current lexical scope or compound statement.
  **L659 CN**: 结束当前词法作用域或复合语句块。
- **L660 EN**: Introduces template parameters or specialization context: `template <typename A> void PopConstructNameIfPresent(const A &a) {`.
  **L660 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A> void PopConstructNameIfPresent(const A &a) {`。
- **L661 EN**: Executes a call or declaration centered on `&optionalName{std::get<0>`.
  **L661 CN**: 执行以 `&optionalName{std::get<0>` 为核心的调用或声明。
- **L662 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L662 CN**: 开始 `if` 控制流语句并计算其条件。
- **L663 EN**: Executes a call or declaration centered on `constructNames_.pop_back`.
  **L663 CN**: 执行以 `constructNames_.pop_back` 为核心的调用或声明。
- **L664 EN**: Closes the current lexical scope or compound statement.
  **L664 CN**: 结束当前词法作用域或复合语句块。
- **L665 EN**: Closes the current lexical scope or compound statement.
  **L665 CN**: 结束当前词法作用域或复合语句块。
- **L666 EN**: Starts a function, method, lambda, or structured scope: `void PopConstructNameIfPresent(const parser::BlockConstruct &blockConstruct) {`.
  **L666 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void PopConstructNameIfPresent(const parser::BlockConstruct &blockConstruct) {`。
- **L667 EN**: Continues the surrounding expression or declaration: `const auto &optionalName{`.
  **L667 CN**: 继续构造周围的表达式或声明：`const auto &optionalName{`。
- **L668 EN**: Continues logic associated with callable symbol `BlockStmt>>`.
  **L668 CN**: 继续与可调用符号 `BlockStmt>>` 相关的逻辑。
- **L669 EN**: Executes a standalone statement or declaration: `.statement.v};`.
  **L669 CN**: 执行一条独立语句或声明：`.statement.v};`。
- **L670 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L670 CN**: 开始 `if` 控制流语句并计算其条件。
- **L671 EN**: Executes a call or declaration centered on `constructNames_.pop_back`.
  **L671 CN**: 执行以 `constructNames_.pop_back` 为核心的调用或声明。
- **L672 EN**: Closes the current lexical scope or compound statement.
  **L672 CN**: 结束当前词法作用域或复合语句块。

### Lines 673-696

````cpp
  }

  template <typename A> void PopConstructName(const A &a) {
    CheckName(a);
    PopScope();
    PopConstructNameIfPresent(a);
  }

  template <typename FIRST, typename CASEBLOCK, typename CASE,
      typename CONSTRUCT>
  void CheckSelectNames(const char *tag, const CONSTRUCT &construct) {
    CheckEndName<FIRST, parser::EndSelectStmt>(tag, construct);
    for (const auto &inner : std::get<std::list<CASEBLOCK>>(construct.t)) {
      CheckOptionalName<FIRST>(
          tag, construct, std::get<parser::Statement<CASE>>(inner.t));
    }
  }

  // C1144
  void PopConstructName(const parser::CaseConstruct &caseConstruct) {
    CheckSelectNames<parser::SelectCaseStmt, parser::CaseConstruct::Case,
        parser::CaseStmt>("SELECT CASE", caseConstruct);
    PopScope();
    PopConstructNameIfPresent(caseConstruct);
````
- **L673 EN**: Closes the current lexical scope or compound statement.
  **L673 CN**: 结束当前词法作用域或复合语句块。
- **L674 EN**: Blank line separating nearby declarations or logic blocks.
  **L674 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L675 EN**: Introduces template parameters or specialization context: `template <typename A> void PopConstructName(const A &a) {`.
  **L675 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A> void PopConstructName(const A &a) {`。
- **L676 EN**: Executes a call or declaration centered on `CheckName`.
  **L676 CN**: 执行以 `CheckName` 为核心的调用或声明。
- **L677 EN**: Executes a call or declaration centered on `PopScope`.
  **L677 CN**: 执行以 `PopScope` 为核心的调用或声明。
- **L678 EN**: Executes a call or declaration centered on `PopConstructNameIfPresent`.
  **L678 CN**: 执行以 `PopConstructNameIfPresent` 为核心的调用或声明。
- **L679 EN**: Closes the current lexical scope or compound statement.
  **L679 CN**: 结束当前词法作用域或复合语句块。
- **L680 EN**: Blank line separating nearby declarations or logic blocks.
  **L680 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L681 EN**: Introduces template parameters or specialization context: `template <typename FIRST, typename CASEBLOCK, typename CASE,`.
  **L681 CN**: 为后续声明引入模板参数或特化上下文：`template <typename FIRST, typename CASEBLOCK, typename CASE,`。
- **L682 EN**: Continues the surrounding expression or declaration: `typename CONSTRUCT>`.
  **L682 CN**: 继续构造周围的表达式或声明：`typename CONSTRUCT>`。
- **L683 EN**: Starts a function, method, lambda, or structured scope: `void CheckSelectNames(const char *tag, const CONSTRUCT &construct) {`.
  **L683 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckSelectNames(const char *tag, const CONSTRUCT &construct) {`。
- **L684 EN**: Executes a call or declaration centered on `parser::EndSelectStmt>`.
  **L684 CN**: 执行以 `parser::EndSelectStmt>` 为核心的调用或声明。
- **L685 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L685 CN**: 开始 `for` 控制流语句并计算其条件。
- **L686 EN**: Continues logic associated with callable symbol `CheckOptionalName<FIRST>`.
  **L686 CN**: 继续与可调用符号 `CheckOptionalName<FIRST>` 相关的逻辑。
- **L687 EN**: Executes a call or declaration centered on `std::get<parser::Statement<CASE>>`.
  **L687 CN**: 执行以 `std::get<parser::Statement<CASE>>` 为核心的调用或声明。
- **L688 EN**: Closes the current lexical scope or compound statement.
  **L688 CN**: 结束当前词法作用域或复合语句块。
- **L689 EN**: Closes the current lexical scope or compound statement.
  **L689 CN**: 结束当前词法作用域或复合语句块。
- **L690 EN**: Blank line separating nearby declarations or logic blocks.
  **L690 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L691 EN**: Comment explains nearby logic, intent, or metadata: `C1144`.
  **L691 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1144`。
- **L692 EN**: Starts a function, method, lambda, or structured scope: `void PopConstructName(const parser::CaseConstruct &caseConstruct) {`.
  **L692 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void PopConstructName(const parser::CaseConstruct &caseConstruct) {`。
- **L693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckSelectNames<parser::SelectCaseStmt, parser::CaseConstruct::Case,`.
  **L693 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckSelectNames<parser::SelectCaseStmt, parser::CaseConstruct::Case,`。
- **L694 EN**: Executes a call or declaration centered on `parser::CaseStmt>`.
  **L694 CN**: 执行以 `parser::CaseStmt>` 为核心的调用或声明。
- **L695 EN**: Executes a call or declaration centered on `PopScope`.
  **L695 CN**: 执行以 `PopScope` 为核心的调用或声明。
- **L696 EN**: Executes a call or declaration centered on `PopConstructNameIfPresent`.
  **L696 CN**: 执行以 `PopConstructNameIfPresent` 为核心的调用或声明。

### Lines 697-720

````cpp
  }

  // C1154, C1156
  void PopConstructName(
      const parser::SelectRankConstruct &selectRankConstruct) {
    CheckSelectNames<parser::SelectRankStmt,
        parser::SelectRankConstruct::RankCase, parser::SelectRankCaseStmt>(
        "SELECT RANK", selectRankConstruct);
    PopScope();
    PopConstructNameIfPresent(selectRankConstruct);
  }

  // C1165
  void PopConstructName(
      const parser::SelectTypeConstruct &selectTypeConstruct) {
    CheckSelectNames<parser::SelectTypeStmt,
        parser::SelectTypeConstruct::TypeCase, parser::TypeGuardStmt>(
        "SELECT TYPE", selectTypeConstruct);
    PopScope();
    PopConstructNameIfPresent(selectTypeConstruct);
  }

  // Checks for missing or mismatching names on various constructs (e.g., BLOCK)
  // and their END statements.  Both names must be present if either one is.
````
- **L697 EN**: Closes the current lexical scope or compound statement.
  **L697 CN**: 结束当前词法作用域或复合语句块。
- **L698 EN**: Blank line separating nearby declarations or logic blocks.
  **L698 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L699 EN**: Comment explains nearby logic, intent, or metadata: `C1154, C1156`.
  **L699 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1154, C1156`。
- **L700 EN**: Continues logic associated with callable symbol `PopConstructName`.
  **L700 CN**: 继续与可调用符号 `PopConstructName` 相关的逻辑。
- **L701 EN**: Continues the surrounding expression or declaration: `const parser::SelectRankConstruct &selectRankConstruct) {`.
  **L701 CN**: 继续构造周围的表达式或声明：`const parser::SelectRankConstruct &selectRankConstruct) {`。
- **L702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckSelectNames<parser::SelectRankStmt,`.
  **L702 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckSelectNames<parser::SelectRankStmt,`。
- **L703 EN**: Continues logic associated with callable symbol `SelectRankCaseStmt>`.
  **L703 CN**: 继续与可调用符号 `SelectRankCaseStmt>` 相关的逻辑。
- **L704 EN**: Executes a standalone statement or declaration: `"SELECT RANK", selectRankConstruct);`.
  **L704 CN**: 执行一条独立语句或声明：`"SELECT RANK", selectRankConstruct);`。
- **L705 EN**: Executes a call or declaration centered on `PopScope`.
  **L705 CN**: 执行以 `PopScope` 为核心的调用或声明。
- **L706 EN**: Executes a call or declaration centered on `PopConstructNameIfPresent`.
  **L706 CN**: 执行以 `PopConstructNameIfPresent` 为核心的调用或声明。
- **L707 EN**: Closes the current lexical scope or compound statement.
  **L707 CN**: 结束当前词法作用域或复合语句块。
- **L708 EN**: Blank line separating nearby declarations or logic blocks.
  **L708 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L709 EN**: Comment explains nearby logic, intent, or metadata: `C1165`.
  **L709 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1165`。
- **L710 EN**: Continues logic associated with callable symbol `PopConstructName`.
  **L710 CN**: 继续与可调用符号 `PopConstructName` 相关的逻辑。
- **L711 EN**: Continues the surrounding expression or declaration: `const parser::SelectTypeConstruct &selectTypeConstruct) {`.
  **L711 CN**: 继续构造周围的表达式或声明：`const parser::SelectTypeConstruct &selectTypeConstruct) {`。
- **L712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckSelectNames<parser::SelectTypeStmt,`.
  **L712 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckSelectNames<parser::SelectTypeStmt,`。
- **L713 EN**: Continues logic associated with callable symbol `TypeGuardStmt>`.
  **L713 CN**: 继续与可调用符号 `TypeGuardStmt>` 相关的逻辑。
- **L714 EN**: Executes a standalone statement or declaration: `"SELECT TYPE", selectTypeConstruct);`.
  **L714 CN**: 执行一条独立语句或声明：`"SELECT TYPE", selectTypeConstruct);`。
- **L715 EN**: Executes a call or declaration centered on `PopScope`.
  **L715 CN**: 执行以 `PopScope` 为核心的调用或声明。
- **L716 EN**: Executes a call or declaration centered on `PopConstructNameIfPresent`.
  **L716 CN**: 执行以 `PopConstructNameIfPresent` 为核心的调用或声明。
- **L717 EN**: Closes the current lexical scope or compound statement.
  **L717 CN**: 结束当前词法作用域或复合语句块。
- **L718 EN**: Blank line separating nearby declarations or logic blocks.
  **L718 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L719 EN**: Comment explains nearby logic, intent, or metadata: `Checks for missing or mismatching names on various constructs (e.g., BLOCK)`.
  **L719 CN**: 注释说明附近代码的逻辑、意图或元数据：`Checks for missing or mismatching names on various constructs (e.g., BLOCK)`。
- **L720 EN**: Comment explains nearby logic, intent, or metadata: `and their END statements.  Both names must be present if either one is.`.
  **L720 CN**: 注释说明附近代码的逻辑、意图或元数据：`and their END statements.  Both names must be present if either one is.`。

### Lines 721-744

````cpp
  template <typename FIRST, typename END, typename CONSTRUCT>
  void CheckEndName(const char *constructTag, const CONSTRUCT &a) {
    const auto &constructStmt{std::get<parser::Statement<FIRST>>(a.t)};
    const auto &endStmt{std::get<parser::Statement<END>>(a.t)};
    const parser::CharBlock *endName{GetStmtName(endStmt)};
    if (const parser::CharBlock * constructName{GetStmtName(constructStmt)}) {
      if (endName) {
        if (*constructName != *endName) {
          context_
              .Say(*endName, "%s construct name mismatch"_err_en_US,
                  constructTag)
              .Attach(*constructName, "should be"_en_US);
        }
      } else {
        context_
            .Say(endStmt.source,
                "%s construct name required but missing"_err_en_US,
                constructTag)
            .Attach(*constructName, "should be"_en_US);
      }
    } else if (endName) {
      context_
          .Say(*endName, "%s construct name unexpected"_err_en_US, constructTag)
          .Attach(
````
- **L721 EN**: Introduces template parameters or specialization context: `template <typename FIRST, typename END, typename CONSTRUCT>`.
  **L721 CN**: 为后续声明引入模板参数或特化上下文：`template <typename FIRST, typename END, typename CONSTRUCT>`。
- **L722 EN**: Starts a function, method, lambda, or structured scope: `void CheckEndName(const char *constructTag, const CONSTRUCT &a) {`.
  **L722 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckEndName(const char *constructTag, const CONSTRUCT &a) {`。
- **L723 EN**: Executes a call or declaration centered on `&constructStmt{std::get<parser::Statement<FIRST>>`.
  **L723 CN**: 执行以 `&constructStmt{std::get<parser::Statement<FIRST>>` 为核心的调用或声明。
- **L724 EN**: Executes a call or declaration centered on `&endStmt{std::get<parser::Statement<END>>`.
  **L724 CN**: 执行以 `&endStmt{std::get<parser::Statement<END>>` 为核心的调用或声明。
- **L725 EN**: Executes a call or declaration centered on `*endName{GetStmtName`.
  **L725 CN**: 执行以 `*endName{GetStmtName` 为核心的调用或声明。
- **L726 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L726 CN**: 开始 `if` 控制流语句并计算其条件。
- **L727 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L727 CN**: 开始 `if` 控制流语句并计算其条件。
- **L728 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L728 CN**: 开始 `if` 控制流语句并计算其条件。
- **L729 EN**: Continues the surrounding expression or declaration: `context_`.
  **L729 CN**: 继续构造周围的表达式或声明：`context_`。
- **L730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(*endName, "%s construct name mismatch"_err_en_US,`.
  **L730 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(*endName, "%s construct name mismatch"_err_en_US,`。
- **L731 EN**: Continues the surrounding expression or declaration: `constructTag)`.
  **L731 CN**: 继续构造周围的表达式或声明：`constructTag)`。
- **L732 EN**: Executes a call or declaration centered on `.Attach`.
  **L732 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L733 EN**: Closes the current lexical scope or compound statement.
  **L733 CN**: 结束当前词法作用域或复合语句块。
- **L734 EN**: Transitions from the previous branch into the alternative path.
  **L734 CN**: 从前一个分支过渡到备选路径。
- **L735 EN**: Continues the surrounding expression or declaration: `context_`.
  **L735 CN**: 继续构造周围的表达式或声明：`context_`。
- **L736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(endStmt.source,`.
  **L736 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(endStmt.source,`。
- **L737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s construct name required but missing"_err_en_US,`.
  **L737 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s construct name required but missing"_err_en_US,`。
- **L738 EN**: Continues the surrounding expression or declaration: `constructTag)`.
  **L738 CN**: 继续构造周围的表达式或声明：`constructTag)`。
- **L739 EN**: Executes a call or declaration centered on `.Attach`.
  **L739 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L740 EN**: Closes the current lexical scope or compound statement.
  **L740 CN**: 结束当前词法作用域或复合语句块。
- **L741 EN**: Transitions from the previous branch into an `else if` condition.
  **L741 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L742 EN**: Continues the surrounding expression or declaration: `context_`.
  **L742 CN**: 继续构造周围的表达式或声明：`context_`。
- **L743 EN**: Continues logic associated with callable symbol `Say`.
  **L743 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L744 EN**: Continues logic associated with callable symbol `Attach`.
  **L744 CN**: 继续与可调用符号 `Attach` 相关的逻辑。

### Lines 745-768

````cpp
              constructStmt.source, "unnamed %s statement"_en_US, constructTag);
    }
  }

  // C1106
  void CheckName(const parser::AssociateConstruct &associateConstruct) {
    CheckEndName<parser::AssociateStmt, parser::EndAssociateStmt>(
        "ASSOCIATE", associateConstruct);
  }
  // C1117
  void CheckName(const parser::CriticalConstruct &criticalConstruct) {
    CheckEndName<parser::CriticalStmt, parser::EndCriticalStmt>(
        "CRITICAL", criticalConstruct);
  }
  // C1131
  void CheckName(const parser::DoConstruct &doConstruct) {
    CheckEndName<parser::NonLabelDoStmt, parser::EndDoStmt>("DO", doConstruct);
    if (auto label{std::get<std::optional<parser::Label>>(
            std::get<parser::Statement<parser::NonLabelDoStmt>>(doConstruct.t)
                .statement.t)}) {
      const auto &endDoStmt{
          std::get<parser::Statement<parser::EndDoStmt>>(doConstruct.t)};
      if (!endDoStmt.label || *endDoStmt.label != *label) {
        context_
````
- **L745 EN**: Executes a standalone statement or declaration: `constructStmt.source, "unnamed %s statement"_en_US, constructTag);`.
  **L745 CN**: 执行一条独立语句或声明：`constructStmt.source, "unnamed %s statement"_en_US, constructTag);`。
- **L746 EN**: Closes the current lexical scope or compound statement.
  **L746 CN**: 结束当前词法作用域或复合语句块。
- **L747 EN**: Closes the current lexical scope or compound statement.
  **L747 CN**: 结束当前词法作用域或复合语句块。
- **L748 EN**: Blank line separating nearby declarations or logic blocks.
  **L748 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L749 EN**: Comment explains nearby logic, intent, or metadata: `C1106`.
  **L749 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1106`。
- **L750 EN**: Starts a function, method, lambda, or structured scope: `void CheckName(const parser::AssociateConstruct &associateConstruct) {`.
  **L750 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckName(const parser::AssociateConstruct &associateConstruct) {`。
- **L751 EN**: Continues logic associated with callable symbol `EndAssociateStmt>`.
  **L751 CN**: 继续与可调用符号 `EndAssociateStmt>` 相关的逻辑。
- **L752 EN**: Executes a standalone statement or declaration: `"ASSOCIATE", associateConstruct);`.
  **L752 CN**: 执行一条独立语句或声明：`"ASSOCIATE", associateConstruct);`。
- **L753 EN**: Closes the current lexical scope or compound statement.
  **L753 CN**: 结束当前词法作用域或复合语句块。
- **L754 EN**: Comment explains nearby logic, intent, or metadata: `C1117`.
  **L754 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1117`。
- **L755 EN**: Starts a function, method, lambda, or structured scope: `void CheckName(const parser::CriticalConstruct &criticalConstruct) {`.
  **L755 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckName(const parser::CriticalConstruct &criticalConstruct) {`。
- **L756 EN**: Continues logic associated with callable symbol `EndCriticalStmt>`.
  **L756 CN**: 继续与可调用符号 `EndCriticalStmt>` 相关的逻辑。
- **L757 EN**: Executes a standalone statement or declaration: `"CRITICAL", criticalConstruct);`.
  **L757 CN**: 执行一条独立语句或声明：`"CRITICAL", criticalConstruct);`。
- **L758 EN**: Closes the current lexical scope or compound statement.
  **L758 CN**: 结束当前词法作用域或复合语句块。
- **L759 EN**: Comment explains nearby logic, intent, or metadata: `C1131`.
  **L759 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1131`。
- **L760 EN**: Starts a function, method, lambda, or structured scope: `void CheckName(const parser::DoConstruct &doConstruct) {`.
  **L760 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckName(const parser::DoConstruct &doConstruct) {`。
- **L761 EN**: Executes a call or declaration centered on `parser::EndDoStmt>`.
  **L761 CN**: 执行以 `parser::EndDoStmt>` 为核心的调用或声明。
- **L762 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L762 CN**: 开始 `if` 控制流语句并计算其条件。
- **L763 EN**: Continues logic associated with callable symbol `NonLabelDoStmt>>`.
  **L763 CN**: 继续与可调用符号 `NonLabelDoStmt>>` 相关的逻辑。
- **L764 EN**: Continues the surrounding expression or declaration: `.statement.t)}) {`.
  **L764 CN**: 继续构造周围的表达式或声明：`.statement.t)}) {`。
- **L765 EN**: Continues the surrounding expression or declaration: `const auto &endDoStmt{`.
  **L765 CN**: 继续构造周围的表达式或声明：`const auto &endDoStmt{`。
- **L766 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::EndDoStmt>>`.
  **L766 CN**: 执行以 `std::get<parser::Statement<parser::EndDoStmt>>` 为核心的调用或声明。
- **L767 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L767 CN**: 开始 `if` 控制流语句并计算其条件。
- **L768 EN**: Continues the surrounding expression or declaration: `context_`.
  **L768 CN**: 继续构造周围的表达式或声明：`context_`。

### Lines 769-792

````cpp
            .Say(endDoStmt.source,
                "END DO statement must have the label '%d' matching its DO statement"_err_en_US,
                *label)
            .Attach(std::get<parser::Statement<parser::NonLabelDoStmt>>(
                        doConstruct.t)
                        .source,
                "corresponding DO statement"_en_US);
      }
    }
  }
  // C1035
  void CheckName(const parser::ForallConstruct &forallConstruct) {
    CheckEndName<parser::ForallConstructStmt, parser::EndForallStmt>(
        "FORALL", forallConstruct);
  }

  // C1109
  void CheckName(const parser::BlockConstruct &blockConstruct) {
    CheckEndName<parser::BlockStmt, parser::EndBlockStmt>(
        "BLOCK", blockConstruct);
  }
  // C1112
  void CheckName(const parser::ChangeTeamConstruct &changeTeamConstruct) {
    CheckEndName<parser::ChangeTeamStmt, parser::EndChangeTeamStmt>(
````
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(endDoStmt.source,`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(endDoStmt.source,`。
- **L770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"END DO statement must have the label '%d' matching its DO statement"_err_en_US,`.
  **L770 CN**: 继续一个多行参数列表、初始化器或聚合项：`"END DO statement must have the label '%d' matching its DO statement"_err_en_US,`。
- **L771 EN**: Comment explains nearby logic, intent, or metadata: `label)`.
  **L771 CN**: 注释说明附近代码的逻辑、意图或元数据：`label)`。
- **L772 EN**: Continues logic associated with callable symbol `Attach`.
  **L772 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L773 EN**: Continues the surrounding expression or declaration: `doConstruct.t)`.
  **L773 CN**: 继续构造周围的表达式或声明：`doConstruct.t)`。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.source,`.
  **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`.source,`。
- **L775 EN**: Executes a standalone statement or declaration: `"corresponding DO statement"_en_US);`.
  **L775 CN**: 执行一条独立语句或声明：`"corresponding DO statement"_en_US);`。
- **L776 EN**: Closes the current lexical scope or compound statement.
  **L776 CN**: 结束当前词法作用域或复合语句块。
- **L777 EN**: Closes the current lexical scope or compound statement.
  **L777 CN**: 结束当前词法作用域或复合语句块。
- **L778 EN**: Closes the current lexical scope or compound statement.
  **L778 CN**: 结束当前词法作用域或复合语句块。
- **L779 EN**: Comment explains nearby logic, intent, or metadata: `C1035`.
  **L779 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1035`。
- **L780 EN**: Starts a function, method, lambda, or structured scope: `void CheckName(const parser::ForallConstruct &forallConstruct) {`.
  **L780 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckName(const parser::ForallConstruct &forallConstruct) {`。
- **L781 EN**: Continues logic associated with callable symbol `EndForallStmt>`.
  **L781 CN**: 继续与可调用符号 `EndForallStmt>` 相关的逻辑。
- **L782 EN**: Executes a standalone statement or declaration: `"FORALL", forallConstruct);`.
  **L782 CN**: 执行一条独立语句或声明：`"FORALL", forallConstruct);`。
- **L783 EN**: Closes the current lexical scope or compound statement.
  **L783 CN**: 结束当前词法作用域或复合语句块。
- **L784 EN**: Blank line separating nearby declarations or logic blocks.
  **L784 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L785 EN**: Comment explains nearby logic, intent, or metadata: `C1109`.
  **L785 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1109`。
- **L786 EN**: Starts a function, method, lambda, or structured scope: `void CheckName(const parser::BlockConstruct &blockConstruct) {`.
  **L786 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckName(const parser::BlockConstruct &blockConstruct) {`。
- **L787 EN**: Continues logic associated with callable symbol `EndBlockStmt>`.
  **L787 CN**: 继续与可调用符号 `EndBlockStmt>` 相关的逻辑。
- **L788 EN**: Executes a standalone statement or declaration: `"BLOCK", blockConstruct);`.
  **L788 CN**: 执行一条独立语句或声明：`"BLOCK", blockConstruct);`。
- **L789 EN**: Closes the current lexical scope or compound statement.
  **L789 CN**: 结束当前词法作用域或复合语句块。
- **L790 EN**: Comment explains nearby logic, intent, or metadata: `C1112`.
  **L790 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1112`。
- **L791 EN**: Starts a function, method, lambda, or structured scope: `void CheckName(const parser::ChangeTeamConstruct &changeTeamConstruct) {`.
  **L791 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckName(const parser::ChangeTeamConstruct &changeTeamConstruct) {`。
- **L792 EN**: Continues logic associated with callable symbol `EndChangeTeamStmt>`.
  **L792 CN**: 继续与可调用符号 `EndChangeTeamStmt>` 相关的逻辑。

### Lines 793-816

````cpp
        "CHANGE TEAM", changeTeamConstruct);
  }

  // C1142
  void CheckName(const parser::IfConstruct &ifConstruct) {
    CheckEndName<parser::IfThenStmt, parser::EndIfStmt>("IF", ifConstruct);
    for (const auto &elseIfBlock :
        std::get<std::list<parser::IfConstruct::ElseIfBlock>>(ifConstruct.t)) {
      CheckOptionalName<parser::IfThenStmt>("IF construct", ifConstruct,
          std::get<parser::Statement<parser::ElseIfStmt>>(elseIfBlock.t));
    }
    if (const auto &elseBlock{
            std::get<std::optional<parser::IfConstruct::ElseBlock>>(
                ifConstruct.t)}) {
      CheckOptionalName<parser::IfThenStmt>("IF construct", ifConstruct,
          std::get<parser::Statement<parser::ElseStmt>>(elseBlock->t));
    }
  }

  // C1033
  void CheckName(const parser::WhereConstruct &whereConstruct) {
    CheckEndName<parser::WhereConstructStmt, parser::EndWhereStmt>(
        "WHERE", whereConstruct);
    for (const auto &maskedElsewhere :
````
- **L793 EN**: Executes a standalone statement or declaration: `"CHANGE TEAM", changeTeamConstruct);`.
  **L793 CN**: 执行一条独立语句或声明：`"CHANGE TEAM", changeTeamConstruct);`。
- **L794 EN**: Closes the current lexical scope or compound statement.
  **L794 CN**: 结束当前词法作用域或复合语句块。
- **L795 EN**: Blank line separating nearby declarations or logic blocks.
  **L795 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L796 EN**: Comment explains nearby logic, intent, or metadata: `C1142`.
  **L796 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1142`。
- **L797 EN**: Starts a function, method, lambda, or structured scope: `void CheckName(const parser::IfConstruct &ifConstruct) {`.
  **L797 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckName(const parser::IfConstruct &ifConstruct) {`。
- **L798 EN**: Executes a call or declaration centered on `parser::EndIfStmt>`.
  **L798 CN**: 执行以 `parser::EndIfStmt>` 为核心的调用或声明。
- **L799 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L799 CN**: 开始 `for` 控制流语句并计算其条件。
- **L800 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::IfConstruct::ElseIfBlock>>(ifConstruct.t)) {`.
  **L800 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::IfConstruct::ElseIfBlock>>(ifConstruct.t)) {`。
- **L801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::IfThenStmt>("IF construct", ifConstruct,`.
  **L801 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::IfThenStmt>("IF construct", ifConstruct,`。
- **L802 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::ElseIfStmt>>`.
  **L802 CN**: 执行以 `std::get<parser::Statement<parser::ElseIfStmt>>` 为核心的调用或声明。
- **L803 EN**: Closes the current lexical scope or compound statement.
  **L803 CN**: 结束当前词法作用域或复合语句块。
- **L804 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L804 CN**: 开始 `if` 控制流语句并计算其条件。
- **L805 EN**: Continues logic associated with callable symbol `ElseBlock>>`.
  **L805 CN**: 继续与可调用符号 `ElseBlock>>` 相关的逻辑。
- **L806 EN**: Continues the surrounding expression or declaration: `ifConstruct.t)}) {`.
  **L806 CN**: 继续构造周围的表达式或声明：`ifConstruct.t)}) {`。
- **L807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::IfThenStmt>("IF construct", ifConstruct,`.
  **L807 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::IfThenStmt>("IF construct", ifConstruct,`。
- **L808 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::ElseStmt>>`.
  **L808 CN**: 执行以 `std::get<parser::Statement<parser::ElseStmt>>` 为核心的调用或声明。
- **L809 EN**: Closes the current lexical scope or compound statement.
  **L809 CN**: 结束当前词法作用域或复合语句块。
- **L810 EN**: Closes the current lexical scope or compound statement.
  **L810 CN**: 结束当前词法作用域或复合语句块。
- **L811 EN**: Blank line separating nearby declarations or logic blocks.
  **L811 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L812 EN**: Comment explains nearby logic, intent, or metadata: `C1033`.
  **L812 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1033`。
- **L813 EN**: Starts a function, method, lambda, or structured scope: `void CheckName(const parser::WhereConstruct &whereConstruct) {`.
  **L813 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckName(const parser::WhereConstruct &whereConstruct) {`。
- **L814 EN**: Continues logic associated with callable symbol `EndWhereStmt>`.
  **L814 CN**: 继续与可调用符号 `EndWhereStmt>` 相关的逻辑。
- **L815 EN**: Executes a standalone statement or declaration: `"WHERE", whereConstruct);`.
  **L815 CN**: 执行一条独立语句或声明：`"WHERE", whereConstruct);`。
- **L816 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L816 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 817-840

````cpp
        std::get<std::list<parser::WhereConstruct::MaskedElsewhere>>(
            whereConstruct.t)) {
      CheckOptionalName<parser::WhereConstructStmt>("WHERE construct",
          whereConstruct,
          std::get<parser::Statement<parser::MaskedElsewhereStmt>>(
              maskedElsewhere.t));
    }
    if (const auto &elsewhere{
            std::get<std::optional<parser::WhereConstruct::Elsewhere>>(
                whereConstruct.t)}) {
      CheckOptionalName<parser::WhereConstructStmt>("WHERE construct",
          whereConstruct,
          std::get<parser::Statement<parser::ElsewhereStmt>>(elsewhere->t));
    }
  }

  // C1134, C1166
  void CheckLabelContext(
      const char *const stmtString, const parser::CharBlock &constructName) {
    const auto iter{std::find(constructNames_.crbegin(),
        constructNames_.crend(), constructName.ToString())};
    if (iter == constructNames_.crend()) {
      context_.Say(constructName, "%s construct-name is not in scope"_err_en_US,
          stmtString);
````
- **L817 EN**: Continues logic associated with callable symbol `MaskedElsewhere>>`.
  **L817 CN**: 继续与可调用符号 `MaskedElsewhere>>` 相关的逻辑。
- **L818 EN**: Continues the surrounding expression or declaration: `whereConstruct.t)) {`.
  **L818 CN**: 继续构造周围的表达式或声明：`whereConstruct.t)) {`。
- **L819 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::WhereConstructStmt>("WHERE construct",`.
  **L819 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::WhereConstructStmt>("WHERE construct",`。
- **L820 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `whereConstruct,`.
  **L820 CN**: 继续一个多行参数列表、初始化器或聚合项：`whereConstruct,`。
- **L821 EN**: Continues logic associated with callable symbol `MaskedElsewhereStmt>>`.
  **L821 CN**: 继续与可调用符号 `MaskedElsewhereStmt>>` 相关的逻辑。
- **L822 EN**: Executes a standalone statement or declaration: `maskedElsewhere.t));`.
  **L822 CN**: 执行一条独立语句或声明：`maskedElsewhere.t));`。
- **L823 EN**: Closes the current lexical scope or compound statement.
  **L823 CN**: 结束当前词法作用域或复合语句块。
- **L824 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L824 CN**: 开始 `if` 控制流语句并计算其条件。
- **L825 EN**: Continues logic associated with callable symbol `Elsewhere>>`.
  **L825 CN**: 继续与可调用符号 `Elsewhere>>` 相关的逻辑。
- **L826 EN**: Continues the surrounding expression or declaration: `whereConstruct.t)}) {`.
  **L826 CN**: 继续构造周围的表达式或声明：`whereConstruct.t)}) {`。
- **L827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOptionalName<parser::WhereConstructStmt>("WHERE construct",`.
  **L827 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOptionalName<parser::WhereConstructStmt>("WHERE construct",`。
- **L828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `whereConstruct,`.
  **L828 CN**: 继续一个多行参数列表、初始化器或聚合项：`whereConstruct,`。
- **L829 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::ElsewhereStmt>>`.
  **L829 CN**: 执行以 `std::get<parser::Statement<parser::ElsewhereStmt>>` 为核心的调用或声明。
- **L830 EN**: Closes the current lexical scope or compound statement.
  **L830 CN**: 结束当前词法作用域或复合语句块。
- **L831 EN**: Closes the current lexical scope or compound statement.
  **L831 CN**: 结束当前词法作用域或复合语句块。
- **L832 EN**: Blank line separating nearby declarations or logic blocks.
  **L832 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L833 EN**: Comment explains nearby logic, intent, or metadata: `C1134, C1166`.
  **L833 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1134, C1166`。
- **L834 EN**: Continues logic associated with callable symbol `CheckLabelContext`.
  **L834 CN**: 继续与可调用符号 `CheckLabelContext` 相关的逻辑。
- **L835 EN**: Continues the surrounding expression or declaration: `const char *const stmtString, const parser::CharBlock &constructName) {`.
  **L835 CN**: 继续构造周围的表达式或声明：`const char *const stmtString, const parser::CharBlock &constructName) {`。
- **L836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const auto iter{std::find(constructNames_.crbegin(),`.
  **L836 CN**: 继续一个多行参数列表、初始化器或聚合项：`const auto iter{std::find(constructNames_.crbegin(),`。
- **L837 EN**: Executes a call or declaration centered on `constructNames_.crend`.
  **L837 CN**: 执行以 `constructNames_.crend` 为核心的调用或声明。
- **L838 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L838 CN**: 开始 `if` 控制流语句并计算其条件。
- **L839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(constructName, "%s construct-name is not in scope"_err_en_US,`.
  **L839 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(constructName, "%s construct-name is not in scope"_err_en_US,`。
- **L840 EN**: Executes a standalone statement or declaration: `stmtString);`.
  **L840 CN**: 执行一条独立语句或声明：`stmtString);`。

### Lines 841-864

````cpp
    }
  }

  // 6.2.5, paragraph 2
  void CheckLabelInRange(parser::Label label) {
    if (label < 1 || label > 99999) {
      context_.Say(currentPosition_, "Label '%u' is out of range"_err_en_US,
          SayLabel(label));
    }
  }

  // 6.2.5., paragraph 2
  void AddTargetLabelDefinition(parser::Label label,
      LabeledStmtClassificationSet labeledStmtClassificationSet,
      ProxyForScope scope, bool isExecutableConstructEndStmt) {
    CheckLabelInRange(label);
    TargetStmtMap &targetStmtMap{disposableMaps_.empty()
            ? programUnits_.back().targetStmts
            : disposableMaps_.back()};
    const auto pair{targetStmtMap.emplace(label,
        LabeledStatementInfoTuplePOD{scope, currentPosition_,
            labeledStmtClassificationSet, isExecutableConstructEndStmt})};
    if (!pair.second) {
      context_.Say(currentPosition_, "Label '%u' is not distinct"_err_en_US,
````
- **L841 EN**: Closes the current lexical scope or compound statement.
  **L841 CN**: 结束当前词法作用域或复合语句块。
- **L842 EN**: Closes the current lexical scope or compound statement.
  **L842 CN**: 结束当前词法作用域或复合语句块。
- **L843 EN**: Blank line separating nearby declarations or logic blocks.
  **L843 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L844 EN**: Comment explains nearby logic, intent, or metadata: `6.2.5, paragraph 2`.
  **L844 CN**: 注释说明附近代码的逻辑、意图或元数据：`6.2.5, paragraph 2`。
- **L845 EN**: Starts a function, method, lambda, or structured scope: `void CheckLabelInRange(parser::Label label) {`.
  **L845 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckLabelInRange(parser::Label label) {`。
- **L846 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L846 CN**: 开始 `if` 控制流语句并计算其条件。
- **L847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(currentPosition_, "Label '%u' is out of range"_err_en_US,`.
  **L847 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(currentPosition_, "Label '%u' is out of range"_err_en_US,`。
- **L848 EN**: Executes a call or declaration centered on `SayLabel`.
  **L848 CN**: 执行以 `SayLabel` 为核心的调用或声明。
- **L849 EN**: Closes the current lexical scope or compound statement.
  **L849 CN**: 结束当前词法作用域或复合语句块。
- **L850 EN**: Closes the current lexical scope or compound statement.
  **L850 CN**: 结束当前词法作用域或复合语句块。
- **L851 EN**: Blank line separating nearby declarations or logic blocks.
  **L851 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L852 EN**: Comment explains nearby logic, intent, or metadata: `6.2.5., paragraph 2`.
  **L852 CN**: 注释说明附近代码的逻辑、意图或元数据：`6.2.5., paragraph 2`。
- **L853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void AddTargetLabelDefinition(parser::Label label,`.
  **L853 CN**: 继续一个多行参数列表、初始化器或聚合项：`void AddTargetLabelDefinition(parser::Label label,`。
- **L854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LabeledStmtClassificationSet labeledStmtClassificationSet,`.
  **L854 CN**: 继续一个多行参数列表、初始化器或聚合项：`LabeledStmtClassificationSet labeledStmtClassificationSet,`。
- **L855 EN**: Continues the surrounding expression or declaration: `ProxyForScope scope, bool isExecutableConstructEndStmt) {`.
  **L855 CN**: 继续构造周围的表达式或声明：`ProxyForScope scope, bool isExecutableConstructEndStmt) {`。
- **L856 EN**: Executes a call or declaration centered on `CheckLabelInRange`.
  **L856 CN**: 执行以 `CheckLabelInRange` 为核心的调用或声明。
- **L857 EN**: Continues logic associated with callable symbol `empty`.
  **L857 CN**: 继续与可调用符号 `empty` 相关的逻辑。
- **L858 EN**: Continues logic associated with callable symbol `back`.
  **L858 CN**: 继续与可调用符号 `back` 相关的逻辑。
- **L859 EN**: Executes a call or declaration centered on `disposableMaps_.back`.
  **L859 CN**: 执行以 `disposableMaps_.back` 为核心的调用或声明。
- **L860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const auto pair{targetStmtMap.emplace(label,`.
  **L860 CN**: 继续一个多行参数列表、初始化器或聚合项：`const auto pair{targetStmtMap.emplace(label,`。
- **L861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LabeledStatementInfoTuplePOD{scope, currentPosition_,`.
  **L861 CN**: 继续一个多行参数列表、初始化器或聚合项：`LabeledStatementInfoTuplePOD{scope, currentPosition_,`。
- **L862 EN**: Executes a standalone statement or declaration: `labeledStmtClassificationSet, isExecutableConstructEndStmt})};`.
  **L862 CN**: 执行一条独立语句或声明：`labeledStmtClassificationSet, isExecutableConstructEndStmt})};`。
- **L863 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L863 CN**: 开始 `if` 控制流语句并计算其条件。
- **L864 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(currentPosition_, "Label '%u' is not distinct"_err_en_US,`.
  **L864 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(currentPosition_, "Label '%u' is not distinct"_err_en_US,`。

### Lines 865-888

````cpp
          SayLabel(label));
    }
  }

  void AddLabelReferenceFromDoStmt(parser::Label label) {
    CheckLabelInRange(label);
    programUnits_.back().doStmtSources.emplace_back(
        label, currentScope_, currentPosition_);
  }

  void AddLabelReferenceToFormatStmt(parser::Label label) {
    CheckLabelInRange(label);
    programUnits_.back().formatStmtSources.emplace_back(
        label, currentScope_, currentPosition_);
  }

  void AddLabelReferenceFromAssignStmt(parser::Label label) {
    CheckLabelInRange(label);
    programUnits_.back().assignStmtSources.emplace_back(
        label, currentScope_, currentPosition_);
  }

  void AddLabelReference(parser::Label label) {
    CheckLabelInRange(label);
````
- **L865 EN**: Executes a call or declaration centered on `SayLabel`.
  **L865 CN**: 执行以 `SayLabel` 为核心的调用或声明。
- **L866 EN**: Closes the current lexical scope or compound statement.
  **L866 CN**: 结束当前词法作用域或复合语句块。
- **L867 EN**: Closes the current lexical scope or compound statement.
  **L867 CN**: 结束当前词法作用域或复合语句块。
- **L868 EN**: Blank line separating nearby declarations or logic blocks.
  **L868 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L869 EN**: Starts a function, method, lambda, or structured scope: `void AddLabelReferenceFromDoStmt(parser::Label label) {`.
  **L869 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AddLabelReferenceFromDoStmt(parser::Label label) {`。
- **L870 EN**: Executes a call or declaration centered on `CheckLabelInRange`.
  **L870 CN**: 执行以 `CheckLabelInRange` 为核心的调用或声明。
- **L871 EN**: Continues logic associated with callable symbol `back`.
  **L871 CN**: 继续与可调用符号 `back` 相关的逻辑。
- **L872 EN**: Executes a standalone statement or declaration: `label, currentScope_, currentPosition_);`.
  **L872 CN**: 执行一条独立语句或声明：`label, currentScope_, currentPosition_);`。
- **L873 EN**: Closes the current lexical scope or compound statement.
  **L873 CN**: 结束当前词法作用域或复合语句块。
- **L874 EN**: Blank line separating nearby declarations or logic blocks.
  **L874 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L875 EN**: Starts a function, method, lambda, or structured scope: `void AddLabelReferenceToFormatStmt(parser::Label label) {`.
  **L875 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AddLabelReferenceToFormatStmt(parser::Label label) {`。
- **L876 EN**: Executes a call or declaration centered on `CheckLabelInRange`.
  **L876 CN**: 执行以 `CheckLabelInRange` 为核心的调用或声明。
- **L877 EN**: Continues logic associated with callable symbol `back`.
  **L877 CN**: 继续与可调用符号 `back` 相关的逻辑。
- **L878 EN**: Executes a standalone statement or declaration: `label, currentScope_, currentPosition_);`.
  **L878 CN**: 执行一条独立语句或声明：`label, currentScope_, currentPosition_);`。
- **L879 EN**: Closes the current lexical scope or compound statement.
  **L879 CN**: 结束当前词法作用域或复合语句块。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L881 EN**: Starts a function, method, lambda, or structured scope: `void AddLabelReferenceFromAssignStmt(parser::Label label) {`.
  **L881 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AddLabelReferenceFromAssignStmt(parser::Label label) {`。
- **L882 EN**: Executes a call or declaration centered on `CheckLabelInRange`.
  **L882 CN**: 执行以 `CheckLabelInRange` 为核心的调用或声明。
- **L883 EN**: Continues logic associated with callable symbol `back`.
  **L883 CN**: 继续与可调用符号 `back` 相关的逻辑。
- **L884 EN**: Executes a standalone statement or declaration: `label, currentScope_, currentPosition_);`.
  **L884 CN**: 执行一条独立语句或声明：`label, currentScope_, currentPosition_);`。
- **L885 EN**: Closes the current lexical scope or compound statement.
  **L885 CN**: 结束当前词法作用域或复合语句块。
- **L886 EN**: Blank line separating nearby declarations or logic blocks.
  **L886 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L887 EN**: Starts a function, method, lambda, or structured scope: `void AddLabelReference(parser::Label label) {`.
  **L887 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AddLabelReference(parser::Label label) {`。
- **L888 EN**: Executes a call or declaration centered on `CheckLabelInRange`.
  **L888 CN**: 执行以 `CheckLabelInRange` 为核心的调用或声明。

### Lines 889-912

````cpp
    programUnits_.back().otherStmtSources.emplace_back(
        label, currentScope_, currentPosition_);
  }

  void AddLabelReference(const std::list<parser::Label> &labels) {
    for (const parser::Label &label : labels) {
      AddLabelReference(label);
    }
  }

  void PushDisposableMap() { disposableMaps_.emplace_back(); }
  void PopDisposableMap() { disposableMaps_.pop_back(); }

  std::vector<UnitAnalysis> programUnits_;
  SemanticsContext &context_;
  parser::CharBlock currentPosition_;
  ProxyForScope currentScope_;
  std::vector<std::string> constructNames_;
  // For labels in derived type definitions and procedure
  // interfaces, which are their own inclusive scopes.  None
  // of these labels can be used as a branch target, but they
  // should be pairwise distinct.
  std::vector<TargetStmtMap> disposableMaps_;
};
````
- **L889 EN**: Continues logic associated with callable symbol `back`.
  **L889 CN**: 继续与可调用符号 `back` 相关的逻辑。
- **L890 EN**: Executes a standalone statement or declaration: `label, currentScope_, currentPosition_);`.
  **L890 CN**: 执行一条独立语句或声明：`label, currentScope_, currentPosition_);`。
- **L891 EN**: Closes the current lexical scope or compound statement.
  **L891 CN**: 结束当前词法作用域或复合语句块。
- **L892 EN**: Blank line separating nearby declarations or logic blocks.
  **L892 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L893 EN**: Starts a function, method, lambda, or structured scope: `void AddLabelReference(const std::list<parser::Label> &labels) {`.
  **L893 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AddLabelReference(const std::list<parser::Label> &labels) {`。
- **L894 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L894 CN**: 开始 `for` 控制流语句并计算其条件。
- **L895 EN**: Executes a call or declaration centered on `AddLabelReference`.
  **L895 CN**: 执行以 `AddLabelReference` 为核心的调用或声明。
- **L896 EN**: Closes the current lexical scope or compound statement.
  **L896 CN**: 结束当前词法作用域或复合语句块。
- **L897 EN**: Closes the current lexical scope or compound statement.
  **L897 CN**: 结束当前词法作用域或复合语句块。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L899 EN**: Continues logic associated with callable symbol `PushDisposableMap`.
  **L899 CN**: 继续与可调用符号 `PushDisposableMap` 相关的逻辑。
- **L900 EN**: Continues logic associated with callable symbol `PopDisposableMap`.
  **L900 CN**: 继续与可调用符号 `PopDisposableMap` 相关的逻辑。
- **L901 EN**: Blank line separating nearby declarations or logic blocks.
  **L901 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L902 EN**: Executes a standalone statement or declaration: `std::vector<UnitAnalysis> programUnits_;`.
  **L902 CN**: 执行一条独立语句或声明：`std::vector<UnitAnalysis> programUnits_;`。
- **L903 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L903 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。
- **L904 EN**: Executes a standalone statement or declaration: `parser::CharBlock currentPosition_;`.
  **L904 CN**: 执行一条独立语句或声明：`parser::CharBlock currentPosition_;`。
- **L905 EN**: Executes a standalone statement or declaration: `ProxyForScope currentScope_;`.
  **L905 CN**: 执行一条独立语句或声明：`ProxyForScope currentScope_;`。
- **L906 EN**: Executes a standalone statement or declaration: `std::vector<std::string> constructNames_;`.
  **L906 CN**: 执行一条独立语句或声明：`std::vector<std::string> constructNames_;`。
- **L907 EN**: Comment explains nearby logic, intent, or metadata: `For labels in derived type definitions and procedure`.
  **L907 CN**: 注释说明附近代码的逻辑、意图或元数据：`For labels in derived type definitions and procedure`。
- **L908 EN**: Comment explains nearby logic, intent, or metadata: `interfaces, which are their own inclusive scopes.  None`.
  **L908 CN**: 注释说明附近代码的逻辑、意图或元数据：`interfaces, which are their own inclusive scopes.  None`。
- **L909 EN**: Comment explains nearby logic, intent, or metadata: `of these labels can be used as a branch target, but they`.
  **L909 CN**: 注释说明附近代码的逻辑、意图或元数据：`of these labels can be used as a branch target, but they`。
- **L910 EN**: Comment explains nearby logic, intent, or metadata: `should be pairwise distinct.`.
  **L910 CN**: 注释说明附近代码的逻辑、意图或元数据：`should be pairwise distinct.`。
- **L911 EN**: Executes a standalone statement or declaration: `std::vector<TargetStmtMap> disposableMaps_;`.
  **L911 CN**: 执行一条独立语句或声明：`std::vector<TargetStmtMap> disposableMaps_;`。
- **L912 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L912 CN**: 结束当前声明作用域，例如类、结构体或枚举。

### Lines 913-936

````cpp

bool InInclusiveScope(const std::vector<ScopeInfo> &scopes, ProxyForScope tail,
    ProxyForScope head) {
  for (; tail != head; tail = scopes[tail].parent) {
    if (!HasScope(tail)) {
      return false;
    }
  }
  return true;
}

ParseTreeAnalyzer LabelAnalysis(
    SemanticsContext &context, const parser::Program &program) {
  ParseTreeAnalyzer analysis{context};
  Walk(program, analysis);
  return analysis;
}

bool InBody(const parser::CharBlock &position,
    const std::pair<parser::CharBlock, parser::CharBlock> &pair) {
  if (position.begin() >= pair.first.begin()) {
    if (position.begin() < pair.second.end()) {
      return true;
    }
````
- **L913 EN**: Blank line separating nearby declarations or logic blocks.
  **L913 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool InInclusiveScope(const std::vector<ScopeInfo> &scopes, ProxyForScope tail,`.
  **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool InInclusiveScope(const std::vector<ScopeInfo> &scopes, ProxyForScope tail,`。
- **L915 EN**: Continues the surrounding expression or declaration: `ProxyForScope head) {`.
  **L915 CN**: 继续构造周围的表达式或声明：`ProxyForScope head) {`。
- **L916 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L916 CN**: 开始 `for` 控制流语句并计算其条件。
- **L917 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L917 CN**: 开始 `if` 控制流语句并计算其条件。
- **L918 EN**: Returns from the current function with `false`.
  **L918 CN**: 以 `false` 从当前函数返回。
- **L919 EN**: Closes the current lexical scope or compound statement.
  **L919 CN**: 结束当前词法作用域或复合语句块。
- **L920 EN**: Closes the current lexical scope or compound statement.
  **L920 CN**: 结束当前词法作用域或复合语句块。
- **L921 EN**: Returns from the current function with `true`.
  **L921 CN**: 以 `true` 从当前函数返回。
- **L922 EN**: Closes the current lexical scope or compound statement.
  **L922 CN**: 结束当前词法作用域或复合语句块。
- **L923 EN**: Blank line separating nearby declarations or logic blocks.
  **L923 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L924 EN**: Continues logic associated with callable symbol `LabelAnalysis`.
  **L924 CN**: 继续与可调用符号 `LabelAnalysis` 相关的逻辑。
- **L925 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::Program &program) {`.
  **L925 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::Program &program) {`。
- **L926 EN**: Executes a standalone statement or declaration: `ParseTreeAnalyzer analysis{context};`.
  **L926 CN**: 执行一条独立语句或声明：`ParseTreeAnalyzer analysis{context};`。
- **L927 EN**: Executes a call or declaration centered on `Walk`.
  **L927 CN**: 执行以 `Walk` 为核心的调用或声明。
- **L928 EN**: Returns from the current function with `analysis`.
  **L928 CN**: 以 `analysis` 从当前函数返回。
- **L929 EN**: Closes the current lexical scope or compound statement.
  **L929 CN**: 结束当前词法作用域或复合语句块。
- **L930 EN**: Blank line separating nearby declarations or logic blocks.
  **L930 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L931 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool InBody(const parser::CharBlock &position,`.
  **L931 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool InBody(const parser::CharBlock &position,`。
- **L932 EN**: Continues the surrounding expression or declaration: `const std::pair<parser::CharBlock, parser::CharBlock> &pair) {`.
  **L932 CN**: 继续构造周围的表达式或声明：`const std::pair<parser::CharBlock, parser::CharBlock> &pair) {`。
- **L933 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L933 CN**: 开始 `if` 控制流语句并计算其条件。
- **L934 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L934 CN**: 开始 `if` 控制流语句并计算其条件。
- **L935 EN**: Returns from the current function with `true`.
  **L935 CN**: 以 `true` 从当前函数返回。
- **L936 EN**: Closes the current lexical scope or compound statement.
  **L936 CN**: 结束当前词法作用域或复合语句块。

### Lines 937-960

````cpp
  }
  return false;
}

static LabeledStatementInfoTuplePOD GetLabel(
    const TargetStmtMap &labels, const parser::Label &label) {
  const auto iter{labels.find(label)};
  if (iter == labels.cend()) {
    return {0u, nullptr, LabeledStmtClassificationSet{}, false};
  } else {
    return iter->second;
  }
}

// 11.1.7.3
void CheckBranchesIntoDoBody(const SourceStmtList &branches,
    const TargetStmtMap &labels, const IndexList &loopBodies,
    SemanticsContext &context) {
  for (const auto &branch : branches) {
    const auto &label{branch.parserLabel};
    auto branchTarget{GetLabel(labels, label)};
    if (HasScope(branchTarget.proxyForScope)) {
      const auto &fromPosition{branch.parserCharBlock};
      const auto &toPosition{branchTarget.parserCharBlock};
````
- **L937 EN**: Closes the current lexical scope or compound statement.
  **L937 CN**: 结束当前词法作用域或复合语句块。
- **L938 EN**: Returns from the current function with `false`.
  **L938 CN**: 以 `false` 从当前函数返回。
- **L939 EN**: Closes the current lexical scope or compound statement.
  **L939 CN**: 结束当前词法作用域或复合语句块。
- **L940 EN**: Blank line separating nearby declarations or logic blocks.
  **L940 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L941 EN**: Continues logic associated with callable symbol `GetLabel`.
  **L941 CN**: 继续与可调用符号 `GetLabel` 相关的逻辑。
- **L942 EN**: Continues the surrounding expression or declaration: `const TargetStmtMap &labels, const parser::Label &label) {`.
  **L942 CN**: 继续构造周围的表达式或声明：`const TargetStmtMap &labels, const parser::Label &label) {`。
- **L943 EN**: Executes a call or declaration centered on `iter{labels.find`.
  **L943 CN**: 执行以 `iter{labels.find` 为核心的调用或声明。
- **L944 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L944 CN**: 开始 `if` 控制流语句并计算其条件。
- **L945 EN**: Returns from the current function with `{0u, nullptr, LabeledStmtClassificationSet{}, false}`.
  **L945 CN**: 以 `{0u, nullptr, LabeledStmtClassificationSet{}, false}` 从当前函数返回。
- **L946 EN**: Transitions from the previous branch into the alternative path.
  **L946 CN**: 从前一个分支过渡到备选路径。
- **L947 EN**: Returns from the current function with `iter->second`.
  **L947 CN**: 以 `iter->second` 从当前函数返回。
- **L948 EN**: Closes the current lexical scope or compound statement.
  **L948 CN**: 结束当前词法作用域或复合语句块。
- **L949 EN**: Closes the current lexical scope or compound statement.
  **L949 CN**: 结束当前词法作用域或复合语句块。
- **L950 EN**: Blank line separating nearby declarations or logic blocks.
  **L950 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L951 EN**: Comment explains nearby logic, intent, or metadata: `11.1.7.3`.
  **L951 CN**: 注释说明附近代码的逻辑、意图或元数据：`11.1.7.3`。
- **L952 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckBranchesIntoDoBody(const SourceStmtList &branches,`.
  **L952 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckBranchesIntoDoBody(const SourceStmtList &branches,`。
- **L953 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TargetStmtMap &labels, const IndexList &loopBodies,`.
  **L953 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TargetStmtMap &labels, const IndexList &loopBodies,`。
- **L954 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context) {`.
  **L954 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context) {`。
- **L955 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L955 CN**: 开始 `for` 控制流语句并计算其条件。
- **L956 EN**: Executes a standalone statement or declaration: `const auto &label{branch.parserLabel};`.
  **L956 CN**: 执行一条独立语句或声明：`const auto &label{branch.parserLabel};`。
- **L957 EN**: Executes a call or declaration centered on `branchTarget{GetLabel`.
  **L957 CN**: 执行以 `branchTarget{GetLabel` 为核心的调用或声明。
- **L958 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L958 CN**: 开始 `if` 控制流语句并计算其条件。
- **L959 EN**: Executes a standalone statement or declaration: `const auto &fromPosition{branch.parserCharBlock};`.
  **L959 CN**: 执行一条独立语句或声明：`const auto &fromPosition{branch.parserCharBlock};`。
- **L960 EN**: Executes a standalone statement or declaration: `const auto &toPosition{branchTarget.parserCharBlock};`.
  **L960 CN**: 执行一条独立语句或声明：`const auto &toPosition{branchTarget.parserCharBlock};`。

### Lines 961-984

````cpp
      for (const auto &body : loopBodies) {
        if (!InBody(fromPosition, body) && InBody(toPosition, body) &&
            context.ShouldWarn(common::LanguageFeature::BranchIntoConstruct)) {
          context
              .Say(
                  fromPosition, "branch into loop body from outside"_warn_en_US)
              .Attach(body.first, "the loop branched into"_en_US)
              .set_languageFeature(
                  common::LanguageFeature::BranchIntoConstruct);
        }
      }
    }
  }
}

void CheckDoNesting(const IndexList &loopBodies, SemanticsContext &context) {
  for (auto i1{loopBodies.cbegin()}; i1 != loopBodies.cend(); ++i1) {
    const auto &v1{*i1};
    for (auto i2{i1 + 1}; i2 != loopBodies.cend(); ++i2) {
      const auto &v2{*i2};
      if (v2.first.begin() < v1.second.end() &&
          v1.second.begin() < v2.second.begin()) {
        context.Say(v1.first, "DO loop doesn't properly nest"_err_en_US)
            .Attach(v2.first, "DO loop conflicts"_en_US);
````
- **L961 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L961 CN**: 开始 `for` 控制流语句并计算其条件。
- **L962 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L962 CN**: 开始 `if` 控制流语句并计算其条件。
- **L963 EN**: Starts a function, method, lambda, or structured scope: `context.ShouldWarn(common::LanguageFeature::BranchIntoConstruct)) {`.
  **L963 CN**: 开始一个函数、方法、lambda 或结构化作用域：`context.ShouldWarn(common::LanguageFeature::BranchIntoConstruct)) {`。
- **L964 EN**: Continues the surrounding expression or declaration: `context`.
  **L964 CN**: 继续构造周围的表达式或声明：`context`。
- **L965 EN**: Continues logic associated with callable symbol `Say`.
  **L965 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L966 EN**: Continues the surrounding expression or declaration: `fromPosition, "branch into loop body from outside"_warn_en_US)`.
  **L966 CN**: 继续构造周围的表达式或声明：`fromPosition, "branch into loop body from outside"_warn_en_US)`。
- **L967 EN**: Continues logic associated with callable symbol `Attach`.
  **L967 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L968 EN**: Continues logic associated with callable symbol `set_languageFeature`.
  **L968 CN**: 继续与可调用符号 `set_languageFeature` 相关的逻辑。
- **L969 EN**: Executes a standalone statement or declaration: `common::LanguageFeature::BranchIntoConstruct);`.
  **L969 CN**: 执行一条独立语句或声明：`common::LanguageFeature::BranchIntoConstruct);`。
- **L970 EN**: Closes the current lexical scope or compound statement.
  **L970 CN**: 结束当前词法作用域或复合语句块。
- **L971 EN**: Closes the current lexical scope or compound statement.
  **L971 CN**: 结束当前词法作用域或复合语句块。
- **L972 EN**: Closes the current lexical scope or compound statement.
  **L972 CN**: 结束当前词法作用域或复合语句块。
- **L973 EN**: Closes the current lexical scope or compound statement.
  **L973 CN**: 结束当前词法作用域或复合语句块。
- **L974 EN**: Closes the current lexical scope or compound statement.
  **L974 CN**: 结束当前词法作用域或复合语句块。
- **L975 EN**: Blank line separating nearby declarations or logic blocks.
  **L975 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L976 EN**: Starts a function, method, lambda, or structured scope: `void CheckDoNesting(const IndexList &loopBodies, SemanticsContext &context) {`.
  **L976 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckDoNesting(const IndexList &loopBodies, SemanticsContext &context) {`。
- **L977 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L977 CN**: 开始 `for` 控制流语句并计算其条件。
- **L978 EN**: Executes a standalone statement or declaration: `const auto &v1{*i1};`.
  **L978 CN**: 执行一条独立语句或声明：`const auto &v1{*i1};`。
- **L979 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L979 CN**: 开始 `for` 控制流语句并计算其条件。
- **L980 EN**: Executes a standalone statement or declaration: `const auto &v2{*i2};`.
  **L980 CN**: 执行一条独立语句或声明：`const auto &v2{*i2};`。
- **L981 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L981 CN**: 开始 `if` 控制流语句并计算其条件。
- **L982 EN**: Starts a function, method, lambda, or structured scope: `v1.second.begin() < v2.second.begin()) {`.
  **L982 CN**: 开始一个函数、方法、lambda 或结构化作用域：`v1.second.begin() < v2.second.begin()) {`。
- **L983 EN**: Continues logic associated with callable symbol `Say`.
  **L983 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L984 EN**: Executes a call or declaration centered on `.Attach`.
  **L984 CN**: 执行以 `.Attach` 为核心的调用或声明。

### Lines 985-1008

````cpp
      }
    }
  }
}

parser::CharBlock SkipLabel(const parser::CharBlock &position) {
  const std::size_t maxPosition{position.size()};
  if (maxPosition && parser::IsDecimalDigit(position[0])) {
    std::size_t i{1l};
    for (; (i < maxPosition) && parser::IsDecimalDigit(position[i]); ++i) {
    }
    for (; (i < maxPosition) && parser::IsWhiteSpace(position[i]); ++i) {
    }
    return parser::CharBlock{position.begin() + i, position.end()};
  }
  return position;
}

ProxyForScope ParentScope(
    const std::vector<ScopeInfo> &scopes, ProxyForScope scope) {
  return scopes[scope].parent;
}

void CheckLabelDoConstraints(const SourceStmtList &dos,
````
- **L985 EN**: Closes the current lexical scope or compound statement.
  **L985 CN**: 结束当前词法作用域或复合语句块。
- **L986 EN**: Closes the current lexical scope or compound statement.
  **L986 CN**: 结束当前词法作用域或复合语句块。
- **L987 EN**: Closes the current lexical scope or compound statement.
  **L987 CN**: 结束当前词法作用域或复合语句块。
- **L988 EN**: Closes the current lexical scope or compound statement.
  **L988 CN**: 结束当前词法作用域或复合语句块。
- **L989 EN**: Blank line separating nearby declarations or logic blocks.
  **L989 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L990 EN**: Starts a function, method, lambda, or structured scope: `parser::CharBlock SkipLabel(const parser::CharBlock &position) {`.
  **L990 CN**: 开始一个函数、方法、lambda 或结构化作用域：`parser::CharBlock SkipLabel(const parser::CharBlock &position) {`。
- **L991 EN**: Executes a call or declaration centered on `maxPosition{position.size`.
  **L991 CN**: 执行以 `maxPosition{position.size` 为核心的调用或声明。
- **L992 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L992 CN**: 开始 `if` 控制流语句并计算其条件。
- **L993 EN**: Executes a standalone statement or declaration: `std::size_t i{1l};`.
  **L993 CN**: 执行一条独立语句或声明：`std::size_t i{1l};`。
- **L994 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L994 CN**: 开始 `for` 控制流语句并计算其条件。
- **L995 EN**: Closes the current lexical scope or compound statement.
  **L995 CN**: 结束当前词法作用域或复合语句块。
- **L996 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L996 CN**: 开始 `for` 控制流语句并计算其条件。
- **L997 EN**: Closes the current lexical scope or compound statement.
  **L997 CN**: 结束当前词法作用域或复合语句块。
- **L998 EN**: Returns from the current function with `parser::CharBlock{position.begin() + i, position.end()}`.
  **L998 CN**: 以 `parser::CharBlock{position.begin() + i, position.end()}` 从当前函数返回。
- **L999 EN**: Closes the current lexical scope or compound statement.
  **L999 CN**: 结束当前词法作用域或复合语句块。
- **L1000 EN**: Returns from the current function with `position`.
  **L1000 CN**: 以 `position` 从当前函数返回。
- **L1001 EN**: Closes the current lexical scope or compound statement.
  **L1001 CN**: 结束当前词法作用域或复合语句块。
- **L1002 EN**: Blank line separating nearby declarations or logic blocks.
  **L1002 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1003 EN**: Continues logic associated with callable symbol `ParentScope`.
  **L1003 CN**: 继续与可调用符号 `ParentScope` 相关的逻辑。
- **L1004 EN**: Continues the surrounding expression or declaration: `const std::vector<ScopeInfo> &scopes, ProxyForScope scope) {`.
  **L1004 CN**: 继续构造周围的表达式或声明：`const std::vector<ScopeInfo> &scopes, ProxyForScope scope) {`。
- **L1005 EN**: Returns from the current function with `scopes[scope].parent`.
  **L1005 CN**: 以 `scopes[scope].parent` 从当前函数返回。
- **L1006 EN**: Closes the current lexical scope or compound statement.
  **L1006 CN**: 结束当前词法作用域或复合语句块。
- **L1007 EN**: Blank line separating nearby declarations or logic blocks.
  **L1007 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1008 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckLabelDoConstraints(const SourceStmtList &dos,`.
  **L1008 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckLabelDoConstraints(const SourceStmtList &dos,`。

### Lines 1009-1032

````cpp
    const SourceStmtList &branches, const TargetStmtMap &labels,
    const std::vector<ScopeInfo> &scopes, SemanticsContext &context) {
  IndexList loopBodies;
  for (const auto &stmt : dos) {
    const auto &label{stmt.parserLabel};
    const auto &scope{stmt.proxyForScope};
    const auto &position{stmt.parserCharBlock};
    auto doTarget{GetLabel(labels, label)};
    if (!HasScope(doTarget.proxyForScope)) {
      // C1133
      context.Say(
          position, "Label '%u' cannot be found"_err_en_US, SayLabel(label));
    } else if (doTarget.parserCharBlock.begin() < position.begin()) {
      // R1119
      context.Say(position,
          "Label '%u' doesn't lexically follow DO stmt"_err_en_US,
          SayLabel(label));

    } else if ((InInclusiveScope(scopes, scope, doTarget.proxyForScope) &&
                   doTarget.labeledStmtClassificationSet.test(
                       TargetStatementEnum::CompatibleDo)) ||
        (doTarget.isExecutableConstructEndStmt &&
            ParentScope(scopes, doTarget.proxyForScope) == scope)) {
      if (context.ShouldWarn(
````
- **L1009 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const SourceStmtList &branches, const TargetStmtMap &labels,`.
  **L1009 CN**: 继续一个多行参数列表、初始化器或聚合项：`const SourceStmtList &branches, const TargetStmtMap &labels,`。
- **L1010 EN**: Continues the surrounding expression or declaration: `const std::vector<ScopeInfo> &scopes, SemanticsContext &context) {`.
  **L1010 CN**: 继续构造周围的表达式或声明：`const std::vector<ScopeInfo> &scopes, SemanticsContext &context) {`。
- **L1011 EN**: Executes a standalone statement or declaration: `IndexList loopBodies;`.
  **L1011 CN**: 执行一条独立语句或声明：`IndexList loopBodies;`。
- **L1012 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1012 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1013 EN**: Executes a standalone statement or declaration: `const auto &label{stmt.parserLabel};`.
  **L1013 CN**: 执行一条独立语句或声明：`const auto &label{stmt.parserLabel};`。
- **L1014 EN**: Executes a standalone statement or declaration: `const auto &scope{stmt.proxyForScope};`.
  **L1014 CN**: 执行一条独立语句或声明：`const auto &scope{stmt.proxyForScope};`。
- **L1015 EN**: Executes a standalone statement or declaration: `const auto &position{stmt.parserCharBlock};`.
  **L1015 CN**: 执行一条独立语句或声明：`const auto &position{stmt.parserCharBlock};`。
- **L1016 EN**: Executes a call or declaration centered on `doTarget{GetLabel`.
  **L1016 CN**: 执行以 `doTarget{GetLabel` 为核心的调用或声明。
- **L1017 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1017 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1018 EN**: Comment explains nearby logic, intent, or metadata: `C1133`.
  **L1018 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1133`。
- **L1019 EN**: Continues logic associated with callable symbol `Say`.
  **L1019 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1020 EN**: Executes a call or declaration centered on `SayLabel`.
  **L1020 CN**: 执行以 `SayLabel` 为核心的调用或声明。
- **L1021 EN**: Transitions from the previous branch into an `else if` condition.
  **L1021 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1022 EN**: Comment explains nearby logic, intent, or metadata: `R1119`.
  **L1022 CN**: 注释说明附近代码的逻辑、意图或元数据：`R1119`。
- **L1023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(position,`.
  **L1023 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(position,`。
- **L1024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Label '%u' doesn't lexically follow DO stmt"_err_en_US,`.
  **L1024 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Label '%u' doesn't lexically follow DO stmt"_err_en_US,`。
- **L1025 EN**: Executes a call or declaration centered on `SayLabel`.
  **L1025 CN**: 执行以 `SayLabel` 为核心的调用或声明。
- **L1026 EN**: Blank line separating nearby declarations or logic blocks.
  **L1026 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1027 EN**: Transitions from the previous branch into an `else if` condition.
  **L1027 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1028 EN**: Continues logic associated with callable symbol `test`.
  **L1028 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1029 EN**: Continues the surrounding expression or declaration: `TargetStatementEnum::CompatibleDo)) ||`.
  **L1029 CN**: 继续构造周围的表达式或声明：`TargetStatementEnum::CompatibleDo)) ||`。
- **L1030 EN**: Continues the surrounding expression or declaration: `(doTarget.isExecutableConstructEndStmt &&`.
  **L1030 CN**: 继续构造周围的表达式或声明：`(doTarget.isExecutableConstructEndStmt &&`。
- **L1031 EN**: Starts a function, method, lambda, or structured scope: `ParentScope(scopes, doTarget.proxyForScope) == scope)) {`.
  **L1031 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ParentScope(scopes, doTarget.proxyForScope) == scope)) {`。
- **L1032 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1032 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1033-1056

````cpp
              common::LanguageFeature::OldLabelDoEndStatements)) {
        context
            .Say(position,
                "A DO loop should terminate with an END DO or CONTINUE"_port_en_US)
            .Attach(doTarget.parserCharBlock,
                "DO loop currently ends at statement:"_en_US)
            .set_languageFeature(
                common::LanguageFeature::OldLabelDoEndStatements);
      }
    } else if (!InInclusiveScope(scopes, scope, doTarget.proxyForScope)) {
      context.Say(position, "Label '%u' is not in DO loop scope"_err_en_US,
          SayLabel(label));
    } else if (!doTarget.labeledStmtClassificationSet.test(
                   TargetStatementEnum::Do)) {
      context
          .Say(doTarget.parserCharBlock,
              "This statement cannot terminate the DO loop"_err_en_US)
          .Attach(position, "which begins at"_en_US);
    } else {
      loopBodies.emplace_back(SkipLabel(position), doTarget.parserCharBlock);
    }
  }

  CheckBranchesIntoDoBody(branches, labels, loopBodies, context);
````
- **L1033 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::OldLabelDoEndStatements)) {`.
  **L1033 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::OldLabelDoEndStatements)) {`。
- **L1034 EN**: Continues the surrounding expression or declaration: `context`.
  **L1034 CN**: 继续构造周围的表达式或声明：`context`。
- **L1035 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(position,`.
  **L1035 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(position,`。
- **L1036 EN**: Continues the surrounding expression or declaration: `"A DO loop should terminate with an END DO or CONTINUE"_port_en_US)`.
  **L1036 CN**: 继续构造周围的表达式或声明：`"A DO loop should terminate with an END DO or CONTINUE"_port_en_US)`。
- **L1037 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(doTarget.parserCharBlock,`.
  **L1037 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(doTarget.parserCharBlock,`。
- **L1038 EN**: Continues the surrounding expression or declaration: `"DO loop currently ends at statement:"_en_US)`.
  **L1038 CN**: 继续构造周围的表达式或声明：`"DO loop currently ends at statement:"_en_US)`。
- **L1039 EN**: Continues logic associated with callable symbol `set_languageFeature`.
  **L1039 CN**: 继续与可调用符号 `set_languageFeature` 相关的逻辑。
- **L1040 EN**: Executes a standalone statement or declaration: `common::LanguageFeature::OldLabelDoEndStatements);`.
  **L1040 CN**: 执行一条独立语句或声明：`common::LanguageFeature::OldLabelDoEndStatements);`。
- **L1041 EN**: Closes the current lexical scope or compound statement.
  **L1041 CN**: 结束当前词法作用域或复合语句块。
- **L1042 EN**: Transitions from the previous branch into an `else if` condition.
  **L1042 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1043 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(position, "Label '%u' is not in DO loop scope"_err_en_US,`.
  **L1043 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(position, "Label '%u' is not in DO loop scope"_err_en_US,`。
- **L1044 EN**: Executes a call or declaration centered on `SayLabel`.
  **L1044 CN**: 执行以 `SayLabel` 为核心的调用或声明。
- **L1045 EN**: Transitions from the previous branch into an `else if` condition.
  **L1045 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1046 EN**: Continues the surrounding expression or declaration: `TargetStatementEnum::Do)) {`.
  **L1046 CN**: 继续构造周围的表达式或声明：`TargetStatementEnum::Do)) {`。
- **L1047 EN**: Continues the surrounding expression or declaration: `context`.
  **L1047 CN**: 继续构造周围的表达式或声明：`context`。
- **L1048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(doTarget.parserCharBlock,`.
  **L1048 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(doTarget.parserCharBlock,`。
- **L1049 EN**: Continues the surrounding expression or declaration: `"This statement cannot terminate the DO loop"_err_en_US)`.
  **L1049 CN**: 继续构造周围的表达式或声明：`"This statement cannot terminate the DO loop"_err_en_US)`。
- **L1050 EN**: Executes a call or declaration centered on `.Attach`.
  **L1050 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L1051 EN**: Transitions from the previous branch into the alternative path.
  **L1051 CN**: 从前一个分支过渡到备选路径。
- **L1052 EN**: Executes a call or declaration centered on `loopBodies.emplace_back`.
  **L1052 CN**: 执行以 `loopBodies.emplace_back` 为核心的调用或声明。
- **L1053 EN**: Closes the current lexical scope or compound statement.
  **L1053 CN**: 结束当前词法作用域或复合语句块。
- **L1054 EN**: Closes the current lexical scope or compound statement.
  **L1054 CN**: 结束当前词法作用域或复合语句块。
- **L1055 EN**: Blank line separating nearby declarations or logic blocks.
  **L1055 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1056 EN**: Executes a call or declaration centered on `CheckBranchesIntoDoBody`.
  **L1056 CN**: 执行以 `CheckBranchesIntoDoBody` 为核心的调用或声明。

### Lines 1057-1080

````cpp
  CheckDoNesting(loopBodies, context);
}

// 6.2.5
void CheckScopeConstraints(const SourceStmtList &stmts,
    const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,
    SemanticsContext &context) {
  for (const auto &stmt : stmts) {
    const auto &label{stmt.parserLabel};
    const auto &scope{stmt.proxyForScope};
    const auto &position{stmt.parserCharBlock};
    auto target{GetLabel(labels, label)};
    if (!HasScope(target.proxyForScope)) {
      context.Say(
          position, "Label '%u' was not found"_err_en_US, SayLabel(label));
    } else if (!InInclusiveScope(scopes, scope, target.proxyForScope)) {
      // Clause 11.1.2.1 prohibits transfer of control to the interior of a
      // block from outside the block, but this does not apply to formats.
      // C1038 and C1034 forbid statements in FORALL and WHERE constructs
      // (resp.) from being branch targets.
      if (target.labeledStmtClassificationSet.test(
              TargetStatementEnum::Format)) {
        continue;
      }
````
- **L1057 EN**: Executes a call or declaration centered on `CheckDoNesting`.
  **L1057 CN**: 执行以 `CheckDoNesting` 为核心的调用或声明。
- **L1058 EN**: Closes the current lexical scope or compound statement.
  **L1058 CN**: 结束当前词法作用域或复合语句块。
- **L1059 EN**: Blank line separating nearby declarations or logic blocks.
  **L1059 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1060 EN**: Comment explains nearby logic, intent, or metadata: `6.2.5`.
  **L1060 CN**: 注释说明附近代码的逻辑、意图或元数据：`6.2.5`。
- **L1061 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckScopeConstraints(const SourceStmtList &stmts,`.
  **L1061 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckScopeConstraints(const SourceStmtList &stmts,`。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,`.
  **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,`。
- **L1063 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context) {`.
  **L1063 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context) {`。
- **L1064 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1064 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1065 EN**: Executes a standalone statement or declaration: `const auto &label{stmt.parserLabel};`.
  **L1065 CN**: 执行一条独立语句或声明：`const auto &label{stmt.parserLabel};`。
- **L1066 EN**: Executes a standalone statement or declaration: `const auto &scope{stmt.proxyForScope};`.
  **L1066 CN**: 执行一条独立语句或声明：`const auto &scope{stmt.proxyForScope};`。
- **L1067 EN**: Executes a standalone statement or declaration: `const auto &position{stmt.parserCharBlock};`.
  **L1067 CN**: 执行一条独立语句或声明：`const auto &position{stmt.parserCharBlock};`。
- **L1068 EN**: Executes a call or declaration centered on `target{GetLabel`.
  **L1068 CN**: 执行以 `target{GetLabel` 为核心的调用或声明。
- **L1069 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1069 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1070 EN**: Continues logic associated with callable symbol `Say`.
  **L1070 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1071 EN**: Executes a call or declaration centered on `SayLabel`.
  **L1071 CN**: 执行以 `SayLabel` 为核心的调用或声明。
- **L1072 EN**: Transitions from the previous branch into an `else if` condition.
  **L1072 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1073 EN**: Comment explains nearby logic, intent, or metadata: `Clause 11.1.2.1 prohibits transfer of control to the interior of a`.
  **L1073 CN**: 注释说明附近代码的逻辑、意图或元数据：`Clause 11.1.2.1 prohibits transfer of control to the interior of a`。
- **L1074 EN**: Comment explains nearby logic, intent, or metadata: `block from outside the block, but this does not apply to formats.`.
  **L1074 CN**: 注释说明附近代码的逻辑、意图或元数据：`block from outside the block, but this does not apply to formats.`。
- **L1075 EN**: Comment explains nearby logic, intent, or metadata: `C1038 and C1034 forbid statements in FORALL and WHERE constructs`.
  **L1075 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1038 and C1034 forbid statements in FORALL and WHERE constructs`。
- **L1076 EN**: Comment explains nearby logic, intent, or metadata: `(resp.) from being branch targets.`.
  **L1076 CN**: 注释说明附近代码的逻辑、意图或元数据：`(resp.) from being branch targets.`。
- **L1077 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1077 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1078 EN**: Continues the surrounding expression or declaration: `TargetStatementEnum::Format)) {`.
  **L1078 CN**: 继续构造周围的表达式或声明：`TargetStatementEnum::Format)) {`。
- **L1079 EN**: Skips to the next loop iteration.
  **L1079 CN**: 跳到下一次循环迭代。
- **L1080 EN**: Closes the current lexical scope or compound statement.
  **L1080 CN**: 结束当前词法作用域或复合语句块。

### Lines 1081-1104

````cpp
      bool isFatal{false};
      ProxyForScope fromScope{scope};
      for (ProxyForScope toScope{target.proxyForScope}; HasScope(toScope);
           toScope = scopes[toScope].parent) {
        while (scopes[fromScope].depth > scopes[toScope].depth) {
          fromScope = scopes[fromScope].parent;
        }
        if (toScope == fromScope) {
          break;
        }
        if (scopes[toScope].isExteriorGotoFatal) {
          isFatal = true;
          break;
        }
      }
      if (isFatal) {
        context.Say(position,
            "Label '%u' is in a construct that prevents its use as a branch target here"_err_en_US,
            SayLabel(label));
      } else if (context.ShouldWarn(
                     common::LanguageFeature::BranchIntoConstruct)) {
        context
            .Say(position,
                "Label '%u' is in a construct that should not be used as a branch target here"_warn_en_US,
````
- **L1081 EN**: Executes a standalone statement or declaration: `bool isFatal{false};`.
  **L1081 CN**: 执行一条独立语句或声明：`bool isFatal{false};`。
- **L1082 EN**: Executes a standalone statement or declaration: `ProxyForScope fromScope{scope};`.
  **L1082 CN**: 执行一条独立语句或声明：`ProxyForScope fromScope{scope};`。
- **L1083 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1083 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1084 EN**: Continues the surrounding expression or declaration: `toScope = scopes[toScope].parent) {`.
  **L1084 CN**: 继续构造周围的表达式或声明：`toScope = scopes[toScope].parent) {`。
- **L1085 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L1085 CN**: 开始 `while` 控制流语句并计算其条件。
- **L1086 EN**: Executes a standalone statement or declaration: `fromScope = scopes[fromScope].parent;`.
  **L1086 CN**: 执行一条独立语句或声明：`fromScope = scopes[fromScope].parent;`。
- **L1087 EN**: Closes the current lexical scope or compound statement.
  **L1087 CN**: 结束当前词法作用域或复合语句块。
- **L1088 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1088 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1089 EN**: Exits the nearest loop or switch statement.
  **L1089 CN**: 退出最近的循环或 switch 语句。
- **L1090 EN**: Closes the current lexical scope or compound statement.
  **L1090 CN**: 结束当前词法作用域或复合语句块。
- **L1091 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1091 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1092 EN**: Executes a standalone statement or declaration: `isFatal = true;`.
  **L1092 CN**: 执行一条独立语句或声明：`isFatal = true;`。
- **L1093 EN**: Exits the nearest loop or switch statement.
  **L1093 CN**: 退出最近的循环或 switch 语句。
- **L1094 EN**: Closes the current lexical scope or compound statement.
  **L1094 CN**: 结束当前词法作用域或复合语句块。
- **L1095 EN**: Closes the current lexical scope or compound statement.
  **L1095 CN**: 结束当前词法作用域或复合语句块。
- **L1096 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1096 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1097 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(position,`.
  **L1097 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(position,`。
- **L1098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Label '%u' is in a construct that prevents its use as a branch target here"_err_en_US,`.
  **L1098 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Label '%u' is in a construct that prevents its use as a branch target here"_err_en_US,`。
- **L1099 EN**: Executes a call or declaration centered on `SayLabel`.
  **L1099 CN**: 执行以 `SayLabel` 为核心的调用或声明。
- **L1100 EN**: Transitions from the previous branch into an `else if` condition.
  **L1100 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1101 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::BranchIntoConstruct)) {`.
  **L1101 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::BranchIntoConstruct)) {`。
- **L1102 EN**: Continues the surrounding expression or declaration: `context`.
  **L1102 CN**: 继续构造周围的表达式或声明：`context`。
- **L1103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(position,`.
  **L1103 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(position,`。
- **L1104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Label '%u' is in a construct that should not be used as a branch target here"_warn_en_US,`.
  **L1104 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Label '%u' is in a construct that should not be used as a branch target here"_warn_en_US,`。

### Lines 1105-1128

````cpp
                SayLabel(label))
            .set_languageFeature(common::LanguageFeature::BranchIntoConstruct);
      }
    }
  }
}

void CheckBranchTargetConstraints(const SourceStmtList &stmts,
    const TargetStmtMap &labels, SemanticsContext &context) {
  for (const auto &stmt : stmts) {
    const auto &label{stmt.parserLabel};
    auto branchTarget{GetLabel(labels, label)};
    if (HasScope(branchTarget.proxyForScope)) {
      if (!branchTarget.labeledStmtClassificationSet.test(
              TargetStatementEnum::Branch) &&
          !branchTarget.labeledStmtClassificationSet.test(
              TargetStatementEnum::CompatibleBranch)) { // error
        context
            .Say(branchTarget.parserCharBlock,
                "Label '%u' is not a branch target"_err_en_US, SayLabel(label))
            .Attach(stmt.parserCharBlock, "Control flow use of '%u'"_en_US,
                SayLabel(label));
      } else if (!branchTarget.labeledStmtClassificationSet.test(
                     TargetStatementEnum::Branch) &&
````
- **L1105 EN**: Continues logic associated with callable symbol `SayLabel`.
  **L1105 CN**: 继续与可调用符号 `SayLabel` 相关的逻辑。
- **L1106 EN**: Executes a call or declaration centered on `.set_languageFeature`.
  **L1106 CN**: 执行以 `.set_languageFeature` 为核心的调用或声明。
- **L1107 EN**: Closes the current lexical scope or compound statement.
  **L1107 CN**: 结束当前词法作用域或复合语句块。
- **L1108 EN**: Closes the current lexical scope or compound statement.
  **L1108 CN**: 结束当前词法作用域或复合语句块。
- **L1109 EN**: Closes the current lexical scope or compound statement.
  **L1109 CN**: 结束当前词法作用域或复合语句块。
- **L1110 EN**: Closes the current lexical scope or compound statement.
  **L1110 CN**: 结束当前词法作用域或复合语句块。
- **L1111 EN**: Blank line separating nearby declarations or logic blocks.
  **L1111 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckBranchTargetConstraints(const SourceStmtList &stmts,`.
  **L1112 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckBranchTargetConstraints(const SourceStmtList &stmts,`。
- **L1113 EN**: Continues the surrounding expression or declaration: `const TargetStmtMap &labels, SemanticsContext &context) {`.
  **L1113 CN**: 继续构造周围的表达式或声明：`const TargetStmtMap &labels, SemanticsContext &context) {`。
- **L1114 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1114 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1115 EN**: Executes a standalone statement or declaration: `const auto &label{stmt.parserLabel};`.
  **L1115 CN**: 执行一条独立语句或声明：`const auto &label{stmt.parserLabel};`。
- **L1116 EN**: Executes a call or declaration centered on `branchTarget{GetLabel`.
  **L1116 CN**: 执行以 `branchTarget{GetLabel` 为核心的调用或声明。
- **L1117 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1117 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1118 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1118 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1119 EN**: Continues the surrounding expression or declaration: `TargetStatementEnum::Branch) &&`.
  **L1119 CN**: 继续构造周围的表达式或声明：`TargetStatementEnum::Branch) &&`。
- **L1120 EN**: Continues logic associated with callable symbol `test`.
  **L1120 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1121 EN**: Continues the surrounding expression or declaration: `TargetStatementEnum::CompatibleBranch)) { // error`.
  **L1121 CN**: 继续构造周围的表达式或声明：`TargetStatementEnum::CompatibleBranch)) { // error`。
- **L1122 EN**: Continues the surrounding expression or declaration: `context`.
  **L1122 CN**: 继续构造周围的表达式或声明：`context`。
- **L1123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(branchTarget.parserCharBlock,`.
  **L1123 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(branchTarget.parserCharBlock,`。
- **L1124 EN**: Continues logic associated with callable symbol `SayLabel`.
  **L1124 CN**: 继续与可调用符号 `SayLabel` 相关的逻辑。
- **L1125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(stmt.parserCharBlock, "Control flow use of '%u'"_en_US,`.
  **L1125 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(stmt.parserCharBlock, "Control flow use of '%u'"_en_US,`。
- **L1126 EN**: Executes a call or declaration centered on `SayLabel`.
  **L1126 CN**: 执行以 `SayLabel` 为核心的调用或声明。
- **L1127 EN**: Transitions from the previous branch into an `else if` condition.
  **L1127 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1128 EN**: Continues the surrounding expression or declaration: `TargetStatementEnum::Branch) &&`.
  **L1128 CN**: 继续构造周围的表达式或声明：`TargetStatementEnum::Branch) &&`。

### Lines 1129-1152

````cpp
          context.ShouldWarn(common::LanguageFeature::BadBranchTarget)) {
        context
            .Say(branchTarget.parserCharBlock,
                "Label '%u' is not a branch target"_warn_en_US, SayLabel(label))
            .Attach(stmt.parserCharBlock, "Control flow use of '%u'"_en_US,
                SayLabel(label))
            .set_languageFeature(common::LanguageFeature::BadBranchTarget);
      }
    }
  }
}

void CheckBranchConstraints(const SourceStmtList &branches,
    const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,
    SemanticsContext &context) {
  CheckScopeConstraints(branches, labels, scopes, context);
  CheckBranchTargetConstraints(branches, labels, context);
}

void CheckDataXferTargetConstraints(const SourceStmtList &stmts,
    const TargetStmtMap &labels, SemanticsContext &context) {
  for (const auto &stmt : stmts) {
    const auto &label{stmt.parserLabel};
    auto ioTarget{GetLabel(labels, label)};
````
- **L1129 EN**: Starts a function, method, lambda, or structured scope: `context.ShouldWarn(common::LanguageFeature::BadBranchTarget)) {`.
  **L1129 CN**: 开始一个函数、方法、lambda 或结构化作用域：`context.ShouldWarn(common::LanguageFeature::BadBranchTarget)) {`。
- **L1130 EN**: Continues the surrounding expression or declaration: `context`.
  **L1130 CN**: 继续构造周围的表达式或声明：`context`。
- **L1131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(branchTarget.parserCharBlock,`.
  **L1131 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(branchTarget.parserCharBlock,`。
- **L1132 EN**: Continues logic associated with callable symbol `SayLabel`.
  **L1132 CN**: 继续与可调用符号 `SayLabel` 相关的逻辑。
- **L1133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(stmt.parserCharBlock, "Control flow use of '%u'"_en_US,`.
  **L1133 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(stmt.parserCharBlock, "Control flow use of '%u'"_en_US,`。
- **L1134 EN**: Continues logic associated with callable symbol `SayLabel`.
  **L1134 CN**: 继续与可调用符号 `SayLabel` 相关的逻辑。
- **L1135 EN**: Executes a call or declaration centered on `.set_languageFeature`.
  **L1135 CN**: 执行以 `.set_languageFeature` 为核心的调用或声明。
- **L1136 EN**: Closes the current lexical scope or compound statement.
  **L1136 CN**: 结束当前词法作用域或复合语句块。
- **L1137 EN**: Closes the current lexical scope or compound statement.
  **L1137 CN**: 结束当前词法作用域或复合语句块。
- **L1138 EN**: Closes the current lexical scope or compound statement.
  **L1138 CN**: 结束当前词法作用域或复合语句块。
- **L1139 EN**: Closes the current lexical scope or compound statement.
  **L1139 CN**: 结束当前词法作用域或复合语句块。
- **L1140 EN**: Blank line separating nearby declarations or logic blocks.
  **L1140 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckBranchConstraints(const SourceStmtList &branches,`.
  **L1141 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckBranchConstraints(const SourceStmtList &branches,`。
- **L1142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,`.
  **L1142 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,`。
- **L1143 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context) {`.
  **L1143 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context) {`。
- **L1144 EN**: Executes a call or declaration centered on `CheckScopeConstraints`.
  **L1144 CN**: 执行以 `CheckScopeConstraints` 为核心的调用或声明。
- **L1145 EN**: Executes a call or declaration centered on `CheckBranchTargetConstraints`.
  **L1145 CN**: 执行以 `CheckBranchTargetConstraints` 为核心的调用或声明。
- **L1146 EN**: Closes the current lexical scope or compound statement.
  **L1146 CN**: 结束当前词法作用域或复合语句块。
- **L1147 EN**: Blank line separating nearby declarations or logic blocks.
  **L1147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckDataXferTargetConstraints(const SourceStmtList &stmts,`.
  **L1148 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckDataXferTargetConstraints(const SourceStmtList &stmts,`。
- **L1149 EN**: Continues the surrounding expression or declaration: `const TargetStmtMap &labels, SemanticsContext &context) {`.
  **L1149 CN**: 继续构造周围的表达式或声明：`const TargetStmtMap &labels, SemanticsContext &context) {`。
- **L1150 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1150 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1151 EN**: Executes a standalone statement or declaration: `const auto &label{stmt.parserLabel};`.
  **L1151 CN**: 执行一条独立语句或声明：`const auto &label{stmt.parserLabel};`。
- **L1152 EN**: Executes a call or declaration centered on `ioTarget{GetLabel`.
  **L1152 CN**: 执行以 `ioTarget{GetLabel` 为核心的调用或声明。

### Lines 1153-1176

````cpp
    if (HasScope(ioTarget.proxyForScope)) {
      if (!ioTarget.labeledStmtClassificationSet.test(
              TargetStatementEnum::Format)) {
        context
            .Say(ioTarget.parserCharBlock, "'%u' not a FORMAT"_err_en_US,
                SayLabel(label))
            .Attach(stmt.parserCharBlock, "data transfer use of '%u'"_en_US,
                SayLabel(label));
      }
    }
  }
}

void CheckDataTransferConstraints(const SourceStmtList &dataTransfers,
    const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,
    SemanticsContext &context) {
  CheckScopeConstraints(dataTransfers, labels, scopes, context);
  CheckDataXferTargetConstraints(dataTransfers, labels, context);
}

void CheckAssignTargetConstraints(const SourceStmtList &stmts,
    const TargetStmtMap &labels, SemanticsContext &context) {
  for (const auto &stmt : stmts) {
    const auto &label{stmt.parserLabel};
````
- **L1153 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1153 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1154 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1154 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1155 EN**: Continues the surrounding expression or declaration: `TargetStatementEnum::Format)) {`.
  **L1155 CN**: 继续构造周围的表达式或声明：`TargetStatementEnum::Format)) {`。
- **L1156 EN**: Continues the surrounding expression or declaration: `context`.
  **L1156 CN**: 继续构造周围的表达式或声明：`context`。
- **L1157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(ioTarget.parserCharBlock, "'%u' not a FORMAT"_err_en_US,`.
  **L1157 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(ioTarget.parserCharBlock, "'%u' not a FORMAT"_err_en_US,`。
- **L1158 EN**: Continues logic associated with callable symbol `SayLabel`.
  **L1158 CN**: 继续与可调用符号 `SayLabel` 相关的逻辑。
- **L1159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Attach(stmt.parserCharBlock, "data transfer use of '%u'"_en_US,`.
  **L1159 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Attach(stmt.parserCharBlock, "data transfer use of '%u'"_en_US,`。
- **L1160 EN**: Executes a call or declaration centered on `SayLabel`.
  **L1160 CN**: 执行以 `SayLabel` 为核心的调用或声明。
- **L1161 EN**: Closes the current lexical scope or compound statement.
  **L1161 CN**: 结束当前词法作用域或复合语句块。
- **L1162 EN**: Closes the current lexical scope or compound statement.
  **L1162 CN**: 结束当前词法作用域或复合语句块。
- **L1163 EN**: Closes the current lexical scope or compound statement.
  **L1163 CN**: 结束当前词法作用域或复合语句块。
- **L1164 EN**: Closes the current lexical scope or compound statement.
  **L1164 CN**: 结束当前词法作用域或复合语句块。
- **L1165 EN**: Blank line separating nearby declarations or logic blocks.
  **L1165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckDataTransferConstraints(const SourceStmtList &dataTransfers,`.
  **L1166 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckDataTransferConstraints(const SourceStmtList &dataTransfers,`。
- **L1167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,`.
  **L1167 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,`。
- **L1168 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context) {`.
  **L1168 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context) {`。
- **L1169 EN**: Executes a call or declaration centered on `CheckScopeConstraints`.
  **L1169 CN**: 执行以 `CheckScopeConstraints` 为核心的调用或声明。
- **L1170 EN**: Executes a call or declaration centered on `CheckDataXferTargetConstraints`.
  **L1170 CN**: 执行以 `CheckDataXferTargetConstraints` 为核心的调用或声明。
- **L1171 EN**: Closes the current lexical scope or compound statement.
  **L1171 CN**: 结束当前词法作用域或复合语句块。
- **L1172 EN**: Blank line separating nearby declarations or logic blocks.
  **L1172 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckAssignTargetConstraints(const SourceStmtList &stmts,`.
  **L1173 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckAssignTargetConstraints(const SourceStmtList &stmts,`。
- **L1174 EN**: Continues the surrounding expression or declaration: `const TargetStmtMap &labels, SemanticsContext &context) {`.
  **L1174 CN**: 继续构造周围的表达式或声明：`const TargetStmtMap &labels, SemanticsContext &context) {`。
- **L1175 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1175 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1176 EN**: Executes a standalone statement or declaration: `const auto &label{stmt.parserLabel};`.
  **L1176 CN**: 执行一条独立语句或声明：`const auto &label{stmt.parserLabel};`。

### Lines 1177-1200

````cpp
    auto target{GetLabel(labels, label)};
    if (HasScope(target.proxyForScope) &&
        !target.labeledStmtClassificationSet.test(
            TargetStatementEnum::Branch) &&
        !target.labeledStmtClassificationSet.test(
            TargetStatementEnum::Format)) {
      parser::Message *msg{nullptr};
      if (!target.labeledStmtClassificationSet.test(
              TargetStatementEnum::CompatibleBranch)) {
        msg = &context.Say(target.parserCharBlock,
            "Label '%u' is not a branch target or FORMAT"_err_en_US,
            SayLabel(label));
      } else if (context.ShouldWarn(common::LanguageFeature::BadBranchTarget)) {
        msg =
            &context
                 .Say(target.parserCharBlock,
                     "Label '%u' is not a branch target or FORMAT"_warn_en_US,
                     SayLabel(label))
                 .set_languageFeature(common::LanguageFeature::BadBranchTarget);
      }
      if (msg) {
        msg->Attach(stmt.parserCharBlock, "ASSIGN statement use of '%u'"_en_US,
            SayLabel(label));
      }
````
- **L1177 EN**: Executes a call or declaration centered on `target{GetLabel`.
  **L1177 CN**: 执行以 `target{GetLabel` 为核心的调用或声明。
- **L1178 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1178 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1179 EN**: Continues logic associated with callable symbol `test`.
  **L1179 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1180 EN**: Continues the surrounding expression or declaration: `TargetStatementEnum::Branch) &&`.
  **L1180 CN**: 继续构造周围的表达式或声明：`TargetStatementEnum::Branch) &&`。
- **L1181 EN**: Continues logic associated with callable symbol `test`.
  **L1181 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1182 EN**: Continues the surrounding expression or declaration: `TargetStatementEnum::Format)) {`.
  **L1182 CN**: 继续构造周围的表达式或声明：`TargetStatementEnum::Format)) {`。
- **L1183 EN**: Executes a standalone statement or declaration: `parser::Message *msg{nullptr};`.
  **L1183 CN**: 执行一条独立语句或声明：`parser::Message *msg{nullptr};`。
- **L1184 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1184 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1185 EN**: Continues the surrounding expression or declaration: `TargetStatementEnum::CompatibleBranch)) {`.
  **L1185 CN**: 继续构造周围的表达式或声明：`TargetStatementEnum::CompatibleBranch)) {`。
- **L1186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg = &context.Say(target.parserCharBlock,`.
  **L1186 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg = &context.Say(target.parserCharBlock,`。
- **L1187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Label '%u' is not a branch target or FORMAT"_err_en_US,`.
  **L1187 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Label '%u' is not a branch target or FORMAT"_err_en_US,`。
- **L1188 EN**: Executes a call or declaration centered on `SayLabel`.
  **L1188 CN**: 执行以 `SayLabel` 为核心的调用或声明。
- **L1189 EN**: Transitions from the previous branch into an `else if` condition.
  **L1189 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1190 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L1190 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L1191 EN**: Continues the surrounding expression or declaration: `&context`.
  **L1191 CN**: 继续构造周围的表达式或声明：`&context`。
- **L1192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(target.parserCharBlock,`.
  **L1192 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(target.parserCharBlock,`。
- **L1193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Label '%u' is not a branch target or FORMAT"_warn_en_US,`.
  **L1193 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Label '%u' is not a branch target or FORMAT"_warn_en_US,`。
- **L1194 EN**: Continues logic associated with callable symbol `SayLabel`.
  **L1194 CN**: 继续与可调用符号 `SayLabel` 相关的逻辑。
- **L1195 EN**: Executes a call or declaration centered on `.set_languageFeature`.
  **L1195 CN**: 执行以 `.set_languageFeature` 为核心的调用或声明。
- **L1196 EN**: Closes the current lexical scope or compound statement.
  **L1196 CN**: 结束当前词法作用域或复合语句块。
- **L1197 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1197 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg->Attach(stmt.parserCharBlock, "ASSIGN statement use of '%u'"_en_US,`.
  **L1198 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg->Attach(stmt.parserCharBlock, "ASSIGN statement use of '%u'"_en_US,`。
- **L1199 EN**: Executes a call or declaration centered on `SayLabel`.
  **L1199 CN**: 执行以 `SayLabel` 为核心的调用或声明。
- **L1200 EN**: Closes the current lexical scope or compound statement.
  **L1200 CN**: 结束当前词法作用域或复合语句块。

### Lines 1201-1224

````cpp
    }
  }
}

void CheckAssignConstraints(const SourceStmtList &assigns,
    const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,
    SemanticsContext &context) {
  CheckScopeConstraints(assigns, labels, scopes, context);
  CheckAssignTargetConstraints(assigns, labels, context);
}

bool CheckConstraints(ParseTreeAnalyzer &&parseTreeAnalysis) {
  auto &context{parseTreeAnalysis.ErrorHandler()};
  for (const auto &programUnit : parseTreeAnalysis.ProgramUnits()) {
    const auto &dos{programUnit.doStmtSources};
    const auto &branches{programUnit.otherStmtSources};
    const auto &labels{programUnit.targetStmts};
    const auto &scopes{programUnit.scopeModel};
    CheckLabelDoConstraints(dos, branches, labels, scopes, context);
    CheckBranchConstraints(branches, labels, scopes, context);
    const auto &dataTransfers{programUnit.formatStmtSources};
    CheckDataTransferConstraints(dataTransfers, labels, scopes, context);
    const auto &assigns{programUnit.assignStmtSources};
    CheckAssignConstraints(assigns, labels, scopes, context);
````
- **L1201 EN**: Closes the current lexical scope or compound statement.
  **L1201 CN**: 结束当前词法作用域或复合语句块。
- **L1202 EN**: Closes the current lexical scope or compound statement.
  **L1202 CN**: 结束当前词法作用域或复合语句块。
- **L1203 EN**: Closes the current lexical scope or compound statement.
  **L1203 CN**: 结束当前词法作用域或复合语句块。
- **L1204 EN**: Blank line separating nearby declarations or logic blocks.
  **L1204 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void CheckAssignConstraints(const SourceStmtList &assigns,`.
  **L1205 CN**: 继续一个多行参数列表、初始化器或聚合项：`void CheckAssignConstraints(const SourceStmtList &assigns,`。
- **L1206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,`.
  **L1206 CN**: 继续一个多行参数列表、初始化器或聚合项：`const TargetStmtMap &labels, const std::vector<ScopeInfo> &scopes,`。
- **L1207 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context) {`.
  **L1207 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context) {`。
- **L1208 EN**: Executes a call or declaration centered on `CheckScopeConstraints`.
  **L1208 CN**: 执行以 `CheckScopeConstraints` 为核心的调用或声明。
- **L1209 EN**: Executes a call or declaration centered on `CheckAssignTargetConstraints`.
  **L1209 CN**: 执行以 `CheckAssignTargetConstraints` 为核心的调用或声明。
- **L1210 EN**: Closes the current lexical scope or compound statement.
  **L1210 CN**: 结束当前词法作用域或复合语句块。
- **L1211 EN**: Blank line separating nearby declarations or logic blocks.
  **L1211 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1212 EN**: Starts a function, method, lambda, or structured scope: `bool CheckConstraints(ParseTreeAnalyzer &&parseTreeAnalysis) {`.
  **L1212 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool CheckConstraints(ParseTreeAnalyzer &&parseTreeAnalysis) {`。
- **L1213 EN**: Executes a call or declaration centered on `&context{parseTreeAnalysis.ErrorHandler`.
  **L1213 CN**: 执行以 `&context{parseTreeAnalysis.ErrorHandler` 为核心的调用或声明。
- **L1214 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1214 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1215 EN**: Executes a standalone statement or declaration: `const auto &dos{programUnit.doStmtSources};`.
  **L1215 CN**: 执行一条独立语句或声明：`const auto &dos{programUnit.doStmtSources};`。
- **L1216 EN**: Executes a standalone statement or declaration: `const auto &branches{programUnit.otherStmtSources};`.
  **L1216 CN**: 执行一条独立语句或声明：`const auto &branches{programUnit.otherStmtSources};`。
- **L1217 EN**: Executes a standalone statement or declaration: `const auto &labels{programUnit.targetStmts};`.
  **L1217 CN**: 执行一条独立语句或声明：`const auto &labels{programUnit.targetStmts};`。
- **L1218 EN**: Executes a standalone statement or declaration: `const auto &scopes{programUnit.scopeModel};`.
  **L1218 CN**: 执行一条独立语句或声明：`const auto &scopes{programUnit.scopeModel};`。
- **L1219 EN**: Executes a call or declaration centered on `CheckLabelDoConstraints`.
  **L1219 CN**: 执行以 `CheckLabelDoConstraints` 为核心的调用或声明。
- **L1220 EN**: Executes a call or declaration centered on `CheckBranchConstraints`.
  **L1220 CN**: 执行以 `CheckBranchConstraints` 为核心的调用或声明。
- **L1221 EN**: Executes a standalone statement or declaration: `const auto &dataTransfers{programUnit.formatStmtSources};`.
  **L1221 CN**: 执行一条独立语句或声明：`const auto &dataTransfers{programUnit.formatStmtSources};`。
- **L1222 EN**: Executes a call or declaration centered on `CheckDataTransferConstraints`.
  **L1222 CN**: 执行以 `CheckDataTransferConstraints` 为核心的调用或声明。
- **L1223 EN**: Executes a standalone statement or declaration: `const auto &assigns{programUnit.assignStmtSources};`.
  **L1223 CN**: 执行一条独立语句或声明：`const auto &assigns{programUnit.assignStmtSources};`。
- **L1224 EN**: Executes a call or declaration centered on `CheckAssignConstraints`.
  **L1224 CN**: 执行以 `CheckAssignConstraints` 为核心的调用或声明。

### Lines 1225-1232

````cpp
  }
  return !context.AnyFatalError();
}

bool ValidateLabels(SemanticsContext &context, const parser::Program &program) {
  return CheckConstraints(LabelAnalysis(context, program));
}
} // namespace Fortran::semantics
````
- **L1225 EN**: Closes the current lexical scope or compound statement.
  **L1225 CN**: 结束当前词法作用域或复合语句块。
- **L1226 EN**: Returns from the current function with `!context.AnyFatalError()`.
  **L1226 CN**: 以 `!context.AnyFatalError()` 从当前函数返回。
- **L1227 EN**: Closes the current lexical scope or compound statement.
  **L1227 CN**: 结束当前词法作用域或复合语句块。
- **L1228 EN**: Blank line separating nearby declarations or logic blocks.
  **L1228 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1229 EN**: Starts a function, method, lambda, or structured scope: `bool ValidateLabels(SemanticsContext &context, const parser::Program &program) {`.
  **L1229 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool ValidateLabels(SemanticsContext &context, const parser::Program &program) {`。
- **L1230 EN**: Returns from the current function with `CheckConstraints(LabelAnalysis(context, program))`.
  **L1230 CN**: 以 `CheckConstraints(LabelAnalysis(context, program))` 从当前函数返回。
- **L1231 EN**: Closes the current lexical scope or compound statement.
  **L1231 CN**: 结束当前词法作用域或复合语句块。
- **L1232 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L1232 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Fortran parse tree handling / Fortran 语法树处理**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Semantic-context management / 语义上下文管理**
- **Scope and symbol resolution / 作用域与符号解析**

## Dependencies / 依赖关系

- `resolve-labels.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Common/enum-set.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Common/template.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Parser/parse-tree-visitor.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/semantics.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `cstdarg`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
- `type_traits`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
