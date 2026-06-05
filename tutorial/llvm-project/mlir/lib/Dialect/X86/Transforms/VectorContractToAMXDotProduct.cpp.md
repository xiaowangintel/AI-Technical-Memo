# VectorContractToAMXDotProduct.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/X86/Transforms/VectorContractToAMXDotProduct.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements MLIR dialect logic, transformation code, or utilities related to `VectorContractToAMXDotProduct`.
- **Purpose (CN)**: 实现与 `VectorContractToAMXDotProduct` 相关的 MLIR 方言逻辑、变换代码或辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===- VectorContractToAMXDotProduct.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/Dialect/Vector/Utils/VectorUtils.h"
#include "mlir/Dialect/X86/Transforms.h"
#include "mlir/Dialect/X86/Utils/X86Utils.h"
#include "mlir/Dialect/X86/X86Dialect.h"

#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/Dominance.h"
#include "mlir/IR/PatternMatch.h"
#include "llvm/Support/Casting.h"

#include "mlir/Pass/Pass.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"

using namespace mlir;
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes "mlir/Dialect/SCF/IR/SCF.h" to access dialect-specific IR, transforms, or shared utilities.
  **L9 CN**: 引入 "mlir/Dialect/SCF/IR/SCF.h" 以使用方言专用 IR、变换或共享工具。
- **L10 EN**: Includes "mlir/Dialect/Vector/IR/VectorOps.h" to access dialect-specific IR, transforms, or shared utilities.
  **L10 CN**: 引入 "mlir/Dialect/Vector/IR/VectorOps.h" 以使用方言专用 IR、变换或共享工具。
- **L11 EN**: Includes "mlir/Dialect/Vector/Utils/VectorUtils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L11 CN**: 引入 "mlir/Dialect/Vector/Utils/VectorUtils.h" 以使用方言专用 IR、变换或共享工具。
- **L12 EN**: Includes "mlir/Dialect/X86/Transforms.h" to access dialect-specific IR, transforms, or shared utilities.
  **L12 CN**: 引入 "mlir/Dialect/X86/Transforms.h" 以使用方言专用 IR、变换或共享工具。
- **L13 EN**: Includes "mlir/Dialect/X86/Utils/X86Utils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L13 CN**: 引入 "mlir/Dialect/X86/Utils/X86Utils.h" 以使用方言专用 IR、变换或共享工具。
- **L14 EN**: Includes "mlir/Dialect/X86/X86Dialect.h" to access dialect-specific IR, transforms, or shared utilities.
  **L14 CN**: 引入 "mlir/Dialect/X86/X86Dialect.h" 以使用方言专用 IR、变换或共享工具。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L16 EN**: Includes "mlir/IR/BuiltinAttributes.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L16 CN**: 引入 "mlir/IR/BuiltinAttributes.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L17 EN**: Includes "mlir/IR/Dominance.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L17 CN**: 引入 "mlir/IR/Dominance.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L18 EN**: Includes "mlir/IR/PatternMatch.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L18 CN**: 引入 "mlir/IR/PatternMatch.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L19 EN**: Includes "llvm/Support/Casting.h" to access LLVM support-library facilities such as diagnostics and casting.
  **L19 CN**: 引入 "llvm/Support/Casting.h" 以使用LLVM 支持库设施，如诊断与类型转换。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L21 EN**: Includes "mlir/Pass/Pass.h" to access pass infrastructure and pass registration support.
  **L21 CN**: 引入 "mlir/Pass/Pass.h" 以使用Pass 基础设施与 pass 注册支持。
- **L22 EN**: Includes "mlir/Transforms/GreedyPatternRewriteDriver.h" to access generic transformation utilities and canonicalization helpers.
  **L22 CN**: 引入 "mlir/Transforms/GreedyPatternRewriteDriver.h" 以使用通用变换工具与规范化辅助逻辑。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Brings namespace `mlir` into local scope.
  **L24 CN**: 将命名空间 `mlir` 引入当前作用域。

### Lines 25-48

````cpp
using namespace mlir::vector;
using namespace mlir::x86;

