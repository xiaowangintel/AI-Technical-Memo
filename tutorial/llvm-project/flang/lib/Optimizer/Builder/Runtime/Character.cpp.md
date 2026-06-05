# Character.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/Builder/Runtime/Character.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements FIR/MLIR optimizer-side utilities and infrastructure for Character.
- **Purpose (CN)**: 实现 Character 相关的 FIR/MLIR 优化器辅助工具与基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===-- Character.cpp -- runtime for CHARACTER type entities --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Optimizer/Builder/Runtime/Character.h"
#include "flang/Optimizer/Builder/BoxValue.h"
#include "flang/Optimizer/Builder/Character.h"
#include "flang/Optimizer/Builder/FIRBuilder.h"
#include "flang/Optimizer/Builder/Runtime/RTBuilder.h"
#include "flang/Optimizer/Builder/Todo.h"
#include "flang/Runtime/character.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"

using namespace Fortran::runtime;
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
- **L9 EN**: Includes "flang/Optimizer/Builder/Runtime/Character.h" to access FIR builder helpers and runtime-construction utilities.
  **L9 CN**: 引入 "flang/Optimizer/Builder/Runtime/Character.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L10 EN**: Includes "flang/Optimizer/Builder/BoxValue.h" to access FIR builder helpers and runtime-construction utilities.
  **L10 CN**: 引入 "flang/Optimizer/Builder/BoxValue.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L11 EN**: Includes "flang/Optimizer/Builder/Character.h" to access FIR builder helpers and runtime-construction utilities.
  **L11 CN**: 引入 "flang/Optimizer/Builder/Character.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L12 EN**: Includes "flang/Optimizer/Builder/FIRBuilder.h" to access FIR builder helpers and runtime-construction utilities.
  **L12 CN**: 引入 "flang/Optimizer/Builder/FIRBuilder.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L13 EN**: Includes "flang/Optimizer/Builder/Runtime/RTBuilder.h" to access FIR builder helpers and runtime-construction utilities.
  **L13 CN**: 引入 "flang/Optimizer/Builder/Runtime/RTBuilder.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L14 EN**: Includes "flang/Optimizer/Builder/Todo.h" to access FIR builder helpers and runtime-construction utilities.
  **L14 CN**: 引入 "flang/Optimizer/Builder/Todo.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L15 EN**: Includes "flang/Runtime/character.h" to access Fortran runtime entry points and descriptor helpers.
  **L15 CN**: 引入 "flang/Runtime/character.h" 以使用Fortran 运行时入口与描述符辅助能力。
- **L16 EN**: Includes "mlir/Dialect/Func/IR/FuncOps.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L16 CN**: 引入 "mlir/Dialect/Func/IR/FuncOps.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L18 EN**: Brings namespace `Fortran::runtime` into the local scope.
  **L18 CN**: 将命名空间 `Fortran::runtime` 引入当前作用域。

### Lines 19-36

````cpp

