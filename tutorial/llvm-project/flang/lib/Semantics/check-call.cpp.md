# check-call.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-call.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements semantic checking, symbol reasoning, and language-rule enforcement for check call.
- **Purpose (CN)**: 实现 check call 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/Semantics/check-call.cpp --------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "check-call.h"
#include "definable.h"
#include "pointer-assignment.h"
#include "flang/Evaluate/characteristics.h"
#include "flang/Evaluate/check-expression.h"
#include "flang/Evaluate/fold-designator.h"
#include "flang/Evaluate/shape.h"
#include "flang/Evaluate/tools.h"
#include "flang/Parser/characters.h"
#include "flang/Parser/message.h"
#include "flang/Semantics/scope.h"
#include "flang/Semantics/tools.h"
#include "llvm/ADT/StringSet.h"
#include <map>
#include <string>

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
- **L9 EN**: Includes "check-call.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "check-call.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "definable.h" to access local declarations paired with this implementation.
  **L10 CN**: 引入 "definable.h" 以使用与该实现配套的本地声明。
- **L11 EN**: Includes "pointer-assignment.h" to access local declarations paired with this implementation.
  **L11 CN**: 引入 "pointer-assignment.h" 以使用与该实现配套的本地声明。
- **L12 EN**: Includes "flang/Evaluate/characteristics.h" to access Fortran constant-folding and evaluation facilities.
  **L12 CN**: 引入 "flang/Evaluate/characteristics.h" 以使用Fortran 常量折叠与求值能力。
- **L13 EN**: Includes "flang/Evaluate/check-expression.h" to access Fortran constant-folding and evaluation facilities.
  **L13 CN**: 引入 "flang/Evaluate/check-expression.h" 以使用Fortran 常量折叠与求值能力。
- **L14 EN**: Includes "flang/Evaluate/fold-designator.h" to access Fortran constant-folding and evaluation facilities.
  **L14 CN**: 引入 "flang/Evaluate/fold-designator.h" 以使用Fortran 常量折叠与求值能力。
- **L15 EN**: Includes "flang/Evaluate/shape.h" to access Fortran constant-folding and evaluation facilities.
  **L15 CN**: 引入 "flang/Evaluate/shape.h" 以使用Fortran 常量折叠与求值能力。
- **L16 EN**: Includes "flang/Evaluate/tools.h" to access Fortran constant-folding and evaluation facilities.
  **L16 CN**: 引入 "flang/Evaluate/tools.h" 以使用Fortran 常量折叠与求值能力。
- **L17 EN**: Includes "flang/Parser/characters.h" to access parse-tree, token, or source representation support.
  **L17 CN**: 引入 "flang/Parser/characters.h" 以使用语法树、词法单元或源码表示支持。
- **L18 EN**: Includes "flang/Parser/message.h" to access parse-tree, token, or source representation support.
  **L18 CN**: 引入 "flang/Parser/message.h" 以使用语法树、词法单元或源码表示支持。
- **L19 EN**: Includes "flang/Semantics/scope.h" to access Fortran semantic analysis, symbol, and type information.
  **L19 CN**: 引入 "flang/Semantics/scope.h" 以使用Fortran 语义分析、符号与类型信息。
- **L20 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L20 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L21 EN**: Includes "llvm/ADT/StringSet.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L21 CN**: 引入 "llvm/ADT/StringSet.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L22 EN**: Includes <map> to access supporting declarations used by this translation unit.
  **L22 CN**: 引入 <map> 以使用当前编译单元使用的辅助声明。
- **L23 EN**: Includes <string> to access supporting declarations used by this translation unit.
  **L23 CN**: 引入 <string> 以使用当前编译单元使用的辅助声明。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 25-48

````cpp
using namespace Fortran::parser::literals;
namespace characteristics = Fortran::evaluate::characteristics;

namespace Fortran::semantics {

static void CheckImplicitInterfaceArg(evaluate::ActualArgument &arg,
    parser::ContextualMessages &messages, SemanticsContext &context) {
  auto restorer{
      messages.SetLocation(arg.sourceLocation().value_or(messages.at()))};
  if (auto kw{arg.keyword()}) {
    messages.Say(*kw,
        "Keyword '%s=' may not appear in a reference to a procedure with an implicit interface"_err_en_US,
        *kw);
  }
  auto type{arg.GetType()};
  if (type) {
    if (type->IsAssumedType()) {
      messages.Say(
          "Assumed type actual argument requires an explicit interface"_err_en_US);
    } else if (type->IsUnlimitedPolymorphic()) {
      messages.Say(
          "Unlimited polymorphic actual argument requires an explicit interface"_err_en_US);
    } else if (const DerivedTypeSpec * derived{GetDerivedTypeSpec(type)}) {
      if (!derived->parameters().empty()) {
````
- **L25 EN**: Brings namespace `Fortran::parser::literals` into the local scope.
  **L25 CN**: 将命名空间 `Fortran::parser::literals` 引入当前作用域。
- **L26 EN**: Initializes variable `characteristics` from the right-hand expression.
  **L26 CN**: 使用右侧表达式初始化变量 `characteristics`。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Opens namespace scope `Fortran::semantics`.
  **L28 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckImplicitInterfaceArg(evaluate::ActualArgument &arg,`.
  **L30 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckImplicitInterfaceArg(evaluate::ActualArgument &arg,`。
- **L31 EN**: Continues the surrounding expression or declaration: `parser::ContextualMessages &messages, SemanticsContext &context) {`.
  **L31 CN**: 继续构造周围的表达式或声明：`parser::ContextualMessages &messages, SemanticsContext &context) {`。
- **L32 EN**: Continues the surrounding expression or declaration: `auto restorer{`.
  **L32 CN**: 继续构造周围的表达式或声明：`auto restorer{`。
- **L33 EN**: Executes a call or declaration centered on `messages.SetLocation`.
  **L33 CN**: 执行以 `messages.SetLocation` 为核心的调用或声明。
- **L34 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L34 CN**: 开始 `if` 控制流语句并计算其条件。
- **L35 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(*kw,`.
  **L35 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(*kw,`。
- **L36 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Keyword '%s=' may not appear in a reference to a procedure with an implicit interface"_err_en_US,`.
  **L36 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Keyword '%s=' may not appear in a reference to a procedure with an implicit interface"_err_en_US,`。
- **L37 EN**: Comment explains nearby logic, intent, or metadata: `kw);`.
  **L37 CN**: 注释说明附近代码的逻辑、意图或元数据：`kw);`。
- **L38 EN**: Closes the current lexical scope or compound statement.
  **L38 CN**: 结束当前词法作用域或复合语句块。
- **L39 EN**: Executes a call or declaration centered on `type{arg.GetType`.
  **L39 CN**: 执行以 `type{arg.GetType` 为核心的调用或声明。
- **L40 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L40 CN**: 开始 `if` 控制流语句并计算其条件。
- **L41 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L41 CN**: 开始 `if` 控制流语句并计算其条件。
- **L42 EN**: Continues logic associated with callable symbol `Say`.
  **L42 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L43 EN**: Executes a standalone statement or declaration: `"Assumed type actual argument requires an explicit interface"_err_en_US);`.
  **L43 CN**: 执行一条独立语句或声明：`"Assumed type actual argument requires an explicit interface"_err_en_US);`。
- **L44 EN**: Transitions from the previous branch into an `else if` condition.
  **L44 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L45 EN**: Continues logic associated with callable symbol `Say`.
  **L45 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L46 EN**: Executes a standalone statement or declaration: `"Unlimited polymorphic actual argument requires an explicit interface"_err_en_US);`.
  **L46 CN**: 执行一条独立语句或声明：`"Unlimited polymorphic actual argument requires an explicit interface"_err_en_US);`。
- **L47 EN**: Transitions from the previous branch into an `else if` condition.
  **L47 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L48 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L48 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 49-72

````cpp
        messages.Say(
            "Parameterized derived type actual argument requires an explicit interface"_err_en_US);
      }
    }
  }
  if (arg.isPercentVal() &&
      (!type || !type->IsLengthlessIntrinsicType() || arg.Rank() != 0)) {
    messages.Say(
        "%VAL argument must be a scalar numeric or logical expression"_err_en_US);
  }
  if (const auto *expr{arg.UnwrapExpr()}) {
    if (const Symbol *base{GetFirstSymbol(*expr)}) {
      context.NoteDefinedSymbol(GetAssociationRoot(*base));
    }
    if (IsBOZLiteral(*expr)) {
      messages.Say("BOZ argument %s requires an explicit interface"_err_en_US,
          expr->AsFortran());
    } else if (evaluate::IsNullPointerOrAllocatable(expr)) {
      messages.Say(
          "Null pointer argument '%s' requires an explicit interface"_err_en_US,
          expr->AsFortran());
    } else if (auto named{evaluate::ExtractNamedEntity(*expr)}) {
      const Symbol &resolved{ResolveAssociations(named->GetLastSymbol())};
      if (IsAssumedRank(resolved)) {
````
- **L49 EN**: Continues logic associated with callable symbol `Say`.
  **L49 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L50 EN**: Executes a standalone statement or declaration: `"Parameterized derived type actual argument requires an explicit interface"_err_en_US);`.
  **L50 CN**: 执行一条独立语句或声明：`"Parameterized derived type actual argument requires an explicit interface"_err_en_US);`。
- **L51 EN**: Closes the current lexical scope or compound statement.
  **L51 CN**: 结束当前词法作用域或复合语句块。
- **L52 EN**: Closes the current lexical scope or compound statement.
  **L52 CN**: 结束当前词法作用域或复合语句块。
- **L53 EN**: Closes the current lexical scope or compound statement.
  **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L54 CN**: 开始 `if` 控制流语句并计算其条件。
- **L55 EN**: Starts a function, method, lambda, or structured scope: `(!type || !type->IsLengthlessIntrinsicType() || arg.Rank() != 0)) {`.
  **L55 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(!type || !type->IsLengthlessIntrinsicType() || arg.Rank() != 0)) {`。
- **L56 EN**: Continues logic associated with callable symbol `Say`.
  **L56 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L57 EN**: Executes a standalone statement or declaration: `"%VAL argument must be a scalar numeric or logical expression"_err_en_US);`.
  **L57 CN**: 执行一条独立语句或声明：`"%VAL argument must be a scalar numeric or logical expression"_err_en_US);`。
- **L58 EN**: Closes the current lexical scope or compound statement.
  **L58 CN**: 结束当前词法作用域或复合语句块。
- **L59 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L59 CN**: 开始 `if` 控制流语句并计算其条件。
- **L60 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L60 CN**: 开始 `if` 控制流语句并计算其条件。
- **L61 EN**: Executes a call or declaration centered on `context.NoteDefinedSymbol`.
  **L61 CN**: 执行以 `context.NoteDefinedSymbol` 为核心的调用或声明。
- **L62 EN**: Closes the current lexical scope or compound statement.
  **L62 CN**: 结束当前词法作用域或复合语句块。
- **L63 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L63 CN**: 开始 `if` 控制流语句并计算其条件。
- **L64 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say("BOZ argument %s requires an explicit interface"_err_en_US,`.
  **L64 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say("BOZ argument %s requires an explicit interface"_err_en_US,`。
- **L65 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L65 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L66 EN**: Transitions from the previous branch into an `else if` condition.
  **L66 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L67 EN**: Continues logic associated with callable symbol `Say`.
  **L67 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L68 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Null pointer argument '%s' requires an explicit interface"_err_en_US,`.
  **L68 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Null pointer argument '%s' requires an explicit interface"_err_en_US,`。
- **L69 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L69 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L70 EN**: Transitions from the previous branch into an `else if` condition.
  **L70 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L71 EN**: Executes a call or declaration centered on `&resolved{ResolveAssociations`.
  **L71 CN**: 执行以 `&resolved{ResolveAssociations` 为核心的调用或声明。
- **L72 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L72 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 73-96

````cpp
        messages.Say(
            "Assumed rank argument '%s' requires an explicit interface"_err_en_US,
            expr->AsFortran());
      }
      const Symbol &symbol{GetAssociationRoot(resolved)};
      if (symbol.attrs().test(Attr::ASYNCHRONOUS)) {
        messages.Say(
            "ASYNCHRONOUS argument '%s' requires an explicit interface"_err_en_US,
            expr->AsFortran());
      }
      if (symbol.attrs().test(Attr::VOLATILE)) {
        messages.Say(
            "VOLATILE argument '%s' requires an explicit interface"_err_en_US,
            expr->AsFortran());
      }
      if (const auto *object{symbol.detailsIf<ObjectEntityDetails>()}) {
        if (object->cudaDataAttr()) {
          messages.Warn(/*inModuleFile=*/false, context.languageFeatures(),
              common::UsageWarning::CUDAUsage,
              "Actual argument '%s' with CUDA data attributes should be passed via an explicit interface"_warn_en_US,
              expr->AsFortran());
        }
      }
    } else if (auto argChars{characteristics::DummyArgument::FromActual(
````
- **L73 EN**: Continues logic associated with callable symbol `Say`.
  **L73 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L74 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed rank argument '%s' requires an explicit interface"_err_en_US,`.
  **L74 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed rank argument '%s' requires an explicit interface"_err_en_US,`。
- **L75 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L75 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L76 EN**: Closes the current lexical scope or compound statement.
  **L76 CN**: 结束当前词法作用域或复合语句块。
- **L77 EN**: Executes a call or declaration centered on `&symbol{GetAssociationRoot`.
  **L77 CN**: 执行以 `&symbol{GetAssociationRoot` 为核心的调用或声明。
- **L78 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L78 CN**: 开始 `if` 控制流语句并计算其条件。
- **L79 EN**: Continues logic associated with callable symbol `Say`.
  **L79 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L80 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ASYNCHRONOUS argument '%s' requires an explicit interface"_err_en_US,`.
  **L80 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ASYNCHRONOUS argument '%s' requires an explicit interface"_err_en_US,`。
- **L81 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L81 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L82 EN**: Closes the current lexical scope or compound statement.
  **L82 CN**: 结束当前词法作用域或复合语句块。
- **L83 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L83 CN**: 开始 `if` 控制流语句并计算其条件。
- **L84 EN**: Continues logic associated with callable symbol `Say`.
  **L84 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L85 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"VOLATILE argument '%s' requires an explicit interface"_err_en_US,`.
  **L85 CN**: 继续一个多行参数列表、初始化器或聚合项：`"VOLATILE argument '%s' requires an explicit interface"_err_en_US,`。
- **L86 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L86 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L87 EN**: Closes the current lexical scope or compound statement.
  **L87 CN**: 结束当前词法作用域或复合语句块。
- **L88 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L88 CN**: 开始 `if` 控制流语句并计算其条件。
- **L89 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L89 CN**: 开始 `if` 控制流语句并计算其条件。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Warn(/*inModuleFile=*/false, context.languageFeatures(),`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Warn(/*inModuleFile=*/false, context.languageFeatures(),`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::UsageWarning::CUDAUsage,`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::UsageWarning::CUDAUsage,`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument '%s' with CUDA data attributes should be passed via an explicit interface"_warn_en_US,`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument '%s' with CUDA data attributes should be passed via an explicit interface"_warn_en_US,`。
- **L93 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L93 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L94 EN**: Closes the current lexical scope or compound statement.
  **L94 CN**: 结束当前词法作用域或复合语句块。
- **L95 EN**: Closes the current lexical scope or compound statement.
  **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Transitions from the previous branch into an `else if` condition.
  **L96 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 97-120

````cpp
                   "actual argument", *expr, context.foldingContext(),
                   /*forImplicitInterface=*/true)}) {
      const auto *argProcDesignator{
          std::get_if<evaluate::ProcedureDesignator>(&expr->u)};
      if (const auto *argProcSymbol{
              argProcDesignator ? argProcDesignator->GetSymbol() : nullptr}) {
        if (!argChars->IsTypelessIntrinsicDummy() && argProcDesignator &&
            argProcDesignator->IsElemental()) { // C1533
          evaluate::SayWithDeclaration(messages, *argProcSymbol,
              "Non-intrinsic ELEMENTAL procedure '%s' may not be passed as an actual argument"_err_en_US,
              argProcSymbol->name());
        } else if (const auto *subp{argProcSymbol->GetUltimate()
                                        .detailsIf<SubprogramDetails>()}) {
          if (subp->stmtFunction()) {
            evaluate::SayWithDeclaration(messages, *argProcSymbol,
                "Statement function '%s' may not be passed as an actual argument"_err_en_US,
                argProcSymbol->name());
          }
        }
      }
    }
  }
}

````
- **L97 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"actual argument", *expr, context.foldingContext(),`.
  **L97 CN**: 继续一个多行参数列表、初始化器或聚合项：`"actual argument", *expr, context.foldingContext(),`。
- **L98 EN**: Comment explains nearby logic, intent, or metadata: `forImplicitInterface=*/true)}) {`.
  **L98 CN**: 注释说明附近代码的逻辑、意图或元数据：`forImplicitInterface=*/true)}) {`。
- **L99 EN**: Continues the surrounding expression or declaration: `const auto *argProcDesignator{`.
  **L99 CN**: 继续构造周围的表达式或声明：`const auto *argProcDesignator{`。
- **L100 EN**: Executes a call or declaration centered on `std::get_if<evaluate::ProcedureDesignator>`.
  **L100 CN**: 执行以 `std::get_if<evaluate::ProcedureDesignator>` 为核心的调用或声明。
- **L101 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L101 CN**: 开始 `if` 控制流语句并计算其条件。
- **L102 EN**: Starts a function, method, lambda, or structured scope: `argProcDesignator ? argProcDesignator->GetSymbol() : nullptr}) {`.
  **L102 CN**: 开始一个函数、方法、lambda 或结构化作用域：`argProcDesignator ? argProcDesignator->GetSymbol() : nullptr}) {`。
- **L103 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L103 CN**: 开始 `if` 控制流语句并计算其条件。
- **L104 EN**: Continues logic associated with callable symbol `IsElemental`.
  **L104 CN**: 继续与可调用符号 `IsElemental` 相关的逻辑。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, *argProcSymbol,`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, *argProcSymbol,`。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Non-intrinsic ELEMENTAL procedure '%s' may not be passed as an actual argument"_err_en_US,`.
  **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Non-intrinsic ELEMENTAL procedure '%s' may not be passed as an actual argument"_err_en_US,`。
- **L107 EN**: Executes a call or declaration centered on `argProcSymbol->name`.
  **L107 CN**: 执行以 `argProcSymbol->name` 为核心的调用或声明。
- **L108 EN**: Transitions from the previous branch into an `else if` condition.
  **L108 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L109 EN**: Starts a function, method, lambda, or structured scope: `.detailsIf<SubprogramDetails>()}) {`.
  **L109 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.detailsIf<SubprogramDetails>()}) {`。
- **L110 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L110 CN**: 开始 `if` 控制流语句并计算其条件。
- **L111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, *argProcSymbol,`.
  **L111 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, *argProcSymbol,`。
- **L112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Statement function '%s' may not be passed as an actual argument"_err_en_US,`.
  **L112 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Statement function '%s' may not be passed as an actual argument"_err_en_US,`。
- **L113 EN**: Executes a call or declaration centered on `argProcSymbol->name`.
  **L113 CN**: 执行以 `argProcSymbol->name` 为核心的调用或声明。
- **L114 EN**: Closes the current lexical scope or compound statement.
  **L114 CN**: 结束当前词法作用域或复合语句块。
- **L115 EN**: Closes the current lexical scope or compound statement.
  **L115 CN**: 结束当前词法作用域或复合语句块。
- **L116 EN**: Closes the current lexical scope or compound statement.
  **L116 CN**: 结束当前词法作用域或复合语句块。
- **L117 EN**: Closes the current lexical scope or compound statement.
  **L117 CN**: 结束当前词法作用域或复合语句块。
- **L118 EN**: Closes the current lexical scope or compound statement.
  **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Closes the current lexical scope or compound statement.
  **L119 CN**: 结束当前词法作用域或复合语句块。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 121-144

````cpp
// F'2023 15.5.2.12p1: "Sequence association only applies when the dummy
// argument is an explicit-shape or assumed-size array."
static bool CanAssociateWithStorageSequence(
    const characteristics::DummyDataObject &dummy) {
  return !dummy.type.attrs().test(
             characteristics::TypeAndShape::Attr::AssumedRank) &&
      !dummy.type.attrs().test(
          characteristics::TypeAndShape::Attr::AssumedShape) &&
      !dummy.attrs.test(characteristics::DummyDataObject::Attr::Allocatable) &&
      !dummy.attrs.test(characteristics::DummyDataObject::Attr::Pointer) &&
      dummy.type.corank() == 0;
}

// When a CHARACTER actual argument is known to be short,
// we extend it on the right with spaces and a warning if
// possible.  When it is long, and not required to be equal,
// the usage conforms to the standard and no warning is needed.
static void CheckCharacterActual(evaluate::Expr<evaluate::SomeType> &actual,
    const characteristics::DummyDataObject &dummy,
    characteristics::TypeAndShape &actualType, SemanticsContext &context,
    parser::ContextualMessages &messages, bool extentErrors,
    const std::string &dummyName) {
  if (dummy.type.type().category() == TypeCategory::Character &&
      actualType.type().category() == TypeCategory::Character &&
````
- **L121 EN**: Comment explains nearby logic, intent, or metadata: `F'2023 15.5.2.12p1: "Sequence association only applies when the dummy`.
  **L121 CN**: 注释说明附近代码的逻辑、意图或元数据：`F'2023 15.5.2.12p1: "Sequence association only applies when the dummy`。
- **L122 EN**: Comment explains nearby logic, intent, or metadata: `argument is an explicit-shape or assumed-size array."`.
  **L122 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument is an explicit-shape or assumed-size array."`。
- **L123 EN**: Continues logic associated with callable symbol `CanAssociateWithStorageSequence`.
  **L123 CN**: 继续与可调用符号 `CanAssociateWithStorageSequence` 相关的逻辑。
- **L124 EN**: Continues the surrounding expression or declaration: `const characteristics::DummyDataObject &dummy) {`.
  **L124 CN**: 继续构造周围的表达式或声明：`const characteristics::DummyDataObject &dummy) {`。
- **L125 EN**: Returns from the current function with `!dummy.type.attrs().test(`.
  **L125 CN**: 以 `!dummy.type.attrs().test(` 从当前函数返回。
- **L126 EN**: Continues the surrounding expression or declaration: `characteristics::TypeAndShape::Attr::AssumedRank) &&`.
  **L126 CN**: 继续构造周围的表达式或声明：`characteristics::TypeAndShape::Attr::AssumedRank) &&`。
- **L127 EN**: Continues logic associated with callable symbol `attrs`.
  **L127 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L128 EN**: Continues the surrounding expression or declaration: `characteristics::TypeAndShape::Attr::AssumedShape) &&`.
  **L128 CN**: 继续构造周围的表达式或声明：`characteristics::TypeAndShape::Attr::AssumedShape) &&`。
- **L129 EN**: Continues logic associated with callable symbol `test`.
  **L129 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L130 EN**: Continues logic associated with callable symbol `test`.
  **L130 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L131 EN**: Executes a call or declaration centered on `dummy.type.corank`.
  **L131 CN**: 执行以 `dummy.type.corank` 为核心的调用或声明。
- **L132 EN**: Closes the current lexical scope or compound statement.
  **L132 CN**: 结束当前词法作用域或复合语句块。
- **L133 EN**: Blank line separating nearby declarations or logic blocks.
  **L133 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L134 EN**: Comment explains nearby logic, intent, or metadata: `When a CHARACTER actual argument is known to be short,`.
  **L134 CN**: 注释说明附近代码的逻辑、意图或元数据：`When a CHARACTER actual argument is known to be short,`。
- **L135 EN**: Comment explains nearby logic, intent, or metadata: `we extend it on the right with spaces and a warning if`.
  **L135 CN**: 注释说明附近代码的逻辑、意图或元数据：`we extend it on the right with spaces and a warning if`。
- **L136 EN**: Comment explains nearby logic, intent, or metadata: `possible.  When it is long, and not required to be equal,`.
  **L136 CN**: 注释说明附近代码的逻辑、意图或元数据：`possible.  When it is long, and not required to be equal,`。
- **L137 EN**: Comment explains nearby logic, intent, or metadata: `the usage conforms to the standard and no warning is needed.`.
  **L137 CN**: 注释说明附近代码的逻辑、意图或元数据：`the usage conforms to the standard and no warning is needed.`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckCharacterActual(evaluate::Expr<evaluate::SomeType> &actual,`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckCharacterActual(evaluate::Expr<evaluate::SomeType> &actual,`。
- **L139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const characteristics::DummyDataObject &dummy,`.
  **L139 CN**: 继续一个多行参数列表、初始化器或聚合项：`const characteristics::DummyDataObject &dummy,`。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::TypeAndShape &actualType, SemanticsContext &context,`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::TypeAndShape &actualType, SemanticsContext &context,`。
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ContextualMessages &messages, bool extentErrors,`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ContextualMessages &messages, bool extentErrors,`。
- **L142 EN**: Continues the surrounding expression or declaration: `const std::string &dummyName) {`.
  **L142 CN**: 继续构造周围的表达式或声明：`const std::string &dummyName) {`。
- **L143 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L143 CN**: 开始 `if` 控制流语句并计算其条件。
- **L144 EN**: Continues logic associated with callable symbol `type`.
  **L144 CN**: 继续与可调用符号 `type` 相关的逻辑。

### Lines 145-168

````cpp
      dummy.type.type().kind() == actualType.type().kind() &&
      !dummy.attrs.test(
          characteristics::DummyDataObject::Attr::DeducedFromActual)) {
    bool actualIsAssumedRank{IsAssumedRank(actual)};
    if (actualIsAssumedRank &&
        !dummy.type.attrs().test(
            characteristics::TypeAndShape::Attr::AssumedRank)) {
      if (!context.languageFeatures().IsEnabled(
              common::LanguageFeature::AssumedRankPassedToNonAssumedRank)) {
        messages.Say(
            "Assumed-rank character array may not be associated with a dummy argument that is not assumed-rank"_err_en_US);
      } else {
        context.Warn(messages,
            common::LanguageFeature::AssumedRankPassedToNonAssumedRank,
            messages.at(),
            "Assumed-rank character array should not be associated with a dummy argument that is not assumed-rank"_port_en_US);
      }
    }
    if (dummy.type.LEN() && actualType.LEN()) {
      evaluate::FoldingContext &foldingContext{context.foldingContext()};
      auto dummyLength{
          ToInt64(Fold(foldingContext, common::Clone(*dummy.type.LEN())))};
      auto actualLength{
          ToInt64(Fold(foldingContext, common::Clone(*actualType.LEN())))};
````
- **L145 EN**: Continues logic associated with callable symbol `type`.
  **L145 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L146 EN**: Continues logic associated with callable symbol `test`.
  **L146 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L147 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject::Attr::DeducedFromActual)) {`.
  **L147 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject::Attr::DeducedFromActual)) {`。
- **L148 EN**: Executes a call or declaration centered on `actualIsAssumedRank{IsAssumedRank`.
  **L148 CN**: 执行以 `actualIsAssumedRank{IsAssumedRank` 为核心的调用或声明。
- **L149 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L149 CN**: 开始 `if` 控制流语句并计算其条件。
- **L150 EN**: Continues logic associated with callable symbol `attrs`.
  **L150 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L151 EN**: Continues the surrounding expression or declaration: `characteristics::TypeAndShape::Attr::AssumedRank)) {`.
  **L151 CN**: 继续构造周围的表达式或声明：`characteristics::TypeAndShape::Attr::AssumedRank)) {`。
- **L152 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L152 CN**: 开始 `if` 控制流语句并计算其条件。
- **L153 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::AssumedRankPassedToNonAssumedRank)) {`.
  **L153 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::AssumedRankPassedToNonAssumedRank)) {`。
- **L154 EN**: Continues logic associated with callable symbol `Say`.
  **L154 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L155 EN**: Executes a standalone statement or declaration: `"Assumed-rank character array may not be associated with a dummy argument that is not assumed-rank"_err_en_US);`.
  **L155 CN**: 执行一条独立语句或声明：`"Assumed-rank character array may not be associated with a dummy argument that is not assumed-rank"_err_en_US);`。
- **L156 EN**: Transitions from the previous branch into the alternative path.
  **L156 CN**: 从前一个分支过渡到备选路径。
- **L157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(messages,`.
  **L157 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(messages,`。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::LanguageFeature::AssumedRankPassedToNonAssumedRank,`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::LanguageFeature::AssumedRankPassedToNonAssumedRank,`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.at(),`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.at(),`。
- **L160 EN**: Executes a standalone statement or declaration: `"Assumed-rank character array should not be associated with a dummy argument that is not assumed-rank"_port_en_US);`.
  **L160 CN**: 执行一条独立语句或声明：`"Assumed-rank character array should not be associated with a dummy argument that is not assumed-rank"_port_en_US);`。
- **L161 EN**: Closes the current lexical scope or compound statement.
  **L161 CN**: 结束当前词法作用域或复合语句块。
- **L162 EN**: Closes the current lexical scope or compound statement.
  **L162 CN**: 结束当前词法作用域或复合语句块。
- **L163 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L163 CN**: 开始 `if` 控制流语句并计算其条件。
- **L164 EN**: Executes a call or declaration centered on `&foldingContext{context.foldingContext`.
  **L164 CN**: 执行以 `&foldingContext{context.foldingContext` 为核心的调用或声明。
- **L165 EN**: Continues the surrounding expression or declaration: `auto dummyLength{`.
  **L165 CN**: 继续构造周围的表达式或声明：`auto dummyLength{`。
- **L166 EN**: Executes a call or declaration centered on `ToInt64`.
  **L166 CN**: 执行以 `ToInt64` 为核心的调用或声明。
- **L167 EN**: Continues the surrounding expression or declaration: `auto actualLength{`.
  **L167 CN**: 继续构造周围的表达式或声明：`auto actualLength{`。
- **L168 EN**: Executes a call or declaration centered on `ToInt64`.
  **L168 CN**: 执行以 `ToInt64` 为核心的调用或声明。

### Lines 169-192

````cpp
      if (dummyLength && actualLength) {
        bool canAssociate{CanAssociateWithStorageSequence(dummy)};
        if (dummy.type.Rank() > 0 && canAssociate) {
          // Character storage sequence association (F'2023 15.5.2.12p4)
          if (auto dummySize{evaluate::ToInt64(evaluate::Fold(
                  foldingContext, evaluate::GetSize(dummy.type.shape())))}) {
            auto dummyChars{*dummySize * *dummyLength};
            if (actualType.Rank() == 0 && !actualIsAssumedRank) {
              evaluate::DesignatorFolder folder{
                  context.foldingContext(), /*getLastComponent=*/true};
              if (auto actualOffset{folder.FoldDesignator(actual)}) {
                std::int64_t actualChars{*actualLength};
                if (IsAllocatableOrPointer(actualOffset->symbol())) {
                  // don't use actualOffset->symbol().size()!
                } else if (static_cast<std::size_t>(actualOffset->offset()) >=
                        actualOffset->symbol().size() ||
                    !evaluate::IsContiguous(
                        actualOffset->symbol(), foldingContext)
                        .value_or(false)) {
                  // If substring, take rest of substring
                  if (*actualLength > 0) {
                    actualChars -=
                        (actualOffset->offset() / actualType.type().kind()) %
                        *actualLength;
````
- **L169 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L169 CN**: 开始 `if` 控制流语句并计算其条件。
- **L170 EN**: Executes a call or declaration centered on `canAssociate{CanAssociateWithStorageSequence`.
  **L170 CN**: 执行以 `canAssociate{CanAssociateWithStorageSequence` 为核心的调用或声明。
- **L171 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L171 CN**: 开始 `if` 控制流语句并计算其条件。
- **L172 EN**: Comment explains nearby logic, intent, or metadata: `Character storage sequence association (F'2023 15.5.2.12p4)`.
  **L172 CN**: 注释说明附近代码的逻辑、意图或元数据：`Character storage sequence association (F'2023 15.5.2.12p4)`。
- **L173 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L173 CN**: 开始 `if` 控制流语句并计算其条件。
- **L174 EN**: Starts a function, method, lambda, or structured scope: `foldingContext, evaluate::GetSize(dummy.type.shape())))}) {`.
  **L174 CN**: 开始一个函数、方法、lambda 或结构化作用域：`foldingContext, evaluate::GetSize(dummy.type.shape())))}) {`。
- **L175 EN**: Executes a standalone statement or declaration: `auto dummyChars{*dummySize * *dummyLength};`.
  **L175 CN**: 执行一条独立语句或声明：`auto dummyChars{*dummySize * *dummyLength};`。
- **L176 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L176 CN**: 开始 `if` 控制流语句并计算其条件。
- **L177 EN**: Continues the surrounding expression or declaration: `evaluate::DesignatorFolder folder{`.
  **L177 CN**: 继续构造周围的表达式或声明：`evaluate::DesignatorFolder folder{`。
- **L178 EN**: Executes a call or declaration centered on `context.foldingContext`.
  **L178 CN**: 执行以 `context.foldingContext` 为核心的调用或声明。
- **L179 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L179 CN**: 开始 `if` 控制流语句并计算其条件。
- **L180 EN**: Executes a standalone statement or declaration: `std::int64_t actualChars{*actualLength};`.
  **L180 CN**: 执行一条独立语句或声明：`std::int64_t actualChars{*actualLength};`。
- **L181 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L181 CN**: 开始 `if` 控制流语句并计算其条件。
- **L182 EN**: Comment explains nearby logic, intent, or metadata: `don't use actualOffset->symbol().size()!`.
  **L182 CN**: 注释说明附近代码的逻辑、意图或元数据：`don't use actualOffset->symbol().size()!`。
- **L183 EN**: Transitions from the previous branch into an `else if` condition.
  **L183 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L184 EN**: Continues logic associated with callable symbol `symbol`.
  **L184 CN**: 继续与可调用符号 `symbol` 相关的逻辑。
- **L185 EN**: Continues logic associated with callable symbol `IsContiguous`.
  **L185 CN**: 继续与可调用符号 `IsContiguous` 相关的逻辑。
- **L186 EN**: Continues logic associated with callable symbol `symbol`.
  **L186 CN**: 继续与可调用符号 `symbol` 相关的逻辑。
- **L187 EN**: Starts a function, method, lambda, or structured scope: `.value_or(false)) {`.
  **L187 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.value_or(false)) {`。
- **L188 EN**: Comment explains nearby logic, intent, or metadata: `If substring, take rest of substring`.
  **L188 CN**: 注释说明附近代码的逻辑、意图或元数据：`If substring, take rest of substring`。
- **L189 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L189 CN**: 开始 `if` 控制流语句并计算其条件。
- **L190 EN**: Continues the surrounding expression or declaration: `actualChars -=`.
  **L190 CN**: 继续构造周围的表达式或声明：`actualChars -=`。
- **L191 EN**: Continues logic associated with callable symbol `offset`.
  **L191 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L192 EN**: Comment explains nearby logic, intent, or metadata: `actualLength;`.
  **L192 CN**: 注释说明附近代码的逻辑、意图或元数据：`actualLength;`。

### Lines 193-216

````cpp
                  }
                } else {
                  actualChars = (static_cast<std::int64_t>(
                                     actualOffset->symbol().size()) -
                                    actualOffset->offset()) /
                      actualType.type().kind();
                }
                if (actualChars < dummyChars) {
                  if (extentErrors) {
                    messages.Say(
                        "Actual argument has fewer characters remaining in storage sequence (%jd) than %s (%jd)"_err_en_US,
                        static_cast<std::intmax_t>(actualChars), dummyName,
                        static_cast<std::intmax_t>(dummyChars));
                  } else {
                    context.Warn(messages,
                        common::UsageWarning::ShortCharacterActual,
                        "Actual argument has fewer characters remaining in storage sequence (%jd) than %s (%jd)"_warn_en_US,
                        static_cast<std::intmax_t>(actualChars), dummyName,
                        static_cast<std::intmax_t>(dummyChars));
                  }
                }
              }
            } else { // actual.type.Rank() > 0
              if (auto actualSize{evaluate::ToInt64(evaluate::Fold(
````
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Transitions from the previous branch into the alternative path.
  **L194 CN**: 从前一个分支过渡到备选路径。
- **L195 EN**: Continues logic associated with callable symbol `int64_t>`.
  **L195 CN**: 继续与可调用符号 `int64_t>` 相关的逻辑。
- **L196 EN**: Continues logic associated with callable symbol `symbol`.
  **L196 CN**: 继续与可调用符号 `symbol` 相关的逻辑。
- **L197 EN**: Continues logic associated with callable symbol `offset`.
  **L197 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L198 EN**: Executes a call or declaration centered on `actualType.type`.
  **L198 CN**: 执行以 `actualType.type` 为核心的调用或声明。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L200 CN**: 开始 `if` 控制流语句并计算其条件。
- **L201 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L201 CN**: 开始 `if` 控制流语句并计算其条件。
- **L202 EN**: Continues logic associated with callable symbol `Say`.
  **L202 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument has fewer characters remaining in storage sequence (%jd) than %s (%jd)"_err_en_US,`.
  **L203 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument has fewer characters remaining in storage sequence (%jd) than %s (%jd)"_err_en_US,`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<std::intmax_t>(actualChars), dummyName,`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<std::intmax_t>(actualChars), dummyName,`。
- **L205 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L205 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L206 EN**: Transitions from the previous branch into the alternative path.
  **L206 CN**: 从前一个分支过渡到备选路径。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(messages,`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(messages,`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::UsageWarning::ShortCharacterActual,`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::UsageWarning::ShortCharacterActual,`。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument has fewer characters remaining in storage sequence (%jd) than %s (%jd)"_warn_en_US,`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument has fewer characters remaining in storage sequence (%jd) than %s (%jd)"_warn_en_US,`。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<std::intmax_t>(actualChars), dummyName,`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<std::intmax_t>(actualChars), dummyName,`。
- **L211 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L211 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L212 EN**: Closes the current lexical scope or compound statement.
  **L212 CN**: 结束当前词法作用域或复合语句块。
- **L213 EN**: Closes the current lexical scope or compound statement.
  **L213 CN**: 结束当前词法作用域或复合语句块。
- **L214 EN**: Closes the current lexical scope or compound statement.
  **L214 CN**: 结束当前词法作用域或复合语句块。
- **L215 EN**: Transitions from the previous branch into the alternative path.
  **L215 CN**: 从前一个分支过渡到备选路径。
- **L216 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L216 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 217-240

````cpp
                      foldingContext, evaluate::GetSize(actualType.shape())))};
                  actualSize &&
                  *actualSize * *actualLength < *dummySize * *dummyLength) {
                if (extentErrors) {
                  messages.Say(
                      "Actual argument array has fewer characters (%jd) than %s array (%jd)"_err_en_US,
                      static_cast<std::intmax_t>(*actualSize * *actualLength),
                      dummyName,
                      static_cast<std::intmax_t>(*dummySize * *dummyLength));
                } else {
                  context.Warn(messages,
                      common::UsageWarning::ShortCharacterActual,
                      "Actual argument array has fewer characters (%jd) than %s array (%jd)"_warn_en_US,
                      static_cast<std::intmax_t>(*actualSize * *actualLength),
                      dummyName,
                      static_cast<std::intmax_t>(*dummySize * *dummyLength));
                }
              }
            }
          }
        } else if (*actualLength != *dummyLength) {
          // Not using storage sequence association, and the lengths don't
          // match.
          if (!canAssociate) {
````
- **L217 EN**: Executes a call or declaration centered on `evaluate::GetSize`.
  **L217 CN**: 执行以 `evaluate::GetSize` 为核心的调用或声明。
- **L218 EN**: Continues the surrounding expression or declaration: `actualSize &&`.
  **L218 CN**: 继续构造周围的表达式或声明：`actualSize &&`。
- **L219 EN**: Comment explains nearby logic, intent, or metadata: `actualSize * *actualLength < *dummySize * *dummyLength) {`.
  **L219 CN**: 注释说明附近代码的逻辑、意图或元数据：`actualSize * *actualLength < *dummySize * *dummyLength) {`。
- **L220 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L220 CN**: 开始 `if` 控制流语句并计算其条件。
- **L221 EN**: Continues logic associated with callable symbol `Say`.
  **L221 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument array has fewer characters (%jd) than %s array (%jd)"_err_en_US,`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument array has fewer characters (%jd) than %s array (%jd)"_err_en_US,`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<std::intmax_t>(*actualSize * *actualLength),`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<std::intmax_t>(*actualSize * *actualLength),`。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dummyName,`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`dummyName,`。
- **L225 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L225 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L226 EN**: Transitions from the previous branch into the alternative path.
  **L226 CN**: 从前一个分支过渡到备选路径。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(messages,`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(messages,`。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::UsageWarning::ShortCharacterActual,`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::UsageWarning::ShortCharacterActual,`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument array has fewer characters (%jd) than %s array (%jd)"_warn_en_US,`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument array has fewer characters (%jd) than %s array (%jd)"_warn_en_US,`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<std::intmax_t>(*actualSize * *actualLength),`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<std::intmax_t>(*actualSize * *actualLength),`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dummyName,`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`dummyName,`。
- **L232 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L232 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L233 EN**: Closes the current lexical scope or compound statement.
  **L233 CN**: 结束当前词法作用域或复合语句块。
- **L234 EN**: Closes the current lexical scope or compound statement.
  **L234 CN**: 结束当前词法作用域或复合语句块。
- **L235 EN**: Closes the current lexical scope or compound statement.
  **L235 CN**: 结束当前词法作用域或复合语句块。
- **L236 EN**: Closes the current lexical scope or compound statement.
  **L236 CN**: 结束当前词法作用域或复合语句块。
- **L237 EN**: Transitions from the previous branch into an `else if` condition.
  **L237 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L238 EN**: Comment explains nearby logic, intent, or metadata: `Not using storage sequence association, and the lengths don't`.
  **L238 CN**: 注释说明附近代码的逻辑、意图或元数据：`Not using storage sequence association, and the lengths don't`。
- **L239 EN**: Comment explains nearby logic, intent, or metadata: `match.`.
  **L239 CN**: 注释说明附近代码的逻辑、意图或元数据：`match.`。
- **L240 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L240 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 241-264

````cpp
            // F'2023 15.5.2.5 paragraph 4
            messages.Say(
                "Actual argument variable length '%jd' does not match the expected length '%jd'"_err_en_US,
                *actualLength, *dummyLength);
          } else if (*actualLength < *dummyLength) {
            CHECK(dummy.type.Rank() == 0);
            bool isVariable{evaluate::IsVariable(actual)};
            if (isVariable) {
              context.Warn(messages, common::UsageWarning::ShortCharacterActual,
                  "Actual argument variable length '%jd' is less than expected length '%jd'"_warn_en_US,
                  *actualLength, *dummyLength);
            } else {
              context.Warn(messages, common::UsageWarning::ShortCharacterActual,
                  "Actual argument expression length '%jd' is less than expected length '%jd'"_warn_en_US,
                  *actualLength, *dummyLength);
            }
            if (!isVariable) {
              auto converted{
                  ConvertToType(dummy.type.type(), std::move(actual))};
              CHECK(converted);
              actual = std::move(*converted);
              actualType.set_LEN(SubscriptIntExpr{*dummyLength});
            }
          }
````
- **L241 EN**: Comment explains nearby logic, intent, or metadata: `F'2023 15.5.2.5 paragraph 4`.
  **L241 CN**: 注释说明附近代码的逻辑、意图或元数据：`F'2023 15.5.2.5 paragraph 4`。
- **L242 EN**: Continues logic associated with callable symbol `Say`.
  **L242 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument variable length '%jd' does not match the expected length '%jd'"_err_en_US,`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument variable length '%jd' does not match the expected length '%jd'"_err_en_US,`。
- **L244 EN**: Comment explains nearby logic, intent, or metadata: `actualLength, *dummyLength);`.
  **L244 CN**: 注释说明附近代码的逻辑、意图或元数据：`actualLength, *dummyLength);`。
- **L245 EN**: Transitions from the previous branch into an `else if` condition.
  **L245 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L246 EN**: Executes a call or declaration centered on `CHECK`.
  **L246 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L247 EN**: Executes a call or declaration centered on `isVariable{evaluate::IsVariable`.
  **L247 CN**: 执行以 `isVariable{evaluate::IsVariable` 为核心的调用或声明。
- **L248 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L248 CN**: 开始 `if` 控制流语句并计算其条件。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(messages, common::UsageWarning::ShortCharacterActual,`.
  **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(messages, common::UsageWarning::ShortCharacterActual,`。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument variable length '%jd' is less than expected length '%jd'"_warn_en_US,`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument variable length '%jd' is less than expected length '%jd'"_warn_en_US,`。
- **L251 EN**: Comment explains nearby logic, intent, or metadata: `actualLength, *dummyLength);`.
  **L251 CN**: 注释说明附近代码的逻辑、意图或元数据：`actualLength, *dummyLength);`。
- **L252 EN**: Transitions from the previous branch into the alternative path.
  **L252 CN**: 从前一个分支过渡到备选路径。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Warn(messages, common::UsageWarning::ShortCharacterActual,`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Warn(messages, common::UsageWarning::ShortCharacterActual,`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument expression length '%jd' is less than expected length '%jd'"_warn_en_US,`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument expression length '%jd' is less than expected length '%jd'"_warn_en_US,`。
- **L255 EN**: Comment explains nearby logic, intent, or metadata: `actualLength, *dummyLength);`.
  **L255 CN**: 注释说明附近代码的逻辑、意图或元数据：`actualLength, *dummyLength);`。
- **L256 EN**: Closes the current lexical scope or compound statement.
  **L256 CN**: 结束当前词法作用域或复合语句块。
- **L257 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L257 CN**: 开始 `if` 控制流语句并计算其条件。
- **L258 EN**: Continues the surrounding expression or declaration: `auto converted{`.
  **L258 CN**: 继续构造周围的表达式或声明：`auto converted{`。
- **L259 EN**: Executes a call or declaration centered on `ConvertToType`.
  **L259 CN**: 执行以 `ConvertToType` 为核心的调用或声明。
- **L260 EN**: Executes a call or declaration centered on `CHECK`.
  **L260 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L261 EN**: Executes a call or declaration centered on `std::move`.
  **L261 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L262 EN**: Executes a call or declaration centered on `actualType.set_LEN`.
  **L262 CN**: 执行以 `actualType.set_LEN` 为核心的调用或声明。
- **L263 EN**: Closes the current lexical scope or compound statement.
  **L263 CN**: 结束当前词法作用域或复合语句块。
- **L264 EN**: Closes the current lexical scope or compound statement.
  **L264 CN**: 结束当前词法作用域或复合语句块。

### Lines 265-288

````cpp
        }
      }
    }
  }
}

// Automatic conversion of different-kind INTEGER scalar actual
// argument expressions (not variables) to INTEGER scalar dummies.
// We return nonstandard INTEGER(8) results from intrinsic functions
// like SIZE() by default in order to facilitate the use of large
// arrays.  Emit a warning when downconverting.
static void ConvertIntegerActual(evaluate::Expr<evaluate::SomeType> &actual,
    const characteristics::TypeAndShape &dummyType,
    characteristics::TypeAndShape &actualType,
    parser::ContextualMessages &messages, SemanticsContext &semanticsContext) {
  if (dummyType.type().category() == TypeCategory::Integer &&
      actualType.type().category() == TypeCategory::Integer &&
      dummyType.type().kind() != actualType.type().kind() &&
      dummyType.Rank() == 0 && actualType.Rank() == 0 &&
      !evaluate::IsVariable(actual)) {
    auto converted{
        evaluate::ConvertToType(dummyType.type(), std::move(actual))};
    CHECK(converted);
    actual = std::move(*converted);
````
- **L265 EN**: Closes the current lexical scope or compound statement.
  **L265 CN**: 结束当前词法作用域或复合语句块。
- **L266 EN**: Closes the current lexical scope or compound statement.
  **L266 CN**: 结束当前词法作用域或复合语句块。
- **L267 EN**: Closes the current lexical scope or compound statement.
  **L267 CN**: 结束当前词法作用域或复合语句块。
- **L268 EN**: Closes the current lexical scope or compound statement.
  **L268 CN**: 结束当前词法作用域或复合语句块。
- **L269 EN**: Closes the current lexical scope or compound statement.
  **L269 CN**: 结束当前词法作用域或复合语句块。
- **L270 EN**: Blank line separating nearby declarations or logic blocks.
  **L270 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L271 EN**: Comment explains nearby logic, intent, or metadata: `Automatic conversion of different-kind INTEGER scalar actual`.
  **L271 CN**: 注释说明附近代码的逻辑、意图或元数据：`Automatic conversion of different-kind INTEGER scalar actual`。
- **L272 EN**: Comment explains nearby logic, intent, or metadata: `argument expressions (not variables) to INTEGER scalar dummies.`.
  **L272 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument expressions (not variables) to INTEGER scalar dummies.`。
- **L273 EN**: Comment explains nearby logic, intent, or metadata: `We return nonstandard INTEGER(8) results from intrinsic functions`.
  **L273 CN**: 注释说明附近代码的逻辑、意图或元数据：`We return nonstandard INTEGER(8) results from intrinsic functions`。
- **L274 EN**: Comment explains nearby logic, intent, or metadata: `like SIZE() by default in order to facilitate the use of large`.
  **L274 CN**: 注释说明附近代码的逻辑、意图或元数据：`like SIZE() by default in order to facilitate the use of large`。
- **L275 EN**: Comment explains nearby logic, intent, or metadata: `arrays.  Emit a warning when downconverting.`.
  **L275 CN**: 注释说明附近代码的逻辑、意图或元数据：`arrays.  Emit a warning when downconverting.`。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void ConvertIntegerActual(evaluate::Expr<evaluate::SomeType> &actual,`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void ConvertIntegerActual(evaluate::Expr<evaluate::SomeType> &actual,`。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const characteristics::TypeAndShape &dummyType,`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`const characteristics::TypeAndShape &dummyType,`。
- **L278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::TypeAndShape &actualType,`.
  **L278 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::TypeAndShape &actualType,`。
- **L279 EN**: Continues the surrounding expression or declaration: `parser::ContextualMessages &messages, SemanticsContext &semanticsContext) {`.
  **L279 CN**: 继续构造周围的表达式或声明：`parser::ContextualMessages &messages, SemanticsContext &semanticsContext) {`。
- **L280 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L280 CN**: 开始 `if` 控制流语句并计算其条件。
- **L281 EN**: Continues logic associated with callable symbol `type`.
  **L281 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L282 EN**: Continues logic associated with callable symbol `type`.
  **L282 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L283 EN**: Continues logic associated with callable symbol `Rank`.
  **L283 CN**: 继续与可调用符号 `Rank` 相关的逻辑。
- **L284 EN**: Starts a function, method, lambda, or structured scope: `!evaluate::IsVariable(actual)) {`.
  **L284 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!evaluate::IsVariable(actual)) {`。
- **L285 EN**: Continues the surrounding expression or declaration: `auto converted{`.
  **L285 CN**: 继续构造周围的表达式或声明：`auto converted{`。
- **L286 EN**: Executes a call or declaration centered on `evaluate::ConvertToType`.
  **L286 CN**: 执行以 `evaluate::ConvertToType` 为核心的调用或声明。
- **L287 EN**: Executes a call or declaration centered on `CHECK`.
  **L287 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L288 EN**: Executes a call or declaration centered on `std::move`.
  **L288 CN**: 执行以 `std::move` 为核心的调用或声明。

### Lines 289-312

````cpp
    if (dummyType.type().kind() < actualType.type().kind()) {
      if (!semanticsContext.IsEnabled(
              common::LanguageFeature::ActualIntegerConvertedToSmallerKind)) {
        messages.Say(
            "Actual argument scalar expression of type INTEGER(%d) cannot be implicitly converted to smaller dummy argument type INTEGER(%d)"_err_en_US,
            actualType.type().kind(), dummyType.type().kind());
      } else {
        semanticsContext.Warn(messages,
            common::LanguageFeature::ActualIntegerConvertedToSmallerKind,
            "Actual argument scalar expression of type INTEGER(%d) was converted to smaller dummy argument type INTEGER(%d)"_port_en_US,
            actualType.type().kind(), dummyType.type().kind());
      }
    }
    actualType = dummyType;
  }
}

// Automatic conversion of different-kind LOGICAL scalar actual argument
// expressions (not variables) to LOGICAL scalar dummies when the dummy is of
// default logical kind. This allows expressions in dummy arguments to work when
// the default logical kind is not the one used in LogicalResult. This will
// always be safe even when downconverting so no warning is needed.
static void ConvertLogicalActual(evaluate::Expr<evaluate::SomeType> &actual,
    const characteristics::TypeAndShape &dummyType,
````
- **L289 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L289 CN**: 开始 `if` 控制流语句并计算其条件。
- **L290 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L290 CN**: 开始 `if` 控制流语句并计算其条件。
- **L291 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::ActualIntegerConvertedToSmallerKind)) {`.
  **L291 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::ActualIntegerConvertedToSmallerKind)) {`。
- **L292 EN**: Continues logic associated with callable symbol `Say`.
  **L292 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument scalar expression of type INTEGER(%d) cannot be implicitly converted to smaller dummy argument type INTEGER(%d)"_err_en_US,`.
  **L293 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument scalar expression of type INTEGER(%d) cannot be implicitly converted to smaller dummy argument type INTEGER(%d)"_err_en_US,`。
- **L294 EN**: Executes a call or declaration centered on `actualType.type`.
  **L294 CN**: 执行以 `actualType.type` 为核心的调用或声明。
- **L295 EN**: Transitions from the previous branch into the alternative path.
  **L295 CN**: 从前一个分支过渡到备选路径。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `semanticsContext.Warn(messages,`.
  **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`semanticsContext.Warn(messages,`。
- **L297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::LanguageFeature::ActualIntegerConvertedToSmallerKind,`.
  **L297 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::LanguageFeature::ActualIntegerConvertedToSmallerKind,`。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument scalar expression of type INTEGER(%d) was converted to smaller dummy argument type INTEGER(%d)"_port_en_US,`.
  **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument scalar expression of type INTEGER(%d) was converted to smaller dummy argument type INTEGER(%d)"_port_en_US,`。
- **L299 EN**: Executes a call or declaration centered on `actualType.type`.
  **L299 CN**: 执行以 `actualType.type` 为核心的调用或声明。
- **L300 EN**: Closes the current lexical scope or compound statement.
  **L300 CN**: 结束当前词法作用域或复合语句块。
- **L301 EN**: Closes the current lexical scope or compound statement.
  **L301 CN**: 结束当前词法作用域或复合语句块。
- **L302 EN**: Executes a standalone statement or declaration: `actualType = dummyType;`.
  **L302 CN**: 执行一条独立语句或声明：`actualType = dummyType;`。
- **L303 EN**: Closes the current lexical scope or compound statement.
  **L303 CN**: 结束当前词法作用域或复合语句块。
- **L304 EN**: Closes the current lexical scope or compound statement.
  **L304 CN**: 结束当前词法作用域或复合语句块。
- **L305 EN**: Blank line separating nearby declarations or logic blocks.
  **L305 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L306 EN**: Comment explains nearby logic, intent, or metadata: `Automatic conversion of different-kind LOGICAL scalar actual argument`.
  **L306 CN**: 注释说明附近代码的逻辑、意图或元数据：`Automatic conversion of different-kind LOGICAL scalar actual argument`。
- **L307 EN**: Comment explains nearby logic, intent, or metadata: `expressions (not variables) to LOGICAL scalar dummies when the dummy is of`.
  **L307 CN**: 注释说明附近代码的逻辑、意图或元数据：`expressions (not variables) to LOGICAL scalar dummies when the dummy is of`。
- **L308 EN**: Comment explains nearby logic, intent, or metadata: `default logical kind. This allows expressions in dummy arguments to work when`.
  **L308 CN**: 注释说明附近代码的逻辑、意图或元数据：`default logical kind. This allows expressions in dummy arguments to work when`。
- **L309 EN**: Comment explains nearby logic, intent, or metadata: `the default logical kind is not the one used in LogicalResult. This will`.
  **L309 CN**: 注释说明附近代码的逻辑、意图或元数据：`the default logical kind is not the one used in LogicalResult. This will`。
- **L310 EN**: Comment explains nearby logic, intent, or metadata: `always be safe even when downconverting so no warning is needed.`.
  **L310 CN**: 注释说明附近代码的逻辑、意图或元数据：`always be safe even when downconverting so no warning is needed.`。
- **L311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void ConvertLogicalActual(evaluate::Expr<evaluate::SomeType> &actual,`.
  **L311 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void ConvertLogicalActual(evaluate::Expr<evaluate::SomeType> &actual,`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const characteristics::TypeAndShape &dummyType,`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`const characteristics::TypeAndShape &dummyType,`。

### Lines 313-336

````cpp
    characteristics::TypeAndShape &actualType) {
  if (dummyType.type().category() == TypeCategory::Logical &&
      actualType.type().category() == TypeCategory::Logical &&
      dummyType.type().kind() != actualType.type().kind() &&
      !evaluate::IsVariable(actual)) {
    auto converted{
        evaluate::ConvertToType(dummyType.type(), std::move(actual))};
    CHECK(converted);
    actual = std::move(*converted);
    actualType = dummyType;
  }
}

static bool DefersSameTypeParameters(
    const DerivedTypeSpec *actual, const DerivedTypeSpec *dummy) {
  if (actual && dummy) {
    for (const auto &pair : actual->parameters()) {
      const ParamValue &actualValue{pair.second};
      const ParamValue *dummyValue{dummy->FindParameter(pair.first)};
      if (!dummyValue ||
          (actualValue.isDeferred() != dummyValue->isDeferred())) {
        return false;
      }
    }
````
- **L313 EN**: Continues the surrounding expression or declaration: `characteristics::TypeAndShape &actualType) {`.
  **L313 CN**: 继续构造周围的表达式或声明：`characteristics::TypeAndShape &actualType) {`。
- **L314 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L314 CN**: 开始 `if` 控制流语句并计算其条件。
- **L315 EN**: Continues logic associated with callable symbol `type`.
  **L315 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L316 EN**: Continues logic associated with callable symbol `type`.
  **L316 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L317 EN**: Starts a function, method, lambda, or structured scope: `!evaluate::IsVariable(actual)) {`.
  **L317 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!evaluate::IsVariable(actual)) {`。
- **L318 EN**: Continues the surrounding expression or declaration: `auto converted{`.
  **L318 CN**: 继续构造周围的表达式或声明：`auto converted{`。
- **L319 EN**: Executes a call or declaration centered on `evaluate::ConvertToType`.
  **L319 CN**: 执行以 `evaluate::ConvertToType` 为核心的调用或声明。
- **L320 EN**: Executes a call or declaration centered on `CHECK`.
  **L320 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L321 EN**: Executes a call or declaration centered on `std::move`.
  **L321 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L322 EN**: Executes a standalone statement or declaration: `actualType = dummyType;`.
  **L322 CN**: 执行一条独立语句或声明：`actualType = dummyType;`。
- **L323 EN**: Closes the current lexical scope or compound statement.
  **L323 CN**: 结束当前词法作用域或复合语句块。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L326 EN**: Continues logic associated with callable symbol `DefersSameTypeParameters`.
  **L326 CN**: 继续与可调用符号 `DefersSameTypeParameters` 相关的逻辑。
- **L327 EN**: Continues the surrounding expression or declaration: `const DerivedTypeSpec *actual, const DerivedTypeSpec *dummy) {`.
  **L327 CN**: 继续构造周围的表达式或声明：`const DerivedTypeSpec *actual, const DerivedTypeSpec *dummy) {`。
- **L328 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L328 CN**: 开始 `if` 控制流语句并计算其条件。
- **L329 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L329 CN**: 开始 `for` 控制流语句并计算其条件。
- **L330 EN**: Executes a standalone statement or declaration: `const ParamValue &actualValue{pair.second};`.
  **L330 CN**: 执行一条独立语句或声明：`const ParamValue &actualValue{pair.second};`。
- **L331 EN**: Executes a call or declaration centered on `*dummyValue{dummy->FindParameter`.
  **L331 CN**: 执行以 `*dummyValue{dummy->FindParameter` 为核心的调用或声明。
- **L332 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L332 CN**: 开始 `if` 控制流语句并计算其条件。
- **L333 EN**: Starts a function, method, lambda, or structured scope: `(actualValue.isDeferred() != dummyValue->isDeferred())) {`.
  **L333 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(actualValue.isDeferred() != dummyValue->isDeferred())) {`。
- **L334 EN**: Returns from the current function with `false`.
  **L334 CN**: 以 `false` 从当前函数返回。
- **L335 EN**: Closes the current lexical scope or compound statement.
  **L335 CN**: 结束当前词法作用域或复合语句块。
- **L336 EN**: Closes the current lexical scope or compound statement.
  **L336 CN**: 结束当前词法作用域或复合语句块。

### Lines 337-360

````cpp
  }
  return true;
}

// List of intrinsics that are skipped when checking for device actual
// arguments.
static const llvm::StringSet<> cudaSkippedIntrinsics = {"__builtin_c_devloc",
    "__builtin_c_f_pointer", "__builtin_c_loc", "__builtin_show_descriptor",
    "allocated", "associated", "kind", "lbound", "loc", "present", "shape",
    "size", "sizeof", "ubound"};

static void CheckExplicitDataArg(const characteristics::DummyDataObject &dummy,
    const std::string &dummyName, evaluate::Expr<evaluate::SomeType> &actual,
    characteristics::TypeAndShape &actualType, bool isElemental,
    SemanticsContext &context, evaluate::FoldingContext &foldingContext,
    const Scope *scope, const evaluate::SpecificIntrinsic *intrinsic,
    bool allowActualArgumentConversions, bool extentErrors,
    const characteristics::Procedure &procedure,
    const evaluate::ActualArgument &arg,
    const characteristics::DummyArgument &dummyArg) {

  // Basic type & rank checking
  parser::ContextualMessages &messages{foldingContext.messages()};
  CheckCharacterActual(
````
- **L337 EN**: Closes the current lexical scope or compound statement.
  **L337 CN**: 结束当前词法作用域或复合语句块。
- **L338 EN**: Returns from the current function with `true`.
  **L338 CN**: 以 `true` 从当前函数返回。
- **L339 EN**: Closes the current lexical scope or compound statement.
  **L339 CN**: 结束当前词法作用域或复合语句块。
- **L340 EN**: Blank line separating nearby declarations or logic blocks.
  **L340 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L341 EN**: Comment explains nearby logic, intent, or metadata: `List of intrinsics that are skipped when checking for device actual`.
  **L341 CN**: 注释说明附近代码的逻辑、意图或元数据：`List of intrinsics that are skipped when checking for device actual`。
- **L342 EN**: Comment explains nearby logic, intent, or metadata: `arguments.`.
  **L342 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments.`。
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static const llvm::StringSet<> cudaSkippedIntrinsics = {"__builtin_c_devloc",`.
  **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`static const llvm::StringSet<> cudaSkippedIntrinsics = {"__builtin_c_devloc",`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"__builtin_c_f_pointer", "__builtin_c_loc", "__builtin_show_descriptor",`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`"__builtin_c_f_pointer", "__builtin_c_loc", "__builtin_show_descriptor",`。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"allocated", "associated", "kind", "lbound", "loc", "present", "shape",`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`"allocated", "associated", "kind", "lbound", "loc", "present", "shape",`。
- **L346 EN**: Executes a standalone statement or declaration: `"size", "sizeof", "ubound"};`.
  **L346 CN**: 执行一条独立语句或声明：`"size", "sizeof", "ubound"};`。
- **L347 EN**: Blank line separating nearby declarations or logic blocks.
  **L347 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckExplicitDataArg(const characteristics::DummyDataObject &dummy,`.
  **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckExplicitDataArg(const characteristics::DummyDataObject &dummy,`。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const std::string &dummyName, evaluate::Expr<evaluate::SomeType> &actual,`.
  **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`const std::string &dummyName, evaluate::Expr<evaluate::SomeType> &actual,`。
- **L350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::TypeAndShape &actualType, bool isElemental,`.
  **L350 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::TypeAndShape &actualType, bool isElemental,`。
- **L351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SemanticsContext &context, evaluate::FoldingContext &foldingContext,`.
  **L351 CN**: 继续一个多行参数列表、初始化器或聚合项：`SemanticsContext &context, evaluate::FoldingContext &foldingContext,`。
- **L352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Scope *scope, const evaluate::SpecificIntrinsic *intrinsic,`.
  **L352 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Scope *scope, const evaluate::SpecificIntrinsic *intrinsic,`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool allowActualArgumentConversions, bool extentErrors,`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool allowActualArgumentConversions, bool extentErrors,`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const characteristics::Procedure &procedure,`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`const characteristics::Procedure &procedure,`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const evaluate::ActualArgument &arg,`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`const evaluate::ActualArgument &arg,`。
- **L356 EN**: Continues the surrounding expression or declaration: `const characteristics::DummyArgument &dummyArg) {`.
  **L356 CN**: 继续构造周围的表达式或声明：`const characteristics::DummyArgument &dummyArg) {`。
- **L357 EN**: Blank line separating nearby declarations or logic blocks.
  **L357 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L358 EN**: Comment explains nearby logic, intent, or metadata: `Basic type & rank checking`.
  **L358 CN**: 注释说明附近代码的逻辑、意图或元数据：`Basic type & rank checking`。
- **L359 EN**: Executes a call or declaration centered on `&messages{foldingContext.messages`.
  **L359 CN**: 执行以 `&messages{foldingContext.messages` 为核心的调用或声明。
- **L360 EN**: Continues logic associated with callable symbol `CheckCharacterActual`.
  **L360 CN**: 继续与可调用符号 `CheckCharacterActual` 相关的逻辑。

### Lines 361-384

````cpp
      actual, dummy, actualType, context, messages, extentErrors, dummyName);
  bool dummyIsAllocatable{
      dummy.attrs.test(characteristics::DummyDataObject::Attr::Allocatable)};
  bool dummyIsPointer{
      dummy.attrs.test(characteristics::DummyDataObject::Attr::Pointer)};
  bool dummyIsAllocatableOrPointer{dummyIsAllocatable || dummyIsPointer};
  allowActualArgumentConversions &= !dummyIsAllocatableOrPointer;
  bool typesCompatibleWithIgnoreTKR{
      (dummy.ignoreTKR.test(common::IgnoreTKR::Type) &&
          (dummy.type.type().category() == TypeCategory::Derived ||
              actualType.type().category() == TypeCategory::Derived ||
              dummy.type.type().category() != actualType.type().category())) ||
      (dummy.ignoreTKR.test(common::IgnoreTKR::Kind) &&
          dummy.type.type().category() == actualType.type().category())};
  allowActualArgumentConversions &= !typesCompatibleWithIgnoreTKR;
  if (allowActualArgumentConversions) {
    ConvertIntegerActual(actual, dummy.type, actualType, messages, context);
    ConvertLogicalActual(actual, dummy.type, actualType);
  }
  bool typesCompatible{typesCompatibleWithIgnoreTKR ||
      dummy.type.type().IsTkCompatibleWith(actualType.type())};
  int dummyRank{dummy.type.Rank()};
  // Used to issue a general warning when we don't generate a specific warning
  // or error for this case.
````
- **L361 EN**: Executes a standalone statement or declaration: `actual, dummy, actualType, context, messages, extentErrors, dummyName);`.
  **L361 CN**: 执行一条独立语句或声明：`actual, dummy, actualType, context, messages, extentErrors, dummyName);`。
- **L362 EN**: Continues the surrounding expression or declaration: `bool dummyIsAllocatable{`.
  **L362 CN**: 继续构造周围的表达式或声明：`bool dummyIsAllocatable{`。
- **L363 EN**: Executes a call or declaration centered on `dummy.attrs.test`.
  **L363 CN**: 执行以 `dummy.attrs.test` 为核心的调用或声明。
- **L364 EN**: Continues the surrounding expression or declaration: `bool dummyIsPointer{`.
  **L364 CN**: 继续构造周围的表达式或声明：`bool dummyIsPointer{`。
- **L365 EN**: Executes a call or declaration centered on `dummy.attrs.test`.
  **L365 CN**: 执行以 `dummy.attrs.test` 为核心的调用或声明。
- **L366 EN**: Executes a standalone statement or declaration: `bool dummyIsAllocatableOrPointer{dummyIsAllocatable || dummyIsPointer};`.
  **L366 CN**: 执行一条独立语句或声明：`bool dummyIsAllocatableOrPointer{dummyIsAllocatable || dummyIsPointer};`。
- **L367 EN**: Executes a standalone statement or declaration: `allowActualArgumentConversions &= !dummyIsAllocatableOrPointer;`.
  **L367 CN**: 执行一条独立语句或声明：`allowActualArgumentConversions &= !dummyIsAllocatableOrPointer;`。
- **L368 EN**: Continues the surrounding expression or declaration: `bool typesCompatibleWithIgnoreTKR{`.
  **L368 CN**: 继续构造周围的表达式或声明：`bool typesCompatibleWithIgnoreTKR{`。
- **L369 EN**: Continues logic associated with callable symbol `test`.
  **L369 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L370 EN**: Continues logic associated with callable symbol `type`.
  **L370 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L371 EN**: Continues logic associated with callable symbol `type`.
  **L371 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L372 EN**: Continues logic associated with callable symbol `type`.
  **L372 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L373 EN**: Continues logic associated with callable symbol `test`.
  **L373 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L374 EN**: Executes a call or declaration centered on `dummy.type.type`.
  **L374 CN**: 执行以 `dummy.type.type` 为核心的调用或声明。
- **L375 EN**: Executes a standalone statement or declaration: `allowActualArgumentConversions &= !typesCompatibleWithIgnoreTKR;`.
  **L375 CN**: 执行一条独立语句或声明：`allowActualArgumentConversions &= !typesCompatibleWithIgnoreTKR;`。
- **L376 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L376 CN**: 开始 `if` 控制流语句并计算其条件。
- **L377 EN**: Executes a call or declaration centered on `ConvertIntegerActual`.
  **L377 CN**: 执行以 `ConvertIntegerActual` 为核心的调用或声明。
- **L378 EN**: Executes a call or declaration centered on `ConvertLogicalActual`.
  **L378 CN**: 执行以 `ConvertLogicalActual` 为核心的调用或声明。
- **L379 EN**: Closes the current lexical scope or compound statement.
  **L379 CN**: 结束当前词法作用域或复合语句块。
- **L380 EN**: Continues the surrounding expression or declaration: `bool typesCompatible{typesCompatibleWithIgnoreTKR ||`.
  **L380 CN**: 继续构造周围的表达式或声明：`bool typesCompatible{typesCompatibleWithIgnoreTKR ||`。
- **L381 EN**: Executes a call or declaration centered on `dummy.type.type`.
  **L381 CN**: 执行以 `dummy.type.type` 为核心的调用或声明。
- **L382 EN**: Executes a call or declaration centered on `dummyRank{dummy.type.Rank`.
  **L382 CN**: 执行以 `dummyRank{dummy.type.Rank` 为核心的调用或声明。
- **L383 EN**: Comment explains nearby logic, intent, or metadata: `Used to issue a general warning when we don't generate a specific warning`.
  **L383 CN**: 注释说明附近代码的逻辑、意图或元数据：`Used to issue a general warning when we don't generate a specific warning`。
- **L384 EN**: Comment explains nearby logic, intent, or metadata: `or error for this case.`.
  **L384 CN**: 注释说明附近代码的逻辑、意图或元数据：`or error for this case.`。

### Lines 385-408

````cpp
  bool volatileOrAsyncNeedsTempDiagnosticIssued{false};
  if (typesCompatible) {
    if (const auto *constantChar{
            evaluate::UnwrapConstantValue<evaluate::Ascii>(actual)};
        constantChar && constantChar->wasHollerith() &&
        dummy.type.type().IsUnlimitedPolymorphic()) {
      foldingContext.Warn(common::LanguageFeature::HollerithPolymorphic,
          "passing Hollerith to unlimited polymorphic as if it were CHARACTER"_port_en_US);
    }
  } else if (dummyRank == 0 && allowActualArgumentConversions) {
    // Extension: pass Hollerith literal to scalar as if it had been BOZ
    if (auto converted{evaluate::HollerithToBOZ(
            foldingContext, actual, dummy.type.type())}) {
      foldingContext.Warn(common::LanguageFeature::HollerithOrCharacterAsBOZ,
          "passing Hollerith or character literal as if it were BOZ"_port_en_US);
      actual = *converted;
      actualType.type() = dummy.type.type();
      typesCompatible = true;
    }
  }
  bool dummyIsAssumedRank{dummy.type.attrs().test(
      characteristics::TypeAndShape::Attr::AssumedRank)};
  bool actualIsAssumedSize{actualType.attrs().test(
      characteristics::TypeAndShape::Attr::AssumedSize)};
````
- **L385 EN**: Executes a standalone statement or declaration: `bool volatileOrAsyncNeedsTempDiagnosticIssued{false};`.
  **L385 CN**: 执行一条独立语句或声明：`bool volatileOrAsyncNeedsTempDiagnosticIssued{false};`。
- **L386 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L386 CN**: 开始 `if` 控制流语句并计算其条件。
- **L387 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L387 CN**: 开始 `if` 控制流语句并计算其条件。
- **L388 EN**: Executes a call or declaration centered on `evaluate::UnwrapConstantValue<evaluate::Ascii>`.
  **L388 CN**: 执行以 `evaluate::UnwrapConstantValue<evaluate::Ascii>` 为核心的调用或声明。
- **L389 EN**: Continues logic associated with callable symbol `wasHollerith`.
  **L389 CN**: 继续与可调用符号 `wasHollerith` 相关的逻辑。
- **L390 EN**: Starts a function, method, lambda, or structured scope: `dummy.type.type().IsUnlimitedPolymorphic()) {`.
  **L390 CN**: 开始一个函数、方法、lambda 或结构化作用域：`dummy.type.type().IsUnlimitedPolymorphic()) {`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::LanguageFeature::HollerithPolymorphic,`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::LanguageFeature::HollerithPolymorphic,`。
- **L392 EN**: Executes a standalone statement or declaration: `"passing Hollerith to unlimited polymorphic as if it were CHARACTER"_port_en_US);`.
  **L392 CN**: 执行一条独立语句或声明：`"passing Hollerith to unlimited polymorphic as if it were CHARACTER"_port_en_US);`。
- **L393 EN**: Closes the current lexical scope or compound statement.
  **L393 CN**: 结束当前词法作用域或复合语句块。
- **L394 EN**: Transitions from the previous branch into an `else if` condition.
  **L394 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L395 EN**: Comment explains nearby logic, intent, or metadata: `Extension: pass Hollerith literal to scalar as if it had been BOZ`.
  **L395 CN**: 注释说明附近代码的逻辑、意图或元数据：`Extension: pass Hollerith literal to scalar as if it had been BOZ`。
- **L396 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L396 CN**: 开始 `if` 控制流语句并计算其条件。
- **L397 EN**: Starts a function, method, lambda, or structured scope: `foldingContext, actual, dummy.type.type())}) {`.
  **L397 CN**: 开始一个函数、方法、lambda 或结构化作用域：`foldingContext, actual, dummy.type.type())}) {`。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::LanguageFeature::HollerithOrCharacterAsBOZ,`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::LanguageFeature::HollerithOrCharacterAsBOZ,`。
- **L399 EN**: Executes a standalone statement or declaration: `"passing Hollerith or character literal as if it were BOZ"_port_en_US);`.
  **L399 CN**: 执行一条独立语句或声明：`"passing Hollerith or character literal as if it were BOZ"_port_en_US);`。
- **L400 EN**: Executes a standalone statement or declaration: `actual = *converted;`.
  **L400 CN**: 执行一条独立语句或声明：`actual = *converted;`。
- **L401 EN**: Executes a call or declaration centered on `actualType.type`.
  **L401 CN**: 执行以 `actualType.type` 为核心的调用或声明。
- **L402 EN**: Executes a standalone statement or declaration: `typesCompatible = true;`.
  **L402 CN**: 执行一条独立语句或声明：`typesCompatible = true;`。
- **L403 EN**: Closes the current lexical scope or compound statement.
  **L403 CN**: 结束当前词法作用域或复合语句块。
- **L404 EN**: Closes the current lexical scope or compound statement.
  **L404 CN**: 结束当前词法作用域或复合语句块。
- **L405 EN**: Continues logic associated with callable symbol `attrs`.
  **L405 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L406 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape::Attr::AssumedRank)};`.
  **L406 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape::Attr::AssumedRank)};`。
- **L407 EN**: Continues logic associated with callable symbol `attrs`.
  **L407 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L408 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape::Attr::AssumedSize)};`.
  **L408 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape::Attr::AssumedSize)};`。

### Lines 409-432

````cpp
  bool actualIsAssumedRank{IsAssumedRank(actual)};
  bool actualIsPointer{evaluate::IsObjectPointer(actual)};
  bool actualIsAllocatable{evaluate::IsAllocatableDesignator(actual)};
  bool actualMayBeAssumedSize{actualIsAssumedSize ||
      (actualIsAssumedRank && !actualIsPointer && !actualIsAllocatable)};
  bool actualIsPolymorphic{actualType.type().IsPolymorphic()};
  const auto *actualDerived{evaluate::GetDerivedTypeSpec(actualType.type())};
  if (typesCompatible) {
    if (isElemental) {
    } else if (dummyIsAssumedRank) {
      if (actualMayBeAssumedSize && dummy.intent == common::Intent::Out) {
        // An INTENT(OUT) dummy might be a no-op at run time
        bool dummyHasSignificantIntentOut{actualIsPolymorphic ||
            (actualDerived &&
                (actualDerived->HasDefaultInitialization(
                     /*ignoreAllocatable=*/false, /*ignorePointer=*/true) ||
                    actualDerived->HasDestruction()))};
        const char *actualDesc{
            actualIsAssumedSize ? "Assumed-size" : "Assumed-rank"};
        if (dummyHasSignificantIntentOut) {
          messages.Say(
              "%s actual argument may not be associated with INTENT(OUT) assumed-rank dummy argument requiring finalization, destruction, or initialization"_err_en_US,
              actualDesc);
        } else {
````
- **L409 EN**: Executes a call or declaration centered on `actualIsAssumedRank{IsAssumedRank`.
  **L409 CN**: 执行以 `actualIsAssumedRank{IsAssumedRank` 为核心的调用或声明。
- **L410 EN**: Executes a call or declaration centered on `actualIsPointer{evaluate::IsObjectPointer`.
  **L410 CN**: 执行以 `actualIsPointer{evaluate::IsObjectPointer` 为核心的调用或声明。
- **L411 EN**: Executes a call or declaration centered on `actualIsAllocatable{evaluate::IsAllocatableDesignator`.
  **L411 CN**: 执行以 `actualIsAllocatable{evaluate::IsAllocatableDesignator` 为核心的调用或声明。
- **L412 EN**: Continues the surrounding expression or declaration: `bool actualMayBeAssumedSize{actualIsAssumedSize ||`.
  **L412 CN**: 继续构造周围的表达式或声明：`bool actualMayBeAssumedSize{actualIsAssumedSize ||`。
- **L413 EN**: Executes a call or declaration centered on `statement`.
  **L413 CN**: 执行以 `statement` 为核心的调用或声明。
- **L414 EN**: Executes a call or declaration centered on `actualIsPolymorphic{actualType.type`.
  **L414 CN**: 执行以 `actualIsPolymorphic{actualType.type` 为核心的调用或声明。
- **L415 EN**: Executes a call or declaration centered on `*actualDerived{evaluate::GetDerivedTypeSpec`.
  **L415 CN**: 执行以 `*actualDerived{evaluate::GetDerivedTypeSpec` 为核心的调用或声明。
- **L416 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L416 CN**: 开始 `if` 控制流语句并计算其条件。
- **L417 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L417 CN**: 开始 `if` 控制流语句并计算其条件。
- **L418 EN**: Transitions from the previous branch into an `else if` condition.
  **L418 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L419 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L419 CN**: 开始 `if` 控制流语句并计算其条件。
- **L420 EN**: Comment explains nearby logic, intent, or metadata: `An INTENT(OUT) dummy might be a no-op at run time`.
  **L420 CN**: 注释说明附近代码的逻辑、意图或元数据：`An INTENT(OUT) dummy might be a no-op at run time`。
- **L421 EN**: Continues the surrounding expression or declaration: `bool dummyHasSignificantIntentOut{actualIsPolymorphic ||`.
  **L421 CN**: 继续构造周围的表达式或声明：`bool dummyHasSignificantIntentOut{actualIsPolymorphic ||`。
- **L422 EN**: Continues the surrounding expression or declaration: `(actualDerived &&`.
  **L422 CN**: 继续构造周围的表达式或声明：`(actualDerived &&`。
- **L423 EN**: Continues logic associated with callable symbol `HasDefaultInitialization`.
  **L423 CN**: 继续与可调用符号 `HasDefaultInitialization` 相关的逻辑。
- **L424 EN**: Comment explains nearby logic, intent, or metadata: `ignoreAllocatable=*/false, /*ignorePointer=*/true) ||`.
  **L424 CN**: 注释说明附近代码的逻辑、意图或元数据：`ignoreAllocatable=*/false, /*ignorePointer=*/true) ||`。
- **L425 EN**: Executes a call or declaration centered on `actualDerived->HasDestruction`.
  **L425 CN**: 执行以 `actualDerived->HasDestruction` 为核心的调用或声明。
- **L426 EN**: Continues the surrounding expression or declaration: `const char *actualDesc{`.
  **L426 CN**: 继续构造周围的表达式或声明：`const char *actualDesc{`。
- **L427 EN**: Executes a standalone statement or declaration: `actualIsAssumedSize ? "Assumed-size" : "Assumed-rank"};`.
  **L427 CN**: 执行一条独立语句或声明：`actualIsAssumedSize ? "Assumed-size" : "Assumed-rank"};`。
- **L428 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L428 CN**: 开始 `if` 控制流语句并计算其条件。
- **L429 EN**: Continues logic associated with callable symbol `Say`.
  **L429 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s actual argument may not be associated with INTENT(OUT) assumed-rank dummy argument requiring finalization, destruction, or initialization"_err_en_US,`.
  **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s actual argument may not be associated with INTENT(OUT) assumed-rank dummy argument requiring finalization, destruction, or initialization"_err_en_US,`。
- **L431 EN**: Executes a standalone statement or declaration: `actualDesc);`.
  **L431 CN**: 执行一条独立语句或声明：`actualDesc);`。
- **L432 EN**: Transitions from the previous branch into the alternative path.
  **L432 CN**: 从前一个分支过渡到备选路径。

### Lines 433-456

````cpp
          foldingContext.Warn(common::UsageWarning::Portability, messages.at(),
              "%s actual argument should not be associated with INTENT(OUT) assumed-rank dummy argument"_port_en_US,
              actualDesc);
        }
      }
    } else if (actualIsAssumedRank) {
      if (actualType.type().category() != TypeCategory::Character &&
          !intrinsic) {
        // A more specific message will have already been emitted for
        // assumed-rank argument that's CHARACTER, a callee that's ELEMENTAL,
        // or an intrinsic procedure that can't handle assumed-rank.
        if (!context.languageFeatures().IsEnabled(
                common::LanguageFeature::AssumedRankPassedToNonAssumedRank)) {
          messages.Say(
              "Assumed-rank actual argument may not be associated with a %s that is not also assumed-rank"_err_en_US,
              dummyName);
        } else {
          foldingContext.Warn(
              common::LanguageFeature::AssumedRankPassedToNonAssumedRank,
              "Assumed-rank actual argument should not be associated with a %s that is not also assumed-rank"_port_en_US,
              dummyName);
        }
      }
    } else if (dummy.ignoreTKR.test(common::IgnoreTKR::Rank)) {
````
- **L433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::Portability, messages.at(),`.
  **L433 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::Portability, messages.at(),`。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s actual argument should not be associated with INTENT(OUT) assumed-rank dummy argument"_port_en_US,`.
  **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s actual argument should not be associated with INTENT(OUT) assumed-rank dummy argument"_port_en_US,`。
- **L435 EN**: Executes a standalone statement or declaration: `actualDesc);`.
  **L435 CN**: 执行一条独立语句或声明：`actualDesc);`。
- **L436 EN**: Closes the current lexical scope or compound statement.
  **L436 CN**: 结束当前词法作用域或复合语句块。
- **L437 EN**: Closes the current lexical scope or compound statement.
  **L437 CN**: 结束当前词法作用域或复合语句块。
- **L438 EN**: Transitions from the previous branch into an `else if` condition.
  **L438 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L439 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L439 CN**: 开始 `if` 控制流语句并计算其条件。
- **L440 EN**: Continues the surrounding expression or declaration: `!intrinsic) {`.
  **L440 CN**: 继续构造周围的表达式或声明：`!intrinsic) {`。
- **L441 EN**: Comment explains nearby logic, intent, or metadata: `A more specific message will have already been emitted for`.
  **L441 CN**: 注释说明附近代码的逻辑、意图或元数据：`A more specific message will have already been emitted for`。
- **L442 EN**: Comment explains nearby logic, intent, or metadata: `assumed-rank argument that's CHARACTER, a callee that's ELEMENTAL,`.
  **L442 CN**: 注释说明附近代码的逻辑、意图或元数据：`assumed-rank argument that's CHARACTER, a callee that's ELEMENTAL,`。
- **L443 EN**: Comment explains nearby logic, intent, or metadata: `or an intrinsic procedure that can't handle assumed-rank.`.
  **L443 CN**: 注释说明附近代码的逻辑、意图或元数据：`or an intrinsic procedure that can't handle assumed-rank.`。
- **L444 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L444 CN**: 开始 `if` 控制流语句并计算其条件。
- **L445 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::AssumedRankPassedToNonAssumedRank)) {`.
  **L445 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::AssumedRankPassedToNonAssumedRank)) {`。
- **L446 EN**: Continues logic associated with callable symbol `Say`.
  **L446 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-rank actual argument may not be associated with a %s that is not also assumed-rank"_err_en_US,`.
  **L447 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-rank actual argument may not be associated with a %s that is not also assumed-rank"_err_en_US,`。
- **L448 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L448 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L449 EN**: Transitions from the previous branch into the alternative path.
  **L449 CN**: 从前一个分支过渡到备选路径。
- **L450 EN**: Continues logic associated with callable symbol `Warn`.
  **L450 CN**: 继续与可调用符号 `Warn` 相关的逻辑。
- **L451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::LanguageFeature::AssumedRankPassedToNonAssumedRank,`.
  **L451 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::LanguageFeature::AssumedRankPassedToNonAssumedRank,`。
- **L452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-rank actual argument should not be associated with a %s that is not also assumed-rank"_port_en_US,`.
  **L452 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-rank actual argument should not be associated with a %s that is not also assumed-rank"_port_en_US,`。
- **L453 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L453 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L454 EN**: Closes the current lexical scope or compound statement.
  **L454 CN**: 结束当前词法作用域或复合语句块。
- **L455 EN**: Closes the current lexical scope or compound statement.
  **L455 CN**: 结束当前词法作用域或复合语句块。
- **L456 EN**: Transitions from the previous branch into an `else if` condition.
  **L456 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 457-480

````cpp
    } else if (dummyRank > 0 && !dummyIsAllocatableOrPointer &&
        !dummy.type.attrs().test(
            characteristics::TypeAndShape::Attr::AssumedShape) &&
        !dummy.type.attrs().test(
            characteristics::TypeAndShape::Attr::DeferredShape) &&
        (actualType.Rank() > 0 || IsArrayElement(actual))) {
      // Sequence association (15.5.2.11) applies -- rank need not match
      // if the actual argument is an array or array element designator,
      // and the dummy is an array, but not assumed-shape or an INTENT(IN)
      // pointer that's standing in for an assumed-shape dummy.
    } else if (dummy.type.shape() && actualType.shape()) {
      // Let CheckConformance accept actual scalars; storage association
      // cases are checked here below.
      CheckConformance(messages, *dummy.type.shape(), *actualType.shape(),
          dummyIsAllocatableOrPointer
              ? evaluate::CheckConformanceFlags::None
              : evaluate::CheckConformanceFlags::RightScalarExpandable,
          "dummy argument", "actual argument");
    }
  } else {
    const auto &len{actualType.LEN()};
    messages.Say(
        "Actual argument type '%s' is not compatible with dummy argument type '%s'"_err_en_US,
        actualType.type().AsFortran(len ? len->AsFortran() : ""),
````
- **L457 EN**: Transitions from the previous branch into an `else if` condition.
  **L457 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L458 EN**: Continues logic associated with callable symbol `attrs`.
  **L458 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L459 EN**: Continues the surrounding expression or declaration: `characteristics::TypeAndShape::Attr::AssumedShape) &&`.
  **L459 CN**: 继续构造周围的表达式或声明：`characteristics::TypeAndShape::Attr::AssumedShape) &&`。
- **L460 EN**: Continues logic associated with callable symbol `attrs`.
  **L460 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L461 EN**: Continues the surrounding expression or declaration: `characteristics::TypeAndShape::Attr::DeferredShape) &&`.
  **L461 CN**: 继续构造周围的表达式或声明：`characteristics::TypeAndShape::Attr::DeferredShape) &&`。
- **L462 EN**: Starts a function, method, lambda, or structured scope: `(actualType.Rank() > 0 || IsArrayElement(actual))) {`.
  **L462 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(actualType.Rank() > 0 || IsArrayElement(actual))) {`。
- **L463 EN**: Comment explains nearby logic, intent, or metadata: `Sequence association (15.5.2.11) applies -- rank need not match`.
  **L463 CN**: 注释说明附近代码的逻辑、意图或元数据：`Sequence association (15.5.2.11) applies -- rank need not match`。
- **L464 EN**: Comment explains nearby logic, intent, or metadata: `if the actual argument is an array or array element designator,`.
  **L464 CN**: 注释说明附近代码的逻辑、意图或元数据：`if the actual argument is an array or array element designator,`。
- **L465 EN**: Comment explains nearby logic, intent, or metadata: `and the dummy is an array, but not assumed-shape or an INTENT(IN)`.
  **L465 CN**: 注释说明附近代码的逻辑、意图或元数据：`and the dummy is an array, but not assumed-shape or an INTENT(IN)`。
- **L466 EN**: Comment explains nearby logic, intent, or metadata: `pointer that's standing in for an assumed-shape dummy.`.
  **L466 CN**: 注释说明附近代码的逻辑、意图或元数据：`pointer that's standing in for an assumed-shape dummy.`。
- **L467 EN**: Transitions from the previous branch into an `else if` condition.
  **L467 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L468 EN**: Comment explains nearby logic, intent, or metadata: `Let CheckConformance accept actual scalars; storage association`.
  **L468 CN**: 注释说明附近代码的逻辑、意图或元数据：`Let CheckConformance accept actual scalars; storage association`。
- **L469 EN**: Comment explains nearby logic, intent, or metadata: `cases are checked here below.`.
  **L469 CN**: 注释说明附近代码的逻辑、意图或元数据：`cases are checked here below.`。
- **L470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckConformance(messages, *dummy.type.shape(), *actualType.shape(),`.
  **L470 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckConformance(messages, *dummy.type.shape(), *actualType.shape(),`。
- **L471 EN**: Continues the surrounding expression or declaration: `dummyIsAllocatableOrPointer`.
  **L471 CN**: 继续构造周围的表达式或声明：`dummyIsAllocatableOrPointer`。
- **L472 EN**: Continues the surrounding expression or declaration: `? evaluate::CheckConformanceFlags::None`.
  **L472 CN**: 继续构造周围的表达式或声明：`? evaluate::CheckConformanceFlags::None`。
- **L473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: evaluate::CheckConformanceFlags::RightScalarExpandable,`.
  **L473 CN**: 继续一个多行参数列表、初始化器或聚合项：`: evaluate::CheckConformanceFlags::RightScalarExpandable,`。
- **L474 EN**: Executes a standalone statement or declaration: `"dummy argument", "actual argument");`.
  **L474 CN**: 执行一条独立语句或声明：`"dummy argument", "actual argument");`。
- **L475 EN**: Closes the current lexical scope or compound statement.
  **L475 CN**: 结束当前词法作用域或复合语句块。
- **L476 EN**: Transitions from the previous branch into the alternative path.
  **L476 CN**: 从前一个分支过渡到备选路径。
- **L477 EN**: Executes a call or declaration centered on `&len{actualType.LEN`.
  **L477 CN**: 执行以 `&len{actualType.LEN` 为核心的调用或声明。
- **L478 EN**: Continues logic associated with callable symbol `Say`.
  **L478 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument type '%s' is not compatible with dummy argument type '%s'"_err_en_US,`.
  **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument type '%s' is not compatible with dummy argument type '%s'"_err_en_US,`。
- **L480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `actualType.type().AsFortran(len ? len->AsFortran() : ""),`.
  **L480 CN**: 继续一个多行参数列表、初始化器或聚合项：`actualType.type().AsFortran(len ? len->AsFortran() : ""),`。

### Lines 481-504

````cpp
        dummy.type.type().AsFortran());
  }

  auto actualCoarrayRef{ExtractCoarrayRef(actual)};
  bool dummyIsAssumedSize{dummy.type.attrs().test(
      characteristics::TypeAndShape::Attr::AssumedSize)};
  bool dummyIsAsynchronous{
      dummy.attrs.test(characteristics::DummyDataObject::Attr::Asynchronous)};
  bool dummyIsVolatile{
      dummy.attrs.test(characteristics::DummyDataObject::Attr::Volatile)};
  bool dummyIsValue{
      dummy.attrs.test(characteristics::DummyDataObject::Attr::Value)};
  bool dummyIsPolymorphic{dummy.type.type().IsPolymorphic()};
  if (actualIsPolymorphic && dummyIsPolymorphic &&
      actualCoarrayRef) { // 15.5.2.4(2)
    messages.Say(
        "Coindexed polymorphic object may not be associated with a polymorphic %s"_err_en_US,
        dummyName);
  }
  if (actualIsPolymorphic && !dummyIsPolymorphic &&
      actualIsAssumedSize) { // 15.5.2.4(2)
    messages.Say(
        "Assumed-size polymorphic array may not be associated with a monomorphic %s"_err_en_US,
        dummyName);
````
- **L481 EN**: Executes a call or declaration centered on `dummy.type.type`.
  **L481 CN**: 执行以 `dummy.type.type` 为核心的调用或声明。
- **L482 EN**: Closes the current lexical scope or compound statement.
  **L482 CN**: 结束当前词法作用域或复合语句块。
- **L483 EN**: Blank line separating nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L484 EN**: Executes a call or declaration centered on `actualCoarrayRef{ExtractCoarrayRef`.
  **L484 CN**: 执行以 `actualCoarrayRef{ExtractCoarrayRef` 为核心的调用或声明。
- **L485 EN**: Continues logic associated with callable symbol `attrs`.
  **L485 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L486 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape::Attr::AssumedSize)};`.
  **L486 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape::Attr::AssumedSize)};`。
- **L487 EN**: Continues the surrounding expression or declaration: `bool dummyIsAsynchronous{`.
  **L487 CN**: 继续构造周围的表达式或声明：`bool dummyIsAsynchronous{`。
- **L488 EN**: Executes a call or declaration centered on `dummy.attrs.test`.
  **L488 CN**: 执行以 `dummy.attrs.test` 为核心的调用或声明。
- **L489 EN**: Continues the surrounding expression or declaration: `bool dummyIsVolatile{`.
  **L489 CN**: 继续构造周围的表达式或声明：`bool dummyIsVolatile{`。
- **L490 EN**: Executes a call or declaration centered on `dummy.attrs.test`.
  **L490 CN**: 执行以 `dummy.attrs.test` 为核心的调用或声明。
- **L491 EN**: Continues the surrounding expression or declaration: `bool dummyIsValue{`.
  **L491 CN**: 继续构造周围的表达式或声明：`bool dummyIsValue{`。
- **L492 EN**: Executes a call or declaration centered on `dummy.attrs.test`.
  **L492 CN**: 执行以 `dummy.attrs.test` 为核心的调用或声明。
- **L493 EN**: Executes a call or declaration centered on `dummyIsPolymorphic{dummy.type.type`.
  **L493 CN**: 执行以 `dummyIsPolymorphic{dummy.type.type` 为核心的调用或声明。
- **L494 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L494 CN**: 开始 `if` 控制流语句并计算其条件。
- **L495 EN**: Continues the surrounding expression or declaration: `actualCoarrayRef) { // 15.5.2.4(2)`.
  **L495 CN**: 继续构造周围的表达式或声明：`actualCoarrayRef) { // 15.5.2.4(2)`。
- **L496 EN**: Continues logic associated with callable symbol `Say`.
  **L496 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Coindexed polymorphic object may not be associated with a polymorphic %s"_err_en_US,`.
  **L497 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Coindexed polymorphic object may not be associated with a polymorphic %s"_err_en_US,`。
- **L498 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L498 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L499 EN**: Closes the current lexical scope or compound statement.
  **L499 CN**: 结束当前词法作用域或复合语句块。
- **L500 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L500 CN**: 开始 `if` 控制流语句并计算其条件。
- **L501 EN**: Continues the surrounding expression or declaration: `actualIsAssumedSize) { // 15.5.2.4(2)`.
  **L501 CN**: 继续构造周围的表达式或声明：`actualIsAssumedSize) { // 15.5.2.4(2)`。
- **L502 EN**: Continues logic associated with callable symbol `Say`.
  **L502 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-size polymorphic array may not be associated with a monomorphic %s"_err_en_US,`.
  **L503 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-size polymorphic array may not be associated with a monomorphic %s"_err_en_US,`。
- **L504 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L504 CN**: 执行一条独立语句或声明：`dummyName);`。

### Lines 505-528

````cpp
  }

  // Derived type actual argument checks
  const Symbol *actualFirstSymbol{evaluate::GetFirstSymbol(actual)};
  bool actualIsAsynchronous{
      actualFirstSymbol && actualFirstSymbol->attrs().test(Attr::ASYNCHRONOUS)};
  bool actualIsVolatile{
      actualFirstSymbol && actualFirstSymbol->attrs().test(Attr::VOLATILE)};
  if (actualDerived && !actualDerived->IsVectorType()) {
    if (dummy.type.type().IsAssumedType()) {
      if (!actualDerived->parameters().empty()) { // 15.5.2.4(2)
        messages.Say(
            "Actual argument associated with TYPE(*) %s may not have a parameterized derived type"_err_en_US,
            dummyName);
      }
      if (const Symbol *
          tbp{FindImmediateComponent(*actualDerived, [](const Symbol &symbol) {
            return symbol.has<ProcBindingDetails>();
          })}) { // 15.5.2.4(2)
        evaluate::SayWithDeclaration(messages, *tbp,
            "Actual argument associated with TYPE(*) %s may not have type-bound procedure '%s'"_err_en_US,
            dummyName, tbp->name());
      }
      auto finals{FinalsForDerivedTypeInstantiation(*actualDerived)};
````
- **L505 EN**: Closes the current lexical scope or compound statement.
  **L505 CN**: 结束当前词法作用域或复合语句块。
- **L506 EN**: Blank line separating nearby declarations or logic blocks.
  **L506 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L507 EN**: Comment explains nearby logic, intent, or metadata: `Derived type actual argument checks`.
  **L507 CN**: 注释说明附近代码的逻辑、意图或元数据：`Derived type actual argument checks`。
- **L508 EN**: Executes a call or declaration centered on `*actualFirstSymbol{evaluate::GetFirstSymbol`.
  **L508 CN**: 执行以 `*actualFirstSymbol{evaluate::GetFirstSymbol` 为核心的调用或声明。
- **L509 EN**: Continues the surrounding expression or declaration: `bool actualIsAsynchronous{`.
  **L509 CN**: 继续构造周围的表达式或声明：`bool actualIsAsynchronous{`。
- **L510 EN**: Executes a call or declaration centered on `actualFirstSymbol->attrs`.
  **L510 CN**: 执行以 `actualFirstSymbol->attrs` 为核心的调用或声明。
- **L511 EN**: Continues the surrounding expression or declaration: `bool actualIsVolatile{`.
  **L511 CN**: 继续构造周围的表达式或声明：`bool actualIsVolatile{`。
- **L512 EN**: Executes a call or declaration centered on `actualFirstSymbol->attrs`.
  **L512 CN**: 执行以 `actualFirstSymbol->attrs` 为核心的调用或声明。
- **L513 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L513 CN**: 开始 `if` 控制流语句并计算其条件。
- **L514 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L514 CN**: 开始 `if` 控制流语句并计算其条件。
- **L515 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L515 CN**: 开始 `if` 控制流语句并计算其条件。
- **L516 EN**: Continues logic associated with callable symbol `Say`.
  **L516 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with TYPE(*) %s may not have a parameterized derived type"_err_en_US,`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with TYPE(*) %s may not have a parameterized derived type"_err_en_US,`。
- **L518 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L518 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L519 EN**: Closes the current lexical scope or compound statement.
  **L519 CN**: 结束当前词法作用域或复合语句块。
- **L520 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L520 CN**: 开始 `if` 控制流语句并计算其条件。
- **L521 EN**: Starts a function, method, lambda, or structured scope: `tbp{FindImmediateComponent(*actualDerived, [](const Symbol &symbol) {`.
  **L521 CN**: 开始一个函数、方法、lambda 或结构化作用域：`tbp{FindImmediateComponent(*actualDerived, [](const Symbol &symbol) {`。
- **L522 EN**: Returns from the current function with `symbol.has<ProcBindingDetails>()`.
  **L522 CN**: 以 `symbol.has<ProcBindingDetails>()` 从当前函数返回。
- **L523 EN**: Continues the surrounding expression or declaration: `})}) { // 15.5.2.4(2)`.
  **L523 CN**: 继续构造周围的表达式或声明：`})}) { // 15.5.2.4(2)`。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, *tbp,`.
  **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, *tbp,`。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with TYPE(*) %s may not have type-bound procedure '%s'"_err_en_US,`.
  **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with TYPE(*) %s may not have type-bound procedure '%s'"_err_en_US,`。
- **L526 EN**: Executes a call or declaration centered on `tbp->name`.
  **L526 CN**: 执行以 `tbp->name` 为核心的调用或声明。
- **L527 EN**: Closes the current lexical scope or compound statement.
  **L527 CN**: 结束当前词法作用域或复合语句块。
- **L528 EN**: Executes a call or declaration centered on `finals{FinalsForDerivedTypeInstantiation`.
  **L528 CN**: 执行以 `finals{FinalsForDerivedTypeInstantiation` 为核心的调用或声明。

### Lines 529-552

````cpp
      if (!finals.empty()) { // 15.5.2.4(2)
        SourceName name{finals.front()->name()};
        if (auto *msg{messages.Say(
                "Actual argument associated with TYPE(*) %s may not have derived type '%s' with FINAL subroutine '%s'"_err_en_US,
                dummyName, actualDerived->typeSymbol().name(), name)}) {
          msg->Attach(name, "FINAL subroutine '%s' in derived type '%s'"_en_US,
              name, actualDerived->typeSymbol().name());
        }
      }
    }
    if (actualCoarrayRef) {
      if (dummy.intent != common::Intent::In && !dummyIsValue) {
        if (auto bad{FindAllocatableUltimateComponent(
                *actualDerived)}) { // 15.5.2.4(6)
          evaluate::SayWithDeclaration(messages, *bad,
              "Coindexed actual argument with ALLOCATABLE ultimate component '%s' must be associated with a %s with VALUE or INTENT(IN) attributes"_err_en_US,
              bad.BuildResultDesignatorName(), dummyName);
        }
      }
      const Symbol &coarray{actualCoarrayRef->GetLastSymbol()};
      if (const DeclTypeSpec * type{coarray.GetType()}) { // C1537
        if (const DerivedTypeSpec * derived{type->AsDerived()}) {
          if (auto bad{semantics::FindPointerUltimateComponent(*derived)}) {
            evaluate::SayWithDeclaration(messages, coarray,
````
- **L529 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L529 CN**: 开始 `if` 控制流语句并计算其条件。
- **L530 EN**: Executes a call or declaration centered on `name{finals.front`.
  **L530 CN**: 执行以 `name{finals.front` 为核心的调用或声明。
- **L531 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L531 CN**: 开始 `if` 控制流语句并计算其条件。
- **L532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with TYPE(*) %s may not have derived type '%s' with FINAL subroutine '%s'"_err_en_US,`.
  **L532 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with TYPE(*) %s may not have derived type '%s' with FINAL subroutine '%s'"_err_en_US,`。
- **L533 EN**: Starts a function, method, lambda, or structured scope: `dummyName, actualDerived->typeSymbol().name(), name)}) {`.
  **L533 CN**: 开始一个函数、方法、lambda 或结构化作用域：`dummyName, actualDerived->typeSymbol().name(), name)}) {`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg->Attach(name, "FINAL subroutine '%s' in derived type '%s'"_en_US,`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg->Attach(name, "FINAL subroutine '%s' in derived type '%s'"_en_US,`。
- **L535 EN**: Executes a call or declaration centered on `actualDerived->typeSymbol`.
  **L535 CN**: 执行以 `actualDerived->typeSymbol` 为核心的调用或声明。
- **L536 EN**: Closes the current lexical scope or compound statement.
  **L536 CN**: 结束当前词法作用域或复合语句块。
- **L537 EN**: Closes the current lexical scope or compound statement.
  **L537 CN**: 结束当前词法作用域或复合语句块。
- **L538 EN**: Closes the current lexical scope or compound statement.
  **L538 CN**: 结束当前词法作用域或复合语句块。
- **L539 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L539 CN**: 开始 `if` 控制流语句并计算其条件。
- **L540 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L540 CN**: 开始 `if` 控制流语句并计算其条件。
- **L541 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L541 CN**: 开始 `if` 控制流语句并计算其条件。
- **L542 EN**: Comment explains nearby logic, intent, or metadata: `actualDerived)}) { // 15.5.2.4(6)`.
  **L542 CN**: 注释说明附近代码的逻辑、意图或元数据：`actualDerived)}) { // 15.5.2.4(6)`。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, *bad,`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, *bad,`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Coindexed actual argument with ALLOCATABLE ultimate component '%s' must be associated with a %s with VALUE or INTENT(IN) attributes"_err_en_US,`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Coindexed actual argument with ALLOCATABLE ultimate component '%s' must be associated with a %s with VALUE or INTENT(IN) attributes"_err_en_US,`。
- **L545 EN**: Executes a call or declaration centered on `bad.BuildResultDesignatorName`.
  **L545 CN**: 执行以 `bad.BuildResultDesignatorName` 为核心的调用或声明。
- **L546 EN**: Closes the current lexical scope or compound statement.
  **L546 CN**: 结束当前词法作用域或复合语句块。
- **L547 EN**: Closes the current lexical scope or compound statement.
  **L547 CN**: 结束当前词法作用域或复合语句块。
- **L548 EN**: Executes a call or declaration centered on `&coarray{actualCoarrayRef->GetLastSymbol`.
  **L548 CN**: 执行以 `&coarray{actualCoarrayRef->GetLastSymbol` 为核心的调用或声明。
- **L549 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L549 CN**: 开始 `if` 控制流语句并计算其条件。
- **L550 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L550 CN**: 开始 `if` 控制流语句并计算其条件。
- **L551 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L551 CN**: 开始 `if` 控制流语句并计算其条件。
- **L552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, coarray,`.
  **L552 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, coarray,`。

### Lines 553-576

````cpp
                "Coindexed object '%s' with POINTER ultimate component '%s' cannot be associated with %s"_err_en_US,
                coarray.name(), bad.BuildResultDesignatorName(), dummyName);
          }
        }
      }
    }
    if (actualIsVolatile != dummyIsVolatile) { // 15.5.2.4(22)
      if (auto bad{semantics::FindCoarrayUltimateComponent(*actualDerived)}) {
        evaluate::SayWithDeclaration(messages, *bad,
            "VOLATILE attribute must match for %s when actual argument has a coarray ultimate component '%s'"_err_en_US,
            dummyName, bad.BuildResultDesignatorName());
      }
    }
  }

  // Rank and shape checks
  const auto *actualLastSymbol{evaluate::GetLastSymbol(actual)};
  if (actualLastSymbol) {
    actualLastSymbol = &ResolveAssociations(*actualLastSymbol);
  }
  int actualRank{actualType.Rank()};
  if (dummyIsValue && dummyRank == 0 &&
      dummy.ignoreTKR.test(common::IgnoreTKR::Rank) && actualRank > 0) {
    messages.Say(
````
- **L553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Coindexed object '%s' with POINTER ultimate component '%s' cannot be associated with %s"_err_en_US,`.
  **L553 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Coindexed object '%s' with POINTER ultimate component '%s' cannot be associated with %s"_err_en_US,`。
- **L554 EN**: Executes a call or declaration centered on `coarray.name`.
  **L554 CN**: 执行以 `coarray.name` 为核心的调用或声明。
- **L555 EN**: Closes the current lexical scope or compound statement.
  **L555 CN**: 结束当前词法作用域或复合语句块。
- **L556 EN**: Closes the current lexical scope or compound statement.
  **L556 CN**: 结束当前词法作用域或复合语句块。
- **L557 EN**: Closes the current lexical scope or compound statement.
  **L557 CN**: 结束当前词法作用域或复合语句块。
- **L558 EN**: Closes the current lexical scope or compound statement.
  **L558 CN**: 结束当前词法作用域或复合语句块。
- **L559 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L559 CN**: 开始 `if` 控制流语句并计算其条件。
- **L560 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L560 CN**: 开始 `if` 控制流语句并计算其条件。
- **L561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, *bad,`.
  **L561 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, *bad,`。
- **L562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"VOLATILE attribute must match for %s when actual argument has a coarray ultimate component '%s'"_err_en_US,`.
  **L562 CN**: 继续一个多行参数列表、初始化器或聚合项：`"VOLATILE attribute must match for %s when actual argument has a coarray ultimate component '%s'"_err_en_US,`。
- **L563 EN**: Executes a call or declaration centered on `bad.BuildResultDesignatorName`.
  **L563 CN**: 执行以 `bad.BuildResultDesignatorName` 为核心的调用或声明。
- **L564 EN**: Closes the current lexical scope or compound statement.
  **L564 CN**: 结束当前词法作用域或复合语句块。
- **L565 EN**: Closes the current lexical scope or compound statement.
  **L565 CN**: 结束当前词法作用域或复合语句块。
- **L566 EN**: Closes the current lexical scope or compound statement.
  **L566 CN**: 结束当前词法作用域或复合语句块。
- **L567 EN**: Blank line separating nearby declarations or logic blocks.
  **L567 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L568 EN**: Comment explains nearby logic, intent, or metadata: `Rank and shape checks`.
  **L568 CN**: 注释说明附近代码的逻辑、意图或元数据：`Rank and shape checks`。
- **L569 EN**: Executes a call or declaration centered on `*actualLastSymbol{evaluate::GetLastSymbol`.
  **L569 CN**: 执行以 `*actualLastSymbol{evaluate::GetLastSymbol` 为核心的调用或声明。
- **L570 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L570 CN**: 开始 `if` 控制流语句并计算其条件。
- **L571 EN**: Executes a call or declaration centered on `&ResolveAssociations`.
  **L571 CN**: 执行以 `&ResolveAssociations` 为核心的调用或声明。
- **L572 EN**: Closes the current lexical scope or compound statement.
  **L572 CN**: 结束当前词法作用域或复合语句块。
- **L573 EN**: Executes a call or declaration centered on `actualRank{actualType.Rank`.
  **L573 CN**: 执行以 `actualRank{actualType.Rank` 为核心的调用或声明。
- **L574 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L574 CN**: 开始 `if` 控制流语句并计算其条件。
- **L575 EN**: Starts a function, method, lambda, or structured scope: `dummy.ignoreTKR.test(common::IgnoreTKR::Rank) && actualRank > 0) {`.
  **L575 CN**: 开始一个函数、方法、lambda 或结构化作用域：`dummy.ignoreTKR.test(common::IgnoreTKR::Rank) && actualRank > 0) {`。
- **L576 EN**: Continues logic associated with callable symbol `Say`.
  **L576 CN**: 继续与可调用符号 `Say` 相关的逻辑。

### Lines 577-600

````cpp
        "Array actual argument may not be associated with IGNORE_TKR(R) scalar %s with VALUE attribute"_err_en_US,
        dummyName);
  } else if (dummy.type.attrs().test(
                 characteristics::TypeAndShape::Attr::AssumedShape)) {
    // 15.5.2.4(16)
    if (actualIsAssumedRank) {
      messages.Say(
          "Assumed-rank actual argument may not be associated with assumed-shape %s"_err_en_US,
          dummyName);
    } else if (actualRank == 0) {
      messages.Say(
          "Scalar actual argument may not be associated with assumed-shape %s"_err_en_US,
          dummyName);
    } else if (actualIsAssumedSize && actualLastSymbol) {
      evaluate::SayWithDeclaration(messages, *actualLastSymbol,
          "Assumed-size array may not be associated with assumed-shape %s"_err_en_US,
          dummyName);
    }
  } else if (dummyIsAssumedSize && dummy.type.type().IsAssumedType() &&
      actualRank == 0 && !actualIsAssumedRank) {
    // F'2023 15.5.2.5 p14 third bullet allows a scalar actual
    // argument to associate with a TYPE(*) assumed-size dummy
    foldingContext.Warn(common::UsageWarning::AssumedTypeSizeDummy,
        "A scalar actual argument for an assumed-size TYPE(*) dummy is not portable"_port_en_US);
````
- **L577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Array actual argument may not be associated with IGNORE_TKR(R) scalar %s with VALUE attribute"_err_en_US,`.
  **L577 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Array actual argument may not be associated with IGNORE_TKR(R) scalar %s with VALUE attribute"_err_en_US,`。
- **L578 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L578 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L579 EN**: Transitions from the previous branch into an `else if` condition.
  **L579 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L580 EN**: Continues the surrounding expression or declaration: `characteristics::TypeAndShape::Attr::AssumedShape)) {`.
  **L580 CN**: 继续构造周围的表达式或声明：`characteristics::TypeAndShape::Attr::AssumedShape)) {`。
- **L581 EN**: Comment explains nearby logic, intent, or metadata: `15.5.2.4(16)`.
  **L581 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.5.2.4(16)`。
- **L582 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L582 CN**: 开始 `if` 控制流语句并计算其条件。
- **L583 EN**: Continues logic associated with callable symbol `Say`.
  **L583 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-rank actual argument may not be associated with assumed-shape %s"_err_en_US,`.
  **L584 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-rank actual argument may not be associated with assumed-shape %s"_err_en_US,`。
- **L585 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L585 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L586 EN**: Transitions from the previous branch into an `else if` condition.
  **L586 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L587 EN**: Continues logic associated with callable symbol `Say`.
  **L587 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Scalar actual argument may not be associated with assumed-shape %s"_err_en_US,`.
  **L588 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Scalar actual argument may not be associated with assumed-shape %s"_err_en_US,`。
- **L589 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L589 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L590 EN**: Transitions from the previous branch into an `else if` condition.
  **L590 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, *actualLastSymbol,`.
  **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, *actualLastSymbol,`。
- **L592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-size array may not be associated with assumed-shape %s"_err_en_US,`.
  **L592 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-size array may not be associated with assumed-shape %s"_err_en_US,`。
- **L593 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L593 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L594 EN**: Closes the current lexical scope or compound statement.
  **L594 CN**: 结束当前词法作用域或复合语句块。
- **L595 EN**: Transitions from the previous branch into an `else if` condition.
  **L595 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L596 EN**: Continues the surrounding expression or declaration: `actualRank == 0 && !actualIsAssumedRank) {`.
  **L596 CN**: 继续构造周围的表达式或声明：`actualRank == 0 && !actualIsAssumedRank) {`。
- **L597 EN**: Comment explains nearby logic, intent, or metadata: `F'2023 15.5.2.5 p14 third bullet allows a scalar actual`.
  **L597 CN**: 注释说明附近代码的逻辑、意图或元数据：`F'2023 15.5.2.5 p14 third bullet allows a scalar actual`。
- **L598 EN**: Comment explains nearby logic, intent, or metadata: `argument to associate with a TYPE(*) assumed-size dummy`.
  **L598 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument to associate with a TYPE(*) assumed-size dummy`。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::AssumedTypeSizeDummy,`.
  **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::AssumedTypeSizeDummy,`。
- **L600 EN**: Executes a call or declaration centered on `TYPE`.
  **L600 CN**: 执行以 `TYPE` 为核心的调用或声明。

### Lines 601-624

````cpp
  } else if (dummyRank > 0) {
    bool basicError{false};
    if (actualRank == 0 && !actualIsAssumedRank &&
        !dummyIsAllocatableOrPointer) {
      // Actual is scalar, dummy is an array.  F'2023 15.5.2.5p14
      if (actualCoarrayRef) {
        basicError = true;
        messages.Say(
            "Coindexed scalar actual argument must be associated with a scalar %s"_err_en_US,
            dummyName);
      }
      bool actualIsArrayElement{IsArrayElement(actual) != nullptr};
      bool actualIsCKindCharacter{
          actualType.type().category() == TypeCategory::Character &&
          actualType.type().kind() == 1};
      if (!actualIsCKindCharacter) {
        if (!actualIsArrayElement && !dummyIsAssumedRank &&
            !dummy.ignoreTKR.test(common::IgnoreTKR::Rank)) {
          basicError = true;
          messages.Say(
              "Whole scalar actual argument may not be associated with a %s array"_err_en_US,
              dummyName);
        }
        if (actualIsPolymorphic) {
````
- **L601 EN**: Transitions from the previous branch into an `else if` condition.
  **L601 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L602 EN**: Executes a standalone statement or declaration: `bool basicError{false};`.
  **L602 CN**: 执行一条独立语句或声明：`bool basicError{false};`。
- **L603 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L603 CN**: 开始 `if` 控制流语句并计算其条件。
- **L604 EN**: Continues the surrounding expression or declaration: `!dummyIsAllocatableOrPointer) {`.
  **L604 CN**: 继续构造周围的表达式或声明：`!dummyIsAllocatableOrPointer) {`。
- **L605 EN**: Comment explains nearby logic, intent, or metadata: `Actual is scalar, dummy is an array.  F'2023 15.5.2.5p14`.
  **L605 CN**: 注释说明附近代码的逻辑、意图或元数据：`Actual is scalar, dummy is an array.  F'2023 15.5.2.5p14`。
- **L606 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L606 CN**: 开始 `if` 控制流语句并计算其条件。
- **L607 EN**: Executes a standalone statement or declaration: `basicError = true;`.
  **L607 CN**: 执行一条独立语句或声明：`basicError = true;`。
- **L608 EN**: Continues logic associated with callable symbol `Say`.
  **L608 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Coindexed scalar actual argument must be associated with a scalar %s"_err_en_US,`.
  **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Coindexed scalar actual argument must be associated with a scalar %s"_err_en_US,`。
- **L610 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L610 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L611 EN**: Closes the current lexical scope or compound statement.
  **L611 CN**: 结束当前词法作用域或复合语句块。
- **L612 EN**: Executes a call or declaration centered on `actualIsArrayElement{IsArrayElement`.
  **L612 CN**: 执行以 `actualIsArrayElement{IsArrayElement` 为核心的调用或声明。
- **L613 EN**: Continues the surrounding expression or declaration: `bool actualIsCKindCharacter{`.
  **L613 CN**: 继续构造周围的表达式或声明：`bool actualIsCKindCharacter{`。
- **L614 EN**: Continues logic associated with callable symbol `type`.
  **L614 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L615 EN**: Executes a call or declaration centered on `actualType.type`.
  **L615 CN**: 执行以 `actualType.type` 为核心的调用或声明。
- **L616 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L616 CN**: 开始 `if` 控制流语句并计算其条件。
- **L617 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L617 CN**: 开始 `if` 控制流语句并计算其条件。
- **L618 EN**: Starts a function, method, lambda, or structured scope: `!dummy.ignoreTKR.test(common::IgnoreTKR::Rank)) {`.
  **L618 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!dummy.ignoreTKR.test(common::IgnoreTKR::Rank)) {`。
- **L619 EN**: Executes a standalone statement or declaration: `basicError = true;`.
  **L619 CN**: 执行一条独立语句或声明：`basicError = true;`。
- **L620 EN**: Continues logic associated with callable symbol `Say`.
  **L620 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Whole scalar actual argument may not be associated with a %s array"_err_en_US,`.
  **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Whole scalar actual argument may not be associated with a %s array"_err_en_US,`。
- **L622 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L622 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L623 EN**: Closes the current lexical scope or compound statement.
  **L623 CN**: 结束当前词法作用域或复合语句块。
- **L624 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L624 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 625-648

````cpp
          basicError = true;
          messages.Say(
              "Polymorphic scalar may not be associated with a %s array"_err_en_US,
              dummyName);
        }
        bool isOkBecauseContiguous{
            context.IsEnabled(
                common::LanguageFeature::ContiguousOkForSeqAssociation) &&
            actualLastSymbol &&
            evaluate::IsContiguous(*actualLastSymbol, foldingContext)
                .value_or(false)};
        if (actualIsArrayElement && actualLastSymbol &&
            !dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {
          if (IsPointer(*actualLastSymbol)) {
            if (isOkBecauseContiguous) {
              foldingContext.Warn(
                  common::LanguageFeature::ContiguousOkForSeqAssociation,
                  "Element of contiguous pointer array is accepted for storage sequence association"_port_en_US);
            } else {
              basicError = true;
              messages.Say(
                  "Element of pointer array may not be associated with a %s array"_err_en_US,
                  dummyName);
            }
````
- **L625 EN**: Executes a standalone statement or declaration: `basicError = true;`.
  **L625 CN**: 执行一条独立语句或声明：`basicError = true;`。
- **L626 EN**: Continues logic associated with callable symbol `Say`.
  **L626 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Polymorphic scalar may not be associated with a %s array"_err_en_US,`.
  **L627 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Polymorphic scalar may not be associated with a %s array"_err_en_US,`。
- **L628 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L628 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L629 EN**: Closes the current lexical scope or compound statement.
  **L629 CN**: 结束当前词法作用域或复合语句块。
- **L630 EN**: Continues the surrounding expression or declaration: `bool isOkBecauseContiguous{`.
  **L630 CN**: 继续构造周围的表达式或声明：`bool isOkBecauseContiguous{`。
- **L631 EN**: Continues logic associated with callable symbol `IsEnabled`.
  **L631 CN**: 继续与可调用符号 `IsEnabled` 相关的逻辑。
- **L632 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::ContiguousOkForSeqAssociation) &&`.
  **L632 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::ContiguousOkForSeqAssociation) &&`。
- **L633 EN**: Continues the surrounding expression or declaration: `actualLastSymbol &&`.
  **L633 CN**: 继续构造周围的表达式或声明：`actualLastSymbol &&`。
- **L634 EN**: Continues logic associated with callable symbol `IsContiguous`.
  **L634 CN**: 继续与可调用符号 `IsContiguous` 相关的逻辑。
- **L635 EN**: Executes a call or declaration centered on `.value_or`.
  **L635 CN**: 执行以 `.value_or` 为核心的调用或声明。
- **L636 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L636 CN**: 开始 `if` 控制流语句并计算其条件。
- **L637 EN**: Starts a function, method, lambda, or structured scope: `!dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {`.
  **L637 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {`。
- **L638 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L638 CN**: 开始 `if` 控制流语句并计算其条件。
- **L639 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L639 CN**: 开始 `if` 控制流语句并计算其条件。
- **L640 EN**: Continues logic associated with callable symbol `Warn`.
  **L640 CN**: 继续与可调用符号 `Warn` 相关的逻辑。
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::LanguageFeature::ContiguousOkForSeqAssociation,`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::LanguageFeature::ContiguousOkForSeqAssociation,`。
- **L642 EN**: Executes a standalone statement or declaration: `"Element of contiguous pointer array is accepted for storage sequence association"_port_en_US);`.
  **L642 CN**: 执行一条独立语句或声明：`"Element of contiguous pointer array is accepted for storage sequence association"_port_en_US);`。
- **L643 EN**: Transitions from the previous branch into the alternative path.
  **L643 CN**: 从前一个分支过渡到备选路径。
- **L644 EN**: Executes a standalone statement or declaration: `basicError = true;`.
  **L644 CN**: 执行一条独立语句或声明：`basicError = true;`。
- **L645 EN**: Continues logic associated with callable symbol `Say`.
  **L645 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Element of pointer array may not be associated with a %s array"_err_en_US,`.
  **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Element of pointer array may not be associated with a %s array"_err_en_US,`。
- **L647 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L647 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L648 EN**: Closes the current lexical scope or compound statement.
  **L648 CN**: 结束当前词法作用域或复合语句块。

### Lines 649-672

````cpp
          } else if (IsAssumedShape(*actualLastSymbol) &&
              !dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {
            if (isOkBecauseContiguous) {
              foldingContext.Warn(
                  common::LanguageFeature::ContiguousOkForSeqAssociation,
                  "Element of contiguous assumed-shape array is accepted for storage sequence association"_port_en_US);
            } else {
              basicError = true;
              messages.Say(
                  "Element of assumed-shape array may not be associated with a %s array"_err_en_US,
                  dummyName);
            }
          }
        }
      }
    }
    // Storage sequence association (F'2023 15.5.2.12p3) checks.
    // Character storage sequence association is checked in
    // CheckCharacterActual().
    if (!basicError &&
        actualType.type().category() != TypeCategory::Character &&
        CanAssociateWithStorageSequence(dummy) &&
        !dummy.attrs.test(
            characteristics::DummyDataObject::Attr::DeducedFromActual)) {
````
- **L649 EN**: Transitions from the previous branch into an `else if` condition.
  **L649 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L650 EN**: Starts a function, method, lambda, or structured scope: `!dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {`.
  **L650 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {`。
- **L651 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L651 CN**: 开始 `if` 控制流语句并计算其条件。
- **L652 EN**: Continues logic associated with callable symbol `Warn`.
  **L652 CN**: 继续与可调用符号 `Warn` 相关的逻辑。
- **L653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::LanguageFeature::ContiguousOkForSeqAssociation,`.
  **L653 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::LanguageFeature::ContiguousOkForSeqAssociation,`。
- **L654 EN**: Executes a standalone statement or declaration: `"Element of contiguous assumed-shape array is accepted for storage sequence association"_port_en_US);`.
  **L654 CN**: 执行一条独立语句或声明：`"Element of contiguous assumed-shape array is accepted for storage sequence association"_port_en_US);`。
- **L655 EN**: Transitions from the previous branch into the alternative path.
  **L655 CN**: 从前一个分支过渡到备选路径。
- **L656 EN**: Executes a standalone statement or declaration: `basicError = true;`.
  **L656 CN**: 执行一条独立语句或声明：`basicError = true;`。
- **L657 EN**: Continues logic associated with callable symbol `Say`.
  **L657 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Element of assumed-shape array may not be associated with a %s array"_err_en_US,`.
  **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Element of assumed-shape array may not be associated with a %s array"_err_en_US,`。
- **L659 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L659 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L660 EN**: Closes the current lexical scope or compound statement.
  **L660 CN**: 结束当前词法作用域或复合语句块。
- **L661 EN**: Closes the current lexical scope or compound statement.
  **L661 CN**: 结束当前词法作用域或复合语句块。
- **L662 EN**: Closes the current lexical scope or compound statement.
  **L662 CN**: 结束当前词法作用域或复合语句块。
- **L663 EN**: Closes the current lexical scope or compound statement.
  **L663 CN**: 结束当前词法作用域或复合语句块。
- **L664 EN**: Closes the current lexical scope or compound statement.
  **L664 CN**: 结束当前词法作用域或复合语句块。
- **L665 EN**: Comment explains nearby logic, intent, or metadata: `Storage sequence association (F'2023 15.5.2.12p3) checks.`.
  **L665 CN**: 注释说明附近代码的逻辑、意图或元数据：`Storage sequence association (F'2023 15.5.2.12p3) checks.`。
- **L666 EN**: Comment explains nearby logic, intent, or metadata: `Character storage sequence association is checked in`.
  **L666 CN**: 注释说明附近代码的逻辑、意图或元数据：`Character storage sequence association is checked in`。
- **L667 EN**: Comment explains nearby logic, intent, or metadata: `CheckCharacterActual().`.
  **L667 CN**: 注释说明附近代码的逻辑、意图或元数据：`CheckCharacterActual().`。
- **L668 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L668 CN**: 开始 `if` 控制流语句并计算其条件。
- **L669 EN**: Continues logic associated with callable symbol `type`.
  **L669 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L670 EN**: Continues logic associated with callable symbol `CanAssociateWithStorageSequence`.
  **L670 CN**: 继续与可调用符号 `CanAssociateWithStorageSequence` 相关的逻辑。
- **L671 EN**: Continues logic associated with callable symbol `test`.
  **L671 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L672 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject::Attr::DeducedFromActual)) {`.
  **L672 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject::Attr::DeducedFromActual)) {`。

### Lines 673-696

````cpp
      if (auto dummySize{evaluate::ToInt64(evaluate::Fold(
              foldingContext, evaluate::GetSize(dummy.type.shape())))}) {
        if (actualIsAssumedRank) {
          if (!context.languageFeatures().IsEnabled(
                  common::LanguageFeature::AssumedRankPassedToNonAssumedRank)) {
            messages.Say(
                "Assumed-rank array may not be associated with a dummy argument that is not assumed-rank"_err_en_US);
          } else {
            foldingContext.Warn(
                common::LanguageFeature::AssumedRankPassedToNonAssumedRank,
                "Assumed-rank array should not be associated with a dummy argument that is not assumed-rank"_port_en_US);
          }
        } else if (actualRank == 0) {
          if (evaluate::IsArrayElement(actual)) {
            // Actual argument is a scalar array element
            evaluate::DesignatorFolder folder{
                context.foldingContext(), /*getLastComponent=*/true};
            if (auto actualOffset{folder.FoldDesignator(actual)}) {
              std::optional<std::int64_t> actualElements;
              if (IsAllocatableOrPointer(actualOffset->symbol())) {
                // don't use actualOffset->symbol().size()!
              } else if (static_cast<std::size_t>(actualOffset->offset()) >=
                      actualOffset->symbol().size() ||
                  !evaluate::IsContiguous(
````
- **L673 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L673 CN**: 开始 `if` 控制流语句并计算其条件。
- **L674 EN**: Starts a function, method, lambda, or structured scope: `foldingContext, evaluate::GetSize(dummy.type.shape())))}) {`.
  **L674 CN**: 开始一个函数、方法、lambda 或结构化作用域：`foldingContext, evaluate::GetSize(dummy.type.shape())))}) {`。
- **L675 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L675 CN**: 开始 `if` 控制流语句并计算其条件。
- **L676 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L676 CN**: 开始 `if` 控制流语句并计算其条件。
- **L677 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::AssumedRankPassedToNonAssumedRank)) {`.
  **L677 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::AssumedRankPassedToNonAssumedRank)) {`。
- **L678 EN**: Continues logic associated with callable symbol `Say`.
  **L678 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L679 EN**: Executes a standalone statement or declaration: `"Assumed-rank array may not be associated with a dummy argument that is not assumed-rank"_err_en_US);`.
  **L679 CN**: 执行一条独立语句或声明：`"Assumed-rank array may not be associated with a dummy argument that is not assumed-rank"_err_en_US);`。
- **L680 EN**: Transitions from the previous branch into the alternative path.
  **L680 CN**: 从前一个分支过渡到备选路径。
- **L681 EN**: Continues logic associated with callable symbol `Warn`.
  **L681 CN**: 继续与可调用符号 `Warn` 相关的逻辑。
- **L682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::LanguageFeature::AssumedRankPassedToNonAssumedRank,`.
  **L682 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::LanguageFeature::AssumedRankPassedToNonAssumedRank,`。
- **L683 EN**: Executes a standalone statement or declaration: `"Assumed-rank array should not be associated with a dummy argument that is not assumed-rank"_port_en_US);`.
  **L683 CN**: 执行一条独立语句或声明：`"Assumed-rank array should not be associated with a dummy argument that is not assumed-rank"_port_en_US);`。
- **L684 EN**: Closes the current lexical scope or compound statement.
  **L684 CN**: 结束当前词法作用域或复合语句块。
- **L685 EN**: Transitions from the previous branch into an `else if` condition.
  **L685 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L686 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L686 CN**: 开始 `if` 控制流语句并计算其条件。
- **L687 EN**: Comment explains nearby logic, intent, or metadata: `Actual argument is a scalar array element`.
  **L687 CN**: 注释说明附近代码的逻辑、意图或元数据：`Actual argument is a scalar array element`。
- **L688 EN**: Continues the surrounding expression or declaration: `evaluate::DesignatorFolder folder{`.
  **L688 CN**: 继续构造周围的表达式或声明：`evaluate::DesignatorFolder folder{`。
- **L689 EN**: Executes a call or declaration centered on `context.foldingContext`.
  **L689 CN**: 执行以 `context.foldingContext` 为核心的调用或声明。
- **L690 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L690 CN**: 开始 `if` 控制流语句并计算其条件。
- **L691 EN**: Executes a standalone statement or declaration: `std::optional<std::int64_t> actualElements;`.
  **L691 CN**: 执行一条独立语句或声明：`std::optional<std::int64_t> actualElements;`。
- **L692 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L692 CN**: 开始 `if` 控制流语句并计算其条件。
- **L693 EN**: Comment explains nearby logic, intent, or metadata: `don't use actualOffset->symbol().size()!`.
  **L693 CN**: 注释说明附近代码的逻辑、意图或元数据：`don't use actualOffset->symbol().size()!`。
- **L694 EN**: Transitions from the previous branch into an `else if` condition.
  **L694 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L695 EN**: Continues logic associated with callable symbol `symbol`.
  **L695 CN**: 继续与可调用符号 `symbol` 相关的逻辑。
- **L696 EN**: Continues logic associated with callable symbol `IsContiguous`.
  **L696 CN**: 继续与可调用符号 `IsContiguous` 相关的逻辑。

### Lines 697-720

````cpp
                      actualOffset->symbol(), foldingContext)
                      .value_or(false)) {
                actualElements = 1;
              } else if (auto actualSymType{evaluate::DynamicType::From(
                             actualOffset->symbol())}) {
                if (auto actualSymTypeBytes{
                        evaluate::ToInt64(evaluate::Fold(foldingContext,
                            actualSymType->MeasureSizeInBytes(
                                foldingContext, false)))};
                    actualSymTypeBytes && *actualSymTypeBytes > 0) {
                  actualElements = (static_cast<std::int64_t>(
                                        actualOffset->symbol().size()) -
                                       actualOffset->offset()) /
                      *actualSymTypeBytes;
                }
              }
              if (actualElements && *actualElements < *dummySize) {
                if (extentErrors) {
                  messages.Say(
                      "Actual argument has fewer elements remaining in storage sequence (%jd) than %s array (%jd)"_err_en_US,
                      static_cast<std::intmax_t>(*actualElements), dummyName,
                      static_cast<std::intmax_t>(*dummySize));
                } else {
                  foldingContext.Warn(common::UsageWarning::ShortArrayActual,
````
- **L697 EN**: Continues logic associated with callable symbol `symbol`.
  **L697 CN**: 继续与可调用符号 `symbol` 相关的逻辑。
- **L698 EN**: Starts a function, method, lambda, or structured scope: `.value_or(false)) {`.
  **L698 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.value_or(false)) {`。
- **L699 EN**: Executes a standalone statement or declaration: `actualElements = 1;`.
  **L699 CN**: 执行一条独立语句或声明：`actualElements = 1;`。
- **L700 EN**: Transitions from the previous branch into an `else if` condition.
  **L700 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L701 EN**: Starts a function, method, lambda, or structured scope: `actualOffset->symbol())}) {`.
  **L701 CN**: 开始一个函数、方法、lambda 或结构化作用域：`actualOffset->symbol())}) {`。
- **L702 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L702 CN**: 开始 `if` 控制流语句并计算其条件。
- **L703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::ToInt64(evaluate::Fold(foldingContext,`.
  **L703 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::ToInt64(evaluate::Fold(foldingContext,`。
- **L704 EN**: Continues logic associated with callable symbol `MeasureSizeInBytes`.
  **L704 CN**: 继续与可调用符号 `MeasureSizeInBytes` 相关的逻辑。
- **L705 EN**: Executes a standalone statement or declaration: `foldingContext, false)))};`.
  **L705 CN**: 执行一条独立语句或声明：`foldingContext, false)))};`。
- **L706 EN**: Continues the surrounding expression or declaration: `actualSymTypeBytes && *actualSymTypeBytes > 0) {`.
  **L706 CN**: 继续构造周围的表达式或声明：`actualSymTypeBytes && *actualSymTypeBytes > 0) {`。
- **L707 EN**: Continues logic associated with callable symbol `int64_t>`.
  **L707 CN**: 继续与可调用符号 `int64_t>` 相关的逻辑。
- **L708 EN**: Continues logic associated with callable symbol `symbol`.
  **L708 CN**: 继续与可调用符号 `symbol` 相关的逻辑。
- **L709 EN**: Continues logic associated with callable symbol `offset`.
  **L709 CN**: 继续与可调用符号 `offset` 相关的逻辑。
- **L710 EN**: Comment explains nearby logic, intent, or metadata: `actualSymTypeBytes;`.
  **L710 CN**: 注释说明附近代码的逻辑、意图或元数据：`actualSymTypeBytes;`。
- **L711 EN**: Closes the current lexical scope or compound statement.
  **L711 CN**: 结束当前词法作用域或复合语句块。
- **L712 EN**: Closes the current lexical scope or compound statement.
  **L712 CN**: 结束当前词法作用域或复合语句块。
- **L713 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L713 CN**: 开始 `if` 控制流语句并计算其条件。
- **L714 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L714 CN**: 开始 `if` 控制流语句并计算其条件。
- **L715 EN**: Continues logic associated with callable symbol `Say`.
  **L715 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument has fewer elements remaining in storage sequence (%jd) than %s array (%jd)"_err_en_US,`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument has fewer elements remaining in storage sequence (%jd) than %s array (%jd)"_err_en_US,`。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<std::intmax_t>(*actualElements), dummyName,`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<std::intmax_t>(*actualElements), dummyName,`。
- **L718 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L718 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L719 EN**: Transitions from the previous branch into the alternative path.
  **L719 CN**: 从前一个分支过渡到备选路径。
- **L720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::ShortArrayActual,`.
  **L720 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::ShortArrayActual,`。

### Lines 721-744

````cpp
                      "Actual argument has fewer elements remaining in storage sequence (%jd) than %s array (%jd)"_warn_en_US,
                      static_cast<std::intmax_t>(*actualElements), dummyName,
                      static_cast<std::intmax_t>(*dummySize));
                }
              }
            }
          }
        } else {
          if (auto actualSize{evaluate::ToInt64(evaluate::Fold(
                  foldingContext, evaluate::GetSize(actualType.shape())))};
              actualSize && *actualSize < *dummySize) {
            if (extentErrors) {
              messages.Say(
                  "Actual argument array has fewer elements (%jd) than %s array (%jd)"_err_en_US,
                  static_cast<std::intmax_t>(*actualSize), dummyName,
                  static_cast<std::intmax_t>(*dummySize));
            } else {
              foldingContext.Warn(common::UsageWarning::ShortArrayActual,
                  "Actual argument array has fewer elements (%jd) than %s array (%jd)"_warn_en_US,
                  static_cast<std::intmax_t>(*actualSize), dummyName,
                  static_cast<std::intmax_t>(*dummySize));
            }
          }
        }
````
- **L721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument has fewer elements remaining in storage sequence (%jd) than %s array (%jd)"_warn_en_US,`.
  **L721 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument has fewer elements remaining in storage sequence (%jd) than %s array (%jd)"_warn_en_US,`。
- **L722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<std::intmax_t>(*actualElements), dummyName,`.
  **L722 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<std::intmax_t>(*actualElements), dummyName,`。
- **L723 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L723 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L724 EN**: Closes the current lexical scope or compound statement.
  **L724 CN**: 结束当前词法作用域或复合语句块。
- **L725 EN**: Closes the current lexical scope or compound statement.
  **L725 CN**: 结束当前词法作用域或复合语句块。
- **L726 EN**: Closes the current lexical scope or compound statement.
  **L726 CN**: 结束当前词法作用域或复合语句块。
- **L727 EN**: Closes the current lexical scope or compound statement.
  **L727 CN**: 结束当前词法作用域或复合语句块。
- **L728 EN**: Transitions from the previous branch into the alternative path.
  **L728 CN**: 从前一个分支过渡到备选路径。
- **L729 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L729 CN**: 开始 `if` 控制流语句并计算其条件。
- **L730 EN**: Executes a call or declaration centered on `evaluate::GetSize`.
  **L730 CN**: 执行以 `evaluate::GetSize` 为核心的调用或声明。
- **L731 EN**: Continues the surrounding expression or declaration: `actualSize && *actualSize < *dummySize) {`.
  **L731 CN**: 继续构造周围的表达式或声明：`actualSize && *actualSize < *dummySize) {`。
- **L732 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L732 CN**: 开始 `if` 控制流语句并计算其条件。
- **L733 EN**: Continues logic associated with callable symbol `Say`.
  **L733 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument array has fewer elements (%jd) than %s array (%jd)"_err_en_US,`.
  **L734 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument array has fewer elements (%jd) than %s array (%jd)"_err_en_US,`。
- **L735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<std::intmax_t>(*actualSize), dummyName,`.
  **L735 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<std::intmax_t>(*actualSize), dummyName,`。
- **L736 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L736 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L737 EN**: Transitions from the previous branch into the alternative path.
  **L737 CN**: 从前一个分支过渡到备选路径。
- **L738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::ShortArrayActual,`.
  **L738 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::ShortArrayActual,`。
- **L739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument array has fewer elements (%jd) than %s array (%jd)"_warn_en_US,`.
  **L739 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument array has fewer elements (%jd) than %s array (%jd)"_warn_en_US,`。
- **L740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<std::intmax_t>(*actualSize), dummyName,`.
  **L740 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<std::intmax_t>(*actualSize), dummyName,`。
- **L741 EN**: Executes a call or declaration centered on `static_cast<std::intmax_t>`.
  **L741 CN**: 执行以 `static_cast<std::intmax_t>` 为核心的调用或声明。
- **L742 EN**: Closes the current lexical scope or compound statement.
  **L742 CN**: 结束当前词法作用域或复合语句块。
- **L743 EN**: Closes the current lexical scope or compound statement.
  **L743 CN**: 结束当前词法作用域或复合语句块。
- **L744 EN**: Closes the current lexical scope or compound statement.
  **L744 CN**: 结束当前词法作用域或复合语句块。

### Lines 745-768

````cpp
      }
    }
  }
  const ObjectEntityDetails *actualLastObject{actualLastSymbol
          ? actualLastSymbol->detailsIf<ObjectEntityDetails>()
          : nullptr};
  if (actualLastObject && actualLastObject->IsCoarray() &&
      dummy.attrs.test(characteristics::DummyDataObject::Attr::Allocatable) &&
      dummy.intent == common::Intent::Out &&
      !(intrinsic &&
          evaluate::AcceptsIntentOutAllocatableCoarray(
              intrinsic->name))) { // C846
    messages.Say(
        "ALLOCATABLE coarray '%s' may not be associated with INTENT(OUT) %s"_err_en_US,
        actualLastSymbol->name(), dummyName);
  }

  // Definability checking
  // Problems with polymorphism are caught in the callee's definition.
  if (scope) {
    std::optional<parser::MessageFixedText> undefinableMessage;
    DefinabilityFlags flags{DefinabilityFlag::PolymorphicOkInPure};
    if (dummy.intent == common::Intent::InOut) {
      flags.set(DefinabilityFlag::AllowEventLockOrNotifyType);
````
- **L745 EN**: Closes the current lexical scope or compound statement.
  **L745 CN**: 结束当前词法作用域或复合语句块。
- **L746 EN**: Closes the current lexical scope or compound statement.
  **L746 CN**: 结束当前词法作用域或复合语句块。
- **L747 EN**: Closes the current lexical scope or compound statement.
  **L747 CN**: 结束当前词法作用域或复合语句块。
- **L748 EN**: Continues the surrounding expression or declaration: `const ObjectEntityDetails *actualLastObject{actualLastSymbol`.
  **L748 CN**: 继续构造周围的表达式或声明：`const ObjectEntityDetails *actualLastObject{actualLastSymbol`。
- **L749 EN**: Continues logic associated with callable symbol `detailsIf<ObjectEntityDetails>`.
  **L749 CN**: 继续与可调用符号 `detailsIf<ObjectEntityDetails>` 相关的逻辑。
- **L750 EN**: Executes a standalone statement or declaration: `: nullptr};`.
  **L750 CN**: 执行一条独立语句或声明：`: nullptr};`。
- **L751 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L751 CN**: 开始 `if` 控制流语句并计算其条件。
- **L752 EN**: Continues logic associated with callable symbol `test`.
  **L752 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L753 EN**: Continues the surrounding expression or declaration: `dummy.intent == common::Intent::Out &&`.
  **L753 CN**: 继续构造周围的表达式或声明：`dummy.intent == common::Intent::Out &&`。
- **L754 EN**: Continues the surrounding expression or declaration: `!(intrinsic &&`.
  **L754 CN**: 继续构造周围的表达式或声明：`!(intrinsic &&`。
- **L755 EN**: Continues logic associated with callable symbol `AcceptsIntentOutAllocatableCoarray`.
  **L755 CN**: 继续与可调用符号 `AcceptsIntentOutAllocatableCoarray` 相关的逻辑。
- **L756 EN**: Continues the surrounding expression or declaration: `intrinsic->name))) { // C846`.
  **L756 CN**: 继续构造周围的表达式或声明：`intrinsic->name))) { // C846`。
- **L757 EN**: Continues logic associated with callable symbol `Say`.
  **L757 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ALLOCATABLE coarray '%s' may not be associated with INTENT(OUT) %s"_err_en_US,`.
  **L758 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ALLOCATABLE coarray '%s' may not be associated with INTENT(OUT) %s"_err_en_US,`。
- **L759 EN**: Executes a call or declaration centered on `actualLastSymbol->name`.
  **L759 CN**: 执行以 `actualLastSymbol->name` 为核心的调用或声明。
- **L760 EN**: Closes the current lexical scope or compound statement.
  **L760 CN**: 结束当前词法作用域或复合语句块。
- **L761 EN**: Blank line separating nearby declarations or logic blocks.
  **L761 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L762 EN**: Comment explains nearby logic, intent, or metadata: `Definability checking`.
  **L762 CN**: 注释说明附近代码的逻辑、意图或元数据：`Definability checking`。
- **L763 EN**: Comment explains nearby logic, intent, or metadata: `Problems with polymorphism are caught in the callee's definition.`.
  **L763 CN**: 注释说明附近代码的逻辑、意图或元数据：`Problems with polymorphism are caught in the callee's definition.`。
- **L764 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L764 CN**: 开始 `if` 控制流语句并计算其条件。
- **L765 EN**: Executes a standalone statement or declaration: `std::optional<parser::MessageFixedText> undefinableMessage;`.
  **L765 CN**: 执行一条独立语句或声明：`std::optional<parser::MessageFixedText> undefinableMessage;`。
- **L766 EN**: Executes a standalone statement or declaration: `DefinabilityFlags flags{DefinabilityFlag::PolymorphicOkInPure};`.
  **L766 CN**: 执行一条独立语句或声明：`DefinabilityFlags flags{DefinabilityFlag::PolymorphicOkInPure};`。
- **L767 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L767 CN**: 开始 `if` 控制流语句并计算其条件。
- **L768 EN**: Executes a call or declaration centered on `flags.set`.
  **L768 CN**: 执行以 `flags.set` 为核心的调用或声明。

### Lines 769-792

````cpp
      flags.set(DefinabilityFlag::OnlyWarnOnImpureFinalInPureContext);
      undefinableMessage =
          "Actual argument associated with INTENT(IN OUT) %s is not definable"_err_en_US;
    } else if (dummy.intent == common::Intent::Out) {
      undefinableMessage =
          "Actual argument associated with INTENT(OUT) %s is not definable"_err_en_US;
    } else if (context.ShouldWarn(common::LanguageFeature::
                       UndefinableAsynchronousOrVolatileActual)) {
      if (dummy.attrs.test(
              characteristics::DummyDataObject::Attr::Asynchronous)) {
        undefinableMessage =
            "Actual argument associated with ASYNCHRONOUS %s is not definable"_warn_en_US;
      } else if (dummy.attrs.test(
                     characteristics::DummyDataObject::Attr::Volatile)) {
        undefinableMessage =
            "Actual argument associated with VOLATILE %s is not definable"_warn_en_US;
      }
    }
    if (undefinableMessage) {
      if (isElemental) { // 15.5.2.4(21)
        flags.set(DefinabilityFlag::VectorSubscriptIsOk);
      }
      if (actualIsPointer && dummyIsPointer) { // 19.6.8
        flags.set(DefinabilityFlag::PointerDefinition);
````
- **L769 EN**: Executes a call or declaration centered on `flags.set`.
  **L769 CN**: 执行以 `flags.set` 为核心的调用或声明。
- **L770 EN**: Continues the surrounding expression or declaration: `undefinableMessage =`.
  **L770 CN**: 继续构造周围的表达式或声明：`undefinableMessage =`。
- **L771 EN**: Executes a call or declaration centered on `INTENT`.
  **L771 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L772 EN**: Transitions from the previous branch into an `else if` condition.
  **L772 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L773 EN**: Continues the surrounding expression or declaration: `undefinableMessage =`.
  **L773 CN**: 继续构造周围的表达式或声明：`undefinableMessage =`。
- **L774 EN**: Executes a call or declaration centered on `INTENT`.
  **L774 CN**: 执行以 `INTENT` 为核心的调用或声明。
- **L775 EN**: Transitions from the previous branch into an `else if` condition.
  **L775 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L776 EN**: Continues the surrounding expression or declaration: `UndefinableAsynchronousOrVolatileActual)) {`.
  **L776 CN**: 继续构造周围的表达式或声明：`UndefinableAsynchronousOrVolatileActual)) {`。
- **L777 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L777 CN**: 开始 `if` 控制流语句并计算其条件。
- **L778 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject::Attr::Asynchronous)) {`.
  **L778 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject::Attr::Asynchronous)) {`。
- **L779 EN**: Continues the surrounding expression or declaration: `undefinableMessage =`.
  **L779 CN**: 继续构造周围的表达式或声明：`undefinableMessage =`。
- **L780 EN**: Executes a standalone statement or declaration: `"Actual argument associated with ASYNCHRONOUS %s is not definable"_warn_en_US;`.
  **L780 CN**: 执行一条独立语句或声明：`"Actual argument associated with ASYNCHRONOUS %s is not definable"_warn_en_US;`。
- **L781 EN**: Transitions from the previous branch into an `else if` condition.
  **L781 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L782 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject::Attr::Volatile)) {`.
  **L782 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject::Attr::Volatile)) {`。
- **L783 EN**: Continues the surrounding expression or declaration: `undefinableMessage =`.
  **L783 CN**: 继续构造周围的表达式或声明：`undefinableMessage =`。
- **L784 EN**: Executes a standalone statement or declaration: `"Actual argument associated with VOLATILE %s is not definable"_warn_en_US;`.
  **L784 CN**: 执行一条独立语句或声明：`"Actual argument associated with VOLATILE %s is not definable"_warn_en_US;`。
- **L785 EN**: Closes the current lexical scope or compound statement.
  **L785 CN**: 结束当前词法作用域或复合语句块。
- **L786 EN**: Closes the current lexical scope or compound statement.
  **L786 CN**: 结束当前词法作用域或复合语句块。
- **L787 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L787 CN**: 开始 `if` 控制流语句并计算其条件。
- **L788 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L788 CN**: 开始 `if` 控制流语句并计算其条件。
- **L789 EN**: Executes a call or declaration centered on `flags.set`.
  **L789 CN**: 执行以 `flags.set` 为核心的调用或声明。
- **L790 EN**: Closes the current lexical scope or compound statement.
  **L790 CN**: 结束当前词法作用域或复合语句块。
- **L791 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L791 CN**: 开始 `if` 控制流语句并计算其条件。
- **L792 EN**: Executes a call or declaration centered on `flags.set`.
  **L792 CN**: 执行以 `flags.set` 为核心的调用或声明。

### Lines 793-816

````cpp
      }
      if (auto whyNot{WhyNotDefinable(messages.at(), *scope, flags, actual)}) {
        if (whyNot->IsFatal()) {
          if (auto *msg{messages.Say(*undefinableMessage, dummyName)}) {
            if (!msg->IsFatal()) {
              volatileOrAsyncNeedsTempDiagnosticIssued = true;
              msg->set_languageFeature(common::LanguageFeature::
                      UndefinableAsynchronousOrVolatileActual);
            }
            msg->Attach(
                std::move(whyNot->set_severity(parser::Severity::Because)));
          }
        } else {
          messages.Say(std::move(*whyNot));
        }
      }
    } else if (dummy.intent != common::Intent::In ||
        (dummyIsPointer && !actualIsPointer) ||
        (intrinsic && intrinsic->name == "loc")) {
      if (auto named{evaluate::ExtractNamedEntity(actual)}) {
        context.NoteDefinedSymbol(named->GetFirstSymbol());
      }
    }
  }
````
- **L793 EN**: Closes the current lexical scope or compound statement.
  **L793 CN**: 结束当前词法作用域或复合语句块。
- **L794 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L794 CN**: 开始 `if` 控制流语句并计算其条件。
- **L795 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L795 CN**: 开始 `if` 控制流语句并计算其条件。
- **L796 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L796 CN**: 开始 `if` 控制流语句并计算其条件。
- **L797 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L797 CN**: 开始 `if` 控制流语句并计算其条件。
- **L798 EN**: Executes a standalone statement or declaration: `volatileOrAsyncNeedsTempDiagnosticIssued = true;`.
  **L798 CN**: 执行一条独立语句或声明：`volatileOrAsyncNeedsTempDiagnosticIssued = true;`。
- **L799 EN**: Continues logic associated with callable symbol `set_languageFeature`.
  **L799 CN**: 继续与可调用符号 `set_languageFeature` 相关的逻辑。
- **L800 EN**: Executes a standalone statement or declaration: `UndefinableAsynchronousOrVolatileActual);`.
  **L800 CN**: 执行一条独立语句或声明：`UndefinableAsynchronousOrVolatileActual);`。
- **L801 EN**: Closes the current lexical scope or compound statement.
  **L801 CN**: 结束当前词法作用域或复合语句块。
- **L802 EN**: Continues logic associated with callable symbol `Attach`.
  **L802 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L803 EN**: Executes a call or declaration centered on `std::move`.
  **L803 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L804 EN**: Closes the current lexical scope or compound statement.
  **L804 CN**: 结束当前词法作用域或复合语句块。
- **L805 EN**: Transitions from the previous branch into the alternative path.
  **L805 CN**: 从前一个分支过渡到备选路径。
- **L806 EN**: Executes a call or declaration centered on `messages.Say`.
  **L806 CN**: 执行以 `messages.Say` 为核心的调用或声明。
- **L807 EN**: Closes the current lexical scope or compound statement.
  **L807 CN**: 结束当前词法作用域或复合语句块。
- **L808 EN**: Closes the current lexical scope or compound statement.
  **L808 CN**: 结束当前词法作用域或复合语句块。
- **L809 EN**: Transitions from the previous branch into an `else if` condition.
  **L809 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L810 EN**: Continues the surrounding expression or declaration: `(dummyIsPointer && !actualIsPointer) ||`.
  **L810 CN**: 继续构造周围的表达式或声明：`(dummyIsPointer && !actualIsPointer) ||`。
- **L811 EN**: Starts a function, method, lambda, or structured scope: `(intrinsic && intrinsic->name == "loc")) {`.
  **L811 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(intrinsic && intrinsic->name == "loc")) {`。
- **L812 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L812 CN**: 开始 `if` 控制流语句并计算其条件。
- **L813 EN**: Executes a call or declaration centered on `context.NoteDefinedSymbol`.
  **L813 CN**: 执行以 `context.NoteDefinedSymbol` 为核心的调用或声明。
- **L814 EN**: Closes the current lexical scope or compound statement.
  **L814 CN**: 结束当前词法作用域或复合语句块。
- **L815 EN**: Closes the current lexical scope or compound statement.
  **L815 CN**: 结束当前词法作用域或复合语句块。
- **L816 EN**: Closes the current lexical scope or compound statement.
  **L816 CN**: 结束当前词法作用域或复合语句块。

### Lines 817-840

````cpp

  bool dummyIsContiguous{
      dummy.attrs.test(characteristics::DummyDataObject::Attr::Contiguous)};
  bool actualIsContiguous{IsSimplyContiguous(actual, foldingContext)};

  // Cases when temporaries might be needed but must not be permitted.
  bool dummyIsAssumedShape{dummy.type.attrs().test(
      characteristics::TypeAndShape::Attr::AssumedShape)};
  bool copyOutNeeded{
      evaluate::ActualArgNeedsCopy(&arg, &dummyArg, foldingContext,
          /*forCopyOut=*/true)
          .value_or(false)};
  if (copyOutNeeded && !dummyIsValue &&
      (dummyIsAsynchronous || dummyIsVolatile)) {
    if (actualIsAsynchronous || actualIsVolatile) {
      if (actualCoarrayRef) { // F'2023 C1547
        messages.Say(
            "Coindexed ASYNCHRONOUS or VOLATILE actual argument may not be associated with %s with ASYNCHRONOUS or VOLATILE attributes unless VALUE"_err_en_US,
            dummyName);
        volatileOrAsyncNeedsTempDiagnosticIssued = true;
      }
      if ((actualRank > 0 || actualIsAssumedRank) && !actualIsContiguous) {
        if (dummyIsContiguous ||
            !(dummyIsAssumedShape || dummyIsAssumedRank ||
````
- **L817 EN**: Blank line separating nearby declarations or logic blocks.
  **L817 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L818 EN**: Continues the surrounding expression or declaration: `bool dummyIsContiguous{`.
  **L818 CN**: 继续构造周围的表达式或声明：`bool dummyIsContiguous{`。
- **L819 EN**: Executes a call or declaration centered on `dummy.attrs.test`.
  **L819 CN**: 执行以 `dummy.attrs.test` 为核心的调用或声明。
- **L820 EN**: Executes a call or declaration centered on `actualIsContiguous{IsSimplyContiguous`.
  **L820 CN**: 执行以 `actualIsContiguous{IsSimplyContiguous` 为核心的调用或声明。
- **L821 EN**: Blank line separating nearby declarations or logic blocks.
  **L821 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L822 EN**: Comment explains nearby logic, intent, or metadata: `Cases when temporaries might be needed but must not be permitted.`.
  **L822 CN**: 注释说明附近代码的逻辑、意图或元数据：`Cases when temporaries might be needed but must not be permitted.`。
- **L823 EN**: Continues logic associated with callable symbol `attrs`.
  **L823 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L824 EN**: Executes a standalone statement or declaration: `characteristics::TypeAndShape::Attr::AssumedShape)};`.
  **L824 CN**: 执行一条独立语句或声明：`characteristics::TypeAndShape::Attr::AssumedShape)};`。
- **L825 EN**: Continues the surrounding expression or declaration: `bool copyOutNeeded{`.
  **L825 CN**: 继续构造周围的表达式或声明：`bool copyOutNeeded{`。
- **L826 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::ActualArgNeedsCopy(&arg, &dummyArg, foldingContext,`.
  **L826 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::ActualArgNeedsCopy(&arg, &dummyArg, foldingContext,`。
- **L827 EN**: Comment explains nearby logic, intent, or metadata: `forCopyOut=*/true)`.
  **L827 CN**: 注释说明附近代码的逻辑、意图或元数据：`forCopyOut=*/true)`。
- **L828 EN**: Executes a call or declaration centered on `.value_or`.
  **L828 CN**: 执行以 `.value_or` 为核心的调用或声明。
- **L829 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L829 CN**: 开始 `if` 控制流语句并计算其条件。
- **L830 EN**: Starts a function, method, lambda, or structured scope: `(dummyIsAsynchronous || dummyIsVolatile)) {`.
  **L830 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(dummyIsAsynchronous || dummyIsVolatile)) {`。
- **L831 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L831 CN**: 开始 `if` 控制流语句并计算其条件。
- **L832 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L832 CN**: 开始 `if` 控制流语句并计算其条件。
- **L833 EN**: Continues logic associated with callable symbol `Say`.
  **L833 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Coindexed ASYNCHRONOUS or VOLATILE actual argument may not be associated with %s with ASYNCHRONOUS or VOLATILE attributes unless VALUE"_err_en_US,`.
  **L834 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Coindexed ASYNCHRONOUS or VOLATILE actual argument may not be associated with %s with ASYNCHRONOUS or VOLATILE attributes unless VALUE"_err_en_US,`。
- **L835 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L835 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L836 EN**: Executes a standalone statement or declaration: `volatileOrAsyncNeedsTempDiagnosticIssued = true;`.
  **L836 CN**: 执行一条独立语句或声明：`volatileOrAsyncNeedsTempDiagnosticIssued = true;`。
- **L837 EN**: Closes the current lexical scope or compound statement.
  **L837 CN**: 结束当前词法作用域或复合语句块。
- **L838 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L838 CN**: 开始 `if` 控制流语句并计算其条件。
- **L839 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L839 CN**: 开始 `if` 控制流语句并计算其条件。
- **L840 EN**: Continues the surrounding expression or declaration: `!(dummyIsAssumedShape || dummyIsAssumedRank ||`.
  **L840 CN**: 继续构造周围的表达式或声明：`!(dummyIsAssumedShape || dummyIsAssumedRank ||`。

### Lines 841-864

````cpp
                (actualIsPointer && dummyIsPointer))) { // F'2023 C1548 & C1549
          messages.Say(
              "ASYNCHRONOUS or VOLATILE actual argument that is not simply contiguous may not be associated with a contiguous ASYNCHRONOUS or VOLATILE %s"_err_en_US,
              dummyName);
          volatileOrAsyncNeedsTempDiagnosticIssued = true;
        }
      }
    } else if (!(dummyIsAssumedShape || dummyIsAssumedRank ||
                   (actualIsPointer && dummyIsPointer)) &&
        evaluate::IsArraySection(actual) && !actualIsContiguous &&
        !evaluate::HasVectorSubscript(actual)) {
      foldingContext.Warn(common::UsageWarning::VolatileOrAsynchronousTemporary,
          "The array section '%s' should not be associated with %s with %s attribute, unless the dummy is assumed-shape or assumed-rank"_warn_en_US,
          actual.AsFortran(), dummyName,
          dummyIsAsynchronous ? "ASYNCHRONOUS" : "VOLATILE");
      volatileOrAsyncNeedsTempDiagnosticIssued = true;
    }
  }
  // General implementation of F'23 15.5.2.5 note 5
  // Adds a less specific error message for any copy-out that could overwrite
  // a unread value in the actual argument.
  // Occurences of `volatileOrAsyncNeedsTempDiagnosticIssued = true` indicate a
  // more specific error message has already been issued. We might be able to
  // clean this up by switching the coding style of ActualArgNeedsCopy to be
````
- **L841 EN**: Continues the surrounding expression or declaration: `(actualIsPointer && dummyIsPointer))) { // F'2023 C1548 & C1549`.
  **L841 CN**: 继续构造周围的表达式或声明：`(actualIsPointer && dummyIsPointer))) { // F'2023 C1548 & C1549`。
- **L842 EN**: Continues logic associated with callable symbol `Say`.
  **L842 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ASYNCHRONOUS or VOLATILE actual argument that is not simply contiguous may not be associated with a contiguous ASYNCHRONOUS or VOLATILE %s"_err_en_US,`.
  **L843 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ASYNCHRONOUS or VOLATILE actual argument that is not simply contiguous may not be associated with a contiguous ASYNCHRONOUS or VOLATILE %s"_err_en_US,`。
- **L844 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L844 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L845 EN**: Executes a standalone statement or declaration: `volatileOrAsyncNeedsTempDiagnosticIssued = true;`.
  **L845 CN**: 执行一条独立语句或声明：`volatileOrAsyncNeedsTempDiagnosticIssued = true;`。
- **L846 EN**: Closes the current lexical scope or compound statement.
  **L846 CN**: 结束当前词法作用域或复合语句块。
- **L847 EN**: Closes the current lexical scope or compound statement.
  **L847 CN**: 结束当前词法作用域或复合语句块。
- **L848 EN**: Transitions from the previous branch into an `else if` condition.
  **L848 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L849 EN**: Continues the surrounding expression or declaration: `(actualIsPointer && dummyIsPointer)) &&`.
  **L849 CN**: 继续构造周围的表达式或声明：`(actualIsPointer && dummyIsPointer)) &&`。
- **L850 EN**: Continues logic associated with callable symbol `IsArraySection`.
  **L850 CN**: 继续与可调用符号 `IsArraySection` 相关的逻辑。
- **L851 EN**: Starts a function, method, lambda, or structured scope: `!evaluate::HasVectorSubscript(actual)) {`.
  **L851 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!evaluate::HasVectorSubscript(actual)) {`。
- **L852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::VolatileOrAsynchronousTemporary,`.
  **L852 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::VolatileOrAsynchronousTemporary,`。
- **L853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The array section '%s' should not be associated with %s with %s attribute, unless the dummy is assumed-shape or assumed-rank"_warn_en_US,`.
  **L853 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The array section '%s' should not be associated with %s with %s attribute, unless the dummy is assumed-shape or assumed-rank"_warn_en_US,`。
- **L854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `actual.AsFortran(), dummyName,`.
  **L854 CN**: 继续一个多行参数列表、初始化器或聚合项：`actual.AsFortran(), dummyName,`。
- **L855 EN**: Executes a standalone statement or declaration: `dummyIsAsynchronous ? "ASYNCHRONOUS" : "VOLATILE");`.
  **L855 CN**: 执行一条独立语句或声明：`dummyIsAsynchronous ? "ASYNCHRONOUS" : "VOLATILE");`。
- **L856 EN**: Executes a standalone statement or declaration: `volatileOrAsyncNeedsTempDiagnosticIssued = true;`.
  **L856 CN**: 执行一条独立语句或声明：`volatileOrAsyncNeedsTempDiagnosticIssued = true;`。
- **L857 EN**: Closes the current lexical scope or compound statement.
  **L857 CN**: 结束当前词法作用域或复合语句块。
- **L858 EN**: Closes the current lexical scope or compound statement.
  **L858 CN**: 结束当前词法作用域或复合语句块。
- **L859 EN**: Comment explains nearby logic, intent, or metadata: `General implementation of F'23 15.5.2.5 note 5`.
  **L859 CN**: 注释说明附近代码的逻辑、意图或元数据：`General implementation of F'23 15.5.2.5 note 5`。
- **L860 EN**: Comment explains nearby logic, intent, or metadata: `Adds a less specific error message for any copy-out that could overwrite`.
  **L860 CN**: 注释说明附近代码的逻辑、意图或元数据：`Adds a less specific error message for any copy-out that could overwrite`。
- **L861 EN**: Comment explains nearby logic, intent, or metadata: `a unread value in the actual argument.`.
  **L861 CN**: 注释说明附近代码的逻辑、意图或元数据：`a unread value in the actual argument.`。
- **L862 EN**: Comment explains nearby logic, intent, or metadata: `Occurences of `volatileOrAsyncNeedsTempDiagnosticIssued = true` indicate a`.
  **L862 CN**: 注释说明附近代码的逻辑、意图或元数据：`Occurences of `volatileOrAsyncNeedsTempDiagnosticIssued = true` indicate a`。
- **L863 EN**: Comment explains nearby logic, intent, or metadata: `more specific error message has already been issued. We might be able to`.
  **L863 CN**: 注释说明附近代码的逻辑、意图或元数据：`more specific error message has already been issued. We might be able to`。
- **L864 EN**: Comment explains nearby logic, intent, or metadata: `clean this up by switching the coding style of ActualArgNeedsCopy to be`.
  **L864 CN**: 注释说明附近代码的逻辑、意图或元数据：`clean this up by switching the coding style of ActualArgNeedsCopy to be`。

### Lines 865-888

````cpp
  // more like WhyNotDefinable.
  if (copyOutNeeded && !volatileOrAsyncNeedsTempDiagnosticIssued) {
    if ((actualIsVolatile || actualIsAsynchronous) &&
        (dummyIsVolatile || dummyIsAsynchronous)) {
      foldingContext.Warn(common::UsageWarning::VolatileOrAsynchronousTemporary,
          "The actual argument '%s' with %s attribute should not be associated with %s with %s attribute, because a temporary copy is required during the call"_warn_en_US,
          actual.AsFortran(), actualIsVolatile ? "VOLATILE" : "ASYNCHRONOUS",
          dummyName, dummyIsVolatile ? "VOLATILE" : "ASYNCHRONOUS");
    }
  }
  // If there are any cases where we don't need a copy and some other compiler
  // does, we issue a portability warning here.
  if (context.ShouldWarn(common::UsageWarning::Portability)) {
    // 3 other compilers error on this case even though it is ok.
    // Possibly as an over-restriction of F'23 C1548.
    if (!copyOutNeeded && !volatileOrAsyncNeedsTempDiagnosticIssued &&
        (!dummyIsValue && (dummyIsAsynchronous || dummyIsVolatile)) &&
        !(actualIsAsynchronous || actualIsVolatile) &&
        !(dummyIsAssumedShape || dummyIsAssumedRank ||
            (actualIsPointer && dummyIsPointer)) &&
        evaluate::IsArraySection(actual) &&
        !evaluate::HasVectorSubscript(actual)) {
      foldingContext.Warn(common::UsageWarning::Portability,
          "The array section '%s' should not be associated with %s with %s attribute, unless the dummy is assumed-shape or assumed-rank"_port_en_US,
````
- **L865 EN**: Comment explains nearby logic, intent, or metadata: `more like WhyNotDefinable.`.
  **L865 CN**: 注释说明附近代码的逻辑、意图或元数据：`more like WhyNotDefinable.`。
- **L866 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L866 CN**: 开始 `if` 控制流语句并计算其条件。
- **L867 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L867 CN**: 开始 `if` 控制流语句并计算其条件。
- **L868 EN**: Starts a function, method, lambda, or structured scope: `(dummyIsVolatile || dummyIsAsynchronous)) {`.
  **L868 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(dummyIsVolatile || dummyIsAsynchronous)) {`。
- **L869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::VolatileOrAsynchronousTemporary,`.
  **L869 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::VolatileOrAsynchronousTemporary,`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The actual argument '%s' with %s attribute should not be associated with %s with %s attribute, because a temporary copy is required during the call"_warn_en_US,`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The actual argument '%s' with %s attribute should not be associated with %s with %s attribute, because a temporary copy is required during the call"_warn_en_US,`。
- **L871 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `actual.AsFortran(), actualIsVolatile ? "VOLATILE" : "ASYNCHRONOUS",`.
  **L871 CN**: 继续一个多行参数列表、初始化器或聚合项：`actual.AsFortran(), actualIsVolatile ? "VOLATILE" : "ASYNCHRONOUS",`。
- **L872 EN**: Executes a standalone statement or declaration: `dummyName, dummyIsVolatile ? "VOLATILE" : "ASYNCHRONOUS");`.
  **L872 CN**: 执行一条独立语句或声明：`dummyName, dummyIsVolatile ? "VOLATILE" : "ASYNCHRONOUS");`。
- **L873 EN**: Closes the current lexical scope or compound statement.
  **L873 CN**: 结束当前词法作用域或复合语句块。
- **L874 EN**: Closes the current lexical scope or compound statement.
  **L874 CN**: 结束当前词法作用域或复合语句块。
- **L875 EN**: Comment explains nearby logic, intent, or metadata: `If there are any cases where we don't need a copy and some other compiler`.
  **L875 CN**: 注释说明附近代码的逻辑、意图或元数据：`If there are any cases where we don't need a copy and some other compiler`。
- **L876 EN**: Comment explains nearby logic, intent, or metadata: `does, we issue a portability warning here.`.
  **L876 CN**: 注释说明附近代码的逻辑、意图或元数据：`does, we issue a portability warning here.`。
- **L877 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L877 CN**: 开始 `if` 控制流语句并计算其条件。
- **L878 EN**: Comment explains nearby logic, intent, or metadata: `3 other compilers error on this case even though it is ok.`.
  **L878 CN**: 注释说明附近代码的逻辑、意图或元数据：`3 other compilers error on this case even though it is ok.`。
- **L879 EN**: Comment explains nearby logic, intent, or metadata: `Possibly as an over-restriction of F'23 C1548.`.
  **L879 CN**: 注释说明附近代码的逻辑、意图或元数据：`Possibly as an over-restriction of F'23 C1548.`。
- **L880 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L880 CN**: 开始 `if` 控制流语句并计算其条件。
- **L881 EN**: Continues the surrounding expression or declaration: `(!dummyIsValue && (dummyIsAsynchronous || dummyIsVolatile)) &&`.
  **L881 CN**: 继续构造周围的表达式或声明：`(!dummyIsValue && (dummyIsAsynchronous || dummyIsVolatile)) &&`。
- **L882 EN**: Continues the surrounding expression or declaration: `!(actualIsAsynchronous || actualIsVolatile) &&`.
  **L882 CN**: 继续构造周围的表达式或声明：`!(actualIsAsynchronous || actualIsVolatile) &&`。
- **L883 EN**: Continues the surrounding expression or declaration: `!(dummyIsAssumedShape || dummyIsAssumedRank ||`.
  **L883 CN**: 继续构造周围的表达式或声明：`!(dummyIsAssumedShape || dummyIsAssumedRank ||`。
- **L884 EN**: Continues the surrounding expression or declaration: `(actualIsPointer && dummyIsPointer)) &&`.
  **L884 CN**: 继续构造周围的表达式或声明：`(actualIsPointer && dummyIsPointer)) &&`。
- **L885 EN**: Continues logic associated with callable symbol `IsArraySection`.
  **L885 CN**: 继续与可调用符号 `IsArraySection` 相关的逻辑。
- **L886 EN**: Starts a function, method, lambda, or structured scope: `!evaluate::HasVectorSubscript(actual)) {`.
  **L886 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!evaluate::HasVectorSubscript(actual)) {`。
- **L887 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::Portability,`.
  **L887 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::Portability,`。
- **L888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The array section '%s' should not be associated with %s with %s attribute, unless the dummy is assumed-shape or assumed-rank"_port_en_US,`.
  **L888 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The array section '%s' should not be associated with %s with %s attribute, unless the dummy is assumed-shape or assumed-rank"_port_en_US,`。

### Lines 889-912

````cpp
          actual.AsFortran(), dummyName,
          dummyIsAsynchronous ? "ASYNCHRONOUS" : "VOLATILE");
    }
    // Possibly an over-restriction of F'23 15.5.2.5 note 5
    if (copyOutNeeded && !volatileOrAsyncNeedsTempDiagnosticIssued) {
      if ((dummyIsVolatile && !actualIsVolatile && !actualIsAsynchronous) ||
          (dummyIsAsynchronous && !actualIsVolatile && !actualIsAsynchronous)) {
        foldingContext.Warn(common::UsageWarning::Portability,
            "The actual argument '%s' should not be associated with %s with %s attribute, because a temporary copy is required during the call"_port_en_US,
            actual.AsFortran(), dummyName,
            dummyIsVolatile ? "VOLATILE" : "ASYNCHRONOUS");
      }
    }
  }

  // 15.5.2.6 -- dummy is ALLOCATABLE
  bool dummyIsOptional{
      dummy.attrs.test(characteristics::DummyDataObject::Attr::Optional)};
  if (dummyIsAllocatable) {
    if (actualIsAllocatable) {
      if (actualCoarrayRef && dummy.intent != common::Intent::In) {
        messages.Say(
            "ALLOCATABLE %s must have INTENT(IN) to be associated with a coindexed actual argument"_err_en_US,
            dummyName);
````
- **L889 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `actual.AsFortran(), dummyName,`.
  **L889 CN**: 继续一个多行参数列表、初始化器或聚合项：`actual.AsFortran(), dummyName,`。
- **L890 EN**: Executes a standalone statement or declaration: `dummyIsAsynchronous ? "ASYNCHRONOUS" : "VOLATILE");`.
  **L890 CN**: 执行一条独立语句或声明：`dummyIsAsynchronous ? "ASYNCHRONOUS" : "VOLATILE");`。
- **L891 EN**: Closes the current lexical scope or compound statement.
  **L891 CN**: 结束当前词法作用域或复合语句块。
- **L892 EN**: Comment explains nearby logic, intent, or metadata: `Possibly an over-restriction of F'23 15.5.2.5 note 5`.
  **L892 CN**: 注释说明附近代码的逻辑、意图或元数据：`Possibly an over-restriction of F'23 15.5.2.5 note 5`。
- **L893 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L893 CN**: 开始 `if` 控制流语句并计算其条件。
- **L894 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L894 CN**: 开始 `if` 控制流语句并计算其条件。
- **L895 EN**: Starts a function, method, lambda, or structured scope: `(dummyIsAsynchronous && !actualIsVolatile && !actualIsAsynchronous)) {`.
  **L895 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(dummyIsAsynchronous && !actualIsVolatile && !actualIsAsynchronous)) {`。
- **L896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::Portability,`.
  **L896 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::Portability,`。
- **L897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The actual argument '%s' should not be associated with %s with %s attribute, because a temporary copy is required during the call"_port_en_US,`.
  **L897 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The actual argument '%s' should not be associated with %s with %s attribute, because a temporary copy is required during the call"_port_en_US,`。
- **L898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `actual.AsFortran(), dummyName,`.
  **L898 CN**: 继续一个多行参数列表、初始化器或聚合项：`actual.AsFortran(), dummyName,`。
- **L899 EN**: Executes a standalone statement or declaration: `dummyIsVolatile ? "VOLATILE" : "ASYNCHRONOUS");`.
  **L899 CN**: 执行一条独立语句或声明：`dummyIsVolatile ? "VOLATILE" : "ASYNCHRONOUS");`。
- **L900 EN**: Closes the current lexical scope or compound statement.
  **L900 CN**: 结束当前词法作用域或复合语句块。
- **L901 EN**: Closes the current lexical scope or compound statement.
  **L901 CN**: 结束当前词法作用域或复合语句块。
- **L902 EN**: Closes the current lexical scope or compound statement.
  **L902 CN**: 结束当前词法作用域或复合语句块。
- **L903 EN**: Blank line separating nearby declarations or logic blocks.
  **L903 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L904 EN**: Comment explains nearby logic, intent, or metadata: `15.5.2.6 -- dummy is ALLOCATABLE`.
  **L904 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.5.2.6 -- dummy is ALLOCATABLE`。
- **L905 EN**: Continues the surrounding expression or declaration: `bool dummyIsOptional{`.
  **L905 CN**: 继续构造周围的表达式或声明：`bool dummyIsOptional{`。
- **L906 EN**: Executes a call or declaration centered on `dummy.attrs.test`.
  **L906 CN**: 执行以 `dummy.attrs.test` 为核心的调用或声明。
- **L907 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L907 CN**: 开始 `if` 控制流语句并计算其条件。
- **L908 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L908 CN**: 开始 `if` 控制流语句并计算其条件。
- **L909 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L909 CN**: 开始 `if` 控制流语句并计算其条件。
- **L910 EN**: Continues logic associated with callable symbol `Say`.
  **L910 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L911 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ALLOCATABLE %s must have INTENT(IN) to be associated with a coindexed actual argument"_err_en_US,`.
  **L911 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ALLOCATABLE %s must have INTENT(IN) to be associated with a coindexed actual argument"_err_en_US,`。
- **L912 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L912 CN**: 执行一条独立语句或声明：`dummyName);`。

### Lines 913-936

````cpp
      }
      if (!actualCoarrayRef && actualLastSymbol && dummy.type.corank() == 0 &&
          actualLastSymbol->Corank() > 0) {
        messages.Say(
            "ALLOCATABLE %s is not a coarray but actual argument has corank %d"_err_en_US,
            dummyName, actualLastSymbol->Corank());
      }
    } else if (evaluate::IsBareNullPointer(&actual)) {
      if (dummyIsOptional) {
      } else if (dummy.intent == common::Intent::Default &&
          context.ShouldWarn(
              common::UsageWarning::NullActualForDefaultIntentAllocatable)) {
        messages.Say(
            "A null pointer should not be associated with allocatable %s without INTENT(IN)"_warn_en_US,
            dummyName);
      } else if (dummy.intent == common::Intent::In) {
        foldingContext.Warn(common::LanguageFeature::NullActualForAllocatable,
            "Allocatable %s is associated with a null pointer"_port_en_US,
            dummyName);
      }
      // INTENT(OUT) and INTENT(IN OUT) cases are caught elsewhere as being
      // undefinable actual arguments.
    } else if (evaluate::IsNullAllocatable(&actual)) {
      if (dummyIsOptional) {
````
- **L913 EN**: Closes the current lexical scope or compound statement.
  **L913 CN**: 结束当前词法作用域或复合语句块。
- **L914 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L914 CN**: 开始 `if` 控制流语句并计算其条件。
- **L915 EN**: Starts a function, method, lambda, or structured scope: `actualLastSymbol->Corank() > 0) {`.
  **L915 CN**: 开始一个函数、方法、lambda 或结构化作用域：`actualLastSymbol->Corank() > 0) {`。
- **L916 EN**: Continues logic associated with callable symbol `Say`.
  **L916 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ALLOCATABLE %s is not a coarray but actual argument has corank %d"_err_en_US,`.
  **L917 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ALLOCATABLE %s is not a coarray but actual argument has corank %d"_err_en_US,`。
- **L918 EN**: Executes a call or declaration centered on `actualLastSymbol->Corank`.
  **L918 CN**: 执行以 `actualLastSymbol->Corank` 为核心的调用或声明。
- **L919 EN**: Closes the current lexical scope or compound statement.
  **L919 CN**: 结束当前词法作用域或复合语句块。
- **L920 EN**: Transitions from the previous branch into an `else if` condition.
  **L920 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L921 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L921 CN**: 开始 `if` 控制流语句并计算其条件。
- **L922 EN**: Transitions from the previous branch into an `else if` condition.
  **L922 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L923 EN**: Continues logic associated with callable symbol `ShouldWarn`.
  **L923 CN**: 继续与可调用符号 `ShouldWarn` 相关的逻辑。
- **L924 EN**: Continues the surrounding expression or declaration: `common::UsageWarning::NullActualForDefaultIntentAllocatable)) {`.
  **L924 CN**: 继续构造周围的表达式或声明：`common::UsageWarning::NullActualForDefaultIntentAllocatable)) {`。
- **L925 EN**: Continues logic associated with callable symbol `Say`.
  **L925 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"A null pointer should not be associated with allocatable %s without INTENT(IN)"_warn_en_US,`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`"A null pointer should not be associated with allocatable %s without INTENT(IN)"_warn_en_US,`。
- **L927 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L927 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L928 EN**: Transitions from the previous branch into an `else if` condition.
  **L928 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::LanguageFeature::NullActualForAllocatable,`.
  **L929 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::LanguageFeature::NullActualForAllocatable,`。
- **L930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Allocatable %s is associated with a null pointer"_port_en_US,`.
  **L930 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Allocatable %s is associated with a null pointer"_port_en_US,`。
- **L931 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L931 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L932 EN**: Closes the current lexical scope or compound statement.
  **L932 CN**: 结束当前词法作用域或复合语句块。
- **L933 EN**: Comment explains nearby logic, intent, or metadata: `INTENT(OUT) and INTENT(IN OUT) cases are caught elsewhere as being`.
  **L933 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTENT(OUT) and INTENT(IN OUT) cases are caught elsewhere as being`。
- **L934 EN**: Comment explains nearby logic, intent, or metadata: `undefinable actual arguments.`.
  **L934 CN**: 注释说明附近代码的逻辑、意图或元数据：`undefinable actual arguments.`。
- **L935 EN**: Transitions from the previous branch into an `else if` condition.
  **L935 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L936 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L936 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 937-960

````cpp
      } else if (dummy.intent == common::Intent::Default &&
          context.ShouldWarn(
              common::UsageWarning::NullActualForDefaultIntentAllocatable)) {
        messages.Say(
            "A null allocatable should not be associated with allocatable %s without INTENT(IN)"_warn_en_US,
            dummyName);
      }
      // INTENT(OUT) and INTENT(IN OUT) cases are caught elsewhere
    } else if (!actualIsAllocatable &&
        !dummy.ignoreTKR.test(common::IgnoreTKR::Pointer)) {
      messages.Say(
          "ALLOCATABLE %s must be associated with an ALLOCATABLE actual argument"_err_en_US,
          dummyName);
    }
  }

  // 15.5.2.7 -- dummy is POINTER
  if (dummyIsPointer) {
    if (actualIsPointer || dummy.intent == common::Intent::In) {
      if (scope) {
        semantics::CheckPointerAssignment(context, messages.at(), dummyName,
            dummy, actual, *scope,
            /*isAssumedRank=*/dummyIsAssumedRank, actualIsPointer);
      }
````
- **L937 EN**: Transitions from the previous branch into an `else if` condition.
  **L937 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L938 EN**: Continues logic associated with callable symbol `ShouldWarn`.
  **L938 CN**: 继续与可调用符号 `ShouldWarn` 相关的逻辑。
- **L939 EN**: Continues the surrounding expression or declaration: `common::UsageWarning::NullActualForDefaultIntentAllocatable)) {`.
  **L939 CN**: 继续构造周围的表达式或声明：`common::UsageWarning::NullActualForDefaultIntentAllocatable)) {`。
- **L940 EN**: Continues logic associated with callable symbol `Say`.
  **L940 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L941 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"A null allocatable should not be associated with allocatable %s without INTENT(IN)"_warn_en_US,`.
  **L941 CN**: 继续一个多行参数列表、初始化器或聚合项：`"A null allocatable should not be associated with allocatable %s without INTENT(IN)"_warn_en_US,`。
- **L942 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L942 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L943 EN**: Closes the current lexical scope or compound statement.
  **L943 CN**: 结束当前词法作用域或复合语句块。
- **L944 EN**: Comment explains nearby logic, intent, or metadata: `INTENT(OUT) and INTENT(IN OUT) cases are caught elsewhere`.
  **L944 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTENT(OUT) and INTENT(IN OUT) cases are caught elsewhere`。
- **L945 EN**: Transitions from the previous branch into an `else if` condition.
  **L945 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L946 EN**: Starts a function, method, lambda, or structured scope: `!dummy.ignoreTKR.test(common::IgnoreTKR::Pointer)) {`.
  **L946 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!dummy.ignoreTKR.test(common::IgnoreTKR::Pointer)) {`。
- **L947 EN**: Continues logic associated with callable symbol `Say`.
  **L947 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ALLOCATABLE %s must be associated with an ALLOCATABLE actual argument"_err_en_US,`.
  **L948 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ALLOCATABLE %s must be associated with an ALLOCATABLE actual argument"_err_en_US,`。
- **L949 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L949 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L950 EN**: Closes the current lexical scope or compound statement.
  **L950 CN**: 结束当前词法作用域或复合语句块。
- **L951 EN**: Closes the current lexical scope or compound statement.
  **L951 CN**: 结束当前词法作用域或复合语句块。
- **L952 EN**: Blank line separating nearby declarations or logic blocks.
  **L952 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L953 EN**: Comment explains nearby logic, intent, or metadata: `15.5.2.7 -- dummy is POINTER`.
  **L953 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.5.2.7 -- dummy is POINTER`。
- **L954 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L954 CN**: 开始 `if` 控制流语句并计算其条件。
- **L955 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L955 CN**: 开始 `if` 控制流语句并计算其条件。
- **L956 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L956 CN**: 开始 `if` 控制流语句并计算其条件。
- **L957 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `semantics::CheckPointerAssignment(context, messages.at(), dummyName,`.
  **L957 CN**: 继续一个多行参数列表、初始化器或聚合项：`semantics::CheckPointerAssignment(context, messages.at(), dummyName,`。
- **L958 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dummy, actual, *scope,`.
  **L958 CN**: 继续一个多行参数列表、初始化器或聚合项：`dummy, actual, *scope,`。
- **L959 EN**: Comment explains nearby logic, intent, or metadata: `isAssumedRank=*/dummyIsAssumedRank, actualIsPointer);`.
  **L959 CN**: 注释说明附近代码的逻辑、意图或元数据：`isAssumedRank=*/dummyIsAssumedRank, actualIsPointer);`。
- **L960 EN**: Closes the current lexical scope or compound statement.
  **L960 CN**: 结束当前词法作用域或复合语句块。

### Lines 961-984

````cpp
    } else if (!actualIsPointer &&
        !dummy.ignoreTKR.test(common::IgnoreTKR::Pointer)) {
      messages.Say(
          "Actual argument associated with POINTER %s must also be POINTER unless INTENT(IN)"_err_en_US,
          dummyName);
    }
  }

  // 15.5.2.5 -- actual & dummy are both POINTER or both ALLOCATABLE
  // For INTENT(IN), and for a polymorphic actual being associated with a
  // monomorphic dummy, we relax two checks that are in Fortran to
  // prevent the callee from changing the type or to avoid having
  // to use a descriptor.
  if (!typesCompatible) {
    // Don't pile on the errors emitted above
  } else if ((actualIsPointer && dummyIsPointer) ||
      (actualIsAllocatable && dummyIsAllocatable)) {
    bool actualIsUnlimited{actualType.type().IsUnlimitedPolymorphic()};
    bool dummyIsUnlimited{dummy.type.type().IsUnlimitedPolymorphic()};
    bool checkTypeCompatibility{true};
    if (actualIsUnlimited != dummyIsUnlimited) {
      checkTypeCompatibility = false;
      if (dummyIsUnlimited && dummy.intent == common::Intent::In &&
          context.IsEnabled(common::LanguageFeature::RelaxedIntentInChecking)) {
````
- **L961 EN**: Transitions from the previous branch into an `else if` condition.
  **L961 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L962 EN**: Starts a function, method, lambda, or structured scope: `!dummy.ignoreTKR.test(common::IgnoreTKR::Pointer)) {`.
  **L962 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!dummy.ignoreTKR.test(common::IgnoreTKR::Pointer)) {`。
- **L963 EN**: Continues logic associated with callable symbol `Say`.
  **L963 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L964 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with POINTER %s must also be POINTER unless INTENT(IN)"_err_en_US,`.
  **L964 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with POINTER %s must also be POINTER unless INTENT(IN)"_err_en_US,`。
- **L965 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L965 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L966 EN**: Closes the current lexical scope or compound statement.
  **L966 CN**: 结束当前词法作用域或复合语句块。
- **L967 EN**: Closes the current lexical scope or compound statement.
  **L967 CN**: 结束当前词法作用域或复合语句块。
- **L968 EN**: Blank line separating nearby declarations or logic blocks.
  **L968 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L969 EN**: Comment explains nearby logic, intent, or metadata: `15.5.2.5 -- actual & dummy are both POINTER or both ALLOCATABLE`.
  **L969 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.5.2.5 -- actual & dummy are both POINTER or both ALLOCATABLE`。
- **L970 EN**: Comment explains nearby logic, intent, or metadata: `For INTENT(IN), and for a polymorphic actual being associated with a`.
  **L970 CN**: 注释说明附近代码的逻辑、意图或元数据：`For INTENT(IN), and for a polymorphic actual being associated with a`。
- **L971 EN**: Comment explains nearby logic, intent, or metadata: `monomorphic dummy, we relax two checks that are in Fortran to`.
  **L971 CN**: 注释说明附近代码的逻辑、意图或元数据：`monomorphic dummy, we relax two checks that are in Fortran to`。
- **L972 EN**: Comment explains nearby logic, intent, or metadata: `prevent the callee from changing the type or to avoid having`.
  **L972 CN**: 注释说明附近代码的逻辑、意图或元数据：`prevent the callee from changing the type or to avoid having`。
- **L973 EN**: Comment explains nearby logic, intent, or metadata: `to use a descriptor.`.
  **L973 CN**: 注释说明附近代码的逻辑、意图或元数据：`to use a descriptor.`。
- **L974 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L974 CN**: 开始 `if` 控制流语句并计算其条件。
- **L975 EN**: Comment explains nearby logic, intent, or metadata: `Don't pile on the errors emitted above`.
  **L975 CN**: 注释说明附近代码的逻辑、意图或元数据：`Don't pile on the errors emitted above`。
- **L976 EN**: Transitions from the previous branch into an `else if` condition.
  **L976 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L977 EN**: Starts a function, method, lambda, or structured scope: `(actualIsAllocatable && dummyIsAllocatable)) {`.
  **L977 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(actualIsAllocatable && dummyIsAllocatable)) {`。
- **L978 EN**: Executes a call or declaration centered on `actualIsUnlimited{actualType.type`.
  **L978 CN**: 执行以 `actualIsUnlimited{actualType.type` 为核心的调用或声明。
- **L979 EN**: Executes a call or declaration centered on `dummyIsUnlimited{dummy.type.type`.
  **L979 CN**: 执行以 `dummyIsUnlimited{dummy.type.type` 为核心的调用或声明。
- **L980 EN**: Executes a standalone statement or declaration: `bool checkTypeCompatibility{true};`.
  **L980 CN**: 执行一条独立语句或声明：`bool checkTypeCompatibility{true};`。
- **L981 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L981 CN**: 开始 `if` 控制流语句并计算其条件。
- **L982 EN**: Executes a standalone statement or declaration: `checkTypeCompatibility = false;`.
  **L982 CN**: 执行一条独立语句或声明：`checkTypeCompatibility = false;`。
- **L983 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L983 CN**: 开始 `if` 控制流语句并计算其条件。
- **L984 EN**: Starts a function, method, lambda, or structured scope: `context.IsEnabled(common::LanguageFeature::RelaxedIntentInChecking)) {`.
  **L984 CN**: 开始一个函数、方法、lambda 或结构化作用域：`context.IsEnabled(common::LanguageFeature::RelaxedIntentInChecking)) {`。

### Lines 985-1008

````cpp
        foldingContext.Warn(common::LanguageFeature::RelaxedIntentInChecking,
            "If a POINTER or ALLOCATABLE dummy or actual argument is unlimited polymorphic, both should be so"_port_en_US);
      } else {
        messages.Say(
            "If a POINTER or ALLOCATABLE dummy or actual argument is unlimited polymorphic, both must be so"_err_en_US);
      }
    } else if (dummyIsPolymorphic != actualIsPolymorphic) {
      if (dummyIsPolymorphic && dummy.intent == common::Intent::In &&
          context.IsEnabled(common::LanguageFeature::RelaxedIntentInChecking)) {
        foldingContext.Warn(common::LanguageFeature::RelaxedIntentInChecking,
            "If a POINTER or ALLOCATABLE dummy or actual argument is polymorphic, both should be so"_port_en_US);
      } else if (actualIsPolymorphic &&
          context.IsEnabled(common::LanguageFeature::
                  PolymorphicActualAllocatableOrPointerToMonomorphicDummy)) {
        foldingContext.Warn(
            common::LanguageFeature::
                PolymorphicActualAllocatableOrPointerToMonomorphicDummy,
            "If a POINTER or ALLOCATABLE actual argument is polymorphic, the corresponding dummy argument should also be so"_port_en_US);
      } else {
        checkTypeCompatibility = false;
        messages.Say(
            "If a POINTER or ALLOCATABLE dummy or actual argument is polymorphic, both must be so"_err_en_US);
      }
    } else if ((dummy.ignoreTKR.test(common::IgnoreTKR::Type) ||
````
- **L985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::LanguageFeature::RelaxedIntentInChecking,`.
  **L985 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::LanguageFeature::RelaxedIntentInChecking,`。
- **L986 EN**: Executes a standalone statement or declaration: `"If a POINTER or ALLOCATABLE dummy or actual argument is unlimited polymorphic, both should be so"_port_en_US);`.
  **L986 CN**: 执行一条独立语句或声明：`"If a POINTER or ALLOCATABLE dummy or actual argument is unlimited polymorphic, both should be so"_port_en_US);`。
- **L987 EN**: Transitions from the previous branch into the alternative path.
  **L987 CN**: 从前一个分支过渡到备选路径。
- **L988 EN**: Continues logic associated with callable symbol `Say`.
  **L988 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L989 EN**: Executes a standalone statement or declaration: `"If a POINTER or ALLOCATABLE dummy or actual argument is unlimited polymorphic, both must be so"_err_en_US);`.
  **L989 CN**: 执行一条独立语句或声明：`"If a POINTER or ALLOCATABLE dummy or actual argument is unlimited polymorphic, both must be so"_err_en_US);`。
- **L990 EN**: Closes the current lexical scope or compound statement.
  **L990 CN**: 结束当前词法作用域或复合语句块。
- **L991 EN**: Transitions from the previous branch into an `else if` condition.
  **L991 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L992 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L992 CN**: 开始 `if` 控制流语句并计算其条件。
- **L993 EN**: Starts a function, method, lambda, or structured scope: `context.IsEnabled(common::LanguageFeature::RelaxedIntentInChecking)) {`.
  **L993 CN**: 开始一个函数、方法、lambda 或结构化作用域：`context.IsEnabled(common::LanguageFeature::RelaxedIntentInChecking)) {`。
- **L994 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::LanguageFeature::RelaxedIntentInChecking,`.
  **L994 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::LanguageFeature::RelaxedIntentInChecking,`。
- **L995 EN**: Executes a standalone statement or declaration: `"If a POINTER or ALLOCATABLE dummy or actual argument is polymorphic, both should be so"_port_en_US);`.
  **L995 CN**: 执行一条独立语句或声明：`"If a POINTER or ALLOCATABLE dummy or actual argument is polymorphic, both should be so"_port_en_US);`。
- **L996 EN**: Transitions from the previous branch into an `else if` condition.
  **L996 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L997 EN**: Continues logic associated with callable symbol `IsEnabled`.
  **L997 CN**: 继续与可调用符号 `IsEnabled` 相关的逻辑。
- **L998 EN**: Continues the surrounding expression or declaration: `PolymorphicActualAllocatableOrPointerToMonomorphicDummy)) {`.
  **L998 CN**: 继续构造周围的表达式或声明：`PolymorphicActualAllocatableOrPointerToMonomorphicDummy)) {`。
- **L999 EN**: Continues logic associated with callable symbol `Warn`.
  **L999 CN**: 继续与可调用符号 `Warn` 相关的逻辑。
- **L1000 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::`.
  **L1000 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::`。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PolymorphicActualAllocatableOrPointerToMonomorphicDummy,`.
  **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`PolymorphicActualAllocatableOrPointerToMonomorphicDummy,`。
- **L1002 EN**: Executes a standalone statement or declaration: `"If a POINTER or ALLOCATABLE actual argument is polymorphic, the corresponding dummy argument should also be so"_port_en_US);`.
  **L1002 CN**: 执行一条独立语句或声明：`"If a POINTER or ALLOCATABLE actual argument is polymorphic, the corresponding dummy argument should also be so"_port_en_US);`。
- **L1003 EN**: Transitions from the previous branch into the alternative path.
  **L1003 CN**: 从前一个分支过渡到备选路径。
- **L1004 EN**: Executes a standalone statement or declaration: `checkTypeCompatibility = false;`.
  **L1004 CN**: 执行一条独立语句或声明：`checkTypeCompatibility = false;`。
- **L1005 EN**: Continues logic associated with callable symbol `Say`.
  **L1005 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1006 EN**: Executes a standalone statement or declaration: `"If a POINTER or ALLOCATABLE dummy or actual argument is polymorphic, both must be so"_err_en_US);`.
  **L1006 CN**: 执行一条独立语句或声明：`"If a POINTER or ALLOCATABLE dummy or actual argument is polymorphic, both must be so"_err_en_US);`。
- **L1007 EN**: Closes the current lexical scope or compound statement.
  **L1007 CN**: 结束当前词法作用域或复合语句块。
- **L1008 EN**: Transitions from the previous branch into an `else if` condition.
  **L1008 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 1009-1032

````cpp
                   dummy.ignoreTKR.test(common::IgnoreTKR::Kind)) &&
        dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {
      // Descriptor based dummy args passed with ignore_tkr(tc) or
      // ignore_tkr(kc) are allowed to have type and kind differences
      checkTypeCompatibility = false;
    }
    if (checkTypeCompatibility && !actualIsUnlimited) {
      if (!actualType.type().IsTkCompatibleWith(dummy.type.type())) {
        if (dummy.intent == common::Intent::In &&
            context.IsEnabled(
                common::LanguageFeature::RelaxedIntentInChecking)) {
          foldingContext.Warn(common::LanguageFeature::RelaxedIntentInChecking,
              "POINTER or ALLOCATABLE dummy and actual arguments should have the same declared type and kind"_port_en_US);
        } else {
          messages.Say(
              "POINTER or ALLOCATABLE dummy and actual arguments must have the same declared type and kind"_err_en_US);
        }
      }
      // 15.5.2.5(4)
      const auto *dummyDerived{evaluate::GetDerivedTypeSpec(dummy.type.type())};
      if (!DefersSameTypeParameters(actualDerived, dummyDerived) ||
          dummy.type.type().HasDeferredTypeParameter() !=
              actualType.type().HasDeferredTypeParameter()) {
        messages.Say(
````
- **L1009 EN**: Continues logic associated with callable symbol `test`.
  **L1009 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1010 EN**: Starts a function, method, lambda, or structured scope: `dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {`.
  **L1010 CN**: 开始一个函数、方法、lambda 或结构化作用域：`dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {`。
- **L1011 EN**: Comment explains nearby logic, intent, or metadata: `Descriptor based dummy args passed with ignore_tkr(tc) or`.
  **L1011 CN**: 注释说明附近代码的逻辑、意图或元数据：`Descriptor based dummy args passed with ignore_tkr(tc) or`。
- **L1012 EN**: Comment explains nearby logic, intent, or metadata: `ignore_tkr(kc) are allowed to have type and kind differences`.
  **L1012 CN**: 注释说明附近代码的逻辑、意图或元数据：`ignore_tkr(kc) are allowed to have type and kind differences`。
- **L1013 EN**: Executes a standalone statement or declaration: `checkTypeCompatibility = false;`.
  **L1013 CN**: 执行一条独立语句或声明：`checkTypeCompatibility = false;`。
- **L1014 EN**: Closes the current lexical scope or compound statement.
  **L1014 CN**: 结束当前词法作用域或复合语句块。
- **L1015 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1015 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1016 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1016 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1017 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1017 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1018 EN**: Continues logic associated with callable symbol `IsEnabled`.
  **L1018 CN**: 继续与可调用符号 `IsEnabled` 相关的逻辑。
- **L1019 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::RelaxedIntentInChecking)) {`.
  **L1019 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::RelaxedIntentInChecking)) {`。
- **L1020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::LanguageFeature::RelaxedIntentInChecking,`.
  **L1020 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::LanguageFeature::RelaxedIntentInChecking,`。
- **L1021 EN**: Executes a standalone statement or declaration: `"POINTER or ALLOCATABLE dummy and actual arguments should have the same declared type and kind"_port_en_US);`.
  **L1021 CN**: 执行一条独立语句或声明：`"POINTER or ALLOCATABLE dummy and actual arguments should have the same declared type and kind"_port_en_US);`。
- **L1022 EN**: Transitions from the previous branch into the alternative path.
  **L1022 CN**: 从前一个分支过渡到备选路径。
- **L1023 EN**: Continues logic associated with callable symbol `Say`.
  **L1023 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1024 EN**: Executes a standalone statement or declaration: `"POINTER or ALLOCATABLE dummy and actual arguments must have the same declared type and kind"_err_en_US);`.
  **L1024 CN**: 执行一条独立语句或声明：`"POINTER or ALLOCATABLE dummy and actual arguments must have the same declared type and kind"_err_en_US);`。
- **L1025 EN**: Closes the current lexical scope or compound statement.
  **L1025 CN**: 结束当前词法作用域或复合语句块。
- **L1026 EN**: Closes the current lexical scope or compound statement.
  **L1026 CN**: 结束当前词法作用域或复合语句块。
- **L1027 EN**: Comment explains nearby logic, intent, or metadata: `15.5.2.5(4)`.
  **L1027 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.5.2.5(4)`。
- **L1028 EN**: Executes a call or declaration centered on `*dummyDerived{evaluate::GetDerivedTypeSpec`.
  **L1028 CN**: 执行以 `*dummyDerived{evaluate::GetDerivedTypeSpec` 为核心的调用或声明。
- **L1029 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1029 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1030 EN**: Continues logic associated with callable symbol `type`.
  **L1030 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L1031 EN**: Starts a function, method, lambda, or structured scope: `actualType.type().HasDeferredTypeParameter()) {`.
  **L1031 CN**: 开始一个函数、方法、lambda 或结构化作用域：`actualType.type().HasDeferredTypeParameter()) {`。
- **L1032 EN**: Continues logic associated with callable symbol `Say`.
  **L1032 CN**: 继续与可调用符号 `Say` 相关的逻辑。

### Lines 1033-1056

````cpp
            "Dummy and actual arguments must defer the same type parameters when POINTER or ALLOCATABLE"_err_en_US);
      }
    }
  }

  // 15.5.2.8 -- coarray dummy arguments
  if (dummy.type.corank() > 0) {
    if (actualType.corank() == 0) {
      messages.Say(
          "Actual argument associated with coarray %s must be a coarray"_err_en_US,
          dummyName);
    } else if (actualType.corank() != dummy.type.corank() &&
        dummyIsAllocatableOrPointer) {
      messages.Say(
          "ALLOCATABLE or POINTER %s has corank %d but actual argument has corank %d"_err_en_US,
          dummyName, dummy.type.corank(), actualType.corank());
    }
    if (dummyIsVolatile) {
      if (!actualIsVolatile) {
        messages.Say(
            "non-VOLATILE coarray may not be associated with VOLATILE coarray %s"_err_en_US,
            dummyName);
      }
    } else {
````
- **L1033 EN**: Executes a standalone statement or declaration: `"Dummy and actual arguments must defer the same type parameters when POINTER or ALLOCATABLE"_err_en_US);`.
  **L1033 CN**: 执行一条独立语句或声明：`"Dummy and actual arguments must defer the same type parameters when POINTER or ALLOCATABLE"_err_en_US);`。
- **L1034 EN**: Closes the current lexical scope or compound statement.
  **L1034 CN**: 结束当前词法作用域或复合语句块。
- **L1035 EN**: Closes the current lexical scope or compound statement.
  **L1035 CN**: 结束当前词法作用域或复合语句块。
- **L1036 EN**: Closes the current lexical scope or compound statement.
  **L1036 CN**: 结束当前词法作用域或复合语句块。
- **L1037 EN**: Blank line separating nearby declarations or logic blocks.
  **L1037 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1038 EN**: Comment explains nearby logic, intent, or metadata: `15.5.2.8 -- coarray dummy arguments`.
  **L1038 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.5.2.8 -- coarray dummy arguments`。
- **L1039 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1039 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1040 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1040 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1041 EN**: Continues logic associated with callable symbol `Say`.
  **L1041 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1042 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with coarray %s must be a coarray"_err_en_US,`.
  **L1042 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with coarray %s must be a coarray"_err_en_US,`。
- **L1043 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1043 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1044 EN**: Transitions from the previous branch into an `else if` condition.
  **L1044 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1045 EN**: Continues the surrounding expression or declaration: `dummyIsAllocatableOrPointer) {`.
  **L1045 CN**: 继续构造周围的表达式或声明：`dummyIsAllocatableOrPointer) {`。
- **L1046 EN**: Continues logic associated with callable symbol `Say`.
  **L1046 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1047 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ALLOCATABLE or POINTER %s has corank %d but actual argument has corank %d"_err_en_US,`.
  **L1047 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ALLOCATABLE or POINTER %s has corank %d but actual argument has corank %d"_err_en_US,`。
- **L1048 EN**: Executes a call or declaration centered on `dummy.type.corank`.
  **L1048 CN**: 执行以 `dummy.type.corank` 为核心的调用或声明。
- **L1049 EN**: Closes the current lexical scope or compound statement.
  **L1049 CN**: 结束当前词法作用域或复合语句块。
- **L1050 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1050 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1051 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1051 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1052 EN**: Continues logic associated with callable symbol `Say`.
  **L1052 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"non-VOLATILE coarray may not be associated with VOLATILE coarray %s"_err_en_US,`.
  **L1053 CN**: 继续一个多行参数列表、初始化器或聚合项：`"non-VOLATILE coarray may not be associated with VOLATILE coarray %s"_err_en_US,`。
- **L1054 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1054 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1055 EN**: Closes the current lexical scope or compound statement.
  **L1055 CN**: 结束当前词法作用域或复合语句块。
- **L1056 EN**: Transitions from the previous branch into the alternative path.
  **L1056 CN**: 从前一个分支过渡到备选路径。

### Lines 1057-1080

````cpp
      if (actualIsVolatile) {
        messages.Say(
            "VOLATILE coarray may not be associated with non-VOLATILE coarray %s"_err_en_US,
            dummyName);
      }
    }
    if (actualRank == dummyRank && !actualIsContiguous) {
      if (dummyIsContiguous) {
        messages.Say(
            "Actual argument associated with a CONTIGUOUS coarray %s must be simply contiguous"_err_en_US,
            dummyName);
      } else if (!dummyIsAssumedShape && !dummyIsAssumedRank) {
        messages.Say(
            "Actual argument associated with coarray %s (not assumed shape or rank) must be simply contiguous"_err_en_US,
            dummyName);
      }
    }
  }

  // NULL(MOLD=) checking for non-intrinsic procedures
  if (!intrinsic && !dummyIsAllocatableOrPointer && !dummyIsOptional &&
      evaluate::IsNullPointer(&actual)) {
    messages.Say(
        "Actual argument associated with %s may not be null pointer %s"_err_en_US,
````
- **L1057 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1057 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1058 EN**: Continues logic associated with callable symbol `Say`.
  **L1058 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"VOLATILE coarray may not be associated with non-VOLATILE coarray %s"_err_en_US,`.
  **L1059 CN**: 继续一个多行参数列表、初始化器或聚合项：`"VOLATILE coarray may not be associated with non-VOLATILE coarray %s"_err_en_US,`。
- **L1060 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1060 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1061 EN**: Closes the current lexical scope or compound statement.
  **L1061 CN**: 结束当前词法作用域或复合语句块。
- **L1062 EN**: Closes the current lexical scope or compound statement.
  **L1062 CN**: 结束当前词法作用域或复合语句块。
- **L1063 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1063 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1064 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1064 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1065 EN**: Continues logic associated with callable symbol `Say`.
  **L1065 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with a CONTIGUOUS coarray %s must be simply contiguous"_err_en_US,`.
  **L1066 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with a CONTIGUOUS coarray %s must be simply contiguous"_err_en_US,`。
- **L1067 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1067 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1068 EN**: Transitions from the previous branch into an `else if` condition.
  **L1068 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1069 EN**: Continues logic associated with callable symbol `Say`.
  **L1069 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with coarray %s (not assumed shape or rank) must be simply contiguous"_err_en_US,`.
  **L1070 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with coarray %s (not assumed shape or rank) must be simply contiguous"_err_en_US,`。
- **L1071 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1071 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1072 EN**: Closes the current lexical scope or compound statement.
  **L1072 CN**: 结束当前词法作用域或复合语句块。
- **L1073 EN**: Closes the current lexical scope or compound statement.
  **L1073 CN**: 结束当前词法作用域或复合语句块。
- **L1074 EN**: Closes the current lexical scope or compound statement.
  **L1074 CN**: 结束当前词法作用域或复合语句块。
- **L1075 EN**: Blank line separating nearby declarations or logic blocks.
  **L1075 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1076 EN**: Comment explains nearby logic, intent, or metadata: `NULL(MOLD=) checking for non-intrinsic procedures`.
  **L1076 CN**: 注释说明附近代码的逻辑、意图或元数据：`NULL(MOLD=) checking for non-intrinsic procedures`。
- **L1077 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1077 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1078 EN**: Starts a function, method, lambda, or structured scope: `evaluate::IsNullPointer(&actual)) {`.
  **L1078 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::IsNullPointer(&actual)) {`。
- **L1079 EN**: Continues logic associated with callable symbol `Say`.
  **L1079 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with %s may not be null pointer %s"_err_en_US,`.
  **L1080 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with %s may not be null pointer %s"_err_en_US,`。

### Lines 1081-1104

````cpp
        dummyName, actual.AsFortran());
  }

  // Warn about dubious actual argument association with a TARGET dummy
  // argument
  bool actualIsVariable{evaluate::IsVariable(actual)};
  if (dummy.attrs.test(characteristics::DummyDataObject::Attr::Target) &&
      context.ShouldWarn(common::UsageWarning::NonTargetPassedToTarget)) {
    bool actualIsTemp{
        !actualIsVariable || HasVectorSubscript(actual) || actualCoarrayRef};
    if (actualIsTemp) {
      foldingContext.Warn(common::UsageWarning::NonTargetPassedToTarget,
          "Any pointer associated with TARGET %s during this call will not be associated with the value of '%s' afterwards"_warn_en_US,
          dummyName, actual.AsFortran());
    } else {
      auto actualSymbolVector{GetSymbolVector(actual)};
      if (!evaluate::GetLastTarget(actualSymbolVector)) {
        foldingContext.Warn(common::UsageWarning::NonTargetPassedToTarget,
            "Any pointer associated with TARGET %s during this call must not be used afterwards, as '%s' is not a target"_warn_en_US,
            dummyName, actual.AsFortran());
      }
    }
  }

````
- **L1081 EN**: Executes a call or declaration centered on `actual.AsFortran`.
  **L1081 CN**: 执行以 `actual.AsFortran` 为核心的调用或声明。
- **L1082 EN**: Closes the current lexical scope or compound statement.
  **L1082 CN**: 结束当前词法作用域或复合语句块。
- **L1083 EN**: Blank line separating nearby declarations or logic blocks.
  **L1083 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1084 EN**: Comment explains nearby logic, intent, or metadata: `Warn about dubious actual argument association with a TARGET dummy`.
  **L1084 CN**: 注释说明附近代码的逻辑、意图或元数据：`Warn about dubious actual argument association with a TARGET dummy`。
- **L1085 EN**: Comment explains nearby logic, intent, or metadata: `argument`.
  **L1085 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument`。
- **L1086 EN**: Executes a call or declaration centered on `actualIsVariable{evaluate::IsVariable`.
  **L1086 CN**: 执行以 `actualIsVariable{evaluate::IsVariable` 为核心的调用或声明。
- **L1087 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1087 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1088 EN**: Starts a function, method, lambda, or structured scope: `context.ShouldWarn(common::UsageWarning::NonTargetPassedToTarget)) {`.
  **L1088 CN**: 开始一个函数、方法、lambda 或结构化作用域：`context.ShouldWarn(common::UsageWarning::NonTargetPassedToTarget)) {`。
- **L1089 EN**: Continues the surrounding expression or declaration: `bool actualIsTemp{`.
  **L1089 CN**: 继续构造周围的表达式或声明：`bool actualIsTemp{`。
- **L1090 EN**: Executes a call or declaration centered on `HasVectorSubscript`.
  **L1090 CN**: 执行以 `HasVectorSubscript` 为核心的调用或声明。
- **L1091 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1091 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1092 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::NonTargetPassedToTarget,`.
  **L1092 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::NonTargetPassedToTarget,`。
- **L1093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Any pointer associated with TARGET %s during this call will not be associated with the value of '%s' afterwards"_warn_en_US,`.
  **L1093 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Any pointer associated with TARGET %s during this call will not be associated with the value of '%s' afterwards"_warn_en_US,`。
- **L1094 EN**: Executes a call or declaration centered on `actual.AsFortran`.
  **L1094 CN**: 执行以 `actual.AsFortran` 为核心的调用或声明。
- **L1095 EN**: Transitions from the previous branch into the alternative path.
  **L1095 CN**: 从前一个分支过渡到备选路径。
- **L1096 EN**: Executes a call or declaration centered on `actualSymbolVector{GetSymbolVector`.
  **L1096 CN**: 执行以 `actualSymbolVector{GetSymbolVector` 为核心的调用或声明。
- **L1097 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1097 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::NonTargetPassedToTarget,`.
  **L1098 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::NonTargetPassedToTarget,`。
- **L1099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Any pointer associated with TARGET %s during this call must not be used afterwards, as '%s' is not a target"_warn_en_US,`.
  **L1099 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Any pointer associated with TARGET %s during this call must not be used afterwards, as '%s' is not a target"_warn_en_US,`。
- **L1100 EN**: Executes a call or declaration centered on `actual.AsFortran`.
  **L1100 CN**: 执行以 `actual.AsFortran` 为核心的调用或声明。
- **L1101 EN**: Closes the current lexical scope or compound statement.
  **L1101 CN**: 结束当前词法作用域或复合语句块。
- **L1102 EN**: Closes the current lexical scope or compound statement.
  **L1102 CN**: 结束当前词法作用域或复合语句块。
- **L1103 EN**: Closes the current lexical scope or compound statement.
  **L1103 CN**: 结束当前词法作用域或复合语句块。
- **L1104 EN**: Blank line separating nearby declarations or logic blocks.
  **L1104 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1105-1128

````cpp
  // CUDA specific checks
  // TODO: These are disabled in OpenACC constructs, which may not be
  // correct when the target is not a GPU.
  if (!intrinsic &&
      !dummy.attrs.test(characteristics::DummyDataObject::Attr::Value) &&
      !FindOpenACCConstructContaining(scope)) {
    std::optional<common::CUDADataAttr> actualDataAttr, dummyDataAttr;
    // For a%b%c, the last symbol with a CUDA data attribute wins
    if (actualIsVariable) {
      for (const Symbol &s : evaluate::GetSymbolVector(actual)) {
        if (const auto *object{s.detailsIf<ObjectEntityDetails>()}) {
          if (auto cudaAttr{object->cudaDataAttr()}) {
            actualDataAttr = *cudaAttr;
          }
        }
      }
    }
    dummyDataAttr = dummy.cudaDataAttr;
    // Treat MANAGED like DEVICE for nonallocatable nonpointer arguments to
    // device subprograms
    if (procedure.cudaSubprogramAttrs.value_or(
            common::CUDASubprogramAttrs::Host) !=
            common::CUDASubprogramAttrs::Host &&
        !dummy.attrs.test(
````
- **L1105 EN**: Comment explains nearby logic, intent, or metadata: `CUDA specific checks`.
  **L1105 CN**: 注释说明附近代码的逻辑、意图或元数据：`CUDA specific checks`。
- **L1106 EN**: Comment records a pending task or caution: `TODO: These are disabled in OpenACC constructs, which may not be`.
  **L1106 CN**: 注释记录待办事项或注意点：`TODO: These are disabled in OpenACC constructs, which may not be`。
- **L1107 EN**: Comment explains nearby logic, intent, or metadata: `correct when the target is not a GPU.`.
  **L1107 CN**: 注释说明附近代码的逻辑、意图或元数据：`correct when the target is not a GPU.`。
- **L1108 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1108 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1109 EN**: Continues logic associated with callable symbol `test`.
  **L1109 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1110 EN**: Starts a function, method, lambda, or structured scope: `!FindOpenACCConstructContaining(scope)) {`.
  **L1110 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!FindOpenACCConstructContaining(scope)) {`。
- **L1111 EN**: Executes a standalone statement or declaration: `std::optional<common::CUDADataAttr> actualDataAttr, dummyDataAttr;`.
  **L1111 CN**: 执行一条独立语句或声明：`std::optional<common::CUDADataAttr> actualDataAttr, dummyDataAttr;`。
- **L1112 EN**: Comment explains nearby logic, intent, or metadata: `For a%b%c, the last symbol with a CUDA data attribute wins`.
  **L1112 CN**: 注释说明附近代码的逻辑、意图或元数据：`For a%b%c, the last symbol with a CUDA data attribute wins`。
- **L1113 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1113 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1114 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1114 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1115 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1115 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1116 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1116 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1117 EN**: Executes a standalone statement or declaration: `actualDataAttr = *cudaAttr;`.
  **L1117 CN**: 执行一条独立语句或声明：`actualDataAttr = *cudaAttr;`。
- **L1118 EN**: Closes the current lexical scope or compound statement.
  **L1118 CN**: 结束当前词法作用域或复合语句块。
- **L1119 EN**: Closes the current lexical scope or compound statement.
  **L1119 CN**: 结束当前词法作用域或复合语句块。
- **L1120 EN**: Closes the current lexical scope or compound statement.
  **L1120 CN**: 结束当前词法作用域或复合语句块。
- **L1121 EN**: Closes the current lexical scope or compound statement.
  **L1121 CN**: 结束当前词法作用域或复合语句块。
- **L1122 EN**: Executes a standalone statement or declaration: `dummyDataAttr = dummy.cudaDataAttr;`.
  **L1122 CN**: 执行一条独立语句或声明：`dummyDataAttr = dummy.cudaDataAttr;`。
- **L1123 EN**: Comment explains nearby logic, intent, or metadata: `Treat MANAGED like DEVICE for nonallocatable nonpointer arguments to`.
  **L1123 CN**: 注释说明附近代码的逻辑、意图或元数据：`Treat MANAGED like DEVICE for nonallocatable nonpointer arguments to`。
- **L1124 EN**: Comment explains nearby logic, intent, or metadata: `device subprograms`.
  **L1124 CN**: 注释说明附近代码的逻辑、意图或元数据：`device subprograms`。
- **L1125 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1125 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1126 EN**: Continues the surrounding expression or declaration: `common::CUDASubprogramAttrs::Host) !=`.
  **L1126 CN**: 继续构造周围的表达式或声明：`common::CUDASubprogramAttrs::Host) !=`。
- **L1127 EN**: Continues the surrounding expression or declaration: `common::CUDASubprogramAttrs::Host &&`.
  **L1127 CN**: 继续构造周围的表达式或声明：`common::CUDASubprogramAttrs::Host &&`。
- **L1128 EN**: Continues logic associated with callable symbol `test`.
  **L1128 CN**: 继续与可调用符号 `test` 相关的逻辑。

### Lines 1129-1152

````cpp
            characteristics::DummyDataObject::Attr::Allocatable) &&
        !dummy.attrs.test(characteristics::DummyDataObject::Attr::Pointer)) {
      if (!dummyDataAttr || *dummyDataAttr == common::CUDADataAttr::Managed) {
        dummyDataAttr = common::CUDADataAttr::Device;
      }
      if ((!actualDataAttr && FindCUDADeviceContext(scope)) ||
          (actualDataAttr &&
              *actualDataAttr == common::CUDADataAttr::Managed)) {
        actualDataAttr = common::CUDADataAttr::Device;
      }
      // For device procedures, treat actual arguments with VALUE attribute as
      // device data; also constant actual arguments and the function result.
      if (!actualDataAttr &&
          (!actualFirstSymbol || IsValue(*actualFirstSymbol) ||
              IsFunctionResult(*actualFirstSymbol)) &&
          (*procedure.cudaSubprogramAttrs ==
              common::CUDASubprogramAttrs::Device)) {
        actualDataAttr = common::CUDADataAttr::Device;
      }
    }
    if (dummyDataAttr == common::CUDADataAttr::Device &&
        (dummyIsAssumedShape || dummyIsAssumedRank) &&
        !dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {
      if (auto contig{evaluate::IsContiguous(actual, foldingContext,
````
- **L1129 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject::Attr::Allocatable) &&`.
  **L1129 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject::Attr::Allocatable) &&`。
- **L1130 EN**: Starts a function, method, lambda, or structured scope: `!dummy.attrs.test(characteristics::DummyDataObject::Attr::Pointer)) {`.
  **L1130 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!dummy.attrs.test(characteristics::DummyDataObject::Attr::Pointer)) {`。
- **L1131 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1131 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1132 EN**: Executes a standalone statement or declaration: `dummyDataAttr = common::CUDADataAttr::Device;`.
  **L1132 CN**: 执行一条独立语句或声明：`dummyDataAttr = common::CUDADataAttr::Device;`。
- **L1133 EN**: Closes the current lexical scope or compound statement.
  **L1133 CN**: 结束当前词法作用域或复合语句块。
- **L1134 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1134 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1135 EN**: Continues the surrounding expression or declaration: `(actualDataAttr &&`.
  **L1135 CN**: 继续构造周围的表达式或声明：`(actualDataAttr &&`。
- **L1136 EN**: Comment explains nearby logic, intent, or metadata: `actualDataAttr == common::CUDADataAttr::Managed)) {`.
  **L1136 CN**: 注释说明附近代码的逻辑、意图或元数据：`actualDataAttr == common::CUDADataAttr::Managed)) {`。
- **L1137 EN**: Executes a standalone statement or declaration: `actualDataAttr = common::CUDADataAttr::Device;`.
  **L1137 CN**: 执行一条独立语句或声明：`actualDataAttr = common::CUDADataAttr::Device;`。
- **L1138 EN**: Closes the current lexical scope or compound statement.
  **L1138 CN**: 结束当前词法作用域或复合语句块。
- **L1139 EN**: Comment explains nearby logic, intent, or metadata: `For device procedures, treat actual arguments with VALUE attribute as`.
  **L1139 CN**: 注释说明附近代码的逻辑、意图或元数据：`For device procedures, treat actual arguments with VALUE attribute as`。
- **L1140 EN**: Comment explains nearby logic, intent, or metadata: `device data; also constant actual arguments and the function result.`.
  **L1140 CN**: 注释说明附近代码的逻辑、意图或元数据：`device data; also constant actual arguments and the function result.`。
- **L1141 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1141 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1142 EN**: Continues logic associated with callable symbol `IsValue`.
  **L1142 CN**: 继续与可调用符号 `IsValue` 相关的逻辑。
- **L1143 EN**: Continues logic associated with callable symbol `IsFunctionResult`.
  **L1143 CN**: 继续与可调用符号 `IsFunctionResult` 相关的逻辑。
- **L1144 EN**: Continues the surrounding expression or declaration: `(*procedure.cudaSubprogramAttrs ==`.
  **L1144 CN**: 继续构造周围的表达式或声明：`(*procedure.cudaSubprogramAttrs ==`。
- **L1145 EN**: Continues the surrounding expression or declaration: `common::CUDASubprogramAttrs::Device)) {`.
  **L1145 CN**: 继续构造周围的表达式或声明：`common::CUDASubprogramAttrs::Device)) {`。
- **L1146 EN**: Executes a standalone statement or declaration: `actualDataAttr = common::CUDADataAttr::Device;`.
  **L1146 CN**: 执行一条独立语句或声明：`actualDataAttr = common::CUDADataAttr::Device;`。
- **L1147 EN**: Closes the current lexical scope or compound statement.
  **L1147 CN**: 结束当前词法作用域或复合语句块。
- **L1148 EN**: Closes the current lexical scope or compound statement.
  **L1148 CN**: 结束当前词法作用域或复合语句块。
- **L1149 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1149 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1150 EN**: Continues the surrounding expression or declaration: `(dummyIsAssumedShape || dummyIsAssumedRank) &&`.
  **L1150 CN**: 继续构造周围的表达式或声明：`(dummyIsAssumedShape || dummyIsAssumedRank) &&`。
- **L1151 EN**: Starts a function, method, lambda, or structured scope: `!dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {`.
  **L1151 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!dummy.ignoreTKR.test(common::IgnoreTKR::Contiguous)) {`。
- **L1152 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1152 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1153-1176

````cpp
              /*namedConstantSectionsAreContiguous=*/true,
              /*firstDimensionStride1=*/true)}) {
        if (!*contig) {
          messages.Say(
              "actual argument associated with assumed shape/rank device %s is known to be discontiguous on its first dimension"_err_en_US,
              dummyName);
        }
      } else {
        messages.Say(
            "actual argument associated with assumed shape/rank device %s is not known to be contiguous on its first dimension"_warn_en_US,
            dummyName);
      }
    }
    bool isHostDeviceProc{procedure.cudaSubprogramAttrs &&
        *procedure.cudaSubprogramAttrs ==
            common::CUDASubprogramAttrs::HostDevice};
    // TYPE(*) assumed-size/rank dummies are opaque buffers (e.g. MPI) and do
    // not impose a CUDA address space on their actual argument.
    bool skipCudaDataAttrCheck{IsCUDAAddressSpaceAgnostic(dummy)};
    if (!skipCudaDataAttrCheck &&
        !common::AreCompatibleCUDADataAttrs(dummyDataAttr, actualDataAttr,
            dummy.ignoreTKR, /*allowUnifiedMatchingRule=*/true,
            isHostDeviceProc, &context.languageFeatures())) {
      auto toStr{[](std::optional<common::CUDADataAttr> x) {
````
- **L1153 EN**: Comment explains nearby logic, intent, or metadata: `namedConstantSectionsAreContiguous=*/true,`.
  **L1153 CN**: 注释说明附近代码的逻辑、意图或元数据：`namedConstantSectionsAreContiguous=*/true,`。
- **L1154 EN**: Comment explains nearby logic, intent, or metadata: `firstDimensionStride1=*/true)}) {`.
  **L1154 CN**: 注释说明附近代码的逻辑、意图或元数据：`firstDimensionStride1=*/true)}) {`。
- **L1155 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1155 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1156 EN**: Continues logic associated with callable symbol `Say`.
  **L1156 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"actual argument associated with assumed shape/rank device %s is known to be discontiguous on its first dimension"_err_en_US,`.
  **L1157 CN**: 继续一个多行参数列表、初始化器或聚合项：`"actual argument associated with assumed shape/rank device %s is known to be discontiguous on its first dimension"_err_en_US,`。
- **L1158 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1158 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1159 EN**: Closes the current lexical scope or compound statement.
  **L1159 CN**: 结束当前词法作用域或复合语句块。
- **L1160 EN**: Transitions from the previous branch into the alternative path.
  **L1160 CN**: 从前一个分支过渡到备选路径。
- **L1161 EN**: Continues logic associated with callable symbol `Say`.
  **L1161 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"actual argument associated with assumed shape/rank device %s is not known to be contiguous on its first dimension"_warn_en_US,`.
  **L1162 CN**: 继续一个多行参数列表、初始化器或聚合项：`"actual argument associated with assumed shape/rank device %s is not known to be contiguous on its first dimension"_warn_en_US,`。
- **L1163 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1163 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1164 EN**: Closes the current lexical scope or compound statement.
  **L1164 CN**: 结束当前词法作用域或复合语句块。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Continues the surrounding expression or declaration: `bool isHostDeviceProc{procedure.cudaSubprogramAttrs &&`.
  **L1166 CN**: 继续构造周围的表达式或声明：`bool isHostDeviceProc{procedure.cudaSubprogramAttrs &&`。
- **L1167 EN**: Comment explains nearby logic, intent, or metadata: `procedure.cudaSubprogramAttrs ==`.
  **L1167 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure.cudaSubprogramAttrs ==`。
- **L1168 EN**: Executes a standalone statement or declaration: `common::CUDASubprogramAttrs::HostDevice};`.
  **L1168 CN**: 执行一条独立语句或声明：`common::CUDASubprogramAttrs::HostDevice};`。
- **L1169 EN**: Comment explains nearby logic, intent, or metadata: `TYPE(*) assumed-size/rank dummies are opaque buffers (e.g. MPI) and do`.
  **L1169 CN**: 注释说明附近代码的逻辑、意图或元数据：`TYPE(*) assumed-size/rank dummies are opaque buffers (e.g. MPI) and do`。
- **L1170 EN**: Comment explains nearby logic, intent, or metadata: `not impose a CUDA address space on their actual argument.`.
  **L1170 CN**: 注释说明附近代码的逻辑、意图或元数据：`not impose a CUDA address space on their actual argument.`。
- **L1171 EN**: Executes a call or declaration centered on `skipCudaDataAttrCheck{IsCUDAAddressSpaceAgnostic`.
  **L1171 CN**: 执行以 `skipCudaDataAttrCheck{IsCUDAAddressSpaceAgnostic` 为核心的调用或声明。
- **L1172 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1172 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!common::AreCompatibleCUDADataAttrs(dummyDataAttr, actualDataAttr,`.
  **L1173 CN**: 继续一个多行参数列表、初始化器或聚合项：`!common::AreCompatibleCUDADataAttrs(dummyDataAttr, actualDataAttr,`。
- **L1174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dummy.ignoreTKR, /*allowUnifiedMatchingRule=*/true,`.
  **L1174 CN**: 继续一个多行参数列表、初始化器或聚合项：`dummy.ignoreTKR, /*allowUnifiedMatchingRule=*/true,`。
- **L1175 EN**: Starts a function, method, lambda, or structured scope: `isHostDeviceProc, &context.languageFeatures())) {`.
  **L1175 CN**: 开始一个函数、方法、lambda 或结构化作用域：`isHostDeviceProc, &context.languageFeatures())) {`。
- **L1176 EN**: Starts a function, method, lambda, or structured scope: `auto toStr{[](std::optional<common::CUDADataAttr> x) {`.
  **L1176 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto toStr{[](std::optional<common::CUDADataAttr> x) {`。

### Lines 1177-1200

````cpp
        return x ? "ATTRIBUTES("s +
                parser::ToUpperCaseLetters(common::EnumToString(*x)) + ")"s
                 : "no CUDA data attribute"s;
      }};
      messages.Say(
          "%s has %s but its associated actual argument has %s"_err_en_US,
          dummyName, toStr(dummyDataAttr), toStr(actualDataAttr));
    }
  }

  // Emit an error message if an actual argument passed to a host intrinsic is
  // on the device.
  if (intrinsic && !FindCUDADeviceContext(scope) &&
      !FindOpenACCConstructContaining(scope) &&
      !HasOpenACCRoutineDirective(scope)) {
    if (!cudaSkippedIntrinsics.contains(intrinsic->name)) {
      std::optional<common::CUDADataAttr> actualDataAttr;
      if (const auto *actualObject{actualLastSymbol
                  ? actualLastSymbol->detailsIf<ObjectEntityDetails>()
                  : nullptr}) {
        actualDataAttr = actualObject->cudaDataAttr();
      }
      if (actualDataAttr && *actualDataAttr == common::CUDADataAttr::Device) {
        messages.Say(
````
- **L1177 EN**: Returns from the current function with `x ? "ATTRIBUTES("s +`.
  **L1177 CN**: 以 `x ? "ATTRIBUTES("s +` 从当前函数返回。
- **L1178 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L1178 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L1179 EN**: Executes a standalone statement or declaration: `: "no CUDA data attribute"s;`.
  **L1179 CN**: 执行一条独立语句或声明：`: "no CUDA data attribute"s;`。
- **L1180 EN**: Executes a standalone statement or declaration: `}};`.
  **L1180 CN**: 执行一条独立语句或声明：`}};`。
- **L1181 EN**: Continues logic associated with callable symbol `Say`.
  **L1181 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s has %s but its associated actual argument has %s"_err_en_US,`.
  **L1182 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s has %s but its associated actual argument has %s"_err_en_US,`。
- **L1183 EN**: Executes a call or declaration centered on `toStr`.
  **L1183 CN**: 执行以 `toStr` 为核心的调用或声明。
- **L1184 EN**: Closes the current lexical scope or compound statement.
  **L1184 CN**: 结束当前词法作用域或复合语句块。
- **L1185 EN**: Closes the current lexical scope or compound statement.
  **L1185 CN**: 结束当前词法作用域或复合语句块。
- **L1186 EN**: Blank line separating nearby declarations or logic blocks.
  **L1186 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1187 EN**: Comment explains nearby logic, intent, or metadata: `Emit an error message if an actual argument passed to a host intrinsic is`.
  **L1187 CN**: 注释说明附近代码的逻辑、意图或元数据：`Emit an error message if an actual argument passed to a host intrinsic is`。
- **L1188 EN**: Comment explains nearby logic, intent, or metadata: `on the device.`.
  **L1188 CN**: 注释说明附近代码的逻辑、意图或元数据：`on the device.`。
- **L1189 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1189 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1190 EN**: Continues logic associated with callable symbol `FindOpenACCConstructContaining`.
  **L1190 CN**: 继续与可调用符号 `FindOpenACCConstructContaining` 相关的逻辑。
- **L1191 EN**: Starts a function, method, lambda, or structured scope: `!HasOpenACCRoutineDirective(scope)) {`.
  **L1191 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!HasOpenACCRoutineDirective(scope)) {`。
- **L1192 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1192 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1193 EN**: Executes a standalone statement or declaration: `std::optional<common::CUDADataAttr> actualDataAttr;`.
  **L1193 CN**: 执行一条独立语句或声明：`std::optional<common::CUDADataAttr> actualDataAttr;`。
- **L1194 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1194 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1195 EN**: Continues logic associated with callable symbol `detailsIf<ObjectEntityDetails>`.
  **L1195 CN**: 继续与可调用符号 `detailsIf<ObjectEntityDetails>` 相关的逻辑。
- **L1196 EN**: Continues the surrounding expression or declaration: `: nullptr}) {`.
  **L1196 CN**: 继续构造周围的表达式或声明：`: nullptr}) {`。
- **L1197 EN**: Executes a call or declaration centered on `actualObject->cudaDataAttr`.
  **L1197 CN**: 执行以 `actualObject->cudaDataAttr` 为核心的调用或声明。
- **L1198 EN**: Closes the current lexical scope or compound statement.
  **L1198 CN**: 结束当前词法作用域或复合语句块。
- **L1199 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1199 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1200 EN**: Continues logic associated with callable symbol `Say`.
  **L1200 CN**: 继续与可调用符号 `Say` 相关的逻辑。

### Lines 1201-1224

````cpp
            "Actual argument %s associated with host intrinsic %s is on the device"_err_en_US,
            actualLastSymbol ? actualLastSymbol->name() : "", intrinsic->name);
      }
    }
  }

  // Warning for breaking F'2023 change with character allocatables
  if (intrinsic && dummy.intent != common::Intent::In) {
    WarnOnDeferredLengthCharacterScalar(
        context, &actual, messages.at(), dummyName.c_str());
  }

  // %VAL() and %REF() checking for explicit interface
  if ((arg.isPercentRef() || arg.isPercentVal()) &&
      dummy.IsPassedByDescriptor(procedure.IsBindC())) {
    messages.Say(
        "%%VAL or %%REF are not allowed for %s that must be passed by means of a descriptor"_err_en_US,
        dummyName);
  }
  if (arg.isPercentVal() &&
      (!actualType.type().IsLengthlessIntrinsicType() ||
          actualType.Rank() != 0)) {
    messages.Say(
        "%VAL argument must be a scalar numeric or logical expression"_err_en_US);
````
- **L1201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument %s associated with host intrinsic %s is on the device"_err_en_US,`.
  **L1201 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument %s associated with host intrinsic %s is on the device"_err_en_US,`。
- **L1202 EN**: Executes a call or declaration centered on `actualLastSymbol->name`.
  **L1202 CN**: 执行以 `actualLastSymbol->name` 为核心的调用或声明。
- **L1203 EN**: Closes the current lexical scope or compound statement.
  **L1203 CN**: 结束当前词法作用域或复合语句块。
- **L1204 EN**: Closes the current lexical scope or compound statement.
  **L1204 CN**: 结束当前词法作用域或复合语句块。
- **L1205 EN**: Closes the current lexical scope or compound statement.
  **L1205 CN**: 结束当前词法作用域或复合语句块。
- **L1206 EN**: Blank line separating nearby declarations or logic blocks.
  **L1206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1207 EN**: Comment explains nearby logic, intent, or metadata: `Warning for breaking F'2023 change with character allocatables`.
  **L1207 CN**: 注释说明附近代码的逻辑、意图或元数据：`Warning for breaking F'2023 change with character allocatables`。
- **L1208 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1208 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1209 EN**: Continues logic associated with callable symbol `WarnOnDeferredLengthCharacterScalar`.
  **L1209 CN**: 继续与可调用符号 `WarnOnDeferredLengthCharacterScalar` 相关的逻辑。
- **L1210 EN**: Executes a call or declaration centered on `messages.at`.
  **L1210 CN**: 执行以 `messages.at` 为核心的调用或声明。
- **L1211 EN**: Closes the current lexical scope or compound statement.
  **L1211 CN**: 结束当前词法作用域或复合语句块。
- **L1212 EN**: Blank line separating nearby declarations or logic blocks.
  **L1212 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1213 EN**: Comment explains nearby logic, intent, or metadata: `%VAL() and %REF() checking for explicit interface`.
  **L1213 CN**: 注释说明附近代码的逻辑、意图或元数据：`%VAL() and %REF() checking for explicit interface`。
- **L1214 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1214 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1215 EN**: Starts a function, method, lambda, or structured scope: `dummy.IsPassedByDescriptor(procedure.IsBindC())) {`.
  **L1215 CN**: 开始一个函数、方法、lambda 或结构化作用域：`dummy.IsPassedByDescriptor(procedure.IsBindC())) {`。
- **L1216 EN**: Continues logic associated with callable symbol `Say`.
  **L1216 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%%VAL or %%REF are not allowed for %s that must be passed by means of a descriptor"_err_en_US,`.
  **L1217 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%%VAL or %%REF are not allowed for %s that must be passed by means of a descriptor"_err_en_US,`。
- **L1218 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1218 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1219 EN**: Closes the current lexical scope or compound statement.
  **L1219 CN**: 结束当前词法作用域或复合语句块。
- **L1220 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1220 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1221 EN**: Continues logic associated with callable symbol `type`.
  **L1221 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L1222 EN**: Starts a function, method, lambda, or structured scope: `actualType.Rank() != 0)) {`.
  **L1222 CN**: 开始一个函数、方法、lambda 或结构化作用域：`actualType.Rank() != 0)) {`。
- **L1223 EN**: Continues logic associated with callable symbol `Say`.
  **L1223 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1224 EN**: Executes a standalone statement or declaration: `"%VAL argument must be a scalar numeric or logical expression"_err_en_US);`.
  **L1224 CN**: 执行一条独立语句或声明：`"%VAL argument must be a scalar numeric or logical expression"_err_en_US);`。

### Lines 1225-1248

````cpp
  }
}

static void CheckProcedureArg(evaluate::ActualArgument &arg,
    const characteristics::Procedure &proc,
    const characteristics::DummyProcedure &dummy, const std::string &dummyName,
    SemanticsContext &context, bool ignoreImplicitVsExplicit) {
  evaluate::FoldingContext &foldingContext{context.foldingContext()};
  parser::ContextualMessages &messages{foldingContext.messages()};
  parser::CharBlock location{arg.sourceLocation().value_or(messages.at())};
  auto restorer{messages.SetLocation(location)};
  const characteristics::Procedure &interface { dummy.procedure.value() };
  if (const auto *expr{arg.UnwrapExpr()}) {
    bool dummyIsPointer{
        dummy.attrs.test(characteristics::DummyProcedure::Attr::Pointer)};
    const auto *argProcDesignator{
        std::get_if<evaluate::ProcedureDesignator>(&expr->u)};
    const auto *argProcSymbol{
        argProcDesignator ? argProcDesignator->GetSymbol() : nullptr};
    if (argProcSymbol) {
      if (const auto *subp{
              argProcSymbol->GetUltimate().detailsIf<SubprogramDetails>()}) {
        if (subp->stmtFunction()) {
          evaluate::SayWithDeclaration(messages, *argProcSymbol,
````
- **L1225 EN**: Closes the current lexical scope or compound statement.
  **L1225 CN**: 结束当前词法作用域或复合语句块。
- **L1226 EN**: Closes the current lexical scope or compound statement.
  **L1226 CN**: 结束当前词法作用域或复合语句块。
- **L1227 EN**: Blank line separating nearby declarations or logic blocks.
  **L1227 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckProcedureArg(evaluate::ActualArgument &arg,`.
  **L1228 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckProcedureArg(evaluate::ActualArgument &arg,`。
- **L1229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const characteristics::Procedure &proc,`.
  **L1229 CN**: 继续一个多行参数列表、初始化器或聚合项：`const characteristics::Procedure &proc,`。
- **L1230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const characteristics::DummyProcedure &dummy, const std::string &dummyName,`.
  **L1230 CN**: 继续一个多行参数列表、初始化器或聚合项：`const characteristics::DummyProcedure &dummy, const std::string &dummyName,`。
- **L1231 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, bool ignoreImplicitVsExplicit) {`.
  **L1231 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, bool ignoreImplicitVsExplicit) {`。
- **L1232 EN**: Executes a call or declaration centered on `&foldingContext{context.foldingContext`.
  **L1232 CN**: 执行以 `&foldingContext{context.foldingContext` 为核心的调用或声明。
- **L1233 EN**: Executes a call or declaration centered on `&messages{foldingContext.messages`.
  **L1233 CN**: 执行以 `&messages{foldingContext.messages` 为核心的调用或声明。
- **L1234 EN**: Executes a call or declaration centered on `location{arg.sourceLocation`.
  **L1234 CN**: 执行以 `location{arg.sourceLocation` 为核心的调用或声明。
- **L1235 EN**: Executes a call or declaration centered on `restorer{messages.SetLocation`.
  **L1235 CN**: 执行以 `restorer{messages.SetLocation` 为核心的调用或声明。
- **L1236 EN**: Executes a call or declaration centered on `dummy.procedure.value`.
  **L1236 CN**: 执行以 `dummy.procedure.value` 为核心的调用或声明。
- **L1237 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1237 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1238 EN**: Continues the surrounding expression or declaration: `bool dummyIsPointer{`.
  **L1238 CN**: 继续构造周围的表达式或声明：`bool dummyIsPointer{`。
- **L1239 EN**: Executes a call or declaration centered on `dummy.attrs.test`.
  **L1239 CN**: 执行以 `dummy.attrs.test` 为核心的调用或声明。
- **L1240 EN**: Continues the surrounding expression or declaration: `const auto *argProcDesignator{`.
  **L1240 CN**: 继续构造周围的表达式或声明：`const auto *argProcDesignator{`。
- **L1241 EN**: Executes a call or declaration centered on `std::get_if<evaluate::ProcedureDesignator>`.
  **L1241 CN**: 执行以 `std::get_if<evaluate::ProcedureDesignator>` 为核心的调用或声明。
- **L1242 EN**: Continues the surrounding expression or declaration: `const auto *argProcSymbol{`.
  **L1242 CN**: 继续构造周围的表达式或声明：`const auto *argProcSymbol{`。
- **L1243 EN**: Executes a call or declaration centered on `argProcDesignator->GetSymbol`.
  **L1243 CN**: 执行以 `argProcDesignator->GetSymbol` 为核心的调用或声明。
- **L1244 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1244 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1245 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1245 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1246 EN**: Starts a function, method, lambda, or structured scope: `argProcSymbol->GetUltimate().detailsIf<SubprogramDetails>()}) {`.
  **L1246 CN**: 开始一个函数、方法、lambda 或结构化作用域：`argProcSymbol->GetUltimate().detailsIf<SubprogramDetails>()}) {`。
- **L1247 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1247 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, *argProcSymbol,`.
  **L1248 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, *argProcSymbol,`。

### Lines 1249-1272

````cpp
              "Statement function '%s' may not be passed as an actual argument"_err_en_US,
              argProcSymbol->name());
          return;
        }
      } else if (argProcSymbol->has<ProcBindingDetails>()) {
        if (!context.IsEnabled(common::LanguageFeature::BindingAsProcedure)) {
          evaluate::SayWithDeclaration(messages, *argProcSymbol,
              "Procedure binding '%s' passed as an actual argument"_err_en_US,
              argProcSymbol->name());
        } else {
          evaluate::WarnWithDeclaration(foldingContext, *argProcSymbol,
              common::LanguageFeature::BindingAsProcedure,
              "Procedure binding '%s' passed as an actual argument"_port_en_US,
              argProcSymbol->name());
        }
      }
    }
    if (auto argChars{characteristics::DummyArgument::FromActual(
            "actual argument", *expr, foldingContext,
            /*forImplicitInterface=*/true)}) {
      if (!argChars->IsTypelessIntrinsicDummy()) {
        if (auto *argProc{
                std::get_if<characteristics::DummyProcedure>(&argChars->u)}) {
          characteristics::Procedure &argInterface{argProc->procedure.value()};
````
- **L1249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Statement function '%s' may not be passed as an actual argument"_err_en_US,`.
  **L1249 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Statement function '%s' may not be passed as an actual argument"_err_en_US,`。
- **L1250 EN**: Executes a call or declaration centered on `argProcSymbol->name`.
  **L1250 CN**: 执行以 `argProcSymbol->name` 为核心的调用或声明。
- **L1251 EN**: Returns from the current function with `void`.
  **L1251 CN**: 以 `void` 从当前函数返回。
- **L1252 EN**: Closes the current lexical scope or compound statement.
  **L1252 CN**: 结束当前词法作用域或复合语句块。
- **L1253 EN**: Transitions from the previous branch into an `else if` condition.
  **L1253 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1254 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1254 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, *argProcSymbol,`.
  **L1255 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, *argProcSymbol,`。
- **L1256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Procedure binding '%s' passed as an actual argument"_err_en_US,`.
  **L1256 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Procedure binding '%s' passed as an actual argument"_err_en_US,`。
- **L1257 EN**: Executes a call or declaration centered on `argProcSymbol->name`.
  **L1257 CN**: 执行以 `argProcSymbol->name` 为核心的调用或声明。
- **L1258 EN**: Transitions from the previous branch into the alternative path.
  **L1258 CN**: 从前一个分支过渡到备选路径。
- **L1259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::WarnWithDeclaration(foldingContext, *argProcSymbol,`.
  **L1259 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::WarnWithDeclaration(foldingContext, *argProcSymbol,`。
- **L1260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::LanguageFeature::BindingAsProcedure,`.
  **L1260 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::LanguageFeature::BindingAsProcedure,`。
- **L1261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Procedure binding '%s' passed as an actual argument"_port_en_US,`.
  **L1261 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Procedure binding '%s' passed as an actual argument"_port_en_US,`。
- **L1262 EN**: Executes a call or declaration centered on `argProcSymbol->name`.
  **L1262 CN**: 执行以 `argProcSymbol->name` 为核心的调用或声明。
- **L1263 EN**: Closes the current lexical scope or compound statement.
  **L1263 CN**: 结束当前词法作用域或复合语句块。
- **L1264 EN**: Closes the current lexical scope or compound statement.
  **L1264 CN**: 结束当前词法作用域或复合语句块。
- **L1265 EN**: Closes the current lexical scope or compound statement.
  **L1265 CN**: 结束当前词法作用域或复合语句块。
- **L1266 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1266 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"actual argument", *expr, foldingContext,`.
  **L1267 CN**: 继续一个多行参数列表、初始化器或聚合项：`"actual argument", *expr, foldingContext,`。
- **L1268 EN**: Comment explains nearby logic, intent, or metadata: `forImplicitInterface=*/true)}) {`.
  **L1268 CN**: 注释说明附近代码的逻辑、意图或元数据：`forImplicitInterface=*/true)}) {`。
- **L1269 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1269 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1270 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1270 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1271 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<characteristics::DummyProcedure>(&argChars->u)}) {`.
  **L1271 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<characteristics::DummyProcedure>(&argChars->u)}) {`。
- **L1272 EN**: Executes a call or declaration centered on `&argInterface{argProc->procedure.value`.
  **L1272 CN**: 执行以 `&argInterface{argProc->procedure.value` 为核心的调用或声明。

### Lines 1273-1296

````cpp
          argInterface.attrs.reset(
              characteristics::Procedure::Attr::NullPointer);
          argInterface.attrs.reset(
              characteristics::Procedure::Attr::NullAllocatable);
          if (!argProcSymbol || argProcSymbol->attrs().test(Attr::INTRINSIC)) {
            // It's ok to pass ELEMENTAL unrestricted intrinsic functions.
            argInterface.attrs.reset(
                characteristics::Procedure::Attr::Elemental);
          } else if (argInterface.attrs.test(
                         characteristics::Procedure::Attr::Elemental)) {
            if (argProcSymbol) { // C1533
              evaluate::SayWithDeclaration(messages, *argProcSymbol,
                  "Non-intrinsic ELEMENTAL procedure '%s' may not be passed as an actual argument"_err_en_US,
                  argProcSymbol->name());
              return; // avoid piling on with checks below
            } else {
              argInterface.attrs.reset(
                  characteristics::Procedure::Attr::NullPointer);
              argInterface.attrs.reset(
                  characteristics::Procedure::Attr::NullAllocatable);
            }
          }
          if (interface.HasExplicitInterface()) {
            std::string whyNot;
````
- **L1273 EN**: Continues logic associated with callable symbol `reset`.
  **L1273 CN**: 继续与可调用符号 `reset` 相关的逻辑。
- **L1274 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attr::NullPointer);`.
  **L1274 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attr::NullPointer);`。
- **L1275 EN**: Continues logic associated with callable symbol `reset`.
  **L1275 CN**: 继续与可调用符号 `reset` 相关的逻辑。
- **L1276 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attr::NullAllocatable);`.
  **L1276 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attr::NullAllocatable);`。
- **L1277 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1277 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1278 EN**: Comment explains nearby logic, intent, or metadata: `It's ok to pass ELEMENTAL unrestricted intrinsic functions.`.
  **L1278 CN**: 注释说明附近代码的逻辑、意图或元数据：`It's ok to pass ELEMENTAL unrestricted intrinsic functions.`。
- **L1279 EN**: Continues logic associated with callable symbol `reset`.
  **L1279 CN**: 继续与可调用符号 `reset` 相关的逻辑。
- **L1280 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attr::Elemental);`.
  **L1280 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attr::Elemental);`。
- **L1281 EN**: Transitions from the previous branch into an `else if` condition.
  **L1281 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1282 EN**: Continues the surrounding expression or declaration: `characteristics::Procedure::Attr::Elemental)) {`.
  **L1282 CN**: 继续构造周围的表达式或声明：`characteristics::Procedure::Attr::Elemental)) {`。
- **L1283 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1283 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, *argProcSymbol,`.
  **L1284 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, *argProcSymbol,`。
- **L1285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Non-intrinsic ELEMENTAL procedure '%s' may not be passed as an actual argument"_err_en_US,`.
  **L1285 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Non-intrinsic ELEMENTAL procedure '%s' may not be passed as an actual argument"_err_en_US,`。
- **L1286 EN**: Executes a call or declaration centered on `argProcSymbol->name`.
  **L1286 CN**: 执行以 `argProcSymbol->name` 为核心的调用或声明。
- **L1287 EN**: Returns from the current function with `; // avoid piling on with checks below`.
  **L1287 CN**: 以 `; // avoid piling on with checks below` 从当前函数返回。
- **L1288 EN**: Transitions from the previous branch into the alternative path.
  **L1288 CN**: 从前一个分支过渡到备选路径。
- **L1289 EN**: Continues logic associated with callable symbol `reset`.
  **L1289 CN**: 继续与可调用符号 `reset` 相关的逻辑。
- **L1290 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attr::NullPointer);`.
  **L1290 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attr::NullPointer);`。
- **L1291 EN**: Continues logic associated with callable symbol `reset`.
  **L1291 CN**: 继续与可调用符号 `reset` 相关的逻辑。
- **L1292 EN**: Executes a standalone statement or declaration: `characteristics::Procedure::Attr::NullAllocatable);`.
  **L1292 CN**: 执行一条独立语句或声明：`characteristics::Procedure::Attr::NullAllocatable);`。
- **L1293 EN**: Closes the current lexical scope or compound statement.
  **L1293 CN**: 结束当前词法作用域或复合语句块。
- **L1294 EN**: Closes the current lexical scope or compound statement.
  **L1294 CN**: 结束当前词法作用域或复合语句块。
- **L1295 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1295 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1296 EN**: Executes a standalone statement or declaration: `std::string whyNot;`.
  **L1296 CN**: 执行一条独立语句或声明：`std::string whyNot;`。

### Lines 1297-1320

````cpp
            std::optional<std::string> warning;
            if (!interface.IsCompatibleWith(argInterface,
                    ignoreImplicitVsExplicit, &whyNot,
                    /*specificIntrinsic=*/nullptr, &warning)) {
              // 15.5.2.9(1): Explicit interfaces must match
              if (argInterface.HasExplicitInterface()) {
                messages.Say(
                    "Actual procedure argument has interface incompatible with %s: %s"_err_en_US,
                    dummyName, whyNot);
                return;
              } else if (proc.IsPure()) {
                messages.Say(
                    "Actual procedure argument for %s of a PURE procedure must have an explicit interface"_err_en_US,
                    dummyName);
              } else {
                foldingContext.Warn(
                    common::UsageWarning::ImplicitInterfaceActual,
                    "Actual procedure argument has an implicit interface which is not known to be compatible with %s which has an explicit interface"_warn_en_US,
                    dummyName);
              }
            } else if (warning) {
              foldingContext.Warn(common::UsageWarning::ProcDummyArgShapes,
                  "Actual procedure argument has possible interface incompatibility with %s: %s"_warn_en_US,
                  dummyName, std::move(*warning));
````
- **L1297 EN**: Executes a standalone statement or declaration: `std::optional<std::string> warning;`.
  **L1297 CN**: 执行一条独立语句或声明：`std::optional<std::string> warning;`。
- **L1298 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1298 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ignoreImplicitVsExplicit, &whyNot,`.
  **L1299 CN**: 继续一个多行参数列表、初始化器或聚合项：`ignoreImplicitVsExplicit, &whyNot,`。
- **L1300 EN**: Comment explains nearby logic, intent, or metadata: `specificIntrinsic=*/nullptr, &warning)) {`.
  **L1300 CN**: 注释说明附近代码的逻辑、意图或元数据：`specificIntrinsic=*/nullptr, &warning)) {`。
- **L1301 EN**: Comment explains nearby logic, intent, or metadata: `15.5.2.9(1): Explicit interfaces must match`.
  **L1301 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.5.2.9(1): Explicit interfaces must match`。
- **L1302 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1302 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1303 EN**: Continues logic associated with callable symbol `Say`.
  **L1303 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual procedure argument has interface incompatible with %s: %s"_err_en_US,`.
  **L1304 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual procedure argument has interface incompatible with %s: %s"_err_en_US,`。
- **L1305 EN**: Executes a standalone statement or declaration: `dummyName, whyNot);`.
  **L1305 CN**: 执行一条独立语句或声明：`dummyName, whyNot);`。
- **L1306 EN**: Returns from the current function with `void`.
  **L1306 CN**: 以 `void` 从当前函数返回。
- **L1307 EN**: Transitions from the previous branch into an `else if` condition.
  **L1307 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1308 EN**: Continues logic associated with callable symbol `Say`.
  **L1308 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual procedure argument for %s of a PURE procedure must have an explicit interface"_err_en_US,`.
  **L1309 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual procedure argument for %s of a PURE procedure must have an explicit interface"_err_en_US,`。
- **L1310 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1310 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1311 EN**: Transitions from the previous branch into the alternative path.
  **L1311 CN**: 从前一个分支过渡到备选路径。
- **L1312 EN**: Continues logic associated with callable symbol `Warn`.
  **L1312 CN**: 继续与可调用符号 `Warn` 相关的逻辑。
- **L1313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::UsageWarning::ImplicitInterfaceActual,`.
  **L1313 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::UsageWarning::ImplicitInterfaceActual,`。
- **L1314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual procedure argument has an implicit interface which is not known to be compatible with %s which has an explicit interface"_warn_en_US,`.
  **L1314 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual procedure argument has an implicit interface which is not known to be compatible with %s which has an explicit interface"_warn_en_US,`。
- **L1315 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1315 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1316 EN**: Closes the current lexical scope or compound statement.
  **L1316 CN**: 结束当前词法作用域或复合语句块。
- **L1317 EN**: Transitions from the previous branch into an `else if` condition.
  **L1317 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::ProcDummyArgShapes,`.
  **L1318 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::ProcDummyArgShapes,`。
- **L1319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual procedure argument has possible interface incompatibility with %s: %s"_warn_en_US,`.
  **L1319 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual procedure argument has possible interface incompatibility with %s: %s"_warn_en_US,`。
- **L1320 EN**: Executes a call or declaration centered on `std::move`.
  **L1320 CN**: 执行以 `std::move` 为核心的调用或声明。

### Lines 1321-1344

````cpp
            }
          } else { // 15.5.2.9(2,3)
            if (interface.IsSubroutine() && argInterface.IsFunction()) {
              messages.Say(
                  "Actual argument associated with procedure %s is a function but must be a subroutine"_err_en_US,
                  dummyName);
            } else if (interface.IsFunction()) {
              if (argInterface.IsFunction()) {
                std::string whyNot;
                if (!interface.functionResult->IsCompatibleWith(
                        *argInterface.functionResult, &whyNot)) {
                  messages.Say(
                      "Actual argument function associated with procedure %s is not compatible: %s"_err_en_US,
                      dummyName, whyNot);
                }
              } else if (argInterface.IsSubroutine()) {
                messages.Say(
                    "Actual argument associated with procedure %s is a subroutine but must be a function"_err_en_US,
                    dummyName);
              }
            }
          }
        } else {
          messages.Say(
````
- **L1321 EN**: Closes the current lexical scope or compound statement.
  **L1321 CN**: 结束当前词法作用域或复合语句块。
- **L1322 EN**: Transitions from the previous branch into the alternative path.
  **L1322 CN**: 从前一个分支过渡到备选路径。
- **L1323 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1323 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1324 EN**: Continues logic associated with callable symbol `Say`.
  **L1324 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with procedure %s is a function but must be a subroutine"_err_en_US,`.
  **L1325 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with procedure %s is a function but must be a subroutine"_err_en_US,`。
- **L1326 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1326 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1327 EN**: Transitions from the previous branch into an `else if` condition.
  **L1327 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1328 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1328 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1329 EN**: Executes a standalone statement or declaration: `std::string whyNot;`.
  **L1329 CN**: 执行一条独立语句或声明：`std::string whyNot;`。
- **L1330 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1330 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1331 EN**: Comment explains nearby logic, intent, or metadata: `argInterface.functionResult, &whyNot)) {`.
  **L1331 CN**: 注释说明附近代码的逻辑、意图或元数据：`argInterface.functionResult, &whyNot)) {`。
- **L1332 EN**: Continues logic associated with callable symbol `Say`.
  **L1332 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument function associated with procedure %s is not compatible: %s"_err_en_US,`.
  **L1333 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument function associated with procedure %s is not compatible: %s"_err_en_US,`。
- **L1334 EN**: Executes a standalone statement or declaration: `dummyName, whyNot);`.
  **L1334 CN**: 执行一条独立语句或声明：`dummyName, whyNot);`。
- **L1335 EN**: Closes the current lexical scope or compound statement.
  **L1335 CN**: 结束当前词法作用域或复合语句块。
- **L1336 EN**: Transitions from the previous branch into an `else if` condition.
  **L1336 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1337 EN**: Continues logic associated with callable symbol `Say`.
  **L1337 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with procedure %s is a subroutine but must be a function"_err_en_US,`.
  **L1338 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with procedure %s is a subroutine but must be a function"_err_en_US,`。
- **L1339 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1339 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1340 EN**: Closes the current lexical scope or compound statement.
  **L1340 CN**: 结束当前词法作用域或复合语句块。
- **L1341 EN**: Closes the current lexical scope or compound statement.
  **L1341 CN**: 结束当前词法作用域或复合语句块。
- **L1342 EN**: Closes the current lexical scope or compound statement.
  **L1342 CN**: 结束当前词法作用域或复合语句块。
- **L1343 EN**: Transitions from the previous branch into the alternative path.
  **L1343 CN**: 从前一个分支过渡到备选路径。
- **L1344 EN**: Continues logic associated with callable symbol `Say`.
  **L1344 CN**: 继续与可调用符号 `Say` 相关的逻辑。

### Lines 1345-1368

````cpp
              "Actual argument associated with procedure %s is not a procedure"_err_en_US,
              dummyName);
        }
      } else if (IsNullPointer(expr)) {
        if (!dummyIsPointer &&
            !dummy.attrs.test(
                characteristics::DummyProcedure::Attr::Optional)) {
          messages.Say(
              "Actual argument associated with procedure %s is a null pointer"_err_en_US,
              dummyName);
        }
      } else {
        messages.Say(
            "Actual argument associated with procedure %s is typeless"_err_en_US,
            dummyName);
      }
    }
    if (dummyIsPointer) {
      if (dummy.intent == common::Intent::In) {
        // need not be definable, can be a target
      } else if (!IsProcedurePointer(*expr)) {
        messages.Say(
            "Actual argument associated with procedure pointer %s is not a procedure pointer"_err_en_US,
            dummyName);
````
- **L1345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with procedure %s is not a procedure"_err_en_US,`.
  **L1345 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with procedure %s is not a procedure"_err_en_US,`。
- **L1346 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1346 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1347 EN**: Closes the current lexical scope or compound statement.
  **L1347 CN**: 结束当前词法作用域或复合语句块。
- **L1348 EN**: Transitions from the previous branch into an `else if` condition.
  **L1348 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1349 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1349 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1350 EN**: Continues logic associated with callable symbol `test`.
  **L1350 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1351 EN**: Continues the surrounding expression or declaration: `characteristics::DummyProcedure::Attr::Optional)) {`.
  **L1351 CN**: 继续构造周围的表达式或声明：`characteristics::DummyProcedure::Attr::Optional)) {`。
- **L1352 EN**: Continues logic associated with callable symbol `Say`.
  **L1352 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with procedure %s is a null pointer"_err_en_US,`.
  **L1353 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with procedure %s is a null pointer"_err_en_US,`。
- **L1354 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1354 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1355 EN**: Closes the current lexical scope or compound statement.
  **L1355 CN**: 结束当前词法作用域或复合语句块。
- **L1356 EN**: Transitions from the previous branch into the alternative path.
  **L1356 CN**: 从前一个分支过渡到备选路径。
- **L1357 EN**: Continues logic associated with callable symbol `Say`.
  **L1357 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with procedure %s is typeless"_err_en_US,`.
  **L1358 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with procedure %s is typeless"_err_en_US,`。
- **L1359 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1359 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1360 EN**: Closes the current lexical scope or compound statement.
  **L1360 CN**: 结束当前词法作用域或复合语句块。
- **L1361 EN**: Closes the current lexical scope or compound statement.
  **L1361 CN**: 结束当前词法作用域或复合语句块。
- **L1362 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1362 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1363 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1363 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1364 EN**: Comment explains nearby logic, intent, or metadata: `need not be definable, can be a target`.
  **L1364 CN**: 注释说明附近代码的逻辑、意图或元数据：`need not be definable, can be a target`。
- **L1365 EN**: Transitions from the previous branch into an `else if` condition.
  **L1365 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1366 EN**: Continues logic associated with callable symbol `Say`.
  **L1366 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with procedure pointer %s is not a procedure pointer"_err_en_US,`.
  **L1367 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with procedure pointer %s is not a procedure pointer"_err_en_US,`。
- **L1368 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1368 CN**: 执行一条独立语句或声明：`dummyName);`。

### Lines 1369-1392

````cpp
      } else if (dummy.intent == common::Intent::Default) {
        // ok, needs to be definable only if defined at run time
      } else {
        DefinabilityFlags flags{DefinabilityFlag::PointerDefinition};
        if (dummy.intent != common::Intent::Out) {
          flags.set(DefinabilityFlag::DoNotNoteDefinition);
        }
        if (auto whyNot{WhyNotDefinable(
                location, context.FindScope(location), flags, *expr)}) {
          if (auto *msg{messages.Say(
                  "Actual argument associated with INTENT(%s) procedure pointer %s is not definable"_err_en_US,
                  dummy.intent == common::Intent::Out ? "OUT" : "IN OUT",
                  dummyName)}) {
            msg->Attach(
                std::move(whyNot->set_severity(parser::Severity::Because)));
          }
        }
      }
    }
  } else {
    messages.Say(
        "Assumed-type argument may not be forwarded as procedure %s"_err_en_US,
        dummyName);
  }
````
- **L1369 EN**: Transitions from the previous branch into an `else if` condition.
  **L1369 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1370 EN**: Comment explains nearby logic, intent, or metadata: `ok, needs to be definable only if defined at run time`.
  **L1370 CN**: 注释说明附近代码的逻辑、意图或元数据：`ok, needs to be definable only if defined at run time`。
- **L1371 EN**: Transitions from the previous branch into the alternative path.
  **L1371 CN**: 从前一个分支过渡到备选路径。
- **L1372 EN**: Executes a standalone statement or declaration: `DefinabilityFlags flags{DefinabilityFlag::PointerDefinition};`.
  **L1372 CN**: 执行一条独立语句或声明：`DefinabilityFlags flags{DefinabilityFlag::PointerDefinition};`。
- **L1373 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1373 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1374 EN**: Executes a call or declaration centered on `flags.set`.
  **L1374 CN**: 执行以 `flags.set` 为核心的调用或声明。
- **L1375 EN**: Closes the current lexical scope or compound statement.
  **L1375 CN**: 结束当前词法作用域或复合语句块。
- **L1376 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1376 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1377 EN**: Starts a function, method, lambda, or structured scope: `location, context.FindScope(location), flags, *expr)}) {`.
  **L1377 CN**: 开始一个函数、方法、lambda 或结构化作用域：`location, context.FindScope(location), flags, *expr)}) {`。
- **L1378 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1378 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with INTENT(%s) procedure pointer %s is not definable"_err_en_US,`.
  **L1379 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with INTENT(%s) procedure pointer %s is not definable"_err_en_US,`。
- **L1380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dummy.intent == common::Intent::Out ? "OUT" : "IN OUT",`.
  **L1380 CN**: 继续一个多行参数列表、初始化器或聚合项：`dummy.intent == common::Intent::Out ? "OUT" : "IN OUT",`。
- **L1381 EN**: Continues the surrounding expression or declaration: `dummyName)}) {`.
  **L1381 CN**: 继续构造周围的表达式或声明：`dummyName)}) {`。
- **L1382 EN**: Continues logic associated with callable symbol `Attach`.
  **L1382 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L1383 EN**: Executes a call or declaration centered on `std::move`.
  **L1383 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L1384 EN**: Closes the current lexical scope or compound statement.
  **L1384 CN**: 结束当前词法作用域或复合语句块。
- **L1385 EN**: Closes the current lexical scope or compound statement.
  **L1385 CN**: 结束当前词法作用域或复合语句块。
- **L1386 EN**: Closes the current lexical scope or compound statement.
  **L1386 CN**: 结束当前词法作用域或复合语句块。
- **L1387 EN**: Closes the current lexical scope or compound statement.
  **L1387 CN**: 结束当前词法作用域或复合语句块。
- **L1388 EN**: Transitions from the previous branch into the alternative path.
  **L1388 CN**: 从前一个分支过渡到备选路径。
- **L1389 EN**: Continues logic associated with callable symbol `Say`.
  **L1389 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-type argument may not be forwarded as procedure %s"_err_en_US,`.
  **L1390 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-type argument may not be forwarded as procedure %s"_err_en_US,`。
- **L1391 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1391 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1392 EN**: Closes the current lexical scope or compound statement.
  **L1392 CN**: 结束当前词法作用域或复合语句块。

### Lines 1393-1416

````cpp
}

// Allow BOZ literal actual arguments when they can be converted to a known
// dummy argument type
static void ConvertBOZLiteralArg(
    evaluate::ActualArgument &arg, const evaluate::DynamicType &type) {
  if (auto *expr{arg.UnwrapExpr()}) {
    if (IsBOZLiteral(*expr)) {
      if (auto converted{evaluate::ConvertToType(type, SomeExpr{*expr})}) {
        arg = std::move(*converted);
      }
    }
  }
}

static void CheckExplicitInterfaceArg(evaluate::ActualArgument &arg,
    const characteristics::DummyArgument &dummy,
    const characteristics::Procedure &proc, SemanticsContext &context,
    const Scope *scope, const evaluate::SpecificIntrinsic *intrinsic,
    bool allowActualArgumentConversions, bool extentErrors,
    bool ignoreImplicitVsExplicit) {
  evaluate::FoldingContext &foldingContext{context.foldingContext()};
  auto &messages{foldingContext.messages()};
  std::string dummyName{"dummy argument"};
````
- **L1393 EN**: Closes the current lexical scope or compound statement.
  **L1393 CN**: 结束当前词法作用域或复合语句块。
- **L1394 EN**: Blank line separating nearby declarations or logic blocks.
  **L1394 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1395 EN**: Comment explains nearby logic, intent, or metadata: `Allow BOZ literal actual arguments when they can be converted to a known`.
  **L1395 CN**: 注释说明附近代码的逻辑、意图或元数据：`Allow BOZ literal actual arguments when they can be converted to a known`。
- **L1396 EN**: Comment explains nearby logic, intent, or metadata: `dummy argument type`.
  **L1396 CN**: 注释说明附近代码的逻辑、意图或元数据：`dummy argument type`。
- **L1397 EN**: Continues logic associated with callable symbol `ConvertBOZLiteralArg`.
  **L1397 CN**: 继续与可调用符号 `ConvertBOZLiteralArg` 相关的逻辑。
- **L1398 EN**: Continues the surrounding expression or declaration: `evaluate::ActualArgument &arg, const evaluate::DynamicType &type) {`.
  **L1398 CN**: 继续构造周围的表达式或声明：`evaluate::ActualArgument &arg, const evaluate::DynamicType &type) {`。
- **L1399 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1399 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1400 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1400 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1401 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1401 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1402 EN**: Executes a call or declaration centered on `std::move`.
  **L1402 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L1403 EN**: Closes the current lexical scope or compound statement.
  **L1403 CN**: 结束当前词法作用域或复合语句块。
- **L1404 EN**: Closes the current lexical scope or compound statement.
  **L1404 CN**: 结束当前词法作用域或复合语句块。
- **L1405 EN**: Closes the current lexical scope or compound statement.
  **L1405 CN**: 结束当前词法作用域或复合语句块。
- **L1406 EN**: Closes the current lexical scope or compound statement.
  **L1406 CN**: 结束当前词法作用域或复合语句块。
- **L1407 EN**: Blank line separating nearby declarations or logic blocks.
  **L1407 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckExplicitInterfaceArg(evaluate::ActualArgument &arg,`.
  **L1408 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckExplicitInterfaceArg(evaluate::ActualArgument &arg,`。
- **L1409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const characteristics::DummyArgument &dummy,`.
  **L1409 CN**: 继续一个多行参数列表、初始化器或聚合项：`const characteristics::DummyArgument &dummy,`。
- **L1410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const characteristics::Procedure &proc, SemanticsContext &context,`.
  **L1410 CN**: 继续一个多行参数列表、初始化器或聚合项：`const characteristics::Procedure &proc, SemanticsContext &context,`。
- **L1411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Scope *scope, const evaluate::SpecificIntrinsic *intrinsic,`.
  **L1411 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Scope *scope, const evaluate::SpecificIntrinsic *intrinsic,`。
- **L1412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool allowActualArgumentConversions, bool extentErrors,`.
  **L1412 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool allowActualArgumentConversions, bool extentErrors,`。
- **L1413 EN**: Continues the surrounding expression or declaration: `bool ignoreImplicitVsExplicit) {`.
  **L1413 CN**: 继续构造周围的表达式或声明：`bool ignoreImplicitVsExplicit) {`。
- **L1414 EN**: Executes a call or declaration centered on `&foldingContext{context.foldingContext`.
  **L1414 CN**: 执行以 `&foldingContext{context.foldingContext` 为核心的调用或声明。
- **L1415 EN**: Executes a call or declaration centered on `&messages{foldingContext.messages`.
  **L1415 CN**: 执行以 `&messages{foldingContext.messages` 为核心的调用或声明。
- **L1416 EN**: Executes a standalone statement or declaration: `std::string dummyName{"dummy argument"};`.
  **L1416 CN**: 执行一条独立语句或声明：`std::string dummyName{"dummy argument"};`。

### Lines 1417-1440

````cpp
  if (!dummy.name.empty()) {
    dummyName += " '"s + parser::ToLowerCaseLetters(dummy.name) + "='";
  }
  auto restorer{
      messages.SetLocation(arg.sourceLocation().value_or(messages.at()))};
  auto CheckActualArgForLabel = [&](evaluate::ActualArgument &arg) {
    if (arg.isAlternateReturn()) {
      messages.Say(
          "Alternate return label '%d' cannot be associated with %s"_err_en_US,
          arg.GetLabel(), dummyName);
      return false;
    } else {
      return true;
    }
  };
  common::visit(
      common::visitors{
          [&](const characteristics::DummyDataObject &object) {
            if (CheckActualArgForLabel(arg)) {
              ConvertBOZLiteralArg(arg, object.type.type());
              if (auto *expr{arg.UnwrapExpr()}) {
                if (auto type{characteristics::TypeAndShape::Characterize(
                        *expr, foldingContext)}) {
                  arg.set_dummyIntent(object.intent);
````
- **L1417 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1417 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1418 EN**: Executes a call or declaration centered on `parser::ToLowerCaseLetters`.
  **L1418 CN**: 执行以 `parser::ToLowerCaseLetters` 为核心的调用或声明。
- **L1419 EN**: Closes the current lexical scope or compound statement.
  **L1419 CN**: 结束当前词法作用域或复合语句块。
- **L1420 EN**: Continues the surrounding expression or declaration: `auto restorer{`.
  **L1420 CN**: 继续构造周围的表达式或声明：`auto restorer{`。
- **L1421 EN**: Executes a call or declaration centered on `messages.SetLocation`.
  **L1421 CN**: 执行以 `messages.SetLocation` 为核心的调用或声明。
- **L1422 EN**: Starts a function, method, lambda, or structured scope: `auto CheckActualArgForLabel = [&](evaluate::ActualArgument &arg) {`.
  **L1422 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto CheckActualArgForLabel = [&](evaluate::ActualArgument &arg) {`。
- **L1423 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1423 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1424 EN**: Continues logic associated with callable symbol `Say`.
  **L1424 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Alternate return label '%d' cannot be associated with %s"_err_en_US,`.
  **L1425 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Alternate return label '%d' cannot be associated with %s"_err_en_US,`。
- **L1426 EN**: Executes a call or declaration centered on `arg.GetLabel`.
  **L1426 CN**: 执行以 `arg.GetLabel` 为核心的调用或声明。
- **L1427 EN**: Returns from the current function with `false`.
  **L1427 CN**: 以 `false` 从当前函数返回。
- **L1428 EN**: Transitions from the previous branch into the alternative path.
  **L1428 CN**: 从前一个分支过渡到备选路径。
- **L1429 EN**: Returns from the current function with `true`.
  **L1429 CN**: 以 `true` 从当前函数返回。
- **L1430 EN**: Closes the current lexical scope or compound statement.
  **L1430 CN**: 结束当前词法作用域或复合语句块。
- **L1431 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1431 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1432 EN**: Continues logic associated with callable symbol `visit`.
  **L1432 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L1433 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L1433 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L1434 EN**: Starts a function, method, lambda, or structured scope: `[&](const characteristics::DummyDataObject &object) {`.
  **L1434 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const characteristics::DummyDataObject &object) {`。
- **L1435 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1435 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1436 EN**: Executes a call or declaration centered on `ConvertBOZLiteralArg`.
  **L1436 CN**: 执行以 `ConvertBOZLiteralArg` 为核心的调用或声明。
- **L1437 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1437 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1438 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1438 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1439 EN**: Comment explains nearby logic, intent, or metadata: `expr, foldingContext)}) {`.
  **L1439 CN**: 注释说明附近代码的逻辑、意图或元数据：`expr, foldingContext)}) {`。
- **L1440 EN**: Executes a call or declaration centered on `arg.set_dummyIntent`.
  **L1440 CN**: 执行以 `arg.set_dummyIntent` 为核心的调用或声明。

### Lines 1441-1464

````cpp
                  bool isElemental{
                      object.type.Rank() == 0 && proc.IsElemental()};
                  CheckExplicitDataArg(object, dummyName, *expr, *type,
                      isElemental, context, foldingContext, scope, intrinsic,
                      allowActualArgumentConversions, extentErrors, proc, arg,
                      dummy);
                } else if (object.type.type().IsTypelessIntrinsicArgument() &&
                    IsBOZLiteral(*expr)) {
                  // ok
                } else if (object.type.type().IsTypelessIntrinsicArgument() &&
                    evaluate::IsNullObjectPointer(expr)) {
                  // ok, ASSOCIATED(NULL(without MOLD=))
                } else if (object.type.attrs().test(characteristics::
                                   TypeAndShape::Attr::AssumedRank) &&
                    evaluate::IsNullObjectPointer(expr) &&
                    (object.attrs.test(
                         characteristics::DummyDataObject::Attr::Allocatable) ||
                        object.attrs.test(
                            characteristics::DummyDataObject::Attr::Pointer) ||
                        !object.attrs.test(characteristics::DummyDataObject::
                                Attr::Optional))) {
                  messages.Say(
                      "NULL() without MOLD= must not be associated with an assumed-rank dummy argument that is ALLOCATABLE, POINTER, or non-OPTIONAL"_err_en_US);
                } else if ((object.attrs.test(characteristics::DummyDataObject::
````
- **L1441 EN**: Continues the surrounding expression or declaration: `bool isElemental{`.
  **L1441 CN**: 继续构造周围的表达式或声明：`bool isElemental{`。
- **L1442 EN**: Executes a call or declaration centered on `object.type.Rank`.
  **L1442 CN**: 执行以 `object.type.Rank` 为核心的调用或声明。
- **L1443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckExplicitDataArg(object, dummyName, *expr, *type,`.
  **L1443 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckExplicitDataArg(object, dummyName, *expr, *type,`。
- **L1444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `isElemental, context, foldingContext, scope, intrinsic,`.
  **L1444 CN**: 继续一个多行参数列表、初始化器或聚合项：`isElemental, context, foldingContext, scope, intrinsic,`。
- **L1445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `allowActualArgumentConversions, extentErrors, proc, arg,`.
  **L1445 CN**: 继续一个多行参数列表、初始化器或聚合项：`allowActualArgumentConversions, extentErrors, proc, arg,`。
- **L1446 EN**: Executes a standalone statement or declaration: `dummy);`.
  **L1446 CN**: 执行一条独立语句或声明：`dummy);`。
- **L1447 EN**: Transitions from the previous branch into an `else if` condition.
  **L1447 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1448 EN**: Starts a function, method, lambda, or structured scope: `IsBOZLiteral(*expr)) {`.
  **L1448 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsBOZLiteral(*expr)) {`。
- **L1449 EN**: Comment explains nearby logic, intent, or metadata: `ok`.
  **L1449 CN**: 注释说明附近代码的逻辑、意图或元数据：`ok`。
- **L1450 EN**: Transitions from the previous branch into an `else if` condition.
  **L1450 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1451 EN**: Starts a function, method, lambda, or structured scope: `evaluate::IsNullObjectPointer(expr)) {`.
  **L1451 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::IsNullObjectPointer(expr)) {`。
- **L1452 EN**: Comment explains nearby logic, intent, or metadata: `ok, ASSOCIATED(NULL(without MOLD=))`.
  **L1452 CN**: 注释说明附近代码的逻辑、意图或元数据：`ok, ASSOCIATED(NULL(without MOLD=))`。
- **L1453 EN**: Transitions from the previous branch into an `else if` condition.
  **L1453 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1454 EN**: Continues the surrounding expression or declaration: `TypeAndShape::Attr::AssumedRank) &&`.
  **L1454 CN**: 继续构造周围的表达式或声明：`TypeAndShape::Attr::AssumedRank) &&`。
- **L1455 EN**: Continues logic associated with callable symbol `IsNullObjectPointer`.
  **L1455 CN**: 继续与可调用符号 `IsNullObjectPointer` 相关的逻辑。
- **L1456 EN**: Continues logic associated with callable symbol `test`.
  **L1456 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1457 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject::Attr::Allocatable) ||`.
  **L1457 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject::Attr::Allocatable) ||`。
- **L1458 EN**: Continues logic associated with callable symbol `test`.
  **L1458 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1459 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject::Attr::Pointer) ||`.
  **L1459 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject::Attr::Pointer) ||`。
- **L1460 EN**: Continues logic associated with callable symbol `test`.
  **L1460 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1461 EN**: Continues the surrounding expression or declaration: `Attr::Optional))) {`.
  **L1461 CN**: 继续构造周围的表达式或声明：`Attr::Optional))) {`。
- **L1462 EN**: Continues logic associated with callable symbol `Say`.
  **L1462 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1463 EN**: Executes a call or declaration centered on `"NULL`.
  **L1463 CN**: 执行以 `"NULL` 为核心的调用或声明。
- **L1464 EN**: Transitions from the previous branch into an `else if` condition.
  **L1464 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 1465-1488

````cpp
                                    Attr::Pointer) ||
                               object.attrs.test(characteristics::
                                       DummyDataObject::Attr::Optional)) &&
                    evaluate::IsNullObjectPointer(expr)) {
                  // FOO(NULL(without MOLD=))
                  if (object.type.type().IsAssumedLengthCharacter()) {
                    messages.Say(
                        "Actual argument associated with %s is a NULL() pointer without a MOLD= to provide a character length"_err_en_US,
                        dummyName);
                  } else if (const DerivedTypeSpec *
                      derived{GetDerivedTypeSpec(object.type.type())}) {
                    for (const auto &[pName, pValue] : derived->parameters()) {
                      if (pValue.isAssumed()) {
                        messages.Say(
                            "Actual argument associated with %s is a NULL() pointer without a MOLD= to provide a value for the assumed type parameter '%s'"_err_en_US,
                            dummyName, pName.ToString());
                        break;
                      }
                    }
                  }
                } else if (object.attrs.test(characteristics::DummyDataObject::
                                   Attr::Allocatable) &&
                    (evaluate::IsNullAllocatable(expr) ||
                        evaluate::IsBareNullPointer(expr))) {
````
- **L1465 EN**: Continues the surrounding expression or declaration: `Attr::Pointer) ||`.
  **L1465 CN**: 继续构造周围的表达式或声明：`Attr::Pointer) ||`。
- **L1466 EN**: Continues logic associated with callable symbol `test`.
  **L1466 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1467 EN**: Continues the surrounding expression or declaration: `DummyDataObject::Attr::Optional)) &&`.
  **L1467 CN**: 继续构造周围的表达式或声明：`DummyDataObject::Attr::Optional)) &&`。
- **L1468 EN**: Starts a function, method, lambda, or structured scope: `evaluate::IsNullObjectPointer(expr)) {`.
  **L1468 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::IsNullObjectPointer(expr)) {`。
- **L1469 EN**: Comment explains nearby logic, intent, or metadata: `FOO(NULL(without MOLD=))`.
  **L1469 CN**: 注释说明附近代码的逻辑、意图或元数据：`FOO(NULL(without MOLD=))`。
- **L1470 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1470 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1471 EN**: Continues logic associated with callable symbol `Say`.
  **L1471 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with %s is a NULL() pointer without a MOLD= to provide a character length"_err_en_US,`.
  **L1472 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with %s is a NULL() pointer without a MOLD= to provide a character length"_err_en_US,`。
- **L1473 EN**: Executes a standalone statement or declaration: `dummyName);`.
  **L1473 CN**: 执行一条独立语句或声明：`dummyName);`。
- **L1474 EN**: Transitions from the previous branch into an `else if` condition.
  **L1474 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1475 EN**: Starts a function, method, lambda, or structured scope: `derived{GetDerivedTypeSpec(object.type.type())}) {`.
  **L1475 CN**: 开始一个函数、方法、lambda 或结构化作用域：`derived{GetDerivedTypeSpec(object.type.type())}) {`。
- **L1476 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1476 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1477 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1477 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1478 EN**: Continues logic associated with callable symbol `Say`.
  **L1478 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument associated with %s is a NULL() pointer without a MOLD= to provide a value for the assumed type parameter '%s'"_err_en_US,`.
  **L1479 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument associated with %s is a NULL() pointer without a MOLD= to provide a value for the assumed type parameter '%s'"_err_en_US,`。
- **L1480 EN**: Executes a call or declaration centered on `pName.ToString`.
  **L1480 CN**: 执行以 `pName.ToString` 为核心的调用或声明。
- **L1481 EN**: Exits the nearest loop or switch statement.
  **L1481 CN**: 退出最近的循环或 switch 语句。
- **L1482 EN**: Closes the current lexical scope or compound statement.
  **L1482 CN**: 结束当前词法作用域或复合语句块。
- **L1483 EN**: Closes the current lexical scope or compound statement.
  **L1483 CN**: 结束当前词法作用域或复合语句块。
- **L1484 EN**: Closes the current lexical scope or compound statement.
  **L1484 CN**: 结束当前词法作用域或复合语句块。
- **L1485 EN**: Transitions from the previous branch into an `else if` condition.
  **L1485 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1486 EN**: Continues the surrounding expression or declaration: `Attr::Allocatable) &&`.
  **L1486 CN**: 继续构造周围的表达式或声明：`Attr::Allocatable) &&`。
- **L1487 EN**: Continues logic associated with callable symbol `IsNullAllocatable`.
  **L1487 CN**: 继续与可调用符号 `IsNullAllocatable` 相关的逻辑。
- **L1488 EN**: Starts a function, method, lambda, or structured scope: `evaluate::IsBareNullPointer(expr))) {`.
  **L1488 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::IsBareNullPointer(expr))) {`。

### Lines 1489-1512

````cpp
                  if (object.intent == common::Intent::Out ||
                      object.intent == common::Intent::InOut) {
                    messages.Say(
                        "NULL() actual argument '%s' may not be associated with allocatable dummy argument %s that is INTENT(OUT) or INTENT(IN OUT)"_err_en_US,
                        expr->AsFortran(), dummyName);
                  } else if (object.intent == common::Intent::Default) {
                    foldingContext.Warn(
                        common::UsageWarning::
                            NullActualForDefaultIntentAllocatable,
                        "NULL() actual argument '%s' should not be associated with allocatable dummy argument %s without INTENT(IN)"_warn_en_US,
                        expr->AsFortran(), dummyName);
                  } else {
                    foldingContext.Warn(
                        common::LanguageFeature::NullActualForAllocatable,
                        "Allocatable %s is associated with %s"_port_en_US,
                        dummyName, expr->AsFortran());
                  }
                } else {
                  messages.Say(
                      "Actual argument '%s' associated with %s is not a variable or typed expression"_err_en_US,
                      expr->AsFortran(), dummyName);
                }
              } else {
                const Symbol &assumed{DEREF(arg.GetAssumedTypeDummy())};
````
- **L1489 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1489 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1490 EN**: Continues the surrounding expression or declaration: `object.intent == common::Intent::InOut) {`.
  **L1490 CN**: 继续构造周围的表达式或声明：`object.intent == common::Intent::InOut) {`。
- **L1491 EN**: Continues logic associated with callable symbol `Say`.
  **L1491 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"NULL() actual argument '%s' may not be associated with allocatable dummy argument %s that is INTENT(OUT) or INTENT(IN OUT)"_err_en_US,`.
  **L1492 CN**: 继续一个多行参数列表、初始化器或聚合项：`"NULL() actual argument '%s' may not be associated with allocatable dummy argument %s that is INTENT(OUT) or INTENT(IN OUT)"_err_en_US,`。
- **L1493 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L1493 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L1494 EN**: Transitions from the previous branch into an `else if` condition.
  **L1494 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1495 EN**: Continues logic associated with callable symbol `Warn`.
  **L1495 CN**: 继续与可调用符号 `Warn` 相关的逻辑。
- **L1496 EN**: Continues the surrounding expression or declaration: `common::UsageWarning::`.
  **L1496 CN**: 继续构造周围的表达式或声明：`common::UsageWarning::`。
- **L1497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NullActualForDefaultIntentAllocatable,`.
  **L1497 CN**: 继续一个多行参数列表、初始化器或聚合项：`NullActualForDefaultIntentAllocatable,`。
- **L1498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"NULL() actual argument '%s' should not be associated with allocatable dummy argument %s without INTENT(IN)"_warn_en_US,`.
  **L1498 CN**: 继续一个多行参数列表、初始化器或聚合项：`"NULL() actual argument '%s' should not be associated with allocatable dummy argument %s without INTENT(IN)"_warn_en_US,`。
- **L1499 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L1499 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L1500 EN**: Transitions from the previous branch into the alternative path.
  **L1500 CN**: 从前一个分支过渡到备选路径。
- **L1501 EN**: Continues logic associated with callable symbol `Warn`.
  **L1501 CN**: 继续与可调用符号 `Warn` 相关的逻辑。
- **L1502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::LanguageFeature::NullActualForAllocatable,`.
  **L1502 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::LanguageFeature::NullActualForAllocatable,`。
- **L1503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Allocatable %s is associated with %s"_port_en_US,`.
  **L1503 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Allocatable %s is associated with %s"_port_en_US,`。
- **L1504 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L1504 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L1505 EN**: Closes the current lexical scope or compound statement.
  **L1505 CN**: 结束当前词法作用域或复合语句块。
- **L1506 EN**: Transitions from the previous branch into the alternative path.
  **L1506 CN**: 从前一个分支过渡到备选路径。
- **L1507 EN**: Continues logic associated with callable symbol `Say`.
  **L1507 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument '%s' associated with %s is not a variable or typed expression"_err_en_US,`.
  **L1508 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument '%s' associated with %s is not a variable or typed expression"_err_en_US,`。
- **L1509 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L1509 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L1510 EN**: Closes the current lexical scope or compound statement.
  **L1510 CN**: 结束当前词法作用域或复合语句块。
- **L1511 EN**: Transitions from the previous branch into the alternative path.
  **L1511 CN**: 从前一个分支过渡到备选路径。
- **L1512 EN**: Executes a call or declaration centered on `&assumed{DEREF`.
  **L1512 CN**: 执行以 `&assumed{DEREF` 为核心的调用或声明。

### Lines 1513-1536

````cpp
                if (!object.type.type().IsAssumedType()) {
                  messages.Say(
                      "Assumed-type '%s' may be associated only with an assumed-type %s"_err_en_US,
                      assumed.name(), dummyName);
                } else if (object.type.attrs().test(characteristics::
                                   TypeAndShape::Attr::AssumedRank) &&
                    !IsAssumedShape(assumed) && !IsAssumedRank(assumed)) {
                  messages.Say( // C711
                      "Assumed-type '%s' must be either assumed shape or assumed rank to be associated with assumed rank %s"_err_en_US,
                      assumed.name(), dummyName);
                }
              }
            }
          },
          [&](const characteristics::DummyProcedure &dummy) {
            if (CheckActualArgForLabel(arg)) {
              CheckProcedureArg(arg, proc, dummy, dummyName, context,
                  ignoreImplicitVsExplicit);
            }
          },
          [&](const characteristics::AlternateReturn &) {
            // All semantic checking is done elsewhere
          },
      },
````
- **L1513 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1513 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1514 EN**: Continues logic associated with callable symbol `Say`.
  **L1514 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-type '%s' may be associated only with an assumed-type %s"_err_en_US,`.
  **L1515 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-type '%s' may be associated only with an assumed-type %s"_err_en_US,`。
- **L1516 EN**: Executes a call or declaration centered on `assumed.name`.
  **L1516 CN**: 执行以 `assumed.name` 为核心的调用或声明。
- **L1517 EN**: Transitions from the previous branch into an `else if` condition.
  **L1517 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1518 EN**: Continues the surrounding expression or declaration: `TypeAndShape::Attr::AssumedRank) &&`.
  **L1518 CN**: 继续构造周围的表达式或声明：`TypeAndShape::Attr::AssumedRank) &&`。
- **L1519 EN**: Starts a function, method, lambda, or structured scope: `!IsAssumedShape(assumed) && !IsAssumedRank(assumed)) {`.
  **L1519 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!IsAssumedShape(assumed) && !IsAssumedRank(assumed)) {`。
- **L1520 EN**: Continues logic associated with callable symbol `Say`.
  **L1520 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-type '%s' must be either assumed shape or assumed rank to be associated with assumed rank %s"_err_en_US,`.
  **L1521 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-type '%s' must be either assumed shape or assumed rank to be associated with assumed rank %s"_err_en_US,`。
- **L1522 EN**: Executes a call or declaration centered on `assumed.name`.
  **L1522 CN**: 执行以 `assumed.name` 为核心的调用或声明。
- **L1523 EN**: Closes the current lexical scope or compound statement.
  **L1523 CN**: 结束当前词法作用域或复合语句块。
- **L1524 EN**: Closes the current lexical scope or compound statement.
  **L1524 CN**: 结束当前词法作用域或复合语句块。
- **L1525 EN**: Closes the current lexical scope or compound statement.
  **L1525 CN**: 结束当前词法作用域或复合语句块。
- **L1526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L1526 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L1527 EN**: Starts a function, method, lambda, or structured scope: `[&](const characteristics::DummyProcedure &dummy) {`.
  **L1527 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const characteristics::DummyProcedure &dummy) {`。
- **L1528 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1528 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckProcedureArg(arg, proc, dummy, dummyName, context,`.
  **L1529 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckProcedureArg(arg, proc, dummy, dummyName, context,`。
- **L1530 EN**: Executes a standalone statement or declaration: `ignoreImplicitVsExplicit);`.
  **L1530 CN**: 执行一条独立语句或声明：`ignoreImplicitVsExplicit);`。
- **L1531 EN**: Closes the current lexical scope or compound statement.
  **L1531 CN**: 结束当前词法作用域或复合语句块。
- **L1532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L1532 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L1533 EN**: Starts a function, method, lambda, or structured scope: `[&](const characteristics::AlternateReturn &) {`.
  **L1533 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const characteristics::AlternateReturn &) {`。
- **L1534 EN**: Comment explains nearby logic, intent, or metadata: `All semantic checking is done elsewhere`.
  **L1534 CN**: 注释说明附近代码的逻辑、意图或元数据：`All semantic checking is done elsewhere`。
- **L1535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L1535 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L1536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L1536 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。

### Lines 1537-1560

````cpp
      dummy.u);
}

static void RearrangeArguments(const characteristics::Procedure &proc,
    evaluate::ActualArguments &actuals, parser::ContextualMessages &messages) {
  CHECK(proc.HasExplicitInterface());
  if (actuals.size() < proc.dummyArguments.size()) {
    actuals.resize(proc.dummyArguments.size());
  } else if (actuals.size() > proc.dummyArguments.size()) {
    messages.Say(
        "Too many actual arguments (%zd) passed to procedure that expects only %zd"_err_en_US,
        actuals.size(), proc.dummyArguments.size());
  }
  std::map<std::string, evaluate::ActualArgument> kwArgs;
  bool anyKeyword{false};
  int which{1};
  for (auto &x : actuals) {
    if (!x) {
    } else if (x->keyword()) {
      auto emplaced{
          kwArgs.try_emplace(x->keyword()->ToString(), std::move(*x))};
      if (!emplaced.second) {
        messages.Say(*x->keyword(),
            "Argument keyword '%s=' appears on more than one effective argument in this procedure reference"_err_en_US,
````
- **L1537 EN**: Executes a standalone statement or declaration: `dummy.u);`.
  **L1537 CN**: 执行一条独立语句或声明：`dummy.u);`。
- **L1538 EN**: Closes the current lexical scope or compound statement.
  **L1538 CN**: 结束当前词法作用域或复合语句块。
- **L1539 EN**: Blank line separating nearby declarations or logic blocks.
  **L1539 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void RearrangeArguments(const characteristics::Procedure &proc,`.
  **L1540 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void RearrangeArguments(const characteristics::Procedure &proc,`。
- **L1541 EN**: Continues the surrounding expression or declaration: `evaluate::ActualArguments &actuals, parser::ContextualMessages &messages) {`.
  **L1541 CN**: 继续构造周围的表达式或声明：`evaluate::ActualArguments &actuals, parser::ContextualMessages &messages) {`。
- **L1542 EN**: Executes a call or declaration centered on `CHECK`.
  **L1542 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L1543 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1543 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1544 EN**: Executes a call or declaration centered on `actuals.resize`.
  **L1544 CN**: 执行以 `actuals.resize` 为核心的调用或声明。
- **L1545 EN**: Transitions from the previous branch into an `else if` condition.
  **L1545 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1546 EN**: Continues logic associated with callable symbol `Say`.
  **L1546 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Too many actual arguments (%zd) passed to procedure that expects only %zd"_err_en_US,`.
  **L1547 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Too many actual arguments (%zd) passed to procedure that expects only %zd"_err_en_US,`。
- **L1548 EN**: Executes a call or declaration centered on `actuals.size`.
  **L1548 CN**: 执行以 `actuals.size` 为核心的调用或声明。
- **L1549 EN**: Closes the current lexical scope or compound statement.
  **L1549 CN**: 结束当前词法作用域或复合语句块。
- **L1550 EN**: Executes a standalone statement or declaration: `std::map<std::string, evaluate::ActualArgument> kwArgs;`.
  **L1550 CN**: 执行一条独立语句或声明：`std::map<std::string, evaluate::ActualArgument> kwArgs;`。
- **L1551 EN**: Executes a standalone statement or declaration: `bool anyKeyword{false};`.
  **L1551 CN**: 执行一条独立语句或声明：`bool anyKeyword{false};`。
- **L1552 EN**: Executes a standalone statement or declaration: `int which{1};`.
  **L1552 CN**: 执行一条独立语句或声明：`int which{1};`。
- **L1553 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1553 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1554 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1554 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1555 EN**: Transitions from the previous branch into an `else if` condition.
  **L1555 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1556 EN**: Continues the surrounding expression or declaration: `auto emplaced{`.
  **L1556 CN**: 继续构造周围的表达式或声明：`auto emplaced{`。
- **L1557 EN**: Executes a call or declaration centered on `kwArgs.try_emplace`.
  **L1557 CN**: 执行以 `kwArgs.try_emplace` 为核心的调用或声明。
- **L1558 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1558 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(*x->keyword(),`.
  **L1559 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(*x->keyword(),`。
- **L1560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Argument keyword '%s=' appears on more than one effective argument in this procedure reference"_err_en_US,`.
  **L1560 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Argument keyword '%s=' appears on more than one effective argument in this procedure reference"_err_en_US,`。

### Lines 1561-1584

````cpp
            *x->keyword());
      }
      x.reset();
      anyKeyword = true;
    } else if (anyKeyword) {
      messages.Say(x ? x->sourceLocation() : std::nullopt,
          "Actual argument #%d without a keyword may not follow any actual argument with a keyword"_err_en_US,
          which);
    }
    ++which;
  }
  if (!kwArgs.empty()) {
    int index{0};
    for (const auto &dummy : proc.dummyArguments) {
      if (!dummy.name.empty()) {
        auto iter{kwArgs.find(dummy.name)};
        if (iter != kwArgs.end()) {
          evaluate::ActualArgument &x{iter->second};
          if (actuals[index]) {
            messages.Say(*x.keyword(),
                "Keyword argument '%s=' has already been specified positionally (#%d) in this procedure reference"_err_en_US,
                *x.keyword(), index + 1);
          } else {
            actuals[index] = std::move(x);
````
- **L1561 EN**: Comment explains nearby logic, intent, or metadata: `x->keyword());`.
  **L1561 CN**: 注释说明附近代码的逻辑、意图或元数据：`x->keyword());`。
- **L1562 EN**: Closes the current lexical scope or compound statement.
  **L1562 CN**: 结束当前词法作用域或复合语句块。
- **L1563 EN**: Executes a call or declaration centered on `x.reset`.
  **L1563 CN**: 执行以 `x.reset` 为核心的调用或声明。
- **L1564 EN**: Executes a standalone statement or declaration: `anyKeyword = true;`.
  **L1564 CN**: 执行一条独立语句或声明：`anyKeyword = true;`。
- **L1565 EN**: Transitions from the previous branch into an `else if` condition.
  **L1565 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(x ? x->sourceLocation() : std::nullopt,`.
  **L1566 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(x ? x->sourceLocation() : std::nullopt,`。
- **L1567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Actual argument #%d without a keyword may not follow any actual argument with a keyword"_err_en_US,`.
  **L1567 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Actual argument #%d without a keyword may not follow any actual argument with a keyword"_err_en_US,`。
- **L1568 EN**: Executes a standalone statement or declaration: `which);`.
  **L1568 CN**: 执行一条独立语句或声明：`which);`。
- **L1569 EN**: Closes the current lexical scope or compound statement.
  **L1569 CN**: 结束当前词法作用域或复合语句块。
- **L1570 EN**: Executes a standalone statement or declaration: `++which;`.
  **L1570 CN**: 执行一条独立语句或声明：`++which;`。
- **L1571 EN**: Closes the current lexical scope or compound statement.
  **L1571 CN**: 结束当前词法作用域或复合语句块。
- **L1572 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1572 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1573 EN**: Executes a standalone statement or declaration: `int index{0};`.
  **L1573 CN**: 执行一条独立语句或声明：`int index{0};`。
- **L1574 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1574 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1575 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1575 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1576 EN**: Executes a call or declaration centered on `iter{kwArgs.find`.
  **L1576 CN**: 执行以 `iter{kwArgs.find` 为核心的调用或声明。
- **L1577 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1577 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1578 EN**: Executes a standalone statement or declaration: `evaluate::ActualArgument &x{iter->second};`.
  **L1578 CN**: 执行一条独立语句或声明：`evaluate::ActualArgument &x{iter->second};`。
- **L1579 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1579 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(*x.keyword(),`.
  **L1580 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(*x.keyword(),`。
- **L1581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Keyword argument '%s=' has already been specified positionally (#%d) in this procedure reference"_err_en_US,`.
  **L1581 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Keyword argument '%s=' has already been specified positionally (#%d) in this procedure reference"_err_en_US,`。
- **L1582 EN**: Comment explains nearby logic, intent, or metadata: `x.keyword(), index + 1);`.
  **L1582 CN**: 注释说明附近代码的逻辑、意图或元数据：`x.keyword(), index + 1);`。
- **L1583 EN**: Transitions from the previous branch into the alternative path.
  **L1583 CN**: 从前一个分支过渡到备选路径。
- **L1584 EN**: Executes a call or declaration centered on `std::move`.
  **L1584 CN**: 执行以 `std::move` 为核心的调用或声明。

### Lines 1585-1608

````cpp
          }
          kwArgs.erase(iter);
        }
      }
      ++index;
    }
    for (auto &bad : kwArgs) {
      evaluate::ActualArgument &x{bad.second};
      messages.Say(*x.keyword(),
          "Argument keyword '%s=' is not recognized for this procedure reference"_err_en_US,
          *x.keyword());
    }
  }
}

// 15.8.1(3) -- In a reference to an elemental procedure, if any argument is an
// array, each actual argument that corresponds to an INTENT(OUT) or
// INTENT(INOUT) dummy argument shall be an array. The actual argument to an
// ELEMENTAL procedure must conform.
static bool CheckElementalConformance(parser::ContextualMessages &messages,
    const characteristics::Procedure &proc, evaluate::ActualArguments &actuals,
    evaluate::FoldingContext &context) {
  std::optional<evaluate::Shape> shape;
  std::string shapeName;
````
- **L1585 EN**: Closes the current lexical scope or compound statement.
  **L1585 CN**: 结束当前词法作用域或复合语句块。
- **L1586 EN**: Executes a call or declaration centered on `kwArgs.erase`.
  **L1586 CN**: 执行以 `kwArgs.erase` 为核心的调用或声明。
- **L1587 EN**: Closes the current lexical scope or compound statement.
  **L1587 CN**: 结束当前词法作用域或复合语句块。
- **L1588 EN**: Closes the current lexical scope or compound statement.
  **L1588 CN**: 结束当前词法作用域或复合语句块。
- **L1589 EN**: Executes a standalone statement or declaration: `++index;`.
  **L1589 CN**: 执行一条独立语句或声明：`++index;`。
- **L1590 EN**: Closes the current lexical scope or compound statement.
  **L1590 CN**: 结束当前词法作用域或复合语句块。
- **L1591 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1591 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1592 EN**: Executes a standalone statement or declaration: `evaluate::ActualArgument &x{bad.second};`.
  **L1592 CN**: 执行一条独立语句或声明：`evaluate::ActualArgument &x{bad.second};`。
- **L1593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(*x.keyword(),`.
  **L1593 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(*x.keyword(),`。
- **L1594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Argument keyword '%s=' is not recognized for this procedure reference"_err_en_US,`.
  **L1594 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Argument keyword '%s=' is not recognized for this procedure reference"_err_en_US,`。
- **L1595 EN**: Comment explains nearby logic, intent, or metadata: `x.keyword());`.
  **L1595 CN**: 注释说明附近代码的逻辑、意图或元数据：`x.keyword());`。
- **L1596 EN**: Closes the current lexical scope or compound statement.
  **L1596 CN**: 结束当前词法作用域或复合语句块。
- **L1597 EN**: Closes the current lexical scope or compound statement.
  **L1597 CN**: 结束当前词法作用域或复合语句块。
- **L1598 EN**: Closes the current lexical scope or compound statement.
  **L1598 CN**: 结束当前词法作用域或复合语句块。
- **L1599 EN**: Blank line separating nearby declarations or logic blocks.
  **L1599 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1600 EN**: Comment explains nearby logic, intent, or metadata: `15.8.1(3) -- In a reference to an elemental procedure, if any argument is an`.
  **L1600 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.8.1(3) -- In a reference to an elemental procedure, if any argument is an`。
- **L1601 EN**: Comment explains nearby logic, intent, or metadata: `array, each actual argument that corresponds to an INTENT(OUT) or`.
  **L1601 CN**: 注释说明附近代码的逻辑、意图或元数据：`array, each actual argument that corresponds to an INTENT(OUT) or`。
- **L1602 EN**: Comment explains nearby logic, intent, or metadata: `INTENT(INOUT) dummy argument shall be an array. The actual argument to an`.
  **L1602 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTENT(INOUT) dummy argument shall be an array. The actual argument to an`。
- **L1603 EN**: Comment explains nearby logic, intent, or metadata: `ELEMENTAL procedure must conform.`.
  **L1603 CN**: 注释说明附近代码的逻辑、意图或元数据：`ELEMENTAL procedure must conform.`。
- **L1604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static bool CheckElementalConformance(parser::ContextualMessages &messages,`.
  **L1604 CN**: 继续一个多行参数列表、初始化器或聚合项：`static bool CheckElementalConformance(parser::ContextualMessages &messages,`。
- **L1605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const characteristics::Procedure &proc, evaluate::ActualArguments &actuals,`.
  **L1605 CN**: 继续一个多行参数列表、初始化器或聚合项：`const characteristics::Procedure &proc, evaluate::ActualArguments &actuals,`。
- **L1606 EN**: Continues the surrounding expression or declaration: `evaluate::FoldingContext &context) {`.
  **L1606 CN**: 继续构造周围的表达式或声明：`evaluate::FoldingContext &context) {`。
- **L1607 EN**: Executes a standalone statement or declaration: `std::optional<evaluate::Shape> shape;`.
  **L1607 CN**: 执行一条独立语句或声明：`std::optional<evaluate::Shape> shape;`。
- **L1608 EN**: Executes a standalone statement or declaration: `std::string shapeName;`.
  **L1608 CN**: 执行一条独立语句或声明：`std::string shapeName;`。

### Lines 1609-1632

````cpp
  int index{0};
  bool hasArrayArg{false};
  for (const auto &arg : actuals) {
    if (arg && !arg->isAlternateReturn() && arg->Rank() > 0) {
      hasArrayArg = true;
      break;
    }
  }
  for (const auto &arg : actuals) {
    const auto &dummy{proc.dummyArguments.at(index++)};
    if (arg) {
      if (const auto *expr{arg->UnwrapExpr()}) {
        if (const auto *wholeSymbol{evaluate::UnwrapWholeSymbolDataRef(arg)}) {
          wholeSymbol = &ResolveAssociations(*wholeSymbol);
          if (IsAssumedSizeArray(*wholeSymbol)) {
            evaluate::SayWithDeclaration(messages, *wholeSymbol,
                "Whole assumed-size array '%s' may not be used as an argument to an elemental procedure"_err_en_US,
                wholeSymbol->name());
          } else if (IsAssumedRank(*wholeSymbol)) {
            evaluate::SayWithDeclaration(messages, *wholeSymbol,
                "Assumed-rank array '%s' may not be used as an argument to an elemental procedure"_err_en_US,
                wholeSymbol->name());
          }
        }
````
- **L1609 EN**: Executes a standalone statement or declaration: `int index{0};`.
  **L1609 CN**: 执行一条独立语句或声明：`int index{0};`。
- **L1610 EN**: Executes a standalone statement or declaration: `bool hasArrayArg{false};`.
  **L1610 CN**: 执行一条独立语句或声明：`bool hasArrayArg{false};`。
- **L1611 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1611 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1612 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1612 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1613 EN**: Executes a standalone statement or declaration: `hasArrayArg = true;`.
  **L1613 CN**: 执行一条独立语句或声明：`hasArrayArg = true;`。
- **L1614 EN**: Exits the nearest loop or switch statement.
  **L1614 CN**: 退出最近的循环或 switch 语句。
- **L1615 EN**: Closes the current lexical scope or compound statement.
  **L1615 CN**: 结束当前词法作用域或复合语句块。
- **L1616 EN**: Closes the current lexical scope or compound statement.
  **L1616 CN**: 结束当前词法作用域或复合语句块。
- **L1617 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1617 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1618 EN**: Executes a call or declaration centered on `&dummy{proc.dummyArguments.at`.
  **L1618 CN**: 执行以 `&dummy{proc.dummyArguments.at` 为核心的调用或声明。
- **L1619 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1619 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1620 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1620 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1621 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1621 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1622 EN**: Executes a call or declaration centered on `&ResolveAssociations`.
  **L1622 CN**: 执行以 `&ResolveAssociations` 为核心的调用或声明。
- **L1623 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1623 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, *wholeSymbol,`.
  **L1624 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, *wholeSymbol,`。
- **L1625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Whole assumed-size array '%s' may not be used as an argument to an elemental procedure"_err_en_US,`.
  **L1625 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Whole assumed-size array '%s' may not be used as an argument to an elemental procedure"_err_en_US,`。
- **L1626 EN**: Executes a call or declaration centered on `wholeSymbol->name`.
  **L1626 CN**: 执行以 `wholeSymbol->name` 为核心的调用或声明。
- **L1627 EN**: Transitions from the previous branch into an `else if` condition.
  **L1627 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::SayWithDeclaration(messages, *wholeSymbol,`.
  **L1628 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::SayWithDeclaration(messages, *wholeSymbol,`。
- **L1629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-rank array '%s' may not be used as an argument to an elemental procedure"_err_en_US,`.
  **L1629 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-rank array '%s' may not be used as an argument to an elemental procedure"_err_en_US,`。
- **L1630 EN**: Executes a call or declaration centered on `wholeSymbol->name`.
  **L1630 CN**: 执行以 `wholeSymbol->name` 为核心的调用或声明。
- **L1631 EN**: Closes the current lexical scope or compound statement.
  **L1631 CN**: 结束当前词法作用域或复合语句块。
- **L1632 EN**: Closes the current lexical scope or compound statement.
  **L1632 CN**: 结束当前词法作用域或复合语句块。

### Lines 1633-1656

````cpp
        if (auto argShape{evaluate::GetShape(context, *expr)}) {
          if (GetRank(*argShape) > 0) {
            std::string argName{"actual argument ("s + expr->AsFortran() +
                ") corresponding to dummy argument #" + std::to_string(index) +
                " ('" + dummy.name + "')"};
            if (shape) {
              if (!evaluate::CheckConformance(messages, *shape, *argShape,
                      evaluate::CheckConformanceFlags::None, shapeName.c_str(),
                      argName.c_str())
                      .value_or(true)) {
                return false;
              }
            } else {
              shape = std::move(argShape);
              shapeName = argName;
            }
          } else if ((dummy.GetIntent() == common::Intent::Out ||
                         dummy.GetIntent() == common::Intent::InOut) &&
              hasArrayArg) {
            messages.Say(
                "In an elemental procedure reference with at least one array argument, actual argument %s that corresponds to an INTENT(OUT) or INTENT(INOUT) dummy argument must be an array"_err_en_US,
                expr->AsFortran());
          }
        }
````
- **L1633 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1633 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1634 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1634 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1635 EN**: Continues logic associated with callable symbol `argument`.
  **L1635 CN**: 继续与可调用符号 `argument` 相关的逻辑。
- **L1636 EN**: Continues logic associated with callable symbol `to_string`.
  **L1636 CN**: 继续与可调用符号 `to_string` 相关的逻辑。
- **L1637 EN**: Executes a call or declaration centered on `"`.
  **L1637 CN**: 执行以 `"` 为核心的调用或声明。
- **L1638 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1638 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1639 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1639 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::CheckConformanceFlags::None, shapeName.c_str(),`.
  **L1640 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::CheckConformanceFlags::None, shapeName.c_str(),`。
- **L1641 EN**: Continues logic associated with callable symbol `c_str`.
  **L1641 CN**: 继续与可调用符号 `c_str` 相关的逻辑。
- **L1642 EN**: Starts a function, method, lambda, or structured scope: `.value_or(true)) {`.
  **L1642 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.value_or(true)) {`。
- **L1643 EN**: Returns from the current function with `false`.
  **L1643 CN**: 以 `false` 从当前函数返回。
- **L1644 EN**: Closes the current lexical scope or compound statement.
  **L1644 CN**: 结束当前词法作用域或复合语句块。
- **L1645 EN**: Transitions from the previous branch into the alternative path.
  **L1645 CN**: 从前一个分支过渡到备选路径。
- **L1646 EN**: Executes a call or declaration centered on `std::move`.
  **L1646 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L1647 EN**: Executes a standalone statement or declaration: `shapeName = argName;`.
  **L1647 CN**: 执行一条独立语句或声明：`shapeName = argName;`。
- **L1648 EN**: Closes the current lexical scope or compound statement.
  **L1648 CN**: 结束当前词法作用域或复合语句块。
- **L1649 EN**: Transitions from the previous branch into an `else if` condition.
  **L1649 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1650 EN**: Continues logic associated with callable symbol `GetIntent`.
  **L1650 CN**: 继续与可调用符号 `GetIntent` 相关的逻辑。
- **L1651 EN**: Continues the surrounding expression or declaration: `hasArrayArg) {`.
  **L1651 CN**: 继续构造周围的表达式或声明：`hasArrayArg) {`。
- **L1652 EN**: Continues logic associated with callable symbol `Say`.
  **L1652 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"In an elemental procedure reference with at least one array argument, actual argument %s that corresponds to an INTENT(OUT) or INTENT(INOUT) dummy argument must be an array"_err_en_US,`.
  **L1653 CN**: 继续一个多行参数列表、初始化器或聚合项：`"In an elemental procedure reference with at least one array argument, actual argument %s that corresponds to an INTENT(OUT) or INTENT(INOUT) dummy argument must be an array"_err_en_US,`。
- **L1654 EN**: Executes a call or declaration centered on `expr->AsFortran`.
  **L1654 CN**: 执行以 `expr->AsFortran` 为核心的调用或声明。
- **L1655 EN**: Closes the current lexical scope or compound statement.
  **L1655 CN**: 结束当前词法作用域或复合语句块。
- **L1656 EN**: Closes the current lexical scope or compound statement.
  **L1656 CN**: 结束当前词法作用域或复合语句块。

### Lines 1657-1680

````cpp
      }
    }
  }
  return true;
}

// ASSOCIATED (16.9.16)
static void CheckAssociated(evaluate::ActualArguments &arguments,
    SemanticsContext &semanticsContext, const Scope *scope) {
  evaluate::FoldingContext &foldingContext{semanticsContext.foldingContext()};
  parser::ContextualMessages &messages{foldingContext.messages()};
  bool ok{true};
  if (arguments.size() < 2) {
    return;
  }
  if (const auto &pointerArg{arguments[0]}) {
    if (const auto *pointerExpr{pointerArg->UnwrapExpr()}) {
      if (!IsPointer(*pointerExpr)) {
        messages.Say(pointerArg->sourceLocation(),
            "POINTER= argument of ASSOCIATED() must be a pointer"_err_en_US);
        return;
      }
      if (const auto &targetArg{arguments[1]}) {
        // The standard requires that the TARGET= argument, when present,
````
- **L1657 EN**: Closes the current lexical scope or compound statement.
  **L1657 CN**: 结束当前词法作用域或复合语句块。
- **L1658 EN**: Closes the current lexical scope or compound statement.
  **L1658 CN**: 结束当前词法作用域或复合语句块。
- **L1659 EN**: Closes the current lexical scope or compound statement.
  **L1659 CN**: 结束当前词法作用域或复合语句块。
- **L1660 EN**: Returns from the current function with `true`.
  **L1660 CN**: 以 `true` 从当前函数返回。
- **L1661 EN**: Closes the current lexical scope or compound statement.
  **L1661 CN**: 结束当前词法作用域或复合语句块。
- **L1662 EN**: Blank line separating nearby declarations or logic blocks.
  **L1662 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1663 EN**: Comment explains nearby logic, intent, or metadata: `ASSOCIATED (16.9.16)`.
  **L1663 CN**: 注释说明附近代码的逻辑、意图或元数据：`ASSOCIATED (16.9.16)`。
- **L1664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckAssociated(evaluate::ActualArguments &arguments,`.
  **L1664 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckAssociated(evaluate::ActualArguments &arguments,`。
- **L1665 EN**: Continues the surrounding expression or declaration: `SemanticsContext &semanticsContext, const Scope *scope) {`.
  **L1665 CN**: 继续构造周围的表达式或声明：`SemanticsContext &semanticsContext, const Scope *scope) {`。
- **L1666 EN**: Executes a call or declaration centered on `&foldingContext{semanticsContext.foldingContext`.
  **L1666 CN**: 执行以 `&foldingContext{semanticsContext.foldingContext` 为核心的调用或声明。
- **L1667 EN**: Executes a call or declaration centered on `&messages{foldingContext.messages`.
  **L1667 CN**: 执行以 `&messages{foldingContext.messages` 为核心的调用或声明。
- **L1668 EN**: Executes a standalone statement or declaration: `bool ok{true};`.
  **L1668 CN**: 执行一条独立语句或声明：`bool ok{true};`。
- **L1669 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1669 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1670 EN**: Returns from the current function with `void`.
  **L1670 CN**: 以 `void` 从当前函数返回。
- **L1671 EN**: Closes the current lexical scope or compound statement.
  **L1671 CN**: 结束当前词法作用域或复合语句块。
- **L1672 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1672 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1673 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1673 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1674 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1674 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(pointerArg->sourceLocation(),`.
  **L1675 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(pointerArg->sourceLocation(),`。
- **L1676 EN**: Executes a call or declaration centered on `ASSOCIATED`.
  **L1676 CN**: 执行以 `ASSOCIATED` 为核心的调用或声明。
- **L1677 EN**: Returns from the current function with `void`.
  **L1677 CN**: 以 `void` 从当前函数返回。
- **L1678 EN**: Closes the current lexical scope or compound statement.
  **L1678 CN**: 结束当前词法作用域或复合语句块。
- **L1679 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1679 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1680 EN**: Comment explains nearby logic, intent, or metadata: `The standard requires that the TARGET= argument, when present,`.
  **L1680 CN**: 注释说明附近代码的逻辑、意图或元数据：`The standard requires that the TARGET= argument, when present,`。

### Lines 1681-1704

````cpp
        // be type compatible with the POINTER= for a data pointer.  In
        // the case of procedure pointers, the standard requires that it
        // be a valid RHS for a pointer assignment that has the POINTER=
        // argument as its LHS.  Some popular compilers misinterpret this
        // requirement more strongly than necessary, and actually validate
        // the POINTER= argument as if it were serving as the LHS of a pointer
        // assignment.  This, perhaps unintentionally, excludes function
        // results, including NULL(), from being used there, as well as
        // INTENT(IN) dummy pointers.  Detect these conditions and emit
        // portability warnings.
        if (semanticsContext.ShouldWarn(common::UsageWarning::Portability)) {
          if (!evaluate::ExtractDataRef(*pointerExpr) &&
              !evaluate::IsProcedurePointer(*pointerExpr)) {
            foldingContext.Warn(common::UsageWarning::Portability,
                pointerArg->sourceLocation(),
                "POINTER= argument of ASSOCIATED() is required by some other compilers to be a pointer"_port_en_US);
          } else if (scope && !evaluate::UnwrapProcedureRef(*pointerExpr)) {
            if (auto whyNot{WhyNotDefinable(
                    pointerArg->sourceLocation().value_or(messages.at()),
                    *scope,
                    DefinabilityFlags{DefinabilityFlag::PointerDefinition,
                        DefinabilityFlag::DoNotNoteDefinition},
                    *pointerExpr)}) {
              if (whyNot->IsFatal()) {
````
- **L1681 EN**: Comment explains nearby logic, intent, or metadata: `be type compatible with the POINTER= for a data pointer.  In`.
  **L1681 CN**: 注释说明附近代码的逻辑、意图或元数据：`be type compatible with the POINTER= for a data pointer.  In`。
- **L1682 EN**: Comment explains nearby logic, intent, or metadata: `the case of procedure pointers, the standard requires that it`.
  **L1682 CN**: 注释说明附近代码的逻辑、意图或元数据：`the case of procedure pointers, the standard requires that it`。
- **L1683 EN**: Comment explains nearby logic, intent, or metadata: `be a valid RHS for a pointer assignment that has the POINTER=`.
  **L1683 CN**: 注释说明附近代码的逻辑、意图或元数据：`be a valid RHS for a pointer assignment that has the POINTER=`。
- **L1684 EN**: Comment explains nearby logic, intent, or metadata: `argument as its LHS.  Some popular compilers misinterpret this`.
  **L1684 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument as its LHS.  Some popular compilers misinterpret this`。
- **L1685 EN**: Comment explains nearby logic, intent, or metadata: `requirement more strongly than necessary, and actually validate`.
  **L1685 CN**: 注释说明附近代码的逻辑、意图或元数据：`requirement more strongly than necessary, and actually validate`。
- **L1686 EN**: Comment explains nearby logic, intent, or metadata: `the POINTER= argument as if it were serving as the LHS of a pointer`.
  **L1686 CN**: 注释说明附近代码的逻辑、意图或元数据：`the POINTER= argument as if it were serving as the LHS of a pointer`。
- **L1687 EN**: Comment explains nearby logic, intent, or metadata: `assignment.  This, perhaps unintentionally, excludes function`.
  **L1687 CN**: 注释说明附近代码的逻辑、意图或元数据：`assignment.  This, perhaps unintentionally, excludes function`。
- **L1688 EN**: Comment explains nearby logic, intent, or metadata: `results, including NULL(), from being used there, as well as`.
  **L1688 CN**: 注释说明附近代码的逻辑、意图或元数据：`results, including NULL(), from being used there, as well as`。
- **L1689 EN**: Comment explains nearby logic, intent, or metadata: `INTENT(IN) dummy pointers.  Detect these conditions and emit`.
  **L1689 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTENT(IN) dummy pointers.  Detect these conditions and emit`。
- **L1690 EN**: Comment explains nearby logic, intent, or metadata: `portability warnings.`.
  **L1690 CN**: 注释说明附近代码的逻辑、意图或元数据：`portability warnings.`。
- **L1691 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1691 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1692 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1692 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1693 EN**: Starts a function, method, lambda, or structured scope: `!evaluate::IsProcedurePointer(*pointerExpr)) {`.
  **L1693 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!evaluate::IsProcedurePointer(*pointerExpr)) {`。
- **L1694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::Portability,`.
  **L1694 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::Portability,`。
- **L1695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `pointerArg->sourceLocation(),`.
  **L1695 CN**: 继续一个多行参数列表、初始化器或聚合项：`pointerArg->sourceLocation(),`。
- **L1696 EN**: Executes a call or declaration centered on `ASSOCIATED`.
  **L1696 CN**: 执行以 `ASSOCIATED` 为核心的调用或声明。
- **L1697 EN**: Transitions from the previous branch into an `else if` condition.
  **L1697 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1698 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1698 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `pointerArg->sourceLocation().value_or(messages.at()),`.
  **L1699 CN**: 继续一个多行参数列表、初始化器或聚合项：`pointerArg->sourceLocation().value_or(messages.at()),`。
- **L1700 EN**: Comment explains nearby logic, intent, or metadata: `scope,`.
  **L1700 CN**: 注释说明附近代码的逻辑、意图或元数据：`scope,`。
- **L1701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefinabilityFlags{DefinabilityFlag::PointerDefinition,`.
  **L1701 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefinabilityFlags{DefinabilityFlag::PointerDefinition,`。
- **L1702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefinabilityFlag::DoNotNoteDefinition},`.
  **L1702 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefinabilityFlag::DoNotNoteDefinition},`。
- **L1703 EN**: Comment explains nearby logic, intent, or metadata: `pointerExpr)}) {`.
  **L1703 CN**: 注释说明附近代码的逻辑、意图或元数据：`pointerExpr)}) {`。
- **L1704 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1704 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1705-1728

````cpp
                if (auto *msg{foldingContext.Warn(
                        common::UsageWarning::Portability,
                        pointerArg->sourceLocation(),
                        "POINTER= argument of ASSOCIATED() is required by some other compilers to be a valid left-hand side of a pointer assignment statement"_port_en_US)}) {
                  msg->Attach(std::move(
                      whyNot->set_severity(parser::Severity::Because)));
                }
              } else {
                messages.Say(std::move(*whyNot));
              }
            }
          }
        }
        if (const auto *targetExpr{targetArg->UnwrapExpr()}) {
          if (IsProcedurePointer(*pointerExpr) &&
              !IsBareNullPointer(pointerExpr)) { // POINTER= is a procedure
            if (auto pointerProc{characteristics::Procedure::Characterize(
                    *pointerExpr, foldingContext)}) {
              if (IsBareNullPointer(targetExpr)) {
              } else if (IsProcedurePointerTarget(*targetExpr)) {
                if (auto targetProc{characteristics::Procedure::Characterize(
                        *targetExpr, foldingContext)}) {
                  bool isCall{!!UnwrapProcedureRef(*targetExpr)};
                  std::string whyNot;
````
- **L1705 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1705 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::UsageWarning::Portability,`.
  **L1706 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::UsageWarning::Portability,`。
- **L1707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `pointerArg->sourceLocation(),`.
  **L1707 CN**: 继续一个多行参数列表、初始化器或聚合项：`pointerArg->sourceLocation(),`。
- **L1708 EN**: Starts a function, method, lambda, or structured scope: `"POINTER= argument of ASSOCIATED() is required by some other compilers to be a valid left-hand side of a pointer assignment statement"_port_en_US)}) {`.
  **L1708 CN**: 开始一个函数、方法、lambda 或结构化作用域：`"POINTER= argument of ASSOCIATED() is required by some other compilers to be a valid left-hand side of a pointer assignment statement"_port_en_US)}) {`。
- **L1709 EN**: Continues logic associated with callable symbol `Attach`.
  **L1709 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L1710 EN**: Executes a call or declaration centered on `whyNot->set_severity`.
  **L1710 CN**: 执行以 `whyNot->set_severity` 为核心的调用或声明。
- **L1711 EN**: Closes the current lexical scope or compound statement.
  **L1711 CN**: 结束当前词法作用域或复合语句块。
- **L1712 EN**: Transitions from the previous branch into the alternative path.
  **L1712 CN**: 从前一个分支过渡到备选路径。
- **L1713 EN**: Executes a call or declaration centered on `messages.Say`.
  **L1713 CN**: 执行以 `messages.Say` 为核心的调用或声明。
- **L1714 EN**: Closes the current lexical scope or compound statement.
  **L1714 CN**: 结束当前词法作用域或复合语句块。
- **L1715 EN**: Closes the current lexical scope or compound statement.
  **L1715 CN**: 结束当前词法作用域或复合语句块。
- **L1716 EN**: Closes the current lexical scope or compound statement.
  **L1716 CN**: 结束当前词法作用域或复合语句块。
- **L1717 EN**: Closes the current lexical scope or compound statement.
  **L1717 CN**: 结束当前词法作用域或复合语句块。
- **L1718 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1718 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1719 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1719 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1720 EN**: Continues logic associated with callable symbol `IsBareNullPointer`.
  **L1720 CN**: 继续与可调用符号 `IsBareNullPointer` 相关的逻辑。
- **L1721 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1721 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1722 EN**: Comment explains nearby logic, intent, or metadata: `pointerExpr, foldingContext)}) {`.
  **L1722 CN**: 注释说明附近代码的逻辑、意图或元数据：`pointerExpr, foldingContext)}) {`。
- **L1723 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1723 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1724 EN**: Transitions from the previous branch into an `else if` condition.
  **L1724 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1725 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1725 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1726 EN**: Comment explains nearby logic, intent, or metadata: `targetExpr, foldingContext)}) {`.
  **L1726 CN**: 注释说明附近代码的逻辑、意图或元数据：`targetExpr, foldingContext)}) {`。
- **L1727 EN**: Executes a call or declaration centered on `isCall{!!UnwrapProcedureRef`.
  **L1727 CN**: 执行以 `isCall{!!UnwrapProcedureRef` 为核心的调用或声明。
- **L1728 EN**: Executes a standalone statement or declaration: `std::string whyNot;`.
  **L1728 CN**: 执行一条独立语句或声明：`std::string whyNot;`。

### Lines 1729-1752

````cpp
                  std::optional<std::string> warning;
                  const auto *targetProcDesignator{
                      evaluate::UnwrapExpr<evaluate::ProcedureDesignator>(
                          *targetExpr)};
                  const evaluate::SpecificIntrinsic *specificIntrinsic{
                      targetProcDesignator
                          ? targetProcDesignator->GetSpecificIntrinsic()
                          : nullptr};
                  std::optional<parser::MessageFixedText> msg{
                      CheckProcCompatibility(isCall, pointerProc, &*targetProc,
                          specificIntrinsic, whyNot, warning,
                          /*ignoreImplicitVsExplicit=*/false)};
                  std::optional<common::UsageWarning> whichWarning;
                  if (!msg && warning &&
                      semanticsContext.ShouldWarn(
                          common::UsageWarning::ProcDummyArgShapes)) {
                    whichWarning = common::UsageWarning::ProcDummyArgShapes;
                    msg =
                        "Procedures '%s' and '%s' may not be completely compatible: %s"_warn_en_US;
                    whyNot = std::move(*warning);
                  } else if (msg && !msg->IsFatal() &&
                      semanticsContext.ShouldWarn(
                          common::UsageWarning::ProcPointerCompatibility)) {
                    whichWarning =
````
- **L1729 EN**: Executes a standalone statement or declaration: `std::optional<std::string> warning;`.
  **L1729 CN**: 执行一条独立语句或声明：`std::optional<std::string> warning;`。
- **L1730 EN**: Continues the surrounding expression or declaration: `const auto *targetProcDesignator{`.
  **L1730 CN**: 继续构造周围的表达式或声明：`const auto *targetProcDesignator{`。
- **L1731 EN**: Continues logic associated with callable symbol `ProcedureDesignator>`.
  **L1731 CN**: 继续与可调用符号 `ProcedureDesignator>` 相关的逻辑。
- **L1732 EN**: Comment explains nearby logic, intent, or metadata: `targetExpr)};`.
  **L1732 CN**: 注释说明附近代码的逻辑、意图或元数据：`targetExpr)};`。
- **L1733 EN**: Continues the surrounding expression or declaration: `const evaluate::SpecificIntrinsic *specificIntrinsic{`.
  **L1733 CN**: 继续构造周围的表达式或声明：`const evaluate::SpecificIntrinsic *specificIntrinsic{`。
- **L1734 EN**: Continues the surrounding expression or declaration: `targetProcDesignator`.
  **L1734 CN**: 继续构造周围的表达式或声明：`targetProcDesignator`。
- **L1735 EN**: Continues logic associated with callable symbol `GetSpecificIntrinsic`.
  **L1735 CN**: 继续与可调用符号 `GetSpecificIntrinsic` 相关的逻辑。
- **L1736 EN**: Executes a standalone statement or declaration: `: nullptr};`.
  **L1736 CN**: 执行一条独立语句或声明：`: nullptr};`。
- **L1737 EN**: Continues the surrounding expression or declaration: `std::optional<parser::MessageFixedText> msg{`.
  **L1737 CN**: 继续构造周围的表达式或声明：`std::optional<parser::MessageFixedText> msg{`。
- **L1738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckProcCompatibility(isCall, pointerProc, &*targetProc,`.
  **L1738 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckProcCompatibility(isCall, pointerProc, &*targetProc,`。
- **L1739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `specificIntrinsic, whyNot, warning,`.
  **L1739 CN**: 继续一个多行参数列表、初始化器或聚合项：`specificIntrinsic, whyNot, warning,`。
- **L1740 EN**: Comment explains nearby logic, intent, or metadata: `ignoreImplicitVsExplicit=*/false)};`.
  **L1740 CN**: 注释说明附近代码的逻辑、意图或元数据：`ignoreImplicitVsExplicit=*/false)};`。
- **L1741 EN**: Executes a standalone statement or declaration: `std::optional<common::UsageWarning> whichWarning;`.
  **L1741 CN**: 执行一条独立语句或声明：`std::optional<common::UsageWarning> whichWarning;`。
- **L1742 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1742 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1743 EN**: Continues logic associated with callable symbol `ShouldWarn`.
  **L1743 CN**: 继续与可调用符号 `ShouldWarn` 相关的逻辑。
- **L1744 EN**: Continues the surrounding expression or declaration: `common::UsageWarning::ProcDummyArgShapes)) {`.
  **L1744 CN**: 继续构造周围的表达式或声明：`common::UsageWarning::ProcDummyArgShapes)) {`。
- **L1745 EN**: Executes a standalone statement or declaration: `whichWarning = common::UsageWarning::ProcDummyArgShapes;`.
  **L1745 CN**: 执行一条独立语句或声明：`whichWarning = common::UsageWarning::ProcDummyArgShapes;`。
- **L1746 EN**: Continues the surrounding expression or declaration: `msg =`.
  **L1746 CN**: 继续构造周围的表达式或声明：`msg =`。
- **L1747 EN**: Executes a standalone statement or declaration: `"Procedures '%s' and '%s' may not be completely compatible: %s"_warn_en_US;`.
  **L1747 CN**: 执行一条独立语句或声明：`"Procedures '%s' and '%s' may not be completely compatible: %s"_warn_en_US;`。
- **L1748 EN**: Executes a call or declaration centered on `std::move`.
  **L1748 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L1749 EN**: Transitions from the previous branch into an `else if` condition.
  **L1749 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1750 EN**: Continues logic associated with callable symbol `ShouldWarn`.
  **L1750 CN**: 继续与可调用符号 `ShouldWarn` 相关的逻辑。
- **L1751 EN**: Continues the surrounding expression or declaration: `common::UsageWarning::ProcPointerCompatibility)) {`.
  **L1751 CN**: 继续构造周围的表达式或声明：`common::UsageWarning::ProcPointerCompatibility)) {`。
- **L1752 EN**: Continues the surrounding expression or declaration: `whichWarning =`.
  **L1752 CN**: 继续构造周围的表达式或声明：`whichWarning =`。

### Lines 1753-1776

````cpp
                        common::UsageWarning::ProcPointerCompatibility;
                  }
                  if (msg && (msg->IsFatal() || whichWarning)) {
                    if (auto *said{messages.Say(std::move(*msg),
                            "pointer '" + pointerExpr->AsFortran() + "'",
                            targetExpr->AsFortran(), whyNot)};
                        said && whichWarning) {
                      said->set_usageWarning(*whichWarning);
                    }
                  }
                }
              } else if (!IsNullProcedurePointer(targetExpr)) {
                messages.Say(
                    "POINTER= argument '%s' is a procedure pointer but the TARGET= argument '%s' is not a procedure or procedure pointer"_err_en_US,
                    pointerExpr->AsFortran(), targetExpr->AsFortran());
              }
            }
          } else if (IsVariable(*targetExpr) || IsNullPointer(targetExpr)) {
            // Object pointer and target
            if (ExtractDataRef(*targetExpr)) {
              if (SymbolVector symbols{GetSymbolVector(*targetExpr)};
                  !evaluate::GetLastTarget(symbols)) {
                parser::Message *msg{messages.Say(targetArg->sourceLocation(),
                    "TARGET= argument '%s' must have either the POINTER or the TARGET attribute"_err_en_US,
````
- **L1753 EN**: Executes a standalone statement or declaration: `common::UsageWarning::ProcPointerCompatibility;`.
  **L1753 CN**: 执行一条独立语句或声明：`common::UsageWarning::ProcPointerCompatibility;`。
- **L1754 EN**: Closes the current lexical scope or compound statement.
  **L1754 CN**: 结束当前词法作用域或复合语句块。
- **L1755 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1755 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1756 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1756 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"pointer '" + pointerExpr->AsFortran() + "'",`.
  **L1757 CN**: 继续一个多行参数列表、初始化器或聚合项：`"pointer '" + pointerExpr->AsFortran() + "'",`。
- **L1758 EN**: Executes a call or declaration centered on `targetExpr->AsFortran`.
  **L1758 CN**: 执行以 `targetExpr->AsFortran` 为核心的调用或声明。
- **L1759 EN**: Continues the surrounding expression or declaration: `said && whichWarning) {`.
  **L1759 CN**: 继续构造周围的表达式或声明：`said && whichWarning) {`。
- **L1760 EN**: Executes a call or declaration centered on `said->set_usageWarning`.
  **L1760 CN**: 执行以 `said->set_usageWarning` 为核心的调用或声明。
- **L1761 EN**: Closes the current lexical scope or compound statement.
  **L1761 CN**: 结束当前词法作用域或复合语句块。
- **L1762 EN**: Closes the current lexical scope or compound statement.
  **L1762 CN**: 结束当前词法作用域或复合语句块。
- **L1763 EN**: Closes the current lexical scope or compound statement.
  **L1763 CN**: 结束当前词法作用域或复合语句块。
- **L1764 EN**: Transitions from the previous branch into an `else if` condition.
  **L1764 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1765 EN**: Continues logic associated with callable symbol `Say`.
  **L1765 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"POINTER= argument '%s' is a procedure pointer but the TARGET= argument '%s' is not a procedure or procedure pointer"_err_en_US,`.
  **L1766 CN**: 继续一个多行参数列表、初始化器或聚合项：`"POINTER= argument '%s' is a procedure pointer but the TARGET= argument '%s' is not a procedure or procedure pointer"_err_en_US,`。
- **L1767 EN**: Executes a call or declaration centered on `pointerExpr->AsFortran`.
  **L1767 CN**: 执行以 `pointerExpr->AsFortran` 为核心的调用或声明。
- **L1768 EN**: Closes the current lexical scope or compound statement.
  **L1768 CN**: 结束当前词法作用域或复合语句块。
- **L1769 EN**: Closes the current lexical scope or compound statement.
  **L1769 CN**: 结束当前词法作用域或复合语句块。
- **L1770 EN**: Transitions from the previous branch into an `else if` condition.
  **L1770 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1771 EN**: Comment explains nearby logic, intent, or metadata: `Object pointer and target`.
  **L1771 CN**: 注释说明附近代码的逻辑、意图或元数据：`Object pointer and target`。
- **L1772 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1772 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1773 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1773 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1774 EN**: Starts a function, method, lambda, or structured scope: `!evaluate::GetLastTarget(symbols)) {`.
  **L1774 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!evaluate::GetLastTarget(symbols)) {`。
- **L1775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::Message *msg{messages.Say(targetArg->sourceLocation(),`.
  **L1775 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::Message *msg{messages.Say(targetArg->sourceLocation(),`。
- **L1776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"TARGET= argument '%s' must have either the POINTER or the TARGET attribute"_err_en_US,`.
  **L1776 CN**: 继续一个多行参数列表、初始化器或聚合项：`"TARGET= argument '%s' must have either the POINTER or the TARGET attribute"_err_en_US,`。

### Lines 1777-1800

````cpp
                    targetExpr->AsFortran())};
                for (SymbolRef ref : symbols) {
                  msg = evaluate::AttachDeclaration(msg, *ref);
                }
              } else if (HasVectorSubscript(*targetExpr) ||
                  ExtractCoarrayRef(*targetExpr)) {
                messages.Say(targetArg->sourceLocation(),
                    "TARGET= argument '%s' may not have a vector subscript or coindexing"_err_en_US,
                    targetExpr->AsFortran());
              }
            }
            if (const auto pointerType{pointerArg->GetType()}) {
              if (const auto targetType{targetArg->GetType()}) {
                ok = pointerType->IsTkCompatibleWith(*targetType) ||
                    targetType->IsTkCompatibleWith(*pointerType);
              }
            }
          } else {
            messages.Say(
                "POINTER= argument '%s' is an object pointer but the TARGET= argument '%s' is not a variable"_err_en_US,
                pointerExpr->AsFortran(), targetExpr->AsFortran());
          }
          if (!IsAssumedRank(*pointerExpr)) {
            if (IsAssumedRank(*targetExpr)) {
````
- **L1777 EN**: Executes a call or declaration centered on `targetExpr->AsFortran`.
  **L1777 CN**: 执行以 `targetExpr->AsFortran` 为核心的调用或声明。
- **L1778 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1778 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1779 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L1779 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L1780 EN**: Closes the current lexical scope or compound statement.
  **L1780 CN**: 结束当前词法作用域或复合语句块。
- **L1781 EN**: Transitions from the previous branch into an `else if` condition.
  **L1781 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1782 EN**: Starts a function, method, lambda, or structured scope: `ExtractCoarrayRef(*targetExpr)) {`.
  **L1782 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ExtractCoarrayRef(*targetExpr)) {`。
- **L1783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(targetArg->sourceLocation(),`.
  **L1783 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(targetArg->sourceLocation(),`。
- **L1784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"TARGET= argument '%s' may not have a vector subscript or coindexing"_err_en_US,`.
  **L1784 CN**: 继续一个多行参数列表、初始化器或聚合项：`"TARGET= argument '%s' may not have a vector subscript or coindexing"_err_en_US,`。
- **L1785 EN**: Executes a call or declaration centered on `targetExpr->AsFortran`.
  **L1785 CN**: 执行以 `targetExpr->AsFortran` 为核心的调用或声明。
- **L1786 EN**: Closes the current lexical scope or compound statement.
  **L1786 CN**: 结束当前词法作用域或复合语句块。
- **L1787 EN**: Closes the current lexical scope or compound statement.
  **L1787 CN**: 结束当前词法作用域或复合语句块。
- **L1788 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1788 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1789 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1789 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1790 EN**: Continues logic associated with callable symbol `IsTkCompatibleWith`.
  **L1790 CN**: 继续与可调用符号 `IsTkCompatibleWith` 相关的逻辑。
- **L1791 EN**: Executes a call or declaration centered on `targetType->IsTkCompatibleWith`.
  **L1791 CN**: 执行以 `targetType->IsTkCompatibleWith` 为核心的调用或声明。
- **L1792 EN**: Closes the current lexical scope or compound statement.
  **L1792 CN**: 结束当前词法作用域或复合语句块。
- **L1793 EN**: Closes the current lexical scope or compound statement.
  **L1793 CN**: 结束当前词法作用域或复合语句块。
- **L1794 EN**: Transitions from the previous branch into the alternative path.
  **L1794 CN**: 从前一个分支过渡到备选路径。
- **L1795 EN**: Continues logic associated with callable symbol `Say`.
  **L1795 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"POINTER= argument '%s' is an object pointer but the TARGET= argument '%s' is not a variable"_err_en_US,`.
  **L1796 CN**: 继续一个多行参数列表、初始化器或聚合项：`"POINTER= argument '%s' is an object pointer but the TARGET= argument '%s' is not a variable"_err_en_US,`。
- **L1797 EN**: Executes a call or declaration centered on `pointerExpr->AsFortran`.
  **L1797 CN**: 执行以 `pointerExpr->AsFortran` 为核心的调用或声明。
- **L1798 EN**: Closes the current lexical scope or compound statement.
  **L1798 CN**: 结束当前词法作用域或复合语句块。
- **L1799 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1799 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1800 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1800 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1801-1824

````cpp
              messages.Say(
                  "TARGET= argument '%s' may not be assumed-rank when POINTER= argument is not"_err_en_US,
                  pointerExpr->AsFortran());
            } else if (pointerExpr->Rank() != targetExpr->Rank()) {
              messages.Say(
                  "POINTER= argument and TARGET= argument have incompatible ranks %d and %d"_err_en_US,
                  pointerExpr->Rank(), targetExpr->Rank());
            }
          }
        }
      }
    }
  } else {
    // No arguments to ASSOCIATED()
    ok = false;
  }
  if (!ok) {
    messages.Say(
        "Arguments of ASSOCIATED() must be a pointer and an optional valid target"_err_en_US);
  }
}

// CO_REDUCE (F'2023 16.9.49)
static void CheckCoReduce(
````
- **L1801 EN**: Continues logic associated with callable symbol `Say`.
  **L1801 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"TARGET= argument '%s' may not be assumed-rank when POINTER= argument is not"_err_en_US,`.
  **L1802 CN**: 继续一个多行参数列表、初始化器或聚合项：`"TARGET= argument '%s' may not be assumed-rank when POINTER= argument is not"_err_en_US,`。
- **L1803 EN**: Executes a call or declaration centered on `pointerExpr->AsFortran`.
  **L1803 CN**: 执行以 `pointerExpr->AsFortran` 为核心的调用或声明。
- **L1804 EN**: Transitions from the previous branch into an `else if` condition.
  **L1804 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1805 EN**: Continues logic associated with callable symbol `Say`.
  **L1805 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"POINTER= argument and TARGET= argument have incompatible ranks %d and %d"_err_en_US,`.
  **L1806 CN**: 继续一个多行参数列表、初始化器或聚合项：`"POINTER= argument and TARGET= argument have incompatible ranks %d and %d"_err_en_US,`。
- **L1807 EN**: Executes a call or declaration centered on `pointerExpr->Rank`.
  **L1807 CN**: 执行以 `pointerExpr->Rank` 为核心的调用或声明。
- **L1808 EN**: Closes the current lexical scope or compound statement.
  **L1808 CN**: 结束当前词法作用域或复合语句块。
- **L1809 EN**: Closes the current lexical scope or compound statement.
  **L1809 CN**: 结束当前词法作用域或复合语句块。
- **L1810 EN**: Closes the current lexical scope or compound statement.
  **L1810 CN**: 结束当前词法作用域或复合语句块。
- **L1811 EN**: Closes the current lexical scope or compound statement.
  **L1811 CN**: 结束当前词法作用域或复合语句块。
- **L1812 EN**: Closes the current lexical scope or compound statement.
  **L1812 CN**: 结束当前词法作用域或复合语句块。
- **L1813 EN**: Transitions from the previous branch into the alternative path.
  **L1813 CN**: 从前一个分支过渡到备选路径。
- **L1814 EN**: Comment explains nearby logic, intent, or metadata: `No arguments to ASSOCIATED()`.
  **L1814 CN**: 注释说明附近代码的逻辑、意图或元数据：`No arguments to ASSOCIATED()`。
- **L1815 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L1815 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L1816 EN**: Closes the current lexical scope or compound statement.
  **L1816 CN**: 结束当前词法作用域或复合语句块。
- **L1817 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1817 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1818 EN**: Continues logic associated with callable symbol `Say`.
  **L1818 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1819 EN**: Executes a call or declaration centered on `ASSOCIATED`.
  **L1819 CN**: 执行以 `ASSOCIATED` 为核心的调用或声明。
- **L1820 EN**: Closes the current lexical scope or compound statement.
  **L1820 CN**: 结束当前词法作用域或复合语句块。
- **L1821 EN**: Closes the current lexical scope or compound statement.
  **L1821 CN**: 结束当前词法作用域或复合语句块。
- **L1822 EN**: Blank line separating nearby declarations or logic blocks.
  **L1822 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1823 EN**: Comment explains nearby logic, intent, or metadata: `CO_REDUCE (F'2023 16.9.49)`.
  **L1823 CN**: 注释说明附近代码的逻辑、意图或元数据：`CO_REDUCE (F'2023 16.9.49)`。
- **L1824 EN**: Continues logic associated with callable symbol `CheckCoReduce`.
  **L1824 CN**: 继续与可调用符号 `CheckCoReduce` 相关的逻辑。

### Lines 1825-1848

````cpp
    evaluate::ActualArguments &arguments, evaluate::FoldingContext &context) {
  parser::ContextualMessages &messages{context.messages()};
  evaluate::CheckForCoindexedObject(
      context.messages(), arguments[0], "co_reduce", "a");
  evaluate::CheckForCoindexedObject(
      context.messages(), arguments[2], "co_reduce", "stat");
  evaluate::CheckForCoindexedObject(
      context.messages(), arguments[3], "co_reduce", "errmsg");

  std::optional<evaluate::DynamicType> aType;
  if (const auto &a{arguments[0]}) {
    aType = a->GetType();
  }
  std::optional<characteristics::Procedure> procChars;
  if (const auto &operation{arguments[1]}) {
    if (const auto *expr{operation->UnwrapExpr()}) {
      if (const auto *designator{
              std::get_if<evaluate::ProcedureDesignator>(&expr->u)}) {
        procChars = characteristics::Procedure::Characterize(
            *designator, context, /*emitError=*/true);
      } else if (const auto *ref{
                     std::get_if<evaluate::ProcedureRef>(&expr->u)}) {
        procChars = characteristics::Procedure::Characterize(*ref, context);
      }
````
- **L1825 EN**: Continues the surrounding expression or declaration: `evaluate::ActualArguments &arguments, evaluate::FoldingContext &context) {`.
  **L1825 CN**: 继续构造周围的表达式或声明：`evaluate::ActualArguments &arguments, evaluate::FoldingContext &context) {`。
- **L1826 EN**: Executes a call or declaration centered on `&messages{context.messages`.
  **L1826 CN**: 执行以 `&messages{context.messages` 为核心的调用或声明。
- **L1827 EN**: Continues logic associated with callable symbol `CheckForCoindexedObject`.
  **L1827 CN**: 继续与可调用符号 `CheckForCoindexedObject` 相关的逻辑。
- **L1828 EN**: Executes a call or declaration centered on `context.messages`.
  **L1828 CN**: 执行以 `context.messages` 为核心的调用或声明。
- **L1829 EN**: Continues logic associated with callable symbol `CheckForCoindexedObject`.
  **L1829 CN**: 继续与可调用符号 `CheckForCoindexedObject` 相关的逻辑。
- **L1830 EN**: Executes a call or declaration centered on `context.messages`.
  **L1830 CN**: 执行以 `context.messages` 为核心的调用或声明。
- **L1831 EN**: Continues logic associated with callable symbol `CheckForCoindexedObject`.
  **L1831 CN**: 继续与可调用符号 `CheckForCoindexedObject` 相关的逻辑。
- **L1832 EN**: Executes a call or declaration centered on `context.messages`.
  **L1832 CN**: 执行以 `context.messages` 为核心的调用或声明。
- **L1833 EN**: Blank line separating nearby declarations or logic blocks.
  **L1833 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1834 EN**: Executes a standalone statement or declaration: `std::optional<evaluate::DynamicType> aType;`.
  **L1834 CN**: 执行一条独立语句或声明：`std::optional<evaluate::DynamicType> aType;`。
- **L1835 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1835 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1836 EN**: Executes a call or declaration centered on `a->GetType`.
  **L1836 CN**: 执行以 `a->GetType` 为核心的调用或声明。
- **L1837 EN**: Closes the current lexical scope or compound statement.
  **L1837 CN**: 结束当前词法作用域或复合语句块。
- **L1838 EN**: Executes a standalone statement or declaration: `std::optional<characteristics::Procedure> procChars;`.
  **L1838 CN**: 执行一条独立语句或声明：`std::optional<characteristics::Procedure> procChars;`。
- **L1839 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1839 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1840 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1840 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1841 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1841 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1842 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<evaluate::ProcedureDesignator>(&expr->u)}) {`.
  **L1842 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<evaluate::ProcedureDesignator>(&expr->u)}) {`。
- **L1843 EN**: Continues logic associated with callable symbol `Characterize`.
  **L1843 CN**: 继续与可调用符号 `Characterize` 相关的逻辑。
- **L1844 EN**: Comment explains nearby logic, intent, or metadata: `designator, context, /*emitError=*/true);`.
  **L1844 CN**: 注释说明附近代码的逻辑、意图或元数据：`designator, context, /*emitError=*/true);`。
- **L1845 EN**: Transitions from the previous branch into an `else if` condition.
  **L1845 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1846 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<evaluate::ProcedureRef>(&expr->u)}) {`.
  **L1846 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<evaluate::ProcedureRef>(&expr->u)}) {`。
- **L1847 EN**: Executes a call or declaration centered on `characteristics::Procedure::Characterize`.
  **L1847 CN**: 执行以 `characteristics::Procedure::Characterize` 为核心的调用或声明。
- **L1848 EN**: Closes the current lexical scope or compound statement.
  **L1848 CN**: 结束当前词法作用域或复合语句块。

### Lines 1849-1872

````cpp
    }
  }

  static constexpr characteristics::DummyDataObject::Attrs notAllowedArgAttrs{
      characteristics::DummyDataObject::Attr::Optional,
      characteristics::DummyDataObject::Attr::Allocatable,
      characteristics::DummyDataObject::Attr::Pointer,
  };
  static constexpr characteristics::FunctionResult::Attrs
      notAllowedFuncResAttrs{
          characteristics::FunctionResult::Attr::Allocatable,
          characteristics::FunctionResult::Attr::Pointer,
      };
  const characteristics::TypeAndShape *result{
      procChars && procChars->functionResult
          ? procChars->functionResult->GetTypeAndShape()
          : nullptr};
  if (!procChars || !procChars->IsPure() ||
      procChars->dummyArguments.size() != 2 || !procChars->functionResult) {
    messages.Say(
        "OPERATION= argument of CO_REDUCE() must be a pure function of two data arguments"_err_en_US);
  } else if (procChars->attrs.test(characteristics::Procedure::Attr::BindC)) {
    messages.Say(
        "A BIND(C) OPERATION= argument of CO_REDUCE() is not supported"_err_en_US);
````
- **L1849 EN**: Closes the current lexical scope or compound statement.
  **L1849 CN**: 结束当前词法作用域或复合语句块。
- **L1850 EN**: Closes the current lexical scope or compound statement.
  **L1850 CN**: 结束当前词法作用域或复合语句块。
- **L1851 EN**: Blank line separating nearby declarations or logic blocks.
  **L1851 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1852 EN**: Continues the surrounding expression or declaration: `static constexpr characteristics::DummyDataObject::Attrs notAllowedArgAttrs{`.
  **L1852 CN**: 继续构造周围的表达式或声明：`static constexpr characteristics::DummyDataObject::Attrs notAllowedArgAttrs{`。
- **L1853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::DummyDataObject::Attr::Optional,`.
  **L1853 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::DummyDataObject::Attr::Optional,`。
- **L1854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::DummyDataObject::Attr::Allocatable,`.
  **L1854 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::DummyDataObject::Attr::Allocatable,`。
- **L1855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::DummyDataObject::Attr::Pointer,`.
  **L1855 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::DummyDataObject::Attr::Pointer,`。
- **L1856 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1856 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1857 EN**: Continues the surrounding expression or declaration: `static constexpr characteristics::FunctionResult::Attrs`.
  **L1857 CN**: 继续构造周围的表达式或声明：`static constexpr characteristics::FunctionResult::Attrs`。
- **L1858 EN**: Continues the surrounding expression or declaration: `notAllowedFuncResAttrs{`.
  **L1858 CN**: 继续构造周围的表达式或声明：`notAllowedFuncResAttrs{`。
- **L1859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::FunctionResult::Attr::Allocatable,`.
  **L1859 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::FunctionResult::Attr::Allocatable,`。
- **L1860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::FunctionResult::Attr::Pointer,`.
  **L1860 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::FunctionResult::Attr::Pointer,`。
- **L1861 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1861 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1862 EN**: Continues the surrounding expression or declaration: `const characteristics::TypeAndShape *result{`.
  **L1862 CN**: 继续构造周围的表达式或声明：`const characteristics::TypeAndShape *result{`。
- **L1863 EN**: Continues the surrounding expression or declaration: `procChars && procChars->functionResult`.
  **L1863 CN**: 继续构造周围的表达式或声明：`procChars && procChars->functionResult`。
- **L1864 EN**: Continues logic associated with callable symbol `GetTypeAndShape`.
  **L1864 CN**: 继续与可调用符号 `GetTypeAndShape` 相关的逻辑。
- **L1865 EN**: Executes a standalone statement or declaration: `: nullptr};`.
  **L1865 CN**: 执行一条独立语句或声明：`: nullptr};`。
- **L1866 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1866 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1867 EN**: Starts a function, method, lambda, or structured scope: `procChars->dummyArguments.size() != 2 || !procChars->functionResult) {`.
  **L1867 CN**: 开始一个函数、方法、lambda 或结构化作用域：`procChars->dummyArguments.size() != 2 || !procChars->functionResult) {`。
- **L1868 EN**: Continues logic associated with callable symbol `Say`.
  **L1868 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1869 EN**: Executes a call or declaration centered on `CO_REDUCE`.
  **L1869 CN**: 执行以 `CO_REDUCE` 为核心的调用或声明。
- **L1870 EN**: Transitions from the previous branch into an `else if` condition.
  **L1870 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1871 EN**: Continues logic associated with callable symbol `Say`.
  **L1871 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1872 EN**: Executes a call or declaration centered on `BIND`.
  **L1872 CN**: 执行以 `BIND` 为核心的调用或声明。

### Lines 1873-1896

````cpp
  } else if (!result || result->Rank() != 0) {
    messages.Say(
        "OPERATION= argument of CO_REDUCE() must be a scalar function"_err_en_US);
  } else if (result->type().IsPolymorphic() ||
      (aType && !aType->IsTkLenCompatibleWith(result->type()))) {
    messages.Say(
        "OPERATION= argument of CO_REDUCE() must have the same type as A="_err_en_US);
  } else if (((procChars->functionResult->attrs & notAllowedFuncResAttrs) !=
                 characteristics::FunctionResult::Attrs{}) ||
      procChars->functionResult->GetTypeAndShape()->type().IsPolymorphic()) {
    messages.Say(
        "Result of OPERATION= procedure of CO_REDUCE() must be scalar and neither allocatable, pointer, nor polymorphic"_err_en_US);
  } else {
    const characteristics::DummyDataObject *data[2]{};
    for (int j{0}; j < 2; ++j) {
      const auto &dummy{procChars->dummyArguments.at(j)};
      data[j] = std::get_if<characteristics::DummyDataObject>(&dummy.u);
    }
    if (!data[0] || !data[1]) {
      messages.Say(
          "OPERATION= argument of CO_REDUCE() may not have dummy procedure arguments"_err_en_US);
    } else {
      for (int j{0}; j < 2; ++j) {
        if (((data[j]->attrs & notAllowedArgAttrs) !=
````
- **L1873 EN**: Transitions from the previous branch into an `else if` condition.
  **L1873 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1874 EN**: Continues logic associated with callable symbol `Say`.
  **L1874 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1875 EN**: Executes a call or declaration centered on `CO_REDUCE`.
  **L1875 CN**: 执行以 `CO_REDUCE` 为核心的调用或声明。
- **L1876 EN**: Transitions from the previous branch into an `else if` condition.
  **L1876 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1877 EN**: Starts a function, method, lambda, or structured scope: `(aType && !aType->IsTkLenCompatibleWith(result->type()))) {`.
  **L1877 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(aType && !aType->IsTkLenCompatibleWith(result->type()))) {`。
- **L1878 EN**: Continues logic associated with callable symbol `Say`.
  **L1878 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1879 EN**: Executes a call or declaration centered on `CO_REDUCE`.
  **L1879 CN**: 执行以 `CO_REDUCE` 为核心的调用或声明。
- **L1880 EN**: Transitions from the previous branch into an `else if` condition.
  **L1880 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1881 EN**: Continues the surrounding expression or declaration: `characteristics::FunctionResult::Attrs{}) ||`.
  **L1881 CN**: 继续构造周围的表达式或声明：`characteristics::FunctionResult::Attrs{}) ||`。
- **L1882 EN**: Starts a function, method, lambda, or structured scope: `procChars->functionResult->GetTypeAndShape()->type().IsPolymorphic()) {`.
  **L1882 CN**: 开始一个函数、方法、lambda 或结构化作用域：`procChars->functionResult->GetTypeAndShape()->type().IsPolymorphic()) {`。
- **L1883 EN**: Continues logic associated with callable symbol `Say`.
  **L1883 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1884 EN**: Executes a call or declaration centered on `CO_REDUCE`.
  **L1884 CN**: 执行以 `CO_REDUCE` 为核心的调用或声明。
- **L1885 EN**: Transitions from the previous branch into the alternative path.
  **L1885 CN**: 从前一个分支过渡到备选路径。
- **L1886 EN**: Executes a standalone statement or declaration: `const characteristics::DummyDataObject *data[2]{};`.
  **L1886 CN**: 执行一条独立语句或声明：`const characteristics::DummyDataObject *data[2]{};`。
- **L1887 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1887 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1888 EN**: Executes a call or declaration centered on `&dummy{procChars->dummyArguments.at`.
  **L1888 CN**: 执行以 `&dummy{procChars->dummyArguments.at` 为核心的调用或声明。
- **L1889 EN**: Executes a call or declaration centered on `std::get_if<characteristics::DummyDataObject>`.
  **L1889 CN**: 执行以 `std::get_if<characteristics::DummyDataObject>` 为核心的调用或声明。
- **L1890 EN**: Closes the current lexical scope or compound statement.
  **L1890 CN**: 结束当前词法作用域或复合语句块。
- **L1891 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1891 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1892 EN**: Continues logic associated with callable symbol `Say`.
  **L1892 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1893 EN**: Executes a call or declaration centered on `CO_REDUCE`.
  **L1893 CN**: 执行以 `CO_REDUCE` 为核心的调用或声明。
- **L1894 EN**: Transitions from the previous branch into the alternative path.
  **L1894 CN**: 从前一个分支过渡到备选路径。
- **L1895 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1895 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1896 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1896 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1897-1920

````cpp
                characteristics::DummyDataObject::Attrs{}) ||
            data[j]->type.Rank() != 0 || data[j]->type.type().IsPolymorphic() ||
            (aType && !data[j]->type.type().IsTkCompatibleWith(*aType))) {
          messages.Say(
              "Arguments of OPERATION= procedure of CO_REDUCE() must be both scalar of the same type as A=, and neither allocatable, pointer, polymorphic, nor optional"_err_en_US);
          break;
        }
      }
      static constexpr characteristics::DummyDataObject::Attrs attrs{
          characteristics::DummyDataObject::Attr::Asynchronous,
          characteristics::DummyDataObject::Attr::Target,
          characteristics::DummyDataObject::Attr::Value,
      };
      if ((data[0]->attrs & attrs) != (data[1]->attrs & attrs)) {
        messages.Say(
            "If either argument of the OPERATION= procedure of CO_REDUCE() has the ASYNCHRONOUS, TARGET, or VALUE attribute, both must have that attribute"_err_en_US);
      }
    }
  }
}

// DATE_AND_TIME (F'2023 16.9.69)
static void CheckDate_And_Time(evaluate::ActualArguments &arguments,
    evaluate::FoldingContext &foldingContext) {
````
- **L1897 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject::Attrs{}) ||`.
  **L1897 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject::Attrs{}) ||`。
- **L1898 EN**: Continues logic associated with callable symbol `Rank`.
  **L1898 CN**: 继续与可调用符号 `Rank` 相关的逻辑。
- **L1899 EN**: Starts a function, method, lambda, or structured scope: `(aType && !data[j]->type.type().IsTkCompatibleWith(*aType))) {`.
  **L1899 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(aType && !data[j]->type.type().IsTkCompatibleWith(*aType))) {`。
- **L1900 EN**: Continues logic associated with callable symbol `Say`.
  **L1900 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1901 EN**: Executes a call or declaration centered on `CO_REDUCE`.
  **L1901 CN**: 执行以 `CO_REDUCE` 为核心的调用或声明。
- **L1902 EN**: Exits the nearest loop or switch statement.
  **L1902 CN**: 退出最近的循环或 switch 语句。
- **L1903 EN**: Closes the current lexical scope or compound statement.
  **L1903 CN**: 结束当前词法作用域或复合语句块。
- **L1904 EN**: Closes the current lexical scope or compound statement.
  **L1904 CN**: 结束当前词法作用域或复合语句块。
- **L1905 EN**: Continues the surrounding expression or declaration: `static constexpr characteristics::DummyDataObject::Attrs attrs{`.
  **L1905 CN**: 继续构造周围的表达式或声明：`static constexpr characteristics::DummyDataObject::Attrs attrs{`。
- **L1906 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::DummyDataObject::Attr::Asynchronous,`.
  **L1906 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::DummyDataObject::Attr::Asynchronous,`。
- **L1907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::DummyDataObject::Attr::Target,`.
  **L1907 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::DummyDataObject::Attr::Target,`。
- **L1908 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::DummyDataObject::Attr::Value,`.
  **L1908 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::DummyDataObject::Attr::Value,`。
- **L1909 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1909 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1910 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1910 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1911 EN**: Continues logic associated with callable symbol `Say`.
  **L1911 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1912 EN**: Executes a call or declaration centered on `CO_REDUCE`.
  **L1912 CN**: 执行以 `CO_REDUCE` 为核心的调用或声明。
- **L1913 EN**: Closes the current lexical scope or compound statement.
  **L1913 CN**: 结束当前词法作用域或复合语句块。
- **L1914 EN**: Closes the current lexical scope or compound statement.
  **L1914 CN**: 结束当前词法作用域或复合语句块。
- **L1915 EN**: Closes the current lexical scope or compound statement.
  **L1915 CN**: 结束当前词法作用域或复合语句块。
- **L1916 EN**: Closes the current lexical scope or compound statement.
  **L1916 CN**: 结束当前词法作用域或复合语句块。
- **L1917 EN**: Blank line separating nearby declarations or logic blocks.
  **L1917 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1918 EN**: Comment explains nearby logic, intent, or metadata: `DATE_AND_TIME (F'2023 16.9.69)`.
  **L1918 CN**: 注释说明附近代码的逻辑、意图或元数据：`DATE_AND_TIME (F'2023 16.9.69)`。
- **L1919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckDate_And_Time(evaluate::ActualArguments &arguments,`.
  **L1919 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckDate_And_Time(evaluate::ActualArguments &arguments,`。
- **L1920 EN**: Continues the surrounding expression or declaration: `evaluate::FoldingContext &foldingContext) {`.
  **L1920 CN**: 继续构造周围的表达式或声明：`evaluate::FoldingContext &foldingContext) {`。

### Lines 1921-1944

````cpp
  if (arguments.size() >= 4 && arguments[3]) {
    if (const auto valuesShape{
            evaluate::GetShape(arguments[3]->UnwrapExpr())}) {
      if (auto extents{
              evaluate::AsConstantExtents(foldingContext, *valuesShape)}) {
        if (!extents->empty() && extents->at(0) < 8) {
          auto &messages{foldingContext.messages()};
          messages.Say(arguments[3]->sourceLocation().value_or(messages.at()),
              "VALUES= argument to DATE_AND_TIME must have at least 8 elements"_err_en_US);
        }
      }
    }
  }
}

// EVENT_QUERY (F'2023 16.9.82)
static void CheckEvent_Query(evaluate::ActualArguments &arguments,
    evaluate::FoldingContext &foldingContext) {
  if (arguments.size() > 0 && arguments[0] &&
      ExtractCoarrayRef(*arguments[0]).has_value()) {
    foldingContext.messages().Say(arguments[0]->sourceLocation(),
        "EVENT= argument to EVENT_QUERY must not be coindexed"_err_en_US);
  }
  if (arguments.size() > 1 && arguments[1]) {
````
- **L1921 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1921 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1922 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1922 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1923 EN**: Starts a function, method, lambda, or structured scope: `evaluate::GetShape(arguments[3]->UnwrapExpr())}) {`.
  **L1923 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::GetShape(arguments[3]->UnwrapExpr())}) {`。
- **L1924 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1924 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1925 EN**: Starts a function, method, lambda, or structured scope: `evaluate::AsConstantExtents(foldingContext, *valuesShape)}) {`.
  **L1925 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::AsConstantExtents(foldingContext, *valuesShape)}) {`。
- **L1926 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1926 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1927 EN**: Executes a call or declaration centered on `&messages{foldingContext.messages`.
  **L1927 CN**: 执行以 `&messages{foldingContext.messages` 为核心的调用或声明。
- **L1928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arguments[3]->sourceLocation().value_or(messages.at()),`.
  **L1928 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arguments[3]->sourceLocation().value_or(messages.at()),`。
- **L1929 EN**: Executes a standalone statement or declaration: `"VALUES= argument to DATE_AND_TIME must have at least 8 elements"_err_en_US);`.
  **L1929 CN**: 执行一条独立语句或声明：`"VALUES= argument to DATE_AND_TIME must have at least 8 elements"_err_en_US);`。
- **L1930 EN**: Closes the current lexical scope or compound statement.
  **L1930 CN**: 结束当前词法作用域或复合语句块。
- **L1931 EN**: Closes the current lexical scope or compound statement.
  **L1931 CN**: 结束当前词法作用域或复合语句块。
- **L1932 EN**: Closes the current lexical scope or compound statement.
  **L1932 CN**: 结束当前词法作用域或复合语句块。
- **L1933 EN**: Closes the current lexical scope or compound statement.
  **L1933 CN**: 结束当前词法作用域或复合语句块。
- **L1934 EN**: Closes the current lexical scope or compound statement.
  **L1934 CN**: 结束当前词法作用域或复合语句块。
- **L1935 EN**: Blank line separating nearby declarations or logic blocks.
  **L1935 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1936 EN**: Comment explains nearby logic, intent, or metadata: `EVENT_QUERY (F'2023 16.9.82)`.
  **L1936 CN**: 注释说明附近代码的逻辑、意图或元数据：`EVENT_QUERY (F'2023 16.9.82)`。
- **L1937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckEvent_Query(evaluate::ActualArguments &arguments,`.
  **L1937 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckEvent_Query(evaluate::ActualArguments &arguments,`。
- **L1938 EN**: Continues the surrounding expression or declaration: `evaluate::FoldingContext &foldingContext) {`.
  **L1938 CN**: 继续构造周围的表达式或声明：`evaluate::FoldingContext &foldingContext) {`。
- **L1939 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1939 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1940 EN**: Starts a function, method, lambda, or structured scope: `ExtractCoarrayRef(*arguments[0]).has_value()) {`.
  **L1940 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ExtractCoarrayRef(*arguments[0]).has_value()) {`。
- **L1941 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.messages().Say(arguments[0]->sourceLocation(),`.
  **L1941 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.messages().Say(arguments[0]->sourceLocation(),`。
- **L1942 EN**: Executes a standalone statement or declaration: `"EVENT= argument to EVENT_QUERY must not be coindexed"_err_en_US);`.
  **L1942 CN**: 执行一条独立语句或声明：`"EVENT= argument to EVENT_QUERY must not be coindexed"_err_en_US);`。
- **L1943 EN**: Closes the current lexical scope or compound statement.
  **L1943 CN**: 结束当前词法作用域或复合语句块。
- **L1944 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1944 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1945-1968

````cpp
    if (auto dyType{arguments[1]->GetType()}) {
      int defaultInt{
          foldingContext.defaults().GetDefaultKind(TypeCategory::Integer)};
      if (dyType->category() == TypeCategory::Integer &&
          dyType->kind() < defaultInt) {
        foldingContext.messages().Say(arguments[1]->sourceLocation(),
            "COUNT= argument to EVENT_QUERY must be an integer with kind >= %d"_err_en_US,
            defaultInt);
      }
    }
  }
  if (arguments.size() > 2 && arguments[2]) {
    if (auto dyType{arguments[2]->GetType()}) {
      if (dyType->category() == TypeCategory::Integer && dyType->kind() < 2) {
        foldingContext.messages().Say(arguments[2]->sourceLocation(),
            "STAT= argument to EVENT_QUERY must be an integer with kind >= 2 when present"_err_en_US);
      }
    }
  }
}

// IMAGE_INDEX (F'2023 16.9.107)
static void CheckImage_Index(evaluate::ActualArguments &arguments,
    parser::ContextualMessages &messages) {
````
- **L1945 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1945 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1946 EN**: Continues the surrounding expression or declaration: `int defaultInt{`.
  **L1946 CN**: 继续构造周围的表达式或声明：`int defaultInt{`。
- **L1947 EN**: Executes a call or declaration centered on `foldingContext.defaults`.
  **L1947 CN**: 执行以 `foldingContext.defaults` 为核心的调用或声明。
- **L1948 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1948 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1949 EN**: Starts a function, method, lambda, or structured scope: `dyType->kind() < defaultInt) {`.
  **L1949 CN**: 开始一个函数、方法、lambda 或结构化作用域：`dyType->kind() < defaultInt) {`。
- **L1950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.messages().Say(arguments[1]->sourceLocation(),`.
  **L1950 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.messages().Say(arguments[1]->sourceLocation(),`。
- **L1951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"COUNT= argument to EVENT_QUERY must be an integer with kind >= %d"_err_en_US,`.
  **L1951 CN**: 继续一个多行参数列表、初始化器或聚合项：`"COUNT= argument to EVENT_QUERY must be an integer with kind >= %d"_err_en_US,`。
- **L1952 EN**: Executes a standalone statement or declaration: `defaultInt);`.
  **L1952 CN**: 执行一条独立语句或声明：`defaultInt);`。
- **L1953 EN**: Closes the current lexical scope or compound statement.
  **L1953 CN**: 结束当前词法作用域或复合语句块。
- **L1954 EN**: Closes the current lexical scope or compound statement.
  **L1954 CN**: 结束当前词法作用域或复合语句块。
- **L1955 EN**: Closes the current lexical scope or compound statement.
  **L1955 CN**: 结束当前词法作用域或复合语句块。
- **L1956 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1956 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1957 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1957 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1958 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1958 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1959 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.messages().Say(arguments[2]->sourceLocation(),`.
  **L1959 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.messages().Say(arguments[2]->sourceLocation(),`。
- **L1960 EN**: Executes a standalone statement or declaration: `"STAT= argument to EVENT_QUERY must be an integer with kind >= 2 when present"_err_en_US);`.
  **L1960 CN**: 执行一条独立语句或声明：`"STAT= argument to EVENT_QUERY must be an integer with kind >= 2 when present"_err_en_US);`。
- **L1961 EN**: Closes the current lexical scope or compound statement.
  **L1961 CN**: 结束当前词法作用域或复合语句块。
- **L1962 EN**: Closes the current lexical scope or compound statement.
  **L1962 CN**: 结束当前词法作用域或复合语句块。
- **L1963 EN**: Closes the current lexical scope or compound statement.
  **L1963 CN**: 结束当前词法作用域或复合语句块。
- **L1964 EN**: Closes the current lexical scope or compound statement.
  **L1964 CN**: 结束当前词法作用域或复合语句块。
- **L1965 EN**: Blank line separating nearby declarations or logic blocks.
  **L1965 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1966 EN**: Comment explains nearby logic, intent, or metadata: `IMAGE_INDEX (F'2023 16.9.107)`.
  **L1966 CN**: 注释说明附近代码的逻辑、意图或元数据：`IMAGE_INDEX (F'2023 16.9.107)`。
- **L1967 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckImage_Index(evaluate::ActualArguments &arguments,`.
  **L1967 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckImage_Index(evaluate::ActualArguments &arguments,`。
- **L1968 EN**: Continues the surrounding expression or declaration: `parser::ContextualMessages &messages) {`.
  **L1968 CN**: 继续构造周围的表达式或声明：`parser::ContextualMessages &messages) {`。

### Lines 1969-1992

````cpp
  if (arguments[1] && arguments[0]) {
    if (const auto subArrShape{
            evaluate::GetShape(arguments[1]->UnwrapExpr())}) {
      if (const auto *coarrayArgSymbol{UnwrapWholeSymbolOrComponentDataRef(
              arguments[0]->UnwrapExpr())}) {
        auto coarrayArgCorank{coarrayArgSymbol->Corank()};
        if (auto subArrSize{evaluate::ToInt64(*subArrShape->front())}) {
          if (subArrSize != coarrayArgCorank) {
            messages.Say(arguments[1]->sourceLocation(),
                "The size of 'SUB=' (%jd) for intrinsic 'image_index' must be equal to the corank of 'COARRAY=' (%d)"_err_en_US,
                static_cast<std::int64_t>(*subArrSize), coarrayArgCorank);
          }
        }
      }
    }
  }
}

// Ensure that any optional argument that might be absent at run time
// does not require data conversion.
static void CheckMaxMin(const characteristics::Procedure &proc,
    evaluate::ActualArguments &arguments,
    parser::ContextualMessages &messages) {
  if (proc.functionResult) {
````
- **L1969 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1969 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1970 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1970 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1971 EN**: Starts a function, method, lambda, or structured scope: `evaluate::GetShape(arguments[1]->UnwrapExpr())}) {`.
  **L1971 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::GetShape(arguments[1]->UnwrapExpr())}) {`。
- **L1972 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1972 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1973 EN**: Starts a function, method, lambda, or structured scope: `arguments[0]->UnwrapExpr())}) {`.
  **L1973 CN**: 开始一个函数、方法、lambda 或结构化作用域：`arguments[0]->UnwrapExpr())}) {`。
- **L1974 EN**: Executes a call or declaration centered on `coarrayArgCorank{coarrayArgSymbol->Corank`.
  **L1974 CN**: 执行以 `coarrayArgCorank{coarrayArgSymbol->Corank` 为核心的调用或声明。
- **L1975 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1975 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1976 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1976 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arguments[1]->sourceLocation(),`.
  **L1977 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arguments[1]->sourceLocation(),`。
- **L1978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The size of 'SUB=' (%jd) for intrinsic 'image_index' must be equal to the corank of 'COARRAY=' (%d)"_err_en_US,`.
  **L1978 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The size of 'SUB=' (%jd) for intrinsic 'image_index' must be equal to the corank of 'COARRAY=' (%d)"_err_en_US,`。
- **L1979 EN**: Executes a call or declaration centered on `static_cast<std::int64_t>`.
  **L1979 CN**: 执行以 `static_cast<std::int64_t>` 为核心的调用或声明。
- **L1980 EN**: Closes the current lexical scope or compound statement.
  **L1980 CN**: 结束当前词法作用域或复合语句块。
- **L1981 EN**: Closes the current lexical scope or compound statement.
  **L1981 CN**: 结束当前词法作用域或复合语句块。
- **L1982 EN**: Closes the current lexical scope or compound statement.
  **L1982 CN**: 结束当前词法作用域或复合语句块。
- **L1983 EN**: Closes the current lexical scope or compound statement.
  **L1983 CN**: 结束当前词法作用域或复合语句块。
- **L1984 EN**: Closes the current lexical scope or compound statement.
  **L1984 CN**: 结束当前词法作用域或复合语句块。
- **L1985 EN**: Closes the current lexical scope or compound statement.
  **L1985 CN**: 结束当前词法作用域或复合语句块。
- **L1986 EN**: Blank line separating nearby declarations or logic blocks.
  **L1986 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1987 EN**: Comment explains nearby logic, intent, or metadata: `Ensure that any optional argument that might be absent at run time`.
  **L1987 CN**: 注释说明附近代码的逻辑、意图或元数据：`Ensure that any optional argument that might be absent at run time`。
- **L1988 EN**: Comment explains nearby logic, intent, or metadata: `does not require data conversion.`.
  **L1988 CN**: 注释说明附近代码的逻辑、意图或元数据：`does not require data conversion.`。
- **L1989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckMaxMin(const characteristics::Procedure &proc,`.
  **L1989 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckMaxMin(const characteristics::Procedure &proc,`。
- **L1990 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::ActualArguments &arguments,`.
  **L1990 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::ActualArguments &arguments,`。
- **L1991 EN**: Continues the surrounding expression or declaration: `parser::ContextualMessages &messages) {`.
  **L1991 CN**: 继续构造周围的表达式或声明：`parser::ContextualMessages &messages) {`。
- **L1992 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1992 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1993-2016

````cpp
    if (const auto *typeAndShape{proc.functionResult->GetTypeAndShape()}) {
      for (std::size_t j{2}; j < arguments.size(); ++j) {
        if (arguments[j]) {
          if (const auto *expr{arguments[j]->UnwrapExpr()};
              expr && evaluate::MayBePassedAsAbsentOptional(*expr)) {
            if (auto thisType{expr->GetType()}) {
              if (thisType->category() == TypeCategory::Character &&
                  typeAndShape->type().category() == TypeCategory::Character &&
                  thisType->kind() == typeAndShape->type().kind()) {
                // don't care about lengths
              } else if (*thisType != typeAndShape->type()) {
                messages.Say(arguments[j]->sourceLocation(),
                    "An actual argument to MAX/MIN requiring data conversion may not be OPTIONAL, POINTER, or ALLOCATABLE"_err_en_US);
              }
            }
          }
        }
      }
    }
  }
}

static void CheckFree(evaluate::ActualArguments &arguments,
    parser::ContextualMessages &messages) {
````
- **L1993 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1993 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1994 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1994 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1995 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1995 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1996 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1996 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1997 EN**: Starts a function, method, lambda, or structured scope: `expr && evaluate::MayBePassedAsAbsentOptional(*expr)) {`.
  **L1997 CN**: 开始一个函数、方法、lambda 或结构化作用域：`expr && evaluate::MayBePassedAsAbsentOptional(*expr)) {`。
- **L1998 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1998 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1999 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1999 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2000 EN**: Continues logic associated with callable symbol `type`.
  **L2000 CN**: 继续与可调用符号 `type` 相关的逻辑。
- **L2001 EN**: Starts a function, method, lambda, or structured scope: `thisType->kind() == typeAndShape->type().kind()) {`.
  **L2001 CN**: 开始一个函数、方法、lambda 或结构化作用域：`thisType->kind() == typeAndShape->type().kind()) {`。
- **L2002 EN**: Comment explains nearby logic, intent, or metadata: `don't care about lengths`.
  **L2002 CN**: 注释说明附近代码的逻辑、意图或元数据：`don't care about lengths`。
- **L2003 EN**: Transitions from the previous branch into an `else if` condition.
  **L2003 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arguments[j]->sourceLocation(),`.
  **L2004 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arguments[j]->sourceLocation(),`。
- **L2005 EN**: Executes a standalone statement or declaration: `"An actual argument to MAX/MIN requiring data conversion may not be OPTIONAL, POINTER, or ALLOCATABLE"_err_en_US);`.
  **L2005 CN**: 执行一条独立语句或声明：`"An actual argument to MAX/MIN requiring data conversion may not be OPTIONAL, POINTER, or ALLOCATABLE"_err_en_US);`。
- **L2006 EN**: Closes the current lexical scope or compound statement.
  **L2006 CN**: 结束当前词法作用域或复合语句块。
- **L2007 EN**: Closes the current lexical scope or compound statement.
  **L2007 CN**: 结束当前词法作用域或复合语句块。
- **L2008 EN**: Closes the current lexical scope or compound statement.
  **L2008 CN**: 结束当前词法作用域或复合语句块。
- **L2009 EN**: Closes the current lexical scope or compound statement.
  **L2009 CN**: 结束当前词法作用域或复合语句块。
- **L2010 EN**: Closes the current lexical scope or compound statement.
  **L2010 CN**: 结束当前词法作用域或复合语句块。
- **L2011 EN**: Closes the current lexical scope or compound statement.
  **L2011 CN**: 结束当前词法作用域或复合语句块。
- **L2012 EN**: Closes the current lexical scope or compound statement.
  **L2012 CN**: 结束当前词法作用域或复合语句块。
- **L2013 EN**: Closes the current lexical scope or compound statement.
  **L2013 CN**: 结束当前词法作用域或复合语句块。
- **L2014 EN**: Blank line separating nearby declarations or logic blocks.
  **L2014 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2015 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckFree(evaluate::ActualArguments &arguments,`.
  **L2015 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckFree(evaluate::ActualArguments &arguments,`。
- **L2016 EN**: Continues the surrounding expression or declaration: `parser::ContextualMessages &messages) {`.
  **L2016 CN**: 继续构造周围的表达式或声明：`parser::ContextualMessages &messages) {`。

### Lines 2017-2040

````cpp
  if (arguments.size() != 1) {
    messages.Say("FREE expects a single argument"_err_en_US);
  }
  auto arg = arguments[0];
  if (const Symbol * symbol{evaluate::UnwrapWholeSymbolDataRef(arg)};
      !symbol || !symbol->test(Symbol::Flag::CrayPointer)) {
    messages.Say("FREE should only be used with Cray pointers"_warn_en_US);
  }
}

// MOVE_ALLOC (F'2023 16.9.147)
static void CheckMove_Alloc(evaluate::ActualArguments &arguments,
    parser::ContextualMessages &messages) {
  if (arguments.size() >= 1) {
    evaluate::CheckForCoindexedObject(
        messages, arguments[0], "move_alloc", "from");
  }
  if (arguments.size() >= 2) {
    evaluate::CheckForCoindexedObject(
        messages, arguments[1], "move_alloc", "to");
    int fromCR{GetCorank(arguments[0])};
    int toCR{GetCorank(arguments[1])};
    if (fromCR != toCR) {
      messages.Say(*arguments[0]->sourceLocation(),
````
- **L2017 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2017 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2018 EN**: Executes a call or declaration centered on `messages.Say`.
  **L2018 CN**: 执行以 `messages.Say` 为核心的调用或声明。
- **L2019 EN**: Closes the current lexical scope or compound statement.
  **L2019 CN**: 结束当前词法作用域或复合语句块。
- **L2020 EN**: Initializes variable `arg` from the right-hand expression.
  **L2020 CN**: 使用右侧表达式初始化变量 `arg`。
- **L2021 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2021 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2022 EN**: Starts a function, method, lambda, or structured scope: `!symbol || !symbol->test(Symbol::Flag::CrayPointer)) {`.
  **L2022 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!symbol || !symbol->test(Symbol::Flag::CrayPointer)) {`。
- **L2023 EN**: Executes a call or declaration centered on `messages.Say`.
  **L2023 CN**: 执行以 `messages.Say` 为核心的调用或声明。
- **L2024 EN**: Closes the current lexical scope or compound statement.
  **L2024 CN**: 结束当前词法作用域或复合语句块。
- **L2025 EN**: Closes the current lexical scope or compound statement.
  **L2025 CN**: 结束当前词法作用域或复合语句块。
- **L2026 EN**: Blank line separating nearby declarations or logic blocks.
  **L2026 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2027 EN**: Comment explains nearby logic, intent, or metadata: `MOVE_ALLOC (F'2023 16.9.147)`.
  **L2027 CN**: 注释说明附近代码的逻辑、意图或元数据：`MOVE_ALLOC (F'2023 16.9.147)`。
- **L2028 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckMove_Alloc(evaluate::ActualArguments &arguments,`.
  **L2028 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckMove_Alloc(evaluate::ActualArguments &arguments,`。
- **L2029 EN**: Continues the surrounding expression or declaration: `parser::ContextualMessages &messages) {`.
  **L2029 CN**: 继续构造周围的表达式或声明：`parser::ContextualMessages &messages) {`。
- **L2030 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2030 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2031 EN**: Continues logic associated with callable symbol `CheckForCoindexedObject`.
  **L2031 CN**: 继续与可调用符号 `CheckForCoindexedObject` 相关的逻辑。
- **L2032 EN**: Executes a standalone statement or declaration: `messages, arguments[0], "move_alloc", "from");`.
  **L2032 CN**: 执行一条独立语句或声明：`messages, arguments[0], "move_alloc", "from");`。
- **L2033 EN**: Closes the current lexical scope or compound statement.
  **L2033 CN**: 结束当前词法作用域或复合语句块。
- **L2034 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2034 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2035 EN**: Continues logic associated with callable symbol `CheckForCoindexedObject`.
  **L2035 CN**: 继续与可调用符号 `CheckForCoindexedObject` 相关的逻辑。
- **L2036 EN**: Executes a standalone statement or declaration: `messages, arguments[1], "move_alloc", "to");`.
  **L2036 CN**: 执行一条独立语句或声明：`messages, arguments[1], "move_alloc", "to");`。
- **L2037 EN**: Executes a call or declaration centered on `fromCR{GetCorank`.
  **L2037 CN**: 执行以 `fromCR{GetCorank` 为核心的调用或声明。
- **L2038 EN**: Executes a call or declaration centered on `toCR{GetCorank`.
  **L2038 CN**: 执行以 `toCR{GetCorank` 为核心的调用或声明。
- **L2039 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2039 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2040 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(*arguments[0]->sourceLocation(),`.
  **L2040 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(*arguments[0]->sourceLocation(),`。

### Lines 2041-2064

````cpp
          "FROM= argument to MOVE_ALLOC has corank %d, but TO= argument has corank %d"_err_en_US,
          fromCR, toCR);
    }
  }
  if (arguments.size() >= 3) {
    evaluate::CheckForCoindexedObject(
        messages, arguments[2], "move_alloc", "stat");
  }
  if (arguments.size() >= 4) {
    evaluate::CheckForCoindexedObject(
        messages, arguments[3], "move_alloc", "errmsg");
  }
  if (arguments.size() >= 2 && arguments[0] && arguments[1]) {
    for (int j{0}; j < 2; ++j) {
      if (const Symbol *
              whole{UnwrapWholeSymbolOrComponentDataRef(arguments[j])};
          !whole || !IsAllocatable(whole->GetUltimate())) {
        messages.Say(*arguments[j]->sourceLocation(),
            "Argument #%d to MOVE_ALLOC must be allocatable"_err_en_US, j + 1);
      }
    }
    auto type0{arguments[0]->GetType()};
    auto type1{arguments[1]->GetType()};
    if (type0 && type1 && type0->IsPolymorphic() && !type1->IsPolymorphic()) {
````
- **L2041 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"FROM= argument to MOVE_ALLOC has corank %d, but TO= argument has corank %d"_err_en_US,`.
  **L2041 CN**: 继续一个多行参数列表、初始化器或聚合项：`"FROM= argument to MOVE_ALLOC has corank %d, but TO= argument has corank %d"_err_en_US,`。
- **L2042 EN**: Executes a standalone statement or declaration: `fromCR, toCR);`.
  **L2042 CN**: 执行一条独立语句或声明：`fromCR, toCR);`。
- **L2043 EN**: Closes the current lexical scope or compound statement.
  **L2043 CN**: 结束当前词法作用域或复合语句块。
- **L2044 EN**: Closes the current lexical scope or compound statement.
  **L2044 CN**: 结束当前词法作用域或复合语句块。
- **L2045 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2045 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2046 EN**: Continues logic associated with callable symbol `CheckForCoindexedObject`.
  **L2046 CN**: 继续与可调用符号 `CheckForCoindexedObject` 相关的逻辑。
- **L2047 EN**: Executes a standalone statement or declaration: `messages, arguments[2], "move_alloc", "stat");`.
  **L2047 CN**: 执行一条独立语句或声明：`messages, arguments[2], "move_alloc", "stat");`。
- **L2048 EN**: Closes the current lexical scope or compound statement.
  **L2048 CN**: 结束当前词法作用域或复合语句块。
- **L2049 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2049 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2050 EN**: Continues logic associated with callable symbol `CheckForCoindexedObject`.
  **L2050 CN**: 继续与可调用符号 `CheckForCoindexedObject` 相关的逻辑。
- **L2051 EN**: Executes a standalone statement or declaration: `messages, arguments[3], "move_alloc", "errmsg");`.
  **L2051 CN**: 执行一条独立语句或声明：`messages, arguments[3], "move_alloc", "errmsg");`。
- **L2052 EN**: Closes the current lexical scope or compound statement.
  **L2052 CN**: 结束当前词法作用域或复合语句块。
- **L2053 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2053 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2054 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2054 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2055 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2055 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2056 EN**: Executes a call or declaration centered on `whole{UnwrapWholeSymbolOrComponentDataRef`.
  **L2056 CN**: 执行以 `whole{UnwrapWholeSymbolOrComponentDataRef` 为核心的调用或声明。
- **L2057 EN**: Starts a function, method, lambda, or structured scope: `!whole || !IsAllocatable(whole->GetUltimate())) {`.
  **L2057 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!whole || !IsAllocatable(whole->GetUltimate())) {`。
- **L2058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(*arguments[j]->sourceLocation(),`.
  **L2058 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(*arguments[j]->sourceLocation(),`。
- **L2059 EN**: Executes a standalone statement or declaration: `"Argument #%d to MOVE_ALLOC must be allocatable"_err_en_US, j + 1);`.
  **L2059 CN**: 执行一条独立语句或声明：`"Argument #%d to MOVE_ALLOC must be allocatable"_err_en_US, j + 1);`。
- **L2060 EN**: Closes the current lexical scope or compound statement.
  **L2060 CN**: 结束当前词法作用域或复合语句块。
- **L2061 EN**: Closes the current lexical scope or compound statement.
  **L2061 CN**: 结束当前词法作用域或复合语句块。
- **L2062 EN**: Executes a call or declaration centered on `type0{arguments[0]->GetType`.
  **L2062 CN**: 执行以 `type0{arguments[0]->GetType` 为核心的调用或声明。
- **L2063 EN**: Executes a call or declaration centered on `type1{arguments[1]->GetType`.
  **L2063 CN**: 执行以 `type1{arguments[1]->GetType` 为核心的调用或声明。
- **L2064 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2064 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2065-2088

````cpp
      messages.Say(arguments[1]->sourceLocation(),
          "When MOVE_ALLOC(FROM=) is polymorphic, TO= must also be polymorphic"_err_en_US);
    }
  }
}

// PRESENT (F'2023 16.9.163)
static void CheckPresent(evaluate::ActualArguments &arguments,
    parser::ContextualMessages &messages) {
  if (arguments.size() == 1) {
    if (const auto &arg{arguments[0]}; arg) {
      const Symbol *symbol{nullptr};
      if (const auto *expr{arg->UnwrapExpr()}) {
        if (const auto *proc{
                std::get_if<evaluate::ProcedureDesignator>(&expr->u)}) {
          symbol = proc->GetSymbol();
        } else {
          symbol = evaluate::UnwrapWholeSymbolDataRef(*expr);
        }
      } else {
        symbol = arg->GetAssumedTypeDummy();
      }
      if (!symbol ||
          !symbol->GetUltimate().attrs().test(semantics::Attr::OPTIONAL)) {
````
- **L2065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arguments[1]->sourceLocation(),`.
  **L2065 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arguments[1]->sourceLocation(),`。
- **L2066 EN**: Executes a call or declaration centered on `MOVE_ALLOC`.
  **L2066 CN**: 执行以 `MOVE_ALLOC` 为核心的调用或声明。
- **L2067 EN**: Closes the current lexical scope or compound statement.
  **L2067 CN**: 结束当前词法作用域或复合语句块。
- **L2068 EN**: Closes the current lexical scope or compound statement.
  **L2068 CN**: 结束当前词法作用域或复合语句块。
- **L2069 EN**: Closes the current lexical scope or compound statement.
  **L2069 CN**: 结束当前词法作用域或复合语句块。
- **L2070 EN**: Blank line separating nearby declarations or logic blocks.
  **L2070 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2071 EN**: Comment explains nearby logic, intent, or metadata: `PRESENT (F'2023 16.9.163)`.
  **L2071 CN**: 注释说明附近代码的逻辑、意图或元数据：`PRESENT (F'2023 16.9.163)`。
- **L2072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckPresent(evaluate::ActualArguments &arguments,`.
  **L2072 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckPresent(evaluate::ActualArguments &arguments,`。
- **L2073 EN**: Continues the surrounding expression or declaration: `parser::ContextualMessages &messages) {`.
  **L2073 CN**: 继续构造周围的表达式或声明：`parser::ContextualMessages &messages) {`。
- **L2074 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2074 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2075 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2075 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2076 EN**: Executes a standalone statement or declaration: `const Symbol *symbol{nullptr};`.
  **L2076 CN**: 执行一条独立语句或声明：`const Symbol *symbol{nullptr};`。
- **L2077 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2077 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2078 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2078 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2079 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<evaluate::ProcedureDesignator>(&expr->u)}) {`.
  **L2079 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<evaluate::ProcedureDesignator>(&expr->u)}) {`。
- **L2080 EN**: Executes a call or declaration centered on `proc->GetSymbol`.
  **L2080 CN**: 执行以 `proc->GetSymbol` 为核心的调用或声明。
- **L2081 EN**: Transitions from the previous branch into the alternative path.
  **L2081 CN**: 从前一个分支过渡到备选路径。
- **L2082 EN**: Executes a call or declaration centered on `evaluate::UnwrapWholeSymbolDataRef`.
  **L2082 CN**: 执行以 `evaluate::UnwrapWholeSymbolDataRef` 为核心的调用或声明。
- **L2083 EN**: Closes the current lexical scope or compound statement.
  **L2083 CN**: 结束当前词法作用域或复合语句块。
- **L2084 EN**: Transitions from the previous branch into the alternative path.
  **L2084 CN**: 从前一个分支过渡到备选路径。
- **L2085 EN**: Executes a call or declaration centered on `arg->GetAssumedTypeDummy`.
  **L2085 CN**: 执行以 `arg->GetAssumedTypeDummy` 为核心的调用或声明。
- **L2086 EN**: Closes the current lexical scope or compound statement.
  **L2086 CN**: 结束当前词法作用域或复合语句块。
- **L2087 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2087 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2088 EN**: Starts a function, method, lambda, or structured scope: `!symbol->GetUltimate().attrs().test(semantics::Attr::OPTIONAL)) {`.
  **L2088 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!symbol->GetUltimate().attrs().test(semantics::Attr::OPTIONAL)) {`。

### Lines 2089-2112

````cpp
        messages.Say(arg ? arg->sourceLocation() : messages.at(),
            "Argument of PRESENT() must be the name of a whole OPTIONAL dummy argument"_err_en_US);
      }
    }
  }
}

// REDUCE (F'2023 16.9.173)
static void CheckReduce(
    evaluate::ActualArguments &arguments, evaluate::FoldingContext &context) {
  std::optional<evaluate::DynamicType> arrayType;
  parser::ContextualMessages &messages{context.messages()};
  if (const auto &array{arguments[0]}) {
    arrayType = array->GetType();
    if (!arguments[/*identity=*/4]) {
      if (const auto *expr{array->UnwrapExpr()}) {
        if (auto shape{
                evaluate::GetShape(context, *expr, /*invariantOnly=*/false)}) {
          if (const auto &dim{arguments[2]}; dim && array->Rank() > 1) {
            // Partial reduction
            auto dimVal{evaluate::ToInt64(dim->UnwrapExpr())};
            std::int64_t j{0};
            int zeroDims{0};
            bool isSelectedDimEmpty{false};
````
- **L2089 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say(arg ? arg->sourceLocation() : messages.at(),`.
  **L2089 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say(arg ? arg->sourceLocation() : messages.at(),`。
- **L2090 EN**: Executes a call or declaration centered on `PRESENT`.
  **L2090 CN**: 执行以 `PRESENT` 为核心的调用或声明。
- **L2091 EN**: Closes the current lexical scope or compound statement.
  **L2091 CN**: 结束当前词法作用域或复合语句块。
- **L2092 EN**: Closes the current lexical scope or compound statement.
  **L2092 CN**: 结束当前词法作用域或复合语句块。
- **L2093 EN**: Closes the current lexical scope or compound statement.
  **L2093 CN**: 结束当前词法作用域或复合语句块。
- **L2094 EN**: Closes the current lexical scope or compound statement.
  **L2094 CN**: 结束当前词法作用域或复合语句块。
- **L2095 EN**: Blank line separating nearby declarations or logic blocks.
  **L2095 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2096 EN**: Comment explains nearby logic, intent, or metadata: `REDUCE (F'2023 16.9.173)`.
  **L2096 CN**: 注释说明附近代码的逻辑、意图或元数据：`REDUCE (F'2023 16.9.173)`。
- **L2097 EN**: Continues logic associated with callable symbol `CheckReduce`.
  **L2097 CN**: 继续与可调用符号 `CheckReduce` 相关的逻辑。
- **L2098 EN**: Continues the surrounding expression or declaration: `evaluate::ActualArguments &arguments, evaluate::FoldingContext &context) {`.
  **L2098 CN**: 继续构造周围的表达式或声明：`evaluate::ActualArguments &arguments, evaluate::FoldingContext &context) {`。
- **L2099 EN**: Executes a standalone statement or declaration: `std::optional<evaluate::DynamicType> arrayType;`.
  **L2099 CN**: 执行一条独立语句或声明：`std::optional<evaluate::DynamicType> arrayType;`。
- **L2100 EN**: Executes a call or declaration centered on `&messages{context.messages`.
  **L2100 CN**: 执行以 `&messages{context.messages` 为核心的调用或声明。
- **L2101 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2101 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2102 EN**: Executes a call or declaration centered on `array->GetType`.
  **L2102 CN**: 执行以 `array->GetType` 为核心的调用或声明。
- **L2103 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2103 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2104 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2104 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2105 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2105 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2106 EN**: Starts a function, method, lambda, or structured scope: `evaluate::GetShape(context, *expr, /*invariantOnly=*/false)}) {`.
  **L2106 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::GetShape(context, *expr, /*invariantOnly=*/false)}) {`。
- **L2107 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2107 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2108 EN**: Comment explains nearby logic, intent, or metadata: `Partial reduction`.
  **L2108 CN**: 注释说明附近代码的逻辑、意图或元数据：`Partial reduction`。
- **L2109 EN**: Executes a call or declaration centered on `dimVal{evaluate::ToInt64`.
  **L2109 CN**: 执行以 `dimVal{evaluate::ToInt64` 为核心的调用或声明。
- **L2110 EN**: Executes a standalone statement or declaration: `std::int64_t j{0};`.
  **L2110 CN**: 执行一条独立语句或声明：`std::int64_t j{0};`。
- **L2111 EN**: Executes a standalone statement or declaration: `int zeroDims{0};`.
  **L2111 CN**: 执行一条独立语句或声明：`int zeroDims{0};`。
- **L2112 EN**: Executes a standalone statement or declaration: `bool isSelectedDimEmpty{false};`.
  **L2112 CN**: 执行一条独立语句或声明：`bool isSelectedDimEmpty{false};`。

### Lines 2113-2136

````cpp
            for (const auto &extent : *shape) {
              ++j;
              if (evaluate::ToInt64(extent) == 0) {
                ++zeroDims;
                isSelectedDimEmpty |= dimVal && j == *dimVal;
              }
            }
            if (isSelectedDimEmpty && zeroDims == 1) {
              messages.Say(
                  "IDENTITY= must be present when DIM=%d and the array has zero extent on that dimension"_err_en_US,
                  static_cast<int>(dimVal.value()));
            }
          } else { // no DIM= or DIM=1 on a vector: total reduction
            for (const auto &extent : *shape) {
              if (evaluate::ToInt64(extent) == 0) {
                messages.Say(
                    "IDENTITY= must be present when the array is empty and the result is scalar"_err_en_US);
                break;
              }
            }
          }
        }
      }
    }
````
- **L2113 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2113 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2114 EN**: Executes a standalone statement or declaration: `++j;`.
  **L2114 CN**: 执行一条独立语句或声明：`++j;`。
- **L2115 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2115 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2116 EN**: Executes a standalone statement or declaration: `++zeroDims;`.
  **L2116 CN**: 执行一条独立语句或声明：`++zeroDims;`。
- **L2117 EN**: Executes a standalone statement or declaration: `isSelectedDimEmpty |= dimVal && j == *dimVal;`.
  **L2117 CN**: 执行一条独立语句或声明：`isSelectedDimEmpty |= dimVal && j == *dimVal;`。
- **L2118 EN**: Closes the current lexical scope or compound statement.
  **L2118 CN**: 结束当前词法作用域或复合语句块。
- **L2119 EN**: Closes the current lexical scope or compound statement.
  **L2119 CN**: 结束当前词法作用域或复合语句块。
- **L2120 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2120 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2121 EN**: Continues logic associated with callable symbol `Say`.
  **L2121 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"IDENTITY= must be present when DIM=%d and the array has zero extent on that dimension"_err_en_US,`.
  **L2122 CN**: 继续一个多行参数列表、初始化器或聚合项：`"IDENTITY= must be present when DIM=%d and the array has zero extent on that dimension"_err_en_US,`。
- **L2123 EN**: Executes a call or declaration centered on `static_cast<int>`.
  **L2123 CN**: 执行以 `static_cast<int>` 为核心的调用或声明。
- **L2124 EN**: Closes the current lexical scope or compound statement.
  **L2124 CN**: 结束当前词法作用域或复合语句块。
- **L2125 EN**: Transitions from the previous branch into the alternative path.
  **L2125 CN**: 从前一个分支过渡到备选路径。
- **L2126 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2126 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2127 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2127 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2128 EN**: Continues logic associated with callable symbol `Say`.
  **L2128 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2129 EN**: Executes a standalone statement or declaration: `"IDENTITY= must be present when the array is empty and the result is scalar"_err_en_US);`.
  **L2129 CN**: 执行一条独立语句或声明：`"IDENTITY= must be present when the array is empty and the result is scalar"_err_en_US);`。
- **L2130 EN**: Exits the nearest loop or switch statement.
  **L2130 CN**: 退出最近的循环或 switch 语句。
- **L2131 EN**: Closes the current lexical scope or compound statement.
  **L2131 CN**: 结束当前词法作用域或复合语句块。
- **L2132 EN**: Closes the current lexical scope or compound statement.
  **L2132 CN**: 结束当前词法作用域或复合语句块。
- **L2133 EN**: Closes the current lexical scope or compound statement.
  **L2133 CN**: 结束当前词法作用域或复合语句块。
- **L2134 EN**: Closes the current lexical scope or compound statement.
  **L2134 CN**: 结束当前词法作用域或复合语句块。
- **L2135 EN**: Closes the current lexical scope or compound statement.
  **L2135 CN**: 结束当前词法作用域或复合语句块。
- **L2136 EN**: Closes the current lexical scope or compound statement.
  **L2136 CN**: 结束当前词法作用域或复合语句块。

### Lines 2137-2160

````cpp
  }
  std::optional<characteristics::Procedure> procChars;
  if (const auto &operation{arguments[1]}) {
    if (const auto *expr{operation->UnwrapExpr()}) {
      if (const auto *designator{
              std::get_if<evaluate::ProcedureDesignator>(&expr->u)}) {
        procChars = characteristics::Procedure::Characterize(
            *designator, context, /*emitError=*/true);
      } else if (const auto *ref{
                     std::get_if<evaluate::ProcedureRef>(&expr->u)}) {
        procChars = characteristics::Procedure::Characterize(*ref, context);
      }
    }
  }
  const auto *result{procChars && procChars->functionResult
          ? procChars->functionResult->GetTypeAndShape()
          : nullptr};
  if (!procChars || !procChars->IsPure() ||
      procChars->dummyArguments.size() != 2 || !procChars->functionResult) {
    messages.Say(
        "OPERATION= argument of REDUCE() must be a pure function of two data arguments"_err_en_US);
  } else if (procChars->attrs.test(characteristics::Procedure::Attr::BindC)) {
    messages.Say(
        "A BIND(C) OPERATION= argument of REDUCE() is not supported"_err_en_US);
````
- **L2137 EN**: Closes the current lexical scope or compound statement.
  **L2137 CN**: 结束当前词法作用域或复合语句块。
- **L2138 EN**: Executes a standalone statement or declaration: `std::optional<characteristics::Procedure> procChars;`.
  **L2138 CN**: 执行一条独立语句或声明：`std::optional<characteristics::Procedure> procChars;`。
- **L2139 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2139 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2140 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2140 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2141 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2141 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2142 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<evaluate::ProcedureDesignator>(&expr->u)}) {`.
  **L2142 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<evaluate::ProcedureDesignator>(&expr->u)}) {`。
- **L2143 EN**: Continues logic associated with callable symbol `Characterize`.
  **L2143 CN**: 继续与可调用符号 `Characterize` 相关的逻辑。
- **L2144 EN**: Comment explains nearby logic, intent, or metadata: `designator, context, /*emitError=*/true);`.
  **L2144 CN**: 注释说明附近代码的逻辑、意图或元数据：`designator, context, /*emitError=*/true);`。
- **L2145 EN**: Transitions from the previous branch into an `else if` condition.
  **L2145 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2146 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<evaluate::ProcedureRef>(&expr->u)}) {`.
  **L2146 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<evaluate::ProcedureRef>(&expr->u)}) {`。
- **L2147 EN**: Executes a call or declaration centered on `characteristics::Procedure::Characterize`.
  **L2147 CN**: 执行以 `characteristics::Procedure::Characterize` 为核心的调用或声明。
- **L2148 EN**: Closes the current lexical scope or compound statement.
  **L2148 CN**: 结束当前词法作用域或复合语句块。
- **L2149 EN**: Closes the current lexical scope or compound statement.
  **L2149 CN**: 结束当前词法作用域或复合语句块。
- **L2150 EN**: Closes the current lexical scope or compound statement.
  **L2150 CN**: 结束当前词法作用域或复合语句块。
- **L2151 EN**: Continues the surrounding expression or declaration: `const auto *result{procChars && procChars->functionResult`.
  **L2151 CN**: 继续构造周围的表达式或声明：`const auto *result{procChars && procChars->functionResult`。
- **L2152 EN**: Continues logic associated with callable symbol `GetTypeAndShape`.
  **L2152 CN**: 继续与可调用符号 `GetTypeAndShape` 相关的逻辑。
- **L2153 EN**: Executes a standalone statement or declaration: `: nullptr};`.
  **L2153 CN**: 执行一条独立语句或声明：`: nullptr};`。
- **L2154 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2154 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2155 EN**: Starts a function, method, lambda, or structured scope: `procChars->dummyArguments.size() != 2 || !procChars->functionResult) {`.
  **L2155 CN**: 开始一个函数、方法、lambda 或结构化作用域：`procChars->dummyArguments.size() != 2 || !procChars->functionResult) {`。
- **L2156 EN**: Continues logic associated with callable symbol `Say`.
  **L2156 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2157 EN**: Executes a call or declaration centered on `REDUCE`.
  **L2157 CN**: 执行以 `REDUCE` 为核心的调用或声明。
- **L2158 EN**: Transitions from the previous branch into an `else if` condition.
  **L2158 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2159 EN**: Continues logic associated with callable symbol `Say`.
  **L2159 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2160 EN**: Executes a call or declaration centered on `BIND`.
  **L2160 CN**: 执行以 `BIND` 为核心的调用或声明。

### Lines 2161-2184

````cpp
  } else if (!result || result->Rank() != 0) {
    messages.Say(
        "OPERATION= argument of REDUCE() must be a scalar function"_err_en_US);
  } else if (result->type().IsPolymorphic() ||
      (arrayType && !arrayType->IsTkLenCompatibleWith(result->type()))) {
    messages.Say(
        "OPERATION= argument of REDUCE() must have the same type as ARRAY="_err_en_US);
  } else {
    const characteristics::DummyDataObject *data[2]{};
    for (int j{0}; j < 2; ++j) {
      const auto &dummy{procChars->dummyArguments.at(j)};
      data[j] = std::get_if<characteristics::DummyDataObject>(&dummy.u);
    }
    if (!data[0] || !data[1]) {
      messages.Say(
          "OPERATION= argument of REDUCE() may not have dummy procedure arguments"_err_en_US);
    } else {
      for (int j{0}; j < 2; ++j) {
        if (data[j]->attrs.test(
                characteristics::DummyDataObject::Attr::Optional) ||
            data[j]->attrs.test(
                characteristics::DummyDataObject::Attr::Allocatable) ||
            data[j]->attrs.test(
                characteristics::DummyDataObject::Attr::Pointer) ||
````
- **L2161 EN**: Transitions from the previous branch into an `else if` condition.
  **L2161 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2162 EN**: Continues logic associated with callable symbol `Say`.
  **L2162 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2163 EN**: Executes a call or declaration centered on `REDUCE`.
  **L2163 CN**: 执行以 `REDUCE` 为核心的调用或声明。
- **L2164 EN**: Transitions from the previous branch into an `else if` condition.
  **L2164 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2165 EN**: Starts a function, method, lambda, or structured scope: `(arrayType && !arrayType->IsTkLenCompatibleWith(result->type()))) {`.
  **L2165 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(arrayType && !arrayType->IsTkLenCompatibleWith(result->type()))) {`。
- **L2166 EN**: Continues logic associated with callable symbol `Say`.
  **L2166 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2167 EN**: Executes a call or declaration centered on `REDUCE`.
  **L2167 CN**: 执行以 `REDUCE` 为核心的调用或声明。
- **L2168 EN**: Transitions from the previous branch into the alternative path.
  **L2168 CN**: 从前一个分支过渡到备选路径。
- **L2169 EN**: Executes a standalone statement or declaration: `const characteristics::DummyDataObject *data[2]{};`.
  **L2169 CN**: 执行一条独立语句或声明：`const characteristics::DummyDataObject *data[2]{};`。
- **L2170 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2170 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2171 EN**: Executes a call or declaration centered on `&dummy{procChars->dummyArguments.at`.
  **L2171 CN**: 执行以 `&dummy{procChars->dummyArguments.at` 为核心的调用或声明。
- **L2172 EN**: Executes a call or declaration centered on `std::get_if<characteristics::DummyDataObject>`.
  **L2172 CN**: 执行以 `std::get_if<characteristics::DummyDataObject>` 为核心的调用或声明。
- **L2173 EN**: Closes the current lexical scope or compound statement.
  **L2173 CN**: 结束当前词法作用域或复合语句块。
- **L2174 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2174 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2175 EN**: Continues logic associated with callable symbol `Say`.
  **L2175 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2176 EN**: Executes a call or declaration centered on `REDUCE`.
  **L2176 CN**: 执行以 `REDUCE` 为核心的调用或声明。
- **L2177 EN**: Transitions from the previous branch into the alternative path.
  **L2177 CN**: 从前一个分支过渡到备选路径。
- **L2178 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2178 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2179 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2179 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2180 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject::Attr::Optional) ||`.
  **L2180 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject::Attr::Optional) ||`。
- **L2181 EN**: Continues logic associated with callable symbol `test`.
  **L2181 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L2182 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject::Attr::Allocatable) ||`.
  **L2182 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject::Attr::Allocatable) ||`。
- **L2183 EN**: Continues logic associated with callable symbol `test`.
  **L2183 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L2184 EN**: Continues the surrounding expression or declaration: `characteristics::DummyDataObject::Attr::Pointer) ||`.
  **L2184 CN**: 继续构造周围的表达式或声明：`characteristics::DummyDataObject::Attr::Pointer) ||`。

### Lines 2185-2208

````cpp
            data[j]->type.Rank() != 0 || data[j]->type.type().IsPolymorphic() ||
            (arrayType &&
                !data[j]->type.type().IsTkCompatibleWith(*arrayType))) {
          messages.Say(
              "Arguments of OPERATION= procedure of REDUCE() must be both scalar of the same type as ARRAY=, and neither allocatable, pointer, polymorphic, nor optional"_err_en_US);
        }
      }
      static constexpr characteristics::DummyDataObject::Attr attrs[]{
          characteristics::DummyDataObject::Attr::Asynchronous,
          characteristics::DummyDataObject::Attr::Target,
          characteristics::DummyDataObject::Attr::Value,
      };
      for (std::size_t j{0}; j < sizeof attrs / sizeof *attrs; ++j) {
        if (data[0]->attrs.test(attrs[j]) != data[1]->attrs.test(attrs[j])) {
          messages.Say(
              "If either argument of the OPERATION= procedure of REDUCE() has the ASYNCHRONOUS, TARGET, or VALUE attribute, both must have that attribute"_err_en_US);
          break;
        }
      }
    }
  }
  // When the MASK= is present and has no .TRUE. element, and there is
  // no IDENTITY=, it's an error.
  if (const auto &mask{arguments[3]}; mask && !arguments[/*identity*/ 4]) {
````
- **L2185 EN**: Continues logic associated with callable symbol `Rank`.
  **L2185 CN**: 继续与可调用符号 `Rank` 相关的逻辑。
- **L2186 EN**: Continues the surrounding expression or declaration: `(arrayType &&`.
  **L2186 CN**: 继续构造周围的表达式或声明：`(arrayType &&`。
- **L2187 EN**: Starts a function, method, lambda, or structured scope: `!data[j]->type.type().IsTkCompatibleWith(*arrayType))) {`.
  **L2187 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!data[j]->type.type().IsTkCompatibleWith(*arrayType))) {`。
- **L2188 EN**: Continues logic associated with callable symbol `Say`.
  **L2188 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2189 EN**: Executes a call or declaration centered on `REDUCE`.
  **L2189 CN**: 执行以 `REDUCE` 为核心的调用或声明。
- **L2190 EN**: Closes the current lexical scope or compound statement.
  **L2190 CN**: 结束当前词法作用域或复合语句块。
- **L2191 EN**: Closes the current lexical scope or compound statement.
  **L2191 CN**: 结束当前词法作用域或复合语句块。
- **L2192 EN**: Continues the surrounding expression or declaration: `static constexpr characteristics::DummyDataObject::Attr attrs[]{`.
  **L2192 CN**: 继续构造周围的表达式或声明：`static constexpr characteristics::DummyDataObject::Attr attrs[]{`。
- **L2193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::DummyDataObject::Attr::Asynchronous,`.
  **L2193 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::DummyDataObject::Attr::Asynchronous,`。
- **L2194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::DummyDataObject::Attr::Target,`.
  **L2194 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::DummyDataObject::Attr::Target,`。
- **L2195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `characteristics::DummyDataObject::Attr::Value,`.
  **L2195 CN**: 继续一个多行参数列表、初始化器或聚合项：`characteristics::DummyDataObject::Attr::Value,`。
- **L2196 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2196 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2197 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2197 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2198 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2198 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2199 EN**: Continues logic associated with callable symbol `Say`.
  **L2199 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2200 EN**: Executes a call or declaration centered on `REDUCE`.
  **L2200 CN**: 执行以 `REDUCE` 为核心的调用或声明。
- **L2201 EN**: Exits the nearest loop or switch statement.
  **L2201 CN**: 退出最近的循环或 switch 语句。
- **L2202 EN**: Closes the current lexical scope or compound statement.
  **L2202 CN**: 结束当前词法作用域或复合语句块。
- **L2203 EN**: Closes the current lexical scope or compound statement.
  **L2203 CN**: 结束当前词法作用域或复合语句块。
- **L2204 EN**: Closes the current lexical scope or compound statement.
  **L2204 CN**: 结束当前词法作用域或复合语句块。
- **L2205 EN**: Closes the current lexical scope or compound statement.
  **L2205 CN**: 结束当前词法作用域或复合语句块。
- **L2206 EN**: Comment explains nearby logic, intent, or metadata: `When the MASK= is present and has no .TRUE. element, and there is`.
  **L2206 CN**: 注释说明附近代码的逻辑、意图或元数据：`When the MASK= is present and has no .TRUE. element, and there is`。
- **L2207 EN**: Comment explains nearby logic, intent, or metadata: `no IDENTITY=, it's an error.`.
  **L2207 CN**: 注释说明附近代码的逻辑、意图或元数据：`no IDENTITY=, it's an error.`。
- **L2208 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2208 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2209-2232

````cpp
    if (const auto *expr{mask->UnwrapExpr()}) {
      if (const auto *logical{
              std::get_if<evaluate::Expr<evaluate::SomeLogical>>(&expr->u)}) {
        if (common::visit(
                [](const auto &kindExpr) {
                  using KindExprType = std::decay_t<decltype(kindExpr)>;
                  using KindLogical = typename KindExprType::Result;
                  if (const auto *c{evaluate::UnwrapConstantValue<KindLogical>(
                          kindExpr)}) {
                    for (const auto &element : c->values()) {
                      if (element.IsTrue()) {
                        return false;
                      }
                    }
                    return true;
                  }
                  return false;
                },
                logical->u)) {
          messages.Say(
              "MASK= has no .TRUE. element, so IDENTITY= must be present"_err_en_US);
        }
      }
    }
````
- **L2209 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2209 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2210 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2210 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2211 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<evaluate::Expr<evaluate::SomeLogical>>(&expr->u)}) {`.
  **L2211 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<evaluate::Expr<evaluate::SomeLogical>>(&expr->u)}) {`。
- **L2212 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2212 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2213 EN**: Starts a function, method, lambda, or structured scope: `[](const auto &kindExpr) {`.
  **L2213 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[](const auto &kindExpr) {`。
- **L2214 EN**: Defines alias `KindExprType` to simplify later code.
  **L2214 CN**: 定义别名 `KindExprType` 以简化后续代码。
- **L2215 EN**: Defines alias `KindLogical` to simplify later code.
  **L2215 CN**: 定义别名 `KindLogical` 以简化后续代码。
- **L2216 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2216 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2217 EN**: Continues the surrounding expression or declaration: `kindExpr)}) {`.
  **L2217 CN**: 继续构造周围的表达式或声明：`kindExpr)}) {`。
- **L2218 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2218 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2219 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2219 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2220 EN**: Returns from the current function with `false`.
  **L2220 CN**: 以 `false` 从当前函数返回。
- **L2221 EN**: Closes the current lexical scope or compound statement.
  **L2221 CN**: 结束当前词法作用域或复合语句块。
- **L2222 EN**: Closes the current lexical scope or compound statement.
  **L2222 CN**: 结束当前词法作用域或复合语句块。
- **L2223 EN**: Returns from the current function with `true`.
  **L2223 CN**: 以 `true` 从当前函数返回。
- **L2224 EN**: Closes the current lexical scope or compound statement.
  **L2224 CN**: 结束当前词法作用域或复合语句块。
- **L2225 EN**: Returns from the current function with `false`.
  **L2225 CN**: 以 `false` 从当前函数返回。
- **L2226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L2226 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L2227 EN**: Continues the surrounding expression or declaration: `logical->u)) {`.
  **L2227 CN**: 继续构造周围的表达式或声明：`logical->u)) {`。
- **L2228 EN**: Continues logic associated with callable symbol `Say`.
  **L2228 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2229 EN**: Executes a standalone statement or declaration: `"MASK= has no .TRUE. element, so IDENTITY= must be present"_err_en_US);`.
  **L2229 CN**: 执行一条独立语句或声明：`"MASK= has no .TRUE. element, so IDENTITY= must be present"_err_en_US);`。
- **L2230 EN**: Closes the current lexical scope or compound statement.
  **L2230 CN**: 结束当前词法作用域或复合语句块。
- **L2231 EN**: Closes the current lexical scope or compound statement.
  **L2231 CN**: 结束当前词法作用域或复合语句块。
- **L2232 EN**: Closes the current lexical scope or compound statement.
  **L2232 CN**: 结束当前词法作用域或复合语句块。

### Lines 2233-2256

````cpp
  }
}

// TRANSFER (16.9.193)
static void CheckTransferOperandType(SemanticsContext &context,
    const evaluate::DynamicType &type, const char *which) {
  if (type.IsPolymorphic()) {
    context.foldingContext().Warn(common::UsageWarning::PolymorphicTransferArg,
        "%s of TRANSFER is polymorphic"_warn_en_US, which);
  } else if (!type.IsUnlimitedPolymorphic() &&
      type.category() == TypeCategory::Derived &&
      context.ShouldWarn(common::UsageWarning::PointerComponentTransferArg)) {
    DirectComponentIterator directs{type.GetDerivedTypeSpec()};
    if (auto bad{std::find_if(directs.begin(), directs.end(), IsDescriptor)};
        bad != directs.end()) {
      evaluate::WarnWithDeclaration(context.foldingContext(), *bad,
          common::UsageWarning::PointerComponentTransferArg,
          "%s of TRANSFER contains allocatable or pointer component %s"_warn_en_US,
          which, bad.BuildResultDesignatorName());
    }
  }
}

static void CheckTransfer(evaluate::ActualArguments &arguments,
````
- **L2233 EN**: Closes the current lexical scope or compound statement.
  **L2233 CN**: 结束当前词法作用域或复合语句块。
- **L2234 EN**: Closes the current lexical scope or compound statement.
  **L2234 CN**: 结束当前词法作用域或复合语句块。
- **L2235 EN**: Blank line separating nearby declarations or logic blocks.
  **L2235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2236 EN**: Comment explains nearby logic, intent, or metadata: `TRANSFER (16.9.193)`.
  **L2236 CN**: 注释说明附近代码的逻辑、意图或元数据：`TRANSFER (16.9.193)`。
- **L2237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckTransferOperandType(SemanticsContext &context,`.
  **L2237 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckTransferOperandType(SemanticsContext &context,`。
- **L2238 EN**: Continues the surrounding expression or declaration: `const evaluate::DynamicType &type, const char *which) {`.
  **L2238 CN**: 继续构造周围的表达式或声明：`const evaluate::DynamicType &type, const char *which) {`。
- **L2239 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2239 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.foldingContext().Warn(common::UsageWarning::PolymorphicTransferArg,`.
  **L2240 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.foldingContext().Warn(common::UsageWarning::PolymorphicTransferArg,`。
- **L2241 EN**: Executes a standalone statement or declaration: `"%s of TRANSFER is polymorphic"_warn_en_US, which);`.
  **L2241 CN**: 执行一条独立语句或声明：`"%s of TRANSFER is polymorphic"_warn_en_US, which);`。
- **L2242 EN**: Transitions from the previous branch into an `else if` condition.
  **L2242 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2243 EN**: Continues logic associated with callable symbol `category`.
  **L2243 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L2244 EN**: Starts a function, method, lambda, or structured scope: `context.ShouldWarn(common::UsageWarning::PointerComponentTransferArg)) {`.
  **L2244 CN**: 开始一个函数、方法、lambda 或结构化作用域：`context.ShouldWarn(common::UsageWarning::PointerComponentTransferArg)) {`。
- **L2245 EN**: Executes a call or declaration centered on `directs{type.GetDerivedTypeSpec`.
  **L2245 CN**: 执行以 `directs{type.GetDerivedTypeSpec` 为核心的调用或声明。
- **L2246 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2246 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2247 EN**: Starts a function, method, lambda, or structured scope: `bad != directs.end()) {`.
  **L2247 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bad != directs.end()) {`。
- **L2248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::WarnWithDeclaration(context.foldingContext(), *bad,`.
  **L2248 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::WarnWithDeclaration(context.foldingContext(), *bad,`。
- **L2249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::UsageWarning::PointerComponentTransferArg,`.
  **L2249 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::UsageWarning::PointerComponentTransferArg,`。
- **L2250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s of TRANSFER contains allocatable or pointer component %s"_warn_en_US,`.
  **L2250 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s of TRANSFER contains allocatable or pointer component %s"_warn_en_US,`。
- **L2251 EN**: Executes a call or declaration centered on `bad.BuildResultDesignatorName`.
  **L2251 CN**: 执行以 `bad.BuildResultDesignatorName` 为核心的调用或声明。
- **L2252 EN**: Closes the current lexical scope or compound statement.
  **L2252 CN**: 结束当前词法作用域或复合语句块。
- **L2253 EN**: Closes the current lexical scope or compound statement.
  **L2253 CN**: 结束当前词法作用域或复合语句块。
- **L2254 EN**: Closes the current lexical scope or compound statement.
  **L2254 CN**: 结束当前词法作用域或复合语句块。
- **L2255 EN**: Blank line separating nearby declarations or logic blocks.
  **L2255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckTransfer(evaluate::ActualArguments &arguments,`.
  **L2256 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckTransfer(evaluate::ActualArguments &arguments,`。

### Lines 2257-2280

````cpp
    SemanticsContext &context, const Scope *scope) {
  evaluate::FoldingContext &foldingContext{context.foldingContext()};
  parser::ContextualMessages &messages{foldingContext.messages()};
  if (arguments.size() >= 2) {
    if (auto source{characteristics::TypeAndShape::Characterize(
            arguments[0], foldingContext)}) {
      CheckTransferOperandType(context, source->type(), "Source");
      if (auto mold{characteristics::TypeAndShape::Characterize(
              arguments[1], foldingContext)}) {
        CheckTransferOperandType(context, mold->type(), "Mold");
        if (mold->Rank() > 0 &&
            evaluate::ToInt64(
                evaluate::Fold(foldingContext,
                    mold->MeasureElementSizeInBytes(foldingContext, false)))
                    .value_or(1) == 0) {
          if (auto sourceSize{evaluate::ToInt64(evaluate::Fold(foldingContext,
                  source->MeasureSizeInBytes(foldingContext)))}) {
            if (*sourceSize > 0) {
              messages.Say(
                  "Element size of MOLD= array may not be zero when SOURCE= is not empty"_err_en_US);
            }
          } else {
            foldingContext.Warn(common::UsageWarning::VoidMold,
                "Element size of MOLD= array may not be zero unless SOURCE= is empty"_warn_en_US);
````
- **L2257 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const Scope *scope) {`.
  **L2257 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const Scope *scope) {`。
- **L2258 EN**: Executes a call or declaration centered on `&foldingContext{context.foldingContext`.
  **L2258 CN**: 执行以 `&foldingContext{context.foldingContext` 为核心的调用或声明。
- **L2259 EN**: Executes a call or declaration centered on `&messages{foldingContext.messages`.
  **L2259 CN**: 执行以 `&messages{foldingContext.messages` 为核心的调用或声明。
- **L2260 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2260 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2261 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2261 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2262 EN**: Continues the surrounding expression or declaration: `arguments[0], foldingContext)}) {`.
  **L2262 CN**: 继续构造周围的表达式或声明：`arguments[0], foldingContext)}) {`。
- **L2263 EN**: Executes a call or declaration centered on `CheckTransferOperandType`.
  **L2263 CN**: 执行以 `CheckTransferOperandType` 为核心的调用或声明。
- **L2264 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2264 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2265 EN**: Continues the surrounding expression or declaration: `arguments[1], foldingContext)}) {`.
  **L2265 CN**: 继续构造周围的表达式或声明：`arguments[1], foldingContext)}) {`。
- **L2266 EN**: Executes a call or declaration centered on `CheckTransferOperandType`.
  **L2266 CN**: 执行以 `CheckTransferOperandType` 为核心的调用或声明。
- **L2267 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2267 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2268 EN**: Continues logic associated with callable symbol `ToInt64`.
  **L2268 CN**: 继续与可调用符号 `ToInt64` 相关的逻辑。
- **L2269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::Fold(foldingContext,`.
  **L2269 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::Fold(foldingContext,`。
- **L2270 EN**: Continues logic associated with callable symbol `MeasureElementSizeInBytes`.
  **L2270 CN**: 继续与可调用符号 `MeasureElementSizeInBytes` 相关的逻辑。
- **L2271 EN**: Starts a function, method, lambda, or structured scope: `.value_or(1) == 0) {`.
  **L2271 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.value_or(1) == 0) {`。
- **L2272 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2272 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2273 EN**: Starts a function, method, lambda, or structured scope: `source->MeasureSizeInBytes(foldingContext)))}) {`.
  **L2273 CN**: 开始一个函数、方法、lambda 或结构化作用域：`source->MeasureSizeInBytes(foldingContext)))}) {`。
- **L2274 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2274 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2275 EN**: Continues logic associated with callable symbol `Say`.
  **L2275 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2276 EN**: Executes a standalone statement or declaration: `"Element size of MOLD= array may not be zero when SOURCE= is not empty"_err_en_US);`.
  **L2276 CN**: 执行一条独立语句或声明：`"Element size of MOLD= array may not be zero when SOURCE= is not empty"_err_en_US);`。
- **L2277 EN**: Closes the current lexical scope or compound statement.
  **L2277 CN**: 结束当前词法作用域或复合语句块。
- **L2278 EN**: Transitions from the previous branch into the alternative path.
  **L2278 CN**: 从前一个分支过渡到备选路径。
- **L2279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::VoidMold,`.
  **L2279 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::VoidMold,`。
- **L2280 EN**: Executes a standalone statement or declaration: `"Element size of MOLD= array may not be zero unless SOURCE= is empty"_warn_en_US);`.
  **L2280 CN**: 执行一条独立语句或声明：`"Element size of MOLD= array may not be zero unless SOURCE= is empty"_warn_en_US);`。

### Lines 2281-2304

````cpp
          }
        }
      }
    }
    if (arguments.size() > 2) { // SIZE=
      if (const Symbol *
          whole{UnwrapWholeSymbolOrComponentDataRef(arguments[2])}) {
        if (IsOptional(*whole)) {
          messages.Say(
              "SIZE= argument may not be the optional dummy argument '%s'"_err_en_US,
              whole->name());
        } else if (context.ShouldWarn(
                       common::UsageWarning::TransferSizePresence) &&
            IsAllocatableOrObjectPointer(whole)) {
          foldingContext.Warn(common::UsageWarning::TransferSizePresence,
              "SIZE= argument that is allocatable or pointer must be present at execution; parenthesize to silence this warning"_warn_en_US);
        }
      }
    }
  }
}

static void CheckSpecificIntrinsic(const characteristics::Procedure &proc,
    evaluate::ActualArguments &arguments, SemanticsContext &context,
````
- **L2281 EN**: Closes the current lexical scope or compound statement.
  **L2281 CN**: 结束当前词法作用域或复合语句块。
- **L2282 EN**: Closes the current lexical scope or compound statement.
  **L2282 CN**: 结束当前词法作用域或复合语句块。
- **L2283 EN**: Closes the current lexical scope or compound statement.
  **L2283 CN**: 结束当前词法作用域或复合语句块。
- **L2284 EN**: Closes the current lexical scope or compound statement.
  **L2284 CN**: 结束当前词法作用域或复合语句块。
- **L2285 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2285 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2286 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2286 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2287 EN**: Starts a function, method, lambda, or structured scope: `whole{UnwrapWholeSymbolOrComponentDataRef(arguments[2])}) {`.
  **L2287 CN**: 开始一个函数、方法、lambda 或结构化作用域：`whole{UnwrapWholeSymbolOrComponentDataRef(arguments[2])}) {`。
- **L2288 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2288 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2289 EN**: Continues logic associated with callable symbol `Say`.
  **L2289 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"SIZE= argument may not be the optional dummy argument '%s'"_err_en_US,`.
  **L2290 CN**: 继续一个多行参数列表、初始化器或聚合项：`"SIZE= argument may not be the optional dummy argument '%s'"_err_en_US,`。
- **L2291 EN**: Executes a call or declaration centered on `whole->name`.
  **L2291 CN**: 执行以 `whole->name` 为核心的调用或声明。
- **L2292 EN**: Transitions from the previous branch into an `else if` condition.
  **L2292 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2293 EN**: Continues the surrounding expression or declaration: `common::UsageWarning::TransferSizePresence) &&`.
  **L2293 CN**: 继续构造周围的表达式或声明：`common::UsageWarning::TransferSizePresence) &&`。
- **L2294 EN**: Starts a function, method, lambda, or structured scope: `IsAllocatableOrObjectPointer(whole)) {`.
  **L2294 CN**: 开始一个函数、方法、lambda 或结构化作用域：`IsAllocatableOrObjectPointer(whole)) {`。
- **L2295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `foldingContext.Warn(common::UsageWarning::TransferSizePresence,`.
  **L2295 CN**: 继续一个多行参数列表、初始化器或聚合项：`foldingContext.Warn(common::UsageWarning::TransferSizePresence,`。
- **L2296 EN**: Executes a standalone statement or declaration: `"SIZE= argument that is allocatable or pointer must be present at execution; parenthesize to silence this warning"_warn_en_US);`.
  **L2296 CN**: 执行一条独立语句或声明：`"SIZE= argument that is allocatable or pointer must be present at execution; parenthesize to silence this warning"_warn_en_US);`。
- **L2297 EN**: Closes the current lexical scope or compound statement.
  **L2297 CN**: 结束当前词法作用域或复合语句块。
- **L2298 EN**: Closes the current lexical scope or compound statement.
  **L2298 CN**: 结束当前词法作用域或复合语句块。
- **L2299 EN**: Closes the current lexical scope or compound statement.
  **L2299 CN**: 结束当前词法作用域或复合语句块。
- **L2300 EN**: Closes the current lexical scope or compound statement.
  **L2300 CN**: 结束当前词法作用域或复合语句块。
- **L2301 EN**: Closes the current lexical scope or compound statement.
  **L2301 CN**: 结束当前词法作用域或复合语句块。
- **L2302 EN**: Blank line separating nearby declarations or logic blocks.
  **L2302 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void CheckSpecificIntrinsic(const characteristics::Procedure &proc,`.
  **L2303 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void CheckSpecificIntrinsic(const characteristics::Procedure &proc,`。
- **L2304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::ActualArguments &arguments, SemanticsContext &context,`.
  **L2304 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::ActualArguments &arguments, SemanticsContext &context,`。

### Lines 2305-2328

````cpp
    const Scope *scope, const evaluate::SpecificIntrinsic &intrinsic) {
  if (intrinsic.name == "associated") {
    CheckAssociated(arguments, context, scope);
  } else if (intrinsic.name == "co_reduce") {
    CheckCoReduce(arguments, context.foldingContext());
  } else if (intrinsic.name == "date_and_time") {
    CheckDate_And_Time(arguments, context.foldingContext());
  } else if (intrinsic.name == "event_query") {
    CheckEvent_Query(arguments, context.foldingContext());
  } else if (intrinsic.name == "image_index") {
    CheckImage_Index(arguments, context.foldingContext().messages());
  } else if (intrinsic.name == "max" || intrinsic.name == "min") {
    CheckMaxMin(proc, arguments, context.foldingContext().messages());
  } else if (intrinsic.name == "move_alloc") {
    CheckMove_Alloc(arguments, context.foldingContext().messages());
  } else if (intrinsic.name == "present") {
    CheckPresent(arguments, context.foldingContext().messages());
  } else if (intrinsic.name == "reduce") {
    CheckReduce(arguments, context.foldingContext());
  } else if (intrinsic.name == "transfer") {
    CheckTransfer(arguments, context, scope);
  } else if (intrinsic.name == "free") {
    CheckFree(arguments, context.foldingContext().messages());
  }
````
- **L2305 EN**: Continues the surrounding expression or declaration: `const Scope *scope, const evaluate::SpecificIntrinsic &intrinsic) {`.
  **L2305 CN**: 继续构造周围的表达式或声明：`const Scope *scope, const evaluate::SpecificIntrinsic &intrinsic) {`。
- **L2306 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2306 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2307 EN**: Executes a call or declaration centered on `CheckAssociated`.
  **L2307 CN**: 执行以 `CheckAssociated` 为核心的调用或声明。
- **L2308 EN**: Transitions from the previous branch into an `else if` condition.
  **L2308 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2309 EN**: Executes a call or declaration centered on `CheckCoReduce`.
  **L2309 CN**: 执行以 `CheckCoReduce` 为核心的调用或声明。
- **L2310 EN**: Transitions from the previous branch into an `else if` condition.
  **L2310 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2311 EN**: Executes a call or declaration centered on `CheckDate_And_Time`.
  **L2311 CN**: 执行以 `CheckDate_And_Time` 为核心的调用或声明。
- **L2312 EN**: Transitions from the previous branch into an `else if` condition.
  **L2312 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2313 EN**: Executes a call or declaration centered on `CheckEvent_Query`.
  **L2313 CN**: 执行以 `CheckEvent_Query` 为核心的调用或声明。
- **L2314 EN**: Transitions from the previous branch into an `else if` condition.
  **L2314 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2315 EN**: Executes a call or declaration centered on `CheckImage_Index`.
  **L2315 CN**: 执行以 `CheckImage_Index` 为核心的调用或声明。
- **L2316 EN**: Transitions from the previous branch into an `else if` condition.
  **L2316 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2317 EN**: Executes a call or declaration centered on `CheckMaxMin`.
  **L2317 CN**: 执行以 `CheckMaxMin` 为核心的调用或声明。
- **L2318 EN**: Transitions from the previous branch into an `else if` condition.
  **L2318 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2319 EN**: Executes a call or declaration centered on `CheckMove_Alloc`.
  **L2319 CN**: 执行以 `CheckMove_Alloc` 为核心的调用或声明。
- **L2320 EN**: Transitions from the previous branch into an `else if` condition.
  **L2320 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2321 EN**: Executes a call or declaration centered on `CheckPresent`.
  **L2321 CN**: 执行以 `CheckPresent` 为核心的调用或声明。
- **L2322 EN**: Transitions from the previous branch into an `else if` condition.
  **L2322 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2323 EN**: Executes a call or declaration centered on `CheckReduce`.
  **L2323 CN**: 执行以 `CheckReduce` 为核心的调用或声明。
- **L2324 EN**: Transitions from the previous branch into an `else if` condition.
  **L2324 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2325 EN**: Executes a call or declaration centered on `CheckTransfer`.
  **L2325 CN**: 执行以 `CheckTransfer` 为核心的调用或声明。
- **L2326 EN**: Transitions from the previous branch into an `else if` condition.
  **L2326 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2327 EN**: Executes a call or declaration centered on `CheckFree`.
  **L2327 CN**: 执行以 `CheckFree` 为核心的调用或声明。
- **L2328 EN**: Closes the current lexical scope or compound statement.
  **L2328 CN**: 结束当前词法作用域或复合语句块。

### Lines 2329-2352

````cpp
}

parser::Messages CheckExplicitInterface(const characteristics::Procedure &proc,
    evaluate::ActualArguments &actuals, SemanticsContext &context,
    const Scope *scope, const evaluate::SpecificIntrinsic *intrinsic,
    bool allowActualArgumentConversions, bool extentErrors,
    bool ignoreImplicitVsExplicit) {
  evaluate::FoldingContext &foldingContext{context.foldingContext()};
  parser::ContextualMessages &messages{foldingContext.messages()};
  parser::Messages buffer;
  auto restorer{messages.SetMessages(buffer)};
  RearrangeArguments(proc, actuals, messages);
  if (!buffer.empty()) {
    return buffer;
  }
  int index{0};
  for (auto &actual : actuals) {
    const auto &dummy{proc.dummyArguments.at(index++)};
    if (actual) {
      CheckExplicitInterfaceArg(*actual, dummy, proc, context, scope, intrinsic,
          allowActualArgumentConversions, extentErrors,
          ignoreImplicitVsExplicit);
    } else if (!dummy.IsOptional()) {
      if (dummy.name.empty()) {
````
- **L2329 EN**: Closes the current lexical scope or compound statement.
  **L2329 CN**: 结束当前词法作用域或复合语句块。
- **L2330 EN**: Blank line separating nearby declarations or logic blocks.
  **L2330 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::Messages CheckExplicitInterface(const characteristics::Procedure &proc,`.
  **L2331 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::Messages CheckExplicitInterface(const characteristics::Procedure &proc,`。
- **L2332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::ActualArguments &actuals, SemanticsContext &context,`.
  **L2332 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::ActualArguments &actuals, SemanticsContext &context,`。
- **L2333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Scope *scope, const evaluate::SpecificIntrinsic *intrinsic,`.
  **L2333 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Scope *scope, const evaluate::SpecificIntrinsic *intrinsic,`。
- **L2334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool allowActualArgumentConversions, bool extentErrors,`.
  **L2334 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool allowActualArgumentConversions, bool extentErrors,`。
- **L2335 EN**: Continues the surrounding expression or declaration: `bool ignoreImplicitVsExplicit) {`.
  **L2335 CN**: 继续构造周围的表达式或声明：`bool ignoreImplicitVsExplicit) {`。
- **L2336 EN**: Executes a call or declaration centered on `&foldingContext{context.foldingContext`.
  **L2336 CN**: 执行以 `&foldingContext{context.foldingContext` 为核心的调用或声明。
- **L2337 EN**: Executes a call or declaration centered on `&messages{foldingContext.messages`.
  **L2337 CN**: 执行以 `&messages{foldingContext.messages` 为核心的调用或声明。
- **L2338 EN**: Executes a standalone statement or declaration: `parser::Messages buffer;`.
  **L2338 CN**: 执行一条独立语句或声明：`parser::Messages buffer;`。
- **L2339 EN**: Executes a call or declaration centered on `restorer{messages.SetMessages`.
  **L2339 CN**: 执行以 `restorer{messages.SetMessages` 为核心的调用或声明。
- **L2340 EN**: Executes a call or declaration centered on `RearrangeArguments`.
  **L2340 CN**: 执行以 `RearrangeArguments` 为核心的调用或声明。
- **L2341 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2341 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2342 EN**: Returns from the current function with `buffer`.
  **L2342 CN**: 以 `buffer` 从当前函数返回。
- **L2343 EN**: Closes the current lexical scope or compound statement.
  **L2343 CN**: 结束当前词法作用域或复合语句块。
- **L2344 EN**: Executes a standalone statement or declaration: `int index{0};`.
  **L2344 CN**: 执行一条独立语句或声明：`int index{0};`。
- **L2345 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2345 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2346 EN**: Executes a call or declaration centered on `&dummy{proc.dummyArguments.at`.
  **L2346 CN**: 执行以 `&dummy{proc.dummyArguments.at` 为核心的调用或声明。
- **L2347 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2347 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckExplicitInterfaceArg(*actual, dummy, proc, context, scope, intrinsic,`.
  **L2348 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckExplicitInterfaceArg(*actual, dummy, proc, context, scope, intrinsic,`。
- **L2349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `allowActualArgumentConversions, extentErrors,`.
  **L2349 CN**: 继续一个多行参数列表、初始化器或聚合项：`allowActualArgumentConversions, extentErrors,`。
- **L2350 EN**: Executes a standalone statement or declaration: `ignoreImplicitVsExplicit);`.
  **L2350 CN**: 执行一条独立语句或声明：`ignoreImplicitVsExplicit);`。
- **L2351 EN**: Transitions from the previous branch into an `else if` condition.
  **L2351 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2352 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2352 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2353-2376

````cpp
        messages.Say(
            "Dummy argument #%d is not OPTIONAL and is not associated with "
            "an actual argument in this procedure reference"_err_en_US,
            index);
      } else {
        messages.Say("Dummy argument '%s=' (#%d) is not OPTIONAL and is not "
                     "associated with an actual argument in this procedure "
                     "reference"_err_en_US,
            dummy.name, index);
      }
    }
  }
  if (proc.IsElemental() && !buffer.AnyFatalError()) {
    CheckElementalConformance(messages, proc, actuals, foldingContext);
  }
  if (intrinsic) {
    CheckSpecificIntrinsic(proc, actuals, context, scope, *intrinsic);
  }
  return buffer;
}

bool CheckInterfaceForGeneric(const characteristics::Procedure &proc,
    evaluate::ActualArguments &actuals, SemanticsContext &context,
    bool allowActualArgumentConversions) {
````
- **L2353 EN**: Continues logic associated with callable symbol `Say`.
  **L2353 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2354 EN**: Continues the surrounding expression or declaration: `"Dummy argument #%d is not OPTIONAL and is not associated with "`.
  **L2354 CN**: 继续构造周围的表达式或声明：`"Dummy argument #%d is not OPTIONAL and is not associated with "`。
- **L2355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"an actual argument in this procedure reference"_err_en_US,`.
  **L2355 CN**: 继续一个多行参数列表、初始化器或聚合项：`"an actual argument in this procedure reference"_err_en_US,`。
- **L2356 EN**: Executes a standalone statement or declaration: `index);`.
  **L2356 CN**: 执行一条独立语句或声明：`index);`。
- **L2357 EN**: Transitions from the previous branch into the alternative path.
  **L2357 CN**: 从前一个分支过渡到备选路径。
- **L2358 EN**: Continues logic associated with callable symbol `Say`.
  **L2358 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2359 EN**: Continues the surrounding expression or declaration: `"associated with an actual argument in this procedure "`.
  **L2359 CN**: 继续构造周围的表达式或声明：`"associated with an actual argument in this procedure "`。
- **L2360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"reference"_err_en_US,`.
  **L2360 CN**: 继续一个多行参数列表、初始化器或聚合项：`"reference"_err_en_US,`。
- **L2361 EN**: Executes a standalone statement or declaration: `dummy.name, index);`.
  **L2361 CN**: 执行一条独立语句或声明：`dummy.name, index);`。
- **L2362 EN**: Closes the current lexical scope or compound statement.
  **L2362 CN**: 结束当前词法作用域或复合语句块。
- **L2363 EN**: Closes the current lexical scope or compound statement.
  **L2363 CN**: 结束当前词法作用域或复合语句块。
- **L2364 EN**: Closes the current lexical scope or compound statement.
  **L2364 CN**: 结束当前词法作用域或复合语句块。
- **L2365 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2365 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2366 EN**: Executes a call or declaration centered on `CheckElementalConformance`.
  **L2366 CN**: 执行以 `CheckElementalConformance` 为核心的调用或声明。
- **L2367 EN**: Closes the current lexical scope or compound statement.
  **L2367 CN**: 结束当前词法作用域或复合语句块。
- **L2368 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2368 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2369 EN**: Executes a call or declaration centered on `CheckSpecificIntrinsic`.
  **L2369 CN**: 执行以 `CheckSpecificIntrinsic` 为核心的调用或声明。
- **L2370 EN**: Closes the current lexical scope or compound statement.
  **L2370 CN**: 结束当前词法作用域或复合语句块。
- **L2371 EN**: Returns from the current function with `buffer`.
  **L2371 CN**: 以 `buffer` 从当前函数返回。
- **L2372 EN**: Closes the current lexical scope or compound statement.
  **L2372 CN**: 结束当前词法作用域或复合语句块。
- **L2373 EN**: Blank line separating nearby declarations or logic blocks.
  **L2373 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool CheckInterfaceForGeneric(const characteristics::Procedure &proc,`.
  **L2374 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool CheckInterfaceForGeneric(const characteristics::Procedure &proc,`。
- **L2375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::ActualArguments &actuals, SemanticsContext &context,`.
  **L2375 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::ActualArguments &actuals, SemanticsContext &context,`。
- **L2376 EN**: Continues the surrounding expression or declaration: `bool allowActualArgumentConversions) {`.
  **L2376 CN**: 继续构造周围的表达式或声明：`bool allowActualArgumentConversions) {`。

### Lines 2377-2400

````cpp
  return proc.HasExplicitInterface() &&
      !CheckExplicitInterface(proc, actuals, context, nullptr, nullptr,
          allowActualArgumentConversions, /*extentErrors=*/false,
          /*ignoreImplicitVsExplicit=*/false)
           .AnyFatalError();
}

bool CheckArgumentIsConstantExprInRange(
    const evaluate::ActualArguments &actuals, int index, int lowerBound,
    int upperBound, parser::ContextualMessages &messages) {
  CHECK(index >= 0 && static_cast<unsigned>(index) < actuals.size());

  const std::optional<evaluate::ActualArgument> &argOptional{actuals[index]};
  if (!argOptional) {
    DIE("Actual argument should have value");
    return false;
  }

  const evaluate::ActualArgument &arg{argOptional.value()};
  const evaluate::Expr<evaluate::SomeType> *argExpr{arg.UnwrapExpr()};
  CHECK(argExpr != nullptr);

  if (!IsConstantExpr(*argExpr)) {
    messages.Say("Actual argument #%d must be a constant expression"_err_en_US,
````
- **L2377 EN**: Returns from the current function with `proc.HasExplicitInterface() &&`.
  **L2377 CN**: 以 `proc.HasExplicitInterface() &&` 从当前函数返回。
- **L2378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!CheckExplicitInterface(proc, actuals, context, nullptr, nullptr,`.
  **L2378 CN**: 继续一个多行参数列表、初始化器或聚合项：`!CheckExplicitInterface(proc, actuals, context, nullptr, nullptr,`。
- **L2379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `allowActualArgumentConversions, /*extentErrors=*/false,`.
  **L2379 CN**: 继续一个多行参数列表、初始化器或聚合项：`allowActualArgumentConversions, /*extentErrors=*/false,`。
- **L2380 EN**: Comment explains nearby logic, intent, or metadata: `ignoreImplicitVsExplicit=*/false)`.
  **L2380 CN**: 注释说明附近代码的逻辑、意图或元数据：`ignoreImplicitVsExplicit=*/false)`。
- **L2381 EN**: Executes a call or declaration centered on `.AnyFatalError`.
  **L2381 CN**: 执行以 `.AnyFatalError` 为核心的调用或声明。
- **L2382 EN**: Closes the current lexical scope or compound statement.
  **L2382 CN**: 结束当前词法作用域或复合语句块。
- **L2383 EN**: Blank line separating nearby declarations or logic blocks.
  **L2383 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2384 EN**: Continues logic associated with callable symbol `CheckArgumentIsConstantExprInRange`.
  **L2384 CN**: 继续与可调用符号 `CheckArgumentIsConstantExprInRange` 相关的逻辑。
- **L2385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const evaluate::ActualArguments &actuals, int index, int lowerBound,`.
  **L2385 CN**: 继续一个多行参数列表、初始化器或聚合项：`const evaluate::ActualArguments &actuals, int index, int lowerBound,`。
- **L2386 EN**: Continues the surrounding expression or declaration: `int upperBound, parser::ContextualMessages &messages) {`.
  **L2386 CN**: 继续构造周围的表达式或声明：`int upperBound, parser::ContextualMessages &messages) {`。
- **L2387 EN**: Executes a call or declaration centered on `CHECK`.
  **L2387 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2388 EN**: Blank line separating nearby declarations or logic blocks.
  **L2388 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2389 EN**: Executes a standalone statement or declaration: `const std::optional<evaluate::ActualArgument> &argOptional{actuals[index]};`.
  **L2389 CN**: 执行一条独立语句或声明：`const std::optional<evaluate::ActualArgument> &argOptional{actuals[index]};`。
- **L2390 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2390 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2391 EN**: Executes a call or declaration centered on `DIE`.
  **L2391 CN**: 执行以 `DIE` 为核心的调用或声明。
- **L2392 EN**: Returns from the current function with `false`.
  **L2392 CN**: 以 `false` 从当前函数返回。
- **L2393 EN**: Closes the current lexical scope or compound statement.
  **L2393 CN**: 结束当前词法作用域或复合语句块。
- **L2394 EN**: Blank line separating nearby declarations or logic blocks.
  **L2394 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2395 EN**: Executes a call or declaration centered on `&arg{argOptional.value`.
  **L2395 CN**: 执行以 `&arg{argOptional.value` 为核心的调用或声明。
- **L2396 EN**: Executes a call or declaration centered on `*argExpr{arg.UnwrapExpr`.
  **L2396 CN**: 执行以 `*argExpr{arg.UnwrapExpr` 为核心的调用或声明。
- **L2397 EN**: Executes a call or declaration centered on `CHECK`.
  **L2397 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2398 EN**: Blank line separating nearby declarations or logic blocks.
  **L2398 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2399 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2399 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say("Actual argument #%d must be a constant expression"_err_en_US,`.
  **L2400 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say("Actual argument #%d must be a constant expression"_err_en_US,`。

### Lines 2401-2424

````cpp
        index + 1);
    return false;
  }

  // This does not imply that the kind of the argument is 8. The kind
  // for the intrinsic's argument should have been check prior. This is just
  // a conversion so that we can read the constant value.
  auto scalarValue{evaluate::ToInt64(argExpr)};
  CHECK(scalarValue.has_value());

  if (*scalarValue < lowerBound || *scalarValue > upperBound) {
    messages.Say(
        "Argument #%d must be a constant expression in range %d to %d"_err_en_US,
        index + 1, lowerBound, upperBound);
    return false;
  }
  return true;
}

bool CheckPPCIntrinsic(const Symbol &generic, const Symbol &specific,
    const evaluate::ActualArguments &actuals,
    evaluate::FoldingContext &context) {
  parser::ContextualMessages &messages{context.messages()};

````
- **L2401 EN**: Executes a standalone statement or declaration: `index + 1);`.
  **L2401 CN**: 执行一条独立语句或声明：`index + 1);`。
- **L2402 EN**: Returns from the current function with `false`.
  **L2402 CN**: 以 `false` 从当前函数返回。
- **L2403 EN**: Closes the current lexical scope or compound statement.
  **L2403 CN**: 结束当前词法作用域或复合语句块。
- **L2404 EN**: Blank line separating nearby declarations or logic blocks.
  **L2404 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2405 EN**: Comment explains nearby logic, intent, or metadata: `This does not imply that the kind of the argument is 8. The kind`.
  **L2405 CN**: 注释说明附近代码的逻辑、意图或元数据：`This does not imply that the kind of the argument is 8. The kind`。
- **L2406 EN**: Comment explains nearby logic, intent, or metadata: `for the intrinsic's argument should have been check prior. This is just`.
  **L2406 CN**: 注释说明附近代码的逻辑、意图或元数据：`for the intrinsic's argument should have been check prior. This is just`。
- **L2407 EN**: Comment explains nearby logic, intent, or metadata: `a conversion so that we can read the constant value.`.
  **L2407 CN**: 注释说明附近代码的逻辑、意图或元数据：`a conversion so that we can read the constant value.`。
- **L2408 EN**: Executes a call or declaration centered on `scalarValue{evaluate::ToInt64`.
  **L2408 CN**: 执行以 `scalarValue{evaluate::ToInt64` 为核心的调用或声明。
- **L2409 EN**: Executes a call or declaration centered on `CHECK`.
  **L2409 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L2410 EN**: Blank line separating nearby declarations or logic blocks.
  **L2410 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2411 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2411 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2412 EN**: Continues logic associated with callable symbol `Say`.
  **L2412 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Argument #%d must be a constant expression in range %d to %d"_err_en_US,`.
  **L2413 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Argument #%d must be a constant expression in range %d to %d"_err_en_US,`。
- **L2414 EN**: Executes a standalone statement or declaration: `index + 1, lowerBound, upperBound);`.
  **L2414 CN**: 执行一条独立语句或声明：`index + 1, lowerBound, upperBound);`。
- **L2415 EN**: Returns from the current function with `false`.
  **L2415 CN**: 以 `false` 从当前函数返回。
- **L2416 EN**: Closes the current lexical scope or compound statement.
  **L2416 CN**: 结束当前词法作用域或复合语句块。
- **L2417 EN**: Returns from the current function with `true`.
  **L2417 CN**: 以 `true` 从当前函数返回。
- **L2418 EN**: Closes the current lexical scope or compound statement.
  **L2418 CN**: 结束当前词法作用域或复合语句块。
- **L2419 EN**: Blank line separating nearby declarations or logic blocks.
  **L2419 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool CheckPPCIntrinsic(const Symbol &generic, const Symbol &specific,`.
  **L2420 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool CheckPPCIntrinsic(const Symbol &generic, const Symbol &specific,`。
- **L2421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const evaluate::ActualArguments &actuals,`.
  **L2421 CN**: 继续一个多行参数列表、初始化器或聚合项：`const evaluate::ActualArguments &actuals,`。
- **L2422 EN**: Continues the surrounding expression or declaration: `evaluate::FoldingContext &context) {`.
  **L2422 CN**: 继续构造周围的表达式或声明：`evaluate::FoldingContext &context) {`。
- **L2423 EN**: Executes a call or declaration centered on `&messages{context.messages`.
  **L2423 CN**: 执行以 `&messages{context.messages` 为核心的调用或声明。
- **L2424 EN**: Blank line separating nearby declarations or logic blocks.
  **L2424 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2425-2448

````cpp
  if (specific.name() == "__ppc_mtfsf") {
    return CheckArgumentIsConstantExprInRange(actuals, 0, 0, 7, messages);
  }
  if (specific.name() == "__ppc_mtfsfi") {
    return CheckArgumentIsConstantExprInRange(actuals, 0, 0, 7, messages) &&
        CheckArgumentIsConstantExprInRange(actuals, 1, 0, 15, messages);
  }
  if (specific.name().ToString().compare(0, 14, "__ppc_vec_sld_") == 0) {
    return CheckArgumentIsConstantExprInRange(actuals, 2, 0, 15, messages);
  }
  if (specific.name().ToString().compare(0, 15, "__ppc_vec_sldw_") == 0) {
    return CheckArgumentIsConstantExprInRange(actuals, 2, 0, 3, messages);
  }
  if (specific.name().ToString().compare(0, 14, "__ppc_vec_ctf_") == 0) {
    return CheckArgumentIsConstantExprInRange(actuals, 1, 0, 31, messages);
  }
  if (specific.name().ToString().compare(0, 16, "__ppc_vec_permi_") == 0) {
    return CheckArgumentIsConstantExprInRange(actuals, 2, 0, 3, messages);
  }
  if (specific.name().ToString().compare(0, 21, "__ppc_vec_splat_s32__") == 0) {
    return CheckArgumentIsConstantExprInRange(actuals, 0, -16, 15, messages);
  }
  if (specific.name().ToString().compare(0, 16, "__ppc_vec_splat_") == 0) {
    // The value of arg2 in vec_splat must be a constant expression that is
````
- **L2425 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2425 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2426 EN**: Returns from the current function with `CheckArgumentIsConstantExprInRange(actuals, 0, 0, 7, messages)`.
  **L2426 CN**: 以 `CheckArgumentIsConstantExprInRange(actuals, 0, 0, 7, messages)` 从当前函数返回。
- **L2427 EN**: Closes the current lexical scope or compound statement.
  **L2427 CN**: 结束当前词法作用域或复合语句块。
- **L2428 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2428 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2429 EN**: Returns from the current function with `CheckArgumentIsConstantExprInRange(actuals, 0, 0, 7, messages) &&`.
  **L2429 CN**: 以 `CheckArgumentIsConstantExprInRange(actuals, 0, 0, 7, messages) &&` 从当前函数返回。
- **L2430 EN**: Executes a call or declaration centered on `CheckArgumentIsConstantExprInRange`.
  **L2430 CN**: 执行以 `CheckArgumentIsConstantExprInRange` 为核心的调用或声明。
- **L2431 EN**: Closes the current lexical scope or compound statement.
  **L2431 CN**: 结束当前词法作用域或复合语句块。
- **L2432 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2432 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2433 EN**: Returns from the current function with `CheckArgumentIsConstantExprInRange(actuals, 2, 0, 15, messages)`.
  **L2433 CN**: 以 `CheckArgumentIsConstantExprInRange(actuals, 2, 0, 15, messages)` 从当前函数返回。
- **L2434 EN**: Closes the current lexical scope or compound statement.
  **L2434 CN**: 结束当前词法作用域或复合语句块。
- **L2435 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2435 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2436 EN**: Returns from the current function with `CheckArgumentIsConstantExprInRange(actuals, 2, 0, 3, messages)`.
  **L2436 CN**: 以 `CheckArgumentIsConstantExprInRange(actuals, 2, 0, 3, messages)` 从当前函数返回。
- **L2437 EN**: Closes the current lexical scope or compound statement.
  **L2437 CN**: 结束当前词法作用域或复合语句块。
- **L2438 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2438 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2439 EN**: Returns from the current function with `CheckArgumentIsConstantExprInRange(actuals, 1, 0, 31, messages)`.
  **L2439 CN**: 以 `CheckArgumentIsConstantExprInRange(actuals, 1, 0, 31, messages)` 从当前函数返回。
- **L2440 EN**: Closes the current lexical scope or compound statement.
  **L2440 CN**: 结束当前词法作用域或复合语句块。
- **L2441 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2441 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2442 EN**: Returns from the current function with `CheckArgumentIsConstantExprInRange(actuals, 2, 0, 3, messages)`.
  **L2442 CN**: 以 `CheckArgumentIsConstantExprInRange(actuals, 2, 0, 3, messages)` 从当前函数返回。
- **L2443 EN**: Closes the current lexical scope or compound statement.
  **L2443 CN**: 结束当前词法作用域或复合语句块。
- **L2444 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2444 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2445 EN**: Returns from the current function with `CheckArgumentIsConstantExprInRange(actuals, 0, -16, 15, messages)`.
  **L2445 CN**: 以 `CheckArgumentIsConstantExprInRange(actuals, 0, -16, 15, messages)` 从当前函数返回。
- **L2446 EN**: Closes the current lexical scope or compound statement.
  **L2446 CN**: 结束当前词法作用域或复合语句块。
- **L2447 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2447 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2448 EN**: Comment explains nearby logic, intent, or metadata: `The value of arg2 in vec_splat must be a constant expression that is`.
  **L2448 CN**: 注释说明附近代码的逻辑、意图或元数据：`The value of arg2 in vec_splat must be a constant expression that is`。

### Lines 2449-2472

````cpp
    // greater than or equal to 0, and less than the number of elements in arg1.
    auto *expr{actuals[0].value().UnwrapExpr()};
    auto type{characteristics::TypeAndShape::Characterize(*expr, context)};
    assert(type && "unknown type");
    const auto *derived{evaluate::GetDerivedTypeSpec(type.value().type())};
    if (derived && derived->IsVectorType()) {
      for (const auto &pair : derived->parameters()) {
        if (pair.first == "element_kind") {
          auto vecElemKind{Fortran::evaluate::ToInt64(pair.second.GetExplicit())
                               .value_or(0)};
          auto numElem{vecElemKind == 0 ? 0 : (16 / vecElemKind)};
          return CheckArgumentIsConstantExprInRange(
              actuals, 1, 0, numElem - 1, messages);
        }
      }
    } else
      assert(false && "vector type is expected");
  }
  return false;
}

bool CheckWindowsIntrinsic(
    const Symbol &intrinsic, evaluate::FoldingContext &foldingContext) {
  parser::ContextualMessages &messages{foldingContext.messages()};
````
- **L2449 EN**: Comment explains nearby logic, intent, or metadata: `greater than or equal to 0, and less than the number of elements in arg1.`.
  **L2449 CN**: 注释说明附近代码的逻辑、意图或元数据：`greater than or equal to 0, and less than the number of elements in arg1.`。
- **L2450 EN**: Executes a call or declaration centered on `*expr{actuals[0].value`.
  **L2450 CN**: 执行以 `*expr{actuals[0].value` 为核心的调用或声明。
- **L2451 EN**: Executes a call or declaration centered on `type{characteristics::TypeAndShape::Characterize`.
  **L2451 CN**: 执行以 `type{characteristics::TypeAndShape::Characterize` 为核心的调用或声明。
- **L2452 EN**: Checks an internal invariant in debug builds.
  **L2452 CN**: 在调试构建中检查内部不变式。
- **L2453 EN**: Executes a call or declaration centered on `*derived{evaluate::GetDerivedTypeSpec`.
  **L2453 CN**: 执行以 `*derived{evaluate::GetDerivedTypeSpec` 为核心的调用或声明。
- **L2454 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2454 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2455 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2455 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2456 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2456 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2457 EN**: Continues logic associated with callable symbol `ToInt64`.
  **L2457 CN**: 继续与可调用符号 `ToInt64` 相关的逻辑。
- **L2458 EN**: Executes a call or declaration centered on `.value_or`.
  **L2458 CN**: 执行以 `.value_or` 为核心的调用或声明。
- **L2459 EN**: Executes a call or declaration centered on `:`.
  **L2459 CN**: 执行以 `:` 为核心的调用或声明。
- **L2460 EN**: Returns from the current function with `CheckArgumentIsConstantExprInRange(`.
  **L2460 CN**: 以 `CheckArgumentIsConstantExprInRange(` 从当前函数返回。
- **L2461 EN**: Executes a standalone statement or declaration: `actuals, 1, 0, numElem - 1, messages);`.
  **L2461 CN**: 执行一条独立语句或声明：`actuals, 1, 0, numElem - 1, messages);`。
- **L2462 EN**: Closes the current lexical scope or compound statement.
  **L2462 CN**: 结束当前词法作用域或复合语句块。
- **L2463 EN**: Closes the current lexical scope or compound statement.
  **L2463 CN**: 结束当前词法作用域或复合语句块。
- **L2464 EN**: Transitions from the previous branch into the alternative path.
  **L2464 CN**: 从前一个分支过渡到备选路径。
- **L2465 EN**: Checks an internal invariant in debug builds.
  **L2465 CN**: 在调试构建中检查内部不变式。
- **L2466 EN**: Closes the current lexical scope or compound statement.
  **L2466 CN**: 结束当前词法作用域或复合语句块。
- **L2467 EN**: Returns from the current function with `false`.
  **L2467 CN**: 以 `false` 从当前函数返回。
- **L2468 EN**: Closes the current lexical scope or compound statement.
  **L2468 CN**: 结束当前词法作用域或复合语句块。
- **L2469 EN**: Blank line separating nearby declarations or logic blocks.
  **L2469 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2470 EN**: Continues logic associated with callable symbol `CheckWindowsIntrinsic`.
  **L2470 CN**: 继续与可调用符号 `CheckWindowsIntrinsic` 相关的逻辑。
- **L2471 EN**: Continues the surrounding expression or declaration: `const Symbol &intrinsic, evaluate::FoldingContext &foldingContext) {`.
  **L2471 CN**: 继续构造周围的表达式或声明：`const Symbol &intrinsic, evaluate::FoldingContext &foldingContext) {`。
- **L2472 EN**: Executes a call or declaration centered on `&messages{foldingContext.messages`.
  **L2472 CN**: 执行以 `&messages{foldingContext.messages` 为核心的调用或声明。

### Lines 2473-2496

````cpp
  // TODO: there are other intrinsics that are unsupported on Windows that
  // should be added here.
  if (intrinsic.name() == "getuid") {
    messages.Say(
        "User IDs do not exist on Windows. This function will always return 1"_warn_en_US);
  }
  if (intrinsic.name() == "getgid") {
    messages.Say(
        "Group IDs do not exist on Windows. This function will always return 1"_warn_en_US);
  }
  return true;
}

bool CheckArguments(const characteristics::Procedure &proc,
    evaluate::ActualArguments &actuals, SemanticsContext &context,
    const Scope &scope, bool treatingExternalAsImplicit,
    bool ignoreImplicitVsExplicit,
    const evaluate::SpecificIntrinsic *intrinsic) {
  bool explicitInterface{proc.HasExplicitInterface()};
  evaluate::FoldingContext foldingContext{context.foldingContext()};
  parser::ContextualMessages &messages{foldingContext.messages()};
  bool allowArgumentConversions{true};
  parser::Messages implicitBuffer;
  if (!explicitInterface || treatingExternalAsImplicit) {
````
- **L2473 EN**: Comment records a pending task or caution: `TODO: there are other intrinsics that are unsupported on Windows that`.
  **L2473 CN**: 注释记录待办事项或注意点：`TODO: there are other intrinsics that are unsupported on Windows that`。
- **L2474 EN**: Comment explains nearby logic, intent, or metadata: `should be added here.`.
  **L2474 CN**: 注释说明附近代码的逻辑、意图或元数据：`should be added here.`。
- **L2475 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2475 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2476 EN**: Continues logic associated with callable symbol `Say`.
  **L2476 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2477 EN**: Executes a standalone statement or declaration: `"User IDs do not exist on Windows. This function will always return 1"_warn_en_US);`.
  **L2477 CN**: 执行一条独立语句或声明：`"User IDs do not exist on Windows. This function will always return 1"_warn_en_US);`。
- **L2478 EN**: Closes the current lexical scope or compound statement.
  **L2478 CN**: 结束当前词法作用域或复合语句块。
- **L2479 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2479 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2480 EN**: Continues logic associated with callable symbol `Say`.
  **L2480 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L2481 EN**: Executes a standalone statement or declaration: `"Group IDs do not exist on Windows. This function will always return 1"_warn_en_US);`.
  **L2481 CN**: 执行一条独立语句或声明：`"Group IDs do not exist on Windows. This function will always return 1"_warn_en_US);`。
- **L2482 EN**: Closes the current lexical scope or compound statement.
  **L2482 CN**: 结束当前词法作用域或复合语句块。
- **L2483 EN**: Returns from the current function with `true`.
  **L2483 CN**: 以 `true` 从当前函数返回。
- **L2484 EN**: Closes the current lexical scope or compound statement.
  **L2484 CN**: 结束当前词法作用域或复合语句块。
- **L2485 EN**: Blank line separating nearby declarations or logic blocks.
  **L2485 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool CheckArguments(const characteristics::Procedure &proc,`.
  **L2486 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool CheckArguments(const characteristics::Procedure &proc,`。
- **L2487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::ActualArguments &actuals, SemanticsContext &context,`.
  **L2487 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::ActualArguments &actuals, SemanticsContext &context,`。
- **L2488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Scope &scope, bool treatingExternalAsImplicit,`.
  **L2488 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Scope &scope, bool treatingExternalAsImplicit,`。
- **L2489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool ignoreImplicitVsExplicit,`.
  **L2489 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool ignoreImplicitVsExplicit,`。
- **L2490 EN**: Continues the surrounding expression or declaration: `const evaluate::SpecificIntrinsic *intrinsic) {`.
  **L2490 CN**: 继续构造周围的表达式或声明：`const evaluate::SpecificIntrinsic *intrinsic) {`。
- **L2491 EN**: Executes a call or declaration centered on `explicitInterface{proc.HasExplicitInterface`.
  **L2491 CN**: 执行以 `explicitInterface{proc.HasExplicitInterface` 为核心的调用或声明。
- **L2492 EN**: Executes a call or declaration centered on `foldingContext{context.foldingContext`.
  **L2492 CN**: 执行以 `foldingContext{context.foldingContext` 为核心的调用或声明。
- **L2493 EN**: Executes a call or declaration centered on `&messages{foldingContext.messages`.
  **L2493 CN**: 执行以 `&messages{foldingContext.messages` 为核心的调用或声明。
- **L2494 EN**: Executes a standalone statement or declaration: `bool allowArgumentConversions{true};`.
  **L2494 CN**: 执行一条独立语句或声明：`bool allowArgumentConversions{true};`。
- **L2495 EN**: Executes a standalone statement or declaration: `parser::Messages implicitBuffer;`.
  **L2495 CN**: 执行一条独立语句或声明：`parser::Messages implicitBuffer;`。
- **L2496 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2496 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2497-2520

````cpp
    {
      auto restorer{messages.SetMessages(implicitBuffer)};
      for (auto &actual : actuals) {
        if (actual) {
          CheckImplicitInterfaceArg(*actual, messages, context);
        }
      }
    }
    if (implicitBuffer.AnyFatalError()) {
      if (auto *msgs{messages.messages()}) {
        msgs->Annex(std::move(implicitBuffer));
      }
      return false; // don't pile on
    }
    allowArgumentConversions = false;
  }
  if (explicitInterface) {
    auto explicitBuffer{CheckExplicitInterface(proc, actuals, context, &scope,
        intrinsic, allowArgumentConversions,
        /*extentErrors=*/true, ignoreImplicitVsExplicit)};
    if (!explicitBuffer.empty()) {
      if (treatingExternalAsImplicit && explicitBuffer.AnyFatalError()) {
        // Combine all messages into one warning
        if (auto *warning{messages.Warn(/*inModuleFile=*/false,
````
- **L2497 EN**: Opens a new lexical scope or compound statement.
  **L2497 CN**: 打开一个新的词法作用域或复合语句块。
- **L2498 EN**: Executes a call or declaration centered on `restorer{messages.SetMessages`.
  **L2498 CN**: 执行以 `restorer{messages.SetMessages` 为核心的调用或声明。
- **L2499 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2499 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2500 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2500 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2501 EN**: Executes a call or declaration centered on `CheckImplicitInterfaceArg`.
  **L2501 CN**: 执行以 `CheckImplicitInterfaceArg` 为核心的调用或声明。
- **L2502 EN**: Closes the current lexical scope or compound statement.
  **L2502 CN**: 结束当前词法作用域或复合语句块。
- **L2503 EN**: Closes the current lexical scope or compound statement.
  **L2503 CN**: 结束当前词法作用域或复合语句块。
- **L2504 EN**: Closes the current lexical scope or compound statement.
  **L2504 CN**: 结束当前词法作用域或复合语句块。
- **L2505 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2505 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2506 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2506 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2507 EN**: Executes a call or declaration centered on `msgs->Annex`.
  **L2507 CN**: 执行以 `msgs->Annex` 为核心的调用或声明。
- **L2508 EN**: Closes the current lexical scope or compound statement.
  **L2508 CN**: 结束当前词法作用域或复合语句块。
- **L2509 EN**: Returns from the current function with `false; // don't pile on`.
  **L2509 CN**: 以 `false; // don't pile on` 从当前函数返回。
- **L2510 EN**: Closes the current lexical scope or compound statement.
  **L2510 CN**: 结束当前词法作用域或复合语句块。
- **L2511 EN**: Executes a standalone statement or declaration: `allowArgumentConversions = false;`.
  **L2511 CN**: 执行一条独立语句或声明：`allowArgumentConversions = false;`。
- **L2512 EN**: Closes the current lexical scope or compound statement.
  **L2512 CN**: 结束当前词法作用域或复合语句块。
- **L2513 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2513 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto explicitBuffer{CheckExplicitInterface(proc, actuals, context, &scope,`.
  **L2514 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto explicitBuffer{CheckExplicitInterface(proc, actuals, context, &scope,`。
- **L2515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `intrinsic, allowArgumentConversions,`.
  **L2515 CN**: 继续一个多行参数列表、初始化器或聚合项：`intrinsic, allowArgumentConversions,`。
- **L2516 EN**: Comment explains nearby logic, intent, or metadata: `extentErrors=*/true, ignoreImplicitVsExplicit)};`.
  **L2516 CN**: 注释说明附近代码的逻辑、意图或元数据：`extentErrors=*/true, ignoreImplicitVsExplicit)};`。
- **L2517 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2517 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2518 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2518 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2519 EN**: Comment explains nearby logic, intent, or metadata: `Combine all messages into one warning`.
  **L2519 CN**: 注释说明附近代码的逻辑、意图或元数据：`Combine all messages into one warning`。
- **L2520 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2520 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2521-2542

````cpp
                context.languageFeatures(),
                common::UsageWarning::KnownBadImplicitInterface,
                "If the procedure's interface were explicit, this reference would be in error"_warn_en_US)}) {
          explicitBuffer.AttachTo(*warning, parser::Severity::Because);
        }
      } else if (auto *msgs{messages.messages()}) {
        msgs->Annex(std::move(explicitBuffer));
      }
      // These messages override any in implicitBuffer.
      return false;
    }
  }
  if (!implicitBuffer.empty()) {
    if (auto *msgs{messages.messages()}) {
      msgs->Annex(std::move(implicitBuffer));
    }
    return false;
  } else {
    return true; // no messages
  }
}
} // namespace Fortran::semantics
````
- **L2521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.languageFeatures(),`.
  **L2521 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.languageFeatures(),`。
- **L2522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::UsageWarning::KnownBadImplicitInterface,`.
  **L2522 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::UsageWarning::KnownBadImplicitInterface,`。
- **L2523 EN**: Continues the surrounding expression or declaration: `"If the procedure's interface were explicit, this reference would be in error"_warn_en_US)}) {`.
  **L2523 CN**: 继续构造周围的表达式或声明：`"If the procedure's interface were explicit, this reference would be in error"_warn_en_US)}) {`。
- **L2524 EN**: Executes a call or declaration centered on `explicitBuffer.AttachTo`.
  **L2524 CN**: 执行以 `explicitBuffer.AttachTo` 为核心的调用或声明。
- **L2525 EN**: Closes the current lexical scope or compound statement.
  **L2525 CN**: 结束当前词法作用域或复合语句块。
- **L2526 EN**: Transitions from the previous branch into an `else if` condition.
  **L2526 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2527 EN**: Executes a call or declaration centered on `msgs->Annex`.
  **L2527 CN**: 执行以 `msgs->Annex` 为核心的调用或声明。
- **L2528 EN**: Closes the current lexical scope or compound statement.
  **L2528 CN**: 结束当前词法作用域或复合语句块。
- **L2529 EN**: Comment explains nearby logic, intent, or metadata: `These messages override any in implicitBuffer.`.
  **L2529 CN**: 注释说明附近代码的逻辑、意图或元数据：`These messages override any in implicitBuffer.`。
- **L2530 EN**: Returns from the current function with `false`.
  **L2530 CN**: 以 `false` 从当前函数返回。
- **L2531 EN**: Closes the current lexical scope or compound statement.
  **L2531 CN**: 结束当前词法作用域或复合语句块。
- **L2532 EN**: Closes the current lexical scope or compound statement.
  **L2532 CN**: 结束当前词法作用域或复合语句块。
- **L2533 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2533 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2534 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2534 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2535 EN**: Executes a call or declaration centered on `msgs->Annex`.
  **L2535 CN**: 执行以 `msgs->Annex` 为核心的调用或声明。
- **L2536 EN**: Closes the current lexical scope or compound statement.
  **L2536 CN**: 结束当前词法作用域或复合语句块。
- **L2537 EN**: Returns from the current function with `false`.
  **L2537 CN**: 以 `false` 从当前函数返回。
- **L2538 EN**: Transitions from the previous branch into the alternative path.
  **L2538 CN**: 从前一个分支过渡到备选路径。
- **L2539 EN**: Returns from the current function with `true; // no messages`.
  **L2539 CN**: 以 `true; // no messages` 从当前函数返回。
- **L2540 EN**: Closes the current lexical scope or compound statement.
  **L2540 CN**: 结束当前词法作用域或复合语句块。
- **L2541 EN**: Closes the current lexical scope or compound statement.
  **L2541 CN**: 结束当前词法作用域或复合语句块。
- **L2542 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L2542 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Semantic-context management / 语义上下文管理**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Fortran type-spec reasoning / Fortran 类型说明推理**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **Constant folding / 常量折叠**
- **Evaluation context management / 求值上下文管理**
- **Diagnostic emission / 诊断信息发出**

## Dependencies / 依赖关系

- `check-call.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `definable.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `pointer-assignment.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Evaluate/characteristics.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/check-expression.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/fold-designator.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/shape.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/tools.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Parser/characters.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/message.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/scope.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `llvm/ADT/StringSet.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
- `map`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
- `string`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