namespace {

// Function to collapse the last two dimension (vnni and k) to help the
// amx.tile_load to correctly load the packed element type.
static Value collapseInnerDims(OpBuilder &builder, mlir::Location loc,
                               Value input) {
  ShapedType inputType = cast<ShapedType>(input.getType());
  int64_t firstDimToCollapse = inputType.getRank() - 2;

  if (inputType.getRank() == 1)
    return input;

  SmallVector<ReassociationIndices> reassociation;
  for (int64_t i = 0; i < firstDimToCollapse; ++i)
    reassociation.push_back(ReassociationIndices{i});

  ReassociationIndices collapsedIndices;
  for (int64_t i = firstDimToCollapse; i < inputType.getRank(); ++i)
    collapsedIndices.push_back(i);

  reassociation.push_back(collapsedIndices);
````
- **L25 EN**: Brings namespace `mlir::vector` into local scope.
  **L25 CN**: 将命名空间 `mlir::vector` 引入当前作用域。
- **L26 EN**: Brings namespace `mlir::x86` into local scope.
  **L26 CN**: 将命名空间 `mlir::x86` 引入当前作用域。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Opens namespace scope ``.
  **L28 CN**: 打开命名空间作用域 ``。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Comment explains nearby logic, invariants, or intent: `Function to collapse the last two dimension (vnni and k) to help the`.
  **L30 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Function to collapse the last two dimension (vnni and k) to help the`。
- **L31 EN**: Comment explains nearby logic, invariants, or intent: `amx.tile_load to correctly load the packed element type.`.
  **L31 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`amx.tile_load to correctly load the packed element type.`。
- **L32 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value collapseInnerDims(OpBuilder &builder, mlir::Location loc,`.
  **L32 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value collapseInnerDims(OpBuilder &builder, mlir::Location loc,`。
- **L33 EN**: Continues the surrounding expression or declaration: `Value input) {`.
  **L33 CN**: 继续构造周围的表达式或声明：`Value input) {`。
- **L34 EN**: Initializes variable `inputType` from the right-hand expression.
  **L34 CN**: 使用右侧表达式初始化变量 `inputType`。
- **L35 EN**: Initializes variable `firstDimToCollapse` from the right-hand expression.
  **L35 CN**: 使用右侧表达式初始化变量 `firstDimToCollapse`。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L37 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L37 CN**: 开始 `if` 控制流语句并计算其条件。
- **L38 EN**: Returns from the current function with `input`.
  **L38 CN**: 以 `input` 从当前函数返回。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L40 EN**: Executes a standalone statement or declaration: `SmallVector<ReassociationIndices> reassociation;`.
  **L40 CN**: 执行一条独立语句或声明：`SmallVector<ReassociationIndices> reassociation;`。
- **L41 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L41 CN**: 开始 `for` 控制流语句并计算其条件。
- **L42 EN**: Executes a call or declaration centered on `reassociation.push_back`.
  **L42 CN**: 执行以 `reassociation.push_back` 为核心的调用或声明。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Executes a standalone statement or declaration: `ReassociationIndices collapsedIndices;`.
  **L44 CN**: 执行一条独立语句或声明：`ReassociationIndices collapsedIndices;`。
- **L45 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L45 CN**: 开始 `for` 控制流语句并计算其条件。
- **L46 EN**: Executes a call or declaration centered on `collapsedIndices.push_back`.
  **L46 CN**: 执行以 `collapsedIndices.push_back` 为核心的调用或声明。
- **L47 EN**: Blank line separating nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L48 EN**: Executes a call or declaration centered on `reassociation.push_back`.
  **L48 CN**: 执行以 `reassociation.push_back` 为核心的调用或声明。

### Lines 49-72

````cpp
  return memref::CollapseShapeOp::create(builder, loc, input, reassociation);
}

// Get the MemRef source and offset index for the operands of
// vector.contract.
static FailureOr<std::pair<Value, SmallVector<Value>>>
getSrcIndxValue(OpBuilder &rewriter, Location loc, Value operand,
                bool isNotAcc) {
  Operation *defOp = operand.getDefiningOp();
  if (!defOp)
    return failure();

  Value srcBuff;
  SmallVector<OpFoldResult> indexVals;
  llvm::TypeSwitch<Operation *>(operand.getDefiningOp())
      .Case<TransferReadOp, LoadOp>([&](auto readOp) {
        indexVals = SmallVector<OpFoldResult>(readOp.getIndices().begin(),
                                              readOp.getIndices().end());
        srcBuff = readOp.getOperand(0);
      });

  if (!srcBuff)
    return failure();

````
- **L49 EN**: Returns from the current function with `memref::CollapseShapeOp::create(builder, loc, input, reassociation)`.
  **L49 CN**: 以 `memref::CollapseShapeOp::create(builder, loc, input, reassociation)` 从当前函数返回。
- **L50 EN**: Closes the current lexical scope or compound statement.
  **L50 CN**: 结束当前词法作用域或复合语句块。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Comment explains nearby logic, invariants, or intent: `Get the MemRef source and offset index for the operands of`.
  **L52 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get the MemRef source and offset index for the operands of`。
- **L53 EN**: Comment explains nearby logic, invariants, or intent: `vector.contract.`.
  **L53 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vector.contract.`。
- **L54 EN**: Continues the surrounding expression or declaration: `static FailureOr<std::pair<Value, SmallVector<Value>>>`.
  **L54 CN**: 继续构造周围的表达式或声明：`static FailureOr<std::pair<Value, SmallVector<Value>>>`。
- **L55 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getSrcIndxValue(OpBuilder &rewriter, Location loc, Value operand,`.
  **L55 CN**: 继续一个多行参数列表、初始化器或聚合项：`getSrcIndxValue(OpBuilder &rewriter, Location loc, Value operand,`。
- **L56 EN**: Continues the surrounding expression or declaration: `bool isNotAcc) {`.
  **L56 CN**: 继续构造周围的表达式或声明：`bool isNotAcc) {`。
- **L57 EN**: Executes a call or declaration centered on `operand.getDefiningOp`.
  **L57 CN**: 执行以 `operand.getDefiningOp` 为核心的调用或声明。
- **L58 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L58 CN**: 开始 `if` 控制流语句并计算其条件。
- **L59 EN**: Returns from the current function with `failure()`.
  **L59 CN**: 以 `failure()` 从当前函数返回。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L61 EN**: Executes a standalone statement or declaration: `Value srcBuff;`.
  **L61 CN**: 执行一条独立语句或声明：`Value srcBuff;`。
- **L62 EN**: Executes a standalone statement or declaration: `SmallVector<OpFoldResult> indexVals;`.
  **L62 CN**: 执行一条独立语句或声明：`SmallVector<OpFoldResult> indexVals;`。
- **L63 EN**: Continues logic associated with callable symbol `getDefiningOp`.
  **L63 CN**: 继续与可调用符号 `getDefiningOp` 相关的逻辑。
- **L64 EN**: Starts a function, method, lambda, or structured scope: `.Case<TransferReadOp, LoadOp>([&](auto readOp) {`.
  **L64 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.Case<TransferReadOp, LoadOp>([&](auto readOp) {`。
- **L65 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `indexVals = SmallVector<OpFoldResult>(readOp.getIndices().begin(),`.
  **L65 CN**: 继续一个多行参数列表、初始化器或聚合项：`indexVals = SmallVector<OpFoldResult>(readOp.getIndices().begin(),`。
- **L66 EN**: Executes a call or declaration centered on `readOp.getIndices`.
  **L66 CN**: 执行以 `readOp.getIndices` 为核心的调用或声明。
- **L67 EN**: Executes a call or declaration centered on `readOp.getOperand`.
  **L67 CN**: 执行以 `readOp.getOperand` 为核心的调用或声明。
- **L68 EN**: Executes a standalone statement or declaration: `});`.
  **L68 CN**: 执行一条独立语句或声明：`});`。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L70 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L70 CN**: 开始 `if` 控制流语句并计算其条件。
- **L71 EN**: Returns from the current function with `failure()`.
  **L71 CN**: 以 `failure()` 从当前函数返回。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 73-96

````cpp
  if (isNotAcc)
    indexVals.pop_back();

  SmallVector<Value> indices;
  indices.reserve(indexVals.size());

  for (OpFoldResult ofr : indexVals) {
    indices.push_back(
        mlir::getValueOrCreateConstantIndexOp(rewriter, loc, ofr));
  }

  if (isNotAcc) {
    srcBuff = collapseInnerDims(rewriter, loc, srcBuff);
  }

  return std::make_pair(srcBuff, indices);
}

// Function to validate the vector.contract operation.
static LogicalResult validateContractOps(OpBuilder &rewriter,
                                         vector::ContractionOp contractOp,
                                         unsigned int blockingFactor,
                                         Value srcBuffLhs, Value srcBuffRhs,
                                         bool srcValidate) {
````
- **L73 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L73 CN**: 开始 `if` 控制流语句并计算其条件。
- **L74 EN**: Executes a call or declaration centered on `indexVals.pop_back`.
  **L74 CN**: 执行以 `indexVals.pop_back` 为核心的调用或声明。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L76 EN**: Executes a standalone statement or declaration: `SmallVector<Value> indices;`.
  **L76 CN**: 执行一条独立语句或声明：`SmallVector<Value> indices;`。
- **L77 EN**: Executes a call or declaration centered on `indices.reserve`.
  **L77 CN**: 执行以 `indices.reserve` 为核心的调用或声明。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L79 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L79 CN**: 开始 `for` 控制流语句并计算其条件。
- **L80 EN**: Continues logic associated with callable symbol `push_back`.
  **L80 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L81 EN**: Executes a call or declaration centered on `mlir::getValueOrCreateConstantIndexOp`.
  **L81 CN**: 执行以 `mlir::getValueOrCreateConstantIndexOp` 为核心的调用或声明。
- **L82 EN**: Closes the current lexical scope or compound statement.
  **L82 CN**: 结束当前词法作用域或复合语句块。
- **L83 EN**: Blank line separating nearby declarations or logic blocks.
  **L83 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L84 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L84 CN**: 开始 `if` 控制流语句并计算其条件。
- **L85 EN**: Executes a call or declaration centered on `collapseInnerDims`.
  **L85 CN**: 执行以 `collapseInnerDims` 为核心的调用或声明。
- **L86 EN**: Closes the current lexical scope or compound statement.
  **L86 CN**: 结束当前词法作用域或复合语句块。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L88 EN**: Returns from the current function with `std::make_pair(srcBuff, indices)`.
  **L88 CN**: 以 `std::make_pair(srcBuff, indices)` 从当前函数返回。
- **L89 EN**: Closes the current lexical scope or compound statement.
  **L89 CN**: 结束当前词法作用域或复合语句块。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L91 EN**: Comment explains nearby logic, invariants, or intent: `Function to validate the vector.contract operation.`.
  **L91 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Function to validate the vector.contract operation.`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static LogicalResult validateContractOps(OpBuilder &rewriter,`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`static LogicalResult validateContractOps(OpBuilder &rewriter,`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::ContractionOp contractOp,`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::ContractionOp contractOp,`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned int blockingFactor,`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned int blockingFactor,`。
- **L95 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value srcBuffLhs, Value srcBuffRhs,`.
  **L95 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value srcBuffLhs, Value srcBuffRhs,`。
- **L96 EN**: Continues the surrounding expression or declaration: `bool srcValidate) {`.
  **L96 CN**: 继续构造周围的表达式或声明：`bool srcValidate) {`。

### Lines 97-120

````cpp

  if (srcValidate) {
    // Get the MemRef buffer of LHS operand.
    auto srcIndxLhs = getSrcIndxValue(rewriter, contractOp.getLoc(),
                                      contractOp.getLhs(), false);
    if (failed(srcIndxLhs))
      return failure();
    auto [buffLhs, indicesLhs] = *srcIndxLhs;

    // Get the MemRef buffer of RHS operand.
    auto srcIndxRhs = getSrcIndxValue(rewriter, contractOp.getLoc(),
                                      contractOp.getRhs(), false);
    if (failed(srcIndxRhs))
      return failure();
    auto [buffRhs, indicesRhs] = *srcIndxRhs;

    // Return failure if the Memref buff didn't match.
    if (buffLhs != srcBuffLhs)
      return failure();

    if (buffRhs != srcBuffRhs)
      return failure();
  }

````
- **L97 EN**: Blank line separating nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L98 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L98 CN**: 开始 `if` 控制流语句并计算其条件。
- **L99 EN**: Comment explains nearby logic, invariants, or intent: `Get the MemRef buffer of LHS operand.`.
  **L99 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get the MemRef buffer of LHS operand.`。
- **L100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto srcIndxLhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`.
  **L100 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto srcIndxLhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`。
- **L101 EN**: Executes a call or declaration centered on `contractOp.getLhs`.
  **L101 CN**: 执行以 `contractOp.getLhs` 为核心的调用或声明。
- **L102 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L102 CN**: 开始 `if` 控制流语句并计算其条件。
- **L103 EN**: Returns from the current function with `failure()`.
  **L103 CN**: 以 `failure()` 从当前函数返回。
- **L104 EN**: Executes a standalone statement or declaration: `auto [buffLhs, indicesLhs] = *srcIndxLhs;`.
  **L104 CN**: 执行一条独立语句或声明：`auto [buffLhs, indicesLhs] = *srcIndxLhs;`。
- **L105 EN**: Blank line separating nearby declarations or logic blocks.
  **L105 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L106 EN**: Comment explains nearby logic, invariants, or intent: `Get the MemRef buffer of RHS operand.`.
  **L106 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get the MemRef buffer of RHS operand.`。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto srcIndxRhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`.
  **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto srcIndxRhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`。
- **L108 EN**: Executes a call or declaration centered on `contractOp.getRhs`.
  **L108 CN**: 执行以 `contractOp.getRhs` 为核心的调用或声明。
- **L109 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L109 CN**: 开始 `if` 控制流语句并计算其条件。
- **L110 EN**: Returns from the current function with `failure()`.
  **L110 CN**: 以 `failure()` 从当前函数返回。
- **L111 EN**: Executes a standalone statement or declaration: `auto [buffRhs, indicesRhs] = *srcIndxRhs;`.
  **L111 CN**: 执行一条独立语句或声明：`auto [buffRhs, indicesRhs] = *srcIndxRhs;`。
- **L112 EN**: Blank line separating nearby declarations or logic blocks.
  **L112 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L113 EN**: Comment explains nearby logic, invariants, or intent: `Return failure if the Memref buff didn't match.`.
  **L113 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return failure if the Memref buff didn't match.`。
- **L114 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L114 CN**: 开始 `if` 控制流语句并计算其条件。
- **L115 EN**: Returns from the current function with `failure()`.
  **L115 CN**: 以 `failure()` 从当前函数返回。
- **L116 EN**: Blank line separating nearby declarations or logic blocks.
  **L116 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L117 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L117 CN**: 开始 `if` 控制流语句并计算其条件。
- **L118 EN**: Returns from the current function with `failure()`.
  **L118 CN**: 以 `failure()` 从当前函数返回。
- **L119 EN**: Closes the current lexical scope or compound statement.
  **L119 CN**: 结束当前词法作用域或复合语句块。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 121-144

````cpp
  VectorType accTy = dyn_cast<VectorType>(contractOp.getAccType());
  if (!accTy)
    return failure();

  // The Accumulator dims should be 16 or 1. Like <1x16x16> or <16x16>.
  ArrayRef<int64_t> accShape = accTy.getShape();
  llvm::SmallVector<int64_t> nonUnitDimAcc;
  llvm::copy_if(accShape, std::back_inserter(nonUnitDimAcc),
                [](int64_t dim) { return (dim != 16 && dim != 1); });

  if (nonUnitDimAcc.size() != 0)
    return failure();

  // The LHS dims should be 16 or vnni or 1. Like <1x16x16x2> or
  // <16x16x4>. The vnni dims should be 2 or 4.
  VectorType lhsTy = contractOp.getLhsType();
  ArrayRef<int64_t> lhsShape = lhsTy.getShape();
  llvm::SmallVector<int64_t> nonUnitDimLhs;
  llvm::copy_if(lhsShape, std::back_inserter(nonUnitDimLhs),
                [](int64_t dim) { return (dim != 16 && dim != 1); });

  if (nonUnitDimLhs.size() != 1)
    return failure();

````
- **L121 EN**: Initializes variable `accTy` from the right-hand expression.
  **L121 CN**: 使用右侧表达式初始化变量 `accTy`。
- **L122 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L122 CN**: 开始 `if` 控制流语句并计算其条件。
- **L123 EN**: Returns from the current function with `failure()`.
  **L123 CN**: 以 `failure()` 从当前函数返回。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L125 EN**: Comment explains nearby logic, invariants, or intent: `The Accumulator dims should be 16 or 1. Like <1x16x16> or <16x16>.`.
  **L125 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The Accumulator dims should be 16 or 1. Like <1x16x16> or <16x16>.`。
- **L126 EN**: Initializes variable `accShape` from the right-hand expression.
  **L126 CN**: 使用右侧表达式初始化变量 `accShape`。
- **L127 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<int64_t> nonUnitDimAcc;`.
  **L127 CN**: 执行一条独立语句或声明：`llvm::SmallVector<int64_t> nonUnitDimAcc;`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::copy_if(accShape, std::back_inserter(nonUnitDimAcc),`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::copy_if(accShape, std::back_inserter(nonUnitDimAcc),`。
- **L129 EN**: Executes a call or declaration centered on `[]`.
  **L129 CN**: 执行以 `[]` 为核心的调用或声明。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L131 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L131 CN**: 开始 `if` 控制流语句并计算其条件。
- **L132 EN**: Returns from the current function with `failure()`.
  **L132 CN**: 以 `failure()` 从当前函数返回。
- **L133 EN**: Blank line separating nearby declarations or logic blocks.
  **L133 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L134 EN**: Comment explains nearby logic, invariants, or intent: `The LHS dims should be 16 or vnni or 1. Like <1x16x16x2> or`.
  **L134 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The LHS dims should be 16 or vnni or 1. Like <1x16x16x2> or`。
- **L135 EN**: Comment explains nearby logic, invariants, or intent: `<16x16x4>. The vnni dims should be 2 or 4.`.
  **L135 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<16x16x4>. The vnni dims should be 2 or 4.`。
- **L136 EN**: Initializes variable `lhsTy` from the right-hand expression.
  **L136 CN**: 使用右侧表达式初始化变量 `lhsTy`。
- **L137 EN**: Initializes variable `lhsShape` from the right-hand expression.
  **L137 CN**: 使用右侧表达式初始化变量 `lhsShape`。
- **L138 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<int64_t> nonUnitDimLhs;`.
  **L138 CN**: 执行一条独立语句或声明：`llvm::SmallVector<int64_t> nonUnitDimLhs;`。
- **L139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::copy_if(lhsShape, std::back_inserter(nonUnitDimLhs),`.
  **L139 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::copy_if(lhsShape, std::back_inserter(nonUnitDimLhs),`。
- **L140 EN**: Executes a call or declaration centered on `[]`.
  **L140 CN**: 执行以 `[]` 为核心的调用或声明。
- **L141 EN**: Blank line separating nearby declarations or logic blocks.
  **L141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L142 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L142 CN**: 开始 `if` 控制流语句并计算其条件。
- **L143 EN**: Returns from the current function with `failure()`.
  **L143 CN**: 以 `failure()` 从当前函数返回。
- **L144 EN**: Blank line separating nearby declarations or logic blocks.
  **L144 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 145-168

````cpp
  if (nonUnitDimLhs[0] != blockingFactor)
    return failure();

  // The RHS dims should be 16 or vnni or 1. Like <1x16x16x2> or
  // <16x16x4>. The vnni dims should be 2 or 4.
  VectorType rhsTy = contractOp.getRhsType();
  ArrayRef<int64_t> rhsShape = rhsTy.getShape();
  llvm::SmallVector<int64_t> nonUnitDimRhs;
  llvm::copy_if(rhsShape, std::back_inserter(nonUnitDimRhs),
                [](int64_t dim) { return (dim != 16 && dim != 1); });

  if (nonUnitDimRhs.size() != 1)
    return failure();

  if (nonUnitDimRhs[0] != blockingFactor)
    return failure();

  return success();
}

// Returns the loop index position to get mapped during the
// MemRef type clone.
static unsigned getIndexPosition(Value operand, scf::ForOp loop) {
  Value iv = loop.getInductionVar();
````
- **L145 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L145 CN**: 开始 `if` 控制流语句并计算其条件。
- **L146 EN**: Returns from the current function with `failure()`.
  **L146 CN**: 以 `failure()` 从当前函数返回。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L148 EN**: Comment explains nearby logic, invariants, or intent: `The RHS dims should be 16 or vnni or 1. Like <1x16x16x2> or`.
  **L148 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The RHS dims should be 16 or vnni or 1. Like <1x16x16x2> or`。
- **L149 EN**: Comment explains nearby logic, invariants, or intent: `<16x16x4>. The vnni dims should be 2 or 4.`.
  **L149 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<16x16x4>. The vnni dims should be 2 or 4.`。
- **L150 EN**: Initializes variable `rhsTy` from the right-hand expression.
  **L150 CN**: 使用右侧表达式初始化变量 `rhsTy`。
- **L151 EN**: Initializes variable `rhsShape` from the right-hand expression.
  **L151 CN**: 使用右侧表达式初始化变量 `rhsShape`。
- **L152 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<int64_t> nonUnitDimRhs;`.
  **L152 CN**: 执行一条独立语句或声明：`llvm::SmallVector<int64_t> nonUnitDimRhs;`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::copy_if(rhsShape, std::back_inserter(nonUnitDimRhs),`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::copy_if(rhsShape, std::back_inserter(nonUnitDimRhs),`。
- **L154 EN**: Executes a call or declaration centered on `[]`.
  **L154 CN**: 执行以 `[]` 为核心的调用或声明。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L156 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L156 CN**: 开始 `if` 控制流语句并计算其条件。
- **L157 EN**: Returns from the current function with `failure()`.
  **L157 CN**: 以 `failure()` 从当前函数返回。
- **L158 EN**: Blank line separating nearby declarations or logic blocks.
  **L158 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L159 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L159 CN**: 开始 `if` 控制流语句并计算其条件。
- **L160 EN**: Returns from the current function with `failure()`.
  **L160 CN**: 以 `failure()` 从当前函数返回。
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L162 EN**: Returns from the current function with `success()`.
  **L162 CN**: 以 `success()` 从当前函数返回。
- **L163 EN**: Closes the current lexical scope or compound statement.
  **L163 CN**: 结束当前词法作用域或复合语句块。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L165 EN**: Comment explains nearby logic, invariants, or intent: `Returns the loop index position to get mapped during the`.
  **L165 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns the loop index position to get mapped during the`。
- **L166 EN**: Comment explains nearby logic, invariants, or intent: `MemRef type clone.`.
  **L166 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MemRef type clone.`。
- **L167 EN**: Starts a function, method, lambda, or structured scope: `static unsigned getIndexPosition(Value operand, scf::ForOp loop) {`.
  **L167 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static unsigned getIndexPosition(Value operand, scf::ForOp loop) {`。
- **L168 EN**: Initializes variable `iv` from the right-hand expression.
  **L168 CN**: 使用右侧表达式初始化变量 `iv`。

### Lines 169-192

````cpp

  Value srcBuff;
  llvm::TypeSwitch<Operation *>(operand.getDefiningOp())
      .Case<TransferReadOp, LoadOp>(
          [&](auto readOp) { srcBuff = readOp.getOperand(0); });

  auto subview = srcBuff.getDefiningOp<memref::SubViewOp>();
  if (!subview)
    return 0;

  auto offsets = subview.getOffsets();

  for (auto it : llvm::enumerate(offsets)) {
    if (it.value() == iv)
      return it.index();
  }

  return 0;
}

// Creates amx.tile_loads.
static amx::TileLoadOp createTileLoads(OpBuilder &rewriter, Location loc,
                                       Value operand, Value mat, Type ipType,
                                       bool rhs, unsigned int offset,
````
- **L169 EN**: Blank line separating nearby declarations or logic blocks.
  **L169 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L170 EN**: Executes a standalone statement or declaration: `Value srcBuff;`.
  **L170 CN**: 执行一条独立语句或声明：`Value srcBuff;`。
- **L171 EN**: Continues logic associated with callable symbol `getDefiningOp`.
  **L171 CN**: 继续与可调用符号 `getDefiningOp` 相关的逻辑。
- **L172 EN**: Continues logic associated with callable symbol `LoadOp>`.
  **L172 CN**: 继续与可调用符号 `LoadOp>` 相关的逻辑。
- **L173 EN**: Executes a call or declaration centered on `[&]`.
  **L173 CN**: 执行以 `[&]` 为核心的调用或声明。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L175 EN**: Initializes variable `subview` from the right-hand expression.
  **L175 CN**: 使用右侧表达式初始化变量 `subview`。
- **L176 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L176 CN**: 开始 `if` 控制流语句并计算其条件。
- **L177 EN**: Returns from the current function with `0`.
  **L177 CN**: 以 `0` 从当前函数返回。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L179 EN**: Initializes variable `offsets` from the right-hand expression.
  **L179 CN**: 使用右侧表达式初始化变量 `offsets`。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L181 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L181 CN**: 开始 `for` 控制流语句并计算其条件。
- **L182 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L182 CN**: 开始 `if` 控制流语句并计算其条件。
- **L183 EN**: Returns from the current function with `it.index()`.
  **L183 CN**: 以 `it.index()` 从当前函数返回。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L186 EN**: Returns from the current function with `0`.
  **L186 CN**: 以 `0` 从当前函数返回。
- **L187 EN**: Closes the current lexical scope or compound statement.
  **L187 CN**: 结束当前词法作用域或复合语句块。
- **L188 EN**: Blank line separating nearby declarations or logic blocks.
  **L188 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L189 EN**: Comment explains nearby logic, invariants, or intent: `Creates amx.tile_loads.`.
  **L189 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Creates amx.tile_loads.`。
- **L190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static amx::TileLoadOp createTileLoads(OpBuilder &rewriter, Location loc,`.
  **L190 CN**: 继续一个多行参数列表、初始化器或聚合项：`static amx::TileLoadOp createTileLoads(OpBuilder &rewriter, Location loc,`。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value operand, Value mat, Type ipType,`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value operand, Value mat, Type ipType,`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool rhs, unsigned int offset,`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool rhs, unsigned int offset,`。

### Lines 193-216

````cpp
                                       bool isVnni) {

  auto srcIndx = getSrcIndxValue(rewriter, loc, operand, false);
  auto [srcBuff, indices] = *srcIndx;
  if (isVnni) {
    indices.pop_back();
  }

  if (rhs && isVnni) {
    auto cOffset = arith::ConstantIndexOp::create(rewriter, loc, offset);
    indices[indices.size() - 1] = arith::MulIOp::create(
        rewriter, loc, indices[indices.size() - 1], cOffset);
  }

  amx::TileType tileType = amx::TileType::get({16, (16 * offset)}, ipType);
  return amx::TileLoadOp::create(rewriter, loc, tileType, mat, indices);
}

static void performShuffle(OpBuilder &rewriter, Location loc, Value matB,
                           Type ipType, unsigned int offset, Value packedBuffer,
                           Value indxToStoreInBuffer) {

  Value c0 = arith::ConstantIndexOp::create(rewriter, loc, 0);
  Value c16 = arith::ConstantIndexOp::create(rewriter, loc, 16);
````
- **L193 EN**: Continues the surrounding expression or declaration: `bool isVnni) {`.
  **L193 CN**: 继续构造周围的表达式或声明：`bool isVnni) {`。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L195 EN**: Initializes variable `srcIndx` from the right-hand expression.
  **L195 CN**: 使用右侧表达式初始化变量 `srcIndx`。
- **L196 EN**: Executes a standalone statement or declaration: `auto [srcBuff, indices] = *srcIndx;`.
  **L196 CN**: 执行一条独立语句或声明：`auto [srcBuff, indices] = *srcIndx;`。
- **L197 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L197 CN**: 开始 `if` 控制流语句并计算其条件。
- **L198 EN**: Executes a call or declaration centered on `indices.pop_back`.
  **L198 CN**: 执行以 `indices.pop_back` 为核心的调用或声明。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L201 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L201 CN**: 开始 `if` 控制流语句并计算其条件。
- **L202 EN**: Initializes variable `cOffset` from the right-hand expression.
  **L202 CN**: 使用右侧表达式初始化变量 `cOffset`。
- **L203 EN**: Continues logic associated with callable symbol `size`.
  **L203 CN**: 继续与可调用符号 `size` 相关的逻辑。
- **L204 EN**: Executes a call or declaration centered on `indices[indices.size`.
  **L204 CN**: 执行以 `indices[indices.size` 为核心的调用或声明。
- **L205 EN**: Closes the current lexical scope or compound statement.
  **L205 CN**: 结束当前词法作用域或复合语句块。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L207 EN**: Initializes variable `tileType` from the right-hand expression.
  **L207 CN**: 使用右侧表达式初始化变量 `tileType`。
- **L208 EN**: Returns from the current function with `amx::TileLoadOp::create(rewriter, loc, tileType, mat, indices)`.
  **L208 CN**: 以 `amx::TileLoadOp::create(rewriter, loc, tileType, mat, indices)` 从当前函数返回。
- **L209 EN**: Closes the current lexical scope or compound statement.
  **L209 CN**: 结束当前词法作用域或复合语句块。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void performShuffle(OpBuilder &rewriter, Location loc, Value matB,`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void performShuffle(OpBuilder &rewriter, Location loc, Value matB,`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Type ipType, unsigned int offset, Value packedBuffer,`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`Type ipType, unsigned int offset, Value packedBuffer,`。
- **L213 EN**: Continues the surrounding expression or declaration: `Value indxToStoreInBuffer) {`.
  **L213 CN**: 继续构造周围的表达式或声明：`Value indxToStoreInBuffer) {`。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L215 EN**: Initializes variable `c0` from the right-hand expression.
  **L215 CN**: 使用右侧表达式初始化变量 `c0`。
- **L216 EN**: Initializes variable `c16` from the right-hand expression.
  **L216 CN**: 使用右侧表达式初始化变量 `c16`。

### Lines 217-240

````cpp

  auto subview = matB.getDefiningOp<mlir::memref::SubViewOp>();
  SmallVector<Value> subviewOffset(subview.getOffsets().size(), c0);

  Value cStep = arith::ConstantIndexOp::create(rewriter, loc, offset);
  Value cBound = arith::ConstantIndexOp::create(rewriter, loc, (16 * offset));
  Value offsetIndx =
      arith::ConstantIndexOp::create(rewriter, loc, (offset / 2));

  scf::ForOp::create(
      rewriter, loc, c0, cBound, cStep, ValueRange{},
      [&](OpBuilder &nestedBuilder, Location loc, Value iv,
          ValueRange iterArgs) {
        subviewOffset[subviewOffset.size() - 2] = iv;
        auto vec1 = vector::LoadOp::create(
            rewriter, loc, VectorType::get((16 * offset), ipType), matB,
            ValueRange(subviewOffset));

        // Increment the iv by 1 or 2 based on the type to load the next 32/64
        // elements
        Value incIV = arith::AddIOp::create(rewriter, loc, offsetIndx, iv);
        subviewOffset[subviewOffset.size() - 2] = incIV;
        auto vec2 = vector::LoadOp::create(
            rewriter, loc, VectorType::get((16 * offset), ipType), matB,
````
- **L217 EN**: Blank line separating nearby declarations or logic blocks.
  **L217 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L218 EN**: Initializes variable `subview` from the right-hand expression.
  **L218 CN**: 使用右侧表达式初始化变量 `subview`。
- **L219 EN**: Executes a call or declaration centered on `subviewOffset`.
  **L219 CN**: 执行以 `subviewOffset` 为核心的调用或声明。
- **L220 EN**: Blank line separating nearby declarations or logic blocks.
  **L220 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L221 EN**: Initializes variable `cStep` from the right-hand expression.
  **L221 CN**: 使用右侧表达式初始化变量 `cStep`。
- **L222 EN**: Initializes variable `cBound` from the right-hand expression.
  **L222 CN**: 使用右侧表达式初始化变量 `cBound`。
- **L223 EN**: Continues the surrounding expression or declaration: `Value offsetIndx =`.
  **L223 CN**: 继续构造周围的表达式或声明：`Value offsetIndx =`。
- **L224 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L224 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L225 EN**: Blank line separating nearby declarations or logic blocks.
  **L225 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L226 EN**: Continues logic associated with callable symbol `create`.
  **L226 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, c0, cBound, cStep, ValueRange{},`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, c0, cBound, cStep, ValueRange{},`。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](OpBuilder &nestedBuilder, Location loc, Value iv,`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](OpBuilder &nestedBuilder, Location loc, Value iv,`。
- **L229 EN**: Continues the surrounding expression or declaration: `ValueRange iterArgs) {`.
  **L229 CN**: 继续构造周围的表达式或声明：`ValueRange iterArgs) {`。
- **L230 EN**: Executes a call or declaration centered on `subviewOffset[subviewOffset.size`.
  **L230 CN**: 执行以 `subviewOffset[subviewOffset.size` 为核心的调用或声明。
- **L231 EN**: Continues logic associated with callable symbol `create`.
  **L231 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get((16 * offset), ipType), matB,`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get((16 * offset), ipType), matB,`。
- **L233 EN**: Executes a call or declaration centered on `ValueRange`.
  **L233 CN**: 执行以 `ValueRange` 为核心的调用或声明。
- **L234 EN**: Blank line separating nearby declarations or logic blocks.
  **L234 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L235 EN**: Comment explains nearby logic, invariants, or intent: `Increment the iv by 1 or 2 based on the type to load the next 32/64`.
  **L235 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Increment the iv by 1 or 2 based on the type to load the next 32/64`。
- **L236 EN**: Comment explains nearby logic, invariants, or intent: `elements`.
  **L236 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`elements`。
- **L237 EN**: Initializes variable `incIV` from the right-hand expression.
  **L237 CN**: 使用右侧表达式初始化变量 `incIV`。
- **L238 EN**: Executes a call or declaration centered on `subviewOffset[subviewOffset.size`.
  **L238 CN**: 执行以 `subviewOffset[subviewOffset.size` 为核心的调用或声明。
- **L239 EN**: Continues logic associated with callable symbol `create`.
  **L239 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get((16 * offset), ipType), matB,`.
  **L240 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get((16 * offset), ipType), matB,`。

### Lines 241-264

````cpp
            ValueRange(subviewOffset));

        vector::ShuffleOp shuffle1;
        vector::ShuffleOp shuffle2;

        if (ipType.isBF16()) {

          shuffle1 = vector::ShuffleOp::create(
              rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,
              vec2,
              ArrayRef<int64_t>{0,  32, 1,  33, 2,  34, 3,  35, 8,  40, 9,
                                41, 10, 42, 11, 43, 16, 48, 17, 49, 18, 50,
                                19, 51, 24, 56, 25, 57, 26, 58, 27, 59});

          shuffle2 = vector::ShuffleOp::create(
              rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,
              vec2,
              ArrayRef<int64_t>{4,  36, 5,  37, 6,  38, 7,  39, 12, 44, 13,
                                45, 14, 46, 15, 47, 20, 52, 21, 53, 22, 54,
                                23, 55, 28, 60, 29, 61, 30, 62, 31, 63});
        }

        if (ipType.isSignlessInteger(8)) {

````
- **L241 EN**: Executes a call or declaration centered on `ValueRange`.
  **L241 CN**: 执行以 `ValueRange` 为核心的调用或声明。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L243 EN**: Executes a standalone statement or declaration: `vector::ShuffleOp shuffle1;`.
  **L243 CN**: 执行一条独立语句或声明：`vector::ShuffleOp shuffle1;`。
- **L244 EN**: Executes a standalone statement or declaration: `vector::ShuffleOp shuffle2;`.
  **L244 CN**: 执行一条独立语句或声明：`vector::ShuffleOp shuffle2;`。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L246 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L246 CN**: 开始 `if` 控制流语句并计算其条件。
- **L247 EN**: Blank line separating nearby declarations or logic blocks.
  **L247 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L248 EN**: Continues logic associated with callable symbol `create`.
  **L248 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,`.
  **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,`。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vec2,`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`vec2,`。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t>{0,  32, 1,  33, 2,  34, 3,  35, 8,  40, 9,`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t>{0,  32, 1,  33, 2,  34, 3,  35, 8,  40, 9,`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `41, 10, 42, 11, 43, 16, 48, 17, 49, 18, 50,`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`41, 10, 42, 11, 43, 16, 48, 17, 49, 18, 50,`。
- **L253 EN**: Executes a standalone statement or declaration: `19, 51, 24, 56, 25, 57, 26, 58, 27, 59});`.
  **L253 CN**: 执行一条独立语句或声明：`19, 51, 24, 56, 25, 57, 26, 58, 27, 59});`。
- **L254 EN**: Blank line separating nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L255 EN**: Continues logic associated with callable symbol `create`.
  **L255 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,`。
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vec2,`.
  **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`vec2,`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t>{4,  36, 5,  37, 6,  38, 7,  39, 12, 44, 13,`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t>{4,  36, 5,  37, 6,  38, 7,  39, 12, 44, 13,`。
- **L259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `45, 14, 46, 15, 47, 20, 52, 21, 53, 22, 54,`.
  **L259 CN**: 继续一个多行参数列表、初始化器或聚合项：`45, 14, 46, 15, 47, 20, 52, 21, 53, 22, 54,`。
- **L260 EN**: Executes a standalone statement or declaration: `23, 55, 28, 60, 29, 61, 30, 62, 31, 63});`.
  **L260 CN**: 执行一条独立语句或声明：`23, 55, 28, 60, 29, 61, 30, 62, 31, 63});`。
- **L261 EN**: Closes the current lexical scope or compound statement.
  **L261 CN**: 结束当前词法作用域或复合语句块。
- **L262 EN**: Blank line separating nearby declarations or logic blocks.
  **L262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L263 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L263 CN**: 开始 `if` 控制流语句并计算其条件。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 265-288

````cpp
          shuffle1 = vector::ShuffleOp::create(
              rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,
              vec2,
              ArrayRef<int64_t>{
                  0,   32,  64, 96,  1,   33,  65,  97,  2,   34,  66,  98, 3,
                  35,  67,  99, 8,   40,  72,  104, 9,   41,  73,  105, 10, 42,
                  74,  106, 11, 43,  75,  107, 16,  48,  80,  112, 17,  49, 81,
                  113, 18,  50, 82,  114, 19,  51,  83,  115, 24,  56,  88, 120,
                  25,  57,  89, 121, 26,  58,  90,  122, 27,  59,  91,  123});

          shuffle2 = vector::ShuffleOp::create(
              rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,
              vec2,
              ArrayRef<int64_t>{
                  4,  36,  68, 100, 5,  37,  69, 101, 6,  38,  70, 102, 7,  39,
                  71, 103, 12, 44,  76, 108, 13, 45,  77, 109, 14, 46,  78, 110,
                  15, 47,  79, 111, 20, 52,  84, 116, 21, 53,  85, 117, 22, 54,
                  86, 118, 23, 55,  87, 119, 28, 60,  92, 124, 29, 61,  93, 125,
                  30, 62,  94, 126, 31, 63,  95, 127});
        }

        // iv to store the shuffled elements
        Value ivShuff1 = arith::DivUIOp::create(rewriter, loc, iv, cStep);
        Value ivShuff2 = arith::AddIOp::create(rewriter, loc, ivShuff1, c16);
````
- **L265 EN**: Continues logic associated with callable symbol `create`.
  **L265 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,`。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vec2,`.
  **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`vec2,`。
- **L268 EN**: Continues the surrounding expression or declaration: `ArrayRef<int64_t>{`.
  **L268 CN**: 继续构造周围的表达式或声明：`ArrayRef<int64_t>{`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0,   32,  64, 96,  1,   33,  65,  97,  2,   34,  66,  98, 3,`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`0,   32,  64, 96,  1,   33,  65,  97,  2,   34,  66,  98, 3,`。
- **L270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `35,  67,  99, 8,   40,  72,  104, 9,   41,  73,  105, 10, 42,`.
  **L270 CN**: 继续一个多行参数列表、初始化器或聚合项：`35,  67,  99, 8,   40,  72,  104, 9,   41,  73,  105, 10, 42,`。
- **L271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `74,  106, 11, 43,  75,  107, 16,  48,  80,  112, 17,  49, 81,`.
  **L271 CN**: 继续一个多行参数列表、初始化器或聚合项：`74,  106, 11, 43,  75,  107, 16,  48,  80,  112, 17,  49, 81,`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `113, 18,  50, 82,  114, 19,  51,  83,  115, 24,  56,  88, 120,`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`113, 18,  50, 82,  114, 19,  51,  83,  115, 24,  56,  88, 120,`。
- **L273 EN**: Executes a standalone statement or declaration: `25,  57,  89, 121, 26,  58,  90,  122, 27,  59,  91,  123});`.
  **L273 CN**: 执行一条独立语句或声明：`25,  57,  89, 121, 26,  58,  90,  122, 27,  59,  91,  123});`。
- **L274 EN**: Blank line separating nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L275 EN**: Continues logic associated with callable symbol `create`.
  **L275 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get({(16 * offset)}, ipType), vec1,`。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vec2,`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`vec2,`。
- **L278 EN**: Continues the surrounding expression or declaration: `ArrayRef<int64_t>{`.
  **L278 CN**: 继续构造周围的表达式或声明：`ArrayRef<int64_t>{`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `4,  36,  68, 100, 5,  37,  69, 101, 6,  38,  70, 102, 7,  39,`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`4,  36,  68, 100, 5,  37,  69, 101, 6,  38,  70, 102, 7,  39,`。
- **L280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `71, 103, 12, 44,  76, 108, 13, 45,  77, 109, 14, 46,  78, 110,`.
  **L280 CN**: 继续一个多行参数列表、初始化器或聚合项：`71, 103, 12, 44,  76, 108, 13, 45,  77, 109, 14, 46,  78, 110,`。
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `15, 47,  79, 111, 20, 52,  84, 116, 21, 53,  85, 117, 22, 54,`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`15, 47,  79, 111, 20, 52,  84, 116, 21, 53,  85, 117, 22, 54,`。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `86, 118, 23, 55,  87, 119, 28, 60,  92, 124, 29, 61,  93, 125,`.
  **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`86, 118, 23, 55,  87, 119, 28, 60,  92, 124, 29, 61,  93, 125,`。
- **L283 EN**: Executes a standalone statement or declaration: `30, 62,  94, 126, 31, 63,  95, 127});`.
  **L283 CN**: 执行一条独立语句或声明：`30, 62,  94, 126, 31, 63,  95, 127});`。
- **L284 EN**: Closes the current lexical scope or compound statement.
  **L284 CN**: 结束当前词法作用域或复合语句块。
- **L285 EN**: Blank line separating nearby declarations or logic blocks.
  **L285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L286 EN**: Comment explains nearby logic, invariants, or intent: `iv to store the shuffled elements`.
  **L286 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`iv to store the shuffled elements`。
- **L287 EN**: Initializes variable `ivShuff1` from the right-hand expression.
  **L287 CN**: 使用右侧表达式初始化变量 `ivShuff1`。
- **L288 EN**: Initializes variable `ivShuff2` from the right-hand expression.
  **L288 CN**: 使用右侧表达式初始化变量 `ivShuff2`。

### Lines 289-312

````cpp

        vector::StoreOp::create(rewriter, loc, shuffle1, packedBuffer,
                                ValueRange{indxToStoreInBuffer, ivShuff1, c0});
        vector::StoreOp::create(rewriter, loc, shuffle2, packedBuffer,
                                ValueRange{indxToStoreInBuffer, ivShuff2, c0});

        scf::YieldOp::create(nestedBuilder, loc);
      });
}

static llvm::DenseMap<Operation *, amx::TileLoadOp>
packInputs(OpBuilder &rewriter, Location loc,
           SmallVector<vector::ContractionOp> ops, Value matB, Type ipType,
           unsigned int offset, Value packedBuffer, bool pack,
           Value indxToStoreInBuffer, Value indxToLoadFromMatB) {

  llvm::DenseMap<Operation *, amx::TileLoadOp> readsToTileLoads;
  Value c0 = arith::ConstantIndexOp::create(rewriter, loc, 0);
  Value c16 = arith::ConstantIndexOp::create(rewriter, loc, 16);

  for (size_t j = 0; j < ops.size(); j++) {
    for (size_t i = 0; i < ops.size(); i++) {

      if (i != j && validatePairVectorContract(ops[j], ops[i], true, 16)) {
````
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::StoreOp::create(rewriter, loc, shuffle1, packedBuffer,`.
  **L290 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::StoreOp::create(rewriter, loc, shuffle1, packedBuffer,`。
- **L291 EN**: Executes a standalone statement or declaration: `ValueRange{indxToStoreInBuffer, ivShuff1, c0});`.
  **L291 CN**: 执行一条独立语句或声明：`ValueRange{indxToStoreInBuffer, ivShuff1, c0});`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::StoreOp::create(rewriter, loc, shuffle2, packedBuffer,`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::StoreOp::create(rewriter, loc, shuffle2, packedBuffer,`。
- **L293 EN**: Executes a standalone statement or declaration: `ValueRange{indxToStoreInBuffer, ivShuff2, c0});`.
  **L293 CN**: 执行一条独立语句或声明：`ValueRange{indxToStoreInBuffer, ivShuff2, c0});`。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L295 EN**: Executes a call or declaration centered on `scf::YieldOp::create`.
  **L295 CN**: 执行以 `scf::YieldOp::create` 为核心的调用或声明。
- **L296 EN**: Executes a standalone statement or declaration: `});`.
  **L296 CN**: 执行一条独立语句或声明：`});`。
- **L297 EN**: Closes the current lexical scope or compound statement.
  **L297 CN**: 结束当前词法作用域或复合语句块。
- **L298 EN**: Blank line separating nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L299 EN**: Continues the surrounding expression or declaration: `static llvm::DenseMap<Operation *, amx::TileLoadOp>`.
  **L299 CN**: 继续构造周围的表达式或声明：`static llvm::DenseMap<Operation *, amx::TileLoadOp>`。
- **L300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `packInputs(OpBuilder &rewriter, Location loc,`.
  **L300 CN**: 继续一个多行参数列表、初始化器或聚合项：`packInputs(OpBuilder &rewriter, Location loc,`。
- **L301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<vector::ContractionOp> ops, Value matB, Type ipType,`.
  **L301 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<vector::ContractionOp> ops, Value matB, Type ipType,`。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned int offset, Value packedBuffer, bool pack,`.
  **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned int offset, Value packedBuffer, bool pack,`。
- **L303 EN**: Continues the surrounding expression or declaration: `Value indxToStoreInBuffer, Value indxToLoadFromMatB) {`.
  **L303 CN**: 继续构造周围的表达式或声明：`Value indxToStoreInBuffer, Value indxToLoadFromMatB) {`。
- **L304 EN**: Blank line separating nearby declarations or logic blocks.
  **L304 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L305 EN**: Executes a standalone statement or declaration: `llvm::DenseMap<Operation *, amx::TileLoadOp> readsToTileLoads;`.
  **L305 CN**: 执行一条独立语句或声明：`llvm::DenseMap<Operation *, amx::TileLoadOp> readsToTileLoads;`。
- **L306 EN**: Initializes variable `c0` from the right-hand expression.
  **L306 CN**: 使用右侧表达式初始化变量 `c0`。
- **L307 EN**: Initializes variable `c16` from the right-hand expression.
  **L307 CN**: 使用右侧表达式初始化变量 `c16`。
- **L308 EN**: Blank line separating nearby declarations or logic blocks.
  **L308 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L309 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L309 CN**: 开始 `for` 控制流语句并计算其条件。
- **L310 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L310 CN**: 开始 `for` 控制流语句并计算其条件。
- **L311 EN**: Blank line separating nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L312 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L312 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 313-336

````cpp

        Operation *readOpRhs = ops[j].getRhs().getDefiningOp();
        auto itRhs = readsToTileLoads.find(readOpRhs);
        if (itRhs != readsToTileLoads.end()) {
          continue;
        }

        if (pack) {
          performShuffle(rewriter, loc, matB, ipType, offset, packedBuffer,
                         indxToStoreInBuffer);
        }

        amx::TileType tileType =
            amx::TileType::get({16, (16 * offset)}, ipType);
        auto loadRow1 =
            amx::TileLoadOp::create(rewriter, loc, tileType, packedBuffer,
                                    ValueRange{indxToLoadFromMatB, c0, c0});

        auto loadRow2 =
            amx::TileLoadOp::create(rewriter, loc, tileType, packedBuffer,
                                    ValueRange{indxToLoadFromMatB, c16, c0});

        readsToTileLoads.try_emplace(readOpRhs, loadRow1);
        readsToTileLoads.try_emplace(ops[i].getRhs().getDefiningOp(), loadRow2);
````
- **L313 EN**: Blank line separating nearby declarations or logic blocks.
  **L313 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L314 EN**: Executes a call or declaration centered on `ops[j].getRhs`.
  **L314 CN**: 执行以 `ops[j].getRhs` 为核心的调用或声明。
- **L315 EN**: Initializes variable `itRhs` from the right-hand expression.
  **L315 CN**: 使用右侧表达式初始化变量 `itRhs`。
- **L316 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L316 CN**: 开始 `if` 控制流语句并计算其条件。
- **L317 EN**: Skips to the next loop iteration.
  **L317 CN**: 跳到下一次循环迭代。
- **L318 EN**: Closes the current lexical scope or compound statement.
  **L318 CN**: 结束当前词法作用域或复合语句块。
- **L319 EN**: Blank line separating nearby declarations or logic blocks.
  **L319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L320 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L320 CN**: 开始 `if` 控制流语句并计算其条件。
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `performShuffle(rewriter, loc, matB, ipType, offset, packedBuffer,`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`performShuffle(rewriter, loc, matB, ipType, offset, packedBuffer,`。
- **L322 EN**: Executes a standalone statement or declaration: `indxToStoreInBuffer);`.
  **L322 CN**: 执行一条独立语句或声明：`indxToStoreInBuffer);`。
- **L323 EN**: Closes the current lexical scope or compound statement.
  **L323 CN**: 结束当前词法作用域或复合语句块。
- **L324 EN**: Blank line separating nearby declarations or logic blocks.
  **L324 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L325 EN**: Continues the surrounding expression or declaration: `amx::TileType tileType =`.
  **L325 CN**: 继续构造周围的表达式或声明：`amx::TileType tileType =`。
- **L326 EN**: Executes a call or declaration centered on `amx::TileType::get`.
  **L326 CN**: 执行以 `amx::TileType::get` 为核心的调用或声明。
- **L327 EN**: Continues the surrounding expression or declaration: `auto loadRow1 =`.
  **L327 CN**: 继续构造周围的表达式或声明：`auto loadRow1 =`。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `amx::TileLoadOp::create(rewriter, loc, tileType, packedBuffer,`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`amx::TileLoadOp::create(rewriter, loc, tileType, packedBuffer,`。
- **L329 EN**: Executes a standalone statement or declaration: `ValueRange{indxToLoadFromMatB, c0, c0});`.
  **L329 CN**: 执行一条独立语句或声明：`ValueRange{indxToLoadFromMatB, c0, c0});`。
- **L330 EN**: Blank line separating nearby declarations or logic blocks.
  **L330 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L331 EN**: Continues the surrounding expression or declaration: `auto loadRow2 =`.
  **L331 CN**: 继续构造周围的表达式或声明：`auto loadRow2 =`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `amx::TileLoadOp::create(rewriter, loc, tileType, packedBuffer,`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`amx::TileLoadOp::create(rewriter, loc, tileType, packedBuffer,`。
- **L333 EN**: Executes a standalone statement or declaration: `ValueRange{indxToLoadFromMatB, c16, c0});`.
  **L333 CN**: 执行一条独立语句或声明：`ValueRange{indxToLoadFromMatB, c16, c0});`。
- **L334 EN**: Blank line separating nearby declarations or logic blocks.
  **L334 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L335 EN**: Executes a call or declaration centered on `readsToTileLoads.try_emplace`.
  **L335 CN**: 执行以 `readsToTileLoads.try_emplace` 为核心的调用或声明。
- **L336 EN**: Executes a call or declaration centered on `readsToTileLoads.try_emplace`.
  **L336 CN**: 执行以 `readsToTileLoads.try_emplace` 为核心的调用或声明。

### Lines 337-360

````cpp
      }
    }
  }

  return readsToTileLoads;
}

// Creates tiled amx dot-products.
static SmallVector<Value>
createTiledDp(OpBuilder &rewriter, Location loc,
              SmallVector<vector::ContractionOp> ops, Value matA, Value matB,
              Type ipType, Type opType, ValueRange accIterArgs,
              unsigned int offset, bool isVnni, Value packedBuffer, bool pack,
              Value indxToStoreInBuffer, Value indxToLoadFromMatB) {

  if (isVnni) {
    matA = collapseInnerDims(rewriter, loc, matA);
    matB = collapseInnerDims(rewriter, loc, matB);
  }

  SmallVector<Value> accumulators;
  // Stores the amx.tile_load operation vs it's equivalent vector tranfer_read
  // or load operations.
  llvm::DenseMap<Operation *, amx::TileLoadOp> readsToTileLoads;
````
- **L337 EN**: Closes the current lexical scope or compound statement.
  **L337 CN**: 结束当前词法作用域或复合语句块。
- **L338 EN**: Closes the current lexical scope or compound statement.
  **L338 CN**: 结束当前词法作用域或复合语句块。
- **L339 EN**: Closes the current lexical scope or compound statement.
  **L339 CN**: 结束当前词法作用域或复合语句块。
- **L340 EN**: Blank line separating nearby declarations or logic blocks.
  **L340 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L341 EN**: Returns from the current function with `readsToTileLoads`.
  **L341 CN**: 以 `readsToTileLoads` 从当前函数返回。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Blank line separating nearby declarations or logic blocks.
  **L343 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L344 EN**: Comment explains nearby logic, invariants, or intent: `Creates tiled amx dot-products.`.
  **L344 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Creates tiled amx dot-products.`。
- **L345 EN**: Continues the surrounding expression or declaration: `static SmallVector<Value>`.
  **L345 CN**: 继续构造周围的表达式或声明：`static SmallVector<Value>`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `createTiledDp(OpBuilder &rewriter, Location loc,`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`createTiledDp(OpBuilder &rewriter, Location loc,`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<vector::ContractionOp> ops, Value matA, Value matB,`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<vector::ContractionOp> ops, Value matA, Value matB,`。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Type ipType, Type opType, ValueRange accIterArgs,`.
  **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`Type ipType, Type opType, ValueRange accIterArgs,`。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned int offset, bool isVnni, Value packedBuffer, bool pack,`.
  **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned int offset, bool isVnni, Value packedBuffer, bool pack,`。
- **L350 EN**: Continues the surrounding expression or declaration: `Value indxToStoreInBuffer, Value indxToLoadFromMatB) {`.
  **L350 CN**: 继续构造周围的表达式或声明：`Value indxToStoreInBuffer, Value indxToLoadFromMatB) {`。
- **L351 EN**: Blank line separating nearby declarations or logic blocks.
  **L351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L352 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L352 CN**: 开始 `if` 控制流语句并计算其条件。
- **L353 EN**: Executes a call or declaration centered on `collapseInnerDims`.
  **L353 CN**: 执行以 `collapseInnerDims` 为核心的调用或声明。
- **L354 EN**: Executes a call or declaration centered on `collapseInnerDims`.
  **L354 CN**: 执行以 `collapseInnerDims` 为核心的调用或声明。
- **L355 EN**: Closes the current lexical scope or compound statement.
  **L355 CN**: 结束当前词法作用域或复合语句块。
- **L356 EN**: Blank line separating nearby declarations or logic blocks.
  **L356 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L357 EN**: Executes a standalone statement or declaration: `SmallVector<Value> accumulators;`.
  **L357 CN**: 执行一条独立语句或声明：`SmallVector<Value> accumulators;`。
- **L358 EN**: Comment explains nearby logic, invariants, or intent: `Stores the amx.tile_load operation vs it's equivalent vector tranfer_read`.
  **L358 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Stores the amx.tile_load operation vs it's equivalent vector tranfer_read`。
- **L359 EN**: Comment explains nearby logic, invariants, or intent: `or load operations.`.
  **L359 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`or load operations.`。
- **L360 EN**: Executes a standalone statement or declaration: `llvm::DenseMap<Operation *, amx::TileLoadOp> readsToTileLoads;`.
  **L360 CN**: 执行一条独立语句或声明：`llvm::DenseMap<Operation *, amx::TileLoadOp> readsToTileLoads;`。

### Lines 361-384

````cpp

  // function call to online pack the input  B matrix
  if (!isVnni) {
    readsToTileLoads =
        packInputs(rewriter, loc, ops, matB, ipType, offset, packedBuffer, pack,
                   indxToStoreInBuffer, indxToLoadFromMatB);
  }

  // Iterate over the contraction operations and compute the tiled dot-product.
  for (size_t i = 0; i < ops.size(); i++) {

    Operation *readOpLhs = ops[i].getLhs().getDefiningOp();
    amx::TileLoadOp tilesLhs;
    auto itLhs = readsToTileLoads.find(readOpLhs);
    if (itLhs != readsToTileLoads.end()) {
      tilesLhs = itLhs->second;
    } else {
      tilesLhs = createTileLoads(rewriter, loc, ops[i].getLhs(), matA, ipType,
                                 false, offset, isVnni);
      readsToTileLoads.try_emplace(readOpLhs, tilesLhs);
    }

    Operation *readOpRhs = ops[i].getRhs().getDefiningOp();
    amx::TileLoadOp tilesRhs;
````
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L362 EN**: Comment explains nearby logic, invariants, or intent: `function call to online pack the input  B matrix`.
  **L362 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`function call to online pack the input  B matrix`。
- **L363 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L363 CN**: 开始 `if` 控制流语句并计算其条件。
- **L364 EN**: Continues the surrounding expression or declaration: `readsToTileLoads =`.
  **L364 CN**: 继续构造周围的表达式或声明：`readsToTileLoads =`。
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `packInputs(rewriter, loc, ops, matB, ipType, offset, packedBuffer, pack,`.
  **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`packInputs(rewriter, loc, ops, matB, ipType, offset, packedBuffer, pack,`。
- **L366 EN**: Executes a standalone statement or declaration: `indxToStoreInBuffer, indxToLoadFromMatB);`.
  **L366 CN**: 执行一条独立语句或声明：`indxToStoreInBuffer, indxToLoadFromMatB);`。
- **L367 EN**: Closes the current lexical scope or compound statement.
  **L367 CN**: 结束当前词法作用域或复合语句块。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L369 EN**: Comment explains nearby logic, invariants, or intent: `Iterate over the contraction operations and compute the tiled dot-product.`.
  **L369 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Iterate over the contraction operations and compute the tiled dot-product.`。
- **L370 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L370 CN**: 开始 `for` 控制流语句并计算其条件。
- **L371 EN**: Blank line separating nearby declarations or logic blocks.
  **L371 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L372 EN**: Executes a call or declaration centered on `ops[i].getLhs`.
  **L372 CN**: 执行以 `ops[i].getLhs` 为核心的调用或声明。
- **L373 EN**: Executes a standalone statement or declaration: `amx::TileLoadOp tilesLhs;`.
  **L373 CN**: 执行一条独立语句或声明：`amx::TileLoadOp tilesLhs;`。
- **L374 EN**: Initializes variable `itLhs` from the right-hand expression.
  **L374 CN**: 使用右侧表达式初始化变量 `itLhs`。
- **L375 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L375 CN**: 开始 `if` 控制流语句并计算其条件。
- **L376 EN**: Executes a standalone statement or declaration: `tilesLhs = itLhs->second;`.
  **L376 CN**: 执行一条独立语句或声明：`tilesLhs = itLhs->second;`。
- **L377 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L377 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tilesLhs = createTileLoads(rewriter, loc, ops[i].getLhs(), matA, ipType,`.
  **L378 CN**: 继续一个多行参数列表、初始化器或聚合项：`tilesLhs = createTileLoads(rewriter, loc, ops[i].getLhs(), matA, ipType,`。
- **L379 EN**: Executes a standalone statement or declaration: `false, offset, isVnni);`.
  **L379 CN**: 执行一条独立语句或声明：`false, offset, isVnni);`。
- **L380 EN**: Executes a call or declaration centered on `readsToTileLoads.try_emplace`.
  **L380 CN**: 执行以 `readsToTileLoads.try_emplace` 为核心的调用或声明。
- **L381 EN**: Closes the current lexical scope or compound statement.
  **L381 CN**: 结束当前词法作用域或复合语句块。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L383 EN**: Executes a call or declaration centered on `ops[i].getRhs`.
  **L383 CN**: 执行以 `ops[i].getRhs` 为核心的调用或声明。
- **L384 EN**: Executes a standalone statement or declaration: `amx::TileLoadOp tilesRhs;`.
  **L384 CN**: 执行一条独立语句或声明：`amx::TileLoadOp tilesRhs;`。

### Lines 385-408

````cpp
    auto itRhs = readsToTileLoads.find(readOpRhs);
    if (itRhs != readsToTileLoads.end()) {
      tilesRhs = itRhs->second;
    } else {
      tilesRhs = createTileLoads(rewriter, loc, ops[i].getRhs(), matB, ipType,
                                 true, offset, isVnni);
      readsToTileLoads.try_emplace(readOpRhs, tilesRhs);
    }

    auto accTileType = amx::TileType::get({16, 16}, opType);

    Value dp;
    if (ipType.isBF16())
      dp = amx::TileMulFOp::create(rewriter, loc, accTileType, tilesLhs,
                                   tilesRhs, accIterArgs[i]);

    if (ipType.isSignlessInteger(8))
      dp = amx::TileMulIOp::create(rewriter, loc, accTileType, tilesLhs,
                                   tilesRhs, accIterArgs[i]);

    accumulators.push_back(dp);
  }
  return accumulators;
}
````
- **L385 EN**: Initializes variable `itRhs` from the right-hand expression.
  **L385 CN**: 使用右侧表达式初始化变量 `itRhs`。
- **L386 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L386 CN**: 开始 `if` 控制流语句并计算其条件。
- **L387 EN**: Executes a standalone statement or declaration: `tilesRhs = itRhs->second;`.
  **L387 CN**: 执行一条独立语句或声明：`tilesRhs = itRhs->second;`。
- **L388 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L388 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tilesRhs = createTileLoads(rewriter, loc, ops[i].getRhs(), matB, ipType,`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`tilesRhs = createTileLoads(rewriter, loc, ops[i].getRhs(), matB, ipType,`。
- **L390 EN**: Executes a standalone statement or declaration: `true, offset, isVnni);`.
  **L390 CN**: 执行一条独立语句或声明：`true, offset, isVnni);`。
- **L391 EN**: Executes a call or declaration centered on `readsToTileLoads.try_emplace`.
  **L391 CN**: 执行以 `readsToTileLoads.try_emplace` 为核心的调用或声明。
- **L392 EN**: Closes the current lexical scope or compound statement.
  **L392 CN**: 结束当前词法作用域或复合语句块。
- **L393 EN**: Blank line separating nearby declarations or logic blocks.
  **L393 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L394 EN**: Initializes variable `accTileType` from the right-hand expression.
  **L394 CN**: 使用右侧表达式初始化变量 `accTileType`。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L396 EN**: Executes a standalone statement or declaration: `Value dp;`.
  **L396 CN**: 执行一条独立语句或声明：`Value dp;`。
- **L397 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L397 CN**: 开始 `if` 控制流语句并计算其条件。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dp = amx::TileMulFOp::create(rewriter, loc, accTileType, tilesLhs,`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`dp = amx::TileMulFOp::create(rewriter, loc, accTileType, tilesLhs,`。
- **L399 EN**: Executes a standalone statement or declaration: `tilesRhs, accIterArgs[i]);`.
  **L399 CN**: 执行一条独立语句或声明：`tilesRhs, accIterArgs[i]);`。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L401 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L401 CN**: 开始 `if` 控制流语句并计算其条件。
- **L402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dp = amx::TileMulIOp::create(rewriter, loc, accTileType, tilesLhs,`.
  **L402 CN**: 继续一个多行参数列表、初始化器或聚合项：`dp = amx::TileMulIOp::create(rewriter, loc, accTileType, tilesLhs,`。
- **L403 EN**: Executes a standalone statement or declaration: `tilesRhs, accIterArgs[i]);`.
  **L403 CN**: 执行一条独立语句或声明：`tilesRhs, accIterArgs[i]);`。
- **L404 EN**: Blank line separating nearby declarations or logic blocks.
  **L404 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L405 EN**: Executes a call or declaration centered on `accumulators.push_back`.
  **L405 CN**: 执行以 `accumulators.push_back` 为核心的调用或声明。
- **L406 EN**: Closes the current lexical scope or compound statement.
  **L406 CN**: 结束当前词法作用域或复合语句块。
- **L407 EN**: Returns from the current function with `accumulators`.
  **L407 CN**: 以 `accumulators` 从当前函数返回。
- **L408 EN**: Closes the current lexical scope or compound statement.
  **L408 CN**: 结束当前词法作用域或复合语句块。

### Lines 409-432

````cpp

static SmallVector<Value> createTileZeros(OpBuilder &rewriter, Location loc,
                                          Type opType, scf::ForOp outerLoop,
                                          int64_t size) {
  rewriter.setInsertionPoint(outerLoop);

  SmallVector<Value> loopItrArgs;
  auto zeroTileType = amx::TileType::get({16, 16}, opType);

  for (int i = 0; i < size; i++) {
    auto zeroTile = amx::TileZeroOp::create(rewriter, loc, zeroTileType);
    loopItrArgs.push_back(zeroTile);
  }
  return loopItrArgs;
}

static Value getIndxToLoadStoreFromPckBuffer(
    OpBuilder &rewriter, Location loc, Value ivInnerLoop, Value ivOuterLoop,
    bool isInnerLoopUBHasOddQuot, bool isInnerLoopUBLarger, bool pack,
    unsigned int blockingFactor) {

  Value c2 = arith::ConstantIndexOp::create(rewriter, loc, 2);
  Value packOffset =
      arith::ConstantIndexOp::create(rewriter, loc, (16 * blockingFactor));
````
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static SmallVector<Value> createTileZeros(OpBuilder &rewriter, Location loc,`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`static SmallVector<Value> createTileZeros(OpBuilder &rewriter, Location loc,`。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Type opType, scf::ForOp outerLoop,`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`Type opType, scf::ForOp outerLoop,`。
- **L412 EN**: Continues the surrounding expression or declaration: `int64_t size) {`.
  **L412 CN**: 继续构造周围的表达式或声明：`int64_t size) {`。
- **L413 EN**: Executes a call or declaration centered on `rewriter.setInsertionPoint`.
  **L413 CN**: 执行以 `rewriter.setInsertionPoint` 为核心的调用或声明。
- **L414 EN**: Blank line separating nearby declarations or logic blocks.
  **L414 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L415 EN**: Executes a standalone statement or declaration: `SmallVector<Value> loopItrArgs;`.
  **L415 CN**: 执行一条独立语句或声明：`SmallVector<Value> loopItrArgs;`。
- **L416 EN**: Initializes variable `zeroTileType` from the right-hand expression.
  **L416 CN**: 使用右侧表达式初始化变量 `zeroTileType`。
- **L417 EN**: Blank line separating nearby declarations or logic blocks.
  **L417 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L418 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L418 CN**: 开始 `for` 控制流语句并计算其条件。
- **L419 EN**: Initializes variable `zeroTile` from the right-hand expression.
  **L419 CN**: 使用右侧表达式初始化变量 `zeroTile`。
- **L420 EN**: Executes a call or declaration centered on `loopItrArgs.push_back`.
  **L420 CN**: 执行以 `loopItrArgs.push_back` 为核心的调用或声明。
- **L421 EN**: Closes the current lexical scope or compound statement.
  **L421 CN**: 结束当前词法作用域或复合语句块。
- **L422 EN**: Returns from the current function with `loopItrArgs`.
  **L422 CN**: 以 `loopItrArgs` 从当前函数返回。
- **L423 EN**: Closes the current lexical scope or compound statement.
  **L423 CN**: 结束当前词法作用域或复合语句块。
- **L424 EN**: Blank line separating nearby declarations or logic blocks.
  **L424 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L425 EN**: Continues logic associated with callable symbol `getIndxToLoadStoreFromPckBuffer`.
  **L425 CN**: 继续与可调用符号 `getIndxToLoadStoreFromPckBuffer` 相关的逻辑。
- **L426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OpBuilder &rewriter, Location loc, Value ivInnerLoop, Value ivOuterLoop,`.
  **L426 CN**: 继续一个多行参数列表、初始化器或聚合项：`OpBuilder &rewriter, Location loc, Value ivInnerLoop, Value ivOuterLoop,`。
- **L427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool isInnerLoopUBHasOddQuot, bool isInnerLoopUBLarger, bool pack,`.
  **L427 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool isInnerLoopUBHasOddQuot, bool isInnerLoopUBLarger, bool pack,`。
- **L428 EN**: Continues the surrounding expression or declaration: `unsigned int blockingFactor) {`.
  **L428 CN**: 继续构造周围的表达式或声明：`unsigned int blockingFactor) {`。
- **L429 EN**: Blank line separating nearby declarations or logic blocks.
  **L429 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L430 EN**: Initializes variable `c2` from the right-hand expression.
  **L430 CN**: 使用右侧表达式初始化变量 `c2`。
- **L431 EN**: Continues the surrounding expression or declaration: `Value packOffset =`.
  **L431 CN**: 继续构造周围的表达式或声明：`Value packOffset =`。
- **L432 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L432 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。

### Lines 433-456

````cpp

  Value quotientInnerLoop =
      arith::DivUIOp::create(rewriter, loc, ivInnerLoop, packOffset);
  Value remInnerLoop = arith::RemUIOp::create(
      rewriter, loc, rewriter.getIndexType(), quotientInnerLoop, c2);

  if (!isInnerLoopUBLarger && !pack) {
    remInnerLoop = arith::RemUIOp::create(
        rewriter, loc, rewriter.getIndexType(), ivOuterLoop, c2);
  }

  if (isInnerLoopUBHasOddQuot) {
    auto remOuterLoop = arith::RemUIOp::create(
        rewriter, loc, rewriter.getIndexType(), ivOuterLoop, c2);
    auto remAdd = arith::AddIOp::create(rewriter, loc, rewriter.getIndexType(),
                                        remInnerLoop, remOuterLoop);
    remInnerLoop = arith::RemUIOp::create(rewriter, loc,
                                          rewriter.getIndexType(), remAdd, c2);
  }

  return remInnerLoop;
}

static scf::ForOp
````
- **L433 EN**: Blank line separating nearby declarations or logic blocks.
  **L433 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L434 EN**: Continues the surrounding expression or declaration: `Value quotientInnerLoop =`.
  **L434 CN**: 继续构造周围的表达式或声明：`Value quotientInnerLoop =`。
- **L435 EN**: Executes a call or declaration centered on `arith::DivUIOp::create`.
  **L435 CN**: 执行以 `arith::DivUIOp::create` 为核心的调用或声明。
- **L436 EN**: Continues logic associated with callable symbol `create`.
  **L436 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L437 EN**: Executes a call or declaration centered on `rewriter.getIndexType`.
  **L437 CN**: 执行以 `rewriter.getIndexType` 为核心的调用或声明。
- **L438 EN**: Blank line separating nearby declarations or logic blocks.
  **L438 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L439 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L439 CN**: 开始 `if` 控制流语句并计算其条件。
- **L440 EN**: Continues logic associated with callable symbol `create`.
  **L440 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L441 EN**: Executes a call or declaration centered on `rewriter.getIndexType`.
  **L441 CN**: 执行以 `rewriter.getIndexType` 为核心的调用或声明。
- **L442 EN**: Closes the current lexical scope or compound statement.
  **L442 CN**: 结束当前词法作用域或复合语句块。
- **L443 EN**: Blank line separating nearby declarations or logic blocks.
  **L443 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L444 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L444 CN**: 开始 `if` 控制流语句并计算其条件。
- **L445 EN**: Continues logic associated with callable symbol `create`.
  **L445 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L446 EN**: Executes a call or declaration centered on `rewriter.getIndexType`.
  **L446 CN**: 执行以 `rewriter.getIndexType` 为核心的调用或声明。
- **L447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto remAdd = arith::AddIOp::create(rewriter, loc, rewriter.getIndexType(),`.
  **L447 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto remAdd = arith::AddIOp::create(rewriter, loc, rewriter.getIndexType(),`。
- **L448 EN**: Executes a standalone statement or declaration: `remInnerLoop, remOuterLoop);`.
  **L448 CN**: 执行一条独立语句或声明：`remInnerLoop, remOuterLoop);`。
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `remInnerLoop = arith::RemUIOp::create(rewriter, loc,`.
  **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`remInnerLoop = arith::RemUIOp::create(rewriter, loc,`。
- **L450 EN**: Executes a call or declaration centered on `rewriter.getIndexType`.
  **L450 CN**: 执行以 `rewriter.getIndexType` 为核心的调用或声明。
- **L451 EN**: Closes the current lexical scope or compound statement.
  **L451 CN**: 结束当前词法作用域或复合语句块。
- **L452 EN**: Blank line separating nearby declarations or logic blocks.
  **L452 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L453 EN**: Returns from the current function with `remInnerLoop`.
  **L453 CN**: 以 `remInnerLoop` 从当前函数返回。
- **L454 EN**: Closes the current lexical scope or compound statement.
  **L454 CN**: 结束当前词法作用域或复合语句块。
- **L455 EN**: Blank line separating nearby declarations or logic blocks.
  **L455 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L456 EN**: Continues the surrounding expression or declaration: `static scf::ForOp`.
  **L456 CN**: 继续构造周围的表达式或声明：`static scf::ForOp`。

### Lines 457-480

````cpp
createLoops(OpBuilder &rewriter, Location loc, Value lowerBound,
            Value upperBound, Value step, SmallVector<Value> loopItrArgs,
            Type ipType, Type opType, unsigned int blockingFactor, bool isVnni,
            Operation *vectorOpLhs, Operation *vectorOpRhs,
            vector::ContractionOp contractOp, scf::ForOp outerLoop,
            scf::ForOp innerLoop, SmallVector<vector::ContractionOp> ops,
            Value ivOuterLoop, Value packedBuffer, bool pack,
            arith::ConstantIndexOp innerLoopIndex, bool isInnerLoopUBLarger,
            bool isInnerLoopUBHasOddQuot) {

  Value c0 = arith::ConstantIndexOp::create(rewriter, loc, 0);
  Value c1 = arith::ConstantIndexOp::create(rewriter, loc, 1);
  Value c2 = arith::ConstantIndexOp::create(rewriter, loc, 2);

  auto newLoop = scf::ForOp::create(
      rewriter, loc, lowerBound, upperBound, step, loopItrArgs,
      [&](OpBuilder &rewriterNewInnerLoop, Location locNewInnerLoop,
          Value ivNewInnerLoop, ValueRange iterArgsNewInnerLoop) {
        IRMapping mapping;
        if (outerLoop)
          mapping.map(vectorOpLhs->getOperand(
                          getIndexPosition(contractOp.getLhs(), outerLoop) + 1),
                      ivOuterLoop);

````
- **L457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `createLoops(OpBuilder &rewriter, Location loc, Value lowerBound,`.
  **L457 CN**: 继续一个多行参数列表、初始化器或聚合项：`createLoops(OpBuilder &rewriter, Location loc, Value lowerBound,`。
- **L458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value upperBound, Value step, SmallVector<Value> loopItrArgs,`.
  **L458 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value upperBound, Value step, SmallVector<Value> loopItrArgs,`。
- **L459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Type ipType, Type opType, unsigned int blockingFactor, bool isVnni,`.
  **L459 CN**: 继续一个多行参数列表、初始化器或聚合项：`Type ipType, Type opType, unsigned int blockingFactor, bool isVnni,`。
- **L460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Operation *vectorOpLhs, Operation *vectorOpRhs,`.
  **L460 CN**: 继续一个多行参数列表、初始化器或聚合项：`Operation *vectorOpLhs, Operation *vectorOpRhs,`。
- **L461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::ContractionOp contractOp, scf::ForOp outerLoop,`.
  **L461 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::ContractionOp contractOp, scf::ForOp outerLoop,`。
- **L462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `scf::ForOp innerLoop, SmallVector<vector::ContractionOp> ops,`.
  **L462 CN**: 继续一个多行参数列表、初始化器或聚合项：`scf::ForOp innerLoop, SmallVector<vector::ContractionOp> ops,`。
- **L463 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value ivOuterLoop, Value packedBuffer, bool pack,`.
  **L463 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value ivOuterLoop, Value packedBuffer, bool pack,`。
- **L464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arith::ConstantIndexOp innerLoopIndex, bool isInnerLoopUBLarger,`.
  **L464 CN**: 继续一个多行参数列表、初始化器或聚合项：`arith::ConstantIndexOp innerLoopIndex, bool isInnerLoopUBLarger,`。
- **L465 EN**: Continues the surrounding expression or declaration: `bool isInnerLoopUBHasOddQuot) {`.
  **L465 CN**: 继续构造周围的表达式或声明：`bool isInnerLoopUBHasOddQuot) {`。
- **L466 EN**: Blank line separating nearby declarations or logic blocks.
  **L466 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L467 EN**: Initializes variable `c0` from the right-hand expression.
  **L467 CN**: 使用右侧表达式初始化变量 `c0`。
- **L468 EN**: Initializes variable `c1` from the right-hand expression.
  **L468 CN**: 使用右侧表达式初始化变量 `c1`。
- **L469 EN**: Initializes variable `c2` from the right-hand expression.
  **L469 CN**: 使用右侧表达式初始化变量 `c2`。
- **L470 EN**: Blank line separating nearby declarations or logic blocks.
  **L470 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L471 EN**: Continues logic associated with callable symbol `create`.
  **L471 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, lowerBound, upperBound, step, loopItrArgs,`.
  **L472 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, lowerBound, upperBound, step, loopItrArgs,`。
- **L473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](OpBuilder &rewriterNewInnerLoop, Location locNewInnerLoop,`.
  **L473 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](OpBuilder &rewriterNewInnerLoop, Location locNewInnerLoop,`。
- **L474 EN**: Continues the surrounding expression or declaration: `Value ivNewInnerLoop, ValueRange iterArgsNewInnerLoop) {`.
  **L474 CN**: 继续构造周围的表达式或声明：`Value ivNewInnerLoop, ValueRange iterArgsNewInnerLoop) {`。
- **L475 EN**: Executes a standalone statement or declaration: `IRMapping mapping;`.
  **L475 CN**: 执行一条独立语句或声明：`IRMapping mapping;`。
- **L476 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L476 CN**: 开始 `if` 控制流语句并计算其条件。
- **L477 EN**: Continues logic associated with callable symbol `map`.
  **L477 CN**: 继续与可调用符号 `map` 相关的逻辑。
- **L478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getIndexPosition(contractOp.getLhs(), outerLoop) + 1),`.
  **L478 CN**: 继续一个多行参数列表、初始化器或聚合项：`getIndexPosition(contractOp.getLhs(), outerLoop) + 1),`。
- **L479 EN**: Executes a standalone statement or declaration: `ivOuterLoop);`.
  **L479 CN**: 执行一条独立语句或声明：`ivOuterLoop);`。
- **L480 EN**: Blank line separating nearby declarations or logic blocks.
  **L480 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 481-504

````cpp
        mapping.map(vectorOpLhs->getOperand(
                        getIndexPosition(contractOp.getLhs(), innerLoop) + 1),
                    ivNewInnerLoop);
        auto lhsClone = rewriterNewInnerLoop.clone(*vectorOpLhs, mapping);

        Value indxToStoreInBuffer = c0;
        Value indxToLoadFromBuffer = c0;

        if (!isVnni) {
          if (outerLoop) {
            if (innerLoopIndex.value() == 0) {
              if (pack) {
                ivNewInnerLoop = c0;
                ivOuterLoop = arith::AddIOp::create(rewriter, locNewInnerLoop,
                                                    c1, ivOuterLoop);

                if (!isInnerLoopUBLarger || isInnerLoopUBHasOddQuot) {
                  indxToStoreInBuffer = arith::RemUIOp::create(
                      rewriter, locNewInnerLoop, rewriter.getIndexType(),
                      ivOuterLoop, c2);
                }

                Value indxToLoadFromMatB = arith::AddIOp::create(
                    rewriter, loc, indxToStoreInBuffer, c1);
````
- **L481 EN**: Continues logic associated with callable symbol `map`.
  **L481 CN**: 继续与可调用符号 `map` 相关的逻辑。
- **L482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getIndexPosition(contractOp.getLhs(), innerLoop) + 1),`.
  **L482 CN**: 继续一个多行参数列表、初始化器或聚合项：`getIndexPosition(contractOp.getLhs(), innerLoop) + 1),`。
- **L483 EN**: Executes a standalone statement or declaration: `ivNewInnerLoop);`.
  **L483 CN**: 执行一条独立语句或声明：`ivNewInnerLoop);`。
- **L484 EN**: Initializes variable `lhsClone` from the right-hand expression.
  **L484 CN**: 使用右侧表达式初始化变量 `lhsClone`。
- **L485 EN**: Blank line separating nearby declarations or logic blocks.
  **L485 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L486 EN**: Initializes variable `indxToStoreInBuffer` from the right-hand expression.
  **L486 CN**: 使用右侧表达式初始化变量 `indxToStoreInBuffer`。
- **L487 EN**: Initializes variable `indxToLoadFromBuffer` from the right-hand expression.
  **L487 CN**: 使用右侧表达式初始化变量 `indxToLoadFromBuffer`。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L489 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L489 CN**: 开始 `if` 控制流语句并计算其条件。
- **L490 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L490 CN**: 开始 `if` 控制流语句并计算其条件。
- **L491 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L491 CN**: 开始 `if` 控制流语句并计算其条件。
- **L492 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L492 CN**: 开始 `if` 控制流语句并计算其条件。
- **L493 EN**: Executes a standalone statement or declaration: `ivNewInnerLoop = c0;`.
  **L493 CN**: 执行一条独立语句或声明：`ivNewInnerLoop = c0;`。
- **L494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ivOuterLoop = arith::AddIOp::create(rewriter, locNewInnerLoop,`.
  **L494 CN**: 继续一个多行参数列表、初始化器或聚合项：`ivOuterLoop = arith::AddIOp::create(rewriter, locNewInnerLoop,`。
- **L495 EN**: Executes a standalone statement or declaration: `c1, ivOuterLoop);`.
  **L495 CN**: 执行一条独立语句或声明：`c1, ivOuterLoop);`。
- **L496 EN**: Blank line separating nearby declarations or logic blocks.
  **L496 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L497 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L497 CN**: 开始 `if` 控制流语句并计算其条件。
- **L498 EN**: Continues logic associated with callable symbol `create`.
  **L498 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, locNewInnerLoop, rewriter.getIndexType(),`.
  **L499 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, locNewInnerLoop, rewriter.getIndexType(),`。
- **L500 EN**: Executes a standalone statement or declaration: `ivOuterLoop, c2);`.
  **L500 CN**: 执行一条独立语句或声明：`ivOuterLoop, c2);`。
- **L501 EN**: Closes the current lexical scope or compound statement.
  **L501 CN**: 结束当前词法作用域或复合语句块。
- **L502 EN**: Blank line separating nearby declarations or logic blocks.
  **L502 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L503 EN**: Continues logic associated with callable symbol `create`.
  **L503 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L504 EN**: Executes a standalone statement or declaration: `rewriter, loc, indxToStoreInBuffer, c1);`.
  **L504 CN**: 执行一条独立语句或声明：`rewriter, loc, indxToStoreInBuffer, c1);`。

### Lines 505-528

````cpp
                indxToLoadFromBuffer = arith::RemUIOp::create(
                    rewriter, loc, rewriter.getIndexType(), indxToLoadFromMatB,
                    c2);
              }

            } else {
              Value nLoadIndx = arith::ConstantIndexOp::create(
                  rewriter, locNewInnerLoop, (16 * blockingFactor));
              ivNewInnerLoop = arith::AddIOp::create(rewriter, locNewInnerLoop,
                                                     nLoadIndx, ivNewInnerLoop);
              indxToStoreInBuffer = getIndxToLoadStoreFromPckBuffer(
                  rewriter, loc, ivNewInnerLoop, ivOuterLoop,
                  isInnerLoopUBHasOddQuot, isInnerLoopUBLarger, pack,
                  blockingFactor);
              Value indxToLoadFromMatB =
                  arith::AddIOp::create(rewriter, loc, indxToStoreInBuffer, c1);
              indxToLoadFromBuffer =
                  arith::RemUIOp::create(rewriter, loc, rewriter.getIndexType(),
                                         indxToLoadFromMatB, c2);
            }
          } else {
            if (pack) {
              Value nLoadIndx = arith::ConstantIndexOp::create(
                  rewriter, locNewInnerLoop, (16 * blockingFactor));
````
- **L505 EN**: Continues logic associated with callable symbol `create`.
  **L505 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, rewriter.getIndexType(), indxToLoadFromMatB,`.
  **L506 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, rewriter.getIndexType(), indxToLoadFromMatB,`。
- **L507 EN**: Executes a standalone statement or declaration: `c2);`.
  **L507 CN**: 执行一条独立语句或声明：`c2);`。
- **L508 EN**: Closes the current lexical scope or compound statement.
  **L508 CN**: 结束当前词法作用域或复合语句块。
- **L509 EN**: Blank line separating nearby declarations or logic blocks.
  **L509 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L510 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L510 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L511 EN**: Continues logic associated with callable symbol `create`.
  **L511 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L512 EN**: Executes a call or declaration centered on `locNewInnerLoop,`.
  **L512 CN**: 执行以 `locNewInnerLoop,` 为核心的调用或声明。
- **L513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ivNewInnerLoop = arith::AddIOp::create(rewriter, locNewInnerLoop,`.
  **L513 CN**: 继续一个多行参数列表、初始化器或聚合项：`ivNewInnerLoop = arith::AddIOp::create(rewriter, locNewInnerLoop,`。
- **L514 EN**: Executes a standalone statement or declaration: `nLoadIndx, ivNewInnerLoop);`.
  **L514 CN**: 执行一条独立语句或声明：`nLoadIndx, ivNewInnerLoop);`。
- **L515 EN**: Continues logic associated with callable symbol `getIndxToLoadStoreFromPckBuffer`.
  **L515 CN**: 继续与可调用符号 `getIndxToLoadStoreFromPckBuffer` 相关的逻辑。
- **L516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, ivNewInnerLoop, ivOuterLoop,`.
  **L516 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, ivNewInnerLoop, ivOuterLoop,`。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `isInnerLoopUBHasOddQuot, isInnerLoopUBLarger, pack,`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`isInnerLoopUBHasOddQuot, isInnerLoopUBLarger, pack,`。
- **L518 EN**: Executes a standalone statement or declaration: `blockingFactor);`.
  **L518 CN**: 执行一条独立语句或声明：`blockingFactor);`。
- **L519 EN**: Continues the surrounding expression or declaration: `Value indxToLoadFromMatB =`.
  **L519 CN**: 继续构造周围的表达式或声明：`Value indxToLoadFromMatB =`。
- **L520 EN**: Executes a call or declaration centered on `arith::AddIOp::create`.
  **L520 CN**: 执行以 `arith::AddIOp::create` 为核心的调用或声明。
- **L521 EN**: Continues the surrounding expression or declaration: `indxToLoadFromBuffer =`.
  **L521 CN**: 继续构造周围的表达式或声明：`indxToLoadFromBuffer =`。
- **L522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arith::RemUIOp::create(rewriter, loc, rewriter.getIndexType(),`.
  **L522 CN**: 继续一个多行参数列表、初始化器或聚合项：`arith::RemUIOp::create(rewriter, loc, rewriter.getIndexType(),`。
- **L523 EN**: Executes a standalone statement or declaration: `indxToLoadFromMatB, c2);`.
  **L523 CN**: 执行一条独立语句或声明：`indxToLoadFromMatB, c2);`。
- **L524 EN**: Closes the current lexical scope or compound statement.
  **L524 CN**: 结束当前词法作用域或复合语句块。
- **L525 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L525 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L526 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L526 CN**: 开始 `if` 控制流语句并计算其条件。
- **L527 EN**: Continues logic associated with callable symbol `create`.
  **L527 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L528 EN**: Executes a call or declaration centered on `locNewInnerLoop,`.
  **L528 CN**: 执行以 `locNewInnerLoop,` 为核心的调用或声明。

### Lines 529-552

````cpp
              ivNewInnerLoop = arith::AddIOp::create(rewriter, locNewInnerLoop,
                                                     nLoadIndx, ivNewInnerLoop);
              Value quotient_K = arith::DivUIOp::create(
                  rewriter, loc, ivNewInnerLoop, nLoadIndx);
              indxToStoreInBuffer = arith::RemUIOp::create(
                  rewriter, loc, rewriter.getIndexType(), quotient_K, c2);

              Value indxToLoadFromMatB =
                  arith::AddIOp::create(rewriter, loc, indxToStoreInBuffer, c1);
              indxToLoadFromBuffer =
                  arith::RemUIOp::create(rewriter, loc, rewriter.getIndexType(),
                                         indxToLoadFromMatB, c2);
            }
          }
        }

        IRMapping rhsMapping;
        if (outerLoop)
          rhsMapping.map(
              vectorOpRhs->getOperand(
                  getIndexPosition(contractOp.getRhs(), outerLoop) + 1),
              ivOuterLoop);

        rhsMapping.map(
````
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ivNewInnerLoop = arith::AddIOp::create(rewriter, locNewInnerLoop,`.
  **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`ivNewInnerLoop = arith::AddIOp::create(rewriter, locNewInnerLoop,`。
- **L530 EN**: Executes a standalone statement or declaration: `nLoadIndx, ivNewInnerLoop);`.
  **L530 CN**: 执行一条独立语句或声明：`nLoadIndx, ivNewInnerLoop);`。
- **L531 EN**: Continues logic associated with callable symbol `create`.
  **L531 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L532 EN**: Executes a standalone statement or declaration: `rewriter, loc, ivNewInnerLoop, nLoadIndx);`.
  **L532 CN**: 执行一条独立语句或声明：`rewriter, loc, ivNewInnerLoop, nLoadIndx);`。
- **L533 EN**: Continues logic associated with callable symbol `create`.
  **L533 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L534 EN**: Executes a call or declaration centered on `rewriter.getIndexType`.
  **L534 CN**: 执行以 `rewriter.getIndexType` 为核心的调用或声明。
- **L535 EN**: Blank line separating nearby declarations or logic blocks.
  **L535 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L536 EN**: Continues the surrounding expression or declaration: `Value indxToLoadFromMatB =`.
  **L536 CN**: 继续构造周围的表达式或声明：`Value indxToLoadFromMatB =`。
- **L537 EN**: Executes a call or declaration centered on `arith::AddIOp::create`.
  **L537 CN**: 执行以 `arith::AddIOp::create` 为核心的调用或声明。
- **L538 EN**: Continues the surrounding expression or declaration: `indxToLoadFromBuffer =`.
  **L538 CN**: 继续构造周围的表达式或声明：`indxToLoadFromBuffer =`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arith::RemUIOp::create(rewriter, loc, rewriter.getIndexType(),`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`arith::RemUIOp::create(rewriter, loc, rewriter.getIndexType(),`。
- **L540 EN**: Executes a standalone statement or declaration: `indxToLoadFromMatB, c2);`.
  **L540 CN**: 执行一条独立语句或声明：`indxToLoadFromMatB, c2);`。
- **L541 EN**: Closes the current lexical scope or compound statement.
  **L541 CN**: 结束当前词法作用域或复合语句块。
- **L542 EN**: Closes the current lexical scope or compound statement.
  **L542 CN**: 结束当前词法作用域或复合语句块。
- **L543 EN**: Closes the current lexical scope or compound statement.
  **L543 CN**: 结束当前词法作用域或复合语句块。
- **L544 EN**: Blank line separating nearby declarations or logic blocks.
  **L544 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L545 EN**: Executes a standalone statement or declaration: `IRMapping rhsMapping;`.
  **L545 CN**: 执行一条独立语句或声明：`IRMapping rhsMapping;`。
- **L546 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L546 CN**: 开始 `if` 控制流语句并计算其条件。
- **L547 EN**: Continues logic associated with callable symbol `map`.
  **L547 CN**: 继续与可调用符号 `map` 相关的逻辑。
- **L548 EN**: Continues logic associated with callable symbol `getOperand`.
  **L548 CN**: 继续与可调用符号 `getOperand` 相关的逻辑。
- **L549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getIndexPosition(contractOp.getRhs(), outerLoop) + 1),`.
  **L549 CN**: 继续一个多行参数列表、初始化器或聚合项：`getIndexPosition(contractOp.getRhs(), outerLoop) + 1),`。
- **L550 EN**: Executes a standalone statement or declaration: `ivOuterLoop);`.
  **L550 CN**: 执行一条独立语句或声明：`ivOuterLoop);`。
- **L551 EN**: Blank line separating nearby declarations or logic blocks.
  **L551 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L552 EN**: Continues logic associated with callable symbol `map`.
  **L552 CN**: 继续与可调用符号 `map` 相关的逻辑。

### Lines 553-576

````cpp
            vectorOpRhs->getOperand(
                getIndexPosition(contractOp.getRhs(), innerLoop) + 1),
            ivNewInnerLoop);
        auto rhsClone = rewriterNewInnerLoop.clone(*vectorOpRhs, rhsMapping);

        Value matB = rhsClone->getResult(0);

        if (!isVnni) {
          if (outerLoop) {
            if (!pack) {
              Value nLoadIndx = arith::ConstantIndexOp::create(
                  rewriter, locNewInnerLoop, (16 * blockingFactor));
              matB = Value();
              indxToLoadFromBuffer = c0;
              indxToLoadFromBuffer = getIndxToLoadStoreFromPckBuffer(
                  rewriter, loc, nLoadIndx, ivOuterLoop,
                  isInnerLoopUBHasOddQuot, isInnerLoopUBLarger, pack,
                  blockingFactor);
            }
          } else {
            if (!pack) {
              Value nLoadIndx = arith::ConstantIndexOp::create(
                  rewriter, locNewInnerLoop, (16 * blockingFactor));
              matB = Value();
````
- **L553 EN**: Continues logic associated with callable symbol `getOperand`.
  **L553 CN**: 继续与可调用符号 `getOperand` 相关的逻辑。
- **L554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getIndexPosition(contractOp.getRhs(), innerLoop) + 1),`.
  **L554 CN**: 继续一个多行参数列表、初始化器或聚合项：`getIndexPosition(contractOp.getRhs(), innerLoop) + 1),`。
- **L555 EN**: Executes a standalone statement or declaration: `ivNewInnerLoop);`.
  **L555 CN**: 执行一条独立语句或声明：`ivNewInnerLoop);`。
- **L556 EN**: Initializes variable `rhsClone` from the right-hand expression.
  **L556 CN**: 使用右侧表达式初始化变量 `rhsClone`。
- **L557 EN**: Blank line separating nearby declarations or logic blocks.
  **L557 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L558 EN**: Initializes variable `matB` from the right-hand expression.
  **L558 CN**: 使用右侧表达式初始化变量 `matB`。
- **L559 EN**: Blank line separating nearby declarations or logic blocks.
  **L559 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L560 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L560 CN**: 开始 `if` 控制流语句并计算其条件。
- **L561 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L561 CN**: 开始 `if` 控制流语句并计算其条件。
- **L562 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L562 CN**: 开始 `if` 控制流语句并计算其条件。
- **L563 EN**: Continues logic associated with callable symbol `create`.
  **L563 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L564 EN**: Executes a call or declaration centered on `locNewInnerLoop,`.
  **L564 CN**: 执行以 `locNewInnerLoop,` 为核心的调用或声明。
- **L565 EN**: Executes a call or declaration centered on `Value`.
  **L565 CN**: 执行以 `Value` 为核心的调用或声明。
- **L566 EN**: Executes a standalone statement or declaration: `indxToLoadFromBuffer = c0;`.
  **L566 CN**: 执行一条独立语句或声明：`indxToLoadFromBuffer = c0;`。
- **L567 EN**: Continues logic associated with callable symbol `getIndxToLoadStoreFromPckBuffer`.
  **L567 CN**: 继续与可调用符号 `getIndxToLoadStoreFromPckBuffer` 相关的逻辑。
- **L568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, nLoadIndx, ivOuterLoop,`.
  **L568 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, nLoadIndx, ivOuterLoop,`。
- **L569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `isInnerLoopUBHasOddQuot, isInnerLoopUBLarger, pack,`.
  **L569 CN**: 继续一个多行参数列表、初始化器或聚合项：`isInnerLoopUBHasOddQuot, isInnerLoopUBLarger, pack,`。
- **L570 EN**: Executes a standalone statement or declaration: `blockingFactor);`.
  **L570 CN**: 执行一条独立语句或声明：`blockingFactor);`。
- **L571 EN**: Closes the current lexical scope or compound statement.
  **L571 CN**: 结束当前词法作用域或复合语句块。
- **L572 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L572 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L573 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L573 CN**: 开始 `if` 控制流语句并计算其条件。
- **L574 EN**: Continues logic associated with callable symbol `create`.
  **L574 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L575 EN**: Executes a call or declaration centered on `locNewInnerLoop,`.
  **L575 CN**: 执行以 `locNewInnerLoop,` 为核心的调用或声明。
- **L576 EN**: Executes a call or declaration centered on `Value`.
  **L576 CN**: 执行以 `Value` 为核心的调用或声明。

### Lines 577-600

````cpp
              Value quotient_K = arith::DivUIOp::create(
                  rewriter, loc, ivNewInnerLoop, nLoadIndx);
              indxToLoadFromBuffer = arith::RemUIOp::create(
                  rewriter, loc, rewriter.getIndexType(), quotient_K, c2);
            }
          }
        }

        // compute tiled dot-product
        SmallVector<Value> accumulators = createTiledDp(
            rewriter, locNewInnerLoop, ops, lhsClone->getResult(0), matB,
            ipType, opType, iterArgsNewInnerLoop, blockingFactor, isVnni,
            packedBuffer, pack, indxToStoreInBuffer, indxToLoadFromBuffer);

        scf::YieldOp::create(rewriterNewInnerLoop, locNewInnerLoop,
                             accumulators);
      });

  return newLoop;
}