/// Generate calls to string handling intrinsics such as index, scan, and
/// verify. These are the descriptor based implementations that take four
/// arguments (string1, string2, back, kind).
template <typename FN>
static void genCharacterSearch(FN func, fir::FirOpBuilder &builder,
                               mlir::Location loc, mlir::Value resultBox,
                               mlir::Value string1Box, mlir::Value string2Box,
                               mlir::Value backBox, mlir::Value kind) {

  auto fTy = func.getFunctionType();
  auto sourceFile = fir::factory::locationToFilename(builder, loc);
  auto sourceLine =
      fir::factory::locationToLineNo(builder, loc, fTy.getInput(6));

  auto args = fir::runtime::createArguments(builder, loc, fTy, resultBox,
                                            string1Box, string2Box, backBox,
                                            kind, sourceFile, sourceLine);
````
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Comment explains nearby logic, intent, or metadata: `Generate calls to string handling intrinsics such as index, scan, and`.
  **L20 CN**: 注释说明附近代码的逻辑、意图或元数据：`Generate calls to string handling intrinsics such as index, scan, and`。
- **L21 EN**: Comment explains nearby logic, intent, or metadata: `verify. These are the descriptor based implementations that take four`.
  **L21 CN**: 注释说明附近代码的逻辑、意图或元数据：`verify. These are the descriptor based implementations that take four`。
- **L22 EN**: Comment explains nearby logic, intent, or metadata: `arguments (string1, string2, back, kind).`.
  **L22 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments (string1, string2, back, kind).`。
- **L23 EN**: Introduces template parameters or specialization context: `template <typename FN>`.
  **L23 CN**: 为后续声明引入模板参数或特化上下文：`template <typename FN>`。
- **L24 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void genCharacterSearch(FN func, fir::FirOpBuilder &builder,`.
  **L24 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void genCharacterSearch(FN func, fir::FirOpBuilder &builder,`。
- **L25 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, mlir::Value resultBox,`.
  **L25 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, mlir::Value resultBox,`。
- **L26 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value string1Box, mlir::Value string2Box,`.
  **L26 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value string1Box, mlir::Value string2Box,`。
- **L27 EN**: Continues the surrounding expression or declaration: `mlir::Value backBox, mlir::Value kind) {`.
  **L27 CN**: 继续构造周围的表达式或声明：`mlir::Value backBox, mlir::Value kind) {`。
- **L28 EN**: Blank line separating nearby declarations or logic blocks.
  **L28 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L29 EN**: Initializes variable `fTy` from the right-hand expression.
  **L29 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L30 EN**: Initializes variable `sourceFile` from the right-hand expression.
  **L30 CN**: 使用右侧表达式初始化变量 `sourceFile`。
- **L31 EN**: Continues the surrounding expression or declaration: `auto sourceLine =`.
  **L31 CN**: 继续构造周围的表达式或声明：`auto sourceLine =`。
- **L32 EN**: Executes a call or declaration centered on `fir::factory::locationToLineNo`.
  **L32 CN**: 执行以 `fir::factory::locationToLineNo` 为核心的调用或声明。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L34 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto args = fir::runtime::createArguments(builder, loc, fTy, resultBox,`.
  **L34 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto args = fir::runtime::createArguments(builder, loc, fTy, resultBox,`。
- **L35 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `string1Box, string2Box, backBox,`.
  **L35 CN**: 继续一个多行参数列表、初始化器或聚合项：`string1Box, string2Box, backBox,`。
- **L36 EN**: Executes a standalone statement or declaration: `kind, sourceFile, sourceLine);`.
  **L36 CN**: 执行一条独立语句或声明：`kind, sourceFile, sourceLine);`。

### Lines 37-54

````cpp
  fir::CallOp::create(builder, loc, func, args);
}

/// Helper function to recover the KIND from the FIR type.
static int discoverKind(mlir::Type ty) {
  if (auto charTy = mlir::dyn_cast<fir::CharacterType>(ty))
    return charTy.getFKind();
  if (auto eleTy = fir::dyn_cast_ptrEleTy(ty))
    return discoverKind(eleTy);
  if (auto arrTy = mlir::dyn_cast<fir::SequenceType>(ty))
    return discoverKind(arrTy.getEleTy());
  if (auto boxTy = mlir::dyn_cast<fir::BoxCharType>(ty))
    return discoverKind(boxTy.getEleTy());
  if (auto boxTy = mlir::dyn_cast<fir::BoxType>(ty))
    return discoverKind(boxTy.getEleTy());
  llvm_unreachable("unexpected character type");
}

````
- **L37 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L37 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L38 EN**: Closes the current lexical scope or compound statement.
  **L38 CN**: 结束当前词法作用域或复合语句块。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L40 EN**: Comment explains nearby logic, intent, or metadata: `Helper function to recover the KIND from the FIR type.`.
  **L40 CN**: 注释说明附近代码的逻辑、意图或元数据：`Helper function to recover the KIND from the FIR type.`。
- **L41 EN**: Starts a function, method, lambda, or structured scope: `static int discoverKind(mlir::Type ty) {`.
  **L41 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static int discoverKind(mlir::Type ty) {`。
- **L42 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L42 CN**: 开始 `if` 控制流语句并计算其条件。
- **L43 EN**: Returns from the current function with `charTy.getFKind()`.
  **L43 CN**: 以 `charTy.getFKind()` 从当前函数返回。
- **L44 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L44 CN**: 开始 `if` 控制流语句并计算其条件。
- **L45 EN**: Returns from the current function with `discoverKind(eleTy)`.
  **L45 CN**: 以 `discoverKind(eleTy)` 从当前函数返回。
- **L46 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L46 CN**: 开始 `if` 控制流语句并计算其条件。
- **L47 EN**: Returns from the current function with `discoverKind(arrTy.getEleTy())`.
  **L47 CN**: 以 `discoverKind(arrTy.getEleTy())` 从当前函数返回。
- **L48 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L48 CN**: 开始 `if` 控制流语句并计算其条件。
- **L49 EN**: Returns from the current function with `discoverKind(boxTy.getEleTy())`.
  **L49 CN**: 以 `discoverKind(boxTy.getEleTy())` 从当前函数返回。
- **L50 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L50 CN**: 开始 `if` 控制流语句并计算其条件。
- **L51 EN**: Returns from the current function with `discoverKind(boxTy.getEleTy())`.
  **L51 CN**: 以 `discoverKind(boxTy.getEleTy())` 从当前函数返回。
- **L52 EN**: Marks this control path as unreachable to LLVM.
  **L52 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L53 EN**: Closes the current lexical scope or compound statement.
  **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 55-72

````cpp
//===----------------------------------------------------------------------===//
// Lower character operations
//===----------------------------------------------------------------------===//

/// Generate a call to the `ADJUST[L|R]` runtime.
///
/// \p resultBox must be an unallocated allocatable used for the temporary
/// result.  \p StringBox must be a fir.box describing the adjustr string
/// argument.  The \p adjustFunc should be a mlir::func::FuncOp for the
/// appropriate runtime entry function.
static void genAdjust(fir::FirOpBuilder &builder, mlir::Location loc,
                      mlir::Value resultBox, mlir::Value stringBox,
                      mlir::func::FuncOp &adjustFunc) {

  auto fTy = adjustFunc.getFunctionType();
  auto sourceLine =
      fir::factory::locationToLineNo(builder, loc, fTy.getInput(3));
  auto sourceFile = fir::factory::locationToFilename(builder, loc);
````
- **L55 EN**: Banner comment marking a file or section boundary.
  **L55 CN**: 横幅注释，用于标记文件或章节边界。
- **L56 EN**: Comment explains nearby logic, intent, or metadata: `Lower character operations`.
  **L56 CN**: 注释说明附近代码的逻辑、意图或元数据：`Lower character operations`。
- **L57 EN**: Banner comment marking a file or section boundary.
  **L57 CN**: 横幅注释，用于标记文件或章节边界。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L59 EN**: Comment explains nearby logic, intent, or metadata: `Generate a call to the `ADJUST[L|R]` runtime.`.
  **L59 CN**: 注释说明附近代码的逻辑、意图或元数据：`Generate a call to the `ADJUST[L|R]` runtime.`。
- **L60 EN**: Separator comment used for visual grouping.
  **L60 CN**: 用于视觉分组的分隔注释。
- **L61 EN**: Comment explains nearby logic, intent, or metadata: `\p resultBox must be an unallocated allocatable used for the temporary`.
  **L61 CN**: 注释说明附近代码的逻辑、意图或元数据：`\p resultBox must be an unallocated allocatable used for the temporary`。
- **L62 EN**: Comment explains nearby logic, intent, or metadata: `result.  \p StringBox must be a fir.box describing the adjustr string`.
  **L62 CN**: 注释说明附近代码的逻辑、意图或元数据：`result.  \p StringBox must be a fir.box describing the adjustr string`。
- **L63 EN**: Comment explains nearby logic, intent, or metadata: `argument.  The \p adjustFunc should be a mlir::func::FuncOp for the`.
  **L63 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument.  The \p adjustFunc should be a mlir::func::FuncOp for the`。
- **L64 EN**: Comment explains nearby logic, intent, or metadata: `appropriate runtime entry function.`.
  **L64 CN**: 注释说明附近代码的逻辑、意图或元数据：`appropriate runtime entry function.`。
- **L65 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void genAdjust(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L65 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void genAdjust(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L66 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value resultBox, mlir::Value stringBox,`.
  **L66 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value resultBox, mlir::Value stringBox,`。
- **L67 EN**: Continues the surrounding expression or declaration: `mlir::func::FuncOp &adjustFunc) {`.
  **L67 CN**: 继续构造周围的表达式或声明：`mlir::func::FuncOp &adjustFunc) {`。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L69 EN**: Initializes variable `fTy` from the right-hand expression.
  **L69 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L70 EN**: Continues the surrounding expression or declaration: `auto sourceLine =`.
  **L70 CN**: 继续构造周围的表达式或声明：`auto sourceLine =`。
- **L71 EN**: Executes a call or declaration centered on `fir::factory::locationToLineNo`.
  **L71 CN**: 执行以 `fir::factory::locationToLineNo` 为核心的调用或声明。
- **L72 EN**: Initializes variable `sourceFile` from the right-hand expression.
  **L72 CN**: 使用右侧表达式初始化变量 `sourceFile`。

### Lines 73-90

````cpp
  auto args = fir::runtime::createArguments(builder, loc, fTy, resultBox,
                                            stringBox, sourceFile, sourceLine);
  fir::CallOp::create(builder, loc, adjustFunc, args);
}

void fir::runtime::genAdjustL(fir::FirOpBuilder &builder, mlir::Location loc,
                              mlir::Value resultBox, mlir::Value stringBox) {
  auto adjustFunc =
      fir::runtime::getRuntimeFunc<mkRTKey(Adjustl)>(loc, builder);
  genAdjust(builder, loc, resultBox, stringBox, adjustFunc);
}

void fir::runtime::genAdjustR(fir::FirOpBuilder &builder, mlir::Location loc,
                              mlir::Value resultBox, mlir::Value stringBox) {
  auto adjustFunc =
      fir::runtime::getRuntimeFunc<mkRTKey(Adjustr)>(loc, builder);
  genAdjust(builder, loc, resultBox, stringBox, adjustFunc);
}
````
- **L73 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto args = fir::runtime::createArguments(builder, loc, fTy, resultBox,`.
  **L73 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto args = fir::runtime::createArguments(builder, loc, fTy, resultBox,`。
- **L74 EN**: Executes a standalone statement or declaration: `stringBox, sourceFile, sourceLine);`.
  **L74 CN**: 执行一条独立语句或声明：`stringBox, sourceFile, sourceLine);`。
- **L75 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L75 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L76 EN**: Closes the current lexical scope or compound statement.
  **L76 CN**: 结束当前词法作用域或复合语句块。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L78 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::runtime::genAdjustL(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L78 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::runtime::genAdjustL(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L79 EN**: Continues the surrounding expression or declaration: `mlir::Value resultBox, mlir::Value stringBox) {`.
  **L79 CN**: 继续构造周围的表达式或声明：`mlir::Value resultBox, mlir::Value stringBox) {`。
- **L80 EN**: Continues the surrounding expression or declaration: `auto adjustFunc =`.
  **L80 CN**: 继续构造周围的表达式或声明：`auto adjustFunc =`。
- **L81 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L81 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L82 EN**: Executes a call or declaration centered on `genAdjust`.
  **L82 CN**: 执行以 `genAdjust` 为核心的调用或声明。
- **L83 EN**: Closes the current lexical scope or compound statement.
  **L83 CN**: 结束当前词法作用域或复合语句块。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L85 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::runtime::genAdjustR(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L85 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::runtime::genAdjustR(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L86 EN**: Continues the surrounding expression or declaration: `mlir::Value resultBox, mlir::Value stringBox) {`.
  **L86 CN**: 继续构造周围的表达式或声明：`mlir::Value resultBox, mlir::Value stringBox) {`。
- **L87 EN**: Continues the surrounding expression or declaration: `auto adjustFunc =`.
  **L87 CN**: 继续构造周围的表达式或声明：`auto adjustFunc =`。
- **L88 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L88 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L89 EN**: Executes a call or declaration centered on `genAdjust`.
  **L89 CN**: 执行以 `genAdjust` 为核心的调用或声明。
- **L90 EN**: Closes the current lexical scope or compound statement.
  **L90 CN**: 结束当前词法作用域或复合语句块。

### Lines 91-108

````cpp

mlir::Value
fir::runtime::genCharCompare(fir::FirOpBuilder &builder, mlir::Location loc,
                             mlir::arith::CmpIPredicate cmp,
                             mlir::Value lhsBuff, mlir::Value lhsLen,
                             mlir::Value rhsBuff, mlir::Value rhsLen) {
  int lhsKind = discoverKind(lhsBuff.getType());
  int rhsKind = discoverKind(rhsBuff.getType());
  if (lhsKind != rhsKind) {
    fir::emitFatalError(loc, "runtime does not support comparison of different "
                             "CHARACTER kind values");
  }
  mlir::func::FuncOp func;
  switch (lhsKind) {
  case 1:
    func = fir::runtime::getRuntimeFunc<mkRTKey(CharacterCompareScalar1)>(
        loc, builder);
    break;
````
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L92 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L92 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::runtime::genCharCompare(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::runtime::genCharCompare(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::arith::CmpIPredicate cmp,`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::arith::CmpIPredicate cmp,`。
- **L95 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value lhsBuff, mlir::Value lhsLen,`.
  **L95 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value lhsBuff, mlir::Value lhsLen,`。
- **L96 EN**: Continues the surrounding expression or declaration: `mlir::Value rhsBuff, mlir::Value rhsLen) {`.
  **L96 CN**: 继续构造周围的表达式或声明：`mlir::Value rhsBuff, mlir::Value rhsLen) {`。
- **L97 EN**: Initializes variable `lhsKind` from the right-hand expression.
  **L97 CN**: 使用右侧表达式初始化变量 `lhsKind`。
- **L98 EN**: Initializes variable `rhsKind` from the right-hand expression.
  **L98 CN**: 使用右侧表达式初始化变量 `rhsKind`。
- **L99 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L99 CN**: 开始 `if` 控制流语句并计算其条件。
- **L100 EN**: Continues logic associated with callable symbol `emitFatalError`.
  **L100 CN**: 继续与可调用符号 `emitFatalError` 相关的逻辑。
- **L101 EN**: Executes a standalone statement or declaration: `"CHARACTER kind values");`.
  **L101 CN**: 执行一条独立语句或声明：`"CHARACTER kind values");`。
- **L102 EN**: Closes the current lexical scope or compound statement.
  **L102 CN**: 结束当前词法作用域或复合语句块。
- **L103 EN**: Executes a standalone statement or declaration: `mlir::func::FuncOp func;`.
  **L103 CN**: 执行一条独立语句或声明：`mlir::func::FuncOp func;`。
- **L104 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L104 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L105 EN**: Introduces a switch dispatch label: `case 1:`.
  **L105 CN**: 引入一个 switch 分发标签：`case 1:`。
- **L106 EN**: Continues logic associated with callable symbol `getRuntimeFunc<mkRTKey`.
  **L106 CN**: 继续与可调用符号 `getRuntimeFunc<mkRTKey` 相关的逻辑。
- **L107 EN**: Executes a standalone statement or declaration: `loc, builder);`.
  **L107 CN**: 执行一条独立语句或声明：`loc, builder);`。
- **L108 EN**: Exits the nearest loop or switch statement.
  **L108 CN**: 退出最近的循环或 switch 语句。

### Lines 109-126

````cpp
  case 2:
    func = fir::runtime::getRuntimeFunc<mkRTKey(CharacterCompareScalar2)>(
        loc, builder);
    break;
  case 4:
    func = fir::runtime::getRuntimeFunc<mkRTKey(CharacterCompareScalar4)>(
        loc, builder);
    break;
  default:
    fir::emitFatalError(
        loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");
  }
  auto fTy = func.getFunctionType();
  auto args = fir::runtime::createArguments(builder, loc, fTy, lhsBuff, rhsBuff,
                                            lhsLen, rhsLen);
  auto tri = fir::CallOp::create(builder, loc, func, args).getResult(0);
  auto zero = builder.createIntegerConstant(loc, tri.getType(), 0);
  return mlir::arith::CmpIOp::create(builder, loc, cmp, tri, zero);
````
- **L109 EN**: Introduces a switch dispatch label: `case 2:`.
  **L109 CN**: 引入一个 switch 分发标签：`case 2:`。
- **L110 EN**: Continues logic associated with callable symbol `getRuntimeFunc<mkRTKey`.
  **L110 CN**: 继续与可调用符号 `getRuntimeFunc<mkRTKey` 相关的逻辑。
- **L111 EN**: Executes a standalone statement or declaration: `loc, builder);`.
  **L111 CN**: 执行一条独立语句或声明：`loc, builder);`。
- **L112 EN**: Exits the nearest loop or switch statement.
  **L112 CN**: 退出最近的循环或 switch 语句。
- **L113 EN**: Introduces a switch dispatch label: `case 4:`.
  **L113 CN**: 引入一个 switch 分发标签：`case 4:`。
- **L114 EN**: Continues logic associated with callable symbol `getRuntimeFunc<mkRTKey`.
  **L114 CN**: 继续与可调用符号 `getRuntimeFunc<mkRTKey` 相关的逻辑。
- **L115 EN**: Executes a standalone statement or declaration: `loc, builder);`.
  **L115 CN**: 执行一条独立语句或声明：`loc, builder);`。
- **L116 EN**: Exits the nearest loop or switch statement.
  **L116 CN**: 退出最近的循环或 switch 语句。
- **L117 EN**: Introduces a switch dispatch label: `default:`.
  **L117 CN**: 引入一个 switch 分发标签：`default:`。
- **L118 EN**: Continues logic associated with callable symbol `emitFatalError`.
  **L118 CN**: 继续与可调用符号 `emitFatalError` 相关的逻辑。
- **L119 EN**: Executes a standalone statement or declaration: `loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");`.
  **L119 CN**: 执行一条独立语句或声明：`loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");`。
