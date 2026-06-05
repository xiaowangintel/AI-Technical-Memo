# CUFDeviceFuncTransform.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/Transforms/CUDA/CUFDeviceFuncTransform.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements optimizer or code-generation passes for CUF Device Func Transform.
- **Purpose (CN)**: 实现 CUF Device Func Transform 相关的优化或代码生成 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===-- CUFDeviceFuncTransform.cpp ----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Optimizer/Builder/CUFCommon.h"
#include "flang/Optimizer/Builder/Todo.h"
#include "flang/Optimizer/Dialect/CUF/CUFOps.h"
#include "flang/Optimizer/Dialect/FIRAttr.h"
#include "flang/Optimizer/Dialect/FIRDialect.h"
#include "flang/Optimizer/Dialect/FIROpsSupport.h"
#include "flang/Optimizer/Support/InternalNames.h"
#include "flang/Optimizer/Transforms/Passes.h"
#include "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
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
- **L9 EN**: Includes "flang/Optimizer/Builder/CUFCommon.h" to access FIR builder helpers and runtime-construction utilities.
  **L9 CN**: 引入 "flang/Optimizer/Builder/CUFCommon.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L10 EN**: Includes "flang/Optimizer/Builder/Todo.h" to access FIR builder helpers and runtime-construction utilities.
  **L10 CN**: 引入 "flang/Optimizer/Builder/Todo.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L11 EN**: Includes "flang/Optimizer/Dialect/CUF/CUFOps.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L11 CN**: 引入 "flang/Optimizer/Dialect/CUF/CUFOps.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L12 EN**: Includes "flang/Optimizer/Dialect/FIRAttr.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L12 CN**: 引入 "flang/Optimizer/Dialect/FIRAttr.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L13 EN**: Includes "flang/Optimizer/Dialect/FIRDialect.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L13 CN**: 引入 "flang/Optimizer/Dialect/FIRDialect.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L14 EN**: Includes "flang/Optimizer/Dialect/FIROpsSupport.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L14 CN**: 引入 "flang/Optimizer/Dialect/FIROpsSupport.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L15 EN**: Includes "flang/Optimizer/Support/InternalNames.h" to access optimizer-side support routines and utilities.
  **L15 CN**: 引入 "flang/Optimizer/Support/InternalNames.h" 以使用优化器侧支持例程与工具。
- **L16 EN**: Includes "flang/Optimizer/Transforms/Passes.h" to access local declarations paired with this implementation.
  **L16 CN**: 引入 "flang/Optimizer/Transforms/Passes.h" 以使用与该实现配套的本地声明。
- **L17 EN**: Includes "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L17 CN**: 引入 "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L18 EN**: Includes "mlir/Dialect/GPU/IR/GPUDialect.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L18 CN**: 引入 "mlir/Dialect/GPU/IR/GPUDialect.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。

### Lines 19-36

````cpp
#include "mlir/Dialect/Index/IR/IndexDialect.h"
#include "mlir/Dialect/Index/IR/IndexOps.h"
#include "mlir/Dialect/LLVMIR/LLVMDialect.h"
#include "mlir/Dialect/LLVMIR/NVVMDialect.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/Pass/Pass.h"
#include "mlir/Transforms/RegionUtils.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/StringSet.h"

namespace fir {
#define GEN_PASS_DEF_CUFDEVICEFUNCTRANSFORM
#include "flang/Optimizer/Transforms/Passes.h.inc"
} // namespace fir

using namespace mlir;

````
- **L19 EN**: Includes "mlir/Dialect/Index/IR/IndexDialect.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L19 CN**: 引入 "mlir/Dialect/Index/IR/IndexDialect.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L20 EN**: Includes "mlir/Dialect/Index/IR/IndexOps.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L20 CN**: 引入 "mlir/Dialect/Index/IR/IndexOps.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L21 EN**: Includes "mlir/Dialect/LLVMIR/LLVMDialect.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L21 CN**: 引入 "mlir/Dialect/LLVMIR/LLVMDialect.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L22 EN**: Includes "mlir/Dialect/LLVMIR/NVVMDialect.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L22 CN**: 引入 "mlir/Dialect/LLVMIR/NVVMDialect.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L23 EN**: Includes "mlir/Dialect/SCF/IR/SCF.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L23 CN**: 引入 "mlir/Dialect/SCF/IR/SCF.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L24 EN**: Includes "mlir/IR/IRMapping.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L24 CN**: 引入 "mlir/IR/IRMapping.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L25 EN**: Includes "mlir/Pass/Pass.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L25 CN**: 引入 "mlir/Pass/Pass.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L26 EN**: Includes "mlir/Transforms/RegionUtils.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L26 CN**: 引入 "mlir/Transforms/RegionUtils.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L27 EN**: Includes "llvm/ADT/SetVector.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L27 CN**: 引入 "llvm/ADT/SetVector.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L28 EN**: Includes "llvm/ADT/StringSet.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L28 CN**: 引入 "llvm/ADT/StringSet.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Opens namespace scope `fir`.
  **L30 CN**: 打开命名空间作用域 `fir`。
- **L31 EN**: Defines macro `GEN_PASS_DEF_CUFDEVICEFUNCTRANSFORM` for conditional compilation or local shorthand.
  **L31 CN**: 定义宏 `GEN_PASS_DEF_CUFDEVICEFUNCTRANSFORM`，用于条件编译或本地简写。
- **L32 EN**: Includes "flang/Optimizer/Transforms/Passes.h.inc" to access supporting declarations used by this translation unit.
  **L32 CN**: 引入 "flang/Optimizer/Transforms/Passes.h.inc" 以使用当前编译单元使用的辅助声明。