// Implements tiled dot-product operation for a vector.contract operation or a
// sequence of vector.contracts inside the reduction loops.
//
````
- **L577 EN**: Continues logic associated with callable symbol `create`.
  **L577 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L578 EN**: Executes a standalone statement or declaration: `rewriter, loc, ivNewInnerLoop, nLoadIndx);`.
  **L578 CN**: 执行一条独立语句或声明：`rewriter, loc, ivNewInnerLoop, nLoadIndx);`。
- **L579 EN**: Continues logic associated with callable symbol `create`.
  **L579 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L580 EN**: Executes a call or declaration centered on `rewriter.getIndexType`.
  **L580 CN**: 执行以 `rewriter.getIndexType` 为核心的调用或声明。
- **L581 EN**: Closes the current lexical scope or compound statement.
  **L581 CN**: 结束当前词法作用域或复合语句块。
- **L582 EN**: Closes the current lexical scope or compound statement.
  **L582 CN**: 结束当前词法作用域或复合语句块。
- **L583 EN**: Closes the current lexical scope or compound statement.
  **L583 CN**: 结束当前词法作用域或复合语句块。
- **L584 EN**: Blank line separating nearby declarations or logic blocks.
  **L584 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L585 EN**: Comment explains nearby logic, invariants, or intent: `compute tiled dot-product`.
  **L585 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`compute tiled dot-product`。