- **L120 EN**: Closes the current lexical scope or compound statement.
  **L120 CN**: 结束当前词法作用域或复合语句块。
- **L121 EN**: Initializes variable `fTy` from the right-hand expression.
  **L121 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto args = fir::runtime::createArguments(builder, loc, fTy, lhsBuff, rhsBuff,`.
  **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto args = fir::runtime::createArguments(builder, loc, fTy, lhsBuff, rhsBuff,`。
- **L123 EN**: Executes a standalone statement or declaration: `lhsLen, rhsLen);`.
  **L123 CN**: 执行一条独立语句或声明：`lhsLen, rhsLen);`。
- **L124 EN**: Initializes variable `tri` from the right-hand expression.
  **L124 CN**: 使用右侧表达式初始化变量 `tri`。
- **L125 EN**: Initializes variable `zero` from the right-hand expression.
  **L125 CN**: 使用右侧表达式初始化变量 `zero`。
- **L126 EN**: Returns from the current function with `mlir::arith::CmpIOp::create(builder, loc, cmp, tri, zero)`.
  **L126 CN**: 以 `mlir::arith::CmpIOp::create(builder, loc, cmp, tri, zero)` 从当前函数返回。

### Lines 127-144

````cpp
}

static mlir::Value allocateIfNotInMemory(fir::FirOpBuilder &builder,
                                         mlir::Location loc, mlir::Value base) {
  if (fir::isa_ref_type(base.getType()))
    return base;
  auto mem =
      fir::AllocaOp::create(builder, loc, base.getType(), /*pinned=*/false);
  fir::StoreOp::create(builder, loc, base, mem);
  return mem;
}

