# check-data.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-data.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: DATA statement semantic analysis. - Applies static semantic checks to the variables in each data-stmt-set with class DataVarChecker; - Invokes conversion of DATA statement values to static initializers.
- **Purpose (CN)**: 实现 check data 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===-- lib/Semantics/check-data.cpp --------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// DATA statement semantic analysis.
// - Applies static semantic checks to the variables in each data-stmt-set with
//   class DataVarChecker;
// - Invokes conversion of DATA statement values to static initializers

#include "check-data.h"
#include "data-to-inits.h"
#include "flang/Evaluate/traverse.h"
#include "flang/Parser/parse-tree.h"
#include "flang/Parser/tools.h"
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
- **L9 EN**: Comment explains nearby logic, intent, or metadata: `DATA statement semantic analysis.`.
  **L9 CN**: 注释说明附近代码的逻辑、意图或元数据：`DATA statement semantic analysis.`。
- **L10 EN**: Comment explains nearby logic, intent, or metadata: `- Applies static semantic checks to the variables in each data-stmt-set with`.
  **L10 CN**: 注释说明附近代码的逻辑、意图或元数据：`- Applies static semantic checks to the variables in each data-stmt-set with`。
- **L11 EN**: Comment explains nearby logic, intent, or metadata: `class DataVarChecker;`.
  **L11 CN**: 注释说明附近代码的逻辑、意图或元数据：`class DataVarChecker;`。
- **L12 EN**: Comment explains nearby logic, intent, or metadata: `- Invokes conversion of DATA statement values to static initializers`.
  **L12 CN**: 注释说明附近代码的逻辑、意图或元数据：`- Invokes conversion of DATA statement values to static initializers`。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L14 EN**: Includes "check-data.h" to access local declarations paired with this implementation.
  **L14 CN**: 引入 "check-data.h" 以使用与该实现配套的本地声明。
- **L15 EN**: Includes "data-to-inits.h" to access local declarations paired with this implementation.
  **L15 CN**: 引入 "data-to-inits.h" 以使用与该实现配套的本地声明。
- **L16 EN**: Includes "flang/Evaluate/traverse.h" to access Fortran constant-folding and evaluation facilities.
  **L16 CN**: 引入 "flang/Evaluate/traverse.h" 以使用Fortran 常量折叠与求值能力。
- **L17 EN**: Includes "flang/Parser/parse-tree.h" to access parse-tree, token, or source representation support.
  **L17 CN**: 引入 "flang/Parser/parse-tree.h" 以使用语法树、词法单元或源码表示支持。
- **L18 EN**: Includes "flang/Parser/tools.h" to access parse-tree, token, or source representation support.
  **L18 CN**: 引入 "flang/Parser/tools.h" 以使用语法树、词法单元或源码表示支持。

### Lines 19-36

````cpp
#include "flang/Semantics/tools.h"
#include <algorithm>
#include <vector>