- **L586 EN**: Continues logic associated with callable symbol `createTiledDp`.
  **L586 CN**: 继续与可调用符号 `createTiledDp` 相关的逻辑。
- **L587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, locNewInnerLoop, ops, lhsClone->getResult(0), matB,`.
  **L587 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, locNewInnerLoop, ops, lhsClone->getResult(0), matB,`。
- **L588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ipType, opType, iterArgsNewInnerLoop, blockingFactor, isVnni,`.
  **L588 CN**: 继续一个多行参数列表、初始化器或聚合项：`ipType, opType, iterArgsNewInnerLoop, blockingFactor, isVnni,`。
- **L589 EN**: Executes a standalone statement or declaration: `packedBuffer, pack, indxToStoreInBuffer, indxToLoadFromBuffer);`.
  **L589 CN**: 执行一条独立语句或声明：`packedBuffer, pack, indxToStoreInBuffer, indxToLoadFromBuffer);`。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `scf::YieldOp::create(rewriterNewInnerLoop, locNewInnerLoop,`.
  **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`scf::YieldOp::create(rewriterNewInnerLoop, locNewInnerLoop,`。
- **L592 EN**: Executes a standalone statement or declaration: `accumulators);`.
  **L592 CN**: 执行一条独立语句或声明：`accumulators);`。
- **L593 EN**: Executes a standalone statement or declaration: `});`.
  **L593 CN**: 执行一条独立语句或声明：`});`。
- **L594 EN**: Blank line separating nearby declarations or logic blocks.
  **L594 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L595 EN**: Returns from the current function with `newLoop`.
  **L595 CN**: 以 `newLoop` 从当前函数返回。
- **L596 EN**: Closes the current lexical scope or compound statement.
  **L596 CN**: 结束当前词法作用域或复合语句块。
- **L597 EN**: Blank line separating nearby declarations or logic blocks.
  **L597 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L598 EN**: Comment explains nearby logic, invariants, or intent: `Implements tiled dot-product operation for a vector.contract operation or a`.
  **L598 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Implements tiled dot-product operation for a vector.contract operation or a`。
- **L599 EN**: Comment explains nearby logic, invariants, or intent: `sequence of vector.contracts inside the reduction loops.`.
  **L599 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sequence of vector.contracts inside the reduction loops.`。
- **L600 EN**: Separator comment used for visual grouping.
  **L600 CN**: 用于视觉分组的分隔注释。

### Lines 601-624

````cpp
// For example:
// Case 1: register blocked vector.contract with prepacked input
// ```
//   vector.transfer_read %arg0 {{.}*} : memref<16x32x4xi8>, vector<16x16x4xi8>
//   vector.transfer_read %arg1 {{.}*} : memref<16x32x4xi8>, vector<16x16x4xi8>
//   vector.contract <16x16x4xi8>, <16x16x4xi8> into <16x16xi32>
//   vector.transfer_write arg2 {{.}*} : vector<16x16xi32>, memref<32x32xi32>
// ```
// to
// ```
//   amx.tile_load %arg0 {{.}*} : memref<16x32x4xi8> into !amx.tile<16x64xi8>
//   amx.tile_load %arg1 {{.}*} : memref<16x32x4xi8> into !amx.tile<16x64xi8>
//   amx.tile_muli !amx.tile<16x64xi8> -> !amx.tile<16x16xi32>
//   amx.tile_store %arg2{{.}*} : memref<32x32xi32>, !amx.tile<16x16xi32>
// ```
//
//
// Case2: vector.contract with register blocked
//
// Output IR with online packing (with s/w pipeline advantage):
// s/w pipeline: load, pack to VNNI, and store the B sub matrix
// of the 0th batch-reduce and K iteration.
// scf.for (0 to 31) {
// 	- load 0th and 1st  vector<32xbf16>, pack into VNNI, store the
````
- **L601 EN**: Comment explains nearby logic, invariants, or intent: `For example:`.
  **L601 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For example:`。
- **L602 EN**: Comment explains nearby logic, invariants, or intent: `Case 1: register blocked vector.contract with prepacked input`.
  **L602 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Case 1: register blocked vector.contract with prepacked input`。
- **L603 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L603 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L604 EN**: Comment explains nearby logic, invariants, or intent: `vector.transfer_read %arg0 {{.}*} : memref<16x32x4xi8>, vector<16x16x4xi8>`.
  **L604 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vector.transfer_read %arg0 {{.}*} : memref<16x32x4xi8>, vector<16x16x4xi8>`。
- **L605 EN**: Comment explains nearby logic, invariants, or intent: `vector.transfer_read %arg1 {{.}*} : memref<16x32x4xi8>, vector<16x16x4xi8>`.
  **L605 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vector.transfer_read %arg1 {{.}*} : memref<16x32x4xi8>, vector<16x16x4xi8>`。
- **L606 EN**: Comment explains nearby logic, invariants, or intent: `vector.contract <16x16x4xi8>, <16x16x4xi8> into <16x16xi32>`.
  **L606 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vector.contract <16x16x4xi8>, <16x16x4xi8> into <16x16xi32>`。
- **L607 EN**: Comment explains nearby logic, invariants, or intent: `vector.transfer_write arg2 {{.}*} : vector<16x16xi32>, memref<32x32xi32>`.
  **L607 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vector.transfer_write arg2 {{.}*} : vector<16x16xi32>, memref<32x32xi32>`。
- **L608 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L608 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L609 EN**: Comment explains nearby logic, invariants, or intent: `to`.
  **L609 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to`。
- **L610 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L610 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L611 EN**: Comment explains nearby logic, invariants, or intent: `amx.tile_load %arg0 {{.}*} : memref<16x32x4xi8> into !amx.tile<16x64xi8>`.
  **L611 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`amx.tile_load %arg0 {{.}*} : memref<16x32x4xi8> into !amx.tile<16x64xi8>`。
- **L612 EN**: Comment explains nearby logic, invariants, or intent: `amx.tile_load %arg1 {{.}*} : memref<16x32x4xi8> into !amx.tile<16x64xi8>`.
  **L612 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`amx.tile_load %arg1 {{.}*} : memref<16x32x4xi8> into !amx.tile<16x64xi8>`。
- **L613 EN**: Comment explains nearby logic, invariants, or intent: `amx.tile_muli !amx.tile<16x64xi8> -> !amx.tile<16x16xi32>`.
  **L613 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`amx.tile_muli !amx.tile<16x64xi8> -> !amx.tile<16x16xi32>`。
- **L614 EN**: Comment explains nearby logic, invariants, or intent: `amx.tile_store %arg2{{.}*} : memref<32x32xi32>, !amx.tile<16x16xi32>`.
  **L614 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`amx.tile_store %arg2{{.}*} : memref<32x32xi32>, !amx.tile<16x16xi32>`。
- **L615 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L615 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L616 EN**: Separator comment used for visual grouping.
  **L616 CN**: 用于视觉分组的分隔注释。
- **L617 EN**: Separator comment used for visual grouping.
  **L617 CN**: 用于视觉分组的分隔注释。
- **L618 EN**: Comment explains nearby logic, invariants, or intent: `Case2: vector.contract with register blocked`.
  **L618 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Case2: vector.contract with register blocked`。
- **L619 EN**: Separator comment used for visual grouping.
  **L619 CN**: 用于视觉分组的分隔注释。
- **L620 EN**: Comment explains nearby logic, invariants, or intent: `Output IR with online packing (with s/w pipeline advantage):`.
  **L620 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Output IR with online packing (with s/w pipeline advantage):`。
- **L621 EN**: Comment explains nearby logic, invariants, or intent: `s/w pipeline: load, pack to VNNI, and store the B sub matrix`.
  **L621 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`s/w pipeline: load, pack to VNNI, and store the B sub matrix`。
- **L622 EN**: Comment explains nearby logic, invariants, or intent: `of the 0th batch-reduce and K iteration.`.
  **L622 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`of the 0th batch-reduce and K iteration.`。
- **L623 EN**: Comment explains nearby logic, invariants, or intent: `scf.for (0 to 31) {`.
  **L623 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scf.for (0 to 31) {`。
- **L624 EN**: Comment explains nearby logic, invariants, or intent: `load 0th and 1st  vector<32xbf16>, pack into VNNI, store the`.
  **L624 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`load 0th and 1st  vector<32xbf16>, pack into VNNI, store the`。

### Lines 625-648

````cpp
// 	first shuffle in 0th and 2nd shuffle in 16th index of the
// 	buffer.
// }
// scf.for (0 to br-2) { batch-reduce loop
//   scf.for (0 to k-2) { K loop
// 	- load A matrix
//	- scf.loop for s/w pipeline: load, pack to VNNI, and store the B sub
// matrix 	for the next K loop iteration 	(c) load VNNI pack B matrix of K
// iteration from the buffer 	(d) compute the tiled dot-product
//   }
//   Last iteration of the the K Loop (k-1) {
//      - load A matrix
//      - scf.loop for s/w pipeline: load, pack to VNNI, and store the B sub
//      matrix for the next batch-reduce + K loop iteration (c) load VNNI pack B
//      matrix of K iteration from the buffer (d) compute the tiled dot-product
//   }
// }
// Last iteration of the batch-reduce loop (br-1) {
//   scf.for (0 to k-2) { K loop
//      - load A matrix
//      - scf.loop for s/w pipeline: load, pack to VNNI, and store the B sub
//      matrix for the next K loop iteration (c) load VNNI pack B matrix of K
//      iteration from the buffer (d) compute the tiled dot-product
//   }
````
- **L625 EN**: Comment explains nearby logic, invariants, or intent: `first shuffle in 0th and 2nd shuffle in 16th index of the`.
  **L625 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`first shuffle in 0th and 2nd shuffle in 16th index of the`。
- **L626 EN**: Comment explains nearby logic, invariants, or intent: `buffer.`.
  **L626 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`buffer.`。
- **L627 EN**: Comment explains nearby logic, invariants, or intent: `}`.
  **L627 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`}`。
- **L628 EN**: Comment explains nearby logic, invariants, or intent: `scf.for (0 to br-2) { batch-reduce loop`.
  **L628 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scf.for (0 to br-2) { batch-reduce loop`。
- **L629 EN**: Comment explains nearby logic, invariants, or intent: `scf.for (0 to k-2) { K loop`.
  **L629 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scf.for (0 to k-2) { K loop`。
- **L630 EN**: Comment explains nearby logic, invariants, or intent: `load A matrix`.
  **L630 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`load A matrix`。
- **L631 EN**: Comment explains nearby logic, invariants, or intent: `scf.loop for s/w pipeline: load, pack to VNNI, and store the B sub`.
  **L631 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scf.loop for s/w pipeline: load, pack to VNNI, and store the B sub`。
- **L632 EN**: Comment explains nearby logic, invariants, or intent: `matrix 	for the next K loop iteration 	(c) load VNNI pack B matrix of K`.
  **L632 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`matrix 	for the next K loop iteration 	(c) load VNNI pack B matrix of K`。
- **L633 EN**: Comment explains nearby logic, invariants, or intent: `iteration from the buffer 	(d) compute the tiled dot-product`.
  **L633 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`iteration from the buffer 	(d) compute the tiled dot-product`。
- **L634 EN**: Comment explains nearby logic, invariants, or intent: `}`.
  **L634 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`}`。
- **L635 EN**: Comment explains nearby logic, invariants, or intent: `Last iteration of the the K Loop (k-1) {`.
  **L635 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Last iteration of the the K Loop (k-1) {`。
- **L636 EN**: Comment explains nearby logic, invariants, or intent: `load A matrix`.
  **L636 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`load A matrix`。
- **L637 EN**: Comment explains nearby logic, invariants, or intent: `scf.loop for s/w pipeline: load, pack to VNNI, and store the B sub`.
  **L637 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scf.loop for s/w pipeline: load, pack to VNNI, and store the B sub`。
- **L638 EN**: Comment explains nearby logic, invariants, or intent: `matrix for the next batch-reduce + K loop iteration (c) load VNNI pack B`.
  **L638 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`matrix for the next batch-reduce + K loop iteration (c) load VNNI pack B`。
- **L639 EN**: Comment explains nearby logic, invariants, or intent: `matrix of K iteration from the buffer (d) compute the tiled dot-product`.
  **L639 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`matrix of K iteration from the buffer (d) compute the tiled dot-product`。
- **L640 EN**: Comment explains nearby logic, invariants, or intent: `}`.
  **L640 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`}`。
- **L641 EN**: Comment explains nearby logic, invariants, or intent: `}`.
  **L641 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`}`。
- **L642 EN**: Comment explains nearby logic, invariants, or intent: `Last iteration of the batch-reduce loop (br-1) {`.
  **L642 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Last iteration of the batch-reduce loop (br-1) {`。
- **L643 EN**: Comment explains nearby logic, invariants, or intent: `scf.for (0 to k-2) { K loop`.
  **L643 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scf.for (0 to k-2) { K loop`。
- **L644 EN**: Comment explains nearby logic, invariants, or intent: `load A matrix`.
  **L644 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`load A matrix`。
- **L645 EN**: Comment explains nearby logic, invariants, or intent: `scf.loop for s/w pipeline: load, pack to VNNI, and store the B sub`.
  **L645 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scf.loop for s/w pipeline: load, pack to VNNI, and store the B sub`。
- **L646 EN**: Comment explains nearby logic, invariants, or intent: `matrix for the next K loop iteration (c) load VNNI pack B matrix of K`.
  **L646 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`matrix for the next K loop iteration (c) load VNNI pack B matrix of K`。
- **L647 EN**: Comment explains nearby logic, invariants, or intent: `iteration from the buffer (d) compute the tiled dot-product`.
  **L647 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`iteration from the buffer (d) compute the tiled dot-product`。
- **L648 EN**: Comment explains nearby logic, invariants, or intent: `}`.
  **L648 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`}`。

### Lines 649-672

````cpp
//   Last iteration of the the K Loop (k-1) {
//      - load A matrix
//      - load VNNI pack B matrix of K iteration from the buffer
//      - compute the tiled dot-product
//   }
// }
//
// scf.for (0 to M)
//   scf.for (0 to N)
//     - Load the ith and i+1th acc
//     - Shuffle them as we packed using vpunpack
//     - Load C matrix and do arith.add with the shuffle
//     - Store back into C matrix
struct VectorContractToAMXDotProduct
    : public OpRewritePattern<vector::ContractionOp> {
  using OpRewritePattern<vector::ContractionOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(vector::ContractionOp contractOp,
                                PatternRewriter &rewriter) const override {

    if (contractOp.getKind() != vector::CombiningKind::ADD)
      return rewriter.notifyMatchFailure(contractOp,
                                         "Expects add combining kind.");

````
- **L649 EN**: Comment explains nearby logic, invariants, or intent: `Last iteration of the the K Loop (k-1) {`.
  **L649 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Last iteration of the the K Loop (k-1) {`。
- **L650 EN**: Comment explains nearby logic, invariants, or intent: `load A matrix`.
  **L650 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`load A matrix`。
- **L651 EN**: Comment explains nearby logic, invariants, or intent: `load VNNI pack B matrix of K iteration from the buffer`.
  **L651 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`load VNNI pack B matrix of K iteration from the buffer`。
- **L652 EN**: Comment explains nearby logic, invariants, or intent: `compute the tiled dot-product`.
  **L652 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`compute the tiled dot-product`。
- **L653 EN**: Comment explains nearby logic, invariants, or intent: `}`.
  **L653 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`}`。
- **L654 EN**: Comment explains nearby logic, invariants, or intent: `}`.
  **L654 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`}`。
- **L655 EN**: Separator comment used for visual grouping.
  **L655 CN**: 用于视觉分组的分隔注释。
- **L656 EN**: Comment explains nearby logic, invariants, or intent: `scf.for (0 to M)`.
  **L656 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scf.for (0 to M)`。
- **L657 EN**: Comment explains nearby logic, invariants, or intent: `scf.for (0 to N)`.
  **L657 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scf.for (0 to N)`。
- **L658 EN**: Comment explains nearby logic, invariants, or intent: `Load the ith and i+1th acc`.
  **L658 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Load the ith and i+1th acc`。
- **L659 EN**: Comment explains nearby logic, invariants, or intent: `Shuffle them as we packed using vpunpack`.
  **L659 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Shuffle them as we packed using vpunpack`。
- **L660 EN**: Comment explains nearby logic, invariants, or intent: `Load C matrix and do arith.add with the shuffle`.
  **L660 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Load C matrix and do arith.add with the shuffle`。
- **L661 EN**: Comment explains nearby logic, invariants, or intent: `Store back into C matrix`.
  **L661 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Store back into C matrix`。
- **L662 EN**: Declares struct `VectorContractToAMXDotProduct`.
  **L662 CN**: 声明 struct `VectorContractToAMXDotProduct`。
- **L663 EN**: Continues the surrounding expression or declaration: `: public OpRewritePattern<vector::ContractionOp> {`.
  **L663 CN**: 继续构造周围的表达式或声明：`: public OpRewritePattern<vector::ContractionOp> {`。
- **L664 EN**: Executes a standalone statement or declaration: `using OpRewritePattern<vector::ContractionOp>::OpRewritePattern;`.
  **L664 CN**: 执行一条独立语句或声明：`using OpRewritePattern<vector::ContractionOp>::OpRewritePattern;`。
- **L665 EN**: Blank line separating nearby declarations or logic blocks.
  **L665 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(vector::ContractionOp contractOp,`.
  **L666 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(vector::ContractionOp contractOp,`。
- **L667 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L667 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L669 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L669 CN**: 开始 `if` 控制流语句并计算其条件。
- **L670 EN**: Returns from the current function with `rewriter.notifyMatchFailure(contractOp,`.
  **L670 CN**: 以 `rewriter.notifyMatchFailure(contractOp,` 从当前函数返回。
- **L671 EN**: Executes a standalone statement or declaration: `"Expects add combining kind.");`.
  **L671 CN**: 执行一条独立语句或声明：`"Expects add combining kind.");`。
- **L672 EN**: Blank line separating nearby declarations or logic blocks.
  **L672 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 673-696

````cpp
    unsigned int blockingFactor =
        contractOp.getLhsType().getElementType().isBF16() ? 2 : 4;
    bool isVnni =
        isInVnniLayout(contractOp.getOperation(),
                       contractOp.getIndexingMapsArray(), blockingFactor);

    VectorType lhsTy = contractOp.getLhsType();
    if (!lhsTy.getElementType().isBF16() &&
        !lhsTy.getElementType().isSignlessInteger(8))
      return rewriter.notifyMatchFailure(
          contractOp, "Only BF16/Int8 lowering is supported.");

    VectorType accTy = dyn_cast<VectorType>(contractOp.getAccType());
    if (!accTy)
      return rewriter.notifyMatchFailure(contractOp, "Wrong accmulator type.");

    if ((lhsTy.getElementType().isBF16() && !accTy.getElementType().isF32()) ||
        (lhsTy.getElementType().isSignlessInteger(8) &&
         !accTy.getElementType().isSignlessInteger(32)))
      return rewriter.notifyMatchFailure(contractOp,
                                         "Only F32 for BF16 or Int32 for Int8 "
                                         "accumulation type is supported.");

    Operation *accReadOp =
````
- **L673 EN**: Continues the surrounding expression or declaration: `unsigned int blockingFactor =`.
  **L673 CN**: 继续构造周围的表达式或声明：`unsigned int blockingFactor =`。
- **L674 EN**: Executes a call or declaration centered on `contractOp.getLhsType`.
  **L674 CN**: 执行以 `contractOp.getLhsType` 为核心的调用或声明。
- **L675 EN**: Continues the surrounding expression or declaration: `bool isVnni =`.
  **L675 CN**: 继续构造周围的表达式或声明：`bool isVnni =`。
- **L676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `isInVnniLayout(contractOp.getOperation(),`.
  **L676 CN**: 继续一个多行参数列表、初始化器或聚合项：`isInVnniLayout(contractOp.getOperation(),`。
- **L677 EN**: Executes a call or declaration centered on `contractOp.getIndexingMapsArray`.
  **L677 CN**: 执行以 `contractOp.getIndexingMapsArray` 为核心的调用或声明。
- **L678 EN**: Blank line separating nearby declarations or logic blocks.
  **L678 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L679 EN**: Initializes variable `lhsTy` from the right-hand expression.
  **L679 CN**: 使用右侧表达式初始化变量 `lhsTy`。
- **L680 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L680 CN**: 开始 `if` 控制流语句并计算其条件。
- **L681 EN**: Continues logic associated with callable symbol `getElementType`.
  **L681 CN**: 继续与可调用符号 `getElementType` 相关的逻辑。
- **L682 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L682 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L683 EN**: Executes a standalone statement or declaration: `contractOp, "Only BF16/Int8 lowering is supported.");`.
  **L683 CN**: 执行一条独立语句或声明：`contractOp, "Only BF16/Int8 lowering is supported.");`。
- **L684 EN**: Blank line separating nearby declarations or logic blocks.
  **L684 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L685 EN**: Initializes variable `accTy` from the right-hand expression.
  **L685 CN**: 使用右侧表达式初始化变量 `accTy`。
- **L686 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L686 CN**: 开始 `if` 控制流语句并计算其条件。
- **L687 EN**: Returns from the current function with `rewriter.notifyMatchFailure(contractOp, "Wrong accmulator type.")`.
  **L687 CN**: 以 `rewriter.notifyMatchFailure(contractOp, "Wrong accmulator type.")` 从当前函数返回。
- **L688 EN**: Blank line separating nearby declarations or logic blocks.
  **L688 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L689 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L689 CN**: 开始 `if` 控制流语句并计算其条件。
- **L690 EN**: Continues logic associated with callable symbol `getElementType`.
  **L690 CN**: 继续与可调用符号 `getElementType` 相关的逻辑。
- **L691 EN**: Continues logic associated with callable symbol `getElementType`.
  **L691 CN**: 继续与可调用符号 `getElementType` 相关的逻辑。
- **L692 EN**: Returns from the current function with `rewriter.notifyMatchFailure(contractOp,`.
  **L692 CN**: 以 `rewriter.notifyMatchFailure(contractOp,` 从当前函数返回。
- **L693 EN**: Continues the surrounding expression or declaration: `"Only F32 for BF16 or Int32 for Int8 "`.
  **L693 CN**: 继续构造周围的表达式或声明：`"Only F32 for BF16 or Int32 for Int8 "`。
- **L694 EN**: Executes a standalone statement or declaration: `"accumulation type is supported.");`.
  **L694 CN**: 执行一条独立语句或声明：`"accumulation type is supported.");`。
- **L695 EN**: Blank line separating nearby declarations or logic blocks.
  **L695 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L696 EN**: Continues the surrounding expression or declaration: `Operation *accReadOp =`.
  **L696 CN**: 继续构造周围的表达式或声明：`Operation *accReadOp =`。

### Lines 697-720

````cpp
        traceToVectorReadLikeParentOperation(contractOp.getAcc());

    Operation *resultWriteOp =
        traceToVectorWriteLikeUserOperation(contractOp.getResult());

    if (!accReadOp || !resultWriteOp)
      return rewriter.notifyMatchFailure(
          contractOp, "The ACC operand of the vector.contract should be a "
                      "transfer_read or a load. And, the result should be "
                      "stored using transfer_write or store.");

    Type ipType = rewriter.getBF16Type();
    Type opType = rewriter.getF32Type();

    if (lhsTy.getElementType().isSignlessInteger(8)) {
      ipType = rewriter.getIntegerType(8);
      opType = rewriter.getIntegerType(32);
    }

    if (accReadOp->getBlock() == contractOp->getBlock() &&
        resultWriteOp->getBlock() != contractOp->getBlock())
      return rewriter.notifyMatchFailure(
          contractOp, "The accumulator store is in different block.");

````
- **L697 EN**: Executes a call or declaration centered on `traceToVectorReadLikeParentOperation`.
  **L697 CN**: 执行以 `traceToVectorReadLikeParentOperation` 为核心的调用或声明。
- **L698 EN**: Blank line separating nearby declarations or logic blocks.
  **L698 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L699 EN**: Continues the surrounding expression or declaration: `Operation *resultWriteOp =`.
  **L699 CN**: 继续构造周围的表达式或声明：`Operation *resultWriteOp =`。
- **L700 EN**: Executes a call or declaration centered on `traceToVectorWriteLikeUserOperation`.
  **L700 CN**: 执行以 `traceToVectorWriteLikeUserOperation` 为核心的调用或声明。
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L702 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L702 CN**: 开始 `if` 控制流语句并计算其条件。
- **L703 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L703 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L704 EN**: Continues the surrounding expression or declaration: `contractOp, "The ACC operand of the vector.contract should be a "`.
  **L704 CN**: 继续构造周围的表达式或声明：`contractOp, "The ACC operand of the vector.contract should be a "`。
- **L705 EN**: Continues the surrounding expression or declaration: `"transfer_read or a load. And, the result should be "`.
  **L705 CN**: 继续构造周围的表达式或声明：`"transfer_read or a load. And, the result should be "`。
- **L706 EN**: Executes a standalone statement or declaration: `"stored using transfer_write or store.");`.
  **L706 CN**: 执行一条独立语句或声明：`"stored using transfer_write or store.");`。
- **L707 EN**: Blank line separating nearby declarations or logic blocks.
  **L707 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L708 EN**: Initializes variable `ipType` from the right-hand expression.
  **L708 CN**: 使用右侧表达式初始化变量 `ipType`。
- **L709 EN**: Initializes variable `opType` from the right-hand expression.
  **L709 CN**: 使用右侧表达式初始化变量 `opType`。
- **L710 EN**: Blank line separating nearby declarations or logic blocks.
  **L710 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L711 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L711 CN**: 开始 `if` 控制流语句并计算其条件。
- **L712 EN**: Executes a call or declaration centered on `rewriter.getIntegerType`.
  **L712 CN**: 执行以 `rewriter.getIntegerType` 为核心的调用或声明。
- **L713 EN**: Executes a call or declaration centered on `rewriter.getIntegerType`.
  **L713 CN**: 执行以 `rewriter.getIntegerType` 为核心的调用或声明。
- **L714 EN**: Closes the current lexical scope or compound statement.
  **L714 CN**: 结束当前词法作用域或复合语句块。
- **L715 EN**: Blank line separating nearby declarations or logic blocks.
  **L715 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L716 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L716 CN**: 开始 `if` 控制流语句并计算其条件。
- **L717 EN**: Continues logic associated with callable symbol `getBlock`.
  **L717 CN**: 继续与可调用符号 `getBlock` 相关的逻辑。
- **L718 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L718 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L719 EN**: Executes a standalone statement or declaration: `contractOp, "The accumulator store is in different block.");`.
  **L719 CN**: 执行一条独立语句或声明：`contractOp, "The accumulator store is in different block.");`。
- **L720 EN**: Blank line separating nearby declarations or logic blocks.
  **L720 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 721-744

````cpp
    if (accReadOp->getBlock() != contractOp->getBlock() &&
        resultWriteOp->getBlock() == contractOp->getBlock())
      return rewriter.notifyMatchFailure(
          contractOp, "The accumulator read is in different block.");

    unsigned int dimValue = blockingFactor;
    if (!isVnni)
      dimValue = 16 * blockingFactor;

    // Case 1: For just one VC rewrite. Where all accumulator read/write
    // within the same block.
    if (accReadOp->getBlock() == contractOp->getBlock() &&
        resultWriteOp->getBlock() == contractOp->getBlock()) {

      bool collapse = false;
      if (isVnni)
        collapse = true;

      LogicalResult validate = validateContractOps(
          rewriter, contractOp, dimValue, Value(), Value(), false);

      if (failed(validate))
        return rewriter.notifyMatchFailure(
            contractOp, "The contract operation doesn't satisfy the operands "
````
- **L721 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L721 CN**: 开始 `if` 控制流语句并计算其条件。
- **L722 EN**: Continues logic associated with callable symbol `getBlock`.
  **L722 CN**: 继续与可调用符号 `getBlock` 相关的逻辑。
- **L723 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L723 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L724 EN**: Executes a standalone statement or declaration: `contractOp, "The accumulator read is in different block.");`.
  **L724 CN**: 执行一条独立语句或声明：`contractOp, "The accumulator read is in different block.");`。
- **L725 EN**: Blank line separating nearby declarations or logic blocks.
  **L725 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L726 EN**: Initializes variable `dimValue` from the right-hand expression.
  **L726 CN**: 使用右侧表达式初始化变量 `dimValue`。
- **L727 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L727 CN**: 开始 `if` 控制流语句并计算其条件。
- **L728 EN**: Executes a standalone statement or declaration: `dimValue = 16 * blockingFactor;`.
  **L728 CN**: 执行一条独立语句或声明：`dimValue = 16 * blockingFactor;`。
- **L729 EN**: Blank line separating nearby declarations or logic blocks.
  **L729 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L730 EN**: Comment explains nearby logic, invariants, or intent: `Case 1: For just one VC rewrite. Where all accumulator read/write`.
  **L730 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Case 1: For just one VC rewrite. Where all accumulator read/write`。