- **L33 EN**: Closes a namespace scope with a trailing comment: `} // namespace fir`.
  **L33 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace fir`。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L35 EN**: Brings namespace `mlir` into the local scope.
  **L35 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 37-54

````cpp
namespace {

class CUFDeviceFuncTransform
    : public fir::impl::CUFDeviceFuncTransformBase<CUFDeviceFuncTransform> {
  using CUFDeviceFuncTransformBase<
      CUFDeviceFuncTransform>::CUFDeviceFuncTransformBase;

  static gpu::GPUFuncOp createGPUFuncOp(mlir::func::FuncOp funcOp,
                                        bool isGlobal, int computeCap) {
    mlir::OpBuilder builder(funcOp.getContext());

    mlir::Region &funcOpBody = funcOp.getBody();
    SetVector<Value> operands;
    for (mlir::Value operand : funcOp.getArguments())
      operands.insert(operand);

    llvm::SmallVector<mlir::Type> funcOperandTypes;
    llvm::SmallVector<mlir::Type> funcResultTypes;
````
- **L37 EN**: Opens namespace scope ``.
  **L37 CN**: 打开命名空间作用域 ``。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L39 EN**: Declares class `CUFDeviceFuncTransform`.
  **L39 CN**: 声明 class `CUFDeviceFuncTransform`。
- **L40 EN**: Continues the surrounding expression or declaration: `: public fir::impl::CUFDeviceFuncTransformBase<CUFDeviceFuncTransform> {`.
  **L40 CN**: 继续构造周围的表达式或声明：`: public fir::impl::CUFDeviceFuncTransformBase<CUFDeviceFuncTransform> {`。
- **L41 EN**: Continues the surrounding expression or declaration: `using CUFDeviceFuncTransformBase<`.
  **L41 CN**: 继续构造周围的表达式或声明：`using CUFDeviceFuncTransformBase<`。
- **L42 EN**: Executes a standalone statement or declaration: `CUFDeviceFuncTransform>::CUFDeviceFuncTransformBase;`.
  **L42 CN**: 执行一条独立语句或声明：`CUFDeviceFuncTransform>::CUFDeviceFuncTransformBase;`。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static gpu::GPUFuncOp createGPUFuncOp(mlir::func::FuncOp funcOp,`.
  **L44 CN**: 继续一个多行参数列表、初始化器或聚合项：`static gpu::GPUFuncOp createGPUFuncOp(mlir::func::FuncOp funcOp,`。
- **L45 EN**: Continues the surrounding expression or declaration: `bool isGlobal, int computeCap) {`.
  **L45 CN**: 继续构造周围的表达式或声明：`bool isGlobal, int computeCap) {`。
- **L46 EN**: Executes a call or declaration centered on `builder`.
  **L46 CN**: 执行以 `builder` 为核心的调用或声明。
- **L47 EN**: Blank line separating nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L48 EN**: Executes a call or declaration centered on `funcOp.getBody`.
  **L48 CN**: 执行以 `funcOp.getBody` 为核心的调用或声明。
- **L49 EN**: Executes a standalone statement or declaration: `SetVector<Value> operands;`.
  **L49 CN**: 执行一条独立语句或声明：`SetVector<Value> operands;`。
- **L50 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L50 CN**: 开始 `for` 控制流语句并计算其条件。
- **L51 EN**: Executes a call or declaration centered on `operands.insert`.
  **L51 CN**: 执行以 `operands.insert` 为核心的调用或声明。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Type> funcOperandTypes;`.
  **L53 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Type> funcOperandTypes;`。
- **L54 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Type> funcResultTypes;`.
  **L54 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Type> funcResultTypes;`。

### Lines 55-72

````cpp
    funcOperandTypes.reserve(funcOp.getArgumentTypes().size());
    funcResultTypes.reserve(funcOp.getResultTypes().size());
    for (mlir::Type opTy : funcOp.getArgumentTypes())
      funcOperandTypes.push_back(opTy);
    for (mlir::Type resTy : funcOp.getResultTypes())
      funcResultTypes.push_back(resTy);

    mlir::Location loc = funcOp.getLoc();

    mlir::FunctionType type = mlir::FunctionType::get(
        funcOp.getContext(), funcOperandTypes, funcResultTypes);

    auto deviceFuncOp =
        gpu::GPUFuncOp::create(builder, loc, funcOp.getName(), type,
                               mlir::TypeRange{}, mlir::TypeRange{});
    if (isGlobal)
      deviceFuncOp.setKernel(true);

````
- **L55 EN**: Executes a call or declaration centered on `funcOperandTypes.reserve`.
  **L55 CN**: 执行以 `funcOperandTypes.reserve` 为核心的调用或声明。
- **L56 EN**: Executes a call or declaration centered on `funcResultTypes.reserve`.
  **L56 CN**: 执行以 `funcResultTypes.reserve` 为核心的调用或声明。
- **L57 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L57 CN**: 开始 `for` 控制流语句并计算其条件。
- **L58 EN**: Executes a call or declaration centered on `funcOperandTypes.push_back`.
  **L58 CN**: 执行以 `funcOperandTypes.push_back` 为核心的调用或声明。
- **L59 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L59 CN**: 开始 `for` 控制流语句并计算其条件。
- **L60 EN**: Executes a call or declaration centered on `funcResultTypes.push_back`.
  **L60 CN**: 执行以 `funcResultTypes.push_back` 为核心的调用或声明。
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L62 EN**: Initializes variable `loc` from the right-hand expression.
  **L62 CN**: 使用右侧表达式初始化变量 `loc`。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L64 EN**: Continues logic associated with callable symbol `get`.
  **L64 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L65 EN**: Executes a call or declaration centered on `funcOp.getContext`.
  **L65 CN**: 执行以 `funcOp.getContext` 为核心的调用或声明。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L67 EN**: Continues the surrounding expression or declaration: `auto deviceFuncOp =`.
  **L67 CN**: 继续构造周围的表达式或声明：`auto deviceFuncOp =`。
- **L68 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::GPUFuncOp::create(builder, loc, funcOp.getName(), type,`.
  **L68 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::GPUFuncOp::create(builder, loc, funcOp.getName(), type,`。
- **L69 EN**: Executes a standalone statement or declaration: `mlir::TypeRange{}, mlir::TypeRange{});`.
  **L69 CN**: 执行一条独立语句或声明：`mlir::TypeRange{}, mlir::TypeRange{});`。
- **L70 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L70 CN**: 开始 `if` 控制流语句并计算其条件。
- **L71 EN**: Executes a call or declaration centered on `deviceFuncOp.setKernel`.
  **L71 CN**: 执行以 `deviceFuncOp.setKernel` 为核心的调用或声明。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 73-90

````cpp
    mlir::Region &deviceFuncBody = deviceFuncOp.getBody();
    mlir::Block &entryBlock = deviceFuncBody.front();

    mlir::IRMapping map;
    for (const auto &operand : enumerate(operands))
      map.map(operand.value(), entryBlock.getArgument(operand.index()));

    funcOpBody.cloneInto(&deviceFuncBody, map);

    deviceFuncOp.walk([](func::ReturnOp op) {
      mlir::OpBuilder replacer(op);
      gpu::ReturnOp gpuReturnOp = gpu::ReturnOp::create(replacer, op.getLoc());
      gpuReturnOp->setOperands(op.getOperands());
      op.erase();
    });

    mlir::Block &funcOpEntry = funcOp.front();
    mlir::Block *clonedFuncOpEntry = map.lookup(&funcOpEntry);
````
- **L73 EN**: Executes a call or declaration centered on `deviceFuncOp.getBody`.
  **L73 CN**: 执行以 `deviceFuncOp.getBody` 为核心的调用或声明。
- **L74 EN**: Executes a call or declaration centered on `deviceFuncBody.front`.
  **L74 CN**: 执行以 `deviceFuncBody.front` 为核心的调用或声明。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L76 EN**: Executes a standalone statement or declaration: `mlir::IRMapping map;`.
  **L76 CN**: 执行一条独立语句或声明：`mlir::IRMapping map;`。
- **L77 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L77 CN**: 开始 `for` 控制流语句并计算其条件。
- **L78 EN**: Executes a call or declaration centered on `map.map`.
  **L78 CN**: 执行以 `map.map` 为核心的调用或声明。
- **L79 EN**: Blank line separating nearby declarations or logic blocks.
  **L79 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L80 EN**: Executes a call or declaration centered on `funcOpBody.cloneInto`.
  **L80 CN**: 执行以 `funcOpBody.cloneInto` 为核心的调用或声明。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L82 EN**: Starts a function, method, lambda, or structured scope: `deviceFuncOp.walk([](func::ReturnOp op) {`.
  **L82 CN**: 开始一个函数、方法、lambda 或结构化作用域：`deviceFuncOp.walk([](func::ReturnOp op) {`。
- **L83 EN**: Executes a call or declaration centered on `replacer`.
  **L83 CN**: 执行以 `replacer` 为核心的调用或声明。
- **L84 EN**: Initializes variable `gpuReturnOp` from the right-hand expression.
  **L84 CN**: 使用右侧表达式初始化变量 `gpuReturnOp`。
- **L85 EN**: Executes a call or declaration centered on `gpuReturnOp->setOperands`.
  **L85 CN**: 执行以 `gpuReturnOp->setOperands` 为核心的调用或声明。
- **L86 EN**: Executes a call or declaration centered on `op.erase`.
  **L86 CN**: 执行以 `op.erase` 为核心的调用或声明。
- **L87 EN**: Executes a standalone statement or declaration: `});`.
  **L87 CN**: 执行一条独立语句或声明：`});`。
- **L88 EN**: Blank line separating nearby declarations or logic blocks.
  **L88 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L89 EN**: Executes a call or declaration centered on `funcOp.front`.
  **L89 CN**: 执行以 `funcOp.front` 为核心的调用或声明。
- **L90 EN**: Executes a call or declaration centered on `map.lookup`.
  **L90 CN**: 执行以 `map.lookup` 为核心的调用或声明。

### Lines 91-108

````cpp

    entryBlock.getOperations().splice(entryBlock.getOperations().end(),
                                      clonedFuncOpEntry->getOperations());
    clonedFuncOpEntry->erase();

    auto launchBoundsAttr =
        funcOp.getOperation()->getAttrOfType<cuf::LaunchBoundsAttr>(
            cuf::getLaunchBoundsAttrName());
    if (launchBoundsAttr) {
      auto maxTPB = launchBoundsAttr.getMaxTPB().getInt();
      auto maxntid =
          builder.getDenseI32ArrayAttr({static_cast<int32_t>(maxTPB), 1, 1});
      deviceFuncOp->setAttr(NVVM::NVVMDialect::getMaxntidAttrName(), maxntid);
      deviceFuncOp->setAttr(NVVM::NVVMDialect::getMinctasmAttrName(),
                            launchBoundsAttr.getMinBPM());
      if (computeCap >= 90 && launchBoundsAttr.getUpperBoundClusterSize())
        deviceFuncOp->setAttr(NVVM::NVVMDialect::getClusterMaxBlocksAttrName(),
                              launchBoundsAttr.getUpperBoundClusterSize());
````
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `entryBlock.getOperations().splice(entryBlock.getOperations().end(),`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`entryBlock.getOperations().splice(entryBlock.getOperations().end(),`。
- **L93 EN**: Executes a call or declaration centered on `clonedFuncOpEntry->getOperations`.
  **L93 CN**: 执行以 `clonedFuncOpEntry->getOperations` 为核心的调用或声明。
- **L94 EN**: Executes a call or declaration centered on `clonedFuncOpEntry->erase`.
  **L94 CN**: 执行以 `clonedFuncOpEntry->erase` 为核心的调用或声明。
- **L95 EN**: Blank line separating nearby declarations or logic blocks.
  **L95 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L96 EN**: Continues the surrounding expression or declaration: `auto launchBoundsAttr =`.
  **L96 CN**: 继续构造周围的表达式或声明：`auto launchBoundsAttr =`。
- **L97 EN**: Continues logic associated with callable symbol `getOperation`.
  **L97 CN**: 继续与可调用符号 `getOperation` 相关的逻辑。
- **L98 EN**: Executes a call or declaration centered on `cuf::getLaunchBoundsAttrName`.
  **L98 CN**: 执行以 `cuf::getLaunchBoundsAttrName` 为核心的调用或声明。
- **L99 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L99 CN**: 开始 `if` 控制流语句并计算其条件。
- **L100 EN**: Initializes variable `maxTPB` from the right-hand expression.
  **L100 CN**: 使用右侧表达式初始化变量 `maxTPB`。
- **L101 EN**: Continues the surrounding expression or declaration: `auto maxntid =`.
  **L101 CN**: 继续构造周围的表达式或声明：`auto maxntid =`。
- **L102 EN**: Executes a call or declaration centered on `builder.getDenseI32ArrayAttr`.
  **L102 CN**: 执行以 `builder.getDenseI32ArrayAttr` 为核心的调用或声明。
- **L103 EN**: Executes a call or declaration centered on `deviceFuncOp->setAttr`.
  **L103 CN**: 执行以 `deviceFuncOp->setAttr` 为核心的调用或声明。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `deviceFuncOp->setAttr(NVVM::NVVMDialect::getMinctasmAttrName(),`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`deviceFuncOp->setAttr(NVVM::NVVMDialect::getMinctasmAttrName(),`。
- **L105 EN**: Executes a call or declaration centered on `launchBoundsAttr.getMinBPM`.
  **L105 CN**: 执行以 `launchBoundsAttr.getMinBPM` 为核心的调用或声明。
- **L106 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L106 CN**: 开始 `if` 控制流语句并计算其条件。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `deviceFuncOp->setAttr(NVVM::NVVMDialect::getClusterMaxBlocksAttrName(),`.
  **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`deviceFuncOp->setAttr(NVVM::NVVMDialect::getClusterMaxBlocksAttrName(),`。
- **L108 EN**: Executes a call or declaration centered on `launchBoundsAttr.getUpperBoundClusterSize`.
  **L108 CN**: 执行以 `launchBoundsAttr.getUpperBoundClusterSize` 为核心的调用或声明。

### Lines 109-126

````cpp
    }

    return deviceFuncOp;
  }

  static void createHostStub(mlir::func::FuncOp funcOp,
                             mlir::SymbolTable &symTab, mlir::ModuleOp mod) {
    mlir::Location loc = funcOp.getLoc();
    mlir::OpBuilder modBuilder(mod.getBodyRegion());
    modBuilder.setInsertionPointToEnd(mod.getBody());
    auto emptyStub = func::FuncOp::create(modBuilder, loc, funcOp.getName(),
                                          funcOp.getFunctionType());
    emptyStub.setVisibility(funcOp.getVisibility());
    emptyStub->setAttrs(funcOp->getAttrs());
    auto entryBlock = emptyStub.addEntryBlock();
    modBuilder.setInsertionPointToEnd(entryBlock);
    func::ReturnOp::create(modBuilder, loc);

````
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L111 EN**: Returns from the current function with `deviceFuncOp`.
  **L111 CN**: 以 `deviceFuncOp` 从当前函数返回。
- **L112 EN**: Closes the current lexical scope or compound statement.
  **L112 CN**: 结束当前词法作用域或复合语句块。
- **L113 EN**: Blank line separating nearby declarations or logic blocks.
  **L113 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void createHostStub(mlir::func::FuncOp funcOp,`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void createHostStub(mlir::func::FuncOp funcOp,`。
- **L115 EN**: Continues the surrounding expression or declaration: `mlir::SymbolTable &symTab, mlir::ModuleOp mod) {`.
  **L115 CN**: 继续构造周围的表达式或声明：`mlir::SymbolTable &symTab, mlir::ModuleOp mod) {`。
- **L116 EN**: Initializes variable `loc` from the right-hand expression.
  **L116 CN**: 使用右侧表达式初始化变量 `loc`。
- **L117 EN**: Executes a call or declaration centered on `modBuilder`.
  **L117 CN**: 执行以 `modBuilder` 为核心的调用或声明。
- **L118 EN**: Executes a call or declaration centered on `modBuilder.setInsertionPointToEnd`.
  **L118 CN**: 执行以 `modBuilder.setInsertionPointToEnd` 为核心的调用或声明。
- **L119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto emptyStub = func::FuncOp::create(modBuilder, loc, funcOp.getName(),`.
  **L119 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto emptyStub = func::FuncOp::create(modBuilder, loc, funcOp.getName(),`。
- **L120 EN**: Executes a call or declaration centered on `funcOp.getFunctionType`.
  **L120 CN**: 执行以 `funcOp.getFunctionType` 为核心的调用或声明。
- **L121 EN**: Executes a call or declaration centered on `emptyStub.setVisibility`.
  **L121 CN**: 执行以 `emptyStub.setVisibility` 为核心的调用或声明。
- **L122 EN**: Executes a call or declaration centered on `emptyStub->setAttrs`.
  **L122 CN**: 执行以 `emptyStub->setAttrs` 为核心的调用或声明。
- **L123 EN**: Initializes variable `entryBlock` from the right-hand expression.
  **L123 CN**: 使用右侧表达式初始化变量 `entryBlock`。
- **L124 EN**: Executes a call or declaration centered on `modBuilder.setInsertionPointToEnd`.
  **L124 CN**: 执行以 `modBuilder.setInsertionPointToEnd` 为核心的调用或声明。
- **L125 EN**: Executes a call or declaration centered on `func::ReturnOp::create`.
  **L125 CN**: 执行以 `func::ReturnOp::create` 为核心的调用或声明。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 127-144

````cpp
    symTab.erase(funcOp);
    symTab.insert(emptyStub);
  }

  static bool isDeviceFunc(mlir::func::FuncOp funcOp) {
    if (auto cudaProcAttr =
            funcOp.getOperation()->getAttrOfType<cuf::ProcAttributeAttr>(
                cuf::getProcAttrName()))
      if (cudaProcAttr.getValue() == cuf::ProcAttribute::Device ||
          cudaProcAttr.getValue() == cuf::ProcAttribute::Global ||
          cudaProcAttr.getValue() == cuf::ProcAttribute::GridGlobal ||
          cudaProcAttr.getValue() == cuf::ProcAttribute::HostDevice)
        return true;
    return false;
  }

  void runOnOperation() override {
    // Working on Module operation because inserting/removing function from the
````
- **L127 EN**: Executes a call or declaration centered on `symTab.erase`.
  **L127 CN**: 执行以 `symTab.erase` 为核心的调用或声明。
- **L128 EN**: Executes a call or declaration centered on `symTab.insert`.
  **L128 CN**: 执行以 `symTab.insert` 为核心的调用或声明。
- **L129 EN**: Closes the current lexical scope or compound statement.
  **L129 CN**: 结束当前词法作用域或复合语句块。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L131 EN**: Starts a function, method, lambda, or structured scope: `static bool isDeviceFunc(mlir::func::FuncOp funcOp) {`.
  **L131 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isDeviceFunc(mlir::func::FuncOp funcOp) {`。
- **L132 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L132 CN**: 开始 `if` 控制流语句并计算其条件。
- **L133 EN**: Continues logic associated with callable symbol `getOperation`.
  **L133 CN**: 继续与可调用符号 `getOperation` 相关的逻辑。
- **L134 EN**: Continues logic associated with callable symbol `getProcAttrName`.
  **L134 CN**: 继续与可调用符号 `getProcAttrName` 相关的逻辑。
- **L135 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L135 CN**: 开始 `if` 控制流语句并计算其条件。
- **L136 EN**: Continues logic associated with callable symbol `getValue`.
  **L136 CN**: 继续与可调用符号 `getValue` 相关的逻辑。
- **L137 EN**: Continues logic associated with callable symbol `getValue`.
  **L137 CN**: 继续与可调用符号 `getValue` 相关的逻辑。
- **L138 EN**: Continues logic associated with callable symbol `getValue`.
  **L138 CN**: 继续与可调用符号 `getValue` 相关的逻辑。
- **L139 EN**: Returns from the current function with `true`.
  **L139 CN**: 以 `true` 从当前函数返回。
- **L140 EN**: Returns from the current function with `false`.
  **L140 CN**: 以 `false` 从当前函数返回。
- **L141 EN**: Closes the current lexical scope or compound statement.
  **L141 CN**: 结束当前词法作用域或复合语句块。
- **L142 EN**: Blank line separating nearby declarations or logic blocks.
  **L142 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L143 EN**: Starts a function, method, lambda, or structured scope: `void runOnOperation() override {`.
  **L143 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void runOnOperation() override {`。
- **L144 EN**: Comment explains nearby logic, intent, or metadata: `Working on Module operation because inserting/removing function from the`.
  **L144 CN**: 注释说明附近代码的逻辑、意图或元数据：`Working on Module operation because inserting/removing function from the`。

### Lines 145-162

````cpp
    // module is not thread-safe.
    ModuleOp mod = getOperation();
    mlir::SymbolTable symbolTable(getOperation());

    auto *ctx = getOperation().getContext();
    mlir::OpBuilder builder(ctx);

    gpu::GPUModuleOp gpuMod = cuf::getOrCreateGPUModule(mod, symbolTable);
    mlir::SymbolTable gpuModSymTab(gpuMod);

    llvm::SetVector<mlir::func::FuncOp> funcsToClone;
    llvm::SetVector<mlir::func::FuncOp> deviceFuncs;
    llvm::SetVector<mlir::func::FuncOp> keepInModule;
    llvm::StringSet<> deviceFuncNames;

    // Look for all function to migrate to the GPU module.
    mod.walk([&](mlir::func::FuncOp op) {
      if (isDeviceFunc(op)) {
````
- **L145 EN**: Comment explains nearby logic, intent, or metadata: `module is not thread-safe.`.
  **L145 CN**: 注释说明附近代码的逻辑、意图或元数据：`module is not thread-safe.`。
- **L146 EN**: Initializes variable `mod` from the right-hand expression.
  **L146 CN**: 使用右侧表达式初始化变量 `mod`。
- **L147 EN**: Executes a call or declaration centered on `symbolTable`.
  **L147 CN**: 执行以 `symbolTable` 为核心的调用或声明。
- **L148 EN**: Blank line separating nearby declarations or logic blocks.
  **L148 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L149 EN**: Executes a call or declaration centered on `getOperation`.
  **L149 CN**: 执行以 `getOperation` 为核心的调用或声明。
- **L150 EN**: Executes a call or declaration centered on `builder`.
  **L150 CN**: 执行以 `builder` 为核心的调用或声明。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L152 EN**: Initializes variable `gpuMod` from the right-hand expression.
  **L152 CN**: 使用右侧表达式初始化变量 `gpuMod`。
- **L153 EN**: Executes a call or declaration centered on `gpuModSymTab`.
  **L153 CN**: 执行以 `gpuModSymTab` 为核心的调用或声明。
- **L154 EN**: Blank line separating nearby declarations or logic blocks.
  **L154 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L155 EN**: Executes a standalone statement or declaration: `llvm::SetVector<mlir::func::FuncOp> funcsToClone;`.
  **L155 CN**: 执行一条独立语句或声明：`llvm::SetVector<mlir::func::FuncOp> funcsToClone;`。
- **L156 EN**: Executes a standalone statement or declaration: `llvm::SetVector<mlir::func::FuncOp> deviceFuncs;`.
  **L156 CN**: 执行一条独立语句或声明：`llvm::SetVector<mlir::func::FuncOp> deviceFuncs;`。
- **L157 EN**: Executes a standalone statement or declaration: `llvm::SetVector<mlir::func::FuncOp> keepInModule;`.
  **L157 CN**: 执行一条独立语句或声明：`llvm::SetVector<mlir::func::FuncOp> keepInModule;`。
- **L158 EN**: Executes a standalone statement or declaration: `llvm::StringSet<> deviceFuncNames;`.
  **L158 CN**: 执行一条独立语句或声明：`llvm::StringSet<> deviceFuncNames;`。
- **L159 EN**: Blank line separating nearby declarations or logic blocks.
  **L159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L160 EN**: Comment explains nearby logic, intent, or metadata: `Look for all function to migrate to the GPU module.`.
  **L160 CN**: 注释说明附近代码的逻辑、意图或元数据：`Look for all function to migrate to the GPU module.`。
- **L161 EN**: Starts a function, method, lambda, or structured scope: `mod.walk([&](mlir::func::FuncOp op) {`.
  **L161 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mod.walk([&](mlir::func::FuncOp op) {`。
- **L162 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L162 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 163-180

````cpp
        deviceFuncs.insert(op);
        deviceFuncNames.insert(op.getSymName());
      }
    });

    auto processCallOp = [&](fir::CallOp op) {
      if (op.getCallee()) {
        auto func = symbolTable.lookup<mlir::func::FuncOp>(
            op.getCallee()->getLeafReference());
        if (deviceFuncs.count(func) == 0)
          funcsToClone.insert(func);
      }
    };

    // Gather all function called by device functions.
    for (auto funcOp : deviceFuncs) {
      funcOp.walk([&](fir::CallOp op) { processCallOp(op); });
      funcOp.walk([&](fir::DispatchOp op) {
````
- **L163 EN**: Executes a call or declaration centered on `deviceFuncs.insert`.
  **L163 CN**: 执行以 `deviceFuncs.insert` 为核心的调用或声明。
- **L164 EN**: Executes a call or declaration centered on `deviceFuncNames.insert`.
  **L164 CN**: 执行以 `deviceFuncNames.insert` 为核心的调用或声明。
- **L165 EN**: Closes the current lexical scope or compound statement.
  **L165 CN**: 结束当前词法作用域或复合语句块。
- **L166 EN**: Executes a standalone statement or declaration: `});`.
  **L166 CN**: 执行一条独立语句或声明：`});`。
- **L167 EN**: Blank line separating nearby declarations or logic blocks.
  **L167 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L168 EN**: Starts a function, method, lambda, or structured scope: `auto processCallOp = [&](fir::CallOp op) {`.
  **L168 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto processCallOp = [&](fir::CallOp op) {`。
- **L169 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L169 CN**: 开始 `if` 控制流语句并计算其条件。
- **L170 EN**: Continues logic associated with callable symbol `FuncOp>`.
  **L170 CN**: 继续与可调用符号 `FuncOp>` 相关的逻辑。
- **L171 EN**: Executes a call or declaration centered on `op.getCallee`.
  **L171 CN**: 执行以 `op.getCallee` 为核心的调用或声明。
- **L172 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L172 CN**: 开始 `if` 控制流语句并计算其条件。
- **L173 EN**: Executes a call or declaration centered on `funcsToClone.insert`.
  **L173 CN**: 执行以 `funcsToClone.insert` 为核心的调用或声明。
- **L174 EN**: Closes the current lexical scope or compound statement.
  **L174 CN**: 结束当前词法作用域或复合语句块。
- **L175 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L175 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L176 EN**: Blank line separating nearby declarations or logic blocks.
  **L176 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L177 EN**: Comment explains nearby logic, intent, or metadata: `Gather all function called by device functions.`.
  **L177 CN**: 注释说明附近代码的逻辑、意图或元数据：`Gather all function called by device functions.`。
- **L178 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L178 CN**: 开始 `for` 控制流语句并计算其条件。
- **L179 EN**: Executes a call or declaration centered on `funcOp.walk`.
  **L179 CN**: 执行以 `funcOp.walk` 为核心的调用或声明。
- **L180 EN**: Starts a function, method, lambda, or structured scope: `funcOp.walk([&](fir::DispatchOp op) {`.
  **L180 CN**: 开始一个函数、方法、lambda 或结构化作用域：`funcOp.walk([&](fir::DispatchOp op) {`。

### Lines 181-198

````cpp
        TODO(op.getLoc(), "type-bound procedure call with dynamic dispatch "
                          "in device procedure");
      });
    }

    // Functions that are referenced in a derived-type binding table must be
    // kept in the host module to avoid LLVM dialect verification errors.
    for (auto globalOp : mod.getOps<fir::GlobalOp>()) {
      if (globalOp.getName().contains(fir::kBindingTableSeparator)) {
        globalOp.walk([&](fir::AddrOfOp addrOfOp) {
          if (deviceFuncNames.contains(addrOfOp.getSymbol().getLeafReference()))
            keepInModule.insert(
                *llvm::find_if(deviceFuncs, [&](mlir::func::FuncOp f) {
                  return f.getSymName() ==
                         addrOfOp.getSymbol().getLeafReference();
                }));
        });
      }
````
- **L181 EN**: Continues logic associated with callable symbol `TODO`.
  **L181 CN**: 继续与可调用符号 `TODO` 相关的逻辑。
- **L182 EN**: Executes a standalone statement or declaration: `"in device procedure");`.
  **L182 CN**: 执行一条独立语句或声明：`"in device procedure");`。
- **L183 EN**: Executes a standalone statement or declaration: `});`.
  **L183 CN**: 执行一条独立语句或声明：`});`。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L186 EN**: Comment explains nearby logic, intent, or metadata: `Functions that are referenced in a derived-type binding table must be`.
  **L186 CN**: 注释说明附近代码的逻辑、意图或元数据：`Functions that are referenced in a derived-type binding table must be`。
- **L187 EN**: Comment explains nearby logic, intent, or metadata: `kept in the host module to avoid LLVM dialect verification errors.`.
  **L187 CN**: 注释说明附近代码的逻辑、意图或元数据：`kept in the host module to avoid LLVM dialect verification errors.`。
- **L188 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L188 CN**: 开始 `for` 控制流语句并计算其条件。
- **L189 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L189 CN**: 开始 `if` 控制流语句并计算其条件。
- **L190 EN**: Starts a function, method, lambda, or structured scope: `globalOp.walk([&](fir::AddrOfOp addrOfOp) {`.
  **L190 CN**: 开始一个函数、方法、lambda 或结构化作用域：`globalOp.walk([&](fir::AddrOfOp addrOfOp) {`。
- **L191 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L191 CN**: 开始 `if` 控制流语句并计算其条件。
- **L192 EN**: Continues logic associated with callable symbol `insert`.
  **L192 CN**: 继续与可调用符号 `insert` 相关的逻辑。
- **L193 EN**: Comment explains nearby logic, intent, or metadata: `llvm::find_if(deviceFuncs, [&](mlir::func::FuncOp f) {`.
  **L193 CN**: 注释说明附近代码的逻辑、意图或元数据：`llvm::find_if(deviceFuncs, [&](mlir::func::FuncOp f) {`。
- **L194 EN**: Returns from the current function with `f.getSymName() ==`.
  **L194 CN**: 以 `f.getSymName() ==` 从当前函数返回。
- **L195 EN**: Executes a call or declaration centered on `addrOfOp.getSymbol`.
  **L195 CN**: 执行以 `addrOfOp.getSymbol` 为核心的调用或声明。
- **L196 EN**: Executes a standalone statement or declaration: `}));`.
  **L196 CN**: 执行一条独立语句或声明：`}));`。
- **L197 EN**: Executes a standalone statement or declaration: `});`.
  **L197 CN**: 执行一条独立语句或声明：`});`。
- **L198 EN**: Closes the current lexical scope or compound statement.
  **L198 CN**: 结束当前词法作用域或复合语句块。

### Lines 199-216

````cpp
    }

    // Gather all functions called by CUF kernels.
    mod.walk([&](cuf::KernelOp kernelOp) {
      kernelOp.walk([&](fir::CallOp op) { processCallOp(op); });
      kernelOp.walk([&](fir::DispatchOp op) {
        TODO(op.getLoc(),
             "type-bound procedure call with dynamic dispatch in cuf kernel");
      });
    });

    for (auto funcOp : funcsToClone)
      gpuModSymTab.insert(funcOp->clone());

    for (auto funcOp : deviceFuncs) {
      auto cudaProcAttr =
          funcOp.getOperation()->getAttrOfType<cuf::ProcAttributeAttr>(
              cuf::getProcAttrName());
````
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L201 EN**: Comment explains nearby logic, intent, or metadata: `Gather all functions called by CUF kernels.`.
  **L201 CN**: 注释说明附近代码的逻辑、意图或元数据：`Gather all functions called by CUF kernels.`。
- **L202 EN**: Starts a function, method, lambda, or structured scope: `mod.walk([&](cuf::KernelOp kernelOp) {`.
  **L202 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mod.walk([&](cuf::KernelOp kernelOp) {`。
- **L203 EN**: Executes a call or declaration centered on `kernelOp.walk`.
  **L203 CN**: 执行以 `kernelOp.walk` 为核心的调用或声明。
- **L204 EN**: Starts a function, method, lambda, or structured scope: `kernelOp.walk([&](fir::DispatchOp op) {`.
  **L204 CN**: 开始一个函数、方法、lambda 或结构化作用域：`kernelOp.walk([&](fir::DispatchOp op) {`。
- **L205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TODO(op.getLoc(),`.
  **L205 CN**: 继续一个多行参数列表、初始化器或聚合项：`TODO(op.getLoc(),`。
- **L206 EN**: Executes a standalone statement or declaration: `"type-bound procedure call with dynamic dispatch in cuf kernel");`.
  **L206 CN**: 执行一条独立语句或声明：`"type-bound procedure call with dynamic dispatch in cuf kernel");`。
- **L207 EN**: Executes a standalone statement or declaration: `});`.
  **L207 CN**: 执行一条独立语句或声明：`});`。
- **L208 EN**: Executes a standalone statement or declaration: `});`.
  **L208 CN**: 执行一条独立语句或声明：`});`。
- **L209 EN**: Blank line separating nearby declarations or logic blocks.
  **L209 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L210 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L210 CN**: 开始 `for` 控制流语句并计算其条件。
- **L211 EN**: Executes a call or declaration centered on `gpuModSymTab.insert`.
  **L211 CN**: 执行以 `gpuModSymTab.insert` 为核心的调用或声明。
- **L212 EN**: Blank line separating nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L213 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L213 CN**: 开始 `for` 控制流语句并计算其条件。
- **L214 EN**: Continues the surrounding expression or declaration: `auto cudaProcAttr =`.
  **L214 CN**: 继续构造周围的表达式或声明：`auto cudaProcAttr =`。
- **L215 EN**: Continues logic associated with callable symbol `getOperation`.
  **L215 CN**: 继续与可调用符号 `getOperation` 相关的逻辑。
- **L216 EN**: Executes a call or declaration centered on `cuf::getProcAttrName`.
  **L216 CN**: 执行以 `cuf::getProcAttrName` 为核心的调用或声明。

### Lines 217-234

````cpp
      auto isGlobal = cudaProcAttr.getValue() == cuf::ProcAttribute::Global ||
                      cudaProcAttr.getValue() == cuf::ProcAttribute::GridGlobal;
      if (funcOp.isDeclaration()) {
        mlir::Operation *clonedFuncOp = funcOp->clone();
        if (isGlobal) {
          clonedFuncOp->setAttr(gpu::GPUDialect::getKernelFuncAttrName(),
                                builder.getUnitAttr());
          clonedFuncOp->removeAttr(cuf::getProcAttrName());
          if (auto funcOp = mlir::dyn_cast<func::FuncOp>(clonedFuncOp))
            funcOp.setNested();
        }
        gpuModSymTab.insert(clonedFuncOp);
      } else {
        gpu::GPUFuncOp deviceFuncOp =
            createGPUFuncOp(funcOp, isGlobal, computeCap);
        gpuModSymTab.insert(deviceFuncOp);

        if (cudaProcAttr.getValue() != cuf::ProcAttribute::HostDevice) {
````
- **L217 EN**: Continues logic associated with callable symbol `getValue`.
  **L217 CN**: 继续与可调用符号 `getValue` 相关的逻辑。
- **L218 EN**: Executes a call or declaration centered on `cudaProcAttr.getValue`.
  **L218 CN**: 执行以 `cudaProcAttr.getValue` 为核心的调用或声明。
- **L219 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L219 CN**: 开始 `if` 控制流语句并计算其条件。
- **L220 EN**: Executes a call or declaration centered on `funcOp->clone`.
  **L220 CN**: 执行以 `funcOp->clone` 为核心的调用或声明。
- **L221 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L221 CN**: 开始 `if` 控制流语句并计算其条件。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `clonedFuncOp->setAttr(gpu::GPUDialect::getKernelFuncAttrName(),`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`clonedFuncOp->setAttr(gpu::GPUDialect::getKernelFuncAttrName(),`。
- **L223 EN**: Executes a call or declaration centered on `builder.getUnitAttr`.
  **L223 CN**: 执行以 `builder.getUnitAttr` 为核心的调用或声明。
- **L224 EN**: Executes a call or declaration centered on `clonedFuncOp->removeAttr`.
  **L224 CN**: 执行以 `clonedFuncOp->removeAttr` 为核心的调用或声明。
- **L225 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L225 CN**: 开始 `if` 控制流语句并计算其条件。
- **L226 EN**: Executes a call or declaration centered on `funcOp.setNested`.
  **L226 CN**: 执行以 `funcOp.setNested` 为核心的调用或声明。
- **L227 EN**: Closes the current lexical scope or compound statement.
  **L227 CN**: 结束当前词法作用域或复合语句块。
- **L228 EN**: Executes a call or declaration centered on `gpuModSymTab.insert`.
  **L228 CN**: 执行以 `gpuModSymTab.insert` 为核心的调用或声明。
- **L229 EN**: Transitions from the previous branch into the alternative path.
  **L229 CN**: 从前一个分支过渡到备选路径。
- **L230 EN**: Continues the surrounding expression or declaration: `gpu::GPUFuncOp deviceFuncOp =`.
  **L230 CN**: 继续构造周围的表达式或声明：`gpu::GPUFuncOp deviceFuncOp =`。
- **L231 EN**: Executes a call or declaration centered on `createGPUFuncOp`.
  **L231 CN**: 执行以 `createGPUFuncOp` 为核心的调用或声明。
- **L232 EN**: Executes a call or declaration centered on `gpuModSymTab.insert`.
  **L232 CN**: 执行以 `gpuModSymTab.insert` 为核心的调用或声明。
- **L233 EN**: Blank line separating nearby declarations or logic blocks.
  **L233 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L234 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L234 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 235-249

````cpp
          // If the function is a global, we need to keep the host side
          // declaration for the kernel registration. Currently we just
          // erase its body but in the future, the body should be rewritten
          // to be able to launch CUDA Fortran kernel from C code.
          if (isGlobal || keepInModule.contains(funcOp))
            createHostStub(funcOp, symbolTable, mod);
          else
            funcOp.erase();
        }
      }
    }
  }
};

} // end anonymous namespace
````
- **L235 EN**: Comment explains nearby logic, intent, or metadata: `If the function is a global, we need to keep the host side`.
  **L235 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the function is a global, we need to keep the host side`。
- **L236 EN**: Comment explains nearby logic, intent, or metadata: `declaration for the kernel registration. Currently we just`.
  **L236 CN**: 注释说明附近代码的逻辑、意图或元数据：`declaration for the kernel registration. Currently we just`。
- **L237 EN**: Comment explains nearby logic, intent, or metadata: `erase its body but in the future, the body should be rewritten`.
  **L237 CN**: 注释说明附近代码的逻辑、意图或元数据：`erase its body but in the future, the body should be rewritten`。
- **L238 EN**: Comment explains nearby logic, intent, or metadata: `to be able to launch CUDA Fortran kernel from C code.`.
  **L238 CN**: 注释说明附近代码的逻辑、意图或元数据：`to be able to launch CUDA Fortran kernel from C code.`。
- **L239 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L239 CN**: 开始 `if` 控制流语句并计算其条件。
- **L240 EN**: Executes a call or declaration centered on `createHostStub`.
  **L240 CN**: 执行以 `createHostStub` 为核心的调用或声明。
- **L241 EN**: Transitions from the previous branch into the alternative path.
  **L241 CN**: 从前一个分支过渡到备选路径。
- **L242 EN**: Executes a call or declaration centered on `funcOp.erase`.
  **L242 CN**: 执行以 `funcOp.erase` 为核心的调用或声明。
- **L243 EN**: Closes the current lexical scope or compound statement.
  **L243 CN**: 结束当前词法作用域或复合语句块。
- **L244 EN**: Closes the current lexical scope or compound statement.
  **L244 CN**: 结束当前词法作用域或复合语句块。
- **L245 EN**: Closes the current lexical scope or compound statement.
  **L245 CN**: 结束当前词法作用域或复合语句块。
- **L246 EN**: Closes the current lexical scope or compound statement.
  **L246 CN**: 结束当前词法作用域或复合语句块。
- **L247 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L247 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L249 EN**: Continues the surrounding expression or declaration: `} // end anonymous namespace`.
  **L249 CN**: 继续构造周围的表达式或声明：`} // end anonymous namespace`。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **Symbol modeling and lookup / 符号建模与查找**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**
- **IR builder orchestration / IR Builder 编排**
- **CUDA-specific lowering or runtime handling / CUDA 专用 lowering 或运行时处理**

## Dependencies / 依赖关系

- `flang/Optimizer/Builder/CUFCommon.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/Todo.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Dialect/CUF/CUFOps.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Dialect/FIRAttr.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Dialect/FIRDialect.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Dialect/FIROpsSupport.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Support/InternalNames.h`: Provides optimizer-side support routines and utilities. / 提供优化器侧支持例程与工具。
- `flang/Optimizer/Transforms/Passes.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/GPU/IR/GPUDialect.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/Index/IR/IndexDialect.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/Index/IR/IndexOps.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/LLVMIR/LLVMDialect.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/LLVMIR/NVVMDialect.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/SCF/IR/SCF.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/IR/IRMapping.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