namespace Fortran::semantics {

// Ensures that references to an implied DO loop control variable are
// represented as such in the "body" of the implied DO loop.
void DataChecker::Enter(const parser::DataImpliedDo &x) {
  const auto &name{parser::UnwrapRef<parser::Name>(
      std::get<parser::DataImpliedDo::Bounds>(x.t).Name())};
  int kind{evaluate::ResultType<evaluate::ImpliedDoIndex>::kind};
  if (const auto dynamicType{evaluate::DynamicType::From(DEREF(name.symbol))}) {
    if (dynamicType->category() == TypeCategory::Integer) {
      kind = dynamicType->kind();
    }
  }
  exprAnalyzer_.AddImpliedDo(name.source, kind);
````
- **L19 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L19 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L20 EN**: Includes <algorithm> to access supporting declarations used by this translation unit.
  **L20 CN**: 引入 <algorithm> 以使用当前编译单元使用的辅助声明。
- **L21 EN**: Includes <vector> to access supporting declarations used by this translation unit.
  **L21 CN**: 引入 <vector> 以使用当前编译单元使用的辅助声明。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L23 EN**: Opens namespace scope `Fortran::semantics`.
  **L23 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L25 EN**: Comment explains nearby logic, intent, or metadata: `Ensures that references to an implied DO loop control variable are`.
  **L25 CN**: 注释说明附近代码的逻辑、意图或元数据：`Ensures that references to an implied DO loop control variable are`。
- **L26 EN**: Comment explains nearby logic, intent, or metadata: `represented as such in the "body" of the implied DO loop.`.
  **L26 CN**: 注释说明附近代码的逻辑、意图或元数据：`represented as such in the "body" of the implied DO loop.`。
- **L27 EN**: Starts a function, method, lambda, or structured scope: `void DataChecker::Enter(const parser::DataImpliedDo &x) {`.
  **L27 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void DataChecker::Enter(const parser::DataImpliedDo &x) {`。
- **L28 EN**: Continues logic associated with callable symbol `Name>`.
  **L28 CN**: 继续与可调用符号 `Name>` 相关的逻辑。
- **L29 EN**: Executes a call or declaration centered on `std::get<parser::DataImpliedDo::Bounds>`.
  **L29 CN**: 执行以 `std::get<parser::DataImpliedDo::Bounds>` 为核心的调用或声明。
- **L30 EN**: Executes a standalone statement or declaration: `int kind{evaluate::ResultType<evaluate::ImpliedDoIndex>::kind};`.
  **L30 CN**: 执行一条独立语句或声明：`int kind{evaluate::ResultType<evaluate::ImpliedDoIndex>::kind};`。
- **L31 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L31 CN**: 开始 `if` 控制流语句并计算其条件。
- **L32 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L32 CN**: 开始 `if` 控制流语句并计算其条件。
- **L33 EN**: Executes a call or declaration centered on `dynamicType->kind`.
  **L33 CN**: 执行以 `dynamicType->kind` 为核心的调用或声明。
- **L34 EN**: Closes the current lexical scope or compound statement.
  **L34 CN**: 结束当前词法作用域或复合语句块。
- **L35 EN**: Closes the current lexical scope or compound statement.
  **L35 CN**: 结束当前词法作用域或复合语句块。
- **L36 EN**: Executes a call or declaration centered on `exprAnalyzer_.AddImpliedDo`.
  **L36 CN**: 执行以 `exprAnalyzer_.AddImpliedDo` 为核心的调用或声明。

### Lines 37-54

````cpp
}

void DataChecker::Leave(const parser::DataImpliedDo &x) {
  const auto &name{parser::UnwrapRef<parser::Name>(
      std::get<parser::DataImpliedDo::Bounds>(x.t).Name())};
  exprAnalyzer_.RemoveImpliedDo(name.source);
}

// DataVarChecker applies static checks once to each variable that appears
// in a data-stmt-set.  These checks are independent of the values that
// correspond to the variables.
class DataVarChecker : public evaluate::AllTraverse<DataVarChecker, true> {
public:
  using Base = evaluate::AllTraverse<DataVarChecker, true>;
  DataVarChecker(SemanticsContext &c, parser::CharBlock src)
      : Base{*this}, context_{c}, source_{src} {}
  using Base::operator();
  bool HasComponentWithoutSubscripts() const {
````
- **L37 EN**: Closes the current lexical scope or compound statement.
  **L37 CN**: 结束当前词法作用域或复合语句块。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L39 EN**: Starts a function, method, lambda, or structured scope: `void DataChecker::Leave(const parser::DataImpliedDo &x) {`.
  **L39 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void DataChecker::Leave(const parser::DataImpliedDo &x) {`。
- **L40 EN**: Continues logic associated with callable symbol `Name>`.
  **L40 CN**: 继续与可调用符号 `Name>` 相关的逻辑。
- **L41 EN**: Executes a call or declaration centered on `std::get<parser::DataImpliedDo::Bounds>`.
  **L41 CN**: 执行以 `std::get<parser::DataImpliedDo::Bounds>` 为核心的调用或声明。
- **L42 EN**: Executes a call or declaration centered on `exprAnalyzer_.RemoveImpliedDo`.
  **L42 CN**: 执行以 `exprAnalyzer_.RemoveImpliedDo` 为核心的调用或声明。
- **L43 EN**: Closes the current lexical scope or compound statement.
  **L43 CN**: 结束当前词法作用域或复合语句块。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L45 EN**: Comment explains nearby logic, intent, or metadata: `DataVarChecker applies static checks once to each variable that appears`.
  **L45 CN**: 注释说明附近代码的逻辑、意图或元数据：`DataVarChecker applies static checks once to each variable that appears`。
- **L46 EN**: Comment explains nearby logic, intent, or metadata: `in a data-stmt-set.  These checks are independent of the values that`.
  **L46 CN**: 注释说明附近代码的逻辑、意图或元数据：`in a data-stmt-set.  These checks are independent of the values that`。
- **L47 EN**: Comment explains nearby logic, intent, or metadata: `correspond to the variables.`.
  **L47 CN**: 注释说明附近代码的逻辑、意图或元数据：`correspond to the variables.`。
- **L48 EN**: Declares class `DataVarChecker`.
  **L48 CN**: 声明 class `DataVarChecker`。
- **L49 EN**: Sets the following members to `public` access.
  **L49 CN**: 将后续成员的访问级别设为 `public`。
- **L50 EN**: Defines alias `Base` to simplify later code.
  **L50 CN**: 定义别名 `Base` 以简化后续代码。
- **L51 EN**: Continues logic associated with callable symbol `DataVarChecker`.
  **L51 CN**: 继续与可调用符号 `DataVarChecker` 相关的逻辑。
- **L52 EN**: Continues the surrounding expression or declaration: `: Base{*this}, context_{c}, source_{src} {}`.
  **L52 CN**: 继续构造周围的表达式或声明：`: Base{*this}, context_{c}, source_{src} {}`。
- **L53 EN**: Executes a call or declaration centered on `Base::operator`.
  **L53 CN**: 执行以 `Base::operator` 为核心的调用或声明。
- **L54 EN**: Starts a function, method, lambda, or structured scope: `bool HasComponentWithoutSubscripts() const {`.
  **L54 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool HasComponentWithoutSubscripts() const {`。

### Lines 55-72

````cpp
    return hasComponent_ && !hasSubscript_;
  }
  bool operator()(const Symbol &symbol) { // C876
    // 8.6.7p(2) - precludes non-pointers of derived types with
    // default component values
    const Scope &scope{context_.FindScope(source_)};
    bool isFirstSymbol{isFirstSymbol_};
    isFirstSymbol_ = false;
    // Ordered so that most egregious errors are first
    if (const char *whyNot{IsProcedure(symbol) && !IsPointer(symbol)
                ? "Procedure"
                : isFirstSymbol && IsHostAssociated(symbol, scope)
                ? "Host-associated object"
                : isFirstSymbol && IsUseAssociated(symbol, scope)
                ? "USE-associated object"
                : IsDummy(symbol)          ? "Dummy argument"
                : IsFunctionResult(symbol) ? "Function result"
                : IsAutomatic(symbol)      ? "Automatic variable"
````
- **L55 EN**: Returns from the current function with `hasComponent_ && !hasSubscript_`.
  **L55 CN**: 以 `hasComponent_ && !hasSubscript_` 从当前函数返回。
- **L56 EN**: Closes the current lexical scope or compound statement.
  **L56 CN**: 结束当前词法作用域或复合语句块。
- **L57 EN**: Continues logic associated with callable symbol `operator`.
  **L57 CN**: 继续与可调用符号 `operator` 相关的逻辑。
- **L58 EN**: Comment explains nearby logic, intent, or metadata: `8.6.7p(2) - precludes non-pointers of derived types with`.
  **L58 CN**: 注释说明附近代码的逻辑、意图或元数据：`8.6.7p(2) - precludes non-pointers of derived types with`。
- **L59 EN**: Comment explains nearby logic, intent, or metadata: `default component values`.
  **L59 CN**: 注释说明附近代码的逻辑、意图或元数据：`default component values`。
- **L60 EN**: Executes a call or declaration centered on `&scope{context_.FindScope`.
  **L60 CN**: 执行以 `&scope{context_.FindScope` 为核心的调用或声明。
- **L61 EN**: Executes a standalone statement or declaration: `bool isFirstSymbol{isFirstSymbol_};`.
  **L61 CN**: 执行一条独立语句或声明：`bool isFirstSymbol{isFirstSymbol_};`。
- **L62 EN**: Executes a standalone statement or declaration: `isFirstSymbol_ = false;`.
  **L62 CN**: 执行一条独立语句或声明：`isFirstSymbol_ = false;`。
- **L63 EN**: Comment explains nearby logic, intent, or metadata: `Ordered so that most egregious errors are first`.
  **L63 CN**: 注释说明附近代码的逻辑、意图或元数据：`Ordered so that most egregious errors are first`。
- **L64 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L64 CN**: 开始 `if` 控制流语句并计算其条件。
- **L65 EN**: Continues the surrounding expression or declaration: `? "Procedure"`.
  **L65 CN**: 继续构造周围的表达式或声明：`? "Procedure"`。
- **L66 EN**: Continues logic associated with callable symbol `IsHostAssociated`.
  **L66 CN**: 继续与可调用符号 `IsHostAssociated` 相关的逻辑。
- **L67 EN**: Continues the surrounding expression or declaration: `? "Host-associated object"`.
  **L67 CN**: 继续构造周围的表达式或声明：`? "Host-associated object"`。
- **L68 EN**: Continues logic associated with callable symbol `IsUseAssociated`.
  **L68 CN**: 继续与可调用符号 `IsUseAssociated` 相关的逻辑。
- **L69 EN**: Continues the surrounding expression or declaration: `? "USE-associated object"`.
  **L69 CN**: 继续构造周围的表达式或声明：`? "USE-associated object"`。
- **L70 EN**: Continues logic associated with callable symbol `IsDummy`.
  **L70 CN**: 继续与可调用符号 `IsDummy` 相关的逻辑。
- **L71 EN**: Continues logic associated with callable symbol `IsFunctionResult`.
  **L71 CN**: 继续与可调用符号 `IsFunctionResult` 相关的逻辑。
- **L72 EN**: Continues logic associated with callable symbol `IsAutomatic`.
  **L72 CN**: 继续与可调用符号 `IsAutomatic` 相关的逻辑。

### Lines 73-90

````cpp
                : IsAllocatable(symbol)    ? "Allocatable"
                : IsInitialized(symbol, true /*ignore DATA*/,
                      true /*ignore allocatable components*/,
                      true /*ignore uninitialized pointer components*/)
                ? "Default-initialized"
                : symbol.has<AssocEntityDetails>() ? "Construct association"
                : isFirstSymbol && IsPointer(symbol) &&
                    (hasComponent_ || hasSubscript_)
                ? "Target of pointer"
                : nullptr}) {
      context_.Say(source_,
          "%s '%s' must not be initialized in a DATA statement"_err_en_US,
          whyNot, symbol.name());
      return false;
    }
    if (IsProcedurePointer(symbol)) {
      if (!context_.IsEnabled(common::LanguageFeature::DataStmtExtensions)) {
        context_.Say(source_,
````
- **L73 EN**: Continues logic associated with callable symbol `IsAllocatable`.
  **L73 CN**: 继续与可调用符号 `IsAllocatable` 相关的逻辑。
- **L74 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: IsInitialized(symbol, true /*ignore DATA*/,`.
  **L74 CN**: 继续一个多行参数列表、初始化器或聚合项：`: IsInitialized(symbol, true /*ignore DATA*/,`。
- **L75 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `true /*ignore allocatable components*/,`.
  **L75 CN**: 继续一个多行参数列表、初始化器或聚合项：`true /*ignore allocatable components*/,`。
- **L76 EN**: Continues the surrounding expression or declaration: `true /*ignore uninitialized pointer components*/)`.
  **L76 CN**: 继续构造周围的表达式或声明：`true /*ignore uninitialized pointer components*/)`。
- **L77 EN**: Continues the surrounding expression or declaration: `? "Default-initialized"`.
  **L77 CN**: 继续构造周围的表达式或声明：`? "Default-initialized"`。
- **L78 EN**: Continues logic associated with callable symbol `has<AssocEntityDetails>`.
  **L78 CN**: 继续与可调用符号 `has<AssocEntityDetails>` 相关的逻辑。
- **L79 EN**: Continues logic associated with callable symbol `IsPointer`.
  **L79 CN**: 继续与可调用符号 `IsPointer` 相关的逻辑。
- **L80 EN**: Continues the surrounding expression or declaration: `(hasComponent_ || hasSubscript_)`.
  **L80 CN**: 继续构造周围的表达式或声明：`(hasComponent_ || hasSubscript_)`。
- **L81 EN**: Continues the surrounding expression or declaration: `? "Target of pointer"`.
  **L81 CN**: 继续构造周围的表达式或声明：`? "Target of pointer"`。
- **L82 EN**: Continues the surrounding expression or declaration: `: nullptr}) {`.
  **L82 CN**: 继续构造周围的表达式或声明：`: nullptr}) {`。
- **L83 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(source_,`.
  **L83 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(source_,`。
- **L84 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s '%s' must not be initialized in a DATA statement"_err_en_US,`.
  **L84 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s '%s' must not be initialized in a DATA statement"_err_en_US,`。
- **L85 EN**: Executes a call or declaration centered on `symbol.name`.
  **L85 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L86 EN**: Returns from the current function with `false`.
  **L86 CN**: 以 `false` 从当前函数返回。
- **L87 EN**: Closes the current lexical scope or compound statement.
  **L87 CN**: 结束当前词法作用域或复合语句块。
- **L88 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L88 CN**: 开始 `if` 控制流语句并计算其条件。
- **L89 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L89 CN**: 开始 `if` 控制流语句并计算其条件。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(source_,`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(source_,`。

### Lines 91-108

````cpp
            "Procedure pointer '%s' may not appear in a DATA statement"_err_en_US,
            symbol.name());
        return false;
      } else {
        context_.Warn(common::LanguageFeature::DataStmtExtensions, source_,
            "Procedure pointer '%s' in a DATA statement is not standard"_port_en_US,
            symbol.name());
      }
    }
    if (IsInBlankCommon(symbol)) {
      if (!context_.IsEnabled(common::LanguageFeature::DataStmtExtensions)) {
        context_.Say(source_,
            "Blank COMMON object '%s' may not appear in a DATA statement"_err_en_US,
            symbol.name());
        return false;
      } else {
        context_.Warn(common::LanguageFeature::DataStmtExtensions, source_,
            "Blank COMMON object '%s' in a DATA statement is not standard"_port_en_US,
````
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Procedure pointer '%s' may not appear in a DATA statement"_err_en_US,`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Procedure pointer '%s' may not appear in a DATA statement"_err_en_US,`。
- **L92 EN**: Executes a call or declaration centered on `symbol.name`.
  **L92 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L93 EN**: Returns from the current function with `false`.
  **L93 CN**: 以 `false` 从当前函数返回。
- **L94 EN**: Transitions from the previous branch into the alternative path.
  **L94 CN**: 从前一个分支过渡到备选路径。
- **L95 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::LanguageFeature::DataStmtExtensions, source_,`.
  **L95 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::LanguageFeature::DataStmtExtensions, source_,`。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Procedure pointer '%s' in a DATA statement is not standard"_port_en_US,`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Procedure pointer '%s' in a DATA statement is not standard"_port_en_US,`。
- **L97 EN**: Executes a call or declaration centered on `symbol.name`.
  **L97 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L98 EN**: Closes the current lexical scope or compound statement.
  **L98 CN**: 结束当前词法作用域或复合语句块。
- **L99 EN**: Closes the current lexical scope or compound statement.
  **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L100 CN**: 开始 `if` 控制流语句并计算其条件。
- **L101 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L101 CN**: 开始 `if` 控制流语句并计算其条件。
- **L102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(source_,`.
  **L102 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(source_,`。
- **L103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Blank COMMON object '%s' may not appear in a DATA statement"_err_en_US,`.
  **L103 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Blank COMMON object '%s' may not appear in a DATA statement"_err_en_US,`。
- **L104 EN**: Executes a call or declaration centered on `symbol.name`.
  **L104 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L105 EN**: Returns from the current function with `false`.
  **L105 CN**: 以 `false` 从当前函数返回。
- **L106 EN**: Transitions from the previous branch into the alternative path.
  **L106 CN**: 从前一个分支过渡到备选路径。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::LanguageFeature::DataStmtExtensions, source_,`.
  **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::LanguageFeature::DataStmtExtensions, source_,`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Blank COMMON object '%s' in a DATA statement is not standard"_port_en_US,`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Blank COMMON object '%s' in a DATA statement is not standard"_port_en_US,`。

### Lines 109-126

````cpp
            symbol.name());
      }
    }
    return true;
  }
  bool operator()(const evaluate::Component &component) {
    hasComponent_ = true;
    const Symbol &lastSymbol{component.GetLastSymbol()};
    if (isPointerAllowed_) {
      if (IsPointer(lastSymbol) && hasSubscript_) { // C877
        context_.Say(source_,
            "Rightmost data object pointer '%s' must not be subscripted"_err_en_US,
            lastSymbol.name().ToString());
        return false;
      }
      auto restorer{common::ScopedSet(isPointerAllowed_, false)};
      return (*this)(component.base()) && (*this)(lastSymbol);
    } else if (IsPointer(lastSymbol)) { // C877
````
- **L109 EN**: Executes a call or declaration centered on `symbol.name`.
  **L109 CN**: 执行以 `symbol.name` 为核心的调用或声明。
- **L110 EN**: Closes the current lexical scope or compound statement.
  **L110 CN**: 结束当前词法作用域或复合语句块。
- **L111 EN**: Closes the current lexical scope or compound statement.
  **L111 CN**: 结束当前词法作用域或复合语句块。
- **L112 EN**: Returns from the current function with `true`.
  **L112 CN**: 以 `true` 从当前函数返回。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Starts a function, method, lambda, or structured scope: `bool operator()(const evaluate::Component &component) {`.
  **L114 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool operator()(const evaluate::Component &component) {`。
- **L115 EN**: Executes a standalone statement or declaration: `hasComponent_ = true;`.
  **L115 CN**: 执行一条独立语句或声明：`hasComponent_ = true;`。
- **L116 EN**: Executes a call or declaration centered on `&lastSymbol{component.GetLastSymbol`.
  **L116 CN**: 执行以 `&lastSymbol{component.GetLastSymbol` 为核心的调用或声明。
- **L117 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L117 CN**: 开始 `if` 控制流语句并计算其条件。
- **L118 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L118 CN**: 开始 `if` 控制流语句并计算其条件。
- **L119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(source_,`.
  **L119 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(source_,`。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Rightmost data object pointer '%s' must not be subscripted"_err_en_US,`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Rightmost data object pointer '%s' must not be subscripted"_err_en_US,`。
- **L121 EN**: Executes a call or declaration centered on `lastSymbol.name`.
  **L121 CN**: 执行以 `lastSymbol.name` 为核心的调用或声明。
- **L122 EN**: Returns from the current function with `false`.
  **L122 CN**: 以 `false` 从当前函数返回。
- **L123 EN**: Closes the current lexical scope or compound statement.
  **L123 CN**: 结束当前词法作用域或复合语句块。
- **L124 EN**: Executes a call or declaration centered on `restorer{common::ScopedSet`.
  **L124 CN**: 执行以 `restorer{common::ScopedSet` 为核心的调用或声明。
- **L125 EN**: Returns from the current function with `(*this)(component.base()) && (*this)(lastSymbol)`.
  **L125 CN**: 以 `(*this)(component.base()) && (*this)(lastSymbol)` 从当前函数返回。
- **L126 EN**: Transitions from the previous branch into an `else if` condition.
  **L126 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 127-144

````cpp
      context_.Say(source_,
          "Data object must not contain pointer '%s' as a non-rightmost part"_err_en_US,
          lastSymbol.name().ToString());
      return false;
    } else {
      return (*this)(component.base()) && (*this)(lastSymbol);
    }
  }
  bool operator()(const evaluate::ArrayRef &arrayRef) {
    hasSubscript_ = true;
    return (*this)(arrayRef.base()) && (*this)(arrayRef.subscript());
  }
  bool operator()(const evaluate::Substring &substring) {
    hasSubscript_ = true;
    return (*this)(substring.parent()) && (*this)(substring.lower()) &&
        (*this)(substring.upper());
  }
  bool operator()(const evaluate::CoarrayRef &) { // C874
````
- **L127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(source_,`.
  **L127 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(source_,`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Data object must not contain pointer '%s' as a non-rightmost part"_err_en_US,`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Data object must not contain pointer '%s' as a non-rightmost part"_err_en_US,`。
- **L129 EN**: Executes a call or declaration centered on `lastSymbol.name`.
  **L129 CN**: 执行以 `lastSymbol.name` 为核心的调用或声明。
- **L130 EN**: Returns from the current function with `false`.
  **L130 CN**: 以 `false` 从当前函数返回。
- **L131 EN**: Transitions from the previous branch into the alternative path.
  **L131 CN**: 从前一个分支过渡到备选路径。
- **L132 EN**: Returns from the current function with `(*this)(component.base()) && (*this)(lastSymbol)`.
  **L132 CN**: 以 `(*this)(component.base()) && (*this)(lastSymbol)` 从当前函数返回。
- **L133 EN**: Closes the current lexical scope or compound statement.
  **L133 CN**: 结束当前词法作用域或复合语句块。
- **L134 EN**: Closes the current lexical scope or compound statement.
  **L134 CN**: 结束当前词法作用域或复合语句块。
- **L135 EN**: Starts a function, method, lambda, or structured scope: `bool operator()(const evaluate::ArrayRef &arrayRef) {`.
  **L135 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool operator()(const evaluate::ArrayRef &arrayRef) {`。
- **L136 EN**: Executes a standalone statement or declaration: `hasSubscript_ = true;`.
  **L136 CN**: 执行一条独立语句或声明：`hasSubscript_ = true;`。
- **L137 EN**: Returns from the current function with `(*this)(arrayRef.base()) && (*this)(arrayRef.subscript())`.
  **L137 CN**: 以 `(*this)(arrayRef.base()) && (*this)(arrayRef.subscript())` 从当前函数返回。
- **L138 EN**: Closes the current lexical scope or compound statement.
  **L138 CN**: 结束当前词法作用域或复合语句块。
- **L139 EN**: Starts a function, method, lambda, or structured scope: `bool operator()(const evaluate::Substring &substring) {`.
  **L139 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool operator()(const evaluate::Substring &substring) {`。
- **L140 EN**: Executes a standalone statement or declaration: `hasSubscript_ = true;`.
  **L140 CN**: 执行一条独立语句或声明：`hasSubscript_ = true;`。
- **L141 EN**: Returns from the current function with `(*this)(substring.parent()) && (*this)(substring.lower()) &&`.
  **L141 CN**: 以 `(*this)(substring.parent()) && (*this)(substring.lower()) &&` 从当前函数返回。
- **L142 EN**: Executes a call or declaration centered on `statement`.
  **L142 CN**: 执行以 `statement` 为核心的调用或声明。
- **L143 EN**: Closes the current lexical scope or compound statement.
  **L143 CN**: 结束当前词法作用域或复合语句块。
- **L144 EN**: Continues logic associated with callable symbol `operator`.
  **L144 CN**: 继续与可调用符号 `operator` 相关的逻辑。

### Lines 145-162

````cpp
    context_.Say(
        source_, "Data object must not be a coindexed variable"_err_en_US);
    return false;
  }
  bool operator()(const evaluate::Subscript &subs) {
    auto restorer1{common::ScopedSet(isPointerAllowed_, false)};
    auto restorer2{common::ScopedSet(isFunctionAllowed_, true)};
    return common::visit(
        common::visitors{
            [&](const evaluate::IndirectSubscriptIntegerExpr &expr) {
              return CheckSubscriptExpr(expr);
            },
            [&](const evaluate::Triplet &triplet) {
              return CheckSubscriptExpr(triplet.lower()) &&
                  CheckSubscriptExpr(triplet.upper()) &&
                  CheckSubscriptExpr(triplet.stride());
            },
        },
````
- **L145 EN**: Continues logic associated with callable symbol `Say`.
  **L145 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L146 EN**: Executes a standalone statement or declaration: `source_, "Data object must not be a coindexed variable"_err_en_US);`.
  **L146 CN**: 执行一条独立语句或声明：`source_, "Data object must not be a coindexed variable"_err_en_US);`。
- **L147 EN**: Returns from the current function with `false`.
  **L147 CN**: 以 `false` 从当前函数返回。
- **L148 EN**: Closes the current lexical scope or compound statement.
  **L148 CN**: 结束当前词法作用域或复合语句块。
- **L149 EN**: Starts a function, method, lambda, or structured scope: `bool operator()(const evaluate::Subscript &subs) {`.
  **L149 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool operator()(const evaluate::Subscript &subs) {`。
- **L150 EN**: Executes a call or declaration centered on `restorer1{common::ScopedSet`.
  **L150 CN**: 执行以 `restorer1{common::ScopedSet` 为核心的调用或声明。
- **L151 EN**: Executes a call or declaration centered on `restorer2{common::ScopedSet`.
  **L151 CN**: 执行以 `restorer2{common::ScopedSet` 为核心的调用或声明。
- **L152 EN**: Returns from the current function with `common::visit(`.
  **L152 CN**: 以 `common::visit(` 从当前函数返回。
- **L153 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L153 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L154 EN**: Starts a function, method, lambda, or structured scope: `[&](const evaluate::IndirectSubscriptIntegerExpr &expr) {`.
  **L154 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const evaluate::IndirectSubscriptIntegerExpr &expr) {`。
- **L155 EN**: Returns from the current function with `CheckSubscriptExpr(expr)`.
  **L155 CN**: 以 `CheckSubscriptExpr(expr)` 从当前函数返回。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L157 EN**: Starts a function, method, lambda, or structured scope: `[&](const evaluate::Triplet &triplet) {`.
  **L157 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const evaluate::Triplet &triplet) {`。
- **L158 EN**: Returns from the current function with `CheckSubscriptExpr(triplet.lower()) &&`.
  **L158 CN**: 以 `CheckSubscriptExpr(triplet.lower()) &&` 从当前函数返回。
- **L159 EN**: Continues logic associated with callable symbol `CheckSubscriptExpr`.
  **L159 CN**: 继续与可调用符号 `CheckSubscriptExpr` 相关的逻辑。
- **L160 EN**: Executes a call or declaration centered on `CheckSubscriptExpr`.
  **L160 CN**: 执行以 `CheckSubscriptExpr` 为核心的调用或声明。
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。

### Lines 163-180

````cpp
        subs.u);
  }
  template <typename T>
  bool operator()(const evaluate::FunctionRef<T> &) const { // C875
    if (isFunctionAllowed_) {
      // Must have been validated as a constant expression
      return true;
    } else {
      context_.Say(source_,
          "Data object variable must not be a function reference"_err_en_US);
      return false;
    }
  }

private:
  bool CheckSubscriptExpr(
      const std::optional<evaluate::IndirectSubscriptIntegerExpr> &x) const {
    return !x || CheckSubscriptExpr(*x);
````
- **L163 EN**: Executes a standalone statement or declaration: `subs.u);`.
  **L163 CN**: 执行一条独立语句或声明：`subs.u);`。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Introduces template parameters or specialization context: `template <typename T>`.
  **L165 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>`。
- **L166 EN**: Continues logic associated with callable symbol `operator`.
  **L166 CN**: 继续与可调用符号 `operator` 相关的逻辑。
- **L167 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L167 CN**: 开始 `if` 控制流语句并计算其条件。
- **L168 EN**: Comment explains nearby logic, intent, or metadata: `Must have been validated as a constant expression`.
  **L168 CN**: 注释说明附近代码的逻辑、意图或元数据：`Must have been validated as a constant expression`。
- **L169 EN**: Returns from the current function with `true`.
  **L169 CN**: 以 `true` 从当前函数返回。
- **L170 EN**: Transitions from the previous branch into the alternative path.
  **L170 CN**: 从前一个分支过渡到备选路径。
- **L171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(source_,`.
  **L171 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(source_,`。
- **L172 EN**: Executes a standalone statement or declaration: `"Data object variable must not be a function reference"_err_en_US);`.
  **L172 CN**: 执行一条独立语句或声明：`"Data object variable must not be a function reference"_err_en_US);`。
- **L173 EN**: Returns from the current function with `false`.
  **L173 CN**: 以 `false` 从当前函数返回。
- **L174 EN**: Closes the current lexical scope or compound statement.
  **L174 CN**: 结束当前词法作用域或复合语句块。
- **L175 EN**: Closes the current lexical scope or compound statement.
  **L175 CN**: 结束当前词法作用域或复合语句块。
- **L176 EN**: Blank line separating nearby declarations or logic blocks.
  **L176 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L177 EN**: Sets the following members to `private` access.
  **L177 CN**: 将后续成员的访问级别设为 `private`。
- **L178 EN**: Continues logic associated with callable symbol `CheckSubscriptExpr`.
  **L178 CN**: 继续与可调用符号 `CheckSubscriptExpr` 相关的逻辑。
- **L179 EN**: Continues the surrounding expression or declaration: `const std::optional<evaluate::IndirectSubscriptIntegerExpr> &x) const {`.
  **L179 CN**: 继续构造周围的表达式或声明：`const std::optional<evaluate::IndirectSubscriptIntegerExpr> &x) const {`。
- **L180 EN**: Returns from the current function with `!x || CheckSubscriptExpr(*x)`.
  **L180 CN**: 以 `!x || CheckSubscriptExpr(*x)` 从当前函数返回。

### Lines 181-198

````cpp
  }
  bool CheckSubscriptExpr(
      const evaluate::IndirectSubscriptIntegerExpr &expr) const {
    return CheckSubscriptExpr(expr.value());
  }
  bool CheckSubscriptExpr(
      const evaluate::Expr<evaluate::SubscriptInteger> &expr) const {
    if (!evaluate::IsConstantExpr(expr, /*context=*/
            nullptr /* to accept unbound implied DO indices */)) { // C875,C881
      context_.Say(
          source_, "Data object must have constant subscripts"_err_en_US);
      return false;
    } else {
      return true;
    }
  }

  SemanticsContext &context_;
````
- **L181 EN**: Closes the current lexical scope or compound statement.
  **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Continues logic associated with callable symbol `CheckSubscriptExpr`.
  **L182 CN**: 继续与可调用符号 `CheckSubscriptExpr` 相关的逻辑。
- **L183 EN**: Continues the surrounding expression or declaration: `const evaluate::IndirectSubscriptIntegerExpr &expr) const {`.
  **L183 CN**: 继续构造周围的表达式或声明：`const evaluate::IndirectSubscriptIntegerExpr &expr) const {`。
- **L184 EN**: Returns from the current function with `CheckSubscriptExpr(expr.value())`.
  **L184 CN**: 以 `CheckSubscriptExpr(expr.value())` 从当前函数返回。
- **L185 EN**: Closes the current lexical scope or compound statement.
  **L185 CN**: 结束当前词法作用域或复合语句块。
- **L186 EN**: Continues logic associated with callable symbol `CheckSubscriptExpr`.
  **L186 CN**: 继续与可调用符号 `CheckSubscriptExpr` 相关的逻辑。
- **L187 EN**: Continues the surrounding expression or declaration: `const evaluate::Expr<evaluate::SubscriptInteger> &expr) const {`.
  **L187 CN**: 继续构造周围的表达式或声明：`const evaluate::Expr<evaluate::SubscriptInteger> &expr) const {`。
- **L188 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L188 CN**: 开始 `if` 控制流语句并计算其条件。
- **L189 EN**: Continues the surrounding expression or declaration: `nullptr /* to accept unbound implied DO indices */)) { // C875,C881`.
  **L189 CN**: 继续构造周围的表达式或声明：`nullptr /* to accept unbound implied DO indices */)) { // C875,C881`。
- **L190 EN**: Continues logic associated with callable symbol `Say`.
  **L190 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L191 EN**: Executes a standalone statement or declaration: `source_, "Data object must have constant subscripts"_err_en_US);`.
  **L191 CN**: 执行一条独立语句或声明：`source_, "Data object must have constant subscripts"_err_en_US);`。
- **L192 EN**: Returns from the current function with `false`.
  **L192 CN**: 以 `false` 从当前函数返回。
- **L193 EN**: Transitions from the previous branch into the alternative path.
  **L193 CN**: 从前一个分支过渡到备选路径。
- **L194 EN**: Returns from the current function with `true`.
  **L194 CN**: 以 `true` 从当前函数返回。
- **L195 EN**: Closes the current lexical scope or compound statement.
  **L195 CN**: 结束当前词法作用域或复合语句块。
- **L196 EN**: Closes the current lexical scope or compound statement.
  **L196 CN**: 结束当前词法作用域或复合语句块。
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L198 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L198 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。

### Lines 199-216

````cpp
  parser::CharBlock source_;
  bool hasComponent_{false};
  bool hasSubscript_{false};
  bool isPointerAllowed_{true};
  bool isFirstSymbol_{true};
  bool isFunctionAllowed_{false};
};

static bool IsValidDataObject(const SomeExpr &expr) { // C878, C879
  return !evaluate::IsConstantExpr(expr) &&
      (evaluate::IsVariable(expr) || evaluate::IsProcedurePointer(expr));
}

void DataChecker::Leave(const parser::DataIDoObject &object) {
  if (const auto *designator{
          std::get_if<parser::Scalar<common::Indirection<parser::Designator>>>(
              &object.u)}) {
    if (MaybeExpr expr{exprAnalyzer_.Analyze(*designator)}) {
````
- **L199 EN**: Executes a standalone statement or declaration: `parser::CharBlock source_;`.
  **L199 CN**: 执行一条独立语句或声明：`parser::CharBlock source_;`。
- **L200 EN**: Executes a standalone statement or declaration: `bool hasComponent_{false};`.
  **L200 CN**: 执行一条独立语句或声明：`bool hasComponent_{false};`。
- **L201 EN**: Executes a standalone statement or declaration: `bool hasSubscript_{false};`.
  **L201 CN**: 执行一条独立语句或声明：`bool hasSubscript_{false};`。
- **L202 EN**: Executes a standalone statement or declaration: `bool isPointerAllowed_{true};`.
  **L202 CN**: 执行一条独立语句或声明：`bool isPointerAllowed_{true};`。
- **L203 EN**: Executes a standalone statement or declaration: `bool isFirstSymbol_{true};`.
  **L203 CN**: 执行一条独立语句或声明：`bool isFirstSymbol_{true};`。
- **L204 EN**: Executes a standalone statement or declaration: `bool isFunctionAllowed_{false};`.
  **L204 CN**: 执行一条独立语句或声明：`bool isFunctionAllowed_{false};`。
- **L205 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L205 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L207 EN**: Continues logic associated with callable symbol `IsValidDataObject`.
  **L207 CN**: 继续与可调用符号 `IsValidDataObject` 相关的逻辑。
- **L208 EN**: Returns from the current function with `!evaluate::IsConstantExpr(expr) &&`.
  **L208 CN**: 以 `!evaluate::IsConstantExpr(expr) &&` 从当前函数返回。
- **L209 EN**: Executes a call or declaration centered on `statement`.
  **L209 CN**: 执行以 `statement` 为核心的调用或声明。
- **L210 EN**: Closes the current lexical scope or compound statement.
  **L210 CN**: 结束当前词法作用域或复合语句块。
- **L211 EN**: Blank line separating nearby declarations or logic blocks.
  **L211 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L212 EN**: Starts a function, method, lambda, or structured scope: `void DataChecker::Leave(const parser::DataIDoObject &object) {`.
  **L212 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void DataChecker::Leave(const parser::DataIDoObject &object) {`。
- **L213 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L213 CN**: 开始 `if` 控制流语句并计算其条件。
- **L214 EN**: Continues logic associated with callable symbol `Designator>>>`.
  **L214 CN**: 继续与可调用符号 `Designator>>>` 相关的逻辑。
- **L215 EN**: Continues the surrounding expression or declaration: `&object.u)}) {`.
  **L215 CN**: 继续构造周围的表达式或声明：`&object.u)}) {`。
- **L216 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L216 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 217-234

````cpp
      auto source{parser::UnwrapRef<parser::Designator>(*designator).source};
      DataVarChecker checker{exprAnalyzer_.context(), source};
      if (checker(*expr)) {
        if (checker.HasComponentWithoutSubscripts()) { // C880
          exprAnalyzer_.context().Say(source,
              "Data implied do structure component must be subscripted"_err_en_US);
        } else if (!IsValidDataObject(*expr)) {
          exprAnalyzer_.context().Say(
              source, "Data implied do object must be a variable"_err_en_US);
        } else {
          return;
        }
      }
    }
    currentSetHasFatalErrors_ = true;
  }
}

````
- **L217 EN**: Executes a call or declaration centered on `source{parser::UnwrapRef<parser::Designator>`.
  **L217 CN**: 执行以 `source{parser::UnwrapRef<parser::Designator>` 为核心的调用或声明。
- **L218 EN**: Executes a call or declaration centered on `checker{exprAnalyzer_.context`.
  **L218 CN**: 执行以 `checker{exprAnalyzer_.context` 为核心的调用或声明。
- **L219 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L219 CN**: 开始 `if` 控制流语句并计算其条件。
- **L220 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L220 CN**: 开始 `if` 控制流语句并计算其条件。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `exprAnalyzer_.context().Say(source,`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`exprAnalyzer_.context().Say(source,`。
- **L222 EN**: Executes a standalone statement or declaration: `"Data implied do structure component must be subscripted"_err_en_US);`.
  **L222 CN**: 执行一条独立语句或声明：`"Data implied do structure component must be subscripted"_err_en_US);`。
- **L223 EN**: Transitions from the previous branch into an `else if` condition.
  **L223 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L224 EN**: Continues logic associated with callable symbol `context`.
  **L224 CN**: 继续与可调用符号 `context` 相关的逻辑。
- **L225 EN**: Executes a standalone statement or declaration: `source, "Data implied do object must be a variable"_err_en_US);`.
  **L225 CN**: 执行一条独立语句或声明：`source, "Data implied do object must be a variable"_err_en_US);`。
- **L226 EN**: Transitions from the previous branch into the alternative path.
  **L226 CN**: 从前一个分支过渡到备选路径。
- **L227 EN**: Returns from the current function with `void`.
  **L227 CN**: 以 `void` 从当前函数返回。
- **L228 EN**: Closes the current lexical scope or compound statement.
  **L228 CN**: 结束当前词法作用域或复合语句块。
- **L229 EN**: Closes the current lexical scope or compound statement.
  **L229 CN**: 结束当前词法作用域或复合语句块。
- **L230 EN**: Closes the current lexical scope or compound statement.
  **L230 CN**: 结束当前词法作用域或复合语句块。
- **L231 EN**: Executes a standalone statement or declaration: `currentSetHasFatalErrors_ = true;`.
  **L231 CN**: 执行一条独立语句或声明：`currentSetHasFatalErrors_ = true;`。
- **L232 EN**: Closes the current lexical scope or compound statement.
  **L232 CN**: 结束当前词法作用域或复合语句块。
- **L233 EN**: Closes the current lexical scope or compound statement.
  **L233 CN**: 结束当前词法作用域或复合语句块。
- **L234 EN**: Blank line separating nearby declarations or logic blocks.
  **L234 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 235-252

````cpp
void DataChecker::Leave(const parser::DataStmtObject &dataObject) {
  common::visit(
      common::visitors{
          [](const parser::DataImpliedDo &) { // has own Enter()/Leave()
          },
          [&](const auto &var) {
            auto expr{exprAnalyzer_.Analyze(var)};
            auto source{parser::FindSourceLocation(dataObject)};
            if (!expr ||
                !DataVarChecker{exprAnalyzer_.context(), source}(*expr)) {
              currentSetHasFatalErrors_ = true;
            } else if (!IsValidDataObject(*expr)) {
              exprAnalyzer_.context().Say(
                  source, "Data statement object must be a variable"_err_en_US);
              currentSetHasFatalErrors_ = true;
            }
          },
      },
````
- **L235 EN**: Starts a function, method, lambda, or structured scope: `void DataChecker::Leave(const parser::DataStmtObject &dataObject) {`.
  **L235 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void DataChecker::Leave(const parser::DataStmtObject &dataObject) {`。
- **L236 EN**: Continues logic associated with callable symbol `visit`.
  **L236 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L237 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L237 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L238 EN**: Continues logic associated with callable symbol `Enter`.
  **L238 CN**: 继续与可调用符号 `Enter` 相关的逻辑。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L240 EN**: Starts a function, method, lambda, or structured scope: `[&](const auto &var) {`.
  **L240 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const auto &var) {`。
- **L241 EN**: Executes a call or declaration centered on `expr{exprAnalyzer_.Analyze`.
  **L241 CN**: 执行以 `expr{exprAnalyzer_.Analyze` 为核心的调用或声明。
- **L242 EN**: Executes a call or declaration centered on `source{parser::FindSourceLocation`.
  **L242 CN**: 执行以 `source{parser::FindSourceLocation` 为核心的调用或声明。
- **L243 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L243 CN**: 开始 `if` 控制流语句并计算其条件。
- **L244 EN**: Starts a function, method, lambda, or structured scope: `!DataVarChecker{exprAnalyzer_.context(), source}(*expr)) {`.
  **L244 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!DataVarChecker{exprAnalyzer_.context(), source}(*expr)) {`。
- **L245 EN**: Executes a standalone statement or declaration: `currentSetHasFatalErrors_ = true;`.
  **L245 CN**: 执行一条独立语句或声明：`currentSetHasFatalErrors_ = true;`。
- **L246 EN**: Transitions from the previous branch into an `else if` condition.
  **L246 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L247 EN**: Continues logic associated with callable symbol `context`.
  **L247 CN**: 继续与可调用符号 `context` 相关的逻辑。
- **L248 EN**: Executes a standalone statement or declaration: `source, "Data statement object must be a variable"_err_en_US);`.
  **L248 CN**: 执行一条独立语句或声明：`source, "Data statement object must be a variable"_err_en_US);`。
- **L249 EN**: Executes a standalone statement or declaration: `currentSetHasFatalErrors_ = true;`.
  **L249 CN**: 执行一条独立语句或声明：`currentSetHasFatalErrors_ = true;`。
- **L250 EN**: Closes the current lexical scope or compound statement.
  **L250 CN**: 结束当前词法作用域或复合语句块。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。

### Lines 253-270

````cpp
      dataObject.u);
}

void DataChecker::Leave(const parser::DataStmtSet &set) {
  if (!currentSetHasFatalErrors_) {
    AccumulateDataInitializations(inits_, exprAnalyzer_, set);
  }
  currentSetHasFatalErrors_ = false;
}

void DataChecker::Leave(const parser::EntityDecl &decl) {
  if (const auto &init{
          std::get<std::optional<parser::Initialization>>(decl.t)}) {
    const Symbol *name{std::get<parser::Name>(decl.t).symbol};
    const auto *list{
        std::get_if<std::list<common::Indirection<parser::DataStmtValue>>>(
            &init->u)};
    if (name && list) {
````
- **L253 EN**: Executes a standalone statement or declaration: `dataObject.u);`.
  **L253 CN**: 执行一条独立语句或声明：`dataObject.u);`。
- **L254 EN**: Closes the current lexical scope or compound statement.
  **L254 CN**: 结束当前词法作用域或复合语句块。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L256 EN**: Starts a function, method, lambda, or structured scope: `void DataChecker::Leave(const parser::DataStmtSet &set) {`.
  **L256 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void DataChecker::Leave(const parser::DataStmtSet &set) {`。
- **L257 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L257 CN**: 开始 `if` 控制流语句并计算其条件。
- **L258 EN**: Executes a call or declaration centered on `AccumulateDataInitializations`.
  **L258 CN**: 执行以 `AccumulateDataInitializations` 为核心的调用或声明。
- **L259 EN**: Closes the current lexical scope or compound statement.
  **L259 CN**: 结束当前词法作用域或复合语句块。
- **L260 EN**: Executes a standalone statement or declaration: `currentSetHasFatalErrors_ = false;`.
  **L260 CN**: 执行一条独立语句或声明：`currentSetHasFatalErrors_ = false;`。
- **L261 EN**: Closes the current lexical scope or compound statement.
  **L261 CN**: 结束当前词法作用域或复合语句块。
- **L262 EN**: Blank line separating nearby declarations or logic blocks.
  **L262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L263 EN**: Starts a function, method, lambda, or structured scope: `void DataChecker::Leave(const parser::EntityDecl &decl) {`.
  **L263 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void DataChecker::Leave(const parser::EntityDecl &decl) {`。
- **L264 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L264 CN**: 开始 `if` 控制流语句并计算其条件。
- **L265 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::optional<parser::Initialization>>(decl.t)}) {`.
  **L265 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::optional<parser::Initialization>>(decl.t)}) {`。
- **L266 EN**: Executes a call or declaration centered on `*name{std::get<parser::Name>`.
  **L266 CN**: 执行以 `*name{std::get<parser::Name>` 为核心的调用或声明。
- **L267 EN**: Continues the surrounding expression or declaration: `const auto *list{`.
  **L267 CN**: 继续构造周围的表达式或声明：`const auto *list{`。
- **L268 EN**: Continues logic associated with callable symbol `DataStmtValue>>>`.
  **L268 CN**: 继续与可调用符号 `DataStmtValue>>>` 相关的逻辑。
- **L269 EN**: Executes a standalone statement or declaration: `&init->u)};`.
  **L269 CN**: 执行一条独立语句或声明：`&init->u)};`。
- **L270 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L270 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 271-280

````cpp
      AccumulateDataInitializations(inits_, exprAnalyzer_, *name, *list);
    }
  }
}

void DataChecker::CompileDataInitializationsIntoInitializers() {
  ConvertToInitializers(inits_, exprAnalyzer_, /*forDerivedTypesOnly=*/false);
}

} // namespace Fortran::semantics
````
- **L271 EN**: Executes a call or declaration centered on `AccumulateDataInitializations`.
  **L271 CN**: 执行以 `AccumulateDataInitializations` 为核心的调用或声明。
- **L272 EN**: Closes the current lexical scope or compound statement.
  **L272 CN**: 结束当前词法作用域或复合语句块。
- **L273 EN**: Closes the current lexical scope or compound statement.
  **L273 CN**: 结束当前词法作用域或复合语句块。
- **L274 EN**: Closes the current lexical scope or compound statement.
  **L274 CN**: 结束当前词法作用域或复合语句块。
- **L275 EN**: Blank line separating nearby declarations or logic blocks.
  **L275 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L276 EN**: Starts a function, method, lambda, or structured scope: `void DataChecker::CompileDataInitializationsIntoInitializers() {`.
  **L276 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void DataChecker::CompileDataInitializationsIntoInitializers() {`。
- **L277 EN**: Executes a call or declaration centered on `ConvertToInitializers`.
  **L277 CN**: 执行以 `ConvertToInitializers` 为核心的调用或声明。
- **L278 EN**: Closes the current lexical scope or compound statement.
  **L278 CN**: 结束当前词法作用域或复合语句块。
- **L279 EN**: Blank line separating nearby declarations or logic blocks.
  **L279 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L280 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L280 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Fortran parse tree handling / Fortran 语法树处理**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Semantic-context management / 语义上下文管理**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **Rule-checking workflow / 规则检查流程**

## Dependencies / 依赖关系

- `check-data.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `data-to-inits.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Evaluate/traverse.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Parser/parse-tree.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/tools.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `algorithm`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
- `vector`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