mlir::Value fir::runtime::genCharCompare(fir::FirOpBuilder &builder,
                                         mlir::Location loc,
                                         mlir::arith::CmpIPredicate cmp,
                                         const fir::ExtendedValue &lhs,
                                         const fir::ExtendedValue &rhs) {
  auto lhsBuffer = allocateIfNotInMemory(builder, loc, fir::getBase(lhs));
````
- **L127 EN**: Closes the current lexical scope or compound statement.
  **L127 CN**: 结束当前词法作用域或复合语句块。
- **L128 EN**: Blank line separating nearby declarations or logic blocks.
  **L128 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static mlir::Value allocateIfNotInMemory(fir::FirOpBuilder &builder,`.
  **L129 CN**: 继续一个多行参数列表、初始化器或聚合项：`static mlir::Value allocateIfNotInMemory(fir::FirOpBuilder &builder,`。
- **L130 EN**: Continues the surrounding expression or declaration: `mlir::Location loc, mlir::Value base) {`.
  **L130 CN**: 继续构造周围的表达式或声明：`mlir::Location loc, mlir::Value base) {`。
- **L131 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L131 CN**: 开始 `if` 控制流语句并计算其条件。
- **L132 EN**: Returns from the current function with `base`.
  **L132 CN**: 以 `base` 从当前函数返回。
- **L133 EN**: Continues the surrounding expression or declaration: `auto mem =`.
  **L133 CN**: 继续构造周围的表达式或声明：`auto mem =`。
- **L134 EN**: Executes a call or declaration centered on `fir::AllocaOp::create`.
  **L134 CN**: 执行以 `fir::AllocaOp::create` 为核心的调用或声明。
- **L135 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L135 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L136 EN**: Returns from the current function with `mem`.
  **L136 CN**: 以 `mem` 从当前函数返回。
- **L137 EN**: Closes the current lexical scope or compound statement.
  **L137 CN**: 结束当前词法作用域或复合语句块。
- **L138 EN**: Blank line separating nearby declarations or logic blocks.
  **L138 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value fir::runtime::genCharCompare(fir::FirOpBuilder &builder,`.
  **L139 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value fir::runtime::genCharCompare(fir::FirOpBuilder &builder,`。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc,`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc,`。
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::arith::CmpIPredicate cmp,`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::arith::CmpIPredicate cmp,`。
- **L142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::ExtendedValue &lhs,`.
  **L142 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::ExtendedValue &lhs,`。
- **L143 EN**: Continues the surrounding expression or declaration: `const fir::ExtendedValue &rhs) {`.
  **L143 CN**: 继续构造周围的表达式或声明：`const fir::ExtendedValue &rhs) {`。
- **L144 EN**: Initializes variable `lhsBuffer` from the right-hand expression.
  **L144 CN**: 使用右侧表达式初始化变量 `lhsBuffer`。

### Lines 145-162

````cpp
  auto rhsBuffer = allocateIfNotInMemory(builder, loc, fir::getBase(rhs));
  return genCharCompare(builder, loc, cmp, lhsBuffer, fir::getLen(lhs),
                        rhsBuffer, fir::getLen(rhs));
}

void fir::runtime::genFCString(fir::FirOpBuilder &builder, mlir::Location loc,
                               mlir::Value resultBox, mlir::Value stringBox,
                               mlir::Value asis) {
  auto func = fir::runtime::getRuntimeFunc<mkRTKey(FCString)>(loc, builder);
  auto fTy = func.getFunctionType();
  auto sourceFile = fir::factory::locationToFilename(builder, loc);
  auto sourceLine =
      fir::factory::locationToLineNo(builder, loc, fTy.getInput(4));
  auto args = fir::runtime::createArguments(
      builder, loc, fTy, resultBox, stringBox, asis, sourceFile, sourceLine);
  fir::CallOp::create(builder, loc, func, args);
}

````
- **L145 EN**: Initializes variable `rhsBuffer` from the right-hand expression.
  **L145 CN**: 使用右侧表达式初始化变量 `rhsBuffer`。
- **L146 EN**: Returns from the current function with `genCharCompare(builder, loc, cmp, lhsBuffer, fir::getLen(lhs),`.
  **L146 CN**: 以 `genCharCompare(builder, loc, cmp, lhsBuffer, fir::getLen(lhs),` 从当前函数返回。
- **L147 EN**: Executes a call or declaration centered on `fir::getLen`.
  **L147 CN**: 执行以 `fir::getLen` 为核心的调用或声明。
- **L148 EN**: Closes the current lexical scope or compound statement.
  **L148 CN**: 结束当前词法作用域或复合语句块。
- **L149 EN**: Blank line separating nearby declarations or logic blocks.
  **L149 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::runtime::genFCString(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::runtime::genFCString(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value resultBox, mlir::Value stringBox,`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value resultBox, mlir::Value stringBox,`。
- **L152 EN**: Continues the surrounding expression or declaration: `mlir::Value asis) {`.
  **L152 CN**: 继续构造周围的表达式或声明：`mlir::Value asis) {`。
- **L153 EN**: Initializes variable `func` from the right-hand expression.
  **L153 CN**: 使用右侧表达式初始化变量 `func`。
- **L154 EN**: Initializes variable `fTy` from the right-hand expression.
  **L154 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L155 EN**: Initializes variable `sourceFile` from the right-hand expression.
  **L155 CN**: 使用右侧表达式初始化变量 `sourceFile`。
- **L156 EN**: Continues the surrounding expression or declaration: `auto sourceLine =`.
  **L156 CN**: 继续构造周围的表达式或声明：`auto sourceLine =`。
- **L157 EN**: Executes a call or declaration centered on `fir::factory::locationToLineNo`.
  **L157 CN**: 执行以 `fir::factory::locationToLineNo` 为核心的调用或声明。
- **L158 EN**: Continues logic associated with callable symbol `createArguments`.
  **L158 CN**: 继续与可调用符号 `createArguments` 相关的逻辑。
- **L159 EN**: Executes a standalone statement or declaration: `builder, loc, fTy, resultBox, stringBox, asis, sourceFile, sourceLine);`.
  **L159 CN**: 执行一条独立语句或声明：`builder, loc, fTy, resultBox, stringBox, asis, sourceFile, sourceLine);`。
- **L160 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L160 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L161 EN**: Closes the current lexical scope or compound statement.
  **L161 CN**: 结束当前词法作用域或复合语句块。
- **L162 EN**: Blank line separating nearby declarations or logic blocks.
  **L162 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 163-180

````cpp
mlir::Value fir::runtime::genIndex(fir::FirOpBuilder &builder,
                                   mlir::Location loc, int kind,
                                   mlir::Value stringBase,
                                   mlir::Value stringLen,
                                   mlir::Value substringBase,
                                   mlir::Value substringLen, mlir::Value back) {
  mlir::func::FuncOp indexFunc;
  switch (kind) {
  case 1:
    indexFunc = fir::runtime::getRuntimeFunc<mkRTKey(Index1)>(loc, builder);
    break;
  case 2:
    indexFunc = fir::runtime::getRuntimeFunc<mkRTKey(Index2)>(loc, builder);
    break;
  case 4:
    indexFunc = fir::runtime::getRuntimeFunc<mkRTKey(Index4)>(loc, builder);
    break;
  default:
````
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value fir::runtime::genIndex(fir::FirOpBuilder &builder,`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value fir::runtime::genIndex(fir::FirOpBuilder &builder,`。
- **L164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, int kind,`.
  **L164 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, int kind,`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value stringBase,`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value stringBase,`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value stringLen,`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value stringLen,`。
- **L167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value substringBase,`.
  **L167 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value substringBase,`。
- **L168 EN**: Continues the surrounding expression or declaration: `mlir::Value substringLen, mlir::Value back) {`.
  **L168 CN**: 继续构造周围的表达式或声明：`mlir::Value substringLen, mlir::Value back) {`。
- **L169 EN**: Executes a standalone statement or declaration: `mlir::func::FuncOp indexFunc;`.
  **L169 CN**: 执行一条独立语句或声明：`mlir::func::FuncOp indexFunc;`。
- **L170 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L170 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L171 EN**: Introduces a switch dispatch label: `case 1:`.
  **L171 CN**: 引入一个 switch 分发标签：`case 1:`。
- **L172 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L172 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L173 EN**: Exits the nearest loop or switch statement.
  **L173 CN**: 退出最近的循环或 switch 语句。
- **L174 EN**: Introduces a switch dispatch label: `case 2:`.
  **L174 CN**: 引入一个 switch 分发标签：`case 2:`。
- **L175 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L175 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L176 EN**: Exits the nearest loop or switch statement.
  **L176 CN**: 退出最近的循环或 switch 语句。
- **L177 EN**: Introduces a switch dispatch label: `case 4:`.
  **L177 CN**: 引入一个 switch 分发标签：`case 4:`。
- **L178 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L178 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L179 EN**: Exits the nearest loop or switch statement.
  **L179 CN**: 退出最近的循环或 switch 语句。
- **L180 EN**: Introduces a switch dispatch label: `default:`.
  **L180 CN**: 引入一个 switch 分发标签：`default:`。

### Lines 181-198

````cpp
    fir::emitFatalError(
        loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");
  }
  auto fTy = indexFunc.getFunctionType();
  auto args =
      fir::runtime::createArguments(builder, loc, fTy, stringBase, stringLen,
                                    substringBase, substringLen, back);
  return fir::CallOp::create(builder, loc, indexFunc, args).getResult(0);
}

mlir::Value fir::runtime::genIndex(fir::FirOpBuilder &builder,
                                   mlir::Location loc,
                                   const fir::ExtendedValue &str,
                                   const fir::ExtendedValue &substr,
                                   mlir::Value back) {
  assert(!substr.getBoxOf<fir::BoxValue>() && !str.getBoxOf<fir::BoxValue>() &&
         "shall use genIndexDescriptor version");
  auto strBuffer = allocateIfNotInMemory(builder, loc, fir::getBase(str));
````
- **L181 EN**: Continues logic associated with callable symbol `emitFatalError`.
  **L181 CN**: 继续与可调用符号 `emitFatalError` 相关的逻辑。
- **L182 EN**: Executes a standalone statement or declaration: `loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");`.
  **L182 CN**: 执行一条独立语句或声明：`loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");`。
- **L183 EN**: Closes the current lexical scope or compound statement.
  **L183 CN**: 结束当前词法作用域或复合语句块。
- **L184 EN**: Initializes variable `fTy` from the right-hand expression.
  **L184 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L185 EN**: Continues the surrounding expression or declaration: `auto args =`.
  **L185 CN**: 继续构造周围的表达式或声明：`auto args =`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::runtime::createArguments(builder, loc, fTy, stringBase, stringLen,`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::runtime::createArguments(builder, loc, fTy, stringBase, stringLen,`。
- **L187 EN**: Executes a standalone statement or declaration: `substringBase, substringLen, back);`.
  **L187 CN**: 执行一条独立语句或声明：`substringBase, substringLen, back);`。
- **L188 EN**: Returns from the current function with `fir::CallOp::create(builder, loc, indexFunc, args).getResult(0)`.
  **L188 CN**: 以 `fir::CallOp::create(builder, loc, indexFunc, args).getResult(0)` 从当前函数返回。
- **L189 EN**: Closes the current lexical scope or compound statement.
  **L189 CN**: 结束当前词法作用域或复合语句块。
- **L190 EN**: Blank line separating nearby declarations or logic blocks.
  **L190 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value fir::runtime::genIndex(fir::FirOpBuilder &builder,`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value fir::runtime::genIndex(fir::FirOpBuilder &builder,`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc,`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc,`。
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::ExtendedValue &str,`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::ExtendedValue &str,`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::ExtendedValue &substr,`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::ExtendedValue &substr,`。
- **L195 EN**: Continues the surrounding expression or declaration: `mlir::Value back) {`.
  **L195 CN**: 继续构造周围的表达式或声明：`mlir::Value back) {`。
- **L196 EN**: Checks an internal invariant in debug builds.
  **L196 CN**: 在调试构建中检查内部不变式。
- **L197 EN**: Executes a standalone statement or declaration: `"shall use genIndexDescriptor version");`.
  **L197 CN**: 执行一条独立语句或声明：`"shall use genIndexDescriptor version");`。
- **L198 EN**: Initializes variable `strBuffer` from the right-hand expression.
  **L198 CN**: 使用右侧表达式初始化变量 `strBuffer`。

### Lines 199-216

````cpp
  auto substrBuffer = allocateIfNotInMemory(builder, loc, fir::getBase(substr));
  int kind = discoverKind(strBuffer.getType());
  return genIndex(builder, loc, kind, strBuffer, fir::getLen(str), substrBuffer,
                  fir::getLen(substr), back);
}

void fir::runtime::genIndexDescriptor(fir::FirOpBuilder &builder,
                                      mlir::Location loc, mlir::Value resultBox,
                                      mlir::Value stringBox,
                                      mlir::Value substringBox,
                                      mlir::Value backOpt, mlir::Value kind) {
  auto indexFunc = fir::runtime::getRuntimeFunc<mkRTKey(Index)>(loc, builder);
  genCharacterSearch(indexFunc, builder, loc, resultBox, stringBox,
                     substringBox, backOpt, kind);
}

void fir::runtime::genRepeat(fir::FirOpBuilder &builder, mlir::Location loc,
                             mlir::Value resultBox, mlir::Value stringBox,
````
- **L199 EN**: Initializes variable `substrBuffer` from the right-hand expression.
  **L199 CN**: 使用右侧表达式初始化变量 `substrBuffer`。
- **L200 EN**: Initializes variable `kind` from the right-hand expression.
  **L200 CN**: 使用右侧表达式初始化变量 `kind`。
- **L201 EN**: Returns from the current function with `genIndex(builder, loc, kind, strBuffer, fir::getLen(str), substrBuffer,`.
  **L201 CN**: 以 `genIndex(builder, loc, kind, strBuffer, fir::getLen(str), substrBuffer,` 从当前函数返回。
- **L202 EN**: Executes a call or declaration centered on `fir::getLen`.
  **L202 CN**: 执行以 `fir::getLen` 为核心的调用或声明。
- **L203 EN**: Closes the current lexical scope or compound statement.
  **L203 CN**: 结束当前词法作用域或复合语句块。
- **L204 EN**: Blank line separating nearby declarations or logic blocks.
  **L204 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::runtime::genIndexDescriptor(fir::FirOpBuilder &builder,`.
  **L205 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::runtime::genIndexDescriptor(fir::FirOpBuilder &builder,`。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, mlir::Value resultBox,`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, mlir::Value resultBox,`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value stringBox,`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value stringBox,`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value substringBox,`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value substringBox,`。
- **L209 EN**: Continues the surrounding expression or declaration: `mlir::Value backOpt, mlir::Value kind) {`.
  **L209 CN**: 继续构造周围的表达式或声明：`mlir::Value backOpt, mlir::Value kind) {`。
- **L210 EN**: Initializes variable `indexFunc` from the right-hand expression.
  **L210 CN**: 使用右侧表达式初始化变量 `indexFunc`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genCharacterSearch(indexFunc, builder, loc, resultBox, stringBox,`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`genCharacterSearch(indexFunc, builder, loc, resultBox, stringBox,`。
- **L212 EN**: Executes a standalone statement or declaration: `substringBox, backOpt, kind);`.
  **L212 CN**: 执行一条独立语句或声明：`substringBox, backOpt, kind);`。
- **L213 EN**: Closes the current lexical scope or compound statement.
  **L213 CN**: 结束当前词法作用域或复合语句块。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::runtime::genRepeat(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::runtime::genRepeat(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value resultBox, mlir::Value stringBox,`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value resultBox, mlir::Value stringBox,`。

### Lines 217-234

````cpp
                             mlir::Value ncopies) {
  auto repeatFunc = fir::runtime::getRuntimeFunc<mkRTKey(Repeat)>(loc, builder);
  auto fTy = repeatFunc.getFunctionType();
  auto sourceFile = fir::factory::locationToFilename(builder, loc);
  auto sourceLine =
      fir::factory::locationToLineNo(builder, loc, fTy.getInput(4));

  auto args = fir::runtime::createArguments(
      builder, loc, fTy, resultBox, stringBox, ncopies, sourceFile, sourceLine);
  fir::CallOp::create(builder, loc, repeatFunc, args);
}

void fir::runtime::genTrim(fir::FirOpBuilder &builder, mlir::Location loc,
                           mlir::Value resultBox, mlir::Value stringBox) {
  auto trimFunc = fir::runtime::getRuntimeFunc<mkRTKey(Trim)>(loc, builder);
  auto fTy = trimFunc.getFunctionType();
  auto sourceFile = fir::factory::locationToFilename(builder, loc);
  auto sourceLine =
````
- **L217 EN**: Continues the surrounding expression or declaration: `mlir::Value ncopies) {`.
  **L217 CN**: 继续构造周围的表达式或声明：`mlir::Value ncopies) {`。
- **L218 EN**: Initializes variable `repeatFunc` from the right-hand expression.
  **L218 CN**: 使用右侧表达式初始化变量 `repeatFunc`。
- **L219 EN**: Initializes variable `fTy` from the right-hand expression.
  **L219 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L220 EN**: Initializes variable `sourceFile` from the right-hand expression.
  **L220 CN**: 使用右侧表达式初始化变量 `sourceFile`。
- **L221 EN**: Continues the surrounding expression or declaration: `auto sourceLine =`.
  **L221 CN**: 继续构造周围的表达式或声明：`auto sourceLine =`。
- **L222 EN**: Executes a call or declaration centered on `fir::factory::locationToLineNo`.
  **L222 CN**: 执行以 `fir::factory::locationToLineNo` 为核心的调用或声明。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L224 EN**: Continues logic associated with callable symbol `createArguments`.
  **L224 CN**: 继续与可调用符号 `createArguments` 相关的逻辑。
- **L225 EN**: Executes a standalone statement or declaration: `builder, loc, fTy, resultBox, stringBox, ncopies, sourceFile, sourceLine);`.
  **L225 CN**: 执行一条独立语句或声明：`builder, loc, fTy, resultBox, stringBox, ncopies, sourceFile, sourceLine);`。
- **L226 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L226 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L227 EN**: Closes the current lexical scope or compound statement.
  **L227 CN**: 结束当前词法作用域或复合语句块。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::runtime::genTrim(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::runtime::genTrim(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L230 EN**: Continues the surrounding expression or declaration: `mlir::Value resultBox, mlir::Value stringBox) {`.
  **L230 CN**: 继续构造周围的表达式或声明：`mlir::Value resultBox, mlir::Value stringBox) {`。
- **L231 EN**: Initializes variable `trimFunc` from the right-hand expression.
  **L231 CN**: 使用右侧表达式初始化变量 `trimFunc`。
- **L232 EN**: Initializes variable `fTy` from the right-hand expression.
  **L232 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L233 EN**: Initializes variable `sourceFile` from the right-hand expression.
  **L233 CN**: 使用右侧表达式初始化变量 `sourceFile`。
- **L234 EN**: Continues the surrounding expression or declaration: `auto sourceLine =`.
  **L234 CN**: 继续构造周围的表达式或声明：`auto sourceLine =`。

### Lines 235-252

````cpp
      fir::factory::locationToLineNo(builder, loc, fTy.getInput(3));

  auto args = fir::runtime::createArguments(builder, loc, fTy, resultBox,
                                            stringBox, sourceFile, sourceLine);
  fir::CallOp::create(builder, loc, trimFunc, args);
}

void fir::runtime::genScanDescriptor(fir::FirOpBuilder &builder,
                                     mlir::Location loc, mlir::Value resultBox,
                                     mlir::Value stringBox, mlir::Value setBox,
                                     mlir::Value backBox, mlir::Value kind) {
  auto func = fir::runtime::getRuntimeFunc<mkRTKey(Scan)>(loc, builder);
  genCharacterSearch(func, builder, loc, resultBox, stringBox, setBox, backBox,
                     kind);
}

mlir::Value fir::runtime::genScan(fir::FirOpBuilder &builder,
                                  mlir::Location loc, int kind,
````
- **L235 EN**: Executes a call or declaration centered on `fir::factory::locationToLineNo`.
  **L235 CN**: 执行以 `fir::factory::locationToLineNo` 为核心的调用或声明。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto args = fir::runtime::createArguments(builder, loc, fTy, resultBox,`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto args = fir::runtime::createArguments(builder, loc, fTy, resultBox,`。
- **L238 EN**: Executes a standalone statement or declaration: `stringBox, sourceFile, sourceLine);`.
  **L238 CN**: 执行一条独立语句或声明：`stringBox, sourceFile, sourceLine);`。
- **L239 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L239 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L240 EN**: Closes the current lexical scope or compound statement.
  **L240 CN**: 结束当前词法作用域或复合语句块。
- **L241 EN**: Blank line separating nearby declarations or logic blocks.
  **L241 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::runtime::genScanDescriptor(fir::FirOpBuilder &builder,`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::runtime::genScanDescriptor(fir::FirOpBuilder &builder,`。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, mlir::Value resultBox,`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, mlir::Value resultBox,`。
- **L244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value stringBox, mlir::Value setBox,`.
  **L244 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value stringBox, mlir::Value setBox,`。
- **L245 EN**: Continues the surrounding expression or declaration: `mlir::Value backBox, mlir::Value kind) {`.
  **L245 CN**: 继续构造周围的表达式或声明：`mlir::Value backBox, mlir::Value kind) {`。
- **L246 EN**: Initializes variable `func` from the right-hand expression.
  **L246 CN**: 使用右侧表达式初始化变量 `func`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genCharacterSearch(func, builder, loc, resultBox, stringBox, setBox, backBox,`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`genCharacterSearch(func, builder, loc, resultBox, stringBox, setBox, backBox,`。
- **L248 EN**: Executes a standalone statement or declaration: `kind);`.
  **L248 CN**: 执行一条独立语句或声明：`kind);`。
- **L249 EN**: Closes the current lexical scope or compound statement.
  **L249 CN**: 结束当前词法作用域或复合语句块。
- **L250 EN**: Blank line separating nearby declarations or logic blocks.
  **L250 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value fir::runtime::genScan(fir::FirOpBuilder &builder,`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value fir::runtime::genScan(fir::FirOpBuilder &builder,`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, int kind,`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, int kind,`。

### Lines 253-270

````cpp
                                  mlir::Value stringBase, mlir::Value stringLen,
                                  mlir::Value setBase, mlir::Value setLen,
                                  mlir::Value back) {
  mlir::func::FuncOp func;
  switch (kind) {
  case 1:
    func = fir::runtime::getRuntimeFunc<mkRTKey(Scan1)>(loc, builder);
    break;
  case 2:
    func = fir::runtime::getRuntimeFunc<mkRTKey(Scan2)>(loc, builder);
    break;
  case 4:
    func = fir::runtime::getRuntimeFunc<mkRTKey(Scan4)>(loc, builder);
    break;
  default:
    fir::emitFatalError(
        loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");
  }
````
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value stringBase, mlir::Value stringLen,`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value stringBase, mlir::Value stringLen,`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value setBase, mlir::Value setLen,`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value setBase, mlir::Value setLen,`。
- **L255 EN**: Continues the surrounding expression or declaration: `mlir::Value back) {`.
  **L255 CN**: 继续构造周围的表达式或声明：`mlir::Value back) {`。
- **L256 EN**: Executes a standalone statement or declaration: `mlir::func::FuncOp func;`.
  **L256 CN**: 执行一条独立语句或声明：`mlir::func::FuncOp func;`。
- **L257 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L257 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L258 EN**: Introduces a switch dispatch label: `case 1:`.
  **L258 CN**: 引入一个 switch 分发标签：`case 1:`。
- **L259 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L259 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L260 EN**: Exits the nearest loop or switch statement.
  **L260 CN**: 退出最近的循环或 switch 语句。
- **L261 EN**: Introduces a switch dispatch label: `case 2:`.
  **L261 CN**: 引入一个 switch 分发标签：`case 2:`。
- **L262 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L262 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L263 EN**: Exits the nearest loop or switch statement.
  **L263 CN**: 退出最近的循环或 switch 语句。
- **L264 EN**: Introduces a switch dispatch label: `case 4:`.
  **L264 CN**: 引入一个 switch 分发标签：`case 4:`。
- **L265 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L265 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L266 EN**: Exits the nearest loop or switch statement.
  **L266 CN**: 退出最近的循环或 switch 语句。
- **L267 EN**: Introduces a switch dispatch label: `default:`.
  **L267 CN**: 引入一个 switch 分发标签：`default:`。
- **L268 EN**: Continues logic associated with callable symbol `emitFatalError`.
  **L268 CN**: 继续与可调用符号 `emitFatalError` 相关的逻辑。
- **L269 EN**: Executes a standalone statement or declaration: `loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");`.
  **L269 CN**: 执行一条独立语句或声明：`loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");`。
- **L270 EN**: Closes the current lexical scope or compound statement.
  **L270 CN**: 结束当前词法作用域或复合语句块。

### Lines 271-288

````cpp
  auto fTy = func.getFunctionType();
  auto args = fir::runtime::createArguments(builder, loc, fTy, stringBase,
                                            stringLen, setBase, setLen, back);
  return fir::CallOp::create(builder, loc, func, args).getResult(0);
}

void fir::runtime::genVerifyDescriptor(fir::FirOpBuilder &builder,
                                       mlir::Location loc,
                                       mlir::Value resultBox,
                                       mlir::Value stringBox,
                                       mlir::Value setBox, mlir::Value backBox,
                                       mlir::Value kind) {
  auto func = fir::runtime::getRuntimeFunc<mkRTKey(Verify)>(loc, builder);
  genCharacterSearch(func, builder, loc, resultBox, stringBox, setBox, backBox,
                     kind);
}

void fir::runtime::genTokenize(fir::FirOpBuilder &builder, mlir::Location loc,
````
- **L271 EN**: Initializes variable `fTy` from the right-hand expression.
  **L271 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto args = fir::runtime::createArguments(builder, loc, fTy, stringBase,`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto args = fir::runtime::createArguments(builder, loc, fTy, stringBase,`。
- **L273 EN**: Executes a standalone statement or declaration: `stringLen, setBase, setLen, back);`.
  **L273 CN**: 执行一条独立语句或声明：`stringLen, setBase, setLen, back);`。
- **L274 EN**: Returns from the current function with `fir::CallOp::create(builder, loc, func, args).getResult(0)`.
  **L274 CN**: 以 `fir::CallOp::create(builder, loc, func, args).getResult(0)` 从当前函数返回。
- **L275 EN**: Closes the current lexical scope or compound statement.
  **L275 CN**: 结束当前词法作用域或复合语句块。
- **L276 EN**: Blank line separating nearby declarations or logic blocks.
  **L276 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::runtime::genVerifyDescriptor(fir::FirOpBuilder &builder,`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::runtime::genVerifyDescriptor(fir::FirOpBuilder &builder,`。
- **L278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc,`.
  **L278 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc,`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value resultBox,`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value resultBox,`。
- **L280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value stringBox,`.
  **L280 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value stringBox,`。
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value setBox, mlir::Value backBox,`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value setBox, mlir::Value backBox,`。
- **L282 EN**: Continues the surrounding expression or declaration: `mlir::Value kind) {`.
  **L282 CN**: 继续构造周围的表达式或声明：`mlir::Value kind) {`。
- **L283 EN**: Initializes variable `func` from the right-hand expression.
  **L283 CN**: 使用右侧表达式初始化变量 `func`。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genCharacterSearch(func, builder, loc, resultBox, stringBox, setBox, backBox,`.
  **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`genCharacterSearch(func, builder, loc, resultBox, stringBox, setBox, backBox,`。
- **L285 EN**: Executes a standalone statement or declaration: `kind);`.
  **L285 CN**: 执行一条独立语句或声明：`kind);`。
- **L286 EN**: Closes the current lexical scope or compound statement.
  **L286 CN**: 结束当前词法作用域或复合语句块。
- **L287 EN**: Blank line separating nearby declarations or logic blocks.
  **L287 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::runtime::genTokenize(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L288 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::runtime::genTokenize(fir::FirOpBuilder &builder, mlir::Location loc,`。

### Lines 289-306

````cpp
                               mlir::Value tokensBox, mlir::Value separatorBox,
                               mlir::Value stringBox, mlir::Value setBox) {
  auto func = fir::runtime::getRuntimeFunc<mkRTKey(Tokenize)>(loc, builder);
  auto fTy = func.getFunctionType();
  auto sourceFile = fir::factory::locationToFilename(builder, loc);
  auto sourceLine =
      fir::factory::locationToLineNo(builder, loc, fTy.getInput(5));
  auto args =
      fir::runtime::createArguments(builder, loc, fTy, tokensBox, separatorBox,
                                    stringBox, setBox, sourceFile, sourceLine);
  fir::CallOp::create(builder, loc, func, args);
}

void fir::runtime::genTokenizePositions(
    fir::FirOpBuilder &builder, mlir::Location loc, mlir::Value firstBox,
    mlir::Value lastBox, mlir::Value stringBox, mlir::Value setBox) {
  auto func =
      fir::runtime::getRuntimeFunc<mkRTKey(TokenizePositions)>(loc, builder);
````
- **L289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value tokensBox, mlir::Value separatorBox,`.
  **L289 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value tokensBox, mlir::Value separatorBox,`。
- **L290 EN**: Continues the surrounding expression or declaration: `mlir::Value stringBox, mlir::Value setBox) {`.
  **L290 CN**: 继续构造周围的表达式或声明：`mlir::Value stringBox, mlir::Value setBox) {`。
- **L291 EN**: Initializes variable `func` from the right-hand expression.
  **L291 CN**: 使用右侧表达式初始化变量 `func`。
- **L292 EN**: Initializes variable `fTy` from the right-hand expression.
  **L292 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L293 EN**: Initializes variable `sourceFile` from the right-hand expression.
  **L293 CN**: 使用右侧表达式初始化变量 `sourceFile`。
- **L294 EN**: Continues the surrounding expression or declaration: `auto sourceLine =`.
  **L294 CN**: 继续构造周围的表达式或声明：`auto sourceLine =`。
- **L295 EN**: Executes a call or declaration centered on `fir::factory::locationToLineNo`.
  **L295 CN**: 执行以 `fir::factory::locationToLineNo` 为核心的调用或声明。
- **L296 EN**: Continues the surrounding expression or declaration: `auto args =`.
  **L296 CN**: 继续构造周围的表达式或声明：`auto args =`。
- **L297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::runtime::createArguments(builder, loc, fTy, tokensBox, separatorBox,`.
  **L297 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::runtime::createArguments(builder, loc, fTy, tokensBox, separatorBox,`。
- **L298 EN**: Executes a standalone statement or declaration: `stringBox, setBox, sourceFile, sourceLine);`.
  **L298 CN**: 执行一条独立语句或声明：`stringBox, setBox, sourceFile, sourceLine);`。
- **L299 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L299 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L300 EN**: Closes the current lexical scope or compound statement.
  **L300 CN**: 结束当前词法作用域或复合语句块。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L302 EN**: Continues logic associated with callable symbol `genTokenizePositions`.
  **L302 CN**: 继续与可调用符号 `genTokenizePositions` 相关的逻辑。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::FirOpBuilder &builder, mlir::Location loc, mlir::Value firstBox,`.
  **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::FirOpBuilder &builder, mlir::Location loc, mlir::Value firstBox,`。
- **L304 EN**: Continues the surrounding expression or declaration: `mlir::Value lastBox, mlir::Value stringBox, mlir::Value setBox) {`.
  **L304 CN**: 继续构造周围的表达式或声明：`mlir::Value lastBox, mlir::Value stringBox, mlir::Value setBox) {`。
- **L305 EN**: Continues the surrounding expression or declaration: `auto func =`.
  **L305 CN**: 继续构造周围的表达式或声明：`auto func =`。
- **L306 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L306 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。

### Lines 307-324

````cpp
  auto fTy = func.getFunctionType();
  auto sourceFile = fir::factory::locationToFilename(builder, loc);
  auto sourceLine =
      fir::factory::locationToLineNo(builder, loc, fTy.getInput(5));
  auto args =
      fir::runtime::createArguments(builder, loc, fTy, firstBox, lastBox,
                                    stringBox, setBox, sourceFile, sourceLine);
  fir::CallOp::create(builder, loc, func, args);
}

mlir::Value fir::runtime::genVerify(fir::FirOpBuilder &builder,
                                    mlir::Location loc, int kind,
                                    mlir::Value stringBase,
                                    mlir::Value stringLen, mlir::Value setBase,
                                    mlir::Value setLen, mlir::Value back) {
  mlir::func::FuncOp func;
  switch (kind) {
  case 1:
````
- **L307 EN**: Initializes variable `fTy` from the right-hand expression.
  **L307 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L308 EN**: Initializes variable `sourceFile` from the right-hand expression.
  **L308 CN**: 使用右侧表达式初始化变量 `sourceFile`。
- **L309 EN**: Continues the surrounding expression or declaration: `auto sourceLine =`.
  **L309 CN**: 继续构造周围的表达式或声明：`auto sourceLine =`。
- **L310 EN**: Executes a call or declaration centered on `fir::factory::locationToLineNo`.
  **L310 CN**: 执行以 `fir::factory::locationToLineNo` 为核心的调用或声明。
- **L311 EN**: Continues the surrounding expression or declaration: `auto args =`.
  **L311 CN**: 继续构造周围的表达式或声明：`auto args =`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::runtime::createArguments(builder, loc, fTy, firstBox, lastBox,`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::runtime::createArguments(builder, loc, fTy, firstBox, lastBox,`。
- **L313 EN**: Executes a standalone statement or declaration: `stringBox, setBox, sourceFile, sourceLine);`.
  **L313 CN**: 执行一条独立语句或声明：`stringBox, setBox, sourceFile, sourceLine);`。
- **L314 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L314 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L315 EN**: Closes the current lexical scope or compound statement.
  **L315 CN**: 结束当前词法作用域或复合语句块。
- **L316 EN**: Blank line separating nearby declarations or logic blocks.
  **L316 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value fir::runtime::genVerify(fir::FirOpBuilder &builder,`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value fir::runtime::genVerify(fir::FirOpBuilder &builder,`。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, int kind,`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, int kind,`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value stringBase,`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value stringBase,`。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value stringLen, mlir::Value setBase,`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value stringLen, mlir::Value setBase,`。
- **L321 EN**: Continues the surrounding expression or declaration: `mlir::Value setLen, mlir::Value back) {`.
  **L321 CN**: 继续构造周围的表达式或声明：`mlir::Value setLen, mlir::Value back) {`。
- **L322 EN**: Executes a standalone statement or declaration: `mlir::func::FuncOp func;`.
  **L322 CN**: 执行一条独立语句或声明：`mlir::func::FuncOp func;`。
- **L323 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L323 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L324 EN**: Introduces a switch dispatch label: `case 1:`.
  **L324 CN**: 引入一个 switch 分发标签：`case 1:`。

### Lines 325-342

````cpp
    func = fir::runtime::getRuntimeFunc<mkRTKey(Verify1)>(loc, builder);
    break;
  case 2:
    func = fir::runtime::getRuntimeFunc<mkRTKey(Verify2)>(loc, builder);
    break;
  case 4:
    func = fir::runtime::getRuntimeFunc<mkRTKey(Verify4)>(loc, builder);
    break;
  default:
    fir::emitFatalError(
        loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");
  }
  auto fTy = func.getFunctionType();
  auto args = fir::runtime::createArguments(builder, loc, fTy, stringBase,
                                            stringLen, setBase, setLen, back);
  return fir::CallOp::create(builder, loc, func, args).getResult(0);
}

````
- **L325 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L325 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L326 EN**: Exits the nearest loop or switch statement.
  **L326 CN**: 退出最近的循环或 switch 语句。
- **L327 EN**: Introduces a switch dispatch label: `case 2:`.
  **L327 CN**: 引入一个 switch 分发标签：`case 2:`。
- **L328 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L328 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L329 EN**: Exits the nearest loop or switch statement.
  **L329 CN**: 退出最近的循环或 switch 语句。
- **L330 EN**: Introduces a switch dispatch label: `case 4:`.
  **L330 CN**: 引入一个 switch 分发标签：`case 4:`。
- **L331 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L331 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L332 EN**: Exits the nearest loop or switch statement.
  **L332 CN**: 退出最近的循环或 switch 语句。
- **L333 EN**: Introduces a switch dispatch label: `default:`.
  **L333 CN**: 引入一个 switch 分发标签：`default:`。
- **L334 EN**: Continues logic associated with callable symbol `emitFatalError`.
  **L334 CN**: 继续与可调用符号 `emitFatalError` 相关的逻辑。
- **L335 EN**: Executes a standalone statement or declaration: `loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");`.
  **L335 CN**: 执行一条独立语句或声明：`loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");`。
- **L336 EN**: Closes the current lexical scope or compound statement.
  **L336 CN**: 结束当前词法作用域或复合语句块。
- **L337 EN**: Initializes variable `fTy` from the right-hand expression.
  **L337 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto args = fir::runtime::createArguments(builder, loc, fTy, stringBase,`.
  **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto args = fir::runtime::createArguments(builder, loc, fTy, stringBase,`。
- **L339 EN**: Executes a standalone statement or declaration: `stringLen, setBase, setLen, back);`.
  **L339 CN**: 执行一条独立语句或声明：`stringLen, setBase, setLen, back);`。
- **L340 EN**: Returns from the current function with `fir::CallOp::create(builder, loc, func, args).getResult(0)`.
  **L340 CN**: 以 `fir::CallOp::create(builder, loc, func, args).getResult(0)` 从当前函数返回。
- **L341 EN**: Closes the current lexical scope or compound statement.
  **L341 CN**: 结束当前词法作用域或复合语句块。
- **L342 EN**: Blank line separating nearby declarations or logic blocks.
  **L342 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 343-360

````cpp
mlir::Value fir::runtime::genSplit(fir::FirOpBuilder &builder,
                                   mlir::Location loc, int kind,
                                   mlir::Value stringBase,
                                   mlir::Value stringLen, mlir::Value setBase,
                                   mlir::Value setLen, mlir::Value pos,
                                   mlir::Value back) {
  mlir::func::FuncOp func;
  switch (kind) {
  case 1:
    func = fir::runtime::getRuntimeFunc<mkRTKey(Split1)>(loc, builder);
    break;
  case 2:
    func = fir::runtime::getRuntimeFunc<mkRTKey(Split2)>(loc, builder);
    break;
  case 4:
    func = fir::runtime::getRuntimeFunc<mkRTKey(Split4)>(loc, builder);
    break;
  default:
````
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value fir::runtime::genSplit(fir::FirOpBuilder &builder,`.
  **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value fir::runtime::genSplit(fir::FirOpBuilder &builder,`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, int kind,`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, int kind,`。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value stringBase,`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value stringBase,`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value stringLen, mlir::Value setBase,`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value stringLen, mlir::Value setBase,`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value setLen, mlir::Value pos,`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value setLen, mlir::Value pos,`。
- **L348 EN**: Continues the surrounding expression or declaration: `mlir::Value back) {`.
  **L348 CN**: 继续构造周围的表达式或声明：`mlir::Value back) {`。
- **L349 EN**: Executes a standalone statement or declaration: `mlir::func::FuncOp func;`.
  **L349 CN**: 执行一条独立语句或声明：`mlir::func::FuncOp func;`。
- **L350 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L350 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L351 EN**: Introduces a switch dispatch label: `case 1:`.
  **L351 CN**: 引入一个 switch 分发标签：`case 1:`。
- **L352 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L352 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L353 EN**: Exits the nearest loop or switch statement.
  **L353 CN**: 退出最近的循环或 switch 语句。
- **L354 EN**: Introduces a switch dispatch label: `case 2:`.
  **L354 CN**: 引入一个 switch 分发标签：`case 2:`。
- **L355 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L355 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L356 EN**: Exits the nearest loop or switch statement.
  **L356 CN**: 退出最近的循环或 switch 语句。
- **L357 EN**: Introduces a switch dispatch label: `case 4:`.
  **L357 CN**: 引入一个 switch 分发标签：`case 4:`。
- **L358 EN**: Executes a call or declaration centered on `fir::runtime::getRuntimeFunc<mkRTKey`.
  **L358 CN**: 执行以 `fir::runtime::getRuntimeFunc<mkRTKey` 为核心的调用或声明。
- **L359 EN**: Exits the nearest loop or switch statement.
  **L359 CN**: 退出最近的循环或 switch 语句。
- **L360 EN**: Introduces a switch dispatch label: `default:`.
  **L360 CN**: 引入一个 switch 分发标签：`default:`。

### Lines 361-368

````cpp
    fir::emitFatalError(
        loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");
  }
  auto fTy = func.getFunctionType();
  auto args = fir::runtime::createArguments(
      builder, loc, fTy, stringBase, stringLen, setBase, setLen, pos, back);
  return fir::CallOp::create(builder, loc, func, args).getResult(0);
}
````
- **L361 EN**: Continues logic associated with callable symbol `emitFatalError`.
  **L361 CN**: 继续与可调用符号 `emitFatalError` 相关的逻辑。
- **L362 EN**: Executes a standalone statement or declaration: `loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");`.
  **L362 CN**: 执行一条独立语句或声明：`loc, "unsupported CHARACTER kind value. Runtime expects 1, 2, or 4.");`。
- **L363 EN**: Closes the current lexical scope or compound statement.
  **L363 CN**: 结束当前词法作用域或复合语句块。
- **L364 EN**: Initializes variable `fTy` from the right-hand expression.
  **L364 CN**: 使用右侧表达式初始化变量 `fTy`。
- **L365 EN**: Continues logic associated with callable symbol `createArguments`.
  **L365 CN**: 继续与可调用符号 `createArguments` 相关的逻辑。
- **L366 EN**: Executes a standalone statement or declaration: `builder, loc, fTy, stringBase, stringLen, setBase, setLen, pos, back);`.
  **L366 CN**: 执行一条独立语句或声明：`builder, loc, fTy, stringBase, stringLen, setBase, setLen, pos, back);`。
- **L367 EN**: Returns from the current function with `fir::CallOp::create(builder, loc, func, args).getResult(0)`.
  **L367 CN**: 以 `fir::CallOp::create(builder, loc, func, args).getResult(0)` 从当前函数返回。
- **L368 EN**: Closes the current lexical scope or compound statement.
  **L368 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**
- **IR builder orchestration / IR Builder 编排**
- **Runtime call integration / 运行时调用集成**
- **Fortran descriptor management / Fortran 描述符管理**

## Dependencies / 依赖关系

- `flang/Optimizer/Builder/Runtime/Character.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/BoxValue.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/Character.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/FIRBuilder.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/Runtime/RTBuilder.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/Todo.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Runtime/character.h`: Provides Fortran runtime entry points and descriptor helpers. / 提供Fortran 运行时入口与描述符辅助能力。
- `mlir/Dialect/Func/IR/FuncOps.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