- **L731 EN**: Comment explains nearby logic, invariants, or intent: `within the same block.`.
  **L731 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`within the same block.`。
- **L732 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L732 CN**: 开始 `if` 控制流语句并计算其条件。
- **L733 EN**: Starts a function, method, lambda, or structured scope: `resultWriteOp->getBlock() == contractOp->getBlock()) {`.
  **L733 CN**: 开始一个函数、方法、lambda 或结构化作用域：`resultWriteOp->getBlock() == contractOp->getBlock()) {`。
- **L734 EN**: Blank line separating nearby declarations or logic blocks.
  **L734 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L735 EN**: Initializes variable `collapse` from the right-hand expression.
  **L735 CN**: 使用右侧表达式初始化变量 `collapse`。
- **L736 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L736 CN**: 开始 `if` 控制流语句并计算其条件。
- **L737 EN**: Executes a standalone statement or declaration: `collapse = true;`.
  **L737 CN**: 执行一条独立语句或声明：`collapse = true;`。
- **L738 EN**: Blank line separating nearby declarations or logic blocks.
  **L738 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L739 EN**: Continues logic associated with callable symbol `validateContractOps`.
  **L739 CN**: 继续与可调用符号 `validateContractOps` 相关的逻辑。
- **L740 EN**: Executes a call or declaration centered on `Value`.
  **L740 CN**: 执行以 `Value` 为核心的调用或声明。
- **L741 EN**: Blank line separating nearby declarations or logic blocks.
  **L741 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L742 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L742 CN**: 开始 `if` 控制流语句并计算其条件。
- **L743 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L743 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L744 EN**: Continues the surrounding expression or declaration: `contractOp, "The contract operation doesn't satisfy the operands "`.
  **L744 CN**: 继续构造周围的表达式或声明：`contractOp, "The contract operation doesn't satisfy the operands "`。

### Lines 745-768

````cpp
                        "dimensions. M, N, and vnni dims are 16, 16, and 2/4. "
                        "The rest dims should be 1.");

      Location loc = contractOp.getLoc();

      auto srcIndxLhs = getSrcIndxValue(rewriter, contractOp.getLoc(),
                                        contractOp.getLhs(), collapse);
      if (failed(srcIndxLhs))
        return rewriter.notifyMatchFailure(contractOp,
                                           "The LHS src is not a MemRef type.");
      auto [srcBuffLhs, indicesLhs] = *srcIndxLhs;

      auto srcIndxRhs = getSrcIndxValue(rewriter, contractOp.getLoc(),
                                        contractOp.getRhs(), collapse);
      if (failed(srcIndxRhs))
        return rewriter.notifyMatchFailure(contractOp,
                                           "The RHS src is not a MemRef type.");
      auto rhsSrc = *srcIndxRhs;
      auto srcBuffRhs = rhsSrc.first;
      auto indicesRhs = rhsSrc.second;

      auto srcIndxAcc = getSrcIndxValue(rewriter, contractOp.getLoc(),
                                        contractOp.getAcc(), false);
      if (failed(srcIndxAcc))
````
- **L745 EN**: Continues the surrounding expression or declaration: `"dimensions. M, N, and vnni dims are 16, 16, and 2/4. "`.
  **L745 CN**: 继续构造周围的表达式或声明：`"dimensions. M, N, and vnni dims are 16, 16, and 2/4. "`。
- **L746 EN**: Executes a standalone statement or declaration: `"The rest dims should be 1.");`.
  **L746 CN**: 执行一条独立语句或声明：`"The rest dims should be 1.");`。
- **L747 EN**: Blank line separating nearby declarations or logic blocks.
  **L747 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L748 EN**: Initializes variable `loc` from the right-hand expression.
  **L748 CN**: 使用右侧表达式初始化变量 `loc`。
- **L749 EN**: Blank line separating nearby declarations or logic blocks.
  **L749 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto srcIndxLhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`.
  **L750 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto srcIndxLhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`。
- **L751 EN**: Executes a call or declaration centered on `contractOp.getLhs`.
  **L751 CN**: 执行以 `contractOp.getLhs` 为核心的调用或声明。
- **L752 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L752 CN**: 开始 `if` 控制流语句并计算其条件。
- **L753 EN**: Returns from the current function with `rewriter.notifyMatchFailure(contractOp,`.
  **L753 CN**: 以 `rewriter.notifyMatchFailure(contractOp,` 从当前函数返回。
- **L754 EN**: Executes a standalone statement or declaration: `"The LHS src is not a MemRef type.");`.
  **L754 CN**: 执行一条独立语句或声明：`"The LHS src is not a MemRef type.");`。
- **L755 EN**: Executes a standalone statement or declaration: `auto [srcBuffLhs, indicesLhs] = *srcIndxLhs;`.
  **L755 CN**: 执行一条独立语句或声明：`auto [srcBuffLhs, indicesLhs] = *srcIndxLhs;`。
- **L756 EN**: Blank line separating nearby declarations or logic blocks.
  **L756 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto srcIndxRhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`.
  **L757 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto srcIndxRhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`。
- **L758 EN**: Executes a call or declaration centered on `contractOp.getRhs`.
  **L758 CN**: 执行以 `contractOp.getRhs` 为核心的调用或声明。
- **L759 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L759 CN**: 开始 `if` 控制流语句并计算其条件。
- **L760 EN**: Returns from the current function with `rewriter.notifyMatchFailure(contractOp,`.
  **L760 CN**: 以 `rewriter.notifyMatchFailure(contractOp,` 从当前函数返回。
- **L761 EN**: Executes a standalone statement or declaration: `"The RHS src is not a MemRef type.");`.
  **L761 CN**: 执行一条独立语句或声明：`"The RHS src is not a MemRef type.");`。
- **L762 EN**: Initializes variable `rhsSrc` from the right-hand expression.
  **L762 CN**: 使用右侧表达式初始化变量 `rhsSrc`。
- **L763 EN**: Initializes variable `srcBuffRhs` from the right-hand expression.
  **L763 CN**: 使用右侧表达式初始化变量 `srcBuffRhs`。
- **L764 EN**: Initializes variable `indicesRhs` from the right-hand expression.
  **L764 CN**: 使用右侧表达式初始化变量 `indicesRhs`。
