# ShuffleVectorFMAOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/X86/Transforms/ShuffleVectorFMAOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements MLIR dialect logic, transformation code, or utilities related to `ShuffleVectorFMAOps`.
- **Purpose (CN)**: 实现与 `ShuffleVectorFMAOps` 相关的 MLIR 方言逻辑、变换代码或辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===- ShuffleVectorFMAOps.cpp --------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/Dialect/X86/Transforms.h"
#include "mlir/Dialect/X86/X86Dialect.h"

#include "mlir/IR/PatternMatch.h"

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
- **L9 EN**: Includes "mlir/Dialect/Vector/IR/VectorOps.h" to access dialect-specific IR, transforms, or shared utilities.
  **L9 CN**: 引入 "mlir/Dialect/Vector/IR/VectorOps.h" 以使用方言专用 IR、变换或共享工具。
- **L10 EN**: Includes "mlir/Dialect/X86/Transforms.h" to access dialect-specific IR, transforms, or shared utilities.
  **L10 CN**: 引入 "mlir/Dialect/X86/Transforms.h" 以使用方言专用 IR、变换或共享工具。
- **L11 EN**: Includes "mlir/Dialect/X86/X86Dialect.h" to access dialect-specific IR, transforms, or shared utilities.
  **L11 CN**: 引入 "mlir/Dialect/X86/X86Dialect.h" 以使用方言专用 IR、变换或共享工具。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Includes "mlir/IR/PatternMatch.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L13 CN**: 引入 "mlir/IR/PatternMatch.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Includes "mlir/Pass/Pass.h" to access pass infrastructure and pass registration support.
  **L15 CN**: 引入 "mlir/Pass/Pass.h" 以使用Pass 基础设施与 pass 注册支持。
- **L16 EN**: Includes "mlir/Transforms/GreedyPatternRewriteDriver.h" to access generic transformation utilities and canonicalization helpers.
  **L16 CN**: 引入 "mlir/Transforms/GreedyPatternRewriteDriver.h" 以使用通用变换工具与规范化辅助逻辑。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L18 EN**: Brings namespace `mlir` into local scope.
  **L18 CN**: 将命名空间 `mlir` 引入当前作用域。

### Lines 19-36

````cpp
using namespace mlir::vector;
using namespace mlir::x86;

