# CustomIntrinsicCall.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Lower/CustomIntrinsicCall.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements lowering from Fortran semantics into FIR/MLIR for Custom Intrinsic Call.
- **Purpose (CN)**: 实现 Custom Intrinsic Call 相关的从 Fortran 语义到 FIR/MLIR 的 lowering 过程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===-- CustomIntrinsicCall.cpp -------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Coding style: https://mlir.llvm.org/getting_started/DeveloperGuide/
//
//===----------------------------------------------------------------------===//

#include "flang/Lower/CustomIntrinsicCall.h"
#include "flang/Evaluate/expression.h"
#include "flang/Evaluate/fold.h"
#include "flang/Evaluate/tools.h"
#include "flang/Lower/StatementContext.h"
#include "flang/Optimizer/Builder/IntrinsicCall.h"
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
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, intent, or metadata: `Coding style: https://mlir.llvm.org/getting_started/DeveloperGuide`.
  **L9 CN**: 注释说明附近代码的逻辑、意图或元数据：`Coding style: https://mlir.llvm.org/getting_started/DeveloperGuide`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Includes "flang/Lower/CustomIntrinsicCall.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L13 CN**: 引入 "flang/Lower/CustomIntrinsicCall.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L14 EN**: Includes "flang/Evaluate/expression.h" to access Fortran constant-folding and evaluation facilities.
  **L14 CN**: 引入 "flang/Evaluate/expression.h" 以使用Fortran 常量折叠与求值能力。
- **L15 EN**: Includes "flang/Evaluate/fold.h" to access Fortran constant-folding and evaluation facilities.
  **L15 CN**: 引入 "flang/Evaluate/fold.h" 以使用Fortran 常量折叠与求值能力。
- **L16 EN**: Includes "flang/Evaluate/tools.h" to access Fortran constant-folding and evaluation facilities.
  **L16 CN**: 引入 "flang/Evaluate/tools.h" 以使用Fortran 常量折叠与求值能力。
- **L17 EN**: Includes "flang/Lower/StatementContext.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L17 CN**: 引入 "flang/Lower/StatementContext.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L18 EN**: Includes "flang/Optimizer/Builder/IntrinsicCall.h" to access FIR builder helpers and runtime-construction utilities.
  **L18 CN**: 引入 "flang/Optimizer/Builder/IntrinsicCall.h" 以使用FIR Builder 辅助工具与运行时构造能力。

### Lines 19-36

````cpp
#include "flang/Optimizer/Builder/Todo.h"
#include "flang/Semantics/tools.h"
#include <optional>