- **L765 EN**: Blank line separating nearby declarations or logic blocks.
  **L765 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto srcIndxAcc = getSrcIndxValue(rewriter, contractOp.getLoc(),`.
  **L766 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto srcIndxAcc = getSrcIndxValue(rewriter, contractOp.getLoc(),`。
- **L767 EN**: Executes a call or declaration centered on `contractOp.getAcc`.
  **L767 CN**: 执行以 `contractOp.getAcc` 为核心的调用或声明。
- **L768 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L768 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 769-792

````cpp
        return rewriter.notifyMatchFailure(contractOp,
                                           "The ACC src is not a MemRef type.");
      auto [srcBuffAcc, indicesAcc] = *srcIndxAcc;

      // amx.tile_loads
      auto tileType = amx::TileType::get({16, (16 * blockingFactor)}, ipType);
      auto loadLhs = amx::TileLoadOp::create(rewriter, loc, tileType,
                                             srcBuffLhs, indicesLhs);

      // Create the subview and then load.
      amx::TileLoadOp loadRhs;
      if (!isVnni) {
        VectorType vecTy;
        SmallVector<OpFoldResult> indexVals;
        llvm::TypeSwitch<Operation *>(contractOp.getRhs().getDefiningOp())
            .Case<TransferReadOp, LoadOp>([&](auto readOp) {
              indexVals = SmallVector<OpFoldResult>(readOp.getIndices().begin(),
                                                    readOp.getIndices().end());
              vecTy = readOp.getType();
            });
        auto one = rewriter.getIndexAttr(1);
        SmallVector<OpFoldResult> strides(indexVals.size(), one);
        SmallVector<OpFoldResult> sizes = getAsIndexOpFoldResult(
            contractOp.getRhs().getDefiningOp()->getContext(),
````
- **L769 EN**: Returns from the current function with `rewriter.notifyMatchFailure(contractOp,`.
  **L769 CN**: 以 `rewriter.notifyMatchFailure(contractOp,` 从当前函数返回。
- **L770 EN**: Executes a standalone statement or declaration: `"The ACC src is not a MemRef type.");`.
  **L770 CN**: 执行一条独立语句或声明：`"The ACC src is not a MemRef type.");`。
- **L771 EN**: Executes a standalone statement or declaration: `auto [srcBuffAcc, indicesAcc] = *srcIndxAcc;`.
  **L771 CN**: 执行一条独立语句或声明：`auto [srcBuffAcc, indicesAcc] = *srcIndxAcc;`。
- **L772 EN**: Blank line separating nearby declarations or logic blocks.
  **L772 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L773 EN**: Comment explains nearby logic, invariants, or intent: `amx.tile_loads`.
  **L773 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`amx.tile_loads`。
- **L774 EN**: Initializes variable `tileType` from the right-hand expression.
  **L774 CN**: 使用右侧表达式初始化变量 `tileType`。
- **L775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto loadLhs = amx::TileLoadOp::create(rewriter, loc, tileType,`.
  **L775 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto loadLhs = amx::TileLoadOp::create(rewriter, loc, tileType,`。
- **L776 EN**: Executes a standalone statement or declaration: `srcBuffLhs, indicesLhs);`.
  **L776 CN**: 执行一条独立语句或声明：`srcBuffLhs, indicesLhs);`。
- **L777 EN**: Blank line separating nearby declarations or logic blocks.
  **L777 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L778 EN**: Comment explains nearby logic, invariants, or intent: `Create the subview and then load.`.
  **L778 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create the subview and then load.`。
- **L779 EN**: Executes a standalone statement or declaration: `amx::TileLoadOp loadRhs;`.
  **L779 CN**: 执行一条独立语句或声明：`amx::TileLoadOp loadRhs;`。
- **L780 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L780 CN**: 开始 `if` 控制流语句并计算其条件。
- **L781 EN**: Executes a standalone statement or declaration: `VectorType vecTy;`.
  **L781 CN**: 执行一条独立语句或声明：`VectorType vecTy;`。
- **L782 EN**: Executes a standalone statement or declaration: `SmallVector<OpFoldResult> indexVals;`.
  **L782 CN**: 执行一条独立语句或声明：`SmallVector<OpFoldResult> indexVals;`。
- **L783 EN**: Continues logic associated with callable symbol `getRhs`.
  **L783 CN**: 继续与可调用符号 `getRhs` 相关的逻辑。
- **L784 EN**: Starts a function, method, lambda, or structured scope: `.Case<TransferReadOp, LoadOp>([&](auto readOp) {`.
  **L784 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.Case<TransferReadOp, LoadOp>([&](auto readOp) {`。
- **L785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `indexVals = SmallVector<OpFoldResult>(readOp.getIndices().begin(),`.
  **L785 CN**: 继续一个多行参数列表、初始化器或聚合项：`indexVals = SmallVector<OpFoldResult>(readOp.getIndices().begin(),`。
- **L786 EN**: Executes a call or declaration centered on `readOp.getIndices`.
  **L786 CN**: 执行以 `readOp.getIndices` 为核心的调用或声明。
- **L787 EN**: Executes a call or declaration centered on `readOp.getType`.
  **L787 CN**: 执行以 `readOp.getType` 为核心的调用或声明。
- **L788 EN**: Executes a standalone statement or declaration: `});`.
  **L788 CN**: 执行一条独立语句或声明：`});`。
- **L789 EN**: Initializes variable `one` from the right-hand expression.
  **L789 CN**: 使用右侧表达式初始化变量 `one`。
- **L790 EN**: Executes a call or declaration centered on `strides`.
  **L790 CN**: 执行以 `strides` 为核心的调用或声明。
- **L791 EN**: Continues logic associated with callable symbol `getAsIndexOpFoldResult`.
  **L791 CN**: 继续与可调用符号 `getAsIndexOpFoldResult` 相关的逻辑。
- **L792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `contractOp.getRhs().getDefiningOp()->getContext(),`.
  **L792 CN**: 继续一个多行参数列表、初始化器或聚合项：`contractOp.getRhs().getDefiningOp()->getContext(),`。

### Lines 793-816

````cpp
            vecTy.getShape());
        auto subview = memref::SubViewOp::create(rewriter, loc, srcBuffRhs,
                                                 indexVals, sizes, strides);
        auto bufferType = MemRefType::get({16, (16 * blockingFactor)}, ipType);
        auto packedBuffer = memref::AllocaOp::create(rewriter, loc, bufferType);

        // create a loop that does online packing.
        Value c0 = arith::ConstantIndexOp::create(rewriter, loc, 0);
        Value step =
            arith::ConstantIndexOp::create(rewriter, loc, blockingFactor);
        Value uBound = arith::ConstantIndexOp::create(rewriter, loc,
                                                      (blockingFactor * 16));
        Value nextLoadIndx =
            arith::ConstantIndexOp::create(rewriter, loc, (blockingFactor / 2));
        Value nextStoreIndx = arith::ConstantIndexOp::create(
            rewriter, loc, 16 * (blockingFactor / 2));

        scf::ForOp::create(
            rewriter, loc, c0, uBound, step, ValueRange{},
            [&](OpBuilder &nestedBuilder, Location loc, Value iv,
                ValueRange iterArgs) {
              Value i1_load =
                  arith::AddIOp::create(rewriter, loc, nextLoadIndx, iv);

````
- **L793 EN**: Executes a call or declaration centered on `vecTy.getShape`.
  **L793 CN**: 执行以 `vecTy.getShape` 为核心的调用或声明。
- **L794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto subview = memref::SubViewOp::create(rewriter, loc, srcBuffRhs,`.
  **L794 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto subview = memref::SubViewOp::create(rewriter, loc, srcBuffRhs,`。
- **L795 EN**: Executes a standalone statement or declaration: `indexVals, sizes, strides);`.
  **L795 CN**: 执行一条独立语句或声明：`indexVals, sizes, strides);`。
- **L796 EN**: Initializes variable `bufferType` from the right-hand expression.
  **L796 CN**: 使用右侧表达式初始化变量 `bufferType`。
- **L797 EN**: Initializes variable `packedBuffer` from the right-hand expression.
  **L797 CN**: 使用右侧表达式初始化变量 `packedBuffer`。
- **L798 EN**: Blank line separating nearby declarations or logic blocks.
  **L798 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L799 EN**: Comment explains nearby logic, invariants, or intent: `create a loop that does online packing.`.
  **L799 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`create a loop that does online packing.`。
- **L800 EN**: Initializes variable `c0` from the right-hand expression.
  **L800 CN**: 使用右侧表达式初始化变量 `c0`。
- **L801 EN**: Continues the surrounding expression or declaration: `Value step =`.
  **L801 CN**: 继续构造周围的表达式或声明：`Value step =`。
- **L802 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L802 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value uBound = arith::ConstantIndexOp::create(rewriter, loc,`.
  **L803 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value uBound = arith::ConstantIndexOp::create(rewriter, loc,`。
- **L804 EN**: Executes a call or declaration centered on `statement`.
  **L804 CN**: 执行以 `statement` 为核心的调用或声明。
- **L805 EN**: Continues the surrounding expression or declaration: `Value nextLoadIndx =`.
  **L805 CN**: 继续构造周围的表达式或声明：`Value nextLoadIndx =`。
- **L806 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L806 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L807 EN**: Continues logic associated with callable symbol `create`.
  **L807 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L808 EN**: Executes a call or declaration centered on `*`.
  **L808 CN**: 执行以 `*` 为核心的调用或声明。
- **L809 EN**: Blank line separating nearby declarations or logic blocks.
  **L809 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L810 EN**: Continues logic associated with callable symbol `create`.
  **L810 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, c0, uBound, step, ValueRange{},`.
  **L811 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, c0, uBound, step, ValueRange{},`。
- **L812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](OpBuilder &nestedBuilder, Location loc, Value iv,`.
  **L812 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](OpBuilder &nestedBuilder, Location loc, Value iv,`。
- **L813 EN**: Continues the surrounding expression or declaration: `ValueRange iterArgs) {`.
  **L813 CN**: 继续构造周围的表达式或声明：`ValueRange iterArgs) {`。
- **L814 EN**: Continues the surrounding expression or declaration: `Value i1_load =`.
  **L814 CN**: 继续构造周围的表达式或声明：`Value i1_load =`。
- **L815 EN**: Executes a call or declaration centered on `arith::AddIOp::create`.
  **L815 CN**: 执行以 `arith::AddIOp::create` 为核心的调用或声明。
- **L816 EN**: Blank line separating nearby declarations or logic blocks.
  **L816 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 817-840

````cpp
              indicesRhs[indicesRhs.size() - 2] = iv;
              ValueRange range1(indicesRhs);
              auto vec1 = vector::LoadOp::create(
                  rewriter, loc,
                  VectorType::get(16 * (blockingFactor / 2), ipType), subview,
                  range1);

              indicesRhs[indicesRhs.size() - 2] = i1_load;
              ValueRange range2(indicesRhs);
              auto vec2 = vector::LoadOp::create(
                  rewriter, loc,
                  VectorType::get(16 * (blockingFactor / 2), ipType), subview,
                  range2);

              vector::ShuffleOp shuffle1;
              vector::ShuffleOp shuffle2;

              if (blockingFactor == 2) {

                shuffle1 = vector::ShuffleOp::create(
                    rewriter, loc, VectorType::get({16}, ipType), vec1, vec2,
                    ArrayRef<int64_t>{0, 16, 1, 17, 2, 18, 3, 19, 4, 20, 5, 21,
                                      6, 22, 7, 23});

````
- **L817 EN**: Executes a call or declaration centered on `indicesRhs[indicesRhs.size`.
  **L817 CN**: 执行以 `indicesRhs[indicesRhs.size` 为核心的调用或声明。
- **L818 EN**: Executes a call or declaration centered on `range1`.
  **L818 CN**: 执行以 `range1` 为核心的调用或声明。
- **L819 EN**: Continues logic associated with callable symbol `create`.
  **L819 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L820 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc,`.
  **L820 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc,`。
- **L821 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VectorType::get(16 * (blockingFactor / 2), ipType), subview,`.
  **L821 CN**: 继续一个多行参数列表、初始化器或聚合项：`VectorType::get(16 * (blockingFactor / 2), ipType), subview,`。
- **L822 EN**: Executes a standalone statement or declaration: `range1);`.
  **L822 CN**: 执行一条独立语句或声明：`range1);`。
- **L823 EN**: Blank line separating nearby declarations or logic blocks.
  **L823 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L824 EN**: Executes a call or declaration centered on `indicesRhs[indicesRhs.size`.
  **L824 CN**: 执行以 `indicesRhs[indicesRhs.size` 为核心的调用或声明。
- **L825 EN**: Executes a call or declaration centered on `range2`.
  **L825 CN**: 执行以 `range2` 为核心的调用或声明。
- **L826 EN**: Continues logic associated with callable symbol `create`.
  **L826 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc,`.
  **L827 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc,`。
- **L828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VectorType::get(16 * (blockingFactor / 2), ipType), subview,`.
  **L828 CN**: 继续一个多行参数列表、初始化器或聚合项：`VectorType::get(16 * (blockingFactor / 2), ipType), subview,`。
- **L829 EN**: Executes a standalone statement or declaration: `range2);`.
  **L829 CN**: 执行一条独立语句或声明：`range2);`。
- **L830 EN**: Blank line separating nearby declarations or logic blocks.
  **L830 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L831 EN**: Executes a standalone statement or declaration: `vector::ShuffleOp shuffle1;`.
  **L831 CN**: 执行一条独立语句或声明：`vector::ShuffleOp shuffle1;`。
- **L832 EN**: Executes a standalone statement or declaration: `vector::ShuffleOp shuffle2;`.
  **L832 CN**: 执行一条独立语句或声明：`vector::ShuffleOp shuffle2;`。
- **L833 EN**: Blank line separating nearby declarations or logic blocks.
  **L833 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L834 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L834 CN**: 开始 `if` 控制流语句并计算其条件。
- **L835 EN**: Blank line separating nearby declarations or logic blocks.
  **L835 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L836 EN**: Continues logic associated with callable symbol `create`.
  **L836 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get({16}, ipType), vec1, vec2,`.
  **L837 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get({16}, ipType), vec1, vec2,`。
- **L838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t>{0, 16, 1, 17, 2, 18, 3, 19, 4, 20, 5, 21,`.
  **L838 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t>{0, 16, 1, 17, 2, 18, 3, 19, 4, 20, 5, 21,`。
- **L839 EN**: Executes a standalone statement or declaration: `6, 22, 7, 23});`.
  **L839 CN**: 执行一条独立语句或声明：`6, 22, 7, 23});`。
- **L840 EN**: Blank line separating nearby declarations or logic blocks.
  **L840 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 841-864

````cpp
                shuffle2 = vector::ShuffleOp::create(
                    rewriter, loc, VectorType::get({16}, ipType), vec1, vec2,
                    ArrayRef<int64_t>{8, 24, 9, 25, 10, 26, 11, 27, 12, 28, 13,
                                      29, 14, 30, 15, 31});
              }

              if (blockingFactor == 4) {
                shuffle1 = vector::ShuffleOp::create(
                    rewriter, loc, VectorType::get({32}, ipType), vec1, vec2,
                    ArrayRef<int64_t>{0, 16, 32, 48, 1, 17, 33, 49,
                                      2, 18, 34, 50, 3, 19, 35, 51,
                                      4, 20, 36, 52, 5, 21, 37, 53,
                                      6, 22, 38, 54, 7, 23, 39, 55});

                shuffle2 = vector::ShuffleOp::create(
                    rewriter, loc, VectorType::get({32}, ipType), vec1, vec2,
                    ArrayRef<int64_t>{8,  24, 40, 56, 9,  25, 41, 57,
                                      10, 26, 42, 58, 11, 27, 43, 59,
                                      12, 28, 44, 60, 13, 29, 45, 61,
                                      14, 30, 46, 62, 15, 31, 47, 63});
              }

              auto rem = arith::RemUIOp::create(
                  rewriter, loc, rewriter.getIndexType(), iv, step);
````
- **L841 EN**: Continues logic associated with callable symbol `create`.
  **L841 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get({16}, ipType), vec1, vec2,`.
  **L842 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get({16}, ipType), vec1, vec2,`。
- **L843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t>{8, 24, 9, 25, 10, 26, 11, 27, 12, 28, 13,`.
  **L843 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t>{8, 24, 9, 25, 10, 26, 11, 27, 12, 28, 13,`。
- **L844 EN**: Executes a standalone statement or declaration: `29, 14, 30, 15, 31});`.
  **L844 CN**: 执行一条独立语句或声明：`29, 14, 30, 15, 31});`。
- **L845 EN**: Closes the current lexical scope or compound statement.
  **L845 CN**: 结束当前词法作用域或复合语句块。
- **L846 EN**: Blank line separating nearby declarations or logic blocks.
  **L846 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L847 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L847 CN**: 开始 `if` 控制流语句并计算其条件。
- **L848 EN**: Continues logic associated with callable symbol `create`.
  **L848 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get({32}, ipType), vec1, vec2,`.
  **L849 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get({32}, ipType), vec1, vec2,`。
- **L850 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t>{0, 16, 32, 48, 1, 17, 33, 49,`.
  **L850 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t>{0, 16, 32, 48, 1, 17, 33, 49,`。
- **L851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `2, 18, 34, 50, 3, 19, 35, 51,`.
  **L851 CN**: 继续一个多行参数列表、初始化器或聚合项：`2, 18, 34, 50, 3, 19, 35, 51,`。
- **L852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `4, 20, 36, 52, 5, 21, 37, 53,`.
  **L852 CN**: 继续一个多行参数列表、初始化器或聚合项：`4, 20, 36, 52, 5, 21, 37, 53,`。
- **L853 EN**: Executes a standalone statement or declaration: `6, 22, 38, 54, 7, 23, 39, 55});`.
  **L853 CN**: 执行一条独立语句或声明：`6, 22, 38, 54, 7, 23, 39, 55});`。
- **L854 EN**: Blank line separating nearby declarations or logic blocks.
  **L854 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L855 EN**: Continues logic associated with callable symbol `create`.
  **L855 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get({32}, ipType), vec1, vec2,`.
  **L856 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get({32}, ipType), vec1, vec2,`。
- **L857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t>{8,  24, 40, 56, 9,  25, 41, 57,`.
  **L857 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t>{8,  24, 40, 56, 9,  25, 41, 57,`。
- **L858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `10, 26, 42, 58, 11, 27, 43, 59,`.
  **L858 CN**: 继续一个多行参数列表、初始化器或聚合项：`10, 26, 42, 58, 11, 27, 43, 59,`。
- **L859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `12, 28, 44, 60, 13, 29, 45, 61,`.
  **L859 CN**: 继续一个多行参数列表、初始化器或聚合项：`12, 28, 44, 60, 13, 29, 45, 61,`。
- **L860 EN**: Executes a standalone statement or declaration: `14, 30, 46, 62, 15, 31, 47, 63});`.
  **L860 CN**: 执行一条独立语句或声明：`14, 30, 46, 62, 15, 31, 47, 63});`。
- **L861 EN**: Closes the current lexical scope or compound statement.
  **L861 CN**: 结束当前词法作用域或复合语句块。
- **L862 EN**: Blank line separating nearby declarations or logic blocks.
  **L862 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L863 EN**: Continues logic associated with callable symbol `create`.
  **L863 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L864 EN**: Executes a call or declaration centered on `rewriter.getIndexType`.
  **L864 CN**: 执行以 `rewriter.getIndexType` 为核心的调用或声明。

### Lines 865-888

````cpp

              vector::StoreOp::create(rewriter, loc, shuffle1, packedBuffer,
                                      ValueRange{rem, c0});
              vector::StoreOp::create(rewriter, loc, shuffle2, packedBuffer,
                                      ValueRange{rem, nextStoreIndx});

              scf::YieldOp::create(nestedBuilder, loc);
            });
        loadRhs = amx::TileLoadOp::create(rewriter, loc, tileType, packedBuffer,
                                          ValueRange{c0, c0});
      } else {

        loadRhs = amx::TileLoadOp::create(rewriter, loc, tileType, srcBuffRhs,
                                          indicesRhs);
      }

      auto tileTypeAcc = amx::TileType::get({16, 16}, opType);
      auto loadAcc = amx::TileLoadOp::create(rewriter, loc, tileTypeAcc,
                                             srcBuffAcc, indicesAcc);

      // Tiled dot-product.
      Value dp;
      if (ipType.isBF16())
        dp = amx::TileMulFOp::create(rewriter, loc, tileTypeAcc, loadLhs,
````
- **L865 EN**: Blank line separating nearby declarations or logic blocks.
  **L865 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::StoreOp::create(rewriter, loc, shuffle1, packedBuffer,`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::StoreOp::create(rewriter, loc, shuffle1, packedBuffer,`。
- **L867 EN**: Executes a standalone statement or declaration: `ValueRange{rem, c0});`.
  **L867 CN**: 执行一条独立语句或声明：`ValueRange{rem, c0});`。
- **L868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::StoreOp::create(rewriter, loc, shuffle2, packedBuffer,`.
  **L868 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::StoreOp::create(rewriter, loc, shuffle2, packedBuffer,`。
- **L869 EN**: Executes a standalone statement or declaration: `ValueRange{rem, nextStoreIndx});`.
  **L869 CN**: 执行一条独立语句或声明：`ValueRange{rem, nextStoreIndx});`。
- **L870 EN**: Blank line separating nearby declarations or logic blocks.
  **L870 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L871 EN**: Executes a call or declaration centered on `scf::YieldOp::create`.
  **L871 CN**: 执行以 `scf::YieldOp::create` 为核心的调用或声明。
- **L872 EN**: Executes a standalone statement or declaration: `});`.
  **L872 CN**: 执行一条独立语句或声明：`});`。
- **L873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loadRhs = amx::TileLoadOp::create(rewriter, loc, tileType, packedBuffer,`.
  **L873 CN**: 继续一个多行参数列表、初始化器或聚合项：`loadRhs = amx::TileLoadOp::create(rewriter, loc, tileType, packedBuffer,`。
- **L874 EN**: Executes a standalone statement or declaration: `ValueRange{c0, c0});`.
  **L874 CN**: 执行一条独立语句或声明：`ValueRange{c0, c0});`。
- **L875 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L875 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L876 EN**: Blank line separating nearby declarations or logic blocks.
  **L876 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loadRhs = amx::TileLoadOp::create(rewriter, loc, tileType, srcBuffRhs,`.
  **L877 CN**: 继续一个多行参数列表、初始化器或聚合项：`loadRhs = amx::TileLoadOp::create(rewriter, loc, tileType, srcBuffRhs,`。
- **L878 EN**: Executes a standalone statement or declaration: `indicesRhs);`.
  **L878 CN**: 执行一条独立语句或声明：`indicesRhs);`。
- **L879 EN**: Closes the current lexical scope or compound statement.
  **L879 CN**: 结束当前词法作用域或复合语句块。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L881 EN**: Initializes variable `tileTypeAcc` from the right-hand expression.
  **L881 CN**: 使用右侧表达式初始化变量 `tileTypeAcc`。
- **L882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto loadAcc = amx::TileLoadOp::create(rewriter, loc, tileTypeAcc,`.
  **L882 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto loadAcc = amx::TileLoadOp::create(rewriter, loc, tileTypeAcc,`。
- **L883 EN**: Executes a standalone statement or declaration: `srcBuffAcc, indicesAcc);`.
  **L883 CN**: 执行一条独立语句或声明：`srcBuffAcc, indicesAcc);`。
- **L884 EN**: Blank line separating nearby declarations or logic blocks.
  **L884 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L885 EN**: Comment explains nearby logic, invariants, or intent: `Tiled dot-product.`.
  **L885 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Tiled dot-product.`。
- **L886 EN**: Executes a standalone statement or declaration: `Value dp;`.
  **L886 CN**: 执行一条独立语句或声明：`Value dp;`。
- **L887 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L887 CN**: 开始 `if` 控制流语句并计算其条件。
- **L888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dp = amx::TileMulFOp::create(rewriter, loc, tileTypeAcc, loadLhs,`.
  **L888 CN**: 继续一个多行参数列表、初始化器或聚合项：`dp = amx::TileMulFOp::create(rewriter, loc, tileTypeAcc, loadLhs,`。

### Lines 889-912

````cpp
                                     loadRhs, loadAcc);

      if (ipType.isSignlessInteger(8))
        dp = amx::TileMulIOp::create(rewriter, loc, tileTypeAcc, loadLhs,
                                     loadRhs, loadAcc);

      amx::TileStoreOp::create(rewriter, loc, srcBuffAcc, indicesAcc, dp);

      rewriter.eraseOp(resultWriteOp);
      return success();
    }

    // Case 2: The acc are passed as iter args through the reduction loop.
    // We support, reduction loop depth until 2. TODO: Support for n-depth
    // reduction loop.
    // TODOs: Re-factor 2a and 2b.
    SmallVector<scf::ForOp> loopLists;
    Operation *current = contractOp;
    while (true) {
      Operation *parent = current->getParentOfType<scf::ForOp>();

      if (!parent)
        return rewriter.notifyMatchFailure(
            contractOp,
````
- **L889 EN**: Executes a standalone statement or declaration: `loadRhs, loadAcc);`.
  **L889 CN**: 执行一条独立语句或声明：`loadRhs, loadAcc);`。
- **L890 EN**: Blank line separating nearby declarations or logic blocks.
  **L890 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L891 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L891 CN**: 开始 `if` 控制流语句并计算其条件。
- **L892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dp = amx::TileMulIOp::create(rewriter, loc, tileTypeAcc, loadLhs,`.
  **L892 CN**: 继续一个多行参数列表、初始化器或聚合项：`dp = amx::TileMulIOp::create(rewriter, loc, tileTypeAcc, loadLhs,`。
- **L893 EN**: Executes a standalone statement or declaration: `loadRhs, loadAcc);`.
  **L893 CN**: 执行一条独立语句或声明：`loadRhs, loadAcc);`。
- **L894 EN**: Blank line separating nearby declarations or logic blocks.
  **L894 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L895 EN**: Executes a call or declaration centered on `amx::TileStoreOp::create`.
  **L895 CN**: 执行以 `amx::TileStoreOp::create` 为核心的调用或声明。
- **L896 EN**: Blank line separating nearby declarations or logic blocks.
  **L896 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L897 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L897 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L898 EN**: Returns from the current function with `success()`.
  **L898 CN**: 以 `success()` 从当前函数返回。
- **L899 EN**: Closes the current lexical scope or compound statement.
  **L899 CN**: 结束当前词法作用域或复合语句块。
- **L900 EN**: Blank line separating nearby declarations or logic blocks.
  **L900 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L901 EN**: Comment explains nearby logic, invariants, or intent: `Case 2: The acc are passed as iter args through the reduction loop.`.
  **L901 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Case 2: The acc are passed as iter args through the reduction loop.`。
- **L902 EN**: Comment records a pending task or caution: `We support, reduction loop depth until 2. TODO: Support for n-depth`.
  **L902 CN**: 注释记录了待办事项或注意点：`We support, reduction loop depth until 2. TODO: Support for n-depth`。
- **L903 EN**: Comment explains nearby logic, invariants, or intent: `reduction loop.`.
  **L903 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`reduction loop.`。
- **L904 EN**: Comment records a pending task or caution: `TODOs: Re-factor 2a and 2b.`.
  **L904 CN**: 注释记录了待办事项或注意点：`TODOs: Re-factor 2a and 2b.`。
- **L905 EN**: Executes a standalone statement or declaration: `SmallVector<scf::ForOp> loopLists;`.
  **L905 CN**: 执行一条独立语句或声明：`SmallVector<scf::ForOp> loopLists;`。
- **L906 EN**: Executes a standalone statement or declaration: `Operation *current = contractOp;`.
  **L906 CN**: 执行一条独立语句或声明：`Operation *current = contractOp;`。
- **L907 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L907 CN**: 开始 `while` 控制流语句并计算其条件。
- **L908 EN**: Executes a call or declaration centered on `current->getParentOfType<scf::ForOp>`.
  **L908 CN**: 执行以 `current->getParentOfType<scf::ForOp>` 为核心的调用或声明。
- **L909 EN**: Blank line separating nearby declarations or logic blocks.
  **L909 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L910 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L910 CN**: 开始 `if` 控制流语句并计算其条件。
- **L911 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L911 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `contractOp,`.
  **L912 CN**: 继续一个多行参数列表、初始化器或聚合项：`contractOp,`。

### Lines 913-936

````cpp
            "Accumulator read and contract op not within scf.for op");

      loopLists.push_back(dyn_cast<scf::ForOp>(parent));

      if (accReadOp->getBlock() == parent->getBlock()) {
        break;
      }

      current = parent;
    }
    if (loopLists.size() > 2 || loopLists.size() == 0)
      return rewriter.notifyMatchFailure(
          contractOp, "Rewrite is supported until reduction loop depth of 2.");

    auto srcIndxLhs = getSrcIndxValue(rewriter, contractOp.getLoc(),
                                      contractOp.getLhs(), false);
    if (failed(srcIndxLhs))
      return rewriter.notifyMatchFailure(contractOp,
                                         "The LHS src is not a MemRef type.");
    auto [srcBuffLhs, indicesLhs] = *srcIndxLhs;

    auto srcIndxRhs = getSrcIndxValue(rewriter, contractOp.getLoc(),
                                      contractOp.getRhs(), false);
    if (failed(srcIndxRhs))
````
- **L913 EN**: Executes a standalone statement or declaration: `"Accumulator read and contract op not within scf.for op");`.
  **L913 CN**: 执行一条独立语句或声明：`"Accumulator read and contract op not within scf.for op");`。
- **L914 EN**: Blank line separating nearby declarations or logic blocks.
  **L914 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L915 EN**: Executes a call or declaration centered on `loopLists.push_back`.
  **L915 CN**: 执行以 `loopLists.push_back` 为核心的调用或声明。
- **L916 EN**: Blank line separating nearby declarations or logic blocks.
  **L916 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L917 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L917 CN**: 开始 `if` 控制流语句并计算其条件。
- **L918 EN**: Exits the nearest loop or switch statement.
  **L918 CN**: 退出最近的循环或 switch 语句。
- **L919 EN**: Closes the current lexical scope or compound statement.
  **L919 CN**: 结束当前词法作用域或复合语句块。
- **L920 EN**: Blank line separating nearby declarations or logic blocks.
  **L920 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L921 EN**: Executes a standalone statement or declaration: `current = parent;`.
  **L921 CN**: 执行一条独立语句或声明：`current = parent;`。
- **L922 EN**: Closes the current lexical scope or compound statement.
  **L922 CN**: 结束当前词法作用域或复合语句块。
- **L923 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L923 CN**: 开始 `if` 控制流语句并计算其条件。
- **L924 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L924 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L925 EN**: Executes a standalone statement or declaration: `contractOp, "Rewrite is supported until reduction loop depth of 2.");`.
  **L925 CN**: 执行一条独立语句或声明：`contractOp, "Rewrite is supported until reduction loop depth of 2.");`。
- **L926 EN**: Blank line separating nearby declarations or logic blocks.
  **L926 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto srcIndxLhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`.
  **L927 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto srcIndxLhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`。
- **L928 EN**: Executes a call or declaration centered on `contractOp.getLhs`.
  **L928 CN**: 执行以 `contractOp.getLhs` 为核心的调用或声明。
- **L929 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L929 CN**: 开始 `if` 控制流语句并计算其条件。
- **L930 EN**: Returns from the current function with `rewriter.notifyMatchFailure(contractOp,`.
  **L930 CN**: 以 `rewriter.notifyMatchFailure(contractOp,` 从当前函数返回。
- **L931 EN**: Executes a standalone statement or declaration: `"The LHS src is not a MemRef type.");`.
  **L931 CN**: 执行一条独立语句或声明：`"The LHS src is not a MemRef type.");`。
- **L932 EN**: Executes a standalone statement or declaration: `auto [srcBuffLhs, indicesLhs] = *srcIndxLhs;`.
  **L932 CN**: 执行一条独立语句或声明：`auto [srcBuffLhs, indicesLhs] = *srcIndxLhs;`。
- **L933 EN**: Blank line separating nearby declarations or logic blocks.
  **L933 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto srcIndxRhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`.
  **L934 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto srcIndxRhs = getSrcIndxValue(rewriter, contractOp.getLoc(),`。
- **L935 EN**: Executes a call or declaration centered on `contractOp.getRhs`.
  **L935 CN**: 执行以 `contractOp.getRhs` 为核心的调用或声明。
- **L936 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L936 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 937-960

````cpp
      return rewriter.notifyMatchFailure(contractOp,
                                         "The RHS src is not a MemRef type.");
    auto [srcBuffRhs, indicesRhs] = *srcIndxRhs;
    Operation *vectorOpLhs;
    llvm::TypeSwitch<Operation *>(contractOp.getLhs().getDefiningOp())
        .Case<TransferReadOp, LoadOp>([&](auto readOp) {
          vectorOpLhs = readOp.getBase().getDefiningOp();
        });

    Operation *vectorOpRhs;
    llvm::TypeSwitch<Operation *>(contractOp.getRhs().getDefiningOp())
        .Case<TransferReadOp, LoadOp>([&](auto readOp) {
          vectorOpRhs = readOp.getBase().getDefiningOp();
        });

    // Retrive all the contaction operation within the loop.
    SmallVector<vector::ContractionOp> ops;
    for (mlir::Operation &op : loopLists[0].getBody()->getOperations()) {

      if (auto contract = llvm::dyn_cast<mlir::vector::ContractionOp>(op)) {

        LogicalResult validate = validateContractOps(
            rewriter, contract, dimValue, srcBuffLhs, srcBuffRhs, true);

````
- **L937 EN**: Returns from the current function with `rewriter.notifyMatchFailure(contractOp,`.
  **L937 CN**: 以 `rewriter.notifyMatchFailure(contractOp,` 从当前函数返回。
- **L938 EN**: Executes a standalone statement or declaration: `"The RHS src is not a MemRef type.");`.
  **L938 CN**: 执行一条独立语句或声明：`"The RHS src is not a MemRef type.");`。
- **L939 EN**: Executes a standalone statement or declaration: `auto [srcBuffRhs, indicesRhs] = *srcIndxRhs;`.
  **L939 CN**: 执行一条独立语句或声明：`auto [srcBuffRhs, indicesRhs] = *srcIndxRhs;`。
- **L940 EN**: Executes a standalone statement or declaration: `Operation *vectorOpLhs;`.
  **L940 CN**: 执行一条独立语句或声明：`Operation *vectorOpLhs;`。
- **L941 EN**: Continues logic associated with callable symbol `getLhs`.
  **L941 CN**: 继续与可调用符号 `getLhs` 相关的逻辑。
- **L942 EN**: Starts a function, method, lambda, or structured scope: `.Case<TransferReadOp, LoadOp>([&](auto readOp) {`.
  **L942 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.Case<TransferReadOp, LoadOp>([&](auto readOp) {`。
- **L943 EN**: Executes a call or declaration centered on `readOp.getBase`.
  **L943 CN**: 执行以 `readOp.getBase` 为核心的调用或声明。
- **L944 EN**: Executes a standalone statement or declaration: `});`.
  **L944 CN**: 执行一条独立语句或声明：`});`。
- **L945 EN**: Blank line separating nearby declarations or logic blocks.
  **L945 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L946 EN**: Executes a standalone statement or declaration: `Operation *vectorOpRhs;`.
  **L946 CN**: 执行一条独立语句或声明：`Operation *vectorOpRhs;`。
- **L947 EN**: Continues logic associated with callable symbol `getRhs`.
  **L947 CN**: 继续与可调用符号 `getRhs` 相关的逻辑。
- **L948 EN**: Starts a function, method, lambda, or structured scope: `.Case<TransferReadOp, LoadOp>([&](auto readOp) {`.
  **L948 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.Case<TransferReadOp, LoadOp>([&](auto readOp) {`。
- **L949 EN**: Executes a call or declaration centered on `readOp.getBase`.
  **L949 CN**: 执行以 `readOp.getBase` 为核心的调用或声明。
- **L950 EN**: Executes a standalone statement or declaration: `});`.
  **L950 CN**: 执行一条独立语句或声明：`});`。
- **L951 EN**: Blank line separating nearby declarations or logic blocks.
  **L951 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L952 EN**: Comment explains nearby logic, invariants, or intent: `Retrive all the contaction operation within the loop.`.
  **L952 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Retrive all the contaction operation within the loop.`。
- **L953 EN**: Executes a standalone statement or declaration: `SmallVector<vector::ContractionOp> ops;`.
  **L953 CN**: 执行一条独立语句或声明：`SmallVector<vector::ContractionOp> ops;`。
- **L954 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L954 CN**: 开始 `for` 控制流语句并计算其条件。
- **L955 EN**: Blank line separating nearby declarations or logic blocks.
  **L955 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L956 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L956 CN**: 开始 `if` 控制流语句并计算其条件。
- **L957 EN**: Blank line separating nearby declarations or logic blocks.
  **L957 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L958 EN**: Continues logic associated with callable symbol `validateContractOps`.
  **L958 CN**: 继续与可调用符号 `validateContractOps` 相关的逻辑。
- **L959 EN**: Executes a standalone statement or declaration: `rewriter, contract, dimValue, srcBuffLhs, srcBuffRhs, true);`.
  **L959 CN**: 执行一条独立语句或声明：`rewriter, contract, dimValue, srcBuffLhs, srcBuffRhs, true);`。
- **L960 EN**: Blank line separating nearby declarations or logic blocks.
  **L960 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 961-984

````cpp
        if (failed(validate))
          return rewriter.notifyMatchFailure(
              contractOp, "The associated contract operations doesn't satisfy "
                          "the re-write conditions either the dimensions are "
                          "wrong or MemRef source are different.");

        ops.push_back(contract);
      }
    }

    if (!isVnni) {
      unsigned int pairCount = 0;
      for (size_t j = 0; j < ops.size(); j++) {
        for (size_t i = j; i < ops.size(); i++) {
          if (i != j && validatePairVectorContract(ops[j], ops[i], true, 16))
            pairCount = pairCount + 2;
        }
      }

      if (pairCount != ops.size())
        return rewriter.notifyMatchFailure(
            contractOp, "Coudn't find the pair vector contract ");
    }

````
- **L961 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L961 CN**: 开始 `if` 控制流语句并计算其条件。
- **L962 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L962 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L963 EN**: Continues the surrounding expression or declaration: `contractOp, "The associated contract operations doesn't satisfy "`.
  **L963 CN**: 继续构造周围的表达式或声明：`contractOp, "The associated contract operations doesn't satisfy "`。
- **L964 EN**: Continues the surrounding expression or declaration: `"the re-write conditions either the dimensions are "`.
  **L964 CN**: 继续构造周围的表达式或声明：`"the re-write conditions either the dimensions are "`。
- **L965 EN**: Executes a standalone statement or declaration: `"wrong or MemRef source are different.");`.
  **L965 CN**: 执行一条独立语句或声明：`"wrong or MemRef source are different.");`。
- **L966 EN**: Blank line separating nearby declarations or logic blocks.
  **L966 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L967 EN**: Executes a call or declaration centered on `ops.push_back`.
  **L967 CN**: 执行以 `ops.push_back` 为核心的调用或声明。
- **L968 EN**: Closes the current lexical scope or compound statement.
  **L968 CN**: 结束当前词法作用域或复合语句块。
- **L969 EN**: Closes the current lexical scope or compound statement.
  **L969 CN**: 结束当前词法作用域或复合语句块。
- **L970 EN**: Blank line separating nearby declarations or logic blocks.
  **L970 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L971 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L971 CN**: 开始 `if` 控制流语句并计算其条件。
- **L972 EN**: Initializes variable `pairCount` from the right-hand expression.
  **L972 CN**: 使用右侧表达式初始化变量 `pairCount`。
- **L973 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L973 CN**: 开始 `for` 控制流语句并计算其条件。
- **L974 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L974 CN**: 开始 `for` 控制流语句并计算其条件。
- **L975 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L975 CN**: 开始 `if` 控制流语句并计算其条件。
- **L976 EN**: Executes a standalone statement or declaration: `pairCount = pairCount + 2;`.
  **L976 CN**: 执行一条独立语句或声明：`pairCount = pairCount + 2;`。
- **L977 EN**: Closes the current lexical scope or compound statement.
  **L977 CN**: 结束当前词法作用域或复合语句块。
- **L978 EN**: Closes the current lexical scope or compound statement.
  **L978 CN**: 结束当前词法作用域或复合语句块。
- **L979 EN**: Blank line separating nearby declarations or logic blocks.
  **L979 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L980 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L980 CN**: 开始 `if` 控制流语句并计算其条件。
- **L981 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L981 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L982 EN**: Executes a standalone statement or declaration: `contractOp, "Coudn't find the pair vector contract ");`.
  **L982 CN**: 执行一条独立语句或声明：`contractOp, "Coudn't find the pair vector contract ");`。
- **L983 EN**: Closes the current lexical scope or compound statement.
  **L983 CN**: 结束当前词法作用域或复合语句块。
- **L984 EN**: Blank line separating nearby declarations or logic blocks.
  **L984 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 985-1008

````cpp
    scf::ForOp innerLoop;
    scf::ForOp outerLoop;

    scf::ForOp newLoop;
    // Case 2a: Reduction loop depth is 2.
    if (loopLists.size() == 2) {
      outerLoop = loopLists[1];
      innerLoop = loopLists[0];

      SmallVector<Value> loopItrArgs = createTileZeros(
          rewriter, outerLoop.getLoc(), opType, outerLoop, ops.size());

      if (isVnni) {
        newLoop = scf::ForOp::create(
            rewriter, outerLoop.getLoc(), outerLoop.getLowerBound(),
            outerLoop.getUpperBound(), outerLoop.getStep(), loopItrArgs,
            [&](OpBuilder &rewriterOuterLoop, Location locOuterLoop,
                Value ivOuterLoop, ValueRange iterArgsOuterLoop) {
              auto newInnerLoop = createLoops(
                  rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),
                  innerLoop.getUpperBound(), innerLoop.getStep(),
                  iterArgsOuterLoop, ipType, opType, blockingFactor, isVnni,
                  vectorOpLhs, vectorOpRhs, contractOp, outerLoop, innerLoop,
                  ops, ivOuterLoop, nullptr, true, nullptr, false, false);
````
- **L985 EN**: Executes a standalone statement or declaration: `scf::ForOp innerLoop;`.
  **L985 CN**: 执行一条独立语句或声明：`scf::ForOp innerLoop;`。
- **L986 EN**: Executes a standalone statement or declaration: `scf::ForOp outerLoop;`.
  **L986 CN**: 执行一条独立语句或声明：`scf::ForOp outerLoop;`。
- **L987 EN**: Blank line separating nearby declarations or logic blocks.
  **L987 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L988 EN**: Executes a standalone statement or declaration: `scf::ForOp newLoop;`.
  **L988 CN**: 执行一条独立语句或声明：`scf::ForOp newLoop;`。
- **L989 EN**: Comment explains nearby logic, invariants, or intent: `Case 2a: Reduction loop depth is 2.`.
  **L989 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Case 2a: Reduction loop depth is 2.`。
- **L990 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L990 CN**: 开始 `if` 控制流语句并计算其条件。
- **L991 EN**: Executes a standalone statement or declaration: `outerLoop = loopLists[1];`.
  **L991 CN**: 执行一条独立语句或声明：`outerLoop = loopLists[1];`。
- **L992 EN**: Executes a standalone statement or declaration: `innerLoop = loopLists[0];`.
  **L992 CN**: 执行一条独立语句或声明：`innerLoop = loopLists[0];`。
- **L993 EN**: Blank line separating nearby declarations or logic blocks.
  **L993 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L994 EN**: Continues logic associated with callable symbol `createTileZeros`.
  **L994 CN**: 继续与可调用符号 `createTileZeros` 相关的逻辑。
- **L995 EN**: Executes a call or declaration centered on `outerLoop.getLoc`.
  **L995 CN**: 执行以 `outerLoop.getLoc` 为核心的调用或声明。
- **L996 EN**: Blank line separating nearby declarations or logic blocks.
  **L996 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L997 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L997 CN**: 开始 `if` 控制流语句并计算其条件。
- **L998 EN**: Continues logic associated with callable symbol `create`.
  **L998 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, outerLoop.getLoc(), outerLoop.getLowerBound(),`.
  **L999 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, outerLoop.getLoc(), outerLoop.getLowerBound(),`。
- **L1000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `outerLoop.getUpperBound(), outerLoop.getStep(), loopItrArgs,`.
  **L1000 CN**: 继续一个多行参数列表、初始化器或聚合项：`outerLoop.getUpperBound(), outerLoop.getStep(), loopItrArgs,`。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](OpBuilder &rewriterOuterLoop, Location locOuterLoop,`.
  **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](OpBuilder &rewriterOuterLoop, Location locOuterLoop,`。
- **L1002 EN**: Continues the surrounding expression or declaration: `Value ivOuterLoop, ValueRange iterArgsOuterLoop) {`.
  **L1002 CN**: 继续构造周围的表达式或声明：`Value ivOuterLoop, ValueRange iterArgsOuterLoop) {`。
- **L1003 EN**: Continues logic associated with callable symbol `createLoops`.
  **L1003 CN**: 继续与可调用符号 `createLoops` 相关的逻辑。
- **L1004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),`.
  **L1004 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),`。
- **L1005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `innerLoop.getUpperBound(), innerLoop.getStep(),`.
  **L1005 CN**: 继续一个多行参数列表、初始化器或聚合项：`innerLoop.getUpperBound(), innerLoop.getStep(),`。
- **L1006 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `iterArgsOuterLoop, ipType, opType, blockingFactor, isVnni,`.
  **L1006 CN**: 继续一个多行参数列表、初始化器或聚合项：`iterArgsOuterLoop, ipType, opType, blockingFactor, isVnni,`。
- **L1007 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vectorOpLhs, vectorOpRhs, contractOp, outerLoop, innerLoop,`.
  **L1007 CN**: 继续一个多行参数列表、初始化器或聚合项：`vectorOpLhs, vectorOpRhs, contractOp, outerLoop, innerLoop,`。
- **L1008 EN**: Executes a standalone statement or declaration: `ops, ivOuterLoop, nullptr, true, nullptr, false, false);`.
  **L1008 CN**: 执行一条独立语句或声明：`ops, ivOuterLoop, nullptr, true, nullptr, false, false);`。

### Lines 1009-1032

````cpp

              scf::YieldOp::create(rewriterOuterLoop, locOuterLoop,
                                   newInnerLoop.getResults());
            });

      } else {

        bool isInnerLoopUBLarger = false;
        bool isInnerLoopUBHasOddQuot = false;

        int64_t ubVal = 16 * blockingFactor;
        mlir::Value ub = innerLoop.getUpperBound();
        if (auto constOp = ub.getDefiningOp<mlir::arith::ConstantOp>()) {
          if (auto intAttr =
                  llvm::dyn_cast<mlir::IntegerAttr>(constOp.getValue())) {
            ubVal = intAttr.getInt();
          }
        }

        isInnerLoopUBLarger = ubVal > 16 * blockingFactor;
        isInnerLoopUBHasOddQuot =
            (((ubVal / (16 * blockingFactor)) % 2) == 1) && isInnerLoopUBLarger;

        rewriter.setInsertionPoint(outerLoop);
````
- **L1009 EN**: Blank line separating nearby declarations or logic blocks.
  **L1009 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `scf::YieldOp::create(rewriterOuterLoop, locOuterLoop,`.
  **L1010 CN**: 继续一个多行参数列表、初始化器或聚合项：`scf::YieldOp::create(rewriterOuterLoop, locOuterLoop,`。
- **L1011 EN**: Executes a call or declaration centered on `newInnerLoop.getResults`.
  **L1011 CN**: 执行以 `newInnerLoop.getResults` 为核心的调用或声明。
- **L1012 EN**: Executes a standalone statement or declaration: `});`.
  **L1012 CN**: 执行一条独立语句或声明：`});`。
- **L1013 EN**: Blank line separating nearby declarations or logic blocks.
  **L1013 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1014 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L1014 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L1015 EN**: Blank line separating nearby declarations or logic blocks.
  **L1015 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1016 EN**: Initializes variable `isInnerLoopUBLarger` from the right-hand expression.
  **L1016 CN**: 使用右侧表达式初始化变量 `isInnerLoopUBLarger`。
- **L1017 EN**: Initializes variable `isInnerLoopUBHasOddQuot` from the right-hand expression.
  **L1017 CN**: 使用右侧表达式初始化变量 `isInnerLoopUBHasOddQuot`。
- **L1018 EN**: Blank line separating nearby declarations or logic blocks.
  **L1018 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1019 EN**: Initializes variable `ubVal` from the right-hand expression.
  **L1019 CN**: 使用右侧表达式初始化变量 `ubVal`。
- **L1020 EN**: Initializes variable `ub` from the right-hand expression.
  **L1020 CN**: 使用右侧表达式初始化变量 `ub`。
- **L1021 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1021 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1022 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1022 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1023 EN**: Starts a function, method, lambda, or structured scope: `llvm::dyn_cast<mlir::IntegerAttr>(constOp.getValue())) {`.
  **L1023 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::dyn_cast<mlir::IntegerAttr>(constOp.getValue())) {`。
- **L1024 EN**: Executes a call or declaration centered on `intAttr.getInt`.
  **L1024 CN**: 执行以 `intAttr.getInt` 为核心的调用或声明。
- **L1025 EN**: Closes the current lexical scope or compound statement.
  **L1025 CN**: 结束当前词法作用域或复合语句块。
- **L1026 EN**: Closes the current lexical scope or compound statement.
  **L1026 CN**: 结束当前词法作用域或复合语句块。
- **L1027 EN**: Blank line separating nearby declarations or logic blocks.
  **L1027 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1028 EN**: Executes a standalone statement or declaration: `isInnerLoopUBLarger = ubVal > 16 * blockingFactor;`.
  **L1028 CN**: 执行一条独立语句或声明：`isInnerLoopUBLarger = ubVal > 16 * blockingFactor;`。
- **L1029 EN**: Continues the surrounding expression or declaration: `isInnerLoopUBHasOddQuot =`.
  **L1029 CN**: 继续构造周围的表达式或声明：`isInnerLoopUBHasOddQuot =`。
- **L1030 EN**: Executes a call or declaration centered on `statement`.
  **L1030 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1031 EN**: Blank line separating nearby declarations or logic blocks.
  **L1031 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1032 EN**: Executes a call or declaration centered on `rewriter.setInsertionPoint`.
  **L1032 CN**: 执行以 `rewriter.setInsertionPoint` 为核心的调用或声明。

### Lines 1033-1056

````cpp

        auto c0 =
            arith::ConstantIndexOp::create(rewriter, outerLoop.getLoc(), 0);
        auto c1 =
            arith::ConstantIndexOp::create(rewriter, outerLoop.getLoc(), 1);
        auto spillLoopBound = arith::ConstantIndexOp::create(
            rewriter, outerLoop.getLoc(), 16 * blockingFactor);

        Value spillOuterLoop = arith::SubIOp::create(
            rewriter, outerLoop.getLoc(), outerLoop.getUpperBound(), c1);
        Value spillInnerLoop =
            arith::SubIOp::create(rewriter, innerLoop.getLoc(),
                                  innerLoop.getUpperBound(), spillLoopBound);
        auto bufferType =
            MemRefType::get({2, 32, (blockingFactor * 16)}, ipType);
        auto packedBuffer =
            memref::AllocaOp::create(rewriter, outerLoop.getLoc(), bufferType);

        // First Shuffling outside the reduction loops
        IRMapping rhsMapping;
        rhsMapping.map(
            vectorOpRhs->getOperand(
                getIndexPosition(contractOp.getRhs(), outerLoop) + 1),
            c0);
````
- **L1033 EN**: Blank line separating nearby declarations or logic blocks.
  **L1033 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1034 EN**: Continues the surrounding expression or declaration: `auto c0 =`.
  **L1034 CN**: 继续构造周围的表达式或声明：`auto c0 =`。
- **L1035 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L1035 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L1036 EN**: Continues the surrounding expression or declaration: `auto c1 =`.
  **L1036 CN**: 继续构造周围的表达式或声明：`auto c1 =`。
- **L1037 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L1037 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L1038 EN**: Continues logic associated with callable symbol `create`.
  **L1038 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1039 EN**: Executes a call or declaration centered on `outerLoop.getLoc`.
  **L1039 CN**: 执行以 `outerLoop.getLoc` 为核心的调用或声明。
- **L1040 EN**: Blank line separating nearby declarations or logic blocks.
  **L1040 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1041 EN**: Continues logic associated with callable symbol `create`.
  **L1041 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1042 EN**: Executes a call or declaration centered on `outerLoop.getLoc`.
  **L1042 CN**: 执行以 `outerLoop.getLoc` 为核心的调用或声明。
- **L1043 EN**: Continues the surrounding expression or declaration: `Value spillInnerLoop =`.
  **L1043 CN**: 继续构造周围的表达式或声明：`Value spillInnerLoop =`。
- **L1044 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arith::SubIOp::create(rewriter, innerLoop.getLoc(),`.
  **L1044 CN**: 继续一个多行参数列表、初始化器或聚合项：`arith::SubIOp::create(rewriter, innerLoop.getLoc(),`。
- **L1045 EN**: Executes a call or declaration centered on `innerLoop.getUpperBound`.
  **L1045 CN**: 执行以 `innerLoop.getUpperBound` 为核心的调用或声明。
- **L1046 EN**: Continues the surrounding expression or declaration: `auto bufferType =`.
  **L1046 CN**: 继续构造周围的表达式或声明：`auto bufferType =`。
- **L1047 EN**: Executes a call or declaration centered on `MemRefType::get`.
  **L1047 CN**: 执行以 `MemRefType::get` 为核心的调用或声明。
- **L1048 EN**: Continues the surrounding expression or declaration: `auto packedBuffer =`.
  **L1048 CN**: 继续构造周围的表达式或声明：`auto packedBuffer =`。
- **L1049 EN**: Executes a call or declaration centered on `memref::AllocaOp::create`.
  **L1049 CN**: 执行以 `memref::AllocaOp::create` 为核心的调用或声明。
- **L1050 EN**: Blank line separating nearby declarations or logic blocks.
  **L1050 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1051 EN**: Comment explains nearby logic, invariants, or intent: `First Shuffling outside the reduction loops`.
  **L1051 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`First Shuffling outside the reduction loops`。
- **L1052 EN**: Executes a standalone statement or declaration: `IRMapping rhsMapping;`.
  **L1052 CN**: 执行一条独立语句或声明：`IRMapping rhsMapping;`。
- **L1053 EN**: Continues logic associated with callable symbol `map`.
  **L1053 CN**: 继续与可调用符号 `map` 相关的逻辑。
- **L1054 EN**: Continues logic associated with callable symbol `getOperand`.
  **L1054 CN**: 继续与可调用符号 `getOperand` 相关的逻辑。
- **L1055 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getIndexPosition(contractOp.getRhs(), outerLoop) + 1),`.
  **L1055 CN**: 继续一个多行参数列表、初始化器或聚合项：`getIndexPosition(contractOp.getRhs(), outerLoop) + 1),`。
- **L1056 EN**: Executes a standalone statement or declaration: `c0);`.
  **L1056 CN**: 执行一条独立语句或声明：`c0);`。

### Lines 1057-1080

````cpp
        rhsMapping.map(
            vectorOpRhs->getOperand(
                getIndexPosition(contractOp.getRhs(), innerLoop) + 1),
            c0);
        auto rhsClone = rewriter.clone(*vectorOpRhs, rhsMapping);

        performShuffle(rewriter, outerLoop.getLoc(), rhsClone->getResult(0),
                       ipType, blockingFactor, packedBuffer, c0);

        // First Set of Loops
        auto newLoopNonSpill = scf::ForOp::create(
            rewriter, outerLoop.getLoc(), outerLoop.getLowerBound(),
            spillOuterLoop, outerLoop.getStep(), loopItrArgs,
            [&](OpBuilder &rewriterOuterLoop, Location locOuterLoop,
                Value ivOuterLoop, ValueRange iterArgsOuterLoop) {
              auto newInnerLoop1 = createLoops(
                  rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),
                  spillInnerLoop, innerLoop.getStep(), iterArgsOuterLoop,
                  ipType, opType, blockingFactor, isVnni, vectorOpLhs,
                  vectorOpRhs, contractOp, outerLoop, innerLoop, ops,
                  ivOuterLoop, packedBuffer, true, spillLoopBound,
                  isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);

              auto newInnerLoop = createLoops(
````
- **L1057 EN**: Continues logic associated with callable symbol `map`.
  **L1057 CN**: 继续与可调用符号 `map` 相关的逻辑。
- **L1058 EN**: Continues logic associated with callable symbol `getOperand`.
  **L1058 CN**: 继续与可调用符号 `getOperand` 相关的逻辑。
- **L1059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getIndexPosition(contractOp.getRhs(), innerLoop) + 1),`.
  **L1059 CN**: 继续一个多行参数列表、初始化器或聚合项：`getIndexPosition(contractOp.getRhs(), innerLoop) + 1),`。
- **L1060 EN**: Executes a standalone statement or declaration: `c0);`.
  **L1060 CN**: 执行一条独立语句或声明：`c0);`。
- **L1061 EN**: Initializes variable `rhsClone` from the right-hand expression.
  **L1061 CN**: 使用右侧表达式初始化变量 `rhsClone`。
- **L1062 EN**: Blank line separating nearby declarations or logic blocks.
  **L1062 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1063 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `performShuffle(rewriter, outerLoop.getLoc(), rhsClone->getResult(0),`.
  **L1063 CN**: 继续一个多行参数列表、初始化器或聚合项：`performShuffle(rewriter, outerLoop.getLoc(), rhsClone->getResult(0),`。
- **L1064 EN**: Executes a standalone statement or declaration: `ipType, blockingFactor, packedBuffer, c0);`.
  **L1064 CN**: 执行一条独立语句或声明：`ipType, blockingFactor, packedBuffer, c0);`。
- **L1065 EN**: Blank line separating nearby declarations or logic blocks.
  **L1065 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1066 EN**: Comment explains nearby logic, invariants, or intent: `First Set of Loops`.
  **L1066 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`First Set of Loops`。
- **L1067 EN**: Continues logic associated with callable symbol `create`.
  **L1067 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1068 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, outerLoop.getLoc(), outerLoop.getLowerBound(),`.
  **L1068 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, outerLoop.getLoc(), outerLoop.getLowerBound(),`。
- **L1069 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `spillOuterLoop, outerLoop.getStep(), loopItrArgs,`.
  **L1069 CN**: 继续一个多行参数列表、初始化器或聚合项：`spillOuterLoop, outerLoop.getStep(), loopItrArgs,`。
- **L1070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](OpBuilder &rewriterOuterLoop, Location locOuterLoop,`.
  **L1070 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](OpBuilder &rewriterOuterLoop, Location locOuterLoop,`。
- **L1071 EN**: Continues the surrounding expression or declaration: `Value ivOuterLoop, ValueRange iterArgsOuterLoop) {`.
  **L1071 CN**: 继续构造周围的表达式或声明：`Value ivOuterLoop, ValueRange iterArgsOuterLoop) {`。
- **L1072 EN**: Continues logic associated with callable symbol `createLoops`.
  **L1072 CN**: 继续与可调用符号 `createLoops` 相关的逻辑。
- **L1073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),`.
  **L1073 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),`。
- **L1074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `spillInnerLoop, innerLoop.getStep(), iterArgsOuterLoop,`.
  **L1074 CN**: 继续一个多行参数列表、初始化器或聚合项：`spillInnerLoop, innerLoop.getStep(), iterArgsOuterLoop,`。
- **L1075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ipType, opType, blockingFactor, isVnni, vectorOpLhs,`.
  **L1075 CN**: 继续一个多行参数列表、初始化器或聚合项：`ipType, opType, blockingFactor, isVnni, vectorOpLhs,`。
- **L1076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vectorOpRhs, contractOp, outerLoop, innerLoop, ops,`.
  **L1076 CN**: 继续一个多行参数列表、初始化器或聚合项：`vectorOpRhs, contractOp, outerLoop, innerLoop, ops,`。
- **L1077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ivOuterLoop, packedBuffer, true, spillLoopBound,`.
  **L1077 CN**: 继续一个多行参数列表、初始化器或聚合项：`ivOuterLoop, packedBuffer, true, spillLoopBound,`。
- **L1078 EN**: Executes a standalone statement or declaration: `isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);`.
  **L1078 CN**: 执行一条独立语句或声明：`isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);`。
- **L1079 EN**: Blank line separating nearby declarations or logic blocks.
  **L1079 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1080 EN**: Continues logic associated with callable symbol `createLoops`.
  **L1080 CN**: 继续与可调用符号 `createLoops` 相关的逻辑。

### Lines 1081-1104

````cpp
                  rewriter, innerLoop.getLoc(), spillInnerLoop,
                  innerLoop.getUpperBound(), innerLoop.getStep(),
                  newInnerLoop1.getResults(), ipType, opType, blockingFactor,
                  isVnni, vectorOpLhs, vectorOpRhs, contractOp, outerLoop,
                  innerLoop, ops, ivOuterLoop, packedBuffer, true, c0,
                  isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);

              scf::YieldOp::create(rewriterOuterLoop, locOuterLoop,
                                   newInnerLoop.getResults());
            });

        // Last set of Loops
        newLoop = scf::ForOp::create(
            rewriter, outerLoop.getLoc(), spillOuterLoop,
            outerLoop.getUpperBound(), outerLoop.getStep(),
            newLoopNonSpill.getResults(),
            [&](OpBuilder &rewriterOuterLoop, Location locOuterLoop,
                Value ivOuterLoop, ValueRange iterArgsOuterLoop) {
              auto newInnerLoop1 = createLoops(
                  rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),
                  spillInnerLoop, innerLoop.getStep(), iterArgsOuterLoop,
                  ipType, opType, blockingFactor, isVnni, vectorOpLhs,
                  vectorOpRhs, contractOp, outerLoop, innerLoop, ops,
                  ivOuterLoop, packedBuffer, true, spillLoopBound,
````
- **L1081 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, innerLoop.getLoc(), spillInnerLoop,`.
  **L1081 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, innerLoop.getLoc(), spillInnerLoop,`。
- **L1082 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `innerLoop.getUpperBound(), innerLoop.getStep(),`.
  **L1082 CN**: 继续一个多行参数列表、初始化器或聚合项：`innerLoop.getUpperBound(), innerLoop.getStep(),`。
- **L1083 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `newInnerLoop1.getResults(), ipType, opType, blockingFactor,`.
  **L1083 CN**: 继续一个多行参数列表、初始化器或聚合项：`newInnerLoop1.getResults(), ipType, opType, blockingFactor,`。
- **L1084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `isVnni, vectorOpLhs, vectorOpRhs, contractOp, outerLoop,`.
  **L1084 CN**: 继续一个多行参数列表、初始化器或聚合项：`isVnni, vectorOpLhs, vectorOpRhs, contractOp, outerLoop,`。
- **L1085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `innerLoop, ops, ivOuterLoop, packedBuffer, true, c0,`.
  **L1085 CN**: 继续一个多行参数列表、初始化器或聚合项：`innerLoop, ops, ivOuterLoop, packedBuffer, true, c0,`。
- **L1086 EN**: Executes a standalone statement or declaration: `isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);`.
  **L1086 CN**: 执行一条独立语句或声明：`isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);`。
- **L1087 EN**: Blank line separating nearby declarations or logic blocks.
  **L1087 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1088 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `scf::YieldOp::create(rewriterOuterLoop, locOuterLoop,`.
  **L1088 CN**: 继续一个多行参数列表、初始化器或聚合项：`scf::YieldOp::create(rewriterOuterLoop, locOuterLoop,`。
- **L1089 EN**: Executes a call or declaration centered on `newInnerLoop.getResults`.
  **L1089 CN**: 执行以 `newInnerLoop.getResults` 为核心的调用或声明。
- **L1090 EN**: Executes a standalone statement or declaration: `});`.
  **L1090 CN**: 执行一条独立语句或声明：`});`。
- **L1091 EN**: Blank line separating nearby declarations or logic blocks.
  **L1091 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1092 EN**: Comment explains nearby logic, invariants, or intent: `Last set of Loops`.
  **L1092 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Last set of Loops`。
- **L1093 EN**: Continues logic associated with callable symbol `create`.
  **L1093 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, outerLoop.getLoc(), spillOuterLoop,`.
  **L1094 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, outerLoop.getLoc(), spillOuterLoop,`。
- **L1095 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `outerLoop.getUpperBound(), outerLoop.getStep(),`.
  **L1095 CN**: 继续一个多行参数列表、初始化器或聚合项：`outerLoop.getUpperBound(), outerLoop.getStep(),`。
- **L1096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `newLoopNonSpill.getResults(),`.
  **L1096 CN**: 继续一个多行参数列表、初始化器或聚合项：`newLoopNonSpill.getResults(),`。
- **L1097 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](OpBuilder &rewriterOuterLoop, Location locOuterLoop,`.
  **L1097 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](OpBuilder &rewriterOuterLoop, Location locOuterLoop,`。
- **L1098 EN**: Continues the surrounding expression or declaration: `Value ivOuterLoop, ValueRange iterArgsOuterLoop) {`.
  **L1098 CN**: 继续构造周围的表达式或声明：`Value ivOuterLoop, ValueRange iterArgsOuterLoop) {`。
- **L1099 EN**: Continues logic associated with callable symbol `createLoops`.
  **L1099 CN**: 继续与可调用符号 `createLoops` 相关的逻辑。
- **L1100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),`.
  **L1100 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),`。
- **L1101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `spillInnerLoop, innerLoop.getStep(), iterArgsOuterLoop,`.
  **L1101 CN**: 继续一个多行参数列表、初始化器或聚合项：`spillInnerLoop, innerLoop.getStep(), iterArgsOuterLoop,`。
- **L1102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ipType, opType, blockingFactor, isVnni, vectorOpLhs,`.
  **L1102 CN**: 继续一个多行参数列表、初始化器或聚合项：`ipType, opType, blockingFactor, isVnni, vectorOpLhs,`。
- **L1103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vectorOpRhs, contractOp, outerLoop, innerLoop, ops,`.
  **L1103 CN**: 继续一个多行参数列表、初始化器或聚合项：`vectorOpRhs, contractOp, outerLoop, innerLoop, ops,`。
- **L1104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ivOuterLoop, packedBuffer, true, spillLoopBound,`.
  **L1104 CN**: 继续一个多行参数列表、初始化器或聚合项：`ivOuterLoop, packedBuffer, true, spillLoopBound,`。

### Lines 1105-1128

````cpp
                  isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);

              auto newInnerLoop = createLoops(
                  rewriter, innerLoop.getLoc(), spillInnerLoop,
                  innerLoop.getUpperBound(), innerLoop.getStep(),
                  newInnerLoop1.getResults(), ipType, opType, blockingFactor,
                  isVnni, vectorOpLhs, vectorOpRhs, contractOp, outerLoop,
                  innerLoop, ops, ivOuterLoop, packedBuffer, false, c0,
                  isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);

              scf::YieldOp::create(rewriterOuterLoop, locOuterLoop,
                                   newInnerLoop.getResults());
            });
      }
    }

    // Case 2b: Reduction loop depth is 1.
    if (loopLists.size() == 1) {
      innerLoop = loopLists[0];

      SmallVector<Value> loopItrArgs = createTileZeros(
          rewriter, innerLoop.getLoc(), opType, innerLoop, ops.size());

      if (isVnni) {
````
- **L1105 EN**: Executes a standalone statement or declaration: `isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);`.
  **L1105 CN**: 执行一条独立语句或声明：`isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);`。
- **L1106 EN**: Blank line separating nearby declarations or logic blocks.
  **L1106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1107 EN**: Continues logic associated with callable symbol `createLoops`.
  **L1107 CN**: 继续与可调用符号 `createLoops` 相关的逻辑。
- **L1108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, innerLoop.getLoc(), spillInnerLoop,`.
  **L1108 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, innerLoop.getLoc(), spillInnerLoop,`。
- **L1109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `innerLoop.getUpperBound(), innerLoop.getStep(),`.
  **L1109 CN**: 继续一个多行参数列表、初始化器或聚合项：`innerLoop.getUpperBound(), innerLoop.getStep(),`。
- **L1110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `newInnerLoop1.getResults(), ipType, opType, blockingFactor,`.
  **L1110 CN**: 继续一个多行参数列表、初始化器或聚合项：`newInnerLoop1.getResults(), ipType, opType, blockingFactor,`。
- **L1111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `isVnni, vectorOpLhs, vectorOpRhs, contractOp, outerLoop,`.
  **L1111 CN**: 继续一个多行参数列表、初始化器或聚合项：`isVnni, vectorOpLhs, vectorOpRhs, contractOp, outerLoop,`。
- **L1112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `innerLoop, ops, ivOuterLoop, packedBuffer, false, c0,`.
  **L1112 CN**: 继续一个多行参数列表、初始化器或聚合项：`innerLoop, ops, ivOuterLoop, packedBuffer, false, c0,`。
- **L1113 EN**: Executes a standalone statement or declaration: `isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);`.
  **L1113 CN**: 执行一条独立语句或声明：`isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);`。
- **L1114 EN**: Blank line separating nearby declarations or logic blocks.
  **L1114 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `scf::YieldOp::create(rewriterOuterLoop, locOuterLoop,`.
  **L1115 CN**: 继续一个多行参数列表、初始化器或聚合项：`scf::YieldOp::create(rewriterOuterLoop, locOuterLoop,`。
- **L1116 EN**: Executes a call or declaration centered on `newInnerLoop.getResults`.
  **L1116 CN**: 执行以 `newInnerLoop.getResults` 为核心的调用或声明。
- **L1117 EN**: Executes a standalone statement or declaration: `});`.
  **L1117 CN**: 执行一条独立语句或声明：`});`。
- **L1118 EN**: Closes the current lexical scope or compound statement.
  **L1118 CN**: 结束当前词法作用域或复合语句块。
- **L1119 EN**: Closes the current lexical scope or compound statement.
  **L1119 CN**: 结束当前词法作用域或复合语句块。
- **L1120 EN**: Blank line separating nearby declarations or logic blocks.
  **L1120 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1121 EN**: Comment explains nearby logic, invariants, or intent: `Case 2b: Reduction loop depth is 1.`.
  **L1121 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Case 2b: Reduction loop depth is 1.`。
- **L1122 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1122 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1123 EN**: Executes a standalone statement or declaration: `innerLoop = loopLists[0];`.
  **L1123 CN**: 执行一条独立语句或声明：`innerLoop = loopLists[0];`。
- **L1124 EN**: Blank line separating nearby declarations or logic blocks.
  **L1124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1125 EN**: Continues logic associated with callable symbol `createTileZeros`.
  **L1125 CN**: 继续与可调用符号 `createTileZeros` 相关的逻辑。
- **L1126 EN**: Executes a call or declaration centered on `innerLoop.getLoc`.
  **L1126 CN**: 执行以 `innerLoop.getLoc` 为核心的调用或声明。
- **L1127 EN**: Blank line separating nearby declarations or logic blocks.
  **L1127 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1128 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1128 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1129-1152

````cpp

        newLoop = createLoops(
            rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),
            innerLoop.getUpperBound(), innerLoop.getStep(), loopItrArgs, ipType,
            opType, blockingFactor, isVnni, vectorOpLhs, vectorOpRhs,
            contractOp, nullptr, innerLoop, ops, nullptr, nullptr, true,
            nullptr, false, false);

      } else {
        bool isInnerLoopUBLarger = false;
        bool isInnerLoopUBHasOddQuot = false;

        int64_t ubVal = 16 * blockingFactor;
        mlir::Value ub = innerLoop.getUpperBound();
        if (auto constOp = ub.getDefiningOp<mlir::arith::ConstantOp>()) {
          if (auto intAttr =
                  llvm::dyn_cast<mlir::IntegerAttr>(constOp.getValue())) {
            ubVal = intAttr.getInt();
          }
        }

        isInnerLoopUBLarger = ubVal > 16 * blockingFactor;
        isInnerLoopUBHasOddQuot =
            (((ubVal / (16 * blockingFactor)) % 2) == 1) && isInnerLoopUBLarger;
````
- **L1129 EN**: Blank line separating nearby declarations or logic blocks.
  **L1129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1130 EN**: Continues logic associated with callable symbol `createLoops`.
  **L1130 CN**: 继续与可调用符号 `createLoops` 相关的逻辑。
- **L1131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),`.
  **L1131 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),`。
- **L1132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `innerLoop.getUpperBound(), innerLoop.getStep(), loopItrArgs, ipType,`.
  **L1132 CN**: 继续一个多行参数列表、初始化器或聚合项：`innerLoop.getUpperBound(), innerLoop.getStep(), loopItrArgs, ipType,`。
- **L1133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `opType, blockingFactor, isVnni, vectorOpLhs, vectorOpRhs,`.
  **L1133 CN**: 继续一个多行参数列表、初始化器或聚合项：`opType, blockingFactor, isVnni, vectorOpLhs, vectorOpRhs,`。
- **L1134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `contractOp, nullptr, innerLoop, ops, nullptr, nullptr, true,`.
  **L1134 CN**: 继续一个多行参数列表、初始化器或聚合项：`contractOp, nullptr, innerLoop, ops, nullptr, nullptr, true,`。
- **L1135 EN**: Executes a standalone statement or declaration: `nullptr, false, false);`.
  **L1135 CN**: 执行一条独立语句或声明：`nullptr, false, false);`。
- **L1136 EN**: Blank line separating nearby declarations or logic blocks.
  **L1136 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1137 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L1137 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L1138 EN**: Initializes variable `isInnerLoopUBLarger` from the right-hand expression.
  **L1138 CN**: 使用右侧表达式初始化变量 `isInnerLoopUBLarger`。
- **L1139 EN**: Initializes variable `isInnerLoopUBHasOddQuot` from the right-hand expression.
  **L1139 CN**: 使用右侧表达式初始化变量 `isInnerLoopUBHasOddQuot`。
- **L1140 EN**: Blank line separating nearby declarations or logic blocks.
  **L1140 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1141 EN**: Initializes variable `ubVal` from the right-hand expression.
  **L1141 CN**: 使用右侧表达式初始化变量 `ubVal`。
- **L1142 EN**: Initializes variable `ub` from the right-hand expression.
  **L1142 CN**: 使用右侧表达式初始化变量 `ub`。
- **L1143 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1143 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1144 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1144 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1145 EN**: Starts a function, method, lambda, or structured scope: `llvm::dyn_cast<mlir::IntegerAttr>(constOp.getValue())) {`.
  **L1145 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::dyn_cast<mlir::IntegerAttr>(constOp.getValue())) {`。
- **L1146 EN**: Executes a call or declaration centered on `intAttr.getInt`.
  **L1146 CN**: 执行以 `intAttr.getInt` 为核心的调用或声明。
- **L1147 EN**: Closes the current lexical scope or compound statement.
  **L1147 CN**: 结束当前词法作用域或复合语句块。
- **L1148 EN**: Closes the current lexical scope or compound statement.
  **L1148 CN**: 结束当前词法作用域或复合语句块。
- **L1149 EN**: Blank line separating nearby declarations or logic blocks.
  **L1149 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1150 EN**: Executes a standalone statement or declaration: `isInnerLoopUBLarger = ubVal > 16 * blockingFactor;`.
  **L1150 CN**: 执行一条独立语句或声明：`isInnerLoopUBLarger = ubVal > 16 * blockingFactor;`。
- **L1151 EN**: Continues the surrounding expression or declaration: `isInnerLoopUBHasOddQuot =`.
  **L1151 CN**: 继续构造周围的表达式或声明：`isInnerLoopUBHasOddQuot =`。
- **L1152 EN**: Executes a call or declaration centered on `statement`.
  **L1152 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1153-1176

````cpp

        rewriter.setInsertionPoint(innerLoop);
        auto c0 =
            arith::ConstantIndexOp::create(rewriter, innerLoop.getLoc(), 0);
        auto spillLoopBound = arith::ConstantIndexOp::create(
            rewriter, innerLoop.getLoc(), 16 * blockingFactor);

        Value spillInnerLoop =
            arith::SubIOp::create(rewriter, innerLoop.getLoc(),
                                  innerLoop.getUpperBound(), spillLoopBound);

        auto bufferType =
            MemRefType::get({2, 32, (blockingFactor * 16)}, ipType);
        auto packedBuffer =
            memref::AllocaOp::create(rewriter, innerLoop.getLoc(), bufferType);

        // First Shuffling outside the reduction loops
        IRMapping rhsMapping;
        rhsMapping.map(
            vectorOpRhs->getOperand(
                getIndexPosition(contractOp.getRhs(), innerLoop) + 1),
            c0);
        auto rhsClone = rewriter.clone(*vectorOpRhs, rhsMapping);

````
- **L1153 EN**: Blank line separating nearby declarations or logic blocks.
  **L1153 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1154 EN**: Executes a call or declaration centered on `rewriter.setInsertionPoint`.
  **L1154 CN**: 执行以 `rewriter.setInsertionPoint` 为核心的调用或声明。
- **L1155 EN**: Continues the surrounding expression or declaration: `auto c0 =`.
  **L1155 CN**: 继续构造周围的表达式或声明：`auto c0 =`。
- **L1156 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L1156 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L1157 EN**: Continues logic associated with callable symbol `create`.
  **L1157 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1158 EN**: Executes a call or declaration centered on `innerLoop.getLoc`.
  **L1158 CN**: 执行以 `innerLoop.getLoc` 为核心的调用或声明。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1160 EN**: Continues the surrounding expression or declaration: `Value spillInnerLoop =`.
  **L1160 CN**: 继续构造周围的表达式或声明：`Value spillInnerLoop =`。
- **L1161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arith::SubIOp::create(rewriter, innerLoop.getLoc(),`.
  **L1161 CN**: 继续一个多行参数列表、初始化器或聚合项：`arith::SubIOp::create(rewriter, innerLoop.getLoc(),`。
- **L1162 EN**: Executes a call or declaration centered on `innerLoop.getUpperBound`.
  **L1162 CN**: 执行以 `innerLoop.getUpperBound` 为核心的调用或声明。
- **L1163 EN**: Blank line separating nearby declarations or logic blocks.
  **L1163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1164 EN**: Continues the surrounding expression or declaration: `auto bufferType =`.
  **L1164 CN**: 继续构造周围的表达式或声明：`auto bufferType =`。
- **L1165 EN**: Executes a call or declaration centered on `MemRefType::get`.
  **L1165 CN**: 执行以 `MemRefType::get` 为核心的调用或声明。
- **L1166 EN**: Continues the surrounding expression or declaration: `auto packedBuffer =`.
  **L1166 CN**: 继续构造周围的表达式或声明：`auto packedBuffer =`。
- **L1167 EN**: Executes a call or declaration centered on `memref::AllocaOp::create`.
  **L1167 CN**: 执行以 `memref::AllocaOp::create` 为核心的调用或声明。
- **L1168 EN**: Blank line separating nearby declarations or logic blocks.
  **L1168 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1169 EN**: Comment explains nearby logic, invariants, or intent: `First Shuffling outside the reduction loops`.
  **L1169 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`First Shuffling outside the reduction loops`。
- **L1170 EN**: Executes a standalone statement or declaration: `IRMapping rhsMapping;`.
  **L1170 CN**: 执行一条独立语句或声明：`IRMapping rhsMapping;`。
- **L1171 EN**: Continues logic associated with callable symbol `map`.
  **L1171 CN**: 继续与可调用符号 `map` 相关的逻辑。
- **L1172 EN**: Continues logic associated with callable symbol `getOperand`.
  **L1172 CN**: 继续与可调用符号 `getOperand` 相关的逻辑。
- **L1173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getIndexPosition(contractOp.getRhs(), innerLoop) + 1),`.
  **L1173 CN**: 继续一个多行参数列表、初始化器或聚合项：`getIndexPosition(contractOp.getRhs(), innerLoop) + 1),`。
- **L1174 EN**: Executes a standalone statement or declaration: `c0);`.
  **L1174 CN**: 执行一条独立语句或声明：`c0);`。
- **L1175 EN**: Initializes variable `rhsClone` from the right-hand expression.
  **L1175 CN**: 使用右侧表达式初始化变量 `rhsClone`。
- **L1176 EN**: Blank line separating nearby declarations or logic blocks.
  **L1176 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1177-1200

````cpp
        performShuffle(rewriter, innerLoop.getLoc(), rhsClone->getResult(0),
                       ipType, blockingFactor, packedBuffer, c0);

        auto newLoopNonSpill = createLoops(
            rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),
            spillInnerLoop, innerLoop.getStep(), loopItrArgs, ipType, opType,
            blockingFactor, isVnni, vectorOpLhs, vectorOpRhs, contractOp,
            nullptr, innerLoop, ops, nullptr, packedBuffer, true,
            spillLoopBound, isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);

        newLoop = createLoops(rewriter, innerLoop.getLoc(), spillInnerLoop,
                              innerLoop.getUpperBound(), innerLoop.getStep(),
                              newLoopNonSpill.getResults(), ipType, opType,
                              blockingFactor, isVnni, vectorOpLhs, vectorOpRhs,
                              contractOp, nullptr, innerLoop, ops, nullptr,
                              packedBuffer, false, c0, isInnerLoopUBLarger,
                              isInnerLoopUBHasOddQuot);
      }

      // This helps the final store back to the acc uses the same code for
      // the both reduction loop depth 1 or 2.
      outerLoop = innerLoop;
    }

