# ConvertCall.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Lower/ConvertCall.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements lowering from Fortran semantics into FIR/MLIR for Convert Call.
- **Purpose (CN)**: 实现 Convert Call 相关的从 Fortran 语义到 FIR/MLIR 的 lowering 过程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- ConvertCall.cpp ---------------------------------------------------===//
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

#include "flang/Lower/ConvertCall.h"
#include "flang/Lower/Allocatable.h"
#include "flang/Lower/ConvertExprToHLFIR.h"
#include "flang/Lower/ConvertProcedureDesignator.h"
#include "flang/Lower/ConvertVariable.h"
#include "flang/Lower/CustomIntrinsicCall.h"
#include "flang/Lower/HlfirIntrinsics.h"
#include "flang/Lower/PFTBuilder.h"
#include "flang/Lower/StatementContext.h"
#include "flang/Lower/SymbolMap.h"
#include "flang/Optimizer/Builder/BoxValue.h"
#include "flang/Optimizer/Builder/CUFCommon.h"
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
- **L13 EN**: Includes "flang/Lower/ConvertCall.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L13 CN**: 引入 "flang/Lower/ConvertCall.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L14 EN**: Includes "flang/Lower/Allocatable.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L14 CN**: 引入 "flang/Lower/Allocatable.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L15 EN**: Includes "flang/Lower/ConvertExprToHLFIR.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L15 CN**: 引入 "flang/Lower/ConvertExprToHLFIR.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L16 EN**: Includes "flang/Lower/ConvertProcedureDesignator.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L16 CN**: 引入 "flang/Lower/ConvertProcedureDesignator.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L17 EN**: Includes "flang/Lower/ConvertVariable.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L17 CN**: 引入 "flang/Lower/ConvertVariable.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L18 EN**: Includes "flang/Lower/CustomIntrinsicCall.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L18 CN**: 引入 "flang/Lower/CustomIntrinsicCall.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L19 EN**: Includes "flang/Lower/HlfirIntrinsics.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L19 CN**: 引入 "flang/Lower/HlfirIntrinsics.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L20 EN**: Includes "flang/Lower/PFTBuilder.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L20 CN**: 引入 "flang/Lower/PFTBuilder.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L21 EN**: Includes "flang/Lower/StatementContext.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L21 CN**: 引入 "flang/Lower/StatementContext.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L22 EN**: Includes "flang/Lower/SymbolMap.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L22 CN**: 引入 "flang/Lower/SymbolMap.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L23 EN**: Includes "flang/Optimizer/Builder/BoxValue.h" to access FIR builder helpers and runtime-construction utilities.
  **L23 CN**: 引入 "flang/Optimizer/Builder/BoxValue.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L24 EN**: Includes "flang/Optimizer/Builder/CUFCommon.h" to access FIR builder helpers and runtime-construction utilities.
  **L24 CN**: 引入 "flang/Optimizer/Builder/CUFCommon.h" 以使用FIR Builder 辅助工具与运行时构造能力。

### Lines 25-48

````cpp
#include "flang/Optimizer/Builder/Character.h"
#include "flang/Optimizer/Builder/FIRBuilder.h"
#include "flang/Optimizer/Builder/HLFIRTools.h"
#include "flang/Optimizer/Builder/IntrinsicCall.h"
#include "flang/Optimizer/Builder/LowLevelIntrinsics.h"
#include "flang/Optimizer/Builder/MutableBox.h"
#include "flang/Optimizer/Builder/Runtime/CUDA/Descriptor.h"
#include "flang/Optimizer/Builder/Runtime/Derived.h"
#include "flang/Optimizer/Builder/Todo.h"
#include "flang/Optimizer/Dialect/CUF/CUFOps.h"
#include "flang/Optimizer/Dialect/FIROpsSupport.h"
#include "flang/Optimizer/HLFIR/HLFIROps.h"
#include "mlir/IR/IRMapping.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include <optional>

#define DEBUG_TYPE "flang-lower-expr"

static llvm::cl::opt<bool> useHlfirIntrinsicOps(
    "use-hlfir-intrinsic-ops", llvm::cl::init(true),
    llvm::cl::desc("Lower via HLFIR transformational intrinsic operations such "
                   "as hlfir.sum"));
````
- **L25 EN**: Includes "flang/Optimizer/Builder/Character.h" to access FIR builder helpers and runtime-construction utilities.
  **L25 CN**: 引入 "flang/Optimizer/Builder/Character.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L26 EN**: Includes "flang/Optimizer/Builder/FIRBuilder.h" to access FIR builder helpers and runtime-construction utilities.
  **L26 CN**: 引入 "flang/Optimizer/Builder/FIRBuilder.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L27 EN**: Includes "flang/Optimizer/Builder/HLFIRTools.h" to access FIR builder helpers and runtime-construction utilities.
  **L27 CN**: 引入 "flang/Optimizer/Builder/HLFIRTools.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L28 EN**: Includes "flang/Optimizer/Builder/IntrinsicCall.h" to access FIR builder helpers and runtime-construction utilities.
  **L28 CN**: 引入 "flang/Optimizer/Builder/IntrinsicCall.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L29 EN**: Includes "flang/Optimizer/Builder/LowLevelIntrinsics.h" to access FIR builder helpers and runtime-construction utilities.
  **L29 CN**: 引入 "flang/Optimizer/Builder/LowLevelIntrinsics.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L30 EN**: Includes "flang/Optimizer/Builder/MutableBox.h" to access FIR builder helpers and runtime-construction utilities.
  **L30 CN**: 引入 "flang/Optimizer/Builder/MutableBox.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L31 EN**: Includes "flang/Optimizer/Builder/Runtime/CUDA/Descriptor.h" to access FIR builder helpers and runtime-construction utilities.
  **L31 CN**: 引入 "flang/Optimizer/Builder/Runtime/CUDA/Descriptor.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L32 EN**: Includes "flang/Optimizer/Builder/Runtime/Derived.h" to access FIR builder helpers and runtime-construction utilities.
  **L32 CN**: 引入 "flang/Optimizer/Builder/Runtime/Derived.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L33 EN**: Includes "flang/Optimizer/Builder/Todo.h" to access FIR builder helpers and runtime-construction utilities.
  **L33 CN**: 引入 "flang/Optimizer/Builder/Todo.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L34 EN**: Includes "flang/Optimizer/Dialect/CUF/CUFOps.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L34 CN**: 引入 "flang/Optimizer/Dialect/CUF/CUFOps.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L35 EN**: Includes "flang/Optimizer/Dialect/FIROpsSupport.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L35 CN**: 引入 "flang/Optimizer/Dialect/FIROpsSupport.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L36 EN**: Includes "flang/Optimizer/HLFIR/HLFIROps.h" to access HLFIR abstractions and transformation support.
  **L36 CN**: 引入 "flang/Optimizer/HLFIR/HLFIROps.h" 以使用HLFIR 抽象与变换支持。
- **L37 EN**: Includes "mlir/IR/IRMapping.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L37 CN**: 引入 "mlir/IR/IRMapping.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L38 EN**: Includes "llvm/ADT/TypeSwitch.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L38 CN**: 引入 "llvm/ADT/TypeSwitch.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L39 EN**: Includes "llvm/Support/CommandLine.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L39 CN**: 引入 "llvm/Support/CommandLine.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L40 EN**: Includes "llvm/Support/Debug.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L40 CN**: 引入 "llvm/Support/Debug.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L41 EN**: Includes <optional> to access supporting declarations used by this translation unit.
  **L41 CN**: 引入 <optional> 以使用当前编译单元使用的辅助声明。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Defines macro `DEBUG_TYPE` for conditional compilation or local shorthand.
  **L43 CN**: 定义宏 `DEBUG_TYPE`，用于条件编译或本地简写。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L45 EN**: Declares a command-line option or tuning knob: `static llvm::cl::opt<bool> useHlfirIntrinsicOps(`.
  **L45 CN**: 声明一个命令行选项或调优开关：`static llvm::cl::opt<bool> useHlfirIntrinsicOps(`。
- **L46 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"use-hlfir-intrinsic-ops", llvm::cl::init(true),`.
  **L46 CN**: 继续一个多行参数列表、初始化器或聚合项：`"use-hlfir-intrinsic-ops", llvm::cl::init(true),`。
- **L47 EN**: Continues logic associated with callable symbol `desc`.
  **L47 CN**: 继续与可调用符号 `desc` 相关的逻辑。
- **L48 EN**: Executes a standalone statement or declaration: `"as hlfir.sum"));`.
  **L48 CN**: 执行一条独立语句或声明：`"as hlfir.sum"));`。

### Lines 49-72

````cpp

static constexpr char tempResultName[] = ".tmp.func_result";

/// Helper to package a Value and its properties into an ExtendedValue.
static fir::ExtendedValue toExtendedValue(mlir::Location loc, mlir::Value base,
                                          llvm::ArrayRef<mlir::Value> extents,
                                          llvm::ArrayRef<mlir::Value> lengths) {
  mlir::Type type = base.getType();
  if (mlir::isa<fir::BaseBoxType>(type))
    return fir::BoxValue(base, /*lbounds=*/{}, lengths, extents);
  type = fir::unwrapRefType(type);
  if (mlir::isa<fir::BaseBoxType>(type))
    return fir::MutableBoxValue(base, lengths, /*mutableProperties*/ {});
  if (auto seqTy = mlir::dyn_cast<fir::SequenceType>(type)) {
    if (seqTy.getDimension() != extents.size())
      fir::emitFatalError(loc, "incorrect number of extents for array");
    if (mlir::isa<fir::CharacterType>(seqTy.getEleTy())) {
      if (lengths.empty())
        fir::emitFatalError(loc, "missing length for character");
      assert(lengths.size() == 1);
      return fir::CharArrayBoxValue(base, lengths[0], extents);
    }
    return fir::ArrayBoxValue(base, extents);
  }
````
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L50 EN**: Executes a standalone statement or declaration: `static constexpr char tempResultName[] = ".tmp.func_result";`.
  **L50 CN**: 执行一条独立语句或声明：`static constexpr char tempResultName[] = ".tmp.func_result";`。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Comment explains nearby logic, intent, or metadata: `Helper to package a Value and its properties into an ExtendedValue.`.
  **L52 CN**: 注释说明附近代码的逻辑、意图或元数据：`Helper to package a Value and its properties into an ExtendedValue.`。
- **L53 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static fir::ExtendedValue toExtendedValue(mlir::Location loc, mlir::Value base,`.
  **L53 CN**: 继续一个多行参数列表、初始化器或聚合项：`static fir::ExtendedValue toExtendedValue(mlir::Location loc, mlir::Value base,`。
- **L54 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::ArrayRef<mlir::Value> extents,`.
  **L54 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::ArrayRef<mlir::Value> extents,`。
- **L55 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<mlir::Value> lengths) {`.
  **L55 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<mlir::Value> lengths) {`。
- **L56 EN**: Initializes variable `type` from the right-hand expression.
  **L56 CN**: 使用右侧表达式初始化变量 `type`。
- **L57 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L57 CN**: 开始 `if` 控制流语句并计算其条件。
- **L58 EN**: Returns from the current function with `fir::BoxValue(base, /*lbounds=*/{}, lengths, extents)`.
  **L58 CN**: 以 `fir::BoxValue(base, /*lbounds=*/{}, lengths, extents)` 从当前函数返回。
- **L59 EN**: Executes a call or declaration centered on `fir::unwrapRefType`.
  **L59 CN**: 执行以 `fir::unwrapRefType` 为核心的调用或声明。
- **L60 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L60 CN**: 开始 `if` 控制流语句并计算其条件。
- **L61 EN**: Returns from the current function with `fir::MutableBoxValue(base, lengths, /*mutableProperties*/ {})`.
  **L61 CN**: 以 `fir::MutableBoxValue(base, lengths, /*mutableProperties*/ {})` 从当前函数返回。
- **L62 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L62 CN**: 开始 `if` 控制流语句并计算其条件。
- **L63 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L63 CN**: 开始 `if` 控制流语句并计算其条件。
- **L64 EN**: Executes a call or declaration centered on `fir::emitFatalError`.
  **L64 CN**: 执行以 `fir::emitFatalError` 为核心的调用或声明。
- **L65 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L65 CN**: 开始 `if` 控制流语句并计算其条件。
- **L66 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L66 CN**: 开始 `if` 控制流语句并计算其条件。
- **L67 EN**: Executes a call or declaration centered on `fir::emitFatalError`.
  **L67 CN**: 执行以 `fir::emitFatalError` 为核心的调用或声明。
- **L68 EN**: Checks an internal invariant in debug builds.
  **L68 CN**: 在调试构建中检查内部不变式。
- **L69 EN**: Returns from the current function with `fir::CharArrayBoxValue(base, lengths[0], extents)`.
  **L69 CN**: 以 `fir::CharArrayBoxValue(base, lengths[0], extents)` 从当前函数返回。
- **L70 EN**: Closes the current lexical scope or compound statement.
  **L70 CN**: 结束当前词法作用域或复合语句块。
- **L71 EN**: Returns from the current function with `fir::ArrayBoxValue(base, extents)`.
  **L71 CN**: 以 `fir::ArrayBoxValue(base, extents)` 从当前函数返回。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。

### Lines 73-96

````cpp
  if (mlir::isa<fir::CharacterType>(type)) {
    if (lengths.empty())
      fir::emitFatalError(loc, "missing length for character");
    assert(lengths.size() == 1);
    return fir::CharBoxValue(base, lengths[0]);
  }
  return base;
}

/// Lower a type(C_PTR/C_FUNPTR) argument with VALUE attribute into a
/// reference. A C pointer can correspond to a Fortran dummy argument of type
/// C_PTR with the VALUE attribute. (see 18.3.6 note 3).
static mlir::Value genRecordCPtrValueArg(fir::FirOpBuilder &builder,
                                         mlir::Location loc, mlir::Value rec,
                                         mlir::Type ty) {
  mlir::Value cAddr = fir::factory::genCPtrOrCFunptrAddr(builder, loc, rec, ty);
  mlir::Value cVal = fir::LoadOp::create(builder, loc, cAddr);
  return builder.createConvert(loc, cAddr.getType(), cVal);
}

// Find the argument that corresponds to the host associations.
// Verify some assumptions about how the signature was built here.
[[maybe_unused]] static unsigned findHostAssocTuplePos(mlir::func::FuncOp fn) {
  // Scan the argument list from last to first as the host associations are
````
- **L73 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L73 CN**: 开始 `if` 控制流语句并计算其条件。
- **L74 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L74 CN**: 开始 `if` 控制流语句并计算其条件。
- **L75 EN**: Executes a call or declaration centered on `fir::emitFatalError`.
  **L75 CN**: 执行以 `fir::emitFatalError` 为核心的调用或声明。
- **L76 EN**: Checks an internal invariant in debug builds.
  **L76 CN**: 在调试构建中检查内部不变式。
- **L77 EN**: Returns from the current function with `fir::CharBoxValue(base, lengths[0])`.
  **L77 CN**: 以 `fir::CharBoxValue(base, lengths[0])` 从当前函数返回。
- **L78 EN**: Closes the current lexical scope or compound statement.
  **L78 CN**: 结束当前词法作用域或复合语句块。
- **L79 EN**: Returns from the current function with `base`.
  **L79 CN**: 以 `base` 从当前函数返回。
- **L80 EN**: Closes the current lexical scope or compound statement.
  **L80 CN**: 结束当前词法作用域或复合语句块。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L82 EN**: Comment explains nearby logic, intent, or metadata: `Lower a type(C_PTR/C_FUNPTR) argument with VALUE attribute into a`.
  **L82 CN**: 注释说明附近代码的逻辑、意图或元数据：`Lower a type(C_PTR/C_FUNPTR) argument with VALUE attribute into a`。
- **L83 EN**: Comment explains nearby logic, intent, or metadata: `reference. A C pointer can correspond to a Fortran dummy argument of type`.
  **L83 CN**: 注释说明附近代码的逻辑、意图或元数据：`reference. A C pointer can correspond to a Fortran dummy argument of type`。
- **L84 EN**: Comment explains nearby logic, intent, or metadata: `C_PTR with the VALUE attribute. (see 18.3.6 note 3).`.
  **L84 CN**: 注释说明附近代码的逻辑、意图或元数据：`C_PTR with the VALUE attribute. (see 18.3.6 note 3).`。
- **L85 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static mlir::Value genRecordCPtrValueArg(fir::FirOpBuilder &builder,`.
  **L85 CN**: 继续一个多行参数列表、初始化器或聚合项：`static mlir::Value genRecordCPtrValueArg(fir::FirOpBuilder &builder,`。
- **L86 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, mlir::Value rec,`.
  **L86 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, mlir::Value rec,`。
- **L87 EN**: Continues the surrounding expression or declaration: `mlir::Type ty) {`.
  **L87 CN**: 继续构造周围的表达式或声明：`mlir::Type ty) {`。
- **L88 EN**: Initializes variable `cAddr` from the right-hand expression.
  **L88 CN**: 使用右侧表达式初始化变量 `cAddr`。
- **L89 EN**: Initializes variable `cVal` from the right-hand expression.
  **L89 CN**: 使用右侧表达式初始化变量 `cVal`。
- **L90 EN**: Returns from the current function with `builder.createConvert(loc, cAddr.getType(), cVal)`.
  **L90 CN**: 以 `builder.createConvert(loc, cAddr.getType(), cVal)` 从当前函数返回。
- **L91 EN**: Closes the current lexical scope or compound statement.
  **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L93 EN**: Comment explains nearby logic, intent, or metadata: `Find the argument that corresponds to the host associations.`.
  **L93 CN**: 注释说明附近代码的逻辑、意图或元数据：`Find the argument that corresponds to the host associations.`。
- **L94 EN**: Comment explains nearby logic, intent, or metadata: `Verify some assumptions about how the signature was built here.`.
  **L94 CN**: 注释说明附近代码的逻辑、意图或元数据：`Verify some assumptions about how the signature was built here.`。
- **L95 EN**: Starts a function, method, lambda, or structured scope: `[[maybe_unused]] static unsigned findHostAssocTuplePos(mlir::func::FuncOp fn) {`.
  **L95 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[[maybe_unused]] static unsigned findHostAssocTuplePos(mlir::func::FuncOp fn) {`。
- **L96 EN**: Comment explains nearby logic, intent, or metadata: `Scan the argument list from last to first as the host associations are`.
  **L96 CN**: 注释说明附近代码的逻辑、意图或元数据：`Scan the argument list from last to first as the host associations are`。

### Lines 97-120

````cpp
  // appended for now.
  for (unsigned i = fn.getNumArguments(); i > 0; --i)
    if (fn.getArgAttr(i - 1, fir::getHostAssocAttrName())) {
      // Host assoc tuple must be last argument (for now).
      assert(i == fn.getNumArguments() && "tuple must be last");
      return i - 1;
    }
  llvm_unreachable("anyFuncArgsHaveAttr failed");
}

mlir::Value
Fortran::lower::argumentHostAssocs(Fortran::lower::AbstractConverter &converter,
                                   mlir::Value arg) {
  if (auto addr = mlir::dyn_cast_or_null<fir::AddrOfOp>(arg.getDefiningOp())) {
    auto &builder = converter.getFirOpBuilder();
    if (auto funcOp = builder.getNamedFunction(addr.getSymbol()))
      if (fir::anyFuncArgsHaveAttr(funcOp, fir::getHostAssocAttrName()))
        return converter.hostAssocTupleValue();
  }
  return {};
}

static bool mustCastFuncOpToCopeWithImplicitInterfaceMismatch(
    mlir::Location loc, Fortran::lower::AbstractConverter &converter,
````
- **L97 EN**: Comment explains nearby logic, intent, or metadata: `appended for now.`.
  **L97 CN**: 注释说明附近代码的逻辑、意图或元数据：`appended for now.`。
- **L98 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L98 CN**: 开始 `for` 控制流语句并计算其条件。
- **L99 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L99 CN**: 开始 `if` 控制流语句并计算其条件。
- **L100 EN**: Comment explains nearby logic, intent, or metadata: `Host assoc tuple must be last argument (for now).`.
  **L100 CN**: 注释说明附近代码的逻辑、意图或元数据：`Host assoc tuple must be last argument (for now).`。
- **L101 EN**: Checks an internal invariant in debug builds.
  **L101 CN**: 在调试构建中检查内部不变式。
- **L102 EN**: Returns from the current function with `i - 1`.
  **L102 CN**: 以 `i - 1` 从当前函数返回。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Marks this control path as unreachable to LLVM.
  **L104 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L105 EN**: Closes the current lexical scope or compound statement.
  **L105 CN**: 结束当前词法作用域或复合语句块。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L107 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L107 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::argumentHostAssocs(Fortran::lower::AbstractConverter &converter,`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::argumentHostAssocs(Fortran::lower::AbstractConverter &converter,`。
- **L109 EN**: Continues the surrounding expression or declaration: `mlir::Value arg) {`.
  **L109 CN**: 继续构造周围的表达式或声明：`mlir::Value arg) {`。
- **L110 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L110 CN**: 开始 `if` 控制流语句并计算其条件。
- **L111 EN**: Executes a call or declaration centered on `converter.getFirOpBuilder`.
  **L111 CN**: 执行以 `converter.getFirOpBuilder` 为核心的调用或声明。
- **L112 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L112 CN**: 开始 `if` 控制流语句并计算其条件。
- **L113 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L113 CN**: 开始 `if` 控制流语句并计算其条件。
- **L114 EN**: Returns from the current function with `converter.hostAssocTupleValue()`.
  **L114 CN**: 以 `converter.hostAssocTupleValue()` 从当前函数返回。
- **L115 EN**: Closes the current lexical scope or compound statement.
  **L115 CN**: 结束当前词法作用域或复合语句块。
- **L116 EN**: Returns from the current function with `{}`.
  **L116 CN**: 以 `{}` 从当前函数返回。
- **L117 EN**: Closes the current lexical scope or compound statement.
  **L117 CN**: 结束当前词法作用域或复合语句块。
- **L118 EN**: Blank line separating nearby declarations or logic blocks.
  **L118 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L119 EN**: Continues logic associated with callable symbol `mustCastFuncOpToCopeWithImplicitInterfaceMismatch`.
  **L119 CN**: 继续与可调用符号 `mustCastFuncOpToCopeWithImplicitInterfaceMismatch` 相关的逻辑。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, Fortran::lower::AbstractConverter &converter,`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, Fortran::lower::AbstractConverter &converter,`。

### Lines 121-144

````cpp
    mlir::FunctionType callSiteType, mlir::FunctionType funcOpType) {
  // Deal with argument number mismatch by making a function pointer so
  // that function type cast can be inserted. Do not emit a warning here
  // because this can happen in legal program if the function is not
  // defined here and it was first passed as an argument without any more
  // information.
  if (callSiteType.getNumResults() != funcOpType.getNumResults() ||
      callSiteType.getNumInputs() != funcOpType.getNumInputs())
    return true;

  // Implicit interface result type mismatch are not standard Fortran, but
  // some compilers are not complaining about it.  The front end is not
  // protecting lowering from this currently. Support this with a
  // discouraging warning.
  // Cast the actual function to the current caller implicit type because
  // that is the behavior we would get if we could not see the definition.
  if (callSiteType.getResults() != funcOpType.getResults()) {
    LLVM_DEBUG(mlir::emitWarning(
        loc, "a return type mismatch is not standard compliant and may "
             "lead to undefined behavior."));
    return true;
  }

  // In HLFIR, there is little attempt to cope with implicit interface
````
- **L121 EN**: Continues the surrounding expression or declaration: `mlir::FunctionType callSiteType, mlir::FunctionType funcOpType) {`.
  **L121 CN**: 继续构造周围的表达式或声明：`mlir::FunctionType callSiteType, mlir::FunctionType funcOpType) {`。
- **L122 EN**: Comment explains nearby logic, intent, or metadata: `Deal with argument number mismatch by making a function pointer so`.
  **L122 CN**: 注释说明附近代码的逻辑、意图或元数据：`Deal with argument number mismatch by making a function pointer so`。
- **L123 EN**: Comment explains nearby logic, intent, or metadata: `that function type cast can be inserted. Do not emit a warning here`.
  **L123 CN**: 注释说明附近代码的逻辑、意图或元数据：`that function type cast can be inserted. Do not emit a warning here`。
- **L124 EN**: Comment explains nearby logic, intent, or metadata: `because this can happen in legal program if the function is not`.
  **L124 CN**: 注释说明附近代码的逻辑、意图或元数据：`because this can happen in legal program if the function is not`。
- **L125 EN**: Comment explains nearby logic, intent, or metadata: `defined here and it was first passed as an argument without any more`.
  **L125 CN**: 注释说明附近代码的逻辑、意图或元数据：`defined here and it was first passed as an argument without any more`。
- **L126 EN**: Comment explains nearby logic, intent, or metadata: `information.`.
  **L126 CN**: 注释说明附近代码的逻辑、意图或元数据：`information.`。
- **L127 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L127 CN**: 开始 `if` 控制流语句并计算其条件。
- **L128 EN**: Continues logic associated with callable symbol `getNumInputs`.
  **L128 CN**: 继续与可调用符号 `getNumInputs` 相关的逻辑。
- **L129 EN**: Returns from the current function with `true`.
  **L129 CN**: 以 `true` 从当前函数返回。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L131 EN**: Comment explains nearby logic, intent, or metadata: `Implicit interface result type mismatch are not standard Fortran, but`.
  **L131 CN**: 注释说明附近代码的逻辑、意图或元数据：`Implicit interface result type mismatch are not standard Fortran, but`。
- **L132 EN**: Comment explains nearby logic, intent, or metadata: `some compilers are not complaining about it.  The front end is not`.
  **L132 CN**: 注释说明附近代码的逻辑、意图或元数据：`some compilers are not complaining about it.  The front end is not`。
- **L133 EN**: Comment explains nearby logic, intent, or metadata: `protecting lowering from this currently. Support this with a`.
  **L133 CN**: 注释说明附近代码的逻辑、意图或元数据：`protecting lowering from this currently. Support this with a`。
- **L134 EN**: Comment explains nearby logic, intent, or metadata: `discouraging warning.`.
  **L134 CN**: 注释说明附近代码的逻辑、意图或元数据：`discouraging warning.`。
- **L135 EN**: Comment explains nearby logic, intent, or metadata: `Cast the actual function to the current caller implicit type because`.
  **L135 CN**: 注释说明附近代码的逻辑、意图或元数据：`Cast the actual function to the current caller implicit type because`。
- **L136 EN**: Comment explains nearby logic, intent, or metadata: `that is the behavior we would get if we could not see the definition.`.
  **L136 CN**: 注释说明附近代码的逻辑、意图或元数据：`that is the behavior we would get if we could not see the definition.`。
- **L137 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L137 CN**: 开始 `if` 控制流语句并计算其条件。
- **L138 EN**: Continues logic associated with callable symbol `LLVM_DEBUG`.
  **L138 CN**: 继续与可调用符号 `LLVM_DEBUG` 相关的逻辑。
- **L139 EN**: Continues the surrounding expression or declaration: `loc, "a return type mismatch is not standard compliant and may "`.
  **L139 CN**: 继续构造周围的表达式或声明：`loc, "a return type mismatch is not standard compliant and may "`。
- **L140 EN**: Executes a standalone statement or declaration: `"lead to undefined behavior."));`.
  **L140 CN**: 执行一条独立语句或声明：`"lead to undefined behavior."));`。
- **L141 EN**: Returns from the current function with `true`.
  **L141 CN**: 以 `true` 从当前函数返回。
- **L142 EN**: Closes the current lexical scope or compound statement.
  **L142 CN**: 结束当前词法作用域或复合语句块。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L144 EN**: Comment explains nearby logic, intent, or metadata: `In HLFIR, there is little attempt to cope with implicit interface`.
  **L144 CN**: 注释说明附近代码的逻辑、意图或元数据：`In HLFIR, there is little attempt to cope with implicit interface`。

### Lines 145-168

````cpp
  // mismatch on the arguments. The argument are always prepared according
  // to the implicit interface. Cast the actual function if any of the
  // argument mismatch cannot be dealt with a simple fir.convert.
  for (auto [actualType, dummyType] :
       llvm::zip(callSiteType.getInputs(), funcOpType.getInputs()))
    if (actualType != dummyType &&
        !fir::ConvertOp::canBeConverted(actualType, dummyType))
      return true;
  return false;
}

static mlir::Value readDim3Value(fir::FirOpBuilder &builder, mlir::Location loc,
                                 mlir::Value dim3Addr, llvm::StringRef comp) {
  mlir::Type i32Ty = builder.getI32Type();
  mlir::Type refI32Ty = fir::ReferenceType::get(i32Ty);
  llvm::SmallVector<mlir::Value> lenParams;

  mlir::Value designate = hlfir::DesignateOp::create(
      builder, loc, refI32Ty, dim3Addr, /*component=*/comp,
      /*componentShape=*/mlir::Value{}, hlfir::DesignateOp::Subscripts{},
      /*substring=*/mlir::ValueRange{}, /*complexPartAttr=*/std::nullopt,
      mlir::Value{}, lenParams);

  return hlfir::loadTrivialScalar(loc, builder, hlfir::Entity{designate});
````
- **L145 EN**: Comment explains nearby logic, intent, or metadata: `mismatch on the arguments. The argument are always prepared according`.
  **L145 CN**: 注释说明附近代码的逻辑、意图或元数据：`mismatch on the arguments. The argument are always prepared according`。
- **L146 EN**: Comment explains nearby logic, intent, or metadata: `to the implicit interface. Cast the actual function if any of the`.
  **L146 CN**: 注释说明附近代码的逻辑、意图或元数据：`to the implicit interface. Cast the actual function if any of the`。
- **L147 EN**: Comment explains nearby logic, intent, or metadata: `argument mismatch cannot be dealt with a simple fir.convert.`.
  **L147 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument mismatch cannot be dealt with a simple fir.convert.`。
- **L148 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L148 CN**: 开始 `for` 控制流语句并计算其条件。
- **L149 EN**: Continues logic associated with callable symbol `zip`.
  **L149 CN**: 继续与可调用符号 `zip` 相关的逻辑。
- **L150 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L150 CN**: 开始 `if` 控制流语句并计算其条件。
- **L151 EN**: Continues logic associated with callable symbol `canBeConverted`.
  **L151 CN**: 继续与可调用符号 `canBeConverted` 相关的逻辑。
- **L152 EN**: Returns from the current function with `true`.
  **L152 CN**: 以 `true` 从当前函数返回。
- **L153 EN**: Returns from the current function with `false`.
  **L153 CN**: 以 `false` 从当前函数返回。
- **L154 EN**: Closes the current lexical scope or compound statement.
  **L154 CN**: 结束当前词法作用域或复合语句块。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static mlir::Value readDim3Value(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`static mlir::Value readDim3Value(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L157 EN**: Continues the surrounding expression or declaration: `mlir::Value dim3Addr, llvm::StringRef comp) {`.
  **L157 CN**: 继续构造周围的表达式或声明：`mlir::Value dim3Addr, llvm::StringRef comp) {`。
- **L158 EN**: Initializes variable `i32Ty` from the right-hand expression.
  **L158 CN**: 使用右侧表达式初始化变量 `i32Ty`。
- **L159 EN**: Initializes variable `refI32Ty` from the right-hand expression.
  **L159 CN**: 使用右侧表达式初始化变量 `refI32Ty`。
- **L160 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> lenParams;`.
  **L160 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> lenParams;`。
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L162 EN**: Continues logic associated with callable symbol `create`.
  **L162 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, refI32Ty, dim3Addr, /*component=*/comp,`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, refI32Ty, dim3Addr, /*component=*/comp,`。
- **L164 EN**: Comment explains nearby logic, intent, or metadata: `componentShape=*/mlir::Value{}, hlfir::DesignateOp::Subscripts{},`.
  **L164 CN**: 注释说明附近代码的逻辑、意图或元数据：`componentShape=*/mlir::Value{}, hlfir::DesignateOp::Subscripts{},`。
- **L165 EN**: Comment explains nearby logic, intent, or metadata: `substring=*/mlir::ValueRange{}, /*complexPartAttr=*/std::nullopt,`.
  **L165 CN**: 注释说明附近代码的逻辑、意图或元数据：`substring=*/mlir::ValueRange{}, /*complexPartAttr=*/std::nullopt,`。
- **L166 EN**: Executes a standalone statement or declaration: `mlir::Value{}, lenParams);`.
  **L166 CN**: 执行一条独立语句或声明：`mlir::Value{}, lenParams);`。
- **L167 EN**: Blank line separating nearby declarations or logic blocks.
  **L167 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L168 EN**: Returns from the current function with `hlfir::loadTrivialScalar(loc, builder, hlfir::Entity{designate})`.
  **L168 CN**: 以 `hlfir::loadTrivialScalar(loc, builder, hlfir::Entity{designate})` 从当前函数返回。

### Lines 169-192

````cpp
}

static mlir::Value remapActualToDummyDescriptor(
    mlir::Location loc, Fortran::lower::AbstractConverter &converter,
    Fortran::lower::SymMap &symMap,
    const Fortran::lower::CallerInterface::PassedEntity &arg,
    Fortran::lower::CallerInterface &caller, bool isBindcCall) {
  fir::FirOpBuilder &builder = converter.getFirOpBuilder();
  mlir::IndexType idxTy = builder.getIndexType();
  mlir::Value zero = builder.createIntegerConstant(loc, idxTy, 0);
  Fortran::lower::StatementContext localStmtCtx;
  auto lowerSpecExpr = [&](const auto &expr,
                           bool isAssumedSizeExtent) -> mlir::Value {
    mlir::Value convertExpr = builder.createConvert(
        loc, idxTy, fir::getBase(converter.genExprValue(expr, localStmtCtx)));
    if (isAssumedSizeExtent)
      return convertExpr;
    return fir::factory::genMaxWithZero(builder, loc, convertExpr);
  };
  bool mapSymbols = caller.mustMapInterfaceSymbolsForDummyArgument(arg);
  if (mapSymbols) {
    symMap.pushScope();
    const Fortran::semantics::Symbol *sym = caller.getDummySymbol(arg);
    assert(sym && "call must have explicit interface to map interface symbols");
````
- **L169 EN**: Closes the current lexical scope or compound statement.
  **L169 CN**: 结束当前词法作用域或复合语句块。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L171 EN**: Continues logic associated with callable symbol `remapActualToDummyDescriptor`.
  **L171 CN**: 继续与可调用符号 `remapActualToDummyDescriptor` 相关的逻辑。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, Fortran::lower::AbstractConverter &converter,`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, Fortran::lower::AbstractConverter &converter,`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::SymMap &symMap,`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::SymMap &symMap,`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::CallerInterface::PassedEntity &arg,`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::CallerInterface::PassedEntity &arg,`。
- **L175 EN**: Continues the surrounding expression or declaration: `Fortran::lower::CallerInterface &caller, bool isBindcCall) {`.
  **L175 CN**: 继续构造周围的表达式或声明：`Fortran::lower::CallerInterface &caller, bool isBindcCall) {`。
- **L176 EN**: Executes a call or declaration centered on `converter.getFirOpBuilder`.
  **L176 CN**: 执行以 `converter.getFirOpBuilder` 为核心的调用或声明。
- **L177 EN**: Initializes variable `idxTy` from the right-hand expression.
  **L177 CN**: 使用右侧表达式初始化变量 `idxTy`。
- **L178 EN**: Initializes variable `zero` from the right-hand expression.
  **L178 CN**: 使用右侧表达式初始化变量 `zero`。
- **L179 EN**: Executes a standalone statement or declaration: `Fortran::lower::StatementContext localStmtCtx;`.
  **L179 CN**: 执行一条独立语句或声明：`Fortran::lower::StatementContext localStmtCtx;`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto lowerSpecExpr = [&](const auto &expr,`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto lowerSpecExpr = [&](const auto &expr,`。
- **L181 EN**: Continues the surrounding expression or declaration: `bool isAssumedSizeExtent) -> mlir::Value {`.
  **L181 CN**: 继续构造周围的表达式或声明：`bool isAssumedSizeExtent) -> mlir::Value {`。
- **L182 EN**: Continues logic associated with callable symbol `createConvert`.
  **L182 CN**: 继续与可调用符号 `createConvert` 相关的逻辑。
- **L183 EN**: Executes a call or declaration centered on `fir::getBase`.
  **L183 CN**: 执行以 `fir::getBase` 为核心的调用或声明。
- **L184 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L184 CN**: 开始 `if` 控制流语句并计算其条件。
- **L185 EN**: Returns from the current function with `convertExpr`.
  **L185 CN**: 以 `convertExpr` 从当前函数返回。
- **L186 EN**: Returns from the current function with `fir::factory::genMaxWithZero(builder, loc, convertExpr)`.
  **L186 CN**: 以 `fir::factory::genMaxWithZero(builder, loc, convertExpr)` 从当前函数返回。
- **L187 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L187 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L188 EN**: Initializes variable `mapSymbols` from the right-hand expression.
  **L188 CN**: 使用右侧表达式初始化变量 `mapSymbols`。
- **L189 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L189 CN**: 开始 `if` 控制流语句并计算其条件。
- **L190 EN**: Executes a call or declaration centered on `symMap.pushScope`.
  **L190 CN**: 执行以 `symMap.pushScope` 为核心的调用或声明。
- **L191 EN**: Executes a call or declaration centered on `caller.getDummySymbol`.
  **L191 CN**: 执行以 `caller.getDummySymbol` 为核心的调用或声明。
- **L192 EN**: Checks an internal invariant in debug builds.
  **L192 CN**: 在调试构建中检查内部不变式。

### Lines 193-216

````cpp
    Fortran::lower::mapCallInterfaceSymbolsForDummyArgument(converter, caller,
                                                            symMap, *sym);
  }
  llvm::SmallVector<mlir::Value> extents;
  llvm::SmallVector<mlir::Value> lengths;
  mlir::Type dummyBoxType = caller.getDummyArgumentType(arg);
  mlir::Type dummyBaseType = fir::unwrapPassByRefType(dummyBoxType);
  if (mlir::isa<fir::SequenceType>(dummyBaseType))
    caller.walkDummyArgumentExtents(
        arg, [&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {
          extents.emplace_back(lowerSpecExpr(e, isAssumedSizeExtent));
        });
  mlir::Value shape;
  if (!extents.empty()) {
    if (isBindcCall) {
      // Preserve zero lower bounds (see F'2023 18.5.3).
      llvm::SmallVector<mlir::Value> lowerBounds(extents.size(), zero);
      shape = builder.genShape(loc, lowerBounds, extents);
    } else {
      shape = builder.genShape(loc, extents);
    }
  }

  hlfir::Entity explicitArgument = hlfir::Entity{caller.getInput(arg)};
````
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::mapCallInterfaceSymbolsForDummyArgument(converter, caller,`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::mapCallInterfaceSymbolsForDummyArgument(converter, caller,`。
- **L194 EN**: Executes a standalone statement or declaration: `symMap, *sym);`.
  **L194 CN**: 执行一条独立语句或声明：`symMap, *sym);`。
- **L195 EN**: Closes the current lexical scope or compound statement.
  **L195 CN**: 结束当前词法作用域或复合语句块。
- **L196 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> extents;`.
  **L196 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> extents;`。
- **L197 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> lengths;`.
  **L197 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> lengths;`。
- **L198 EN**: Initializes variable `dummyBoxType` from the right-hand expression.
  **L198 CN**: 使用右侧表达式初始化变量 `dummyBoxType`。
- **L199 EN**: Initializes variable `dummyBaseType` from the right-hand expression.
  **L199 CN**: 使用右侧表达式初始化变量 `dummyBaseType`。
- **L200 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L200 CN**: 开始 `if` 控制流语句并计算其条件。
- **L201 EN**: Continues logic associated with callable symbol `walkDummyArgumentExtents`.
  **L201 CN**: 继续与可调用符号 `walkDummyArgumentExtents` 相关的逻辑。
- **L202 EN**: Starts a function, method, lambda, or structured scope: `arg, [&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {`.
  **L202 CN**: 开始一个函数、方法、lambda 或结构化作用域：`arg, [&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {`。
- **L203 EN**: Executes a call or declaration centered on `extents.emplace_back`.
  **L203 CN**: 执行以 `extents.emplace_back` 为核心的调用或声明。
- **L204 EN**: Executes a standalone statement or declaration: `});`.
  **L204 CN**: 执行一条独立语句或声明：`});`。
- **L205 EN**: Executes a standalone statement or declaration: `mlir::Value shape;`.
  **L205 CN**: 执行一条独立语句或声明：`mlir::Value shape;`。
- **L206 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L206 CN**: 开始 `if` 控制流语句并计算其条件。
- **L207 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L207 CN**: 开始 `if` 控制流语句并计算其条件。
- **L208 EN**: Comment explains nearby logic, intent, or metadata: `Preserve zero lower bounds (see F'2023 18.5.3).`.
  **L208 CN**: 注释说明附近代码的逻辑、意图或元数据：`Preserve zero lower bounds (see F'2023 18.5.3).`。
- **L209 EN**: Executes a call or declaration centered on `lowerBounds`.
  **L209 CN**: 执行以 `lowerBounds` 为核心的调用或声明。
- **L210 EN**: Executes a call or declaration centered on `builder.genShape`.
  **L210 CN**: 执行以 `builder.genShape` 为核心的调用或声明。
- **L211 EN**: Transitions from the previous branch into the alternative path.
  **L211 CN**: 从前一个分支过渡到备选路径。
- **L212 EN**: Executes a call or declaration centered on `builder.genShape`.
  **L212 CN**: 执行以 `builder.genShape` 为核心的调用或声明。
- **L213 EN**: Closes the current lexical scope or compound statement.
  **L213 CN**: 结束当前词法作用域或复合语句块。
- **L214 EN**: Closes the current lexical scope or compound statement.
  **L214 CN**: 结束当前词法作用域或复合语句块。
- **L215 EN**: Blank line separating nearby declarations or logic blocks.
  **L215 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L216 EN**: Initializes variable `explicitArgument` from the right-hand expression.
  **L216 CN**: 使用右侧表达式初始化变量 `explicitArgument`。

### Lines 217-240

````cpp
  mlir::Type dummyElementType = fir::unwrapSequenceType(dummyBaseType);
  if (auto recType = llvm::dyn_cast<fir::RecordType>(dummyElementType))
    if (recType.getNumLenParams() > 0)
      TODO(loc, "sequence association of length parameterized derived type "
                "dummy arguments");
  if (fir::isa_char(dummyElementType))
    lengths.emplace_back(hlfir::genCharLength(loc, builder, explicitArgument));
  mlir::Value baseAddr =
      hlfir::genVariableRawAddress(loc, builder, explicitArgument);
  baseAddr = builder.createConvert(loc, fir::ReferenceType::get(dummyBaseType),
                                   baseAddr);
  mlir::Value mold;
  if (fir::isPolymorphicType(dummyBoxType))
    mold = explicitArgument;
  mlir::Value remapped =
      fir::EmboxOp::create(builder, loc, dummyBoxType, baseAddr, shape,
                           /*slice=*/mlir::Value{}, lengths, mold);
  if (mapSymbols)
    symMap.popScope();
  return remapped;
}

/// Create a descriptor for sequenced associated descriptor that are passed
/// by descriptor. Sequence association (F'2023 15.5.2.12) implies that the
````
- **L217 EN**: Initializes variable `dummyElementType` from the right-hand expression.
  **L217 CN**: 使用右侧表达式初始化变量 `dummyElementType`。
- **L218 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L218 CN**: 开始 `if` 控制流语句并计算其条件。
- **L219 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L219 CN**: 开始 `if` 控制流语句并计算其条件。
- **L220 EN**: Continues logic associated with callable symbol `TODO`.
  **L220 CN**: 继续与可调用符号 `TODO` 相关的逻辑。
- **L221 EN**: Executes a standalone statement or declaration: `"dummy arguments");`.
  **L221 CN**: 执行一条独立语句或声明：`"dummy arguments");`。
- **L222 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L222 CN**: 开始 `if` 控制流语句并计算其条件。
- **L223 EN**: Executes a call or declaration centered on `lengths.emplace_back`.
  **L223 CN**: 执行以 `lengths.emplace_back` 为核心的调用或声明。
- **L224 EN**: Continues the surrounding expression or declaration: `mlir::Value baseAddr =`.
  **L224 CN**: 继续构造周围的表达式或声明：`mlir::Value baseAddr =`。
- **L225 EN**: Executes a call or declaration centered on `hlfir::genVariableRawAddress`.
  **L225 CN**: 执行以 `hlfir::genVariableRawAddress` 为核心的调用或声明。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `baseAddr = builder.createConvert(loc, fir::ReferenceType::get(dummyBaseType),`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`baseAddr = builder.createConvert(loc, fir::ReferenceType::get(dummyBaseType),`。
- **L227 EN**: Executes a standalone statement or declaration: `baseAddr);`.
  **L227 CN**: 执行一条独立语句或声明：`baseAddr);`。
- **L228 EN**: Executes a standalone statement or declaration: `mlir::Value mold;`.
  **L228 CN**: 执行一条独立语句或声明：`mlir::Value mold;`。
- **L229 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L229 CN**: 开始 `if` 控制流语句并计算其条件。
- **L230 EN**: Executes a standalone statement or declaration: `mold = explicitArgument;`.
  **L230 CN**: 执行一条独立语句或声明：`mold = explicitArgument;`。
- **L231 EN**: Continues the surrounding expression or declaration: `mlir::Value remapped =`.
  **L231 CN**: 继续构造周围的表达式或声明：`mlir::Value remapped =`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::EmboxOp::create(builder, loc, dummyBoxType, baseAddr, shape,`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::EmboxOp::create(builder, loc, dummyBoxType, baseAddr, shape,`。
- **L233 EN**: Comment explains nearby logic, intent, or metadata: `slice=*/mlir::Value{}, lengths, mold);`.
  **L233 CN**: 注释说明附近代码的逻辑、意图或元数据：`slice=*/mlir::Value{}, lengths, mold);`。
- **L234 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L234 CN**: 开始 `if` 控制流语句并计算其条件。
- **L235 EN**: Executes a call or declaration centered on `symMap.popScope`.
  **L235 CN**: 执行以 `symMap.popScope` 为核心的调用或声明。
- **L236 EN**: Returns from the current function with `remapped`.
  **L236 CN**: 以 `remapped` 从当前函数返回。
- **L237 EN**: Closes the current lexical scope or compound statement.
  **L237 CN**: 结束当前词法作用域或复合语句块。
- **L238 EN**: Blank line separating nearby declarations or logic blocks.
  **L238 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L239 EN**: Comment explains nearby logic, intent, or metadata: `Create a descriptor for sequenced associated descriptor that are passed`.
  **L239 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create a descriptor for sequenced associated descriptor that are passed`。
- **L240 EN**: Comment explains nearby logic, intent, or metadata: `by descriptor. Sequence association (F'2023 15.5.2.12) implies that the`.
  **L240 CN**: 注释说明附近代码的逻辑、意图或元数据：`by descriptor. Sequence association (F'2023 15.5.2.12) implies that the`。

### Lines 241-264

````cpp
/// dummy shape and rank need to not be the same as the actual argument. This
/// helper creates a descriptor based on the dummy shape and rank (sequence
/// association can only happen with explicit and assumed-size array) so that it
/// is safe to assume the rank of the incoming descriptor inside the callee.
/// This helper must be called once all the actual arguments have been lowered
/// and placed inside "caller". Copy-in/copy-out must already have been
/// generated if needed using the actual argument shape (the dummy shape may be
/// assumed-size).
static void remapActualToDummyDescriptors(
    mlir::Location loc, Fortran::lower::AbstractConverter &converter,
    Fortran::lower::SymMap &symMap,
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    Fortran::lower::CallerInterface &caller, bool isBindcCall) {
  fir::FirOpBuilder &builder = converter.getFirOpBuilder();
  for (auto [preparedActual, arg] :
       llvm::zip(loweredActuals, caller.getPassedArguments())) {
    if (arg.isSequenceAssociatedDescriptor()) {
      if (!preparedActual.value().handleDynamicOptional()) {
        mlir::Value remapped = remapActualToDummyDescriptor(
            loc, converter, symMap, arg, caller, isBindcCall);
        caller.placeInput(arg, remapped);
      } else {
        // Absent optional actual argument descriptor cannot be read and
        // remapped unconditionally.
````
- **L241 EN**: Comment explains nearby logic, intent, or metadata: `dummy shape and rank need to not be the same as the actual argument. This`.
  **L241 CN**: 注释说明附近代码的逻辑、意图或元数据：`dummy shape and rank need to not be the same as the actual argument. This`。
- **L242 EN**: Comment explains nearby logic, intent, or metadata: `helper creates a descriptor based on the dummy shape and rank (sequence`.
  **L242 CN**: 注释说明附近代码的逻辑、意图或元数据：`helper creates a descriptor based on the dummy shape and rank (sequence`。
- **L243 EN**: Comment explains nearby logic, intent, or metadata: `association can only happen with explicit and assumed-size array) so that it`.
  **L243 CN**: 注释说明附近代码的逻辑、意图或元数据：`association can only happen with explicit and assumed-size array) so that it`。
- **L244 EN**: Comment explains nearby logic, intent, or metadata: `is safe to assume the rank of the incoming descriptor inside the callee.`.
  **L244 CN**: 注释说明附近代码的逻辑、意图或元数据：`is safe to assume the rank of the incoming descriptor inside the callee.`。
- **L245 EN**: Comment explains nearby logic, intent, or metadata: `This helper must be called once all the actual arguments have been lowered`.
  **L245 CN**: 注释说明附近代码的逻辑、意图或元数据：`This helper must be called once all the actual arguments have been lowered`。
- **L246 EN**: Comment explains nearby logic, intent, or metadata: `and placed inside "caller". Copy-in/copy-out must already have been`.
  **L246 CN**: 注释说明附近代码的逻辑、意图或元数据：`and placed inside "caller". Copy-in/copy-out must already have been`。
- **L247 EN**: Comment explains nearby logic, intent, or metadata: `generated if needed using the actual argument shape (the dummy shape may be`.
  **L247 CN**: 注释说明附近代码的逻辑、意图或元数据：`generated if needed using the actual argument shape (the dummy shape may be`。
- **L248 EN**: Comment explains nearby logic, intent, or metadata: `assumed-size).`.
  **L248 CN**: 注释说明附近代码的逻辑、意图或元数据：`assumed-size).`。
- **L249 EN**: Continues logic associated with callable symbol `remapActualToDummyDescriptors`.
  **L249 CN**: 继续与可调用符号 `remapActualToDummyDescriptors` 相关的逻辑。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, Fortran::lower::AbstractConverter &converter,`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, Fortran::lower::AbstractConverter &converter,`。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::SymMap &symMap,`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::SymMap &symMap,`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L253 EN**: Continues the surrounding expression or declaration: `Fortran::lower::CallerInterface &caller, bool isBindcCall) {`.
  **L253 CN**: 继续构造周围的表达式或声明：`Fortran::lower::CallerInterface &caller, bool isBindcCall) {`。
- **L254 EN**: Executes a call or declaration centered on `converter.getFirOpBuilder`.
  **L254 CN**: 执行以 `converter.getFirOpBuilder` 为核心的调用或声明。
- **L255 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L255 CN**: 开始 `for` 控制流语句并计算其条件。
- **L256 EN**: Starts a function, method, lambda, or structured scope: `llvm::zip(loweredActuals, caller.getPassedArguments())) {`.
  **L256 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::zip(loweredActuals, caller.getPassedArguments())) {`。
- **L257 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L257 CN**: 开始 `if` 控制流语句并计算其条件。
- **L258 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L258 CN**: 开始 `if` 控制流语句并计算其条件。
- **L259 EN**: Continues logic associated with callable symbol `remapActualToDummyDescriptor`.
  **L259 CN**: 继续与可调用符号 `remapActualToDummyDescriptor` 相关的逻辑。
- **L260 EN**: Executes a standalone statement or declaration: `loc, converter, symMap, arg, caller, isBindcCall);`.
  **L260 CN**: 执行一条独立语句或声明：`loc, converter, symMap, arg, caller, isBindcCall);`。
- **L261 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L261 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L262 EN**: Transitions from the previous branch into the alternative path.
  **L262 CN**: 从前一个分支过渡到备选路径。
- **L263 EN**: Comment explains nearby logic, intent, or metadata: `Absent optional actual argument descriptor cannot be read and`.
  **L263 CN**: 注释说明附近代码的逻辑、意图或元数据：`Absent optional actual argument descriptor cannot be read and`。
- **L264 EN**: Comment explains nearby logic, intent, or metadata: `remapped unconditionally.`.
  **L264 CN**: 注释说明附近代码的逻辑、意图或元数据：`remapped unconditionally.`。

### Lines 265-288

````cpp
        mlir::Type dummyType = caller.getDummyArgumentType(arg);
        mlir::Value isPresent = preparedActual.value().getIsPresent();
        auto &argLambdaCapture = arg;
        mlir::Value remapped =
            builder
                .genIfOp(loc, {dummyType}, isPresent,
                         /*withElseRegion=*/true)
                .genThen([&]() {
                  mlir::Value newBox = remapActualToDummyDescriptor(
                      loc, converter, symMap, argLambdaCapture, caller,
                      isBindcCall);
                  fir::ResultOp::create(builder, loc, newBox);
                })
                .genElse([&]() {
                  mlir::Value absent =
                      fir::AbsentOp::create(builder, loc, dummyType);
                  fir::ResultOp::create(builder, loc, absent);
                })
                .getResults()[0];
        caller.placeInput(arg, remapped);
      }
    }
  }
}
````
- **L265 EN**: Initializes variable `dummyType` from the right-hand expression.
  **L265 CN**: 使用右侧表达式初始化变量 `dummyType`。
- **L266 EN**: Initializes variable `isPresent` from the right-hand expression.
  **L266 CN**: 使用右侧表达式初始化变量 `isPresent`。
- **L267 EN**: Executes a standalone statement or declaration: `auto &argLambdaCapture = arg;`.
  **L267 CN**: 执行一条独立语句或声明：`auto &argLambdaCapture = arg;`。
- **L268 EN**: Continues the surrounding expression or declaration: `mlir::Value remapped =`.
  **L268 CN**: 继续构造周围的表达式或声明：`mlir::Value remapped =`。
- **L269 EN**: Continues the surrounding expression or declaration: `builder`.
  **L269 CN**: 继续构造周围的表达式或声明：`builder`。
- **L270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.genIfOp(loc, {dummyType}, isPresent,`.
  **L270 CN**: 继续一个多行参数列表、初始化器或聚合项：`.genIfOp(loc, {dummyType}, isPresent,`。
- **L271 EN**: Comment explains nearby logic, intent, or metadata: `withElseRegion=*/true)`.
  **L271 CN**: 注释说明附近代码的逻辑、意图或元数据：`withElseRegion=*/true)`。
- **L272 EN**: Starts a function, method, lambda, or structured scope: `.genThen([&]() {`.
  **L272 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genThen([&]() {`。
- **L273 EN**: Continues logic associated with callable symbol `remapActualToDummyDescriptor`.
  **L273 CN**: 继续与可调用符号 `remapActualToDummyDescriptor` 相关的逻辑。
- **L274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, converter, symMap, argLambdaCapture, caller,`.
  **L274 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, converter, symMap, argLambdaCapture, caller,`。
- **L275 EN**: Executes a standalone statement or declaration: `isBindcCall);`.
  **L275 CN**: 执行一条独立语句或声明：`isBindcCall);`。
- **L276 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L276 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L277 EN**: Continues the surrounding expression or declaration: `})`.
  **L277 CN**: 继续构造周围的表达式或声明：`})`。
- **L278 EN**: Starts a function, method, lambda, or structured scope: `.genElse([&]() {`.
  **L278 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genElse([&]() {`。
- **L279 EN**: Continues the surrounding expression or declaration: `mlir::Value absent =`.
  **L279 CN**: 继续构造周围的表达式或声明：`mlir::Value absent =`。
- **L280 EN**: Executes a call or declaration centered on `fir::AbsentOp::create`.
  **L280 CN**: 执行以 `fir::AbsentOp::create` 为核心的调用或声明。
- **L281 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L281 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L282 EN**: Continues the surrounding expression or declaration: `})`.
  **L282 CN**: 继续构造周围的表达式或声明：`})`。
- **L283 EN**: Executes a call or declaration centered on `.getResults`.
  **L283 CN**: 执行以 `.getResults` 为核心的调用或声明。
- **L284 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L284 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L285 EN**: Closes the current lexical scope or compound statement.
  **L285 CN**: 结束当前词法作用域或复合语句块。
- **L286 EN**: Closes the current lexical scope or compound statement.
  **L286 CN**: 结束当前词法作用域或复合语句块。
- **L287 EN**: Closes the current lexical scope or compound statement.
  **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Closes the current lexical scope or compound statement.
  **L288 CN**: 结束当前词法作用域或复合语句块。

### Lines 289-312

````cpp

static void
getResultLengthFromElementalOp(fir::FirOpBuilder &builder,
                               llvm::SmallVectorImpl<mlir::Value> &lengths) {
  auto elemental = llvm::dyn_cast_or_null<hlfir::ElementalOp>(
      builder.getInsertionBlock()->getParentOp());
  if (elemental)
    for (mlir::Value len : elemental.getTypeparams())
      lengths.push_back(len);
}

// Go through the args. Any descriptor args that have ignore_tkr(c) cause
// function type modification to avoid changing the descriptor args.
static std::optional<mlir::FunctionType>
getTypeWithIgnoreTkrC(mlir::FunctionType funcType,
                      Fortran::lower::CallerInterface &caller,
                      mlir::MLIRContext *context) {
  llvm::SmallVector<mlir::Type> newInputs =
      llvm::to_vector(funcType.getInputs());
  bool typeChanged = false;
  for (const auto &arg : caller.getPassedArguments()) {
    if (arg.firArgument >= 0 &&
        arg.firArgument < static_cast<int>(newInputs.size())) {

````
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L290 EN**: Continues the surrounding expression or declaration: `static void`.
  **L290 CN**: 继续构造周围的表达式或声明：`static void`。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getResultLengthFromElementalOp(fir::FirOpBuilder &builder,`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`getResultLengthFromElementalOp(fir::FirOpBuilder &builder,`。
- **L292 EN**: Continues the surrounding expression or declaration: `llvm::SmallVectorImpl<mlir::Value> &lengths) {`.
  **L292 CN**: 继续构造周围的表达式或声明：`llvm::SmallVectorImpl<mlir::Value> &lengths) {`。
- **L293 EN**: Continues logic associated with callable symbol `ElementalOp>`.
  **L293 CN**: 继续与可调用符号 `ElementalOp>` 相关的逻辑。
- **L294 EN**: Executes a call or declaration centered on `builder.getInsertionBlock`.
  **L294 CN**: 执行以 `builder.getInsertionBlock` 为核心的调用或声明。
- **L295 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L295 CN**: 开始 `if` 控制流语句并计算其条件。
- **L296 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L296 CN**: 开始 `for` 控制流语句并计算其条件。
- **L297 EN**: Executes a call or declaration centered on `lengths.push_back`.
  **L297 CN**: 执行以 `lengths.push_back` 为核心的调用或声明。
- **L298 EN**: Closes the current lexical scope or compound statement.
  **L298 CN**: 结束当前词法作用域或复合语句块。
- **L299 EN**: Blank line separating nearby declarations or logic blocks.
  **L299 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L300 EN**: Comment explains nearby logic, intent, or metadata: `Go through the args. Any descriptor args that have ignore_tkr(c) cause`.
  **L300 CN**: 注释说明附近代码的逻辑、意图或元数据：`Go through the args. Any descriptor args that have ignore_tkr(c) cause`。
- **L301 EN**: Comment explains nearby logic, intent, or metadata: `function type modification to avoid changing the descriptor args.`.
  **L301 CN**: 注释说明附近代码的逻辑、意图或元数据：`function type modification to avoid changing the descriptor args.`。
- **L302 EN**: Continues the surrounding expression or declaration: `static std::optional<mlir::FunctionType>`.
  **L302 CN**: 继续构造周围的表达式或声明：`static std::optional<mlir::FunctionType>`。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getTypeWithIgnoreTkrC(mlir::FunctionType funcType,`.
  **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`getTypeWithIgnoreTkrC(mlir::FunctionType funcType,`。
- **L304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::CallerInterface &caller,`.
  **L304 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::CallerInterface &caller,`。
- **L305 EN**: Continues the surrounding expression or declaration: `mlir::MLIRContext *context) {`.
  **L305 CN**: 继续构造周围的表达式或声明：`mlir::MLIRContext *context) {`。
- **L306 EN**: Continues the surrounding expression or declaration: `llvm::SmallVector<mlir::Type> newInputs =`.
  **L306 CN**: 继续构造周围的表达式或声明：`llvm::SmallVector<mlir::Type> newInputs =`。
- **L307 EN**: Executes a call or declaration centered on `llvm::to_vector`.
  **L307 CN**: 执行以 `llvm::to_vector` 为核心的调用或声明。
- **L308 EN**: Initializes variable `typeChanged` from the right-hand expression.
  **L308 CN**: 使用右侧表达式初始化变量 `typeChanged`。
- **L309 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L309 CN**: 开始 `for` 控制流语句并计算其条件。
- **L310 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L310 CN**: 开始 `if` 控制流语句并计算其条件。
- **L311 EN**: Starts a function, method, lambda, or structured scope: `arg.firArgument < static_cast<int>(newInputs.size())) {`.
  **L311 CN**: 开始一个函数、方法、lambda 或结构化作用域：`arg.firArgument < static_cast<int>(newInputs.size())) {`。
- **L312 EN**: Blank line separating nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 313-336

````cpp
      // Only need to change the arg type for ignore_tkr(c)
      if (!arg.testTKR(Fortran::common::IgnoreTKR::Contiguous))
        continue;

      mlir::Type expectedType = newInputs[arg.firArgument];
      // Cast is only needed for descriptors
      if (!fir::isa_box_type(expectedType))
        continue;

      // Handle ignore_tkr(c) for descriptors
      mlir::Value actual = caller.getInput(arg);
      if (!actual)
        continue;

      mlir::Type actualType = actual.getType();
      if (fir::isBoxAddress(actualType)) {
        newInputs[arg.firArgument] = actualType;
        typeChanged = true;
      }
    }
  }

  if (typeChanged) {
    // At least one of the arguments had its type changed, so need to
````
- **L313 EN**: Comment explains nearby logic, intent, or metadata: `Only need to change the arg type for ignore_tkr(c)`.
  **L313 CN**: 注释说明附近代码的逻辑、意图或元数据：`Only need to change the arg type for ignore_tkr(c)`。
- **L314 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L314 CN**: 开始 `if` 控制流语句并计算其条件。
- **L315 EN**: Skips to the next loop iteration.
  **L315 CN**: 跳到下一次循环迭代。
- **L316 EN**: Blank line separating nearby declarations or logic blocks.
  **L316 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L317 EN**: Initializes variable `expectedType` from the right-hand expression.
  **L317 CN**: 使用右侧表达式初始化变量 `expectedType`。
- **L318 EN**: Comment explains nearby logic, intent, or metadata: `Cast is only needed for descriptors`.
  **L318 CN**: 注释说明附近代码的逻辑、意图或元数据：`Cast is only needed for descriptors`。
- **L319 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L319 CN**: 开始 `if` 控制流语句并计算其条件。
- **L320 EN**: Skips to the next loop iteration.
  **L320 CN**: 跳到下一次循环迭代。
- **L321 EN**: Blank line separating nearby declarations or logic blocks.
  **L321 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L322 EN**: Comment explains nearby logic, intent, or metadata: `Handle ignore_tkr(c) for descriptors`.
  **L322 CN**: 注释说明附近代码的逻辑、意图或元数据：`Handle ignore_tkr(c) for descriptors`。
- **L323 EN**: Initializes variable `actual` from the right-hand expression.
  **L323 CN**: 使用右侧表达式初始化变量 `actual`。
- **L324 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L324 CN**: 开始 `if` 控制流语句并计算其条件。
- **L325 EN**: Skips to the next loop iteration.
  **L325 CN**: 跳到下一次循环迭代。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L327 EN**: Initializes variable `actualType` from the right-hand expression.
  **L327 CN**: 使用右侧表达式初始化变量 `actualType`。
- **L328 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L328 CN**: 开始 `if` 控制流语句并计算其条件。
- **L329 EN**: Executes a standalone statement or declaration: `newInputs[arg.firArgument] = actualType;`.
  **L329 CN**: 执行一条独立语句或声明：`newInputs[arg.firArgument] = actualType;`。
- **L330 EN**: Executes a standalone statement or declaration: `typeChanged = true;`.
  **L330 CN**: 执行一条独立语句或声明：`typeChanged = true;`。
- **L331 EN**: Closes the current lexical scope or compound statement.
  **L331 CN**: 结束当前词法作用域或复合语句块。
- **L332 EN**: Closes the current lexical scope or compound statement.
  **L332 CN**: 结束当前词法作用域或复合语句块。
- **L333 EN**: Closes the current lexical scope or compound statement.
  **L333 CN**: 结束当前词法作用域或复合语句块。
- **L334 EN**: Blank line separating nearby declarations or logic blocks.
  **L334 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L335 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L335 CN**: 开始 `if` 控制流语句并计算其条件。
- **L336 EN**: Comment explains nearby logic, intent, or metadata: `At least one of the arguments had its type changed, so need to`.
  **L336 CN**: 注释说明附近代码的逻辑、意图或元数据：`At least one of the arguments had its type changed, so need to`。

### Lines 337-360

````cpp
    // create a new function type to be used in a cast.
    return mlir::FunctionType::get(context, newInputs, funcType.getResults());
  }

  return std::nullopt;
}

static bool mustDestroyOrFinalizeFunctionResult(
    mlir::FunctionType callSiteType,
    std::optional<Fortran::evaluate::DynamicType> retTy) {
  if (callSiteType.getNumResults() == 0 || !retTy.has_value())
    return false;
  if (fir::isPointerType(callSiteType.getResult(0)))
    return false;
  if (retTy->IsPolymorphic() || retTy->IsUnlimitedPolymorphic())
    return true;
  if (retTy->category() != Fortran::common::TypeCategory::Derived)
    return false;
  return Fortran::semantics::MayRequireFinalization(
             retTy->GetDerivedTypeSpec()) ||
         hlfir::mayHaveAllocatableComponent(callSiteType.getResult(0));
}

std::tuple<Fortran::lower::LoweredResult, bool, mlir::Operation *>
````
- **L337 EN**: Comment explains nearby logic, intent, or metadata: `create a new function type to be used in a cast.`.
  **L337 CN**: 注释说明附近代码的逻辑、意图或元数据：`create a new function type to be used in a cast.`。
- **L338 EN**: Returns from the current function with `mlir::FunctionType::get(context, newInputs, funcType.getResults())`.
  **L338 CN**: 以 `mlir::FunctionType::get(context, newInputs, funcType.getResults())` 从当前函数返回。
- **L339 EN**: Closes the current lexical scope or compound statement.
  **L339 CN**: 结束当前词法作用域或复合语句块。
- **L340 EN**: Blank line separating nearby declarations or logic blocks.
  **L340 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L341 EN**: Returns from the current function with `std::nullopt`.
  **L341 CN**: 以 `std::nullopt` 从当前函数返回。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Blank line separating nearby declarations or logic blocks.
  **L343 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L344 EN**: Continues logic associated with callable symbol `mustDestroyOrFinalizeFunctionResult`.
  **L344 CN**: 继续与可调用符号 `mustDestroyOrFinalizeFunctionResult` 相关的逻辑。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::FunctionType callSiteType,`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::FunctionType callSiteType,`。
- **L346 EN**: Continues the surrounding expression or declaration: `std::optional<Fortran::evaluate::DynamicType> retTy) {`.
  **L346 CN**: 继续构造周围的表达式或声明：`std::optional<Fortran::evaluate::DynamicType> retTy) {`。
- **L347 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L347 CN**: 开始 `if` 控制流语句并计算其条件。
- **L348 EN**: Returns from the current function with `false`.
  **L348 CN**: 以 `false` 从当前函数返回。
- **L349 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L349 CN**: 开始 `if` 控制流语句并计算其条件。
- **L350 EN**: Returns from the current function with `false`.
  **L350 CN**: 以 `false` 从当前函数返回。
- **L351 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L351 CN**: 开始 `if` 控制流语句并计算其条件。
- **L352 EN**: Returns from the current function with `true`.
  **L352 CN**: 以 `true` 从当前函数返回。
- **L353 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L353 CN**: 开始 `if` 控制流语句并计算其条件。
- **L354 EN**: Returns from the current function with `false`.
  **L354 CN**: 以 `false` 从当前函数返回。
- **L355 EN**: Returns from the current function with `Fortran::semantics::MayRequireFinalization(`.
  **L355 CN**: 以 `Fortran::semantics::MayRequireFinalization(` 从当前函数返回。
- **L356 EN**: Continues logic associated with callable symbol `GetDerivedTypeSpec`.
  **L356 CN**: 继续与可调用符号 `GetDerivedTypeSpec` 相关的逻辑。
- **L357 EN**: Executes a call or declaration centered on `hlfir::mayHaveAllocatableComponent`.
  **L357 CN**: 执行以 `hlfir::mayHaveAllocatableComponent` 为核心的调用或声明。
- **L358 EN**: Closes the current lexical scope or compound statement.
  **L358 CN**: 结束当前词法作用域或复合语句块。
- **L359 EN**: Blank line separating nearby declarations or logic blocks.
  **L359 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L360 EN**: Continues the surrounding expression or declaration: `std::tuple<Fortran::lower::LoweredResult, bool, mlir::Operation *>`.
  **L360 CN**: 继续构造周围的表达式或声明：`std::tuple<Fortran::lower::LoweredResult, bool, mlir::Operation *>`。

### Lines 361-384

````cpp
Fortran::lower::genCallOpAndResult(
    mlir::Location loc, Fortran::lower::AbstractConverter &converter,
    Fortran::lower::SymMap &symMap, Fortran::lower::StatementContext &stmtCtx,
    Fortran::lower::CallerInterface &caller, mlir::FunctionType callSiteType,
    std::optional<mlir::Type> resultType, bool isElemental) {
  fir::FirOpBuilder &builder = converter.getFirOpBuilder();
  using PassBy = Fortran::lower::CallerInterface::PassEntityBy;
  bool mustPopSymMap = false;
  mlir::Operation *callOp = nullptr;

  llvm::SmallVector<mlir::Value> resultLengths;
  if (isElemental)
    getResultLengthFromElementalOp(builder, resultLengths);
  if (caller.mustMapInterfaceSymbolsForResult() && resultLengths.empty()) {
    // Do not map the dummy symbols again inside the loop to compute elemental
    // function result whose length was already computed outside of the loop.
    symMap.pushScope();
    mustPopSymMap = true;
    Fortran::lower::mapCallInterfaceSymbolsForResult(converter, caller, symMap);
  }
  // If this is an indirect call, retrieve the function address. Also retrieve
  // the result length if this is a character function (note that this length
  // will be used only if there is no explicit length in the local interface).
  mlir::Value funcPointer;
````
- **L361 EN**: Continues logic associated with callable symbol `genCallOpAndResult`.
  **L361 CN**: 继续与可调用符号 `genCallOpAndResult` 相关的逻辑。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, Fortran::lower::AbstractConverter &converter,`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, Fortran::lower::AbstractConverter &converter,`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::SymMap &symMap, Fortran::lower::StatementContext &stmtCtx,`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::SymMap &symMap, Fortran::lower::StatementContext &stmtCtx,`。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::CallerInterface &caller, mlir::FunctionType callSiteType,`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::CallerInterface &caller, mlir::FunctionType callSiteType,`。
- **L365 EN**: Continues the surrounding expression or declaration: `std::optional<mlir::Type> resultType, bool isElemental) {`.
  **L365 CN**: 继续构造周围的表达式或声明：`std::optional<mlir::Type> resultType, bool isElemental) {`。
- **L366 EN**: Executes a call or declaration centered on `converter.getFirOpBuilder`.
  **L366 CN**: 执行以 `converter.getFirOpBuilder` 为核心的调用或声明。
- **L367 EN**: Defines alias `PassBy` to simplify later code.
  **L367 CN**: 定义别名 `PassBy` 以简化后续代码。
- **L368 EN**: Initializes variable `mustPopSymMap` from the right-hand expression.
  **L368 CN**: 使用右侧表达式初始化变量 `mustPopSymMap`。
- **L369 EN**: Executes a standalone statement or declaration: `mlir::Operation *callOp = nullptr;`.
  **L369 CN**: 执行一条独立语句或声明：`mlir::Operation *callOp = nullptr;`。
- **L370 EN**: Blank line separating nearby declarations or logic blocks.
  **L370 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L371 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> resultLengths;`.
  **L371 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> resultLengths;`。
- **L372 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L372 CN**: 开始 `if` 控制流语句并计算其条件。
- **L373 EN**: Executes a call or declaration centered on `getResultLengthFromElementalOp`.
  **L373 CN**: 执行以 `getResultLengthFromElementalOp` 为核心的调用或声明。
- **L374 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L374 CN**: 开始 `if` 控制流语句并计算其条件。
- **L375 EN**: Comment explains nearby logic, intent, or metadata: `Do not map the dummy symbols again inside the loop to compute elemental`.
  **L375 CN**: 注释说明附近代码的逻辑、意图或元数据：`Do not map the dummy symbols again inside the loop to compute elemental`。
- **L376 EN**: Comment explains nearby logic, intent, or metadata: `function result whose length was already computed outside of the loop.`.
  **L376 CN**: 注释说明附近代码的逻辑、意图或元数据：`function result whose length was already computed outside of the loop.`。
- **L377 EN**: Executes a call or declaration centered on `symMap.pushScope`.
  **L377 CN**: 执行以 `symMap.pushScope` 为核心的调用或声明。
- **L378 EN**: Executes a standalone statement or declaration: `mustPopSymMap = true;`.
  **L378 CN**: 执行一条独立语句或声明：`mustPopSymMap = true;`。
- **L379 EN**: Executes a call or declaration centered on `Fortran::lower::mapCallInterfaceSymbolsForResult`.
  **L379 CN**: 执行以 `Fortran::lower::mapCallInterfaceSymbolsForResult` 为核心的调用或声明。
- **L380 EN**: Closes the current lexical scope or compound statement.
  **L380 CN**: 结束当前词法作用域或复合语句块。
- **L381 EN**: Comment explains nearby logic, intent, or metadata: `If this is an indirect call, retrieve the function address. Also retrieve`.
  **L381 CN**: 注释说明附近代码的逻辑、意图或元数据：`If this is an indirect call, retrieve the function address. Also retrieve`。
- **L382 EN**: Comment explains nearby logic, intent, or metadata: `the result length if this is a character function (note that this length`.
  **L382 CN**: 注释说明附近代码的逻辑、意图或元数据：`the result length if this is a character function (note that this length`。
- **L383 EN**: Comment explains nearby logic, intent, or metadata: `will be used only if there is no explicit length in the local interface).`.
  **L383 CN**: 注释说明附近代码的逻辑、意图或元数据：`will be used only if there is no explicit length in the local interface).`。
- **L384 EN**: Executes a standalone statement or declaration: `mlir::Value funcPointer;`.
  **L384 CN**: 执行一条独立语句或声明：`mlir::Value funcPointer;`。

### Lines 385-408

````cpp
  mlir::Value charFuncPointerLength;
  if (const Fortran::evaluate::ProcedureDesignator *procDesignator =
          caller.getIfIndirectCall()) {
    if (mlir::Value passedArg = caller.getIfPassedArg()) {
      // Procedure pointer component call with PASS argument. To avoid
      // "double" lowering of the ComponentRef, semantics only place the
      // ComponentRef in the ActualArguments, not in the ProcedureDesignator (
      // that is only the component symbol).
      // Fetch the passed argument and addresses of its procedure pointer
      // component.
      funcPointer = Fortran::lower::derefPassProcPointerComponent(
          loc, converter, *procDesignator, passedArg, symMap, stmtCtx);
    } else {
      Fortran::lower::SomeExpr expr{*procDesignator};
      fir::ExtendedValue loweredProc =
          converter.genExprAddr(loc, expr, stmtCtx);
      funcPointer = fir::getBase(loweredProc);
      // Dummy procedure may have assumed length, in which case the result
      // length was passed along the dummy procedure.
      // This is not possible with procedure pointer components.
      if (const fir::CharBoxValue *charBox = loweredProc.getCharBox())
        charFuncPointerLength = charBox->getLen();
    }
  }
````
- **L385 EN**: Executes a standalone statement or declaration: `mlir::Value charFuncPointerLength;`.
  **L385 CN**: 执行一条独立语句或声明：`mlir::Value charFuncPointerLength;`。
- **L386 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L386 CN**: 开始 `if` 控制流语句并计算其条件。
- **L387 EN**: Starts a function, method, lambda, or structured scope: `caller.getIfIndirectCall()) {`.
  **L387 CN**: 开始一个函数、方法、lambda 或结构化作用域：`caller.getIfIndirectCall()) {`。
- **L388 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L388 CN**: 开始 `if` 控制流语句并计算其条件。
- **L389 EN**: Comment explains nearby logic, intent, or metadata: `Procedure pointer component call with PASS argument. To avoid`.
  **L389 CN**: 注释说明附近代码的逻辑、意图或元数据：`Procedure pointer component call with PASS argument. To avoid`。
- **L390 EN**: Comment explains nearby logic, intent, or metadata: `"double" lowering of the ComponentRef, semantics only place the`.
  **L390 CN**: 注释说明附近代码的逻辑、意图或元数据：`"double" lowering of the ComponentRef, semantics only place the`。
- **L391 EN**: Comment explains nearby logic, intent, or metadata: `ComponentRef in the ActualArguments, not in the ProcedureDesignator (`.
  **L391 CN**: 注释说明附近代码的逻辑、意图或元数据：`ComponentRef in the ActualArguments, not in the ProcedureDesignator (`。
- **L392 EN**: Comment explains nearby logic, intent, or metadata: `that is only the component symbol).`.
  **L392 CN**: 注释说明附近代码的逻辑、意图或元数据：`that is only the component symbol).`。
- **L393 EN**: Comment explains nearby logic, intent, or metadata: `Fetch the passed argument and addresses of its procedure pointer`.
  **L393 CN**: 注释说明附近代码的逻辑、意图或元数据：`Fetch the passed argument and addresses of its procedure pointer`。
- **L394 EN**: Comment explains nearby logic, intent, or metadata: `component.`.
  **L394 CN**: 注释说明附近代码的逻辑、意图或元数据：`component.`。
- **L395 EN**: Continues logic associated with callable symbol `derefPassProcPointerComponent`.
  **L395 CN**: 继续与可调用符号 `derefPassProcPointerComponent` 相关的逻辑。
- **L396 EN**: Executes a standalone statement or declaration: `loc, converter, *procDesignator, passedArg, symMap, stmtCtx);`.
  **L396 CN**: 执行一条独立语句或声明：`loc, converter, *procDesignator, passedArg, symMap, stmtCtx);`。
- **L397 EN**: Transitions from the previous branch into the alternative path.
  **L397 CN**: 从前一个分支过渡到备选路径。
- **L398 EN**: Executes a standalone statement or declaration: `Fortran::lower::SomeExpr expr{*procDesignator};`.
  **L398 CN**: 执行一条独立语句或声明：`Fortran::lower::SomeExpr expr{*procDesignator};`。
- **L399 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue loweredProc =`.
  **L399 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue loweredProc =`。
- **L400 EN**: Executes a call or declaration centered on `converter.genExprAddr`.
  **L400 CN**: 执行以 `converter.genExprAddr` 为核心的调用或声明。
- **L401 EN**: Executes a call or declaration centered on `fir::getBase`.
  **L401 CN**: 执行以 `fir::getBase` 为核心的调用或声明。
- **L402 EN**: Comment explains nearby logic, intent, or metadata: `Dummy procedure may have assumed length, in which case the result`.
  **L402 CN**: 注释说明附近代码的逻辑、意图或元数据：`Dummy procedure may have assumed length, in which case the result`。
- **L403 EN**: Comment explains nearby logic, intent, or metadata: `length was passed along the dummy procedure.`.
  **L403 CN**: 注释说明附近代码的逻辑、意图或元数据：`length was passed along the dummy procedure.`。
- **L404 EN**: Comment explains nearby logic, intent, or metadata: `This is not possible with procedure pointer components.`.
  **L404 CN**: 注释说明附近代码的逻辑、意图或元数据：`This is not possible with procedure pointer components.`。
- **L405 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L405 CN**: 开始 `if` 控制流语句并计算其条件。
- **L406 EN**: Executes a call or declaration centered on `charBox->getLen`.
  **L406 CN**: 执行以 `charBox->getLen` 为核心的调用或声明。
- **L407 EN**: Closes the current lexical scope or compound statement.
  **L407 CN**: 结束当前词法作用域或复合语句块。
- **L408 EN**: Closes the current lexical scope or compound statement.
  **L408 CN**: 结束当前词法作用域或复合语句块。

### Lines 409-432

````cpp
  const bool isExprCall =
      callSiteType.getNumResults() == 1 &&
      llvm::isa<fir::SequenceType>(callSiteType.getResult(0));

  mlir::IndexType idxTy = builder.getIndexType();
  auto lowerSpecExpr = [&](const auto &expr) -> mlir::Value {
    mlir::Value convertExpr = builder.createConvert(
        loc, idxTy, fir::getBase(converter.genExprValue(expr, stmtCtx)));
    return fir::factory::genMaxWithZero(builder, loc, convertExpr);
  };
  mlir::Value arrayResultShape;
  hlfir::EvaluateInMemoryOp evaluateInMemory;
  auto allocatedResult = [&]() -> std::optional<fir::ExtendedValue> {
    llvm::SmallVector<mlir::Value> extents;
    llvm::SmallVector<mlir::Value> lengths;
    if (!caller.callerAllocateResult())
      return {};
    mlir::Type type = caller.getResultStorageType();
    if (mlir::isa<fir::SequenceType>(type))
      caller.walkResultExtents(
          [&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {
            assert(!isAssumedSizeExtent && "result cannot be assumed-size");
            extents.emplace_back(lowerSpecExpr(e));
          });
````
- **L409 EN**: Continues the surrounding expression or declaration: `const bool isExprCall =`.
  **L409 CN**: 继续构造周围的表达式或声明：`const bool isExprCall =`。
- **L410 EN**: Continues logic associated with callable symbol `getNumResults`.
  **L410 CN**: 继续与可调用符号 `getNumResults` 相关的逻辑。
- **L411 EN**: Executes a call or declaration centered on `llvm::isa<fir::SequenceType>`.
  **L411 CN**: 执行以 `llvm::isa<fir::SequenceType>` 为核心的调用或声明。
- **L412 EN**: Blank line separating nearby declarations or logic blocks.
  **L412 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L413 EN**: Initializes variable `idxTy` from the right-hand expression.
  **L413 CN**: 使用右侧表达式初始化变量 `idxTy`。
- **L414 EN**: Starts a function, method, lambda, or structured scope: `auto lowerSpecExpr = [&](const auto &expr) -> mlir::Value {`.
  **L414 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto lowerSpecExpr = [&](const auto &expr) -> mlir::Value {`。
- **L415 EN**: Continues logic associated with callable symbol `createConvert`.
  **L415 CN**: 继续与可调用符号 `createConvert` 相关的逻辑。
- **L416 EN**: Executes a call or declaration centered on `fir::getBase`.
  **L416 CN**: 执行以 `fir::getBase` 为核心的调用或声明。
- **L417 EN**: Returns from the current function with `fir::factory::genMaxWithZero(builder, loc, convertExpr)`.
  **L417 CN**: 以 `fir::factory::genMaxWithZero(builder, loc, convertExpr)` 从当前函数返回。
- **L418 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L418 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L419 EN**: Executes a standalone statement or declaration: `mlir::Value arrayResultShape;`.
  **L419 CN**: 执行一条独立语句或声明：`mlir::Value arrayResultShape;`。
- **L420 EN**: Executes a standalone statement or declaration: `hlfir::EvaluateInMemoryOp evaluateInMemory;`.
  **L420 CN**: 执行一条独立语句或声明：`hlfir::EvaluateInMemoryOp evaluateInMemory;`。
- **L421 EN**: Starts a function, method, lambda, or structured scope: `auto allocatedResult = [&]() -> std::optional<fir::ExtendedValue> {`.
  **L421 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto allocatedResult = [&]() -> std::optional<fir::ExtendedValue> {`。
- **L422 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> extents;`.
  **L422 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> extents;`。
- **L423 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> lengths;`.
  **L423 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> lengths;`。
- **L424 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L424 CN**: 开始 `if` 控制流语句并计算其条件。
- **L425 EN**: Returns from the current function with `{}`.
  **L425 CN**: 以 `{}` 从当前函数返回。
- **L426 EN**: Initializes variable `type` from the right-hand expression.
  **L426 CN**: 使用右侧表达式初始化变量 `type`。
- **L427 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L427 CN**: 开始 `if` 控制流语句并计算其条件。
- **L428 EN**: Continues logic associated with callable symbol `walkResultExtents`.
  **L428 CN**: 继续与可调用符号 `walkResultExtents` 相关的逻辑。
- **L429 EN**: Starts a function, method, lambda, or structured scope: `[&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {`.
  **L429 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {`。
- **L430 EN**: Checks an internal invariant in debug builds.
  **L430 CN**: 在调试构建中检查内部不变式。
- **L431 EN**: Executes a call or declaration centered on `extents.emplace_back`.
  **L431 CN**: 执行以 `extents.emplace_back` 为核心的调用或声明。
- **L432 EN**: Executes a standalone statement or declaration: `});`.
  **L432 CN**: 执行一条独立语句或声明：`});`。

### Lines 433-456

````cpp
    if (resultLengths.empty()) {
      caller.walkResultLengths(
          [&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {
            assert(!isAssumedSizeExtent && "result cannot be assumed-size");
            lengths.emplace_back(lowerSpecExpr(e));
          });
    } else {
      // Use lengths precomputed before elemental loops.
      lengths = resultLengths;
    }

    // Result length parameters should not be provided to box storage
    // allocation and save_results, but they are still useful information to
    // keep in the ExtendedValue if non-deferred.
    if (!mlir::isa<fir::BoxType>(type)) {
      if (fir::isa_char(fir::unwrapSequenceType(type)) && lengths.empty()) {
        // Calling an assumed length function. This is only possible if this
        // is a call to a character dummy procedure.
        if (!charFuncPointerLength)
          fir::emitFatalError(loc, "failed to retrieve character function "
                                   "length while calling it");
        lengths.push_back(charFuncPointerLength);
      }
      resultLengths = lengths;
````
- **L433 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L433 CN**: 开始 `if` 控制流语句并计算其条件。
- **L434 EN**: Continues logic associated with callable symbol `walkResultLengths`.
  **L434 CN**: 继续与可调用符号 `walkResultLengths` 相关的逻辑。
- **L435 EN**: Starts a function, method, lambda, or structured scope: `[&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {`.
  **L435 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {`。
- **L436 EN**: Checks an internal invariant in debug builds.
  **L436 CN**: 在调试构建中检查内部不变式。
- **L437 EN**: Executes a call or declaration centered on `lengths.emplace_back`.
  **L437 CN**: 执行以 `lengths.emplace_back` 为核心的调用或声明。
- **L438 EN**: Executes a standalone statement or declaration: `});`.
  **L438 CN**: 执行一条独立语句或声明：`});`。
- **L439 EN**: Transitions from the previous branch into the alternative path.
  **L439 CN**: 从前一个分支过渡到备选路径。
- **L440 EN**: Comment explains nearby logic, intent, or metadata: `Use lengths precomputed before elemental loops.`.
  **L440 CN**: 注释说明附近代码的逻辑、意图或元数据：`Use lengths precomputed before elemental loops.`。
- **L441 EN**: Executes a standalone statement or declaration: `lengths = resultLengths;`.
  **L441 CN**: 执行一条独立语句或声明：`lengths = resultLengths;`。
- **L442 EN**: Closes the current lexical scope or compound statement.
  **L442 CN**: 结束当前词法作用域或复合语句块。
- **L443 EN**: Blank line separating nearby declarations or logic blocks.
  **L443 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L444 EN**: Comment explains nearby logic, intent, or metadata: `Result length parameters should not be provided to box storage`.
  **L444 CN**: 注释说明附近代码的逻辑、意图或元数据：`Result length parameters should not be provided to box storage`。
- **L445 EN**: Comment explains nearby logic, intent, or metadata: `allocation and save_results, but they are still useful information to`.
  **L445 CN**: 注释说明附近代码的逻辑、意图或元数据：`allocation and save_results, but they are still useful information to`。
- **L446 EN**: Comment explains nearby logic, intent, or metadata: `keep in the ExtendedValue if non-deferred.`.
  **L446 CN**: 注释说明附近代码的逻辑、意图或元数据：`keep in the ExtendedValue if non-deferred.`。
- **L447 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L447 CN**: 开始 `if` 控制流语句并计算其条件。
- **L448 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L448 CN**: 开始 `if` 控制流语句并计算其条件。
- **L449 EN**: Comment explains nearby logic, intent, or metadata: `Calling an assumed length function. This is only possible if this`.
  **L449 CN**: 注释说明附近代码的逻辑、意图或元数据：`Calling an assumed length function. This is only possible if this`。
- **L450 EN**: Comment explains nearby logic, intent, or metadata: `is a call to a character dummy procedure.`.
  **L450 CN**: 注释说明附近代码的逻辑、意图或元数据：`is a call to a character dummy procedure.`。
- **L451 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L451 CN**: 开始 `if` 控制流语句并计算其条件。
- **L452 EN**: Continues logic associated with callable symbol `emitFatalError`.
  **L452 CN**: 继续与可调用符号 `emitFatalError` 相关的逻辑。
- **L453 EN**: Executes a standalone statement or declaration: `"length while calling it");`.
  **L453 CN**: 执行一条独立语句或声明：`"length while calling it");`。
- **L454 EN**: Executes a call or declaration centered on `lengths.push_back`.
  **L454 CN**: 执行以 `lengths.push_back` 为核心的调用或声明。
- **L455 EN**: Closes the current lexical scope or compound statement.
  **L455 CN**: 结束当前词法作用域或复合语句块。
- **L456 EN**: Executes a standalone statement or declaration: `resultLengths = lengths;`.
  **L456 CN**: 执行一条独立语句或声明：`resultLengths = lengths;`。

### Lines 457-480

````cpp
    }

    if (!extents.empty())
      arrayResultShape = builder.genShape(loc, extents);

    if (isExprCall) {
      mlir::Type exprType = hlfir::getExprType(type);
      evaluateInMemory = hlfir::EvaluateInMemoryOp::create(
          builder, loc, exprType, arrayResultShape, resultLengths);
      builder.setInsertionPointToStart(&evaluateInMemory.getBody().front());
      return toExtendedValue(loc, evaluateInMemory.getMemory(), extents,
                             lengths);
    }

    if ((!extents.empty() || !lengths.empty()) && !isElemental) {
      // Note: in the elemental context, the alloca ownership inside the
      // elemental region is implicit, and later pass in lowering (stack
      // reclaim) fir.do_loop will be in charge of emitting any stack
      // save/restore if needed.
      auto *bldr = &converter.getFirOpBuilder();
      mlir::Value sp = bldr->genStackSave(loc);
      stmtCtx.attachCleanup(
          [bldr, loc, sp]() { bldr->genStackRestore(loc, sp); });
    }
````
- **L457 EN**: Closes the current lexical scope or compound statement.
  **L457 CN**: 结束当前词法作用域或复合语句块。
- **L458 EN**: Blank line separating nearby declarations or logic blocks.
  **L458 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L459 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L459 CN**: 开始 `if` 控制流语句并计算其条件。
- **L460 EN**: Executes a call or declaration centered on `builder.genShape`.
  **L460 CN**: 执行以 `builder.genShape` 为核心的调用或声明。
- **L461 EN**: Blank line separating nearby declarations or logic blocks.
  **L461 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L462 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L462 CN**: 开始 `if` 控制流语句并计算其条件。
- **L463 EN**: Initializes variable `exprType` from the right-hand expression.
  **L463 CN**: 使用右侧表达式初始化变量 `exprType`。
- **L464 EN**: Continues logic associated with callable symbol `create`.
  **L464 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L465 EN**: Executes a standalone statement or declaration: `builder, loc, exprType, arrayResultShape, resultLengths);`.
  **L465 CN**: 执行一条独立语句或声明：`builder, loc, exprType, arrayResultShape, resultLengths);`。
- **L466 EN**: Executes a call or declaration centered on `builder.setInsertionPointToStart`.
  **L466 CN**: 执行以 `builder.setInsertionPointToStart` 为核心的调用或声明。
- **L467 EN**: Returns from the current function with `toExtendedValue(loc, evaluateInMemory.getMemory(), extents,`.
  **L467 CN**: 以 `toExtendedValue(loc, evaluateInMemory.getMemory(), extents,` 从当前函数返回。
- **L468 EN**: Executes a standalone statement or declaration: `lengths);`.
  **L468 CN**: 执行一条独立语句或声明：`lengths);`。
- **L469 EN**: Closes the current lexical scope or compound statement.
  **L469 CN**: 结束当前词法作用域或复合语句块。
- **L470 EN**: Blank line separating nearby declarations or logic blocks.
  **L470 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L471 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L471 CN**: 开始 `if` 控制流语句并计算其条件。
- **L472 EN**: Comment explains nearby logic, intent, or metadata: `Note: in the elemental context, the alloca ownership inside the`.
  **L472 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note: in the elemental context, the alloca ownership inside the`。
- **L473 EN**: Comment explains nearby logic, intent, or metadata: `elemental region is implicit, and later pass in lowering (stack`.
  **L473 CN**: 注释说明附近代码的逻辑、意图或元数据：`elemental region is implicit, and later pass in lowering (stack`。
- **L474 EN**: Comment explains nearby logic, intent, or metadata: `reclaim) fir.do_loop will be in charge of emitting any stack`.
  **L474 CN**: 注释说明附近代码的逻辑、意图或元数据：`reclaim) fir.do_loop will be in charge of emitting any stack`。
- **L475 EN**: Comment explains nearby logic, intent, or metadata: `save/restore if needed.`.
  **L475 CN**: 注释说明附近代码的逻辑、意图或元数据：`save/restore if needed.`。
- **L476 EN**: Executes a call or declaration centered on `&converter.getFirOpBuilder`.
  **L476 CN**: 执行以 `&converter.getFirOpBuilder` 为核心的调用或声明。
- **L477 EN**: Initializes variable `sp` from the right-hand expression.
  **L477 CN**: 使用右侧表达式初始化变量 `sp`。
- **L478 EN**: Continues logic associated with callable symbol `attachCleanup`.
  **L478 CN**: 继续与可调用符号 `attachCleanup` 相关的逻辑。
- **L479 EN**: Executes a call or declaration centered on `sp]`.
  **L479 CN**: 执行以 `sp]` 为核心的调用或声明。
- **L480 EN**: Closes the current lexical scope or compound statement.
  **L480 CN**: 结束当前词法作用域或复合语句块。

### Lines 481-504

````cpp
    mlir::Value temp =
        builder.createTemporary(loc, type, ".result", extents, resultLengths);
    return toExtendedValue(loc, temp, extents, lengths);
  }();

  if (mustPopSymMap)
    symMap.popScope();

  // Place allocated result
  if (allocatedResult) {
    if (std::optional<Fortran::lower::CallInterface<
            Fortran::lower::CallerInterface>::PassedEntity>
            resultArg = caller.getPassedResult()) {
      if (resultArg->passBy == PassBy::AddressAndLength)
        caller.placeAddressAndLengthInput(*resultArg,
                                          fir::getBase(*allocatedResult),
                                          fir::getLen(*allocatedResult));
      else if (resultArg->passBy == PassBy::BaseAddress)
        caller.placeInput(*resultArg, fir::getBase(*allocatedResult));
      else
        fir::emitFatalError(
            loc, "only expect character scalar result to be passed by ref");
    }
  }
````
- **L481 EN**: Continues the surrounding expression or declaration: `mlir::Value temp =`.
  **L481 CN**: 继续构造周围的表达式或声明：`mlir::Value temp =`。
- **L482 EN**: Executes a call or declaration centered on `builder.createTemporary`.
  **L482 CN**: 执行以 `builder.createTemporary` 为核心的调用或声明。
- **L483 EN**: Returns from the current function with `toExtendedValue(loc, temp, extents, lengths)`.
  **L483 CN**: 以 `toExtendedValue(loc, temp, extents, lengths)` 从当前函数返回。
- **L484 EN**: Executes a call or declaration centered on `}`.
  **L484 CN**: 执行以 `}` 为核心的调用或声明。
- **L485 EN**: Blank line separating nearby declarations or logic blocks.
  **L485 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L486 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L486 CN**: 开始 `if` 控制流语句并计算其条件。
- **L487 EN**: Executes a call or declaration centered on `symMap.popScope`.
  **L487 CN**: 执行以 `symMap.popScope` 为核心的调用或声明。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L489 EN**: Comment explains nearby logic, intent, or metadata: `Place allocated result`.
  **L489 CN**: 注释说明附近代码的逻辑、意图或元数据：`Place allocated result`。
- **L490 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L490 CN**: 开始 `if` 控制流语句并计算其条件。
- **L491 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L491 CN**: 开始 `if` 控制流语句并计算其条件。
- **L492 EN**: Continues the surrounding expression or declaration: `Fortran::lower::CallerInterface>::PassedEntity>`.
  **L492 CN**: 继续构造周围的表达式或声明：`Fortran::lower::CallerInterface>::PassedEntity>`。
- **L493 EN**: Starts a function, method, lambda, or structured scope: `resultArg = caller.getPassedResult()) {`.
  **L493 CN**: 开始一个函数、方法、lambda 或结构化作用域：`resultArg = caller.getPassedResult()) {`。
- **L494 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L494 CN**: 开始 `if` 控制流语句并计算其条件。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `caller.placeAddressAndLengthInput(*resultArg,`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`caller.placeAddressAndLengthInput(*resultArg,`。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::getBase(*allocatedResult),`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::getBase(*allocatedResult),`。
- **L497 EN**: Executes a call or declaration centered on `fir::getLen`.
  **L497 CN**: 执行以 `fir::getLen` 为核心的调用或声明。
- **L498 EN**: Starts the alternative branch of the preceding conditional.
  **L498 CN**: 开始前一个条件语句的备选分支。
- **L499 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L499 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L500 EN**: Transitions from the previous branch into the alternative path.
  **L500 CN**: 从前一个分支过渡到备选路径。
- **L501 EN**: Continues logic associated with callable symbol `emitFatalError`.
  **L501 CN**: 继续与可调用符号 `emitFatalError` 相关的逻辑。
- **L502 EN**: Executes a standalone statement or declaration: `loc, "only expect character scalar result to be passed by ref");`.
  **L502 CN**: 执行一条独立语句或声明：`loc, "only expect character scalar result to be passed by ref");`。
- **L503 EN**: Closes the current lexical scope or compound statement.
  **L503 CN**: 结束当前词法作用域或复合语句块。
- **L504 EN**: Closes the current lexical scope or compound statement.
  **L504 CN**: 结束当前词法作用域或复合语句块。

### Lines 505-528

````cpp

  // In older Fortran, procedure argument types are inferred. This may lead
  // different view of what the function signature is in different locations.
  // Casts are inserted as needed below to accommodate this.

  // The mlir::func::FuncOp type prevails, unless it has a different number of
  // arguments which can happen in legal program if it was passed as a dummy
  // procedure argument earlier with no further type information.
  mlir::SymbolRefAttr funcSymbolAttr;
  bool addHostAssociations = false;
  if (!funcPointer) {
    mlir::FunctionType funcOpType = caller.getFuncOp().getFunctionType();
    mlir::SymbolRefAttr symbolAttr =
        builder.getSymbolRefAttr(caller.getMangledName());
    if (callSiteType.getNumResults() == funcOpType.getNumResults() &&
        callSiteType.getNumInputs() + 1 == funcOpType.getNumInputs() &&
        fir::anyFuncArgsHaveAttr(caller.getFuncOp(),
                                 fir::getHostAssocAttrName())) {
      // The number of arguments is off by one, and we're lowering a function
      // with host associations. Modify call to include host associations
      // argument by appending the value at the end of the operands.
      assert(funcOpType.getInput(findHostAssocTuplePos(caller.getFuncOp())) ==
             converter.hostAssocTupleValue().getType());
      addHostAssociations = true;
````
- **L505 EN**: Blank line separating nearby declarations or logic blocks.
  **L505 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L506 EN**: Comment explains nearby logic, intent, or metadata: `In older Fortran, procedure argument types are inferred. This may lead`.
  **L506 CN**: 注释说明附近代码的逻辑、意图或元数据：`In older Fortran, procedure argument types are inferred. This may lead`。
- **L507 EN**: Comment explains nearby logic, intent, or metadata: `different view of what the function signature is in different locations.`.
  **L507 CN**: 注释说明附近代码的逻辑、意图或元数据：`different view of what the function signature is in different locations.`。
- **L508 EN**: Comment explains nearby logic, intent, or metadata: `Casts are inserted as needed below to accommodate this.`.
  **L508 CN**: 注释说明附近代码的逻辑、意图或元数据：`Casts are inserted as needed below to accommodate this.`。
- **L509 EN**: Blank line separating nearby declarations or logic blocks.
  **L509 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L510 EN**: Comment explains nearby logic, intent, or metadata: `The mlir::func::FuncOp type prevails, unless it has a different number of`.
  **L510 CN**: 注释说明附近代码的逻辑、意图或元数据：`The mlir::func::FuncOp type prevails, unless it has a different number of`。
- **L511 EN**: Comment explains nearby logic, intent, or metadata: `arguments which can happen in legal program if it was passed as a dummy`.
  **L511 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments which can happen in legal program if it was passed as a dummy`。
- **L512 EN**: Comment explains nearby logic, intent, or metadata: `procedure argument earlier with no further type information.`.
  **L512 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure argument earlier with no further type information.`。
- **L513 EN**: Executes a standalone statement or declaration: `mlir::SymbolRefAttr funcSymbolAttr;`.
  **L513 CN**: 执行一条独立语句或声明：`mlir::SymbolRefAttr funcSymbolAttr;`。
- **L514 EN**: Initializes variable `addHostAssociations` from the right-hand expression.
  **L514 CN**: 使用右侧表达式初始化变量 `addHostAssociations`。
- **L515 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L515 CN**: 开始 `if` 控制流语句并计算其条件。
- **L516 EN**: Initializes variable `funcOpType` from the right-hand expression.
  **L516 CN**: 使用右侧表达式初始化变量 `funcOpType`。
- **L517 EN**: Continues the surrounding expression or declaration: `mlir::SymbolRefAttr symbolAttr =`.
  **L517 CN**: 继续构造周围的表达式或声明：`mlir::SymbolRefAttr symbolAttr =`。
- **L518 EN**: Executes a call or declaration centered on `builder.getSymbolRefAttr`.
  **L518 CN**: 执行以 `builder.getSymbolRefAttr` 为核心的调用或声明。
- **L519 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L519 CN**: 开始 `if` 控制流语句并计算其条件。
- **L520 EN**: Continues logic associated with callable symbol `getNumInputs`.
  **L520 CN**: 继续与可调用符号 `getNumInputs` 相关的逻辑。
- **L521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::anyFuncArgsHaveAttr(caller.getFuncOp(),`.
  **L521 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::anyFuncArgsHaveAttr(caller.getFuncOp(),`。
- **L522 EN**: Starts a function, method, lambda, or structured scope: `fir::getHostAssocAttrName())) {`.
  **L522 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::getHostAssocAttrName())) {`。
- **L523 EN**: Comment explains nearby logic, intent, or metadata: `The number of arguments is off by one, and we're lowering a function`.
  **L523 CN**: 注释说明附近代码的逻辑、意图或元数据：`The number of arguments is off by one, and we're lowering a function`。
- **L524 EN**: Comment explains nearby logic, intent, or metadata: `with host associations. Modify call to include host associations`.
  **L524 CN**: 注释说明附近代码的逻辑、意图或元数据：`with host associations. Modify call to include host associations`。
- **L525 EN**: Comment explains nearby logic, intent, or metadata: `argument by appending the value at the end of the operands.`.
  **L525 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument by appending the value at the end of the operands.`。
- **L526 EN**: Checks an internal invariant in debug builds.
  **L526 CN**: 在调试构建中检查内部不变式。
- **L527 EN**: Executes a call or declaration centered on `converter.hostAssocTupleValue`.
  **L527 CN**: 执行以 `converter.hostAssocTupleValue` 为核心的调用或声明。
- **L528 EN**: Executes a standalone statement or declaration: `addHostAssociations = true;`.
  **L528 CN**: 执行一条独立语句或声明：`addHostAssociations = true;`。

### Lines 529-552

````cpp
    }
    // When this is not a call to an internal procedure (where there is a
    // mismatch due to the extra argument, but the interface is otherwise
    // explicit and safe), handle interface mismatch due to F77 implicit
    // interface "abuse" with a function address cast if needed.
    if (!addHostAssociations &&
        mustCastFuncOpToCopeWithImplicitInterfaceMismatch(
            loc, converter, callSiteType, funcOpType))
      funcPointer = fir::AddrOfOp::create(builder, loc, funcOpType, symbolAttr);
    else
      funcSymbolAttr = symbolAttr;

    // Issue a warning if the procedure name conflicts with
    // a runtime function name a call to which has been already
    // lowered (implying that the FuncOp has been created).
    // The behavior is undefined in this case.
    if (caller.getFuncOp()->hasAttrOfType<mlir::UnitAttr>(
            fir::FIROpsDialect::getFirRuntimeAttrName()))
      LLVM_DEBUG(mlir::emitWarning(
          loc,
          llvm::Twine("function name '") +
              llvm::Twine(symbolAttr.getLeafReference()) +
              llvm::Twine("' conflicts with a runtime function name used by "
                          "Flang - this may lead to undefined behavior")));
````
- **L529 EN**: Closes the current lexical scope or compound statement.
  **L529 CN**: 结束当前词法作用域或复合语句块。
- **L530 EN**: Comment explains nearby logic, intent, or metadata: `When this is not a call to an internal procedure (where there is a`.
  **L530 CN**: 注释说明附近代码的逻辑、意图或元数据：`When this is not a call to an internal procedure (where there is a`。
- **L531 EN**: Comment explains nearby logic, intent, or metadata: `mismatch due to the extra argument, but the interface is otherwise`.
  **L531 CN**: 注释说明附近代码的逻辑、意图或元数据：`mismatch due to the extra argument, but the interface is otherwise`。
- **L532 EN**: Comment explains nearby logic, intent, or metadata: `explicit and safe), handle interface mismatch due to F77 implicit`.
  **L532 CN**: 注释说明附近代码的逻辑、意图或元数据：`explicit and safe), handle interface mismatch due to F77 implicit`。
- **L533 EN**: Comment explains nearby logic, intent, or metadata: `interface "abuse" with a function address cast if needed.`.
  **L533 CN**: 注释说明附近代码的逻辑、意图或元数据：`interface "abuse" with a function address cast if needed.`。
- **L534 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L534 CN**: 开始 `if` 控制流语句并计算其条件。
- **L535 EN**: Continues logic associated with callable symbol `mustCastFuncOpToCopeWithImplicitInterfaceMismatch`.
  **L535 CN**: 继续与可调用符号 `mustCastFuncOpToCopeWithImplicitInterfaceMismatch` 相关的逻辑。
- **L536 EN**: Continues the surrounding expression or declaration: `loc, converter, callSiteType, funcOpType))`.
  **L536 CN**: 继续构造周围的表达式或声明：`loc, converter, callSiteType, funcOpType))`。
- **L537 EN**: Executes a call or declaration centered on `fir::AddrOfOp::create`.
  **L537 CN**: 执行以 `fir::AddrOfOp::create` 为核心的调用或声明。
- **L538 EN**: Transitions from the previous branch into the alternative path.
  **L538 CN**: 从前一个分支过渡到备选路径。
- **L539 EN**: Executes a standalone statement or declaration: `funcSymbolAttr = symbolAttr;`.
  **L539 CN**: 执行一条独立语句或声明：`funcSymbolAttr = symbolAttr;`。
- **L540 EN**: Blank line separating nearby declarations or logic blocks.
  **L540 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L541 EN**: Comment explains nearby logic, intent, or metadata: `Issue a warning if the procedure name conflicts with`.
  **L541 CN**: 注释说明附近代码的逻辑、意图或元数据：`Issue a warning if the procedure name conflicts with`。
- **L542 EN**: Comment explains nearby logic, intent, or metadata: `a runtime function name a call to which has been already`.
  **L542 CN**: 注释说明附近代码的逻辑、意图或元数据：`a runtime function name a call to which has been already`。
- **L543 EN**: Comment explains nearby logic, intent, or metadata: `lowered (implying that the FuncOp has been created).`.
  **L543 CN**: 注释说明附近代码的逻辑、意图或元数据：`lowered (implying that the FuncOp has been created).`。
- **L544 EN**: Comment explains nearby logic, intent, or metadata: `The behavior is undefined in this case.`.
  **L544 CN**: 注释说明附近代码的逻辑、意图或元数据：`The behavior is undefined in this case.`。
- **L545 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L545 CN**: 开始 `if` 控制流语句并计算其条件。
- **L546 EN**: Continues logic associated with callable symbol `getFirRuntimeAttrName`.
  **L546 CN**: 继续与可调用符号 `getFirRuntimeAttrName` 相关的逻辑。
- **L547 EN**: Continues logic associated with callable symbol `LLVM_DEBUG`.
  **L547 CN**: 继续与可调用符号 `LLVM_DEBUG` 相关的逻辑。
- **L548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc,`.
  **L548 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc,`。
- **L549 EN**: Continues logic associated with callable symbol `Twine`.
  **L549 CN**: 继续与可调用符号 `Twine` 相关的逻辑。
- **L550 EN**: Continues logic associated with callable symbol `Twine`.
  **L550 CN**: 继续与可调用符号 `Twine` 相关的逻辑。
- **L551 EN**: Continues logic associated with callable symbol `Twine`.
  **L551 CN**: 继续与可调用符号 `Twine` 相关的逻辑。
- **L552 EN**: Executes a standalone statement or declaration: `"Flang - this may lead to undefined behavior")));`.
  **L552 CN**: 执行一条独立语句或声明：`"Flang - this may lead to undefined behavior")));`。

### Lines 553-576

````cpp
  }

  mlir::FunctionType funcType =
      funcPointer ? callSiteType : caller.getFuncOp().getFunctionType();

  // If we have any ignore_tkr(c) dummy args, adjust the function type to
  // have these args match the caller.
  if (auto modifiedFuncType =
          getTypeWithIgnoreTkrC(funcType, caller, builder.getContext())) {
    // Note: funcPointer would only be non-null here, if we are already
    // processing indirect function call. In such case we can re-use the same
    // funcPointer and we'll cast it below the the modified funcType.
    if (!funcPointer) {
      // We want to cast the function to a different type, in order to avoid
      // changing/casting some of the args. The cast will generate a new
      // function pointer, so that we would make a function call not through
      // the original function symbol, but through the new function pointer
      // (an indirect function call).
      mlir::SymbolRefAttr symbolAttr =
          builder.getSymbolRefAttr(caller.getMangledName());
      // Create pointer to original function. This pointer will be cast later.
      funcPointer = fir::AddrOfOp::create(builder, loc, funcType, symbolAttr);
      funcSymbolAttr = {}; // This marks it as indirect call
    }
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L555 EN**: Continues the surrounding expression or declaration: `mlir::FunctionType funcType =`.
  **L555 CN**: 继续构造周围的表达式或声明：`mlir::FunctionType funcType =`。
- **L556 EN**: Executes a call or declaration centered on `caller.getFuncOp`.
  **L556 CN**: 执行以 `caller.getFuncOp` 为核心的调用或声明。
- **L557 EN**: Blank line separating nearby declarations or logic blocks.
  **L557 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L558 EN**: Comment explains nearby logic, intent, or metadata: `If we have any ignore_tkr(c) dummy args, adjust the function type to`.
  **L558 CN**: 注释说明附近代码的逻辑、意图或元数据：`If we have any ignore_tkr(c) dummy args, adjust the function type to`。
- **L559 EN**: Comment explains nearby logic, intent, or metadata: `have these args match the caller.`.
  **L559 CN**: 注释说明附近代码的逻辑、意图或元数据：`have these args match the caller.`。
- **L560 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L560 CN**: 开始 `if` 控制流语句并计算其条件。
- **L561 EN**: Starts a function, method, lambda, or structured scope: `getTypeWithIgnoreTkrC(funcType, caller, builder.getContext())) {`.
  **L561 CN**: 开始一个函数、方法、lambda 或结构化作用域：`getTypeWithIgnoreTkrC(funcType, caller, builder.getContext())) {`。
- **L562 EN**: Comment explains nearby logic, intent, or metadata: `Note: funcPointer would only be non-null here, if we are already`.
  **L562 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note: funcPointer would only be non-null here, if we are already`。
- **L563 EN**: Comment explains nearby logic, intent, or metadata: `processing indirect function call. In such case we can re-use the same`.
  **L563 CN**: 注释说明附近代码的逻辑、意图或元数据：`processing indirect function call. In such case we can re-use the same`。
- **L564 EN**: Comment explains nearby logic, intent, or metadata: `funcPointer and we'll cast it below the the modified funcType.`.
  **L564 CN**: 注释说明附近代码的逻辑、意图或元数据：`funcPointer and we'll cast it below the the modified funcType.`。
- **L565 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L565 CN**: 开始 `if` 控制流语句并计算其条件。
- **L566 EN**: Comment explains nearby logic, intent, or metadata: `We want to cast the function to a different type, in order to avoid`.
  **L566 CN**: 注释说明附近代码的逻辑、意图或元数据：`We want to cast the function to a different type, in order to avoid`。
- **L567 EN**: Comment explains nearby logic, intent, or metadata: `changing/casting some of the args. The cast will generate a new`.
  **L567 CN**: 注释说明附近代码的逻辑、意图或元数据：`changing/casting some of the args. The cast will generate a new`。
- **L568 EN**: Comment explains nearby logic, intent, or metadata: `function pointer, so that we would make a function call not through`.
  **L568 CN**: 注释说明附近代码的逻辑、意图或元数据：`function pointer, so that we would make a function call not through`。
- **L569 EN**: Comment explains nearby logic, intent, or metadata: `the original function symbol, but through the new function pointer`.
  **L569 CN**: 注释说明附近代码的逻辑、意图或元数据：`the original function symbol, but through the new function pointer`。
- **L570 EN**: Comment explains nearby logic, intent, or metadata: `(an indirect function call).`.
  **L570 CN**: 注释说明附近代码的逻辑、意图或元数据：`(an indirect function call).`。
- **L571 EN**: Continues the surrounding expression or declaration: `mlir::SymbolRefAttr symbolAttr =`.
  **L571 CN**: 继续构造周围的表达式或声明：`mlir::SymbolRefAttr symbolAttr =`。
- **L572 EN**: Executes a call or declaration centered on `builder.getSymbolRefAttr`.
  **L572 CN**: 执行以 `builder.getSymbolRefAttr` 为核心的调用或声明。
- **L573 EN**: Comment explains nearby logic, intent, or metadata: `Create pointer to original function. This pointer will be cast later.`.
  **L573 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create pointer to original function. This pointer will be cast later.`。
- **L574 EN**: Executes a call or declaration centered on `fir::AddrOfOp::create`.
  **L574 CN**: 执行以 `fir::AddrOfOp::create` 为核心的调用或声明。
- **L575 EN**: Continues the surrounding expression or declaration: `funcSymbolAttr = {}; // This marks it as indirect call`.
  **L575 CN**: 继续构造周围的表达式或声明：`funcSymbolAttr = {}; // This marks it as indirect call`。
- **L576 EN**: Closes the current lexical scope or compound statement.
  **L576 CN**: 结束当前词法作用域或复合语句块。

### Lines 577-600

````cpp
    funcType = *modifiedFuncType;
  }

  llvm::SmallVector<mlir::Value> operands;
  // First operand of indirect call is the function pointer. Cast it to
  // required function type for the call to handle procedures that have a
  // compatible interface in Fortran, but that have different signatures in
  // FIR.
  if (funcPointer) {
    operands.push_back(
        mlir::isa<fir::BoxProcType>(funcPointer.getType())
            ? fir::BoxAddrOp::create(builder, loc, funcType, funcPointer)
            : builder.createConvert(loc, funcType, funcPointer));
  }

  // Deal with potential mismatches in arguments types. Passing an array to a
  // scalar argument should for instance be tolerated here.
  for (auto [fst, snd] : llvm::zip(caller.getInputs(), funcType.getInputs())) {
    // When passing arguments to a procedure that can be called by implicit
    // interface, allow any character actual arguments to be passed to dummy
    // arguments of any type and vice versa.
    mlir::Value cast;
    auto *context = builder.getContext();

````
- **L577 EN**: Executes a standalone statement or declaration: `funcType = *modifiedFuncType;`.
  **L577 CN**: 执行一条独立语句或声明：`funcType = *modifiedFuncType;`。
- **L578 EN**: Closes the current lexical scope or compound statement.
  **L578 CN**: 结束当前词法作用域或复合语句块。
- **L579 EN**: Blank line separating nearby declarations or logic blocks.
  **L579 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L580 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> operands;`.
  **L580 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> operands;`。
- **L581 EN**: Comment explains nearby logic, intent, or metadata: `First operand of indirect call is the function pointer. Cast it to`.
  **L581 CN**: 注释说明附近代码的逻辑、意图或元数据：`First operand of indirect call is the function pointer. Cast it to`。
- **L582 EN**: Comment explains nearby logic, intent, or metadata: `required function type for the call to handle procedures that have a`.
  **L582 CN**: 注释说明附近代码的逻辑、意图或元数据：`required function type for the call to handle procedures that have a`。
- **L583 EN**: Comment explains nearby logic, intent, or metadata: `compatible interface in Fortran, but that have different signatures in`.
  **L583 CN**: 注释说明附近代码的逻辑、意图或元数据：`compatible interface in Fortran, but that have different signatures in`。
- **L584 EN**: Comment explains nearby logic, intent, or metadata: `FIR.`.
  **L584 CN**: 注释说明附近代码的逻辑、意图或元数据：`FIR.`。
- **L585 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L585 CN**: 开始 `if` 控制流语句并计算其条件。
- **L586 EN**: Continues logic associated with callable symbol `push_back`.
  **L586 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L587 EN**: Continues logic associated with callable symbol `BoxProcType>`.
  **L587 CN**: 继续与可调用符号 `BoxProcType>` 相关的逻辑。
- **L588 EN**: Continues logic associated with callable symbol `create`.
  **L588 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L589 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L589 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L590 EN**: Closes the current lexical scope or compound statement.
  **L590 CN**: 结束当前词法作用域或复合语句块。
- **L591 EN**: Blank line separating nearby declarations or logic blocks.
  **L591 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L592 EN**: Comment explains nearby logic, intent, or metadata: `Deal with potential mismatches in arguments types. Passing an array to a`.
  **L592 CN**: 注释说明附近代码的逻辑、意图或元数据：`Deal with potential mismatches in arguments types. Passing an array to a`。
- **L593 EN**: Comment explains nearby logic, intent, or metadata: `scalar argument should for instance be tolerated here.`.
  **L593 CN**: 注释说明附近代码的逻辑、意图或元数据：`scalar argument should for instance be tolerated here.`。
- **L594 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L594 CN**: 开始 `for` 控制流语句并计算其条件。
- **L595 EN**: Comment explains nearby logic, intent, or metadata: `When passing arguments to a procedure that can be called by implicit`.
  **L595 CN**: 注释说明附近代码的逻辑、意图或元数据：`When passing arguments to a procedure that can be called by implicit`。
- **L596 EN**: Comment explains nearby logic, intent, or metadata: `interface, allow any character actual arguments to be passed to dummy`.
  **L596 CN**: 注释说明附近代码的逻辑、意图或元数据：`interface, allow any character actual arguments to be passed to dummy`。
- **L597 EN**: Comment explains nearby logic, intent, or metadata: `arguments of any type and vice versa.`.
  **L597 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments of any type and vice versa.`。
- **L598 EN**: Executes a standalone statement or declaration: `mlir::Value cast;`.
  **L598 CN**: 执行一条独立语句或声明：`mlir::Value cast;`。
- **L599 EN**: Executes a call or declaration centered on `builder.getContext`.
  **L599 CN**: 执行以 `builder.getContext` 为核心的调用或声明。
- **L600 EN**: Blank line separating nearby declarations or logic blocks.
  **L600 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 601-624

````cpp
    if (mlir::isa<fir::BoxProcType>(snd) &&
        mlir::isa<mlir::FunctionType>(fst.getType())) {
      mlir::FunctionType funcTy = mlir::FunctionType::get(context, {}, {});
      fir::BoxProcType boxProcTy = builder.getBoxProcType(funcTy);
      if (mlir::Value host = argumentHostAssocs(converter, fst)) {
        cast = fir::EmboxProcOp::create(builder, loc, boxProcTy,
                                        llvm::ArrayRef<mlir::Value>{fst, host});
      } else {
        cast = fir::EmboxProcOp::create(builder, loc, boxProcTy, fst);
      }
    } else {
      mlir::Type fromTy = fir::unwrapRefType(fst.getType());
      if (fir::isa_builtin_cptr_type(fromTy) &&
          Fortran::lower::isCPtrArgByValueType(snd)) {
        cast = genRecordCPtrValueArg(builder, loc, fst, fromTy);
      } else if (fir::isa_derived(snd) && !fir::isa_derived(fst.getType())) {
        // TODO: remove this TODO once the old lowering is gone.
        TODO(loc, "derived type argument passed by value");
      } else {
        // With the lowering to HLFIR, box arguments have already been built
        // according to the attributes, rank, bounds, and type they should have.
        // Do not attempt any reboxing here that could break this.
        // When dealing with a dummy character argument (fir.boxchar), the
        // effective argument might be a non-character raw pointer. This may
````
- **L601 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L601 CN**: 开始 `if` 控制流语句并计算其条件。
- **L602 EN**: Starts a function, method, lambda, or structured scope: `mlir::isa<mlir::FunctionType>(fst.getType())) {`.
  **L602 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::isa<mlir::FunctionType>(fst.getType())) {`。
- **L603 EN**: Initializes variable `funcTy` from the right-hand expression.
  **L603 CN**: 使用右侧表达式初始化变量 `funcTy`。
- **L604 EN**: Initializes variable `boxProcTy` from the right-hand expression.
  **L604 CN**: 使用右侧表达式初始化变量 `boxProcTy`。
- **L605 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L605 CN**: 开始 `if` 控制流语句并计算其条件。
- **L606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `cast = fir::EmboxProcOp::create(builder, loc, boxProcTy,`.
  **L606 CN**: 继续一个多行参数列表、初始化器或聚合项：`cast = fir::EmboxProcOp::create(builder, loc, boxProcTy,`。
- **L607 EN**: Executes a standalone statement or declaration: `llvm::ArrayRef<mlir::Value>{fst, host});`.
  **L607 CN**: 执行一条独立语句或声明：`llvm::ArrayRef<mlir::Value>{fst, host});`。
- **L608 EN**: Transitions from the previous branch into the alternative path.
  **L608 CN**: 从前一个分支过渡到备选路径。
- **L609 EN**: Executes a call or declaration centered on `fir::EmboxProcOp::create`.
  **L609 CN**: 执行以 `fir::EmboxProcOp::create` 为核心的调用或声明。
- **L610 EN**: Closes the current lexical scope or compound statement.
  **L610 CN**: 结束当前词法作用域或复合语句块。
- **L611 EN**: Transitions from the previous branch into the alternative path.
  **L611 CN**: 从前一个分支过渡到备选路径。
- **L612 EN**: Initializes variable `fromTy` from the right-hand expression.
  **L612 CN**: 使用右侧表达式初始化变量 `fromTy`。
- **L613 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L613 CN**: 开始 `if` 控制流语句并计算其条件。
- **L614 EN**: Starts a function, method, lambda, or structured scope: `Fortran::lower::isCPtrArgByValueType(snd)) {`.
  **L614 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Fortran::lower::isCPtrArgByValueType(snd)) {`。
- **L615 EN**: Executes a call or declaration centered on `genRecordCPtrValueArg`.
  **L615 CN**: 执行以 `genRecordCPtrValueArg` 为核心的调用或声明。
- **L616 EN**: Transitions from the previous branch into an `else if` condition.
  **L616 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L617 EN**: Comment records a pending task or caution: `TODO: remove this TODO once the old lowering is gone.`.
  **L617 CN**: 注释记录待办事项或注意点：`TODO: remove this TODO once the old lowering is gone.`。
- **L618 EN**: Executes a call or declaration centered on `TODO`.
  **L618 CN**: 执行以 `TODO` 为核心的调用或声明。
- **L619 EN**: Transitions from the previous branch into the alternative path.
  **L619 CN**: 从前一个分支过渡到备选路径。
- **L620 EN**: Comment explains nearby logic, intent, or metadata: `With the lowering to HLFIR, box arguments have already been built`.
  **L620 CN**: 注释说明附近代码的逻辑、意图或元数据：`With the lowering to HLFIR, box arguments have already been built`。
- **L621 EN**: Comment explains nearby logic, intent, or metadata: `according to the attributes, rank, bounds, and type they should have.`.
  **L621 CN**: 注释说明附近代码的逻辑、意图或元数据：`according to the attributes, rank, bounds, and type they should have.`。
- **L622 EN**: Comment explains nearby logic, intent, or metadata: `Do not attempt any reboxing here that could break this.`.
  **L622 CN**: 注释说明附近代码的逻辑、意图或元数据：`Do not attempt any reboxing here that could break this.`。
- **L623 EN**: Comment explains nearby logic, intent, or metadata: `When dealing with a dummy character argument (fir.boxchar), the`.
  **L623 CN**: 注释说明附近代码的逻辑、意图或元数据：`When dealing with a dummy character argument (fir.boxchar), the`。
- **L624 EN**: Comment explains nearby logic, intent, or metadata: `effective argument might be a non-character raw pointer. This may`.
  **L624 CN**: 注释说明附近代码的逻辑、意图或元数据：`effective argument might be a non-character raw pointer. This may`。

### Lines 625-648

````cpp
        // happen when calling an implicit interface that was previously called
        // with a character argument, or when calling an explicit interface with
        // an IgnoreTKR dummy character arguments. Allow creating a fir.boxchar
        // from the raw pointer, which requires a non-trivial type conversion.
        const bool allowCharacterConversions = true;
        bool isVolatile = fir::isa_volatile_type(snd);
        cast = builder.createVolatileCast(loc, isVolatile, fst);
        cast = builder.convertWithSemantics(loc, snd, cast,
                                            allowCharacterConversions,
                                            /*allowRebox=*/false);
      }
    }
    operands.push_back(cast);
  }

  // Add host associations as necessary.
  if (addHostAssociations)
    operands.push_back(converter.hostAssocTupleValue());

  mlir::Value callResult;
  unsigned callNumResults;
  fir::FortranProcedureFlagsEnumAttr procAttrs =
      caller.getProcedureAttrs(builder.getContext());

````
- **L625 EN**: Comment explains nearby logic, intent, or metadata: `happen when calling an implicit interface that was previously called`.
  **L625 CN**: 注释说明附近代码的逻辑、意图或元数据：`happen when calling an implicit interface that was previously called`。
- **L626 EN**: Comment explains nearby logic, intent, or metadata: `with a character argument, or when calling an explicit interface with`.
  **L626 CN**: 注释说明附近代码的逻辑、意图或元数据：`with a character argument, or when calling an explicit interface with`。
- **L627 EN**: Comment explains nearby logic, intent, or metadata: `an IgnoreTKR dummy character arguments. Allow creating a fir.boxchar`.
  **L627 CN**: 注释说明附近代码的逻辑、意图或元数据：`an IgnoreTKR dummy character arguments. Allow creating a fir.boxchar`。
- **L628 EN**: Comment explains nearby logic, intent, or metadata: `from the raw pointer, which requires a non-trivial type conversion.`.
  **L628 CN**: 注释说明附近代码的逻辑、意图或元数据：`from the raw pointer, which requires a non-trivial type conversion.`。
- **L629 EN**: Initializes variable `allowCharacterConversions` from the right-hand expression.
  **L629 CN**: 使用右侧表达式初始化变量 `allowCharacterConversions`。
- **L630 EN**: Initializes variable `isVolatile` from the right-hand expression.
  **L630 CN**: 使用右侧表达式初始化变量 `isVolatile`。
- **L631 EN**: Executes a call or declaration centered on `builder.createVolatileCast`.
  **L631 CN**: 执行以 `builder.createVolatileCast` 为核心的调用或声明。
- **L632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `cast = builder.convertWithSemantics(loc, snd, cast,`.
  **L632 CN**: 继续一个多行参数列表、初始化器或聚合项：`cast = builder.convertWithSemantics(loc, snd, cast,`。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `allowCharacterConversions,`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`allowCharacterConversions,`。
- **L634 EN**: Comment explains nearby logic, intent, or metadata: `allowRebox=*/false);`.
  **L634 CN**: 注释说明附近代码的逻辑、意图或元数据：`allowRebox=*/false);`。
- **L635 EN**: Closes the current lexical scope or compound statement.
  **L635 CN**: 结束当前词法作用域或复合语句块。
- **L636 EN**: Closes the current lexical scope or compound statement.
  **L636 CN**: 结束当前词法作用域或复合语句块。
- **L637 EN**: Executes a call or declaration centered on `operands.push_back`.
  **L637 CN**: 执行以 `operands.push_back` 为核心的调用或声明。
- **L638 EN**: Closes the current lexical scope or compound statement.
  **L638 CN**: 结束当前词法作用域或复合语句块。
- **L639 EN**: Blank line separating nearby declarations or logic blocks.
  **L639 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L640 EN**: Comment explains nearby logic, intent, or metadata: `Add host associations as necessary.`.
  **L640 CN**: 注释说明附近代码的逻辑、意图或元数据：`Add host associations as necessary.`。
- **L641 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L641 CN**: 开始 `if` 控制流语句并计算其条件。
- **L642 EN**: Executes a call or declaration centered on `operands.push_back`.
  **L642 CN**: 执行以 `operands.push_back` 为核心的调用或声明。
- **L643 EN**: Blank line separating nearby declarations or logic blocks.
  **L643 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L644 EN**: Executes a standalone statement or declaration: `mlir::Value callResult;`.
  **L644 CN**: 执行一条独立语句或声明：`mlir::Value callResult;`。
- **L645 EN**: Executes a standalone statement or declaration: `unsigned callNumResults;`.
  **L645 CN**: 执行一条独立语句或声明：`unsigned callNumResults;`。
- **L646 EN**: Continues the surrounding expression or declaration: `fir::FortranProcedureFlagsEnumAttr procAttrs =`.
  **L646 CN**: 继续构造周围的表达式或声明：`fir::FortranProcedureFlagsEnumAttr procAttrs =`。
- **L647 EN**: Executes a call or declaration centered on `caller.getProcedureAttrs`.
  **L647 CN**: 执行以 `caller.getProcedureAttrs` 为核心的调用或声明。
- **L648 EN**: Blank line separating nearby declarations or logic blocks.
  **L648 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 649-672

````cpp
  if (converter.getLoweringOptions().getCUDARuntimeCheck()) {
    if (caller.getCallDescription().chevrons().empty() &&
        !cuf::isCUDADeviceContext(builder.getRegion())) {
      for (auto [oper, arg] :
           llvm::zip(operands, caller.getPassedArguments())) {
        if (arg.testTKR(Fortran::common::IgnoreTKR::Contiguous))
          continue;
        if (auto boxTy = mlir::dyn_cast<fir::BaseBoxType>(oper.getType())) {
          const Fortran::semantics::Symbol *sym = caller.getDummySymbol(arg);
          if (sym && Fortran::evaluate::IsCUDADeviceSymbol(*sym))
            fir::runtime::cuda::genDescriptorCheckSection(builder, loc, oper);
        }
      }
    }
  }

  if (!caller.getCallDescription().chevrons().empty()) {
    // A call to a CUDA kernel with the chevron syntax.

    mlir::Type i32Ty = builder.getI32Type();
    mlir::Value one = builder.createIntegerConstant(loc, i32Ty, 1);

    mlir::Value grid_x, grid_y, grid_z;
    if (caller.getCallDescription().chevrons()[0].GetType()->category() ==
````
- **L649 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L649 CN**: 开始 `if` 控制流语句并计算其条件。
- **L650 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L650 CN**: 开始 `if` 控制流语句并计算其条件。
- **L651 EN**: Starts a function, method, lambda, or structured scope: `!cuf::isCUDADeviceContext(builder.getRegion())) {`.
  **L651 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!cuf::isCUDADeviceContext(builder.getRegion())) {`。
- **L652 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L652 CN**: 开始 `for` 控制流语句并计算其条件。
- **L653 EN**: Starts a function, method, lambda, or structured scope: `llvm::zip(operands, caller.getPassedArguments())) {`.
  **L653 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::zip(operands, caller.getPassedArguments())) {`。
- **L654 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L654 CN**: 开始 `if` 控制流语句并计算其条件。
- **L655 EN**: Skips to the next loop iteration.
  **L655 CN**: 跳到下一次循环迭代。
- **L656 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L656 CN**: 开始 `if` 控制流语句并计算其条件。
- **L657 EN**: Executes a call or declaration centered on `caller.getDummySymbol`.
  **L657 CN**: 执行以 `caller.getDummySymbol` 为核心的调用或声明。
- **L658 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L658 CN**: 开始 `if` 控制流语句并计算其条件。
- **L659 EN**: Executes a call or declaration centered on `fir::runtime::cuda::genDescriptorCheckSection`.
  **L659 CN**: 执行以 `fir::runtime::cuda::genDescriptorCheckSection` 为核心的调用或声明。
- **L660 EN**: Closes the current lexical scope or compound statement.
  **L660 CN**: 结束当前词法作用域或复合语句块。
- **L661 EN**: Closes the current lexical scope or compound statement.
  **L661 CN**: 结束当前词法作用域或复合语句块。
- **L662 EN**: Closes the current lexical scope or compound statement.
  **L662 CN**: 结束当前词法作用域或复合语句块。
- **L663 EN**: Closes the current lexical scope or compound statement.
  **L663 CN**: 结束当前词法作用域或复合语句块。
- **L664 EN**: Blank line separating nearby declarations or logic blocks.
  **L664 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L665 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L665 CN**: 开始 `if` 控制流语句并计算其条件。
- **L666 EN**: Comment explains nearby logic, intent, or metadata: `A call to a CUDA kernel with the chevron syntax.`.
  **L666 CN**: 注释说明附近代码的逻辑、意图或元数据：`A call to a CUDA kernel with the chevron syntax.`。
- **L667 EN**: Blank line separating nearby declarations or logic blocks.
  **L667 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L668 EN**: Initializes variable `i32Ty` from the right-hand expression.
  **L668 CN**: 使用右侧表达式初始化变量 `i32Ty`。
- **L669 EN**: Initializes variable `one` from the right-hand expression.
  **L669 CN**: 使用右侧表达式初始化变量 `one`。
- **L670 EN**: Blank line separating nearby declarations or logic blocks.
  **L670 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L671 EN**: Executes a standalone statement or declaration: `mlir::Value grid_x, grid_y, grid_z;`.
  **L671 CN**: 执行一条独立语句或声明：`mlir::Value grid_x, grid_y, grid_z;`。
- **L672 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L672 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 673-696

````cpp
        Fortran::common::TypeCategory::Integer) {
      // If grid is an integer, it is converted to dim3(grid,1,1). Since z is
      // not used for the number of thread blocks, it is omitted in the op.
      grid_x = builder.createConvert(
          loc, i32Ty,
          fir::getBase(converter.genExprValue(
              caller.getCallDescription().chevrons()[0], stmtCtx)));
      grid_y = one;
      grid_z = one;
    } else {
      auto dim3Addr = converter.genExprAddr(
          caller.getCallDescription().chevrons()[0], stmtCtx);
      grid_x = readDim3Value(builder, loc, fir::getBase(dim3Addr), "x");
      grid_y = readDim3Value(builder, loc, fir::getBase(dim3Addr), "y");
      grid_z = readDim3Value(builder, loc, fir::getBase(dim3Addr), "z");
    }

    mlir::Value block_x, block_y, block_z;
    if (caller.getCallDescription().chevrons()[1].GetType()->category() ==
        Fortran::common::TypeCategory::Integer) {
      // If block is an integer, it is converted to dim3(block,1,1).
      block_x = builder.createConvert(
          loc, i32Ty,
          fir::getBase(converter.genExprValue(
````
- **L673 EN**: Continues the surrounding expression or declaration: `Fortran::common::TypeCategory::Integer) {`.
  **L673 CN**: 继续构造周围的表达式或声明：`Fortran::common::TypeCategory::Integer) {`。
- **L674 EN**: Comment explains nearby logic, intent, or metadata: `If grid is an integer, it is converted to dim3(grid,1,1). Since z is`.
  **L674 CN**: 注释说明附近代码的逻辑、意图或元数据：`If grid is an integer, it is converted to dim3(grid,1,1). Since z is`。
- **L675 EN**: Comment explains nearby logic, intent, or metadata: `not used for the number of thread blocks, it is omitted in the op.`.
  **L675 CN**: 注释说明附近代码的逻辑、意图或元数据：`not used for the number of thread blocks, it is omitted in the op.`。
- **L676 EN**: Continues logic associated with callable symbol `createConvert`.
  **L676 CN**: 继续与可调用符号 `createConvert` 相关的逻辑。
- **L677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, i32Ty,`.
  **L677 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, i32Ty,`。
- **L678 EN**: Continues logic associated with callable symbol `getBase`.
  **L678 CN**: 继续与可调用符号 `getBase` 相关的逻辑。
- **L679 EN**: Executes a call or declaration centered on `caller.getCallDescription`.
  **L679 CN**: 执行以 `caller.getCallDescription` 为核心的调用或声明。
- **L680 EN**: Executes a standalone statement or declaration: `grid_y = one;`.
  **L680 CN**: 执行一条独立语句或声明：`grid_y = one;`。
- **L681 EN**: Executes a standalone statement or declaration: `grid_z = one;`.
  **L681 CN**: 执行一条独立语句或声明：`grid_z = one;`。
- **L682 EN**: Transitions from the previous branch into the alternative path.
  **L682 CN**: 从前一个分支过渡到备选路径。
- **L683 EN**: Continues logic associated with callable symbol `genExprAddr`.
  **L683 CN**: 继续与可调用符号 `genExprAddr` 相关的逻辑。
- **L684 EN**: Executes a call or declaration centered on `caller.getCallDescription`.
  **L684 CN**: 执行以 `caller.getCallDescription` 为核心的调用或声明。
- **L685 EN**: Executes a call or declaration centered on `readDim3Value`.
  **L685 CN**: 执行以 `readDim3Value` 为核心的调用或声明。
- **L686 EN**: Executes a call or declaration centered on `readDim3Value`.
  **L686 CN**: 执行以 `readDim3Value` 为核心的调用或声明。
- **L687 EN**: Executes a call or declaration centered on `readDim3Value`.
  **L687 CN**: 执行以 `readDim3Value` 为核心的调用或声明。
- **L688 EN**: Closes the current lexical scope or compound statement.
  **L688 CN**: 结束当前词法作用域或复合语句块。
- **L689 EN**: Blank line separating nearby declarations or logic blocks.
  **L689 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L690 EN**: Executes a standalone statement or declaration: `mlir::Value block_x, block_y, block_z;`.
  **L690 CN**: 执行一条独立语句或声明：`mlir::Value block_x, block_y, block_z;`。
- **L691 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L691 CN**: 开始 `if` 控制流语句并计算其条件。
- **L692 EN**: Continues the surrounding expression or declaration: `Fortran::common::TypeCategory::Integer) {`.
  **L692 CN**: 继续构造周围的表达式或声明：`Fortran::common::TypeCategory::Integer) {`。
- **L693 EN**: Comment explains nearby logic, intent, or metadata: `If block is an integer, it is converted to dim3(block,1,1).`.
  **L693 CN**: 注释说明附近代码的逻辑、意图或元数据：`If block is an integer, it is converted to dim3(block,1,1).`。
- **L694 EN**: Continues logic associated with callable symbol `createConvert`.
  **L694 CN**: 继续与可调用符号 `createConvert` 相关的逻辑。
- **L695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, i32Ty,`.
  **L695 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, i32Ty,`。
- **L696 EN**: Continues logic associated with callable symbol `getBase`.
  **L696 CN**: 继续与可调用符号 `getBase` 相关的逻辑。

### Lines 697-720

````cpp
              caller.getCallDescription().chevrons()[1], stmtCtx)));
      block_y = one;
      block_z = one;
    } else {
      auto dim3Addr = converter.genExprAddr(
          caller.getCallDescription().chevrons()[1], stmtCtx);
      block_x = readDim3Value(builder, loc, fir::getBase(dim3Addr), "x");
      block_y = readDim3Value(builder, loc, fir::getBase(dim3Addr), "y");
      block_z = readDim3Value(builder, loc, fir::getBase(dim3Addr), "z");
    }

    mlir::Value bytes; // bytes is optional.
    if (caller.getCallDescription().chevrons().size() > 2)
      bytes = builder.createConvert(
          loc, i32Ty,
          fir::getBase(converter.genExprValue(
              caller.getCallDescription().chevrons()[2], stmtCtx)));

    mlir::Value stream; // stream is optional.
    if (caller.getCallDescription().chevrons().size() > 3) {
      stream = fir::getBase(converter.genExprAddr(
          caller.getCallDescription().chevrons()[3], stmtCtx));
      if (!fir::unwrapRefType(stream.getType()).isInteger(64)) {
        auto i64Ty = mlir::IntegerType::get(builder.getContext(), 64);
````
- **L697 EN**: Executes a call or declaration centered on `caller.getCallDescription`.
  **L697 CN**: 执行以 `caller.getCallDescription` 为核心的调用或声明。
- **L698 EN**: Executes a standalone statement or declaration: `block_y = one;`.
  **L698 CN**: 执行一条独立语句或声明：`block_y = one;`。
- **L699 EN**: Executes a standalone statement or declaration: `block_z = one;`.
  **L699 CN**: 执行一条独立语句或声明：`block_z = one;`。
- **L700 EN**: Transitions from the previous branch into the alternative path.
  **L700 CN**: 从前一个分支过渡到备选路径。
- **L701 EN**: Continues logic associated with callable symbol `genExprAddr`.
  **L701 CN**: 继续与可调用符号 `genExprAddr` 相关的逻辑。
- **L702 EN**: Executes a call or declaration centered on `caller.getCallDescription`.
  **L702 CN**: 执行以 `caller.getCallDescription` 为核心的调用或声明。
- **L703 EN**: Executes a call or declaration centered on `readDim3Value`.
  **L703 CN**: 执行以 `readDim3Value` 为核心的调用或声明。
- **L704 EN**: Executes a call or declaration centered on `readDim3Value`.
  **L704 CN**: 执行以 `readDim3Value` 为核心的调用或声明。
- **L705 EN**: Executes a call or declaration centered on `readDim3Value`.
  **L705 CN**: 执行以 `readDim3Value` 为核心的调用或声明。
- **L706 EN**: Closes the current lexical scope or compound statement.
  **L706 CN**: 结束当前词法作用域或复合语句块。
- **L707 EN**: Blank line separating nearby declarations or logic blocks.
  **L707 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L708 EN**: Continues the surrounding expression or declaration: `mlir::Value bytes; // bytes is optional.`.
  **L708 CN**: 继续构造周围的表达式或声明：`mlir::Value bytes; // bytes is optional.`。
- **L709 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L709 CN**: 开始 `if` 控制流语句并计算其条件。
- **L710 EN**: Continues logic associated with callable symbol `createConvert`.
  **L710 CN**: 继续与可调用符号 `createConvert` 相关的逻辑。
- **L711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, i32Ty,`.
  **L711 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, i32Ty,`。
- **L712 EN**: Continues logic associated with callable symbol `getBase`.
  **L712 CN**: 继续与可调用符号 `getBase` 相关的逻辑。
- **L713 EN**: Executes a call or declaration centered on `caller.getCallDescription`.
  **L713 CN**: 执行以 `caller.getCallDescription` 为核心的调用或声明。
- **L714 EN**: Blank line separating nearby declarations or logic blocks.
  **L714 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L715 EN**: Continues the surrounding expression or declaration: `mlir::Value stream; // stream is optional.`.
  **L715 CN**: 继续构造周围的表达式或声明：`mlir::Value stream; // stream is optional.`。
- **L716 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L716 CN**: 开始 `if` 控制流语句并计算其条件。
- **L717 EN**: Continues logic associated with callable symbol `getBase`.
  **L717 CN**: 继续与可调用符号 `getBase` 相关的逻辑。
- **L718 EN**: Executes a call or declaration centered on `caller.getCallDescription`.
  **L718 CN**: 执行以 `caller.getCallDescription` 为核心的调用或声明。
- **L719 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L719 CN**: 开始 `if` 控制流语句并计算其条件。
- **L720 EN**: Initializes variable `i64Ty` from the right-hand expression.
  **L720 CN**: 使用右侧表达式初始化变量 `i64Ty`。

### Lines 721-744

````cpp
        mlir::Value newStream = builder.createTemporary(loc, i64Ty);
        mlir::Value load = fir::LoadOp::create(builder, loc, stream);
        mlir::Value conv = fir::ConvertOp::create(builder, loc, i64Ty, load);
        fir::StoreOp::create(builder, loc, conv, newStream);
        stream = newStream;
      }
    }

    callOp = cuf::KernelLaunchOp::create(
        builder, loc, funcType.getResults(), funcSymbolAttr, grid_x, grid_y,
        grid_z, block_x, block_y, block_z, bytes, stream, operands,
        /*arg_attrs=*/nullptr, /*res_attrs=*/nullptr);
    callNumResults = 0;
  } else if (caller.requireDispatchCall()) {
    // Procedure call requiring a dynamic dispatch. Call is created with
    // fir.dispatch.

    // Get the raw procedure name. The procedure name is not mangled in the
    // binding table, but there can be a suffix to distinguish bindings of
    // the same name (which happens only when PRIVATE bindings exist in
    // ancestor types in other modules).
    const auto &ultimateSymbol =
        caller.getCallDescription().proc().GetSymbol()->GetUltimate();
    std::string procName = ultimateSymbol.name().ToString();
````
- **L721 EN**: Initializes variable `newStream` from the right-hand expression.
  **L721 CN**: 使用右侧表达式初始化变量 `newStream`。
- **L722 EN**: Initializes variable `load` from the right-hand expression.
  **L722 CN**: 使用右侧表达式初始化变量 `load`。
- **L723 EN**: Initializes variable `conv` from the right-hand expression.
  **L723 CN**: 使用右侧表达式初始化变量 `conv`。
- **L724 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L724 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L725 EN**: Executes a standalone statement or declaration: `stream = newStream;`.
  **L725 CN**: 执行一条独立语句或声明：`stream = newStream;`。
- **L726 EN**: Closes the current lexical scope or compound statement.
  **L726 CN**: 结束当前词法作用域或复合语句块。
- **L727 EN**: Closes the current lexical scope or compound statement.
  **L727 CN**: 结束当前词法作用域或复合语句块。
- **L728 EN**: Blank line separating nearby declarations or logic blocks.
  **L728 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L729 EN**: Continues logic associated with callable symbol `create`.
  **L729 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, funcType.getResults(), funcSymbolAttr, grid_x, grid_y,`.
  **L730 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, funcType.getResults(), funcSymbolAttr, grid_x, grid_y,`。
- **L731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `grid_z, block_x, block_y, block_z, bytes, stream, operands,`.
  **L731 CN**: 继续一个多行参数列表、初始化器或聚合项：`grid_z, block_x, block_y, block_z, bytes, stream, operands,`。
- **L732 EN**: Comment explains nearby logic, intent, or metadata: `arg_attrs=*/nullptr, /*res_attrs=*/nullptr);`.
  **L732 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg_attrs=*/nullptr, /*res_attrs=*/nullptr);`。
- **L733 EN**: Executes a standalone statement or declaration: `callNumResults = 0;`.
  **L733 CN**: 执行一条独立语句或声明：`callNumResults = 0;`。
- **L734 EN**: Transitions from the previous branch into an `else if` condition.
  **L734 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L735 EN**: Comment explains nearby logic, intent, or metadata: `Procedure call requiring a dynamic dispatch. Call is created with`.
  **L735 CN**: 注释说明附近代码的逻辑、意图或元数据：`Procedure call requiring a dynamic dispatch. Call is created with`。
- **L736 EN**: Comment explains nearby logic, intent, or metadata: `fir.dispatch.`.
  **L736 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.dispatch.`。
- **L737 EN**: Blank line separating nearby declarations or logic blocks.
  **L737 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L738 EN**: Comment explains nearby logic, intent, or metadata: `Get the raw procedure name. The procedure name is not mangled in the`.
  **L738 CN**: 注释说明附近代码的逻辑、意图或元数据：`Get the raw procedure name. The procedure name is not mangled in the`。
- **L739 EN**: Comment explains nearby logic, intent, or metadata: `binding table, but there can be a suffix to distinguish bindings of`.
  **L739 CN**: 注释说明附近代码的逻辑、意图或元数据：`binding table, but there can be a suffix to distinguish bindings of`。
- **L740 EN**: Comment explains nearby logic, intent, or metadata: `the same name (which happens only when PRIVATE bindings exist in`.
  **L740 CN**: 注释说明附近代码的逻辑、意图或元数据：`the same name (which happens only when PRIVATE bindings exist in`。
- **L741 EN**: Comment explains nearby logic, intent, or metadata: `ancestor types in other modules).`.
  **L741 CN**: 注释说明附近代码的逻辑、意图或元数据：`ancestor types in other modules).`。
- **L742 EN**: Continues the surrounding expression or declaration: `const auto &ultimateSymbol =`.
  **L742 CN**: 继续构造周围的表达式或声明：`const auto &ultimateSymbol =`。
- **L743 EN**: Executes a call or declaration centered on `caller.getCallDescription`.
  **L743 CN**: 执行以 `caller.getCallDescription` 为核心的调用或声明。
- **L744 EN**: Initializes variable `procName` from the right-hand expression.
  **L744 CN**: 使用右侧表达式初始化变量 `procName`。

### Lines 745-768

````cpp
    if (const auto &binding{
            ultimateSymbol.get<Fortran::semantics::ProcBindingDetails>()};
        binding.numPrivatesNotOverridden() > 0)
      procName += "."s + std::to_string(binding.numPrivatesNotOverridden());
    fir::DispatchOp dispatch;
    if (std::optional<unsigned> passArg = caller.getPassArgIndex()) {
      // PASS, PASS(arg-name)
      // Note that caller.getInputs is used instead of operands to get the
      // passed object because interface mismatch issues may have inserted a
      // cast to the operand with a different declared type, which would break
      // later type bound call resolution in the FIR to FIR pass.
      mlir::Value passActual = caller.getInputs()[*passArg];
      if (std::optional<mlir::Value> original = caller.getOriginalPassArg())
        passActual = *original;
      dispatch = fir::DispatchOp::create(
          builder, loc, funcType.getResults(), builder.getStringAttr(procName),
          passActual, operands, builder.getI32IntegerAttr(*passArg),
          /*arg_attrs=*/nullptr,
          /*res_attrs=*/nullptr, procAttrs);
      callOp = dispatch;
    } else {
      // NOPASS
      const Fortran::evaluate::Component *component =
          caller.getCallDescription().proc().GetComponent();
````
- **L745 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L745 CN**: 开始 `if` 控制流语句并计算其条件。
- **L746 EN**: Executes a call or declaration centered on `ultimateSymbol.get<Fortran::semantics::ProcBindingDetails>`.
  **L746 CN**: 执行以 `ultimateSymbol.get<Fortran::semantics::ProcBindingDetails>` 为核心的调用或声明。
- **L747 EN**: Continues logic associated with callable symbol `numPrivatesNotOverridden`.
  **L747 CN**: 继续与可调用符号 `numPrivatesNotOverridden` 相关的逻辑。
- **L748 EN**: Executes a call or declaration centered on `std::to_string`.
  **L748 CN**: 执行以 `std::to_string` 为核心的调用或声明。
- **L749 EN**: Executes a standalone statement or declaration: `fir::DispatchOp dispatch;`.
  **L749 CN**: 执行一条独立语句或声明：`fir::DispatchOp dispatch;`。
- **L750 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L750 CN**: 开始 `if` 控制流语句并计算其条件。
- **L751 EN**: Comment explains nearby logic, intent, or metadata: `PASS, PASS(arg-name)`.
  **L751 CN**: 注释说明附近代码的逻辑、意图或元数据：`PASS, PASS(arg-name)`。
- **L752 EN**: Comment explains nearby logic, intent, or metadata: `Note that caller.getInputs is used instead of operands to get the`.
  **L752 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note that caller.getInputs is used instead of operands to get the`。
- **L753 EN**: Comment explains nearby logic, intent, or metadata: `passed object because interface mismatch issues may have inserted a`.
  **L753 CN**: 注释说明附近代码的逻辑、意图或元数据：`passed object because interface mismatch issues may have inserted a`。
- **L754 EN**: Comment explains nearby logic, intent, or metadata: `cast to the operand with a different declared type, which would break`.
  **L754 CN**: 注释说明附近代码的逻辑、意图或元数据：`cast to the operand with a different declared type, which would break`。
- **L755 EN**: Comment explains nearby logic, intent, or metadata: `later type bound call resolution in the FIR to FIR pass.`.
  **L755 CN**: 注释说明附近代码的逻辑、意图或元数据：`later type bound call resolution in the FIR to FIR pass.`。
- **L756 EN**: Initializes variable `passActual` from the right-hand expression.
  **L756 CN**: 使用右侧表达式初始化变量 `passActual`。
- **L757 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L757 CN**: 开始 `if` 控制流语句并计算其条件。
- **L758 EN**: Executes a standalone statement or declaration: `passActual = *original;`.
  **L758 CN**: 执行一条独立语句或声明：`passActual = *original;`。
- **L759 EN**: Continues logic associated with callable symbol `create`.
  **L759 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, funcType.getResults(), builder.getStringAttr(procName),`.
  **L760 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, funcType.getResults(), builder.getStringAttr(procName),`。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `passActual, operands, builder.getI32IntegerAttr(*passArg),`.
  **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`passActual, operands, builder.getI32IntegerAttr(*passArg),`。
- **L762 EN**: Comment explains nearby logic, intent, or metadata: `arg_attrs=*/nullptr,`.
  **L762 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg_attrs=*/nullptr,`。
- **L763 EN**: Comment explains nearby logic, intent, or metadata: `res_attrs=*/nullptr, procAttrs);`.
  **L763 CN**: 注释说明附近代码的逻辑、意图或元数据：`res_attrs=*/nullptr, procAttrs);`。
- **L764 EN**: Executes a standalone statement or declaration: `callOp = dispatch;`.
  **L764 CN**: 执行一条独立语句或声明：`callOp = dispatch;`。
- **L765 EN**: Transitions from the previous branch into the alternative path.
  **L765 CN**: 从前一个分支过渡到备选路径。
- **L766 EN**: Comment explains nearby logic, intent, or metadata: `NOPASS`.
  **L766 CN**: 注释说明附近代码的逻辑、意图或元数据：`NOPASS`。
- **L767 EN**: Continues the surrounding expression or declaration: `const Fortran::evaluate::Component *component =`.
  **L767 CN**: 继续构造周围的表达式或声明：`const Fortran::evaluate::Component *component =`。
- **L768 EN**: Executes a call or declaration centered on `caller.getCallDescription`.
  **L768 CN**: 执行以 `caller.getCallDescription` 为核心的调用或声明。

### Lines 769-792

````cpp
      assert(component && "expect component for type-bound procedure call.");

      fir::ExtendedValue dataRefValue = Fortran::lower::convertDataRefToValue(
          loc, converter, component->base(), symMap, stmtCtx);
      mlir::Value passObject = fir::getBase(dataRefValue);

      if (fir::isa_ref_type(passObject.getType()))
        passObject = fir::LoadOp::create(builder, loc, passObject);
      dispatch = fir::DispatchOp::create(
          builder, loc, funcType.getResults(), builder.getStringAttr(procName),
          passObject, operands, nullptr, /*arg_attrs=*/nullptr,
          /*res_attrs=*/nullptr, procAttrs);
      callOp = dispatch;
    }
    callNumResults = dispatch.getNumResults();
    if (callNumResults != 0)
      callResult = dispatch.getResult(0);
  } else {
    // Standard procedure call with fir.call.
    fir::FortranInlineEnumAttr inlineAttr;

    if (caller.getCallDescription().hasNoInline())
      inlineAttr = fir::FortranInlineEnumAttr::get(
          builder.getContext(), fir::FortranInlineEnum::no_inline);
````
- **L769 EN**: Checks an internal invariant in debug builds.
  **L769 CN**: 在调试构建中检查内部不变式。
- **L770 EN**: Blank line separating nearby declarations or logic blocks.
  **L770 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L771 EN**: Continues logic associated with callable symbol `convertDataRefToValue`.
  **L771 CN**: 继续与可调用符号 `convertDataRefToValue` 相关的逻辑。
- **L772 EN**: Executes a call or declaration centered on `component->base`.
  **L772 CN**: 执行以 `component->base` 为核心的调用或声明。
- **L773 EN**: Initializes variable `passObject` from the right-hand expression.
  **L773 CN**: 使用右侧表达式初始化变量 `passObject`。
- **L774 EN**: Blank line separating nearby declarations or logic blocks.
  **L774 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L775 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L775 CN**: 开始 `if` 控制流语句并计算其条件。
- **L776 EN**: Executes a call or declaration centered on `fir::LoadOp::create`.
  **L776 CN**: 执行以 `fir::LoadOp::create` 为核心的调用或声明。
- **L777 EN**: Continues logic associated with callable symbol `create`.
  **L777 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, funcType.getResults(), builder.getStringAttr(procName),`.
  **L778 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, funcType.getResults(), builder.getStringAttr(procName),`。
- **L779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `passObject, operands, nullptr, /*arg_attrs=*/nullptr,`.
  **L779 CN**: 继续一个多行参数列表、初始化器或聚合项：`passObject, operands, nullptr, /*arg_attrs=*/nullptr,`。
- **L780 EN**: Comment explains nearby logic, intent, or metadata: `res_attrs=*/nullptr, procAttrs);`.
  **L780 CN**: 注释说明附近代码的逻辑、意图或元数据：`res_attrs=*/nullptr, procAttrs);`。
- **L781 EN**: Executes a standalone statement or declaration: `callOp = dispatch;`.
  **L781 CN**: 执行一条独立语句或声明：`callOp = dispatch;`。
- **L782 EN**: Closes the current lexical scope or compound statement.
  **L782 CN**: 结束当前词法作用域或复合语句块。
- **L783 EN**: Executes a call or declaration centered on `dispatch.getNumResults`.
  **L783 CN**: 执行以 `dispatch.getNumResults` 为核心的调用或声明。
- **L784 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L784 CN**: 开始 `if` 控制流语句并计算其条件。
- **L785 EN**: Executes a call or declaration centered on `dispatch.getResult`.
  **L785 CN**: 执行以 `dispatch.getResult` 为核心的调用或声明。
- **L786 EN**: Transitions from the previous branch into the alternative path.
  **L786 CN**: 从前一个分支过渡到备选路径。
- **L787 EN**: Comment explains nearby logic, intent, or metadata: `Standard procedure call with fir.call.`.
  **L787 CN**: 注释说明附近代码的逻辑、意图或元数据：`Standard procedure call with fir.call.`。
- **L788 EN**: Executes a standalone statement or declaration: `fir::FortranInlineEnumAttr inlineAttr;`.
  **L788 CN**: 执行一条独立语句或声明：`fir::FortranInlineEnumAttr inlineAttr;`。
- **L789 EN**: Blank line separating nearby declarations or logic blocks.
  **L789 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L790 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L790 CN**: 开始 `if` 控制流语句并计算其条件。
- **L791 EN**: Continues logic associated with callable symbol `get`.
  **L791 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L792 EN**: Executes a call or declaration centered on `builder.getContext`.
  **L792 CN**: 执行以 `builder.getContext` 为核心的调用或声明。

### Lines 793-816

````cpp
    else if (caller.getCallDescription().hasInlineHint())
      inlineAttr = fir::FortranInlineEnumAttr::get(
          builder.getContext(), fir::FortranInlineEnum::inline_hint);
    else if (caller.getCallDescription().hasAlwaysInline())
      inlineAttr = fir::FortranInlineEnumAttr::get(
          builder.getContext(), fir::FortranInlineEnum::always_inline);
    auto call = fir::CallOp::create(
        builder, loc, funcType.getResults(), funcSymbolAttr, operands,
        /*arg_attrs=*/nullptr, /*res_attrs=*/nullptr, procAttrs, inlineAttr,
        /*accessGroups=*/mlir::ArrayAttr{});
    callOp = call;

    callNumResults = call.getNumResults();
    if (callNumResults != 0)
      callResult = call.getResult(0);
  }

  std::optional<Fortran::evaluate::DynamicType> retTy =
      caller.getCallDescription().proc().GetType();
  // With HLFIR lowering, isElemental must be set to true
  // if we are producing an elemental call. In this case,
  // the elemental results must not be destroyed, instead,
  // the resulting array result will be finalized/destroyed
  // as needed by hlfir.destroy.
````
- **L793 EN**: Starts the alternative branch of the preceding conditional.
  **L793 CN**: 开始前一个条件语句的备选分支。
- **L794 EN**: Continues logic associated with callable symbol `get`.
  **L794 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L795 EN**: Executes a call or declaration centered on `builder.getContext`.
  **L795 CN**: 执行以 `builder.getContext` 为核心的调用或声明。
- **L796 EN**: Starts the alternative branch of the preceding conditional.
  **L796 CN**: 开始前一个条件语句的备选分支。
- **L797 EN**: Continues logic associated with callable symbol `get`.
  **L797 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L798 EN**: Executes a call or declaration centered on `builder.getContext`.
  **L798 CN**: 执行以 `builder.getContext` 为核心的调用或声明。
- **L799 EN**: Continues logic associated with callable symbol `create`.
  **L799 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, funcType.getResults(), funcSymbolAttr, operands,`.
  **L800 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, funcType.getResults(), funcSymbolAttr, operands,`。
- **L801 EN**: Comment explains nearby logic, intent, or metadata: `arg_attrs=*/nullptr, /*res_attrs=*/nullptr, procAttrs, inlineAttr,`.
  **L801 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg_attrs=*/nullptr, /*res_attrs=*/nullptr, procAttrs, inlineAttr,`。
- **L802 EN**: Comment explains nearby logic, intent, or metadata: `accessGroups=*/mlir::ArrayAttr{});`.
  **L802 CN**: 注释说明附近代码的逻辑、意图或元数据：`accessGroups=*/mlir::ArrayAttr{});`。
- **L803 EN**: Executes a standalone statement or declaration: `callOp = call;`.
  **L803 CN**: 执行一条独立语句或声明：`callOp = call;`。
- **L804 EN**: Blank line separating nearby declarations or logic blocks.
  **L804 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L805 EN**: Executes a call or declaration centered on `call.getNumResults`.
  **L805 CN**: 执行以 `call.getNumResults` 为核心的调用或声明。
- **L806 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L806 CN**: 开始 `if` 控制流语句并计算其条件。
- **L807 EN**: Executes a call or declaration centered on `call.getResult`.
  **L807 CN**: 执行以 `call.getResult` 为核心的调用或声明。
- **L808 EN**: Closes the current lexical scope or compound statement.
  **L808 CN**: 结束当前词法作用域或复合语句块。
- **L809 EN**: Blank line separating nearby declarations or logic blocks.
  **L809 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L810 EN**: Continues the surrounding expression or declaration: `std::optional<Fortran::evaluate::DynamicType> retTy =`.
  **L810 CN**: 继续构造周围的表达式或声明：`std::optional<Fortran::evaluate::DynamicType> retTy =`。
- **L811 EN**: Executes a call or declaration centered on `caller.getCallDescription`.
  **L811 CN**: 执行以 `caller.getCallDescription` 为核心的调用或声明。
- **L812 EN**: Comment explains nearby logic, intent, or metadata: `With HLFIR lowering, isElemental must be set to true`.
  **L812 CN**: 注释说明附近代码的逻辑、意图或元数据：`With HLFIR lowering, isElemental must be set to true`。
- **L813 EN**: Comment explains nearby logic, intent, or metadata: `if we are producing an elemental call. In this case,`.
  **L813 CN**: 注释说明附近代码的逻辑、意图或元数据：`if we are producing an elemental call. In this case,`。
- **L814 EN**: Comment explains nearby logic, intent, or metadata: `the elemental results must not be destroyed, instead,`.
  **L814 CN**: 注释说明附近代码的逻辑、意图或元数据：`the elemental results must not be destroyed, instead,`。
- **L815 EN**: Comment explains nearby logic, intent, or metadata: `the resulting array result will be finalized/destroyed`.
  **L815 CN**: 注释说明附近代码的逻辑、意图或元数据：`the resulting array result will be finalized/destroyed`。
- **L816 EN**: Comment explains nearby logic, intent, or metadata: `as needed by hlfir.destroy.`.
  **L816 CN**: 注释说明附近代码的逻辑、意图或元数据：`as needed by hlfir.destroy.`。

### Lines 817-840

````cpp
  const bool mustFinalizeResult =
      !isElemental && mustDestroyOrFinalizeFunctionResult(callSiteType, retTy);

  if (caller.mustSaveResult()) {
    assert(allocatedResult.has_value());
    fir::SaveResultOp::create(builder, loc, callResult,
                              fir::getBase(*allocatedResult), arrayResultShape,
                              resultLengths);
  }

  if (evaluateInMemory) {
    builder.setInsertionPointAfter(evaluateInMemory);
    mlir::Value expr = evaluateInMemory.getResult();
    fir::FirOpBuilder *bldr = &converter.getFirOpBuilder();
    if (!isElemental)
      stmtCtx.attachCleanup([bldr, loc, expr, mustFinalizeResult]() {
        hlfir::DestroyOp::create(*bldr, loc, expr,
                                 /*finalize=*/mustFinalizeResult);
      });
    return {LoweredResult{hlfir::EntityWithAttributes{expr}},
            mustFinalizeResult, callOp};
  }

  // Insert clean-up for the result.
````
- **L817 EN**: Continues the surrounding expression or declaration: `const bool mustFinalizeResult =`.
  **L817 CN**: 继续构造周围的表达式或声明：`const bool mustFinalizeResult =`。
- **L818 EN**: Executes a call or declaration centered on `mustDestroyOrFinalizeFunctionResult`.
  **L818 CN**: 执行以 `mustDestroyOrFinalizeFunctionResult` 为核心的调用或声明。
- **L819 EN**: Blank line separating nearby declarations or logic blocks.
  **L819 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L820 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L820 CN**: 开始 `if` 控制流语句并计算其条件。
- **L821 EN**: Checks an internal invariant in debug builds.
  **L821 CN**: 在调试构建中检查内部不变式。
- **L822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::SaveResultOp::create(builder, loc, callResult,`.
  **L822 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::SaveResultOp::create(builder, loc, callResult,`。
- **L823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::getBase(*allocatedResult), arrayResultShape,`.
  **L823 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::getBase(*allocatedResult), arrayResultShape,`。
- **L824 EN**: Executes a standalone statement or declaration: `resultLengths);`.
  **L824 CN**: 执行一条独立语句或声明：`resultLengths);`。
- **L825 EN**: Closes the current lexical scope or compound statement.
  **L825 CN**: 结束当前词法作用域或复合语句块。
- **L826 EN**: Blank line separating nearby declarations or logic blocks.
  **L826 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L827 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L827 CN**: 开始 `if` 控制流语句并计算其条件。
- **L828 EN**: Executes a call or declaration centered on `builder.setInsertionPointAfter`.
  **L828 CN**: 执行以 `builder.setInsertionPointAfter` 为核心的调用或声明。
- **L829 EN**: Initializes variable `expr` from the right-hand expression.
  **L829 CN**: 使用右侧表达式初始化变量 `expr`。
- **L830 EN**: Executes a call or declaration centered on `&converter.getFirOpBuilder`.
  **L830 CN**: 执行以 `&converter.getFirOpBuilder` 为核心的调用或声明。
- **L831 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L831 CN**: 开始 `if` 控制流语句并计算其条件。
- **L832 EN**: Starts a function, method, lambda, or structured scope: `stmtCtx.attachCleanup([bldr, loc, expr, mustFinalizeResult]() {`.
  **L832 CN**: 开始一个函数、方法、lambda 或结构化作用域：`stmtCtx.attachCleanup([bldr, loc, expr, mustFinalizeResult]() {`。
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `hlfir::DestroyOp::create(*bldr, loc, expr,`.
  **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`hlfir::DestroyOp::create(*bldr, loc, expr,`。
- **L834 EN**: Comment explains nearby logic, intent, or metadata: `finalize=*/mustFinalizeResult);`.
  **L834 CN**: 注释说明附近代码的逻辑、意图或元数据：`finalize=*/mustFinalizeResult);`。
- **L835 EN**: Executes a standalone statement or declaration: `});`.
  **L835 CN**: 执行一条独立语句或声明：`});`。
- **L836 EN**: Returns from the current function with `{LoweredResult{hlfir::EntityWithAttributes{expr}},`.
  **L836 CN**: 以 `{LoweredResult{hlfir::EntityWithAttributes{expr}},` 从当前函数返回。
- **L837 EN**: Executes a standalone statement or declaration: `mustFinalizeResult, callOp};`.
  **L837 CN**: 执行一条独立语句或声明：`mustFinalizeResult, callOp};`。
- **L838 EN**: Closes the current lexical scope or compound statement.
  **L838 CN**: 结束当前词法作用域或复合语句块。
- **L839 EN**: Blank line separating nearby declarations or logic blocks.
  **L839 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L840 EN**: Comment explains nearby logic, intent, or metadata: `Insert clean-up for the result.`.
  **L840 CN**: 注释说明附近代码的逻辑、意图或元数据：`Insert clean-up for the result.`。

### Lines 841-864

````cpp
  // In HLFIR, this is skipped when the result does not need to be finalized
  // because the result is moved to an expression that will deal with the
  // finalization.
  if (allocatedResult && mustFinalizeResult) {
    // The result must be optionally destroyed (if it is of a derived type
    // that may need finalization or deallocation of the components).
    // For an allocatable result we have to free the memory allocated
    // for the top-level entity. Note that the Destroy calls below
    // do not deallocate the top-level entity. The two clean-ups
    // must be pushed in reverse order, so that the final order is:
    //   Destroy(desc)
    //   free(desc->base_addr)
    allocatedResult->match(
        [&](const fir::MutableBoxValue &box) {
          if (box.isAllocatable()) {
            // 9.7.3.2 point 4. Deallocate allocatable results. Note that
            // finalization was done independently by calling
            // genDerivedTypeDestroy above and is not triggered by this inline
            // deallocation.
            fir::FirOpBuilder *bldr = &converter.getFirOpBuilder();
            stmtCtx.attachCleanup([bldr, loc, box]() {
              fir::factory::genFreememIfAllocated(*bldr, loc, box);
            });
          }
````
- **L841 EN**: Comment explains nearby logic, intent, or metadata: `In HLFIR, this is skipped when the result does not need to be finalized`.
  **L841 CN**: 注释说明附近代码的逻辑、意图或元数据：`In HLFIR, this is skipped when the result does not need to be finalized`。
- **L842 EN**: Comment explains nearby logic, intent, or metadata: `because the result is moved to an expression that will deal with the`.
  **L842 CN**: 注释说明附近代码的逻辑、意图或元数据：`because the result is moved to an expression that will deal with the`。
- **L843 EN**: Comment explains nearby logic, intent, or metadata: `finalization.`.
  **L843 CN**: 注释说明附近代码的逻辑、意图或元数据：`finalization.`。
- **L844 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L844 CN**: 开始 `if` 控制流语句并计算其条件。
- **L845 EN**: Comment explains nearby logic, intent, or metadata: `The result must be optionally destroyed (if it is of a derived type`.
  **L845 CN**: 注释说明附近代码的逻辑、意图或元数据：`The result must be optionally destroyed (if it is of a derived type`。
- **L846 EN**: Comment explains nearby logic, intent, or metadata: `that may need finalization or deallocation of the components).`.
  **L846 CN**: 注释说明附近代码的逻辑、意图或元数据：`that may need finalization or deallocation of the components).`。
- **L847 EN**: Comment explains nearby logic, intent, or metadata: `For an allocatable result we have to free the memory allocated`.
  **L847 CN**: 注释说明附近代码的逻辑、意图或元数据：`For an allocatable result we have to free the memory allocated`。
- **L848 EN**: Comment explains nearby logic, intent, or metadata: `for the top-level entity. Note that the Destroy calls below`.
  **L848 CN**: 注释说明附近代码的逻辑、意图或元数据：`for the top-level entity. Note that the Destroy calls below`。
- **L849 EN**: Comment explains nearby logic, intent, or metadata: `do not deallocate the top-level entity. The two clean-ups`.
  **L849 CN**: 注释说明附近代码的逻辑、意图或元数据：`do not deallocate the top-level entity. The two clean-ups`。
- **L850 EN**: Comment explains nearby logic, intent, or metadata: `must be pushed in reverse order, so that the final order is:`.
  **L850 CN**: 注释说明附近代码的逻辑、意图或元数据：`must be pushed in reverse order, so that the final order is:`。
- **L851 EN**: Comment explains nearby logic, intent, or metadata: `Destroy(desc)`.
  **L851 CN**: 注释说明附近代码的逻辑、意图或元数据：`Destroy(desc)`。
- **L852 EN**: Comment explains nearby logic, intent, or metadata: `free(desc->base_addr)`.
  **L852 CN**: 注释说明附近代码的逻辑、意图或元数据：`free(desc->base_addr)`。
- **L853 EN**: Continues logic associated with callable symbol `match`.
  **L853 CN**: 继续与可调用符号 `match` 相关的逻辑。
- **L854 EN**: Starts a function, method, lambda, or structured scope: `[&](const fir::MutableBoxValue &box) {`.
  **L854 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const fir::MutableBoxValue &box) {`。
- **L855 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L855 CN**: 开始 `if` 控制流语句并计算其条件。
- **L856 EN**: Comment explains nearby logic, intent, or metadata: `9.7.3.2 point 4. Deallocate allocatable results. Note that`.
  **L856 CN**: 注释说明附近代码的逻辑、意图或元数据：`9.7.3.2 point 4. Deallocate allocatable results. Note that`。
- **L857 EN**: Comment explains nearby logic, intent, or metadata: `finalization was done independently by calling`.
  **L857 CN**: 注释说明附近代码的逻辑、意图或元数据：`finalization was done independently by calling`。
- **L858 EN**: Comment explains nearby logic, intent, or metadata: `genDerivedTypeDestroy above and is not triggered by this inline`.
  **L858 CN**: 注释说明附近代码的逻辑、意图或元数据：`genDerivedTypeDestroy above and is not triggered by this inline`。
- **L859 EN**: Comment explains nearby logic, intent, or metadata: `deallocation.`.
  **L859 CN**: 注释说明附近代码的逻辑、意图或元数据：`deallocation.`。
- **L860 EN**: Executes a call or declaration centered on `&converter.getFirOpBuilder`.
  **L860 CN**: 执行以 `&converter.getFirOpBuilder` 为核心的调用或声明。
- **L861 EN**: Starts a function, method, lambda, or structured scope: `stmtCtx.attachCleanup([bldr, loc, box]() {`.
  **L861 CN**: 开始一个函数、方法、lambda 或结构化作用域：`stmtCtx.attachCleanup([bldr, loc, box]() {`。
- **L862 EN**: Executes a call or declaration centered on `fir::factory::genFreememIfAllocated`.
  **L862 CN**: 执行以 `fir::factory::genFreememIfAllocated` 为核心的调用或声明。
- **L863 EN**: Executes a standalone statement or declaration: `});`.
  **L863 CN**: 执行一条独立语句或声明：`});`。
- **L864 EN**: Closes the current lexical scope or compound statement.
  **L864 CN**: 结束当前词法作用域或复合语句块。

### Lines 865-888

````cpp
        },
        [](const auto &) {});

    // 7.5.6.3 point 5. Derived-type finalization for nonpointer function.
    // Note that this is also done for derived type with no final routines
    // that have allocatable components to ensure the allocatable
    // components are deallocated.
    if (mustFinalizeResult) {
      auto *bldr = &converter.getFirOpBuilder();
      stmtCtx.attachCleanup([bldr, loc, allocatedResult]() {
        mlir::Value box = bldr->createBox(loc, *allocatedResult);
        fir::runtime::genDerivedTypeDestroy(*bldr, loc, box);
      });
    }
    return {LoweredResult{*allocatedResult}, mustFinalizeResult, callOp};
  }

  if (allocatedResult)
    return {LoweredResult{*allocatedResult}, /*resultIsFinalized=*/false,
            callOp};

  // subroutine call
  if (!resultType)
    return {LoweredResult{fir::ExtendedValue{mlir::Value{}}},
````
- **L865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L865 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L866 EN**: Executes a call or declaration centered on `[]`.
  **L866 CN**: 执行以 `[]` 为核心的调用或声明。
- **L867 EN**: Blank line separating nearby declarations or logic blocks.
  **L867 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L868 EN**: Comment explains nearby logic, intent, or metadata: `7.5.6.3 point 5. Derived-type finalization for nonpointer function.`.
  **L868 CN**: 注释说明附近代码的逻辑、意图或元数据：`7.5.6.3 point 5. Derived-type finalization for nonpointer function.`。
- **L869 EN**: Comment explains nearby logic, intent, or metadata: `Note that this is also done for derived type with no final routines`.
  **L869 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note that this is also done for derived type with no final routines`。
- **L870 EN**: Comment explains nearby logic, intent, or metadata: `that have allocatable components to ensure the allocatable`.
  **L870 CN**: 注释说明附近代码的逻辑、意图或元数据：`that have allocatable components to ensure the allocatable`。
- **L871 EN**: Comment explains nearby logic, intent, or metadata: `components are deallocated.`.
  **L871 CN**: 注释说明附近代码的逻辑、意图或元数据：`components are deallocated.`。
- **L872 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L872 CN**: 开始 `if` 控制流语句并计算其条件。
- **L873 EN**: Executes a call or declaration centered on `&converter.getFirOpBuilder`.
  **L873 CN**: 执行以 `&converter.getFirOpBuilder` 为核心的调用或声明。
- **L874 EN**: Starts a function, method, lambda, or structured scope: `stmtCtx.attachCleanup([bldr, loc, allocatedResult]() {`.
  **L874 CN**: 开始一个函数、方法、lambda 或结构化作用域：`stmtCtx.attachCleanup([bldr, loc, allocatedResult]() {`。
- **L875 EN**: Initializes variable `box` from the right-hand expression.
  **L875 CN**: 使用右侧表达式初始化变量 `box`。
- **L876 EN**: Executes a call or declaration centered on `fir::runtime::genDerivedTypeDestroy`.
  **L876 CN**: 执行以 `fir::runtime::genDerivedTypeDestroy` 为核心的调用或声明。
- **L877 EN**: Executes a standalone statement or declaration: `});`.
  **L877 CN**: 执行一条独立语句或声明：`});`。
- **L878 EN**: Closes the current lexical scope or compound statement.
  **L878 CN**: 结束当前词法作用域或复合语句块。
- **L879 EN**: Returns from the current function with `{LoweredResult{*allocatedResult}, mustFinalizeResult, callOp}`.
  **L879 CN**: 以 `{LoweredResult{*allocatedResult}, mustFinalizeResult, callOp}` 从当前函数返回。
- **L880 EN**: Closes the current lexical scope or compound statement.
  **L880 CN**: 结束当前词法作用域或复合语句块。
- **L881 EN**: Blank line separating nearby declarations or logic blocks.
  **L881 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L882 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L882 CN**: 开始 `if` 控制流语句并计算其条件。
- **L883 EN**: Returns from the current function with `{LoweredResult{*allocatedResult}, /*resultIsFinalized=*/false,`.
  **L883 CN**: 以 `{LoweredResult{*allocatedResult}, /*resultIsFinalized=*/false,` 从当前函数返回。
- **L884 EN**: Executes a standalone statement or declaration: `callOp};`.
  **L884 CN**: 执行一条独立语句或声明：`callOp};`。
- **L885 EN**: Blank line separating nearby declarations or logic blocks.
  **L885 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L886 EN**: Comment explains nearby logic, intent, or metadata: `subroutine call`.
  **L886 CN**: 注释说明附近代码的逻辑、意图或元数据：`subroutine call`。
- **L887 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L887 CN**: 开始 `if` 控制流语句并计算其条件。
- **L888 EN**: Returns from the current function with `{LoweredResult{fir::ExtendedValue{mlir::Value{}}},`.
  **L888 CN**: 以 `{LoweredResult{fir::ExtendedValue{mlir::Value{}}},` 从当前函数返回。

### Lines 889-912

````cpp
            /*resultIsFinalized=*/false, callOp};

  // For now, Fortran return values are implemented with a single MLIR
  // function return value.
  assert(callNumResults == 1 && "Expected exactly one result in FUNCTION call");
  (void)callNumResults;

  // Call a BIND(C) function that return a char.
  if (caller.characterize().IsBindC() &&
      mlir::isa<fir::CharacterType>(funcType.getResults()[0])) {
    fir::CharacterType charTy =
        mlir::dyn_cast<fir::CharacterType>(funcType.getResults()[0]);
    mlir::Value len = builder.createIntegerConstant(
        loc, builder.getCharacterLengthType(), charTy.getLen());
    return {
        LoweredResult{fir::ExtendedValue{fir::CharBoxValue{callResult, len}}},
        /*resultIsFinalized=*/false, callOp};
  }

  return {LoweredResult{fir::ExtendedValue{callResult}},
          /*resultIsFinalized=*/false, callOp};
}

static hlfir::EntityWithAttributes genStmtFunctionRef(
````
- **L889 EN**: Comment explains nearby logic, intent, or metadata: `resultIsFinalized=*/false, callOp};`.
  **L889 CN**: 注释说明附近代码的逻辑、意图或元数据：`resultIsFinalized=*/false, callOp};`。
- **L890 EN**: Blank line separating nearby declarations or logic blocks.
  **L890 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L891 EN**: Comment explains nearby logic, intent, or metadata: `For now, Fortran return values are implemented with a single MLIR`.
  **L891 CN**: 注释说明附近代码的逻辑、意图或元数据：`For now, Fortran return values are implemented with a single MLIR`。
- **L892 EN**: Comment explains nearby logic, intent, or metadata: `function return value.`.
  **L892 CN**: 注释说明附近代码的逻辑、意图或元数据：`function return value.`。
- **L893 EN**: Checks an internal invariant in debug builds.
  **L893 CN**: 在调试构建中检查内部不变式。
- **L894 EN**: Executes a call or declaration centered on `statement`.
  **L894 CN**: 执行以 `statement` 为核心的调用或声明。
- **L895 EN**: Blank line separating nearby declarations or logic blocks.
  **L895 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L896 EN**: Comment explains nearby logic, intent, or metadata: `Call a BIND(C) function that return a char.`.
  **L896 CN**: 注释说明附近代码的逻辑、意图或元数据：`Call a BIND(C) function that return a char.`。
- **L897 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L897 CN**: 开始 `if` 控制流语句并计算其条件。
- **L898 EN**: Starts a function, method, lambda, or structured scope: `mlir::isa<fir::CharacterType>(funcType.getResults()[0])) {`.
  **L898 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::isa<fir::CharacterType>(funcType.getResults()[0])) {`。
- **L899 EN**: Continues the surrounding expression or declaration: `fir::CharacterType charTy =`.
  **L899 CN**: 继续构造周围的表达式或声明：`fir::CharacterType charTy =`。
- **L900 EN**: Executes a call or declaration centered on `mlir::dyn_cast<fir::CharacterType>`.
  **L900 CN**: 执行以 `mlir::dyn_cast<fir::CharacterType>` 为核心的调用或声明。
- **L901 EN**: Continues logic associated with callable symbol `createIntegerConstant`.
  **L901 CN**: 继续与可调用符号 `createIntegerConstant` 相关的逻辑。
- **L902 EN**: Executes a call or declaration centered on `builder.getCharacterLengthType`.
  **L902 CN**: 执行以 `builder.getCharacterLengthType` 为核心的调用或声明。
- **L903 EN**: Returns from the current function with `{`.
  **L903 CN**: 以 `{` 从当前函数返回。
- **L904 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LoweredResult{fir::ExtendedValue{fir::CharBoxValue{callResult, len}}},`.
  **L904 CN**: 继续一个多行参数列表、初始化器或聚合项：`LoweredResult{fir::ExtendedValue{fir::CharBoxValue{callResult, len}}},`。
- **L905 EN**: Comment explains nearby logic, intent, or metadata: `resultIsFinalized=*/false, callOp};`.
  **L905 CN**: 注释说明附近代码的逻辑、意图或元数据：`resultIsFinalized=*/false, callOp};`。
- **L906 EN**: Closes the current lexical scope or compound statement.
  **L906 CN**: 结束当前词法作用域或复合语句块。
- **L907 EN**: Blank line separating nearby declarations or logic blocks.
  **L907 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L908 EN**: Returns from the current function with `{LoweredResult{fir::ExtendedValue{callResult}},`.
  **L908 CN**: 以 `{LoweredResult{fir::ExtendedValue{callResult}},` 从当前函数返回。
- **L909 EN**: Comment explains nearby logic, intent, or metadata: `resultIsFinalized=*/false, callOp};`.
  **L909 CN**: 注释说明附近代码的逻辑、意图或元数据：`resultIsFinalized=*/false, callOp};`。
- **L910 EN**: Closes the current lexical scope or compound statement.
  **L910 CN**: 结束当前词法作用域或复合语句块。
- **L911 EN**: Blank line separating nearby declarations or logic blocks.
  **L911 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L912 EN**: Continues logic associated with callable symbol `genStmtFunctionRef`.
  **L912 CN**: 继续与可调用符号 `genStmtFunctionRef` 相关的逻辑。

### Lines 913-936

````cpp
    mlir::Location loc, Fortran::lower::AbstractConverter &converter,
    Fortran::lower::SymMap &symMap, Fortran::lower::StatementContext &stmtCtx,
    const Fortran::evaluate::ProcedureRef &procRef) {
  const Fortran::semantics::Symbol *symbol = procRef.proc().GetSymbol();
  assert(symbol && "expected symbol in ProcedureRef of statement functions");
  const auto &details = symbol->get<Fortran::semantics::SubprogramDetails>();
  fir::FirOpBuilder &builder = converter.getFirOpBuilder();

  // Statement functions have their own scope, we just need to associate
  // the dummy symbols to argument expressions. There are no
  // optional/alternate return arguments. Statement functions cannot be
  // recursive (directly or indirectly) so it is safe to add dummy symbols to
  // the local map here.
  symMap.pushScope();
  llvm::SmallVector<hlfir::AssociateOp> exprAssociations;
  for (auto [arg, bind] : llvm::zip(details.dummyArgs(), procRef.arguments())) {
    assert(arg && "alternate return in statement function");
    assert(bind && "optional argument in statement function");
    const auto *expr = bind->UnwrapExpr();
    // TODO: assumed type in statement function, that surprisingly seems
    // allowed, probably because nobody thought of restricting this usage.
    // gfortran/ifort compiles this.
    assert(expr && "assumed type used as statement function argument");
    // As per Fortran 2018 C1580, statement function arguments can only be
````
- **L913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, Fortran::lower::AbstractConverter &converter,`.
  **L913 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, Fortran::lower::AbstractConverter &converter,`。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::SymMap &symMap, Fortran::lower::StatementContext &stmtCtx,`.
  **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::SymMap &symMap, Fortran::lower::StatementContext &stmtCtx,`。
- **L915 EN**: Continues the surrounding expression or declaration: `const Fortran::evaluate::ProcedureRef &procRef) {`.
  **L915 CN**: 继续构造周围的表达式或声明：`const Fortran::evaluate::ProcedureRef &procRef) {`。
- **L916 EN**: Executes a call or declaration centered on `procRef.proc`.
  **L916 CN**: 执行以 `procRef.proc` 为核心的调用或声明。
- **L917 EN**: Checks an internal invariant in debug builds.
  **L917 CN**: 在调试构建中检查内部不变式。
- **L918 EN**: Executes a call or declaration centered on `symbol->get<Fortran::semantics::SubprogramDetails>`.
  **L918 CN**: 执行以 `symbol->get<Fortran::semantics::SubprogramDetails>` 为核心的调用或声明。
- **L919 EN**: Executes a call or declaration centered on `converter.getFirOpBuilder`.
  **L919 CN**: 执行以 `converter.getFirOpBuilder` 为核心的调用或声明。
- **L920 EN**: Blank line separating nearby declarations or logic blocks.
  **L920 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L921 EN**: Comment explains nearby logic, intent, or metadata: `Statement functions have their own scope, we just need to associate`.
  **L921 CN**: 注释说明附近代码的逻辑、意图或元数据：`Statement functions have their own scope, we just need to associate`。
- **L922 EN**: Comment explains nearby logic, intent, or metadata: `the dummy symbols to argument expressions. There are no`.
  **L922 CN**: 注释说明附近代码的逻辑、意图或元数据：`the dummy symbols to argument expressions. There are no`。
- **L923 EN**: Comment explains nearby logic, intent, or metadata: `optional/alternate return arguments. Statement functions cannot be`.
  **L923 CN**: 注释说明附近代码的逻辑、意图或元数据：`optional/alternate return arguments. Statement functions cannot be`。
- **L924 EN**: Comment explains nearby logic, intent, or metadata: `recursive (directly or indirectly) so it is safe to add dummy symbols to`.
  **L924 CN**: 注释说明附近代码的逻辑、意图或元数据：`recursive (directly or indirectly) so it is safe to add dummy symbols to`。
- **L925 EN**: Comment explains nearby logic, intent, or metadata: `the local map here.`.
  **L925 CN**: 注释说明附近代码的逻辑、意图或元数据：`the local map here.`。
- **L926 EN**: Executes a call or declaration centered on `symMap.pushScope`.
  **L926 CN**: 执行以 `symMap.pushScope` 为核心的调用或声明。
- **L927 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<hlfir::AssociateOp> exprAssociations;`.
  **L927 CN**: 执行一条独立语句或声明：`llvm::SmallVector<hlfir::AssociateOp> exprAssociations;`。
- **L928 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L928 CN**: 开始 `for` 控制流语句并计算其条件。
- **L929 EN**: Checks an internal invariant in debug builds.
  **L929 CN**: 在调试构建中检查内部不变式。
- **L930 EN**: Checks an internal invariant in debug builds.
  **L930 CN**: 在调试构建中检查内部不变式。
- **L931 EN**: Executes a call or declaration centered on `bind->UnwrapExpr`.
  **L931 CN**: 执行以 `bind->UnwrapExpr` 为核心的调用或声明。
- **L932 EN**: Comment records a pending task or caution: `TODO: assumed type in statement function, that surprisingly seems`.
  **L932 CN**: 注释记录待办事项或注意点：`TODO: assumed type in statement function, that surprisingly seems`。
- **L933 EN**: Comment explains nearby logic, intent, or metadata: `allowed, probably because nobody thought of restricting this usage.`.
  **L933 CN**: 注释说明附近代码的逻辑、意图或元数据：`allowed, probably because nobody thought of restricting this usage.`。
- **L934 EN**: Comment explains nearby logic, intent, or metadata: `gfortran/ifort compiles this.`.
  **L934 CN**: 注释说明附近代码的逻辑、意图或元数据：`gfortran/ifort compiles this.`。
- **L935 EN**: Checks an internal invariant in debug builds.
  **L935 CN**: 在调试构建中检查内部不变式。
- **L936 EN**: Comment explains nearby logic, intent, or metadata: `As per Fortran 2018 C1580, statement function arguments can only be`.
  **L936 CN**: 注释说明附近代码的逻辑、意图或元数据：`As per Fortran 2018 C1580, statement function arguments can only be`。

### Lines 937-960

````cpp
    // scalars.
    // The only care is to use the dummy character explicit length if any
    // instead of the actual argument length (that can be bigger).
    hlfir::EntityWithAttributes loweredArg = Fortran::lower::convertExprToHLFIR(
        loc, converter, *expr, symMap, stmtCtx);
    fir::FortranVariableOpInterface variableIface = loweredArg.getIfVariable();
    if (!variableIface) {
      // So far only FortranVariableOpInterface can be mapped to symbols.
      // Create an hlfir.associate to create a variable from a potential
      // value argument.
      mlir::Type argType = converter.genType(*arg);
      auto associate = hlfir::genAssociateExpr(
          loc, builder, loweredArg, argType, toStringRef(arg->name()));
      exprAssociations.push_back(associate);
      variableIface = associate;
    }
    const Fortran::semantics::DeclTypeSpec *type = arg->GetType();
    if (type &&
        type->category() == Fortran::semantics::DeclTypeSpec::Character) {
      // Instantiate character as if it was a normal dummy argument so that the
      // statement function dummy character length is applied and dealt with
      // correctly.
      symMap.addSymbol(*arg, variableIface.getBase());
      Fortran::lower::mapSymbolAttributes(converter, *arg, symMap, stmtCtx);
````
- **L937 EN**: Comment explains nearby logic, intent, or metadata: `scalars.`.
  **L937 CN**: 注释说明附近代码的逻辑、意图或元数据：`scalars.`。
- **L938 EN**: Comment explains nearby logic, intent, or metadata: `The only care is to use the dummy character explicit length if any`.
  **L938 CN**: 注释说明附近代码的逻辑、意图或元数据：`The only care is to use the dummy character explicit length if any`。
- **L939 EN**: Comment explains nearby logic, intent, or metadata: `instead of the actual argument length (that can be bigger).`.
  **L939 CN**: 注释说明附近代码的逻辑、意图或元数据：`instead of the actual argument length (that can be bigger).`。
- **L940 EN**: Continues logic associated with callable symbol `convertExprToHLFIR`.
  **L940 CN**: 继续与可调用符号 `convertExprToHLFIR` 相关的逻辑。
- **L941 EN**: Executes a standalone statement or declaration: `loc, converter, *expr, symMap, stmtCtx);`.
  **L941 CN**: 执行一条独立语句或声明：`loc, converter, *expr, symMap, stmtCtx);`。
- **L942 EN**: Initializes variable `variableIface` from the right-hand expression.
  **L942 CN**: 使用右侧表达式初始化变量 `variableIface`。
- **L943 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L943 CN**: 开始 `if` 控制流语句并计算其条件。
- **L944 EN**: Comment explains nearby logic, intent, or metadata: `So far only FortranVariableOpInterface can be mapped to symbols.`.
  **L944 CN**: 注释说明附近代码的逻辑、意图或元数据：`So far only FortranVariableOpInterface can be mapped to symbols.`。
- **L945 EN**: Comment explains nearby logic, intent, or metadata: `Create an hlfir.associate to create a variable from a potential`.
  **L945 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create an hlfir.associate to create a variable from a potential`。
- **L946 EN**: Comment explains nearby logic, intent, or metadata: `value argument.`.
  **L946 CN**: 注释说明附近代码的逻辑、意图或元数据：`value argument.`。
- **L947 EN**: Initializes variable `argType` from the right-hand expression.
  **L947 CN**: 使用右侧表达式初始化变量 `argType`。
- **L948 EN**: Continues logic associated with callable symbol `genAssociateExpr`.
  **L948 CN**: 继续与可调用符号 `genAssociateExpr` 相关的逻辑。
- **L949 EN**: Executes a call or declaration centered on `toStringRef`.
  **L949 CN**: 执行以 `toStringRef` 为核心的调用或声明。
- **L950 EN**: Executes a call or declaration centered on `exprAssociations.push_back`.
  **L950 CN**: 执行以 `exprAssociations.push_back` 为核心的调用或声明。
- **L951 EN**: Executes a standalone statement or declaration: `variableIface = associate;`.
  **L951 CN**: 执行一条独立语句或声明：`variableIface = associate;`。
- **L952 EN**: Closes the current lexical scope or compound statement.
  **L952 CN**: 结束当前词法作用域或复合语句块。
- **L953 EN**: Executes a call or declaration centered on `arg->GetType`.
  **L953 CN**: 执行以 `arg->GetType` 为核心的调用或声明。
- **L954 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L954 CN**: 开始 `if` 控制流语句并计算其条件。
- **L955 EN**: Starts a function, method, lambda, or structured scope: `type->category() == Fortran::semantics::DeclTypeSpec::Character) {`.
  **L955 CN**: 开始一个函数、方法、lambda 或结构化作用域：`type->category() == Fortran::semantics::DeclTypeSpec::Character) {`。
- **L956 EN**: Comment explains nearby logic, intent, or metadata: `Instantiate character as if it was a normal dummy argument so that the`.
  **L956 CN**: 注释说明附近代码的逻辑、意图或元数据：`Instantiate character as if it was a normal dummy argument so that the`。
- **L957 EN**: Comment explains nearby logic, intent, or metadata: `statement function dummy character length is applied and dealt with`.
  **L957 CN**: 注释说明附近代码的逻辑、意图或元数据：`statement function dummy character length is applied and dealt with`。
- **L958 EN**: Comment explains nearby logic, intent, or metadata: `correctly.`.
  **L958 CN**: 注释说明附近代码的逻辑、意图或元数据：`correctly.`。
- **L959 EN**: Executes a call or declaration centered on `symMap.addSymbol`.
  **L959 CN**: 执行以 `symMap.addSymbol` 为核心的调用或声明。
- **L960 EN**: Executes a call or declaration centered on `Fortran::lower::mapSymbolAttributes`.
  **L960 CN**: 执行以 `Fortran::lower::mapSymbolAttributes` 为核心的调用或声明。

### Lines 961-984

````cpp
    } else {
      // No need to create an extra hlfir.declare otherwise for
      // numerical and logical scalar dummies.
      symMap.addVariableDefinition(*arg, variableIface);
    }
  }

  // Explicitly map statement function host associated symbols to their
  // parent scope lowered symbol box.
  for (const Fortran::semantics::SymbolRef &sym :
       Fortran::evaluate::CollectSymbols(*details.stmtFunction()))
    if (const auto *details =
            sym->detailsIf<Fortran::semantics::HostAssocDetails>())
      converter.copySymbolBinding(details->symbol(), sym);

  hlfir::Entity result = Fortran::lower::convertExprToHLFIR(
      loc, converter, details.stmtFunction().value(), symMap, stmtCtx);
  symMap.popScope();
  // The result must not be a variable.
  result = hlfir::loadTrivialScalar(loc, builder, result);
  if (result.isVariable())
    result = hlfir::Entity{hlfir::AsExprOp::create(builder, loc, result)};
  for (auto associate : exprAssociations)
    hlfir::EndAssociateOp::create(builder, loc, associate);
````
- **L961 EN**: Transitions from the previous branch into the alternative path.
  **L961 CN**: 从前一个分支过渡到备选路径。
- **L962 EN**: Comment explains nearby logic, intent, or metadata: `No need to create an extra hlfir.declare otherwise for`.
  **L962 CN**: 注释说明附近代码的逻辑、意图或元数据：`No need to create an extra hlfir.declare otherwise for`。
- **L963 EN**: Comment explains nearby logic, intent, or metadata: `numerical and logical scalar dummies.`.
  **L963 CN**: 注释说明附近代码的逻辑、意图或元数据：`numerical and logical scalar dummies.`。
- **L964 EN**: Executes a call or declaration centered on `symMap.addVariableDefinition`.
  **L964 CN**: 执行以 `symMap.addVariableDefinition` 为核心的调用或声明。
- **L965 EN**: Closes the current lexical scope or compound statement.
  **L965 CN**: 结束当前词法作用域或复合语句块。
- **L966 EN**: Closes the current lexical scope or compound statement.
  **L966 CN**: 结束当前词法作用域或复合语句块。
- **L967 EN**: Blank line separating nearby declarations or logic blocks.
  **L967 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L968 EN**: Comment explains nearby logic, intent, or metadata: `Explicitly map statement function host associated symbols to their`.
  **L968 CN**: 注释说明附近代码的逻辑、意图或元数据：`Explicitly map statement function host associated symbols to their`。
- **L969 EN**: Comment explains nearby logic, intent, or metadata: `parent scope lowered symbol box.`.
  **L969 CN**: 注释说明附近代码的逻辑、意图或元数据：`parent scope lowered symbol box.`。
- **L970 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L970 CN**: 开始 `for` 控制流语句并计算其条件。
- **L971 EN**: Continues logic associated with callable symbol `CollectSymbols`.
  **L971 CN**: 继续与可调用符号 `CollectSymbols` 相关的逻辑。
- **L972 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L972 CN**: 开始 `if` 控制流语句并计算其条件。
- **L973 EN**: Continues logic associated with callable symbol `HostAssocDetails>`.
  **L973 CN**: 继续与可调用符号 `HostAssocDetails>` 相关的逻辑。
- **L974 EN**: Executes a call or declaration centered on `converter.copySymbolBinding`.
  **L974 CN**: 执行以 `converter.copySymbolBinding` 为核心的调用或声明。
- **L975 EN**: Blank line separating nearby declarations or logic blocks.
  **L975 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L976 EN**: Continues logic associated with callable symbol `convertExprToHLFIR`.
  **L976 CN**: 继续与可调用符号 `convertExprToHLFIR` 相关的逻辑。
- **L977 EN**: Executes a call or declaration centered on `details.stmtFunction`.
  **L977 CN**: 执行以 `details.stmtFunction` 为核心的调用或声明。
- **L978 EN**: Executes a call or declaration centered on `symMap.popScope`.
  **L978 CN**: 执行以 `symMap.popScope` 为核心的调用或声明。
- **L979 EN**: Comment explains nearby logic, intent, or metadata: `The result must not be a variable.`.
  **L979 CN**: 注释说明附近代码的逻辑、意图或元数据：`The result must not be a variable.`。
- **L980 EN**: Executes a call or declaration centered on `hlfir::loadTrivialScalar`.
  **L980 CN**: 执行以 `hlfir::loadTrivialScalar` 为核心的调用或声明。
- **L981 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L981 CN**: 开始 `if` 控制流语句并计算其条件。
- **L982 EN**: Executes a call or declaration centered on `hlfir::Entity{hlfir::AsExprOp::create`.
  **L982 CN**: 执行以 `hlfir::Entity{hlfir::AsExprOp::create` 为核心的调用或声明。
- **L983 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L983 CN**: 开始 `for` 控制流语句并计算其条件。
- **L984 EN**: Executes a call or declaration centered on `hlfir::EndAssociateOp::create`.
  **L984 CN**: 执行以 `hlfir::EndAssociateOp::create` 为核心的调用或声明。

### Lines 985-1008

````cpp
  return hlfir::EntityWithAttributes{result};
}

namespace {
// Structure to hold the information about the call and the lowering context.
// This structure is intended to help threading the information
// through the various lowering calls without having to pass every
// required structure one by one.
struct CallContext {
  CallContext(const Fortran::evaluate::ProcedureRef &procRef,
              std::optional<mlir::Type> resultType, mlir::Location loc,
              Fortran::lower::AbstractConverter &converter,
              Fortran::lower::SymMap &symMap,
              Fortran::lower::StatementContext &stmtCtx, bool doCopyIn = true)
      : procRef{procRef}, converter{converter}, symMap{symMap},
        stmtCtx{stmtCtx}, resultType{resultType}, loc{loc}, doCopyIn{doCopyIn} {
  }

  fir::FirOpBuilder &getBuilder() { return converter.getFirOpBuilder(); }

  std::string getProcedureName() const {
    if (const Fortran::semantics::Symbol *sym = procRef.proc().GetSymbol())
      return sym->GetUltimate().name().ToString();
    return procRef.proc().GetName();
````
- **L985 EN**: Returns from the current function with `hlfir::EntityWithAttributes{result}`.
  **L985 CN**: 以 `hlfir::EntityWithAttributes{result}` 从当前函数返回。
- **L986 EN**: Closes the current lexical scope or compound statement.
  **L986 CN**: 结束当前词法作用域或复合语句块。
- **L987 EN**: Blank line separating nearby declarations or logic blocks.
  **L987 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L988 EN**: Opens namespace scope ``.
  **L988 CN**: 打开命名空间作用域 ``。
- **L989 EN**: Comment explains nearby logic, intent, or metadata: `Structure to hold the information about the call and the lowering context.`.
  **L989 CN**: 注释说明附近代码的逻辑、意图或元数据：`Structure to hold the information about the call and the lowering context.`。
- **L990 EN**: Comment explains nearby logic, intent, or metadata: `This structure is intended to help threading the information`.
  **L990 CN**: 注释说明附近代码的逻辑、意图或元数据：`This structure is intended to help threading the information`。
- **L991 EN**: Comment explains nearby logic, intent, or metadata: `through the various lowering calls without having to pass every`.
  **L991 CN**: 注释说明附近代码的逻辑、意图或元数据：`through the various lowering calls without having to pass every`。
- **L992 EN**: Comment explains nearby logic, intent, or metadata: `required structure one by one.`.
  **L992 CN**: 注释说明附近代码的逻辑、意图或元数据：`required structure one by one.`。
- **L993 EN**: Declares struct `CallContext`.
  **L993 CN**: 声明 struct `CallContext`。
- **L994 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CallContext(const Fortran::evaluate::ProcedureRef &procRef,`.
  **L994 CN**: 继续一个多行参数列表、初始化器或聚合项：`CallContext(const Fortran::evaluate::ProcedureRef &procRef,`。
- **L995 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::optional<mlir::Type> resultType, mlir::Location loc,`.
  **L995 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::optional<mlir::Type> resultType, mlir::Location loc,`。
- **L996 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::AbstractConverter &converter,`.
  **L996 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::AbstractConverter &converter,`。
- **L997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::SymMap &symMap,`.
  **L997 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::SymMap &symMap,`。
- **L998 EN**: Continues the surrounding expression or declaration: `Fortran::lower::StatementContext &stmtCtx, bool doCopyIn = true)`.
  **L998 CN**: 继续构造周围的表达式或声明：`Fortran::lower::StatementContext &stmtCtx, bool doCopyIn = true)`。
- **L999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: procRef{procRef}, converter{converter}, symMap{symMap},`.
  **L999 CN**: 继续一个多行参数列表、初始化器或聚合项：`: procRef{procRef}, converter{converter}, symMap{symMap},`。
- **L1000 EN**: Continues the surrounding expression or declaration: `stmtCtx{stmtCtx}, resultType{resultType}, loc{loc}, doCopyIn{doCopyIn} {`.
  **L1000 CN**: 继续构造周围的表达式或声明：`stmtCtx{stmtCtx}, resultType{resultType}, loc{loc}, doCopyIn{doCopyIn} {`。
- **L1001 EN**: Closes the current lexical scope or compound statement.
  **L1001 CN**: 结束当前词法作用域或复合语句块。
- **L1002 EN**: Blank line separating nearby declarations or logic blocks.
  **L1002 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1003 EN**: Continues logic associated with callable symbol `getBuilder`.
  **L1003 CN**: 继续与可调用符号 `getBuilder` 相关的逻辑。
- **L1004 EN**: Blank line separating nearby declarations or logic blocks.
  **L1004 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1005 EN**: Starts a function, method, lambda, or structured scope: `std::string getProcedureName() const {`.
  **L1005 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::string getProcedureName() const {`。
- **L1006 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1006 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1007 EN**: Returns from the current function with `sym->GetUltimate().name().ToString()`.
  **L1007 CN**: 以 `sym->GetUltimate().name().ToString()` 从当前函数返回。
- **L1008 EN**: Returns from the current function with `procRef.proc().GetName()`.
  **L1008 CN**: 以 `procRef.proc().GetName()` 从当前函数返回。

### Lines 1009-1032

````cpp
  }

  /// Is this a call to an elemental procedure with at least one array argument?
  bool isElementalProcWithArrayArgs() const {
    if (procRef.IsElemental())
      for (const std::optional<Fortran::evaluate::ActualArgument> &arg :
           procRef.arguments())
        if (arg && arg->Rank() != 0)
          return true;
    return false;
  }

  /// Is this a statement function reference?
  bool isStatementFunctionCall() const {
    if (const Fortran::semantics::Symbol *symbol = procRef.proc().GetSymbol())
      if (const auto *details =
              symbol->detailsIf<Fortran::semantics::SubprogramDetails>())
        return details->stmtFunction().has_value();
    return false;
  }

  /// Is this a call to a BIND(C) procedure?
  bool isBindcCall() const {
    if (const Fortran::semantics::Symbol *symbol = procRef.proc().GetSymbol())
````
- **L1009 EN**: Closes the current lexical scope or compound statement.
  **L1009 CN**: 结束当前词法作用域或复合语句块。
- **L1010 EN**: Blank line separating nearby declarations or logic blocks.
  **L1010 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1011 EN**: Comment explains nearby logic, intent, or metadata: `Is this a call to an elemental procedure with at least one array argument?`.
  **L1011 CN**: 注释说明附近代码的逻辑、意图或元数据：`Is this a call to an elemental procedure with at least one array argument?`。
- **L1012 EN**: Starts a function, method, lambda, or structured scope: `bool isElementalProcWithArrayArgs() const {`.
  **L1012 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool isElementalProcWithArrayArgs() const {`。
- **L1013 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1013 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1014 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1014 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1015 EN**: Continues logic associated with callable symbol `arguments`.
  **L1015 CN**: 继续与可调用符号 `arguments` 相关的逻辑。
- **L1016 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1016 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1017 EN**: Returns from the current function with `true`.
  **L1017 CN**: 以 `true` 从当前函数返回。
- **L1018 EN**: Returns from the current function with `false`.
  **L1018 CN**: 以 `false` 从当前函数返回。
- **L1019 EN**: Closes the current lexical scope or compound statement.
  **L1019 CN**: 结束当前词法作用域或复合语句块。
- **L1020 EN**: Blank line separating nearby declarations or logic blocks.
  **L1020 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1021 EN**: Comment explains nearby logic, intent, or metadata: `Is this a statement function reference?`.
  **L1021 CN**: 注释说明附近代码的逻辑、意图或元数据：`Is this a statement function reference?`。
- **L1022 EN**: Starts a function, method, lambda, or structured scope: `bool isStatementFunctionCall() const {`.
  **L1022 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool isStatementFunctionCall() const {`。
- **L1023 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1023 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1024 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1024 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1025 EN**: Continues logic associated with callable symbol `SubprogramDetails>`.
  **L1025 CN**: 继续与可调用符号 `SubprogramDetails>` 相关的逻辑。
- **L1026 EN**: Returns from the current function with `details->stmtFunction().has_value()`.
  **L1026 CN**: 以 `details->stmtFunction().has_value()` 从当前函数返回。
- **L1027 EN**: Returns from the current function with `false`.
  **L1027 CN**: 以 `false` 从当前函数返回。
- **L1028 EN**: Closes the current lexical scope or compound statement.
  **L1028 CN**: 结束当前词法作用域或复合语句块。
- **L1029 EN**: Blank line separating nearby declarations or logic blocks.
  **L1029 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1030 EN**: Comment explains nearby logic, intent, or metadata: `Is this a call to a BIND(C) procedure?`.
  **L1030 CN**: 注释说明附近代码的逻辑、意图或元数据：`Is this a call to a BIND(C) procedure?`。
- **L1031 EN**: Starts a function, method, lambda, or structured scope: `bool isBindcCall() const {`.
  **L1031 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool isBindcCall() const {`。
- **L1032 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1032 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1033-1056

````cpp
      return Fortran::semantics::IsBindCProcedure(*symbol);
    return false;
  }

  const Fortran::evaluate::ProcedureRef &procRef;
  Fortran::lower::AbstractConverter &converter;
  Fortran::lower::SymMap &symMap;
  Fortran::lower::StatementContext &stmtCtx;
  std::optional<mlir::Type> resultType;
  mlir::Location loc;
  bool doCopyIn;
};

using ExvAndCleanup =
    std::pair<fir::ExtendedValue, std::optional<hlfir::CleanupFunction>>;
} // namespace

// Helper to transform a fir::ExtendedValue to an hlfir::EntityWithAttributes.
static hlfir::EntityWithAttributes
extendedValueToHlfirEntity(mlir::Location loc, fir::FirOpBuilder &builder,
                           const fir::ExtendedValue &exv, llvm::StringRef name,
                           mlir::Operation *insertBefore = nullptr) {
  mlir::Value firBase = fir::getBase(exv);
  mlir::Type firBaseTy = firBase.getType();
````
- **L1033 EN**: Returns from the current function with `Fortran::semantics::IsBindCProcedure(*symbol)`.
  **L1033 CN**: 以 `Fortran::semantics::IsBindCProcedure(*symbol)` 从当前函数返回。
- **L1034 EN**: Returns from the current function with `false`.
  **L1034 CN**: 以 `false` 从当前函数返回。
- **L1035 EN**: Closes the current lexical scope or compound statement.
  **L1035 CN**: 结束当前词法作用域或复合语句块。
- **L1036 EN**: Blank line separating nearby declarations or logic blocks.
  **L1036 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1037 EN**: Executes a standalone statement or declaration: `const Fortran::evaluate::ProcedureRef &procRef;`.
  **L1037 CN**: 执行一条独立语句或声明：`const Fortran::evaluate::ProcedureRef &procRef;`。
- **L1038 EN**: Executes a standalone statement or declaration: `Fortran::lower::AbstractConverter &converter;`.
  **L1038 CN**: 执行一条独立语句或声明：`Fortran::lower::AbstractConverter &converter;`。
- **L1039 EN**: Executes a standalone statement or declaration: `Fortran::lower::SymMap &symMap;`.
  **L1039 CN**: 执行一条独立语句或声明：`Fortran::lower::SymMap &symMap;`。
- **L1040 EN**: Executes a standalone statement or declaration: `Fortran::lower::StatementContext &stmtCtx;`.
  **L1040 CN**: 执行一条独立语句或声明：`Fortran::lower::StatementContext &stmtCtx;`。
- **L1041 EN**: Executes a standalone statement or declaration: `std::optional<mlir::Type> resultType;`.
  **L1041 CN**: 执行一条独立语句或声明：`std::optional<mlir::Type> resultType;`。
- **L1042 EN**: Executes a standalone statement or declaration: `mlir::Location loc;`.
  **L1042 CN**: 执行一条独立语句或声明：`mlir::Location loc;`。
- **L1043 EN**: Executes a standalone statement or declaration: `bool doCopyIn;`.
  **L1043 CN**: 执行一条独立语句或声明：`bool doCopyIn;`。
- **L1044 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1044 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1045 EN**: Blank line separating nearby declarations or logic blocks.
  **L1045 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1046 EN**: Defines alias `ExvAndCleanup` to simplify later code.
  **L1046 CN**: 定义别名 `ExvAndCleanup` 以简化后续代码。
- **L1047 EN**: Executes a standalone statement or declaration: `std::pair<fir::ExtendedValue, std::optional<hlfir::CleanupFunction>>;`.
  **L1047 CN**: 执行一条独立语句或声明：`std::pair<fir::ExtendedValue, std::optional<hlfir::CleanupFunction>>;`。
- **L1048 EN**: Closes a namespace scope with a trailing comment: `} // namespace`.
  **L1048 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L1049 EN**: Blank line separating nearby declarations or logic blocks.
  **L1049 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1050 EN**: Comment explains nearby logic, intent, or metadata: `Helper to transform a fir::ExtendedValue to an hlfir::EntityWithAttributes.`.
  **L1050 CN**: 注释说明附近代码的逻辑、意图或元数据：`Helper to transform a fir::ExtendedValue to an hlfir::EntityWithAttributes.`。
- **L1051 EN**: Continues the surrounding expression or declaration: `static hlfir::EntityWithAttributes`.
  **L1051 CN**: 继续构造周围的表达式或声明：`static hlfir::EntityWithAttributes`。
- **L1052 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `extendedValueToHlfirEntity(mlir::Location loc, fir::FirOpBuilder &builder,`.
  **L1052 CN**: 继续一个多行参数列表、初始化器或聚合项：`extendedValueToHlfirEntity(mlir::Location loc, fir::FirOpBuilder &builder,`。
- **L1053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::ExtendedValue &exv, llvm::StringRef name,`.
  **L1053 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::ExtendedValue &exv, llvm::StringRef name,`。
- **L1054 EN**: Continues the surrounding expression or declaration: `mlir::Operation *insertBefore = nullptr) {`.
  **L1054 CN**: 继续构造周围的表达式或声明：`mlir::Operation *insertBefore = nullptr) {`。
- **L1055 EN**: Initializes variable `firBase` from the right-hand expression.
  **L1055 CN**: 使用右侧表达式初始化变量 `firBase`。
- **L1056 EN**: Initializes variable `firBaseTy` from the right-hand expression.
  **L1056 CN**: 使用右侧表达式初始化变量 `firBaseTy`。

### Lines 1057-1080

````cpp
  if (fir::isa_trivial(firBaseTy))
    return hlfir::EntityWithAttributes{firBase};
  if (auto charTy = mlir::dyn_cast<fir::CharacterType>(firBase.getType())) {
    // CHAR() intrinsic and BIND(C) procedures returning CHARACTER(1)
    // are lowered to a fir.char<kind,1> that is not in memory.
    // This tends to cause a lot of bugs because the rest of the
    // infrastructure is mostly tested with characters that are
    // in memory.
    // To avoid having to deal with this special case here and there,
    // place it in memory here. If this turns out to be suboptimal,
    // this could be fixed, but for now llvm opt -O1 is able to get
    // rid of the memory indirection in a = char(b), so there is
    // little incentive to increase the compiler complexity.
    hlfir::Entity storage{builder.createTemporary(loc, charTy)};
    fir::StoreOp::create(builder, loc, firBase, storage);
    auto asExpr = hlfir::AsExprOp::create(
        builder, loc, storage, /*mustFree=*/builder.createBool(loc, false));
    return hlfir::EntityWithAttributes{asExpr.getResult()};
  }
  // TODO: better scoping model in FIR.
  // The declare for result storage allocated on the callee side must
  // currently be emitted before the call so that MLIR level inlining does not
  // break aliasing by introducing a fir.dummy_scope between the alloca and
  // fir.declare that leads the alias analysis to think that the result
````
- **L1057 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1057 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1058 EN**: Returns from the current function with `hlfir::EntityWithAttributes{firBase}`.
  **L1058 CN**: 以 `hlfir::EntityWithAttributes{firBase}` 从当前函数返回。
- **L1059 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1059 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1060 EN**: Comment explains nearby logic, intent, or metadata: `CHAR() intrinsic and BIND(C) procedures returning CHARACTER(1)`.
  **L1060 CN**: 注释说明附近代码的逻辑、意图或元数据：`CHAR() intrinsic and BIND(C) procedures returning CHARACTER(1)`。
- **L1061 EN**: Comment explains nearby logic, intent, or metadata: `are lowered to a fir.char<kind,1> that is not in memory.`.
  **L1061 CN**: 注释说明附近代码的逻辑、意图或元数据：`are lowered to a fir.char<kind,1> that is not in memory.`。
- **L1062 EN**: Comment explains nearby logic, intent, or metadata: `This tends to cause a lot of bugs because the rest of the`.
  **L1062 CN**: 注释说明附近代码的逻辑、意图或元数据：`This tends to cause a lot of bugs because the rest of the`。
- **L1063 EN**: Comment explains nearby logic, intent, or metadata: `infrastructure is mostly tested with characters that are`.
  **L1063 CN**: 注释说明附近代码的逻辑、意图或元数据：`infrastructure is mostly tested with characters that are`。
- **L1064 EN**: Comment explains nearby logic, intent, or metadata: `in memory.`.
  **L1064 CN**: 注释说明附近代码的逻辑、意图或元数据：`in memory.`。
- **L1065 EN**: Comment explains nearby logic, intent, or metadata: `To avoid having to deal with this special case here and there,`.
  **L1065 CN**: 注释说明附近代码的逻辑、意图或元数据：`To avoid having to deal with this special case here and there,`。
- **L1066 EN**: Comment explains nearby logic, intent, or metadata: `place it in memory here. If this turns out to be suboptimal,`.
  **L1066 CN**: 注释说明附近代码的逻辑、意图或元数据：`place it in memory here. If this turns out to be suboptimal,`。
- **L1067 EN**: Comment explains nearby logic, intent, or metadata: `this could be fixed, but for now llvm opt -O1 is able to get`.
  **L1067 CN**: 注释说明附近代码的逻辑、意图或元数据：`this could be fixed, but for now llvm opt -O1 is able to get`。
- **L1068 EN**: Comment explains nearby logic, intent, or metadata: `rid of the memory indirection in a = char(b), so there is`.
  **L1068 CN**: 注释说明附近代码的逻辑、意图或元数据：`rid of the memory indirection in a = char(b), so there is`。
- **L1069 EN**: Comment explains nearby logic, intent, or metadata: `little incentive to increase the compiler complexity.`.
  **L1069 CN**: 注释说明附近代码的逻辑、意图或元数据：`little incentive to increase the compiler complexity.`。
- **L1070 EN**: Executes a call or declaration centered on `storage{builder.createTemporary`.
  **L1070 CN**: 执行以 `storage{builder.createTemporary` 为核心的调用或声明。
- **L1071 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L1071 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L1072 EN**: Continues logic associated with callable symbol `create`.
  **L1072 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1073 EN**: Executes a call or declaration centered on `/*mustFree=*/builder.createBool`.
  **L1073 CN**: 执行以 `/*mustFree=*/builder.createBool` 为核心的调用或声明。
- **L1074 EN**: Returns from the current function with `hlfir::EntityWithAttributes{asExpr.getResult()}`.
  **L1074 CN**: 以 `hlfir::EntityWithAttributes{asExpr.getResult()}` 从当前函数返回。
- **L1075 EN**: Closes the current lexical scope or compound statement.
  **L1075 CN**: 结束当前词法作用域或复合语句块。
- **L1076 EN**: Comment records a pending task or caution: `TODO: better scoping model in FIR.`.
  **L1076 CN**: 注释记录待办事项或注意点：`TODO: better scoping model in FIR.`。
- **L1077 EN**: Comment explains nearby logic, intent, or metadata: `The declare for result storage allocated on the callee side must`.
  **L1077 CN**: 注释说明附近代码的逻辑、意图或元数据：`The declare for result storage allocated on the callee side must`。
- **L1078 EN**: Comment explains nearby logic, intent, or metadata: `currently be emitted before the call so that MLIR level inlining does not`.
  **L1078 CN**: 注释说明附近代码的逻辑、意图或元数据：`currently be emitted before the call so that MLIR level inlining does not`。
- **L1079 EN**: Comment explains nearby logic, intent, or metadata: `break aliasing by introducing a fir.dummy_scope between the alloca and`.
  **L1079 CN**: 注释说明附近代码的逻辑、意图或元数据：`break aliasing by introducing a fir.dummy_scope between the alloca and`。
- **L1080 EN**: Comment explains nearby logic, intent, or metadata: `fir.declare that leads the alias analysis to think that the result`.
  **L1080 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.declare that leads the alias analysis to think that the result`。

### Lines 1081-1104

````cpp
  // allocation is a local inside the callee scope that cannot alias with the
  // usage of that temporary inside the callee because they are made through a
  // declare with the TARGET attribute.
  mlir::OpBuilder::InsertionGuard guard(builder);
  if (insertBefore)
    builder.setInsertionPoint(insertBefore);
  hlfir::EntityWithAttributes declare = hlfir::genDeclare(
      loc, builder, exv, name, fir::FortranVariableFlagsAttr{});
  // Replace the fir.save_result "to" by the declare results instead of
  // directly using the alloca.
  if (insertBefore && insertBefore->getNumResults() == 1)
    for (auto resUser : insertBefore->getResult(0).getUsers())
      if (auto save_result = llvm::dyn_cast<fir::SaveResultOp>(resUser))
        save_result.getMemrefMutable().assign(declare.getFirBase());
  return declare;
}
namespace {
/// Structure to hold the clean-up related to a dummy argument preparation
/// that may have to be done after a call (copy-out or temporary deallocation).
struct CallCleanUp {
  struct CopyIn {
    void genCleanUp(mlir::Location loc, fir::FirOpBuilder &builder) {
      hlfir::CopyOutOp::create(builder, loc, tempBox, wasCopied, copyBackVar);
    }
````
- **L1081 EN**: Comment explains nearby logic, intent, or metadata: `allocation is a local inside the callee scope that cannot alias with the`.
  **L1081 CN**: 注释说明附近代码的逻辑、意图或元数据：`allocation is a local inside the callee scope that cannot alias with the`。
- **L1082 EN**: Comment explains nearby logic, intent, or metadata: `usage of that temporary inside the callee because they are made through a`.
  **L1082 CN**: 注释说明附近代码的逻辑、意图或元数据：`usage of that temporary inside the callee because they are made through a`。
- **L1083 EN**: Comment explains nearby logic, intent, or metadata: `declare with the TARGET attribute.`.
  **L1083 CN**: 注释说明附近代码的逻辑、意图或元数据：`declare with the TARGET attribute.`。
- **L1084 EN**: Executes a call or declaration centered on `guard`.
  **L1084 CN**: 执行以 `guard` 为核心的调用或声明。
- **L1085 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1085 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1086 EN**: Executes a call or declaration centered on `builder.setInsertionPoint`.
  **L1086 CN**: 执行以 `builder.setInsertionPoint` 为核心的调用或声明。
- **L1087 EN**: Continues logic associated with callable symbol `genDeclare`.
  **L1087 CN**: 继续与可调用符号 `genDeclare` 相关的逻辑。
- **L1088 EN**: Executes a standalone statement or declaration: `loc, builder, exv, name, fir::FortranVariableFlagsAttr{});`.
  **L1088 CN**: 执行一条独立语句或声明：`loc, builder, exv, name, fir::FortranVariableFlagsAttr{});`。
- **L1089 EN**: Comment explains nearby logic, intent, or metadata: `Replace the fir.save_result "to" by the declare results instead of`.
  **L1089 CN**: 注释说明附近代码的逻辑、意图或元数据：`Replace the fir.save_result "to" by the declare results instead of`。
- **L1090 EN**: Comment explains nearby logic, intent, or metadata: `directly using the alloca.`.
  **L1090 CN**: 注释说明附近代码的逻辑、意图或元数据：`directly using the alloca.`。
- **L1091 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1091 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1092 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1092 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1093 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1093 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1094 EN**: Executes a call or declaration centered on `save_result.getMemrefMutable`.
  **L1094 CN**: 执行以 `save_result.getMemrefMutable` 为核心的调用或声明。
- **L1095 EN**: Returns from the current function with `declare`.
  **L1095 CN**: 以 `declare` 从当前函数返回。
- **L1096 EN**: Closes the current lexical scope or compound statement.
  **L1096 CN**: 结束当前词法作用域或复合语句块。
- **L1097 EN**: Opens namespace scope ``.
  **L1097 CN**: 打开命名空间作用域 ``。
- **L1098 EN**: Comment explains nearby logic, intent, or metadata: `Structure to hold the clean-up related to a dummy argument preparation`.
  **L1098 CN**: 注释说明附近代码的逻辑、意图或元数据：`Structure to hold the clean-up related to a dummy argument preparation`。
- **L1099 EN**: Comment explains nearby logic, intent, or metadata: `that may have to be done after a call (copy-out or temporary deallocation).`.
  **L1099 CN**: 注释说明附近代码的逻辑、意图或元数据：`that may have to be done after a call (copy-out or temporary deallocation).`。
- **L1100 EN**: Declares struct `CallCleanUp`.
  **L1100 CN**: 声明 struct `CallCleanUp`。
- **L1101 EN**: Declares struct `CopyIn`.
  **L1101 CN**: 声明 struct `CopyIn`。
- **L1102 EN**: Starts a function, method, lambda, or structured scope: `void genCleanUp(mlir::Location loc, fir::FirOpBuilder &builder) {`.
  **L1102 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void genCleanUp(mlir::Location loc, fir::FirOpBuilder &builder) {`。
- **L1103 EN**: Executes a call or declaration centered on `hlfir::CopyOutOp::create`.
  **L1103 CN**: 执行以 `hlfir::CopyOutOp::create` 为核心的调用或声明。
- **L1104 EN**: Closes the current lexical scope or compound statement.
  **L1104 CN**: 结束当前词法作用域或复合语句块。

### Lines 1105-1128

````cpp
    // address of the descriptor holding the temp if a temp was created.
    mlir::Value tempBox;
    // Boolean indicating if a copy was made or not.
    mlir::Value wasCopied;
    // copyBackVar may be null if copy back is not needed.
    mlir::Value copyBackVar;
  };
  struct ExprAssociate {
    void genCleanUp(mlir::Location loc, fir::FirOpBuilder &builder) {
      hlfir::EndAssociateOp::create(builder, loc, tempVar, mustFree);
    }
    mlir::Value tempVar;
    mlir::Value mustFree;
  };

  /// Generate clean-up code.
  /// If \p postponeAssociates is true, the ExprAssociate clean-up
  /// is not generated, and instead the corresponding CallCleanUp
  /// object is returned as the result.
  std::optional<CallCleanUp> genCleanUp(mlir::Location loc,
                                        fir::FirOpBuilder &builder,
                                        bool postponeAssociates) {
    std::optional<CallCleanUp> postponed;
    Fortran::common::visit(Fortran::common::visitors{
````
- **L1105 EN**: Comment explains nearby logic, intent, or metadata: `address of the descriptor holding the temp if a temp was created.`.
  **L1105 CN**: 注释说明附近代码的逻辑、意图或元数据：`address of the descriptor holding the temp if a temp was created.`。
- **L1106 EN**: Executes a standalone statement or declaration: `mlir::Value tempBox;`.
  **L1106 CN**: 执行一条独立语句或声明：`mlir::Value tempBox;`。
- **L1107 EN**: Comment explains nearby logic, intent, or metadata: `Boolean indicating if a copy was made or not.`.
  **L1107 CN**: 注释说明附近代码的逻辑、意图或元数据：`Boolean indicating if a copy was made or not.`。
- **L1108 EN**: Executes a standalone statement or declaration: `mlir::Value wasCopied;`.
  **L1108 CN**: 执行一条独立语句或声明：`mlir::Value wasCopied;`。
- **L1109 EN**: Comment explains nearby logic, intent, or metadata: `copyBackVar may be null if copy back is not needed.`.
  **L1109 CN**: 注释说明附近代码的逻辑、意图或元数据：`copyBackVar may be null if copy back is not needed.`。
- **L1110 EN**: Executes a standalone statement or declaration: `mlir::Value copyBackVar;`.
  **L1110 CN**: 执行一条独立语句或声明：`mlir::Value copyBackVar;`。
- **L1111 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1111 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1112 EN**: Declares struct `ExprAssociate`.
  **L1112 CN**: 声明 struct `ExprAssociate`。
- **L1113 EN**: Starts a function, method, lambda, or structured scope: `void genCleanUp(mlir::Location loc, fir::FirOpBuilder &builder) {`.
  **L1113 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void genCleanUp(mlir::Location loc, fir::FirOpBuilder &builder) {`。
- **L1114 EN**: Executes a call or declaration centered on `hlfir::EndAssociateOp::create`.
  **L1114 CN**: 执行以 `hlfir::EndAssociateOp::create` 为核心的调用或声明。
- **L1115 EN**: Closes the current lexical scope or compound statement.
  **L1115 CN**: 结束当前词法作用域或复合语句块。
- **L1116 EN**: Executes a standalone statement or declaration: `mlir::Value tempVar;`.
  **L1116 CN**: 执行一条独立语句或声明：`mlir::Value tempVar;`。
- **L1117 EN**: Executes a standalone statement or declaration: `mlir::Value mustFree;`.
  **L1117 CN**: 执行一条独立语句或声明：`mlir::Value mustFree;`。
- **L1118 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1118 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1119 EN**: Blank line separating nearby declarations or logic blocks.
  **L1119 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1120 EN**: Comment explains nearby logic, intent, or metadata: `Generate clean-up code.`.
  **L1120 CN**: 注释说明附近代码的逻辑、意图或元数据：`Generate clean-up code.`。
- **L1121 EN**: Comment explains nearby logic, intent, or metadata: `If \p postponeAssociates is true, the ExprAssociate clean-up`.
  **L1121 CN**: 注释说明附近代码的逻辑、意图或元数据：`If \p postponeAssociates is true, the ExprAssociate clean-up`。
- **L1122 EN**: Comment explains nearby logic, intent, or metadata: `is not generated, and instead the corresponding CallCleanUp`.
  **L1122 CN**: 注释说明附近代码的逻辑、意图或元数据：`is not generated, and instead the corresponding CallCleanUp`。
- **L1123 EN**: Comment explains nearby logic, intent, or metadata: `object is returned as the result.`.
  **L1123 CN**: 注释说明附近代码的逻辑、意图或元数据：`object is returned as the result.`。
- **L1124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::optional<CallCleanUp> genCleanUp(mlir::Location loc,`.
  **L1124 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::optional<CallCleanUp> genCleanUp(mlir::Location loc,`。
- **L1125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::FirOpBuilder &builder,`.
  **L1125 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::FirOpBuilder &builder,`。
- **L1126 EN**: Continues the surrounding expression or declaration: `bool postponeAssociates) {`.
  **L1126 CN**: 继续构造周围的表达式或声明：`bool postponeAssociates) {`。
- **L1127 EN**: Executes a standalone statement or declaration: `std::optional<CallCleanUp> postponed;`.
  **L1127 CN**: 执行一条独立语句或声明：`std::optional<CallCleanUp> postponed;`。
- **L1128 EN**: Starts a function, method, lambda, or structured scope: `Fortran::common::visit(Fortran::common::visitors{`.
  **L1128 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Fortran::common::visit(Fortran::common::visitors{`。

### Lines 1129-1152

````cpp
                               [&](CopyIn &c) { c.genCleanUp(loc, builder); },
                               [&](ExprAssociate &c) {
                                 if (postponeAssociates)
                                   postponed = CallCleanUp{c};
                                 else
                                   c.genCleanUp(loc, builder);
                               },
                           },
                           cleanUp);
    return postponed;
  }
  std::variant<CopyIn, ExprAssociate> cleanUp;
};

/// Structure representing a prepared dummy argument.
/// It holds the value to be passed in the call and any related
/// clean-ups to be done after the call.
struct PreparedDummyArgument {
  void pushCopyInCleanUp(mlir::Value tempBox, mlir::Value wasCopied,
                         mlir::Value copyBackVar) {
    cleanups.emplace_back(
        CallCleanUp{CallCleanUp::CopyIn{tempBox, wasCopied, copyBackVar}});
  }
  void pushExprAssociateCleanUp(mlir::Value tempVar, mlir::Value wasCopied) {
````
- **L1129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](CopyIn &c) { c.genCleanUp(loc, builder); },`.
  **L1129 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](CopyIn &c) { c.genCleanUp(loc, builder); },`。
- **L1130 EN**: Starts a function, method, lambda, or structured scope: `[&](ExprAssociate &c) {`.
  **L1130 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](ExprAssociate &c) {`。
- **L1131 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1131 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1132 EN**: Executes a standalone statement or declaration: `postponed = CallCleanUp{c};`.
  **L1132 CN**: 执行一条独立语句或声明：`postponed = CallCleanUp{c};`。
- **L1133 EN**: Transitions from the previous branch into the alternative path.
  **L1133 CN**: 从前一个分支过渡到备选路径。
- **L1134 EN**: Executes a call or declaration centered on `c.genCleanUp`.
  **L1134 CN**: 执行以 `c.genCleanUp` 为核心的调用或声明。
- **L1135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L1135 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L1136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L1136 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L1137 EN**: Executes a standalone statement or declaration: `cleanUp);`.
  **L1137 CN**: 执行一条独立语句或声明：`cleanUp);`。
- **L1138 EN**: Returns from the current function with `postponed`.
  **L1138 CN**: 以 `postponed` 从当前函数返回。
- **L1139 EN**: Closes the current lexical scope or compound statement.
  **L1139 CN**: 结束当前词法作用域或复合语句块。
- **L1140 EN**: Executes a standalone statement or declaration: `std::variant<CopyIn, ExprAssociate> cleanUp;`.
  **L1140 CN**: 执行一条独立语句或声明：`std::variant<CopyIn, ExprAssociate> cleanUp;`。
- **L1141 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1141 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1142 EN**: Blank line separating nearby declarations or logic blocks.
  **L1142 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1143 EN**: Comment explains nearby logic, intent, or metadata: `Structure representing a prepared dummy argument.`.
  **L1143 CN**: 注释说明附近代码的逻辑、意图或元数据：`Structure representing a prepared dummy argument.`。
- **L1144 EN**: Comment explains nearby logic, intent, or metadata: `It holds the value to be passed in the call and any related`.
  **L1144 CN**: 注释说明附近代码的逻辑、意图或元数据：`It holds the value to be passed in the call and any related`。
- **L1145 EN**: Comment explains nearby logic, intent, or metadata: `clean-ups to be done after the call.`.
  **L1145 CN**: 注释说明附近代码的逻辑、意图或元数据：`clean-ups to be done after the call.`。
- **L1146 EN**: Declares struct `PreparedDummyArgument`.
  **L1146 CN**: 声明 struct `PreparedDummyArgument`。
- **L1147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void pushCopyInCleanUp(mlir::Value tempBox, mlir::Value wasCopied,`.
  **L1147 CN**: 继续一个多行参数列表、初始化器或聚合项：`void pushCopyInCleanUp(mlir::Value tempBox, mlir::Value wasCopied,`。
- **L1148 EN**: Continues the surrounding expression or declaration: `mlir::Value copyBackVar) {`.
  **L1148 CN**: 继续构造周围的表达式或声明：`mlir::Value copyBackVar) {`。
- **L1149 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L1149 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L1150 EN**: Executes a standalone statement or declaration: `CallCleanUp{CallCleanUp::CopyIn{tempBox, wasCopied, copyBackVar}});`.
  **L1150 CN**: 执行一条独立语句或声明：`CallCleanUp{CallCleanUp::CopyIn{tempBox, wasCopied, copyBackVar}});`。
- **L1151 EN**: Closes the current lexical scope or compound statement.
  **L1151 CN**: 结束当前词法作用域或复合语句块。
- **L1152 EN**: Starts a function, method, lambda, or structured scope: `void pushExprAssociateCleanUp(mlir::Value tempVar, mlir::Value wasCopied) {`.
  **L1152 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void pushExprAssociateCleanUp(mlir::Value tempVar, mlir::Value wasCopied) {`。

### Lines 1153-1176

````cpp
    cleanups.emplace_back(
        CallCleanUp{CallCleanUp::ExprAssociate{tempVar, wasCopied}});
  }
  void pushExprAssociateCleanUp(hlfir::AssociateOp associate) {
    mlir::Value hlfirBase = associate.getBase();
    mlir::Value firBase = associate.getFirBase();
    cleanups.emplace_back(CallCleanUp{CallCleanUp::ExprAssociate{
        hlfir::mayHaveAllocatableComponent(hlfirBase.getType()) ? hlfirBase
                                                                : firBase,
        associate.getMustFreeStrorageFlag()}});
  }

  mlir::Value dummy;
  // NOTE: the clean-ups are executed in reverse order.
  llvm::SmallVector<CallCleanUp, 2> cleanups;
};

/// Structure to help conditionally preparing a dummy argument based
/// on the actual argument presence.
/// It helps "wrapping" the dummy and the clean-up information in
/// an if (present) {...}:
///
///  %conditionallyPrepared = fir.if (%present) {
///    fir.result %preparedDummy
````
- **L1153 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L1153 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L1154 EN**: Executes a standalone statement or declaration: `CallCleanUp{CallCleanUp::ExprAssociate{tempVar, wasCopied}});`.
  **L1154 CN**: 执行一条独立语句或声明：`CallCleanUp{CallCleanUp::ExprAssociate{tempVar, wasCopied}});`。
- **L1155 EN**: Closes the current lexical scope or compound statement.
  **L1155 CN**: 结束当前词法作用域或复合语句块。
- **L1156 EN**: Starts a function, method, lambda, or structured scope: `void pushExprAssociateCleanUp(hlfir::AssociateOp associate) {`.
  **L1156 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void pushExprAssociateCleanUp(hlfir::AssociateOp associate) {`。
- **L1157 EN**: Initializes variable `hlfirBase` from the right-hand expression.
  **L1157 CN**: 使用右侧表达式初始化变量 `hlfirBase`。
- **L1158 EN**: Initializes variable `firBase` from the right-hand expression.
  **L1158 CN**: 使用右侧表达式初始化变量 `firBase`。
- **L1159 EN**: Starts a function, method, lambda, or structured scope: `cleanups.emplace_back(CallCleanUp{CallCleanUp::ExprAssociate{`.
  **L1159 CN**: 开始一个函数、方法、lambda 或结构化作用域：`cleanups.emplace_back(CallCleanUp{CallCleanUp::ExprAssociate{`。
- **L1160 EN**: Continues logic associated with callable symbol `mayHaveAllocatableComponent`.
  **L1160 CN**: 继续与可调用符号 `mayHaveAllocatableComponent` 相关的逻辑。
- **L1161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: firBase,`.
  **L1161 CN**: 继续一个多行参数列表、初始化器或聚合项：`: firBase,`。
- **L1162 EN**: Executes a call or declaration centered on `associate.getMustFreeStrorageFlag`.
  **L1162 CN**: 执行以 `associate.getMustFreeStrorageFlag` 为核心的调用或声明。
- **L1163 EN**: Closes the current lexical scope or compound statement.
  **L1163 CN**: 结束当前词法作用域或复合语句块。
- **L1164 EN**: Blank line separating nearby declarations or logic blocks.
  **L1164 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1165 EN**: Executes a standalone statement or declaration: `mlir::Value dummy;`.
  **L1165 CN**: 执行一条独立语句或声明：`mlir::Value dummy;`。
- **L1166 EN**: Comment highlights an implementation note: `NOTE: the clean-ups are executed in reverse order.`.
  **L1166 CN**: 注释强调了一条实现说明：`NOTE: the clean-ups are executed in reverse order.`。
- **L1167 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<CallCleanUp, 2> cleanups;`.
  **L1167 CN**: 执行一条独立语句或声明：`llvm::SmallVector<CallCleanUp, 2> cleanups;`。
- **L1168 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1168 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1169 EN**: Blank line separating nearby declarations or logic blocks.
  **L1169 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1170 EN**: Comment explains nearby logic, intent, or metadata: `Structure to help conditionally preparing a dummy argument based`.
  **L1170 CN**: 注释说明附近代码的逻辑、意图或元数据：`Structure to help conditionally preparing a dummy argument based`。
- **L1171 EN**: Comment explains nearby logic, intent, or metadata: `on the actual argument presence.`.
  **L1171 CN**: 注释说明附近代码的逻辑、意图或元数据：`on the actual argument presence.`。
- **L1172 EN**: Comment explains nearby logic, intent, or metadata: `It helps "wrapping" the dummy and the clean-up information in`.
  **L1172 CN**: 注释说明附近代码的逻辑、意图或元数据：`It helps "wrapping" the dummy and the clean-up information in`。
- **L1173 EN**: Comment explains nearby logic, intent, or metadata: `an if (present) {...}:`.
  **L1173 CN**: 注释说明附近代码的逻辑、意图或元数据：`an if (present) {...}:`。
- **L1174 EN**: Separator comment used for visual grouping.
  **L1174 CN**: 用于视觉分组的分隔注释。
- **L1175 EN**: Comment explains nearby logic, intent, or metadata: `%conditionallyPrepared = fir.if (%present) {`.
  **L1175 CN**: 注释说明附近代码的逻辑、意图或元数据：`%conditionallyPrepared = fir.if (%present) {`。
- **L1176 EN**: Comment explains nearby logic, intent, or metadata: `fir.result %preparedDummy`.
  **L1176 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.result %preparedDummy`。

### Lines 1177-1200

````cpp
///  } else {
///    fir.result %absent
///  }
///
struct ConditionallyPreparedDummy {
  /// Create ConditionallyPreparedDummy from a preparedDummy that must
  /// be wrapped in a fir.if.
  ConditionallyPreparedDummy(PreparedDummyArgument &preparedDummy) {
    thenResultValues.push_back(preparedDummy.dummy);
    for (const CallCleanUp &c : preparedDummy.cleanups) {
      if (const auto *copyInCleanUp =
              std::get_if<CallCleanUp::CopyIn>(&c.cleanUp)) {
        thenResultValues.push_back(copyInCleanUp->wasCopied);
        if (copyInCleanUp->copyBackVar)
          thenResultValues.push_back(copyInCleanUp->copyBackVar);
      } else {
        const auto &exprAssociate =
            std::get<CallCleanUp::ExprAssociate>(c.cleanUp);
        thenResultValues.push_back(exprAssociate.tempVar);
        thenResultValues.push_back(exprAssociate.mustFree);
      }
    }
  }

````
- **L1177 EN**: Comment explains nearby logic, intent, or metadata: `} else {`.
  **L1177 CN**: 注释说明附近代码的逻辑、意图或元数据：`} else {`。
- **L1178 EN**: Comment explains nearby logic, intent, or metadata: `fir.result %absent`.
  **L1178 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.result %absent`。
- **L1179 EN**: Comment explains nearby logic, intent, or metadata: `}`.
  **L1179 CN**: 注释说明附近代码的逻辑、意图或元数据：`}`。
- **L1180 EN**: Separator comment used for visual grouping.
  **L1180 CN**: 用于视觉分组的分隔注释。
- **L1181 EN**: Declares struct `ConditionallyPreparedDummy`.
  **L1181 CN**: 声明 struct `ConditionallyPreparedDummy`。
- **L1182 EN**: Comment explains nearby logic, intent, or metadata: `Create ConditionallyPreparedDummy from a preparedDummy that must`.
  **L1182 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create ConditionallyPreparedDummy from a preparedDummy that must`。
- **L1183 EN**: Comment explains nearby logic, intent, or metadata: `be wrapped in a fir.if.`.
  **L1183 CN**: 注释说明附近代码的逻辑、意图或元数据：`be wrapped in a fir.if.`。
- **L1184 EN**: Starts a function, method, lambda, or structured scope: `ConditionallyPreparedDummy(PreparedDummyArgument &preparedDummy) {`.
  **L1184 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ConditionallyPreparedDummy(PreparedDummyArgument &preparedDummy) {`。
- **L1185 EN**: Executes a call or declaration centered on `thenResultValues.push_back`.
  **L1185 CN**: 执行以 `thenResultValues.push_back` 为核心的调用或声明。
- **L1186 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1186 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1187 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1187 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1188 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<CallCleanUp::CopyIn>(&c.cleanUp)) {`.
  **L1188 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<CallCleanUp::CopyIn>(&c.cleanUp)) {`。
- **L1189 EN**: Executes a call or declaration centered on `thenResultValues.push_back`.
  **L1189 CN**: 执行以 `thenResultValues.push_back` 为核心的调用或声明。
- **L1190 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1190 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1191 EN**: Executes a call or declaration centered on `thenResultValues.push_back`.
  **L1191 CN**: 执行以 `thenResultValues.push_back` 为核心的调用或声明。
- **L1192 EN**: Transitions from the previous branch into the alternative path.
  **L1192 CN**: 从前一个分支过渡到备选路径。
- **L1193 EN**: Continues the surrounding expression or declaration: `const auto &exprAssociate =`.
  **L1193 CN**: 继续构造周围的表达式或声明：`const auto &exprAssociate =`。
- **L1194 EN**: Executes a call or declaration centered on `std::get<CallCleanUp::ExprAssociate>`.
  **L1194 CN**: 执行以 `std::get<CallCleanUp::ExprAssociate>` 为核心的调用或声明。
- **L1195 EN**: Executes a call or declaration centered on `thenResultValues.push_back`.
  **L1195 CN**: 执行以 `thenResultValues.push_back` 为核心的调用或声明。
- **L1196 EN**: Executes a call or declaration centered on `thenResultValues.push_back`.
  **L1196 CN**: 执行以 `thenResultValues.push_back` 为核心的调用或声明。
- **L1197 EN**: Closes the current lexical scope or compound statement.
  **L1197 CN**: 结束当前词法作用域或复合语句块。
- **L1198 EN**: Closes the current lexical scope or compound statement.
  **L1198 CN**: 结束当前词法作用域或复合语句块。
- **L1199 EN**: Closes the current lexical scope or compound statement.
  **L1199 CN**: 结束当前词法作用域或复合语句块。
- **L1200 EN**: Blank line separating nearby declarations or logic blocks.
  **L1200 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1201-1224

````cpp
  /// Get the result types of the wrapping fir.if that must be created.
  llvm::SmallVector<mlir::Type> getIfResulTypes() const {
    llvm::SmallVector<mlir::Type> types;
    for (mlir::Value res : thenResultValues)
      types.push_back(res.getType());
    return types;
  }

  /// Generate the "fir.result %preparedDummy" in the then branch of the
  /// wrapping fir.if.
  void genThenResult(mlir::Location loc, fir::FirOpBuilder &builder) const {
    fir::ResultOp::create(builder, loc, thenResultValues);
  }

  /// Generate the "fir.result %absent" in the else branch of the
  /// wrapping fir.if.
  void genElseResult(mlir::Location loc, fir::FirOpBuilder &builder) const {
    llvm::SmallVector<mlir::Value> elseResultValues;
    mlir::Type i1Type = builder.getI1Type();
    for (mlir::Value res : thenResultValues) {
      mlir::Type type = res.getType();
      if (type == i1Type)
        elseResultValues.push_back(builder.createBool(loc, false));
      else
````
- **L1201 EN**: Comment explains nearby logic, intent, or metadata: `Get the result types of the wrapping fir.if that must be created.`.
  **L1201 CN**: 注释说明附近代码的逻辑、意图或元数据：`Get the result types of the wrapping fir.if that must be created.`。
- **L1202 EN**: Starts a function, method, lambda, or structured scope: `llvm::SmallVector<mlir::Type> getIfResulTypes() const {`.
  **L1202 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::SmallVector<mlir::Type> getIfResulTypes() const {`。
- **L1203 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Type> types;`.
  **L1203 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Type> types;`。
- **L1204 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1204 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1205 EN**: Executes a call or declaration centered on `types.push_back`.
  **L1205 CN**: 执行以 `types.push_back` 为核心的调用或声明。
- **L1206 EN**: Returns from the current function with `types`.
  **L1206 CN**: 以 `types` 从当前函数返回。
- **L1207 EN**: Closes the current lexical scope or compound statement.
  **L1207 CN**: 结束当前词法作用域或复合语句块。
- **L1208 EN**: Blank line separating nearby declarations or logic blocks.
  **L1208 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1209 EN**: Comment explains nearby logic, intent, or metadata: `Generate the "fir.result %preparedDummy" in the then branch of the`.
  **L1209 CN**: 注释说明附近代码的逻辑、意图或元数据：`Generate the "fir.result %preparedDummy" in the then branch of the`。
- **L1210 EN**: Comment explains nearby logic, intent, or metadata: `wrapping fir.if.`.
  **L1210 CN**: 注释说明附近代码的逻辑、意图或元数据：`wrapping fir.if.`。
- **L1211 EN**: Starts a function, method, lambda, or structured scope: `void genThenResult(mlir::Location loc, fir::FirOpBuilder &builder) const {`.
  **L1211 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void genThenResult(mlir::Location loc, fir::FirOpBuilder &builder) const {`。
- **L1212 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L1212 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L1213 EN**: Closes the current lexical scope or compound statement.
  **L1213 CN**: 结束当前词法作用域或复合语句块。
- **L1214 EN**: Blank line separating nearby declarations or logic blocks.
  **L1214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1215 EN**: Comment explains nearby logic, intent, or metadata: `Generate the "fir.result %absent" in the else branch of the`.
  **L1215 CN**: 注释说明附近代码的逻辑、意图或元数据：`Generate the "fir.result %absent" in the else branch of the`。
- **L1216 EN**: Comment explains nearby logic, intent, or metadata: `wrapping fir.if.`.
  **L1216 CN**: 注释说明附近代码的逻辑、意图或元数据：`wrapping fir.if.`。
- **L1217 EN**: Starts a function, method, lambda, or structured scope: `void genElseResult(mlir::Location loc, fir::FirOpBuilder &builder) const {`.
  **L1217 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void genElseResult(mlir::Location loc, fir::FirOpBuilder &builder) const {`。
- **L1218 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> elseResultValues;`.
  **L1218 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> elseResultValues;`。
- **L1219 EN**: Initializes variable `i1Type` from the right-hand expression.
  **L1219 CN**: 使用右侧表达式初始化变量 `i1Type`。
- **L1220 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1220 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1221 EN**: Initializes variable `type` from the right-hand expression.
  **L1221 CN**: 使用右侧表达式初始化变量 `type`。
- **L1222 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1222 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1223 EN**: Starts the alternative branch of the preceding conditional.
  **L1223 CN**: 开始前一个条件语句的备选分支。
- **L1224 EN**: Transitions from the previous branch into the alternative path.
  **L1224 CN**: 从前一个分支过渡到备选路径。

### Lines 1225-1248

````cpp
        elseResultValues.push_back(builder.genAbsentOp(loc, type));
    }
    fir::ResultOp::create(builder, loc, elseResultValues);
  }

  /// Once the fir.if has been created, get the resulting %conditionallyPrepared
  /// dummy argument.
  PreparedDummyArgument
  getPreparedDummy(fir::IfOp ifOp,
                   const PreparedDummyArgument &unconditionalDummy) {
    PreparedDummyArgument preparedDummy;
    preparedDummy.dummy = ifOp.getResults()[0];
    for (const CallCleanUp &c : unconditionalDummy.cleanups) {
      if (const auto *copyInCleanUp =
              std::get_if<CallCleanUp::CopyIn>(&c.cleanUp)) {
        mlir::Value copyBackVar;
        if (copyInCleanUp->copyBackVar)
          copyBackVar = ifOp.getResults().back();
        // tempBox is an hlfir.copy_in argument created outside of the
        // fir.if region. It needs not to be threaded as a fir.if result.
        preparedDummy.pushCopyInCleanUp(copyInCleanUp->tempBox,
                                        ifOp.getResults()[1], copyBackVar);
      } else {
        preparedDummy.pushExprAssociateCleanUp(ifOp.getResults()[1],
````
- **L1225 EN**: Starts the alternative branch of the preceding conditional.
  **L1225 CN**: 开始前一个条件语句的备选分支。
- **L1226 EN**: Closes the current lexical scope or compound statement.
  **L1226 CN**: 结束当前词法作用域或复合语句块。
- **L1227 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L1227 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L1228 EN**: Closes the current lexical scope or compound statement.
  **L1228 CN**: 结束当前词法作用域或复合语句块。
- **L1229 EN**: Blank line separating nearby declarations or logic blocks.
  **L1229 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1230 EN**: Comment explains nearby logic, intent, or metadata: `Once the fir.if has been created, get the resulting %conditionallyPrepared`.
  **L1230 CN**: 注释说明附近代码的逻辑、意图或元数据：`Once the fir.if has been created, get the resulting %conditionallyPrepared`。
- **L1231 EN**: Comment explains nearby logic, intent, or metadata: `dummy argument.`.
  **L1231 CN**: 注释说明附近代码的逻辑、意图或元数据：`dummy argument.`。
- **L1232 EN**: Continues the surrounding expression or declaration: `PreparedDummyArgument`.
  **L1232 CN**: 继续构造周围的表达式或声明：`PreparedDummyArgument`。
- **L1233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getPreparedDummy(fir::IfOp ifOp,`.
  **L1233 CN**: 继续一个多行参数列表、初始化器或聚合项：`getPreparedDummy(fir::IfOp ifOp,`。
- **L1234 EN**: Continues the surrounding expression or declaration: `const PreparedDummyArgument &unconditionalDummy) {`.
  **L1234 CN**: 继续构造周围的表达式或声明：`const PreparedDummyArgument &unconditionalDummy) {`。
- **L1235 EN**: Executes a standalone statement or declaration: `PreparedDummyArgument preparedDummy;`.
  **L1235 CN**: 执行一条独立语句或声明：`PreparedDummyArgument preparedDummy;`。
- **L1236 EN**: Executes a call or declaration centered on `ifOp.getResults`.
  **L1236 CN**: 执行以 `ifOp.getResults` 为核心的调用或声明。
- **L1237 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1237 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1238 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1238 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1239 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<CallCleanUp::CopyIn>(&c.cleanUp)) {`.
  **L1239 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<CallCleanUp::CopyIn>(&c.cleanUp)) {`。
- **L1240 EN**: Executes a standalone statement or declaration: `mlir::Value copyBackVar;`.
  **L1240 CN**: 执行一条独立语句或声明：`mlir::Value copyBackVar;`。
- **L1241 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1241 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1242 EN**: Executes a call or declaration centered on `ifOp.getResults`.
  **L1242 CN**: 执行以 `ifOp.getResults` 为核心的调用或声明。
- **L1243 EN**: Comment explains nearby logic, intent, or metadata: `tempBox is an hlfir.copy_in argument created outside of the`.
  **L1243 CN**: 注释说明附近代码的逻辑、意图或元数据：`tempBox is an hlfir.copy_in argument created outside of the`。
- **L1244 EN**: Comment explains nearby logic, intent, or metadata: `fir.if region. It needs not to be threaded as a fir.if result.`.
  **L1244 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.if region. It needs not to be threaded as a fir.if result.`。
- **L1245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `preparedDummy.pushCopyInCleanUp(copyInCleanUp->tempBox,`.
  **L1245 CN**: 继续一个多行参数列表、初始化器或聚合项：`preparedDummy.pushCopyInCleanUp(copyInCleanUp->tempBox,`。
- **L1246 EN**: Executes a call or declaration centered on `ifOp.getResults`.
  **L1246 CN**: 执行以 `ifOp.getResults` 为核心的调用或声明。
- **L1247 EN**: Transitions from the previous branch into the alternative path.
  **L1247 CN**: 从前一个分支过渡到备选路径。
- **L1248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `preparedDummy.pushExprAssociateCleanUp(ifOp.getResults()[1],`.
  **L1248 CN**: 继续一个多行参数列表、初始化器或聚合项：`preparedDummy.pushExprAssociateCleanUp(ifOp.getResults()[1],`。

### Lines 1249-1272

````cpp
                                               ifOp.getResults()[2]);
      }
    }
    return preparedDummy;
  }

  llvm::SmallVector<mlir::Value> thenResultValues;
};
} // namespace

/// Fix-up the fact that it is supported to pass a character procedure
/// designator to a non character procedure dummy procedure and vice-versa, even
/// in case of explicit interface. Uglier cases where an object is passed as
/// procedure designator or vice versa are handled only for implicit interfaces
/// (refused by semantics with explicit interface), and handled with a funcOp
/// cast like other implicit interface mismatches.
static hlfir::Entity fixProcedureDummyMismatch(mlir::Location loc,
                                               fir::FirOpBuilder &builder,
                                               hlfir::Entity actual,
                                               mlir::Type dummyType) {
  if (mlir::isa<fir::BoxProcType>(actual.getType()) &&
      fir::isCharacterProcedureTuple(dummyType)) {
    mlir::Value length =
        fir::UndefOp::create(builder, loc, builder.getCharacterLengthType());
````
- **L1249 EN**: Executes a call or declaration centered on `ifOp.getResults`.
  **L1249 CN**: 执行以 `ifOp.getResults` 为核心的调用或声明。
- **L1250 EN**: Closes the current lexical scope or compound statement.
  **L1250 CN**: 结束当前词法作用域或复合语句块。
- **L1251 EN**: Closes the current lexical scope or compound statement.
  **L1251 CN**: 结束当前词法作用域或复合语句块。
- **L1252 EN**: Returns from the current function with `preparedDummy`.
  **L1252 CN**: 以 `preparedDummy` 从当前函数返回。
- **L1253 EN**: Closes the current lexical scope or compound statement.
  **L1253 CN**: 结束当前词法作用域或复合语句块。
- **L1254 EN**: Blank line separating nearby declarations or logic blocks.
  **L1254 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1255 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> thenResultValues;`.
  **L1255 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> thenResultValues;`。
- **L1256 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1256 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1257 EN**: Closes a namespace scope with a trailing comment: `} // namespace`.
  **L1257 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L1258 EN**: Blank line separating nearby declarations or logic blocks.
  **L1258 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1259 EN**: Comment explains nearby logic, intent, or metadata: `Fix-up the fact that it is supported to pass a character procedure`.
  **L1259 CN**: 注释说明附近代码的逻辑、意图或元数据：`Fix-up the fact that it is supported to pass a character procedure`。
- **L1260 EN**: Comment explains nearby logic, intent, or metadata: `designator to a non character procedure dummy procedure and vice-versa, even`.
  **L1260 CN**: 注释说明附近代码的逻辑、意图或元数据：`designator to a non character procedure dummy procedure and vice-versa, even`。
- **L1261 EN**: Comment explains nearby logic, intent, or metadata: `in case of explicit interface. Uglier cases where an object is passed as`.
  **L1261 CN**: 注释说明附近代码的逻辑、意图或元数据：`in case of explicit interface. Uglier cases where an object is passed as`。
- **L1262 EN**: Comment explains nearby logic, intent, or metadata: `procedure designator or vice versa are handled only for implicit interfaces`.
  **L1262 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure designator or vice versa are handled only for implicit interfaces`。
- **L1263 EN**: Comment explains nearby logic, intent, or metadata: `(refused by semantics with explicit interface), and handled with a funcOp`.
  **L1263 CN**: 注释说明附近代码的逻辑、意图或元数据：`(refused by semantics with explicit interface), and handled with a funcOp`。
- **L1264 EN**: Comment explains nearby logic, intent, or metadata: `cast like other implicit interface mismatches.`.
  **L1264 CN**: 注释说明附近代码的逻辑、意图或元数据：`cast like other implicit interface mismatches.`。
- **L1265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static hlfir::Entity fixProcedureDummyMismatch(mlir::Location loc,`.
  **L1265 CN**: 继续一个多行参数列表、初始化器或聚合项：`static hlfir::Entity fixProcedureDummyMismatch(mlir::Location loc,`。
- **L1266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::FirOpBuilder &builder,`.
  **L1266 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::FirOpBuilder &builder,`。
- **L1267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `hlfir::Entity actual,`.
  **L1267 CN**: 继续一个多行参数列表、初始化器或聚合项：`hlfir::Entity actual,`。
- **L1268 EN**: Continues the surrounding expression or declaration: `mlir::Type dummyType) {`.
  **L1268 CN**: 继续构造周围的表达式或声明：`mlir::Type dummyType) {`。
- **L1269 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1269 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1270 EN**: Starts a function, method, lambda, or structured scope: `fir::isCharacterProcedureTuple(dummyType)) {`.
  **L1270 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::isCharacterProcedureTuple(dummyType)) {`。
- **L1271 EN**: Continues the surrounding expression or declaration: `mlir::Value length =`.
  **L1271 CN**: 继续构造周围的表达式或声明：`mlir::Value length =`。
- **L1272 EN**: Executes a call or declaration centered on `fir::UndefOp::create`.
  **L1272 CN**: 执行以 `fir::UndefOp::create` 为核心的调用或声明。

### Lines 1273-1296

````cpp
    mlir::Value tuple = fir::factory::createCharacterProcedureTuple(
        builder, loc, dummyType, actual, length);
    return hlfir::Entity{tuple};
  }
  assert(fir::isCharacterProcedureTuple(actual.getType()) &&
         mlir::isa<fir::BoxProcType>(dummyType) &&
         "unsupported dummy procedure mismatch with the actual argument");
  mlir::Value boxProc = fir::factory::extractCharacterProcedureTuple(
                            builder, loc, actual, /*openBoxProc=*/false)
                            .first;
  return hlfir::Entity{boxProc};
}

mlir::Value static getZeroLowerBounds(mlir::Location loc,
                                      fir::FirOpBuilder &builder,
                                      hlfir::Entity entity) {
  assert(!entity.isAssumedRank() &&
         "assumed-rank must use fir.rebox_assumed_rank");
  if (entity.getRank() < 1)
    return {};
  mlir::Value zero =
      builder.createIntegerConstant(loc, builder.getIndexType(), 0);
  llvm::SmallVector<mlir::Value> lowerBounds(entity.getRank(), zero);
  return builder.genShift(loc, lowerBounds);
````
- **L1273 EN**: Continues logic associated with callable symbol `createCharacterProcedureTuple`.
  **L1273 CN**: 继续与可调用符号 `createCharacterProcedureTuple` 相关的逻辑。
- **L1274 EN**: Executes a standalone statement or declaration: `builder, loc, dummyType, actual, length);`.
  **L1274 CN**: 执行一条独立语句或声明：`builder, loc, dummyType, actual, length);`。
- **L1275 EN**: Returns from the current function with `hlfir::Entity{tuple}`.
  **L1275 CN**: 以 `hlfir::Entity{tuple}` 从当前函数返回。
- **L1276 EN**: Closes the current lexical scope or compound statement.
  **L1276 CN**: 结束当前词法作用域或复合语句块。
- **L1277 EN**: Checks an internal invariant in debug builds.
  **L1277 CN**: 在调试构建中检查内部不变式。
- **L1278 EN**: Continues logic associated with callable symbol `BoxProcType>`.
  **L1278 CN**: 继续与可调用符号 `BoxProcType>` 相关的逻辑。
- **L1279 EN**: Executes a standalone statement or declaration: `"unsupported dummy procedure mismatch with the actual argument");`.
  **L1279 CN**: 执行一条独立语句或声明：`"unsupported dummy procedure mismatch with the actual argument");`。
- **L1280 EN**: Continues logic associated with callable symbol `extractCharacterProcedureTuple`.
  **L1280 CN**: 继续与可调用符号 `extractCharacterProcedureTuple` 相关的逻辑。
- **L1281 EN**: Continues the surrounding expression or declaration: `builder, loc, actual, /*openBoxProc=*/false)`.
  **L1281 CN**: 继续构造周围的表达式或声明：`builder, loc, actual, /*openBoxProc=*/false)`。
- **L1282 EN**: Executes a standalone statement or declaration: `.first;`.
  **L1282 CN**: 执行一条独立语句或声明：`.first;`。
- **L1283 EN**: Returns from the current function with `hlfir::Entity{boxProc}`.
  **L1283 CN**: 以 `hlfir::Entity{boxProc}` 从当前函数返回。
- **L1284 EN**: Closes the current lexical scope or compound statement.
  **L1284 CN**: 结束当前词法作用域或复合语句块。
- **L1285 EN**: Blank line separating nearby declarations or logic blocks.
  **L1285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value static getZeroLowerBounds(mlir::Location loc,`.
  **L1286 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value static getZeroLowerBounds(mlir::Location loc,`。
- **L1287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::FirOpBuilder &builder,`.
  **L1287 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::FirOpBuilder &builder,`。
- **L1288 EN**: Continues the surrounding expression or declaration: `hlfir::Entity entity) {`.
  **L1288 CN**: 继续构造周围的表达式或声明：`hlfir::Entity entity) {`。
- **L1289 EN**: Checks an internal invariant in debug builds.
  **L1289 CN**: 在调试构建中检查内部不变式。
- **L1290 EN**: Executes a standalone statement or declaration: `"assumed-rank must use fir.rebox_assumed_rank");`.
  **L1290 CN**: 执行一条独立语句或声明：`"assumed-rank must use fir.rebox_assumed_rank");`。
- **L1291 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1291 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1292 EN**: Returns from the current function with `{}`.
  **L1292 CN**: 以 `{}` 从当前函数返回。
- **L1293 EN**: Continues the surrounding expression or declaration: `mlir::Value zero =`.
  **L1293 CN**: 继续构造周围的表达式或声明：`mlir::Value zero =`。
- **L1294 EN**: Executes a call or declaration centered on `builder.createIntegerConstant`.
  **L1294 CN**: 执行以 `builder.createIntegerConstant` 为核心的调用或声明。
- **L1295 EN**: Executes a call or declaration centered on `lowerBounds`.
  **L1295 CN**: 执行以 `lowerBounds` 为核心的调用或声明。
- **L1296 EN**: Returns from the current function with `builder.genShift(loc, lowerBounds)`.
  **L1296 CN**: 以 `builder.genShift(loc, lowerBounds)` 从当前函数返回。

### Lines 1297-1320

````cpp
}

static bool isParameterObjectOrSubObject(hlfir::Entity entity) {
  mlir::Value base = entity;
  bool foundParameter = false;
  while (mlir::Operation *op = base ? base.getDefiningOp() : nullptr) {
    base =
        llvm::TypeSwitch<mlir::Operation *, mlir::Value>(op)
            .Case<hlfir::DeclareOp>([&](auto declare) -> mlir::Value {
              foundParameter |= hlfir::Entity{declare}.isParameter();
              return foundParameter ? mlir::Value{} : declare.getMemref();
            })
            .Case<hlfir::DesignateOp, hlfir::ParentComponentOp, fir::EmboxOp>(
                [&](auto op) -> mlir::Value { return op.getMemref(); })
            .Case<fir::ReboxOp>(
                [&](auto rebox) -> mlir::Value { return rebox.getBox(); })
            .Case<fir::ConvertOp>(
                [&](auto convert) -> mlir::Value { return convert.getValue(); })
            .Default([](mlir::Operation *) -> mlir::Value { return nullptr; });
  }
  return foundParameter;
}

/// When dummy is not ALLOCATABLE, POINTER and is not passed in register,
````
- **L1297 EN**: Closes the current lexical scope or compound statement.
  **L1297 CN**: 结束当前词法作用域或复合语句块。
- **L1298 EN**: Blank line separating nearby declarations or logic blocks.
  **L1298 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1299 EN**: Starts a function, method, lambda, or structured scope: `static bool isParameterObjectOrSubObject(hlfir::Entity entity) {`.
  **L1299 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isParameterObjectOrSubObject(hlfir::Entity entity) {`。
- **L1300 EN**: Initializes variable `base` from the right-hand expression.
  **L1300 CN**: 使用右侧表达式初始化变量 `base`。
- **L1301 EN**: Initializes variable `foundParameter` from the right-hand expression.
  **L1301 CN**: 使用右侧表达式初始化变量 `foundParameter`。
- **L1302 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L1302 CN**: 开始 `while` 控制流语句并计算其条件。
- **L1303 EN**: Continues the surrounding expression or declaration: `base =`.
  **L1303 CN**: 继续构造周围的表达式或声明：`base =`。
- **L1304 EN**: Continues logic associated with callable symbol `Value>`.
  **L1304 CN**: 继续与可调用符号 `Value>` 相关的逻辑。
- **L1305 EN**: Starts a function, method, lambda, or structured scope: `.Case<hlfir::DeclareOp>([&](auto declare) -> mlir::Value {`.
  **L1305 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.Case<hlfir::DeclareOp>([&](auto declare) -> mlir::Value {`。
- **L1306 EN**: Executes a call or declaration centered on `hlfir::Entity{declare}.isParameter`.
  **L1306 CN**: 执行以 `hlfir::Entity{declare}.isParameter` 为核心的调用或声明。
- **L1307 EN**: Returns from the current function with `foundParameter ? mlir::Value{} : declare.getMemref()`.
  **L1307 CN**: 以 `foundParameter ? mlir::Value{} : declare.getMemref()` 从当前函数返回。
- **L1308 EN**: Continues the surrounding expression or declaration: `})`.
  **L1308 CN**: 继续构造周围的表达式或声明：`})`。
- **L1309 EN**: Continues logic associated with callable symbol `EmboxOp>`.
  **L1309 CN**: 继续与可调用符号 `EmboxOp>` 相关的逻辑。
- **L1310 EN**: Continues logic associated with callable symbol `getMemref`.
  **L1310 CN**: 继续与可调用符号 `getMemref` 相关的逻辑。
- **L1311 EN**: Continues logic associated with callable symbol `ReboxOp>`.
  **L1311 CN**: 继续与可调用符号 `ReboxOp>` 相关的逻辑。
- **L1312 EN**: Continues logic associated with callable symbol `getBox`.
  **L1312 CN**: 继续与可调用符号 `getBox` 相关的逻辑。
- **L1313 EN**: Continues logic associated with callable symbol `ConvertOp>`.
  **L1313 CN**: 继续与可调用符号 `ConvertOp>` 相关的逻辑。
- **L1314 EN**: Continues logic associated with callable symbol `getValue`.
  **L1314 CN**: 继续与可调用符号 `getValue` 相关的逻辑。
- **L1315 EN**: Executes a call or declaration centered on `.Default`.
  **L1315 CN**: 执行以 `.Default` 为核心的调用或声明。
- **L1316 EN**: Closes the current lexical scope or compound statement.
  **L1316 CN**: 结束当前词法作用域或复合语句块。
- **L1317 EN**: Returns from the current function with `foundParameter`.
  **L1317 CN**: 以 `foundParameter` 从当前函数返回。
- **L1318 EN**: Closes the current lexical scope or compound statement.
  **L1318 CN**: 结束当前词法作用域或复合语句块。
- **L1319 EN**: Blank line separating nearby declarations or logic blocks.
  **L1319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1320 EN**: Comment explains nearby logic, intent, or metadata: `When dummy is not ALLOCATABLE, POINTER and is not passed in register,`.
  **L1320 CN**: 注释说明附近代码的逻辑、意图或元数据：`When dummy is not ALLOCATABLE, POINTER and is not passed in register,`。

### Lines 1321-1344

````cpp
/// prepare the actual argument according to the interface. Do as needed:
/// - address element if this is an array argument in an elemental call.
/// - set dynamic type to the dummy type if the dummy is not polymorphic.
/// - copy-in into contiguous variable if the dummy must be contiguous
/// - copy into a temporary if the dummy has the VALUE attribute.
/// - package the prepared dummy as required (fir.box, fir.class,
///   fir.box_char...).
/// This function should only be called with an actual that is present.
/// The optional aspects must be handled by this function user.
///
/// Note: while Fortran::lower::CallerInterface::PassedEntity (the type of arg)
/// is technically a template type, in the prepare*ActualArgument() calls
/// it resolves to Fortran::evaluate::ActualArgument *
static PreparedDummyArgument preparePresentUserCallActualArgument(
    mlir::Location loc, fir::FirOpBuilder &builder,
    const Fortran::lower::PreparedActualArgument &preparedActual,
    mlir::Type dummyType,
    const Fortran::lower::CallerInterface::PassedEntity &arg,
    CallContext &callContext) {

  // Step 1: get the actual argument, which includes addressing the
  // element if this is an array in an elemental call.
  hlfir::Entity actual = preparedActual.getActual(loc, builder);

````
- **L1321 EN**: Comment explains nearby logic, intent, or metadata: `prepare the actual argument according to the interface. Do as needed:`.
  **L1321 CN**: 注释说明附近代码的逻辑、意图或元数据：`prepare the actual argument according to the interface. Do as needed:`。
- **L1322 EN**: Comment explains nearby logic, intent, or metadata: `- address element if this is an array argument in an elemental call.`.
  **L1322 CN**: 注释说明附近代码的逻辑、意图或元数据：`- address element if this is an array argument in an elemental call.`。
- **L1323 EN**: Comment explains nearby logic, intent, or metadata: `- set dynamic type to the dummy type if the dummy is not polymorphic.`.
  **L1323 CN**: 注释说明附近代码的逻辑、意图或元数据：`- set dynamic type to the dummy type if the dummy is not polymorphic.`。
- **L1324 EN**: Comment explains nearby logic, intent, or metadata: `- copy-in into contiguous variable if the dummy must be contiguous`.
  **L1324 CN**: 注释说明附近代码的逻辑、意图或元数据：`- copy-in into contiguous variable if the dummy must be contiguous`。
- **L1325 EN**: Comment explains nearby logic, intent, or metadata: `- copy into a temporary if the dummy has the VALUE attribute.`.
  **L1325 CN**: 注释说明附近代码的逻辑、意图或元数据：`- copy into a temporary if the dummy has the VALUE attribute.`。
- **L1326 EN**: Comment explains nearby logic, intent, or metadata: `- package the prepared dummy as required (fir.box, fir.class,`.
  **L1326 CN**: 注释说明附近代码的逻辑、意图或元数据：`- package the prepared dummy as required (fir.box, fir.class,`。
- **L1327 EN**: Comment explains nearby logic, intent, or metadata: `fir.box_char...).`.
  **L1327 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.box_char...).`。
- **L1328 EN**: Comment explains nearby logic, intent, or metadata: `This function should only be called with an actual that is present.`.
  **L1328 CN**: 注释说明附近代码的逻辑、意图或元数据：`This function should only be called with an actual that is present.`。
- **L1329 EN**: Comment explains nearby logic, intent, or metadata: `The optional aspects must be handled by this function user.`.
  **L1329 CN**: 注释说明附近代码的逻辑、意图或元数据：`The optional aspects must be handled by this function user.`。
- **L1330 EN**: Separator comment used for visual grouping.
  **L1330 CN**: 用于视觉分组的分隔注释。
- **L1331 EN**: Comment explains nearby logic, intent, or metadata: `Note: while Fortran::lower::CallerInterface::PassedEntity (the type of arg)`.
  **L1331 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note: while Fortran::lower::CallerInterface::PassedEntity (the type of arg)`。
- **L1332 EN**: Comment explains nearby logic, intent, or metadata: `is technically a template type, in the prepare*ActualArgument() calls`.
  **L1332 CN**: 注释说明附近代码的逻辑、意图或元数据：`is technically a template type, in the prepare*ActualArgument() calls`。
- **L1333 EN**: Comment explains nearby logic, intent, or metadata: `it resolves to Fortran::evaluate::ActualArgument`.
  **L1333 CN**: 注释说明附近代码的逻辑、意图或元数据：`it resolves to Fortran::evaluate::ActualArgument`。
- **L1334 EN**: Continues logic associated with callable symbol `preparePresentUserCallActualArgument`.
  **L1334 CN**: 继续与可调用符号 `preparePresentUserCallActualArgument` 相关的逻辑。
- **L1335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, fir::FirOpBuilder &builder,`.
  **L1335 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, fir::FirOpBuilder &builder,`。
- **L1336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArgument &preparedActual,`.
  **L1336 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArgument &preparedActual,`。
- **L1337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Type dummyType,`.
  **L1337 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Type dummyType,`。
- **L1338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::CallerInterface::PassedEntity &arg,`.
  **L1338 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::CallerInterface::PassedEntity &arg,`。
- **L1339 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L1339 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L1340 EN**: Blank line separating nearby declarations or logic blocks.
  **L1340 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1341 EN**: Comment explains nearby logic, intent, or metadata: `Step 1: get the actual argument, which includes addressing the`.
  **L1341 CN**: 注释说明附近代码的逻辑、意图或元数据：`Step 1: get the actual argument, which includes addressing the`。
- **L1342 EN**: Comment explains nearby logic, intent, or metadata: `element if this is an array in an elemental call.`.
  **L1342 CN**: 注释说明附近代码的逻辑、意图或元数据：`element if this is an array in an elemental call.`。
- **L1343 EN**: Initializes variable `actual` from the right-hand expression.
  **L1343 CN**: 使用右侧表达式初始化变量 `actual`。
- **L1344 EN**: Blank line separating nearby declarations or logic blocks.
  **L1344 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1345-1368

````cpp
  if (arg.testTKR(Fortran::common::IgnoreTKR::Contiguous) &&
      actual.isBoxAddress() && fir::isBoxAddressOrValue(dummyType)) {
    // With ignore_tkr(c), pointer to a descriptor should be passed as is
    return PreparedDummyArgument{actual, /*cleanups=*/{}};
  }

  // Handle procedure arguments (procedure pointers should go through
  // prepareProcedurePointerActualArgument).
  if (hlfir::isFortranProcedureValue(dummyType)) {
    // Procedure pointer or function returns procedure pointer actual to
    // procedure dummy.
    if (actual.isProcedurePointer()) {
      actual = hlfir::derefPointersAndAllocatables(loc, builder, actual);
      return PreparedDummyArgument{actual, /*cleanups=*/{}};
    }
    // Procedure actual to procedure dummy.
    assert(actual.isProcedure());
    // Do nothing if this is a procedure argument. It is already a
    // fir.boxproc/fir.tuple<fir.boxproc, len> as it should.
    if (!mlir::isa<fir::BoxProcType>(actual.getType()) &&
        actual.getType() != dummyType)
      // The actual argument may be a procedure that returns character (a
      // fir.tuple<fir.boxproc, len>) while the dummy is not. Extract the tuple
      // in that case.
````
- **L1345 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1345 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1346 EN**: Starts a function, method, lambda, or structured scope: `actual.isBoxAddress() && fir::isBoxAddressOrValue(dummyType)) {`.
  **L1346 CN**: 开始一个函数、方法、lambda 或结构化作用域：`actual.isBoxAddress() && fir::isBoxAddressOrValue(dummyType)) {`。
- **L1347 EN**: Comment explains nearby logic, intent, or metadata: `With ignore_tkr(c), pointer to a descriptor should be passed as is`.
  **L1347 CN**: 注释说明附近代码的逻辑、意图或元数据：`With ignore_tkr(c), pointer to a descriptor should be passed as is`。
- **L1348 EN**: Returns from the current function with `PreparedDummyArgument{actual, /*cleanups=*/{}}`.
  **L1348 CN**: 以 `PreparedDummyArgument{actual, /*cleanups=*/{}}` 从当前函数返回。
- **L1349 EN**: Closes the current lexical scope or compound statement.
  **L1349 CN**: 结束当前词法作用域或复合语句块。
- **L1350 EN**: Blank line separating nearby declarations or logic blocks.
  **L1350 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1351 EN**: Comment explains nearby logic, intent, or metadata: `Handle procedure arguments (procedure pointers should go through`.
  **L1351 CN**: 注释说明附近代码的逻辑、意图或元数据：`Handle procedure arguments (procedure pointers should go through`。
- **L1352 EN**: Comment explains nearby logic, intent, or metadata: `prepareProcedurePointerActualArgument).`.
  **L1352 CN**: 注释说明附近代码的逻辑、意图或元数据：`prepareProcedurePointerActualArgument).`。
- **L1353 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1353 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1354 EN**: Comment explains nearby logic, intent, or metadata: `Procedure pointer or function returns procedure pointer actual to`.
  **L1354 CN**: 注释说明附近代码的逻辑、意图或元数据：`Procedure pointer or function returns procedure pointer actual to`。
- **L1355 EN**: Comment explains nearby logic, intent, or metadata: `procedure dummy.`.
  **L1355 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure dummy.`。
- **L1356 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1356 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1357 EN**: Executes a call or declaration centered on `hlfir::derefPointersAndAllocatables`.
  **L1357 CN**: 执行以 `hlfir::derefPointersAndAllocatables` 为核心的调用或声明。
- **L1358 EN**: Returns from the current function with `PreparedDummyArgument{actual, /*cleanups=*/{}}`.
  **L1358 CN**: 以 `PreparedDummyArgument{actual, /*cleanups=*/{}}` 从当前函数返回。
- **L1359 EN**: Closes the current lexical scope or compound statement.
  **L1359 CN**: 结束当前词法作用域或复合语句块。
- **L1360 EN**: Comment explains nearby logic, intent, or metadata: `Procedure actual to procedure dummy.`.
  **L1360 CN**: 注释说明附近代码的逻辑、意图或元数据：`Procedure actual to procedure dummy.`。
- **L1361 EN**: Checks an internal invariant in debug builds.
  **L1361 CN**: 在调试构建中检查内部不变式。
- **L1362 EN**: Comment explains nearby logic, intent, or metadata: `Do nothing if this is a procedure argument. It is already a`.
  **L1362 CN**: 注释说明附近代码的逻辑、意图或元数据：`Do nothing if this is a procedure argument. It is already a`。
- **L1363 EN**: Comment explains nearby logic, intent, or metadata: `fir.boxproc/fir.tuple<fir.boxproc, len> as it should.`.
  **L1363 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.boxproc/fir.tuple<fir.boxproc, len> as it should.`。
- **L1364 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1364 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1365 EN**: Continues logic associated with callable symbol `getType`.
  **L1365 CN**: 继续与可调用符号 `getType` 相关的逻辑。
- **L1366 EN**: Comment explains nearby logic, intent, or metadata: `The actual argument may be a procedure that returns character (a`.
  **L1366 CN**: 注释说明附近代码的逻辑、意图或元数据：`The actual argument may be a procedure that returns character (a`。
- **L1367 EN**: Comment explains nearby logic, intent, or metadata: `fir.tuple<fir.boxproc, len>) while the dummy is not. Extract the tuple`.
  **L1367 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.tuple<fir.boxproc, len>) while the dummy is not. Extract the tuple`。
- **L1368 EN**: Comment explains nearby logic, intent, or metadata: `in that case.`.
  **L1368 CN**: 注释说明附近代码的逻辑、意图或元数据：`in that case.`。

### Lines 1369-1392

````cpp
      actual = fixProcedureDummyMismatch(loc, builder, actual, dummyType);
    return PreparedDummyArgument{actual, /*cleanups=*/{}};
  }

  const bool ignoreTKRtype = arg.testTKR(Fortran::common::IgnoreTKR::Type);
  const bool passingPolymorphicToNonPolymorphic =
      actual.isPolymorphic() && !fir::isPolymorphicType(dummyType) &&
      !ignoreTKRtype;

  // When passing a CLASS(T) to TYPE(T), only the "T" part must be
  // passed. Unless the entity is a scalar passed by raw address, a
  // new descriptor must be made using the dummy argument type as
  // dynamic type. This must be done before any copy/copy-in because the
  // dynamic type matters to determine the contiguity.
  const bool mustSetDynamicTypeToDummyType =
      passingPolymorphicToNonPolymorphic &&
      (actual.isArray() || mlir::isa<fir::BaseBoxType>(dummyType));

  bool mustDoCopyIn{false};
  bool mustDoCopyOut{false};

  if (callContext.doCopyIn) {
    Fortran::evaluate::FoldingContext &foldingContext{
        callContext.converter.getFoldingContext()};
````
- **L1369 EN**: Executes a call or declaration centered on `fixProcedureDummyMismatch`.
  **L1369 CN**: 执行以 `fixProcedureDummyMismatch` 为核心的调用或声明。
- **L1370 EN**: Returns from the current function with `PreparedDummyArgument{actual, /*cleanups=*/{}}`.
  **L1370 CN**: 以 `PreparedDummyArgument{actual, /*cleanups=*/{}}` 从当前函数返回。
- **L1371 EN**: Closes the current lexical scope or compound statement.
  **L1371 CN**: 结束当前词法作用域或复合语句块。
- **L1372 EN**: Blank line separating nearby declarations or logic blocks.
  **L1372 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1373 EN**: Initializes variable `ignoreTKRtype` from the right-hand expression.
  **L1373 CN**: 使用右侧表达式初始化变量 `ignoreTKRtype`。
- **L1374 EN**: Continues the surrounding expression or declaration: `const bool passingPolymorphicToNonPolymorphic =`.
  **L1374 CN**: 继续构造周围的表达式或声明：`const bool passingPolymorphicToNonPolymorphic =`。
- **L1375 EN**: Continues logic associated with callable symbol `isPolymorphic`.
  **L1375 CN**: 继续与可调用符号 `isPolymorphic` 相关的逻辑。
- **L1376 EN**: Executes a standalone statement or declaration: `!ignoreTKRtype;`.
  **L1376 CN**: 执行一条独立语句或声明：`!ignoreTKRtype;`。
- **L1377 EN**: Blank line separating nearby declarations or logic blocks.
  **L1377 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1378 EN**: Comment explains nearby logic, intent, or metadata: `When passing a CLASS(T) to TYPE(T), only the "T" part must be`.
  **L1378 CN**: 注释说明附近代码的逻辑、意图或元数据：`When passing a CLASS(T) to TYPE(T), only the "T" part must be`。
- **L1379 EN**: Comment explains nearby logic, intent, or metadata: `passed. Unless the entity is a scalar passed by raw address, a`.
  **L1379 CN**: 注释说明附近代码的逻辑、意图或元数据：`passed. Unless the entity is a scalar passed by raw address, a`。
- **L1380 EN**: Comment explains nearby logic, intent, or metadata: `new descriptor must be made using the dummy argument type as`.
  **L1380 CN**: 注释说明附近代码的逻辑、意图或元数据：`new descriptor must be made using the dummy argument type as`。
- **L1381 EN**: Comment explains nearby logic, intent, or metadata: `dynamic type. This must be done before any copy/copy-in because the`.
  **L1381 CN**: 注释说明附近代码的逻辑、意图或元数据：`dynamic type. This must be done before any copy/copy-in because the`。
- **L1382 EN**: Comment explains nearby logic, intent, or metadata: `dynamic type matters to determine the contiguity.`.
  **L1382 CN**: 注释说明附近代码的逻辑、意图或元数据：`dynamic type matters to determine the contiguity.`。
- **L1383 EN**: Continues the surrounding expression or declaration: `const bool mustSetDynamicTypeToDummyType =`.
  **L1383 CN**: 继续构造周围的表达式或声明：`const bool mustSetDynamicTypeToDummyType =`。
- **L1384 EN**: Continues the surrounding expression or declaration: `passingPolymorphicToNonPolymorphic &&`.
  **L1384 CN**: 继续构造周围的表达式或声明：`passingPolymorphicToNonPolymorphic &&`。
- **L1385 EN**: Executes a call or declaration centered on `statement`.
  **L1385 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1386 EN**: Blank line separating nearby declarations or logic blocks.
  **L1386 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1387 EN**: Executes a standalone statement or declaration: `bool mustDoCopyIn{false};`.
  **L1387 CN**: 执行一条独立语句或声明：`bool mustDoCopyIn{false};`。
- **L1388 EN**: Executes a standalone statement or declaration: `bool mustDoCopyOut{false};`.
  **L1388 CN**: 执行一条独立语句或声明：`bool mustDoCopyOut{false};`。
- **L1389 EN**: Blank line separating nearby declarations or logic blocks.
  **L1389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1390 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1390 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1391 EN**: Continues the surrounding expression or declaration: `Fortran::evaluate::FoldingContext &foldingContext{`.
  **L1391 CN**: 继续构造周围的表达式或声明：`Fortran::evaluate::FoldingContext &foldingContext{`。
- **L1392 EN**: Executes a call or declaration centered on `callContext.converter.getFoldingContext`.
  **L1392 CN**: 执行以 `callContext.converter.getFoldingContext` 为核心的调用或声明。

### Lines 1393-1416

````cpp

    bool suggestCopyIn = Fortran::evaluate::ActualArgNeedsCopy(
                             arg.entity, arg.characteristics, foldingContext,
                             /*forCopyOut=*/false)
                             .value_or(true);
    bool suggestCopyOut = Fortran::evaluate::ActualArgNeedsCopy(
                              arg.entity, arg.characteristics, foldingContext,
                              /*forCopyOut=*/true)
                              .value_or(true);
    mustDoCopyIn = actual.isArray() && suggestCopyIn;
    mustDoCopyOut = actual.isArray() && suggestCopyOut;
  }

  const bool actualIsAssumedRank = actual.isAssumedRank();
  // Create dummy type with actual argument rank when the dummy is an assumed
  // rank. That way, all the operation to create dummy descriptors are ranked if
  // the actual argument is ranked, which allows simple code generation.
  // Also do the same when the dummy is a sequence associated descriptor
  // because the actual shape/rank may mismatch with the dummy, and the dummy
  // may be an assumed-size array, so any descriptor manipulation should use the
  // actual argument shape information. A descriptor with the dummy shape
  // information will be created later when all actual arguments are ready.
  mlir::Type dummyTypeWithActualRank = dummyType;
  if (auto baseBoxDummy = mlir::dyn_cast<fir::BaseBoxType>(dummyType)) {
````
- **L1393 EN**: Blank line separating nearby declarations or logic blocks.
  **L1393 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1394 EN**: Continues logic associated with callable symbol `ActualArgNeedsCopy`.
  **L1394 CN**: 继续与可调用符号 `ActualArgNeedsCopy` 相关的逻辑。
- **L1395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arg.entity, arg.characteristics, foldingContext,`.
  **L1395 CN**: 继续一个多行参数列表、初始化器或聚合项：`arg.entity, arg.characteristics, foldingContext,`。
- **L1396 EN**: Comment explains nearby logic, intent, or metadata: `forCopyOut=*/false)`.
  **L1396 CN**: 注释说明附近代码的逻辑、意图或元数据：`forCopyOut=*/false)`。
- **L1397 EN**: Executes a call or declaration centered on `.value_or`.
  **L1397 CN**: 执行以 `.value_or` 为核心的调用或声明。
- **L1398 EN**: Continues logic associated with callable symbol `ActualArgNeedsCopy`.
  **L1398 CN**: 继续与可调用符号 `ActualArgNeedsCopy` 相关的逻辑。
- **L1399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arg.entity, arg.characteristics, foldingContext,`.
  **L1399 CN**: 继续一个多行参数列表、初始化器或聚合项：`arg.entity, arg.characteristics, foldingContext,`。
- **L1400 EN**: Comment explains nearby logic, intent, or metadata: `forCopyOut=*/true)`.
  **L1400 CN**: 注释说明附近代码的逻辑、意图或元数据：`forCopyOut=*/true)`。
- **L1401 EN**: Executes a call or declaration centered on `.value_or`.
  **L1401 CN**: 执行以 `.value_or` 为核心的调用或声明。
- **L1402 EN**: Executes a call or declaration centered on `actual.isArray`.
  **L1402 CN**: 执行以 `actual.isArray` 为核心的调用或声明。
- **L1403 EN**: Executes a call or declaration centered on `actual.isArray`.
  **L1403 CN**: 执行以 `actual.isArray` 为核心的调用或声明。
- **L1404 EN**: Closes the current lexical scope or compound statement.
  **L1404 CN**: 结束当前词法作用域或复合语句块。
- **L1405 EN**: Blank line separating nearby declarations or logic blocks.
  **L1405 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1406 EN**: Initializes variable `actualIsAssumedRank` from the right-hand expression.
  **L1406 CN**: 使用右侧表达式初始化变量 `actualIsAssumedRank`。
- **L1407 EN**: Comment explains nearby logic, intent, or metadata: `Create dummy type with actual argument rank when the dummy is an assumed`.
  **L1407 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create dummy type with actual argument rank when the dummy is an assumed`。
- **L1408 EN**: Comment explains nearby logic, intent, or metadata: `rank. That way, all the operation to create dummy descriptors are ranked if`.
  **L1408 CN**: 注释说明附近代码的逻辑、意图或元数据：`rank. That way, all the operation to create dummy descriptors are ranked if`。
- **L1409 EN**: Comment explains nearby logic, intent, or metadata: `the actual argument is ranked, which allows simple code generation.`.
  **L1409 CN**: 注释说明附近代码的逻辑、意图或元数据：`the actual argument is ranked, which allows simple code generation.`。
- **L1410 EN**: Comment explains nearby logic, intent, or metadata: `Also do the same when the dummy is a sequence associated descriptor`.
  **L1410 CN**: 注释说明附近代码的逻辑、意图或元数据：`Also do the same when the dummy is a sequence associated descriptor`。
- **L1411 EN**: Comment explains nearby logic, intent, or metadata: `because the actual shape/rank may mismatch with the dummy, and the dummy`.
  **L1411 CN**: 注释说明附近代码的逻辑、意图或元数据：`because the actual shape/rank may mismatch with the dummy, and the dummy`。
- **L1412 EN**: Comment explains nearby logic, intent, or metadata: `may be an assumed-size array, so any descriptor manipulation should use the`.
  **L1412 CN**: 注释说明附近代码的逻辑、意图或元数据：`may be an assumed-size array, so any descriptor manipulation should use the`。
- **L1413 EN**: Comment explains nearby logic, intent, or metadata: `actual argument shape information. A descriptor with the dummy shape`.
  **L1413 CN**: 注释说明附近代码的逻辑、意图或元数据：`actual argument shape information. A descriptor with the dummy shape`。
- **L1414 EN**: Comment explains nearby logic, intent, or metadata: `information will be created later when all actual arguments are ready.`.
  **L1414 CN**: 注释说明附近代码的逻辑、意图或元数据：`information will be created later when all actual arguments are ready.`。
- **L1415 EN**: Initializes variable `dummyTypeWithActualRank` from the right-hand expression.
  **L1415 CN**: 使用右侧表达式初始化变量 `dummyTypeWithActualRank`。
- **L1416 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1416 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1417-1440

````cpp
    if (baseBoxDummy.isAssumedRank() ||
        arg.testTKR(Fortran::common::IgnoreTKR::Rank) ||
        arg.isSequenceAssociatedDescriptor()) {
      mlir::Type actualTy =
          hlfir::getFortranElementOrSequenceType(actual.getType());
      dummyTypeWithActualRank = baseBoxDummy.getBoxTypeWithNewShape(actualTy);
    }
  }
  // Preserve the actual type in the argument preparation in case IgnoreTKR(t)
  // is set (descriptors must be created with the actual type in this case, and
  // copy-in/copy-out should be driven by the contiguity with regard to the
  // actual type).
  if (ignoreTKRtype) {
    if (auto boxCharType =
            mlir::dyn_cast<fir::BoxCharType>(dummyTypeWithActualRank)) {
      auto maybeActualCharType =
          mlir::dyn_cast<fir::CharacterType>(actual.getFortranElementType());
      if (!maybeActualCharType ||
          maybeActualCharType.getFKind() != boxCharType.getKind()) {
        // When passing to a fir.boxchar with ignore(tk), prepare the argument
        // as if only the raw address must be passed.
        dummyTypeWithActualRank =
            fir::ReferenceType::get(actual.getElementOrSequenceType());
      }
````
- **L1417 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1417 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1418 EN**: Continues logic associated with callable symbol `testTKR`.
  **L1418 CN**: 继续与可调用符号 `testTKR` 相关的逻辑。
- **L1419 EN**: Starts a function, method, lambda, or structured scope: `arg.isSequenceAssociatedDescriptor()) {`.
  **L1419 CN**: 开始一个函数、方法、lambda 或结构化作用域：`arg.isSequenceAssociatedDescriptor()) {`。
- **L1420 EN**: Continues the surrounding expression or declaration: `mlir::Type actualTy =`.
  **L1420 CN**: 继续构造周围的表达式或声明：`mlir::Type actualTy =`。
- **L1421 EN**: Executes a call or declaration centered on `hlfir::getFortranElementOrSequenceType`.
  **L1421 CN**: 执行以 `hlfir::getFortranElementOrSequenceType` 为核心的调用或声明。
- **L1422 EN**: Executes a call or declaration centered on `baseBoxDummy.getBoxTypeWithNewShape`.
  **L1422 CN**: 执行以 `baseBoxDummy.getBoxTypeWithNewShape` 为核心的调用或声明。
- **L1423 EN**: Closes the current lexical scope or compound statement.
  **L1423 CN**: 结束当前词法作用域或复合语句块。
- **L1424 EN**: Closes the current lexical scope or compound statement.
  **L1424 CN**: 结束当前词法作用域或复合语句块。
- **L1425 EN**: Comment explains nearby logic, intent, or metadata: `Preserve the actual type in the argument preparation in case IgnoreTKR(t)`.
  **L1425 CN**: 注释说明附近代码的逻辑、意图或元数据：`Preserve the actual type in the argument preparation in case IgnoreTKR(t)`。
- **L1426 EN**: Comment explains nearby logic, intent, or metadata: `is set (descriptors must be created with the actual type in this case, and`.
  **L1426 CN**: 注释说明附近代码的逻辑、意图或元数据：`is set (descriptors must be created with the actual type in this case, and`。
- **L1427 EN**: Comment explains nearby logic, intent, or metadata: `copy-in/copy-out should be driven by the contiguity with regard to the`.
  **L1427 CN**: 注释说明附近代码的逻辑、意图或元数据：`copy-in/copy-out should be driven by the contiguity with regard to the`。
- **L1428 EN**: Comment explains nearby logic, intent, or metadata: `actual type).`.
  **L1428 CN**: 注释说明附近代码的逻辑、意图或元数据：`actual type).`。
- **L1429 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1429 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1430 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1430 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1431 EN**: Starts a function, method, lambda, or structured scope: `mlir::dyn_cast<fir::BoxCharType>(dummyTypeWithActualRank)) {`.
  **L1431 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::dyn_cast<fir::BoxCharType>(dummyTypeWithActualRank)) {`。
- **L1432 EN**: Continues the surrounding expression or declaration: `auto maybeActualCharType =`.
  **L1432 CN**: 继续构造周围的表达式或声明：`auto maybeActualCharType =`。
- **L1433 EN**: Executes a call or declaration centered on `mlir::dyn_cast<fir::CharacterType>`.
  **L1433 CN**: 执行以 `mlir::dyn_cast<fir::CharacterType>` 为核心的调用或声明。
- **L1434 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1434 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1435 EN**: Starts a function, method, lambda, or structured scope: `maybeActualCharType.getFKind() != boxCharType.getKind()) {`.
  **L1435 CN**: 开始一个函数、方法、lambda 或结构化作用域：`maybeActualCharType.getFKind() != boxCharType.getKind()) {`。
- **L1436 EN**: Comment explains nearby logic, intent, or metadata: `When passing to a fir.boxchar with ignore(tk), prepare the argument`.
  **L1436 CN**: 注释说明附近代码的逻辑、意图或元数据：`When passing to a fir.boxchar with ignore(tk), prepare the argument`。
- **L1437 EN**: Comment explains nearby logic, intent, or metadata: `as if only the raw address must be passed.`.
  **L1437 CN**: 注释说明附近代码的逻辑、意图或元数据：`as if only the raw address must be passed.`。
- **L1438 EN**: Continues the surrounding expression or declaration: `dummyTypeWithActualRank =`.
  **L1438 CN**: 继续构造周围的表达式或声明：`dummyTypeWithActualRank =`。
- **L1439 EN**: Executes a call or declaration centered on `fir::ReferenceType::get`.
  **L1439 CN**: 执行以 `fir::ReferenceType::get` 为核心的调用或声明。
- **L1440 EN**: Closes the current lexical scope or compound statement.
  **L1440 CN**: 结束当前词法作用域或复合语句块。

### Lines 1441-1464

````cpp
      // Otherwise, the actual is already a character with the same kind as the
      // dummy and can be passed normally.
    } else {
      dummyTypeWithActualRank = fir::changeElementType(
          dummyTypeWithActualRank, actual.getFortranElementType(),
          actual.isPolymorphic());
    }
  }

  PreparedDummyArgument preparedDummy;

  // Helpers to generate hlfir.copy_in operation and register the related
  // hlfir.copy_out creation.
  auto genCopyIn = [&](hlfir::Entity var, bool doCopyOut) -> hlfir::Entity {
    auto baseBoxTy = mlir::dyn_cast<fir::BaseBoxType>(var.getType());
    assert(baseBoxTy && "expect non simply contiguous variables to be boxes");
    // Create allocatable descriptor for the potential temporary.
    mlir::Type tempBoxType = baseBoxTy.getBoxTypeWithNewAttr(
        fir::BaseBoxType::Attribute::Allocatable);
    mlir::Value tempBox = builder.createTemporary(loc, tempBoxType);
    auto copyIn = hlfir::CopyInOp::create(builder, loc, var, tempBox,
                                          /*var_is_present=*/mlir::Value{});
    // Register the copy-out after the call.
    preparedDummy.pushCopyInCleanUp(copyIn.getTempBox(), copyIn.getWasCopied(),
````
- **L1441 EN**: Comment explains nearby logic, intent, or metadata: `Otherwise, the actual is already a character with the same kind as the`.
  **L1441 CN**: 注释说明附近代码的逻辑、意图或元数据：`Otherwise, the actual is already a character with the same kind as the`。
- **L1442 EN**: Comment explains nearby logic, intent, or metadata: `dummy and can be passed normally.`.
  **L1442 CN**: 注释说明附近代码的逻辑、意图或元数据：`dummy and can be passed normally.`。
- **L1443 EN**: Transitions from the previous branch into the alternative path.
  **L1443 CN**: 从前一个分支过渡到备选路径。
- **L1444 EN**: Continues logic associated with callable symbol `changeElementType`.
  **L1444 CN**: 继续与可调用符号 `changeElementType` 相关的逻辑。
- **L1445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dummyTypeWithActualRank, actual.getFortranElementType(),`.
  **L1445 CN**: 继续一个多行参数列表、初始化器或聚合项：`dummyTypeWithActualRank, actual.getFortranElementType(),`。
- **L1446 EN**: Executes a call or declaration centered on `actual.isPolymorphic`.
  **L1446 CN**: 执行以 `actual.isPolymorphic` 为核心的调用或声明。
- **L1447 EN**: Closes the current lexical scope or compound statement.
  **L1447 CN**: 结束当前词法作用域或复合语句块。
- **L1448 EN**: Closes the current lexical scope or compound statement.
  **L1448 CN**: 结束当前词法作用域或复合语句块。
- **L1449 EN**: Blank line separating nearby declarations or logic blocks.
  **L1449 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1450 EN**: Executes a standalone statement or declaration: `PreparedDummyArgument preparedDummy;`.
  **L1450 CN**: 执行一条独立语句或声明：`PreparedDummyArgument preparedDummy;`。
- **L1451 EN**: Blank line separating nearby declarations or logic blocks.
  **L1451 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1452 EN**: Comment explains nearby logic, intent, or metadata: `Helpers to generate hlfir.copy_in operation and register the related`.
  **L1452 CN**: 注释说明附近代码的逻辑、意图或元数据：`Helpers to generate hlfir.copy_in operation and register the related`。
- **L1453 EN**: Comment explains nearby logic, intent, or metadata: `hlfir.copy_out creation.`.
  **L1453 CN**: 注释说明附近代码的逻辑、意图或元数据：`hlfir.copy_out creation.`。
- **L1454 EN**: Starts a function, method, lambda, or structured scope: `auto genCopyIn = [&](hlfir::Entity var, bool doCopyOut) -> hlfir::Entity {`.
  **L1454 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto genCopyIn = [&](hlfir::Entity var, bool doCopyOut) -> hlfir::Entity {`。
- **L1455 EN**: Initializes variable `baseBoxTy` from the right-hand expression.
  **L1455 CN**: 使用右侧表达式初始化变量 `baseBoxTy`。
- **L1456 EN**: Checks an internal invariant in debug builds.
  **L1456 CN**: 在调试构建中检查内部不变式。
- **L1457 EN**: Comment explains nearby logic, intent, or metadata: `Create allocatable descriptor for the potential temporary.`.
  **L1457 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create allocatable descriptor for the potential temporary.`。
- **L1458 EN**: Continues logic associated with callable symbol `getBoxTypeWithNewAttr`.
  **L1458 CN**: 继续与可调用符号 `getBoxTypeWithNewAttr` 相关的逻辑。
- **L1459 EN**: Executes a standalone statement or declaration: `fir::BaseBoxType::Attribute::Allocatable);`.
  **L1459 CN**: 执行一条独立语句或声明：`fir::BaseBoxType::Attribute::Allocatable);`。
- **L1460 EN**: Initializes variable `tempBox` from the right-hand expression.
  **L1460 CN**: 使用右侧表达式初始化变量 `tempBox`。
- **L1461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto copyIn = hlfir::CopyInOp::create(builder, loc, var, tempBox,`.
  **L1461 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto copyIn = hlfir::CopyInOp::create(builder, loc, var, tempBox,`。
- **L1462 EN**: Comment explains nearby logic, intent, or metadata: `var_is_present=*/mlir::Value{});`.
  **L1462 CN**: 注释说明附近代码的逻辑、意图或元数据：`var_is_present=*/mlir::Value{});`。
- **L1463 EN**: Comment explains nearby logic, intent, or metadata: `Register the copy-out after the call.`.
  **L1463 CN**: 注释说明附近代码的逻辑、意图或元数据：`Register the copy-out after the call.`。
- **L1464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `preparedDummy.pushCopyInCleanUp(copyIn.getTempBox(), copyIn.getWasCopied(),`.
  **L1464 CN**: 继续一个多行参数列表、初始化器或聚合项：`preparedDummy.pushCopyInCleanUp(copyIn.getTempBox(), copyIn.getWasCopied(),`。

### Lines 1465-1488

````cpp
                                    doCopyOut ? copyIn.getVar()
                                              : mlir::Value{});
    return hlfir::Entity{copyIn.getCopiedIn()};
  };

  auto genSetDynamicTypeToDummyType = [&](hlfir::Entity var) -> hlfir::Entity {
    fir::BaseBoxType boxType = fir::BoxType::get(
        hlfir::getFortranElementOrSequenceType(dummyTypeWithActualRank));
    if (actualIsAssumedRank)
      return hlfir::Entity{fir::ReboxAssumedRankOp::create(
          builder, loc, boxType, var,
          fir::LowerBoundModifierAttribute::SetToOnes)};
    // Use actual shape when creating descriptor with dummy type, the dummy
    // shape may be unknown in case of sequence association.
    mlir::Type actualTy =
        hlfir::getFortranElementOrSequenceType(actual.getType());
    boxType = boxType.getBoxTypeWithNewShape(actualTy);
    return hlfir::Entity{fir::ReboxOp::create(builder, loc, boxType, var,
                                              /*shape=*/mlir::Value{},
                                              /*slice=*/mlir::Value{})};
  };

  // Step 2: prepare the storage for the dummy arguments, ensuring that it
  // matches the dummy requirements (e.g., must be contiguous or must be
````
- **L1465 EN**: Continues logic associated with callable symbol `getVar`.
  **L1465 CN**: 继续与可调用符号 `getVar` 相关的逻辑。
- **L1466 EN**: Executes a standalone statement or declaration: `: mlir::Value{});`.
  **L1466 CN**: 执行一条独立语句或声明：`: mlir::Value{});`。
- **L1467 EN**: Returns from the current function with `hlfir::Entity{copyIn.getCopiedIn()}`.
  **L1467 CN**: 以 `hlfir::Entity{copyIn.getCopiedIn()}` 从当前函数返回。
- **L1468 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1468 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1469 EN**: Blank line separating nearby declarations or logic blocks.
  **L1469 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1470 EN**: Starts a function, method, lambda, or structured scope: `auto genSetDynamicTypeToDummyType = [&](hlfir::Entity var) -> hlfir::Entity {`.
  **L1470 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto genSetDynamicTypeToDummyType = [&](hlfir::Entity var) -> hlfir::Entity {`。
- **L1471 EN**: Continues logic associated with callable symbol `get`.
  **L1471 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L1472 EN**: Executes a call or declaration centered on `hlfir::getFortranElementOrSequenceType`.
  **L1472 CN**: 执行以 `hlfir::getFortranElementOrSequenceType` 为核心的调用或声明。
- **L1473 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1473 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1474 EN**: Returns from the current function with `hlfir::Entity{fir::ReboxAssumedRankOp::create(`.
  **L1474 CN**: 以 `hlfir::Entity{fir::ReboxAssumedRankOp::create(` 从当前函数返回。
- **L1475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, boxType, var,`.
  **L1475 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, boxType, var,`。
- **L1476 EN**: Executes a standalone statement or declaration: `fir::LowerBoundModifierAttribute::SetToOnes)};`.
  **L1476 CN**: 执行一条独立语句或声明：`fir::LowerBoundModifierAttribute::SetToOnes)};`。
- **L1477 EN**: Comment explains nearby logic, intent, or metadata: `Use actual shape when creating descriptor with dummy type, the dummy`.
  **L1477 CN**: 注释说明附近代码的逻辑、意图或元数据：`Use actual shape when creating descriptor with dummy type, the dummy`。
- **L1478 EN**: Comment explains nearby logic, intent, or metadata: `shape may be unknown in case of sequence association.`.
  **L1478 CN**: 注释说明附近代码的逻辑、意图或元数据：`shape may be unknown in case of sequence association.`。
- **L1479 EN**: Continues the surrounding expression or declaration: `mlir::Type actualTy =`.
  **L1479 CN**: 继续构造周围的表达式或声明：`mlir::Type actualTy =`。
- **L1480 EN**: Executes a call or declaration centered on `hlfir::getFortranElementOrSequenceType`.
  **L1480 CN**: 执行以 `hlfir::getFortranElementOrSequenceType` 为核心的调用或声明。
- **L1481 EN**: Executes a call or declaration centered on `boxType.getBoxTypeWithNewShape`.
  **L1481 CN**: 执行以 `boxType.getBoxTypeWithNewShape` 为核心的调用或声明。
- **L1482 EN**: Returns from the current function with `hlfir::Entity{fir::ReboxOp::create(builder, loc, boxType, var,`.
  **L1482 CN**: 以 `hlfir::Entity{fir::ReboxOp::create(builder, loc, boxType, var,` 从当前函数返回。
- **L1483 EN**: Comment explains nearby logic, intent, or metadata: `shape=*/mlir::Value{},`.
  **L1483 CN**: 注释说明附近代码的逻辑、意图或元数据：`shape=*/mlir::Value{},`。
- **L1484 EN**: Comment explains nearby logic, intent, or metadata: `slice=*/mlir::Value{})};`.
  **L1484 CN**: 注释说明附近代码的逻辑、意图或元数据：`slice=*/mlir::Value{})};`。
- **L1485 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1485 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1486 EN**: Blank line separating nearby declarations or logic blocks.
  **L1486 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1487 EN**: Comment explains nearby logic, intent, or metadata: `Step 2: prepare the storage for the dummy arguments, ensuring that it`.
  **L1487 CN**: 注释说明附近代码的逻辑、意图或元数据：`Step 2: prepare the storage for the dummy arguments, ensuring that it`。
- **L1488 EN**: Comment explains nearby logic, intent, or metadata: `matches the dummy requirements (e.g., must be contiguous or must be`.
  **L1488 CN**: 注释说明附近代码的逻辑、意图或元数据：`matches the dummy requirements (e.g., must be contiguous or must be`。

### Lines 1489-1512

````cpp
  // a temporary).
  hlfir::Entity entity =
      hlfir::derefPointersAndAllocatables(loc, builder, actual);
  if (entity.isVariable()) {
    // Set dynamic type if needed before any copy-in or copy so that the dummy
    // is contiguous according to the dummy type.
    if (mustSetDynamicTypeToDummyType)
      entity = genSetDynamicTypeToDummyType(entity);
    if (arg.hasValueAttribute() ||
        // Constant expressions might be lowered as variables with
        // 'parameter' attribute. Even though the constant expressions
        // are not definable and explicit assignments to them are not
        // possible, we have to create a temporary copies when we pass
        // them down the call stack because of potential compiler
        // generated writes in copy-out.
        isParameterObjectOrSubObject(entity)) {
      // Make a copy in a temporary.
      auto copy = hlfir::AsExprOp::create(builder, loc, entity);
      mlir::Type storageType = entity.getType();
      mlir::NamedAttribute byRefAttr = fir::getAdaptToByRefAttr(builder);
      hlfir::AssociateOp associate = hlfir::genAssociateExpr(
          loc, builder, hlfir::Entity{copy}, storageType, "", byRefAttr);
      entity = hlfir::Entity{associate.getBase()};
      // Register the temporary destruction after the call.
````
- **L1489 EN**: Comment explains nearby logic, intent, or metadata: `a temporary).`.
  **L1489 CN**: 注释说明附近代码的逻辑、意图或元数据：`a temporary).`。
- **L1490 EN**: Continues the surrounding expression or declaration: `hlfir::Entity entity =`.
  **L1490 CN**: 继续构造周围的表达式或声明：`hlfir::Entity entity =`。
- **L1491 EN**: Executes a call or declaration centered on `hlfir::derefPointersAndAllocatables`.
  **L1491 CN**: 执行以 `hlfir::derefPointersAndAllocatables` 为核心的调用或声明。
- **L1492 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1492 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1493 EN**: Comment explains nearby logic, intent, or metadata: `Set dynamic type if needed before any copy-in or copy so that the dummy`.
  **L1493 CN**: 注释说明附近代码的逻辑、意图或元数据：`Set dynamic type if needed before any copy-in or copy so that the dummy`。
- **L1494 EN**: Comment explains nearby logic, intent, or metadata: `is contiguous according to the dummy type.`.
  **L1494 CN**: 注释说明附近代码的逻辑、意图或元数据：`is contiguous according to the dummy type.`。
- **L1495 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1495 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1496 EN**: Executes a call or declaration centered on `genSetDynamicTypeToDummyType`.
  **L1496 CN**: 执行以 `genSetDynamicTypeToDummyType` 为核心的调用或声明。
- **L1497 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1497 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1498 EN**: Comment explains nearby logic, intent, or metadata: `Constant expressions might be lowered as variables with`.
  **L1498 CN**: 注释说明附近代码的逻辑、意图或元数据：`Constant expressions might be lowered as variables with`。
- **L1499 EN**: Comment explains nearby logic, intent, or metadata: `'parameter' attribute. Even though the constant expressions`.
  **L1499 CN**: 注释说明附近代码的逻辑、意图或元数据：`'parameter' attribute. Even though the constant expressions`。
- **L1500 EN**: Comment explains nearby logic, intent, or metadata: `are not definable and explicit assignments to them are not`.
  **L1500 CN**: 注释说明附近代码的逻辑、意图或元数据：`are not definable and explicit assignments to them are not`。
- **L1501 EN**: Comment explains nearby logic, intent, or metadata: `possible, we have to create a temporary copies when we pass`.
  **L1501 CN**: 注释说明附近代码的逻辑、意图或元数据：`possible, we have to create a temporary copies when we pass`。
- **L1502 EN**: Comment explains nearby logic, intent, or metadata: `them down the call stack because of potential compiler`.
  **L1502 CN**: 注释说明附近代码的逻辑、意图或元数据：`them down the call stack because of potential compiler`。
- **L1503 EN**: Comment explains nearby logic, intent, or metadata: `generated writes in copy-out.`.
  **L1503 CN**: 注释说明附近代码的逻辑、意图或元数据：`generated writes in copy-out.`。
- **L1504 EN**: Starts a function, method, lambda, or structured scope: `isParameterObjectOrSubObject(entity)) {`.
  **L1504 CN**: 开始一个函数、方法、lambda 或结构化作用域：`isParameterObjectOrSubObject(entity)) {`。
- **L1505 EN**: Comment explains nearby logic, intent, or metadata: `Make a copy in a temporary.`.
  **L1505 CN**: 注释说明附近代码的逻辑、意图或元数据：`Make a copy in a temporary.`。
- **L1506 EN**: Initializes variable `copy` from the right-hand expression.
  **L1506 CN**: 使用右侧表达式初始化变量 `copy`。
- **L1507 EN**: Initializes variable `storageType` from the right-hand expression.
  **L1507 CN**: 使用右侧表达式初始化变量 `storageType`。
- **L1508 EN**: Initializes variable `byRefAttr` from the right-hand expression.
  **L1508 CN**: 使用右侧表达式初始化变量 `byRefAttr`。
- **L1509 EN**: Continues logic associated with callable symbol `genAssociateExpr`.
  **L1509 CN**: 继续与可调用符号 `genAssociateExpr` 相关的逻辑。
- **L1510 EN**: Executes a standalone statement or declaration: `loc, builder, hlfir::Entity{copy}, storageType, "", byRefAttr);`.
  **L1510 CN**: 执行一条独立语句或声明：`loc, builder, hlfir::Entity{copy}, storageType, "", byRefAttr);`。
- **L1511 EN**: Executes a call or declaration centered on `hlfir::Entity{associate.getBase`.
  **L1511 CN**: 执行以 `hlfir::Entity{associate.getBase` 为核心的调用或声明。
- **L1512 EN**: Comment explains nearby logic, intent, or metadata: `Register the temporary destruction after the call.`.
  **L1512 CN**: 注释说明附近代码的逻辑、意图或元数据：`Register the temporary destruction after the call.`。

### Lines 1513-1536

````cpp
      preparedDummy.pushExprAssociateCleanUp(associate);
    } else if (mustDoCopyIn || mustDoCopyOut) {
      // Copy-in non contiguous variables.
      //
      // TODO: copy-in and copy-out are now determined separately, in order
      // to allow more fine grained copying. While currently both copy-in
      // and copy-out are must be done together, these copy operations could
      // be separated in the future. (This is related to TODO comment below.)
      //
      // TODO: for non-finalizable monomorphic derived type actual
      // arguments associated with INTENT(OUT) dummy arguments
      // we may avoid doing the copy and only allocate the temporary.
      // The codegen would do a "mold" allocation instead of "sourced"
      // allocation for the temp in this case. We can communicate
      // this to the codegen via some CopyInOp flag.
      // This is a performance concern.
      entity = genCopyIn(entity, mustDoCopyOut);
    }
  } else {
    const Fortran::lower::SomeExpr *expr = arg.entity->UnwrapExpr();
    assert(expr && "expression actual argument cannot be an assumed type");
    // The actual is an expression value, place it into a temporary
    // and register the temporary destruction after the call.
    mlir::Type storageType = callContext.converter.genType(*expr);
````
- **L1513 EN**: Executes a call or declaration centered on `preparedDummy.pushExprAssociateCleanUp`.
  **L1513 CN**: 执行以 `preparedDummy.pushExprAssociateCleanUp` 为核心的调用或声明。
- **L1514 EN**: Transitions from the previous branch into an `else if` condition.
  **L1514 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1515 EN**: Comment explains nearby logic, intent, or metadata: `Copy-in non contiguous variables.`.
  **L1515 CN**: 注释说明附近代码的逻辑、意图或元数据：`Copy-in non contiguous variables.`。
- **L1516 EN**: Separator comment used for visual grouping.
  **L1516 CN**: 用于视觉分组的分隔注释。
- **L1517 EN**: Comment records a pending task or caution: `TODO: copy-in and copy-out are now determined separately, in order`.
  **L1517 CN**: 注释记录待办事项或注意点：`TODO: copy-in and copy-out are now determined separately, in order`。
- **L1518 EN**: Comment explains nearby logic, intent, or metadata: `to allow more fine grained copying. While currently both copy-in`.
  **L1518 CN**: 注释说明附近代码的逻辑、意图或元数据：`to allow more fine grained copying. While currently both copy-in`。
- **L1519 EN**: Comment explains nearby logic, intent, or metadata: `and copy-out are must be done together, these copy operations could`.
  **L1519 CN**: 注释说明附近代码的逻辑、意图或元数据：`and copy-out are must be done together, these copy operations could`。
- **L1520 EN**: Comment records a pending task or caution: `be separated in the future. (This is related to TODO comment below.)`.
  **L1520 CN**: 注释记录待办事项或注意点：`be separated in the future. (This is related to TODO comment below.)`。
- **L1521 EN**: Separator comment used for visual grouping.
  **L1521 CN**: 用于视觉分组的分隔注释。
- **L1522 EN**: Comment records a pending task or caution: `TODO: for non-finalizable monomorphic derived type actual`.
  **L1522 CN**: 注释记录待办事项或注意点：`TODO: for non-finalizable monomorphic derived type actual`。
- **L1523 EN**: Comment explains nearby logic, intent, or metadata: `arguments associated with INTENT(OUT) dummy arguments`.
  **L1523 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments associated with INTENT(OUT) dummy arguments`。
- **L1524 EN**: Comment explains nearby logic, intent, or metadata: `we may avoid doing the copy and only allocate the temporary.`.
  **L1524 CN**: 注释说明附近代码的逻辑、意图或元数据：`we may avoid doing the copy and only allocate the temporary.`。
- **L1525 EN**: Comment explains nearby logic, intent, or metadata: `The codegen would do a "mold" allocation instead of "sourced"`.
  **L1525 CN**: 注释说明附近代码的逻辑、意图或元数据：`The codegen would do a "mold" allocation instead of "sourced"`。
- **L1526 EN**: Comment explains nearby logic, intent, or metadata: `allocation for the temp in this case. We can communicate`.
  **L1526 CN**: 注释说明附近代码的逻辑、意图或元数据：`allocation for the temp in this case. We can communicate`。
- **L1527 EN**: Comment explains nearby logic, intent, or metadata: `this to the codegen via some CopyInOp flag.`.
  **L1527 CN**: 注释说明附近代码的逻辑、意图或元数据：`this to the codegen via some CopyInOp flag.`。
- **L1528 EN**: Comment explains nearby logic, intent, or metadata: `This is a performance concern.`.
  **L1528 CN**: 注释说明附近代码的逻辑、意图或元数据：`This is a performance concern.`。
- **L1529 EN**: Executes a call or declaration centered on `genCopyIn`.
  **L1529 CN**: 执行以 `genCopyIn` 为核心的调用或声明。
- **L1530 EN**: Closes the current lexical scope or compound statement.
  **L1530 CN**: 结束当前词法作用域或复合语句块。
- **L1531 EN**: Transitions from the previous branch into the alternative path.
  **L1531 CN**: 从前一个分支过渡到备选路径。
- **L1532 EN**: Executes a call or declaration centered on `arg.entity->UnwrapExpr`.
  **L1532 CN**: 执行以 `arg.entity->UnwrapExpr` 为核心的调用或声明。
- **L1533 EN**: Checks an internal invariant in debug builds.
  **L1533 CN**: 在调试构建中检查内部不变式。
- **L1534 EN**: Comment explains nearby logic, intent, or metadata: `The actual is an expression value, place it into a temporary`.
  **L1534 CN**: 注释说明附近代码的逻辑、意图或元数据：`The actual is an expression value, place it into a temporary`。
- **L1535 EN**: Comment explains nearby logic, intent, or metadata: `and register the temporary destruction after the call.`.
  **L1535 CN**: 注释说明附近代码的逻辑、意图或元数据：`and register the temporary destruction after the call.`。
- **L1536 EN**: Initializes variable `storageType` from the right-hand expression.
  **L1536 CN**: 使用右侧表达式初始化变量 `storageType`。

### Lines 1537-1560

````cpp
    mlir::NamedAttribute byRefAttr = fir::getAdaptToByRefAttr(builder);
    hlfir::AssociateOp associate = hlfir::genAssociateExpr(
        loc, builder, entity, storageType, "", byRefAttr);
    entity = hlfir::Entity{associate.getBase()};
    preparedDummy.pushExprAssociateCleanUp(associate);
    // Rebox the actual argument to the dummy argument's type, and make sure
    // that we pass a contiguous entity (i.e. make copy-in, if needed).
    //
    // TODO: this can probably be optimized by associating the expression with
    // properly typed temporary, but this needs either a new operation or
    // making the hlfir.associate more complex.
    if (mustSetDynamicTypeToDummyType) {
      entity = genSetDynamicTypeToDummyType(entity);
      entity = genCopyIn(entity, /*doCopyOut=*/false);
    }
  }

  // Step 3: now that the dummy argument storage has been prepared, package
  // it according to the interface.
  mlir::Value addr;
  if (mlir::isa<fir::BoxCharType>(dummyTypeWithActualRank)) {
    // Cast the argument to match the volatility of the dummy argument.
    auto nonVolatileEntity = hlfir::Entity{builder.createVolatileCast(
        loc, fir::isa_volatile_type(dummyType), entity)};
````
- **L1537 EN**: Initializes variable `byRefAttr` from the right-hand expression.
  **L1537 CN**: 使用右侧表达式初始化变量 `byRefAttr`。
- **L1538 EN**: Continues logic associated with callable symbol `genAssociateExpr`.
  **L1538 CN**: 继续与可调用符号 `genAssociateExpr` 相关的逻辑。
- **L1539 EN**: Executes a standalone statement or declaration: `loc, builder, entity, storageType, "", byRefAttr);`.
  **L1539 CN**: 执行一条独立语句或声明：`loc, builder, entity, storageType, "", byRefAttr);`。
- **L1540 EN**: Executes a call or declaration centered on `hlfir::Entity{associate.getBase`.
  **L1540 CN**: 执行以 `hlfir::Entity{associate.getBase` 为核心的调用或声明。
- **L1541 EN**: Executes a call or declaration centered on `preparedDummy.pushExprAssociateCleanUp`.
  **L1541 CN**: 执行以 `preparedDummy.pushExprAssociateCleanUp` 为核心的调用或声明。
- **L1542 EN**: Comment explains nearby logic, intent, or metadata: `Rebox the actual argument to the dummy argument's type, and make sure`.
  **L1542 CN**: 注释说明附近代码的逻辑、意图或元数据：`Rebox the actual argument to the dummy argument's type, and make sure`。
- **L1543 EN**: Comment explains nearby logic, intent, or metadata: `that we pass a contiguous entity (i.e. make copy-in, if needed).`.
  **L1543 CN**: 注释说明附近代码的逻辑、意图或元数据：`that we pass a contiguous entity (i.e. make copy-in, if needed).`。
- **L1544 EN**: Separator comment used for visual grouping.
  **L1544 CN**: 用于视觉分组的分隔注释。
- **L1545 EN**: Comment records a pending task or caution: `TODO: this can probably be optimized by associating the expression with`.
  **L1545 CN**: 注释记录待办事项或注意点：`TODO: this can probably be optimized by associating the expression with`。
- **L1546 EN**: Comment explains nearby logic, intent, or metadata: `properly typed temporary, but this needs either a new operation or`.
  **L1546 CN**: 注释说明附近代码的逻辑、意图或元数据：`properly typed temporary, but this needs either a new operation or`。
- **L1547 EN**: Comment explains nearby logic, intent, or metadata: `making the hlfir.associate more complex.`.
  **L1547 CN**: 注释说明附近代码的逻辑、意图或元数据：`making the hlfir.associate more complex.`。
- **L1548 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1548 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1549 EN**: Executes a call or declaration centered on `genSetDynamicTypeToDummyType`.
  **L1549 CN**: 执行以 `genSetDynamicTypeToDummyType` 为核心的调用或声明。
- **L1550 EN**: Executes a call or declaration centered on `genCopyIn`.
  **L1550 CN**: 执行以 `genCopyIn` 为核心的调用或声明。
- **L1551 EN**: Closes the current lexical scope or compound statement.
  **L1551 CN**: 结束当前词法作用域或复合语句块。
- **L1552 EN**: Closes the current lexical scope or compound statement.
  **L1552 CN**: 结束当前词法作用域或复合语句块。
- **L1553 EN**: Blank line separating nearby declarations or logic blocks.
  **L1553 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1554 EN**: Comment explains nearby logic, intent, or metadata: `Step 3: now that the dummy argument storage has been prepared, package`.
  **L1554 CN**: 注释说明附近代码的逻辑、意图或元数据：`Step 3: now that the dummy argument storage has been prepared, package`。
- **L1555 EN**: Comment explains nearby logic, intent, or metadata: `it according to the interface.`.
  **L1555 CN**: 注释说明附近代码的逻辑、意图或元数据：`it according to the interface.`。
- **L1556 EN**: Executes a standalone statement or declaration: `mlir::Value addr;`.
  **L1556 CN**: 执行一条独立语句或声明：`mlir::Value addr;`。
- **L1557 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1557 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1558 EN**: Comment explains nearby logic, intent, or metadata: `Cast the argument to match the volatility of the dummy argument.`.
  **L1558 CN**: 注释说明附近代码的逻辑、意图或元数据：`Cast the argument to match the volatility of the dummy argument.`。
- **L1559 EN**: Continues logic associated with callable symbol `createVolatileCast`.
  **L1559 CN**: 继续与可调用符号 `createVolatileCast` 相关的逻辑。
- **L1560 EN**: Executes a call or declaration centered on `fir::isa_volatile_type`.
  **L1560 CN**: 执行以 `fir::isa_volatile_type` 为核心的调用或声明。

### Lines 1561-1584

````cpp
    addr = hlfir::genVariableBoxChar(loc, builder, nonVolatileEntity);
  } else if (mlir::isa<fir::BaseBoxType>(dummyTypeWithActualRank)) {
    entity = hlfir::genVariableBox(loc, builder, entity);
    // Ensures the box has the right attributes and that it holds an
    // addendum if needed.
    fir::BaseBoxType actualBoxType =
        mlir::cast<fir::BaseBoxType>(entity.getType());
    mlir::Type boxEleType = actualBoxType.getEleTy();
    // For now, assume it is not OK to pass the allocatable/pointer
    // descriptor to a non pointer/allocatable dummy. That is a strict
    // interpretation of 18.3.6 point 4 that stipulates the descriptor
    // has the dummy attributes in BIND(C) contexts.
    const bool actualBoxHasAllocatableOrPointerFlag =
        fir::isa_ref_type(boxEleType);
    // Fortran 2018 18.5.3, pp3: BIND(C) non pointer allocatable descriptors
    // must have zero lower bounds.
    bool needsZeroLowerBounds = callContext.isBindcCall() && entity.isArray();
    // On the callee side, the current code generated for unlimited
    // polymorphic might unconditionally read the addendum. Intrinsic type
    // descriptors may not have an addendum, the rebox below will create a
    // descriptor with an addendum in such case.
    const bool actualBoxHasAddendum = fir::boxHasAddendum(actualBoxType);
    const bool needToAddAddendum =
        fir::isUnlimitedPolymorphicType(dummyTypeWithActualRank) &&
````
- **L1561 EN**: Executes a call or declaration centered on `hlfir::genVariableBoxChar`.
  **L1561 CN**: 执行以 `hlfir::genVariableBoxChar` 为核心的调用或声明。
- **L1562 EN**: Transitions from the previous branch into an `else if` condition.
  **L1562 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1563 EN**: Executes a call or declaration centered on `hlfir::genVariableBox`.
  **L1563 CN**: 执行以 `hlfir::genVariableBox` 为核心的调用或声明。
- **L1564 EN**: Comment explains nearby logic, intent, or metadata: `Ensures the box has the right attributes and that it holds an`.
  **L1564 CN**: 注释说明附近代码的逻辑、意图或元数据：`Ensures the box has the right attributes and that it holds an`。
- **L1565 EN**: Comment explains nearby logic, intent, or metadata: `addendum if needed.`.
  **L1565 CN**: 注释说明附近代码的逻辑、意图或元数据：`addendum if needed.`。
- **L1566 EN**: Continues the surrounding expression or declaration: `fir::BaseBoxType actualBoxType =`.
  **L1566 CN**: 继续构造周围的表达式或声明：`fir::BaseBoxType actualBoxType =`。
- **L1567 EN**: Executes a call or declaration centered on `mlir::cast<fir::BaseBoxType>`.
  **L1567 CN**: 执行以 `mlir::cast<fir::BaseBoxType>` 为核心的调用或声明。
- **L1568 EN**: Initializes variable `boxEleType` from the right-hand expression.
  **L1568 CN**: 使用右侧表达式初始化变量 `boxEleType`。
- **L1569 EN**: Comment explains nearby logic, intent, or metadata: `For now, assume it is not OK to pass the allocatable/pointer`.
  **L1569 CN**: 注释说明附近代码的逻辑、意图或元数据：`For now, assume it is not OK to pass the allocatable/pointer`。
- **L1570 EN**: Comment explains nearby logic, intent, or metadata: `descriptor to a non pointer/allocatable dummy. That is a strict`.
  **L1570 CN**: 注释说明附近代码的逻辑、意图或元数据：`descriptor to a non pointer/allocatable dummy. That is a strict`。
- **L1571 EN**: Comment explains nearby logic, intent, or metadata: `interpretation of 18.3.6 point 4 that stipulates the descriptor`.
  **L1571 CN**: 注释说明附近代码的逻辑、意图或元数据：`interpretation of 18.3.6 point 4 that stipulates the descriptor`。
- **L1572 EN**: Comment explains nearby logic, intent, or metadata: `has the dummy attributes in BIND(C) contexts.`.
  **L1572 CN**: 注释说明附近代码的逻辑、意图或元数据：`has the dummy attributes in BIND(C) contexts.`。
- **L1573 EN**: Continues the surrounding expression or declaration: `const bool actualBoxHasAllocatableOrPointerFlag =`.
  **L1573 CN**: 继续构造周围的表达式或声明：`const bool actualBoxHasAllocatableOrPointerFlag =`。
- **L1574 EN**: Executes a call or declaration centered on `fir::isa_ref_type`.
  **L1574 CN**: 执行以 `fir::isa_ref_type` 为核心的调用或声明。
- **L1575 EN**: Comment explains nearby logic, intent, or metadata: `Fortran 2018 18.5.3, pp3: BIND(C) non pointer allocatable descriptors`.
  **L1575 CN**: 注释说明附近代码的逻辑、意图或元数据：`Fortran 2018 18.5.3, pp3: BIND(C) non pointer allocatable descriptors`。
- **L1576 EN**: Comment explains nearby logic, intent, or metadata: `must have zero lower bounds.`.
  **L1576 CN**: 注释说明附近代码的逻辑、意图或元数据：`must have zero lower bounds.`。
- **L1577 EN**: Initializes variable `needsZeroLowerBounds` from the right-hand expression.
  **L1577 CN**: 使用右侧表达式初始化变量 `needsZeroLowerBounds`。
- **L1578 EN**: Comment explains nearby logic, intent, or metadata: `On the callee side, the current code generated for unlimited`.
  **L1578 CN**: 注释说明附近代码的逻辑、意图或元数据：`On the callee side, the current code generated for unlimited`。
- **L1579 EN**: Comment explains nearby logic, intent, or metadata: `polymorphic might unconditionally read the addendum. Intrinsic type`.
  **L1579 CN**: 注释说明附近代码的逻辑、意图或元数据：`polymorphic might unconditionally read the addendum. Intrinsic type`。
- **L1580 EN**: Comment explains nearby logic, intent, or metadata: `descriptors may not have an addendum, the rebox below will create a`.
  **L1580 CN**: 注释说明附近代码的逻辑、意图或元数据：`descriptors may not have an addendum, the rebox below will create a`。
- **L1581 EN**: Comment explains nearby logic, intent, or metadata: `descriptor with an addendum in such case.`.
  **L1581 CN**: 注释说明附近代码的逻辑、意图或元数据：`descriptor with an addendum in such case.`。
- **L1582 EN**: Initializes variable `actualBoxHasAddendum` from the right-hand expression.
  **L1582 CN**: 使用右侧表达式初始化变量 `actualBoxHasAddendum`。
- **L1583 EN**: Continues the surrounding expression or declaration: `const bool needToAddAddendum =`.
  **L1583 CN**: 继续构造周围的表达式或声明：`const bool needToAddAddendum =`。
- **L1584 EN**: Continues logic associated with callable symbol `isUnlimitedPolymorphicType`.
  **L1584 CN**: 继续与可调用符号 `isUnlimitedPolymorphicType` 相关的逻辑。

### Lines 1585-1608

````cpp
        !actualBoxHasAddendum;
    if (needToAddAddendum || actualBoxHasAllocatableOrPointerFlag ||
        needsZeroLowerBounds) {
      if (actualIsAssumedRank) {
        auto lbModifier = needsZeroLowerBounds
                              ? fir::LowerBoundModifierAttribute::SetToZeroes
                              : fir::LowerBoundModifierAttribute::SetToOnes;
        entity = hlfir::Entity{fir::ReboxAssumedRankOp::create(
            builder, loc, dummyTypeWithActualRank, entity, lbModifier)};
      } else {
        mlir::Value shift{};
        if (needsZeroLowerBounds)
          shift = getZeroLowerBounds(loc, builder, entity);
        entity = hlfir::Entity{fir::ReboxOp::create(
            builder, loc, dummyTypeWithActualRank, entity, /*shape=*/shift,
            /*slice=*/mlir::Value{})};
      }
    }
    addr = entity;
  } else {
    addr = hlfir::genVariableRawAddress(loc, builder, entity);
  }

  // If the volatility of the input type does not match the dummy type,
````
- **L1585 EN**: Executes a standalone statement or declaration: `!actualBoxHasAddendum;`.
  **L1585 CN**: 执行一条独立语句或声明：`!actualBoxHasAddendum;`。
- **L1586 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1586 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1587 EN**: Continues the surrounding expression or declaration: `needsZeroLowerBounds) {`.
  **L1587 CN**: 继续构造周围的表达式或声明：`needsZeroLowerBounds) {`。
- **L1588 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1588 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1589 EN**: Continues the surrounding expression or declaration: `auto lbModifier = needsZeroLowerBounds`.
  **L1589 CN**: 继续构造周围的表达式或声明：`auto lbModifier = needsZeroLowerBounds`。
- **L1590 EN**: Continues the surrounding expression or declaration: `? fir::LowerBoundModifierAttribute::SetToZeroes`.
  **L1590 CN**: 继续构造周围的表达式或声明：`? fir::LowerBoundModifierAttribute::SetToZeroes`。
- **L1591 EN**: Executes a standalone statement or declaration: `: fir::LowerBoundModifierAttribute::SetToOnes;`.
  **L1591 CN**: 执行一条独立语句或声明：`: fir::LowerBoundModifierAttribute::SetToOnes;`。
- **L1592 EN**: Continues logic associated with callable symbol `create`.
  **L1592 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1593 EN**: Executes a standalone statement or declaration: `builder, loc, dummyTypeWithActualRank, entity, lbModifier)};`.
  **L1593 CN**: 执行一条独立语句或声明：`builder, loc, dummyTypeWithActualRank, entity, lbModifier)};`。
- **L1594 EN**: Transitions from the previous branch into the alternative path.
  **L1594 CN**: 从前一个分支过渡到备选路径。
- **L1595 EN**: Executes a standalone statement or declaration: `mlir::Value shift{};`.
  **L1595 CN**: 执行一条独立语句或声明：`mlir::Value shift{};`。
- **L1596 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1596 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1597 EN**: Executes a call or declaration centered on `getZeroLowerBounds`.
  **L1597 CN**: 执行以 `getZeroLowerBounds` 为核心的调用或声明。
- **L1598 EN**: Continues logic associated with callable symbol `create`.
  **L1598 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, dummyTypeWithActualRank, entity, /*shape=*/shift,`.
  **L1599 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, dummyTypeWithActualRank, entity, /*shape=*/shift,`。
- **L1600 EN**: Comment explains nearby logic, intent, or metadata: `slice=*/mlir::Value{})};`.
  **L1600 CN**: 注释说明附近代码的逻辑、意图或元数据：`slice=*/mlir::Value{})};`。
- **L1601 EN**: Closes the current lexical scope or compound statement.
  **L1601 CN**: 结束当前词法作用域或复合语句块。
- **L1602 EN**: Closes the current lexical scope or compound statement.
  **L1602 CN**: 结束当前词法作用域或复合语句块。
- **L1603 EN**: Executes a standalone statement or declaration: `addr = entity;`.
  **L1603 CN**: 执行一条独立语句或声明：`addr = entity;`。
- **L1604 EN**: Transitions from the previous branch into the alternative path.
  **L1604 CN**: 从前一个分支过渡到备选路径。
- **L1605 EN**: Executes a call or declaration centered on `hlfir::genVariableRawAddress`.
  **L1605 CN**: 执行以 `hlfir::genVariableRawAddress` 为核心的调用或声明。
- **L1606 EN**: Closes the current lexical scope or compound statement.
  **L1606 CN**: 结束当前词法作用域或复合语句块。
- **L1607 EN**: Blank line separating nearby declarations or logic blocks.
  **L1607 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1608 EN**: Comment explains nearby logic, intent, or metadata: `If the volatility of the input type does not match the dummy type,`.
  **L1608 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the volatility of the input type does not match the dummy type,`。

### Lines 1609-1632

````cpp
  // we need to cast the argument.
  const bool isToTypeVolatile = fir::isa_volatile_type(dummyTypeWithActualRank);
  addr = builder.createVolatileCast(loc, isToTypeVolatile, addr);

  // For ranked actual passed to assumed-rank dummy, the cast to assumed-rank
  // box is inserted when building the fir.call op. Inserting it here would
  // cause the fir.if results to be assumed-rank in case of OPTIONAL dummy,
  // causing extra runtime costs due to the unknown runtime size of assumed-rank
  // descriptors.
  // For TKR dummy characters, the boxchar creation also happens later when
  // creating the fir.call .
  preparedDummy.dummy =
      builder.createConvert(loc, dummyTypeWithActualRank, addr);
  return preparedDummy;
}

/// When dummy is not ALLOCATABLE, POINTER and is not passed in register,
/// prepare the actual argument according to the interface, taking care
/// of any optional aspect.
static PreparedDummyArgument prepareUserCallActualArgument(
    mlir::Location loc, fir::FirOpBuilder &builder,
    const Fortran::lower::PreparedActualArgument &preparedActual,
    mlir::Type dummyType,
    const Fortran::lower::CallerInterface::PassedEntity &arg,
````
- **L1609 EN**: Comment explains nearby logic, intent, or metadata: `we need to cast the argument.`.
  **L1609 CN**: 注释说明附近代码的逻辑、意图或元数据：`we need to cast the argument.`。
- **L1610 EN**: Initializes variable `isToTypeVolatile` from the right-hand expression.
  **L1610 CN**: 使用右侧表达式初始化变量 `isToTypeVolatile`。
- **L1611 EN**: Executes a call or declaration centered on `builder.createVolatileCast`.
  **L1611 CN**: 执行以 `builder.createVolatileCast` 为核心的调用或声明。
- **L1612 EN**: Blank line separating nearby declarations or logic blocks.
  **L1612 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1613 EN**: Comment explains nearby logic, intent, or metadata: `For ranked actual passed to assumed-rank dummy, the cast to assumed-rank`.
  **L1613 CN**: 注释说明附近代码的逻辑、意图或元数据：`For ranked actual passed to assumed-rank dummy, the cast to assumed-rank`。
- **L1614 EN**: Comment explains nearby logic, intent, or metadata: `box is inserted when building the fir.call op. Inserting it here would`.
  **L1614 CN**: 注释说明附近代码的逻辑、意图或元数据：`box is inserted when building the fir.call op. Inserting it here would`。
- **L1615 EN**: Comment explains nearby logic, intent, or metadata: `cause the fir.if results to be assumed-rank in case of OPTIONAL dummy,`.
  **L1615 CN**: 注释说明附近代码的逻辑、意图或元数据：`cause the fir.if results to be assumed-rank in case of OPTIONAL dummy,`。
- **L1616 EN**: Comment explains nearby logic, intent, or metadata: `causing extra runtime costs due to the unknown runtime size of assumed-rank`.
  **L1616 CN**: 注释说明附近代码的逻辑、意图或元数据：`causing extra runtime costs due to the unknown runtime size of assumed-rank`。
- **L1617 EN**: Comment explains nearby logic, intent, or metadata: `descriptors.`.
  **L1617 CN**: 注释说明附近代码的逻辑、意图或元数据：`descriptors.`。
- **L1618 EN**: Comment explains nearby logic, intent, or metadata: `For TKR dummy characters, the boxchar creation also happens later when`.
  **L1618 CN**: 注释说明附近代码的逻辑、意图或元数据：`For TKR dummy characters, the boxchar creation also happens later when`。
- **L1619 EN**: Comment explains nearby logic, intent, or metadata: `creating the fir.call .`.
  **L1619 CN**: 注释说明附近代码的逻辑、意图或元数据：`creating the fir.call .`。
- **L1620 EN**: Continues the surrounding expression or declaration: `preparedDummy.dummy =`.
  **L1620 CN**: 继续构造周围的表达式或声明：`preparedDummy.dummy =`。
- **L1621 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L1621 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L1622 EN**: Returns from the current function with `preparedDummy`.
  **L1622 CN**: 以 `preparedDummy` 从当前函数返回。
- **L1623 EN**: Closes the current lexical scope or compound statement.
  **L1623 CN**: 结束当前词法作用域或复合语句块。
- **L1624 EN**: Blank line separating nearby declarations or logic blocks.
  **L1624 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1625 EN**: Comment explains nearby logic, intent, or metadata: `When dummy is not ALLOCATABLE, POINTER and is not passed in register,`.
  **L1625 CN**: 注释说明附近代码的逻辑、意图或元数据：`When dummy is not ALLOCATABLE, POINTER and is not passed in register,`。
- **L1626 EN**: Comment explains nearby logic, intent, or metadata: `prepare the actual argument according to the interface, taking care`.
  **L1626 CN**: 注释说明附近代码的逻辑、意图或元数据：`prepare the actual argument according to the interface, taking care`。
- **L1627 EN**: Comment explains nearby logic, intent, or metadata: `of any optional aspect.`.
  **L1627 CN**: 注释说明附近代码的逻辑、意图或元数据：`of any optional aspect.`。
- **L1628 EN**: Continues logic associated with callable symbol `prepareUserCallActualArgument`.
  **L1628 CN**: 继续与可调用符号 `prepareUserCallActualArgument` 相关的逻辑。
- **L1629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, fir::FirOpBuilder &builder,`.
  **L1629 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, fir::FirOpBuilder &builder,`。
- **L1630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArgument &preparedActual,`.
  **L1630 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArgument &preparedActual,`。
- **L1631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Type dummyType,`.
  **L1631 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Type dummyType,`。
- **L1632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::CallerInterface::PassedEntity &arg,`.
  **L1632 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::CallerInterface::PassedEntity &arg,`。

### Lines 1633-1656

````cpp
    CallContext &callContext) {
  if (!preparedActual.handleDynamicOptional())
    return preparePresentUserCallActualArgument(loc, builder, preparedActual,
                                                dummyType, arg, callContext);

  // Conditional dummy argument preparation. The actual may be absent
  // at runtime, causing any addressing, copy, and packaging to have
  // undefined behavior.
  // To simplify the handling of this case, the "normal" dummy preparation
  // helper is used, except its generated code is wrapped inside a
  // fir.if(present).
  mlir::Value isPresent = preparedActual.getIsPresent();
  mlir::OpBuilder::InsertPoint insertPt = builder.saveInsertionPoint();

  // Code generated in a preparation block that will become the
  // "then" block in "if (present) then {} else {}". The reason
  // for this unusual if/then/else generation is that the number
  // and types of the if results will depend on how the argument
  // is prepared, and forecasting that here would be brittle.
  auto badIfOp = fir::IfOp::create(builder, loc, dummyType, isPresent,
                                   /*withElseRegion=*/false);
  mlir::Block *preparationBlock = &badIfOp.getThenRegion().front();
  builder.setInsertionPointToStart(preparationBlock);
  PreparedDummyArgument unconditionalDummy =
````
- **L1633 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L1633 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L1634 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1634 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1635 EN**: Returns from the current function with `preparePresentUserCallActualArgument(loc, builder, preparedActual,`.
  **L1635 CN**: 以 `preparePresentUserCallActualArgument(loc, builder, preparedActual,` 从当前函数返回。
- **L1636 EN**: Executes a standalone statement or declaration: `dummyType, arg, callContext);`.
  **L1636 CN**: 执行一条独立语句或声明：`dummyType, arg, callContext);`。
- **L1637 EN**: Blank line separating nearby declarations or logic blocks.
  **L1637 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1638 EN**: Comment explains nearby logic, intent, or metadata: `Conditional dummy argument preparation. The actual may be absent`.
  **L1638 CN**: 注释说明附近代码的逻辑、意图或元数据：`Conditional dummy argument preparation. The actual may be absent`。
- **L1639 EN**: Comment explains nearby logic, intent, or metadata: `at runtime, causing any addressing, copy, and packaging to have`.
  **L1639 CN**: 注释说明附近代码的逻辑、意图或元数据：`at runtime, causing any addressing, copy, and packaging to have`。
- **L1640 EN**: Comment explains nearby logic, intent, or metadata: `undefined behavior.`.
  **L1640 CN**: 注释说明附近代码的逻辑、意图或元数据：`undefined behavior.`。
- **L1641 EN**: Comment explains nearby logic, intent, or metadata: `To simplify the handling of this case, the "normal" dummy preparation`.
  **L1641 CN**: 注释说明附近代码的逻辑、意图或元数据：`To simplify the handling of this case, the "normal" dummy preparation`。
- **L1642 EN**: Comment explains nearby logic, intent, or metadata: `helper is used, except its generated code is wrapped inside a`.
  **L1642 CN**: 注释说明附近代码的逻辑、意图或元数据：`helper is used, except its generated code is wrapped inside a`。
- **L1643 EN**: Comment explains nearby logic, intent, or metadata: `fir.if(present).`.
  **L1643 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.if(present).`。
- **L1644 EN**: Initializes variable `isPresent` from the right-hand expression.
  **L1644 CN**: 使用右侧表达式初始化变量 `isPresent`。
- **L1645 EN**: Initializes variable `insertPt` from the right-hand expression.
  **L1645 CN**: 使用右侧表达式初始化变量 `insertPt`。
- **L1646 EN**: Blank line separating nearby declarations or logic blocks.
  **L1646 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1647 EN**: Comment explains nearby logic, intent, or metadata: `Code generated in a preparation block that will become the`.
  **L1647 CN**: 注释说明附近代码的逻辑、意图或元数据：`Code generated in a preparation block that will become the`。
- **L1648 EN**: Comment explains nearby logic, intent, or metadata: `"then" block in "if (present) then {} else {}". The reason`.
  **L1648 CN**: 注释说明附近代码的逻辑、意图或元数据：`"then" block in "if (present) then {} else {}". The reason`。
- **L1649 EN**: Comment explains nearby logic, intent, or metadata: `for this unusual if/then/else generation is that the number`.
  **L1649 CN**: 注释说明附近代码的逻辑、意图或元数据：`for this unusual if/then/else generation is that the number`。
- **L1650 EN**: Comment explains nearby logic, intent, or metadata: `and types of the if results will depend on how the argument`.
  **L1650 CN**: 注释说明附近代码的逻辑、意图或元数据：`and types of the if results will depend on how the argument`。
- **L1651 EN**: Comment explains nearby logic, intent, or metadata: `is prepared, and forecasting that here would be brittle.`.
  **L1651 CN**: 注释说明附近代码的逻辑、意图或元数据：`is prepared, and forecasting that here would be brittle.`。
- **L1652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto badIfOp = fir::IfOp::create(builder, loc, dummyType, isPresent,`.
  **L1652 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto badIfOp = fir::IfOp::create(builder, loc, dummyType, isPresent,`。
- **L1653 EN**: Comment explains nearby logic, intent, or metadata: `withElseRegion=*/false);`.
  **L1653 CN**: 注释说明附近代码的逻辑、意图或元数据：`withElseRegion=*/false);`。
- **L1654 EN**: Executes a call or declaration centered on `&badIfOp.getThenRegion`.
  **L1654 CN**: 执行以 `&badIfOp.getThenRegion` 为核心的调用或声明。
- **L1655 EN**: Executes a call or declaration centered on `builder.setInsertionPointToStart`.
  **L1655 CN**: 执行以 `builder.setInsertionPointToStart` 为核心的调用或声明。
- **L1656 EN**: Continues the surrounding expression or declaration: `PreparedDummyArgument unconditionalDummy =`.
  **L1656 CN**: 继续构造周围的表达式或声明：`PreparedDummyArgument unconditionalDummy =`。

### Lines 1657-1680

````cpp
      preparePresentUserCallActualArgument(loc, builder, preparedActual,
                                           dummyType, arg, callContext);
  builder.restoreInsertionPoint(insertPt);

  // TODO: when forwarding an optional to an optional of the same kind
  // (i.e, unconditionalDummy.dummy was not created in preparationBlock),
  // the if/then/else generation could be skipped to improve the generated
  // code.

  // Now that the result types of the ifOp can be deduced, generate
  // the "real" ifOp (operation result types cannot be changed, so
  // badIfOp cannot be modified and used here).
  llvm::SmallVector<mlir::Type> ifOpResultTypes;
  ConditionallyPreparedDummy conditionalDummy(unconditionalDummy);
  auto ifOp = fir::IfOp::create(builder, loc,
                                conditionalDummy.getIfResulTypes(), isPresent,
                                /*withElseRegion=*/true);
  // Move "preparationBlock" into the "then" of the new
  // fir.if operation and create fir.result propagating
  // unconditionalDummy.
  preparationBlock->moveBefore(&ifOp.getThenRegion().back());
  ifOp.getThenRegion().back().erase();
  builder.setInsertionPointToEnd(&ifOp.getThenRegion().front());
  conditionalDummy.genThenResult(loc, builder);
````
- **L1657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `preparePresentUserCallActualArgument(loc, builder, preparedActual,`.
  **L1657 CN**: 继续一个多行参数列表、初始化器或聚合项：`preparePresentUserCallActualArgument(loc, builder, preparedActual,`。
- **L1658 EN**: Executes a standalone statement or declaration: `dummyType, arg, callContext);`.
  **L1658 CN**: 执行一条独立语句或声明：`dummyType, arg, callContext);`。
- **L1659 EN**: Executes a call or declaration centered on `builder.restoreInsertionPoint`.
  **L1659 CN**: 执行以 `builder.restoreInsertionPoint` 为核心的调用或声明。
- **L1660 EN**: Blank line separating nearby declarations or logic blocks.
  **L1660 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1661 EN**: Comment records a pending task or caution: `TODO: when forwarding an optional to an optional of the same kind`.
  **L1661 CN**: 注释记录待办事项或注意点：`TODO: when forwarding an optional to an optional of the same kind`。
- **L1662 EN**: Comment explains nearby logic, intent, or metadata: `(i.e, unconditionalDummy.dummy was not created in preparationBlock),`.
  **L1662 CN**: 注释说明附近代码的逻辑、意图或元数据：`(i.e, unconditionalDummy.dummy was not created in preparationBlock),`。
- **L1663 EN**: Comment explains nearby logic, intent, or metadata: `the if/then/else generation could be skipped to improve the generated`.
  **L1663 CN**: 注释说明附近代码的逻辑、意图或元数据：`the if/then/else generation could be skipped to improve the generated`。
- **L1664 EN**: Comment explains nearby logic, intent, or metadata: `code.`.
  **L1664 CN**: 注释说明附近代码的逻辑、意图或元数据：`code.`。
- **L1665 EN**: Blank line separating nearby declarations or logic blocks.
  **L1665 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1666 EN**: Comment explains nearby logic, intent, or metadata: `Now that the result types of the ifOp can be deduced, generate`.
  **L1666 CN**: 注释说明附近代码的逻辑、意图或元数据：`Now that the result types of the ifOp can be deduced, generate`。
- **L1667 EN**: Comment explains nearby logic, intent, or metadata: `the "real" ifOp (operation result types cannot be changed, so`.
  **L1667 CN**: 注释说明附近代码的逻辑、意图或元数据：`the "real" ifOp (operation result types cannot be changed, so`。
- **L1668 EN**: Comment explains nearby logic, intent, or metadata: `badIfOp cannot be modified and used here).`.
  **L1668 CN**: 注释说明附近代码的逻辑、意图或元数据：`badIfOp cannot be modified and used here).`。
- **L1669 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Type> ifOpResultTypes;`.
  **L1669 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Type> ifOpResultTypes;`。
- **L1670 EN**: Executes a call or declaration centered on `conditionalDummy`.
  **L1670 CN**: 执行以 `conditionalDummy` 为核心的调用或声明。
- **L1671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto ifOp = fir::IfOp::create(builder, loc,`.
  **L1671 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto ifOp = fir::IfOp::create(builder, loc,`。
- **L1672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `conditionalDummy.getIfResulTypes(), isPresent,`.
  **L1672 CN**: 继续一个多行参数列表、初始化器或聚合项：`conditionalDummy.getIfResulTypes(), isPresent,`。
- **L1673 EN**: Comment explains nearby logic, intent, or metadata: `withElseRegion=*/true);`.
  **L1673 CN**: 注释说明附近代码的逻辑、意图或元数据：`withElseRegion=*/true);`。
- **L1674 EN**: Comment explains nearby logic, intent, or metadata: `Move "preparationBlock" into the "then" of the new`.
  **L1674 CN**: 注释说明附近代码的逻辑、意图或元数据：`Move "preparationBlock" into the "then" of the new`。
- **L1675 EN**: Comment explains nearby logic, intent, or metadata: `fir.if operation and create fir.result propagating`.
  **L1675 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.if operation and create fir.result propagating`。
- **L1676 EN**: Comment explains nearby logic, intent, or metadata: `unconditionalDummy.`.
  **L1676 CN**: 注释说明附近代码的逻辑、意图或元数据：`unconditionalDummy.`。
- **L1677 EN**: Executes a call or declaration centered on `preparationBlock->moveBefore`.
  **L1677 CN**: 执行以 `preparationBlock->moveBefore` 为核心的调用或声明。
- **L1678 EN**: Executes a call or declaration centered on `ifOp.getThenRegion`.
  **L1678 CN**: 执行以 `ifOp.getThenRegion` 为核心的调用或声明。
- **L1679 EN**: Executes a call or declaration centered on `builder.setInsertionPointToEnd`.
  **L1679 CN**: 执行以 `builder.setInsertionPointToEnd` 为核心的调用或声明。
- **L1680 EN**: Executes a call or declaration centered on `conditionalDummy.genThenResult`.
  **L1680 CN**: 执行以 `conditionalDummy.genThenResult` 为核心的调用或声明。

### Lines 1681-1704

````cpp

  // Generate "else" branch with returning absent values.
  builder.setInsertionPointToStart(&ifOp.getElseRegion().front());
  conditionalDummy.genElseResult(loc, builder);

  // Build dummy from IfOpResults.
  builder.setInsertionPointAfter(ifOp);
  PreparedDummyArgument result =
      conditionalDummy.getPreparedDummy(ifOp, unconditionalDummy);
  badIfOp->erase();
  return result;
}

/// Prepare actual argument for a procedure pointer dummy.
static PreparedDummyArgument prepareProcedurePointerActualArgument(
    mlir::Location loc, fir::FirOpBuilder &builder,
    const Fortran::lower::PreparedActualArgument &preparedActual,
    mlir::Type dummyType,
    const Fortran::lower::CallerInterface::PassedEntity &arg,
    CallContext &callContext) {

  // NULL() actual to procedure pointer dummy
  if (Fortran::evaluate::UnwrapExpr<Fortran::evaluate::NullPointer>(
          *arg.entity) &&
````
- **L1681 EN**: Blank line separating nearby declarations or logic blocks.
  **L1681 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1682 EN**: Comment explains nearby logic, intent, or metadata: `Generate "else" branch with returning absent values.`.
  **L1682 CN**: 注释说明附近代码的逻辑、意图或元数据：`Generate "else" branch with returning absent values.`。
- **L1683 EN**: Executes a call or declaration centered on `builder.setInsertionPointToStart`.
  **L1683 CN**: 执行以 `builder.setInsertionPointToStart` 为核心的调用或声明。
- **L1684 EN**: Executes a call or declaration centered on `conditionalDummy.genElseResult`.
  **L1684 CN**: 执行以 `conditionalDummy.genElseResult` 为核心的调用或声明。
- **L1685 EN**: Blank line separating nearby declarations or logic blocks.
  **L1685 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1686 EN**: Comment explains nearby logic, intent, or metadata: `Build dummy from IfOpResults.`.
  **L1686 CN**: 注释说明附近代码的逻辑、意图或元数据：`Build dummy from IfOpResults.`。
- **L1687 EN**: Executes a call or declaration centered on `builder.setInsertionPointAfter`.
  **L1687 CN**: 执行以 `builder.setInsertionPointAfter` 为核心的调用或声明。
- **L1688 EN**: Continues the surrounding expression or declaration: `PreparedDummyArgument result =`.
  **L1688 CN**: 继续构造周围的表达式或声明：`PreparedDummyArgument result =`。
- **L1689 EN**: Executes a call or declaration centered on `conditionalDummy.getPreparedDummy`.
  **L1689 CN**: 执行以 `conditionalDummy.getPreparedDummy` 为核心的调用或声明。
- **L1690 EN**: Executes a call or declaration centered on `badIfOp->erase`.
  **L1690 CN**: 执行以 `badIfOp->erase` 为核心的调用或声明。
- **L1691 EN**: Returns from the current function with `result`.
  **L1691 CN**: 以 `result` 从当前函数返回。
- **L1692 EN**: Closes the current lexical scope or compound statement.
  **L1692 CN**: 结束当前词法作用域或复合语句块。
- **L1693 EN**: Blank line separating nearby declarations or logic blocks.
  **L1693 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1694 EN**: Comment explains nearby logic, intent, or metadata: `Prepare actual argument for a procedure pointer dummy.`.
  **L1694 CN**: 注释说明附近代码的逻辑、意图或元数据：`Prepare actual argument for a procedure pointer dummy.`。
- **L1695 EN**: Continues logic associated with callable symbol `prepareProcedurePointerActualArgument`.
  **L1695 CN**: 继续与可调用符号 `prepareProcedurePointerActualArgument` 相关的逻辑。
- **L1696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, fir::FirOpBuilder &builder,`.
  **L1696 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, fir::FirOpBuilder &builder,`。
- **L1697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArgument &preparedActual,`.
  **L1697 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArgument &preparedActual,`。
- **L1698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Type dummyType,`.
  **L1698 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Type dummyType,`。
- **L1699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::CallerInterface::PassedEntity &arg,`.
  **L1699 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::CallerInterface::PassedEntity &arg,`。
- **L1700 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L1700 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L1701 EN**: Blank line separating nearby declarations or logic blocks.
  **L1701 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1702 EN**: Comment explains nearby logic, intent, or metadata: `NULL() actual to procedure pointer dummy`.
  **L1702 CN**: 注释说明附近代码的逻辑、意图或元数据：`NULL() actual to procedure pointer dummy`。
- **L1703 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1703 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1704 EN**: Comment explains nearby logic, intent, or metadata: `arg.entity) &&`.
  **L1704 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg.entity) &&`。

### Lines 1705-1728

````cpp
      fir::isBoxProcAddressType(dummyType)) {
    auto boxTy{Fortran::lower::getUntypedBoxProcType(builder.getContext())};
    auto tempBoxProc{builder.createTemporary(loc, boxTy)};
    hlfir::Entity nullBoxProc(
        fir::factory::createNullBoxProc(builder, loc, boxTy));
    fir::StoreOp::create(builder, loc, nullBoxProc, tempBoxProc);
    return PreparedDummyArgument{tempBoxProc, /*cleanups=*/{}};
  }
  hlfir::Entity actual = preparedActual.getActual(loc, builder);
  if (actual.isProcedurePointer())
    return PreparedDummyArgument{actual, /*cleanups=*/{}};
  assert(actual.isProcedure());
  // Procedure actual to procedure pointer dummy.
  auto tempBoxProc{builder.createTemporary(loc, actual.getType())};
  fir::StoreOp::create(builder, loc, actual, tempBoxProc);
  return PreparedDummyArgument{tempBoxProc, /*cleanups=*/{}};
}

/// Prepare arguments of calls to user procedures with actual arguments that
/// have been pre-lowered but not yet prepared according to the interface.
void prepareUserCallArguments(
    Fortran::lower::PreparedActualArguments &loweredActuals,
    Fortran::lower::CallerInterface &caller, mlir::FunctionType callSiteType,
    CallContext &callContext, llvm::SmallVector<CallCleanUp> &callCleanUps) {
````
- **L1705 EN**: Starts a function, method, lambda, or structured scope: `fir::isBoxProcAddressType(dummyType)) {`.
  **L1705 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::isBoxProcAddressType(dummyType)) {`。
- **L1706 EN**: Executes a call or declaration centered on `boxTy{Fortran::lower::getUntypedBoxProcType`.
  **L1706 CN**: 执行以 `boxTy{Fortran::lower::getUntypedBoxProcType` 为核心的调用或声明。
- **L1707 EN**: Executes a call or declaration centered on `tempBoxProc{builder.createTemporary`.
  **L1707 CN**: 执行以 `tempBoxProc{builder.createTemporary` 为核心的调用或声明。
- **L1708 EN**: Continues logic associated with callable symbol `nullBoxProc`.
  **L1708 CN**: 继续与可调用符号 `nullBoxProc` 相关的逻辑。
- **L1709 EN**: Executes a call or declaration centered on `fir::factory::createNullBoxProc`.
  **L1709 CN**: 执行以 `fir::factory::createNullBoxProc` 为核心的调用或声明。
- **L1710 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L1710 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L1711 EN**: Returns from the current function with `PreparedDummyArgument{tempBoxProc, /*cleanups=*/{}}`.
  **L1711 CN**: 以 `PreparedDummyArgument{tempBoxProc, /*cleanups=*/{}}` 从当前函数返回。
- **L1712 EN**: Closes the current lexical scope or compound statement.
  **L1712 CN**: 结束当前词法作用域或复合语句块。
- **L1713 EN**: Initializes variable `actual` from the right-hand expression.
  **L1713 CN**: 使用右侧表达式初始化变量 `actual`。
- **L1714 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1714 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1715 EN**: Returns from the current function with `PreparedDummyArgument{actual, /*cleanups=*/{}}`.
  **L1715 CN**: 以 `PreparedDummyArgument{actual, /*cleanups=*/{}}` 从当前函数返回。
- **L1716 EN**: Checks an internal invariant in debug builds.
  **L1716 CN**: 在调试构建中检查内部不变式。
- **L1717 EN**: Comment explains nearby logic, intent, or metadata: `Procedure actual to procedure pointer dummy.`.
  **L1717 CN**: 注释说明附近代码的逻辑、意图或元数据：`Procedure actual to procedure pointer dummy.`。
- **L1718 EN**: Executes a call or declaration centered on `tempBoxProc{builder.createTemporary`.
  **L1718 CN**: 执行以 `tempBoxProc{builder.createTemporary` 为核心的调用或声明。
- **L1719 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L1719 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L1720 EN**: Returns from the current function with `PreparedDummyArgument{tempBoxProc, /*cleanups=*/{}}`.
  **L1720 CN**: 以 `PreparedDummyArgument{tempBoxProc, /*cleanups=*/{}}` 从当前函数返回。
- **L1721 EN**: Closes the current lexical scope or compound statement.
  **L1721 CN**: 结束当前词法作用域或复合语句块。
- **L1722 EN**: Blank line separating nearby declarations or logic blocks.
  **L1722 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1723 EN**: Comment explains nearby logic, intent, or metadata: `Prepare arguments of calls to user procedures with actual arguments that`.
  **L1723 CN**: 注释说明附近代码的逻辑、意图或元数据：`Prepare arguments of calls to user procedures with actual arguments that`。
- **L1724 EN**: Comment explains nearby logic, intent, or metadata: `have been pre-lowered but not yet prepared according to the interface.`.
  **L1724 CN**: 注释说明附近代码的逻辑、意图或元数据：`have been pre-lowered but not yet prepared according to the interface.`。
- **L1725 EN**: Continues logic associated with callable symbol `prepareUserCallArguments`.
  **L1725 CN**: 继续与可调用符号 `prepareUserCallArguments` 相关的逻辑。
- **L1726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L1726 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L1727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::CallerInterface &caller, mlir::FunctionType callSiteType,`.
  **L1727 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::CallerInterface &caller, mlir::FunctionType callSiteType,`。
- **L1728 EN**: Continues the surrounding expression or declaration: `CallContext &callContext, llvm::SmallVector<CallCleanUp> &callCleanUps) {`.
  **L1728 CN**: 继续构造周围的表达式或声明：`CallContext &callContext, llvm::SmallVector<CallCleanUp> &callCleanUps) {`。

### Lines 1729-1752

````cpp
  using PassBy = Fortran::lower::CallerInterface::PassEntityBy;
  mlir::Location loc = callContext.loc;
  bool mustRemapActualToDummyDescriptors = false;
  fir::FirOpBuilder &builder = callContext.getBuilder();
  std::optional<unsigned> passArg = caller.getPassArgIndex();
  int argIndex = -1;
  for (auto [preparedActual, arg] :
       llvm::zip(loweredActuals, caller.getPassedArguments())) {
    ++argIndex;
    bool thisIsPassArg = passArg && argIndex == static_cast<int>(*passArg);
    mlir::Type argTy = callSiteType.getInput(arg.firArgument);
    if (!preparedActual) {
      // Optional dummy argument for which there is no actual argument.
      caller.placeInput(arg, builder.genAbsentOp(loc, argTy));
      continue;
    }

    switch (arg.passBy) {
    case PassBy::Value: {
      // True pass-by-value semantics.
      assert(!preparedActual->handleDynamicOptional() && "cannot be optional");
      hlfir::Entity actual = preparedActual->getActual(loc, builder);
      hlfir::Entity value = hlfir::loadTrivialScalar(loc, builder, actual);

````
- **L1729 EN**: Defines alias `PassBy` to simplify later code.
  **L1729 CN**: 定义别名 `PassBy` 以简化后续代码。
- **L1730 EN**: Initializes variable `loc` from the right-hand expression.
  **L1730 CN**: 使用右侧表达式初始化变量 `loc`。
- **L1731 EN**: Initializes variable `mustRemapActualToDummyDescriptors` from the right-hand expression.
  **L1731 CN**: 使用右侧表达式初始化变量 `mustRemapActualToDummyDescriptors`。
- **L1732 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L1732 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L1733 EN**: Initializes variable `passArg` from the right-hand expression.
  **L1733 CN**: 使用右侧表达式初始化变量 `passArg`。
- **L1734 EN**: Initializes variable `argIndex` from the right-hand expression.
  **L1734 CN**: 使用右侧表达式初始化变量 `argIndex`。
- **L1735 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1735 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1736 EN**: Starts a function, method, lambda, or structured scope: `llvm::zip(loweredActuals, caller.getPassedArguments())) {`.
  **L1736 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::zip(loweredActuals, caller.getPassedArguments())) {`。
- **L1737 EN**: Executes a standalone statement or declaration: `++argIndex;`.
  **L1737 CN**: 执行一条独立语句或声明：`++argIndex;`。
- **L1738 EN**: Initializes variable `thisIsPassArg` from the right-hand expression.
  **L1738 CN**: 使用右侧表达式初始化变量 `thisIsPassArg`。
- **L1739 EN**: Initializes variable `argTy` from the right-hand expression.
  **L1739 CN**: 使用右侧表达式初始化变量 `argTy`。
- **L1740 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1740 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1741 EN**: Comment explains nearby logic, intent, or metadata: `Optional dummy argument for which there is no actual argument.`.
  **L1741 CN**: 注释说明附近代码的逻辑、意图或元数据：`Optional dummy argument for which there is no actual argument.`。
- **L1742 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L1742 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L1743 EN**: Skips to the next loop iteration.
  **L1743 CN**: 跳到下一次循环迭代。
- **L1744 EN**: Closes the current lexical scope or compound statement.
  **L1744 CN**: 结束当前词法作用域或复合语句块。
- **L1745 EN**: Blank line separating nearby declarations or logic blocks.
  **L1745 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1746 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1746 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1747 EN**: Introduces a switch dispatch label: `case PassBy::Value: {`.
  **L1747 CN**: 引入一个 switch 分发标签：`case PassBy::Value: {`。
- **L1748 EN**: Comment explains nearby logic, intent, or metadata: `True pass-by-value semantics.`.
  **L1748 CN**: 注释说明附近代码的逻辑、意图或元数据：`True pass-by-value semantics.`。
- **L1749 EN**: Checks an internal invariant in debug builds.
  **L1749 CN**: 在调试构建中检查内部不变式。
- **L1750 EN**: Initializes variable `actual` from the right-hand expression.
  **L1750 CN**: 使用右侧表达式初始化变量 `actual`。
- **L1751 EN**: Initializes variable `value` from the right-hand expression.
  **L1751 CN**: 使用右侧表达式初始化变量 `value`。
- **L1752 EN**: Blank line separating nearby declarations or logic blocks.
  **L1752 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1753-1776

````cpp
      mlir::Type eleTy = value.getFortranElementType();
      if (fir::isa_builtin_cptr_type(eleTy)) {
        // Pass-by-value argument of type(C_PTR/C_FUNPTR).
        // Load the __address component and pass it by value.
        if (value.isValue()) {
          auto associate = hlfir::genAssociateExpr(loc, builder, value, eleTy,
                                                   "adapt.cptrbyval");
          value = hlfir::Entity{genRecordCPtrValueArg(
              builder, loc, associate.getFirBase(), eleTy)};
          hlfir::EndAssociateOp::create(builder, loc, associate);
        } else {
          value =
              hlfir::Entity{genRecordCPtrValueArg(builder, loc, value, eleTy)};
        }
      } else if (fir::isa_derived(value.getFortranElementType()) ||
                 value.isCharacter()) {
        // BIND(C), VALUE derived type or character. The value must really
        // be loaded here.
        auto [exv, cleanup] = hlfir::convertToValue(loc, builder, value);
        mlir::Value loadedValue = fir::getBase(exv);
        // Character actual arguments may have unknown length or a length longer
        // than one. Cast the memory ref to the dummy type so that the load is
        // valid and only loads what is needed.
        if (mlir::Type baseTy = fir::dyn_cast_ptrEleTy(loadedValue.getType()))
````
- **L1753 EN**: Initializes variable `eleTy` from the right-hand expression.
  **L1753 CN**: 使用右侧表达式初始化变量 `eleTy`。
- **L1754 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1754 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1755 EN**: Comment explains nearby logic, intent, or metadata: `Pass-by-value argument of type(C_PTR/C_FUNPTR).`.
  **L1755 CN**: 注释说明附近代码的逻辑、意图或元数据：`Pass-by-value argument of type(C_PTR/C_FUNPTR).`。
- **L1756 EN**: Comment explains nearby logic, intent, or metadata: `Load the __address component and pass it by value.`.
  **L1756 CN**: 注释说明附近代码的逻辑、意图或元数据：`Load the __address component and pass it by value.`。
- **L1757 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1757 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto associate = hlfir::genAssociateExpr(loc, builder, value, eleTy,`.
  **L1758 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto associate = hlfir::genAssociateExpr(loc, builder, value, eleTy,`。
- **L1759 EN**: Executes a standalone statement or declaration: `"adapt.cptrbyval");`.
  **L1759 CN**: 执行一条独立语句或声明：`"adapt.cptrbyval");`。
- **L1760 EN**: Continues logic associated with callable symbol `genRecordCPtrValueArg`.
  **L1760 CN**: 继续与可调用符号 `genRecordCPtrValueArg` 相关的逻辑。
- **L1761 EN**: Executes a call or declaration centered on `associate.getFirBase`.
  **L1761 CN**: 执行以 `associate.getFirBase` 为核心的调用或声明。
- **L1762 EN**: Executes a call or declaration centered on `hlfir::EndAssociateOp::create`.
  **L1762 CN**: 执行以 `hlfir::EndAssociateOp::create` 为核心的调用或声明。
- **L1763 EN**: Transitions from the previous branch into the alternative path.
  **L1763 CN**: 从前一个分支过渡到备选路径。
- **L1764 EN**: Continues the surrounding expression or declaration: `value =`.
  **L1764 CN**: 继续构造周围的表达式或声明：`value =`。
- **L1765 EN**: Executes a call or declaration centered on `hlfir::Entity{genRecordCPtrValueArg`.
  **L1765 CN**: 执行以 `hlfir::Entity{genRecordCPtrValueArg` 为核心的调用或声明。
- **L1766 EN**: Closes the current lexical scope or compound statement.
  **L1766 CN**: 结束当前词法作用域或复合语句块。
- **L1767 EN**: Transitions from the previous branch into an `else if` condition.
  **L1767 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1768 EN**: Starts a function, method, lambda, or structured scope: `value.isCharacter()) {`.
  **L1768 CN**: 开始一个函数、方法、lambda 或结构化作用域：`value.isCharacter()) {`。
- **L1769 EN**: Comment explains nearby logic, intent, or metadata: `BIND(C), VALUE derived type or character. The value must really`.
  **L1769 CN**: 注释说明附近代码的逻辑、意图或元数据：`BIND(C), VALUE derived type or character. The value must really`。
- **L1770 EN**: Comment explains nearby logic, intent, or metadata: `be loaded here.`.
  **L1770 CN**: 注释说明附近代码的逻辑、意图或元数据：`be loaded here.`。
- **L1771 EN**: Executes a call or declaration centered on `hlfir::convertToValue`.
  **L1771 CN**: 执行以 `hlfir::convertToValue` 为核心的调用或声明。
- **L1772 EN**: Initializes variable `loadedValue` from the right-hand expression.
  **L1772 CN**: 使用右侧表达式初始化变量 `loadedValue`。
- **L1773 EN**: Comment explains nearby logic, intent, or metadata: `Character actual arguments may have unknown length or a length longer`.
  **L1773 CN**: 注释说明附近代码的逻辑、意图或元数据：`Character actual arguments may have unknown length or a length longer`。
- **L1774 EN**: Comment explains nearby logic, intent, or metadata: `than one. Cast the memory ref to the dummy type so that the load is`.
  **L1774 CN**: 注释说明附近代码的逻辑、意图或元数据：`than one. Cast the memory ref to the dummy type so that the load is`。
- **L1775 EN**: Comment explains nearby logic, intent, or metadata: `valid and only loads what is needed.`.
  **L1775 CN**: 注释说明附近代码的逻辑、意图或元数据：`valid and only loads what is needed.`。
- **L1776 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1776 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1777-1800

````cpp
          if (fir::isa_char(baseTy))
            loadedValue = builder.createConvert(
                loc, fir::ReferenceType::get(argTy), loadedValue);
        if (fir::isa_ref_type(loadedValue.getType()))
          loadedValue = fir::LoadOp::create(builder, loc, loadedValue);
        caller.placeInput(arg, loadedValue);
        if (cleanup)
          (*cleanup)();
        break;
      }
      // For %VAL arguments, we should pass the value directly without
      // conversion to reference types.
      caller.placeInput(arg, builder.createConvert(loc, argTy, value));

    } break;
    case PassBy::BaseAddressValueAttribute:
    case PassBy::CharBoxValueAttribute:
    case PassBy::Box:
    case PassBy::BaseAddress:
    case PassBy::BoxChar: {
      PreparedDummyArgument preparedDummy = prepareUserCallActualArgument(
          loc, builder, *preparedActual, argTy, arg, callContext);
      callCleanUps.append(preparedDummy.cleanups.rbegin(),
                          preparedDummy.cleanups.rend());
````
- **L1777 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1777 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1778 EN**: Continues logic associated with callable symbol `createConvert`.
  **L1778 CN**: 继续与可调用符号 `createConvert` 相关的逻辑。
- **L1779 EN**: Executes a call or declaration centered on `fir::ReferenceType::get`.
  **L1779 CN**: 执行以 `fir::ReferenceType::get` 为核心的调用或声明。
- **L1780 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1780 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1781 EN**: Executes a call or declaration centered on `fir::LoadOp::create`.
  **L1781 CN**: 执行以 `fir::LoadOp::create` 为核心的调用或声明。
- **L1782 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L1782 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L1783 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1783 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1784 EN**: Executes a call or declaration centered on `statement`.
  **L1784 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1785 EN**: Exits the nearest loop or switch statement.
  **L1785 CN**: 退出最近的循环或 switch 语句。
- **L1786 EN**: Closes the current lexical scope or compound statement.
  **L1786 CN**: 结束当前词法作用域或复合语句块。
- **L1787 EN**: Comment explains nearby logic, intent, or metadata: `For %VAL arguments, we should pass the value directly without`.
  **L1787 CN**: 注释说明附近代码的逻辑、意图或元数据：`For %VAL arguments, we should pass the value directly without`。
- **L1788 EN**: Comment explains nearby logic, intent, or metadata: `conversion to reference types.`.
  **L1788 CN**: 注释说明附近代码的逻辑、意图或元数据：`conversion to reference types.`。
- **L1789 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L1789 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L1790 EN**: Blank line separating nearby declarations or logic blocks.
  **L1790 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1791 EN**: Executes a standalone statement or declaration: `} break;`.
  **L1791 CN**: 执行一条独立语句或声明：`} break;`。
- **L1792 EN**: Introduces a switch dispatch label: `case PassBy::BaseAddressValueAttribute:`.
  **L1792 CN**: 引入一个 switch 分发标签：`case PassBy::BaseAddressValueAttribute:`。
- **L1793 EN**: Introduces a switch dispatch label: `case PassBy::CharBoxValueAttribute:`.
  **L1793 CN**: 引入一个 switch 分发标签：`case PassBy::CharBoxValueAttribute:`。
- **L1794 EN**: Introduces a switch dispatch label: `case PassBy::Box:`.
  **L1794 CN**: 引入一个 switch 分发标签：`case PassBy::Box:`。
- **L1795 EN**: Introduces a switch dispatch label: `case PassBy::BaseAddress:`.
  **L1795 CN**: 引入一个 switch 分发标签：`case PassBy::BaseAddress:`。
- **L1796 EN**: Introduces a switch dispatch label: `case PassBy::BoxChar: {`.
  **L1796 CN**: 引入一个 switch 分发标签：`case PassBy::BoxChar: {`。
- **L1797 EN**: Continues logic associated with callable symbol `prepareUserCallActualArgument`.
  **L1797 CN**: 继续与可调用符号 `prepareUserCallActualArgument` 相关的逻辑。
- **L1798 EN**: Executes a standalone statement or declaration: `loc, builder, *preparedActual, argTy, arg, callContext);`.
  **L1798 CN**: 执行一条独立语句或声明：`loc, builder, *preparedActual, argTy, arg, callContext);`。
- **L1799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `callCleanUps.append(preparedDummy.cleanups.rbegin(),`.
  **L1799 CN**: 继续一个多行参数列表、初始化器或聚合项：`callCleanUps.append(preparedDummy.cleanups.rbegin(),`。
- **L1800 EN**: Executes a call or declaration centered on `preparedDummy.cleanups.rend`.
  **L1800 CN**: 执行以 `preparedDummy.cleanups.rend` 为核心的调用或声明。

### Lines 1801-1824

````cpp
      caller.placeInput(arg, preparedDummy.dummy);
      if (arg.passBy == PassBy::Box)
        mustRemapActualToDummyDescriptors |=
            arg.isSequenceAssociatedDescriptor();
    } break;
    case PassBy::BoxProcRef: {
      PreparedDummyArgument preparedDummy =
          prepareProcedurePointerActualArgument(loc, builder, *preparedActual,
                                                argTy, arg, callContext);
      callCleanUps.append(preparedDummy.cleanups.rbegin(),
                          preparedDummy.cleanups.rend());
      caller.placeInput(arg, preparedDummy.dummy);
    } break;
    case PassBy::AddressAndLength:
      // PassBy::AddressAndLength is only used for character results. Results
      // are not handled here.
      fir::emitFatalError(
          loc, "unexpected PassBy::AddressAndLength for actual arguments");
      break;
    case PassBy::CharProcTuple: {
      hlfir::Entity actual = preparedActual->getActual(loc, builder);
      if (actual.isProcedurePointer())
        actual = hlfir::derefPointersAndAllocatables(loc, builder, actual);
      if (!fir::isCharacterProcedureTuple(actual.getType()))
````
- **L1801 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L1801 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L1802 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1802 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1803 EN**: Continues the surrounding expression or declaration: `mustRemapActualToDummyDescriptors |=`.
  **L1803 CN**: 继续构造周围的表达式或声明：`mustRemapActualToDummyDescriptors |=`。
- **L1804 EN**: Executes a call or declaration centered on `arg.isSequenceAssociatedDescriptor`.
  **L1804 CN**: 执行以 `arg.isSequenceAssociatedDescriptor` 为核心的调用或声明。
- **L1805 EN**: Executes a standalone statement or declaration: `} break;`.
  **L1805 CN**: 执行一条独立语句或声明：`} break;`。
- **L1806 EN**: Introduces a switch dispatch label: `case PassBy::BoxProcRef: {`.
  **L1806 CN**: 引入一个 switch 分发标签：`case PassBy::BoxProcRef: {`。
- **L1807 EN**: Continues the surrounding expression or declaration: `PreparedDummyArgument preparedDummy =`.
  **L1807 CN**: 继续构造周围的表达式或声明：`PreparedDummyArgument preparedDummy =`。
- **L1808 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `prepareProcedurePointerActualArgument(loc, builder, *preparedActual,`.
  **L1808 CN**: 继续一个多行参数列表、初始化器或聚合项：`prepareProcedurePointerActualArgument(loc, builder, *preparedActual,`。
- **L1809 EN**: Executes a standalone statement or declaration: `argTy, arg, callContext);`.
  **L1809 CN**: 执行一条独立语句或声明：`argTy, arg, callContext);`。
- **L1810 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `callCleanUps.append(preparedDummy.cleanups.rbegin(),`.
  **L1810 CN**: 继续一个多行参数列表、初始化器或聚合项：`callCleanUps.append(preparedDummy.cleanups.rbegin(),`。
- **L1811 EN**: Executes a call or declaration centered on `preparedDummy.cleanups.rend`.
  **L1811 CN**: 执行以 `preparedDummy.cleanups.rend` 为核心的调用或声明。
- **L1812 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L1812 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L1813 EN**: Executes a standalone statement or declaration: `} break;`.
  **L1813 CN**: 执行一条独立语句或声明：`} break;`。
- **L1814 EN**: Introduces a switch dispatch label: `case PassBy::AddressAndLength:`.
  **L1814 CN**: 引入一个 switch 分发标签：`case PassBy::AddressAndLength:`。
- **L1815 EN**: Comment explains nearby logic, intent, or metadata: `PassBy::AddressAndLength is only used for character results. Results`.
  **L1815 CN**: 注释说明附近代码的逻辑、意图或元数据：`PassBy::AddressAndLength is only used for character results. Results`。
- **L1816 EN**: Comment explains nearby logic, intent, or metadata: `are not handled here.`.
  **L1816 CN**: 注释说明附近代码的逻辑、意图或元数据：`are not handled here.`。
- **L1817 EN**: Continues logic associated with callable symbol `emitFatalError`.
  **L1817 CN**: 继续与可调用符号 `emitFatalError` 相关的逻辑。
- **L1818 EN**: Executes a standalone statement or declaration: `loc, "unexpected PassBy::AddressAndLength for actual arguments");`.
  **L1818 CN**: 执行一条独立语句或声明：`loc, "unexpected PassBy::AddressAndLength for actual arguments");`。
- **L1819 EN**: Exits the nearest loop or switch statement.
  **L1819 CN**: 退出最近的循环或 switch 语句。
- **L1820 EN**: Introduces a switch dispatch label: `case PassBy::CharProcTuple: {`.
  **L1820 CN**: 引入一个 switch 分发标签：`case PassBy::CharProcTuple: {`。
- **L1821 EN**: Initializes variable `actual` from the right-hand expression.
  **L1821 CN**: 使用右侧表达式初始化变量 `actual`。
- **L1822 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1822 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1823 EN**: Executes a call or declaration centered on `hlfir::derefPointersAndAllocatables`.
  **L1823 CN**: 执行以 `hlfir::derefPointersAndAllocatables` 为核心的调用或声明。
- **L1824 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1824 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1825-1848

````cpp
        actual = fixProcedureDummyMismatch(loc, builder, actual, argTy);
      caller.placeInput(arg, actual);
    } break;
    case PassBy::MutableBox: {
      const Fortran::lower::SomeExpr *expr = arg.entity->UnwrapExpr();
      // C709 and C710.
      assert(expr && "cannot pass TYPE(*) to POINTER or ALLOCATABLE");
      hlfir::Entity actual = preparedActual->getActual(loc, builder);
      if (Fortran::evaluate::UnwrapExpr<Fortran::evaluate::NullPointer>(
              *expr)) {
        // If expr is NULL(), the mutableBox created must be a deallocated
        // pointer with the dummy argument characteristics (see table 16.5
        // in Fortran 2018 standard).
        // No length parameters are set for the created box because any non
        // deferred type parameters of the dummy will be evaluated on the
        // callee side, and it is illegal to use NULL without a MOLD if any
        // dummy length parameters are assumed.
        mlir::Type boxTy = fir::dyn_cast_ptrEleTy(argTy);
        assert(boxTy && mlir::isa<fir::BaseBoxType>(boxTy) &&
               "must be a fir.box type");
        mlir::Value boxStorage =
            fir::factory::genNullBoxStorage(builder, loc, boxTy);
        caller.placeInput(arg, boxStorage);
        continue;
````
- **L1825 EN**: Executes a call or declaration centered on `fixProcedureDummyMismatch`.
  **L1825 CN**: 执行以 `fixProcedureDummyMismatch` 为核心的调用或声明。
- **L1826 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L1826 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L1827 EN**: Executes a standalone statement or declaration: `} break;`.
  **L1827 CN**: 执行一条独立语句或声明：`} break;`。
- **L1828 EN**: Introduces a switch dispatch label: `case PassBy::MutableBox: {`.
  **L1828 CN**: 引入一个 switch 分发标签：`case PassBy::MutableBox: {`。
- **L1829 EN**: Executes a call or declaration centered on `arg.entity->UnwrapExpr`.
  **L1829 CN**: 执行以 `arg.entity->UnwrapExpr` 为核心的调用或声明。
- **L1830 EN**: Comment explains nearby logic, intent, or metadata: `C709 and C710.`.
  **L1830 CN**: 注释说明附近代码的逻辑、意图或元数据：`C709 and C710.`。
- **L1831 EN**: Checks an internal invariant in debug builds.
  **L1831 CN**: 在调试构建中检查内部不变式。
- **L1832 EN**: Initializes variable `actual` from the right-hand expression.
  **L1832 CN**: 使用右侧表达式初始化变量 `actual`。
- **L1833 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1833 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1834 EN**: Comment explains nearby logic, intent, or metadata: `expr)) {`.
  **L1834 CN**: 注释说明附近代码的逻辑、意图或元数据：`expr)) {`。
- **L1835 EN**: Comment explains nearby logic, intent, or metadata: `If expr is NULL(), the mutableBox created must be a deallocated`.
  **L1835 CN**: 注释说明附近代码的逻辑、意图或元数据：`If expr is NULL(), the mutableBox created must be a deallocated`。
- **L1836 EN**: Comment explains nearby logic, intent, or metadata: `pointer with the dummy argument characteristics (see table 16.5`.
  **L1836 CN**: 注释说明附近代码的逻辑、意图或元数据：`pointer with the dummy argument characteristics (see table 16.5`。
- **L1837 EN**: Comment explains nearby logic, intent, or metadata: `in Fortran 2018 standard).`.
  **L1837 CN**: 注释说明附近代码的逻辑、意图或元数据：`in Fortran 2018 standard).`。
- **L1838 EN**: Comment explains nearby logic, intent, or metadata: `No length parameters are set for the created box because any non`.
  **L1838 CN**: 注释说明附近代码的逻辑、意图或元数据：`No length parameters are set for the created box because any non`。
- **L1839 EN**: Comment explains nearby logic, intent, or metadata: `deferred type parameters of the dummy will be evaluated on the`.
  **L1839 CN**: 注释说明附近代码的逻辑、意图或元数据：`deferred type parameters of the dummy will be evaluated on the`。
- **L1840 EN**: Comment explains nearby logic, intent, or metadata: `callee side, and it is illegal to use NULL without a MOLD if any`.
  **L1840 CN**: 注释说明附近代码的逻辑、意图或元数据：`callee side, and it is illegal to use NULL without a MOLD if any`。
- **L1841 EN**: Comment explains nearby logic, intent, or metadata: `dummy length parameters are assumed.`.
  **L1841 CN**: 注释说明附近代码的逻辑、意图或元数据：`dummy length parameters are assumed.`。
- **L1842 EN**: Initializes variable `boxTy` from the right-hand expression.
  **L1842 CN**: 使用右侧表达式初始化变量 `boxTy`。
- **L1843 EN**: Checks an internal invariant in debug builds.
  **L1843 CN**: 在调试构建中检查内部不变式。
- **L1844 EN**: Executes a standalone statement or declaration: `"must be a fir.box type");`.
  **L1844 CN**: 执行一条独立语句或声明：`"must be a fir.box type");`。
- **L1845 EN**: Continues the surrounding expression or declaration: `mlir::Value boxStorage =`.
  **L1845 CN**: 继续构造周围的表达式或声明：`mlir::Value boxStorage =`。
- **L1846 EN**: Executes a call or declaration centered on `fir::factory::genNullBoxStorage`.
  **L1846 CN**: 执行以 `fir::factory::genNullBoxStorage` 为核心的调用或声明。
- **L1847 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L1847 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L1848 EN**: Skips to the next loop iteration.
  **L1848 CN**: 跳到下一次循环迭代。

### Lines 1849-1872

````cpp
      }
      if (arg.testTKR(Fortran::common::IgnoreTKR::Contiguous) &&
          actual.isBoxAddress()) {
        // With ignore_tkr(c), pointer to a descriptor should be passed as is
        caller.placeInput(arg, actual);
        continue;
      }
      if (fir::isPointerType(argTy) &&
          (!Fortran::evaluate::IsObjectPointer(*expr) || thisIsPassArg)) {
        // Passing a non POINTER actual argument to a POINTER dummy argument.
        // Create a pointer of the dummy argument type and assign the actual
        // argument to it.
        auto dataTy = llvm::cast<fir::BaseBoxType>(fir::unwrapRefType(argTy));
        fir::ExtendedValue actualExv = Fortran::lower::convertToAddress(
            loc, callContext.converter, actual, callContext.stmtCtx,
            hlfir::getFortranElementType(dataTy));
        if (thisIsPassArg)
          caller.setOriginalPassArg(fir::getBase(actualExv));
        // If the dummy is an assumed-rank pointer, allocate a pointer
        // descriptor with the actual argument rank (if it is not assumed-rank
        // itself).
        if (dataTy.isAssumedRank()) {
          dataTy =
              dataTy.getBoxTypeWithNewShape(fir::getBase(actualExv).getType());
````
- **L1849 EN**: Closes the current lexical scope or compound statement.
  **L1849 CN**: 结束当前词法作用域或复合语句块。
- **L1850 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1850 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1851 EN**: Starts a function, method, lambda, or structured scope: `actual.isBoxAddress()) {`.
  **L1851 CN**: 开始一个函数、方法、lambda 或结构化作用域：`actual.isBoxAddress()) {`。
- **L1852 EN**: Comment explains nearby logic, intent, or metadata: `With ignore_tkr(c), pointer to a descriptor should be passed as is`.
  **L1852 CN**: 注释说明附近代码的逻辑、意图或元数据：`With ignore_tkr(c), pointer to a descriptor should be passed as is`。
- **L1853 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L1853 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L1854 EN**: Skips to the next loop iteration.
  **L1854 CN**: 跳到下一次循环迭代。
- **L1855 EN**: Closes the current lexical scope or compound statement.
  **L1855 CN**: 结束当前词法作用域或复合语句块。
- **L1856 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1856 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1857 EN**: Starts a function, method, lambda, or structured scope: `(!Fortran::evaluate::IsObjectPointer(*expr) || thisIsPassArg)) {`.
  **L1857 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(!Fortran::evaluate::IsObjectPointer(*expr) || thisIsPassArg)) {`。
- **L1858 EN**: Comment explains nearby logic, intent, or metadata: `Passing a non POINTER actual argument to a POINTER dummy argument.`.
  **L1858 CN**: 注释说明附近代码的逻辑、意图或元数据：`Passing a non POINTER actual argument to a POINTER dummy argument.`。
- **L1859 EN**: Comment explains nearby logic, intent, or metadata: `Create a pointer of the dummy argument type and assign the actual`.
  **L1859 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create a pointer of the dummy argument type and assign the actual`。
- **L1860 EN**: Comment explains nearby logic, intent, or metadata: `argument to it.`.
  **L1860 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument to it.`。
- **L1861 EN**: Initializes variable `dataTy` from the right-hand expression.
  **L1861 CN**: 使用右侧表达式初始化变量 `dataTy`。
- **L1862 EN**: Continues logic associated with callable symbol `convertToAddress`.
  **L1862 CN**: 继续与可调用符号 `convertToAddress` 相关的逻辑。
- **L1863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, callContext.converter, actual, callContext.stmtCtx,`.
  **L1863 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, callContext.converter, actual, callContext.stmtCtx,`。
- **L1864 EN**: Executes a call or declaration centered on `hlfir::getFortranElementType`.
  **L1864 CN**: 执行以 `hlfir::getFortranElementType` 为核心的调用或声明。
- **L1865 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1865 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1866 EN**: Executes a call or declaration centered on `caller.setOriginalPassArg`.
  **L1866 CN**: 执行以 `caller.setOriginalPassArg` 为核心的调用或声明。
- **L1867 EN**: Comment explains nearby logic, intent, or metadata: `If the dummy is an assumed-rank pointer, allocate a pointer`.
  **L1867 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the dummy is an assumed-rank pointer, allocate a pointer`。
- **L1868 EN**: Comment explains nearby logic, intent, or metadata: `descriptor with the actual argument rank (if it is not assumed-rank`.
  **L1868 CN**: 注释说明附近代码的逻辑、意图或元数据：`descriptor with the actual argument rank (if it is not assumed-rank`。
- **L1869 EN**: Comment explains nearby logic, intent, or metadata: `itself).`.
  **L1869 CN**: 注释说明附近代码的逻辑、意图或元数据：`itself).`。
- **L1870 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1870 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1871 EN**: Continues the surrounding expression or declaration: `dataTy =`.
  **L1871 CN**: 继续构造周围的表达式或声明：`dataTy =`。
- **L1872 EN**: Executes a call or declaration centered on `dataTy.getBoxTypeWithNewShape`.
  **L1872 CN**: 执行以 `dataTy.getBoxTypeWithNewShape` 为核心的调用或声明。

### Lines 1873-1896

````cpp
        }
        mlir::Value irBox = builder.createTemporary(loc, dataTy);
        fir::MutableBoxValue ptrBox(irBox,
                                    /*nonDeferredParams=*/mlir::ValueRange{},
                                    /*mutableProperties=*/{});
        fir::factory::associateMutableBox(builder, loc, ptrBox, actualExv,
                                          /*lbounds=*/{});
        caller.placeInput(arg, irBox);
        continue;
      }
      // Passing a POINTER to a POINTER, or an ALLOCATABLE to an ALLOCATABLE.
      assert(actual.isMutableBox() && "actual must be a mutable box");
      if (fir::isAllocatableType(argTy) && arg.isIntentOut() &&
          callContext.isBindcCall()) {
        // INTENT(OUT) allocatables are deallocated on the callee side,
        // but BIND(C) procedures may be implemented in C, so deallocation is
        // also done on the caller side (if the procedure is implemented in
        // Fortran, the deallocation attempt in the callee will be a no-op).
        auto [exv, cleanup] =
            hlfir::translateToExtendedValue(loc, builder, actual);
        const auto *mutableBox = exv.getBoxOf<fir::MutableBoxValue>();
        assert(mutableBox && !cleanup && "expect allocatable");
        Fortran::lower::genDeallocateIfAllocated(callContext.converter,
                                                 *mutableBox, loc);
````
- **L1873 EN**: Closes the current lexical scope or compound statement.
  **L1873 CN**: 结束当前词法作用域或复合语句块。
- **L1874 EN**: Initializes variable `irBox` from the right-hand expression.
  **L1874 CN**: 使用右侧表达式初始化变量 `irBox`。
- **L1875 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::MutableBoxValue ptrBox(irBox,`.
  **L1875 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::MutableBoxValue ptrBox(irBox,`。
- **L1876 EN**: Comment explains nearby logic, intent, or metadata: `nonDeferredParams=*/mlir::ValueRange{},`.
  **L1876 CN**: 注释说明附近代码的逻辑、意图或元数据：`nonDeferredParams=*/mlir::ValueRange{},`。
- **L1877 EN**: Comment explains nearby logic, intent, or metadata: `mutableProperties=*/{});`.
  **L1877 CN**: 注释说明附近代码的逻辑、意图或元数据：`mutableProperties=*/{});`。
- **L1878 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::factory::associateMutableBox(builder, loc, ptrBox, actualExv,`.
  **L1878 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::factory::associateMutableBox(builder, loc, ptrBox, actualExv,`。
- **L1879 EN**: Comment explains nearby logic, intent, or metadata: `lbounds=*/{});`.
  **L1879 CN**: 注释说明附近代码的逻辑、意图或元数据：`lbounds=*/{});`。
- **L1880 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L1880 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L1881 EN**: Skips to the next loop iteration.
  **L1881 CN**: 跳到下一次循环迭代。
- **L1882 EN**: Closes the current lexical scope or compound statement.
  **L1882 CN**: 结束当前词法作用域或复合语句块。
- **L1883 EN**: Comment explains nearby logic, intent, or metadata: `Passing a POINTER to a POINTER, or an ALLOCATABLE to an ALLOCATABLE.`.
  **L1883 CN**: 注释说明附近代码的逻辑、意图或元数据：`Passing a POINTER to a POINTER, or an ALLOCATABLE to an ALLOCATABLE.`。
- **L1884 EN**: Checks an internal invariant in debug builds.
  **L1884 CN**: 在调试构建中检查内部不变式。
- **L1885 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1885 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1886 EN**: Starts a function, method, lambda, or structured scope: `callContext.isBindcCall()) {`.
  **L1886 CN**: 开始一个函数、方法、lambda 或结构化作用域：`callContext.isBindcCall()) {`。
- **L1887 EN**: Comment explains nearby logic, intent, or metadata: `INTENT(OUT) allocatables are deallocated on the callee side,`.
  **L1887 CN**: 注释说明附近代码的逻辑、意图或元数据：`INTENT(OUT) allocatables are deallocated on the callee side,`。
- **L1888 EN**: Comment explains nearby logic, intent, or metadata: `but BIND(C) procedures may be implemented in C, so deallocation is`.
  **L1888 CN**: 注释说明附近代码的逻辑、意图或元数据：`but BIND(C) procedures may be implemented in C, so deallocation is`。
- **L1889 EN**: Comment explains nearby logic, intent, or metadata: `also done on the caller side (if the procedure is implemented in`.
  **L1889 CN**: 注释说明附近代码的逻辑、意图或元数据：`also done on the caller side (if the procedure is implemented in`。
- **L1890 EN**: Comment explains nearby logic, intent, or metadata: `Fortran, the deallocation attempt in the callee will be a no-op).`.
  **L1890 CN**: 注释说明附近代码的逻辑、意图或元数据：`Fortran, the deallocation attempt in the callee will be a no-op).`。
- **L1891 EN**: Continues the surrounding expression or declaration: `auto [exv, cleanup] =`.
  **L1891 CN**: 继续构造周围的表达式或声明：`auto [exv, cleanup] =`。
- **L1892 EN**: Executes a call or declaration centered on `hlfir::translateToExtendedValue`.
  **L1892 CN**: 执行以 `hlfir::translateToExtendedValue` 为核心的调用或声明。
- **L1893 EN**: Executes a call or declaration centered on `exv.getBoxOf<fir::MutableBoxValue>`.
  **L1893 CN**: 执行以 `exv.getBoxOf<fir::MutableBoxValue>` 为核心的调用或声明。
- **L1894 EN**: Checks an internal invariant in debug builds.
  **L1894 CN**: 在调试构建中检查内部不变式。
- **L1895 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::genDeallocateIfAllocated(callContext.converter,`.
  **L1895 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::genDeallocateIfAllocated(callContext.converter,`。
- **L1896 EN**: Comment explains nearby logic, intent, or metadata: `mutableBox, loc);`.
  **L1896 CN**: 注释说明附近代码的逻辑、意图或元数据：`mutableBox, loc);`。

### Lines 1897-1920

````cpp
      }
      caller.placeInput(arg, actual);
    } break;
    }
  }

  // Handle cases where caller must allocate the result or a fir.box for it.
  if (mustRemapActualToDummyDescriptors)
    remapActualToDummyDescriptors(loc, callContext.converter,
                                  callContext.symMap, loweredActuals, caller,
                                  callContext.isBindcCall());
}

/// Lower calls to user procedures with actual arguments that have been
/// pre-lowered but not yet prepared according to the interface.
/// This can be called for elemental procedures, but only with scalar
/// arguments: if there are array arguments, it must be provided with
/// the array argument elements value and will return the corresponding
/// scalar result value.
static std::optional<hlfir::EntityWithAttributes>
genUserCall(Fortran::lower::PreparedActualArguments &loweredActuals,
            Fortran::lower::CallerInterface &caller,
            mlir::FunctionType callSiteType, CallContext &callContext) {
  mlir::Location loc = callContext.loc;
````
- **L1897 EN**: Closes the current lexical scope or compound statement.
  **L1897 CN**: 结束当前词法作用域或复合语句块。
- **L1898 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L1898 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L1899 EN**: Executes a standalone statement or declaration: `} break;`.
  **L1899 CN**: 执行一条独立语句或声明：`} break;`。
- **L1900 EN**: Closes the current lexical scope or compound statement.
  **L1900 CN**: 结束当前词法作用域或复合语句块。
- **L1901 EN**: Closes the current lexical scope or compound statement.
  **L1901 CN**: 结束当前词法作用域或复合语句块。
- **L1902 EN**: Blank line separating nearby declarations or logic blocks.
  **L1902 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1903 EN**: Comment explains nearby logic, intent, or metadata: `Handle cases where caller must allocate the result or a fir.box for it.`.
  **L1903 CN**: 注释说明附近代码的逻辑、意图或元数据：`Handle cases where caller must allocate the result or a fir.box for it.`。
- **L1904 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1904 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1905 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `remapActualToDummyDescriptors(loc, callContext.converter,`.
  **L1905 CN**: 继续一个多行参数列表、初始化器或聚合项：`remapActualToDummyDescriptors(loc, callContext.converter,`。
- **L1906 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `callContext.symMap, loweredActuals, caller,`.
  **L1906 CN**: 继续一个多行参数列表、初始化器或聚合项：`callContext.symMap, loweredActuals, caller,`。
- **L1907 EN**: Executes a call or declaration centered on `callContext.isBindcCall`.
  **L1907 CN**: 执行以 `callContext.isBindcCall` 为核心的调用或声明。
- **L1908 EN**: Closes the current lexical scope or compound statement.
  **L1908 CN**: 结束当前词法作用域或复合语句块。
- **L1909 EN**: Blank line separating nearby declarations or logic blocks.
  **L1909 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1910 EN**: Comment explains nearby logic, intent, or metadata: `Lower calls to user procedures with actual arguments that have been`.
  **L1910 CN**: 注释说明附近代码的逻辑、意图或元数据：`Lower calls to user procedures with actual arguments that have been`。
- **L1911 EN**: Comment explains nearby logic, intent, or metadata: `pre-lowered but not yet prepared according to the interface.`.
  **L1911 CN**: 注释说明附近代码的逻辑、意图或元数据：`pre-lowered but not yet prepared according to the interface.`。
- **L1912 EN**: Comment explains nearby logic, intent, or metadata: `This can be called for elemental procedures, but only with scalar`.
  **L1912 CN**: 注释说明附近代码的逻辑、意图或元数据：`This can be called for elemental procedures, but only with scalar`。
- **L1913 EN**: Comment explains nearby logic, intent, or metadata: `arguments: if there are array arguments, it must be provided with`.
  **L1913 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments: if there are array arguments, it must be provided with`。
- **L1914 EN**: Comment explains nearby logic, intent, or metadata: `the array argument elements value and will return the corresponding`.
  **L1914 CN**: 注释说明附近代码的逻辑、意图或元数据：`the array argument elements value and will return the corresponding`。
- **L1915 EN**: Comment explains nearby logic, intent, or metadata: `scalar result value.`.
  **L1915 CN**: 注释说明附近代码的逻辑、意图或元数据：`scalar result value.`。
- **L1916 EN**: Continues the surrounding expression or declaration: `static std::optional<hlfir::EntityWithAttributes>`.
  **L1916 CN**: 继续构造周围的表达式或声明：`static std::optional<hlfir::EntityWithAttributes>`。
- **L1917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genUserCall(Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L1917 CN**: 继续一个多行参数列表、初始化器或聚合项：`genUserCall(Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L1918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::CallerInterface &caller,`.
  **L1918 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::CallerInterface &caller,`。
- **L1919 EN**: Continues the surrounding expression or declaration: `mlir::FunctionType callSiteType, CallContext &callContext) {`.
  **L1919 CN**: 继续构造周围的表达式或声明：`mlir::FunctionType callSiteType, CallContext &callContext) {`。
- **L1920 EN**: Initializes variable `loc` from the right-hand expression.
  **L1920 CN**: 使用右侧表达式初始化变量 `loc`。

### Lines 1921-1944

````cpp
  llvm::SmallVector<CallCleanUp> callCleanUps;
  fir::FirOpBuilder &builder = callContext.getBuilder();

  prepareUserCallArguments(loweredActuals, caller, callSiteType, callContext,
                           callCleanUps);

  const bool isElemental = callContext.isElementalProcWithArrayArgs();
  // Prepare lowered arguments according to the interface
  // and map the lowered values to the dummy
  // arguments.
  auto [loweredResult, resultIsFinalized, callOp] =
      Fortran::lower::genCallOpAndResult(
          loc, callContext.converter, callContext.symMap, callContext.stmtCtx,
          caller, callSiteType, callContext.resultType, isElemental);

  // Clean-up associations and copy-in.
  // The association clean-ups are postponed to the end of the statement
  // lowering. The copy-in clean-ups may be delayed as well,
  // but they are done immediately after the call currently.
  llvm::SmallVector<CallCleanUp> associateCleanups;
  for (auto cleanUp : callCleanUps) {
    auto postponed =
        cleanUp.genCleanUp(loc, builder, /*postponeAssociates=*/true);
    if (postponed)
````
- **L1921 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<CallCleanUp> callCleanUps;`.
  **L1921 CN**: 执行一条独立语句或声明：`llvm::SmallVector<CallCleanUp> callCleanUps;`。
- **L1922 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L1922 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L1923 EN**: Blank line separating nearby declarations or logic blocks.
  **L1923 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1924 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `prepareUserCallArguments(loweredActuals, caller, callSiteType, callContext,`.
  **L1924 CN**: 继续一个多行参数列表、初始化器或聚合项：`prepareUserCallArguments(loweredActuals, caller, callSiteType, callContext,`。
- **L1925 EN**: Executes a standalone statement or declaration: `callCleanUps);`.
  **L1925 CN**: 执行一条独立语句或声明：`callCleanUps);`。
- **L1926 EN**: Blank line separating nearby declarations or logic blocks.
  **L1926 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1927 EN**: Initializes variable `isElemental` from the right-hand expression.
  **L1927 CN**: 使用右侧表达式初始化变量 `isElemental`。
- **L1928 EN**: Comment explains nearby logic, intent, or metadata: `Prepare lowered arguments according to the interface`.
  **L1928 CN**: 注释说明附近代码的逻辑、意图或元数据：`Prepare lowered arguments according to the interface`。
- **L1929 EN**: Comment explains nearby logic, intent, or metadata: `and map the lowered values to the dummy`.
  **L1929 CN**: 注释说明附近代码的逻辑、意图或元数据：`and map the lowered values to the dummy`。
- **L1930 EN**: Comment explains nearby logic, intent, or metadata: `arguments.`.
  **L1930 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments.`。
- **L1931 EN**: Continues the surrounding expression or declaration: `auto [loweredResult, resultIsFinalized, callOp] =`.
  **L1931 CN**: 继续构造周围的表达式或声明：`auto [loweredResult, resultIsFinalized, callOp] =`。
- **L1932 EN**: Continues logic associated with callable symbol `genCallOpAndResult`.
  **L1932 CN**: 继续与可调用符号 `genCallOpAndResult` 相关的逻辑。
- **L1933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, callContext.converter, callContext.symMap, callContext.stmtCtx,`.
  **L1933 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, callContext.converter, callContext.symMap, callContext.stmtCtx,`。
- **L1934 EN**: Executes a standalone statement or declaration: `caller, callSiteType, callContext.resultType, isElemental);`.
  **L1934 CN**: 执行一条独立语句或声明：`caller, callSiteType, callContext.resultType, isElemental);`。
- **L1935 EN**: Blank line separating nearby declarations or logic blocks.
  **L1935 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1936 EN**: Comment explains nearby logic, intent, or metadata: `Clean-up associations and copy-in.`.
  **L1936 CN**: 注释说明附近代码的逻辑、意图或元数据：`Clean-up associations and copy-in.`。
- **L1937 EN**: Comment explains nearby logic, intent, or metadata: `The association clean-ups are postponed to the end of the statement`.
  **L1937 CN**: 注释说明附近代码的逻辑、意图或元数据：`The association clean-ups are postponed to the end of the statement`。
- **L1938 EN**: Comment explains nearby logic, intent, or metadata: `lowering. The copy-in clean-ups may be delayed as well,`.
  **L1938 CN**: 注释说明附近代码的逻辑、意图或元数据：`lowering. The copy-in clean-ups may be delayed as well,`。
- **L1939 EN**: Comment explains nearby logic, intent, or metadata: `but they are done immediately after the call currently.`.
  **L1939 CN**: 注释说明附近代码的逻辑、意图或元数据：`but they are done immediately after the call currently.`。
- **L1940 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<CallCleanUp> associateCleanups;`.
  **L1940 CN**: 执行一条独立语句或声明：`llvm::SmallVector<CallCleanUp> associateCleanups;`。
- **L1941 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1941 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1942 EN**: Continues the surrounding expression or declaration: `auto postponed =`.
  **L1942 CN**: 继续构造周围的表达式或声明：`auto postponed =`。
- **L1943 EN**: Executes a call or declaration centered on `cleanUp.genCleanUp`.
  **L1943 CN**: 执行以 `cleanUp.genCleanUp` 为核心的调用或声明。
- **L1944 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1944 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1945-1968

````cpp
      associateCleanups.push_back(*postponed);
  }

  fir::FirOpBuilder *bldr = &builder;
  callContext.stmtCtx.attachCleanup([=]() {
    for (auto cleanUp : associateCleanups)
      (void)cleanUp.genCleanUp(loc, *bldr, /*postponeAssociates=*/false);
  });
  if (auto *entity = std::get_if<hlfir::EntityWithAttributes>(&loweredResult))
    return *entity;

  auto &result = std::get<fir::ExtendedValue>(loweredResult);

  // For procedure pointer function result, just return the call.
  if (callContext.resultType &&
      mlir::isa<fir::BoxProcType>(*callContext.resultType))
    return hlfir::EntityWithAttributes(fir::getBase(result));

  if (!fir::getBase(result))
    return std::nullopt; // subroutine call.

  if (fir::isPointerType(fir::getBase(result).getType()))
    return extendedValueToHlfirEntity(loc, builder, result, tempResultName);

````
- **L1945 EN**: Executes a call or declaration centered on `associateCleanups.push_back`.
  **L1945 CN**: 执行以 `associateCleanups.push_back` 为核心的调用或声明。
- **L1946 EN**: Closes the current lexical scope or compound statement.
  **L1946 CN**: 结束当前词法作用域或复合语句块。
- **L1947 EN**: Blank line separating nearby declarations or logic blocks.
  **L1947 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1948 EN**: Executes a standalone statement or declaration: `fir::FirOpBuilder *bldr = &builder;`.
  **L1948 CN**: 执行一条独立语句或声明：`fir::FirOpBuilder *bldr = &builder;`。
- **L1949 EN**: Starts a function, method, lambda, or structured scope: `callContext.stmtCtx.attachCleanup([=]() {`.
  **L1949 CN**: 开始一个函数、方法、lambda 或结构化作用域：`callContext.stmtCtx.attachCleanup([=]() {`。
- **L1950 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1950 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1951 EN**: Executes a call or declaration centered on `statement`.
  **L1951 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1952 EN**: Executes a standalone statement or declaration: `});`.
  **L1952 CN**: 执行一条独立语句或声明：`});`。
- **L1953 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1953 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1954 EN**: Returns from the current function with `*entity`.
  **L1954 CN**: 以 `*entity` 从当前函数返回。
- **L1955 EN**: Blank line separating nearby declarations or logic blocks.
  **L1955 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1956 EN**: Executes a call or declaration centered on `std::get<fir::ExtendedValue>`.
  **L1956 CN**: 执行以 `std::get<fir::ExtendedValue>` 为核心的调用或声明。
- **L1957 EN**: Blank line separating nearby declarations or logic blocks.
  **L1957 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1958 EN**: Comment explains nearby logic, intent, or metadata: `For procedure pointer function result, just return the call.`.
  **L1958 CN**: 注释说明附近代码的逻辑、意图或元数据：`For procedure pointer function result, just return the call.`。
- **L1959 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1959 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1960 EN**: Continues logic associated with callable symbol `BoxProcType>`.
  **L1960 CN**: 继续与可调用符号 `BoxProcType>` 相关的逻辑。
- **L1961 EN**: Returns from the current function with `hlfir::EntityWithAttributes(fir::getBase(result))`.
  **L1961 CN**: 以 `hlfir::EntityWithAttributes(fir::getBase(result))` 从当前函数返回。
- **L1962 EN**: Blank line separating nearby declarations or logic blocks.
  **L1962 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1963 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1963 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1964 EN**: Returns from the current function with `std::nullopt; // subroutine call.`.
  **L1964 CN**: 以 `std::nullopt; // subroutine call.` 从当前函数返回。
- **L1965 EN**: Blank line separating nearby declarations or logic blocks.
  **L1965 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1966 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1966 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1967 EN**: Returns from the current function with `extendedValueToHlfirEntity(loc, builder, result, tempResultName)`.
  **L1967 CN**: 以 `extendedValueToHlfirEntity(loc, builder, result, tempResultName)` 从当前函数返回。
- **L1968 EN**: Blank line separating nearby declarations or logic blocks.
  **L1968 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1969-1992

````cpp
  if (!resultIsFinalized) {
    hlfir::Entity resultEntity = extendedValueToHlfirEntity(
        loc, builder, result, tempResultName, /*insertBefore=*/callOp);
    // Allocatable result must be freed, other results are stack allocated.
    const auto *allocatable = result.getBoxOf<fir::MutableBoxValue>();
    const bool mustFree = allocatable != nullptr;
    resultEntity = loadTrivialScalar(loc, builder, resultEntity);
    if (resultEntity.isVariable()) {
      // If the result has no finalization, it can be moved into an expression.
      mlir::Value asExpr = hlfir::AsExprOp::create(
          builder, loc, resultEntity, builder.createBool(loc, mustFree));
      if (!isElemental) {
        // Insert clean-up for the expression, except for elemental call where
        // the cleaned-up is inserted at the array level.
        callContext.stmtCtx.attachCleanup([bldr = &builder, loc, asExpr]() {
          hlfir::DestroyOp::create(*bldr, loc, asExpr, /*finalize=*/false);
        });
      }
      return hlfir::EntityWithAttributes{asExpr};
    }
    if (allocatable)
      callContext.stmtCtx.attachCleanup(
          [bldr = &builder, loc, box = *allocatable]() {
            fir::factory::genFreememIfAllocated(*bldr, loc, box);
````
- **L1969 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1969 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1970 EN**: Continues logic associated with callable symbol `extendedValueToHlfirEntity`.
  **L1970 CN**: 继续与可调用符号 `extendedValueToHlfirEntity` 相关的逻辑。
- **L1971 EN**: Executes a standalone statement or declaration: `loc, builder, result, tempResultName, /*insertBefore=*/callOp);`.
  **L1971 CN**: 执行一条独立语句或声明：`loc, builder, result, tempResultName, /*insertBefore=*/callOp);`。
- **L1972 EN**: Comment explains nearby logic, intent, or metadata: `Allocatable result must be freed, other results are stack allocated.`.
  **L1972 CN**: 注释说明附近代码的逻辑、意图或元数据：`Allocatable result must be freed, other results are stack allocated.`。
- **L1973 EN**: Executes a call or declaration centered on `result.getBoxOf<fir::MutableBoxValue>`.
  **L1973 CN**: 执行以 `result.getBoxOf<fir::MutableBoxValue>` 为核心的调用或声明。
- **L1974 EN**: Initializes variable `mustFree` from the right-hand expression.
  **L1974 CN**: 使用右侧表达式初始化变量 `mustFree`。
- **L1975 EN**: Executes a call or declaration centered on `loadTrivialScalar`.
  **L1975 CN**: 执行以 `loadTrivialScalar` 为核心的调用或声明。
- **L1976 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1976 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1977 EN**: Comment explains nearby logic, intent, or metadata: `If the result has no finalization, it can be moved into an expression.`.
  **L1977 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the result has no finalization, it can be moved into an expression.`。
- **L1978 EN**: Continues logic associated with callable symbol `create`.
  **L1978 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1979 EN**: Executes a call or declaration centered on `builder.createBool`.
  **L1979 CN**: 执行以 `builder.createBool` 为核心的调用或声明。
- **L1980 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1980 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1981 EN**: Comment explains nearby logic, intent, or metadata: `Insert clean-up for the expression, except for elemental call where`.
  **L1981 CN**: 注释说明附近代码的逻辑、意图或元数据：`Insert clean-up for the expression, except for elemental call where`。
- **L1982 EN**: Comment explains nearby logic, intent, or metadata: `the cleaned-up is inserted at the array level.`.
  **L1982 CN**: 注释说明附近代码的逻辑、意图或元数据：`the cleaned-up is inserted at the array level.`。
- **L1983 EN**: Starts a function, method, lambda, or structured scope: `callContext.stmtCtx.attachCleanup([bldr = &builder, loc, asExpr]() {`.
  **L1983 CN**: 开始一个函数、方法、lambda 或结构化作用域：`callContext.stmtCtx.attachCleanup([bldr = &builder, loc, asExpr]() {`。
- **L1984 EN**: Executes a call or declaration centered on `hlfir::DestroyOp::create`.
  **L1984 CN**: 执行以 `hlfir::DestroyOp::create` 为核心的调用或声明。
- **L1985 EN**: Executes a standalone statement or declaration: `});`.
  **L1985 CN**: 执行一条独立语句或声明：`});`。
- **L1986 EN**: Closes the current lexical scope or compound statement.
  **L1986 CN**: 结束当前词法作用域或复合语句块。
- **L1987 EN**: Returns from the current function with `hlfir::EntityWithAttributes{asExpr}`.
  **L1987 CN**: 以 `hlfir::EntityWithAttributes{asExpr}` 从当前函数返回。
- **L1988 EN**: Closes the current lexical scope or compound statement.
  **L1988 CN**: 结束当前词法作用域或复合语句块。
- **L1989 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1989 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1990 EN**: Continues logic associated with callable symbol `attachCleanup`.
  **L1990 CN**: 继续与可调用符号 `attachCleanup` 相关的逻辑。
- **L1991 EN**: Starts a function, method, lambda, or structured scope: `[bldr = &builder, loc, box = *allocatable]() {`.
  **L1991 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[bldr = &builder, loc, box = *allocatable]() {`。
- **L1992 EN**: Executes a call or declaration centered on `fir::factory::genFreememIfAllocated`.
  **L1992 CN**: 执行以 `fir::factory::genFreememIfAllocated` 为核心的调用或声明。

### Lines 1993-2016

````cpp
          });
    return hlfir::EntityWithAttributes{resultEntity};
  }
  // If the result has finalization, it cannot be moved because use of its
  // value have been created in the statement context and may be emitted
  // after the hlfir.expr destroy, so the result is kept as a variable in
  // HLFIR. This may lead to copies when passing the result to an argument
  // with VALUE, and this do not convey the fact that the result will not
  // change, but is correct, and using hlfir.expr without the move would
  // trigger a copy that may be avoided.

  // Load allocatable results before emitting the hlfir.declare and drop its
  // lower bounds: this is not a variable From the Fortran point of view, so
  // the lower bounds are ones when inquired on the caller side.
  const auto *allocatable = result.getBoxOf<fir::MutableBoxValue>();
  fir::ExtendedValue loadedResult =
      allocatable
          ? fir::factory::genMutableBoxRead(builder, loc, *allocatable,
                                            /*mayBePolymorphic=*/true,
                                            /*preserveLowerBounds=*/false)
          : result;
  return extendedValueToHlfirEntity(
      loc, builder, loadedResult, tempResultName,
      /*insertBefore=*/!allocatable ? callOp : nullptr);
````
- **L1993 EN**: Executes a standalone statement or declaration: `});`.
  **L1993 CN**: 执行一条独立语句或声明：`});`。
- **L1994 EN**: Returns from the current function with `hlfir::EntityWithAttributes{resultEntity}`.
  **L1994 CN**: 以 `hlfir::EntityWithAttributes{resultEntity}` 从当前函数返回。
- **L1995 EN**: Closes the current lexical scope or compound statement.
  **L1995 CN**: 结束当前词法作用域或复合语句块。
- **L1996 EN**: Comment explains nearby logic, intent, or metadata: `If the result has finalization, it cannot be moved because use of its`.
  **L1996 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the result has finalization, it cannot be moved because use of its`。
- **L1997 EN**: Comment explains nearby logic, intent, or metadata: `value have been created in the statement context and may be emitted`.
  **L1997 CN**: 注释说明附近代码的逻辑、意图或元数据：`value have been created in the statement context and may be emitted`。
- **L1998 EN**: Comment explains nearby logic, intent, or metadata: `after the hlfir.expr destroy, so the result is kept as a variable in`.
  **L1998 CN**: 注释说明附近代码的逻辑、意图或元数据：`after the hlfir.expr destroy, so the result is kept as a variable in`。
- **L1999 EN**: Comment explains nearby logic, intent, or metadata: `HLFIR. This may lead to copies when passing the result to an argument`.
  **L1999 CN**: 注释说明附近代码的逻辑、意图或元数据：`HLFIR. This may lead to copies when passing the result to an argument`。
- **L2000 EN**: Comment explains nearby logic, intent, or metadata: `with VALUE, and this do not convey the fact that the result will not`.
  **L2000 CN**: 注释说明附近代码的逻辑、意图或元数据：`with VALUE, and this do not convey the fact that the result will not`。
- **L2001 EN**: Comment explains nearby logic, intent, or metadata: `change, but is correct, and using hlfir.expr without the move would`.
  **L2001 CN**: 注释说明附近代码的逻辑、意图或元数据：`change, but is correct, and using hlfir.expr without the move would`。
- **L2002 EN**: Comment explains nearby logic, intent, or metadata: `trigger a copy that may be avoided.`.
  **L2002 CN**: 注释说明附近代码的逻辑、意图或元数据：`trigger a copy that may be avoided.`。
- **L2003 EN**: Blank line separating nearby declarations or logic blocks.
  **L2003 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2004 EN**: Comment explains nearby logic, intent, or metadata: `Load allocatable results before emitting the hlfir.declare and drop its`.
  **L2004 CN**: 注释说明附近代码的逻辑、意图或元数据：`Load allocatable results before emitting the hlfir.declare and drop its`。
- **L2005 EN**: Comment explains nearby logic, intent, or metadata: `lower bounds: this is not a variable From the Fortran point of view, so`.
  **L2005 CN**: 注释说明附近代码的逻辑、意图或元数据：`lower bounds: this is not a variable From the Fortran point of view, so`。
- **L2006 EN**: Comment explains nearby logic, intent, or metadata: `the lower bounds are ones when inquired on the caller side.`.
  **L2006 CN**: 注释说明附近代码的逻辑、意图或元数据：`the lower bounds are ones when inquired on the caller side.`。
- **L2007 EN**: Executes a call or declaration centered on `result.getBoxOf<fir::MutableBoxValue>`.
  **L2007 CN**: 执行以 `result.getBoxOf<fir::MutableBoxValue>` 为核心的调用或声明。
- **L2008 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue loadedResult =`.
  **L2008 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue loadedResult =`。
- **L2009 EN**: Continues the surrounding expression or declaration: `allocatable`.
  **L2009 CN**: 继续构造周围的表达式或声明：`allocatable`。
- **L2010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `? fir::factory::genMutableBoxRead(builder, loc, *allocatable,`.
  **L2010 CN**: 继续一个多行参数列表、初始化器或聚合项：`? fir::factory::genMutableBoxRead(builder, loc, *allocatable,`。
- **L2011 EN**: Comment explains nearby logic, intent, or metadata: `mayBePolymorphic=*/true,`.
  **L2011 CN**: 注释说明附近代码的逻辑、意图或元数据：`mayBePolymorphic=*/true,`。
- **L2012 EN**: Comment explains nearby logic, intent, or metadata: `preserveLowerBounds=*/false)`.
  **L2012 CN**: 注释说明附近代码的逻辑、意图或元数据：`preserveLowerBounds=*/false)`。
- **L2013 EN**: Executes a standalone statement or declaration: `: result;`.
  **L2013 CN**: 执行一条独立语句或声明：`: result;`。
- **L2014 EN**: Returns from the current function with `extendedValueToHlfirEntity(`.
  **L2014 CN**: 以 `extendedValueToHlfirEntity(` 从当前函数返回。
- **L2015 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, builder, loadedResult, tempResultName,`.
  **L2015 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, builder, loadedResult, tempResultName,`。
- **L2016 EN**: Comment explains nearby logic, intent, or metadata: `insertBefore=*/!allocatable ? callOp : nullptr);`.
  **L2016 CN**: 注释说明附近代码的逻辑、意图或元数据：`insertBefore=*/!allocatable ? callOp : nullptr);`。

### Lines 2017-2040

````cpp
}

/// Create an optional dummy argument value from an entity that may be
/// absent. \p actualGetter callback returns hlfir::Entity denoting
/// the lowered actual argument. \p actualGetter can only return numerical
/// or logical scalar entity.
/// If the entity is considered absent according to 15.5.2.12 point 1., the
/// returned value is zero (or false), otherwise it is the value of the entity.
/// \p eleType specifies the entity's Fortran element type.
template <typename T>
static ExvAndCleanup genOptionalValue(fir::FirOpBuilder &builder,
                                      mlir::Location loc, mlir::Type eleType,
                                      T actualGetter, mlir::Value isPresent) {
  return {builder
              .genIfOp(loc, {eleType}, isPresent,
                       /*withElseRegion=*/true)
              .genThen([&]() {
                hlfir::Entity entity = actualGetter(loc, builder);
                assert(eleType == entity.getFortranElementType() &&
                       "result type mismatch in genOptionalValue");
                assert(entity.isScalar() && fir::isa_trivial(eleType) &&
                       "must be a numerical or logical scalar");
                mlir::Value val =
                    hlfir::loadTrivialScalar(loc, builder, entity);
````
- **L2017 EN**: Closes the current lexical scope or compound statement.
  **L2017 CN**: 结束当前词法作用域或复合语句块。
- **L2018 EN**: Blank line separating nearby declarations or logic blocks.
  **L2018 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2019 EN**: Comment explains nearby logic, intent, or metadata: `Create an optional dummy argument value from an entity that may be`.
  **L2019 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create an optional dummy argument value from an entity that may be`。
- **L2020 EN**: Comment explains nearby logic, intent, or metadata: `absent. \p actualGetter callback returns hlfir::Entity denoting`.
  **L2020 CN**: 注释说明附近代码的逻辑、意图或元数据：`absent. \p actualGetter callback returns hlfir::Entity denoting`。
- **L2021 EN**: Comment explains nearby logic, intent, or metadata: `the lowered actual argument. \p actualGetter can only return numerical`.
  **L2021 CN**: 注释说明附近代码的逻辑、意图或元数据：`the lowered actual argument. \p actualGetter can only return numerical`。
- **L2022 EN**: Comment explains nearby logic, intent, or metadata: `or logical scalar entity.`.
  **L2022 CN**: 注释说明附近代码的逻辑、意图或元数据：`or logical scalar entity.`。
- **L2023 EN**: Comment explains nearby logic, intent, or metadata: `If the entity is considered absent according to 15.5.2.12 point 1., the`.
  **L2023 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the entity is considered absent according to 15.5.2.12 point 1., the`。
- **L2024 EN**: Comment explains nearby logic, intent, or metadata: `returned value is zero (or false), otherwise it is the value of the entity.`.
  **L2024 CN**: 注释说明附近代码的逻辑、意图或元数据：`returned value is zero (or false), otherwise it is the value of the entity.`。
- **L2025 EN**: Comment explains nearby logic, intent, or metadata: `\p eleType specifies the entity's Fortran element type.`.
  **L2025 CN**: 注释说明附近代码的逻辑、意图或元数据：`\p eleType specifies the entity's Fortran element type.`。
- **L2026 EN**: Introduces template parameters or specialization context: `template <typename T>`.
  **L2026 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>`。
- **L2027 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static ExvAndCleanup genOptionalValue(fir::FirOpBuilder &builder,`.
  **L2027 CN**: 继续一个多行参数列表、初始化器或聚合项：`static ExvAndCleanup genOptionalValue(fir::FirOpBuilder &builder,`。
- **L2028 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, mlir::Type eleType,`.
  **L2028 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, mlir::Type eleType,`。
- **L2029 EN**: Continues the surrounding expression or declaration: `T actualGetter, mlir::Value isPresent) {`.
  **L2029 CN**: 继续构造周围的表达式或声明：`T actualGetter, mlir::Value isPresent) {`。
- **L2030 EN**: Returns from the current function with `{builder`.
  **L2030 CN**: 以 `{builder` 从当前函数返回。
- **L2031 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.genIfOp(loc, {eleType}, isPresent,`.
  **L2031 CN**: 继续一个多行参数列表、初始化器或聚合项：`.genIfOp(loc, {eleType}, isPresent,`。
- **L2032 EN**: Comment explains nearby logic, intent, or metadata: `withElseRegion=*/true)`.
  **L2032 CN**: 注释说明附近代码的逻辑、意图或元数据：`withElseRegion=*/true)`。
- **L2033 EN**: Starts a function, method, lambda, or structured scope: `.genThen([&]() {`.
  **L2033 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genThen([&]() {`。
- **L2034 EN**: Initializes variable `entity` from the right-hand expression.
  **L2034 CN**: 使用右侧表达式初始化变量 `entity`。
- **L2035 EN**: Checks an internal invariant in debug builds.
  **L2035 CN**: 在调试构建中检查内部不变式。
- **L2036 EN**: Executes a standalone statement or declaration: `"result type mismatch in genOptionalValue");`.
  **L2036 CN**: 执行一条独立语句或声明：`"result type mismatch in genOptionalValue");`。
- **L2037 EN**: Checks an internal invariant in debug builds.
  **L2037 CN**: 在调试构建中检查内部不变式。
- **L2038 EN**: Executes a standalone statement or declaration: `"must be a numerical or logical scalar");`.
  **L2038 CN**: 执行一条独立语句或声明：`"must be a numerical or logical scalar");`。
- **L2039 EN**: Continues the surrounding expression or declaration: `mlir::Value val =`.
  **L2039 CN**: 继续构造周围的表达式或声明：`mlir::Value val =`。
- **L2040 EN**: Executes a call or declaration centered on `hlfir::loadTrivialScalar`.
  **L2040 CN**: 执行以 `hlfir::loadTrivialScalar` 为核心的调用或声明。

### Lines 2041-2064

````cpp
                fir::ResultOp::create(builder, loc, val);
              })
              .genElse([&]() {
                mlir::Value zero =
                    fir::factory::createZeroValue(builder, loc, eleType);
                fir::ResultOp::create(builder, loc, zero);
              })
              .getResults()[0],
          std::nullopt};
}

/// Create an optional dummy argument address from \p entity that may be
/// absent. If \p entity is considered absent according to 15.5.2.12 point 1.,
/// the returned value is a null pointer, otherwise it is the address of \p
/// entity.
static ExvAndCleanup genOptionalAddr(fir::FirOpBuilder &builder,
                                     mlir::Location loc, hlfir::Entity entity,
                                     mlir::Value isPresent) {
  auto [exv, cleanup] = hlfir::translateToExtendedValue(loc, builder, entity);
  // If it is an exv pointer/allocatable, then it cannot be absent
  // because it is passed to a non-pointer/non-allocatable.
  if (const auto *box = exv.getBoxOf<fir::MutableBoxValue>())
    return {fir::factory::genMutableBoxRead(builder, loc, *box), cleanup};
  // If this is not a POINTER or ALLOCATABLE, then it is already an OPTIONAL
````
- **L2041 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L2041 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L2042 EN**: Continues the surrounding expression or declaration: `})`.
  **L2042 CN**: 继续构造周围的表达式或声明：`})`。
- **L2043 EN**: Starts a function, method, lambda, or structured scope: `.genElse([&]() {`.
  **L2043 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genElse([&]() {`。
- **L2044 EN**: Continues the surrounding expression or declaration: `mlir::Value zero =`.
  **L2044 CN**: 继续构造周围的表达式或声明：`mlir::Value zero =`。
- **L2045 EN**: Executes a call or declaration centered on `fir::factory::createZeroValue`.
  **L2045 CN**: 执行以 `fir::factory::createZeroValue` 为核心的调用或声明。
- **L2046 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L2046 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L2047 EN**: Continues the surrounding expression or declaration: `})`.
  **L2047 CN**: 继续构造周围的表达式或声明：`})`。
- **L2048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.getResults()[0],`.
  **L2048 CN**: 继续一个多行参数列表、初始化器或聚合项：`.getResults()[0],`。
- **L2049 EN**: Executes a standalone statement or declaration: `std::nullopt};`.
  **L2049 CN**: 执行一条独立语句或声明：`std::nullopt};`。
- **L2050 EN**: Closes the current lexical scope or compound statement.
  **L2050 CN**: 结束当前词法作用域或复合语句块。
- **L2051 EN**: Blank line separating nearby declarations or logic blocks.
  **L2051 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2052 EN**: Comment explains nearby logic, intent, or metadata: `Create an optional dummy argument address from \p entity that may be`.
  **L2052 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create an optional dummy argument address from \p entity that may be`。
- **L2053 EN**: Comment explains nearby logic, intent, or metadata: `absent. If \p entity is considered absent according to 15.5.2.12 point 1.,`.
  **L2053 CN**: 注释说明附近代码的逻辑、意图或元数据：`absent. If \p entity is considered absent according to 15.5.2.12 point 1.,`。
- **L2054 EN**: Comment explains nearby logic, intent, or metadata: `the returned value is a null pointer, otherwise it is the address of \p`.
  **L2054 CN**: 注释说明附近代码的逻辑、意图或元数据：`the returned value is a null pointer, otherwise it is the address of \p`。
- **L2055 EN**: Comment explains nearby logic, intent, or metadata: `entity.`.
  **L2055 CN**: 注释说明附近代码的逻辑、意图或元数据：`entity.`。
- **L2056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static ExvAndCleanup genOptionalAddr(fir::FirOpBuilder &builder,`.
  **L2056 CN**: 继续一个多行参数列表、初始化器或聚合项：`static ExvAndCleanup genOptionalAddr(fir::FirOpBuilder &builder,`。
- **L2057 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, hlfir::Entity entity,`.
  **L2057 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, hlfir::Entity entity,`。
- **L2058 EN**: Continues the surrounding expression or declaration: `mlir::Value isPresent) {`.
  **L2058 CN**: 继续构造周围的表达式或声明：`mlir::Value isPresent) {`。
- **L2059 EN**: Executes a call or declaration centered on `hlfir::translateToExtendedValue`.
  **L2059 CN**: 执行以 `hlfir::translateToExtendedValue` 为核心的调用或声明。
- **L2060 EN**: Comment explains nearby logic, intent, or metadata: `If it is an exv pointer/allocatable, then it cannot be absent`.
  **L2060 CN**: 注释说明附近代码的逻辑、意图或元数据：`If it is an exv pointer/allocatable, then it cannot be absent`。
- **L2061 EN**: Comment explains nearby logic, intent, or metadata: `because it is passed to a non-pointer/non-allocatable.`.
  **L2061 CN**: 注释说明附近代码的逻辑、意图或元数据：`because it is passed to a non-pointer/non-allocatable.`。
- **L2062 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2062 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2063 EN**: Returns from the current function with `{fir::factory::genMutableBoxRead(builder, loc, *box), cleanup}`.
  **L2063 CN**: 以 `{fir::factory::genMutableBoxRead(builder, loc, *box), cleanup}` 从当前函数返回。
- **L2064 EN**: Comment explains nearby logic, intent, or metadata: `If this is not a POINTER or ALLOCATABLE, then it is already an OPTIONAL`.
  **L2064 CN**: 注释说明附近代码的逻辑、意图或元数据：`If this is not a POINTER or ALLOCATABLE, then it is already an OPTIONAL`。

### Lines 2065-2088

````cpp
  // address and can be passed directly.
  return {exv, cleanup};
}

/// Create an optional dummy argument address from \p entity that may be
/// absent. If \p entity is considered absent according to 15.5.2.12 point 1.,
/// the returned value is an absent fir.box, otherwise it is a fir.box
/// describing \p entity.
static ExvAndCleanup genOptionalBox(fir::FirOpBuilder &builder,
                                    mlir::Location loc, hlfir::Entity entity,
                                    mlir::Value isPresent) {
  auto [exv, cleanup] = hlfir::translateToExtendedValue(loc, builder, entity);

  // Non allocatable/pointer optional box -> simply forward
  if (exv.getBoxOf<fir::BoxValue>())
    return {exv, cleanup};

  fir::ExtendedValue newExv = exv;
  // Optional allocatable/pointer -> Cannot be absent, but need to translate
  // unallocated/diassociated into absent fir.box.
  if (const auto *box = exv.getBoxOf<fir::MutableBoxValue>())
    newExv = fir::factory::genMutableBoxRead(builder, loc, *box);

  // createBox will not do create any invalid memory dereferences if exv is
````
- **L2065 EN**: Comment explains nearby logic, intent, or metadata: `address and can be passed directly.`.
  **L2065 CN**: 注释说明附近代码的逻辑、意图或元数据：`address and can be passed directly.`。
- **L2066 EN**: Returns from the current function with `{exv, cleanup}`.
  **L2066 CN**: 以 `{exv, cleanup}` 从当前函数返回。
- **L2067 EN**: Closes the current lexical scope or compound statement.
  **L2067 CN**: 结束当前词法作用域或复合语句块。
- **L2068 EN**: Blank line separating nearby declarations or logic blocks.
  **L2068 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2069 EN**: Comment explains nearby logic, intent, or metadata: `Create an optional dummy argument address from \p entity that may be`.
  **L2069 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create an optional dummy argument address from \p entity that may be`。
- **L2070 EN**: Comment explains nearby logic, intent, or metadata: `absent. If \p entity is considered absent according to 15.5.2.12 point 1.,`.
  **L2070 CN**: 注释说明附近代码的逻辑、意图或元数据：`absent. If \p entity is considered absent according to 15.5.2.12 point 1.,`。
- **L2071 EN**: Comment explains nearby logic, intent, or metadata: `the returned value is an absent fir.box, otherwise it is a fir.box`.
  **L2071 CN**: 注释说明附近代码的逻辑、意图或元数据：`the returned value is an absent fir.box, otherwise it is a fir.box`。
- **L2072 EN**: Comment explains nearby logic, intent, or metadata: `describing \p entity.`.
  **L2072 CN**: 注释说明附近代码的逻辑、意图或元数据：`describing \p entity.`。
- **L2073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static ExvAndCleanup genOptionalBox(fir::FirOpBuilder &builder,`.
  **L2073 CN**: 继续一个多行参数列表、初始化器或聚合项：`static ExvAndCleanup genOptionalBox(fir::FirOpBuilder &builder,`。
- **L2074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, hlfir::Entity entity,`.
  **L2074 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, hlfir::Entity entity,`。
- **L2075 EN**: Continues the surrounding expression or declaration: `mlir::Value isPresent) {`.
  **L2075 CN**: 继续构造周围的表达式或声明：`mlir::Value isPresent) {`。
- **L2076 EN**: Executes a call or declaration centered on `hlfir::translateToExtendedValue`.
  **L2076 CN**: 执行以 `hlfir::translateToExtendedValue` 为核心的调用或声明。
- **L2077 EN**: Blank line separating nearby declarations or logic blocks.
  **L2077 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2078 EN**: Comment explains nearby logic, intent, or metadata: `Non allocatable/pointer optional box -> simply forward`.
  **L2078 CN**: 注释说明附近代码的逻辑、意图或元数据：`Non allocatable/pointer optional box -> simply forward`。
- **L2079 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2079 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2080 EN**: Returns from the current function with `{exv, cleanup}`.
  **L2080 CN**: 以 `{exv, cleanup}` 从当前函数返回。
- **L2081 EN**: Blank line separating nearby declarations or logic blocks.
  **L2081 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2082 EN**: Initializes variable `newExv` from the right-hand expression.
  **L2082 CN**: 使用右侧表达式初始化变量 `newExv`。
- **L2083 EN**: Comment explains nearby logic, intent, or metadata: `Optional allocatable/pointer -> Cannot be absent, but need to translate`.
  **L2083 CN**: 注释说明附近代码的逻辑、意图或元数据：`Optional allocatable/pointer -> Cannot be absent, but need to translate`。
- **L2084 EN**: Comment explains nearby logic, intent, or metadata: `unallocated/diassociated into absent fir.box.`.
  **L2084 CN**: 注释说明附近代码的逻辑、意图或元数据：`unallocated/diassociated into absent fir.box.`。
- **L2085 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2085 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2086 EN**: Executes a call or declaration centered on `fir::factory::genMutableBoxRead`.
  **L2086 CN**: 执行以 `fir::factory::genMutableBoxRead` 为核心的调用或声明。
- **L2087 EN**: Blank line separating nearby declarations or logic blocks.
  **L2087 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2088 EN**: Comment explains nearby logic, intent, or metadata: `createBox will not do create any invalid memory dereferences if exv is`.
  **L2088 CN**: 注释说明附近代码的逻辑、意图或元数据：`createBox will not do create any invalid memory dereferences if exv is`。

### Lines 2089-2112

````cpp
  // absent. The created fir.box will not be usable, but the SelectOp below
  // ensures it won't be.
  mlir::Value box = builder.createBox(loc, newExv);
  mlir::Type boxType = box.getType();
  auto absent = fir::AbsentOp::create(builder, loc, boxType);
  auto boxOrAbsent = mlir::arith::SelectOp::create(builder, loc, boxType,
                                                   isPresent, box, absent);
  return {fir::BoxValue(boxOrAbsent), cleanup};
}

/// Lower calls to intrinsic procedures with custom optional handling where the
/// actual arguments have been pre-lowered
static std::optional<hlfir::EntityWithAttributes> genCustomIntrinsicRefCore(
    Fortran::lower::PreparedActualArguments &loweredActuals,
    const Fortran::evaluate::SpecificIntrinsic *intrinsic,
    CallContext &callContext) {
  auto &builder = callContext.getBuilder();
  const auto &loc = callContext.loc;
  assert(intrinsic &&
         Fortran::lower::intrinsicRequiresCustomOptionalHandling(
             callContext.procRef, *intrinsic, callContext.converter));

  // helper to get a particular prepared argument
  auto getArgument = [&](std::size_t i, bool loadArg) -> fir::ExtendedValue {
````
- **L2089 EN**: Comment explains nearby logic, intent, or metadata: `absent. The created fir.box will not be usable, but the SelectOp below`.
  **L2089 CN**: 注释说明附近代码的逻辑、意图或元数据：`absent. The created fir.box will not be usable, but the SelectOp below`。
- **L2090 EN**: Comment explains nearby logic, intent, or metadata: `ensures it won't be.`.
  **L2090 CN**: 注释说明附近代码的逻辑、意图或元数据：`ensures it won't be.`。
- **L2091 EN**: Initializes variable `box` from the right-hand expression.
  **L2091 CN**: 使用右侧表达式初始化变量 `box`。
- **L2092 EN**: Initializes variable `boxType` from the right-hand expression.
  **L2092 CN**: 使用右侧表达式初始化变量 `boxType`。
- **L2093 EN**: Initializes variable `absent` from the right-hand expression.
  **L2093 CN**: 使用右侧表达式初始化变量 `absent`。
- **L2094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto boxOrAbsent = mlir::arith::SelectOp::create(builder, loc, boxType,`.
  **L2094 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto boxOrAbsent = mlir::arith::SelectOp::create(builder, loc, boxType,`。
- **L2095 EN**: Executes a standalone statement or declaration: `isPresent, box, absent);`.
  **L2095 CN**: 执行一条独立语句或声明：`isPresent, box, absent);`。
- **L2096 EN**: Returns from the current function with `{fir::BoxValue(boxOrAbsent), cleanup}`.
  **L2096 CN**: 以 `{fir::BoxValue(boxOrAbsent), cleanup}` 从当前函数返回。
- **L2097 EN**: Closes the current lexical scope or compound statement.
  **L2097 CN**: 结束当前词法作用域或复合语句块。
- **L2098 EN**: Blank line separating nearby declarations or logic blocks.
  **L2098 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2099 EN**: Comment explains nearby logic, intent, or metadata: `Lower calls to intrinsic procedures with custom optional handling where the`.
  **L2099 CN**: 注释说明附近代码的逻辑、意图或元数据：`Lower calls to intrinsic procedures with custom optional handling where the`。
- **L2100 EN**: Comment explains nearby logic, intent, or metadata: `actual arguments have been pre-lowered`.
  **L2100 CN**: 注释说明附近代码的逻辑、意图或元数据：`actual arguments have been pre-lowered`。
- **L2101 EN**: Continues logic associated with callable symbol `genCustomIntrinsicRefCore`.
  **L2101 CN**: 继续与可调用符号 `genCustomIntrinsicRefCore` 相关的逻辑。
- **L2102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L2102 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L2103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::SpecificIntrinsic *intrinsic,`.
  **L2103 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::SpecificIntrinsic *intrinsic,`。
- **L2104 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L2104 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L2105 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L2105 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L2106 EN**: Executes a standalone statement or declaration: `const auto &loc = callContext.loc;`.
  **L2106 CN**: 执行一条独立语句或声明：`const auto &loc = callContext.loc;`。
- **L2107 EN**: Checks an internal invariant in debug builds.
  **L2107 CN**: 在调试构建中检查内部不变式。
- **L2108 EN**: Continues logic associated with callable symbol `intrinsicRequiresCustomOptionalHandling`.
  **L2108 CN**: 继续与可调用符号 `intrinsicRequiresCustomOptionalHandling` 相关的逻辑。
- **L2109 EN**: Executes a standalone statement or declaration: `callContext.procRef, *intrinsic, callContext.converter));`.
  **L2109 CN**: 执行一条独立语句或声明：`callContext.procRef, *intrinsic, callContext.converter));`。
- **L2110 EN**: Blank line separating nearby declarations or logic blocks.
  **L2110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2111 EN**: Comment explains nearby logic, intent, or metadata: `helper to get a particular prepared argument`.
  **L2111 CN**: 注释说明附近代码的逻辑、意图或元数据：`helper to get a particular prepared argument`。
- **L2112 EN**: Starts a function, method, lambda, or structured scope: `auto getArgument = [&](std::size_t i, bool loadArg) -> fir::ExtendedValue {`.
  **L2112 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto getArgument = [&](std::size_t i, bool loadArg) -> fir::ExtendedValue {`。

### Lines 2113-2136

````cpp
    if (!loweredActuals[i])
      return fir::getAbsentIntrinsicArgument();
    hlfir::Entity actual = loweredActuals[i]->getActual(loc, builder);
    if (loadArg && fir::conformsWithPassByRef(actual.getType())) {
      return hlfir::loadTrivialScalar(loc, builder, actual);
    }
    return Fortran::lower::translateToExtendedValue(loc, builder, actual,
                                                    callContext.stmtCtx);
  };
  // helper to get the isPresent flag for a particular prepared argument
  auto isPresent = [&](std::size_t i) -> std::optional<mlir::Value> {
    if (!loweredActuals[i])
      return {builder.createBool(loc, false)};
    if (loweredActuals[i]->handleDynamicOptional())
      return {loweredActuals[i]->getIsPresent()};
    return std::nullopt;
  };

  assert(callContext.resultType &&
         "the elemental intrinsics with custom handling are all functions");
  // if callContext.resultType is an array then this was originally an elemental
  // call. What we are lowering here is inside the kernel of the hlfir.elemental
  // so we should return the scalar type. If the return type is already a scalar
  // then it should be unchanged here.
````
- **L2113 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2113 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2114 EN**: Returns from the current function with `fir::getAbsentIntrinsicArgument()`.
  **L2114 CN**: 以 `fir::getAbsentIntrinsicArgument()` 从当前函数返回。
- **L2115 EN**: Initializes variable `actual` from the right-hand expression.
  **L2115 CN**: 使用右侧表达式初始化变量 `actual`。
- **L2116 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2116 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2117 EN**: Returns from the current function with `hlfir::loadTrivialScalar(loc, builder, actual)`.
  **L2117 CN**: 以 `hlfir::loadTrivialScalar(loc, builder, actual)` 从当前函数返回。
- **L2118 EN**: Closes the current lexical scope or compound statement.
  **L2118 CN**: 结束当前词法作用域或复合语句块。
- **L2119 EN**: Returns from the current function with `Fortran::lower::translateToExtendedValue(loc, builder, actual,`.
  **L2119 CN**: 以 `Fortran::lower::translateToExtendedValue(loc, builder, actual,` 从当前函数返回。
- **L2120 EN**: Executes a standalone statement or declaration: `callContext.stmtCtx);`.
  **L2120 CN**: 执行一条独立语句或声明：`callContext.stmtCtx);`。
- **L2121 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2121 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2122 EN**: Comment explains nearby logic, intent, or metadata: `helper to get the isPresent flag for a particular prepared argument`.
  **L2122 CN**: 注释说明附近代码的逻辑、意图或元数据：`helper to get the isPresent flag for a particular prepared argument`。
- **L2123 EN**: Starts a function, method, lambda, or structured scope: `auto isPresent = [&](std::size_t i) -> std::optional<mlir::Value> {`.
  **L2123 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto isPresent = [&](std::size_t i) -> std::optional<mlir::Value> {`。
- **L2124 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2124 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2125 EN**: Returns from the current function with `{builder.createBool(loc, false)}`.
  **L2125 CN**: 以 `{builder.createBool(loc, false)}` 从当前函数返回。
- **L2126 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2126 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2127 EN**: Returns from the current function with `{loweredActuals[i]->getIsPresent()}`.
  **L2127 CN**: 以 `{loweredActuals[i]->getIsPresent()}` 从当前函数返回。
- **L2128 EN**: Returns from the current function with `std::nullopt`.
  **L2128 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2129 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2129 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2130 EN**: Blank line separating nearby declarations or logic blocks.
  **L2130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2131 EN**: Checks an internal invariant in debug builds.
  **L2131 CN**: 在调试构建中检查内部不变式。
- **L2132 EN**: Executes a standalone statement or declaration: `"the elemental intrinsics with custom handling are all functions");`.
  **L2132 CN**: 执行一条独立语句或声明：`"the elemental intrinsics with custom handling are all functions");`。
- **L2133 EN**: Comment explains nearby logic, intent, or metadata: `if callContext.resultType is an array then this was originally an elemental`.
  **L2133 CN**: 注释说明附近代码的逻辑、意图或元数据：`if callContext.resultType is an array then this was originally an elemental`。
- **L2134 EN**: Comment explains nearby logic, intent, or metadata: `call. What we are lowering here is inside the kernel of the hlfir.elemental`.
  **L2134 CN**: 注释说明附近代码的逻辑、意图或元数据：`call. What we are lowering here is inside the kernel of the hlfir.elemental`。
- **L2135 EN**: Comment explains nearby logic, intent, or metadata: `so we should return the scalar type. If the return type is already a scalar`.
  **L2135 CN**: 注释说明附近代码的逻辑、意图或元数据：`so we should return the scalar type. If the return type is already a scalar`。
- **L2136 EN**: Comment explains nearby logic, intent, or metadata: `then it should be unchanged here.`.
  **L2136 CN**: 注释说明附近代码的逻辑、意图或元数据：`then it should be unchanged here.`。

### Lines 2137-2160

````cpp
  mlir::Type resTy = hlfir::getFortranElementType(*callContext.resultType);
  fir::ExtendedValue result = Fortran::lower::lowerCustomIntrinsic(
      builder, loc, callContext.getProcedureName(), resTy, isPresent,
      getArgument, loweredActuals.size(), callContext.stmtCtx);

  return {hlfir::EntityWithAttributes{extendedValueToHlfirEntity(
      loc, builder, result, ".tmp.custom_intrinsic_result")}};
}

/// Lower calls to intrinsic procedures with actual arguments that have been
/// pre-lowered but have not yet been prepared according to the interface.
static std::optional<hlfir::EntityWithAttributes>
genIntrinsicRefCore(Fortran::lower::PreparedActualArguments &loweredActuals,
                    const Fortran::evaluate::SpecificIntrinsic *intrinsic,
                    const fir::IntrinsicHandlerEntry &intrinsicEntry,
                    CallContext &callContext) {
  auto &converter = callContext.converter;
  if (intrinsic && Fortran::lower::intrinsicRequiresCustomOptionalHandling(
                       callContext.procRef, *intrinsic, converter))
    return genCustomIntrinsicRefCore(loweredActuals, intrinsic, callContext);
  llvm::SmallVector<fir::ExtendedValue> operands;
  llvm::SmallVector<hlfir::CleanupFunction> cleanupFns;
  auto addToCleanups = [&cleanupFns](std::optional<hlfir::CleanupFunction> fn) {
    if (fn)
````
- **L2137 EN**: Initializes variable `resTy` from the right-hand expression.
  **L2137 CN**: 使用右侧表达式初始化变量 `resTy`。
- **L2138 EN**: Continues logic associated with callable symbol `lowerCustomIntrinsic`.
  **L2138 CN**: 继续与可调用符号 `lowerCustomIntrinsic` 相关的逻辑。
- **L2139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, callContext.getProcedureName(), resTy, isPresent,`.
  **L2139 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, callContext.getProcedureName(), resTy, isPresent,`。
- **L2140 EN**: Executes a call or declaration centered on `loweredActuals.size`.
  **L2140 CN**: 执行以 `loweredActuals.size` 为核心的调用或声明。
- **L2141 EN**: Blank line separating nearby declarations or logic blocks.
  **L2141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2142 EN**: Returns from the current function with `{hlfir::EntityWithAttributes{extendedValueToHlfirEntity(`.
  **L2142 CN**: 以 `{hlfir::EntityWithAttributes{extendedValueToHlfirEntity(` 从当前函数返回。
- **L2143 EN**: Executes a standalone statement or declaration: `loc, builder, result, ".tmp.custom_intrinsic_result")}};`.
  **L2143 CN**: 执行一条独立语句或声明：`loc, builder, result, ".tmp.custom_intrinsic_result")}};`。
- **L2144 EN**: Closes the current lexical scope or compound statement.
  **L2144 CN**: 结束当前词法作用域或复合语句块。
- **L2145 EN**: Blank line separating nearby declarations or logic blocks.
  **L2145 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2146 EN**: Comment explains nearby logic, intent, or metadata: `Lower calls to intrinsic procedures with actual arguments that have been`.
  **L2146 CN**: 注释说明附近代码的逻辑、意图或元数据：`Lower calls to intrinsic procedures with actual arguments that have been`。
- **L2147 EN**: Comment explains nearby logic, intent, or metadata: `pre-lowered but have not yet been prepared according to the interface.`.
  **L2147 CN**: 注释说明附近代码的逻辑、意图或元数据：`pre-lowered but have not yet been prepared according to the interface.`。
- **L2148 EN**: Continues the surrounding expression or declaration: `static std::optional<hlfir::EntityWithAttributes>`.
  **L2148 CN**: 继续构造周围的表达式或声明：`static std::optional<hlfir::EntityWithAttributes>`。
- **L2149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genIntrinsicRefCore(Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L2149 CN**: 继续一个多行参数列表、初始化器或聚合项：`genIntrinsicRefCore(Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L2150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::SpecificIntrinsic *intrinsic,`.
  **L2150 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::SpecificIntrinsic *intrinsic,`。
- **L2151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicHandlerEntry &intrinsicEntry,`.
  **L2151 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicHandlerEntry &intrinsicEntry,`。
- **L2152 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L2152 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L2153 EN**: Executes a standalone statement or declaration: `auto &converter = callContext.converter;`.
  **L2153 CN**: 执行一条独立语句或声明：`auto &converter = callContext.converter;`。
- **L2154 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2154 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2155 EN**: Continues the surrounding expression or declaration: `callContext.procRef, *intrinsic, converter))`.
  **L2155 CN**: 继续构造周围的表达式或声明：`callContext.procRef, *intrinsic, converter))`。
- **L2156 EN**: Returns from the current function with `genCustomIntrinsicRefCore(loweredActuals, intrinsic, callContext)`.
  **L2156 CN**: 以 `genCustomIntrinsicRefCore(loweredActuals, intrinsic, callContext)` 从当前函数返回。
- **L2157 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<fir::ExtendedValue> operands;`.
  **L2157 CN**: 执行一条独立语句或声明：`llvm::SmallVector<fir::ExtendedValue> operands;`。
- **L2158 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<hlfir::CleanupFunction> cleanupFns;`.
  **L2158 CN**: 执行一条独立语句或声明：`llvm::SmallVector<hlfir::CleanupFunction> cleanupFns;`。
- **L2159 EN**: Starts a function, method, lambda, or structured scope: `auto addToCleanups = [&cleanupFns](std::optional<hlfir::CleanupFunction> fn) {`.
  **L2159 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto addToCleanups = [&cleanupFns](std::optional<hlfir::CleanupFunction> fn) {`。
- **L2160 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2160 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2161-2184

````cpp
      cleanupFns.emplace_back(std::move(*fn));
  };
  auto &stmtCtx = callContext.stmtCtx;
  fir::FirOpBuilder &builder = callContext.getBuilder();
  mlir::Location loc = callContext.loc;
  const fir::IntrinsicArgumentLoweringRules *argLowering =
      intrinsicEntry.getArgumentLoweringRules();
  for (auto arg : llvm::enumerate(loweredActuals)) {
    if (!arg.value()) {
      operands.emplace_back(fir::getAbsentIntrinsicArgument());
      continue;
    }
    if (!argLowering) {
      // No argument lowering instruction, lower by value.
      assert(!arg.value()->handleDynamicOptional() &&
             "should use genOptionalValue");
      hlfir::Entity actual = arg.value()->getActual(loc, builder);
      operands.emplace_back(
          Fortran::lower::convertToValue(loc, converter, actual, stmtCtx));
      continue;
    }
    // Helper to get the type of the Fortran expression in case it is a
    // computed value that must be placed in memory (logicals are computed as
    // i1, but must be placed in memory as fir.logical).
````
- **L2161 EN**: Executes a call or declaration centered on `cleanupFns.emplace_back`.
  **L2161 CN**: 执行以 `cleanupFns.emplace_back` 为核心的调用或声明。
- **L2162 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2162 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2163 EN**: Executes a standalone statement or declaration: `auto &stmtCtx = callContext.stmtCtx;`.
  **L2163 CN**: 执行一条独立语句或声明：`auto &stmtCtx = callContext.stmtCtx;`。
- **L2164 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L2164 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L2165 EN**: Initializes variable `loc` from the right-hand expression.
  **L2165 CN**: 使用右侧表达式初始化变量 `loc`。
- **L2166 EN**: Continues the surrounding expression or declaration: `const fir::IntrinsicArgumentLoweringRules *argLowering =`.
  **L2166 CN**: 继续构造周围的表达式或声明：`const fir::IntrinsicArgumentLoweringRules *argLowering =`。
- **L2167 EN**: Executes a call or declaration centered on `intrinsicEntry.getArgumentLoweringRules`.
  **L2167 CN**: 执行以 `intrinsicEntry.getArgumentLoweringRules` 为核心的调用或声明。
- **L2168 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2168 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2169 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2169 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2170 EN**: Executes a call or declaration centered on `operands.emplace_back`.
  **L2170 CN**: 执行以 `operands.emplace_back` 为核心的调用或声明。
- **L2171 EN**: Skips to the next loop iteration.
  **L2171 CN**: 跳到下一次循环迭代。
- **L2172 EN**: Closes the current lexical scope or compound statement.
  **L2172 CN**: 结束当前词法作用域或复合语句块。
- **L2173 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2173 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2174 EN**: Comment explains nearby logic, intent, or metadata: `No argument lowering instruction, lower by value.`.
  **L2174 CN**: 注释说明附近代码的逻辑、意图或元数据：`No argument lowering instruction, lower by value.`。
- **L2175 EN**: Checks an internal invariant in debug builds.
  **L2175 CN**: 在调试构建中检查内部不变式。
- **L2176 EN**: Executes a standalone statement or declaration: `"should use genOptionalValue");`.
  **L2176 CN**: 执行一条独立语句或声明：`"should use genOptionalValue");`。
- **L2177 EN**: Initializes variable `actual` from the right-hand expression.
  **L2177 CN**: 使用右侧表达式初始化变量 `actual`。
- **L2178 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L2178 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L2179 EN**: Executes a call or declaration centered on `Fortran::lower::convertToValue`.
  **L2179 CN**: 执行以 `Fortran::lower::convertToValue` 为核心的调用或声明。
- **L2180 EN**: Skips to the next loop iteration.
  **L2180 CN**: 跳到下一次循环迭代。
- **L2181 EN**: Closes the current lexical scope or compound statement.
  **L2181 CN**: 结束当前词法作用域或复合语句块。
- **L2182 EN**: Comment explains nearby logic, intent, or metadata: `Helper to get the type of the Fortran expression in case it is a`.
  **L2182 CN**: 注释说明附近代码的逻辑、意图或元数据：`Helper to get the type of the Fortran expression in case it is a`。
- **L2183 EN**: Comment explains nearby logic, intent, or metadata: `computed value that must be placed in memory (logicals are computed as`.
  **L2183 CN**: 注释说明附近代码的逻辑、意图或元数据：`computed value that must be placed in memory (logicals are computed as`。
- **L2184 EN**: Comment explains nearby logic, intent, or metadata: `i1, but must be placed in memory as fir.logical).`.
  **L2184 CN**: 注释说明附近代码的逻辑、意图或元数据：`i1, but must be placed in memory as fir.logical).`。

### Lines 2185-2208

````cpp
    auto getActualFortranElementType = [&]() -> mlir::Type {
      if (const Fortran::lower::SomeExpr *expr =
              callContext.procRef.UnwrapArgExpr(arg.index())) {

        mlir::Type type = converter.genType(*expr);
        return hlfir::getFortranElementType(type);
      }
      // TYPE(*): is already in memory anyway. Can return none
      // here.
      return builder.getNoneType();
    };
    // Ad-hoc argument lowering handling.
    fir::ArgLoweringRule argRules =
        fir::lowerIntrinsicArgumentAs(*argLowering, arg.index());
    if (arg.value()->handleDynamicOptional()) {
      mlir::Value isPresent = arg.value()->getIsPresent();
      switch (argRules.lowerAs) {
      case fir::LowerIntrinsicArgAs::Value: {
        // In case of elemental call, getActual() may produce
        // a designator denoting the array element to be passed
        // to the subprogram. If the actual array is dynamically
        // optional the designator must be generated under
        // isPresent check, because the box bounds reads will be
        // generated in the codegen. These reads are illegal,
````
- **L2185 EN**: Starts a function, method, lambda, or structured scope: `auto getActualFortranElementType = [&]() -> mlir::Type {`.
  **L2185 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto getActualFortranElementType = [&]() -> mlir::Type {`。
- **L2186 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2186 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2187 EN**: Starts a function, method, lambda, or structured scope: `callContext.procRef.UnwrapArgExpr(arg.index())) {`.
  **L2187 CN**: 开始一个函数、方法、lambda 或结构化作用域：`callContext.procRef.UnwrapArgExpr(arg.index())) {`。
- **L2188 EN**: Blank line separating nearby declarations or logic blocks.
  **L2188 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2189 EN**: Initializes variable `type` from the right-hand expression.
  **L2189 CN**: 使用右侧表达式初始化变量 `type`。
- **L2190 EN**: Returns from the current function with `hlfir::getFortranElementType(type)`.
  **L2190 CN**: 以 `hlfir::getFortranElementType(type)` 从当前函数返回。
- **L2191 EN**: Closes the current lexical scope or compound statement.
  **L2191 CN**: 结束当前词法作用域或复合语句块。
- **L2192 EN**: Comment explains nearby logic, intent, or metadata: `TYPE(*): is already in memory anyway. Can return none`.
  **L2192 CN**: 注释说明附近代码的逻辑、意图或元数据：`TYPE(*): is already in memory anyway. Can return none`。
- **L2193 EN**: Comment explains nearby logic, intent, or metadata: `here.`.
  **L2193 CN**: 注释说明附近代码的逻辑、意图或元数据：`here.`。
- **L2194 EN**: Returns from the current function with `builder.getNoneType()`.
  **L2194 CN**: 以 `builder.getNoneType()` 从当前函数返回。
- **L2195 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2195 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2196 EN**: Comment explains nearby logic, intent, or metadata: `Ad-hoc argument lowering handling.`.
  **L2196 CN**: 注释说明附近代码的逻辑、意图或元数据：`Ad-hoc argument lowering handling.`。
- **L2197 EN**: Continues the surrounding expression or declaration: `fir::ArgLoweringRule argRules =`.
  **L2197 CN**: 继续构造周围的表达式或声明：`fir::ArgLoweringRule argRules =`。
- **L2198 EN**: Executes a call or declaration centered on `fir::lowerIntrinsicArgumentAs`.
  **L2198 CN**: 执行以 `fir::lowerIntrinsicArgumentAs` 为核心的调用或声明。
- **L2199 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2199 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2200 EN**: Initializes variable `isPresent` from the right-hand expression.
  **L2200 CN**: 使用右侧表达式初始化变量 `isPresent`。
- **L2201 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2201 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2202 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Value: {`.
  **L2202 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Value: {`。
- **L2203 EN**: Comment explains nearby logic, intent, or metadata: `In case of elemental call, getActual() may produce`.
  **L2203 CN**: 注释说明附近代码的逻辑、意图或元数据：`In case of elemental call, getActual() may produce`。
- **L2204 EN**: Comment explains nearby logic, intent, or metadata: `a designator denoting the array element to be passed`.
  **L2204 CN**: 注释说明附近代码的逻辑、意图或元数据：`a designator denoting the array element to be passed`。
- **L2205 EN**: Comment explains nearby logic, intent, or metadata: `to the subprogram. If the actual array is dynamically`.
  **L2205 CN**: 注释说明附近代码的逻辑、意图或元数据：`to the subprogram. If the actual array is dynamically`。
- **L2206 EN**: Comment explains nearby logic, intent, or metadata: `optional the designator must be generated under`.
  **L2206 CN**: 注释说明附近代码的逻辑、意图或元数据：`optional the designator must be generated under`。
- **L2207 EN**: Comment explains nearby logic, intent, or metadata: `isPresent check, because the box bounds reads will be`.
  **L2207 CN**: 注释说明附近代码的逻辑、意图或元数据：`isPresent check, because the box bounds reads will be`。
- **L2208 EN**: Comment explains nearby logic, intent, or metadata: `generated in the codegen. These reads are illegal,`.
  **L2208 CN**: 注释说明附近代码的逻辑、意图或元数据：`generated in the codegen. These reads are illegal,`。

### Lines 2209-2232

````cpp
        // if the dynamically optional argument is absent.
        auto getActualCb = [&](mlir::Location loc,
                               fir::FirOpBuilder &builder) -> hlfir::Entity {
          return arg.value()->getActual(loc, builder);
        };
        auto [exv, cleanup] =
            genOptionalValue(builder, loc, getActualFortranElementType(),
                             getActualCb, isPresent);
        addToCleanups(std::move(cleanup));
        operands.emplace_back(exv);
        continue;
      }
      case fir::LowerIntrinsicArgAs::Addr: {
        hlfir::Entity actual = arg.value()->getActual(loc, builder);
        auto [exv, cleanup] = genOptionalAddr(builder, loc, actual, isPresent);
        addToCleanups(std::move(cleanup));
        operands.emplace_back(exv);
        continue;
      }
      case fir::LowerIntrinsicArgAs::Box: {
        hlfir::Entity actual = arg.value()->getActual(loc, builder);
        auto [exv, cleanup] = genOptionalBox(builder, loc, actual, isPresent);
        addToCleanups(std::move(cleanup));
        operands.emplace_back(exv);
````
- **L2209 EN**: Comment explains nearby logic, intent, or metadata: `if the dynamically optional argument is absent.`.
  **L2209 CN**: 注释说明附近代码的逻辑、意图或元数据：`if the dynamically optional argument is absent.`。
- **L2210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto getActualCb = [&](mlir::Location loc,`.
  **L2210 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto getActualCb = [&](mlir::Location loc,`。
- **L2211 EN**: Continues the surrounding expression or declaration: `fir::FirOpBuilder &builder) -> hlfir::Entity {`.
  **L2211 CN**: 继续构造周围的表达式或声明：`fir::FirOpBuilder &builder) -> hlfir::Entity {`。
- **L2212 EN**: Returns from the current function with `arg.value()->getActual(loc, builder)`.
  **L2212 CN**: 以 `arg.value()->getActual(loc, builder)` 从当前函数返回。
- **L2213 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2213 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2214 EN**: Continues the surrounding expression or declaration: `auto [exv, cleanup] =`.
  **L2214 CN**: 继续构造周围的表达式或声明：`auto [exv, cleanup] =`。
- **L2215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genOptionalValue(builder, loc, getActualFortranElementType(),`.
  **L2215 CN**: 继续一个多行参数列表、初始化器或聚合项：`genOptionalValue(builder, loc, getActualFortranElementType(),`。
- **L2216 EN**: Executes a standalone statement or declaration: `getActualCb, isPresent);`.
  **L2216 CN**: 执行一条独立语句或声明：`getActualCb, isPresent);`。
- **L2217 EN**: Executes a call or declaration centered on `addToCleanups`.
  **L2217 CN**: 执行以 `addToCleanups` 为核心的调用或声明。
- **L2218 EN**: Executes a call or declaration centered on `operands.emplace_back`.
  **L2218 CN**: 执行以 `operands.emplace_back` 为核心的调用或声明。
- **L2219 EN**: Skips to the next loop iteration.
  **L2219 CN**: 跳到下一次循环迭代。
- **L2220 EN**: Closes the current lexical scope or compound statement.
  **L2220 CN**: 结束当前词法作用域或复合语句块。
- **L2221 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Addr: {`.
  **L2221 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Addr: {`。
- **L2222 EN**: Initializes variable `actual` from the right-hand expression.
  **L2222 CN**: 使用右侧表达式初始化变量 `actual`。
- **L2223 EN**: Executes a call or declaration centered on `genOptionalAddr`.
  **L2223 CN**: 执行以 `genOptionalAddr` 为核心的调用或声明。
- **L2224 EN**: Executes a call or declaration centered on `addToCleanups`.
  **L2224 CN**: 执行以 `addToCleanups` 为核心的调用或声明。
- **L2225 EN**: Executes a call or declaration centered on `operands.emplace_back`.
  **L2225 CN**: 执行以 `operands.emplace_back` 为核心的调用或声明。
- **L2226 EN**: Skips to the next loop iteration.
  **L2226 CN**: 跳到下一次循环迭代。
- **L2227 EN**: Closes the current lexical scope or compound statement.
  **L2227 CN**: 结束当前词法作用域或复合语句块。
- **L2228 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Box: {`.
  **L2228 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Box: {`。
- **L2229 EN**: Initializes variable `actual` from the right-hand expression.
  **L2229 CN**: 使用右侧表达式初始化变量 `actual`。
- **L2230 EN**: Executes a call or declaration centered on `genOptionalBox`.
  **L2230 CN**: 执行以 `genOptionalBox` 为核心的调用或声明。
- **L2231 EN**: Executes a call or declaration centered on `addToCleanups`.
  **L2231 CN**: 执行以 `addToCleanups` 为核心的调用或声明。
- **L2232 EN**: Executes a call or declaration centered on `operands.emplace_back`.
  **L2232 CN**: 执行以 `operands.emplace_back` 为核心的调用或声明。

### Lines 2233-2256

````cpp
        continue;
      }
      case fir::LowerIntrinsicArgAs::Inquired: {
        hlfir::Entity actual = arg.value()->getActual(loc, builder);
        auto [exv, cleanup] =
            hlfir::translateToExtendedValue(loc, builder, actual);
        addToCleanups(std::move(cleanup));
        operands.emplace_back(exv);
        continue;
      }
      }
      llvm_unreachable("bad switch");
    }

    hlfir::Entity actual = arg.value()->getActual(loc, builder);
    switch (argRules.lowerAs) {
    case fir::LowerIntrinsicArgAs::Value:
      operands.emplace_back(
          Fortran::lower::convertToValue(loc, converter, actual, stmtCtx));
      continue;
    case fir::LowerIntrinsicArgAs::Addr:
      operands.emplace_back(Fortran::lower::convertToAddress(
          loc, converter, actual, stmtCtx, getActualFortranElementType()));
      continue;
````
- **L2233 EN**: Skips to the next loop iteration.
  **L2233 CN**: 跳到下一次循环迭代。
- **L2234 EN**: Closes the current lexical scope or compound statement.
  **L2234 CN**: 结束当前词法作用域或复合语句块。
- **L2235 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Inquired: {`.
  **L2235 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Inquired: {`。
- **L2236 EN**: Initializes variable `actual` from the right-hand expression.
  **L2236 CN**: 使用右侧表达式初始化变量 `actual`。
- **L2237 EN**: Continues the surrounding expression or declaration: `auto [exv, cleanup] =`.
  **L2237 CN**: 继续构造周围的表达式或声明：`auto [exv, cleanup] =`。
- **L2238 EN**: Executes a call or declaration centered on `hlfir::translateToExtendedValue`.
  **L2238 CN**: 执行以 `hlfir::translateToExtendedValue` 为核心的调用或声明。
- **L2239 EN**: Executes a call or declaration centered on `addToCleanups`.
  **L2239 CN**: 执行以 `addToCleanups` 为核心的调用或声明。
- **L2240 EN**: Executes a call or declaration centered on `operands.emplace_back`.
  **L2240 CN**: 执行以 `operands.emplace_back` 为核心的调用或声明。
- **L2241 EN**: Skips to the next loop iteration.
  **L2241 CN**: 跳到下一次循环迭代。
- **L2242 EN**: Closes the current lexical scope or compound statement.
  **L2242 CN**: 结束当前词法作用域或复合语句块。
- **L2243 EN**: Closes the current lexical scope or compound statement.
  **L2243 CN**: 结束当前词法作用域或复合语句块。
- **L2244 EN**: Marks this control path as unreachable to LLVM.
  **L2244 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2245 EN**: Closes the current lexical scope or compound statement.
  **L2245 CN**: 结束当前词法作用域或复合语句块。
- **L2246 EN**: Blank line separating nearby declarations or logic blocks.
  **L2246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2247 EN**: Initializes variable `actual` from the right-hand expression.
  **L2247 CN**: 使用右侧表达式初始化变量 `actual`。
- **L2248 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2248 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2249 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Value:`.
  **L2249 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Value:`。
- **L2250 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L2250 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L2251 EN**: Executes a call or declaration centered on `Fortran::lower::convertToValue`.
  **L2251 CN**: 执行以 `Fortran::lower::convertToValue` 为核心的调用或声明。
- **L2252 EN**: Skips to the next loop iteration.
  **L2252 CN**: 跳到下一次循环迭代。
- **L2253 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Addr:`.
  **L2253 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Addr:`。
- **L2254 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L2254 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L2255 EN**: Executes a call or declaration centered on `getActualFortranElementType`.
  **L2255 CN**: 执行以 `getActualFortranElementType` 为核心的调用或声明。
- **L2256 EN**: Skips to the next loop iteration.
  **L2256 CN**: 跳到下一次循环迭代。

### Lines 2257-2280

````cpp
    case fir::LowerIntrinsicArgAs::Box:
      operands.emplace_back(Fortran::lower::convertToBox(
          loc, converter, actual, stmtCtx, getActualFortranElementType()));
      continue;
    case fir::LowerIntrinsicArgAs::Inquired:
      if (const Fortran::lower::SomeExpr *expr =
              callContext.procRef.UnwrapArgExpr(arg.index())) {
        if (Fortran::evaluate::UnwrapExpr<Fortran::evaluate::NullPointer>(
                *expr)) {
          // NULL() pointer without a MOLD must be passed as a deallocated
          // pointer (see table 16.5 in Fortran 2018 standard).
          // !fir.box<!fir.ptr<none>> should always be valid in this context.
          mlir::Type noneTy = mlir::NoneType::get(builder.getContext());
          mlir::Type nullPtrTy = fir::PointerType::get(noneTy);
          mlir::Type boxTy = fir::BoxType::get(nullPtrTy);
          mlir::Value boxStorage =
              fir::factory::genNullBoxStorage(builder, loc, boxTy);
          hlfir::EntityWithAttributes nullBoxEntity =
              extendedValueToHlfirEntity(loc, builder, boxStorage,
                                         ".tmp.null_box");
          operands.emplace_back(Fortran::lower::translateToExtendedValue(
              loc, builder, nullBoxEntity, stmtCtx));
          continue;
        }
````
- **L2257 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Box:`.
  **L2257 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Box:`。
- **L2258 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L2258 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L2259 EN**: Executes a call or declaration centered on `getActualFortranElementType`.
  **L2259 CN**: 执行以 `getActualFortranElementType` 为核心的调用或声明。
- **L2260 EN**: Skips to the next loop iteration.
  **L2260 CN**: 跳到下一次循环迭代。
- **L2261 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Inquired:`.
  **L2261 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Inquired:`。
- **L2262 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2262 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2263 EN**: Starts a function, method, lambda, or structured scope: `callContext.procRef.UnwrapArgExpr(arg.index())) {`.
  **L2263 CN**: 开始一个函数、方法、lambda 或结构化作用域：`callContext.procRef.UnwrapArgExpr(arg.index())) {`。
- **L2264 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2264 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2265 EN**: Comment explains nearby logic, intent, or metadata: `expr)) {`.
  **L2265 CN**: 注释说明附近代码的逻辑、意图或元数据：`expr)) {`。
- **L2266 EN**: Comment explains nearby logic, intent, or metadata: `NULL() pointer without a MOLD must be passed as a deallocated`.
  **L2266 CN**: 注释说明附近代码的逻辑、意图或元数据：`NULL() pointer without a MOLD must be passed as a deallocated`。
- **L2267 EN**: Comment explains nearby logic, intent, or metadata: `pointer (see table 16.5 in Fortran 2018 standard).`.
  **L2267 CN**: 注释说明附近代码的逻辑、意图或元数据：`pointer (see table 16.5 in Fortran 2018 standard).`。
- **L2268 EN**: Comment explains nearby logic, intent, or metadata: `fir.box<!fir.ptr<none>> should always be valid in this context.`.
  **L2268 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.box<!fir.ptr<none>> should always be valid in this context.`。
- **L2269 EN**: Initializes variable `noneTy` from the right-hand expression.
  **L2269 CN**: 使用右侧表达式初始化变量 `noneTy`。
- **L2270 EN**: Initializes variable `nullPtrTy` from the right-hand expression.
  **L2270 CN**: 使用右侧表达式初始化变量 `nullPtrTy`。
- **L2271 EN**: Initializes variable `boxTy` from the right-hand expression.
  **L2271 CN**: 使用右侧表达式初始化变量 `boxTy`。
- **L2272 EN**: Continues the surrounding expression or declaration: `mlir::Value boxStorage =`.
  **L2272 CN**: 继续构造周围的表达式或声明：`mlir::Value boxStorage =`。
- **L2273 EN**: Executes a call or declaration centered on `fir::factory::genNullBoxStorage`.
  **L2273 CN**: 执行以 `fir::factory::genNullBoxStorage` 为核心的调用或声明。
- **L2274 EN**: Continues the surrounding expression or declaration: `hlfir::EntityWithAttributes nullBoxEntity =`.
  **L2274 CN**: 继续构造周围的表达式或声明：`hlfir::EntityWithAttributes nullBoxEntity =`。
- **L2275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `extendedValueToHlfirEntity(loc, builder, boxStorage,`.
  **L2275 CN**: 继续一个多行参数列表、初始化器或聚合项：`extendedValueToHlfirEntity(loc, builder, boxStorage,`。
- **L2276 EN**: Executes a standalone statement or declaration: `".tmp.null_box");`.
  **L2276 CN**: 执行一条独立语句或声明：`".tmp.null_box");`。
- **L2277 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L2277 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L2278 EN**: Executes a standalone statement or declaration: `loc, builder, nullBoxEntity, stmtCtx));`.
  **L2278 CN**: 执行一条独立语句或声明：`loc, builder, nullBoxEntity, stmtCtx));`。
- **L2279 EN**: Skips to the next loop iteration.
  **L2279 CN**: 跳到下一次循环迭代。
- **L2280 EN**: Closes the current lexical scope or compound statement.
  **L2280 CN**: 结束当前词法作用域或复合语句块。

### Lines 2281-2304

````cpp
      }
      // Place hlfir.expr in memory, and unbox fir.boxchar. Other entities
      // are translated to fir::ExtendedValue without transformation (notably,
      // pointers/allocatable are not dereferenced).
      // TODO: once lowering to FIR retires, UBOUND and LBOUND can be simplified
      // since the fir.box lowered here are now guaranteed to contain the local
      // lower bounds thanks to the hlfir.declare (the extra rebox can be
      // removed).
      operands.emplace_back(Fortran::lower::translateToExtendedValue(
          loc, builder, actual, stmtCtx));
      continue;
    }
    llvm_unreachable("bad switch");
  }
  // genIntrinsicCall needs the scalar type, even if this is a transformational
  // procedure returning an array.
  std::optional<mlir::Type> scalarResultType;
  if (callContext.resultType)
    scalarResultType = hlfir::getFortranElementType(*callContext.resultType);
  const std::string intrinsicName = callContext.getProcedureName();
  // Let the intrinsic library lower the intrinsic procedure call.
  auto [resultExv, mustBeFreed] = genIntrinsicCall(
      builder, loc, intrinsicEntry, scalarResultType, operands, &converter);
  for (const hlfir::CleanupFunction &fn : cleanupFns)
````
- **L2281 EN**: Closes the current lexical scope or compound statement.
  **L2281 CN**: 结束当前词法作用域或复合语句块。
- **L2282 EN**: Comment explains nearby logic, intent, or metadata: `Place hlfir.expr in memory, and unbox fir.boxchar. Other entities`.
  **L2282 CN**: 注释说明附近代码的逻辑、意图或元数据：`Place hlfir.expr in memory, and unbox fir.boxchar. Other entities`。
- **L2283 EN**: Comment explains nearby logic, intent, or metadata: `are translated to fir::ExtendedValue without transformation (notably,`.
  **L2283 CN**: 注释说明附近代码的逻辑、意图或元数据：`are translated to fir::ExtendedValue without transformation (notably,`。
- **L2284 EN**: Comment explains nearby logic, intent, or metadata: `pointers/allocatable are not dereferenced).`.
  **L2284 CN**: 注释说明附近代码的逻辑、意图或元数据：`pointers/allocatable are not dereferenced).`。
- **L2285 EN**: Comment records a pending task or caution: `TODO: once lowering to FIR retires, UBOUND and LBOUND can be simplified`.
  **L2285 CN**: 注释记录待办事项或注意点：`TODO: once lowering to FIR retires, UBOUND and LBOUND can be simplified`。
- **L2286 EN**: Comment explains nearby logic, intent, or metadata: `since the fir.box lowered here are now guaranteed to contain the local`.
  **L2286 CN**: 注释说明附近代码的逻辑、意图或元数据：`since the fir.box lowered here are now guaranteed to contain the local`。
- **L2287 EN**: Comment explains nearby logic, intent, or metadata: `lower bounds thanks to the hlfir.declare (the extra rebox can be`.
  **L2287 CN**: 注释说明附近代码的逻辑、意图或元数据：`lower bounds thanks to the hlfir.declare (the extra rebox can be`。
- **L2288 EN**: Comment explains nearby logic, intent, or metadata: `removed).`.
  **L2288 CN**: 注释说明附近代码的逻辑、意图或元数据：`removed).`。
- **L2289 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L2289 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L2290 EN**: Executes a standalone statement or declaration: `loc, builder, actual, stmtCtx));`.
  **L2290 CN**: 执行一条独立语句或声明：`loc, builder, actual, stmtCtx));`。
- **L2291 EN**: Skips to the next loop iteration.
  **L2291 CN**: 跳到下一次循环迭代。
- **L2292 EN**: Closes the current lexical scope or compound statement.
  **L2292 CN**: 结束当前词法作用域或复合语句块。
- **L2293 EN**: Marks this control path as unreachable to LLVM.
  **L2293 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2294 EN**: Closes the current lexical scope or compound statement.
  **L2294 CN**: 结束当前词法作用域或复合语句块。
- **L2295 EN**: Comment explains nearby logic, intent, or metadata: `genIntrinsicCall needs the scalar type, even if this is a transformational`.
  **L2295 CN**: 注释说明附近代码的逻辑、意图或元数据：`genIntrinsicCall needs the scalar type, even if this is a transformational`。
- **L2296 EN**: Comment explains nearby logic, intent, or metadata: `procedure returning an array.`.
  **L2296 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure returning an array.`。
- **L2297 EN**: Executes a standalone statement or declaration: `std::optional<mlir::Type> scalarResultType;`.
  **L2297 CN**: 执行一条独立语句或声明：`std::optional<mlir::Type> scalarResultType;`。
- **L2298 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2298 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2299 EN**: Executes a call or declaration centered on `hlfir::getFortranElementType`.
  **L2299 CN**: 执行以 `hlfir::getFortranElementType` 为核心的调用或声明。
- **L2300 EN**: Initializes variable `intrinsicName` from the right-hand expression.
  **L2300 CN**: 使用右侧表达式初始化变量 `intrinsicName`。
- **L2301 EN**: Comment explains nearby logic, intent, or metadata: `Let the intrinsic library lower the intrinsic procedure call.`.
  **L2301 CN**: 注释说明附近代码的逻辑、意图或元数据：`Let the intrinsic library lower the intrinsic procedure call.`。
- **L2302 EN**: Continues logic associated with callable symbol `genIntrinsicCall`.
  **L2302 CN**: 继续与可调用符号 `genIntrinsicCall` 相关的逻辑。
- **L2303 EN**: Executes a standalone statement or declaration: `builder, loc, intrinsicEntry, scalarResultType, operands, &converter);`.
  **L2303 CN**: 执行一条独立语句或声明：`builder, loc, intrinsicEntry, scalarResultType, operands, &converter);`。
- **L2304 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2304 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 2305-2328

````cpp
    fn();
  if (!fir::getBase(resultExv))
    return std::nullopt;
  hlfir::EntityWithAttributes resultEntity = extendedValueToHlfirEntity(
      loc, builder, resultExv, ".tmp.intrinsic_result");
  // Move result into memory into an hlfir.expr since they are immutable from
  // that point, and the result storage is some temp. "Null" is special: it
  // returns a null pointer variable that should not be transformed into a value
  // (what matters is the memory address).
  if (resultEntity.isVariable() && intrinsicName != "null") {
    assert(!fir::isa_trivial(fir::unwrapRefType(resultEntity.getType())) &&
           "expect intrinsic scalar results to not be in memory");
    hlfir::AsExprOp asExpr;
    // Character/Derived MERGE lowering returns one of its argument address
    // (this is the only intrinsic implemented in that way so far). The
    // ownership of this address cannot be taken here since it may not be a
    // temp.
    if (intrinsicName == "merge")
      asExpr = hlfir::AsExprOp::create(builder, loc, resultEntity);
    else
      asExpr = hlfir::AsExprOp::create(builder, loc, resultEntity,
                                       builder.createBool(loc, mustBeFreed));
    resultEntity = hlfir::EntityWithAttributes{asExpr.getResult()};
  }
````
- **L2305 EN**: Executes a call or declaration centered on `fn`.
  **L2305 CN**: 执行以 `fn` 为核心的调用或声明。
- **L2306 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2306 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2307 EN**: Returns from the current function with `std::nullopt`.
  **L2307 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2308 EN**: Continues logic associated with callable symbol `extendedValueToHlfirEntity`.
  **L2308 CN**: 继续与可调用符号 `extendedValueToHlfirEntity` 相关的逻辑。
- **L2309 EN**: Executes a standalone statement or declaration: `loc, builder, resultExv, ".tmp.intrinsic_result");`.
  **L2309 CN**: 执行一条独立语句或声明：`loc, builder, resultExv, ".tmp.intrinsic_result");`。
- **L2310 EN**: Comment explains nearby logic, intent, or metadata: `Move result into memory into an hlfir.expr since they are immutable from`.
  **L2310 CN**: 注释说明附近代码的逻辑、意图或元数据：`Move result into memory into an hlfir.expr since they are immutable from`。
- **L2311 EN**: Comment explains nearby logic, intent, or metadata: `that point, and the result storage is some temp. "Null" is special: it`.
  **L2311 CN**: 注释说明附近代码的逻辑、意图或元数据：`that point, and the result storage is some temp. "Null" is special: it`。
- **L2312 EN**: Comment explains nearby logic, intent, or metadata: `returns a null pointer variable that should not be transformed into a value`.
  **L2312 CN**: 注释说明附近代码的逻辑、意图或元数据：`returns a null pointer variable that should not be transformed into a value`。
- **L2313 EN**: Comment explains nearby logic, intent, or metadata: `(what matters is the memory address).`.
  **L2313 CN**: 注释说明附近代码的逻辑、意图或元数据：`(what matters is the memory address).`。
- **L2314 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2314 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2315 EN**: Checks an internal invariant in debug builds.
  **L2315 CN**: 在调试构建中检查内部不变式。
- **L2316 EN**: Executes a standalone statement or declaration: `"expect intrinsic scalar results to not be in memory");`.
  **L2316 CN**: 执行一条独立语句或声明：`"expect intrinsic scalar results to not be in memory");`。
- **L2317 EN**: Executes a standalone statement or declaration: `hlfir::AsExprOp asExpr;`.
  **L2317 CN**: 执行一条独立语句或声明：`hlfir::AsExprOp asExpr;`。
- **L2318 EN**: Comment explains nearby logic, intent, or metadata: `Character/Derived MERGE lowering returns one of its argument address`.
  **L2318 CN**: 注释说明附近代码的逻辑、意图或元数据：`Character/Derived MERGE lowering returns one of its argument address`。
- **L2319 EN**: Comment explains nearby logic, intent, or metadata: `(this is the only intrinsic implemented in that way so far). The`.
  **L2319 CN**: 注释说明附近代码的逻辑、意图或元数据：`(this is the only intrinsic implemented in that way so far). The`。
- **L2320 EN**: Comment explains nearby logic, intent, or metadata: `ownership of this address cannot be taken here since it may not be a`.
  **L2320 CN**: 注释说明附近代码的逻辑、意图或元数据：`ownership of this address cannot be taken here since it may not be a`。
- **L2321 EN**: Comment explains nearby logic, intent, or metadata: `temp.`.
  **L2321 CN**: 注释说明附近代码的逻辑、意图或元数据：`temp.`。
- **L2322 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2322 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2323 EN**: Executes a call or declaration centered on `hlfir::AsExprOp::create`.
  **L2323 CN**: 执行以 `hlfir::AsExprOp::create` 为核心的调用或声明。
- **L2324 EN**: Transitions from the previous branch into the alternative path.
  **L2324 CN**: 从前一个分支过渡到备选路径。
- **L2325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `asExpr = hlfir::AsExprOp::create(builder, loc, resultEntity,`.
  **L2325 CN**: 继续一个多行参数列表、初始化器或聚合项：`asExpr = hlfir::AsExprOp::create(builder, loc, resultEntity,`。
- **L2326 EN**: Executes a call or declaration centered on `builder.createBool`.
  **L2326 CN**: 执行以 `builder.createBool` 为核心的调用或声明。
- **L2327 EN**: Executes a call or declaration centered on `hlfir::EntityWithAttributes{asExpr.getResult`.
  **L2327 CN**: 执行以 `hlfir::EntityWithAttributes{asExpr.getResult` 为核心的调用或声明。
- **L2328 EN**: Closes the current lexical scope or compound statement.
  **L2328 CN**: 结束当前词法作用域或复合语句块。

### Lines 2329-2352

````cpp
  return resultEntity;
}

/// Lower calls to intrinsic procedures with actual arguments that have been
/// pre-lowered but have not yet been prepared according to the interface.
static std::optional<hlfir::EntityWithAttributes> genHLFIRIntrinsicRefCore(
    Fortran::lower::PreparedActualArguments &loweredActuals,
    const Fortran::evaluate::SpecificIntrinsic *intrinsic,
    const fir::IntrinsicHandlerEntry &intrinsicEntry,
    CallContext &callContext) {
  // Delegate intrinsics with custom optional handling to
  // genCustomIntrinsicRefCore before attempting any HLFIR op lowering. This
  // ensures consistent dispatch symmetry with genIntrinsicRefCore and
  // genIntrinsicRef, both of which check for custom optional handling before
  // reaching the HLFIR intrinsic path.
  if (intrinsic && Fortran::lower::intrinsicRequiresCustomOptionalHandling(
                       callContext.procRef, *intrinsic, callContext.converter))
    return genCustomIntrinsicRefCore(loweredActuals, intrinsic, callContext);
  // Try lowering transformational intrinsic ops to HLFIR ops if enabled
  // (transformational always have a result type)
  if (useHlfirIntrinsicOps && callContext.resultType) {
    fir::FirOpBuilder &builder = callContext.getBuilder();
    mlir::Location loc = callContext.loc;
    const std::string intrinsicName = callContext.getProcedureName();
````
- **L2329 EN**: Returns from the current function with `resultEntity`.
  **L2329 CN**: 以 `resultEntity` 从当前函数返回。
- **L2330 EN**: Closes the current lexical scope or compound statement.
  **L2330 CN**: 结束当前词法作用域或复合语句块。
- **L2331 EN**: Blank line separating nearby declarations or logic blocks.
  **L2331 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2332 EN**: Comment explains nearby logic, intent, or metadata: `Lower calls to intrinsic procedures with actual arguments that have been`.
  **L2332 CN**: 注释说明附近代码的逻辑、意图或元数据：`Lower calls to intrinsic procedures with actual arguments that have been`。
- **L2333 EN**: Comment explains nearby logic, intent, or metadata: `pre-lowered but have not yet been prepared according to the interface.`.
  **L2333 CN**: 注释说明附近代码的逻辑、意图或元数据：`pre-lowered but have not yet been prepared according to the interface.`。
- **L2334 EN**: Continues logic associated with callable symbol `genHLFIRIntrinsicRefCore`.
  **L2334 CN**: 继续与可调用符号 `genHLFIRIntrinsicRefCore` 相关的逻辑。
- **L2335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L2335 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L2336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::SpecificIntrinsic *intrinsic,`.
  **L2336 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::SpecificIntrinsic *intrinsic,`。
- **L2337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicHandlerEntry &intrinsicEntry,`.
  **L2337 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicHandlerEntry &intrinsicEntry,`。
- **L2338 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L2338 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L2339 EN**: Comment explains nearby logic, intent, or metadata: `Delegate intrinsics with custom optional handling to`.
  **L2339 CN**: 注释说明附近代码的逻辑、意图或元数据：`Delegate intrinsics with custom optional handling to`。
- **L2340 EN**: Comment explains nearby logic, intent, or metadata: `genCustomIntrinsicRefCore before attempting any HLFIR op lowering. This`.
  **L2340 CN**: 注释说明附近代码的逻辑、意图或元数据：`genCustomIntrinsicRefCore before attempting any HLFIR op lowering. This`。
- **L2341 EN**: Comment explains nearby logic, intent, or metadata: `ensures consistent dispatch symmetry with genIntrinsicRefCore and`.
  **L2341 CN**: 注释说明附近代码的逻辑、意图或元数据：`ensures consistent dispatch symmetry with genIntrinsicRefCore and`。
- **L2342 EN**: Comment explains nearby logic, intent, or metadata: `genIntrinsicRef, both of which check for custom optional handling before`.
  **L2342 CN**: 注释说明附近代码的逻辑、意图或元数据：`genIntrinsicRef, both of which check for custom optional handling before`。
- **L2343 EN**: Comment explains nearby logic, intent, or metadata: `reaching the HLFIR intrinsic path.`.
  **L2343 CN**: 注释说明附近代码的逻辑、意图或元数据：`reaching the HLFIR intrinsic path.`。
- **L2344 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2344 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2345 EN**: Continues the surrounding expression or declaration: `callContext.procRef, *intrinsic, callContext.converter))`.
  **L2345 CN**: 继续构造周围的表达式或声明：`callContext.procRef, *intrinsic, callContext.converter))`。
- **L2346 EN**: Returns from the current function with `genCustomIntrinsicRefCore(loweredActuals, intrinsic, callContext)`.
  **L2346 CN**: 以 `genCustomIntrinsicRefCore(loweredActuals, intrinsic, callContext)` 从当前函数返回。
- **L2347 EN**: Comment explains nearby logic, intent, or metadata: `Try lowering transformational intrinsic ops to HLFIR ops if enabled`.
  **L2347 CN**: 注释说明附近代码的逻辑、意图或元数据：`Try lowering transformational intrinsic ops to HLFIR ops if enabled`。
- **L2348 EN**: Comment explains nearby logic, intent, or metadata: `(transformational always have a result type)`.
  **L2348 CN**: 注释说明附近代码的逻辑、意图或元数据：`(transformational always have a result type)`。
- **L2349 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2349 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2350 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L2350 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L2351 EN**: Initializes variable `loc` from the right-hand expression.
  **L2351 CN**: 使用右侧表达式初始化变量 `loc`。
- **L2352 EN**: Initializes variable `intrinsicName` from the right-hand expression.
  **L2352 CN**: 使用右侧表达式初始化变量 `intrinsicName`。

### Lines 2353-2376

````cpp
    const fir::IntrinsicArgumentLoweringRules *argLowering =
        intrinsicEntry.getArgumentLoweringRules();
    mlir::Type resultType =
        callContext.isElementalProcWithArrayArgs()
            ? hlfir::getFortranElementType(*callContext.resultType)
            : *callContext.resultType;

    std::optional<hlfir::EntityWithAttributes> res =
        Fortran::lower::lowerHlfirIntrinsic(builder, loc, intrinsicName,
                                            loweredActuals, argLowering,
                                            resultType);
    if (res)
      return res;
  }

  // fallback to calling the intrinsic via fir.call
  return genIntrinsicRefCore(loweredActuals, intrinsic, intrinsicEntry,
                             callContext);
}

namespace {
template <typename ElementalCallBuilderImpl>
class ElementalCallBuilder {
public:
````
- **L2353 EN**: Continues the surrounding expression or declaration: `const fir::IntrinsicArgumentLoweringRules *argLowering =`.
  **L2353 CN**: 继续构造周围的表达式或声明：`const fir::IntrinsicArgumentLoweringRules *argLowering =`。
- **L2354 EN**: Executes a call or declaration centered on `intrinsicEntry.getArgumentLoweringRules`.
  **L2354 CN**: 执行以 `intrinsicEntry.getArgumentLoweringRules` 为核心的调用或声明。
- **L2355 EN**: Continues the surrounding expression or declaration: `mlir::Type resultType =`.
  **L2355 CN**: 继续构造周围的表达式或声明：`mlir::Type resultType =`。
- **L2356 EN**: Continues logic associated with callable symbol `isElementalProcWithArrayArgs`.
  **L2356 CN**: 继续与可调用符号 `isElementalProcWithArrayArgs` 相关的逻辑。
- **L2357 EN**: Continues logic associated with callable symbol `getFortranElementType`.
  **L2357 CN**: 继续与可调用符号 `getFortranElementType` 相关的逻辑。
- **L2358 EN**: Executes a standalone statement or declaration: `: *callContext.resultType;`.
  **L2358 CN**: 执行一条独立语句或声明：`: *callContext.resultType;`。
- **L2359 EN**: Blank line separating nearby declarations or logic blocks.
  **L2359 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2360 EN**: Continues the surrounding expression or declaration: `std::optional<hlfir::EntityWithAttributes> res =`.
  **L2360 CN**: 继续构造周围的表达式或声明：`std::optional<hlfir::EntityWithAttributes> res =`。
- **L2361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::lowerHlfirIntrinsic(builder, loc, intrinsicName,`.
  **L2361 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::lowerHlfirIntrinsic(builder, loc, intrinsicName,`。
- **L2362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loweredActuals, argLowering,`.
  **L2362 CN**: 继续一个多行参数列表、初始化器或聚合项：`loweredActuals, argLowering,`。
- **L2363 EN**: Executes a standalone statement or declaration: `resultType);`.
  **L2363 CN**: 执行一条独立语句或声明：`resultType);`。
- **L2364 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2364 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2365 EN**: Returns from the current function with `res`.
  **L2365 CN**: 以 `res` 从当前函数返回。
- **L2366 EN**: Closes the current lexical scope or compound statement.
  **L2366 CN**: 结束当前词法作用域或复合语句块。
- **L2367 EN**: Blank line separating nearby declarations or logic blocks.
  **L2367 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2368 EN**: Comment explains nearby logic, intent, or metadata: `fallback to calling the intrinsic via fir.call`.
  **L2368 CN**: 注释说明附近代码的逻辑、意图或元数据：`fallback to calling the intrinsic via fir.call`。
- **L2369 EN**: Returns from the current function with `genIntrinsicRefCore(loweredActuals, intrinsic, intrinsicEntry,`.
  **L2369 CN**: 以 `genIntrinsicRefCore(loweredActuals, intrinsic, intrinsicEntry,` 从当前函数返回。
- **L2370 EN**: Executes a standalone statement or declaration: `callContext);`.
  **L2370 CN**: 执行一条独立语句或声明：`callContext);`。
- **L2371 EN**: Closes the current lexical scope or compound statement.
  **L2371 CN**: 结束当前词法作用域或复合语句块。
- **L2372 EN**: Blank line separating nearby declarations or logic blocks.
  **L2372 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2373 EN**: Opens namespace scope ``.
  **L2373 CN**: 打开命名空间作用域 ``。
- **L2374 EN**: Introduces template parameters or specialization context: `template <typename ElementalCallBuilderImpl>`.
  **L2374 CN**: 为后续声明引入模板参数或特化上下文：`template <typename ElementalCallBuilderImpl>`。
- **L2375 EN**: Declares class `ElementalCallBuilder`.
  **L2375 CN**: 声明 class `ElementalCallBuilder`。
- **L2376 EN**: Sets the following members to `public` access.
  **L2376 CN**: 将后续成员的访问级别设为 `public`。

### Lines 2377-2400

````cpp
  std::optional<hlfir::EntityWithAttributes>
  genElementalCall(Fortran::lower::PreparedActualArguments &loweredActuals,
                   bool isImpure, CallContext &callContext) {
    mlir::Location loc = callContext.loc;
    fir::FirOpBuilder &builder = callContext.getBuilder();
    unsigned numArgs = loweredActuals.size();
    // Step 1: dereference pointers/allocatables and compute elemental shape.
    mlir::Value shape;
    Fortran::lower::PreparedActualArgument *optionalWithShape;
    // 10.1.4 p5. Impure elemental procedures must be called in element order.
    bool mustBeOrdered = isImpure;
    for (unsigned i = 0; i < numArgs; ++i) {
      auto &preparedActual = loweredActuals[i];
      if (preparedActual) {
        // Elemental procedure dummy arguments cannot be pointer/allocatables
        // (C15100), so it is safe to dereference any pointer or allocatable
        // actual argument now instead of doing this inside the elemental
        // region.
        preparedActual->derefPointersAndAllocatables(loc, builder);
        // Better to load scalars outside of the loop when possible.
        if (!preparedActual->handleDynamicOptional() &&
            impl().canLoadActualArgumentBeforeLoop(i))
          preparedActual->loadTrivialScalar(loc, builder);
        // TODO: merge shape instead of using the first one.
````
- **L2377 EN**: Continues the surrounding expression or declaration: `std::optional<hlfir::EntityWithAttributes>`.
  **L2377 CN**: 继续构造周围的表达式或声明：`std::optional<hlfir::EntityWithAttributes>`。
- **L2378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genElementalCall(Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L2378 CN**: 继续一个多行参数列表、初始化器或聚合项：`genElementalCall(Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L2379 EN**: Continues the surrounding expression or declaration: `bool isImpure, CallContext &callContext) {`.
  **L2379 CN**: 继续构造周围的表达式或声明：`bool isImpure, CallContext &callContext) {`。
- **L2380 EN**: Initializes variable `loc` from the right-hand expression.
  **L2380 CN**: 使用右侧表达式初始化变量 `loc`。
- **L2381 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L2381 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L2382 EN**: Initializes variable `numArgs` from the right-hand expression.
  **L2382 CN**: 使用右侧表达式初始化变量 `numArgs`。
- **L2383 EN**: Comment explains nearby logic, intent, or metadata: `Step 1: dereference pointers/allocatables and compute elemental shape.`.
  **L2383 CN**: 注释说明附近代码的逻辑、意图或元数据：`Step 1: dereference pointers/allocatables and compute elemental shape.`。
- **L2384 EN**: Executes a standalone statement or declaration: `mlir::Value shape;`.
  **L2384 CN**: 执行一条独立语句或声明：`mlir::Value shape;`。
- **L2385 EN**: Executes a standalone statement or declaration: `Fortran::lower::PreparedActualArgument *optionalWithShape;`.
  **L2385 CN**: 执行一条独立语句或声明：`Fortran::lower::PreparedActualArgument *optionalWithShape;`。
- **L2386 EN**: Comment explains nearby logic, intent, or metadata: `10.1.4 p5. Impure elemental procedures must be called in element order.`.
  **L2386 CN**: 注释说明附近代码的逻辑、意图或元数据：`10.1.4 p5. Impure elemental procedures must be called in element order.`。
- **L2387 EN**: Initializes variable `mustBeOrdered` from the right-hand expression.
  **L2387 CN**: 使用右侧表达式初始化变量 `mustBeOrdered`。
- **L2388 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2388 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2389 EN**: Executes a standalone statement or declaration: `auto &preparedActual = loweredActuals[i];`.
  **L2389 CN**: 执行一条独立语句或声明：`auto &preparedActual = loweredActuals[i];`。
- **L2390 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2390 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2391 EN**: Comment explains nearby logic, intent, or metadata: `Elemental procedure dummy arguments cannot be pointer/allocatables`.
  **L2391 CN**: 注释说明附近代码的逻辑、意图或元数据：`Elemental procedure dummy arguments cannot be pointer/allocatables`。
- **L2392 EN**: Comment explains nearby logic, intent, or metadata: `(C15100), so it is safe to dereference any pointer or allocatable`.
  **L2392 CN**: 注释说明附近代码的逻辑、意图或元数据：`(C15100), so it is safe to dereference any pointer or allocatable`。
- **L2393 EN**: Comment explains nearby logic, intent, or metadata: `actual argument now instead of doing this inside the elemental`.
  **L2393 CN**: 注释说明附近代码的逻辑、意图或元数据：`actual argument now instead of doing this inside the elemental`。
- **L2394 EN**: Comment explains nearby logic, intent, or metadata: `region.`.
  **L2394 CN**: 注释说明附近代码的逻辑、意图或元数据：`region.`。
- **L2395 EN**: Executes a call or declaration centered on `preparedActual->derefPointersAndAllocatables`.
  **L2395 CN**: 执行以 `preparedActual->derefPointersAndAllocatables` 为核心的调用或声明。
- **L2396 EN**: Comment explains nearby logic, intent, or metadata: `Better to load scalars outside of the loop when possible.`.
  **L2396 CN**: 注释说明附近代码的逻辑、意图或元数据：`Better to load scalars outside of the loop when possible.`。
- **L2397 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2397 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2398 EN**: Continues logic associated with callable symbol `impl`.
  **L2398 CN**: 继续与可调用符号 `impl` 相关的逻辑。
- **L2399 EN**: Executes a call or declaration centered on `preparedActual->loadTrivialScalar`.
  **L2399 CN**: 执行以 `preparedActual->loadTrivialScalar` 为核心的调用或声明。
- **L2400 EN**: Comment records a pending task or caution: `TODO: merge shape instead of using the first one.`.
  **L2400 CN**: 注释记录待办事项或注意点：`TODO: merge shape instead of using the first one.`。

### Lines 2401-2424

````cpp
        if (!shape && preparedActual->isArray()) {
          if (preparedActual->handleDynamicOptional())
            optionalWithShape = &*preparedActual;
          else
            shape = preparedActual->genShape(loc, builder);
        }
        // 15.8.3 p1. Elemental procedure with intent(out)/intent(inout)
        // arguments must be called in element order.
        if (impl().argMayBeModifiedByCall(i))
          mustBeOrdered = true;
      }
    }
    if (!shape && optionalWithShape) {
      // If all array operands appear in optional positions, then none of them
      // is allowed to be absent as per 15.5.2.12 point 3. (6). Just pick the
      // first operand.
      shape = optionalWithShape->genShape(loc, builder);
      // TODO: There is an opportunity to add a runtime check here that
      // this array is present as required. Also, the optionality of all actual
      // could be checked and reset given the Fortran requirement.
      optionalWithShape->resetOptionalAspect();
    }
    assert(shape &&
           "elemental array calls must have at least one array arguments");
````
- **L2401 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2401 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2402 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2402 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2403 EN**: Executes a standalone statement or declaration: `optionalWithShape = &*preparedActual;`.
  **L2403 CN**: 执行一条独立语句或声明：`optionalWithShape = &*preparedActual;`。
- **L2404 EN**: Transitions from the previous branch into the alternative path.
  **L2404 CN**: 从前一个分支过渡到备选路径。
- **L2405 EN**: Executes a call or declaration centered on `preparedActual->genShape`.
  **L2405 CN**: 执行以 `preparedActual->genShape` 为核心的调用或声明。
- **L2406 EN**: Closes the current lexical scope or compound statement.
  **L2406 CN**: 结束当前词法作用域或复合语句块。
- **L2407 EN**: Comment explains nearby logic, intent, or metadata: `15.8.3 p1. Elemental procedure with intent(out)/intent(inout)`.
  **L2407 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.8.3 p1. Elemental procedure with intent(out)/intent(inout)`。
- **L2408 EN**: Comment explains nearby logic, intent, or metadata: `arguments must be called in element order.`.
  **L2408 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments must be called in element order.`。
- **L2409 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2409 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2410 EN**: Executes a standalone statement or declaration: `mustBeOrdered = true;`.
  **L2410 CN**: 执行一条独立语句或声明：`mustBeOrdered = true;`。
- **L2411 EN**: Closes the current lexical scope or compound statement.
  **L2411 CN**: 结束当前词法作用域或复合语句块。
- **L2412 EN**: Closes the current lexical scope or compound statement.
  **L2412 CN**: 结束当前词法作用域或复合语句块。
- **L2413 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2413 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2414 EN**: Comment explains nearby logic, intent, or metadata: `If all array operands appear in optional positions, then none of them`.
  **L2414 CN**: 注释说明附近代码的逻辑、意图或元数据：`If all array operands appear in optional positions, then none of them`。
- **L2415 EN**: Comment explains nearby logic, intent, or metadata: `is allowed to be absent as per 15.5.2.12 point 3. (6). Just pick the`.
  **L2415 CN**: 注释说明附近代码的逻辑、意图或元数据：`is allowed to be absent as per 15.5.2.12 point 3. (6). Just pick the`。
- **L2416 EN**: Comment explains nearby logic, intent, or metadata: `first operand.`.
  **L2416 CN**: 注释说明附近代码的逻辑、意图或元数据：`first operand.`。
- **L2417 EN**: Executes a call or declaration centered on `optionalWithShape->genShape`.
  **L2417 CN**: 执行以 `optionalWithShape->genShape` 为核心的调用或声明。
- **L2418 EN**: Comment records a pending task or caution: `TODO: There is an opportunity to add a runtime check here that`.
  **L2418 CN**: 注释记录待办事项或注意点：`TODO: There is an opportunity to add a runtime check here that`。
- **L2419 EN**: Comment explains nearby logic, intent, or metadata: `this array is present as required. Also, the optionality of all actual`.
  **L2419 CN**: 注释说明附近代码的逻辑、意图或元数据：`this array is present as required. Also, the optionality of all actual`。
- **L2420 EN**: Comment explains nearby logic, intent, or metadata: `could be checked and reset given the Fortran requirement.`.
  **L2420 CN**: 注释说明附近代码的逻辑、意图或元数据：`could be checked and reset given the Fortran requirement.`。
- **L2421 EN**: Executes a call or declaration centered on `optionalWithShape->resetOptionalAspect`.
  **L2421 CN**: 执行以 `optionalWithShape->resetOptionalAspect` 为核心的调用或声明。
- **L2422 EN**: Closes the current lexical scope or compound statement.
  **L2422 CN**: 结束当前词法作用域或复合语句块。
- **L2423 EN**: Checks an internal invariant in debug builds.
  **L2423 CN**: 在调试构建中检查内部不变式。
- **L2424 EN**: Executes a standalone statement or declaration: `"elemental array calls must have at least one array arguments");`.
  **L2424 CN**: 执行一条独立语句或声明：`"elemental array calls must have at least one array arguments");`。

### Lines 2425-2448

````cpp

    // Evaluate the actual argument array expressions before the elemental
    // call of an impure subprogram or a subprogram with intent(out) or
    // intent(inout) arguments. Note that the scalar arguments are handled
    // above.
    if (mustBeOrdered) {
      for (auto &preparedActual : loweredActuals) {
        if (preparedActual) {
          if (hlfir::AssociateOp associate =
                  preparedActual->associateIfArrayExpr(loc, builder)) {
            fir::FirOpBuilder *bldr = &builder;
            callContext.stmtCtx.attachCleanup([=]() {
              hlfir::EndAssociateOp::create(*bldr, loc, associate);
            });
          }
        }
      }
    }

    // Push a new local scope so that any temps made inside the elemental
    // iterations are cleaned up inside the iterations.
    if (!callContext.resultType) {
      // Subroutine case. Generate call inside loop nest.
      hlfir::LoopNest loopNest =
````
- **L2425 EN**: Blank line separating nearby declarations or logic blocks.
  **L2425 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2426 EN**: Comment explains nearby logic, intent, or metadata: `Evaluate the actual argument array expressions before the elemental`.
  **L2426 CN**: 注释说明附近代码的逻辑、意图或元数据：`Evaluate the actual argument array expressions before the elemental`。
- **L2427 EN**: Comment explains nearby logic, intent, or metadata: `call of an impure subprogram or a subprogram with intent(out) or`.
  **L2427 CN**: 注释说明附近代码的逻辑、意图或元数据：`call of an impure subprogram or a subprogram with intent(out) or`。
- **L2428 EN**: Comment explains nearby logic, intent, or metadata: `intent(inout) arguments. Note that the scalar arguments are handled`.
  **L2428 CN**: 注释说明附近代码的逻辑、意图或元数据：`intent(inout) arguments. Note that the scalar arguments are handled`。
- **L2429 EN**: Comment explains nearby logic, intent, or metadata: `above.`.
  **L2429 CN**: 注释说明附近代码的逻辑、意图或元数据：`above.`。
- **L2430 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2430 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2431 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2431 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2432 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2432 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2433 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2433 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2434 EN**: Starts a function, method, lambda, or structured scope: `preparedActual->associateIfArrayExpr(loc, builder)) {`.
  **L2434 CN**: 开始一个函数、方法、lambda 或结构化作用域：`preparedActual->associateIfArrayExpr(loc, builder)) {`。
- **L2435 EN**: Executes a standalone statement or declaration: `fir::FirOpBuilder *bldr = &builder;`.
  **L2435 CN**: 执行一条独立语句或声明：`fir::FirOpBuilder *bldr = &builder;`。
- **L2436 EN**: Starts a function, method, lambda, or structured scope: `callContext.stmtCtx.attachCleanup([=]() {`.
  **L2436 CN**: 开始一个函数、方法、lambda 或结构化作用域：`callContext.stmtCtx.attachCleanup([=]() {`。
- **L2437 EN**: Executes a call or declaration centered on `hlfir::EndAssociateOp::create`.
  **L2437 CN**: 执行以 `hlfir::EndAssociateOp::create` 为核心的调用或声明。
- **L2438 EN**: Executes a standalone statement or declaration: `});`.
  **L2438 CN**: 执行一条独立语句或声明：`});`。
- **L2439 EN**: Closes the current lexical scope or compound statement.
  **L2439 CN**: 结束当前词法作用域或复合语句块。
- **L2440 EN**: Closes the current lexical scope or compound statement.
  **L2440 CN**: 结束当前词法作用域或复合语句块。
- **L2441 EN**: Closes the current lexical scope or compound statement.
  **L2441 CN**: 结束当前词法作用域或复合语句块。
- **L2442 EN**: Closes the current lexical scope or compound statement.
  **L2442 CN**: 结束当前词法作用域或复合语句块。
- **L2443 EN**: Blank line separating nearby declarations or logic blocks.
  **L2443 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2444 EN**: Comment explains nearby logic, intent, or metadata: `Push a new local scope so that any temps made inside the elemental`.
  **L2444 CN**: 注释说明附近代码的逻辑、意图或元数据：`Push a new local scope so that any temps made inside the elemental`。
- **L2445 EN**: Comment explains nearby logic, intent, or metadata: `iterations are cleaned up inside the iterations.`.
  **L2445 CN**: 注释说明附近代码的逻辑、意图或元数据：`iterations are cleaned up inside the iterations.`。
- **L2446 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2446 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2447 EN**: Comment explains nearby logic, intent, or metadata: `Subroutine case. Generate call inside loop nest.`.
  **L2447 CN**: 注释说明附近代码的逻辑、意图或元数据：`Subroutine case. Generate call inside loop nest.`。
- **L2448 EN**: Continues the surrounding expression or declaration: `hlfir::LoopNest loopNest =`.
  **L2448 CN**: 继续构造周围的表达式或声明：`hlfir::LoopNest loopNest =`。

### Lines 2449-2472

````cpp
          hlfir::genLoopNest(loc, builder, shape, !mustBeOrdered);
      mlir::ValueRange oneBasedIndices = loopNest.oneBasedIndices;
      auto insPt = builder.saveInsertionPoint();
      builder.setInsertionPointToStart(loopNest.body);
      callContext.stmtCtx.pushScope();
      for (auto &preparedActual : loweredActuals)
        if (preparedActual)
          preparedActual->setElementalIndices(oneBasedIndices);
      impl().genElementalKernel(loweredActuals, callContext);
      callContext.stmtCtx.finalizeAndPop();
      builder.restoreInsertionPoint(insPt);
      return std::nullopt;
    }
    // Function case: generate call inside hlfir.elemental
    mlir::Type elementType =
        hlfir::getFortranElementType(*callContext.resultType);
    // Get result length parameters.
    llvm::SmallVector<mlir::Value> typeParams;
    if (mlir::isa<fir::CharacterType>(elementType) ||
        fir::isRecordWithTypeParameters(elementType)) {
      auto charType = mlir::dyn_cast<fir::CharacterType>(elementType);
      if (charType && charType.hasConstantLen())
        typeParams.push_back(builder.createIntegerConstant(
            loc, builder.getIndexType(), charType.getLen()));
````
- **L2449 EN**: Executes a call or declaration centered on `hlfir::genLoopNest`.
  **L2449 CN**: 执行以 `hlfir::genLoopNest` 为核心的调用或声明。
- **L2450 EN**: Initializes variable `oneBasedIndices` from the right-hand expression.
  **L2450 CN**: 使用右侧表达式初始化变量 `oneBasedIndices`。
- **L2451 EN**: Initializes variable `insPt` from the right-hand expression.
  **L2451 CN**: 使用右侧表达式初始化变量 `insPt`。
- **L2452 EN**: Executes a call or declaration centered on `builder.setInsertionPointToStart`.
  **L2452 CN**: 执行以 `builder.setInsertionPointToStart` 为核心的调用或声明。
- **L2453 EN**: Executes a call or declaration centered on `callContext.stmtCtx.pushScope`.
  **L2453 CN**: 执行以 `callContext.stmtCtx.pushScope` 为核心的调用或声明。
- **L2454 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2454 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2455 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2455 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2456 EN**: Executes a call or declaration centered on `preparedActual->setElementalIndices`.
  **L2456 CN**: 执行以 `preparedActual->setElementalIndices` 为核心的调用或声明。
- **L2457 EN**: Executes a call or declaration centered on `impl`.
  **L2457 CN**: 执行以 `impl` 为核心的调用或声明。
- **L2458 EN**: Executes a call or declaration centered on `callContext.stmtCtx.finalizeAndPop`.
  **L2458 CN**: 执行以 `callContext.stmtCtx.finalizeAndPop` 为核心的调用或声明。
- **L2459 EN**: Executes a call or declaration centered on `builder.restoreInsertionPoint`.
  **L2459 CN**: 执行以 `builder.restoreInsertionPoint` 为核心的调用或声明。
- **L2460 EN**: Returns from the current function with `std::nullopt`.
  **L2460 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2461 EN**: Closes the current lexical scope or compound statement.
  **L2461 CN**: 结束当前词法作用域或复合语句块。
- **L2462 EN**: Comment explains nearby logic, intent, or metadata: `Function case: generate call inside hlfir.elemental`.
  **L2462 CN**: 注释说明附近代码的逻辑、意图或元数据：`Function case: generate call inside hlfir.elemental`。
- **L2463 EN**: Continues the surrounding expression or declaration: `mlir::Type elementType =`.
  **L2463 CN**: 继续构造周围的表达式或声明：`mlir::Type elementType =`。
- **L2464 EN**: Executes a call or declaration centered on `hlfir::getFortranElementType`.
  **L2464 CN**: 执行以 `hlfir::getFortranElementType` 为核心的调用或声明。
- **L2465 EN**: Comment explains nearby logic, intent, or metadata: `Get result length parameters.`.
  **L2465 CN**: 注释说明附近代码的逻辑、意图或元数据：`Get result length parameters.`。
- **L2466 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> typeParams;`.
  **L2466 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> typeParams;`。
- **L2467 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2467 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2468 EN**: Starts a function, method, lambda, or structured scope: `fir::isRecordWithTypeParameters(elementType)) {`.
  **L2468 CN**: 开始一个函数、方法、lambda 或结构化作用域：`fir::isRecordWithTypeParameters(elementType)) {`。
- **L2469 EN**: Initializes variable `charType` from the right-hand expression.
  **L2469 CN**: 使用右侧表达式初始化变量 `charType`。
- **L2470 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2470 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2471 EN**: Continues logic associated with callable symbol `push_back`.
  **L2471 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L2472 EN**: Executes a call or declaration centered on `builder.getIndexType`.
  **L2472 CN**: 执行以 `builder.getIndexType` 为核心的调用或声明。

### Lines 2473-2496

````cpp
      else if (charType)
        typeParams.push_back(impl().computeDynamicCharacterResultLength(
            loweredActuals, callContext));
      else
        TODO(
            loc,
            "compute elemental PDT function result length parameters in HLFIR");
    }
    auto genKernel = [&](mlir::Location l, fir::FirOpBuilder &b,
                         mlir::ValueRange oneBasedIndices) -> hlfir::Entity {
      callContext.stmtCtx.pushScope();
      for (auto &preparedActual : loweredActuals)
        if (preparedActual)
          preparedActual->setElementalIndices(oneBasedIndices);
      auto res = *impl().genElementalKernel(loweredActuals, callContext);
      callContext.stmtCtx.finalizeAndPop();
      // Note that an hlfir.destroy is not emitted for the result since it
      // is still used by the hlfir.yield_element that also marks its last
      // use.
      return res;
    };
    mlir::Value polymorphicMold;
    if (fir::isPolymorphicType(*callContext.resultType))
      polymorphicMold =
````
- **L2473 EN**: Starts the alternative branch of the preceding conditional.
  **L2473 CN**: 开始前一个条件语句的备选分支。
- **L2474 EN**: Continues logic associated with callable symbol `push_back`.
  **L2474 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L2475 EN**: Executes a standalone statement or declaration: `loweredActuals, callContext));`.
  **L2475 CN**: 执行一条独立语句或声明：`loweredActuals, callContext));`。
- **L2476 EN**: Transitions from the previous branch into the alternative path.
  **L2476 CN**: 从前一个分支过渡到备选路径。
- **L2477 EN**: Continues logic associated with callable symbol `TODO`.
  **L2477 CN**: 继续与可调用符号 `TODO` 相关的逻辑。
- **L2478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc,`.
  **L2478 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc,`。
- **L2479 EN**: Executes a standalone statement or declaration: `"compute elemental PDT function result length parameters in HLFIR");`.
  **L2479 CN**: 执行一条独立语句或声明：`"compute elemental PDT function result length parameters in HLFIR");`。
- **L2480 EN**: Closes the current lexical scope or compound statement.
  **L2480 CN**: 结束当前词法作用域或复合语句块。
- **L2481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto genKernel = [&](mlir::Location l, fir::FirOpBuilder &b,`.
  **L2481 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto genKernel = [&](mlir::Location l, fir::FirOpBuilder &b,`。
- **L2482 EN**: Continues the surrounding expression or declaration: `mlir::ValueRange oneBasedIndices) -> hlfir::Entity {`.
  **L2482 CN**: 继续构造周围的表达式或声明：`mlir::ValueRange oneBasedIndices) -> hlfir::Entity {`。
- **L2483 EN**: Executes a call or declaration centered on `callContext.stmtCtx.pushScope`.
  **L2483 CN**: 执行以 `callContext.stmtCtx.pushScope` 为核心的调用或声明。
- **L2484 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2484 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2485 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2485 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2486 EN**: Executes a call or declaration centered on `preparedActual->setElementalIndices`.
  **L2486 CN**: 执行以 `preparedActual->setElementalIndices` 为核心的调用或声明。
- **L2487 EN**: Initializes variable `res` from the right-hand expression.
  **L2487 CN**: 使用右侧表达式初始化变量 `res`。
- **L2488 EN**: Executes a call or declaration centered on `callContext.stmtCtx.finalizeAndPop`.
  **L2488 CN**: 执行以 `callContext.stmtCtx.finalizeAndPop` 为核心的调用或声明。
- **L2489 EN**: Comment explains nearby logic, intent, or metadata: `Note that an hlfir.destroy is not emitted for the result since it`.
  **L2489 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note that an hlfir.destroy is not emitted for the result since it`。
- **L2490 EN**: Comment explains nearby logic, intent, or metadata: `is still used by the hlfir.yield_element that also marks its last`.
  **L2490 CN**: 注释说明附近代码的逻辑、意图或元数据：`is still used by the hlfir.yield_element that also marks its last`。
- **L2491 EN**: Comment explains nearby logic, intent, or metadata: `use.`.
  **L2491 CN**: 注释说明附近代码的逻辑、意图或元数据：`use.`。
- **L2492 EN**: Returns from the current function with `res`.
  **L2492 CN**: 以 `res` 从当前函数返回。
- **L2493 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2493 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2494 EN**: Executes a standalone statement or declaration: `mlir::Value polymorphicMold;`.
  **L2494 CN**: 执行一条独立语句或声明：`mlir::Value polymorphicMold;`。
- **L2495 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2495 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2496 EN**: Continues the surrounding expression or declaration: `polymorphicMold =`.
  **L2496 CN**: 继续构造周围的表达式或声明：`polymorphicMold =`。

### Lines 2497-2520

````cpp
          impl().getPolymorphicResultMold(loweredActuals, callContext);
    mlir::Value elemental =
        hlfir::genElementalOp(loc, builder, elementType, shape, typeParams,
                              genKernel, !mustBeOrdered, polymorphicMold);
    // If the function result requires finalization, then it has to be done
    // for the array result of the elemental call. We have to communicate
    // this via the DestroyOp's attribute.
    bool mustFinalizeExpr = impl().resultMayRequireFinalization(callContext);
    fir::FirOpBuilder *bldr = &builder;
    callContext.stmtCtx.attachCleanup([=]() {
      hlfir::DestroyOp::create(*bldr, loc, elemental, mustFinalizeExpr);
    });
    return hlfir::EntityWithAttributes{elemental};
  }

private:
  ElementalCallBuilderImpl &impl() {
    return *static_cast<ElementalCallBuilderImpl *>(this);
  }
};

/// Helper for computing elemental function result specification
/// expressions that depends on dummy symbols. See
/// computeDynamicCharacterResultLength below.
````
- **L2497 EN**: Executes a call or declaration centered on `impl`.
  **L2497 CN**: 执行以 `impl` 为核心的调用或声明。
- **L2498 EN**: Continues the surrounding expression or declaration: `mlir::Value elemental =`.
  **L2498 CN**: 继续构造周围的表达式或声明：`mlir::Value elemental =`。
- **L2499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `hlfir::genElementalOp(loc, builder, elementType, shape, typeParams,`.
  **L2499 CN**: 继续一个多行参数列表、初始化器或聚合项：`hlfir::genElementalOp(loc, builder, elementType, shape, typeParams,`。
- **L2500 EN**: Executes a standalone statement or declaration: `genKernel, !mustBeOrdered, polymorphicMold);`.
  **L2500 CN**: 执行一条独立语句或声明：`genKernel, !mustBeOrdered, polymorphicMold);`。
- **L2501 EN**: Comment explains nearby logic, intent, or metadata: `If the function result requires finalization, then it has to be done`.
  **L2501 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the function result requires finalization, then it has to be done`。
- **L2502 EN**: Comment explains nearby logic, intent, or metadata: `for the array result of the elemental call. We have to communicate`.
  **L2502 CN**: 注释说明附近代码的逻辑、意图或元数据：`for the array result of the elemental call. We have to communicate`。
- **L2503 EN**: Comment explains nearby logic, intent, or metadata: `this via the DestroyOp's attribute.`.
  **L2503 CN**: 注释说明附近代码的逻辑、意图或元数据：`this via the DestroyOp's attribute.`。
- **L2504 EN**: Initializes variable `mustFinalizeExpr` from the right-hand expression.
  **L2504 CN**: 使用右侧表达式初始化变量 `mustFinalizeExpr`。
- **L2505 EN**: Executes a standalone statement or declaration: `fir::FirOpBuilder *bldr = &builder;`.
  **L2505 CN**: 执行一条独立语句或声明：`fir::FirOpBuilder *bldr = &builder;`。
- **L2506 EN**: Starts a function, method, lambda, or structured scope: `callContext.stmtCtx.attachCleanup([=]() {`.
  **L2506 CN**: 开始一个函数、方法、lambda 或结构化作用域：`callContext.stmtCtx.attachCleanup([=]() {`。
- **L2507 EN**: Executes a call or declaration centered on `hlfir::DestroyOp::create`.
  **L2507 CN**: 执行以 `hlfir::DestroyOp::create` 为核心的调用或声明。
- **L2508 EN**: Executes a standalone statement or declaration: `});`.
  **L2508 CN**: 执行一条独立语句或声明：`});`。
- **L2509 EN**: Returns from the current function with `hlfir::EntityWithAttributes{elemental}`.
  **L2509 CN**: 以 `hlfir::EntityWithAttributes{elemental}` 从当前函数返回。
- **L2510 EN**: Closes the current lexical scope or compound statement.
  **L2510 CN**: 结束当前词法作用域或复合语句块。
- **L2511 EN**: Blank line separating nearby declarations or logic blocks.
  **L2511 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2512 EN**: Sets the following members to `private` access.
  **L2512 CN**: 将后续成员的访问级别设为 `private`。
- **L2513 EN**: Starts a function, method, lambda, or structured scope: `ElementalCallBuilderImpl &impl() {`.
  **L2513 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ElementalCallBuilderImpl &impl() {`。
- **L2514 EN**: Returns from the current function with `*static_cast<ElementalCallBuilderImpl *>(this)`.
  **L2514 CN**: 以 `*static_cast<ElementalCallBuilderImpl *>(this)` 从当前函数返回。
- **L2515 EN**: Closes the current lexical scope or compound statement.
  **L2515 CN**: 结束当前词法作用域或复合语句块。
- **L2516 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2516 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2517 EN**: Blank line separating nearby declarations or logic blocks.
  **L2517 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2518 EN**: Comment explains nearby logic, intent, or metadata: `Helper for computing elemental function result specification`.
  **L2518 CN**: 注释说明附近代码的逻辑、意图或元数据：`Helper for computing elemental function result specification`。
- **L2519 EN**: Comment explains nearby logic, intent, or metadata: `expressions that depends on dummy symbols. See`.
  **L2519 CN**: 注释说明附近代码的逻辑、意图或元数据：`expressions that depends on dummy symbols. See`。
- **L2520 EN**: Comment explains nearby logic, intent, or metadata: `computeDynamicCharacterResultLength below.`.
  **L2520 CN**: 注释说明附近代码的逻辑、意图或元数据：`computeDynamicCharacterResultLength below.`。

### Lines 2521-2544

````cpp
static mlir::Value genMockDummyForElementalResultSpecifications(
    fir::FirOpBuilder &builder, mlir::Location loc, mlir::Type dummyType,
    Fortran::lower::PreparedActualArgument &preparedActual) {
  // One is used as the mock address instead of NULL so that PRESENT inquires
  // work (this is the only valid thing that specification can do with the
  // address thanks to Fortran 2023 C15121).
  mlir::Value one =
      builder.createIntegerConstant(loc, builder.getIntPtrType(), 1);
  if (auto boxCharType = llvm::dyn_cast<fir::BoxCharType>(dummyType)) {
    mlir::Value addr = builder.createConvert(
        loc, fir::ReferenceType::get(boxCharType.getEleTy()), one);
    mlir::Value len = preparedActual.genCharLength(loc, builder);
    return fir::EmboxCharOp::create(builder, loc, boxCharType, addr, len);
  }
  if (auto box = llvm::dyn_cast<fir::BaseBoxType>(dummyType)) {
    mlir::Value addr =
        builder.createConvert(loc, box.getBaseAddressType(), one);
    llvm::SmallVector<mlir::Value> lenParams;
    preparedActual.genLengthParameters(loc, builder, lenParams);
    mlir::Value mold;
    if (fir::isPolymorphicType(box))
      mold = preparedActual.getPolymorphicMold(loc);
    return fir::EmboxOp::create(builder, loc, box, addr,
                                /*shape=*/mlir::Value{},
````
- **L2521 EN**: Continues logic associated with callable symbol `genMockDummyForElementalResultSpecifications`.
  **L2521 CN**: 继续与可调用符号 `genMockDummyForElementalResultSpecifications` 相关的逻辑。
- **L2522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::FirOpBuilder &builder, mlir::Location loc, mlir::Type dummyType,`.
  **L2522 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::FirOpBuilder &builder, mlir::Location loc, mlir::Type dummyType,`。
- **L2523 EN**: Continues the surrounding expression or declaration: `Fortran::lower::PreparedActualArgument &preparedActual) {`.
  **L2523 CN**: 继续构造周围的表达式或声明：`Fortran::lower::PreparedActualArgument &preparedActual) {`。
- **L2524 EN**: Comment explains nearby logic, intent, or metadata: `One is used as the mock address instead of NULL so that PRESENT inquires`.
  **L2524 CN**: 注释说明附近代码的逻辑、意图或元数据：`One is used as the mock address instead of NULL so that PRESENT inquires`。
- **L2525 EN**: Comment explains nearby logic, intent, or metadata: `work (this is the only valid thing that specification can do with the`.
  **L2525 CN**: 注释说明附近代码的逻辑、意图或元数据：`work (this is the only valid thing that specification can do with the`。
- **L2526 EN**: Comment explains nearby logic, intent, or metadata: `address thanks to Fortran 2023 C15121).`.
  **L2526 CN**: 注释说明附近代码的逻辑、意图或元数据：`address thanks to Fortran 2023 C15121).`。
- **L2527 EN**: Continues the surrounding expression or declaration: `mlir::Value one =`.
  **L2527 CN**: 继续构造周围的表达式或声明：`mlir::Value one =`。
- **L2528 EN**: Executes a call or declaration centered on `builder.createIntegerConstant`.
  **L2528 CN**: 执行以 `builder.createIntegerConstant` 为核心的调用或声明。
- **L2529 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2529 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2530 EN**: Continues logic associated with callable symbol `createConvert`.
  **L2530 CN**: 继续与可调用符号 `createConvert` 相关的逻辑。
- **L2531 EN**: Executes a call or declaration centered on `fir::ReferenceType::get`.
  **L2531 CN**: 执行以 `fir::ReferenceType::get` 为核心的调用或声明。
- **L2532 EN**: Initializes variable `len` from the right-hand expression.
  **L2532 CN**: 使用右侧表达式初始化变量 `len`。
- **L2533 EN**: Returns from the current function with `fir::EmboxCharOp::create(builder, loc, boxCharType, addr, len)`.
  **L2533 CN**: 以 `fir::EmboxCharOp::create(builder, loc, boxCharType, addr, len)` 从当前函数返回。
- **L2534 EN**: Closes the current lexical scope or compound statement.
  **L2534 CN**: 结束当前词法作用域或复合语句块。
- **L2535 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2535 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2536 EN**: Continues the surrounding expression or declaration: `mlir::Value addr =`.
  **L2536 CN**: 继续构造周围的表达式或声明：`mlir::Value addr =`。
- **L2537 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L2537 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L2538 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> lenParams;`.
  **L2538 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> lenParams;`。
- **L2539 EN**: Executes a call or declaration centered on `preparedActual.genLengthParameters`.
  **L2539 CN**: 执行以 `preparedActual.genLengthParameters` 为核心的调用或声明。
- **L2540 EN**: Executes a standalone statement or declaration: `mlir::Value mold;`.
  **L2540 CN**: 执行一条独立语句或声明：`mlir::Value mold;`。
- **L2541 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2541 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2542 EN**: Executes a call or declaration centered on `preparedActual.getPolymorphicMold`.
  **L2542 CN**: 执行以 `preparedActual.getPolymorphicMold` 为核心的调用或声明。
- **L2543 EN**: Returns from the current function with `fir::EmboxOp::create(builder, loc, box, addr,`.
  **L2543 CN**: 以 `fir::EmboxOp::create(builder, loc, box, addr,` 从当前函数返回。
- **L2544 EN**: Comment explains nearby logic, intent, or metadata: `shape=*/mlir::Value{},`.
  **L2544 CN**: 注释说明附近代码的逻辑、意图或元数据：`shape=*/mlir::Value{},`。

### Lines 2545-2568

````cpp
                                /*slice=*/mlir::Value{}, lenParams, mold);
  }
  // Values of arguments should not be used in elemental procedure specification
  // expressions as per C15121, so it makes no sense to have a specification
  // expression requiring a symbol that is passed by value (there is no good
  // value to create here).
  assert(fir::isa_ref_type(dummyType) &&
         (fir::isa_trivial(fir::unwrapRefType(dummyType)) ||
          fir::isa_char(fir::unwrapRefType(dummyType))) &&
         "Only expect symbols inquired in elemental procedure result "
         "specifications to be passed in memory");
  return builder.createConvert(loc, dummyType, one);
}

class ElementalUserCallBuilder
    : public ElementalCallBuilder<ElementalUserCallBuilder> {
public:
  ElementalUserCallBuilder(Fortran::lower::CallerInterface &caller,
                           mlir::FunctionType callSiteType)
      : caller{caller}, callSiteType{callSiteType} {}
  std::optional<hlfir::Entity>
  genElementalKernel(Fortran::lower::PreparedActualArguments &loweredActuals,
                     CallContext &callContext) {
    return genUserCall(loweredActuals, caller, callSiteType, callContext);
````
- **L2545 EN**: Comment explains nearby logic, intent, or metadata: `slice=*/mlir::Value{}, lenParams, mold);`.
  **L2545 CN**: 注释说明附近代码的逻辑、意图或元数据：`slice=*/mlir::Value{}, lenParams, mold);`。
- **L2546 EN**: Closes the current lexical scope or compound statement.
  **L2546 CN**: 结束当前词法作用域或复合语句块。
- **L2547 EN**: Comment explains nearby logic, intent, or metadata: `Values of arguments should not be used in elemental procedure specification`.
  **L2547 CN**: 注释说明附近代码的逻辑、意图或元数据：`Values of arguments should not be used in elemental procedure specification`。
- **L2548 EN**: Comment explains nearby logic, intent, or metadata: `expressions as per C15121, so it makes no sense to have a specification`.
  **L2548 CN**: 注释说明附近代码的逻辑、意图或元数据：`expressions as per C15121, so it makes no sense to have a specification`。
- **L2549 EN**: Comment explains nearby logic, intent, or metadata: `expression requiring a symbol that is passed by value (there is no good`.
  **L2549 CN**: 注释说明附近代码的逻辑、意图或元数据：`expression requiring a symbol that is passed by value (there is no good`。
- **L2550 EN**: Comment explains nearby logic, intent, or metadata: `value to create here).`.
  **L2550 CN**: 注释说明附近代码的逻辑、意图或元数据：`value to create here).`。
- **L2551 EN**: Checks an internal invariant in debug builds.
  **L2551 CN**: 在调试构建中检查内部不变式。
- **L2552 EN**: Continues logic associated with callable symbol `isa_trivial`.
  **L2552 CN**: 继续与可调用符号 `isa_trivial` 相关的逻辑。
- **L2553 EN**: Continues logic associated with callable symbol `isa_char`.
  **L2553 CN**: 继续与可调用符号 `isa_char` 相关的逻辑。
- **L2554 EN**: Continues the surrounding expression or declaration: `"Only expect symbols inquired in elemental procedure result "`.
  **L2554 CN**: 继续构造周围的表达式或声明：`"Only expect symbols inquired in elemental procedure result "`。
- **L2555 EN**: Executes a standalone statement or declaration: `"specifications to be passed in memory");`.
  **L2555 CN**: 执行一条独立语句或声明：`"specifications to be passed in memory");`。
- **L2556 EN**: Returns from the current function with `builder.createConvert(loc, dummyType, one)`.
  **L2556 CN**: 以 `builder.createConvert(loc, dummyType, one)` 从当前函数返回。
- **L2557 EN**: Closes the current lexical scope or compound statement.
  **L2557 CN**: 结束当前词法作用域或复合语句块。
- **L2558 EN**: Blank line separating nearby declarations or logic blocks.
  **L2558 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2559 EN**: Declares class `ElementalUserCallBuilder`.
  **L2559 CN**: 声明 class `ElementalUserCallBuilder`。
- **L2560 EN**: Continues the surrounding expression or declaration: `: public ElementalCallBuilder<ElementalUserCallBuilder> {`.
  **L2560 CN**: 继续构造周围的表达式或声明：`: public ElementalCallBuilder<ElementalUserCallBuilder> {`。
- **L2561 EN**: Sets the following members to `public` access.
  **L2561 CN**: 将后续成员的访问级别设为 `public`。
- **L2562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ElementalUserCallBuilder(Fortran::lower::CallerInterface &caller,`.
  **L2562 CN**: 继续一个多行参数列表、初始化器或聚合项：`ElementalUserCallBuilder(Fortran::lower::CallerInterface &caller,`。
- **L2563 EN**: Continues the surrounding expression or declaration: `mlir::FunctionType callSiteType)`.
  **L2563 CN**: 继续构造周围的表达式或声明：`mlir::FunctionType callSiteType)`。
- **L2564 EN**: Continues the surrounding expression or declaration: `: caller{caller}, callSiteType{callSiteType} {}`.
  **L2564 CN**: 继续构造周围的表达式或声明：`: caller{caller}, callSiteType{callSiteType} {}`。
- **L2565 EN**: Continues the surrounding expression or declaration: `std::optional<hlfir::Entity>`.
  **L2565 CN**: 继续构造周围的表达式或声明：`std::optional<hlfir::Entity>`。
- **L2566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genElementalKernel(Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L2566 CN**: 继续一个多行参数列表、初始化器或聚合项：`genElementalKernel(Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L2567 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L2567 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L2568 EN**: Returns from the current function with `genUserCall(loweredActuals, caller, callSiteType, callContext)`.
  **L2568 CN**: 以 `genUserCall(loweredActuals, caller, callSiteType, callContext)` 从当前函数返回。

### Lines 2569-2592

````cpp
  }

  bool argMayBeModifiedByCall(unsigned argIdx) const {
    assert(argIdx < caller.getPassedArguments().size() && "bad argument index");
    return caller.getPassedArguments()[argIdx].mayBeModifiedByCall();
  }

  bool canLoadActualArgumentBeforeLoop(unsigned argIdx) const {
    using PassBy = Fortran::lower::CallerInterface::PassEntityBy;
    const auto &passedArgs{caller.getPassedArguments()};
    assert(argIdx < passedArgs.size() && "bad argument index");
    // If the actual argument does not need to be passed via an address,
    // or will be passed in the address of a temporary copy, it can be loaded
    // before the elemental loop nest.
    const auto &arg{passedArgs[argIdx]};
    return arg.passBy == PassBy::Value ||
           arg.passBy == PassBy::BaseAddressValueAttribute;
  }

  mlir::Value computeDynamicCharacterResultLength(
      Fortran::lower::PreparedActualArguments &loweredActuals,
      CallContext &callContext) {

    fir::FirOpBuilder &builder = callContext.getBuilder();
````
- **L2569 EN**: Closes the current lexical scope or compound statement.
  **L2569 CN**: 结束当前词法作用域或复合语句块。
- **L2570 EN**: Blank line separating nearby declarations or logic blocks.
  **L2570 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2571 EN**: Starts a function, method, lambda, or structured scope: `bool argMayBeModifiedByCall(unsigned argIdx) const {`.
  **L2571 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool argMayBeModifiedByCall(unsigned argIdx) const {`。
- **L2572 EN**: Checks an internal invariant in debug builds.
  **L2572 CN**: 在调试构建中检查内部不变式。
- **L2573 EN**: Returns from the current function with `caller.getPassedArguments()[argIdx].mayBeModifiedByCall()`.
  **L2573 CN**: 以 `caller.getPassedArguments()[argIdx].mayBeModifiedByCall()` 从当前函数返回。
- **L2574 EN**: Closes the current lexical scope or compound statement.
  **L2574 CN**: 结束当前词法作用域或复合语句块。
- **L2575 EN**: Blank line separating nearby declarations or logic blocks.
  **L2575 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2576 EN**: Starts a function, method, lambda, or structured scope: `bool canLoadActualArgumentBeforeLoop(unsigned argIdx) const {`.
  **L2576 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool canLoadActualArgumentBeforeLoop(unsigned argIdx) const {`。
- **L2577 EN**: Defines alias `PassBy` to simplify later code.
  **L2577 CN**: 定义别名 `PassBy` 以简化后续代码。
- **L2578 EN**: Executes a call or declaration centered on `&passedArgs{caller.getPassedArguments`.
  **L2578 CN**: 执行以 `&passedArgs{caller.getPassedArguments` 为核心的调用或声明。
- **L2579 EN**: Checks an internal invariant in debug builds.
  **L2579 CN**: 在调试构建中检查内部不变式。
- **L2580 EN**: Comment explains nearby logic, intent, or metadata: `If the actual argument does not need to be passed via an address,`.
  **L2580 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the actual argument does not need to be passed via an address,`。
- **L2581 EN**: Comment explains nearby logic, intent, or metadata: `or will be passed in the address of a temporary copy, it can be loaded`.
  **L2581 CN**: 注释说明附近代码的逻辑、意图或元数据：`or will be passed in the address of a temporary copy, it can be loaded`。
- **L2582 EN**: Comment explains nearby logic, intent, or metadata: `before the elemental loop nest.`.
  **L2582 CN**: 注释说明附近代码的逻辑、意图或元数据：`before the elemental loop nest.`。
- **L2583 EN**: Executes a standalone statement or declaration: `const auto &arg{passedArgs[argIdx]};`.
  **L2583 CN**: 执行一条独立语句或声明：`const auto &arg{passedArgs[argIdx]};`。
- **L2584 EN**: Returns from the current function with `arg.passBy == PassBy::Value ||`.
  **L2584 CN**: 以 `arg.passBy == PassBy::Value ||` 从当前函数返回。
- **L2585 EN**: Executes a standalone statement or declaration: `arg.passBy == PassBy::BaseAddressValueAttribute;`.
  **L2585 CN**: 执行一条独立语句或声明：`arg.passBy == PassBy::BaseAddressValueAttribute;`。
- **L2586 EN**: Closes the current lexical scope or compound statement.
  **L2586 CN**: 结束当前词法作用域或复合语句块。
- **L2587 EN**: Blank line separating nearby declarations or logic blocks.
  **L2587 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2588 EN**: Continues logic associated with callable symbol `computeDynamicCharacterResultLength`.
  **L2588 CN**: 继续与可调用符号 `computeDynamicCharacterResultLength` 相关的逻辑。
- **L2589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L2589 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L2590 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L2590 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L2591 EN**: Blank line separating nearby declarations or logic blocks.
  **L2591 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2592 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L2592 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。

### Lines 2593-2616

````cpp
    mlir::Location loc = callContext.loc;
    auto &converter = callContext.converter;

    // Gather the dummy argument symbols required directly or indirectly to
    // evaluate the result symbol specification expressions.
    llvm::SmallPtrSet<const Fortran::semantics::Symbol *, 4>
        requiredDummySymbols;
    const Fortran::semantics::Symbol &result = caller.getResultSymbol();
    for (Fortran::lower::pft::Variable var :
         Fortran::lower::pft::getDependentVariableList(result))
      if (var.hasSymbol()) {
        const Fortran::semantics::Symbol &sym = var.getSymbol();
        if (Fortran::semantics::IsDummy(sym) && sym.owner() == result.owner())
          requiredDummySymbols.insert(&sym);
      }

    // Prepare mock FIR arguments for each dummy arguments required in the
    // result specifications. These mock arguments will have the same properties
    // (dynamic type and type parameters) as the actual arguments, except for
    // the address. Such mock argument are needed because this evaluation is
    // happening before the loop for the elemental call (the array result
    // storage must be allocated before the loops if any is needed, so the
    // result properties must be known before the loops). So it is not possible
    // to just pick an element (like the first one) and use that because the
````
- **L2593 EN**: Initializes variable `loc` from the right-hand expression.
  **L2593 CN**: 使用右侧表达式初始化变量 `loc`。
- **L2594 EN**: Executes a standalone statement or declaration: `auto &converter = callContext.converter;`.
  **L2594 CN**: 执行一条独立语句或声明：`auto &converter = callContext.converter;`。
- **L2595 EN**: Blank line separating nearby declarations or logic blocks.
  **L2595 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2596 EN**: Comment explains nearby logic, intent, or metadata: `Gather the dummy argument symbols required directly or indirectly to`.
  **L2596 CN**: 注释说明附近代码的逻辑、意图或元数据：`Gather the dummy argument symbols required directly or indirectly to`。
- **L2597 EN**: Comment explains nearby logic, intent, or metadata: `evaluate the result symbol specification expressions.`.
  **L2597 CN**: 注释说明附近代码的逻辑、意图或元数据：`evaluate the result symbol specification expressions.`。
- **L2598 EN**: Continues the surrounding expression or declaration: `llvm::SmallPtrSet<const Fortran::semantics::Symbol *, 4>`.
  **L2598 CN**: 继续构造周围的表达式或声明：`llvm::SmallPtrSet<const Fortran::semantics::Symbol *, 4>`。
- **L2599 EN**: Executes a standalone statement or declaration: `requiredDummySymbols;`.
  **L2599 CN**: 执行一条独立语句或声明：`requiredDummySymbols;`。
- **L2600 EN**: Executes a call or declaration centered on `caller.getResultSymbol`.
  **L2600 CN**: 执行以 `caller.getResultSymbol` 为核心的调用或声明。
- **L2601 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2601 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2602 EN**: Continues logic associated with callable symbol `getDependentVariableList`.
  **L2602 CN**: 继续与可调用符号 `getDependentVariableList` 相关的逻辑。
- **L2603 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2603 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2604 EN**: Executes a call or declaration centered on `var.getSymbol`.
  **L2604 CN**: 执行以 `var.getSymbol` 为核心的调用或声明。
- **L2605 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2605 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2606 EN**: Executes a call or declaration centered on `requiredDummySymbols.insert`.
  **L2606 CN**: 执行以 `requiredDummySymbols.insert` 为核心的调用或声明。
- **L2607 EN**: Closes the current lexical scope or compound statement.
  **L2607 CN**: 结束当前词法作用域或复合语句块。
- **L2608 EN**: Blank line separating nearby declarations or logic blocks.
  **L2608 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2609 EN**: Comment explains nearby logic, intent, or metadata: `Prepare mock FIR arguments for each dummy arguments required in the`.
  **L2609 CN**: 注释说明附近代码的逻辑、意图或元数据：`Prepare mock FIR arguments for each dummy arguments required in the`。
- **L2610 EN**: Comment explains nearby logic, intent, or metadata: `result specifications. These mock arguments will have the same properties`.
  **L2610 CN**: 注释说明附近代码的逻辑、意图或元数据：`result specifications. These mock arguments will have the same properties`。
- **L2611 EN**: Comment explains nearby logic, intent, or metadata: `(dynamic type and type parameters) as the actual arguments, except for`.
  **L2611 CN**: 注释说明附近代码的逻辑、意图或元数据：`(dynamic type and type parameters) as the actual arguments, except for`。
- **L2612 EN**: Comment explains nearby logic, intent, or metadata: `the address. Such mock argument are needed because this evaluation is`.
  **L2612 CN**: 注释说明附近代码的逻辑、意图或元数据：`the address. Such mock argument are needed because this evaluation is`。
- **L2613 EN**: Comment explains nearby logic, intent, or metadata: `happening before the loop for the elemental call (the array result`.
  **L2613 CN**: 注释说明附近代码的逻辑、意图或元数据：`happening before the loop for the elemental call (the array result`。
- **L2614 EN**: Comment explains nearby logic, intent, or metadata: `storage must be allocated before the loops if any is needed, so the`.
  **L2614 CN**: 注释说明附近代码的逻辑、意图或元数据：`storage must be allocated before the loops if any is needed, so the`。
- **L2615 EN**: Comment explains nearby logic, intent, or metadata: `result properties must be known before the loops). So it is not possible`.
  **L2615 CN**: 注释说明附近代码的逻辑、意图或元数据：`result properties must be known before the loops). So it is not possible`。
- **L2616 EN**: Comment explains nearby logic, intent, or metadata: `to just pick an element (like the first one) and use that because the`.
  **L2616 CN**: 注释说明附近代码的逻辑、意图或元数据：`to just pick an element (like the first one) and use that because the`。

### Lines 2617-2640

````cpp
    // normal argument preparation have effects (vector subscripted actual
    // argument will require reading the vector subscript and VALUE arguments
    // preparation involve copies of the data. This could cause segfaults in
    // case of zero size arrays and is in general pointless extra computation
    // since the data cannot be used in the specification expression as per
    // C15121).
    if (!requiredDummySymbols.empty()) {
      const Fortran::semantics::SubprogramDetails *iface =
          caller.getInterfaceDetails();
      assert(iface && "interface must be explicit when result specification "
                      "depends upon dummy symbols");
      for (auto [maybePreparedActual, arg, sym] : llvm::zip(
               loweredActuals, caller.getPassedArguments(), iface->dummyArgs()))
        if (requiredDummySymbols.contains(sym)) {
          mlir::Type dummyType = callSiteType.getInput(arg.firArgument);

          if (!maybePreparedActual.has_value()) {
            mlir::Value mockArgValue =
                fir::AbsentOp::create(builder, loc, dummyType);
            caller.placeInput(arg, mockArgValue);
            continue;
          }

          Fortran::lower::PreparedActualArgument &preparedActual =
````
- **L2617 EN**: Comment explains nearby logic, intent, or metadata: `normal argument preparation have effects (vector subscripted actual`.
  **L2617 CN**: 注释说明附近代码的逻辑、意图或元数据：`normal argument preparation have effects (vector subscripted actual`。
- **L2618 EN**: Comment explains nearby logic, intent, or metadata: `argument will require reading the vector subscript and VALUE arguments`.
  **L2618 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument will require reading the vector subscript and VALUE arguments`。
- **L2619 EN**: Comment explains nearby logic, intent, or metadata: `preparation involve copies of the data. This could cause segfaults in`.
  **L2619 CN**: 注释说明附近代码的逻辑、意图或元数据：`preparation involve copies of the data. This could cause segfaults in`。
- **L2620 EN**: Comment explains nearby logic, intent, or metadata: `case of zero size arrays and is in general pointless extra computation`.
  **L2620 CN**: 注释说明附近代码的逻辑、意图或元数据：`case of zero size arrays and is in general pointless extra computation`。
- **L2621 EN**: Comment explains nearby logic, intent, or metadata: `since the data cannot be used in the specification expression as per`.
  **L2621 CN**: 注释说明附近代码的逻辑、意图或元数据：`since the data cannot be used in the specification expression as per`。
- **L2622 EN**: Comment explains nearby logic, intent, or metadata: `C15121).`.
  **L2622 CN**: 注释说明附近代码的逻辑、意图或元数据：`C15121).`。
- **L2623 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2623 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2624 EN**: Continues the surrounding expression or declaration: `const Fortran::semantics::SubprogramDetails *iface =`.
  **L2624 CN**: 继续构造周围的表达式或声明：`const Fortran::semantics::SubprogramDetails *iface =`。
- **L2625 EN**: Executes a call or declaration centered on `caller.getInterfaceDetails`.
  **L2625 CN**: 执行以 `caller.getInterfaceDetails` 为核心的调用或声明。
- **L2626 EN**: Checks an internal invariant in debug builds.
  **L2626 CN**: 在调试构建中检查内部不变式。
- **L2627 EN**: Executes a standalone statement or declaration: `"depends upon dummy symbols");`.
  **L2627 CN**: 执行一条独立语句或声明：`"depends upon dummy symbols");`。
- **L2628 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2628 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2629 EN**: Continues logic associated with callable symbol `getPassedArguments`.
  **L2629 CN**: 继续与可调用符号 `getPassedArguments` 相关的逻辑。
- **L2630 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2630 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2631 EN**: Initializes variable `dummyType` from the right-hand expression.
  **L2631 CN**: 使用右侧表达式初始化变量 `dummyType`。
- **L2632 EN**: Blank line separating nearby declarations or logic blocks.
  **L2632 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2633 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2633 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2634 EN**: Continues the surrounding expression or declaration: `mlir::Value mockArgValue =`.
  **L2634 CN**: 继续构造周围的表达式或声明：`mlir::Value mockArgValue =`。
- **L2635 EN**: Executes a call or declaration centered on `fir::AbsentOp::create`.
  **L2635 CN**: 执行以 `fir::AbsentOp::create` 为核心的调用或声明。
- **L2636 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L2636 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L2637 EN**: Skips to the next loop iteration.
  **L2637 CN**: 跳到下一次循环迭代。
- **L2638 EN**: Closes the current lexical scope or compound statement.
  **L2638 CN**: 结束当前词法作用域或复合语句块。
- **L2639 EN**: Blank line separating nearby declarations or logic blocks.
  **L2639 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2640 EN**: Continues the surrounding expression or declaration: `Fortran::lower::PreparedActualArgument &preparedActual =`.
  **L2640 CN**: 继续构造周围的表达式或声明：`Fortran::lower::PreparedActualArgument &preparedActual =`。

### Lines 2641-2664

````cpp
              maybePreparedActual.value();

          if (preparedActual.handleDynamicOptional()) {
            mlir::Value isPresent = preparedActual.getIsPresent();
            mlir::Value mockArgValue =
                builder
                    .genIfOp(loc, {dummyType}, isPresent,
                             /*withElseRegion=*/true)
                    .genThen([&]() {
                      mlir::Value mockArgValue =
                          genMockDummyForElementalResultSpecifications(
                              builder, loc, dummyType, preparedActual);
                      fir::ResultOp::create(builder, loc, mockArgValue);
                    })
                    .genElse([&]() {
                      mlir::Value absent =
                          fir::AbsentOp::create(builder, loc, dummyType);
                      fir::ResultOp::create(builder, loc, absent);
                    })
                    .getResults()[0];
            caller.placeInput(arg, mockArgValue);
          } else {
            mlir::Value mockArgValue =
                genMockDummyForElementalResultSpecifications(
````
- **L2641 EN**: Executes a call or declaration centered on `maybePreparedActual.value`.
  **L2641 CN**: 执行以 `maybePreparedActual.value` 为核心的调用或声明。
- **L2642 EN**: Blank line separating nearby declarations or logic blocks.
  **L2642 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2643 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2643 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2644 EN**: Initializes variable `isPresent` from the right-hand expression.
  **L2644 CN**: 使用右侧表达式初始化变量 `isPresent`。
- **L2645 EN**: Continues the surrounding expression or declaration: `mlir::Value mockArgValue =`.
  **L2645 CN**: 继续构造周围的表达式或声明：`mlir::Value mockArgValue =`。
- **L2646 EN**: Continues the surrounding expression or declaration: `builder`.
  **L2646 CN**: 继续构造周围的表达式或声明：`builder`。
- **L2647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.genIfOp(loc, {dummyType}, isPresent,`.
  **L2647 CN**: 继续一个多行参数列表、初始化器或聚合项：`.genIfOp(loc, {dummyType}, isPresent,`。
- **L2648 EN**: Comment explains nearby logic, intent, or metadata: `withElseRegion=*/true)`.
  **L2648 CN**: 注释说明附近代码的逻辑、意图或元数据：`withElseRegion=*/true)`。
- **L2649 EN**: Starts a function, method, lambda, or structured scope: `.genThen([&]() {`.
  **L2649 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genThen([&]() {`。
- **L2650 EN**: Continues the surrounding expression or declaration: `mlir::Value mockArgValue =`.
  **L2650 CN**: 继续构造周围的表达式或声明：`mlir::Value mockArgValue =`。
- **L2651 EN**: Continues logic associated with callable symbol `genMockDummyForElementalResultSpecifications`.
  **L2651 CN**: 继续与可调用符号 `genMockDummyForElementalResultSpecifications` 相关的逻辑。
- **L2652 EN**: Executes a standalone statement or declaration: `builder, loc, dummyType, preparedActual);`.
  **L2652 CN**: 执行一条独立语句或声明：`builder, loc, dummyType, preparedActual);`。
- **L2653 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L2653 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L2654 EN**: Continues the surrounding expression or declaration: `})`.
  **L2654 CN**: 继续构造周围的表达式或声明：`})`。
- **L2655 EN**: Starts a function, method, lambda, or structured scope: `.genElse([&]() {`.
  **L2655 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genElse([&]() {`。
- **L2656 EN**: Continues the surrounding expression or declaration: `mlir::Value absent =`.
  **L2656 CN**: 继续构造周围的表达式或声明：`mlir::Value absent =`。
- **L2657 EN**: Executes a call or declaration centered on `fir::AbsentOp::create`.
  **L2657 CN**: 执行以 `fir::AbsentOp::create` 为核心的调用或声明。
- **L2658 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L2658 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L2659 EN**: Continues the surrounding expression or declaration: `})`.
  **L2659 CN**: 继续构造周围的表达式或声明：`})`。
- **L2660 EN**: Executes a call or declaration centered on `.getResults`.
  **L2660 CN**: 执行以 `.getResults` 为核心的调用或声明。
- **L2661 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L2661 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L2662 EN**: Transitions from the previous branch into the alternative path.
  **L2662 CN**: 从前一个分支过渡到备选路径。
- **L2663 EN**: Continues the surrounding expression or declaration: `mlir::Value mockArgValue =`.
  **L2663 CN**: 继续构造周围的表达式或声明：`mlir::Value mockArgValue =`。
- **L2664 EN**: Continues logic associated with callable symbol `genMockDummyForElementalResultSpecifications`.
  **L2664 CN**: 继续与可调用符号 `genMockDummyForElementalResultSpecifications` 相关的逻辑。

### Lines 2665-2688

````cpp
                    builder, loc, dummyType, preparedActual);
            caller.placeInput(arg, mockArgValue);
          }
        }
    }

    // Map symbols required by the result specification expressions to SSA
    // values. This will both finish mapping the mock value created above if
    // any, and deal with any module/common block variables accessed in the
    // specification expressions.
    // Map prepared argument to dummy symbol to be able to lower spec expr.
    callContext.symMap.pushScope();
    Fortran::lower::mapCallInterfaceSymbolsForResult(converter, caller,
                                                     callContext.symMap);

    // Evaluate the result length expression.
    mlir::Type idxTy = builder.getIndexType();
    auto lowerSpecExpr = [&](const auto &expr) -> mlir::Value {
      mlir::Value convertExpr = builder.createConvert(
          loc, idxTy,
          fir::getBase(converter.genExprValue(expr, callContext.stmtCtx)));
      return fir::factory::genMaxWithZero(builder, loc, convertExpr);
    };

````
- **L2665 EN**: Executes a standalone statement or declaration: `builder, loc, dummyType, preparedActual);`.
  **L2665 CN**: 执行一条独立语句或声明：`builder, loc, dummyType, preparedActual);`。
- **L2666 EN**: Executes a call or declaration centered on `caller.placeInput`.
  **L2666 CN**: 执行以 `caller.placeInput` 为核心的调用或声明。
- **L2667 EN**: Closes the current lexical scope or compound statement.
  **L2667 CN**: 结束当前词法作用域或复合语句块。
- **L2668 EN**: Closes the current lexical scope or compound statement.
  **L2668 CN**: 结束当前词法作用域或复合语句块。
- **L2669 EN**: Closes the current lexical scope or compound statement.
  **L2669 CN**: 结束当前词法作用域或复合语句块。
- **L2670 EN**: Blank line separating nearby declarations or logic blocks.
  **L2670 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2671 EN**: Comment explains nearby logic, intent, or metadata: `Map symbols required by the result specification expressions to SSA`.
  **L2671 CN**: 注释说明附近代码的逻辑、意图或元数据：`Map symbols required by the result specification expressions to SSA`。
- **L2672 EN**: Comment explains nearby logic, intent, or metadata: `values. This will both finish mapping the mock value created above if`.
  **L2672 CN**: 注释说明附近代码的逻辑、意图或元数据：`values. This will both finish mapping the mock value created above if`。
- **L2673 EN**: Comment explains nearby logic, intent, or metadata: `any, and deal with any module/common block variables accessed in the`.
  **L2673 CN**: 注释说明附近代码的逻辑、意图或元数据：`any, and deal with any module/common block variables accessed in the`。
- **L2674 EN**: Comment explains nearby logic, intent, or metadata: `specification expressions.`.
  **L2674 CN**: 注释说明附近代码的逻辑、意图或元数据：`specification expressions.`。
- **L2675 EN**: Comment explains nearby logic, intent, or metadata: `Map prepared argument to dummy symbol to be able to lower spec expr.`.
  **L2675 CN**: 注释说明附近代码的逻辑、意图或元数据：`Map prepared argument to dummy symbol to be able to lower spec expr.`。
- **L2676 EN**: Executes a call or declaration centered on `callContext.symMap.pushScope`.
  **L2676 CN**: 执行以 `callContext.symMap.pushScope` 为核心的调用或声明。
- **L2677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::mapCallInterfaceSymbolsForResult(converter, caller,`.
  **L2677 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::mapCallInterfaceSymbolsForResult(converter, caller,`。
- **L2678 EN**: Executes a standalone statement or declaration: `callContext.symMap);`.
  **L2678 CN**: 执行一条独立语句或声明：`callContext.symMap);`。
- **L2679 EN**: Blank line separating nearby declarations or logic blocks.
  **L2679 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2680 EN**: Comment explains nearby logic, intent, or metadata: `Evaluate the result length expression.`.
  **L2680 CN**: 注释说明附近代码的逻辑、意图或元数据：`Evaluate the result length expression.`。
- **L2681 EN**: Initializes variable `idxTy` from the right-hand expression.
  **L2681 CN**: 使用右侧表达式初始化变量 `idxTy`。
- **L2682 EN**: Starts a function, method, lambda, or structured scope: `auto lowerSpecExpr = [&](const auto &expr) -> mlir::Value {`.
  **L2682 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto lowerSpecExpr = [&](const auto &expr) -> mlir::Value {`。
- **L2683 EN**: Continues logic associated with callable symbol `createConvert`.
  **L2683 CN**: 继续与可调用符号 `createConvert` 相关的逻辑。
- **L2684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, idxTy,`.
  **L2684 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, idxTy,`。
- **L2685 EN**: Executes a call or declaration centered on `fir::getBase`.
  **L2685 CN**: 执行以 `fir::getBase` 为核心的调用或声明。
- **L2686 EN**: Returns from the current function with `fir::factory::genMaxWithZero(builder, loc, convertExpr)`.
  **L2686 CN**: 以 `fir::factory::genMaxWithZero(builder, loc, convertExpr)` 从当前函数返回。
- **L2687 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2687 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2688 EN**: Blank line separating nearby declarations or logic blocks.
  **L2688 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2689-2712

````cpp
    llvm::SmallVector<mlir::Value> lengths;
    caller.walkResultLengths(
        [&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {
          assert(!isAssumedSizeExtent && "result cannot be assumed-size");
          lengths.emplace_back(lowerSpecExpr(e));
        });
    callContext.symMap.popScope();
    assert(lengths.size() == 1 && "expect 1 length parameter for the result");
    return lengths[0];
  }

  mlir::Value getPolymorphicResultMold(
      Fortran::lower::PreparedActualArguments &loweredActuals,
      CallContext &callContext) {
    fir::emitFatalError(callContext.loc,
                        "elemental function call with polymorphic result");
    return {};
  }

  bool resultMayRequireFinalization(CallContext &callContext) const {
    std::optional<Fortran::evaluate::DynamicType> retTy =
        caller.getCallDescription().proc().GetType();
    if (!retTy)
      return false;
````
- **L2689 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> lengths;`.
  **L2689 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> lengths;`。
- **L2690 EN**: Continues logic associated with callable symbol `walkResultLengths`.
  **L2690 CN**: 继续与可调用符号 `walkResultLengths` 相关的逻辑。
- **L2691 EN**: Starts a function, method, lambda, or structured scope: `[&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {`.
  **L2691 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const Fortran::lower::SomeExpr &e, bool isAssumedSizeExtent) {`。
- **L2692 EN**: Checks an internal invariant in debug builds.
  **L2692 CN**: 在调试构建中检查内部不变式。
- **L2693 EN**: Executes a call or declaration centered on `lengths.emplace_back`.
  **L2693 CN**: 执行以 `lengths.emplace_back` 为核心的调用或声明。
- **L2694 EN**: Executes a standalone statement or declaration: `});`.
  **L2694 CN**: 执行一条独立语句或声明：`});`。
- **L2695 EN**: Executes a call or declaration centered on `callContext.symMap.popScope`.
  **L2695 CN**: 执行以 `callContext.symMap.popScope` 为核心的调用或声明。
- **L2696 EN**: Checks an internal invariant in debug builds.
  **L2696 CN**: 在调试构建中检查内部不变式。
- **L2697 EN**: Returns from the current function with `lengths[0]`.
  **L2697 CN**: 以 `lengths[0]` 从当前函数返回。
- **L2698 EN**: Closes the current lexical scope or compound statement.
  **L2698 CN**: 结束当前词法作用域或复合语句块。
- **L2699 EN**: Blank line separating nearby declarations or logic blocks.
  **L2699 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2700 EN**: Continues logic associated with callable symbol `getPolymorphicResultMold`.
  **L2700 CN**: 继续与可调用符号 `getPolymorphicResultMold` 相关的逻辑。
- **L2701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L2701 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L2702 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L2702 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L2703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::emitFatalError(callContext.loc,`.
  **L2703 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::emitFatalError(callContext.loc,`。
- **L2704 EN**: Executes a standalone statement or declaration: `"elemental function call with polymorphic result");`.
  **L2704 CN**: 执行一条独立语句或声明：`"elemental function call with polymorphic result");`。
- **L2705 EN**: Returns from the current function with `{}`.
  **L2705 CN**: 以 `{}` 从当前函数返回。
- **L2706 EN**: Closes the current lexical scope or compound statement.
  **L2706 CN**: 结束当前词法作用域或复合语句块。
- **L2707 EN**: Blank line separating nearby declarations or logic blocks.
  **L2707 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2708 EN**: Starts a function, method, lambda, or structured scope: `bool resultMayRequireFinalization(CallContext &callContext) const {`.
  **L2708 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool resultMayRequireFinalization(CallContext &callContext) const {`。
- **L2709 EN**: Continues the surrounding expression or declaration: `std::optional<Fortran::evaluate::DynamicType> retTy =`.
  **L2709 CN**: 继续构造周围的表达式或声明：`std::optional<Fortran::evaluate::DynamicType> retTy =`。
- **L2710 EN**: Executes a call or declaration centered on `caller.getCallDescription`.
  **L2710 CN**: 执行以 `caller.getCallDescription` 为核心的调用或声明。
- **L2711 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2711 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2712 EN**: Returns from the current function with `false`.
  **L2712 CN**: 以 `false` 从当前函数返回。

### Lines 2713-2736

````cpp

    if (retTy->IsPolymorphic() || retTy->IsUnlimitedPolymorphic())
      fir::emitFatalError(
          callContext.loc,
          "elemental function call with [unlimited-]polymorphic result");

    if (retTy->category() == Fortran::common::TypeCategory::Derived) {
      const Fortran::semantics::DerivedTypeSpec &typeSpec =
          retTy->GetDerivedTypeSpec();
      return Fortran::semantics::IsFinalizable(typeSpec);
    }

    return false;
  }

private:
  Fortran::lower::CallerInterface &caller;
  mlir::FunctionType callSiteType;
};

class ElementalIntrinsicCallBuilder
    : public ElementalCallBuilder<ElementalIntrinsicCallBuilder> {
public:
  ElementalIntrinsicCallBuilder(
````
- **L2713 EN**: Blank line separating nearby declarations or logic blocks.
  **L2713 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2714 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2714 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2715 EN**: Continues logic associated with callable symbol `emitFatalError`.
  **L2715 CN**: 继续与可调用符号 `emitFatalError` 相关的逻辑。
- **L2716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `callContext.loc,`.
  **L2716 CN**: 继续一个多行参数列表、初始化器或聚合项：`callContext.loc,`。
- **L2717 EN**: Executes a standalone statement or declaration: `"elemental function call with [unlimited-]polymorphic result");`.
  **L2717 CN**: 执行一条独立语句或声明：`"elemental function call with [unlimited-]polymorphic result");`。
- **L2718 EN**: Blank line separating nearby declarations or logic blocks.
  **L2718 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2719 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2719 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2720 EN**: Continues the surrounding expression or declaration: `const Fortran::semantics::DerivedTypeSpec &typeSpec =`.
  **L2720 CN**: 继续构造周围的表达式或声明：`const Fortran::semantics::DerivedTypeSpec &typeSpec =`。
- **L2721 EN**: Executes a call or declaration centered on `retTy->GetDerivedTypeSpec`.
  **L2721 CN**: 执行以 `retTy->GetDerivedTypeSpec` 为核心的调用或声明。
- **L2722 EN**: Returns from the current function with `Fortran::semantics::IsFinalizable(typeSpec)`.
  **L2722 CN**: 以 `Fortran::semantics::IsFinalizable(typeSpec)` 从当前函数返回。
- **L2723 EN**: Closes the current lexical scope or compound statement.
  **L2723 CN**: 结束当前词法作用域或复合语句块。
- **L2724 EN**: Blank line separating nearby declarations or logic blocks.
  **L2724 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2725 EN**: Returns from the current function with `false`.
  **L2725 CN**: 以 `false` 从当前函数返回。
- **L2726 EN**: Closes the current lexical scope or compound statement.
  **L2726 CN**: 结束当前词法作用域或复合语句块。
- **L2727 EN**: Blank line separating nearby declarations or logic blocks.
  **L2727 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2728 EN**: Sets the following members to `private` access.
  **L2728 CN**: 将后续成员的访问级别设为 `private`。
- **L2729 EN**: Executes a standalone statement or declaration: `Fortran::lower::CallerInterface &caller;`.
  **L2729 CN**: 执行一条独立语句或声明：`Fortran::lower::CallerInterface &caller;`。
- **L2730 EN**: Executes a standalone statement or declaration: `mlir::FunctionType callSiteType;`.
  **L2730 CN**: 执行一条独立语句或声明：`mlir::FunctionType callSiteType;`。
- **L2731 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2731 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2732 EN**: Blank line separating nearby declarations or logic blocks.
  **L2732 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2733 EN**: Declares class `ElementalIntrinsicCallBuilder`.
  **L2733 CN**: 声明 class `ElementalIntrinsicCallBuilder`。
- **L2734 EN**: Continues the surrounding expression or declaration: `: public ElementalCallBuilder<ElementalIntrinsicCallBuilder> {`.
  **L2734 CN**: 继续构造周围的表达式或声明：`: public ElementalCallBuilder<ElementalIntrinsicCallBuilder> {`。
- **L2735 EN**: Sets the following members to `public` access.
  **L2735 CN**: 将后续成员的访问级别设为 `public`。
- **L2736 EN**: Continues logic associated with callable symbol `ElementalIntrinsicCallBuilder`.
  **L2736 CN**: 继续与可调用符号 `ElementalIntrinsicCallBuilder` 相关的逻辑。

### Lines 2737-2760

````cpp
      const Fortran::evaluate::SpecificIntrinsic *intrinsic,
      const fir::IntrinsicHandlerEntry &intrinsicEntry, bool isFunction)
      : intrinsic{intrinsic}, intrinsicEntry{intrinsicEntry},
        isFunction{isFunction} {}
  std::optional<hlfir::Entity>
  genElementalKernel(Fortran::lower::PreparedActualArguments &loweredActuals,
                     CallContext &callContext) {
    return genHLFIRIntrinsicRefCore(loweredActuals, intrinsic, intrinsicEntry,
                                    callContext);
  }
  // Elemental intrinsic functions cannot modify their arguments.
  bool argMayBeModifiedByCall(int) const { return !isFunction; }
  bool canLoadActualArgumentBeforeLoop(int) const {
    // Elemental intrinsic functions never need the actual addresses
    // of their arguments.
    return isFunction;
  }

  mlir::Value computeDynamicCharacterResultLength(
      Fortran::lower::PreparedActualArguments &loweredActuals,
      CallContext &callContext) {
    if (intrinsic)
      if (intrinsic->name == "adjustr" || intrinsic->name == "adjustl" ||
          intrinsic->name == "merge")
````
- **L2737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::SpecificIntrinsic *intrinsic,`.
  **L2737 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::SpecificIntrinsic *intrinsic,`。
- **L2738 EN**: Continues the surrounding expression or declaration: `const fir::IntrinsicHandlerEntry &intrinsicEntry, bool isFunction)`.
  **L2738 CN**: 继续构造周围的表达式或声明：`const fir::IntrinsicHandlerEntry &intrinsicEntry, bool isFunction)`。
- **L2739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: intrinsic{intrinsic}, intrinsicEntry{intrinsicEntry},`.
  **L2739 CN**: 继续一个多行参数列表、初始化器或聚合项：`: intrinsic{intrinsic}, intrinsicEntry{intrinsicEntry},`。
- **L2740 EN**: Continues the surrounding expression or declaration: `isFunction{isFunction} {}`.
  **L2740 CN**: 继续构造周围的表达式或声明：`isFunction{isFunction} {}`。
- **L2741 EN**: Continues the surrounding expression or declaration: `std::optional<hlfir::Entity>`.
  **L2741 CN**: 继续构造周围的表达式或声明：`std::optional<hlfir::Entity>`。
- **L2742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genElementalKernel(Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L2742 CN**: 继续一个多行参数列表、初始化器或聚合项：`genElementalKernel(Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L2743 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L2743 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L2744 EN**: Returns from the current function with `genHLFIRIntrinsicRefCore(loweredActuals, intrinsic, intrinsicEntry,`.
  **L2744 CN**: 以 `genHLFIRIntrinsicRefCore(loweredActuals, intrinsic, intrinsicEntry,` 从当前函数返回。
- **L2745 EN**: Executes a standalone statement or declaration: `callContext);`.
  **L2745 CN**: 执行一条独立语句或声明：`callContext);`。
- **L2746 EN**: Closes the current lexical scope or compound statement.
  **L2746 CN**: 结束当前词法作用域或复合语句块。
- **L2747 EN**: Comment explains nearby logic, intent, or metadata: `Elemental intrinsic functions cannot modify their arguments.`.
  **L2747 CN**: 注释说明附近代码的逻辑、意图或元数据：`Elemental intrinsic functions cannot modify their arguments.`。
- **L2748 EN**: Continues logic associated with callable symbol `argMayBeModifiedByCall`.
  **L2748 CN**: 继续与可调用符号 `argMayBeModifiedByCall` 相关的逻辑。
- **L2749 EN**: Starts a function, method, lambda, or structured scope: `bool canLoadActualArgumentBeforeLoop(int) const {`.
  **L2749 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool canLoadActualArgumentBeforeLoop(int) const {`。
- **L2750 EN**: Comment explains nearby logic, intent, or metadata: `Elemental intrinsic functions never need the actual addresses`.
  **L2750 CN**: 注释说明附近代码的逻辑、意图或元数据：`Elemental intrinsic functions never need the actual addresses`。
- **L2751 EN**: Comment explains nearby logic, intent, or metadata: `of their arguments.`.
  **L2751 CN**: 注释说明附近代码的逻辑、意图或元数据：`of their arguments.`。
- **L2752 EN**: Returns from the current function with `isFunction`.
  **L2752 CN**: 以 `isFunction` 从当前函数返回。
- **L2753 EN**: Closes the current lexical scope or compound statement.
  **L2753 CN**: 结束当前词法作用域或复合语句块。
- **L2754 EN**: Blank line separating nearby declarations or logic blocks.
  **L2754 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2755 EN**: Continues logic associated with callable symbol `computeDynamicCharacterResultLength`.
  **L2755 CN**: 继续与可调用符号 `computeDynamicCharacterResultLength` 相关的逻辑。
- **L2756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L2756 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L2757 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L2757 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L2758 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2758 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2759 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2759 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2760 EN**: Continues the surrounding expression or declaration: `intrinsic->name == "merge")`.
  **L2760 CN**: 继续构造周围的表达式或声明：`intrinsic->name == "merge")`。

### Lines 2761-2784

````cpp
        return loweredActuals[0].value().genCharLength(
            callContext.loc, callContext.getBuilder());
    // Character MIN/MAX result length is the length of the longest
    // argument that is present.
    assert(intrinsic &&
           (intrinsic->name == "min" || intrinsic->name == "max") &&
           "unexpected elemental intrinsic with character result");
    fir::FirOpBuilder &builder = callContext.getBuilder();
    mlir::Location loc = callContext.loc;
    mlir::Type idxTy = builder.getIndexType();
    mlir::Value resultLength;
    for (auto &preparedActual : loweredActuals) {
      if (!preparedActual)
        continue;
      mlir::Value argLen;
      if (preparedActual->handleDynamicOptional()) {
        // genCharLength must not be called on an absent optional: the
        // descriptor may be unreadable (e.g. assumed-length character).
        // Guard it with a fir.if so the read only happens when present.
        mlir::Value zero = builder.createIntegerConstant(loc, idxTy, 0);
        mlir::Value isPresent = preparedActual->getIsPresent();
        auto &capture = *preparedActual;
        argLen =
            builder
````
- **L2761 EN**: Returns from the current function with `loweredActuals[0].value().genCharLength(`.
  **L2761 CN**: 以 `loweredActuals[0].value().genCharLength(` 从当前函数返回。
- **L2762 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L2762 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L2763 EN**: Comment explains nearby logic, intent, or metadata: `Character MIN/MAX result length is the length of the longest`.
  **L2763 CN**: 注释说明附近代码的逻辑、意图或元数据：`Character MIN/MAX result length is the length of the longest`。
- **L2764 EN**: Comment explains nearby logic, intent, or metadata: `argument that is present.`.
  **L2764 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument that is present.`。
- **L2765 EN**: Checks an internal invariant in debug builds.
  **L2765 CN**: 在调试构建中检查内部不变式。
- **L2766 EN**: Continues the surrounding expression or declaration: `(intrinsic->name == "min" || intrinsic->name == "max") &&`.
  **L2766 CN**: 继续构造周围的表达式或声明：`(intrinsic->name == "min" || intrinsic->name == "max") &&`。
- **L2767 EN**: Executes a standalone statement or declaration: `"unexpected elemental intrinsic with character result");`.
  **L2767 CN**: 执行一条独立语句或声明：`"unexpected elemental intrinsic with character result");`。
- **L2768 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L2768 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L2769 EN**: Initializes variable `loc` from the right-hand expression.
  **L2769 CN**: 使用右侧表达式初始化变量 `loc`。
- **L2770 EN**: Initializes variable `idxTy` from the right-hand expression.
  **L2770 CN**: 使用右侧表达式初始化变量 `idxTy`。
- **L2771 EN**: Executes a standalone statement or declaration: `mlir::Value resultLength;`.
  **L2771 CN**: 执行一条独立语句或声明：`mlir::Value resultLength;`。
- **L2772 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2772 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2773 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2773 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2774 EN**: Skips to the next loop iteration.
  **L2774 CN**: 跳到下一次循环迭代。
- **L2775 EN**: Executes a standalone statement or declaration: `mlir::Value argLen;`.
  **L2775 CN**: 执行一条独立语句或声明：`mlir::Value argLen;`。
- **L2776 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2776 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2777 EN**: Comment explains nearby logic, intent, or metadata: `genCharLength must not be called on an absent optional: the`.
  **L2777 CN**: 注释说明附近代码的逻辑、意图或元数据：`genCharLength must not be called on an absent optional: the`。
- **L2778 EN**: Comment explains nearby logic, intent, or metadata: `descriptor may be unreadable (e.g. assumed-length character).`.
  **L2778 CN**: 注释说明附近代码的逻辑、意图或元数据：`descriptor may be unreadable (e.g. assumed-length character).`。
- **L2779 EN**: Comment explains nearby logic, intent, or metadata: `Guard it with a fir.if so the read only happens when present.`.
  **L2779 CN**: 注释说明附近代码的逻辑、意图或元数据：`Guard it with a fir.if so the read only happens when present.`。
- **L2780 EN**: Initializes variable `zero` from the right-hand expression.
  **L2780 CN**: 使用右侧表达式初始化变量 `zero`。
- **L2781 EN**: Initializes variable `isPresent` from the right-hand expression.
  **L2781 CN**: 使用右侧表达式初始化变量 `isPresent`。
- **L2782 EN**: Executes a standalone statement or declaration: `auto &capture = *preparedActual;`.
  **L2782 CN**: 执行一条独立语句或声明：`auto &capture = *preparedActual;`。
- **L2783 EN**: Continues the surrounding expression or declaration: `argLen =`.
  **L2783 CN**: 继续构造周围的表达式或声明：`argLen =`。
- **L2784 EN**: Continues the surrounding expression or declaration: `builder`.
  **L2784 CN**: 继续构造周围的表达式或声明：`builder`。

### Lines 2785-2808

````cpp
                .genIfOp(loc, {idxTy}, isPresent,
                         /*withElseRegion=*/true)
                .genThen([&]() {
                  mlir::Value len = capture.genCharLength(loc, builder);
                  len = builder.createConvert(loc, idxTy, len);
                  fir::ResultOp::create(builder, loc, len);
                })
                .genElse([&]() { fir::ResultOp::create(builder, loc, zero); })
                .getResults()[0];
      } else {
        argLen = preparedActual->genCharLength(loc, builder);
        argLen = builder.createConvert(loc, idxTy, argLen);
      }
      if (!resultLength) {
        resultLength = argLen;
      } else {
        mlir::Value cmp = mlir::arith::CmpIOp::create(
            builder, loc, mlir::arith::CmpIPredicate::sgt, argLen,
            resultLength);
        resultLength = mlir::arith::SelectOp::create(builder, loc, cmp, argLen,
                                                     resultLength);
      }
    }
    assert(resultLength && "MIN/MAX must have at least two arguments");
````
- **L2785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.genIfOp(loc, {idxTy}, isPresent,`.
  **L2785 CN**: 继续一个多行参数列表、初始化器或聚合项：`.genIfOp(loc, {idxTy}, isPresent,`。
- **L2786 EN**: Comment explains nearby logic, intent, or metadata: `withElseRegion=*/true)`.
  **L2786 CN**: 注释说明附近代码的逻辑、意图或元数据：`withElseRegion=*/true)`。
- **L2787 EN**: Starts a function, method, lambda, or structured scope: `.genThen([&]() {`.
  **L2787 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genThen([&]() {`。
- **L2788 EN**: Initializes variable `len` from the right-hand expression.
  **L2788 CN**: 使用右侧表达式初始化变量 `len`。
- **L2789 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L2789 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L2790 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L2790 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L2791 EN**: Continues the surrounding expression or declaration: `})`.
  **L2791 CN**: 继续构造周围的表达式或声明：`})`。
- **L2792 EN**: Continues logic associated with callable symbol `genElse`.
  **L2792 CN**: 继续与可调用符号 `genElse` 相关的逻辑。
- **L2793 EN**: Executes a call or declaration centered on `.getResults`.
  **L2793 CN**: 执行以 `.getResults` 为核心的调用或声明。
- **L2794 EN**: Transitions from the previous branch into the alternative path.
  **L2794 CN**: 从前一个分支过渡到备选路径。
- **L2795 EN**: Executes a call or declaration centered on `preparedActual->genCharLength`.
  **L2795 CN**: 执行以 `preparedActual->genCharLength` 为核心的调用或声明。
- **L2796 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L2796 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L2797 EN**: Closes the current lexical scope or compound statement.
  **L2797 CN**: 结束当前词法作用域或复合语句块。
- **L2798 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2798 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2799 EN**: Executes a standalone statement or declaration: `resultLength = argLen;`.
  **L2799 CN**: 执行一条独立语句或声明：`resultLength = argLen;`。
- **L2800 EN**: Transitions from the previous branch into the alternative path.
  **L2800 CN**: 从前一个分支过渡到备选路径。
- **L2801 EN**: Continues logic associated with callable symbol `create`.
  **L2801 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L2802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, mlir::arith::CmpIPredicate::sgt, argLen,`.
  **L2802 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, mlir::arith::CmpIPredicate::sgt, argLen,`。
- **L2803 EN**: Executes a standalone statement or declaration: `resultLength);`.
  **L2803 CN**: 执行一条独立语句或声明：`resultLength);`。
- **L2804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `resultLength = mlir::arith::SelectOp::create(builder, loc, cmp, argLen,`.
  **L2804 CN**: 继续一个多行参数列表、初始化器或聚合项：`resultLength = mlir::arith::SelectOp::create(builder, loc, cmp, argLen,`。
- **L2805 EN**: Executes a standalone statement or declaration: `resultLength);`.
  **L2805 CN**: 执行一条独立语句或声明：`resultLength);`。
- **L2806 EN**: Closes the current lexical scope or compound statement.
  **L2806 CN**: 结束当前词法作用域或复合语句块。
- **L2807 EN**: Closes the current lexical scope or compound statement.
  **L2807 CN**: 结束当前词法作用域或复合语句块。
- **L2808 EN**: Checks an internal invariant in debug builds.
  **L2808 CN**: 在调试构建中检查内部不变式。

### Lines 2809-2832

````cpp
    return resultLength;
  }

  mlir::Value getPolymorphicResultMold(
      Fortran::lower::PreparedActualArguments &loweredActuals,
      CallContext &callContext) {
    if (!intrinsic)
      return {};

    if (intrinsic->name == "merge") {
      // MERGE seems to be the only elemental function that can produce
      // polymorphic result. The MERGE's result is polymorphic iff
      // both TSOURCE and FSOURCE are polymorphic, and they also must have
      // the same declared and dynamic types. So any of them can be used
      // for the mold.
      assert(!loweredActuals.empty());
      return loweredActuals.front()->getPolymorphicMold(callContext.loc);
    }

    return {};
  }

  bool resultMayRequireFinalization(
      [[maybe_unused]] CallContext &callContext) const {
````
- **L2809 EN**: Returns from the current function with `resultLength`.
  **L2809 CN**: 以 `resultLength` 从当前函数返回。
- **L2810 EN**: Closes the current lexical scope or compound statement.
  **L2810 CN**: 结束当前词法作用域或复合语句块。
- **L2811 EN**: Blank line separating nearby declarations or logic blocks.
  **L2811 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2812 EN**: Continues logic associated with callable symbol `getPolymorphicResultMold`.
  **L2812 CN**: 继续与可调用符号 `getPolymorphicResultMold` 相关的逻辑。
- **L2813 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L2813 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L2814 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L2814 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L2815 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2815 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2816 EN**: Returns from the current function with `{}`.
  **L2816 CN**: 以 `{}` 从当前函数返回。
- **L2817 EN**: Blank line separating nearby declarations or logic blocks.
  **L2817 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2818 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2818 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2819 EN**: Comment explains nearby logic, intent, or metadata: `MERGE seems to be the only elemental function that can produce`.
  **L2819 CN**: 注释说明附近代码的逻辑、意图或元数据：`MERGE seems to be the only elemental function that can produce`。
- **L2820 EN**: Comment explains nearby logic, intent, or metadata: `polymorphic result. The MERGE's result is polymorphic iff`.
  **L2820 CN**: 注释说明附近代码的逻辑、意图或元数据：`polymorphic result. The MERGE's result is polymorphic iff`。
- **L2821 EN**: Comment explains nearby logic, intent, or metadata: `both TSOURCE and FSOURCE are polymorphic, and they also must have`.
  **L2821 CN**: 注释说明附近代码的逻辑、意图或元数据：`both TSOURCE and FSOURCE are polymorphic, and they also must have`。
- **L2822 EN**: Comment explains nearby logic, intent, or metadata: `the same declared and dynamic types. So any of them can be used`.
  **L2822 CN**: 注释说明附近代码的逻辑、意图或元数据：`the same declared and dynamic types. So any of them can be used`。
- **L2823 EN**: Comment explains nearby logic, intent, or metadata: `for the mold.`.
  **L2823 CN**: 注释说明附近代码的逻辑、意图或元数据：`for the mold.`。
- **L2824 EN**: Checks an internal invariant in debug builds.
  **L2824 CN**: 在调试构建中检查内部不变式。
- **L2825 EN**: Returns from the current function with `loweredActuals.front()->getPolymorphicMold(callContext.loc)`.
  **L2825 CN**: 以 `loweredActuals.front()->getPolymorphicMold(callContext.loc)` 从当前函数返回。
- **L2826 EN**: Closes the current lexical scope or compound statement.
  **L2826 CN**: 结束当前词法作用域或复合语句块。
- **L2827 EN**: Blank line separating nearby declarations or logic blocks.
  **L2827 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2828 EN**: Returns from the current function with `{}`.
  **L2828 CN**: 以 `{}` 从当前函数返回。
- **L2829 EN**: Closes the current lexical scope or compound statement.
  **L2829 CN**: 结束当前词法作用域或复合语句块。
- **L2830 EN**: Blank line separating nearby declarations or logic blocks.
  **L2830 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2831 EN**: Continues logic associated with callable symbol `resultMayRequireFinalization`.
  **L2831 CN**: 继续与可调用符号 `resultMayRequireFinalization` 相关的逻辑。
- **L2832 EN**: Continues the surrounding expression or declaration: `[[maybe_unused]] CallContext &callContext) const {`.
  **L2832 CN**: 继续构造周围的表达式或声明：`[[maybe_unused]] CallContext &callContext) const {`。

### Lines 2833-2856

````cpp
    // FIXME: need access to the CallerInterface's return type
    // to check if the result may need finalization (e.g. the result
    // of MERGE).
    return false;
  }

private:
  const Fortran::evaluate::SpecificIntrinsic *intrinsic;
  fir::IntrinsicHandlerEntry intrinsicEntry;
  const bool isFunction;
};
} // namespace

static std::optional<mlir::Value>
genIsPresentIfArgMaybeAbsent(mlir::Location loc, hlfir::Entity actual,
                             const Fortran::lower::SomeExpr &expr,
                             CallContext &callContext,
                             bool passAsAllocatableOrPointer) {
  if (!Fortran::evaluate::MayBePassedAsAbsentOptional(expr))
    return std::nullopt;
  fir::FirOpBuilder &builder = callContext.getBuilder();
  if (!passAsAllocatableOrPointer &&
      Fortran::evaluate::IsAllocatableOrPointerObject(expr)) {
    // Passing Allocatable/Pointer to non-pointer/non-allocatable OPTIONAL.
````
- **L2833 EN**: Comment records a pending task or caution: `FIXME: need access to the CallerInterface's return type`.
  **L2833 CN**: 注释记录待办事项或注意点：`FIXME: need access to the CallerInterface's return type`。
- **L2834 EN**: Comment explains nearby logic, intent, or metadata: `to check if the result may need finalization (e.g. the result`.
  **L2834 CN**: 注释说明附近代码的逻辑、意图或元数据：`to check if the result may need finalization (e.g. the result`。
- **L2835 EN**: Comment explains nearby logic, intent, or metadata: `of MERGE).`.
  **L2835 CN**: 注释说明附近代码的逻辑、意图或元数据：`of MERGE).`。
- **L2836 EN**: Returns from the current function with `false`.
  **L2836 CN**: 以 `false` 从当前函数返回。
- **L2837 EN**: Closes the current lexical scope or compound statement.
  **L2837 CN**: 结束当前词法作用域或复合语句块。
- **L2838 EN**: Blank line separating nearby declarations or logic blocks.
  **L2838 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2839 EN**: Sets the following members to `private` access.
  **L2839 CN**: 将后续成员的访问级别设为 `private`。
- **L2840 EN**: Executes a standalone statement or declaration: `const Fortran::evaluate::SpecificIntrinsic *intrinsic;`.
  **L2840 CN**: 执行一条独立语句或声明：`const Fortran::evaluate::SpecificIntrinsic *intrinsic;`。
- **L2841 EN**: Executes a standalone statement or declaration: `fir::IntrinsicHandlerEntry intrinsicEntry;`.
  **L2841 CN**: 执行一条独立语句或声明：`fir::IntrinsicHandlerEntry intrinsicEntry;`。
- **L2842 EN**: Executes a standalone statement or declaration: `const bool isFunction;`.
  **L2842 CN**: 执行一条独立语句或声明：`const bool isFunction;`。
- **L2843 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2843 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2844 EN**: Closes a namespace scope with a trailing comment: `} // namespace`.
  **L2844 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L2845 EN**: Blank line separating nearby declarations or logic blocks.
  **L2845 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2846 EN**: Continues the surrounding expression or declaration: `static std::optional<mlir::Value>`.
  **L2846 CN**: 继续构造周围的表达式或声明：`static std::optional<mlir::Value>`。
- **L2847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genIsPresentIfArgMaybeAbsent(mlir::Location loc, hlfir::Entity actual,`.
  **L2847 CN**: 继续一个多行参数列表、初始化器或聚合项：`genIsPresentIfArgMaybeAbsent(mlir::Location loc, hlfir::Entity actual,`。
- **L2848 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::SomeExpr &expr,`.
  **L2848 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::SomeExpr &expr,`。
- **L2849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CallContext &callContext,`.
  **L2849 CN**: 继续一个多行参数列表、初始化器或聚合项：`CallContext &callContext,`。
- **L2850 EN**: Continues the surrounding expression or declaration: `bool passAsAllocatableOrPointer) {`.
  **L2850 CN**: 继续构造周围的表达式或声明：`bool passAsAllocatableOrPointer) {`。
- **L2851 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2851 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2852 EN**: Returns from the current function with `std::nullopt`.
  **L2852 CN**: 以 `std::nullopt` 从当前函数返回。
- **L2853 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L2853 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L2854 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2854 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2855 EN**: Starts a function, method, lambda, or structured scope: `Fortran::evaluate::IsAllocatableOrPointerObject(expr)) {`.
  **L2855 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Fortran::evaluate::IsAllocatableOrPointerObject(expr)) {`。
- **L2856 EN**: Comment explains nearby logic, intent, or metadata: `Passing Allocatable/Pointer to non-pointer/non-allocatable OPTIONAL.`.
  **L2856 CN**: 注释说明附近代码的逻辑、意图或元数据：`Passing Allocatable/Pointer to non-pointer/non-allocatable OPTIONAL.`。

### Lines 2857-2880

````cpp
    // Fortran 2018 15.5.2.12 point 1: If unallocated/disassociated, it is
    // as if the argument was absent. The main care here is to not do a
    // copy-in/copy-out because the temp address, even though pointing to a
    // null size storage, would not be a nullptr and therefore the argument
    // would not be considered absent on the callee side. Note: if the
    // allocatable/pointer is also optional, it cannot be absent as per
    // 15.5.2.12 point 7. and 8. We rely on this to un-conditionally read
    // the allocatable/pointer descriptor here.
    mlir::Value addr = genVariableRawAddress(loc, builder, actual);
    return builder.genIsNotNullAddr(loc, addr);
  }
  // TODO: what if passing allocatable target to optional intent(in) pointer?
  // May fall into the category above if the allocatable is not optional.

  // Passing an optional to an optional.
  return fir::IsPresentOp::create(builder, loc, builder.getI1Type(), actual)
      .getResult();
}

// Lower a reference to an elemental intrinsic procedure with array arguments
// and custom optional handling
static std::optional<hlfir::EntityWithAttributes>
genCustomElementalIntrinsicRef(
    const Fortran::evaluate::SpecificIntrinsic *intrinsic,
````
- **L2857 EN**: Comment explains nearby logic, intent, or metadata: `Fortran 2018 15.5.2.12 point 1: If unallocated/disassociated, it is`.
  **L2857 CN**: 注释说明附近代码的逻辑、意图或元数据：`Fortran 2018 15.5.2.12 point 1: If unallocated/disassociated, it is`。
- **L2858 EN**: Comment explains nearby logic, intent, or metadata: `as if the argument was absent. The main care here is to not do a`.
  **L2858 CN**: 注释说明附近代码的逻辑、意图或元数据：`as if the argument was absent. The main care here is to not do a`。
- **L2859 EN**: Comment explains nearby logic, intent, or metadata: `copy-in/copy-out because the temp address, even though pointing to a`.
  **L2859 CN**: 注释说明附近代码的逻辑、意图或元数据：`copy-in/copy-out because the temp address, even though pointing to a`。
- **L2860 EN**: Comment explains nearby logic, intent, or metadata: `null size storage, would not be a nullptr and therefore the argument`.
  **L2860 CN**: 注释说明附近代码的逻辑、意图或元数据：`null size storage, would not be a nullptr and therefore the argument`。
- **L2861 EN**: Comment explains nearby logic, intent, or metadata: `would not be considered absent on the callee side. Note: if the`.
  **L2861 CN**: 注释说明附近代码的逻辑、意图或元数据：`would not be considered absent on the callee side. Note: if the`。
- **L2862 EN**: Comment explains nearby logic, intent, or metadata: `allocatable/pointer is also optional, it cannot be absent as per`.
  **L2862 CN**: 注释说明附近代码的逻辑、意图或元数据：`allocatable/pointer is also optional, it cannot be absent as per`。
- **L2863 EN**: Comment explains nearby logic, intent, or metadata: `15.5.2.12 point 7. and 8. We rely on this to un-conditionally read`.
  **L2863 CN**: 注释说明附近代码的逻辑、意图或元数据：`15.5.2.12 point 7. and 8. We rely on this to un-conditionally read`。
- **L2864 EN**: Comment explains nearby logic, intent, or metadata: `the allocatable/pointer descriptor here.`.
  **L2864 CN**: 注释说明附近代码的逻辑、意图或元数据：`the allocatable/pointer descriptor here.`。
- **L2865 EN**: Initializes variable `addr` from the right-hand expression.
  **L2865 CN**: 使用右侧表达式初始化变量 `addr`。
- **L2866 EN**: Returns from the current function with `builder.genIsNotNullAddr(loc, addr)`.
  **L2866 CN**: 以 `builder.genIsNotNullAddr(loc, addr)` 从当前函数返回。
- **L2867 EN**: Closes the current lexical scope or compound statement.
  **L2867 CN**: 结束当前词法作用域或复合语句块。
- **L2868 EN**: Comment records a pending task or caution: `TODO: what if passing allocatable target to optional intent(in) pointer?`.
  **L2868 CN**: 注释记录待办事项或注意点：`TODO: what if passing allocatable target to optional intent(in) pointer?`。
- **L2869 EN**: Comment explains nearby logic, intent, or metadata: `May fall into the category above if the allocatable is not optional.`.
  **L2869 CN**: 注释说明附近代码的逻辑、意图或元数据：`May fall into the category above if the allocatable is not optional.`。
- **L2870 EN**: Blank line separating nearby declarations or logic blocks.
  **L2870 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2871 EN**: Comment explains nearby logic, intent, or metadata: `Passing an optional to an optional.`.
  **L2871 CN**: 注释说明附近代码的逻辑、意图或元数据：`Passing an optional to an optional.`。
- **L2872 EN**: Returns from the current function with `fir::IsPresentOp::create(builder, loc, builder.getI1Type(), actual)`.
  **L2872 CN**: 以 `fir::IsPresentOp::create(builder, loc, builder.getI1Type(), actual)` 从当前函数返回。
- **L2873 EN**: Executes a call or declaration centered on `.getResult`.
  **L2873 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L2874 EN**: Closes the current lexical scope or compound statement.
  **L2874 CN**: 结束当前词法作用域或复合语句块。
- **L2875 EN**: Blank line separating nearby declarations or logic blocks.
  **L2875 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2876 EN**: Comment explains nearby logic, intent, or metadata: `Lower a reference to an elemental intrinsic procedure with array arguments`.
  **L2876 CN**: 注释说明附近代码的逻辑、意图或元数据：`Lower a reference to an elemental intrinsic procedure with array arguments`。
- **L2877 EN**: Comment explains nearby logic, intent, or metadata: `and custom optional handling`.
  **L2877 CN**: 注释说明附近代码的逻辑、意图或元数据：`and custom optional handling`。
- **L2878 EN**: Continues the surrounding expression or declaration: `static std::optional<hlfir::EntityWithAttributes>`.
  **L2878 CN**: 继续构造周围的表达式或声明：`static std::optional<hlfir::EntityWithAttributes>`。
- **L2879 EN**: Continues logic associated with callable symbol `genCustomElementalIntrinsicRef`.
  **L2879 CN**: 继续与可调用符号 `genCustomElementalIntrinsicRef` 相关的逻辑。
- **L2880 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::evaluate::SpecificIntrinsic *intrinsic,`.
  **L2880 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::evaluate::SpecificIntrinsic *intrinsic,`。

### Lines 2881-2904

````cpp
    CallContext &callContext) {
  assert(callContext.isElementalProcWithArrayArgs() &&
         "Use genCustomIntrinsicRef for scalar calls");
  mlir::Location loc = callContext.loc;
  auto &converter = callContext.converter;
  Fortran::lower::PreparedActualArguments operands;
  assert(intrinsic && Fortran::lower::intrinsicRequiresCustomOptionalHandling(
                          callContext.procRef, *intrinsic, converter));

  // callback for optional arguments
  auto prepareOptionalArg = [&](const Fortran::lower::SomeExpr &expr) {
    hlfir::EntityWithAttributes actual = Fortran::lower::convertExprToHLFIR(
        loc, converter, expr, callContext.symMap, callContext.stmtCtx);
    std::optional<mlir::Value> isPresent =
        genIsPresentIfArgMaybeAbsent(loc, actual, expr, callContext,
                                     /*passAsAllocatableOrPointer=*/false);
    operands.emplace_back(
        Fortran::lower::PreparedActualArgument{actual, isPresent});
  };

  // callback for non-optional arguments
  auto prepareOtherArg = [&](const Fortran::lower::SomeExpr &expr,
                             fir::LowerIntrinsicArgAs lowerAs) {
    hlfir::EntityWithAttributes actual = Fortran::lower::convertExprToHLFIR(
````
- **L2881 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L2881 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L2882 EN**: Checks an internal invariant in debug builds.
  **L2882 CN**: 在调试构建中检查内部不变式。
- **L2883 EN**: Executes a standalone statement or declaration: `"Use genCustomIntrinsicRef for scalar calls");`.
  **L2883 CN**: 执行一条独立语句或声明：`"Use genCustomIntrinsicRef for scalar calls");`。
- **L2884 EN**: Initializes variable `loc` from the right-hand expression.
  **L2884 CN**: 使用右侧表达式初始化变量 `loc`。
- **L2885 EN**: Executes a standalone statement or declaration: `auto &converter = callContext.converter;`.
  **L2885 CN**: 执行一条独立语句或声明：`auto &converter = callContext.converter;`。
- **L2886 EN**: Executes a standalone statement or declaration: `Fortran::lower::PreparedActualArguments operands;`.
  **L2886 CN**: 执行一条独立语句或声明：`Fortran::lower::PreparedActualArguments operands;`。
- **L2887 EN**: Checks an internal invariant in debug builds.
  **L2887 CN**: 在调试构建中检查内部不变式。
- **L2888 EN**: Executes a standalone statement or declaration: `callContext.procRef, *intrinsic, converter));`.
  **L2888 CN**: 执行一条独立语句或声明：`callContext.procRef, *intrinsic, converter));`。
- **L2889 EN**: Blank line separating nearby declarations or logic blocks.
  **L2889 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2890 EN**: Comment explains nearby logic, intent, or metadata: `callback for optional arguments`.
  **L2890 CN**: 注释说明附近代码的逻辑、意图或元数据：`callback for optional arguments`。
- **L2891 EN**: Starts a function, method, lambda, or structured scope: `auto prepareOptionalArg = [&](const Fortran::lower::SomeExpr &expr) {`.
  **L2891 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto prepareOptionalArg = [&](const Fortran::lower::SomeExpr &expr) {`。
- **L2892 EN**: Continues logic associated with callable symbol `convertExprToHLFIR`.
  **L2892 CN**: 继续与可调用符号 `convertExprToHLFIR` 相关的逻辑。
- **L2893 EN**: Executes a standalone statement or declaration: `loc, converter, expr, callContext.symMap, callContext.stmtCtx);`.
  **L2893 CN**: 执行一条独立语句或声明：`loc, converter, expr, callContext.symMap, callContext.stmtCtx);`。
- **L2894 EN**: Continues the surrounding expression or declaration: `std::optional<mlir::Value> isPresent =`.
  **L2894 CN**: 继续构造周围的表达式或声明：`std::optional<mlir::Value> isPresent =`。
- **L2895 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genIsPresentIfArgMaybeAbsent(loc, actual, expr, callContext,`.
  **L2895 CN**: 继续一个多行参数列表、初始化器或聚合项：`genIsPresentIfArgMaybeAbsent(loc, actual, expr, callContext,`。
- **L2896 EN**: Comment explains nearby logic, intent, or metadata: `passAsAllocatableOrPointer=*/false);`.
  **L2896 CN**: 注释说明附近代码的逻辑、意图或元数据：`passAsAllocatableOrPointer=*/false);`。
- **L2897 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L2897 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L2898 EN**: Executes a standalone statement or declaration: `Fortran::lower::PreparedActualArgument{actual, isPresent});`.
  **L2898 CN**: 执行一条独立语句或声明：`Fortran::lower::PreparedActualArgument{actual, isPresent});`。
- **L2899 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2899 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2900 EN**: Blank line separating nearby declarations or logic blocks.
  **L2900 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2901 EN**: Comment explains nearby logic, intent, or metadata: `callback for non-optional arguments`.
  **L2901 CN**: 注释说明附近代码的逻辑、意图或元数据：`callback for non-optional arguments`。
- **L2902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto prepareOtherArg = [&](const Fortran::lower::SomeExpr &expr,`.
  **L2902 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto prepareOtherArg = [&](const Fortran::lower::SomeExpr &expr,`。
- **L2903 EN**: Continues the surrounding expression or declaration: `fir::LowerIntrinsicArgAs lowerAs) {`.
  **L2903 CN**: 继续构造周围的表达式或声明：`fir::LowerIntrinsicArgAs lowerAs) {`。
- **L2904 EN**: Continues logic associated with callable symbol `convertExprToHLFIR`.
  **L2904 CN**: 继续与可调用符号 `convertExprToHLFIR` 相关的逻辑。

### Lines 2905-2928

````cpp
        loc, converter, expr, callContext.symMap, callContext.stmtCtx);
    operands.emplace_back(Fortran::lower::PreparedActualArgument{
        actual, /*isPresent=*/std::nullopt});
  };

  Fortran::lower::prepareCustomIntrinsicArgument(
      callContext.procRef, *intrinsic, callContext.resultType,
      prepareOptionalArg, prepareOtherArg, converter);

  std::optional<fir::IntrinsicHandlerEntry> intrinsicEntry =
      fir::lookupIntrinsicHandler(callContext.getBuilder(),
                                  callContext.getProcedureName(),
                                  callContext.resultType);
  assert(intrinsicEntry.has_value() &&
         "intrinsic with custom handling for OPTIONAL arguments must have "
         "lowering entries");
  // All of the custom intrinsic elementals with custom handling are pure
  // functions
  return ElementalIntrinsicCallBuilder{intrinsic, *intrinsicEntry,
                                       /*isFunction=*/true}
      .genElementalCall(operands, /*isImpure=*/false, callContext);
}

// Lower a reference to an intrinsic procedure with custom optional handling
````
- **L2905 EN**: Executes a standalone statement or declaration: `loc, converter, expr, callContext.symMap, callContext.stmtCtx);`.
  **L2905 CN**: 执行一条独立语句或声明：`loc, converter, expr, callContext.symMap, callContext.stmtCtx);`。
- **L2906 EN**: Starts a function, method, lambda, or structured scope: `operands.emplace_back(Fortran::lower::PreparedActualArgument{`.
  **L2906 CN**: 开始一个函数、方法、lambda 或结构化作用域：`operands.emplace_back(Fortran::lower::PreparedActualArgument{`。
- **L2907 EN**: Executes a standalone statement or declaration: `actual, /*isPresent=*/std::nullopt});`.
  **L2907 CN**: 执行一条独立语句或声明：`actual, /*isPresent=*/std::nullopt});`。
- **L2908 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2908 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2909 EN**: Blank line separating nearby declarations or logic blocks.
  **L2909 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2910 EN**: Continues logic associated with callable symbol `prepareCustomIntrinsicArgument`.
  **L2910 CN**: 继续与可调用符号 `prepareCustomIntrinsicArgument` 相关的逻辑。
- **L2911 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `callContext.procRef, *intrinsic, callContext.resultType,`.
  **L2911 CN**: 继续一个多行参数列表、初始化器或聚合项：`callContext.procRef, *intrinsic, callContext.resultType,`。
- **L2912 EN**: Executes a standalone statement or declaration: `prepareOptionalArg, prepareOtherArg, converter);`.
  **L2912 CN**: 执行一条独立语句或声明：`prepareOptionalArg, prepareOtherArg, converter);`。
- **L2913 EN**: Blank line separating nearby declarations or logic blocks.
  **L2913 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2914 EN**: Continues the surrounding expression or declaration: `std::optional<fir::IntrinsicHandlerEntry> intrinsicEntry =`.
  **L2914 CN**: 继续构造周围的表达式或声明：`std::optional<fir::IntrinsicHandlerEntry> intrinsicEntry =`。
- **L2915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::lookupIntrinsicHandler(callContext.getBuilder(),`.
  **L2915 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::lookupIntrinsicHandler(callContext.getBuilder(),`。
- **L2916 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `callContext.getProcedureName(),`.
  **L2916 CN**: 继续一个多行参数列表、初始化器或聚合项：`callContext.getProcedureName(),`。
- **L2917 EN**: Executes a standalone statement or declaration: `callContext.resultType);`.
  **L2917 CN**: 执行一条独立语句或声明：`callContext.resultType);`。
- **L2918 EN**: Checks an internal invariant in debug builds.
  **L2918 CN**: 在调试构建中检查内部不变式。
- **L2919 EN**: Continues the surrounding expression or declaration: `"intrinsic with custom handling for OPTIONAL arguments must have "`.
  **L2919 CN**: 继续构造周围的表达式或声明：`"intrinsic with custom handling for OPTIONAL arguments must have "`。
- **L2920 EN**: Executes a standalone statement or declaration: `"lowering entries");`.
  **L2920 CN**: 执行一条独立语句或声明：`"lowering entries");`。
- **L2921 EN**: Comment explains nearby logic, intent, or metadata: `All of the custom intrinsic elementals with custom handling are pure`.
  **L2921 CN**: 注释说明附近代码的逻辑、意图或元数据：`All of the custom intrinsic elementals with custom handling are pure`。
- **L2922 EN**: Comment explains nearby logic, intent, or metadata: `functions`.
  **L2922 CN**: 注释说明附近代码的逻辑、意图或元数据：`functions`。
- **L2923 EN**: Returns from the current function with `ElementalIntrinsicCallBuilder{intrinsic, *intrinsicEntry,`.
  **L2923 CN**: 以 `ElementalIntrinsicCallBuilder{intrinsic, *intrinsicEntry,` 从当前函数返回。
- **L2924 EN**: Comment explains nearby logic, intent, or metadata: `isFunction=*/true}`.
  **L2924 CN**: 注释说明附近代码的逻辑、意图或元数据：`isFunction=*/true}`。
- **L2925 EN**: Executes a call or declaration centered on `.genElementalCall`.
  **L2925 CN**: 执行以 `.genElementalCall` 为核心的调用或声明。
- **L2926 EN**: Closes the current lexical scope or compound statement.
  **L2926 CN**: 结束当前词法作用域或复合语句块。
- **L2927 EN**: Blank line separating nearby declarations or logic blocks.
  **L2927 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2928 EN**: Comment explains nearby logic, intent, or metadata: `Lower a reference to an intrinsic procedure with custom optional handling`.
  **L2928 CN**: 注释说明附近代码的逻辑、意图或元数据：`Lower a reference to an intrinsic procedure with custom optional handling`。

### Lines 2929-2952

````cpp
static std::optional<hlfir::EntityWithAttributes>
genCustomIntrinsicRef(const Fortran::evaluate::SpecificIntrinsic *intrinsic,
                      CallContext &callContext) {
  assert(!callContext.isElementalProcWithArrayArgs() &&
         "Needs to be run through ElementalIntrinsicCallBuilder first");
  mlir::Location loc = callContext.loc;
  fir::FirOpBuilder &builder = callContext.getBuilder();
  auto &converter = callContext.converter;
  auto &stmtCtx = callContext.stmtCtx;
  assert(intrinsic && Fortran::lower::intrinsicRequiresCustomOptionalHandling(
                          callContext.procRef, *intrinsic, converter));
  Fortran::lower::PreparedActualArguments loweredActuals;

  // callback for optional arguments
  auto prepareOptionalArg = [&](const Fortran::lower::SomeExpr &expr) {
    hlfir::EntityWithAttributes actual = Fortran::lower::convertExprToHLFIR(
        loc, converter, expr, callContext.symMap, callContext.stmtCtx);
    mlir::Value isPresent =
        genIsPresentIfArgMaybeAbsent(loc, actual, expr, callContext,
                                     /*passAsAllocatableOrPointer*/ false)
            .value();
    loweredActuals.emplace_back(
        Fortran::lower::PreparedActualArgument{actual, {isPresent}});
  };
````
- **L2929 EN**: Continues the surrounding expression or declaration: `static std::optional<hlfir::EntityWithAttributes>`.
  **L2929 CN**: 继续构造周围的表达式或声明：`static std::optional<hlfir::EntityWithAttributes>`。
- **L2930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genCustomIntrinsicRef(const Fortran::evaluate::SpecificIntrinsic *intrinsic,`.
  **L2930 CN**: 继续一个多行参数列表、初始化器或聚合项：`genCustomIntrinsicRef(const Fortran::evaluate::SpecificIntrinsic *intrinsic,`。
- **L2931 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L2931 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L2932 EN**: Checks an internal invariant in debug builds.
  **L2932 CN**: 在调试构建中检查内部不变式。
- **L2933 EN**: Executes a standalone statement or declaration: `"Needs to be run through ElementalIntrinsicCallBuilder first");`.
  **L2933 CN**: 执行一条独立语句或声明：`"Needs to be run through ElementalIntrinsicCallBuilder first");`。
- **L2934 EN**: Initializes variable `loc` from the right-hand expression.
  **L2934 CN**: 使用右侧表达式初始化变量 `loc`。
- **L2935 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L2935 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L2936 EN**: Executes a standalone statement or declaration: `auto &converter = callContext.converter;`.
  **L2936 CN**: 执行一条独立语句或声明：`auto &converter = callContext.converter;`。
- **L2937 EN**: Executes a standalone statement or declaration: `auto &stmtCtx = callContext.stmtCtx;`.
  **L2937 CN**: 执行一条独立语句或声明：`auto &stmtCtx = callContext.stmtCtx;`。
- **L2938 EN**: Checks an internal invariant in debug builds.
  **L2938 CN**: 在调试构建中检查内部不变式。
- **L2939 EN**: Executes a standalone statement or declaration: `callContext.procRef, *intrinsic, converter));`.
  **L2939 CN**: 执行一条独立语句或声明：`callContext.procRef, *intrinsic, converter));`。
- **L2940 EN**: Executes a standalone statement or declaration: `Fortran::lower::PreparedActualArguments loweredActuals;`.
  **L2940 CN**: 执行一条独立语句或声明：`Fortran::lower::PreparedActualArguments loweredActuals;`。
- **L2941 EN**: Blank line separating nearby declarations or logic blocks.
  **L2941 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2942 EN**: Comment explains nearby logic, intent, or metadata: `callback for optional arguments`.
  **L2942 CN**: 注释说明附近代码的逻辑、意图或元数据：`callback for optional arguments`。
- **L2943 EN**: Starts a function, method, lambda, or structured scope: `auto prepareOptionalArg = [&](const Fortran::lower::SomeExpr &expr) {`.
  **L2943 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto prepareOptionalArg = [&](const Fortran::lower::SomeExpr &expr) {`。
- **L2944 EN**: Continues logic associated with callable symbol `convertExprToHLFIR`.
  **L2944 CN**: 继续与可调用符号 `convertExprToHLFIR` 相关的逻辑。
- **L2945 EN**: Executes a standalone statement or declaration: `loc, converter, expr, callContext.symMap, callContext.stmtCtx);`.
  **L2945 CN**: 执行一条独立语句或声明：`loc, converter, expr, callContext.symMap, callContext.stmtCtx);`。
- **L2946 EN**: Continues the surrounding expression or declaration: `mlir::Value isPresent =`.
  **L2946 CN**: 继续构造周围的表达式或声明：`mlir::Value isPresent =`。
- **L2947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genIsPresentIfArgMaybeAbsent(loc, actual, expr, callContext,`.
  **L2947 CN**: 继续一个多行参数列表、初始化器或聚合项：`genIsPresentIfArgMaybeAbsent(loc, actual, expr, callContext,`。
- **L2948 EN**: Comment explains nearby logic, intent, or metadata: `passAsAllocatableOrPointer*/ false)`.
  **L2948 CN**: 注释说明附近代码的逻辑、意图或元数据：`passAsAllocatableOrPointer*/ false)`。
- **L2949 EN**: Executes a call or declaration centered on `.value`.
  **L2949 CN**: 执行以 `.value` 为核心的调用或声明。
- **L2950 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L2950 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L2951 EN**: Executes a standalone statement or declaration: `Fortran::lower::PreparedActualArgument{actual, {isPresent}});`.
  **L2951 CN**: 执行一条独立语句或声明：`Fortran::lower::PreparedActualArgument{actual, {isPresent}});`。
- **L2952 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2952 CN**: 结束当前声明作用域，例如类、结构体或枚举。

### Lines 2953-2976

````cpp

  // callback for non-optional arguments
  auto prepareOtherArg = [&](const Fortran::lower::SomeExpr &expr,
                             fir::LowerIntrinsicArgAs lowerAs) {
    auto getActualFortranElementType = [&]() -> mlir::Type {
      return hlfir::getFortranElementType(converter.genType(expr));
    };
    hlfir::EntityWithAttributes actual = Fortran::lower::convertExprToHLFIR(
        loc, converter, expr, callContext.symMap, callContext.stmtCtx);
    std::optional<fir::ExtendedValue> exv;
    switch (lowerAs) {
    case fir::LowerIntrinsicArgAs::Value:
      exv = Fortran::lower::convertToValue(loc, converter, actual, stmtCtx);
      break;
    case fir::LowerIntrinsicArgAs::Addr:
      exv = Fortran::lower::convertToAddress(loc, converter, actual, stmtCtx,
                                             getActualFortranElementType());
      break;
    case fir::LowerIntrinsicArgAs::Box:
      exv = Fortran::lower::convertToBox(loc, converter, actual, stmtCtx,
                                         getActualFortranElementType());
      break;
    case fir::LowerIntrinsicArgAs::Inquired:
      exv = Fortran::lower::translateToExtendedValue(loc, builder, actual,
````
- **L2953 EN**: Blank line separating nearby declarations or logic blocks.
  **L2953 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2954 EN**: Comment explains nearby logic, intent, or metadata: `callback for non-optional arguments`.
  **L2954 CN**: 注释说明附近代码的逻辑、意图或元数据：`callback for non-optional arguments`。
- **L2955 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto prepareOtherArg = [&](const Fortran::lower::SomeExpr &expr,`.
  **L2955 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto prepareOtherArg = [&](const Fortran::lower::SomeExpr &expr,`。
- **L2956 EN**: Continues the surrounding expression or declaration: `fir::LowerIntrinsicArgAs lowerAs) {`.
  **L2956 CN**: 继续构造周围的表达式或声明：`fir::LowerIntrinsicArgAs lowerAs) {`。
- **L2957 EN**: Starts a function, method, lambda, or structured scope: `auto getActualFortranElementType = [&]() -> mlir::Type {`.
  **L2957 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto getActualFortranElementType = [&]() -> mlir::Type {`。
- **L2958 EN**: Returns from the current function with `hlfir::getFortranElementType(converter.genType(expr))`.
  **L2958 CN**: 以 `hlfir::getFortranElementType(converter.genType(expr))` 从当前函数返回。
- **L2959 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2959 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2960 EN**: Continues logic associated with callable symbol `convertExprToHLFIR`.
  **L2960 CN**: 继续与可调用符号 `convertExprToHLFIR` 相关的逻辑。
- **L2961 EN**: Executes a standalone statement or declaration: `loc, converter, expr, callContext.symMap, callContext.stmtCtx);`.
  **L2961 CN**: 执行一条独立语句或声明：`loc, converter, expr, callContext.symMap, callContext.stmtCtx);`。
- **L2962 EN**: Executes a standalone statement or declaration: `std::optional<fir::ExtendedValue> exv;`.
  **L2962 CN**: 执行一条独立语句或声明：`std::optional<fir::ExtendedValue> exv;`。
- **L2963 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2963 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2964 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Value:`.
  **L2964 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Value:`。
- **L2965 EN**: Executes a call or declaration centered on `Fortran::lower::convertToValue`.
  **L2965 CN**: 执行以 `Fortran::lower::convertToValue` 为核心的调用或声明。
- **L2966 EN**: Exits the nearest loop or switch statement.
  **L2966 CN**: 退出最近的循环或 switch 语句。
- **L2967 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Addr:`.
  **L2967 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Addr:`。
- **L2968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `exv = Fortran::lower::convertToAddress(loc, converter, actual, stmtCtx,`.
  **L2968 CN**: 继续一个多行参数列表、初始化器或聚合项：`exv = Fortran::lower::convertToAddress(loc, converter, actual, stmtCtx,`。
- **L2969 EN**: Executes a call or declaration centered on `getActualFortranElementType`.
  **L2969 CN**: 执行以 `getActualFortranElementType` 为核心的调用或声明。
- **L2970 EN**: Exits the nearest loop or switch statement.
  **L2970 CN**: 退出最近的循环或 switch 语句。
- **L2971 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Box:`.
  **L2971 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Box:`。
- **L2972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `exv = Fortran::lower::convertToBox(loc, converter, actual, stmtCtx,`.
  **L2972 CN**: 继续一个多行参数列表、初始化器或聚合项：`exv = Fortran::lower::convertToBox(loc, converter, actual, stmtCtx,`。
- **L2973 EN**: Executes a call or declaration centered on `getActualFortranElementType`.
  **L2973 CN**: 执行以 `getActualFortranElementType` 为核心的调用或声明。
- **L2974 EN**: Exits the nearest loop or switch statement.
  **L2974 CN**: 退出最近的循环或 switch 语句。
- **L2975 EN**: Introduces a switch dispatch label: `case fir::LowerIntrinsicArgAs::Inquired:`.
  **L2975 CN**: 引入一个 switch 分发标签：`case fir::LowerIntrinsicArgAs::Inquired:`。
- **L2976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `exv = Fortran::lower::translateToExtendedValue(loc, builder, actual,`.
  **L2976 CN**: 继续一个多行参数列表、初始化器或聚合项：`exv = Fortran::lower::translateToExtendedValue(loc, builder, actual,`。

### Lines 2977-3000

````cpp
                                                     stmtCtx);
      break;
    }
    if (!exv)
      llvm_unreachable("bad switch");
    actual = extendedValueToHlfirEntity(loc, builder, exv.value(),
                                        "tmp.custom_intrinsic_arg");
    loweredActuals.emplace_back(Fortran::lower::PreparedActualArgument{
        actual, /*isPresent=*/std::nullopt});
  };

  Fortran::lower::prepareCustomIntrinsicArgument(
      callContext.procRef, *intrinsic, callContext.resultType,
      prepareOptionalArg, prepareOtherArg, converter);

  return genCustomIntrinsicRefCore(loweredActuals, intrinsic, callContext);
}

/// Lower an intrinsic procedure reference.
/// \p intrinsic is null if this is an intrinsic module procedure that must be
/// lowered as if it were an intrinsic module procedure (like C_LOC which is a
/// procedure from intrinsic module iso_c_binding). Otherwise, \p intrinsic
/// must not be null.

````
- **L2977 EN**: Executes a standalone statement or declaration: `stmtCtx);`.
  **L2977 CN**: 执行一条独立语句或声明：`stmtCtx);`。
- **L2978 EN**: Exits the nearest loop or switch statement.
  **L2978 CN**: 退出最近的循环或 switch 语句。
- **L2979 EN**: Closes the current lexical scope or compound statement.
  **L2979 CN**: 结束当前词法作用域或复合语句块。
- **L2980 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2980 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2981 EN**: Marks this control path as unreachable to LLVM.
  **L2981 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `actual = extendedValueToHlfirEntity(loc, builder, exv.value(),`.
  **L2982 CN**: 继续一个多行参数列表、初始化器或聚合项：`actual = extendedValueToHlfirEntity(loc, builder, exv.value(),`。
- **L2983 EN**: Executes a standalone statement or declaration: `"tmp.custom_intrinsic_arg");`.
  **L2983 CN**: 执行一条独立语句或声明：`"tmp.custom_intrinsic_arg");`。
- **L2984 EN**: Starts a function, method, lambda, or structured scope: `loweredActuals.emplace_back(Fortran::lower::PreparedActualArgument{`.
  **L2984 CN**: 开始一个函数、方法、lambda 或结构化作用域：`loweredActuals.emplace_back(Fortran::lower::PreparedActualArgument{`。
- **L2985 EN**: Executes a standalone statement or declaration: `actual, /*isPresent=*/std::nullopt});`.
  **L2985 CN**: 执行一条独立语句或声明：`actual, /*isPresent=*/std::nullopt});`。
- **L2986 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L2986 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L2987 EN**: Blank line separating nearby declarations or logic blocks.
  **L2987 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2988 EN**: Continues logic associated with callable symbol `prepareCustomIntrinsicArgument`.
  **L2988 CN**: 继续与可调用符号 `prepareCustomIntrinsicArgument` 相关的逻辑。
- **L2989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `callContext.procRef, *intrinsic, callContext.resultType,`.
  **L2989 CN**: 继续一个多行参数列表、初始化器或聚合项：`callContext.procRef, *intrinsic, callContext.resultType,`。
- **L2990 EN**: Executes a standalone statement or declaration: `prepareOptionalArg, prepareOtherArg, converter);`.
  **L2990 CN**: 执行一条独立语句或声明：`prepareOptionalArg, prepareOtherArg, converter);`。
- **L2991 EN**: Blank line separating nearby declarations or logic blocks.
  **L2991 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2992 EN**: Returns from the current function with `genCustomIntrinsicRefCore(loweredActuals, intrinsic, callContext)`.
  **L2992 CN**: 以 `genCustomIntrinsicRefCore(loweredActuals, intrinsic, callContext)` 从当前函数返回。
- **L2993 EN**: Closes the current lexical scope or compound statement.
  **L2993 CN**: 结束当前词法作用域或复合语句块。
- **L2994 EN**: Blank line separating nearby declarations or logic blocks.
  **L2994 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2995 EN**: Comment explains nearby logic, intent, or metadata: `Lower an intrinsic procedure reference.`.
  **L2995 CN**: 注释说明附近代码的逻辑、意图或元数据：`Lower an intrinsic procedure reference.`。
- **L2996 EN**: Comment explains nearby logic, intent, or metadata: `\p intrinsic is null if this is an intrinsic module procedure that must be`.
  **L2996 CN**: 注释说明附近代码的逻辑、意图或元数据：`\p intrinsic is null if this is an intrinsic module procedure that must be`。
- **L2997 EN**: Comment explains nearby logic, intent, or metadata: `lowered as if it were an intrinsic module procedure (like C_LOC which is a`.
  **L2997 CN**: 注释说明附近代码的逻辑、意图或元数据：`lowered as if it were an intrinsic module procedure (like C_LOC which is a`。
- **L2998 EN**: Comment explains nearby logic, intent, or metadata: `procedure from intrinsic module iso_c_binding). Otherwise, \p intrinsic`.
  **L2998 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure from intrinsic module iso_c_binding). Otherwise, \p intrinsic`。
- **L2999 EN**: Comment explains nearby logic, intent, or metadata: `must not be null.`.
  **L2999 CN**: 注释说明附近代码的逻辑、意图或元数据：`must not be null.`。
- **L3000 EN**: Blank line separating nearby declarations or logic blocks.
  **L3000 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3001-3024

````cpp
static std::optional<hlfir::EntityWithAttributes>
genIntrinsicRef(const Fortran::evaluate::SpecificIntrinsic *intrinsic,
                const fir::IntrinsicHandlerEntry &intrinsicEntry,
                CallContext &callContext) {
  mlir::Location loc = callContext.loc;
  Fortran::lower::PreparedActualArguments loweredActuals;
  const fir::IntrinsicArgumentLoweringRules *argLowering =
      intrinsicEntry.getArgumentLoweringRules();
  for (const auto &arg : llvm::enumerate(callContext.procRef.arguments())) {

    if (!arg.value()) {
      // Absent optional.
      loweredActuals.push_back(std::nullopt);
      continue;
    }
    auto *expr =
        Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>(arg.value());
    if (!expr) {
      // TYPE(*) dummy. They are only allowed as argument of a few intrinsics
      // that do not take optional arguments: see Fortran 2018 standard C710.
      const Fortran::evaluate::Symbol *assumedTypeSym =
          arg.value()->GetAssumedTypeDummy();
      if (!assumedTypeSym)
        fir::emitFatalError(loc,
````
- **L3001 EN**: Continues the surrounding expression or declaration: `static std::optional<hlfir::EntityWithAttributes>`.
  **L3001 CN**: 继续构造周围的表达式或声明：`static std::optional<hlfir::EntityWithAttributes>`。
- **L3002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genIntrinsicRef(const Fortran::evaluate::SpecificIntrinsic *intrinsic,`.
  **L3002 CN**: 继续一个多行参数列表、初始化器或聚合项：`genIntrinsicRef(const Fortran::evaluate::SpecificIntrinsic *intrinsic,`。
- **L3003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicHandlerEntry &intrinsicEntry,`.
  **L3003 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicHandlerEntry &intrinsicEntry,`。
- **L3004 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L3004 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L3005 EN**: Initializes variable `loc` from the right-hand expression.
  **L3005 CN**: 使用右侧表达式初始化变量 `loc`。
- **L3006 EN**: Executes a standalone statement or declaration: `Fortran::lower::PreparedActualArguments loweredActuals;`.
  **L3006 CN**: 执行一条独立语句或声明：`Fortran::lower::PreparedActualArguments loweredActuals;`。
- **L3007 EN**: Continues the surrounding expression or declaration: `const fir::IntrinsicArgumentLoweringRules *argLowering =`.
  **L3007 CN**: 继续构造周围的表达式或声明：`const fir::IntrinsicArgumentLoweringRules *argLowering =`。
- **L3008 EN**: Executes a call or declaration centered on `intrinsicEntry.getArgumentLoweringRules`.
  **L3008 CN**: 执行以 `intrinsicEntry.getArgumentLoweringRules` 为核心的调用或声明。
- **L3009 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3009 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3010 EN**: Blank line separating nearby declarations or logic blocks.
  **L3010 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3011 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3011 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3012 EN**: Comment explains nearby logic, intent, or metadata: `Absent optional.`.
  **L3012 CN**: 注释说明附近代码的逻辑、意图或元数据：`Absent optional.`。
- **L3013 EN**: Executes a call or declaration centered on `loweredActuals.push_back`.
  **L3013 CN**: 执行以 `loweredActuals.push_back` 为核心的调用或声明。
- **L3014 EN**: Skips to the next loop iteration.
  **L3014 CN**: 跳到下一次循环迭代。
- **L3015 EN**: Closes the current lexical scope or compound statement.
  **L3015 CN**: 结束当前词法作用域或复合语句块。
- **L3016 EN**: Continues the surrounding expression or declaration: `auto *expr =`.
  **L3016 CN**: 继续构造周围的表达式或声明：`auto *expr =`。
- **L3017 EN**: Executes a call or declaration centered on `Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>`.
  **L3017 CN**: 执行以 `Fortran::evaluate::UnwrapExpr<Fortran::lower::SomeExpr>` 为核心的调用或声明。
- **L3018 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3018 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3019 EN**: Comment explains nearby logic, intent, or metadata: `TYPE(*) dummy. They are only allowed as argument of a few intrinsics`.
  **L3019 CN**: 注释说明附近代码的逻辑、意图或元数据：`TYPE(*) dummy. They are only allowed as argument of a few intrinsics`。
- **L3020 EN**: Comment explains nearby logic, intent, or metadata: `that do not take optional arguments: see Fortran 2018 standard C710.`.
  **L3020 CN**: 注释说明附近代码的逻辑、意图或元数据：`that do not take optional arguments: see Fortran 2018 standard C710.`。
- **L3021 EN**: Continues the surrounding expression or declaration: `const Fortran::evaluate::Symbol *assumedTypeSym =`.
  **L3021 CN**: 继续构造周围的表达式或声明：`const Fortran::evaluate::Symbol *assumedTypeSym =`。
- **L3022 EN**: Executes a call or declaration centered on `arg.value`.
  **L3022 CN**: 执行以 `arg.value` 为核心的调用或声明。
- **L3023 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3023 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::emitFatalError(loc,`.
  **L3024 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::emitFatalError(loc,`。

### Lines 3025-3048

````cpp
                            "expected assumed-type symbol as actual argument");
      std::optional<fir::FortranVariableOpInterface> var =
          callContext.symMap.lookupVariableDefinition(*assumedTypeSym);
      if (!var)
        fir::emitFatalError(loc, "assumed-type symbol was not lowered");
      assert(
          (!argLowering ||
           !fir::lowerIntrinsicArgumentAs(*argLowering, arg.index())
                .handleDynamicOptional) &&
          "TYPE(*) are not expected to appear as optional intrinsic arguments");
      loweredActuals.push_back(Fortran::lower::PreparedActualArgument{
          hlfir::Entity{*var}, /*isPresent=*/std::nullopt});
      continue;
    }
    // arguments of bitwise comparison functions may not have nsw flag
    // even if -fno-wrapv is enabled
    mlir::arith::IntegerOverflowFlags iofBackup{};
    auto isBitwiseComparison = [](const std::string intrinsicName) -> bool {
      if (intrinsicName == "bge" || intrinsicName == "bgt" ||
          intrinsicName == "ble" || intrinsicName == "blt")
        return true;
      return false;
    };
    if (isBitwiseComparison(callContext.getProcedureName())) {
````
- **L3025 EN**: Executes a standalone statement or declaration: `"expected assumed-type symbol as actual argument");`.
  **L3025 CN**: 执行一条独立语句或声明：`"expected assumed-type symbol as actual argument");`。
- **L3026 EN**: Continues the surrounding expression or declaration: `std::optional<fir::FortranVariableOpInterface> var =`.
  **L3026 CN**: 继续构造周围的表达式或声明：`std::optional<fir::FortranVariableOpInterface> var =`。
- **L3027 EN**: Executes a call or declaration centered on `callContext.symMap.lookupVariableDefinition`.
  **L3027 CN**: 执行以 `callContext.symMap.lookupVariableDefinition` 为核心的调用或声明。
- **L3028 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3028 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3029 EN**: Executes a call or declaration centered on `fir::emitFatalError`.
  **L3029 CN**: 执行以 `fir::emitFatalError` 为核心的调用或声明。
- **L3030 EN**: Checks an internal invariant in debug builds.
  **L3030 CN**: 在调试构建中检查内部不变式。
- **L3031 EN**: Continues the surrounding expression or declaration: `(!argLowering ||`.
  **L3031 CN**: 继续构造周围的表达式或声明：`(!argLowering ||`。
- **L3032 EN**: Continues logic associated with callable symbol `lowerIntrinsicArgumentAs`.
  **L3032 CN**: 继续与可调用符号 `lowerIntrinsicArgumentAs` 相关的逻辑。
- **L3033 EN**: Continues the surrounding expression or declaration: `.handleDynamicOptional) &&`.
  **L3033 CN**: 继续构造周围的表达式或声明：`.handleDynamicOptional) &&`。
- **L3034 EN**: Executes a call or declaration centered on `"TYPE`.
  **L3034 CN**: 执行以 `"TYPE` 为核心的调用或声明。
- **L3035 EN**: Starts a function, method, lambda, or structured scope: `loweredActuals.push_back(Fortran::lower::PreparedActualArgument{`.
  **L3035 CN**: 开始一个函数、方法、lambda 或结构化作用域：`loweredActuals.push_back(Fortran::lower::PreparedActualArgument{`。
- **L3036 EN**: Executes a standalone statement or declaration: `hlfir::Entity{*var}, /*isPresent=*/std::nullopt});`.
  **L3036 CN**: 执行一条独立语句或声明：`hlfir::Entity{*var}, /*isPresent=*/std::nullopt});`。
- **L3037 EN**: Skips to the next loop iteration.
  **L3037 CN**: 跳到下一次循环迭代。
- **L3038 EN**: Closes the current lexical scope or compound statement.
  **L3038 CN**: 结束当前词法作用域或复合语句块。
- **L3039 EN**: Comment explains nearby logic, intent, or metadata: `arguments of bitwise comparison functions may not have nsw flag`.
  **L3039 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments of bitwise comparison functions may not have nsw flag`。
- **L3040 EN**: Comment explains nearby logic, intent, or metadata: `even if -fno-wrapv is enabled`.
  **L3040 CN**: 注释说明附近代码的逻辑、意图或元数据：`even if -fno-wrapv is enabled`。
- **L3041 EN**: Executes a standalone statement or declaration: `mlir::arith::IntegerOverflowFlags iofBackup{};`.
  **L3041 CN**: 执行一条独立语句或声明：`mlir::arith::IntegerOverflowFlags iofBackup{};`。
- **L3042 EN**: Starts a function, method, lambda, or structured scope: `auto isBitwiseComparison = [](const std::string intrinsicName) -> bool {`.
  **L3042 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto isBitwiseComparison = [](const std::string intrinsicName) -> bool {`。
- **L3043 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3043 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3044 EN**: Continues the surrounding expression or declaration: `intrinsicName == "ble" || intrinsicName == "blt")`.
  **L3044 CN**: 继续构造周围的表达式或声明：`intrinsicName == "ble" || intrinsicName == "blt")`。
- **L3045 EN**: Returns from the current function with `true`.
  **L3045 CN**: 以 `true` 从当前函数返回。
- **L3046 EN**: Returns from the current function with `false`.
  **L3046 CN**: 以 `false` 从当前函数返回。
- **L3047 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L3047 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L3048 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3048 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3049-3072

````cpp
      iofBackup = callContext.getBuilder().getIntegerOverflowFlags();
      callContext.getBuilder().setIntegerOverflowFlags(
          mlir::arith::IntegerOverflowFlags::none);
    }
    auto loweredActual = Fortran::lower::convertExprToHLFIR(
        loc, callContext.converter, *expr, callContext.symMap,
        callContext.stmtCtx);
    if (isBitwiseComparison(callContext.getProcedureName()))
      callContext.getBuilder().setIntegerOverflowFlags(iofBackup);

    std::optional<mlir::Value> isPresent;
    if (argLowering) {
      fir::ArgLoweringRule argRules =
          fir::lowerIntrinsicArgumentAs(*argLowering, arg.index());
      if (argRules.handleDynamicOptional)
        isPresent =
            genIsPresentIfArgMaybeAbsent(loc, loweredActual, *expr, callContext,
                                         /*passAsAllocatableOrPointer=*/false);
    }
    loweredActuals.push_back(
        Fortran::lower::PreparedActualArgument{loweredActual, isPresent});
  }

  if (callContext.isElementalProcWithArrayArgs()) {
````
- **L3049 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L3049 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L3050 EN**: Continues logic associated with callable symbol `getBuilder`.
  **L3050 CN**: 继续与可调用符号 `getBuilder` 相关的逻辑。
- **L3051 EN**: Executes a standalone statement or declaration: `mlir::arith::IntegerOverflowFlags::none);`.
  **L3051 CN**: 执行一条独立语句或声明：`mlir::arith::IntegerOverflowFlags::none);`。
- **L3052 EN**: Closes the current lexical scope or compound statement.
  **L3052 CN**: 结束当前词法作用域或复合语句块。
- **L3053 EN**: Continues logic associated with callable symbol `convertExprToHLFIR`.
  **L3053 CN**: 继续与可调用符号 `convertExprToHLFIR` 相关的逻辑。
- **L3054 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, callContext.converter, *expr, callContext.symMap,`.
  **L3054 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, callContext.converter, *expr, callContext.symMap,`。
- **L3055 EN**: Executes a standalone statement or declaration: `callContext.stmtCtx);`.
  **L3055 CN**: 执行一条独立语句或声明：`callContext.stmtCtx);`。
- **L3056 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3056 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3057 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L3057 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L3058 EN**: Blank line separating nearby declarations or logic blocks.
  **L3058 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3059 EN**: Executes a standalone statement or declaration: `std::optional<mlir::Value> isPresent;`.
  **L3059 CN**: 执行一条独立语句或声明：`std::optional<mlir::Value> isPresent;`。
- **L3060 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3060 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3061 EN**: Continues the surrounding expression or declaration: `fir::ArgLoweringRule argRules =`.
  **L3061 CN**: 继续构造周围的表达式或声明：`fir::ArgLoweringRule argRules =`。
- **L3062 EN**: Executes a call or declaration centered on `fir::lowerIntrinsicArgumentAs`.
  **L3062 CN**: 执行以 `fir::lowerIntrinsicArgumentAs` 为核心的调用或声明。
- **L3063 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3063 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3064 EN**: Continues the surrounding expression or declaration: `isPresent =`.
  **L3064 CN**: 继续构造周围的表达式或声明：`isPresent =`。
- **L3065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genIsPresentIfArgMaybeAbsent(loc, loweredActual, *expr, callContext,`.
  **L3065 CN**: 继续一个多行参数列表、初始化器或聚合项：`genIsPresentIfArgMaybeAbsent(loc, loweredActual, *expr, callContext,`。
- **L3066 EN**: Comment explains nearby logic, intent, or metadata: `passAsAllocatableOrPointer=*/false);`.
  **L3066 CN**: 注释说明附近代码的逻辑、意图或元数据：`passAsAllocatableOrPointer=*/false);`。
- **L3067 EN**: Closes the current lexical scope or compound statement.
  **L3067 CN**: 结束当前词法作用域或复合语句块。
- **L3068 EN**: Continues logic associated with callable symbol `push_back`.
  **L3068 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L3069 EN**: Executes a standalone statement or declaration: `Fortran::lower::PreparedActualArgument{loweredActual, isPresent});`.
  **L3069 CN**: 执行一条独立语句或声明：`Fortran::lower::PreparedActualArgument{loweredActual, isPresent});`。
- **L3070 EN**: Closes the current lexical scope or compound statement.
  **L3070 CN**: 结束当前词法作用域或复合语句块。
- **L3071 EN**: Blank line separating nearby declarations or logic blocks.
  **L3071 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3072 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3072 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3073-3096

````cpp
    // All intrinsic elemental functions are pure.
    const bool isFunction = callContext.resultType.has_value();
    return ElementalIntrinsicCallBuilder{intrinsic, intrinsicEntry, isFunction}
        .genElementalCall(loweredActuals, /*isImpure=*/!isFunction,
                          callContext);
  }
  std::optional<hlfir::EntityWithAttributes> result = genHLFIRIntrinsicRefCore(
      loweredActuals, intrinsic, intrinsicEntry, callContext);
  if (result && mlir::isa<hlfir::ExprType>(result->getType())) {
    fir::FirOpBuilder *bldr = &callContext.getBuilder();
    callContext.stmtCtx.attachCleanup(
        [=]() { hlfir::DestroyOp::create(*bldr, loc, *result); });
  }
  return result;
}

static std::optional<hlfir::EntityWithAttributes>
genIntrinsicRef(const Fortran::evaluate::SpecificIntrinsic *intrinsic,
                CallContext &callContext) {
  mlir::Location loc = callContext.loc;
  auto &converter = callContext.converter;
  if (intrinsic && Fortran::lower::intrinsicRequiresCustomOptionalHandling(
                       callContext.procRef, *intrinsic, converter)) {
    if (callContext.isElementalProcWithArrayArgs())
````
- **L3073 EN**: Comment explains nearby logic, intent, or metadata: `All intrinsic elemental functions are pure.`.
  **L3073 CN**: 注释说明附近代码的逻辑、意图或元数据：`All intrinsic elemental functions are pure.`。
- **L3074 EN**: Initializes variable `isFunction` from the right-hand expression.
  **L3074 CN**: 使用右侧表达式初始化变量 `isFunction`。
- **L3075 EN**: Returns from the current function with `ElementalIntrinsicCallBuilder{intrinsic, intrinsicEntry, isFunction}`.
  **L3075 CN**: 以 `ElementalIntrinsicCallBuilder{intrinsic, intrinsicEntry, isFunction}` 从当前函数返回。
- **L3076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.genElementalCall(loweredActuals, /*isImpure=*/!isFunction,`.
  **L3076 CN**: 继续一个多行参数列表、初始化器或聚合项：`.genElementalCall(loweredActuals, /*isImpure=*/!isFunction,`。
- **L3077 EN**: Executes a standalone statement or declaration: `callContext);`.
  **L3077 CN**: 执行一条独立语句或声明：`callContext);`。
- **L3078 EN**: Closes the current lexical scope or compound statement.
  **L3078 CN**: 结束当前词法作用域或复合语句块。
- **L3079 EN**: Continues logic associated with callable symbol `genHLFIRIntrinsicRefCore`.
  **L3079 CN**: 继续与可调用符号 `genHLFIRIntrinsicRefCore` 相关的逻辑。
- **L3080 EN**: Executes a standalone statement or declaration: `loweredActuals, intrinsic, intrinsicEntry, callContext);`.
  **L3080 CN**: 执行一条独立语句或声明：`loweredActuals, intrinsic, intrinsicEntry, callContext);`。
- **L3081 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3081 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3082 EN**: Executes a call or declaration centered on `&callContext.getBuilder`.
  **L3082 CN**: 执行以 `&callContext.getBuilder` 为核心的调用或声明。
- **L3083 EN**: Continues logic associated with callable symbol `attachCleanup`.
  **L3083 CN**: 继续与可调用符号 `attachCleanup` 相关的逻辑。
- **L3084 EN**: Executes a call or declaration centered on `[=]`.
  **L3084 CN**: 执行以 `[=]` 为核心的调用或声明。
- **L3085 EN**: Closes the current lexical scope or compound statement.
  **L3085 CN**: 结束当前词法作用域或复合语句块。
- **L3086 EN**: Returns from the current function with `result`.
  **L3086 CN**: 以 `result` 从当前函数返回。
- **L3087 EN**: Closes the current lexical scope or compound statement.
  **L3087 CN**: 结束当前词法作用域或复合语句块。
- **L3088 EN**: Blank line separating nearby declarations or logic blocks.
  **L3088 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3089 EN**: Continues the surrounding expression or declaration: `static std::optional<hlfir::EntityWithAttributes>`.
  **L3089 CN**: 继续构造周围的表达式或声明：`static std::optional<hlfir::EntityWithAttributes>`。
- **L3090 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genIntrinsicRef(const Fortran::evaluate::SpecificIntrinsic *intrinsic,`.
  **L3090 CN**: 继续一个多行参数列表、初始化器或聚合项：`genIntrinsicRef(const Fortran::evaluate::SpecificIntrinsic *intrinsic,`。
- **L3091 EN**: Continues the surrounding expression or declaration: `CallContext &callContext) {`.
  **L3091 CN**: 继续构造周围的表达式或声明：`CallContext &callContext) {`。
- **L3092 EN**: Initializes variable `loc` from the right-hand expression.
  **L3092 CN**: 使用右侧表达式初始化变量 `loc`。
- **L3093 EN**: Executes a standalone statement or declaration: `auto &converter = callContext.converter;`.
  **L3093 CN**: 执行一条独立语句或声明：`auto &converter = callContext.converter;`。
- **L3094 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3094 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3095 EN**: Continues the surrounding expression or declaration: `callContext.procRef, *intrinsic, converter)) {`.
  **L3095 CN**: 继续构造周围的表达式或声明：`callContext.procRef, *intrinsic, converter)) {`。
- **L3096 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3096 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3097-3120

````cpp
      return genCustomElementalIntrinsicRef(intrinsic, callContext);
    return genCustomIntrinsicRef(intrinsic, callContext);
  }
  std::optional<fir::IntrinsicHandlerEntry> intrinsicEntry =
      fir::lookupIntrinsicHandler(callContext.getBuilder(),
                                  callContext.getProcedureName(),
                                  callContext.resultType);
  if (!intrinsicEntry)
    fir::crashOnMissingIntrinsic(loc, callContext.getProcedureName());
  return genIntrinsicRef(intrinsic, *intrinsicEntry, callContext);
}

/// Main entry point to lower procedure references, regardless of what they are.
static std::optional<hlfir::EntityWithAttributes>
genProcedureRef(CallContext &callContext) {
  mlir::Location loc = callContext.loc;
  fir::FirOpBuilder &builder = callContext.getBuilder();
  if (auto *intrinsic = callContext.procRef.proc().GetSpecificIntrinsic())
    return genIntrinsicRef(intrinsic, callContext);
  // Intercept non BIND(C) module procedure reference that have lowering
  // handlers defined for there name. Otherwise, lower them as user
  // procedure calls and expect the implementation to be part of
  // runtime libraries with the proper name mangling.
  if (Fortran::lower::isIntrinsicModuleProcRef(callContext.procRef) &&
````
- **L3097 EN**: Returns from the current function with `genCustomElementalIntrinsicRef(intrinsic, callContext)`.
  **L3097 CN**: 以 `genCustomElementalIntrinsicRef(intrinsic, callContext)` 从当前函数返回。
- **L3098 EN**: Returns from the current function with `genCustomIntrinsicRef(intrinsic, callContext)`.
  **L3098 CN**: 以 `genCustomIntrinsicRef(intrinsic, callContext)` 从当前函数返回。
- **L3099 EN**: Closes the current lexical scope or compound statement.
  **L3099 CN**: 结束当前词法作用域或复合语句块。
- **L3100 EN**: Continues the surrounding expression or declaration: `std::optional<fir::IntrinsicHandlerEntry> intrinsicEntry =`.
  **L3100 CN**: 继续构造周围的表达式或声明：`std::optional<fir::IntrinsicHandlerEntry> intrinsicEntry =`。
- **L3101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::lookupIntrinsicHandler(callContext.getBuilder(),`.
  **L3101 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::lookupIntrinsicHandler(callContext.getBuilder(),`。
- **L3102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `callContext.getProcedureName(),`.
  **L3102 CN**: 继续一个多行参数列表、初始化器或聚合项：`callContext.getProcedureName(),`。
- **L3103 EN**: Executes a standalone statement or declaration: `callContext.resultType);`.
  **L3103 CN**: 执行一条独立语句或声明：`callContext.resultType);`。
- **L3104 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3104 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3105 EN**: Executes a call or declaration centered on `fir::crashOnMissingIntrinsic`.
  **L3105 CN**: 执行以 `fir::crashOnMissingIntrinsic` 为核心的调用或声明。
- **L3106 EN**: Returns from the current function with `genIntrinsicRef(intrinsic, *intrinsicEntry, callContext)`.
  **L3106 CN**: 以 `genIntrinsicRef(intrinsic, *intrinsicEntry, callContext)` 从当前函数返回。
- **L3107 EN**: Closes the current lexical scope or compound statement.
  **L3107 CN**: 结束当前词法作用域或复合语句块。
- **L3108 EN**: Blank line separating nearby declarations or logic blocks.
  **L3108 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3109 EN**: Comment explains nearby logic, intent, or metadata: `Main entry point to lower procedure references, regardless of what they are.`.
  **L3109 CN**: 注释说明附近代码的逻辑、意图或元数据：`Main entry point to lower procedure references, regardless of what they are.`。
- **L3110 EN**: Continues the surrounding expression or declaration: `static std::optional<hlfir::EntityWithAttributes>`.
  **L3110 CN**: 继续构造周围的表达式或声明：`static std::optional<hlfir::EntityWithAttributes>`。
- **L3111 EN**: Starts a function, method, lambda, or structured scope: `genProcedureRef(CallContext &callContext) {`.
  **L3111 CN**: 开始一个函数、方法、lambda 或结构化作用域：`genProcedureRef(CallContext &callContext) {`。
- **L3112 EN**: Initializes variable `loc` from the right-hand expression.
  **L3112 CN**: 使用右侧表达式初始化变量 `loc`。
- **L3113 EN**: Executes a call or declaration centered on `callContext.getBuilder`.
  **L3113 CN**: 执行以 `callContext.getBuilder` 为核心的调用或声明。
- **L3114 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3114 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3115 EN**: Returns from the current function with `genIntrinsicRef(intrinsic, callContext)`.
  **L3115 CN**: 以 `genIntrinsicRef(intrinsic, callContext)` 从当前函数返回。
- **L3116 EN**: Comment explains nearby logic, intent, or metadata: `Intercept non BIND(C) module procedure reference that have lowering`.
  **L3116 CN**: 注释说明附近代码的逻辑、意图或元数据：`Intercept non BIND(C) module procedure reference that have lowering`。
- **L3117 EN**: Comment explains nearby logic, intent, or metadata: `handlers defined for there name. Otherwise, lower them as user`.
  **L3117 CN**: 注释说明附近代码的逻辑、意图或元数据：`handlers defined for there name. Otherwise, lower them as user`。
- **L3118 EN**: Comment explains nearby logic, intent, or metadata: `procedure calls and expect the implementation to be part of`.
  **L3118 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure calls and expect the implementation to be part of`。
- **L3119 EN**: Comment explains nearby logic, intent, or metadata: `runtime libraries with the proper name mangling.`.
  **L3119 CN**: 注释说明附近代码的逻辑、意图或元数据：`runtime libraries with the proper name mangling.`。
- **L3120 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3120 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3121-3144

````cpp
      !callContext.isBindcCall())
    if (std::optional<fir::IntrinsicHandlerEntry> intrinsicEntry =
            fir::lookupIntrinsicHandler(builder, callContext.getProcedureName(),
                                        callContext.resultType))
      return genIntrinsicRef(nullptr, *intrinsicEntry, callContext);

  if (callContext.isStatementFunctionCall())
    return genStmtFunctionRef(loc, callContext.converter, callContext.symMap,
                              callContext.stmtCtx, callContext.procRef);

  Fortran::lower::CallerInterface caller(callContext.procRef,
                                         callContext.converter);
  mlir::FunctionType callSiteType = caller.genFunctionType();
  const bool isElemental = callContext.isElementalProcWithArrayArgs();
  Fortran::lower::PreparedActualArguments loweredActuals;
  // Lower the actual arguments
  for (const Fortran::lower::CallInterface<
           Fortran::lower::CallerInterface>::PassedEntity &arg :
       caller.getPassedArguments())
    if (const auto *actual = arg.entity) {
      const auto *expr = actual->UnwrapExpr();
      if (!expr) {
        // TYPE(*) actual argument.
        const Fortran::evaluate::Symbol *assumedTypeSym =
````
- **L3121 EN**: Continues logic associated with callable symbol `isBindcCall`.
  **L3121 CN**: 继续与可调用符号 `isBindcCall` 相关的逻辑。
- **L3122 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3122 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::lookupIntrinsicHandler(builder, callContext.getProcedureName(),`.
  **L3123 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::lookupIntrinsicHandler(builder, callContext.getProcedureName(),`。
- **L3124 EN**: Continues the surrounding expression or declaration: `callContext.resultType))`.
  **L3124 CN**: 继续构造周围的表达式或声明：`callContext.resultType))`。
- **L3125 EN**: Returns from the current function with `genIntrinsicRef(nullptr, *intrinsicEntry, callContext)`.
  **L3125 CN**: 以 `genIntrinsicRef(nullptr, *intrinsicEntry, callContext)` 从当前函数返回。
- **L3126 EN**: Blank line separating nearby declarations or logic blocks.
  **L3126 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3127 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3127 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3128 EN**: Returns from the current function with `genStmtFunctionRef(loc, callContext.converter, callContext.symMap,`.
  **L3128 CN**: 以 `genStmtFunctionRef(loc, callContext.converter, callContext.symMap,` 从当前函数返回。
- **L3129 EN**: Executes a standalone statement or declaration: `callContext.stmtCtx, callContext.procRef);`.
  **L3129 CN**: 执行一条独立语句或声明：`callContext.stmtCtx, callContext.procRef);`。
- **L3130 EN**: Blank line separating nearby declarations or logic blocks.
  **L3130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::CallerInterface caller(callContext.procRef,`.
  **L3131 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::CallerInterface caller(callContext.procRef,`。
- **L3132 EN**: Executes a standalone statement or declaration: `callContext.converter);`.
  **L3132 CN**: 执行一条独立语句或声明：`callContext.converter);`。
- **L3133 EN**: Initializes variable `callSiteType` from the right-hand expression.
  **L3133 CN**: 使用右侧表达式初始化变量 `callSiteType`。
- **L3134 EN**: Initializes variable `isElemental` from the right-hand expression.
  **L3134 CN**: 使用右侧表达式初始化变量 `isElemental`。
- **L3135 EN**: Executes a standalone statement or declaration: `Fortran::lower::PreparedActualArguments loweredActuals;`.
  **L3135 CN**: 执行一条独立语句或声明：`Fortran::lower::PreparedActualArguments loweredActuals;`。
- **L3136 EN**: Comment explains nearby logic, intent, or metadata: `Lower the actual arguments`.
  **L3136 CN**: 注释说明附近代码的逻辑、意图或元数据：`Lower the actual arguments`。
- **L3137 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L3137 CN**: 开始 `for` 控制流语句并计算其条件。
- **L3138 EN**: Continues the surrounding expression or declaration: `Fortran::lower::CallerInterface>::PassedEntity &arg :`.
  **L3138 CN**: 继续构造周围的表达式或声明：`Fortran::lower::CallerInterface>::PassedEntity &arg :`。
- **L3139 EN**: Continues logic associated with callable symbol `getPassedArguments`.
  **L3139 CN**: 继续与可调用符号 `getPassedArguments` 相关的逻辑。
- **L3140 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3140 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3141 EN**: Executes a call or declaration centered on `actual->UnwrapExpr`.
  **L3141 CN**: 执行以 `actual->UnwrapExpr` 为核心的调用或声明。
- **L3142 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3142 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3143 EN**: Comment explains nearby logic, intent, or metadata: `TYPE(*) actual argument.`.
  **L3143 CN**: 注释说明附近代码的逻辑、意图或元数据：`TYPE(*) actual argument.`。
- **L3144 EN**: Continues the surrounding expression or declaration: `const Fortran::evaluate::Symbol *assumedTypeSym =`.
  **L3144 CN**: 继续构造周围的表达式或声明：`const Fortran::evaluate::Symbol *assumedTypeSym =`。

### Lines 3145-3168

````cpp
            actual->GetAssumedTypeDummy();
        if (!assumedTypeSym)
          fir::emitFatalError(
              loc, "expected assumed-type symbol as actual argument");
        std::optional<fir::FortranVariableOpInterface> var =
            callContext.symMap.lookupVariableDefinition(*assumedTypeSym);
        if (!var)
          fir::emitFatalError(loc, "assumed-type symbol was not lowered");
        hlfir::Entity actual{*var};
        std::optional<mlir::Value> isPresent;
        if (arg.isOptional()) {
          // Passing an optional TYPE(*) to an optional TYPE(*). Note that
          // TYPE(*) cannot be ALLOCATABLE/POINTER (C709) so there is no
          // need to cover the case of passing an ALLOCATABLE/POINTER to an
          // OPTIONAL.
          isPresent = fir::IsPresentOp::create(builder, loc,
                                               builder.getI1Type(), actual)
                          .getResult();
        }
        loweredActuals.push_back(Fortran::lower::PreparedActualArgument{
            hlfir::Entity{*var}, isPresent});
        continue;
      }

````
- **L3145 EN**: Executes a call or declaration centered on `actual->GetAssumedTypeDummy`.
  **L3145 CN**: 执行以 `actual->GetAssumedTypeDummy` 为核心的调用或声明。
- **L3146 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3146 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3147 EN**: Continues logic associated with callable symbol `emitFatalError`.
  **L3147 CN**: 继续与可调用符号 `emitFatalError` 相关的逻辑。
- **L3148 EN**: Executes a standalone statement or declaration: `loc, "expected assumed-type symbol as actual argument");`.
  **L3148 CN**: 执行一条独立语句或声明：`loc, "expected assumed-type symbol as actual argument");`。
- **L3149 EN**: Continues the surrounding expression or declaration: `std::optional<fir::FortranVariableOpInterface> var =`.
  **L3149 CN**: 继续构造周围的表达式或声明：`std::optional<fir::FortranVariableOpInterface> var =`。
- **L3150 EN**: Executes a call or declaration centered on `callContext.symMap.lookupVariableDefinition`.
  **L3150 CN**: 执行以 `callContext.symMap.lookupVariableDefinition` 为核心的调用或声明。
- **L3151 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3151 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3152 EN**: Executes a call or declaration centered on `fir::emitFatalError`.
  **L3152 CN**: 执行以 `fir::emitFatalError` 为核心的调用或声明。
- **L3153 EN**: Executes a standalone statement or declaration: `hlfir::Entity actual{*var};`.
  **L3153 CN**: 执行一条独立语句或声明：`hlfir::Entity actual{*var};`。
- **L3154 EN**: Executes a standalone statement or declaration: `std::optional<mlir::Value> isPresent;`.
  **L3154 CN**: 执行一条独立语句或声明：`std::optional<mlir::Value> isPresent;`。
- **L3155 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3155 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3156 EN**: Comment explains nearby logic, intent, or metadata: `Passing an optional TYPE(*) to an optional TYPE(*). Note that`.
  **L3156 CN**: 注释说明附近代码的逻辑、意图或元数据：`Passing an optional TYPE(*) to an optional TYPE(*). Note that`。
- **L3157 EN**: Comment explains nearby logic, intent, or metadata: `TYPE(*) cannot be ALLOCATABLE/POINTER (C709) so there is no`.
  **L3157 CN**: 注释说明附近代码的逻辑、意图或元数据：`TYPE(*) cannot be ALLOCATABLE/POINTER (C709) so there is no`。
- **L3158 EN**: Comment explains nearby logic, intent, or metadata: `need to cover the case of passing an ALLOCATABLE/POINTER to an`.
  **L3158 CN**: 注释说明附近代码的逻辑、意图或元数据：`need to cover the case of passing an ALLOCATABLE/POINTER to an`。
- **L3159 EN**: Comment explains nearby logic, intent, or metadata: `OPTIONAL.`.
  **L3159 CN**: 注释说明附近代码的逻辑、意图或元数据：`OPTIONAL.`。
- **L3160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `isPresent = fir::IsPresentOp::create(builder, loc,`.
  **L3160 CN**: 继续一个多行参数列表、初始化器或聚合项：`isPresent = fir::IsPresentOp::create(builder, loc,`。
- **L3161 EN**: Continues logic associated with callable symbol `getI1Type`.
  **L3161 CN**: 继续与可调用符号 `getI1Type` 相关的逻辑。
- **L3162 EN**: Executes a call or declaration centered on `.getResult`.
  **L3162 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L3163 EN**: Closes the current lexical scope or compound statement.
  **L3163 CN**: 结束当前词法作用域或复合语句块。
- **L3164 EN**: Starts a function, method, lambda, or structured scope: `loweredActuals.push_back(Fortran::lower::PreparedActualArgument{`.
  **L3164 CN**: 开始一个函数、方法、lambda 或结构化作用域：`loweredActuals.push_back(Fortran::lower::PreparedActualArgument{`。
- **L3165 EN**: Executes a standalone statement or declaration: `hlfir::Entity{*var}, isPresent});`.
  **L3165 CN**: 执行一条独立语句或声明：`hlfir::Entity{*var}, isPresent});`。
- **L3166 EN**: Skips to the next loop iteration.
  **L3166 CN**: 跳到下一次循环迭代。
- **L3167 EN**: Closes the current lexical scope or compound statement.
  **L3167 CN**: 结束当前词法作用域或复合语句块。
- **L3168 EN**: Blank line separating nearby declarations or logic blocks.
  **L3168 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3169-3192

````cpp
      if (Fortran::evaluate::UnwrapExpr<Fortran::evaluate::NullPointer>(
              *expr)) {
        if ((arg.passBy !=
             Fortran::lower::CallerInterface::PassEntityBy::MutableBox) &&
            (arg.passBy !=
             Fortran::lower::CallerInterface::PassEntityBy::BoxProcRef)) {
          assert(
              arg.isOptional() &&
              "NULL must be passed only to pointer, allocatable, or OPTIONAL");
          // Trying to lower NULL() outside of any context would lead to
          // trouble. NULL() here is equivalent to not providing the
          // actual argument.
          loweredActuals.emplace_back(std::nullopt);
          continue;
        }
      }

      if (isElemental && !arg.hasValueAttribute() &&
          Fortran::evaluate::IsVariable(*expr) &&
          Fortran::evaluate::HasVectorSubscript(*expr)) {
        // Vector subscripted arguments are copied in calls, except in elemental
        // calls without VALUE attribute where Fortran 2018 15.5.2.4 point 21
        // does not apply and the address of each element must be passed.
        hlfir::ElementalAddrOp elementalAddr =
````
- **L3169 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3169 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3170 EN**: Comment explains nearby logic, intent, or metadata: `expr)) {`.
  **L3170 CN**: 注释说明附近代码的逻辑、意图或元数据：`expr)) {`。
- **L3171 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3171 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3172 EN**: Continues the surrounding expression or declaration: `Fortran::lower::CallerInterface::PassEntityBy::MutableBox) &&`.
  **L3172 CN**: 继续构造周围的表达式或声明：`Fortran::lower::CallerInterface::PassEntityBy::MutableBox) &&`。
- **L3173 EN**: Continues the surrounding expression or declaration: `(arg.passBy !=`.
  **L3173 CN**: 继续构造周围的表达式或声明：`(arg.passBy !=`。
- **L3174 EN**: Continues the surrounding expression or declaration: `Fortran::lower::CallerInterface::PassEntityBy::BoxProcRef)) {`.
  **L3174 CN**: 继续构造周围的表达式或声明：`Fortran::lower::CallerInterface::PassEntityBy::BoxProcRef)) {`。
- **L3175 EN**: Checks an internal invariant in debug builds.
  **L3175 CN**: 在调试构建中检查内部不变式。
- **L3176 EN**: Continues logic associated with callable symbol `isOptional`.
  **L3176 CN**: 继续与可调用符号 `isOptional` 相关的逻辑。
- **L3177 EN**: Executes a standalone statement or declaration: `"NULL must be passed only to pointer, allocatable, or OPTIONAL");`.
  **L3177 CN**: 执行一条独立语句或声明：`"NULL must be passed only to pointer, allocatable, or OPTIONAL");`。
- **L3178 EN**: Comment explains nearby logic, intent, or metadata: `Trying to lower NULL() outside of any context would lead to`.
  **L3178 CN**: 注释说明附近代码的逻辑、意图或元数据：`Trying to lower NULL() outside of any context would lead to`。
- **L3179 EN**: Comment explains nearby logic, intent, or metadata: `trouble. NULL() here is equivalent to not providing the`.
  **L3179 CN**: 注释说明附近代码的逻辑、意图或元数据：`trouble. NULL() here is equivalent to not providing the`。
- **L3180 EN**: Comment explains nearby logic, intent, or metadata: `actual argument.`.
  **L3180 CN**: 注释说明附近代码的逻辑、意图或元数据：`actual argument.`。
- **L3181 EN**: Executes a call or declaration centered on `loweredActuals.emplace_back`.
  **L3181 CN**: 执行以 `loweredActuals.emplace_back` 为核心的调用或声明。
- **L3182 EN**: Skips to the next loop iteration.
  **L3182 CN**: 跳到下一次循环迭代。
- **L3183 EN**: Closes the current lexical scope or compound statement.
  **L3183 CN**: 结束当前词法作用域或复合语句块。
- **L3184 EN**: Closes the current lexical scope or compound statement.
  **L3184 CN**: 结束当前词法作用域或复合语句块。
- **L3185 EN**: Blank line separating nearby declarations or logic blocks.
  **L3185 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3186 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3186 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3187 EN**: Continues logic associated with callable symbol `IsVariable`.
  **L3187 CN**: 继续与可调用符号 `IsVariable` 相关的逻辑。
- **L3188 EN**: Starts a function, method, lambda, or structured scope: `Fortran::evaluate::HasVectorSubscript(*expr)) {`.
  **L3188 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Fortran::evaluate::HasVectorSubscript(*expr)) {`。
- **L3189 EN**: Comment explains nearby logic, intent, or metadata: `Vector subscripted arguments are copied in calls, except in elemental`.
  **L3189 CN**: 注释说明附近代码的逻辑、意图或元数据：`Vector subscripted arguments are copied in calls, except in elemental`。
- **L3190 EN**: Comment explains nearby logic, intent, or metadata: `calls without VALUE attribute where Fortran 2018 15.5.2.4 point 21`.
  **L3190 CN**: 注释说明附近代码的逻辑、意图或元数据：`calls without VALUE attribute where Fortran 2018 15.5.2.4 point 21`。
- **L3191 EN**: Comment explains nearby logic, intent, or metadata: `does not apply and the address of each element must be passed.`.
  **L3191 CN**: 注释说明附近代码的逻辑、意图或元数据：`does not apply and the address of each element must be passed.`。
- **L3192 EN**: Continues the surrounding expression or declaration: `hlfir::ElementalAddrOp elementalAddr =`.
  **L3192 CN**: 继续构造周围的表达式或声明：`hlfir::ElementalAddrOp elementalAddr =`。

### Lines 3193-3216

````cpp
            Fortran::lower::convertVectorSubscriptedExprToElementalAddr(
                loc, callContext.converter, *expr, callContext.symMap,
                callContext.stmtCtx);
        loweredActuals.emplace_back(
            Fortran::lower::PreparedActualArgument{elementalAddr});
        continue;
      }

      auto loweredActual = Fortran::lower::convertExprToHLFIR(
          loc, callContext.converter, *expr, callContext.symMap,
          callContext.stmtCtx);
      std::optional<mlir::Value> isPresent;
      if (arg.isOptional())
        isPresent = genIsPresentIfArgMaybeAbsent(
            loc, loweredActual, *expr, callContext,
            arg.passBy ==
                Fortran::lower::CallerInterface::PassEntityBy::MutableBox);

      loweredActuals.emplace_back(
          Fortran::lower::PreparedActualArgument{loweredActual, isPresent});
    } else {
      // Optional dummy argument for which there is no actual argument.
      loweredActuals.emplace_back(std::nullopt);
    }
````
- **L3193 EN**: Continues logic associated with callable symbol `convertVectorSubscriptedExprToElementalAddr`.
  **L3193 CN**: 继续与可调用符号 `convertVectorSubscriptedExprToElementalAddr` 相关的逻辑。
- **L3194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, callContext.converter, *expr, callContext.symMap,`.
  **L3194 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, callContext.converter, *expr, callContext.symMap,`。
- **L3195 EN**: Executes a standalone statement or declaration: `callContext.stmtCtx);`.
  **L3195 CN**: 执行一条独立语句或声明：`callContext.stmtCtx);`。
- **L3196 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L3196 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L3197 EN**: Executes a standalone statement or declaration: `Fortran::lower::PreparedActualArgument{elementalAddr});`.
  **L3197 CN**: 执行一条独立语句或声明：`Fortran::lower::PreparedActualArgument{elementalAddr});`。
- **L3198 EN**: Skips to the next loop iteration.
  **L3198 CN**: 跳到下一次循环迭代。
- **L3199 EN**: Closes the current lexical scope or compound statement.
  **L3199 CN**: 结束当前词法作用域或复合语句块。
- **L3200 EN**: Blank line separating nearby declarations or logic blocks.
  **L3200 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3201 EN**: Continues logic associated with callable symbol `convertExprToHLFIR`.
  **L3201 CN**: 继续与可调用符号 `convertExprToHLFIR` 相关的逻辑。
- **L3202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, callContext.converter, *expr, callContext.symMap,`.
  **L3202 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, callContext.converter, *expr, callContext.symMap,`。
- **L3203 EN**: Executes a standalone statement or declaration: `callContext.stmtCtx);`.
  **L3203 CN**: 执行一条独立语句或声明：`callContext.stmtCtx);`。
- **L3204 EN**: Executes a standalone statement or declaration: `std::optional<mlir::Value> isPresent;`.
  **L3204 CN**: 执行一条独立语句或声明：`std::optional<mlir::Value> isPresent;`。
- **L3205 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3205 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3206 EN**: Continues logic associated with callable symbol `genIsPresentIfArgMaybeAbsent`.
  **L3206 CN**: 继续与可调用符号 `genIsPresentIfArgMaybeAbsent` 相关的逻辑。
- **L3207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, loweredActual, *expr, callContext,`.
  **L3207 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, loweredActual, *expr, callContext,`。
- **L3208 EN**: Continues the surrounding expression or declaration: `arg.passBy ==`.
  **L3208 CN**: 继续构造周围的表达式或声明：`arg.passBy ==`。
- **L3209 EN**: Executes a standalone statement or declaration: `Fortran::lower::CallerInterface::PassEntityBy::MutableBox);`.
  **L3209 CN**: 执行一条独立语句或声明：`Fortran::lower::CallerInterface::PassEntityBy::MutableBox);`。
- **L3210 EN**: Blank line separating nearby declarations or logic blocks.
  **L3210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3211 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L3211 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L3212 EN**: Executes a standalone statement or declaration: `Fortran::lower::PreparedActualArgument{loweredActual, isPresent});`.
  **L3212 CN**: 执行一条独立语句或声明：`Fortran::lower::PreparedActualArgument{loweredActual, isPresent});`。
- **L3213 EN**: Transitions from the previous branch into the alternative path.
  **L3213 CN**: 从前一个分支过渡到备选路径。
- **L3214 EN**: Comment explains nearby logic, intent, or metadata: `Optional dummy argument for which there is no actual argument.`.
  **L3214 CN**: 注释说明附近代码的逻辑、意图或元数据：`Optional dummy argument for which there is no actual argument.`。
- **L3215 EN**: Executes a call or declaration centered on `loweredActuals.emplace_back`.
  **L3215 CN**: 执行以 `loweredActuals.emplace_back` 为核心的调用或声明。
- **L3216 EN**: Closes the current lexical scope or compound statement.
  **L3216 CN**: 结束当前词法作用域或复合语句块。

### Lines 3217-3240

````cpp
  if (isElemental) {
    bool isImpure = false;
    if (const Fortran::semantics::Symbol *procSym =
            callContext.procRef.proc().GetSymbol())
      isImpure = !Fortran::semantics::IsPureProcedure(*procSym);
    return ElementalUserCallBuilder{caller, callSiteType}.genElementalCall(
        loweredActuals, isImpure, callContext);
  }
  return genUserCall(loweredActuals, caller, callSiteType, callContext);
}

hlfir::Entity Fortran::lower::PreparedActualArgument::getActual(
    mlir::Location loc, fir::FirOpBuilder &builder) const {
  if (auto *actualEntity = std::get_if<hlfir::Entity>(&actual)) {
    if (oneBasedElementalIndices)
      return hlfir::getElementAt(loc, builder, *actualEntity,
                                 *oneBasedElementalIndices);
    return *actualEntity;
  }
  assert(oneBasedElementalIndices && "expect elemental context");
  hlfir::ElementalAddrOp elementalAddr =
      std::get<hlfir::ElementalAddrOp>(actual);
  mlir::IRMapping mapper;
  auto alwaysFalse = [](hlfir::ElementalOp) -> bool { return false; };
````
- **L3217 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3217 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3218 EN**: Initializes variable `isImpure` from the right-hand expression.
  **L3218 CN**: 使用右侧表达式初始化变量 `isImpure`。
- **L3219 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3219 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3220 EN**: Continues logic associated with callable symbol `proc`.
  **L3220 CN**: 继续与可调用符号 `proc` 相关的逻辑。
- **L3221 EN**: Executes a call or declaration centered on `!Fortran::semantics::IsPureProcedure`.
  **L3221 CN**: 执行以 `!Fortran::semantics::IsPureProcedure` 为核心的调用或声明。
- **L3222 EN**: Returns from the current function with `ElementalUserCallBuilder{caller, callSiteType}.genElementalCall(`.
  **L3222 CN**: 以 `ElementalUserCallBuilder{caller, callSiteType}.genElementalCall(` 从当前函数返回。
- **L3223 EN**: Executes a standalone statement or declaration: `loweredActuals, isImpure, callContext);`.
  **L3223 CN**: 执行一条独立语句或声明：`loweredActuals, isImpure, callContext);`。
- **L3224 EN**: Closes the current lexical scope or compound statement.
  **L3224 CN**: 结束当前词法作用域或复合语句块。
- **L3225 EN**: Returns from the current function with `genUserCall(loweredActuals, caller, callSiteType, callContext)`.
  **L3225 CN**: 以 `genUserCall(loweredActuals, caller, callSiteType, callContext)` 从当前函数返回。
- **L3226 EN**: Closes the current lexical scope or compound statement.
  **L3226 CN**: 结束当前词法作用域或复合语句块。
- **L3227 EN**: Blank line separating nearby declarations or logic blocks.
  **L3227 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3228 EN**: Continues logic associated with callable symbol `getActual`.
  **L3228 CN**: 继续与可调用符号 `getActual` 相关的逻辑。
- **L3229 EN**: Continues the surrounding expression or declaration: `mlir::Location loc, fir::FirOpBuilder &builder) const {`.
  **L3229 CN**: 继续构造周围的表达式或声明：`mlir::Location loc, fir::FirOpBuilder &builder) const {`。
- **L3230 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3230 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3231 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3231 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3232 EN**: Returns from the current function with `hlfir::getElementAt(loc, builder, *actualEntity,`.
  **L3232 CN**: 以 `hlfir::getElementAt(loc, builder, *actualEntity,` 从当前函数返回。
- **L3233 EN**: Comment explains nearby logic, intent, or metadata: `oneBasedElementalIndices);`.
  **L3233 CN**: 注释说明附近代码的逻辑、意图或元数据：`oneBasedElementalIndices);`。
- **L3234 EN**: Returns from the current function with `*actualEntity`.
  **L3234 CN**: 以 `*actualEntity` 从当前函数返回。
- **L3235 EN**: Closes the current lexical scope or compound statement.
  **L3235 CN**: 结束当前词法作用域或复合语句块。
- **L3236 EN**: Checks an internal invariant in debug builds.
  **L3236 CN**: 在调试构建中检查内部不变式。
- **L3237 EN**: Continues the surrounding expression or declaration: `hlfir::ElementalAddrOp elementalAddr =`.
  **L3237 CN**: 继续构造周围的表达式或声明：`hlfir::ElementalAddrOp elementalAddr =`。
- **L3238 EN**: Executes a call or declaration centered on `std::get<hlfir::ElementalAddrOp>`.
  **L3238 CN**: 执行以 `std::get<hlfir::ElementalAddrOp>` 为核心的调用或声明。
- **L3239 EN**: Executes a standalone statement or declaration: `mlir::IRMapping mapper;`.
  **L3239 CN**: 执行一条独立语句或声明：`mlir::IRMapping mapper;`。
- **L3240 EN**: Initializes variable `alwaysFalse` from the right-hand expression.
  **L3240 CN**: 使用右侧表达式初始化变量 `alwaysFalse`。

### Lines 3241-3264

````cpp
  mlir::Value addr = hlfir::inlineElementalOp(
      loc, builder, elementalAddr, *oneBasedElementalIndices, mapper,
      /*mustRecursivelyInline=*/alwaysFalse);
  assert(elementalAddr.getCleanup().empty() && "no clean-up expected");
  elementalAddr.erase();
  return hlfir::Entity{addr};
}

bool Fortran::lower::isIntrinsicModuleProcRef(
    const Fortran::evaluate::ProcedureRef &procRef) {
  const Fortran::semantics::Symbol *symbol = procRef.proc().GetSymbol();
  if (!symbol)
    return false;
  const Fortran::semantics::Symbol *module =
      symbol->GetUltimate().owner().GetSymbol();
  return module && module->attrs().test(Fortran::semantics::Attr::INTRINSIC);
}

std::optional<hlfir::EntityWithAttributes> Fortran::lower::convertCallToHLFIR(
    mlir::Location loc, Fortran::lower::AbstractConverter &converter,
    const evaluate::ProcedureRef &procRef, std::optional<mlir::Type> resultType,
    Fortran::lower::SymMap &symMap, Fortran::lower::StatementContext &stmtCtx) {
  auto &builder = converter.getFirOpBuilder();
  if (resultType && !procRef.IsElemental() &&
````
- **L3241 EN**: Continues logic associated with callable symbol `inlineElementalOp`.
  **L3241 CN**: 继续与可调用符号 `inlineElementalOp` 相关的逻辑。
- **L3242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, builder, elementalAddr, *oneBasedElementalIndices, mapper,`.
  **L3242 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, builder, elementalAddr, *oneBasedElementalIndices, mapper,`。
- **L3243 EN**: Comment explains nearby logic, intent, or metadata: `mustRecursivelyInline=*/alwaysFalse);`.
  **L3243 CN**: 注释说明附近代码的逻辑、意图或元数据：`mustRecursivelyInline=*/alwaysFalse);`。
- **L3244 EN**: Checks an internal invariant in debug builds.
  **L3244 CN**: 在调试构建中检查内部不变式。
- **L3245 EN**: Executes a call or declaration centered on `elementalAddr.erase`.
  **L3245 CN**: 执行以 `elementalAddr.erase` 为核心的调用或声明。
- **L3246 EN**: Returns from the current function with `hlfir::Entity{addr}`.
  **L3246 CN**: 以 `hlfir::Entity{addr}` 从当前函数返回。
- **L3247 EN**: Closes the current lexical scope or compound statement.
  **L3247 CN**: 结束当前词法作用域或复合语句块。
- **L3248 EN**: Blank line separating nearby declarations or logic blocks.
  **L3248 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3249 EN**: Continues logic associated with callable symbol `isIntrinsicModuleProcRef`.
  **L3249 CN**: 继续与可调用符号 `isIntrinsicModuleProcRef` 相关的逻辑。
- **L3250 EN**: Continues the surrounding expression or declaration: `const Fortran::evaluate::ProcedureRef &procRef) {`.
  **L3250 CN**: 继续构造周围的表达式或声明：`const Fortran::evaluate::ProcedureRef &procRef) {`。
- **L3251 EN**: Executes a call or declaration centered on `procRef.proc`.
  **L3251 CN**: 执行以 `procRef.proc` 为核心的调用或声明。
- **L3252 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3252 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3253 EN**: Returns from the current function with `false`.
  **L3253 CN**: 以 `false` 从当前函数返回。
- **L3254 EN**: Continues the surrounding expression or declaration: `const Fortran::semantics::Symbol *module =`.
  **L3254 CN**: 继续构造周围的表达式或声明：`const Fortran::semantics::Symbol *module =`。
- **L3255 EN**: Executes a call or declaration centered on `symbol->GetUltimate`.
  **L3255 CN**: 执行以 `symbol->GetUltimate` 为核心的调用或声明。
- **L3256 EN**: Returns from the current function with `module && module->attrs().test(Fortran::semantics::Attr::INTRINSIC)`.
  **L3256 CN**: 以 `module && module->attrs().test(Fortran::semantics::Attr::INTRINSIC)` 从当前函数返回。
- **L3257 EN**: Closes the current lexical scope or compound statement.
  **L3257 CN**: 结束当前词法作用域或复合语句块。
- **L3258 EN**: Blank line separating nearby declarations or logic blocks.
  **L3258 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3259 EN**: Continues logic associated with callable symbol `convertCallToHLFIR`.
  **L3259 CN**: 继续与可调用符号 `convertCallToHLFIR` 相关的逻辑。
- **L3260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, Fortran::lower::AbstractConverter &converter,`.
  **L3260 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, Fortran::lower::AbstractConverter &converter,`。
- **L3261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const evaluate::ProcedureRef &procRef, std::optional<mlir::Type> resultType,`.
  **L3261 CN**: 继续一个多行参数列表、初始化器或聚合项：`const evaluate::ProcedureRef &procRef, std::optional<mlir::Type> resultType,`。
- **L3262 EN**: Continues the surrounding expression or declaration: `Fortran::lower::SymMap &symMap, Fortran::lower::StatementContext &stmtCtx) {`.
  **L3262 CN**: 继续构造周围的表达式或声明：`Fortran::lower::SymMap &symMap, Fortran::lower::StatementContext &stmtCtx) {`。
- **L3263 EN**: Executes a call or declaration centered on `converter.getFirOpBuilder`.
  **L3263 CN**: 执行以 `converter.getFirOpBuilder` 为核心的调用或声明。
- **L3264 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3264 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 3265-3288

````cpp
      hlfir::isInsideHlfirWhereMaskedExpression(builder.getRegion()) &&
      !builder.getRegion().getParentOfType<hlfir::ExactlyOnceOp>()) {
    // Non elemental calls inside a where-assignment-stmt must be executed
    // exactly once without mask control. Lower them in a special region so that
    // this can be enforced whenscheduling forall/where expression evaluations.
    Fortran::lower::StatementContext localStmtCtx;
    mlir::Type bogusType = builder.getIndexType();
    auto exactlyOnce = hlfir::ExactlyOnceOp::create(builder, loc, bogusType);
    mlir::Block *block = builder.createBlock(&exactlyOnce.getBody());
    builder.setInsertionPointToStart(block);
    CallContext callContext(procRef, resultType, loc, converter, symMap,
                            localStmtCtx);
    std::optional<hlfir::EntityWithAttributes> res =
        genProcedureRef(callContext);
    assert(res.has_value() && "must be a function");
    auto yield = hlfir::YieldOp::create(builder, loc, *res);
    Fortran::lower::genCleanUpInRegionIfAny(loc, builder, yield.getCleanup(),
                                            localStmtCtx);
    builder.setInsertionPointAfter(exactlyOnce);
    exactlyOnce->getResult(0).setType(res->getType());
    if (hlfir::isFortranValue(exactlyOnce.getResult()))
      return hlfir::EntityWithAttributes{exactlyOnce.getResult()};
    // Create hlfir.declare for the result to satisfy
    // hlfir::EntityWithAttributes requirements.
````
- **L3265 EN**: Continues logic associated with callable symbol `isInsideHlfirWhereMaskedExpression`.
  **L3265 CN**: 继续与可调用符号 `isInsideHlfirWhereMaskedExpression` 相关的逻辑。
- **L3266 EN**: Starts a function, method, lambda, or structured scope: `!builder.getRegion().getParentOfType<hlfir::ExactlyOnceOp>()) {`.
  **L3266 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!builder.getRegion().getParentOfType<hlfir::ExactlyOnceOp>()) {`。
- **L3267 EN**: Comment explains nearby logic, intent, or metadata: `Non elemental calls inside a where-assignment-stmt must be executed`.
  **L3267 CN**: 注释说明附近代码的逻辑、意图或元数据：`Non elemental calls inside a where-assignment-stmt must be executed`。
- **L3268 EN**: Comment explains nearby logic, intent, or metadata: `exactly once without mask control. Lower them in a special region so that`.
  **L3268 CN**: 注释说明附近代码的逻辑、意图或元数据：`exactly once without mask control. Lower them in a special region so that`。
- **L3269 EN**: Comment explains nearby logic, intent, or metadata: `this can be enforced whenscheduling forall/where expression evaluations.`.
  **L3269 CN**: 注释说明附近代码的逻辑、意图或元数据：`this can be enforced whenscheduling forall/where expression evaluations.`。
- **L3270 EN**: Executes a standalone statement or declaration: `Fortran::lower::StatementContext localStmtCtx;`.
  **L3270 CN**: 执行一条独立语句或声明：`Fortran::lower::StatementContext localStmtCtx;`。
- **L3271 EN**: Initializes variable `bogusType` from the right-hand expression.
  **L3271 CN**: 使用右侧表达式初始化变量 `bogusType`。
- **L3272 EN**: Initializes variable `exactlyOnce` from the right-hand expression.
  **L3272 CN**: 使用右侧表达式初始化变量 `exactlyOnce`。
- **L3273 EN**: Executes a call or declaration centered on `builder.createBlock`.
  **L3273 CN**: 执行以 `builder.createBlock` 为核心的调用或声明。
- **L3274 EN**: Executes a call or declaration centered on `builder.setInsertionPointToStart`.
  **L3274 CN**: 执行以 `builder.setInsertionPointToStart` 为核心的调用或声明。
- **L3275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CallContext callContext(procRef, resultType, loc, converter, symMap,`.
  **L3275 CN**: 继续一个多行参数列表、初始化器或聚合项：`CallContext callContext(procRef, resultType, loc, converter, symMap,`。
- **L3276 EN**: Executes a standalone statement or declaration: `localStmtCtx);`.
  **L3276 CN**: 执行一条独立语句或声明：`localStmtCtx);`。
- **L3277 EN**: Continues the surrounding expression or declaration: `std::optional<hlfir::EntityWithAttributes> res =`.
  **L3277 CN**: 继续构造周围的表达式或声明：`std::optional<hlfir::EntityWithAttributes> res =`。
- **L3278 EN**: Executes a call or declaration centered on `genProcedureRef`.
  **L3278 CN**: 执行以 `genProcedureRef` 为核心的调用或声明。
- **L3279 EN**: Checks an internal invariant in debug builds.
  **L3279 CN**: 在调试构建中检查内部不变式。
- **L3280 EN**: Initializes variable `yield` from the right-hand expression.
  **L3280 CN**: 使用右侧表达式初始化变量 `yield`。
- **L3281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::lower::genCleanUpInRegionIfAny(loc, builder, yield.getCleanup(),`.
  **L3281 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::lower::genCleanUpInRegionIfAny(loc, builder, yield.getCleanup(),`。
- **L3282 EN**: Executes a standalone statement or declaration: `localStmtCtx);`.
  **L3282 CN**: 执行一条独立语句或声明：`localStmtCtx);`。
- **L3283 EN**: Executes a call or declaration centered on `builder.setInsertionPointAfter`.
  **L3283 CN**: 执行以 `builder.setInsertionPointAfter` 为核心的调用或声明。
- **L3284 EN**: Executes a call or declaration centered on `exactlyOnce->getResult`.
  **L3284 CN**: 执行以 `exactlyOnce->getResult` 为核心的调用或声明。
- **L3285 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L3285 CN**: 开始 `if` 控制流语句并计算其条件。
- **L3286 EN**: Returns from the current function with `hlfir::EntityWithAttributes{exactlyOnce.getResult()}`.
  **L3286 CN**: 以 `hlfir::EntityWithAttributes{exactlyOnce.getResult()}` 从当前函数返回。
- **L3287 EN**: Comment explains nearby logic, intent, or metadata: `Create hlfir.declare for the result to satisfy`.
  **L3287 CN**: 注释说明附近代码的逻辑、意图或元数据：`Create hlfir.declare for the result to satisfy`。
- **L3288 EN**: Comment explains nearby logic, intent, or metadata: `hlfir::EntityWithAttributes requirements.`.
  **L3288 CN**: 注释说明附近代码的逻辑、意图或元数据：`hlfir::EntityWithAttributes requirements.`。

### Lines 3289-3312

````cpp
    auto [exv, cleanup] = hlfir::translateToExtendedValue(
        loc, builder, hlfir::Entity{exactlyOnce});
    assert(!cleanup && "resut is a variable");
    return hlfir::genDeclare(loc, builder, exv, ".func.pointer.result",
                             fir::FortranVariableFlagsAttr{});
  }
  CallContext callContext(procRef, resultType, loc, converter, symMap, stmtCtx);
  return genProcedureRef(callContext);
}

void Fortran::lower::convertUserDefinedAssignmentToHLFIR(
    mlir::Location loc, Fortran::lower::AbstractConverter &converter,
    const evaluate::ProcedureRef &procRef, hlfir::Entity lhs, hlfir::Entity rhs,
    Fortran::lower::SymMap &symMap) {
  Fortran::lower::StatementContext definedAssignmentContext;
  // For defined assignment, don't use regular copy-in/copy-out mechanism:
  // defined assignment generates hlfir.region_assign construct, and this
  // construct automatically handles any copy-in.
  CallContext callContext(procRef, /*resultType=*/std::nullopt, loc, converter,
                          symMap, definedAssignmentContext, /*doCopyIn=*/false);
  Fortran::lower::CallerInterface caller(procRef, converter);
  mlir::FunctionType callSiteType = caller.genFunctionType();
  PreparedActualArgument preparedLhs{lhs, /*isPresent=*/std::nullopt};
  PreparedActualArgument preparedRhs{rhs, /*isPresent=*/std::nullopt};
````
- **L3289 EN**: Continues logic associated with callable symbol `translateToExtendedValue`.
  **L3289 CN**: 继续与可调用符号 `translateToExtendedValue` 相关的逻辑。
- **L3290 EN**: Executes a standalone statement or declaration: `loc, builder, hlfir::Entity{exactlyOnce});`.
  **L3290 CN**: 执行一条独立语句或声明：`loc, builder, hlfir::Entity{exactlyOnce});`。
- **L3291 EN**: Checks an internal invariant in debug builds.
  **L3291 CN**: 在调试构建中检查内部不变式。
- **L3292 EN**: Returns from the current function with `hlfir::genDeclare(loc, builder, exv, ".func.pointer.result",`.
  **L3292 CN**: 以 `hlfir::genDeclare(loc, builder, exv, ".func.pointer.result",` 从当前函数返回。
- **L3293 EN**: Executes a standalone statement or declaration: `fir::FortranVariableFlagsAttr{});`.
  **L3293 CN**: 执行一条独立语句或声明：`fir::FortranVariableFlagsAttr{});`。
- **L3294 EN**: Closes the current lexical scope or compound statement.
  **L3294 CN**: 结束当前词法作用域或复合语句块。
- **L3295 EN**: Executes a call or declaration centered on `callContext`.
  **L3295 CN**: 执行以 `callContext` 为核心的调用或声明。
- **L3296 EN**: Returns from the current function with `genProcedureRef(callContext)`.
  **L3296 CN**: 以 `genProcedureRef(callContext)` 从当前函数返回。
- **L3297 EN**: Closes the current lexical scope or compound statement.
  **L3297 CN**: 结束当前词法作用域或复合语句块。
- **L3298 EN**: Blank line separating nearby declarations or logic blocks.
  **L3298 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3299 EN**: Continues logic associated with callable symbol `convertUserDefinedAssignmentToHLFIR`.
  **L3299 CN**: 继续与可调用符号 `convertUserDefinedAssignmentToHLFIR` 相关的逻辑。
- **L3300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, Fortran::lower::AbstractConverter &converter,`.
  **L3300 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, Fortran::lower::AbstractConverter &converter,`。
- **L3301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const evaluate::ProcedureRef &procRef, hlfir::Entity lhs, hlfir::Entity rhs,`.
  **L3301 CN**: 继续一个多行参数列表、初始化器或聚合项：`const evaluate::ProcedureRef &procRef, hlfir::Entity lhs, hlfir::Entity rhs,`。
- **L3302 EN**: Continues the surrounding expression or declaration: `Fortran::lower::SymMap &symMap) {`.
  **L3302 CN**: 继续构造周围的表达式或声明：`Fortran::lower::SymMap &symMap) {`。
- **L3303 EN**: Executes a standalone statement or declaration: `Fortran::lower::StatementContext definedAssignmentContext;`.
  **L3303 CN**: 执行一条独立语句或声明：`Fortran::lower::StatementContext definedAssignmentContext;`。
- **L3304 EN**: Comment explains nearby logic, intent, or metadata: `For defined assignment, don't use regular copy-in/copy-out mechanism:`.
  **L3304 CN**: 注释说明附近代码的逻辑、意图或元数据：`For defined assignment, don't use regular copy-in/copy-out mechanism:`。
- **L3305 EN**: Comment explains nearby logic, intent, or metadata: `defined assignment generates hlfir.region_assign construct, and this`.
  **L3305 CN**: 注释说明附近代码的逻辑、意图或元数据：`defined assignment generates hlfir.region_assign construct, and this`。
- **L3306 EN**: Comment explains nearby logic, intent, or metadata: `construct automatically handles any copy-in.`.
  **L3306 CN**: 注释说明附近代码的逻辑、意图或元数据：`construct automatically handles any copy-in.`。
- **L3307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CallContext callContext(procRef, /*resultType=*/std::nullopt, loc, converter,`.
  **L3307 CN**: 继续一个多行参数列表、初始化器或聚合项：`CallContext callContext(procRef, /*resultType=*/std::nullopt, loc, converter,`。
- **L3308 EN**: Executes a standalone statement or declaration: `symMap, definedAssignmentContext, /*doCopyIn=*/false);`.
  **L3308 CN**: 执行一条独立语句或声明：`symMap, definedAssignmentContext, /*doCopyIn=*/false);`。
- **L3309 EN**: Executes a call or declaration centered on `caller`.
  **L3309 CN**: 执行以 `caller` 为核心的调用或声明。
- **L3310 EN**: Initializes variable `callSiteType` from the right-hand expression.
  **L3310 CN**: 使用右侧表达式初始化变量 `callSiteType`。
- **L3311 EN**: Executes a standalone statement or declaration: `PreparedActualArgument preparedLhs{lhs, /*isPresent=*/std::nullopt};`.
  **L3311 CN**: 执行一条独立语句或声明：`PreparedActualArgument preparedLhs{lhs, /*isPresent=*/std::nullopt};`。
- **L3312 EN**: Executes a standalone statement or declaration: `PreparedActualArgument preparedRhs{rhs, /*isPresent=*/std::nullopt};`.
  **L3312 CN**: 执行一条独立语句或声明：`PreparedActualArgument preparedRhs{rhs, /*isPresent=*/std::nullopt};`。

### Lines 3313-3316

````cpp
  PreparedActualArguments loweredActuals{preparedLhs, preparedRhs};
  genUserCall(loweredActuals, caller, callSiteType, callContext);
  return;
}
````
- **L3313 EN**: Executes a standalone statement or declaration: `PreparedActualArguments loweredActuals{preparedLhs, preparedRhs};`.
  **L3313 CN**: 执行一条独立语句或声明：`PreparedActualArguments loweredActuals{preparedLhs, preparedRhs};`。
- **L3314 EN**: Executes a call or declaration centered on `genUserCall`.
  **L3314 CN**: 执行以 `genUserCall` 为核心的调用或声明。
- **L3315 EN**: Returns from the current function with `void`.
  **L3315 CN**: 以 `void` 从当前函数返回。
- **L3316 EN**: Closes the current lexical scope or compound statement.
  **L3316 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Lowering to FIR/MLIR / 向 FIR/MLIR 的 lowering**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Fortran type-spec reasoning / Fortran 类型说明推理**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **Evaluation context management / 求值上下文管理**
- **Lowering converter orchestration / lowering 转换器编排**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **HLFIR-specific abstractions / HLFIR 专用抽象**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**

## Dependencies / 依赖关系

- `flang/Lower/ConvertCall.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Lower/Allocatable.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Lower/ConvertExprToHLFIR.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Lower/ConvertProcedureDesignator.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Lower/ConvertVariable.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Lower/CustomIntrinsicCall.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Lower/HlfirIntrinsics.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Lower/PFTBuilder.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Lower/StatementContext.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Lower/SymbolMap.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Optimizer/Builder/BoxValue.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/CUFCommon.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/Character.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/FIRBuilder.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/HLFIRTools.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/IntrinsicCall.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