namespace {

// Validates whether the given operation is an x86 operation and has only
// one consumer.
static bool validateFMAOperands(Value op) {
  if (auto cvt = op.getDefiningOp<x86::avx::CvtPackedEvenIndexedToF32Op>())
    return cvt.getResult().hasOneUse();

  if (auto bcst = op.getDefiningOp<x86::avx::BcstToPackedF32Op>())
    return bcst.getResult().hasOneUse();

  return false;
}

// Validates the vector.fma operation on the following conditions:
````
- **L19 EN**: Brings namespace `mlir::vector` into local scope.
  **L19 CN**: 将命名空间 `mlir::vector` 引入当前作用域。
- **L20 EN**: Brings namespace `mlir::x86` into local scope.
  **L20 CN**: 将命名空间 `mlir::x86` 引入当前作用域。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Opens namespace scope ``.
  **L22 CN**: 打开命名空间作用域 ``。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Comment explains nearby logic, invariants, or intent: `Validates whether the given operation is an x86 operation and has only`.
  **L24 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Validates whether the given operation is an x86 operation and has only`。
- **L25 EN**: Comment explains nearby logic, invariants, or intent: `one consumer.`.
  **L25 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`one consumer.`。
- **L26 EN**: Starts a function, method, lambda, or structured scope: `static bool validateFMAOperands(Value op) {`.
  **L26 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool validateFMAOperands(Value op) {`。
- **L27 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L27 CN**: 开始 `if` 控制流语句并计算其条件。
- **L28 EN**: Returns from the current function with `cvt.getResult().hasOneUse()`.
  **L28 CN**: 以 `cvt.getResult().hasOneUse()` 从当前函数返回。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L30 CN**: 开始 `if` 控制流语句并计算其条件。
- **L31 EN**: Returns from the current function with `bcst.getResult().hasOneUse()`.
  **L31 CN**: 以 `bcst.getResult().hasOneUse()` 从当前函数返回。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L33 EN**: Returns from the current function with `false`.
  **L33 CN**: 以 `false` 从当前函数返回。
- **L34 EN**: Closes the current lexical scope or compound statement.
  **L34 CN**: 结束当前词法作用域或复合语句块。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L36 EN**: Comment explains nearby logic, invariants, or intent: `Validates the vector.fma operation on the following conditions:`.
  **L36 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Validates the vector.fma operation on the following conditions:`。

### Lines 37-54

````cpp
// (i) one of the lhs or rhs defining operation should be
// CvtPackedEvenIndexedToF32Op, (ii) the lhs or rhs defining operation should be
// an x86 operation and has only one consumer, (iii) all operations
// are in the same block, and (iv) ths FMA has only one user.
static bool validateVectorFMAOp(vector::FMAOp fmaOp) {
  Value lhs = fmaOp.getLhs();
  Value rhs = fmaOp.getRhs();

  if (!isa<x86::avx::CvtPackedEvenIndexedToF32Op>(lhs.getDefiningOp()) &&
      !isa<x86::avx::CvtPackedEvenIndexedToF32Op>(rhs.getDefiningOp()))
    return false;

  if (!validateFMAOperands(lhs) || !validateFMAOperands(rhs))
    return false;

  if (lhs.getDefiningOp()->getBlock() != rhs.getDefiningOp()->getBlock())
    return false;

````
- **L37 EN**: Comment explains nearby logic, invariants, or intent: `(i) one of the lhs or rhs defining operation should be`.
  **L37 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(i) one of the lhs or rhs defining operation should be`。
- **L38 EN**: Comment explains nearby logic, invariants, or intent: `CvtPackedEvenIndexedToF32Op, (ii) the lhs or rhs defining operation should be`.
  **L38 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`CvtPackedEvenIndexedToF32Op, (ii) the lhs or rhs defining operation should be`。
- **L39 EN**: Comment explains nearby logic, invariants, or intent: `an x86 operation and has only one consumer, (iii) all operations`.
  **L39 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`an x86 operation and has only one consumer, (iii) all operations`。
- **L40 EN**: Comment explains nearby logic, invariants, or intent: `are in the same block, and (iv) ths FMA has only one user.`.
  **L40 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`are in the same block, and (iv) ths FMA has only one user.`。
- **L41 EN**: Starts a function, method, lambda, or structured scope: `static bool validateVectorFMAOp(vector::FMAOp fmaOp) {`.
  **L41 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool validateVectorFMAOp(vector::FMAOp fmaOp) {`。
- **L42 EN**: Initializes variable `lhs` from the right-hand expression.
  **L42 CN**: 使用右侧表达式初始化变量 `lhs`。
- **L43 EN**: Initializes variable `rhs` from the right-hand expression.
  **L43 CN**: 使用右侧表达式初始化变量 `rhs`。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L45 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L45 CN**: 开始 `if` 控制流语句并计算其条件。
- **L46 EN**: Continues logic associated with callable symbol `CvtPackedEvenIndexedToF32Op>`.
  **L46 CN**: 继续与可调用符号 `CvtPackedEvenIndexedToF32Op>` 相关的逻辑。
- **L47 EN**: Returns from the current function with `false`.
  **L47 CN**: 以 `false` 从当前函数返回。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L49 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L49 CN**: 开始 `if` 控制流语句并计算其条件。
- **L50 EN**: Returns from the current function with `false`.
  **L50 CN**: 以 `false` 从当前函数返回。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L52 CN**: 开始 `if` 控制流语句并计算其条件。
- **L53 EN**: Returns from the current function with `false`.
  **L53 CN**: 以 `false` 从当前函数返回。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 55-72

````cpp
  if (lhs.getDefiningOp()->getBlock() != fmaOp->getBlock())
    return false;

  if (!fmaOp.getResult().hasOneUse())
    return false;

  Operation *consumer = *fmaOp.getResult().getUsers().begin();
  if (consumer->getBlock() != fmaOp->getBlock())
    return false;

  return true;
}

// Moves vector.fma along with the lhs and rhs defining operation before its
// consumer. If the consumer is vector.ShapeCastOp and has only one user then
// move before the consumer of vector.ShapeCastOp.
// TODO: Move before first consumer, if there are multiple.
static void moveFMA(PatternRewriter &rewriter, vector::FMAOp fmaOp) {
````
- **L55 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L55 CN**: 开始 `if` 控制流语句并计算其条件。
- **L56 EN**: Returns from the current function with `false`.
  **L56 CN**: 以 `false` 从当前函数返回。
- **L57 EN**: Blank line separating nearby declarations or logic blocks.
  **L57 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L58 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L58 CN**: 开始 `if` 控制流语句并计算其条件。
- **L59 EN**: Returns from the current function with `false`.
  **L59 CN**: 以 `false` 从当前函数返回。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L61 EN**: Executes a call or declaration centered on `*fmaOp.getResult`.
  **L61 CN**: 执行以 `*fmaOp.getResult` 为核心的调用或声明。
- **L62 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L62 CN**: 开始 `if` 控制流语句并计算其条件。
- **L63 EN**: Returns from the current function with `false`.
  **L63 CN**: 以 `false` 从当前函数返回。
- **L64 EN**: Blank line separating nearby declarations or logic blocks.
  **L64 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L65 EN**: Returns from the current function with `true`.
  **L65 CN**: 以 `true` 从当前函数返回。
- **L66 EN**: Closes the current lexical scope or compound statement.
  **L66 CN**: 结束当前词法作用域或复合语句块。
- **L67 EN**: Blank line separating nearby declarations or logic blocks.
  **L67 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L68 EN**: Comment explains nearby logic, invariants, or intent: `Moves vector.fma along with the lhs and rhs defining operation before its`.
  **L68 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Moves vector.fma along with the lhs and rhs defining operation before its`。
- **L69 EN**: Comment explains nearby logic, invariants, or intent: `consumer. If the consumer is vector.ShapeCastOp and has only one user then`.
  **L69 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`consumer. If the consumer is vector.ShapeCastOp and has only one user then`。
- **L70 EN**: Comment explains nearby logic, invariants, or intent: `move before the consumer of vector.ShapeCastOp.`.
  **L70 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`move before the consumer of vector.ShapeCastOp.`。
- **L71 EN**: Comment records a pending task or caution: `TODO: Move before first consumer, if there are multiple.`.
  **L71 CN**: 注释记录了待办事项或注意点：`TODO: Move before first consumer, if there are multiple.`。
- **L72 EN**: Starts a function, method, lambda, or structured scope: `static void moveFMA(PatternRewriter &rewriter, vector::FMAOp fmaOp) {`.
  **L72 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static void moveFMA(PatternRewriter &rewriter, vector::FMAOp fmaOp) {`。

### Lines 73-90

````cpp
  Operation *consumer = *fmaOp.getResult().getUsers().begin();

  if (auto shapeCastOp = dyn_cast<vector::ShapeCastOp>(consumer)) {
    if (shapeCastOp.getResult().hasOneUse()) {
      Operation *nxtConsumer = *shapeCastOp.getResult().getUsers().begin();
      if (nxtConsumer->getBlock() == fmaOp->getBlock()) {
        consumer = *shapeCastOp.getResult().getUsers().begin();
        rewriter.moveOpBefore(fmaOp.getLhs().getDefiningOp(), consumer);
        rewriter.moveOpBefore(fmaOp.getRhs().getDefiningOp(), consumer);
        rewriter.moveOpBefore(fmaOp.getOperation(), consumer);
        rewriter.moveOpBefore(shapeCastOp.getOperation(), consumer);
        return;
      }
    }
  }

  rewriter.moveOpBefore(fmaOp.getLhs().getDefiningOp(), consumer);
  rewriter.moveOpBefore(fmaOp.getRhs().getDefiningOp(), consumer);
````
- **L73 EN**: Executes a call or declaration centered on `*fmaOp.getResult`.
  **L73 CN**: 执行以 `*fmaOp.getResult` 为核心的调用或声明。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L75 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L75 CN**: 开始 `if` 控制流语句并计算其条件。
- **L76 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L76 CN**: 开始 `if` 控制流语句并计算其条件。
- **L77 EN**: Executes a call or declaration centered on `*shapeCastOp.getResult`.
  **L77 CN**: 执行以 `*shapeCastOp.getResult` 为核心的调用或声明。
- **L78 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L78 CN**: 开始 `if` 控制流语句并计算其条件。
- **L79 EN**: Executes a call or declaration centered on `*shapeCastOp.getResult`.
  **L79 CN**: 执行以 `*shapeCastOp.getResult` 为核心的调用或声明。
- **L80 EN**: Executes a call or declaration centered on `rewriter.moveOpBefore`.
  **L80 CN**: 执行以 `rewriter.moveOpBefore` 为核心的调用或声明。
- **L81 EN**: Executes a call or declaration centered on `rewriter.moveOpBefore`.
  **L81 CN**: 执行以 `rewriter.moveOpBefore` 为核心的调用或声明。
- **L82 EN**: Executes a call or declaration centered on `rewriter.moveOpBefore`.
  **L82 CN**: 执行以 `rewriter.moveOpBefore` 为核心的调用或声明。
- **L83 EN**: Executes a call or declaration centered on `rewriter.moveOpBefore`.
  **L83 CN**: 执行以 `rewriter.moveOpBefore` 为核心的调用或声明。
- **L84 EN**: Returns from the current function with `void`.
  **L84 CN**: 以 `void` 从当前函数返回。
- **L85 EN**: Closes the current lexical scope or compound statement.
  **L85 CN**: 结束当前词法作用域或复合语句块。
- **L86 EN**: Closes the current lexical scope or compound statement.
  **L86 CN**: 结束当前词法作用域或复合语句块。
- **L87 EN**: Closes the current lexical scope or compound statement.
  **L87 CN**: 结束当前词法作用域或复合语句块。
- **L88 EN**: Blank line separating nearby declarations or logic blocks.
  **L88 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L89 EN**: Executes a call or declaration centered on `rewriter.moveOpBefore`.
  **L89 CN**: 执行以 `rewriter.moveOpBefore` 为核心的调用或声明。
- **L90 EN**: Executes a call or declaration centered on `rewriter.moveOpBefore`.
  **L90 CN**: 执行以 `rewriter.moveOpBefore` 为核心的调用或声明。

### Lines 91-108

````cpp
  rewriter.moveOpBefore(fmaOp.getOperation(), consumer);

  return;
}

// Shuffle FMAs with x86 operations as operands such that
// FMAs are grouped with respect to odd/even packed index.
//
// For example:
// ```
//   %1 = x86.avx.bcst_to_f32.packed
//   %2 = x86.avx.cvt.packed.odd.indexed_to_f32
//   %3 = vector.fma %1, %2, %arg1
//   %4 = x86.avx.bcst_to_f32.packed
//   %5 = x86.avx.cvt.packed.even.indexed_to_f32
//   %6 = vector.fma %4, %5, %3
//   %7 = x86.avx.bcst_to_f32.packed
//   %8 = x86.avx.cvt.packed.odd.indexed_to_f32
````
- **L91 EN**: Executes a call or declaration centered on `rewriter.moveOpBefore`.
  **L91 CN**: 执行以 `rewriter.moveOpBefore` 为核心的调用或声明。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L93 EN**: Returns from the current function with `void`.
  **L93 CN**: 以 `void` 从当前函数返回。
- **L94 EN**: Closes the current lexical scope or compound statement.
  **L94 CN**: 结束当前词法作用域或复合语句块。
- **L95 EN**: Blank line separating nearby declarations or logic blocks.
  **L95 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L96 EN**: Comment explains nearby logic, invariants, or intent: `Shuffle FMAs with x86 operations as operands such that`.
  **L96 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Shuffle FMAs with x86 operations as operands such that`。
- **L97 EN**: Comment explains nearby logic, invariants, or intent: `FMAs are grouped with respect to odd/even packed index.`.
  **L97 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FMAs are grouped with respect to odd/even packed index.`。
- **L98 EN**: Separator comment used for visual grouping.
  **L98 CN**: 用于视觉分组的分隔注释。
- **L99 EN**: Comment explains nearby logic, invariants, or intent: `For example:`.
  **L99 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For example:`。
- **L100 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L100 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L101 EN**: Comment explains nearby logic, invariants, or intent: `%1 = x86.avx.bcst_to_f32.packed`.
  **L101 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%1 = x86.avx.bcst_to_f32.packed`。
- **L102 EN**: Comment explains nearby logic, invariants, or intent: `%2 = x86.avx.cvt.packed.odd.indexed_to_f32`.
  **L102 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%2 = x86.avx.cvt.packed.odd.indexed_to_f32`。
- **L103 EN**: Comment explains nearby logic, invariants, or intent: `%3 = vector.fma %1, %2, %arg1`.
  **L103 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%3 = vector.fma %1, %2, %arg1`。
- **L104 EN**: Comment explains nearby logic, invariants, or intent: `%4 = x86.avx.bcst_to_f32.packed`.
  **L104 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%4 = x86.avx.bcst_to_f32.packed`。
- **L105 EN**: Comment explains nearby logic, invariants, or intent: `%5 = x86.avx.cvt.packed.even.indexed_to_f32`.
  **L105 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%5 = x86.avx.cvt.packed.even.indexed_to_f32`。
- **L106 EN**: Comment explains nearby logic, invariants, or intent: `%6 = vector.fma %4, %5, %3`.
  **L106 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%6 = vector.fma %4, %5, %3`。
- **L107 EN**: Comment explains nearby logic, invariants, or intent: `%7 = x86.avx.bcst_to_f32.packed`.
  **L107 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%7 = x86.avx.bcst_to_f32.packed`。
- **L108 EN**: Comment explains nearby logic, invariants, or intent: `%8 = x86.avx.cvt.packed.odd.indexed_to_f32`.
  **L108 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%8 = x86.avx.cvt.packed.odd.indexed_to_f32`。

### Lines 109-126

````cpp
//   %9 = vector.fma %7, %8, %arg2
//   %10 = x86.avx.bcst_to_f32.packed
//   %11 = x86.avx.cvt.packed.even.indexed_to_f32
//   %12 = vector.fma %10, %11, %9
//   yield %6, %12
// ```
// to
// ```
//   %1 = x86.avx.bcst_to_f32.packed
//   %2 = x86.avx.cvt.packed.odd.indexed_to_f32
//   %3 = vector.fma %1, %2, %arg1
//   %7 = x86.avx.bcst_to_f32.packed
//   %8 = x86.avx.cvt.packed.odd.indexed_to_f32
//   %9 = vector.fma %7, %8, %arg2
//   %4 = x86.avx.bcst_to_f32.packed
//   %5 = x86.avx.cvt.packed.even.indexed_to_f32
//   %6 = vector.fma %4, %5, %3
//   %10 = x86.avx.bcst_to_f32.packed
````
- **L109 EN**: Comment explains nearby logic, invariants, or intent: `%9 = vector.fma %7, %8, %arg2`.
  **L109 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%9 = vector.fma %7, %8, %arg2`。
- **L110 EN**: Comment explains nearby logic, invariants, or intent: `%10 = x86.avx.bcst_to_f32.packed`.
  **L110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%10 = x86.avx.bcst_to_f32.packed`。
- **L111 EN**: Comment explains nearby logic, invariants, or intent: `%11 = x86.avx.cvt.packed.even.indexed_to_f32`.
  **L111 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%11 = x86.avx.cvt.packed.even.indexed_to_f32`。
- **L112 EN**: Comment explains nearby logic, invariants, or intent: `%12 = vector.fma %10, %11, %9`.
  **L112 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%12 = vector.fma %10, %11, %9`。
- **L113 EN**: Comment explains nearby logic, invariants, or intent: `yield %6, %12`.
  **L113 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`yield %6, %12`。
- **L114 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L114 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L115 EN**: Comment explains nearby logic, invariants, or intent: `to`.
  **L115 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to`。
- **L116 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L116 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L117 EN**: Comment explains nearby logic, invariants, or intent: `%1 = x86.avx.bcst_to_f32.packed`.
  **L117 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%1 = x86.avx.bcst_to_f32.packed`。
- **L118 EN**: Comment explains nearby logic, invariants, or intent: `%2 = x86.avx.cvt.packed.odd.indexed_to_f32`.
  **L118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%2 = x86.avx.cvt.packed.odd.indexed_to_f32`。
- **L119 EN**: Comment explains nearby logic, invariants, or intent: `%3 = vector.fma %1, %2, %arg1`.
  **L119 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%3 = vector.fma %1, %2, %arg1`。
- **L120 EN**: Comment explains nearby logic, invariants, or intent: `%7 = x86.avx.bcst_to_f32.packed`.
  **L120 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%7 = x86.avx.bcst_to_f32.packed`。
- **L121 EN**: Comment explains nearby logic, invariants, or intent: `%8 = x86.avx.cvt.packed.odd.indexed_to_f32`.
  **L121 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%8 = x86.avx.cvt.packed.odd.indexed_to_f32`。
- **L122 EN**: Comment explains nearby logic, invariants, or intent: `%9 = vector.fma %7, %8, %arg2`.
  **L122 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%9 = vector.fma %7, %8, %arg2`。
- **L123 EN**: Comment explains nearby logic, invariants, or intent: `%4 = x86.avx.bcst_to_f32.packed`.
  **L123 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%4 = x86.avx.bcst_to_f32.packed`。
- **L124 EN**: Comment explains nearby logic, invariants, or intent: `%5 = x86.avx.cvt.packed.even.indexed_to_f32`.
  **L124 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%5 = x86.avx.cvt.packed.even.indexed_to_f32`。
- **L125 EN**: Comment explains nearby logic, invariants, or intent: `%6 = vector.fma %4, %5, %3`.
  **L125 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%6 = vector.fma %4, %5, %3`。
- **L126 EN**: Comment explains nearby logic, invariants, or intent: `%10 = x86.avx.bcst_to_f32.packed`.
  **L126 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%10 = x86.avx.bcst_to_f32.packed`。

### Lines 127-144

````cpp
//   %11 = x86.avx.cvt.packed.even.indexed_to_f32
//   %12 = vector.fma %10, %11, %9
//   yield %9, %12
// ```
// TODO: Shuffling supported only if the FMA, lhs/rhs defining operations
// have only one consumer. Have to extend this pass for multiple consumers.
struct ShuffleVectorFMAOps : public OpRewritePattern<vector::FMAOp> {
  using OpRewritePattern<vector::FMAOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(vector::FMAOp fmaOp,
                                PatternRewriter &rewriter) const override {

    if (!validateVectorFMAOp(fmaOp))
      return failure();

    llvm::SmallVector<vector::FMAOp> fmaOps;
    Operation *nextOp = fmaOp;
    bool stopAtNextDependentFMA = true;
````
- **L127 EN**: Comment explains nearby logic, invariants, or intent: `%11 = x86.avx.cvt.packed.even.indexed_to_f32`.
  **L127 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%11 = x86.avx.cvt.packed.even.indexed_to_f32`。
- **L128 EN**: Comment explains nearby logic, invariants, or intent: `%12 = vector.fma %10, %11, %9`.
  **L128 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%12 = vector.fma %10, %11, %9`。
- **L129 EN**: Comment explains nearby logic, invariants, or intent: `yield %9, %12`.
  **L129 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`yield %9, %12`。
- **L130 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L130 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L131 EN**: Comment records a pending task or caution: `TODO: Shuffling supported only if the FMA, lhs/rhs defining operations`.
  **L131 CN**: 注释记录了待办事项或注意点：`TODO: Shuffling supported only if the FMA, lhs/rhs defining operations`。
- **L132 EN**: Comment explains nearby logic, invariants, or intent: `have only one consumer. Have to extend this pass for multiple consumers.`.
  **L132 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`have only one consumer. Have to extend this pass for multiple consumers.`。
- **L133 EN**: Declares struct `ShuffleVectorFMAOps`.
  **L133 CN**: 声明 struct `ShuffleVectorFMAOps`。
- **L134 EN**: Executes a standalone statement or declaration: `using OpRewritePattern<vector::FMAOp>::OpRewritePattern;`.
  **L134 CN**: 执行一条独立语句或声明：`using OpRewritePattern<vector::FMAOp>::OpRewritePattern;`。
- **L135 EN**: Blank line separating nearby declarations or logic blocks.
  **L135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(vector::FMAOp fmaOp,`.
  **L136 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(vector::FMAOp fmaOp,`。
- **L137 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L137 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L138 EN**: Blank line separating nearby declarations or logic blocks.
  **L138 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L139 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L139 CN**: 开始 `if` 控制流语句并计算其条件。
- **L140 EN**: Returns from the current function with `failure()`.
  **L140 CN**: 以 `failure()` 从当前函数返回。
- **L141 EN**: Blank line separating nearby declarations or logic blocks.
  **L141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L142 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<vector::FMAOp> fmaOps;`.
  **L142 CN**: 执行一条独立语句或声明：`llvm::SmallVector<vector::FMAOp> fmaOps;`。
- **L143 EN**: Executes a standalone statement or declaration: `Operation *nextOp = fmaOp;`.
  **L143 CN**: 执行一条独立语句或声明：`Operation *nextOp = fmaOp;`。
- **L144 EN**: Initializes variable `stopAtNextDependentFMA` from the right-hand expression.
  **L144 CN**: 使用右侧表达式初始化变量 `stopAtNextDependentFMA`。

### Lines 145-162

````cpp

    // Break the loop and return failure if the immediate next FMA op
    // have CvtPackedEvenIndexedToF32Op in it's lhs/rhs defining ops.
    while ((nextOp = nextOp->getNextNode())) {
      auto fma = dyn_cast<vector::FMAOp>(nextOp);
      if (!fma)
        continue;

      bool hasX86CvtOperand = isa<x86::avx::CvtPackedEvenIndexedToF32Op>(
                                  fma.getLhs().getDefiningOp()) ||
                              isa<x86::avx::CvtPackedEvenIndexedToF32Op>(
                                  fma.getRhs().getDefiningOp());

      if (hasX86CvtOperand && stopAtNextDependentFMA)
        break;

      if (validateVectorFMAOp(fma))
        fmaOps.push_back(fma);
````
- **L145 EN**: Blank line separating nearby declarations or logic blocks.
  **L145 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L146 EN**: Comment explains nearby logic, invariants, or intent: `Break the loop and return failure if the immediate next FMA op`.
  **L146 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Break the loop and return failure if the immediate next FMA op`。
- **L147 EN**: Comment explains nearby logic, invariants, or intent: `have CvtPackedEvenIndexedToF32Op in it's lhs/rhs defining ops.`.
  **L147 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`have CvtPackedEvenIndexedToF32Op in it's lhs/rhs defining ops.`。
- **L148 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L148 CN**: 开始 `while` 控制流语句并计算其条件。
- **L149 EN**: Initializes variable `fma` from the right-hand expression.
  **L149 CN**: 使用右侧表达式初始化变量 `fma`。
- **L150 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L150 CN**: 开始 `if` 控制流语句并计算其条件。
- **L151 EN**: Skips to the next loop iteration.
  **L151 CN**: 跳到下一次循环迭代。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L153 EN**: Continues logic associated with callable symbol `CvtPackedEvenIndexedToF32Op>`.
  **L153 CN**: 继续与可调用符号 `CvtPackedEvenIndexedToF32Op>` 相关的逻辑。
- **L154 EN**: Continues logic associated with callable symbol `getLhs`.
  **L154 CN**: 继续与可调用符号 `getLhs` 相关的逻辑。
- **L155 EN**: Continues logic associated with callable symbol `CvtPackedEvenIndexedToF32Op>`.
  **L155 CN**: 继续与可调用符号 `CvtPackedEvenIndexedToF32Op>` 相关的逻辑。
- **L156 EN**: Executes a call or declaration centered on `fma.getRhs`.
  **L156 CN**: 执行以 `fma.getRhs` 为核心的调用或声明。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L158 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L158 CN**: 开始 `if` 控制流语句并计算其条件。
- **L159 EN**: Exits the nearest loop or switch statement.
  **L159 CN**: 退出最近的循环或 switch 语句。
- **L160 EN**: Blank line separating nearby declarations or logic blocks.
  **L160 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L161 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L161 CN**: 开始 `if` 控制流语句并计算其条件。
- **L162 EN**: Executes a call or declaration centered on `fmaOps.push_back`.
  **L162 CN**: 执行以 `fmaOps.push_back` 为核心的调用或声明。

### Lines 163-180

````cpp

      stopAtNextDependentFMA = false;
    }

    if (fmaOps.empty())
      return rewriter.notifyMatchFailure(
          fmaOp, "No eligible FMA operations were found: the operation may "
                 "already be shuffled, there may be no following FMAs, or the "
                 "following FMAs do not satisfy the shuffle conditions.");

    fmaOps.push_back(fmaOp);
    for (auto fmaOp : fmaOps)
      moveFMA(rewriter, fmaOp);

    return success();
  }
};

````
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L164 EN**: Executes a standalone statement or declaration: `stopAtNextDependentFMA = false;`.
  **L164 CN**: 执行一条独立语句或声明：`stopAtNextDependentFMA = false;`。
- **L165 EN**: Closes the current lexical scope or compound statement.
  **L165 CN**: 结束当前词法作用域或复合语句块。
- **L166 EN**: Blank line separating nearby declarations or logic blocks.
  **L166 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L167 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L167 CN**: 开始 `if` 控制流语句并计算其条件。
- **L168 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L168 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L169 EN**: Continues the surrounding expression or declaration: `fmaOp, "No eligible FMA operations were found: the operation may "`.
  **L169 CN**: 继续构造周围的表达式或声明：`fmaOp, "No eligible FMA operations were found: the operation may "`。
- **L170 EN**: Continues the surrounding expression or declaration: `"already be shuffled, there may be no following FMAs, or the "`.
  **L170 CN**: 继续构造周围的表达式或声明：`"already be shuffled, there may be no following FMAs, or the "`。
- **L171 EN**: Executes a standalone statement or declaration: `"following FMAs do not satisfy the shuffle conditions.");`.
  **L171 CN**: 执行一条独立语句或声明：`"following FMAs do not satisfy the shuffle conditions.");`。
- **L172 EN**: Blank line separating nearby declarations or logic blocks.
  **L172 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L173 EN**: Executes a call or declaration centered on `fmaOps.push_back`.
  **L173 CN**: 执行以 `fmaOps.push_back` 为核心的调用或声明。
- **L174 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L174 CN**: 开始 `for` 控制流语句并计算其条件。
- **L175 EN**: Executes a call or declaration centered on `moveFMA`.
  **L175 CN**: 执行以 `moveFMA` 为核心的调用或声明。
- **L176 EN**: Blank line separating nearby declarations or logic blocks.
  **L176 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L177 EN**: Returns from the current function with `success()`.
  **L177 CN**: 以 `success()` 从当前函数返回。
- **L178 EN**: Closes the current lexical scope or compound statement.
  **L178 CN**: 结束当前词法作用域或复合语句块。
- **L179 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L179 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 181-185

````cpp
} // namespace

void x86::populateShuffleVectorFMAOpsPatterns(RewritePatternSet &patterns) {
  patterns.add<ShuffleVectorFMAOps>(patterns.getContext());
}
````
- **L181 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L181 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L183 EN**: Starts a function, method, lambda, or structured scope: `void x86::populateShuffleVectorFMAOpsPatterns(RewritePatternSet &patterns) {`.
  **L183 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void x86::populateShuffleVectorFMAOpsPatterns(RewritePatternSet &patterns) {`。
- **L184 EN**: Executes a call or declaration centered on `patterns.add<ShuffleVectorFMAOps>`.
  **L184 CN**: 执行以 `patterns.add<ShuffleVectorFMAOps>` 为核心的调用或声明。
- **L185 EN**: Closes the current lexical scope or compound statement.
  **L185 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Rewrite-driven lowering / 基于重写的 lowering**
- **Dialect definition and registration / 方言定义与注册**
- **Operation semantics and verification / 操作语义与验证**
- **Pattern-based rewriting / 基于模式的重写**
- **Pattern collection management / 重写模式集合管理**
- **Failure-aware result propagation / 带失败语义的结果传播**
- **IR mutation through rewriter helpers / 通过重写器辅助工具进行 IR 变换**
- **Greedy canonicalization driver / 贪心规范化驱动**
- **Basic block ownership and traversal / 基本块拥有关系与遍历**
- **SSA value representation / SSA 值表示**

## Dependencies / 依赖关系

- `mlir/Dialect/Vector/IR/VectorOps.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/X86/Transforms.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/X86/X86Dialect.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/IR/PatternMatch.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `mlir/Pass/Pass.h`: Provides pass infrastructure and pass registration support. / 提供Pass 基础设施与 pass 注册支持。
- `mlir/Transforms/GreedyPatternRewriteDriver.h`: Provides generic transformation utilities and canonicalization helpers. / 提供通用变换工具与规范化辅助逻辑。