````
- **L1177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `performShuffle(rewriter, innerLoop.getLoc(), rhsClone->getResult(0),`.
  **L1177 CN**: 继续一个多行参数列表、初始化器或聚合项：`performShuffle(rewriter, innerLoop.getLoc(), rhsClone->getResult(0),`。
- **L1178 EN**: Executes a standalone statement or declaration: `ipType, blockingFactor, packedBuffer, c0);`.
  **L1178 CN**: 执行一条独立语句或声明：`ipType, blockingFactor, packedBuffer, c0);`。
- **L1179 EN**: Blank line separating nearby declarations or logic blocks.
  **L1179 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1180 EN**: Continues logic associated with callable symbol `createLoops`.
  **L1180 CN**: 继续与可调用符号 `createLoops` 相关的逻辑。
- **L1181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),`.
  **L1181 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, innerLoop.getLoc(), innerLoop.getLowerBound(),`。
- **L1182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `spillInnerLoop, innerLoop.getStep(), loopItrArgs, ipType, opType,`.
  **L1182 CN**: 继续一个多行参数列表、初始化器或聚合项：`spillInnerLoop, innerLoop.getStep(), loopItrArgs, ipType, opType,`。
- **L1183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `blockingFactor, isVnni, vectorOpLhs, vectorOpRhs, contractOp,`.
  **L1183 CN**: 继续一个多行参数列表、初始化器或聚合项：`blockingFactor, isVnni, vectorOpLhs, vectorOpRhs, contractOp,`。
- **L1184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `nullptr, innerLoop, ops, nullptr, packedBuffer, true,`.
  **L1184 CN**: 继续一个多行参数列表、初始化器或聚合项：`nullptr, innerLoop, ops, nullptr, packedBuffer, true,`。
- **L1185 EN**: Executes a standalone statement or declaration: `spillLoopBound, isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);`.
  **L1185 CN**: 执行一条独立语句或声明：`spillLoopBound, isInnerLoopUBLarger, isInnerLoopUBHasOddQuot);`。
- **L1186 EN**: Blank line separating nearby declarations or logic blocks.
  **L1186 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `newLoop = createLoops(rewriter, innerLoop.getLoc(), spillInnerLoop,`.
  **L1187 CN**: 继续一个多行参数列表、初始化器或聚合项：`newLoop = createLoops(rewriter, innerLoop.getLoc(), spillInnerLoop,`。
- **L1188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `innerLoop.getUpperBound(), innerLoop.getStep(),`.
  **L1188 CN**: 继续一个多行参数列表、初始化器或聚合项：`innerLoop.getUpperBound(), innerLoop.getStep(),`。
- **L1189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `newLoopNonSpill.getResults(), ipType, opType,`.
  **L1189 CN**: 继续一个多行参数列表、初始化器或聚合项：`newLoopNonSpill.getResults(), ipType, opType,`。
- **L1190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `blockingFactor, isVnni, vectorOpLhs, vectorOpRhs,`.
  **L1190 CN**: 继续一个多行参数列表、初始化器或聚合项：`blockingFactor, isVnni, vectorOpLhs, vectorOpRhs,`。
- **L1191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `contractOp, nullptr, innerLoop, ops, nullptr,`.
  **L1191 CN**: 继续一个多行参数列表、初始化器或聚合项：`contractOp, nullptr, innerLoop, ops, nullptr,`。
- **L1192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `packedBuffer, false, c0, isInnerLoopUBLarger,`.
  **L1192 CN**: 继续一个多行参数列表、初始化器或聚合项：`packedBuffer, false, c0, isInnerLoopUBLarger,`。
- **L1193 EN**: Executes a standalone statement or declaration: `isInnerLoopUBHasOddQuot);`.
  **L1193 CN**: 执行一条独立语句或声明：`isInnerLoopUBHasOddQuot);`。
- **L1194 EN**: Closes the current lexical scope or compound statement.
  **L1194 CN**: 结束当前词法作用域或复合语句块。
- **L1195 EN**: Blank line separating nearby declarations or logic blocks.
  **L1195 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1196 EN**: Comment explains nearby logic, invariants, or intent: `This helps the final store back to the acc uses the same code for`.
  **L1196 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This helps the final store back to the acc uses the same code for`。
- **L1197 EN**: Comment explains nearby logic, invariants, or intent: `the both reduction loop depth 1 or 2.`.
  **L1197 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the both reduction loop depth 1 or 2.`。
- **L1198 EN**: Executes a standalone statement or declaration: `outerLoop = innerLoop;`.
  **L1198 CN**: 执行一条独立语句或声明：`outerLoop = innerLoop;`。
- **L1199 EN**: Closes the current lexical scope or compound statement.
  **L1199 CN**: 结束当前词法作用域或复合语句块。
- **L1200 EN**: Blank line separating nearby declarations or logic blocks.
  **L1200 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1201-1224

````cpp
    // Copy the amx tile accumulation results to a MemRef buffer, add the
    // initial accumulation value, and store back to the C-Matrix

    if (!isVnni) {
      Location loc = outerLoop.getLoc();
      Operation *accReadOp =
          traceToVectorReadLikeParentOperation(contractOp.getAcc());

      Value srcBuffAcc;
      SmallVector<Value> indicesAcc;

      llvm::TypeSwitch<Operation *>(accReadOp).Case<TransferReadOp, LoadOp>(
          [&](auto readOp) {
            srcBuffAcc = readOp.getOperand(0);

            auto indices = readOp.getIndices();
            indicesAcc.reserve(indices.size());

            llvm::transform(indices, std::back_inserter(indicesAcc),
                            [&](OpFoldResult ofr) {
                              return mlir::getValueOrCreateConstantIndexOp(
                                  rewriter, loc, ofr);
                            });
          });
````
- **L1201 EN**: Comment explains nearby logic, invariants, or intent: `Copy the amx tile accumulation results to a MemRef buffer, add the`.
  **L1201 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copy the amx tile accumulation results to a MemRef buffer, add the`。
- **L1202 EN**: Comment explains nearby logic, invariants, or intent: `initial accumulation value, and store back to the C-Matrix`.
  **L1202 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`initial accumulation value, and store back to the C-Matrix`。
- **L1203 EN**: Blank line separating nearby declarations or logic blocks.
  **L1203 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1204 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1204 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1205 EN**: Initializes variable `loc` from the right-hand expression.
  **L1205 CN**: 使用右侧表达式初始化变量 `loc`。
- **L1206 EN**: Continues the surrounding expression or declaration: `Operation *accReadOp =`.
  **L1206 CN**: 继续构造周围的表达式或声明：`Operation *accReadOp =`。
- **L1207 EN**: Executes a call or declaration centered on `traceToVectorReadLikeParentOperation`.
  **L1207 CN**: 执行以 `traceToVectorReadLikeParentOperation` 为核心的调用或声明。
- **L1208 EN**: Blank line separating nearby declarations or logic blocks.
  **L1208 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1209 EN**: Executes a standalone statement or declaration: `Value srcBuffAcc;`.
  **L1209 CN**: 执行一条独立语句或声明：`Value srcBuffAcc;`。
- **L1210 EN**: Executes a standalone statement or declaration: `SmallVector<Value> indicesAcc;`.
  **L1210 CN**: 执行一条独立语句或声明：`SmallVector<Value> indicesAcc;`。
- **L1211 EN**: Blank line separating nearby declarations or logic blocks.
  **L1211 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1212 EN**: Continues logic associated with callable symbol `LoadOp>`.
  **L1212 CN**: 继续与可调用符号 `LoadOp>` 相关的逻辑。
- **L1213 EN**: Starts a function, method, lambda, or structured scope: `[&](auto readOp) {`.
  **L1213 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](auto readOp) {`。
- **L1214 EN**: Executes a call or declaration centered on `readOp.getOperand`.
  **L1214 CN**: 执行以 `readOp.getOperand` 为核心的调用或声明。
- **L1215 EN**: Blank line separating nearby declarations or logic blocks.
  **L1215 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1216 EN**: Initializes variable `indices` from the right-hand expression.
  **L1216 CN**: 使用右侧表达式初始化变量 `indices`。
- **L1217 EN**: Executes a call or declaration centered on `indicesAcc.reserve`.
  **L1217 CN**: 执行以 `indicesAcc.reserve` 为核心的调用或声明。
- **L1218 EN**: Blank line separating nearby declarations or logic blocks.
  **L1218 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::transform(indices, std::back_inserter(indicesAcc),`.
  **L1219 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::transform(indices, std::back_inserter(indicesAcc),`。
- **L1220 EN**: Starts a function, method, lambda, or structured scope: `[&](OpFoldResult ofr) {`.
  **L1220 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](OpFoldResult ofr) {`。
- **L1221 EN**: Returns from the current function with `mlir::getValueOrCreateConstantIndexOp(`.
  **L1221 CN**: 以 `mlir::getValueOrCreateConstantIndexOp(` 从当前函数返回。
- **L1222 EN**: Executes a standalone statement or declaration: `rewriter, loc, ofr);`.
  **L1222 CN**: 执行一条独立语句或声明：`rewriter, loc, ofr);`。
- **L1223 EN**: Executes a standalone statement or declaration: `});`.
  **L1223 CN**: 执行一条独立语句或声明：`});`。
- **L1224 EN**: Executes a standalone statement or declaration: `});`.
  **L1224 CN**: 执行一条独立语句或声明：`});`。

### Lines 1225-1248

````cpp

      auto outputShapes =
          mlir::cast<mlir::MemRefType>(srcBuffAcc.getType()).getShape();
      unsigned int M = outputShapes[outputShapes.size() - 2];
      unsigned int N = outputShapes[outputShapes.size() - 1];

      SmallVector<Value> dps = newLoop.getResults();
      auto bufferType = MemRefType::get({M, N}, opType);
      auto resultBuffer = memref::AllocaOp::create(rewriter, loc, bufferType);

      // Store the amx tiled-dot product output into an MxN memref.
      for (unsigned int i = 0, k = 0; i < M; i = i + 16) {
        for (unsigned int j = 0; j < N; j = j + 16) {
          Value indexOp_i = arith::ConstantIndexOp::create(rewriter, loc, i);
          Value indexOp_j = arith::ConstantIndexOp::create(rewriter, loc, j);
          amx::TileStoreOp::create(rewriter, loc, resultBuffer,
                                   ValueRange{indexOp_i, indexOp_j}, dps[k]);
          k++;
        }
      }
      auto c0 = arith::ConstantIndexOp::create(rewriter, loc, 0);
      auto c16 = arith::ConstantIndexOp::create(rewriter, loc, 16);
      auto one = arith::ConstantIndexOp::create(rewriter, loc, 1);
      auto mBound = arith::ConstantIndexOp::create(rewriter, loc, N);
````
- **L1225 EN**: Blank line separating nearby declarations or logic blocks.
  **L1225 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1226 EN**: Continues the surrounding expression or declaration: `auto outputShapes =`.
  **L1226 CN**: 继续构造周围的表达式或声明：`auto outputShapes =`。
- **L1227 EN**: Executes a call or declaration centered on `mlir::cast<mlir::MemRefType>`.
  **L1227 CN**: 执行以 `mlir::cast<mlir::MemRefType>` 为核心的调用或声明。
- **L1228 EN**: Initializes variable `M` from the right-hand expression.
  **L1228 CN**: 使用右侧表达式初始化变量 `M`。
- **L1229 EN**: Initializes variable `N` from the right-hand expression.
  **L1229 CN**: 使用右侧表达式初始化变量 `N`。
- **L1230 EN**: Blank line separating nearby declarations or logic blocks.
  **L1230 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1231 EN**: Initializes variable `dps` from the right-hand expression.
  **L1231 CN**: 使用右侧表达式初始化变量 `dps`。
- **L1232 EN**: Initializes variable `bufferType` from the right-hand expression.
  **L1232 CN**: 使用右侧表达式初始化变量 `bufferType`。
- **L1233 EN**: Initializes variable `resultBuffer` from the right-hand expression.
  **L1233 CN**: 使用右侧表达式初始化变量 `resultBuffer`。
- **L1234 EN**: Blank line separating nearby declarations or logic blocks.
  **L1234 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1235 EN**: Comment explains nearby logic, invariants, or intent: `Store the amx tiled-dot product output into an MxN memref.`.
  **L1235 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Store the amx tiled-dot product output into an MxN memref.`。