/// Is this a call to MIN or MAX intrinsic with arguments that may be absent at
/// runtime? This is a special case because MIN and MAX can have any number of
/// arguments.
static bool isMinOrMaxWithDynamicallyOptionalArg(
    llvm::StringRef name, const Fortran::evaluate::ProcedureRef &procRef) {
  if (name != "min" && name != "max")
    return false;
  const auto &args = procRef.arguments();
  std::size_t argSize = args.size();
  if (argSize <= 2)
    return false;
  for (std::size_t i = 2; i < argSize; ++i) {
    if (auto *expr =
            Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>(args[i]))
````
- **L19 EN**: Includes "flang/Optimizer/Builder/Todo.h" to access FIR builder helpers and runtime-construction utilities.
  **L19 CN**: 引入 "flang/Optimizer/Builder/Todo.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L20 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L20 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L21 EN**: Includes <optional> to access supporting declarations used by this translation unit.
  **L21 CN**: 引入 <optional> 以使用当前编译单元使用的辅助声明。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L23 EN**: Comment explains nearby logic, intent, or metadata: `Is this a call to MIN or MAX intrinsic with arguments that may be absent at`.
  **L23 CN**: 注释说明附近代码的逻辑、意图或元数据：`Is this a call to MIN or MAX intrinsic with arguments that may be absent at`。
- **L24 EN**: Comment explains nearby logic, intent, or metadata: `runtime? This is a special case because MIN and MAX can have any number of`.
  **L24 CN**: 注释说明附近代码的逻辑、意图或元数据：`runtime? This is a special case because MIN and MAX can have any number of`。
- **L25 EN**: Comment explains nearby logic, intent, or metadata: `arguments.`.
  **L25 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments.`。
- **L26 EN**: Continues logic associated with callable symbol `isMinOrMaxWithDynamicallyOptionalArg`.
  **L26 CN**: 继续与可调用符号 `isMinOrMaxWithDynamicallyOptionalArg` 相关的逻辑。
- **L27 EN**: Continues the surrounding expression or declaration: `llvm::StringRef name, const Fortran::evaluate::ProcedureRef &procRef) {`.
  **L27 CN**: 继续构造周围的表达式或声明：`llvm::StringRef name, const Fortran::evaluate::ProcedureRef &procRef) {`。
- **L28 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L28 CN**: 开始 `if` 控制流语句并计算其条件。
- **L29 EN**: Returns from the current function with `false`.
  **L29 CN**: 以 `false` 从当前函数返回。
- **L30 EN**: Executes a call or declaration centered on `procRef.arguments`.
  **L30 CN**: 执行以 `procRef.arguments` 为核心的调用或声明。
- **L31 EN**: Initializes variable `argSize` from the right-hand expression.
  **L31 CN**: 使用右侧表达式初始化变量 `argSize`。
- **L32 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L32 CN**: 开始 `if` 控制流语句并计算其条件。
- **L33 EN**: Returns from the current function with `false`.
  **L33 CN**: 以 `false` 从当前函数返回。
- **L34 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L34 CN**: 开始 `for` 控制流语句并计算其条件。
- **L35 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L35 CN**: 开始 `if` 控制流语句并计算其条件。
- **L36 EN**: Continues logic associated with callable symbol `SomeExpr>`.
  **L36 CN**: 继续与可调用符号 `SomeExpr>` 相关的逻辑。

### Lines 37-54

````cpp
      if (Fortran::evaluate::MayBePassedAsAbsentOptional(*expr))
        return true;
  }
  return false;
}

/// Is this a call to ISHFTC intrinsic with a SIZE argument that may be absent
/// at runtime? This is a special case because the SIZE value to be applied
/// when absent is not zero.
static bool isIshftcWithDynamicallyOptionalArg(
    llvm::StringRef name, const Fortran::evaluate::ProcedureRef &procRef) {
  if (name != "ishftc" || procRef.arguments().size() < 3)
    return false;
  auto *expr = Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>(
      procRef.arguments()[2]);
  return expr && Fortran::evaluate::MayBePassedAsAbsentOptional(*expr);
}

````
- **L37 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L37 CN**: 开始 `if` 控制流语句并计算其条件。
- **L38 EN**: Returns from the current function with `true`.
  **L38 CN**: 以 `true` 从当前函数返回。
- **L39 EN**: Closes the current lexical scope or compound statement.
  **L39 CN**: 结束当前词法作用域或复合语句块。
- **L40 EN**: Returns from the current function with `false`.
  **L40 CN**: 以 `false` 从当前函数返回。
- **L41 EN**: Closes the current lexical scope or compound statement.
  **L41 CN**: 结束当前词法作用域或复合语句块。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Comment explains nearby logic, intent, or metadata: `Is this a call to ISHFTC intrinsic with a SIZE argument that may be absent`.
  **L43 CN**: 注释说明附近代码的逻辑、意图或元数据：`Is this a call to ISHFTC intrinsic with a SIZE argument that may be absent`。
- **L44 EN**: Comment explains nearby logic, intent, or metadata: `at runtime? This is a special case because the SIZE value to be applied`.
  **L44 CN**: 注释说明附近代码的逻辑、意图或元数据：`at runtime? This is a special case because the SIZE value to be applied`。
- **L45 EN**: Comment explains nearby logic, intent, or metadata: `when absent is not zero.`.
  **L45 CN**: 注释说明附近代码的逻辑、意图或元数据：`when absent is not zero.`。
- **L46 EN**: Continues logic associated with callable symbol `isIshftcWithDynamicallyOptionalArg`.
  **L46 CN**: 继续与可调用符号 `isIshftcWithDynamicallyOptionalArg` 相关的逻辑。
- **L47 EN**: Continues the surrounding expression or declaration: `llvm::StringRef name, const Fortran::evaluate::ProcedureRef &procRef) {`.
  **L47 CN**: 继续构造周围的表达式或声明：`llvm::StringRef name, const Fortran::evaluate::ProcedureRef &procRef) {`。
- **L48 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L48 CN**: 开始 `if` 控制流语句并计算其条件。
- **L49 EN**: Returns from the current function with `false`.
  **L49 CN**: 以 `false` 从当前函数返回。
- **L50 EN**: Continues logic associated with callable symbol `SomeExpr>`.
  **L50 CN**: 继续与可调用符号 `SomeExpr>` 相关的逻辑。
- **L51 EN**: Executes a call or declaration centered on `procRef.arguments`.
  **L51 CN**: 执行以 `procRef.arguments` 为核心的调用或声明。
- **L52 EN**: Returns from the current function with `expr && Fortran::evaluate::MayBePassedAsAbsentOptional(*expr)`.
  **L52 CN**: 以 `expr && Fortran::evaluate::MayBePassedAsAbsentOptional(*expr)` 从当前函数返回。
- **L53 EN**: Closes the current lexical scope or compound statement.
  **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 55-72

````cpp
/// Is this a call to ASSOCIATED where the TARGET is an OPTIONAL (but not a
/// deallocated allocatable or disassociated pointer)?
/// Subtle: contrary to other intrinsic optional arguments, disassociated
/// POINTER and unallocated ALLOCATABLE actual argument are not considered
/// absent here. This is because ASSOCIATED has special requirements for TARGET
/// actual arguments that are POINTERs. There is no precise requirements for
/// ALLOCATABLEs, but all existing Fortran compilers treat them similarly to
/// POINTERs. That is: unallocated TARGETs cause ASSOCIATED to rerun false.  The
/// runtime deals with the disassociated/unallocated case. Simply ensures that
/// TARGET that are OPTIONAL get conditionally emboxed here to convey the
/// optional aspect to the runtime.
static bool isAssociatedWithDynamicallyOptionalArg(
    llvm::StringRef name, const Fortran::evaluate::ProcedureRef &procRef) {
  if (name != "associated" || procRef.arguments().size() < 2)
    return false;
  auto *expr = Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>(
      procRef.arguments()[1]);
  const Fortran::semantics::Symbol *sym{
````
- **L55 EN**: Comment explains nearby logic, intent, or metadata: `Is this a call to ASSOCIATED where the TARGET is an OPTIONAL (but not a`.
  **L55 CN**: 注释说明附近代码的逻辑、意图或元数据：`Is this a call to ASSOCIATED where the TARGET is an OPTIONAL (but not a`。
- **L56 EN**: Comment explains nearby logic, intent, or metadata: `deallocated allocatable or disassociated pointer)?`.
  **L56 CN**: 注释说明附近代码的逻辑、意图或元数据：`deallocated allocatable or disassociated pointer)?`。
- **L57 EN**: Comment explains nearby logic, intent, or metadata: `Subtle: contrary to other intrinsic optional arguments, disassociated`.
  **L57 CN**: 注释说明附近代码的逻辑、意图或元数据：`Subtle: contrary to other intrinsic optional arguments, disassociated`。
- **L58 EN**: Comment explains nearby logic, intent, or metadata: `POINTER and unallocated ALLOCATABLE actual argument are not considered`.
  **L58 CN**: 注释说明附近代码的逻辑、意图或元数据：`POINTER and unallocated ALLOCATABLE actual argument are not considered`。
- **L59 EN**: Comment explains nearby logic, intent, or metadata: `absent here. This is because ASSOCIATED has special requirements for TARGET`.
  **L59 CN**: 注释说明附近代码的逻辑、意图或元数据：`absent here. This is because ASSOCIATED has special requirements for TARGET`。
- **L60 EN**: Comment explains nearby logic, intent, or metadata: `actual arguments that are POINTERs. There is no precise requirements for`.
  **L60 CN**: 注释说明附近代码的逻辑、意图或元数据：`actual arguments that are POINTERs. There is no precise requirements for`。
- **L61 EN**: Comment explains nearby logic, intent, or metadata: `ALLOCATABLEs, but all existing Fortran compilers treat them similarly to`.
  **L61 CN**: 注释说明附近代码的逻辑、意图或元数据：`ALLOCATABLEs, but all existing Fortran compilers treat them similarly to`。
- **L62 EN**: Comment explains nearby logic, intent, or metadata: `POINTERs. That is: unallocated TARGETs cause ASSOCIATED to rerun false.  The`.
  **L62 CN**: 注释说明附近代码的逻辑、意图或元数据：`POINTERs. That is: unallocated TARGETs cause ASSOCIATED to rerun false.  The`。
- **L63 EN**: Comment explains nearby logic, intent, or metadata: `runtime deals with the disassociated/unallocated case. Simply ensures that`.
  **L63 CN**: 注释说明附近代码的逻辑、意图或元数据：`runtime deals with the disassociated/unallocated case. Simply ensures that`。
- **L64 EN**: Comment explains nearby logic, intent, or metadata: `TARGET that are OPTIONAL get conditionally emboxed here to convey the`.
  **L64 CN**: 注释说明附近代码的逻辑、意图或元数据：`TARGET that are OPTIONAL get conditionally emboxed here to convey the`。
- **L65 EN**: Comment explains nearby logic, intent, or metadata: `optional aspect to the runtime.`.
  **L65 CN**: 注释说明附近代码的逻辑、意图或元数据：`optional aspect to the runtime.`。
- **L66 EN**: Continues logic associated with callable symbol `isAssociatedWithDynamicallyOptionalArg`.
  **L66 CN**: 继续与可调用符号 `isAssociatedWithDynamicallyOptionalArg` 相关的逻辑。
- **L67 EN**: Continues the surrounding expression or declaration: `llvm::StringRef name, const Fortran::evaluate::ProcedureRef &procRef) {`.
  **L67 CN**: 继续构造周围的表达式或声明：`llvm::StringRef name, const Fortran::evaluate::ProcedureRef &procRef) {`。
- **L68 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L68 CN**: 开始 `if` 控制流语句并计算其条件。
- **L69 EN**: Returns from the current function with `false`.
  **L69 CN**: 以 `false` 从当前函数返回。
- **L70 EN**: Continues logic associated with callable symbol `SomeExpr>`.
  **L70 CN**: 继续与可调用符号 `SomeExpr>` 相关的逻辑。
- **L71 EN**: Executes a call or declaration centered on `procRef.arguments`.
  **L71 CN**: 执行以 `procRef.arguments` 为核心的调用或声明。
- **L72 EN**: Continues the surrounding expression or declaration: `const Fortran::semantics::Symbol *sym{`.
  **L72 CN**: 继续构造周围的表达式或声明：`const Fortran::semantics::Symbol *sym{`。

### Lines 73-90

````cpp
      expr ? Fortran::evaluate::UnwrapWholeSymbolOrComponentDataRef(expr)
           : nullptr};
  return (sym && Fortran::semantics::IsOptional(*sym));
}

bool Fortran::lower::intrinsicRequiresCustomOptionalHandling(
    const Fortran::evaluate::ProcedureRef &procRef,
    const Fortran::evaluate::SpecificIntrinsic &intrinsic,
    AbstractConverter &converter) {
  llvm::StringRef name = intrinsic.name;
  return isMinOrMaxWithDynamicallyOptionalArg(name, procRef) ||
         isIshftcWithDynamicallyOptionalArg(name, procRef) ||
         isAssociatedWithDynamicallyOptionalArg(name, procRef);
}

/// Generate the FIR+MLIR operations for the generic intrinsic \p name
/// with arguments \p args and the expected result type \p resultType.
/// Returned fir::ExtendedValue is the returned Fortran intrinsic value.
````
- **L73 EN**: Continues logic associated with callable symbol `UnwrapWholeSymbolOrComponentDataRef`.
  **L73 CN**: 继续与可调用符号 `UnwrapWholeSymbolOrComponentDataRef` 相关的逻辑。
- **L74 EN**: Executes a standalone statement or declaration: `: nullptr};`.
  **L74 CN**: 执行一条独立语句或声明：`: nullptr};`。
- **L75 EN**: Returns from the current function with `(sym && Fortran::semantics::IsOptional(*sym))`.
  **L75 CN**: 以 `(sym && Fortran::semantics::IsOptional(*sym))` 从当前函数返回。
- **L76 EN**: Closes the current lexical scope or compound statement.
  **L76 CN**: 结束当前词法作用域或复合语句块。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L78 EN**: Continues logic associated with callable symbol `intrinsicRequiresCustomOptionalHandling`.
  **L78 CN**: 继续与可调用符号 `intrinsicRequiresCustomOptionalHandling` 相关的逻辑。
- **L79 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::ProcedureRef &procRef,`.
  **L79 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::ProcedureRef &procRef,`。
- **L80 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::SpecificIntrinsic &intrinsic,`.
  **L80 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::SpecificIntrinsic &intrinsic,`。
- **L81 EN**: Continues the surrounding expression or declaration: `AbstractConverter &converter) {`.
  **L81 CN**: 继续构造周围的表达式或声明：`AbstractConverter &converter) {`。
- **L82 EN**: Initializes variable `name` from the right-hand expression.
  **L82 CN**: 使用右侧表达式初始化变量 `name`。
- **L83 EN**: Returns from the current function with `isMinOrMaxWithDynamicallyOptionalArg(name, procRef) ||`.
  **L83 CN**: 以 `isMinOrMaxWithDynamicallyOptionalArg(name, procRef) ||` 从当前函数返回。
- **L84 EN**: Continues logic associated with callable symbol `isIshftcWithDynamicallyOptionalArg`.
  **L84 CN**: 继续与可调用符号 `isIshftcWithDynamicallyOptionalArg` 相关的逻辑。
- **L85 EN**: Executes a call or declaration centered on `isAssociatedWithDynamicallyOptionalArg`.
  **L85 CN**: 执行以 `isAssociatedWithDynamicallyOptionalArg` 为核心的调用或声明。
- **L86 EN**: Closes the current lexical scope or compound statement.
  **L86 CN**: 结束当前词法作用域或复合语句块。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L88 EN**: Comment explains nearby logic, intent, or metadata: `Generate the FIR+MLIR operations for the generic intrinsic \p name`.
  **L88 CN**: 注释说明附近代码的逻辑、意图或元数据：`Generate the FIR+MLIR operations for the generic intrinsic \p name`。
- **L89 EN**: Comment explains nearby logic, intent, or metadata: `with arguments \p args and the expected result type \p resultType.`.
  **L89 CN**: 注释说明附近代码的逻辑、意图或元数据：`with arguments \p args and the expected result type \p resultType.`。
- **L90 EN**: Comment explains nearby logic, intent, or metadata: `Returned fir::ExtendedValue is the returned Fortran intrinsic value.`.
  **L90 CN**: 注释说明附近代码的逻辑、意图或元数据：`Returned fir::ExtendedValue is the returned Fortran intrinsic value.`。

### Lines 91-108

````cpp
fir::ExtendedValue
Fortran::lower::genIntrinsicCall(fir::FirOpBuilder &builder, mlir::Location loc,
                                 llvm::StringRef name,
                                 std::optional<mlir::Type> resultType,
                                 llvm::ArrayRef<fir::ExtendedValue> args,
                                 Fortran::lower::StatementContext &stmtCtx,
                                 Fortran::lower::AbstractConverter *converter) {
  auto [result, mustBeFreed] =
      fir::genIntrinsicCall(builder, loc, name, resultType, args, converter);
  if (mustBeFreed) {
    mlir::Value addr = fir::getBase(result);
    if (auto *box = result.getBoxOf<fir::BoxValue>())
      addr =
          fir::BoxAddrOp::create(builder, loc, box->getMemTy(), box->getAddr());
    fir::FirOpBuilder *bldr = &builder;
    stmtCtx.attachCleanup([=]() { fir::FreeMemOp::create(*bldr, loc, addr); });
  }
  return result;
````
- **L91 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L91 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::genIntrinsicCall(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::genIntrinsicCall(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::StringRef name,`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::StringRef name,`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::optional<mlir::Type> resultType,`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::optional<mlir::Type> resultType,`。
- **L95 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::ArrayRef<fir::ExtendedValue> args,`.
  **L95 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::ArrayRef<fir::ExtendedValue> args,`。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::StatementContext &stmtCtx,`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::StatementContext &stmtCtx,`。
- **L97 EN**: Continues the surrounding expression or declaration: `Fortran::lower::AbstractConverter *converter) {`.
  **L97 CN**: 继续构造周围的表达式或声明：`Fortran::lower::AbstractConverter *converter) {`。
- **L98 EN**: Continues the surrounding expression or declaration: `auto [result, mustBeFreed] =`.
  **L98 CN**: 继续构造周围的表达式或声明：`auto [result, mustBeFreed] =`。
- **L99 EN**: Executes a call or declaration centered on `fir::genIntrinsicCall`.
  **L99 CN**: 执行以 `fir::genIntrinsicCall` 为核心的调用或声明。
- **L100 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L100 CN**: 开始 `if` 控制流语句并计算其条件。
- **L101 EN**: Initializes variable `addr` from the right-hand expression.
  **L101 CN**: 使用右侧表达式初始化变量 `addr`。
- **L102 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L102 CN**: 开始 `if` 控制流语句并计算其条件。
- **L103 EN**: Continues the surrounding expression or declaration: `addr =`.
  **L103 CN**: 继续构造周围的表达式或声明：`addr =`。
- **L104 EN**: Executes a call or declaration centered on `fir::BoxAddrOp::create`.
  **L104 CN**: 执行以 `fir::BoxAddrOp::create` 为核心的调用或声明。
- **L105 EN**: Executes a standalone statement or declaration: `fir::FirOpBuilder *bldr = &builder;`.
  **L105 CN**: 执行一条独立语句或声明：`fir::FirOpBuilder *bldr = &builder;`。
- **L106 EN**: Executes a call or declaration centered on `stmtCtx.attachCleanup`.
  **L106 CN**: 执行以 `stmtCtx.attachCleanup` 为核心的调用或声明。
- **L107 EN**: Closes the current lexical scope or compound statement.
  **L107 CN**: 结束当前词法作用域或复合语句块。
- **L108 EN**: Returns from the current function with `result`.
  **L108 CN**: 以 `result` 从当前函数返回。

### Lines 109-126

````cpp
}

static void prepareMinOrMaxArguments(
    const Fortran::evaluate::ProcedureRef &procRef,
    const Fortran::evaluate::SpecificIntrinsic &intrinsic,
    std::optional<mlir::Type> retTy,
    const Fortran::lower::OperandPrepare &prepareOptionalArgument,
    const Fortran::lower::OperandPrepareAs &prepareOtherArgument,
    Fortran::lower::AbstractConverter &converter) {
  assert(retTy && "MIN and MAX must have a return type");
  mlir::Type resultType = *retTy;
  mlir::Location loc = converter.getCurrentLocation();
  if (fir::isa_char(fir::unwrapSequenceType(resultType)))
    TODO(loc, "CHARACTER MIN and MAX with dynamically optional arguments");
  for (auto arg : llvm::enumerate(procRef.arguments())) {
    const auto *expr =
        Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>(arg.value());
    if (!expr)
````
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L111 EN**: Continues logic associated with callable symbol `prepareMinOrMaxArguments`.
  **L111 CN**: 继续与可调用符号 `prepareMinOrMaxArguments` 相关的逻辑。
- **L112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::ProcedureRef &procRef,`.
  **L112 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::ProcedureRef &procRef,`。
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::SpecificIntrinsic &intrinsic,`.
  **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::SpecificIntrinsic &intrinsic,`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::optional<mlir::Type> retTy,`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::optional<mlir::Type> retTy,`。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandPrepare &prepareOptionalArgument,`.
  **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandPrepare &prepareOptionalArgument,`。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandPrepareAs &prepareOtherArgument,`.
  **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandPrepareAs &prepareOtherArgument,`。
- **L117 EN**: Continues the surrounding expression or declaration: `Fortran::lower::AbstractConverter &converter) {`.
  **L117 CN**: 继续构造周围的表达式或声明：`Fortran::lower::AbstractConverter &converter) {`。
- **L118 EN**: Checks an internal invariant in debug builds.
  **L118 CN**: 在调试构建中检查内部不变式。
- **L119 EN**: Initializes variable `resultType` from the right-hand expression.
  **L119 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L120 EN**: Initializes variable `loc` from the right-hand expression.
  **L120 CN**: 使用右侧表达式初始化变量 `loc`。
- **L121 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L121 CN**: 开始 `if` 控制流语句并计算其条件。
- **L122 EN**: Executes a call or declaration centered on `TODO`.
  **L122 CN**: 执行以 `TODO` 为核心的调用或声明。
- **L123 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L123 CN**: 开始 `for` 控制流语句并计算其条件。
- **L124 EN**: Continues the surrounding expression or declaration: `const auto *expr =`.
  **L124 CN**: 继续构造周围的表达式或声明：`const auto *expr =`。
- **L125 EN**: Executes a call or declaration centered on `Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>`.
  **L125 CN**: 执行以 `Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>` 为核心的调用或声明。
- **L126 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L126 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 127-144

````cpp
      continue;
    if (arg.index() <= 1 ||
        !Fortran::evaluate::MayBePassedAsAbsentOptional(*expr)) {
      // Non optional arguments.
      prepareOtherArgument(*expr, fir::LowerIntrinsicArgAs::Value);
    } else {
      // Dynamically optional arguments.
      // Subtle: even for scalar the if-then-else will be generated in the loop
      // nest because the then part will require the current extremum value that
      // may depend on previous array element argument and cannot be outlined.
      prepareOptionalArgument(*expr);
    }
  }
}

static fir::ExtendedValue
lowerMinOrMax(fir::FirOpBuilder &builder, mlir::Location loc,
              llvm::StringRef name, std::optional<mlir::Type> retTy,
````
- **L127 EN**: Skips to the next loop iteration.
  **L127 CN**: 跳到下一次循环迭代。
- **L128 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L128 CN**: 开始 `if` 控制流语句并计算其条件。
- **L129 EN**: Starts a function, method, lambda, or structured scope: `!Fortran::evaluate::MayBePassedAsAbsentOptional(*expr)) {`.
  **L129 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!Fortran::evaluate::MayBePassedAsAbsentOptional(*expr)) {`。
- **L130 EN**: Comment explains nearby logic, intent, or metadata: `Non optional arguments.`.
  **L130 CN**: 注释说明附近代码的逻辑、意图或元数据：`Non optional arguments.`。
- **L131 EN**: Executes a call or declaration centered on `prepareOtherArgument`.
  **L131 CN**: 执行以 `prepareOtherArgument` 为核心的调用或声明。
- **L132 EN**: Transitions from the previous branch into the alternative path.
  **L132 CN**: 从前一个分支过渡到备选路径。
- **L133 EN**: Comment explains nearby logic, intent, or metadata: `Dynamically optional arguments.`.
  **L133 CN**: 注释说明附近代码的逻辑、意图或元数据：`Dynamically optional arguments.`。
- **L134 EN**: Comment explains nearby logic, intent, or metadata: `Subtle: even for scalar the if-then-else will be generated in the loop`.
  **L134 CN**: 注释说明附近代码的逻辑、意图或元数据：`Subtle: even for scalar the if-then-else will be generated in the loop`。
- **L135 EN**: Comment explains nearby logic, intent, or metadata: `nest because the then part will require the current extremum value that`.
  **L135 CN**: 注释说明附近代码的逻辑、意图或元数据：`nest because the then part will require the current extremum value that`。
- **L136 EN**: Comment explains nearby logic, intent, or metadata: `may depend on previous array element argument and cannot be outlined.`.
  **L136 CN**: 注释说明附近代码的逻辑、意图或元数据：`may depend on previous array element argument and cannot be outlined.`。
- **L137 EN**: Executes a call or declaration centered on `prepareOptionalArgument`.
  **L137 CN**: 执行以 `prepareOptionalArgument` 为核心的调用或声明。
- **L138 EN**: Closes the current lexical scope or compound statement.
  **L138 CN**: 结束当前词法作用域或复合语句块。
- **L139 EN**: Closes the current lexical scope or compound statement.
  **L139 CN**: 结束当前词法作用域或复合语句块。
- **L140 EN**: Closes the current lexical scope or compound statement.
  **L140 CN**: 结束当前词法作用域或复合语句块。
- **L141 EN**: Blank line separating nearby declarations or logic blocks.
  **L141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L142 EN**: Continues the surrounding expression or declaration: `static fir::ExtendedValue`.
  **L142 CN**: 继续构造周围的表达式或声明：`static fir::ExtendedValue`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerMinOrMax(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerMinOrMax(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::StringRef name, std::optional<mlir::Type> retTy,`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::StringRef name, std::optional<mlir::Type> retTy,`。

### Lines 145-162

````cpp
              const Fortran::lower::OperandPresent &isPresentCheck,
              const Fortran::lower::OperandGetter &getOperand,
              std::size_t numOperands,
              Fortran::lower::StatementContext &stmtCtx) {
  assert(numOperands >= 2 && !isPresentCheck(0) && !isPresentCheck(1) &&
         "min/max must have at least two non-optional args");
  assert(retTy && "MIN and MAX must have a return type");
  mlir::Type resultType = *retTy;
  llvm::SmallVector<fir::ExtendedValue> args;
  const bool loadOperand = true;
  args.push_back(getOperand(0, loadOperand));
  args.push_back(getOperand(1, loadOperand));
  mlir::Value extremum = fir::getBase(
      genIntrinsicCall(builder, loc, name, resultType, args, stmtCtx));

  for (std::size_t opIndex = 2; opIndex < numOperands; ++opIndex) {
    if (std::optional<mlir::Value> isPresentRuntimeCheck =
            isPresentCheck(opIndex)) {
````
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandPresent &isPresentCheck,`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandPresent &isPresentCheck,`。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandGetter &getOperand,`.
  **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandGetter &getOperand,`。
- **L147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::size_t numOperands,`.
  **L147 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::size_t numOperands,`。
- **L148 EN**: Continues the surrounding expression or declaration: `Fortran::lower::StatementContext &stmtCtx) {`.
  **L148 CN**: 继续构造周围的表达式或声明：`Fortran::lower::StatementContext &stmtCtx) {`。
- **L149 EN**: Checks an internal invariant in debug builds.
  **L149 CN**: 在调试构建中检查内部不变式。
- **L150 EN**: Executes a standalone statement or declaration: `"min/max must have at least two non-optional args");`.
  **L150 CN**: 执行一条独立语句或声明：`"min/max must have at least two non-optional args");`。
- **L151 EN**: Checks an internal invariant in debug builds.
  **L151 CN**: 在调试构建中检查内部不变式。
- **L152 EN**: Initializes variable `resultType` from the right-hand expression.
  **L152 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L153 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<fir::ExtendedValue> args;`.
  **L153 CN**: 执行一条独立语句或声明：`llvm::SmallVector<fir::ExtendedValue> args;`。
- **L154 EN**: Initializes variable `loadOperand` from the right-hand expression.
  **L154 CN**: 使用右侧表达式初始化变量 `loadOperand`。
- **L155 EN**: Executes a call or declaration centered on `args.push_back`.
  **L155 CN**: 执行以 `args.push_back` 为核心的调用或声明。
- **L156 EN**: Executes a call or declaration centered on `args.push_back`.
  **L156 CN**: 执行以 `args.push_back` 为核心的调用或声明。
- **L157 EN**: Continues logic associated with callable symbol `getBase`.
  **L157 CN**: 继续与可调用符号 `getBase` 相关的逻辑。
- **L158 EN**: Executes a call or declaration centered on `genIntrinsicCall`.
  **L158 CN**: 执行以 `genIntrinsicCall` 为核心的调用或声明。
- **L159 EN**: Blank line separating nearby declarations or logic blocks.
  **L159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L160 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L160 CN**: 开始 `for` 控制流语句并计算其条件。
- **L161 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L161 CN**: 开始 `if` 控制流语句并计算其条件。
- **L162 EN**: Starts a function, method, lambda, or structured scope: `isPresentCheck(opIndex)) {`.
  **L162 CN**: 开始一个函数、方法、lambda 或结构化作用域：`isPresentCheck(opIndex)) {`。

### Lines 163-180

````cpp
      // Argument is dynamically optional.
      extremum =
          builder
              .genIfOp(loc, {resultType}, *isPresentRuntimeCheck,
                       /*withElseRegion=*/true)
              .genThen([&]() {
                llvm::SmallVector<fir::ExtendedValue> args;
                args.emplace_back(extremum);
                args.emplace_back(getOperand(opIndex, loadOperand));
                fir::ExtendedValue newExtremum = genIntrinsicCall(
                    builder, loc, name, resultType, args, stmtCtx);
                fir::ResultOp::create(builder, loc, fir::getBase(newExtremum));
              })
              .genElse([&]() { fir::ResultOp::create(builder, loc, extremum); })
              .getResults()[0];
    } else {
      // Argument is know to be present at compile time.
      llvm::SmallVector<fir::ExtendedValue> args;
````
- **L163 EN**: Comment explains nearby logic, intent, or metadata: `Argument is dynamically optional.`.
  **L163 CN**: 注释说明附近代码的逻辑、意图或元数据：`Argument is dynamically optional.`。
- **L164 EN**: Continues the surrounding expression or declaration: `extremum =`.
  **L164 CN**: 继续构造周围的表达式或声明：`extremum =`。
- **L165 EN**: Continues the surrounding expression or declaration: `builder`.
  **L165 CN**: 继续构造周围的表达式或声明：`builder`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.genIfOp(loc, {resultType}, *isPresentRuntimeCheck,`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`.genIfOp(loc, {resultType}, *isPresentRuntimeCheck,`。
- **L167 EN**: Comment explains nearby logic, intent, or metadata: `withElseRegion=*/true)`.
  **L167 CN**: 注释说明附近代码的逻辑、意图或元数据：`withElseRegion=*/true)`。
- **L168 EN**: Starts a function, method, lambda, or structured scope: `.genThen([&]() {`.
  **L168 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genThen([&]() {`。
- **L169 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<fir::ExtendedValue> args;`.
  **L169 CN**: 执行一条独立语句或声明：`llvm::SmallVector<fir::ExtendedValue> args;`。
- **L170 EN**: Executes a call or declaration centered on `args.emplace_back`.
  **L170 CN**: 执行以 `args.emplace_back` 为核心的调用或声明。
- **L171 EN**: Executes a call or declaration centered on `args.emplace_back`.
  **L171 CN**: 执行以 `args.emplace_back` 为核心的调用或声明。
- **L172 EN**: Continues logic associated with callable symbol `genIntrinsicCall`.
  **L172 CN**: 继续与可调用符号 `genIntrinsicCall` 相关的逻辑。
- **L173 EN**: Executes a standalone statement or declaration: `builder, loc, name, resultType, args, stmtCtx);`.
  **L173 CN**: 执行一条独立语句或声明：`builder, loc, name, resultType, args, stmtCtx);`。
- **L174 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L174 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L175 EN**: Continues the surrounding expression or declaration: `})`.
  **L175 CN**: 继续构造周围的表达式或声明：`})`。
- **L176 EN**: Continues logic associated with callable symbol `genElse`.
  **L176 CN**: 继续与可调用符号 `genElse` 相关的逻辑。
- **L177 EN**: Executes a call or declaration centered on `.getResults`.
  **L177 CN**: 执行以 `.getResults` 为核心的调用或声明。
- **L178 EN**: Transitions from the previous branch into the alternative path.
  **L178 CN**: 从前一个分支过渡到备选路径。
- **L179 EN**: Comment explains nearby logic, intent, or metadata: `Argument is know to be present at compile time.`.
  **L179 CN**: 注释说明附近代码的逻辑、意图或元数据：`Argument is know to be present at compile time.`。
- **L180 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<fir::ExtendedValue> args;`.
  **L180 CN**: 执行一条独立语句或声明：`llvm::SmallVector<fir::ExtendedValue> args;`。

### Lines 181-198

````cpp
      args.emplace_back(extremum);
      args.emplace_back(getOperand(opIndex, loadOperand));
      extremum = fir::getBase(
          genIntrinsicCall(builder, loc, name, resultType, args, stmtCtx));
    }
  }
  return extremum;
}

static void prepareIshftcArguments(
    const Fortran::evaluate::ProcedureRef &procRef,
    const Fortran::evaluate::SpecificIntrinsic &intrinsic,
    std::optional<mlir::Type> retTy,
    const Fortran::lower::OperandPrepare &prepareOptionalArgument,
    const Fortran::lower::OperandPrepareAs &prepareOtherArgument,
    Fortran::lower::AbstractConverter &converter) {
  for (auto arg : llvm::enumerate(procRef.arguments())) {
    const auto *expr =
````
- **L181 EN**: Executes a call or declaration centered on `args.emplace_back`.
  **L181 CN**: 执行以 `args.emplace_back` 为核心的调用或声明。
- **L182 EN**: Executes a call or declaration centered on `args.emplace_back`.
  **L182 CN**: 执行以 `args.emplace_back` 为核心的调用或声明。
- **L183 EN**: Continues logic associated with callable symbol `getBase`.
  **L183 CN**: 继续与可调用符号 `getBase` 相关的逻辑。
- **L184 EN**: Executes a call or declaration centered on `genIntrinsicCall`.
  **L184 CN**: 执行以 `genIntrinsicCall` 为核心的调用或声明。
- **L185 EN**: Closes the current lexical scope or compound statement.
  **L185 CN**: 结束当前词法作用域或复合语句块。
- **L186 EN**: Closes the current lexical scope or compound statement.
  **L186 CN**: 结束当前词法作用域或复合语句块。
- **L187 EN**: Returns from the current function with `extremum`.
  **L187 CN**: 以 `extremum` 从当前函数返回。
- **L188 EN**: Closes the current lexical scope or compound statement.
  **L188 CN**: 结束当前词法作用域或复合语句块。
- **L189 EN**: Blank line separating nearby declarations or logic blocks.
  **L189 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L190 EN**: Continues logic associated with callable symbol `prepareIshftcArguments`.
  **L190 CN**: 继续与可调用符号 `prepareIshftcArguments` 相关的逻辑。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::ProcedureRef &procRef,`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::ProcedureRef &procRef,`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::SpecificIntrinsic &intrinsic,`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::SpecificIntrinsic &intrinsic,`。
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::optional<mlir::Type> retTy,`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::optional<mlir::Type> retTy,`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandPrepare &prepareOptionalArgument,`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandPrepare &prepareOptionalArgument,`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandPrepareAs &prepareOtherArgument,`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandPrepareAs &prepareOtherArgument,`。
- **L196 EN**: Continues the surrounding expression or declaration: `Fortran::lower::AbstractConverter &converter) {`.
  **L196 CN**: 继续构造周围的表达式或声明：`Fortran::lower::AbstractConverter &converter) {`。
- **L197 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L197 CN**: 开始 `for` 控制流语句并计算其条件。
- **L198 EN**: Continues the surrounding expression or declaration: `const auto *expr =`.
  **L198 CN**: 继续构造周围的表达式或声明：`const auto *expr =`。

### Lines 199-216

````cpp
        Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>(arg.value());
    assert(expr && "expected all ISHFTC argument to be textually present here");
    if (arg.index() == 2) {
      assert(Fortran::evaluate::MayBePassedAsAbsentOptional(*expr) &&
             "expected ISHFTC SIZE arg to be dynamically optional");
      prepareOptionalArgument(*expr);
    } else {
      // Non optional arguments.
      prepareOtherArgument(*expr, fir::LowerIntrinsicArgAs::Value);
    }
  }
}

static fir::ExtendedValue
lowerIshftc(fir::FirOpBuilder &builder, mlir::Location loc,
            llvm::StringRef name, std::optional<mlir::Type> retTy,
            const Fortran::lower::OperandPresent &isPresentCheck,
            const Fortran::lower::OperandGetter &getOperand,
````
- **L199 EN**: Executes a call or declaration centered on `Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>`.
  **L199 CN**: 执行以 `Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>` 为核心的调用或声明。
- **L200 EN**: Checks an internal invariant in debug builds.
  **L200 CN**: 在调试构建中检查内部不变式。
- **L201 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L201 CN**: 开始 `if` 控制流语句并计算其条件。
- **L202 EN**: Checks an internal invariant in debug builds.
  **L202 CN**: 在调试构建中检查内部不变式。
- **L203 EN**: Executes a standalone statement or declaration: `"expected ISHFTC SIZE arg to be dynamically optional");`.
  **L203 CN**: 执行一条独立语句或声明：`"expected ISHFTC SIZE arg to be dynamically optional");`。
- **L204 EN**: Executes a call or declaration centered on `prepareOptionalArgument`.
  **L204 CN**: 执行以 `prepareOptionalArgument` 为核心的调用或声明。
- **L205 EN**: Transitions from the previous branch into the alternative path.
  **L205 CN**: 从前一个分支过渡到备选路径。
- **L206 EN**: Comment explains nearby logic, intent, or metadata: `Non optional arguments.`.
  **L206 CN**: 注释说明附近代码的逻辑、意图或元数据：`Non optional arguments.`。
- **L207 EN**: Executes a call or declaration centered on `prepareOtherArgument`.
  **L207 CN**: 执行以 `prepareOtherArgument` 为核心的调用或声明。
- **L208 EN**: Closes the current lexical scope or compound statement.
  **L208 CN**: 结束当前词法作用域或复合语句块。
- **L209 EN**: Closes the current lexical scope or compound statement.
  **L209 CN**: 结束当前词法作用域或复合语句块。
- **L210 EN**: Closes the current lexical scope or compound statement.
  **L210 CN**: 结束当前词法作用域或复合语句块。
- **L211 EN**: Blank line separating nearby declarations or logic blocks.
  **L211 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L212 EN**: Continues the surrounding expression or declaration: `static fir::ExtendedValue`.
  **L212 CN**: 继续构造周围的表达式或声明：`static fir::ExtendedValue`。
- **L213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerIshftc(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L213 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerIshftc(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::StringRef name, std::optional<mlir::Type> retTy,`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::StringRef name, std::optional<mlir::Type> retTy,`。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandPresent &isPresentCheck,`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandPresent &isPresentCheck,`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandGetter &getOperand,`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandGetter &getOperand,`。

### Lines 217-234

````cpp
            std::size_t numOperands,
            Fortran::lower::StatementContext &stmtCtx) {
  assert(numOperands == 3 && !isPresentCheck(0) && !isPresentCheck(1) &&
         isPresentCheck(2) &&
         "only ISHFTC SIZE arg is expected to be dynamically optional here");
  assert(retTy && "ISFHTC must have a return type");
  mlir::Type resultType = *retTy;
  llvm::SmallVector<fir::ExtendedValue> args;
  const bool loadOperand = true;
  args.push_back(getOperand(0, loadOperand));
  args.push_back(getOperand(1, loadOperand));
  auto iPC = isPresentCheck(2);
  assert(iPC.has_value());
  args.push_back(
      builder
          .genIfOp(loc, {resultType}, *iPC,
                   /*withElseRegion=*/true)
          .genThen([&]() {
````
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::size_t numOperands,`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::size_t numOperands,`。
- **L218 EN**: Continues the surrounding expression or declaration: `Fortran::lower::StatementContext &stmtCtx) {`.
  **L218 CN**: 继续构造周围的表达式或声明：`Fortran::lower::StatementContext &stmtCtx) {`。
- **L219 EN**: Checks an internal invariant in debug builds.
  **L219 CN**: 在调试构建中检查内部不变式。
- **L220 EN**: Continues logic associated with callable symbol `isPresentCheck`.
  **L220 CN**: 继续与可调用符号 `isPresentCheck` 相关的逻辑。
- **L221 EN**: Executes a standalone statement or declaration: `"only ISHFTC SIZE arg is expected to be dynamically optional here");`.
  **L221 CN**: 执行一条独立语句或声明：`"only ISHFTC SIZE arg is expected to be dynamically optional here");`。
- **L222 EN**: Checks an internal invariant in debug builds.
  **L222 CN**: 在调试构建中检查内部不变式。
- **L223 EN**: Initializes variable `resultType` from the right-hand expression.
  **L223 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L224 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<fir::ExtendedValue> args;`.
  **L224 CN**: 执行一条独立语句或声明：`llvm::SmallVector<fir::ExtendedValue> args;`。
- **L225 EN**: Initializes variable `loadOperand` from the right-hand expression.
  **L225 CN**: 使用右侧表达式初始化变量 `loadOperand`。
- **L226 EN**: Executes a call or declaration centered on `args.push_back`.
  **L226 CN**: 执行以 `args.push_back` 为核心的调用或声明。
- **L227 EN**: Executes a call or declaration centered on `args.push_back`.
  **L227 CN**: 执行以 `args.push_back` 为核心的调用或声明。
- **L228 EN**: Initializes variable `iPC` from the right-hand expression.
  **L228 CN**: 使用右侧表达式初始化变量 `iPC`。
- **L229 EN**: Checks an internal invariant in debug builds.
  **L229 CN**: 在调试构建中检查内部不变式。
- **L230 EN**: Continues logic associated with callable symbol `push_back`.
  **L230 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L231 EN**: Continues the surrounding expression or declaration: `builder`.
  **L231 CN**: 继续构造周围的表达式或声明：`builder`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.genIfOp(loc, {resultType}, *iPC,`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`.genIfOp(loc, {resultType}, *iPC,`。
- **L233 EN**: Comment explains nearby logic, intent, or metadata: `withElseRegion=*/true)`.
  **L233 CN**: 注释说明附近代码的逻辑、意图或元数据：`withElseRegion=*/true)`。
- **L234 EN**: Starts a function, method, lambda, or structured scope: `.genThen([&]() {`.
  **L234 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genThen([&]() {`。

### Lines 235-252

````cpp
            fir::ExtendedValue sizeExv = getOperand(2, loadOperand);
            mlir::Value size =
                builder.createConvert(loc, resultType, fir::getBase(sizeExv));
            fir::ResultOp::create(builder, loc, size);
          })
          .genElse([&]() {
            mlir::Value bitSize = builder.createIntegerConstant(
                loc, resultType,
                mlir::cast<mlir::IntegerType>(resultType).getWidth());
            fir::ResultOp::create(builder, loc, bitSize);
          })
          .getResults()[0]);
  return genIntrinsicCall(builder, loc, name, resultType, args, stmtCtx);
}

static void prepareAssociatedArguments(
    const Fortran::evaluate::ProcedureRef &procRef,
    const Fortran::evaluate::SpecificIntrinsic &intrinsic,
````
- **L235 EN**: Initializes variable `sizeExv` from the right-hand expression.
  **L235 CN**: 使用右侧表达式初始化变量 `sizeExv`。
- **L236 EN**: Continues the surrounding expression or declaration: `mlir::Value size =`.
  **L236 CN**: 继续构造周围的表达式或声明：`mlir::Value size =`。
- **L237 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L237 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L238 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L238 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L239 EN**: Continues the surrounding expression or declaration: `})`.
  **L239 CN**: 继续构造周围的表达式或声明：`})`。
- **L240 EN**: Starts a function, method, lambda, or structured scope: `.genElse([&]() {`.
  **L240 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genElse([&]() {`。
- **L241 EN**: Continues logic associated with callable symbol `createIntegerConstant`.
  **L241 CN**: 继续与可调用符号 `createIntegerConstant` 相关的逻辑。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, resultType,`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, resultType,`。
- **L243 EN**: Executes a call or declaration centered on `mlir::cast<mlir::IntegerType>`.
  **L243 CN**: 执行以 `mlir::cast<mlir::IntegerType>` 为核心的调用或声明。
- **L244 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L244 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L245 EN**: Continues the surrounding expression or declaration: `})`.
  **L245 CN**: 继续构造周围的表达式或声明：`})`。
- **L246 EN**: Executes a call or declaration centered on `.getResults`.
  **L246 CN**: 执行以 `.getResults` 为核心的调用或声明。
- **L247 EN**: Returns from the current function with `genIntrinsicCall(builder, loc, name, resultType, args, stmtCtx)`.
  **L247 CN**: 以 `genIntrinsicCall(builder, loc, name, resultType, args, stmtCtx)` 从当前函数返回。
- **L248 EN**: Closes the current lexical scope or compound statement.
  **L248 CN**: 结束当前词法作用域或复合语句块。
- **L249 EN**: Blank line separating nearby declarations or logic blocks.
  **L249 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L250 EN**: Continues logic associated with callable symbol `prepareAssociatedArguments`.
  **L250 CN**: 继续与可调用符号 `prepareAssociatedArguments` 相关的逻辑。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::ProcedureRef &procRef,`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::ProcedureRef &procRef,`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::SpecificIntrinsic &intrinsic,`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::SpecificIntrinsic &intrinsic,`。

### Lines 253-270

````cpp
    std::optional<mlir::Type> retTy,
    const Fortran::lower::OperandPrepare &prepareOptionalArgument,
    const Fortran::lower::OperandPrepareAs &prepareOtherArgument,
    Fortran::lower::AbstractConverter &converter) {
  const auto *pointer = procRef.UnwrapArgExpr(0);
  const auto *optionalTarget = procRef.UnwrapArgExpr(1);
  assert(pointer && optionalTarget &&
         "expected call to associated with a target");
  prepareOtherArgument(*pointer, fir::LowerIntrinsicArgAs::Inquired);
  prepareOptionalArgument(*optionalTarget);
}

static fir::ExtendedValue
lowerAssociated(fir::FirOpBuilder &builder, mlir::Location loc,
                llvm::StringRef name, std::optional<mlir::Type> resultType,
                const Fortran::lower::OperandPresent &isPresentCheck,
                const Fortran::lower::OperandGetter &getOperand,
                std::size_t numOperands,
````
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::optional<mlir::Type> retTy,`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::optional<mlir::Type> retTy,`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandPrepare &prepareOptionalArgument,`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandPrepare &prepareOptionalArgument,`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandPrepareAs &prepareOtherArgument,`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandPrepareAs &prepareOtherArgument,`。
- **L256 EN**: Continues the surrounding expression or declaration: `Fortran::lower::AbstractConverter &converter) {`.
  **L256 CN**: 继续构造周围的表达式或声明：`Fortran::lower::AbstractConverter &converter) {`。
- **L257 EN**: Executes a call or declaration centered on `procRef.UnwrapArgExpr`.
  **L257 CN**: 执行以 `procRef.UnwrapArgExpr` 为核心的调用或声明。
- **L258 EN**: Executes a call or declaration centered on `procRef.UnwrapArgExpr`.
  **L258 CN**: 执行以 `procRef.UnwrapArgExpr` 为核心的调用或声明。
- **L259 EN**: Checks an internal invariant in debug builds.
  **L259 CN**: 在调试构建中检查内部不变式。
- **L260 EN**: Executes a standalone statement or declaration: `"expected call to associated with a target");`.
  **L260 CN**: 执行一条独立语句或声明：`"expected call to associated with a target");`。
- **L261 EN**: Executes a call or declaration centered on `prepareOtherArgument`.
  **L261 CN**: 执行以 `prepareOtherArgument` 为核心的调用或声明。
- **L262 EN**: Executes a call or declaration centered on `prepareOptionalArgument`.
  **L262 CN**: 执行以 `prepareOptionalArgument` 为核心的调用或声明。
- **L263 EN**: Closes the current lexical scope or compound statement.
  **L263 CN**: 结束当前词法作用域或复合语句块。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L265 EN**: Continues the surrounding expression or declaration: `static fir::ExtendedValue`.
  **L265 CN**: 继续构造周围的表达式或声明：`static fir::ExtendedValue`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerAssociated(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerAssociated(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::StringRef name, std::optional<mlir::Type> resultType,`.
  **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::StringRef name, std::optional<mlir::Type> resultType,`。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandPresent &isPresentCheck,`.
  **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandPresent &isPresentCheck,`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::OperandGetter &getOperand,`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::OperandGetter &getOperand,`。
- **L270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::size_t numOperands,`.
  **L270 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::size_t numOperands,`。

### Lines 271-288

````cpp
                Fortran::lower::StatementContext &stmtCtx) {
  assert(numOperands == 2 && "expect two arguments when TARGET is OPTIONAL");
  llvm::SmallVector<fir::ExtendedValue> args;
  args.push_back(getOperand(0, /*loadOperand=*/false));
  // Ensure a null descriptor is passed to the code lowering Associated if
  // TARGET is absent.
  fir::ExtendedValue targetExv = getOperand(1, /*loadOperand=*/false);
  mlir::Value targetBase = fir::getBase(targetExv);
  // subtle: isPresentCheck would test for an unallocated/disassociated target,
  // while the optionality of the target pointer/allocatable is what must be
  // checked here.
  mlir::Value isPresent =
      fir::IsPresentOp::create(builder, loc, builder.getI1Type(), targetBase);
  mlir::Type targetType = fir::unwrapRefType(targetBase.getType());
  mlir::Type targetValueType = fir::unwrapPassByRefType(targetType);
  mlir::Type boxType = mlir::isa<fir::BaseBoxType>(targetType)
                           ? targetType
                           : fir::BoxType::get(targetValueType);
````
- **L271 EN**: Continues the surrounding expression or declaration: `Fortran::lower::StatementContext &stmtCtx) {`.
  **L271 CN**: 继续构造周围的表达式或声明：`Fortran::lower::StatementContext &stmtCtx) {`。
- **L272 EN**: Checks an internal invariant in debug builds.
  **L272 CN**: 在调试构建中检查内部不变式。
- **L273 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<fir::ExtendedValue> args;`.
  **L273 CN**: 执行一条独立语句或声明：`llvm::SmallVector<fir::ExtendedValue> args;`。
- **L274 EN**: Executes a call or declaration centered on `args.push_back`.
  **L274 CN**: 执行以 `args.push_back` 为核心的调用或声明。
- **L275 EN**: Comment explains nearby logic, intent, or metadata: `Ensure a null descriptor is passed to the code lowering Associated if`.
  **L275 CN**: 注释说明附近代码的逻辑、意图或元数据：`Ensure a null descriptor is passed to the code lowering Associated if`。
- **L276 EN**: Comment explains nearby logic, intent, or metadata: `TARGET is absent.`.
  **L276 CN**: 注释说明附近代码的逻辑、意图或元数据：`TARGET is absent.`。
- **L277 EN**: Initializes variable `targetExv` from the right-hand expression.
  **L277 CN**: 使用右侧表达式初始化变量 `targetExv`。
- **L278 EN**: Initializes variable `targetBase` from the right-hand expression.
  **L278 CN**: 使用右侧表达式初始化变量 `targetBase`。
- **L279 EN**: Comment explains nearby logic, intent, or metadata: `subtle: isPresentCheck would test for an unallocated/disassociated target,`.
  **L279 CN**: 注释说明附近代码的逻辑、意图或元数据：`subtle: isPresentCheck would test for an unallocated/disassociated target,`。
- **L280 EN**: Comment explains nearby logic, intent, or metadata: `while the optionality of the target pointer/allocatable is what must be`.
  **L280 CN**: 注释说明附近代码的逻辑、意图或元数据：`while the optionality of the target pointer/allocatable is what must be`。
- **L281 EN**: Comment explains nearby logic, intent, or metadata: `checked here.`.
  **L281 CN**: 注释说明附近代码的逻辑、意图或元数据：`checked here.`。
- **L282 EN**: Continues the surrounding expression or declaration: `mlir::Value isPresent =`.
  **L282 CN**: 继续构造周围的表达式或声明：`mlir::Value isPresent =`。
- **L283 EN**: Executes a call or declaration centered on `fir::IsPresentOp::create`.
  **L283 CN**: 执行以 `fir::IsPresentOp::create` 为核心的调用或声明。
- **L284 EN**: Initializes variable `targetType` from the right-hand expression.
  **L284 CN**: 使用右侧表达式初始化变量 `targetType`。
- **L285 EN**: Initializes variable `targetValueType` from the right-hand expression.
  **L285 CN**: 使用右侧表达式初始化变量 `targetValueType`。
- **L286 EN**: Continues logic associated with callable symbol `BaseBoxType>`.
  **L286 CN**: 继续与可调用符号 `BaseBoxType>` 相关的逻辑。
- **L287 EN**: Continues the surrounding expression or declaration: `? targetType`.
  **L287 CN**: 继续构造周围的表达式或声明：`? targetType`。
- **L288 EN**: Executes a call or declaration centered on `fir::BoxType::get`.
  **L288 CN**: 执行以 `fir::BoxType::get` 为核心的调用或声明。

### Lines 289-306

````cpp
  fir::BoxValue targetBox =
      builder
          .genIfOp(loc, {boxType}, isPresent,
                   /*withElseRegion=*/true)
          .genThen([&]() {
            mlir::Value box = builder.createBox(loc, targetExv);
            mlir::Value cast = builder.createConvert(loc, boxType, box);
            fir::ResultOp::create(builder, loc, cast);
          })
          .genElse([&]() {
            mlir::Value absentBox =
                fir::AbsentOp::create(builder, loc, boxType);
            fir::ResultOp::create(builder, loc, absentBox);
          })
          .getResults()[0];
  args.emplace_back(std::move(targetBox));
  return genIntrinsicCall(builder, loc, name, resultType, args, stmtCtx);
}
````
- **L289 EN**: Continues the surrounding expression or declaration: `fir::BoxValue targetBox =`.
  **L289 CN**: 继续构造周围的表达式或声明：`fir::BoxValue targetBox =`。
- **L290 EN**: Continues the surrounding expression or declaration: `builder`.
  **L290 CN**: 继续构造周围的表达式或声明：`builder`。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.genIfOp(loc, {boxType}, isPresent,`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`.genIfOp(loc, {boxType}, isPresent,`。
- **L292 EN**: Comment explains nearby logic, intent, or metadata: `withElseRegion=*/true)`.
  **L292 CN**: 注释说明附近代码的逻辑、意图或元数据：`withElseRegion=*/true)`。
- **L293 EN**: Starts a function, method, lambda, or structured scope: `.genThen([&]() {`.
  **L293 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genThen([&]() {`。
- **L294 EN**: Initializes variable `box` from the right-hand expression.
  **L294 CN**: 使用右侧表达式初始化变量 `box`。
- **L295 EN**: Initializes variable `cast` from the right-hand expression.
  **L295 CN**: 使用右侧表达式初始化变量 `cast`。
- **L296 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L296 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L297 EN**: Continues the surrounding expression or declaration: `})`.
  **L297 CN**: 继续构造周围的表达式或声明：`})`。
- **L298 EN**: Starts a function, method, lambda, or structured scope: `.genElse([&]() {`.
  **L298 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genElse([&]() {`。
- **L299 EN**: Continues the surrounding expression or declaration: `mlir::Value absentBox =`.
  **L299 CN**: 继续构造周围的表达式或声明：`mlir::Value absentBox =`。
- **L300 EN**: Executes a call or declaration centered on `fir::AbsentOp::create`.
  **L300 CN**: 执行以 `fir::AbsentOp::create` 为核心的调用或声明。
- **L301 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L301 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L302 EN**: Continues the surrounding expression or declaration: `})`.
  **L302 CN**: 继续构造周围的表达式或声明：`})`。
- **L303 EN**: Executes a call or declaration centered on `.getResults`.
  **L303 CN**: 执行以 `.getResults` 为核心的调用或声明。
- **L304 EN**: Executes a call or declaration centered on `args.emplace_back`.
  **L304 CN**: 执行以 `args.emplace_back` 为核心的调用或声明。
- **L305 EN**: Returns from the current function with `genIntrinsicCall(builder, loc, name, resultType, args, stmtCtx)`.
  **L305 CN**: 以 `genIntrinsicCall(builder, loc, name, resultType, args, stmtCtx)` 从当前函数返回。
- **L306 EN**: Closes the current lexical scope or compound statement.
  **L306 CN**: 结束当前词法作用域或复合语句块。

### Lines 307-324

````cpp

void Fortran::lower::prepareCustomIntrinsicArgument(
    const Fortran::evaluate::ProcedureRef &procRef,
    const Fortran::evaluate::SpecificIntrinsic &intrinsic,
    std::optional<mlir::Type> retTy,
    const OperandPrepare &prepareOptionalArgument,
    const OperandPrepareAs &prepareOtherArgument,
    AbstractConverter &converter) {
  llvm::StringRef name = intrinsic.name;
  if (name == "min" || name == "max")
    return prepareMinOrMaxArguments(procRef, intrinsic, retTy,
                                    prepareOptionalArgument,
                                    prepareOtherArgument, converter);
  if (name == "associated")
    return prepareAssociatedArguments(procRef, intrinsic, retTy,
                                      prepareOptionalArgument,
                                      prepareOtherArgument, converter);
  assert(name == "ishftc" && "unexpected custom intrinsic argument call");
````
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L308 EN**: Continues logic associated with callable symbol `prepareCustomIntrinsicArgument`.
  **L308 CN**: 继续与可调用符号 `prepareCustomIntrinsicArgument` 相关的逻辑。
- **L309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::ProcedureRef &procRef,`.
  **L309 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::ProcedureRef &procRef,`。
- **L310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::SpecificIntrinsic &intrinsic,`.
  **L310 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::SpecificIntrinsic &intrinsic,`。
- **L311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::optional<mlir::Type> retTy,`.
  **L311 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::optional<mlir::Type> retTy,`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const OperandPrepare &prepareOptionalArgument,`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`const OperandPrepare &prepareOptionalArgument,`。
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const OperandPrepareAs &prepareOtherArgument,`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`const OperandPrepareAs &prepareOtherArgument,`。
- **L314 EN**: Continues the surrounding expression or declaration: `AbstractConverter &converter) {`.
  **L314 CN**: 继续构造周围的表达式或声明：`AbstractConverter &converter) {`。
- **L315 EN**: Initializes variable `name` from the right-hand expression.
  **L315 CN**: 使用右侧表达式初始化变量 `name`。
- **L316 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L316 CN**: 开始 `if` 控制流语句并计算其条件。
- **L317 EN**: Returns from the current function with `prepareMinOrMaxArguments(procRef, intrinsic, retTy,`.
  **L317 CN**: 以 `prepareMinOrMaxArguments(procRef, intrinsic, retTy,` 从当前函数返回。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `prepareOptionalArgument,`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`prepareOptionalArgument,`。
- **L319 EN**: Executes a standalone statement or declaration: `prepareOtherArgument, converter);`.
  **L319 CN**: 执行一条独立语句或声明：`prepareOtherArgument, converter);`。
- **L320 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L320 CN**: 开始 `if` 控制流语句并计算其条件。
- **L321 EN**: Returns from the current function with `prepareAssociatedArguments(procRef, intrinsic, retTy,`.
  **L321 CN**: 以 `prepareAssociatedArguments(procRef, intrinsic, retTy,` 从当前函数返回。
- **L322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `prepareOptionalArgument,`.
  **L322 CN**: 继续一个多行参数列表、初始化器或聚合项：`prepareOptionalArgument,`。
- **L323 EN**: Executes a standalone statement or declaration: `prepareOtherArgument, converter);`.
  **L323 CN**: 执行一条独立语句或声明：`prepareOtherArgument, converter);`。
- **L324 EN**: Checks an internal invariant in debug builds.
  **L324 CN**: 在调试构建中检查内部不变式。

### Lines 325-342

````cpp
  return prepareIshftcArguments(procRef, intrinsic, retTy,
                                prepareOptionalArgument, prepareOtherArgument,
                                converter);
}

fir::ExtendedValue Fortran::lower::lowerCustomIntrinsic(
    fir::FirOpBuilder &builder, mlir::Location loc, llvm::StringRef name,
    std::optional<mlir::Type> retTy, const OperandPresent &isPresentCheck,
    const OperandGetter &getOperand, std::size_t numOperands,
    Fortran::lower::StatementContext &stmtCtx) {
  if (name == "min" || name == "max")
    return lowerMinOrMax(builder, loc, name, retTy, isPresentCheck, getOperand,
                         numOperands, stmtCtx);
  if (name == "associated")
    return lowerAssociated(builder, loc, name, retTy, isPresentCheck,
                           getOperand, numOperands, stmtCtx);
  assert(name == "ishftc" && "unexpected custom intrinsic call");
  return lowerIshftc(builder, loc, name, retTy, isPresentCheck, getOperand,
````
- **L325 EN**: Returns from the current function with `prepareIshftcArguments(procRef, intrinsic, retTy,`.
  **L325 CN**: 以 `prepareIshftcArguments(procRef, intrinsic, retTy,` 从当前函数返回。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `prepareOptionalArgument, prepareOtherArgument,`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`prepareOptionalArgument, prepareOtherArgument,`。
- **L327 EN**: Executes a standalone statement or declaration: `converter);`.
  **L327 CN**: 执行一条独立语句或声明：`converter);`。
- **L328 EN**: Closes the current lexical scope or compound statement.
  **L328 CN**: 结束当前词法作用域或复合语句块。
- **L329 EN**: Blank line separating nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L330 EN**: Continues logic associated with callable symbol `lowerCustomIntrinsic`.
  **L330 CN**: 继续与可调用符号 `lowerCustomIntrinsic` 相关的逻辑。
- **L331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::FirOpBuilder &builder, mlir::Location loc, llvm::StringRef name,`.
  **L331 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::FirOpBuilder &builder, mlir::Location loc, llvm::StringRef name,`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::optional<mlir::Type> retTy, const OperandPresent &isPresentCheck,`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::optional<mlir::Type> retTy, const OperandPresent &isPresentCheck,`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const OperandGetter &getOperand, std::size_t numOperands,`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`const OperandGetter &getOperand, std::size_t numOperands,`。
- **L334 EN**: Continues the surrounding expression or declaration: `Fortran::lower::StatementContext &stmtCtx) {`.
  **L334 CN**: 继续构造周围的表达式或声明：`Fortran::lower::StatementContext &stmtCtx) {`。
- **L335 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L335 CN**: 开始 `if` 控制流语句并计算其条件。
- **L336 EN**: Returns from the current function with `lowerMinOrMax(builder, loc, name, retTy, isPresentCheck, getOperand,`.
  **L336 CN**: 以 `lowerMinOrMax(builder, loc, name, retTy, isPresentCheck, getOperand,` 从当前函数返回。
- **L337 EN**: Executes a standalone statement or declaration: `numOperands, stmtCtx);`.
  **L337 CN**: 执行一条独立语句或声明：`numOperands, stmtCtx);`。
- **L338 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L338 CN**: 开始 `if` 控制流语句并计算其条件。
- **L339 EN**: Returns from the current function with `lowerAssociated(builder, loc, name, retTy, isPresentCheck,`.
  **L339 CN**: 以 `lowerAssociated(builder, loc, name, retTy, isPresentCheck,` 从当前函数返回。
- **L340 EN**: Executes a standalone statement or declaration: `getOperand, numOperands, stmtCtx);`.
  **L340 CN**: 执行一条独立语句或声明：`getOperand, numOperands, stmtCtx);`。
- **L341 EN**: Checks an internal invariant in debug builds.
  **L341 CN**: 在调试构建中检查内部不变式。
- **L342 EN**: Returns from the current function with `lowerIshftc(builder, loc, name, retTy, isPresentCheck, getOperand,`.
  **L342 CN**: 以 `lowerIshftc(builder, loc, name, retTy, isPresentCheck, getOperand,` 从当前函数返回。

### Lines 343-344

````cpp
                     numOperands, stmtCtx);
}
````
- **L343 EN**: Executes a standalone statement or declaration: `numOperands, stmtCtx);`.
  **L343 CN**: 执行一条独立语句或声明：`numOperands, stmtCtx);`。
- **L344 EN**: Closes the current lexical scope or compound statement.
  **L344 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Lowering to FIR/MLIR / 向 FIR/MLIR 的 lowering**
- **Symbol modeling and lookup / 符号建模与查找**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **Lowering converter orchestration / lowering 转换器编排**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**
- **IR builder orchestration / IR Builder 编排**
- **Runtime call integration / 运行时调用集成**

## Dependencies / 依赖关系

- `flang/Lower/CustomIntrinsicCall.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Evaluate/expression.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/fold.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/tools.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Lower/StatementContext.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Optimizer/Builder/IntrinsicCall.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/Todo.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `optional`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