- **L1236 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1236 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1237 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1237 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1238 EN**: Initializes variable `indexOp_i` from the right-hand expression.
  **L1238 CN**: 使用右侧表达式初始化变量 `indexOp_i`。
- **L1239 EN**: Initializes variable `indexOp_j` from the right-hand expression.
  **L1239 CN**: 使用右侧表达式初始化变量 `indexOp_j`。
- **L1240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `amx::TileStoreOp::create(rewriter, loc, resultBuffer,`.
  **L1240 CN**: 继续一个多行参数列表、初始化器或聚合项：`amx::TileStoreOp::create(rewriter, loc, resultBuffer,`。
- **L1241 EN**: Executes a standalone statement or declaration: `ValueRange{indexOp_i, indexOp_j}, dps[k]);`.
  **L1241 CN**: 执行一条独立语句或声明：`ValueRange{indexOp_i, indexOp_j}, dps[k]);`。
- **L1242 EN**: Executes a standalone statement or declaration: `k++;`.
  **L1242 CN**: 执行一条独立语句或声明：`k++;`。
- **L1243 EN**: Closes the current lexical scope or compound statement.
  **L1243 CN**: 结束当前词法作用域或复合语句块。
- **L1244 EN**: Closes the current lexical scope or compound statement.
  **L1244 CN**: 结束当前词法作用域或复合语句块。
- **L1245 EN**: Initializes variable `c0` from the right-hand expression.
  **L1245 CN**: 使用右侧表达式初始化变量 `c0`。
- **L1246 EN**: Initializes variable `c16` from the right-hand expression.
  **L1246 CN**: 使用右侧表达式初始化变量 `c16`。
- **L1247 EN**: Initializes variable `one` from the right-hand expression.
  **L1247 CN**: 使用右侧表达式初始化变量 `one`。
- **L1248 EN**: Initializes variable `mBound` from the right-hand expression.
  **L1248 CN**: 使用右侧表达式初始化变量 `mBound`。

### Lines 1249-1272

````cpp

      // Create a loop that iterates over the MxN memerf, retrives two rows +
      // shuffle them, add up the C element values and stores them back.
      scf::ForOp::create(
          rewriter, loc, c0, mBound, one, ValueRange{},
          [&](OpBuilder &nestedBuilder, Location loc, Value iv,
              ValueRange iterArgs) {
            auto row = vector::LoadOp::create(rewriter, loc,
                                              VectorType::get(16, opType),
                                              resultBuffer, ValueRange{iv, c0});

            auto row2 = vector::LoadOp::create(
                rewriter, loc, VectorType::get(16, opType), resultBuffer,
                ValueRange{iv, c16});

            auto shuffle1 = vector::ShuffleOp::create(
                rewriter, loc, VectorType::get(16, opType), row, row2,
                ArrayRef<int64_t>{0, 1, 2, 3, 16, 17, 18, 19, 4, 5, 6, 7, 20,
                                  21, 22, 23});

            auto shuffle2 = vector::ShuffleOp::create(
                rewriter, loc, VectorType::get(16, opType), row, row2,
                ArrayRef<int64_t>{8, 9, 10, 11, 24, 25, 26, 27, 12, 13, 14, 15,
                                  28, 29, 30, 31});
````
- **L1249 EN**: Blank line separating nearby declarations or logic blocks.
  **L1249 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1250 EN**: Comment explains nearby logic, invariants, or intent: `Create a loop that iterates over the MxN memerf, retrives two rows +`.
  **L1250 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create a loop that iterates over the MxN memerf, retrives two rows +`。
- **L1251 EN**: Comment explains nearby logic, invariants, or intent: `shuffle them, add up the C element values and stores them back.`.
  **L1251 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`shuffle them, add up the C element values and stores them back.`。
- **L1252 EN**: Continues logic associated with callable symbol `create`.
  **L1252 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, c0, mBound, one, ValueRange{},`.
  **L1253 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, c0, mBound, one, ValueRange{},`。
- **L1254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](OpBuilder &nestedBuilder, Location loc, Value iv,`.
  **L1254 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](OpBuilder &nestedBuilder, Location loc, Value iv,`。
- **L1255 EN**: Continues the surrounding expression or declaration: `ValueRange iterArgs) {`.
  **L1255 CN**: 继续构造周围的表达式或声明：`ValueRange iterArgs) {`。
- **L1256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto row = vector::LoadOp::create(rewriter, loc,`.
  **L1256 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto row = vector::LoadOp::create(rewriter, loc,`。
- **L1257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VectorType::get(16, opType),`.
  **L1257 CN**: 继续一个多行参数列表、初始化器或聚合项：`VectorType::get(16, opType),`。
- **L1258 EN**: Executes a standalone statement or declaration: `resultBuffer, ValueRange{iv, c0});`.
  **L1258 CN**: 执行一条独立语句或声明：`resultBuffer, ValueRange{iv, c0});`。
- **L1259 EN**: Blank line separating nearby declarations or logic blocks.
  **L1259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1260 EN**: Continues logic associated with callable symbol `create`.
  **L1260 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get(16, opType), resultBuffer,`.
  **L1261 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get(16, opType), resultBuffer,`。
- **L1262 EN**: Executes a standalone statement or declaration: `ValueRange{iv, c16});`.
  **L1262 CN**: 执行一条独立语句或声明：`ValueRange{iv, c16});`。
- **L1263 EN**: Blank line separating nearby declarations or logic blocks.
  **L1263 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1264 EN**: Continues logic associated with callable symbol `create`.
  **L1264 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get(16, opType), row, row2,`.
  **L1265 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get(16, opType), row, row2,`。
- **L1266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t>{0, 1, 2, 3, 16, 17, 18, 19, 4, 5, 6, 7, 20,`.
  **L1266 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t>{0, 1, 2, 3, 16, 17, 18, 19, 4, 5, 6, 7, 20,`。
- **L1267 EN**: Executes a standalone statement or declaration: `21, 22, 23});`.
  **L1267 CN**: 执行一条独立语句或声明：`21, 22, 23});`。
- **L1268 EN**: Blank line separating nearby declarations or logic blocks.
  **L1268 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1269 EN**: Continues logic associated with callable symbol `create`.
  **L1269 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get(16, opType), row, row2,`.
  **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get(16, opType), row, row2,`。
- **L1271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t>{8, 9, 10, 11, 24, 25, 26, 27, 12, 13, 14, 15,`.
  **L1271 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t>{8, 9, 10, 11, 24, 25, 26, 27, 12, 13, 14, 15,`。
- **L1272 EN**: Executes a standalone statement or declaration: `28, 29, 30, 31});`.
  **L1272 CN**: 执行一条独立语句或声明：`28, 29, 30, 31});`。

### Lines 1273-1296

````cpp

            indicesAcc[indicesAcc.size() - 2] = iv;
            indicesAcc[indicesAcc.size() - 1] = c0;

            Value valueCRow1 = vector::LoadOp::create(
                rewriter, loc, VectorType::get(16, opType), srcBuffAcc,
                indicesAcc);
            indicesAcc[indicesAcc.size() - 1] = c16;

            Value valueCRow2 = vector::LoadOp::create(
                rewriter, loc, VectorType::get(16, opType), srcBuffAcc,
                indicesAcc);

            Value addOp;
            Value addOp2;

            if (ipType.isBF16()) {
              addOp =
                  arith::AddFOp::create(rewriter, loc, shuffle1, valueCRow1);

              addOp2 =
                  arith::AddFOp::create(rewriter, loc, shuffle2, valueCRow2);
            }

````
- **L1273 EN**: Blank line separating nearby declarations or logic blocks.
  **L1273 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1274 EN**: Executes a call or declaration centered on `indicesAcc[indicesAcc.size`.
  **L1274 CN**: 执行以 `indicesAcc[indicesAcc.size` 为核心的调用或声明。
- **L1275 EN**: Executes a call or declaration centered on `indicesAcc[indicesAcc.size`.
  **L1275 CN**: 执行以 `indicesAcc[indicesAcc.size` 为核心的调用或声明。
- **L1276 EN**: Blank line separating nearby declarations or logic blocks.
  **L1276 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1277 EN**: Continues logic associated with callable symbol `create`.
  **L1277 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get(16, opType), srcBuffAcc,`.
  **L1278 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get(16, opType), srcBuffAcc,`。
- **L1279 EN**: Executes a standalone statement or declaration: `indicesAcc);`.
  **L1279 CN**: 执行一条独立语句或声明：`indicesAcc);`。
- **L1280 EN**: Executes a call or declaration centered on `indicesAcc[indicesAcc.size`.
  **L1280 CN**: 执行以 `indicesAcc[indicesAcc.size` 为核心的调用或声明。
- **L1281 EN**: Blank line separating nearby declarations or logic blocks.
  **L1281 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1282 EN**: Continues logic associated with callable symbol `create`.
  **L1282 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get(16, opType), srcBuffAcc,`.
  **L1283 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get(16, opType), srcBuffAcc,`。
- **L1284 EN**: Executes a standalone statement or declaration: `indicesAcc);`.
  **L1284 CN**: 执行一条独立语句或声明：`indicesAcc);`。
- **L1285 EN**: Blank line separating nearby declarations or logic blocks.
  **L1285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1286 EN**: Executes a standalone statement or declaration: `Value addOp;`.
  **L1286 CN**: 执行一条独立语句或声明：`Value addOp;`。
- **L1287 EN**: Executes a standalone statement or declaration: `Value addOp2;`.
  **L1287 CN**: 执行一条独立语句或声明：`Value addOp2;`。
- **L1288 EN**: Blank line separating nearby declarations or logic blocks.
  **L1288 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1289 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1289 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1290 EN**: Continues the surrounding expression or declaration: `addOp =`.
  **L1290 CN**: 继续构造周围的表达式或声明：`addOp =`。
- **L1291 EN**: Executes a call or declaration centered on `arith::AddFOp::create`.
  **L1291 CN**: 执行以 `arith::AddFOp::create` 为核心的调用或声明。
- **L1292 EN**: Blank line separating nearby declarations or logic blocks.
  **L1292 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1293 EN**: Continues the surrounding expression or declaration: `addOp2 =`.
  **L1293 CN**: 继续构造周围的表达式或声明：`addOp2 =`。
- **L1294 EN**: Executes a call or declaration centered on `arith::AddFOp::create`.
  **L1294 CN**: 执行以 `arith::AddFOp::create` 为核心的调用或声明。
- **L1295 EN**: Closes the current lexical scope or compound statement.
  **L1295 CN**: 结束当前词法作用域或复合语句块。
- **L1296 EN**: Blank line separating nearby declarations or logic blocks.
  **L1296 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1297-1320

````cpp
            if (ipType.isSignlessInteger(8)) {
              addOp =
                  arith::AddIOp::create(rewriter, loc, shuffle1, valueCRow1);

              addOp2 =
                  arith::AddIOp::create(rewriter, loc, shuffle2, valueCRow2);
            }
            indicesAcc[indicesAcc.size() - 1] = c0;
            vector::StoreOp::create(rewriter, loc, addOp, srcBuffAcc,
                                    indicesAcc);
            indicesAcc[indicesAcc.size() - 1] = c16;
            vector::StoreOp::create(rewriter, loc, addOp2, srcBuffAcc,
                                    indicesAcc);

            scf::YieldOp::create(nestedBuilder, loc);
          });
    }

    auto bufferType = MemRefType::get({16, 16}, opType);
    auto resultBuffer =
        memref::AllocaOp::create(rewriter, outerLoop.getLoc(), bufferType);
    SmallVector<Value> dps = newLoop.getResults();

    for (size_t i = 0; i < ops.size(); i++) {
````
- **L1297 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1297 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1298 EN**: Continues the surrounding expression or declaration: `addOp =`.
  **L1298 CN**: 继续构造周围的表达式或声明：`addOp =`。
- **L1299 EN**: Executes a call or declaration centered on `arith::AddIOp::create`.
  **L1299 CN**: 执行以 `arith::AddIOp::create` 为核心的调用或声明。
- **L1300 EN**: Blank line separating nearby declarations or logic blocks.
  **L1300 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1301 EN**: Continues the surrounding expression or declaration: `addOp2 =`.
  **L1301 CN**: 继续构造周围的表达式或声明：`addOp2 =`。
- **L1302 EN**: Executes a call or declaration centered on `arith::AddIOp::create`.
  **L1302 CN**: 执行以 `arith::AddIOp::create` 为核心的调用或声明。
- **L1303 EN**: Closes the current lexical scope or compound statement.
  **L1303 CN**: 结束当前词法作用域或复合语句块。
- **L1304 EN**: Executes a call or declaration centered on `indicesAcc[indicesAcc.size`.
  **L1304 CN**: 执行以 `indicesAcc[indicesAcc.size` 为核心的调用或声明。
- **L1305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::StoreOp::create(rewriter, loc, addOp, srcBuffAcc,`.
  **L1305 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::StoreOp::create(rewriter, loc, addOp, srcBuffAcc,`。
- **L1306 EN**: Executes a standalone statement or declaration: `indicesAcc);`.
  **L1306 CN**: 执行一条独立语句或声明：`indicesAcc);`。
- **L1307 EN**: Executes a call or declaration centered on `indicesAcc[indicesAcc.size`.
  **L1307 CN**: 执行以 `indicesAcc[indicesAcc.size` 为核心的调用或声明。
- **L1308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::StoreOp::create(rewriter, loc, addOp2, srcBuffAcc,`.
  **L1308 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::StoreOp::create(rewriter, loc, addOp2, srcBuffAcc,`。
- **L1309 EN**: Executes a standalone statement or declaration: `indicesAcc);`.
  **L1309 CN**: 执行一条独立语句或声明：`indicesAcc);`。
- **L1310 EN**: Blank line separating nearby declarations or logic blocks.
  **L1310 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1311 EN**: Executes a call or declaration centered on `scf::YieldOp::create`.
  **L1311 CN**: 执行以 `scf::YieldOp::create` 为核心的调用或声明。
- **L1312 EN**: Executes a standalone statement or declaration: `});`.
  **L1312 CN**: 执行一条独立语句或声明：`});`。
- **L1313 EN**: Closes the current lexical scope or compound statement.
  **L1313 CN**: 结束当前词法作用域或复合语句块。
- **L1314 EN**: Blank line separating nearby declarations or logic blocks.
  **L1314 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1315 EN**: Initializes variable `bufferType` from the right-hand expression.
  **L1315 CN**: 使用右侧表达式初始化变量 `bufferType`。
- **L1316 EN**: Continues the surrounding expression or declaration: `auto resultBuffer =`.
  **L1316 CN**: 继续构造周围的表达式或声明：`auto resultBuffer =`。
- **L1317 EN**: Executes a call or declaration centered on `memref::AllocaOp::create`.
  **L1317 CN**: 执行以 `memref::AllocaOp::create` 为核心的调用或声明。
- **L1318 EN**: Initializes variable `dps` from the right-hand expression.
  **L1318 CN**: 使用右侧表达式初始化变量 `dps`。
- **L1319 EN**: Blank line separating nearby declarations or logic blocks.
  **L1319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1320 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1320 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 1321-1344

````cpp
      vector::ContractionOp contOp = ops[i];
      Operation *resultWriteOp =
          traceToVectorWriteLikeUserOperation(contOp.getResult());
      if (isVnni) {
        rewriter.setInsertionPoint(resultWriteOp);

        Value indexOp_0 =
            arith::ConstantIndexOp::create(rewriter, outerLoop.getLoc(), 0);

        amx::TileStoreOp::create(rewriter, outerLoop.getLoc(), resultBuffer,
                                 ValueRange{indexOp_0, indexOp_0}, dps[i]);

        auto c0 =
            arith::ConstantIndexOp::create(rewriter, outerLoop.getLoc(), 0);
        auto one =
            arith::ConstantIndexOp::create(rewriter, outerLoop.getLoc(), 1);
        auto mBound =
            arith::ConstantIndexOp::create(rewriter, outerLoop.getLoc(), 16);

        scf::ForOp::create(
            rewriter, outerLoop.getLoc(), c0, mBound, one, ValueRange{},
            [&](OpBuilder &builder, Location loc, Value iv,
                ValueRange iterArgs) {
              auto resultAcc = vector::LoadOp::create(
````
- **L1321 EN**: Initializes variable `contOp` from the right-hand expression.
  **L1321 CN**: 使用右侧表达式初始化变量 `contOp`。
- **L1322 EN**: Continues the surrounding expression or declaration: `Operation *resultWriteOp =`.
  **L1322 CN**: 继续构造周围的表达式或声明：`Operation *resultWriteOp =`。
- **L1323 EN**: Executes a call or declaration centered on `traceToVectorWriteLikeUserOperation`.
  **L1323 CN**: 执行以 `traceToVectorWriteLikeUserOperation` 为核心的调用或声明。
- **L1324 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1324 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1325 EN**: Executes a call or declaration centered on `rewriter.setInsertionPoint`.
  **L1325 CN**: 执行以 `rewriter.setInsertionPoint` 为核心的调用或声明。
- **L1326 EN**: Blank line separating nearby declarations or logic blocks.
  **L1326 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1327 EN**: Continues the surrounding expression or declaration: `Value indexOp_0 =`.
  **L1327 CN**: 继续构造周围的表达式或声明：`Value indexOp_0 =`。
- **L1328 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L1328 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L1329 EN**: Blank line separating nearby declarations or logic blocks.
  **L1329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `amx::TileStoreOp::create(rewriter, outerLoop.getLoc(), resultBuffer,`.
  **L1330 CN**: 继续一个多行参数列表、初始化器或聚合项：`amx::TileStoreOp::create(rewriter, outerLoop.getLoc(), resultBuffer,`。
- **L1331 EN**: Executes a standalone statement or declaration: `ValueRange{indexOp_0, indexOp_0}, dps[i]);`.
  **L1331 CN**: 执行一条独立语句或声明：`ValueRange{indexOp_0, indexOp_0}, dps[i]);`。
- **L1332 EN**: Blank line separating nearby declarations or logic blocks.
  **L1332 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1333 EN**: Continues the surrounding expression or declaration: `auto c0 =`.
  **L1333 CN**: 继续构造周围的表达式或声明：`auto c0 =`。
- **L1334 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L1334 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L1335 EN**: Continues the surrounding expression or declaration: `auto one =`.
  **L1335 CN**: 继续构造周围的表达式或声明：`auto one =`。
- **L1336 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L1336 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L1337 EN**: Continues the surrounding expression or declaration: `auto mBound =`.
  **L1337 CN**: 继续构造周围的表达式或声明：`auto mBound =`。
- **L1338 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L1338 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L1339 EN**: Blank line separating nearby declarations or logic blocks.
  **L1339 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1340 EN**: Continues logic associated with callable symbol `create`.
  **L1340 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, outerLoop.getLoc(), c0, mBound, one, ValueRange{},`.
  **L1341 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, outerLoop.getLoc(), c0, mBound, one, ValueRange{},`。
- **L1342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](OpBuilder &builder, Location loc, Value iv,`.
  **L1342 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](OpBuilder &builder, Location loc, Value iv,`。
- **L1343 EN**: Continues the surrounding expression or declaration: `ValueRange iterArgs) {`.
  **L1343 CN**: 继续构造周围的表达式或声明：`ValueRange iterArgs) {`。
- **L1344 EN**: Continues logic associated with callable symbol `create`.
  **L1344 CN**: 继续与可调用符号 `create` 相关的逻辑。

### Lines 1345-1368

````cpp
                  rewriter, loc, VectorType::get(16, opType), resultBuffer,
                  ValueRange{iv, c0});

              Operation *accReadOp =
                  traceToVectorReadLikeParentOperation(ops[i].getAcc());

              Value srcBuffAcc;
              SmallVector<Value> indicesAcc;

              llvm::TypeSwitch<Operation *>(accReadOp)
                  .Case<TransferReadOp, LoadOp>([&](auto readOp) {
                    srcBuffAcc = readOp.getOperand(0);

                    auto indices = readOp.getIndices();
                    indicesAcc.reserve(indices.size());

                    llvm::transform(
                        indices, std::back_inserter(indicesAcc),
                        [&](OpFoldResult ofr) {
                          return mlir::getValueOrCreateConstantIndexOp(
                              rewriter, loc, ofr);
                        });
                  });

````
- **L1345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, VectorType::get(16, opType), resultBuffer,`.
  **L1345 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, VectorType::get(16, opType), resultBuffer,`。
- **L1346 EN**: Executes a standalone statement or declaration: `ValueRange{iv, c0});`.
  **L1346 CN**: 执行一条独立语句或声明：`ValueRange{iv, c0});`。
- **L1347 EN**: Blank line separating nearby declarations or logic blocks.
  **L1347 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1348 EN**: Continues the surrounding expression or declaration: `Operation *accReadOp =`.
  **L1348 CN**: 继续构造周围的表达式或声明：`Operation *accReadOp =`。
- **L1349 EN**: Executes a call or declaration centered on `traceToVectorReadLikeParentOperation`.
  **L1349 CN**: 执行以 `traceToVectorReadLikeParentOperation` 为核心的调用或声明。
- **L1350 EN**: Blank line separating nearby declarations or logic blocks.
  **L1350 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1351 EN**: Executes a standalone statement or declaration: `Value srcBuffAcc;`.
  **L1351 CN**: 执行一条独立语句或声明：`Value srcBuffAcc;`。
- **L1352 EN**: Executes a standalone statement or declaration: `SmallVector<Value> indicesAcc;`.
  **L1352 CN**: 执行一条独立语句或声明：`SmallVector<Value> indicesAcc;`。
- **L1353 EN**: Blank line separating nearby declarations or logic blocks.
  **L1353 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1354 EN**: Continues the surrounding expression or declaration: `llvm::TypeSwitch<Operation *>(accReadOp)`.
  **L1354 CN**: 继续构造周围的表达式或声明：`llvm::TypeSwitch<Operation *>(accReadOp)`。
- **L1355 EN**: Starts a function, method, lambda, or structured scope: `.Case<TransferReadOp, LoadOp>([&](auto readOp) {`.
  **L1355 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.Case<TransferReadOp, LoadOp>([&](auto readOp) {`。
- **L1356 EN**: Executes a call or declaration centered on `readOp.getOperand`.
  **L1356 CN**: 执行以 `readOp.getOperand` 为核心的调用或声明。
- **L1357 EN**: Blank line separating nearby declarations or logic blocks.
  **L1357 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1358 EN**: Initializes variable `indices` from the right-hand expression.
  **L1358 CN**: 使用右侧表达式初始化变量 `indices`。
- **L1359 EN**: Executes a call or declaration centered on `indicesAcc.reserve`.
  **L1359 CN**: 执行以 `indicesAcc.reserve` 为核心的调用或声明。
- **L1360 EN**: Blank line separating nearby declarations or logic blocks.
  **L1360 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1361 EN**: Continues logic associated with callable symbol `transform`.
  **L1361 CN**: 继续与可调用符号 `transform` 相关的逻辑。
- **L1362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `indices, std::back_inserter(indicesAcc),`.
  **L1362 CN**: 继续一个多行参数列表、初始化器或聚合项：`indices, std::back_inserter(indicesAcc),`。
- **L1363 EN**: Starts a function, method, lambda, or structured scope: `[&](OpFoldResult ofr) {`.
  **L1363 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](OpFoldResult ofr) {`。
- **L1364 EN**: Returns from the current function with `mlir::getValueOrCreateConstantIndexOp(`.
  **L1364 CN**: 以 `mlir::getValueOrCreateConstantIndexOp(` 从当前函数返回。
- **L1365 EN**: Executes a standalone statement or declaration: `rewriter, loc, ofr);`.
  **L1365 CN**: 执行一条独立语句或声明：`rewriter, loc, ofr);`。
- **L1366 EN**: Executes a standalone statement or declaration: `});`.
  **L1366 CN**: 执行一条独立语句或声明：`});`。
- **L1367 EN**: Executes a standalone statement or declaration: `});`.
  **L1367 CN**: 执行一条独立语句或声明：`});`。
- **L1368 EN**: Blank line separating nearby declarations or logic blocks.
  **L1368 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1369-1392

````cpp
              Value sum =
                  arith::AddIOp::create(builder, loc, iv, indicesAcc[0]);
              indicesAcc[indicesAcc.size() - 2] = sum;

              auto acc = vector::LoadOp::create(rewriter, loc,
                                                VectorType::get(16, opType),
                                                srcBuffAcc, indicesAcc);
              Value addition;
              if (ipType.isBF16())
                addition = arith::AddFOp::create(rewriter, loc, resultAcc, acc);

              if (ipType.isSignlessInteger(8))
                addition = arith::AddIOp::create(rewriter, loc, resultAcc, acc);

              vector::StoreOp::create(builder, loc, addition, srcBuffAcc,
                                      indicesAcc);

              scf::YieldOp::create(builder, outerLoop.getLoc());
            });
      }

      rewriter.eraseOp(resultWriteOp);
    }

````
- **L1369 EN**: Continues the surrounding expression or declaration: `Value sum =`.
  **L1369 CN**: 继续构造周围的表达式或声明：`Value sum =`。
- **L1370 EN**: Executes a call or declaration centered on `arith::AddIOp::create`.
  **L1370 CN**: 执行以 `arith::AddIOp::create` 为核心的调用或声明。
- **L1371 EN**: Executes a call or declaration centered on `indicesAcc[indicesAcc.size`.
  **L1371 CN**: 执行以 `indicesAcc[indicesAcc.size` 为核心的调用或声明。
- **L1372 EN**: Blank line separating nearby declarations or logic blocks.
  **L1372 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto acc = vector::LoadOp::create(rewriter, loc,`.
  **L1373 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto acc = vector::LoadOp::create(rewriter, loc,`。
- **L1374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VectorType::get(16, opType),`.
  **L1374 CN**: 继续一个多行参数列表、初始化器或聚合项：`VectorType::get(16, opType),`。
- **L1375 EN**: Executes a standalone statement or declaration: `srcBuffAcc, indicesAcc);`.
  **L1375 CN**: 执行一条独立语句或声明：`srcBuffAcc, indicesAcc);`。
- **L1376 EN**: Executes a standalone statement or declaration: `Value addition;`.
  **L1376 CN**: 执行一条独立语句或声明：`Value addition;`。
- **L1377 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1377 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1378 EN**: Executes a call or declaration centered on `arith::AddFOp::create`.
  **L1378 CN**: 执行以 `arith::AddFOp::create` 为核心的调用或声明。
- **L1379 EN**: Blank line separating nearby declarations or logic blocks.
  **L1379 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1380 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1380 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1381 EN**: Executes a call or declaration centered on `arith::AddIOp::create`.
  **L1381 CN**: 执行以 `arith::AddIOp::create` 为核心的调用或声明。
- **L1382 EN**: Blank line separating nearby declarations or logic blocks.
  **L1382 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::StoreOp::create(builder, loc, addition, srcBuffAcc,`.
  **L1383 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::StoreOp::create(builder, loc, addition, srcBuffAcc,`。
- **L1384 EN**: Executes a standalone statement or declaration: `indicesAcc);`.
  **L1384 CN**: 执行一条独立语句或声明：`indicesAcc);`。
- **L1385 EN**: Blank line separating nearby declarations or logic blocks.
  **L1385 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1386 EN**: Executes a call or declaration centered on `scf::YieldOp::create`.
  **L1386 CN**: 执行以 `scf::YieldOp::create` 为核心的调用或声明。
- **L1387 EN**: Executes a standalone statement or declaration: `});`.
  **L1387 CN**: 执行一条独立语句或声明：`});`。
- **L1388 EN**: Closes the current lexical scope or compound statement.
  **L1388 CN**: 结束当前词法作用域或复合语句块。
- **L1389 EN**: Blank line separating nearby declarations or logic blocks.
  **L1389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1390 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L1390 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L1391 EN**: Closes the current lexical scope or compound statement.
  **L1391 CN**: 结束当前词法作用域或复合语句块。
- **L1392 EN**: Blank line separating nearby declarations or logic blocks.
  **L1392 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1393-1402

````cpp
    return success();
  }
};

} // namespace

void x86::populateVectorContractToAMXDotProductPatterns(
    RewritePatternSet &patterns) {
  patterns.add<VectorContractToAMXDotProduct>(patterns.getContext());
}
````
- **L1393 EN**: Returns from the current function with `success()`.
  **L1393 CN**: 以 `success()` 从当前函数返回。
- **L1394 EN**: Closes the current lexical scope or compound statement.
  **L1394 CN**: 结束当前词法作用域或复合语句块。
- **L1395 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1395 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1396 EN**: Blank line separating nearby declarations or logic blocks.
  **L1396 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1397 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L1397 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L1398 EN**: Blank line separating nearby declarations or logic blocks.
  **L1398 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1399 EN**: Continues logic associated with callable symbol `populateVectorContractToAMXDotProductPatterns`.
  **L1399 CN**: 继续与可调用符号 `populateVectorContractToAMXDotProductPatterns` 相关的逻辑。
- **L1400 EN**: Continues the surrounding expression or declaration: `RewritePatternSet &patterns) {`.
  **L1400 CN**: 继续构造周围的表达式或声明：`RewritePatternSet &patterns) {`。
- **L1401 EN**: Executes a call or declaration centered on `patterns.add<VectorContractToAMXDotProduct>`.
  **L1401 CN**: 执行以 `patterns.add<VectorContractToAMXDotProduct>` 为核心的调用或声明。
- **L1402 EN**: Closes the current lexical scope or compound statement.
  **L1402 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Rewrite-driven lowering / 基于重写的 lowering**
- **Dialect definition and registration / 方言定义与注册**
- **Operation semantics and verification / 操作语义与验证**
- **Pattern-based rewriting / 基于模式的重写**
- **Pattern collection management / 重写模式集合管理**
- **Failure-aware result propagation / 带失败语义的结果传播**
- **Optional success payloads / 带成功载荷的可失败结果**
- **IR mutation through rewriter helpers / 通过重写器辅助工具进行 IR 变换**
- **Greedy canonicalization driver / 贪心规范化驱动**
- **Attribute representation / 属性表示**

## Dependencies / 依赖关系

- `mlir/Dialect/SCF/IR/SCF.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Vector/IR/VectorOps.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Vector/Utils/VectorUtils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/X86/Transforms.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/X86/Utils/X86Utils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/X86/X86Dialect.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/IR/BuiltinAttributes.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `mlir/IR/Dominance.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `mlir/IR/PatternMatch.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `llvm/Support/Casting.h`: Provides LLVM support-library facilities such as diagnostics and casting. / 提供LLVM 支持库设施，如诊断与类型转换。
- `mlir/Pass/Pass.h`: Provides pass infrastructure and pass registration support. / 提供Pass 基础设施与 pass 注册支持。
- `mlir/Transforms/GreedyPatternRewriteDriver.h`: Provides generic transformation utilities and canonicalization helpers. / 提供通用变换工具与规范化辅助逻辑。
