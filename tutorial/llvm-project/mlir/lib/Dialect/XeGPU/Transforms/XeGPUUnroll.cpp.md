# XeGPUUnroll.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/XeGPU/Transforms/XeGPUUnroll.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains patterns for unrolling XeGPU operations. It follows a similar concept and design as vector unroll patterns, serving as a complement to them.
- **Purpose (CN)**: 实现 XeGPU lowering、调度与重写 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===- XeGPUUnroll.cpp - patterns to do unrolling ---------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains patterns for unrolling XeGPU operations. It follows a
// similar concept and design as vector unroll patterns, serving as a complement
// to them.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/Dialect/XeGPU/IR/XeGPU.h"
#include "mlir/Dialect/XeGPU/Transforms/Transforms.h"
#include "mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h"
#include "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/Support/DebugLog.h"

namespace mlir {
namespace xegpu {
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
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file contains patterns for unrolling XeGPU operations. It follows a`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file contains patterns for unrolling XeGPU operations. It follows a`。
- **L10 EN**: Comment explains nearby logic, invariants, or intent: `similar concept and design as vector unroll patterns, serving as a complement`.
  **L10 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`similar concept and design as vector unroll patterns, serving as a complement`。
- **L11 EN**: Comment explains nearby logic, invariants, or intent: `to them.`.
  **L11 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to them.`。
- **L12 EN**: Separator comment used for visual grouping.
  **L12 CN**: 用于视觉分组的分隔注释。
- **L13 EN**: Banner comment marking a file or section boundary.
  **L13 CN**: 横幅注释，用于标记文件或章节边界。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Includes "mlir/Dialect/Utils/IndexingUtils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L15 CN**: 引入 "mlir/Dialect/Utils/IndexingUtils.h" 以使用方言专用 IR、变换或共享工具。
- **L16 EN**: Includes "mlir/Dialect/XeGPU/IR/XeGPU.h" to access dialect-specific IR, transforms, or shared utilities.
  **L16 CN**: 引入 "mlir/Dialect/XeGPU/IR/XeGPU.h" 以使用方言专用 IR、变换或共享工具。
- **L17 EN**: Includes "mlir/Dialect/XeGPU/Transforms/Transforms.h" to access dialect-specific IR, transforms, or shared utilities.
  **L17 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/Transforms.h" 以使用方言专用 IR、变换或共享工具。
- **L18 EN**: Includes "mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h" to access dialect-specific IR, transforms, or shared utilities.
  **L18 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h" 以使用方言专用 IR、变换或共享工具。
- **L19 EN**: Includes "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L19 CN**: 引入 "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h" 以使用方言专用 IR、变换或共享工具。
- **L20 EN**: Includes "llvm/ADT/STLExtras.h" to access LLVM ADT containers and low-level utility types.
  **L20 CN**: 引入 "llvm/ADT/STLExtras.h" 以使用LLVM ADT 容器与底层工具类型。
- **L21 EN**: Includes "llvm/Support/DebugLog.h" to access LLVM support-library facilities such as diagnostics and casting.
  **L21 CN**: 引入 "llvm/Support/DebugLog.h" 以使用LLVM 支持库设施，如诊断与类型转换。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L23 EN**: Opens namespace scope `mlir`.
  **L23 CN**: 打开命名空间作用域 `mlir`。
- **L24 EN**: Opens namespace scope `xegpu`.
  **L24 CN**: 打开命名空间作用域 `xegpu`。

### Lines 25-48

````cpp
#define GEN_PASS_DEF_XEGPUUNROLL
#include "mlir/Dialect/XeGPU/Transforms/Passes.h.inc"
} // namespace xegpu
} // namespace mlir

#define DEBUG_TYPE "xegpu-unroll"

using namespace mlir;

namespace {

template <typename SourceOp>
struct UnrollPattern : public OpRewritePattern<SourceOp> {
  UnrollPattern(MLIRContext *context, const xegpu::UnrollOptions &options,
                PatternBenefit benefit = 1)
      : OpRewritePattern<SourceOp>(context, benefit), options(options) {}

protected:
  /// Return the target shape for the given `op`. Return std::nullopt if the
  /// op shouldn't be or cannot be unrolled.
  std::optional<SmallVector<int64_t>> getTargetShape(Operation *op) const {
    LDBG() << "Get unroll shape for: " << *op;

    if (options.filterConstraint && failed(options.filterConstraint(op))) {
````
- **L25 EN**: Defines macro `GEN_PASS_DEF_XEGPUUNROLL` for generated declarations, local shorthand, or conditional logic.
  **L25 CN**: 定义宏 `GEN_PASS_DEF_XEGPUUNROLL`，供生成式声明、本地简写或条件逻辑使用。
- **L26 EN**: Includes "mlir/Dialect/XeGPU/Transforms/Passes.h.inc" to access dialect-specific IR, transforms, or shared utilities.
  **L26 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/Passes.h.inc" 以使用方言专用 IR、变换或共享工具。
- **L27 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace xegpu`.
  **L27 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace xegpu`。
- **L28 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace mlir`.
  **L28 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace mlir`。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Defines macro `DEBUG_TYPE` for generated declarations, local shorthand, or conditional logic.
  **L30 CN**: 定义宏 `DEBUG_TYPE`，供生成式声明、本地简写或条件逻辑使用。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L32 EN**: Brings namespace `mlir` into local scope.
  **L32 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L34 EN**: Opens namespace scope ``.
  **L34 CN**: 打开命名空间作用域 ``。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L36 EN**: Introduces template parameters or specialization context: `template <typename SourceOp>`.
  **L36 CN**: 为后续声明引入模板参数或特化上下文：`template <typename SourceOp>`。
- **L37 EN**: Declares struct `UnrollPattern`.
  **L37 CN**: 声明 struct `UnrollPattern`。
- **L38 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnrollPattern(MLIRContext *context, const xegpu::UnrollOptions &options,`.
  **L38 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnrollPattern(MLIRContext *context, const xegpu::UnrollOptions &options,`。
- **L39 EN**: Continues the surrounding expression or declaration: `PatternBenefit benefit = 1)`.
  **L39 CN**: 继续构造周围的表达式或声明：`PatternBenefit benefit = 1)`。
- **L40 EN**: Continues logic associated with callable symbol `OpRewritePattern<SourceOp>`.
  **L40 CN**: 继续与可调用符号 `OpRewritePattern<SourceOp>` 相关的逻辑。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L42 EN**: Sets the following members to `protected` access.
  **L42 CN**: 将后续成员的访问级别设为 `protected`。
- **L43 EN**: Comment explains nearby logic, invariants, or intent: `Return the target shape for the given `op`. Return std::nullopt if the`.
  **L43 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return the target shape for the given `op`. Return std::nullopt if the`。
- **L44 EN**: Comment explains nearby logic, invariants, or intent: `op shouldn't be or cannot be unrolled.`.
  **L44 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`op shouldn't be or cannot be unrolled.`。
- **L45 EN**: Starts a function, method, lambda, or structured scope: `std::optional<SmallVector<int64_t>> getTargetShape(Operation *op) const {`.
  **L45 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::optional<SmallVector<int64_t>> getTargetShape(Operation *op) const {`。
- **L46 EN**: Executes a call or declaration centered on `LDBG`.
  **L46 CN**: 执行以 `LDBG` 为核心的调用或声明。
- **L47 EN**: Blank line separating nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L48 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L48 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 49-72

````cpp
      LDBG() << "--no filter constraint -> BAIL";
      return std::nullopt;
    }

    assert(options.nativeShape &&
           "expects the native shape for native shape call back function.");
    auto nativeShape = options.nativeShape(op);
    return nativeShape;
  }

  SmallVector<Type> getUnrolledTypes(ShapedType type,
                                     ArrayRef<int64_t> tileShape,
                                     bool returnSingleType = false) const {
    return options.getUnrolledTypes(type, tileShape, returnSingleType);
  }

  /// Emulate the the unpack behavior using insert_strided_slice for VectorType
  /// values and unrealized_conversion_cast for TensorDescType values.
  Value unpack(ValueRange srcs, Type destTy, ArrayRef<int64_t> blockSize,
               Location loc, PatternRewriter &rewriter) const {
    if (auto vecTy = dyn_cast<VectorType>(destTy)) {
      auto shape = vecTy.getShape();
      return xegpu::createVectorWithShapeFromValues(rewriter, loc, srcs, shape);
    }
````
- **L49 EN**: Executes a call or declaration centered on `LDBG`.
  **L49 CN**: 执行以 `LDBG` 为核心的调用或声明。
- **L50 EN**: Returns from the current function with `std::nullopt`.
  **L50 CN**: 以 `std::nullopt` 从当前函数返回。
- **L51 EN**: Closes the current lexical scope or compound statement.
  **L51 CN**: 结束当前词法作用域或复合语句块。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Checks an internal invariant in debug builds.
  **L53 CN**: 在调试构建中检查内部不变式。
- **L54 EN**: Executes a standalone statement or declaration: `"expects the native shape for native shape call back function.");`.
  **L54 CN**: 执行一条独立语句或声明：`"expects the native shape for native shape call back function.");`。
- **L55 EN**: Initializes variable `nativeShape` from the right-hand expression.
  **L55 CN**: 使用右侧表达式初始化变量 `nativeShape`。
- **L56 EN**: Returns from the current function with `nativeShape`.
  **L56 CN**: 以 `nativeShape` 从当前函数返回。
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L59 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<Type> getUnrolledTypes(ShapedType type,`.
  **L59 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<Type> getUnrolledTypes(ShapedType type,`。
- **L60 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t> tileShape,`.
  **L60 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t> tileShape,`。
- **L61 EN**: Continues the surrounding expression or declaration: `bool returnSingleType = false) const {`.
  **L61 CN**: 继续构造周围的表达式或声明：`bool returnSingleType = false) const {`。
- **L62 EN**: Returns from the current function with `options.getUnrolledTypes(type, tileShape, returnSingleType)`.
  **L62 CN**: 以 `options.getUnrolledTypes(type, tileShape, returnSingleType)` 从当前函数返回。
- **L63 EN**: Closes the current lexical scope or compound statement.
  **L63 CN**: 结束当前词法作用域或复合语句块。
- **L64 EN**: Blank line separating nearby declarations or logic blocks.
  **L64 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L65 EN**: Comment explains nearby logic, invariants, or intent: `Emulate the the unpack behavior using insert_strided_slice for VectorType`.
  **L65 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Emulate the the unpack behavior using insert_strided_slice for VectorType`。
- **L66 EN**: Comment explains nearby logic, invariants, or intent: `values and unrealized_conversion_cast for TensorDescType values.`.
  **L66 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`values and unrealized_conversion_cast for TensorDescType values.`。
- **L67 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value unpack(ValueRange srcs, Type destTy, ArrayRef<int64_t> blockSize,`.
  **L67 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value unpack(ValueRange srcs, Type destTy, ArrayRef<int64_t> blockSize,`。
- **L68 EN**: Continues the surrounding expression or declaration: `Location loc, PatternRewriter &rewriter) const {`.
  **L68 CN**: 继续构造周围的表达式或声明：`Location loc, PatternRewriter &rewriter) const {`。
- **L69 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L69 CN**: 开始 `if` 控制流语句并计算其条件。
- **L70 EN**: Initializes variable `shape` from the right-hand expression.
  **L70 CN**: 使用右侧表达式初始化变量 `shape`。
- **L71 EN**: Returns from the current function with `xegpu::createVectorWithShapeFromValues(rewriter, loc, srcs, shape)`.
  **L71 CN**: 以 `xegpu::createVectorWithShapeFromValues(rewriter, loc, srcs, shape)` 从当前函数返回。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。

### Lines 73-96

````cpp

    if (isa<xegpu::TensorDescType>(destTy)) {
      auto attr = NamedAttribute(rewriter.getStringAttr(unpackAttrName),
                                 rewriter.getUnitAttr());
      auto blkAttr = NamedAttribute(rewriter.getStringAttr(blockAttrName),
                                    rewriter.getDenseI64ArrayAttr(blockSize));
      auto castOp = UnrealizedConversionCastOp::create(
          rewriter, loc, destTy, srcs,
          ArrayRef<NamedAttribute>({attr, blkAttr}));
      return castOp.getResult(0);
    }

    llvm_unreachable("Unexpected destTy.");
    return Value();
  }

  /// Emulate the the pack behavior using extract_strided_slice for VectorType
  /// values and unrealized_conversion_cast for TensorDescType values.
  SmallVector<Value> pack(Value src, TypeRange destTypes,
                          ArrayRef<int64_t> blockSize, Location loc,
                          PatternRewriter &rewriter) const {
    if (auto vecTy = dyn_cast<VectorType>(src.getType())) {
      return xegpu::extractVectorsWithShapeFromValue(rewriter, loc, src,
                                                     blockSize);
````
- **L73 EN**: Blank line separating nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L74 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L74 CN**: 开始 `if` 控制流语句并计算其条件。
- **L75 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto attr = NamedAttribute(rewriter.getStringAttr(unpackAttrName),`.
  **L75 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto attr = NamedAttribute(rewriter.getStringAttr(unpackAttrName),`。
- **L76 EN**: Executes a call or declaration centered on `rewriter.getUnitAttr`.
  **L76 CN**: 执行以 `rewriter.getUnitAttr` 为核心的调用或声明。
- **L77 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto blkAttr = NamedAttribute(rewriter.getStringAttr(blockAttrName),`.
  **L77 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto blkAttr = NamedAttribute(rewriter.getStringAttr(blockAttrName),`。
- **L78 EN**: Executes a call or declaration centered on `rewriter.getDenseI64ArrayAttr`.
  **L78 CN**: 执行以 `rewriter.getDenseI64ArrayAttr` 为核心的调用或声明。
- **L79 EN**: Continues logic associated with callable symbol `create`.
  **L79 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L80 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, destTy, srcs,`.
  **L80 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, destTy, srcs,`。
- **L81 EN**: Executes a call or declaration centered on `ArrayRef<NamedAttribute>`.
  **L81 CN**: 执行以 `ArrayRef<NamedAttribute>` 为核心的调用或声明。
- **L82 EN**: Returns from the current function with `castOp.getResult(0)`.
  **L82 CN**: 以 `castOp.getResult(0)` 从当前函数返回。
- **L83 EN**: Closes the current lexical scope or compound statement.
  **L83 CN**: 结束当前词法作用域或复合语句块。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L85 EN**: Marks this control path as unreachable.
  **L85 CN**: 将该控制路径标记为不可达。
- **L86 EN**: Returns from the current function with `Value()`.
  **L86 CN**: 以 `Value()` 从当前函数返回。
- **L87 EN**: Closes the current lexical scope or compound statement.
  **L87 CN**: 结束当前词法作用域或复合语句块。
- **L88 EN**: Blank line separating nearby declarations or logic blocks.
  **L88 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L89 EN**: Comment explains nearby logic, invariants, or intent: `Emulate the the pack behavior using extract_strided_slice for VectorType`.
  **L89 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Emulate the the pack behavior using extract_strided_slice for VectorType`。
- **L90 EN**: Comment explains nearby logic, invariants, or intent: `values and unrealized_conversion_cast for TensorDescType values.`.
  **L90 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`values and unrealized_conversion_cast for TensorDescType values.`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<Value> pack(Value src, TypeRange destTypes,`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<Value> pack(Value src, TypeRange destTypes,`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t> blockSize, Location loc,`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t> blockSize, Location loc,`。
- **L93 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const {`.
  **L93 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const {`。
- **L94 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L94 CN**: 开始 `if` 控制流语句并计算其条件。
- **L95 EN**: Returns from the current function with `xegpu::extractVectorsWithShapeFromValue(rewriter, loc, src,`.
  **L95 CN**: 以 `xegpu::extractVectorsWithShapeFromValue(rewriter, loc, src,` 从当前函数返回。
- **L96 EN**: Executes a standalone statement or declaration: `blockSize);`.
  **L96 CN**: 执行一条独立语句或声明：`blockSize);`。

### Lines 97-120

````cpp
    }

    if (isa<xegpu::TensorDescType>(src.getType())) {
      auto attr = NamedAttribute(rewriter.getStringAttr(packAttrName),
                                 rewriter.getUnitAttr());
      auto blkAttr = NamedAttribute(rewriter.getStringAttr(blockAttrName),
                                    rewriter.getDenseI64ArrayAttr(blockSize));
      auto castOp = UnrealizedConversionCastOp::create(
          rewriter, loc, destTypes, src,
          ArrayRef<NamedAttribute>({attr, blkAttr}));
      return castOp.getResults();
    }

    llvm_unreachable("Unexpected src type.");
    return SmallVector<Value>();
  }

  /// Helper to pack operands for DPAS-like operations with early return if
  /// no unrolling is needed.
  SmallVector<Value> packOperandForDpas(Value operand,
                                        ArrayRef<int64_t> blockSize,
                                        Location loc,
                                        PatternRewriter &rewriter) const {
    auto vecType = cast<VectorType>(operand.getType());
````
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L99 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L99 CN**: 开始 `if` 控制流语句并计算其条件。
- **L100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto attr = NamedAttribute(rewriter.getStringAttr(packAttrName),`.
  **L100 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto attr = NamedAttribute(rewriter.getStringAttr(packAttrName),`。
- **L101 EN**: Executes a call or declaration centered on `rewriter.getUnitAttr`.
  **L101 CN**: 执行以 `rewriter.getUnitAttr` 为核心的调用或声明。
- **L102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto blkAttr = NamedAttribute(rewriter.getStringAttr(blockAttrName),`.
  **L102 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto blkAttr = NamedAttribute(rewriter.getStringAttr(blockAttrName),`。
- **L103 EN**: Executes a call or declaration centered on `rewriter.getDenseI64ArrayAttr`.
  **L103 CN**: 执行以 `rewriter.getDenseI64ArrayAttr` 为核心的调用或声明。
- **L104 EN**: Continues logic associated with callable symbol `create`.
  **L104 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, destTypes, src,`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, destTypes, src,`。
- **L106 EN**: Executes a call or declaration centered on `ArrayRef<NamedAttribute>`.
  **L106 CN**: 执行以 `ArrayRef<NamedAttribute>` 为核心的调用或声明。
- **L107 EN**: Returns from the current function with `castOp.getResults()`.
  **L107 CN**: 以 `castOp.getResults()` 从当前函数返回。
- **L108 EN**: Closes the current lexical scope or compound statement.
  **L108 CN**: 结束当前词法作用域或复合语句块。
- **L109 EN**: Blank line separating nearby declarations or logic blocks.
  **L109 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L110 EN**: Marks this control path as unreachable.
  **L110 CN**: 将该控制路径标记为不可达。
- **L111 EN**: Returns from the current function with `SmallVector<Value>()`.
  **L111 CN**: 以 `SmallVector<Value>()` 从当前函数返回。
- **L112 EN**: Closes the current lexical scope or compound statement.
  **L112 CN**: 结束当前词法作用域或复合语句块。
- **L113 EN**: Blank line separating nearby declarations or logic blocks.
  **L113 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L114 EN**: Comment explains nearby logic, invariants, or intent: `Helper to pack operands for DPAS-like operations with early return if`.
  **L114 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper to pack operands for DPAS-like operations with early return if`。
- **L115 EN**: Comment explains nearby logic, invariants, or intent: `no unrolling is needed.`.
  **L115 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`no unrolling is needed.`。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<Value> packOperandForDpas(Value operand,`.
  **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<Value> packOperandForDpas(Value operand,`。
- **L117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t> blockSize,`.
  **L117 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t> blockSize,`。
- **L118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Location loc,`.
  **L118 CN**: 继续一个多行参数列表、初始化器或聚合项：`Location loc,`。
- **L119 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const {`.
  **L119 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const {`。
- **L120 EN**: Initializes variable `vecType` from the right-hand expression.
  **L120 CN**: 使用右侧表达式初始化变量 `vecType`。

### Lines 121-144

````cpp
    std::optional<SmallVector<int64_t>> grids =
        computeShapeRatio(vecType.getShape(), blockSize);
    assert(grids && "Expecting grids to be computed.");
    auto numNewOps = computeProduct(*grids);
    if (numNewOps == 1)
      return SmallVector<Value>({operand});
    VectorType newVecTy =
        vecType.cloneWith(blockSize, vecType.getElementType());
    SmallVector<Type> convertedTypes(numNewOps, newVecTy);
    return pack(operand, convertedTypes, blockSize, loc, rewriter);
  }

private:
  const char *const packAttrName = "__xegpu_blocking_pack__";
  const char *const unpackAttrName = "__xegpu_blocking_unpack__";
  const char *const blockAttrName = "__xegpu_blocking_tile_shape__";

  xegpu::UnrollOptions options;
};

// Generic helper function for unrolling operations with offsets.
//
// Iterates over tile offsets within the tensor descriptor shape and calls
// the provided createOp function for each computed offset. This is used by
````
- **L121 EN**: Continues the surrounding expression or declaration: `std::optional<SmallVector<int64_t>> grids =`.
  **L121 CN**: 继续构造周围的表达式或声明：`std::optional<SmallVector<int64_t>> grids =`。
- **L122 EN**: Executes a call or declaration centered on `computeShapeRatio`.
  **L122 CN**: 执行以 `computeShapeRatio` 为核心的调用或声明。
- **L123 EN**: Checks an internal invariant in debug builds.
  **L123 CN**: 在调试构建中检查内部不变式。
- **L124 EN**: Initializes variable `numNewOps` from the right-hand expression.
  **L124 CN**: 使用右侧表达式初始化变量 `numNewOps`。
- **L125 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L125 CN**: 开始 `if` 控制流语句并计算其条件。
- **L126 EN**: Returns from the current function with `SmallVector<Value>({operand})`.
  **L126 CN**: 以 `SmallVector<Value>({operand})` 从当前函数返回。
- **L127 EN**: Continues the surrounding expression or declaration: `VectorType newVecTy =`.
  **L127 CN**: 继续构造周围的表达式或声明：`VectorType newVecTy =`。
- **L128 EN**: Executes a call or declaration centered on `vecType.cloneWith`.
  **L128 CN**: 执行以 `vecType.cloneWith` 为核心的调用或声明。
- **L129 EN**: Executes a call or declaration centered on `convertedTypes`.
  **L129 CN**: 执行以 `convertedTypes` 为核心的调用或声明。
- **L130 EN**: Returns from the current function with `pack(operand, convertedTypes, blockSize, loc, rewriter)`.
  **L130 CN**: 以 `pack(operand, convertedTypes, blockSize, loc, rewriter)` 从当前函数返回。
- **L131 EN**: Closes the current lexical scope or compound statement.
  **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L133 EN**: Sets the following members to `private` access.
  **L133 CN**: 将后续成员的访问级别设为 `private`。
- **L134 EN**: Initializes variable `packAttrName` from the right-hand expression.
  **L134 CN**: 使用右侧表达式初始化变量 `packAttrName`。
- **L135 EN**: Initializes variable `unpackAttrName` from the right-hand expression.
  **L135 CN**: 使用右侧表达式初始化变量 `unpackAttrName`。
- **L136 EN**: Initializes variable `blockAttrName` from the right-hand expression.
  **L136 CN**: 使用右侧表达式初始化变量 `blockAttrName`。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L138 EN**: Executes a standalone statement or declaration: `xegpu::UnrollOptions options;`.
  **L138 CN**: 执行一条独立语句或声明：`xegpu::UnrollOptions options;`。
- **L139 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L139 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L141 EN**: Comment explains nearby logic, invariants, or intent: `Generic helper function for unrolling operations with offsets.`.
  **L141 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generic helper function for unrolling operations with offsets.`。
- **L142 EN**: Separator comment used for visual grouping.
  **L142 CN**: 用于视觉分组的分隔注释。
- **L143 EN**: Comment explains nearby logic, invariants, or intent: `Iterates over tile offsets within the tensor descriptor shape and calls`.
  **L143 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Iterates over tile offsets within the tensor descriptor shape and calls`。
- **L144 EN**: Comment explains nearby logic, invariants, or intent: `the provided createOp function for each computed offset. This is used by`.
  **L144 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the provided createOp function for each computed offset. This is used by`。

### Lines 145-168

````cpp
// operations like LoadNd, StoreNd, CreateNdDesc, and PrefetchNd when they
// have explicit offsets that need to be adjusted for each unrolled tile.
SmallVector<Value> computeUnrolledOffsets(
    SmallVector<OpFoldResult> mixedOffsets, xegpu::TensorDescType tdescTy,
    ArrayRef<int64_t> targetShape,
    const std::function<Value(SmallVector<OpFoldResult>)> &createOp,
    Location loc, PatternRewriter &rewriter) {
  int64_t rank = tdescTy.getRank();
  ArrayRef<int64_t> shape = tdescTy.getShape();

  auto addi = [&](OpFoldResult a, int64_t b) -> Value {
    std::optional<int64_t> maybeInt = getConstantIntValue(a);
    if (maybeInt) {
      return arith::ConstantIndexOp::create(rewriter, loc, *maybeInt + b);
    } else {
      auto aV = llvm::cast<Value>(a);
      auto bV = arith::ConstantIndexOp::create(rewriter, loc, b);
      return rewriter.createOrFold<arith::AddIOp>(loc, aV, bV);
    }
  };

  SmallVector<OpFoldResult> oldOffsets = llvm::to_vector(
      llvm::drop_begin(mixedOffsets, mixedOffsets.size() - rank));
  auto validIdxes =
````
- **L145 EN**: Comment explains nearby logic, invariants, or intent: `operations like LoadNd, StoreNd, CreateNdDesc, and PrefetchNd when they`.
  **L145 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`operations like LoadNd, StoreNd, CreateNdDesc, and PrefetchNd when they`。
- **L146 EN**: Comment explains nearby logic, invariants, or intent: `have explicit offsets that need to be adjusted for each unrolled tile.`.
  **L146 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`have explicit offsets that need to be adjusted for each unrolled tile.`。
- **L147 EN**: Continues logic associated with callable symbol `computeUnrolledOffsets`.
  **L147 CN**: 继续与可调用符号 `computeUnrolledOffsets` 相关的逻辑。
- **L148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<OpFoldResult> mixedOffsets, xegpu::TensorDescType tdescTy,`.
  **L148 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<OpFoldResult> mixedOffsets, xegpu::TensorDescType tdescTy,`。
- **L149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t> targetShape,`.
  **L149 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t> targetShape,`。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const std::function<Value(SmallVector<OpFoldResult>)> &createOp,`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`const std::function<Value(SmallVector<OpFoldResult>)> &createOp,`。
- **L151 EN**: Continues the surrounding expression or declaration: `Location loc, PatternRewriter &rewriter) {`.
  **L151 CN**: 继续构造周围的表达式或声明：`Location loc, PatternRewriter &rewriter) {`。
- **L152 EN**: Initializes variable `rank` from the right-hand expression.
  **L152 CN**: 使用右侧表达式初始化变量 `rank`。
- **L153 EN**: Initializes variable `shape` from the right-hand expression.
  **L153 CN**: 使用右侧表达式初始化变量 `shape`。
- **L154 EN**: Blank line separating nearby declarations or logic blocks.
  **L154 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L155 EN**: Starts a function, method, lambda, or structured scope: `auto addi = [&](OpFoldResult a, int64_t b) -> Value {`.
  **L155 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto addi = [&](OpFoldResult a, int64_t b) -> Value {`。
- **L156 EN**: Initializes variable `maybeInt` from the right-hand expression.
  **L156 CN**: 使用右侧表达式初始化变量 `maybeInt`。
- **L157 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L157 CN**: 开始 `if` 控制流语句并计算其条件。
- **L158 EN**: Returns from the current function with `arith::ConstantIndexOp::create(rewriter, loc, *maybeInt + b)`.
  **L158 CN**: 以 `arith::ConstantIndexOp::create(rewriter, loc, *maybeInt + b)` 从当前函数返回。
- **L159 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L159 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L160 EN**: Initializes variable `aV` from the right-hand expression.
  **L160 CN**: 使用右侧表达式初始化变量 `aV`。
- **L161 EN**: Initializes variable `bV` from the right-hand expression.
  **L161 CN**: 使用右侧表达式初始化变量 `bV`。
- **L162 EN**: Returns from the current function with `rewriter.createOrFold<arith::AddIOp>(loc, aV, bV)`.
  **L162 CN**: 以 `rewriter.createOrFold<arith::AddIOp>(loc, aV, bV)` 从当前函数返回。
- **L163 EN**: Closes the current lexical scope or compound statement.
  **L163 CN**: 结束当前词法作用域或复合语句块。
- **L164 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L164 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L166 EN**: Continues logic associated with callable symbol `to_vector`.
  **L166 CN**: 继续与可调用符号 `to_vector` 相关的逻辑。
- **L167 EN**: Executes a call or declaration centered on `llvm::drop_begin`.
  **L167 CN**: 执行以 `llvm::drop_begin` 为核心的调用或声明。
- **L168 EN**: Continues the surrounding expression or declaration: `auto validIdxes =`.
  **L168 CN**: 继续构造周围的表达式或声明：`auto validIdxes =`。

### Lines 169-192

````cpp
      llvm::seq<int64_t>(mixedOffsets.size() - rank, mixedOffsets.size());

  SmallVector<Value> newOps;
  for (SmallVector<int64_t> offsets :
       StaticTileOffsetRange(shape, targetShape)) {

    for (auto [idx, oldOff, offset] :
         llvm::zip(validIdxes, oldOffsets, offsets))
      mixedOffsets[idx] = addi(oldOff, offset);

    auto newOp = createOp(mixedOffsets);
    newOps.push_back(newOp);
  }
  return newOps;
}

struct UnrollCreateNdOp : public UnrollPattern<xegpu::CreateNdDescOp> {
  using UnrollPattern<xegpu::CreateNdDescOp>::UnrollPattern;
  LogicalResult matchAndRewrite(xegpu::CreateNdDescOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    xegpu::TensorDescType tdescTy = op.getType();

    std::optional<SmallVector<int64_t>> targetShape = getTargetShape(op);
````
- **L169 EN**: Executes a call or declaration centered on `llvm::seq<int64_t>`.
  **L169 CN**: 执行以 `llvm::seq<int64_t>` 为核心的调用或声明。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L171 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newOps;`.
  **L171 CN**: 执行一条独立语句或声明：`SmallVector<Value> newOps;`。
- **L172 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L172 CN**: 开始 `for` 控制流语句并计算其条件。
- **L173 EN**: Starts a function, method, lambda, or structured scope: `StaticTileOffsetRange(shape, targetShape)) {`.
  **L173 CN**: 开始一个函数、方法、lambda 或结构化作用域：`StaticTileOffsetRange(shape, targetShape)) {`。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L175 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L175 CN**: 开始 `for` 控制流语句并计算其条件。
- **L176 EN**: Continues logic associated with callable symbol `zip`.
  **L176 CN**: 继续与可调用符号 `zip` 相关的逻辑。
- **L177 EN**: Executes a call or declaration centered on `addi`.
  **L177 CN**: 执行以 `addi` 为核心的调用或声明。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L179 EN**: Initializes variable `newOp` from the right-hand expression.
  **L179 CN**: 使用右侧表达式初始化变量 `newOp`。
- **L180 EN**: Executes a call or declaration centered on `newOps.push_back`.
  **L180 CN**: 执行以 `newOps.push_back` 为核心的调用或声明。
- **L181 EN**: Closes the current lexical scope or compound statement.
  **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Returns from the current function with `newOps`.
  **L182 CN**: 以 `newOps` 从当前函数返回。
- **L183 EN**: Closes the current lexical scope or compound statement.
  **L183 CN**: 结束当前词法作用域或复合语句块。
- **L184 EN**: Blank line separating nearby declarations or logic blocks.
  **L184 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L185 EN**: Declares struct `UnrollCreateNdOp`.
  **L185 CN**: 声明 struct `UnrollCreateNdOp`。
- **L186 EN**: Executes a standalone statement or declaration: `using UnrollPattern<xegpu::CreateNdDescOp>::UnrollPattern;`.
  **L186 CN**: 执行一条独立语句或声明：`using UnrollPattern<xegpu::CreateNdDescOp>::UnrollPattern;`。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::CreateNdDescOp op,`.
  **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::CreateNdDescOp op,`。
- **L188 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L188 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L189 EN**: Initializes variable `loc` from the right-hand expression.
  **L189 CN**: 使用右侧表达式初始化变量 `loc`。
- **L190 EN**: Initializes variable `tdescTy` from the right-hand expression.
  **L190 CN**: 使用右侧表达式初始化变量 `tdescTy`。
- **L191 EN**: Blank line separating nearby declarations or logic blocks.
  **L191 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L192 EN**: Initializes variable `targetShape` from the right-hand expression.
  **L192 CN**: 使用右侧表达式初始化变量 `targetShape`。

### Lines 193-216

````cpp
    if (!targetShape)
      return failure();

    SmallVector<Value> newOps;

    auto newTdescTy = getUnrolledTypes(tdescTy, *targetShape)[0];
    auto newOp =
        xegpu::CreateNdDescOp::create(rewriter, loc, newTdescTy, op.getSource(),
                                      op.getMixedSizes(), op.getMixedStrides());
    newOps.push_back(newOp);
    Value castOp = unpack(newOps, tdescTy, *targetShape, loc, rewriter);
    rewriter.replaceOp(op, castOp);

    return success();
  }
};

struct UnrollPrefetchNdOp : public UnrollPattern<xegpu::PrefetchNdOp> {
  using UnrollPattern<xegpu::PrefetchNdOp>::UnrollPattern;
  LogicalResult matchAndRewrite(xegpu::PrefetchNdOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    xegpu::TensorDescType tdescTy = op.getTensorDescType();

````
- **L193 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L193 CN**: 开始 `if` 控制流语句并计算其条件。
- **L194 EN**: Returns from the current function with `failure()`.
  **L194 CN**: 以 `failure()` 从当前函数返回。
- **L195 EN**: Blank line separating nearby declarations or logic blocks.
  **L195 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L196 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newOps;`.
  **L196 CN**: 执行一条独立语句或声明：`SmallVector<Value> newOps;`。
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L198 EN**: Initializes variable `newTdescTy` from the right-hand expression.
  **L198 CN**: 使用右侧表达式初始化变量 `newTdescTy`。
- **L199 EN**: Continues the surrounding expression or declaration: `auto newOp =`.
  **L199 CN**: 继续构造周围的表达式或声明：`auto newOp =`。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::CreateNdDescOp::create(rewriter, loc, newTdescTy, op.getSource(),`.
  **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::CreateNdDescOp::create(rewriter, loc, newTdescTy, op.getSource(),`。
- **L201 EN**: Executes a call or declaration centered on `op.getMixedSizes`.
  **L201 CN**: 执行以 `op.getMixedSizes` 为核心的调用或声明。
- **L202 EN**: Executes a call or declaration centered on `newOps.push_back`.
  **L202 CN**: 执行以 `newOps.push_back` 为核心的调用或声明。
- **L203 EN**: Initializes variable `castOp` from the right-hand expression.
  **L203 CN**: 使用右侧表达式初始化变量 `castOp`。
- **L204 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L204 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L206 EN**: Returns from the current function with `success()`.
  **L206 CN**: 以 `success()` 从当前函数返回。
- **L207 EN**: Closes the current lexical scope or compound statement.
  **L207 CN**: 结束当前词法作用域或复合语句块。
- **L208 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L208 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L209 EN**: Blank line separating nearby declarations or logic blocks.
  **L209 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L210 EN**: Declares struct `UnrollPrefetchNdOp`.
  **L210 CN**: 声明 struct `UnrollPrefetchNdOp`。
- **L211 EN**: Executes a standalone statement or declaration: `using UnrollPattern<xegpu::PrefetchNdOp>::UnrollPattern;`.
  **L211 CN**: 执行一条独立语句或声明：`using UnrollPattern<xegpu::PrefetchNdOp>::UnrollPattern;`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::PrefetchNdOp op,`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::PrefetchNdOp op,`。
- **L213 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L213 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L214 EN**: Initializes variable `loc` from the right-hand expression.
  **L214 CN**: 使用右侧表达式初始化变量 `loc`。
- **L215 EN**: Initializes variable `tdescTy` from the right-hand expression.
  **L215 CN**: 使用右侧表达式初始化变量 `tdescTy`。
- **L216 EN**: Blank line separating nearby declarations or logic blocks.
  **L216 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 217-240

````cpp
    std::optional<SmallVector<int64_t>> targetShape = getTargetShape(op);
    if (!targetShape)
      return failure();

    xegpu::DistributeLayoutAttr layout = op.getLayoutAttr();
    if (layout)
      layout = layout.dropInstData();

    SmallVector<Type> convertedTdescTypes =
        getUnrolledTypes(tdescTy, *targetShape, /*returnSingleType*/ true);

    SmallVector<Value> convertedTdesc = pack(
        op.getTensorDesc(), convertedTdescTypes, *targetShape, loc, rewriter);

    auto createPrefetch = [&](SmallVector<OpFoldResult> offsets) -> Value {
      xegpu::PrefetchNdOp::create(rewriter, loc, convertedTdesc[0], offsets,
                                  op.getL1HintAttr(), op.getL2HintAttr(),
                                  op.getL3HintAttr(), layout);
      // return dummy Value to satisfy function's signature
      return nullptr;
    };

    computeUnrolledOffsets(op.getMixedOffsets(), tdescTy, *targetShape,
                           createPrefetch, loc, rewriter);
````
- **L217 EN**: Initializes variable `targetShape` from the right-hand expression.
  **L217 CN**: 使用右侧表达式初始化变量 `targetShape`。
- **L218 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L218 CN**: 开始 `if` 控制流语句并计算其条件。
- **L219 EN**: Returns from the current function with `failure()`.
  **L219 CN**: 以 `failure()` 从当前函数返回。
- **L220 EN**: Blank line separating nearby declarations or logic blocks.
  **L220 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L221 EN**: Initializes variable `layout` from the right-hand expression.
  **L221 CN**: 使用右侧表达式初始化变量 `layout`。
- **L222 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L222 CN**: 开始 `if` 控制流语句并计算其条件。
- **L223 EN**: Executes a call or declaration centered on `layout.dropInstData`.
  **L223 CN**: 执行以 `layout.dropInstData` 为核心的调用或声明。
- **L224 EN**: Blank line separating nearby declarations or logic blocks.
  **L224 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L225 EN**: Continues the surrounding expression or declaration: `SmallVector<Type> convertedTdescTypes =`.
  **L225 CN**: 继续构造周围的表达式或声明：`SmallVector<Type> convertedTdescTypes =`。
- **L226 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L226 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L227 EN**: Blank line separating nearby declarations or logic blocks.
  **L227 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L228 EN**: Continues logic associated with callable symbol `pack`.
  **L228 CN**: 继续与可调用符号 `pack` 相关的逻辑。
- **L229 EN**: Executes a call or declaration centered on `op.getTensorDesc`.
  **L229 CN**: 执行以 `op.getTensorDesc` 为核心的调用或声明。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L231 EN**: Starts a function, method, lambda, or structured scope: `auto createPrefetch = [&](SmallVector<OpFoldResult> offsets) -> Value {`.
  **L231 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto createPrefetch = [&](SmallVector<OpFoldResult> offsets) -> Value {`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::PrefetchNdOp::create(rewriter, loc, convertedTdesc[0], offsets,`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::PrefetchNdOp::create(rewriter, loc, convertedTdesc[0], offsets,`。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op.getL1HintAttr(), op.getL2HintAttr(),`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`op.getL1HintAttr(), op.getL2HintAttr(),`。
- **L234 EN**: Executes a call or declaration centered on `op.getL3HintAttr`.
  **L234 CN**: 执行以 `op.getL3HintAttr` 为核心的调用或声明。
- **L235 EN**: Comment explains nearby logic, invariants, or intent: `return dummy Value to satisfy function's signature`.
  **L235 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`return dummy Value to satisfy function's signature`。
- **L236 EN**: Returns from the current function with `nullptr`.
  **L236 CN**: 以 `nullptr` 从当前函数返回。
- **L237 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L237 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L238 EN**: Blank line separating nearby declarations or logic blocks.
  **L238 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `computeUnrolledOffsets(op.getMixedOffsets(), tdescTy, *targetShape,`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`computeUnrolledOffsets(op.getMixedOffsets(), tdescTy, *targetShape,`。
- **L240 EN**: Executes a standalone statement or declaration: `createPrefetch, loc, rewriter);`.
  **L240 CN**: 执行一条独立语句或声明：`createPrefetch, loc, rewriter);`。

### Lines 241-264

````cpp

    rewriter.eraseOp(op);
    return success();
  }
};

struct UnrollLoadNdOp : public UnrollPattern<xegpu::LoadNdOp> {
  using UnrollPattern<xegpu::LoadNdOp>::UnrollPattern;
  LogicalResult matchAndRewrite(xegpu::LoadNdOp op,
                                PatternRewriter &rewriter) const override {

    Location loc = op.getLoc();
    VectorType valueTy = op.getType();
    xegpu::TensorDescType tdescTy = op.getTensorDescType();

    std::optional<SmallVector<int64_t>> targetShape = getTargetShape(op);
    if (!targetShape)
      return failure();

    xegpu::DistributeLayoutAttr layout = op.getLayoutAttr();
    if (layout)
      layout = layout.dropInstData();

    Type elemTy = tdescTy.getElementType();
````
- **L241 EN**: Blank line separating nearby declarations or logic blocks.
  **L241 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L242 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L242 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L243 EN**: Returns from the current function with `success()`.
  **L243 CN**: 以 `success()` 从当前函数返回。
- **L244 EN**: Closes the current lexical scope or compound statement.
  **L244 CN**: 结束当前词法作用域或复合语句块。
- **L245 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L245 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L247 EN**: Declares struct `UnrollLoadNdOp`.
  **L247 CN**: 声明 struct `UnrollLoadNdOp`。
- **L248 EN**: Executes a standalone statement or declaration: `using UnrollPattern<xegpu::LoadNdOp>::UnrollPattern;`.
  **L248 CN**: 执行一条独立语句或声明：`using UnrollPattern<xegpu::LoadNdOp>::UnrollPattern;`。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::LoadNdOp op,`.
  **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::LoadNdOp op,`。
- **L250 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L250 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L252 EN**: Initializes variable `loc` from the right-hand expression.
  **L252 CN**: 使用右侧表达式初始化变量 `loc`。
- **L253 EN**: Initializes variable `valueTy` from the right-hand expression.
  **L253 CN**: 使用右侧表达式初始化变量 `valueTy`。
- **L254 EN**: Initializes variable `tdescTy` from the right-hand expression.
  **L254 CN**: 使用右侧表达式初始化变量 `tdescTy`。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L256 EN**: Initializes variable `targetShape` from the right-hand expression.
  **L256 CN**: 使用右侧表达式初始化变量 `targetShape`。
- **L257 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L257 CN**: 开始 `if` 控制流语句并计算其条件。
- **L258 EN**: Returns from the current function with `failure()`.
  **L258 CN**: 以 `failure()` 从当前函数返回。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L260 EN**: Initializes variable `layout` from the right-hand expression.
  **L260 CN**: 使用右侧表达式初始化变量 `layout`。
- **L261 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L261 CN**: 开始 `if` 控制流语句并计算其条件。
- **L262 EN**: Executes a call or declaration centered on `layout.dropInstData`.
  **L262 CN**: 执行以 `layout.dropInstData` 为核心的调用或声明。
- **L263 EN**: Blank line separating nearby declarations or logic blocks.
  **L263 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L264 EN**: Initializes variable `elemTy` from the right-hand expression.
  **L264 CN**: 使用右侧表达式初始化变量 `elemTy`。

### Lines 265-288

````cpp
    VectorType newValueTy = valueTy.cloneWith(*targetShape, elemTy);

    SmallVector<Type> convertedTdescTypes =
        getUnrolledTypes(tdescTy, *targetShape, /*returnSingleType*/ true);

    SmallVector<Value> convertedTdescs = pack(
        op.getTensorDesc(), convertedTdescTypes, *targetShape, loc, rewriter);
    SmallVector<Value> newOps;

    auto createLoad = [&](SmallVector<OpFoldResult> offsets) {
      return xegpu::LoadNdOp::create(
          rewriter, loc, newValueTy, convertedTdescs[0], offsets,
          op.getPackedAttr(), op.getTransposeAttr(), op.getL1HintAttr(),
          op.getL2HintAttr(), op.getL3HintAttr(), layout);
    };
    newOps = computeUnrolledOffsets(op.getMixedOffsets(), tdescTy, *targetShape,
                                    createLoad, loc, rewriter);

    Value castOp = unpack(newOps, op.getType(), *targetShape, loc, rewriter);

    rewriter.replaceOp(op, castOp);
    return success();
  }
};
````
- **L265 EN**: Initializes variable `newValueTy` from the right-hand expression.
  **L265 CN**: 使用右侧表达式初始化变量 `newValueTy`。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L267 EN**: Continues the surrounding expression or declaration: `SmallVector<Type> convertedTdescTypes =`.
  **L267 CN**: 继续构造周围的表达式或声明：`SmallVector<Type> convertedTdescTypes =`。
- **L268 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L268 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L270 EN**: Continues logic associated with callable symbol `pack`.
  **L270 CN**: 继续与可调用符号 `pack` 相关的逻辑。
- **L271 EN**: Executes a call or declaration centered on `op.getTensorDesc`.
  **L271 CN**: 执行以 `op.getTensorDesc` 为核心的调用或声明。
- **L272 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newOps;`.
  **L272 CN**: 执行一条独立语句或声明：`SmallVector<Value> newOps;`。
- **L273 EN**: Blank line separating nearby declarations or logic blocks.
  **L273 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L274 EN**: Starts a function, method, lambda, or structured scope: `auto createLoad = [&](SmallVector<OpFoldResult> offsets) {`.
  **L274 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto createLoad = [&](SmallVector<OpFoldResult> offsets) {`。
- **L275 EN**: Returns from the current function with `xegpu::LoadNdOp::create(`.
  **L275 CN**: 以 `xegpu::LoadNdOp::create(` 从当前函数返回。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, newValueTy, convertedTdescs[0], offsets,`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, newValueTy, convertedTdescs[0], offsets,`。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op.getPackedAttr(), op.getTransposeAttr(), op.getL1HintAttr(),`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`op.getPackedAttr(), op.getTransposeAttr(), op.getL1HintAttr(),`。
- **L278 EN**: Executes a call or declaration centered on `op.getL2HintAttr`.
  **L278 CN**: 执行以 `op.getL2HintAttr` 为核心的调用或声明。
- **L279 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L279 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `newOps = computeUnrolledOffsets(op.getMixedOffsets(), tdescTy, *targetShape,`.
  **L280 CN**: 继续一个多行参数列表、初始化器或聚合项：`newOps = computeUnrolledOffsets(op.getMixedOffsets(), tdescTy, *targetShape,`。
- **L281 EN**: Executes a standalone statement or declaration: `createLoad, loc, rewriter);`.
  **L281 CN**: 执行一条独立语句或声明：`createLoad, loc, rewriter);`。
- **L282 EN**: Blank line separating nearby declarations or logic blocks.
  **L282 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L283 EN**: Initializes variable `castOp` from the right-hand expression.
  **L283 CN**: 使用右侧表达式初始化变量 `castOp`。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L285 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L285 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L286 EN**: Returns from the current function with `success()`.
  **L286 CN**: 以 `success()` 从当前函数返回。
- **L287 EN**: Closes the current lexical scope or compound statement.
  **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L288 CN**: 结束当前声明作用域，例如类、结构体或枚举。

### Lines 289-312

````cpp

struct UnrollStoreNdOp : public UnrollPattern<xegpu::StoreNdOp> {
  using UnrollPattern<xegpu::StoreNdOp>::UnrollPattern;
  LogicalResult matchAndRewrite(xegpu::StoreNdOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    VectorType valueTy = op.getValueType();
    xegpu::TensorDescType tdescTy = op.getTensorDescType();

    std::optional<SmallVector<int64_t>> targetShape = getTargetShape(op);
    if (!targetShape)
      return failure();

    xegpu::DistributeLayoutAttr layout = op.getLayoutAttr();
    if (layout)
      layout = layout.dropInstData();

    SmallVector<Type> convertedValTypes =
        getUnrolledTypes(valueTy, *targetShape);
    SmallVector<Type> convertedTdescTypes =
        getUnrolledTypes(tdescTy, *targetShape, /*returnSingleType*/ true);

    SmallVector<Value> convertedTdescs = pack(
        op.getTensorDesc(), convertedTdescTypes, *targetShape, loc, rewriter);
````
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L290 EN**: Declares struct `UnrollStoreNdOp`.
  **L290 CN**: 声明 struct `UnrollStoreNdOp`。
- **L291 EN**: Executes a standalone statement or declaration: `using UnrollPattern<xegpu::StoreNdOp>::UnrollPattern;`.
  **L291 CN**: 执行一条独立语句或声明：`using UnrollPattern<xegpu::StoreNdOp>::UnrollPattern;`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::StoreNdOp op,`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::StoreNdOp op,`。
- **L293 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L293 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L294 EN**: Initializes variable `loc` from the right-hand expression.
  **L294 CN**: 使用右侧表达式初始化变量 `loc`。
- **L295 EN**: Initializes variable `valueTy` from the right-hand expression.
  **L295 CN**: 使用右侧表达式初始化变量 `valueTy`。
- **L296 EN**: Initializes variable `tdescTy` from the right-hand expression.
  **L296 CN**: 使用右侧表达式初始化变量 `tdescTy`。
- **L297 EN**: Blank line separating nearby declarations or logic blocks.
  **L297 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L298 EN**: Initializes variable `targetShape` from the right-hand expression.
  **L298 CN**: 使用右侧表达式初始化变量 `targetShape`。
- **L299 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L299 CN**: 开始 `if` 控制流语句并计算其条件。
- **L300 EN**: Returns from the current function with `failure()`.
  **L300 CN**: 以 `failure()` 从当前函数返回。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L302 EN**: Initializes variable `layout` from the right-hand expression.
  **L302 CN**: 使用右侧表达式初始化变量 `layout`。
- **L303 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L303 CN**: 开始 `if` 控制流语句并计算其条件。
- **L304 EN**: Executes a call or declaration centered on `layout.dropInstData`.
  **L304 CN**: 执行以 `layout.dropInstData` 为核心的调用或声明。
- **L305 EN**: Blank line separating nearby declarations or logic blocks.
  **L305 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L306 EN**: Continues the surrounding expression or declaration: `SmallVector<Type> convertedValTypes =`.
  **L306 CN**: 继续构造周围的表达式或声明：`SmallVector<Type> convertedValTypes =`。
- **L307 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L307 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L308 EN**: Continues the surrounding expression or declaration: `SmallVector<Type> convertedTdescTypes =`.
  **L308 CN**: 继续构造周围的表达式或声明：`SmallVector<Type> convertedTdescTypes =`。
- **L309 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L309 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L310 EN**: Blank line separating nearby declarations or logic blocks.
  **L310 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L311 EN**: Continues logic associated with callable symbol `pack`.
  **L311 CN**: 继续与可调用符号 `pack` 相关的逻辑。
- **L312 EN**: Executes a call or declaration centered on `op.getTensorDesc`.
  **L312 CN**: 执行以 `op.getTensorDesc` 为核心的调用或声明。

### Lines 313-336

````cpp

    SmallVector<Value> convertedValues =
        pack(op.getValue(), convertedValTypes, *targetShape, loc, rewriter);

    size_t valueIndex = 0;
    auto createStore = [&](SmallVector<OpFoldResult> offsets) {
      xegpu::StoreNdOp::create(rewriter, loc, convertedValues[valueIndex++],
                               convertedTdescs[0], offsets, op.getL1HintAttr(),
                               op.getL2HintAttr(), op.getL3HintAttr(), layout);
      // return dummy Value to satisfy function's signature
      return nullptr;
    };

    computeUnrolledOffsets(op.getMixedOffsets(), tdescTy, *targetShape,
                           createStore, loc, rewriter);

    rewriter.eraseOp(op);
    return success();
  }
};

struct UnrollDpasOp : public UnrollPattern<xegpu::DpasOp> {
  using UnrollPattern<xegpu::DpasOp>::UnrollPattern;
  LogicalResult matchAndRewrite(xegpu::DpasOp op,
````
- **L313 EN**: Blank line separating nearby declarations or logic blocks.
  **L313 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L314 EN**: Continues the surrounding expression or declaration: `SmallVector<Value> convertedValues =`.
  **L314 CN**: 继续构造周围的表达式或声明：`SmallVector<Value> convertedValues =`。
- **L315 EN**: Executes a call or declaration centered on `pack`.
  **L315 CN**: 执行以 `pack` 为核心的调用或声明。
- **L316 EN**: Blank line separating nearby declarations or logic blocks.
  **L316 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L317 EN**: Initializes variable `valueIndex` from the right-hand expression.
  **L317 CN**: 使用右侧表达式初始化变量 `valueIndex`。
- **L318 EN**: Starts a function, method, lambda, or structured scope: `auto createStore = [&](SmallVector<OpFoldResult> offsets) {`.
  **L318 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto createStore = [&](SmallVector<OpFoldResult> offsets) {`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::StoreNdOp::create(rewriter, loc, convertedValues[valueIndex++],`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::StoreNdOp::create(rewriter, loc, convertedValues[valueIndex++],`。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `convertedTdescs[0], offsets, op.getL1HintAttr(),`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`convertedTdescs[0], offsets, op.getL1HintAttr(),`。
- **L321 EN**: Executes a call or declaration centered on `op.getL2HintAttr`.
  **L321 CN**: 执行以 `op.getL2HintAttr` 为核心的调用或声明。
- **L322 EN**: Comment explains nearby logic, invariants, or intent: `return dummy Value to satisfy function's signature`.
  **L322 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`return dummy Value to satisfy function's signature`。
- **L323 EN**: Returns from the current function with `nullptr`.
  **L323 CN**: 以 `nullptr` 从当前函数返回。
- **L324 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L324 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `computeUnrolledOffsets(op.getMixedOffsets(), tdescTy, *targetShape,`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`computeUnrolledOffsets(op.getMixedOffsets(), tdescTy, *targetShape,`。
- **L327 EN**: Executes a standalone statement or declaration: `createStore, loc, rewriter);`.
  **L327 CN**: 执行一条独立语句或声明：`createStore, loc, rewriter);`。
- **L328 EN**: Blank line separating nearby declarations or logic blocks.
  **L328 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L329 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L329 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L330 EN**: Returns from the current function with `success()`.
  **L330 CN**: 以 `success()` 从当前函数返回。
- **L331 EN**: Closes the current lexical scope or compound statement.
  **L331 CN**: 结束当前词法作用域或复合语句块。
- **L332 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L332 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L333 EN**: Blank line separating nearby declarations or logic blocks.
  **L333 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L334 EN**: Declares struct `UnrollDpasOp`.
  **L334 CN**: 声明 struct `UnrollDpasOp`。
- **L335 EN**: Executes a standalone statement or declaration: `using UnrollPattern<xegpu::DpasOp>::UnrollPattern;`.
  **L335 CN**: 执行一条独立语句或声明：`using UnrollPattern<xegpu::DpasOp>::UnrollPattern;`。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::DpasOp op,`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::DpasOp op,`。

### Lines 337-360

````cpp
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();

    std::optional<SmallVector<int64_t>> targetShape = getTargetShape(op);
    if (!targetShape || targetShape->size() != 3)
      return failure();
    auto M = (*targetShape)[0];
    auto K = (*targetShape)[1];
    auto N = (*targetShape)[2];

    int64_t aBlockSize[2] = {M, K};
    int64_t bBlockSize[2] = {K, N};
    int64_t cBlockSize[2] = {M, N};

    auto a = op.getLhs();
    auto b = op.getRhs();
    auto c = op.getAcc();

    SmallVector<Value> aVals = packOperandForDpas(a, aBlockSize, loc, rewriter);
    SmallVector<Value> bVals = packOperandForDpas(b, bBlockSize, loc, rewriter);
    SmallVector<Value> cVals;
    if (c)
      cVals = packOperandForDpas(c, cBlockSize, loc, rewriter);

````
- **L337 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L337 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L338 EN**: Initializes variable `loc` from the right-hand expression.
  **L338 CN**: 使用右侧表达式初始化变量 `loc`。
- **L339 EN**: Blank line separating nearby declarations or logic blocks.
  **L339 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L340 EN**: Initializes variable `targetShape` from the right-hand expression.
  **L340 CN**: 使用右侧表达式初始化变量 `targetShape`。
- **L341 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L341 CN**: 开始 `if` 控制流语句并计算其条件。
- **L342 EN**: Returns from the current function with `failure()`.
  **L342 CN**: 以 `failure()` 从当前函数返回。
- **L343 EN**: Initializes variable `M` from the right-hand expression.
  **L343 CN**: 使用右侧表达式初始化变量 `M`。
- **L344 EN**: Initializes variable `K` from the right-hand expression.
  **L344 CN**: 使用右侧表达式初始化变量 `K`。
- **L345 EN**: Initializes variable `N` from the right-hand expression.
  **L345 CN**: 使用右侧表达式初始化变量 `N`。
- **L346 EN**: Blank line separating nearby declarations or logic blocks.
  **L346 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L347 EN**: Executes a standalone statement or declaration: `int64_t aBlockSize[2] = {M, K};`.
  **L347 CN**: 执行一条独立语句或声明：`int64_t aBlockSize[2] = {M, K};`。
- **L348 EN**: Executes a standalone statement or declaration: `int64_t bBlockSize[2] = {K, N};`.
  **L348 CN**: 执行一条独立语句或声明：`int64_t bBlockSize[2] = {K, N};`。
- **L349 EN**: Executes a standalone statement or declaration: `int64_t cBlockSize[2] = {M, N};`.
  **L349 CN**: 执行一条独立语句或声明：`int64_t cBlockSize[2] = {M, N};`。
- **L350 EN**: Blank line separating nearby declarations or logic blocks.
  **L350 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L351 EN**: Initializes variable `a` from the right-hand expression.
  **L351 CN**: 使用右侧表达式初始化变量 `a`。
- **L352 EN**: Initializes variable `b` from the right-hand expression.
  **L352 CN**: 使用右侧表达式初始化变量 `b`。
- **L353 EN**: Initializes variable `c` from the right-hand expression.
  **L353 CN**: 使用右侧表达式初始化变量 `c`。
- **L354 EN**: Blank line separating nearby declarations or logic blocks.
  **L354 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L355 EN**: Initializes variable `aVals` from the right-hand expression.
  **L355 CN**: 使用右侧表达式初始化变量 `aVals`。
- **L356 EN**: Initializes variable `bVals` from the right-hand expression.
  **L356 CN**: 使用右侧表达式初始化变量 `bVals`。
- **L357 EN**: Executes a standalone statement or declaration: `SmallVector<Value> cVals;`.
  **L357 CN**: 执行一条独立语句或声明：`SmallVector<Value> cVals;`。
- **L358 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L358 CN**: 开始 `if` 控制流语句并计算其条件。
- **L359 EN**: Executes a call or declaration centered on `packOperandForDpas`.
  **L359 CN**: 执行以 `packOperandForDpas` 为核心的调用或声明。
- **L360 EN**: Blank line separating nearby declarations or logic blocks.
  **L360 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 361-384

````cpp
    auto ranges = c ? SmallVector<ValueRange>({aVals, bVals, cVals})
                    : SmallVector<ValueRange>({aVals, bVals});
    if (llvm::any_of(ranges, [](auto &v) { return v.size() == 0; }) ||
        llvm::all_of(ranges, [](auto &v) { return v.size() == 1; }))
      return failure();

    VectorType resultTy = op.getResult().getType();
    auto vecTy = VectorType::get(cBlockSize, resultTy.getElementType());

    auto aShape = a.getType().getShape();
    auto bShape = b.getType().getShape();
    int64_t mIters = aShape[0] / M;
    int64_t kIters = aShape[1] / K;
    int64_t nIters = bShape[1] / N;

    SmallVector<Value> newOps;
    for (int64_t i = 0; i < mIters; ++i) {
      for (int64_t j = 0; j < nIters; ++j) {
        Value tmpC;
        if (c)
          tmpC = cVals[i * nIters + j];

        for (int64_t k = 0; k < kIters; ++k) {
          Value aVec = aVals[i * kIters + k];
````
- **L361 EN**: Continues logic associated with callable symbol `SmallVector<ValueRange>`.
  **L361 CN**: 继续与可调用符号 `SmallVector<ValueRange>` 相关的逻辑。
- **L362 EN**: Executes a call or declaration centered on `SmallVector<ValueRange>`.
  **L362 CN**: 执行以 `SmallVector<ValueRange>` 为核心的调用或声明。
- **L363 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L363 CN**: 开始 `if` 控制流语句并计算其条件。
- **L364 EN**: Continues logic associated with callable symbol `all_of`.
  **L364 CN**: 继续与可调用符号 `all_of` 相关的逻辑。
- **L365 EN**: Returns from the current function with `failure()`.
  **L365 CN**: 以 `failure()` 从当前函数返回。
- **L366 EN**: Blank line separating nearby declarations or logic blocks.
  **L366 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L367 EN**: Initializes variable `resultTy` from the right-hand expression.
  **L367 CN**: 使用右侧表达式初始化变量 `resultTy`。
- **L368 EN**: Initializes variable `vecTy` from the right-hand expression.
  **L368 CN**: 使用右侧表达式初始化变量 `vecTy`。
- **L369 EN**: Blank line separating nearby declarations or logic blocks.
  **L369 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L370 EN**: Initializes variable `aShape` from the right-hand expression.
  **L370 CN**: 使用右侧表达式初始化变量 `aShape`。
- **L371 EN**: Initializes variable `bShape` from the right-hand expression.
  **L371 CN**: 使用右侧表达式初始化变量 `bShape`。
- **L372 EN**: Initializes variable `mIters` from the right-hand expression.
  **L372 CN**: 使用右侧表达式初始化变量 `mIters`。
- **L373 EN**: Initializes variable `kIters` from the right-hand expression.
  **L373 CN**: 使用右侧表达式初始化变量 `kIters`。
- **L374 EN**: Initializes variable `nIters` from the right-hand expression.
  **L374 CN**: 使用右侧表达式初始化变量 `nIters`。
- **L375 EN**: Blank line separating nearby declarations or logic blocks.
  **L375 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L376 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newOps;`.
  **L376 CN**: 执行一条独立语句或声明：`SmallVector<Value> newOps;`。
- **L377 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L377 CN**: 开始 `for` 控制流语句并计算其条件。
- **L378 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L378 CN**: 开始 `for` 控制流语句并计算其条件。
- **L379 EN**: Executes a standalone statement or declaration: `Value tmpC;`.
  **L379 CN**: 执行一条独立语句或声明：`Value tmpC;`。
- **L380 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L380 CN**: 开始 `if` 控制流语句并计算其条件。
- **L381 EN**: Executes a standalone statement or declaration: `tmpC = cVals[i * nIters + j];`.
  **L381 CN**: 执行一条独立语句或声明：`tmpC = cVals[i * nIters + j];`。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L383 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L383 CN**: 开始 `for` 控制流语句并计算其条件。
- **L384 EN**: Initializes variable `aVec` from the right-hand expression.
  **L384 CN**: 使用右侧表达式初始化变量 `aVec`。

### Lines 385-408

````cpp
          Value bVec = bVals[k * nIters + j];
          SmallVector<Value> operands({aVec, bVec});
          if (tmpC)
            operands.push_back(tmpC);

          tmpC =
              xegpu::DpasOp::create(rewriter, loc, vecTy, operands,
                                    xegpu::dropInstDataOnAttrs(op->getAttrs()));
        }
        newOps.push_back(tmpC);
      }
    }
    Value castOp = unpack(newOps, resultTy, cBlockSize, loc, rewriter);
    rewriter.replaceOp(op, castOp);
    return success();
  }
};

struct UnrollDpasMxOp : public UnrollPattern<xegpu::DpasMxOp> {
  using UnrollPattern<xegpu::DpasMxOp>::UnrollPattern;
  LogicalResult matchAndRewrite(xegpu::DpasMxOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();

````
- **L385 EN**: Initializes variable `bVec` from the right-hand expression.
  **L385 CN**: 使用右侧表达式初始化变量 `bVec`。
- **L386 EN**: Executes a call or declaration centered on `operands`.
  **L386 CN**: 执行以 `operands` 为核心的调用或声明。
- **L387 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L387 CN**: 开始 `if` 控制流语句并计算其条件。
- **L388 EN**: Executes a call or declaration centered on `operands.push_back`.
  **L388 CN**: 执行以 `operands.push_back` 为核心的调用或声明。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L390 EN**: Continues the surrounding expression or declaration: `tmpC =`.
  **L390 CN**: 继续构造周围的表达式或声明：`tmpC =`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::DpasOp::create(rewriter, loc, vecTy, operands,`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::DpasOp::create(rewriter, loc, vecTy, operands,`。
- **L392 EN**: Executes a call or declaration centered on `xegpu::dropInstDataOnAttrs`.
  **L392 CN**: 执行以 `xegpu::dropInstDataOnAttrs` 为核心的调用或声明。
- **L393 EN**: Closes the current lexical scope or compound statement.
  **L393 CN**: 结束当前词法作用域或复合语句块。
- **L394 EN**: Executes a call or declaration centered on `newOps.push_back`.
  **L394 CN**: 执行以 `newOps.push_back` 为核心的调用或声明。
- **L395 EN**: Closes the current lexical scope or compound statement.
  **L395 CN**: 结束当前词法作用域或复合语句块。
- **L396 EN**: Closes the current lexical scope or compound statement.
  **L396 CN**: 结束当前词法作用域或复合语句块。
- **L397 EN**: Initializes variable `castOp` from the right-hand expression.
  **L397 CN**: 使用右侧表达式初始化变量 `castOp`。
- **L398 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L398 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L399 EN**: Returns from the current function with `success()`.
  **L399 CN**: 以 `success()` 从当前函数返回。
- **L400 EN**: Closes the current lexical scope or compound statement.
  **L400 CN**: 结束当前词法作用域或复合语句块。
- **L401 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L401 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L403 EN**: Declares struct `UnrollDpasMxOp`.
  **L403 CN**: 声明 struct `UnrollDpasMxOp`。
- **L404 EN**: Executes a standalone statement or declaration: `using UnrollPattern<xegpu::DpasMxOp>::UnrollPattern;`.
  **L404 CN**: 执行一条独立语句或声明：`using UnrollPattern<xegpu::DpasMxOp>::UnrollPattern;`。
- **L405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::DpasMxOp op,`.
  **L405 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::DpasMxOp op,`。
- **L406 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L406 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L407 EN**: Initializes variable `loc` from the right-hand expression.
  **L407 CN**: 使用右侧表达式初始化变量 `loc`。
- **L408 EN**: Blank line separating nearby declarations or logic blocks.
  **L408 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 409-432

````cpp
    std::optional<SmallVector<int64_t>> targetShape = getTargetShape(op);
    if (!targetShape || targetShape->size() != 4)
      return failure();
    auto M = (*targetShape)[0];
    auto K = (*targetShape)[1];
    auto N = (*targetShape)[2];
    auto S = (*targetShape)[3];

    int64_t aBlockSize[2] = {M, K};
    int64_t bBlockSize[2] = {K, N};
    int64_t cBlockSize[2] = {M, N};
    int64_t aScaleBlockSize[2] = {M, S};
    int64_t bScaleBlockSize[2] = {S, N};

    auto a = op.getA();
    auto b = op.getB();
    auto c = op.getAcc();
    auto ascale = dyn_cast<TypedValue<VectorType>>(op.getScaleA());
    auto bscale = dyn_cast<TypedValue<VectorType>>(op.getScaleB());

    SmallVector<Value> aVals = packOperandForDpas(a, aBlockSize, loc, rewriter);
    SmallVector<Value> bVals = packOperandForDpas(b, bBlockSize, loc, rewriter);
    SmallVector<Value> cVals;
    if (c)
````
- **L409 EN**: Initializes variable `targetShape` from the right-hand expression.
  **L409 CN**: 使用右侧表达式初始化变量 `targetShape`。
- **L410 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L410 CN**: 开始 `if` 控制流语句并计算其条件。
- **L411 EN**: Returns from the current function with `failure()`.
  **L411 CN**: 以 `failure()` 从当前函数返回。
- **L412 EN**: Initializes variable `M` from the right-hand expression.
  **L412 CN**: 使用右侧表达式初始化变量 `M`。
- **L413 EN**: Initializes variable `K` from the right-hand expression.
  **L413 CN**: 使用右侧表达式初始化变量 `K`。
- **L414 EN**: Initializes variable `N` from the right-hand expression.
  **L414 CN**: 使用右侧表达式初始化变量 `N`。
- **L415 EN**: Initializes variable `S` from the right-hand expression.
  **L415 CN**: 使用右侧表达式初始化变量 `S`。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L417 EN**: Executes a standalone statement or declaration: `int64_t aBlockSize[2] = {M, K};`.
  **L417 CN**: 执行一条独立语句或声明：`int64_t aBlockSize[2] = {M, K};`。
- **L418 EN**: Executes a standalone statement or declaration: `int64_t bBlockSize[2] = {K, N};`.
  **L418 CN**: 执行一条独立语句或声明：`int64_t bBlockSize[2] = {K, N};`。
- **L419 EN**: Executes a standalone statement or declaration: `int64_t cBlockSize[2] = {M, N};`.
  **L419 CN**: 执行一条独立语句或声明：`int64_t cBlockSize[2] = {M, N};`。
- **L420 EN**: Executes a standalone statement or declaration: `int64_t aScaleBlockSize[2] = {M, S};`.
  **L420 CN**: 执行一条独立语句或声明：`int64_t aScaleBlockSize[2] = {M, S};`。
- **L421 EN**: Executes a standalone statement or declaration: `int64_t bScaleBlockSize[2] = {S, N};`.
  **L421 CN**: 执行一条独立语句或声明：`int64_t bScaleBlockSize[2] = {S, N};`。
- **L422 EN**: Blank line separating nearby declarations or logic blocks.
  **L422 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L423 EN**: Initializes variable `a` from the right-hand expression.
  **L423 CN**: 使用右侧表达式初始化变量 `a`。
- **L424 EN**: Initializes variable `b` from the right-hand expression.
  **L424 CN**: 使用右侧表达式初始化变量 `b`。
- **L425 EN**: Initializes variable `c` from the right-hand expression.
  **L425 CN**: 使用右侧表达式初始化变量 `c`。
- **L426 EN**: Initializes variable `ascale` from the right-hand expression.
  **L426 CN**: 使用右侧表达式初始化变量 `ascale`。
- **L427 EN**: Initializes variable `bscale` from the right-hand expression.
  **L427 CN**: 使用右侧表达式初始化变量 `bscale`。
- **L428 EN**: Blank line separating nearby declarations or logic blocks.
  **L428 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L429 EN**: Initializes variable `aVals` from the right-hand expression.
  **L429 CN**: 使用右侧表达式初始化变量 `aVals`。
- **L430 EN**: Initializes variable `bVals` from the right-hand expression.
  **L430 CN**: 使用右侧表达式初始化变量 `bVals`。
- **L431 EN**: Executes a standalone statement or declaration: `SmallVector<Value> cVals;`.
  **L431 CN**: 执行一条独立语句或声明：`SmallVector<Value> cVals;`。
- **L432 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L432 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 433-456

````cpp
      cVals = packOperandForDpas(c, cBlockSize, loc, rewriter);
    SmallVector<Value> aScaleVals;
    if (ascale)
      aScaleVals = packOperandForDpas(ascale, aScaleBlockSize, loc, rewriter);
    SmallVector<Value> bScaleVals;
    if (bscale)
      bScaleVals = packOperandForDpas(bscale, bScaleBlockSize, loc, rewriter);

    VectorType resultTy = op.getResult().getType();
    auto vecTy = VectorType::get(cBlockSize, resultTy.getElementType());

    auto aShape = a.getType().getShape();
    auto bShape = b.getType().getShape();
    int64_t mIters = aShape[0] / M;
    int64_t kIters = aShape[1] / K;
    int64_t nIters = bShape[1] / N;

    SmallVector<Value> newOps;
    xegpu::DpasMxOp newDpasMxOp;
    for (int64_t i = 0; i < mIters; ++i) {
      for (int64_t j = 0; j < nIters; ++j) {
        Value tmpC;
        if (c)
          tmpC = cVals[i * nIters + j];
````
- **L433 EN**: Executes a call or declaration centered on `packOperandForDpas`.
  **L433 CN**: 执行以 `packOperandForDpas` 为核心的调用或声明。
- **L434 EN**: Executes a standalone statement or declaration: `SmallVector<Value> aScaleVals;`.
  **L434 CN**: 执行一条独立语句或声明：`SmallVector<Value> aScaleVals;`。
- **L435 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L435 CN**: 开始 `if` 控制流语句并计算其条件。
- **L436 EN**: Executes a call or declaration centered on `packOperandForDpas`.
  **L436 CN**: 执行以 `packOperandForDpas` 为核心的调用或声明。
- **L437 EN**: Executes a standalone statement or declaration: `SmallVector<Value> bScaleVals;`.
  **L437 CN**: 执行一条独立语句或声明：`SmallVector<Value> bScaleVals;`。
- **L438 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L438 CN**: 开始 `if` 控制流语句并计算其条件。
- **L439 EN**: Executes a call or declaration centered on `packOperandForDpas`.
  **L439 CN**: 执行以 `packOperandForDpas` 为核心的调用或声明。
- **L440 EN**: Blank line separating nearby declarations or logic blocks.
  **L440 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L441 EN**: Initializes variable `resultTy` from the right-hand expression.
  **L441 CN**: 使用右侧表达式初始化变量 `resultTy`。
- **L442 EN**: Initializes variable `vecTy` from the right-hand expression.
  **L442 CN**: 使用右侧表达式初始化变量 `vecTy`。
- **L443 EN**: Blank line separating nearby declarations or logic blocks.
  **L443 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L444 EN**: Initializes variable `aShape` from the right-hand expression.
  **L444 CN**: 使用右侧表达式初始化变量 `aShape`。
- **L445 EN**: Initializes variable `bShape` from the right-hand expression.
  **L445 CN**: 使用右侧表达式初始化变量 `bShape`。
- **L446 EN**: Initializes variable `mIters` from the right-hand expression.
  **L446 CN**: 使用右侧表达式初始化变量 `mIters`。
- **L447 EN**: Initializes variable `kIters` from the right-hand expression.
  **L447 CN**: 使用右侧表达式初始化变量 `kIters`。
- **L448 EN**: Initializes variable `nIters` from the right-hand expression.
  **L448 CN**: 使用右侧表达式初始化变量 `nIters`。
- **L449 EN**: Blank line separating nearby declarations or logic blocks.
  **L449 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L450 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newOps;`.
  **L450 CN**: 执行一条独立语句或声明：`SmallVector<Value> newOps;`。
- **L451 EN**: Executes a standalone statement or declaration: `xegpu::DpasMxOp newDpasMxOp;`.
  **L451 CN**: 执行一条独立语句或声明：`xegpu::DpasMxOp newDpasMxOp;`。
- **L452 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L452 CN**: 开始 `for` 控制流语句并计算其条件。
- **L453 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L453 CN**: 开始 `for` 控制流语句并计算其条件。
- **L454 EN**: Executes a standalone statement or declaration: `Value tmpC;`.
  **L454 CN**: 执行一条独立语句或声明：`Value tmpC;`。
- **L455 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L455 CN**: 开始 `if` 控制流语句并计算其条件。
- **L456 EN**: Executes a standalone statement or declaration: `tmpC = cVals[i * nIters + j];`.
  **L456 CN**: 执行一条独立语句或声明：`tmpC = cVals[i * nIters + j];`。

### Lines 457-480

````cpp

        for (int64_t k = 0; k < kIters; ++k) {
          Value aVec = aVals[i * kIters + k];
          Value bVec = bVals[k * nIters + j];
          SmallVector<Value> operands({aVec, bVec});
          if (tmpC)
            operands.push_back(tmpC);
          if (ascale)
            operands.push_back(aScaleVals[i * kIters + k]);
          if (bscale)
            operands.push_back(bScaleVals[k * nIters + j]);

          newDpasMxOp = xegpu::DpasMxOp::create(
              rewriter, loc, vecTy, operands,
              xegpu::dropInstDataOnAttrs(op->getAttrs()));
          tmpC = newDpasMxOp.getResult();
        }
        newOps.push_back(newDpasMxOp);
      }
    }
    Value castOp = unpack(newOps, resultTy, cBlockSize, loc, rewriter);
    rewriter.replaceOp(op, castOp);
    return success();
  }
````
- **L457 EN**: Blank line separating nearby declarations or logic blocks.
  **L457 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L458 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L458 CN**: 开始 `for` 控制流语句并计算其条件。
- **L459 EN**: Initializes variable `aVec` from the right-hand expression.
  **L459 CN**: 使用右侧表达式初始化变量 `aVec`。
- **L460 EN**: Initializes variable `bVec` from the right-hand expression.
  **L460 CN**: 使用右侧表达式初始化变量 `bVec`。
- **L461 EN**: Executes a call or declaration centered on `operands`.
  **L461 CN**: 执行以 `operands` 为核心的调用或声明。
- **L462 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L462 CN**: 开始 `if` 控制流语句并计算其条件。
- **L463 EN**: Executes a call or declaration centered on `operands.push_back`.
  **L463 CN**: 执行以 `operands.push_back` 为核心的调用或声明。
- **L464 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L464 CN**: 开始 `if` 控制流语句并计算其条件。
- **L465 EN**: Executes a call or declaration centered on `operands.push_back`.
  **L465 CN**: 执行以 `operands.push_back` 为核心的调用或声明。
- **L466 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L466 CN**: 开始 `if` 控制流语句并计算其条件。
- **L467 EN**: Executes a call or declaration centered on `operands.push_back`.
  **L467 CN**: 执行以 `operands.push_back` 为核心的调用或声明。
- **L468 EN**: Blank line separating nearby declarations or logic blocks.
  **L468 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L469 EN**: Continues logic associated with callable symbol `create`.
  **L469 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, vecTy, operands,`.
  **L470 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, vecTy, operands,`。
- **L471 EN**: Executes a call or declaration centered on `xegpu::dropInstDataOnAttrs`.
  **L471 CN**: 执行以 `xegpu::dropInstDataOnAttrs` 为核心的调用或声明。
- **L472 EN**: Executes a call or declaration centered on `newDpasMxOp.getResult`.
  **L472 CN**: 执行以 `newDpasMxOp.getResult` 为核心的调用或声明。
- **L473 EN**: Closes the current lexical scope or compound statement.
  **L473 CN**: 结束当前词法作用域或复合语句块。
- **L474 EN**: Executes a call or declaration centered on `newOps.push_back`.
  **L474 CN**: 执行以 `newOps.push_back` 为核心的调用或声明。
- **L475 EN**: Closes the current lexical scope or compound statement.
  **L475 CN**: 结束当前词法作用域或复合语句块。
- **L476 EN**: Closes the current lexical scope or compound statement.
  **L476 CN**: 结束当前词法作用域或复合语句块。
- **L477 EN**: Initializes variable `castOp` from the right-hand expression.
  **L477 CN**: 使用右侧表达式初始化变量 `castOp`。
- **L478 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L478 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L479 EN**: Returns from the current function with `success()`.
  **L479 CN**: 以 `success()` 从当前函数返回。
- **L480 EN**: Closes the current lexical scope or compound statement.
  **L480 CN**: 结束当前词法作用域或复合语句块。

### Lines 481-504

````cpp
};

/// This pattern handles the unrolling of LoadGatherOp with offsets (gathered
/// load).
/// It unrolls the offsets and mask operands accordingly, and creates multiple
/// LoadGatherOp with the unrolled operands.
struct UnrollLoadGatherOp : public UnrollPattern<xegpu::LoadGatherOp> {
  using UnrollPattern<xegpu::LoadGatherOp>::UnrollPattern;
  LogicalResult matchAndRewrite(xegpu::LoadGatherOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    VectorType valueTy = llvm::dyn_cast<VectorType>(op.getType());
    Value offsets = op.getOffsets();
    Value mask = op.getMask();

    std::optional<SmallVector<int64_t>> targetShape = getTargetShape(op);
    if (!targetShape)
      return failure();

    SmallVector<int64_t> targetMaskShape(*targetShape);
    int64_t chunkSize = 1;
    if (auto chunkSizeAttr = op->getAttr("chunk_size")) {
      if (auto intAttr = llvm::dyn_cast<IntegerAttr>(chunkSizeAttr))
        chunkSize = intAttr.getInt();
````
- **L481 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L481 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L483 EN**: Comment explains nearby logic, invariants, or intent: `This pattern handles the unrolling of LoadGatherOp with offsets (gathered`.
  **L483 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern handles the unrolling of LoadGatherOp with offsets (gathered`。
- **L484 EN**: Comment explains nearby logic, invariants, or intent: `load).`.
  **L484 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`load).`。
- **L485 EN**: Comment explains nearby logic, invariants, or intent: `It unrolls the offsets and mask operands accordingly, and creates multiple`.
  **L485 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`It unrolls the offsets and mask operands accordingly, and creates multiple`。
- **L486 EN**: Comment explains nearby logic, invariants, or intent: `LoadGatherOp with the unrolled operands.`.
  **L486 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LoadGatherOp with the unrolled operands.`。
- **L487 EN**: Declares struct `UnrollLoadGatherOp`.
  **L487 CN**: 声明 struct `UnrollLoadGatherOp`。
- **L488 EN**: Executes a standalone statement or declaration: `using UnrollPattern<xegpu::LoadGatherOp>::UnrollPattern;`.
  **L488 CN**: 执行一条独立语句或声明：`using UnrollPattern<xegpu::LoadGatherOp>::UnrollPattern;`。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::LoadGatherOp op,`.
  **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::LoadGatherOp op,`。
- **L490 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L490 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L491 EN**: Initializes variable `loc` from the right-hand expression.
  **L491 CN**: 使用右侧表达式初始化变量 `loc`。
- **L492 EN**: Initializes variable `valueTy` from the right-hand expression.
  **L492 CN**: 使用右侧表达式初始化变量 `valueTy`。
- **L493 EN**: Initializes variable `offsets` from the right-hand expression.
  **L493 CN**: 使用右侧表达式初始化变量 `offsets`。
- **L494 EN**: Initializes variable `mask` from the right-hand expression.
  **L494 CN**: 使用右侧表达式初始化变量 `mask`。
- **L495 EN**: Blank line separating nearby declarations or logic blocks.
  **L495 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L496 EN**: Initializes variable `targetShape` from the right-hand expression.
  **L496 CN**: 使用右侧表达式初始化变量 `targetShape`。
- **L497 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L497 CN**: 开始 `if` 控制流语句并计算其条件。
- **L498 EN**: Returns from the current function with `failure()`.
  **L498 CN**: 以 `failure()` 从当前函数返回。
- **L499 EN**: Blank line separating nearby declarations or logic blocks.
  **L499 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L500 EN**: Executes a call or declaration centered on `targetMaskShape`.
  **L500 CN**: 执行以 `targetMaskShape` 为核心的调用或声明。
- **L501 EN**: Initializes variable `chunkSize` from the right-hand expression.
  **L501 CN**: 使用右侧表达式初始化变量 `chunkSize`。
- **L502 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L502 CN**: 开始 `if` 控制流语句并计算其条件。
- **L503 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L503 CN**: 开始 `if` 控制流语句并计算其条件。
- **L504 EN**: Executes a call or declaration centered on `intAttr.getInt`.
  **L504 CN**: 执行以 `intAttr.getInt` 为核心的调用或声明。

### Lines 505-528

````cpp
    }

    // Unroll mask and offsets with correct shape
    VectorType maskTy = llvm::dyn_cast<VectorType>(mask.getType());
    VectorType offsetsTy = llvm::dyn_cast<VectorType>(offsets.getType());
    Type elemTy = valueTy.getElementType();
    VectorType newValueTy = VectorType::get(*targetShape, elemTy);

    SmallVector<Type> convertedMaskTypes;
    SmallVector<Value> convertedMasks;
    SmallVector<Type> convertedOffsetTypes;
    SmallVector<Value> convertedOffsets;

    if (chunkSize > 1) {
      // For chunked loads, mask and offsets have one less dimension
      targetMaskShape.pop_back();
      int64_t blockedChunkSize = targetShape->back();
      int64_t numNewChunks = chunkSize / blockedChunkSize;
      chunkSize = blockedChunkSize;

      convertedMaskTypes = getUnrolledTypes(maskTy, targetMaskShape);
      convertedOffsetTypes = getUnrolledTypes(offsetsTy, targetMaskShape);

      SmallVector<Value> convertedMasksBase =
````
- **L505 EN**: Closes the current lexical scope or compound statement.
  **L505 CN**: 结束当前词法作用域或复合语句块。
- **L506 EN**: Blank line separating nearby declarations or logic blocks.
  **L506 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L507 EN**: Comment explains nearby logic, invariants, or intent: `Unroll mask and offsets with correct shape`.
  **L507 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unroll mask and offsets with correct shape`。
- **L508 EN**: Initializes variable `maskTy` from the right-hand expression.
  **L508 CN**: 使用右侧表达式初始化变量 `maskTy`。
- **L509 EN**: Initializes variable `offsetsTy` from the right-hand expression.
  **L509 CN**: 使用右侧表达式初始化变量 `offsetsTy`。
- **L510 EN**: Initializes variable `elemTy` from the right-hand expression.
  **L510 CN**: 使用右侧表达式初始化变量 `elemTy`。
- **L511 EN**: Initializes variable `newValueTy` from the right-hand expression.
  **L511 CN**: 使用右侧表达式初始化变量 `newValueTy`。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L513 EN**: Executes a standalone statement or declaration: `SmallVector<Type> convertedMaskTypes;`.
  **L513 CN**: 执行一条独立语句或声明：`SmallVector<Type> convertedMaskTypes;`。
- **L514 EN**: Executes a standalone statement or declaration: `SmallVector<Value> convertedMasks;`.
  **L514 CN**: 执行一条独立语句或声明：`SmallVector<Value> convertedMasks;`。
- **L515 EN**: Executes a standalone statement or declaration: `SmallVector<Type> convertedOffsetTypes;`.
  **L515 CN**: 执行一条独立语句或声明：`SmallVector<Type> convertedOffsetTypes;`。
- **L516 EN**: Executes a standalone statement or declaration: `SmallVector<Value> convertedOffsets;`.
  **L516 CN**: 执行一条独立语句或声明：`SmallVector<Value> convertedOffsets;`。
- **L517 EN**: Blank line separating nearby declarations or logic blocks.
  **L517 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L518 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L518 CN**: 开始 `if` 控制流语句并计算其条件。
- **L519 EN**: Comment explains nearby logic, invariants, or intent: `For chunked loads, mask and offsets have one less dimension`.
  **L519 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For chunked loads, mask and offsets have one less dimension`。
- **L520 EN**: Executes a call or declaration centered on `targetMaskShape.pop_back`.
  **L520 CN**: 执行以 `targetMaskShape.pop_back` 为核心的调用或声明。
- **L521 EN**: Initializes variable `blockedChunkSize` from the right-hand expression.
  **L521 CN**: 使用右侧表达式初始化变量 `blockedChunkSize`。
- **L522 EN**: Initializes variable `numNewChunks` from the right-hand expression.
  **L522 CN**: 使用右侧表达式初始化变量 `numNewChunks`。
- **L523 EN**: Executes a standalone statement or declaration: `chunkSize = blockedChunkSize;`.
  **L523 CN**: 执行一条独立语句或声明：`chunkSize = blockedChunkSize;`。
- **L524 EN**: Blank line separating nearby declarations or logic blocks.
  **L524 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L525 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L525 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L526 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L526 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L528 EN**: Continues the surrounding expression or declaration: `SmallVector<Value> convertedMasksBase =`.
  **L528 CN**: 继续构造周围的表达式或声明：`SmallVector<Value> convertedMasksBase =`。

### Lines 529-552

````cpp
          pack(mask, convertedMaskTypes, targetMaskShape, loc, rewriter);
      SmallVector<Value> convertedOffsetsBase =
          pack(offsets, convertedOffsetTypes, targetMaskShape, loc, rewriter);

      for (auto maskVal : convertedMasksBase)
        convertedMasks.append(numNewChunks, maskVal);

      for (auto [baseOffset, offsetType] :
           llvm::zip(convertedOffsetsBase, convertedOffsetTypes)) {
        for (int64_t i = 0; i < numNewChunks; ++i) {
          Value inc = arith::ConstantIndexOp::create(rewriter, loc,
                                                     i * blockedChunkSize);
          Value incVec =
              vector::BroadcastOp::create(rewriter, loc, offsetType, inc);
          Value offsetVal =
              arith::AddIOp::create(rewriter, loc, baseOffset, incVec);
          convertedOffsets.push_back(offsetVal);
        }
      }
    } else {
      convertedMaskTypes = getUnrolledTypes(maskTy, targetMaskShape);
      convertedMasks =
          pack(mask, convertedMaskTypes, targetMaskShape, loc, rewriter);

````
- **L529 EN**: Executes a call or declaration centered on `pack`.
  **L529 CN**: 执行以 `pack` 为核心的调用或声明。
- **L530 EN**: Continues the surrounding expression or declaration: `SmallVector<Value> convertedOffsetsBase =`.
  **L530 CN**: 继续构造周围的表达式或声明：`SmallVector<Value> convertedOffsetsBase =`。
- **L531 EN**: Executes a call or declaration centered on `pack`.
  **L531 CN**: 执行以 `pack` 为核心的调用或声明。
- **L532 EN**: Blank line separating nearby declarations or logic blocks.
  **L532 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L533 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L533 CN**: 开始 `for` 控制流语句并计算其条件。
- **L534 EN**: Executes a call or declaration centered on `convertedMasks.append`.
  **L534 CN**: 执行以 `convertedMasks.append` 为核心的调用或声明。
- **L535 EN**: Blank line separating nearby declarations or logic blocks.
  **L535 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L536 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L536 CN**: 开始 `for` 控制流语句并计算其条件。
- **L537 EN**: Starts a function, method, lambda, or structured scope: `llvm::zip(convertedOffsetsBase, convertedOffsetTypes)) {`.
  **L537 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::zip(convertedOffsetsBase, convertedOffsetTypes)) {`。
- **L538 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L538 CN**: 开始 `for` 控制流语句并计算其条件。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value inc = arith::ConstantIndexOp::create(rewriter, loc,`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value inc = arith::ConstantIndexOp::create(rewriter, loc,`。
- **L540 EN**: Executes a standalone statement or declaration: `i * blockedChunkSize);`.
  **L540 CN**: 执行一条独立语句或声明：`i * blockedChunkSize);`。
- **L541 EN**: Continues the surrounding expression or declaration: `Value incVec =`.
  **L541 CN**: 继续构造周围的表达式或声明：`Value incVec =`。
- **L542 EN**: Executes a call or declaration centered on `vector::BroadcastOp::create`.
  **L542 CN**: 执行以 `vector::BroadcastOp::create` 为核心的调用或声明。
- **L543 EN**: Continues the surrounding expression or declaration: `Value offsetVal =`.
  **L543 CN**: 继续构造周围的表达式或声明：`Value offsetVal =`。
- **L544 EN**: Executes a call or declaration centered on `arith::AddIOp::create`.
  **L544 CN**: 执行以 `arith::AddIOp::create` 为核心的调用或声明。
- **L545 EN**: Executes a call or declaration centered on `convertedOffsets.push_back`.
  **L545 CN**: 执行以 `convertedOffsets.push_back` 为核心的调用或声明。
- **L546 EN**: Closes the current lexical scope or compound statement.
  **L546 CN**: 结束当前词法作用域或复合语句块。
- **L547 EN**: Closes the current lexical scope or compound statement.
  **L547 CN**: 结束当前词法作用域或复合语句块。
- **L548 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L548 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L549 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L549 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L550 EN**: Continues the surrounding expression or declaration: `convertedMasks =`.
  **L550 CN**: 继续构造周围的表达式或声明：`convertedMasks =`。
- **L551 EN**: Executes a call or declaration centered on `pack`.
  **L551 CN**: 执行以 `pack` 为核心的调用或声明。
- **L552 EN**: Blank line separating nearby declarations or logic blocks.
  **L552 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 553-576

````cpp
      convertedOffsetTypes = getUnrolledTypes(offsetsTy, *targetShape);
      convertedOffsets =
          pack(offsets, convertedOffsetTypes, *targetShape, loc, rewriter);
    }

    auto layout = op.getLayoutAttr();
    if (layout)
      layout = layout.dropInstData();

    SmallVector<Value> newOps;
    for (auto [o, m] : llvm::zip(convertedOffsets, convertedMasks)) {
      auto newOp = xegpu::LoadGatherOp::create(
          rewriter, loc, newValueTy, op.getSource(), o, m,
          rewriter.getI64IntegerAttr(chunkSize), op.getL1HintAttr(),
          op.getL2HintAttr(), op.getL3HintAttr(), layout);
      newOps.push_back(newOp);
    }

    Value castOp = unpack(newOps, op.getType(), *targetShape, loc, rewriter);
    rewriter.replaceOp(op, castOp);
    return success();
  }
};

````
- **L553 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L553 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L554 EN**: Continues the surrounding expression or declaration: `convertedOffsets =`.
  **L554 CN**: 继续构造周围的表达式或声明：`convertedOffsets =`。
- **L555 EN**: Executes a call or declaration centered on `pack`.
  **L555 CN**: 执行以 `pack` 为核心的调用或声明。
- **L556 EN**: Closes the current lexical scope or compound statement.
  **L556 CN**: 结束当前词法作用域或复合语句块。
- **L557 EN**: Blank line separating nearby declarations or logic blocks.
  **L557 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L558 EN**: Initializes variable `layout` from the right-hand expression.
  **L558 CN**: 使用右侧表达式初始化变量 `layout`。
- **L559 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L559 CN**: 开始 `if` 控制流语句并计算其条件。
- **L560 EN**: Executes a call or declaration centered on `layout.dropInstData`.
  **L560 CN**: 执行以 `layout.dropInstData` 为核心的调用或声明。
- **L561 EN**: Blank line separating nearby declarations or logic blocks.
  **L561 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L562 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newOps;`.
  **L562 CN**: 执行一条独立语句或声明：`SmallVector<Value> newOps;`。
- **L563 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L563 CN**: 开始 `for` 控制流语句并计算其条件。
- **L564 EN**: Continues logic associated with callable symbol `create`.
  **L564 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, newValueTy, op.getSource(), o, m,`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, newValueTy, op.getSource(), o, m,`。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter.getI64IntegerAttr(chunkSize), op.getL1HintAttr(),`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter.getI64IntegerAttr(chunkSize), op.getL1HintAttr(),`。
- **L567 EN**: Executes a call or declaration centered on `op.getL2HintAttr`.
  **L567 CN**: 执行以 `op.getL2HintAttr` 为核心的调用或声明。
- **L568 EN**: Executes a call or declaration centered on `newOps.push_back`.
  **L568 CN**: 执行以 `newOps.push_back` 为核心的调用或声明。
- **L569 EN**: Closes the current lexical scope or compound statement.
  **L569 CN**: 结束当前词法作用域或复合语句块。
- **L570 EN**: Blank line separating nearby declarations or logic blocks.
  **L570 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L571 EN**: Initializes variable `castOp` from the right-hand expression.
  **L571 CN**: 使用右侧表达式初始化变量 `castOp`。
- **L572 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L572 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L573 EN**: Returns from the current function with `success()`.
  **L573 CN**: 以 `success()` 从当前函数返回。
- **L574 EN**: Closes the current lexical scope or compound statement.
  **L574 CN**: 结束当前词法作用域或复合语句块。
- **L575 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L575 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L576 EN**: Blank line separating nearby declarations or logic blocks.
  **L576 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 577-600

````cpp
/// This pattern handles the unrolling of StoreScatterOp with offsets (scattered
/// store).
/// It unrolls the offsets and mask operands accordingly, and creates multiple
/// StoreScatterOp with the unrolled operands.
struct UnrollStoreScatterOp : public UnrollPattern<xegpu::StoreScatterOp> {
  using UnrollPattern<xegpu::StoreScatterOp>::UnrollPattern;
  LogicalResult matchAndRewrite(xegpu::StoreScatterOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    VectorType valueTy = llvm::dyn_cast<VectorType>(op.getValue().getType());
    Value offsets = op.getOffsets();
    Value mask = op.getMask();

    std::optional<SmallVector<int64_t>> targetShape = getTargetShape(op);
    if (!targetShape)
      return failure();

    int64_t chunkSize = 1;
    if (auto chunkSizeAttr = op->getAttr("chunk_size")) {
      if (auto intAttr = llvm::dyn_cast<IntegerAttr>(chunkSizeAttr))
        chunkSize = intAttr.getInt();
    }

    SmallVector<int64_t> targetMaskShape(*targetShape);
````
- **L577 EN**: Comment explains nearby logic, invariants, or intent: `This pattern handles the unrolling of StoreScatterOp with offsets (scattered`.
  **L577 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern handles the unrolling of StoreScatterOp with offsets (scattered`。
- **L578 EN**: Comment explains nearby logic, invariants, or intent: `store).`.
  **L578 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`store).`。
- **L579 EN**: Comment explains nearby logic, invariants, or intent: `It unrolls the offsets and mask operands accordingly, and creates multiple`.
  **L579 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`It unrolls the offsets and mask operands accordingly, and creates multiple`。
- **L580 EN**: Comment explains nearby logic, invariants, or intent: `StoreScatterOp with the unrolled operands.`.
  **L580 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`StoreScatterOp with the unrolled operands.`。
- **L581 EN**: Declares struct `UnrollStoreScatterOp`.
  **L581 CN**: 声明 struct `UnrollStoreScatterOp`。
- **L582 EN**: Executes a standalone statement or declaration: `using UnrollPattern<xegpu::StoreScatterOp>::UnrollPattern;`.
  **L582 CN**: 执行一条独立语句或声明：`using UnrollPattern<xegpu::StoreScatterOp>::UnrollPattern;`。
- **L583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::StoreScatterOp op,`.
  **L583 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::StoreScatterOp op,`。
- **L584 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L584 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L585 EN**: Initializes variable `loc` from the right-hand expression.
  **L585 CN**: 使用右侧表达式初始化变量 `loc`。
- **L586 EN**: Initializes variable `valueTy` from the right-hand expression.
  **L586 CN**: 使用右侧表达式初始化变量 `valueTy`。
- **L587 EN**: Initializes variable `offsets` from the right-hand expression.
  **L587 CN**: 使用右侧表达式初始化变量 `offsets`。
- **L588 EN**: Initializes variable `mask` from the right-hand expression.
  **L588 CN**: 使用右侧表达式初始化变量 `mask`。
- **L589 EN**: Blank line separating nearby declarations or logic blocks.
  **L589 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L590 EN**: Initializes variable `targetShape` from the right-hand expression.
  **L590 CN**: 使用右侧表达式初始化变量 `targetShape`。
- **L591 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L591 CN**: 开始 `if` 控制流语句并计算其条件。
- **L592 EN**: Returns from the current function with `failure()`.
  **L592 CN**: 以 `failure()` 从当前函数返回。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L594 EN**: Initializes variable `chunkSize` from the right-hand expression.
  **L594 CN**: 使用右侧表达式初始化变量 `chunkSize`。
- **L595 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L595 CN**: 开始 `if` 控制流语句并计算其条件。
- **L596 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L596 CN**: 开始 `if` 控制流语句并计算其条件。
- **L597 EN**: Executes a call or declaration centered on `intAttr.getInt`.
  **L597 CN**: 执行以 `intAttr.getInt` 为核心的调用或声明。
- **L598 EN**: Closes the current lexical scope or compound statement.
  **L598 CN**: 结束当前词法作用域或复合语句块。
- **L599 EN**: Blank line separating nearby declarations or logic blocks.
  **L599 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L600 EN**: Executes a call or declaration centered on `targetMaskShape`.
  **L600 CN**: 执行以 `targetMaskShape` 为核心的调用或声明。

### Lines 601-624

````cpp
    VectorType maskTy = llvm::dyn_cast<VectorType>(mask.getType());
    VectorType offsetsTy = llvm::dyn_cast<VectorType>(offsets.getType());

    SmallVector<Type> convertedMaskTypes;
    SmallVector<Value> convertedMasks;
    SmallVector<Type> convertedOffsetTypes;
    SmallVector<Value> convertedOffsets;

    if (chunkSize > 1) {
      targetMaskShape.pop_back();
      int64_t blockedChunkSize = targetShape->back();
      int64_t numNewChunks = chunkSize / blockedChunkSize;
      chunkSize = blockedChunkSize;

      convertedMaskTypes = getUnrolledTypes(maskTy, targetMaskShape);
      convertedOffsetTypes = getUnrolledTypes(offsetsTy, targetMaskShape);

      SmallVector<Value> convertedMasksBase =
          pack(mask, convertedMaskTypes, targetMaskShape, loc, rewriter);
      SmallVector<Value> convertedOffsetsBase =
          pack(offsets, convertedOffsetTypes, targetMaskShape, loc, rewriter);

      for (auto maskVal : convertedMasksBase)
        convertedMasks.append(numNewChunks, maskVal);
````
- **L601 EN**: Initializes variable `maskTy` from the right-hand expression.
  **L601 CN**: 使用右侧表达式初始化变量 `maskTy`。
- **L602 EN**: Initializes variable `offsetsTy` from the right-hand expression.
  **L602 CN**: 使用右侧表达式初始化变量 `offsetsTy`。
- **L603 EN**: Blank line separating nearby declarations or logic blocks.
  **L603 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L604 EN**: Executes a standalone statement or declaration: `SmallVector<Type> convertedMaskTypes;`.
  **L604 CN**: 执行一条独立语句或声明：`SmallVector<Type> convertedMaskTypes;`。
- **L605 EN**: Executes a standalone statement or declaration: `SmallVector<Value> convertedMasks;`.
  **L605 CN**: 执行一条独立语句或声明：`SmallVector<Value> convertedMasks;`。
- **L606 EN**: Executes a standalone statement or declaration: `SmallVector<Type> convertedOffsetTypes;`.
  **L606 CN**: 执行一条独立语句或声明：`SmallVector<Type> convertedOffsetTypes;`。
- **L607 EN**: Executes a standalone statement or declaration: `SmallVector<Value> convertedOffsets;`.
  **L607 CN**: 执行一条独立语句或声明：`SmallVector<Value> convertedOffsets;`。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L609 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L609 CN**: 开始 `if` 控制流语句并计算其条件。
- **L610 EN**: Executes a call or declaration centered on `targetMaskShape.pop_back`.
  **L610 CN**: 执行以 `targetMaskShape.pop_back` 为核心的调用或声明。
- **L611 EN**: Initializes variable `blockedChunkSize` from the right-hand expression.
  **L611 CN**: 使用右侧表达式初始化变量 `blockedChunkSize`。
- **L612 EN**: Initializes variable `numNewChunks` from the right-hand expression.
  **L612 CN**: 使用右侧表达式初始化变量 `numNewChunks`。
- **L613 EN**: Executes a standalone statement or declaration: `chunkSize = blockedChunkSize;`.
  **L613 CN**: 执行一条独立语句或声明：`chunkSize = blockedChunkSize;`。
- **L614 EN**: Blank line separating nearby declarations or logic blocks.
  **L614 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L615 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L615 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L616 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L616 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L618 EN**: Continues the surrounding expression or declaration: `SmallVector<Value> convertedMasksBase =`.
  **L618 CN**: 继续构造周围的表达式或声明：`SmallVector<Value> convertedMasksBase =`。
- **L619 EN**: Executes a call or declaration centered on `pack`.
  **L619 CN**: 执行以 `pack` 为核心的调用或声明。
- **L620 EN**: Continues the surrounding expression or declaration: `SmallVector<Value> convertedOffsetsBase =`.
  **L620 CN**: 继续构造周围的表达式或声明：`SmallVector<Value> convertedOffsetsBase =`。
- **L621 EN**: Executes a call or declaration centered on `pack`.
  **L621 CN**: 执行以 `pack` 为核心的调用或声明。
- **L622 EN**: Blank line separating nearby declarations or logic blocks.
  **L622 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L623 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L623 CN**: 开始 `for` 控制流语句并计算其条件。
- **L624 EN**: Executes a call or declaration centered on `convertedMasks.append`.
  **L624 CN**: 执行以 `convertedMasks.append` 为核心的调用或声明。

### Lines 625-648

````cpp

      for (auto [baseOffset, offsetType] :
           llvm::zip(convertedOffsetsBase, convertedOffsetTypes)) {
        for (int64_t i = 0; i < numNewChunks; ++i) {
          Value inc = arith::ConstantIndexOp::create(rewriter, loc,
                                                     i * blockedChunkSize);
          Value incVec =
              vector::BroadcastOp::create(rewriter, loc, offsetType, inc);
          Value offsetVal =
              arith::AddIOp::create(rewriter, loc, baseOffset, incVec);
          convertedOffsets.push_back(offsetVal);
        }
      }
    } else {
      convertedMaskTypes = getUnrolledTypes(maskTy, targetMaskShape);
      convertedMasks =
          pack(mask, convertedMaskTypes, targetMaskShape, loc, rewriter);

      convertedOffsetTypes = getUnrolledTypes(offsetsTy, *targetShape);
      convertedOffsets =
          pack(offsets, convertedOffsetTypes, *targetShape, loc, rewriter);
    }

    SmallVector<Type> convertedValTypes =
````
- **L625 EN**: Blank line separating nearby declarations or logic blocks.
  **L625 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L626 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L626 CN**: 开始 `for` 控制流语句并计算其条件。
- **L627 EN**: Starts a function, method, lambda, or structured scope: `llvm::zip(convertedOffsetsBase, convertedOffsetTypes)) {`.
  **L627 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::zip(convertedOffsetsBase, convertedOffsetTypes)) {`。
- **L628 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L628 CN**: 开始 `for` 控制流语句并计算其条件。
- **L629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value inc = arith::ConstantIndexOp::create(rewriter, loc,`.
  **L629 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value inc = arith::ConstantIndexOp::create(rewriter, loc,`。
- **L630 EN**: Executes a standalone statement or declaration: `i * blockedChunkSize);`.
  **L630 CN**: 执行一条独立语句或声明：`i * blockedChunkSize);`。
- **L631 EN**: Continues the surrounding expression or declaration: `Value incVec =`.
  **L631 CN**: 继续构造周围的表达式或声明：`Value incVec =`。
- **L632 EN**: Executes a call or declaration centered on `vector::BroadcastOp::create`.
  **L632 CN**: 执行以 `vector::BroadcastOp::create` 为核心的调用或声明。
- **L633 EN**: Continues the surrounding expression or declaration: `Value offsetVal =`.
  **L633 CN**: 继续构造周围的表达式或声明：`Value offsetVal =`。
- **L634 EN**: Executes a call or declaration centered on `arith::AddIOp::create`.
  **L634 CN**: 执行以 `arith::AddIOp::create` 为核心的调用或声明。
- **L635 EN**: Executes a call or declaration centered on `convertedOffsets.push_back`.
  **L635 CN**: 执行以 `convertedOffsets.push_back` 为核心的调用或声明。
- **L636 EN**: Closes the current lexical scope or compound statement.
  **L636 CN**: 结束当前词法作用域或复合语句块。
- **L637 EN**: Closes the current lexical scope or compound statement.
  **L637 CN**: 结束当前词法作用域或复合语句块。
- **L638 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L638 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L639 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L639 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L640 EN**: Continues the surrounding expression or declaration: `convertedMasks =`.
  **L640 CN**: 继续构造周围的表达式或声明：`convertedMasks =`。
- **L641 EN**: Executes a call or declaration centered on `pack`.
  **L641 CN**: 执行以 `pack` 为核心的调用或声明。
- **L642 EN**: Blank line separating nearby declarations or logic blocks.
  **L642 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L643 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L643 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L644 EN**: Continues the surrounding expression or declaration: `convertedOffsets =`.
  **L644 CN**: 继续构造周围的表达式或声明：`convertedOffsets =`。
- **L645 EN**: Executes a call or declaration centered on `pack`.
  **L645 CN**: 执行以 `pack` 为核心的调用或声明。
- **L646 EN**: Closes the current lexical scope or compound statement.
  **L646 CN**: 结束当前词法作用域或复合语句块。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L648 EN**: Continues the surrounding expression or declaration: `SmallVector<Type> convertedValTypes =`.
  **L648 CN**: 继续构造周围的表达式或声明：`SmallVector<Type> convertedValTypes =`。

### Lines 649-672

````cpp
        getUnrolledTypes(valueTy, *targetShape);
    SmallVector<Value> convertedValues =
        pack(op.getValue(), convertedValTypes, *targetShape, loc, rewriter);

    auto layout = op.getLayoutAttr();
    if (layout)
      layout = layout.dropInstData();

    for (auto [v, o, m] :
         llvm::zip(convertedValues, convertedOffsets, convertedMasks)) {
      xegpu::StoreScatterOp::create(rewriter, loc, v, op.getDest(), o, m,
                                    rewriter.getI64IntegerAttr(chunkSize),
                                    op.getL1HintAttr(), op.getL2HintAttr(),
                                    op.getL3HintAttr(), layout);
    }

    rewriter.eraseOp(op);
    return success();
  }
};

struct UnrollLoadMatrixOp : public UnrollPattern<xegpu::LoadMatrixOp> {
  using UnrollPattern<xegpu::LoadMatrixOp>::UnrollPattern;
  LogicalResult matchAndRewrite(xegpu::LoadMatrixOp op,
````
- **L649 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L649 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L650 EN**: Continues the surrounding expression or declaration: `SmallVector<Value> convertedValues =`.
  **L650 CN**: 继续构造周围的表达式或声明：`SmallVector<Value> convertedValues =`。
- **L651 EN**: Executes a call or declaration centered on `pack`.
  **L651 CN**: 执行以 `pack` 为核心的调用或声明。
- **L652 EN**: Blank line separating nearby declarations or logic blocks.
  **L652 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L653 EN**: Initializes variable `layout` from the right-hand expression.
  **L653 CN**: 使用右侧表达式初始化变量 `layout`。
- **L654 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L654 CN**: 开始 `if` 控制流语句并计算其条件。
- **L655 EN**: Executes a call or declaration centered on `layout.dropInstData`.
  **L655 CN**: 执行以 `layout.dropInstData` 为核心的调用或声明。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L657 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L657 CN**: 开始 `for` 控制流语句并计算其条件。
- **L658 EN**: Starts a function, method, lambda, or structured scope: `llvm::zip(convertedValues, convertedOffsets, convertedMasks)) {`.
  **L658 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::zip(convertedValues, convertedOffsets, convertedMasks)) {`。
- **L659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::StoreScatterOp::create(rewriter, loc, v, op.getDest(), o, m,`.
  **L659 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::StoreScatterOp::create(rewriter, loc, v, op.getDest(), o, m,`。
- **L660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter.getI64IntegerAttr(chunkSize),`.
  **L660 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter.getI64IntegerAttr(chunkSize),`。
- **L661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op.getL1HintAttr(), op.getL2HintAttr(),`.
  **L661 CN**: 继续一个多行参数列表、初始化器或聚合项：`op.getL1HintAttr(), op.getL2HintAttr(),`。
- **L662 EN**: Executes a call or declaration centered on `op.getL3HintAttr`.
  **L662 CN**: 执行以 `op.getL3HintAttr` 为核心的调用或声明。
- **L663 EN**: Closes the current lexical scope or compound statement.
  **L663 CN**: 结束当前词法作用域或复合语句块。
- **L664 EN**: Blank line separating nearby declarations or logic blocks.
  **L664 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L665 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L665 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L666 EN**: Returns from the current function with `success()`.
  **L666 CN**: 以 `success()` 从当前函数返回。
- **L667 EN**: Closes the current lexical scope or compound statement.
  **L667 CN**: 结束当前词法作用域或复合语句块。
- **L668 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L668 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L669 EN**: Blank line separating nearby declarations or logic blocks.
  **L669 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L670 EN**: Declares struct `UnrollLoadMatrixOp`.
  **L670 CN**: 声明 struct `UnrollLoadMatrixOp`。
- **L671 EN**: Executes a standalone statement or declaration: `using UnrollPattern<xegpu::LoadMatrixOp>::UnrollPattern;`.
  **L671 CN**: 执行一条独立语句或声明：`using UnrollPattern<xegpu::LoadMatrixOp>::UnrollPattern;`。
- **L672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::LoadMatrixOp op,`.
  **L672 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::LoadMatrixOp op,`。

### Lines 673-696

````cpp
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    VectorType valueTy = llvm::dyn_cast<VectorType>(op.getType());
    assert(valueTy && "the value type must be vector type!");

    std::optional<SmallVector<int64_t>> targetShape = getTargetShape(op);
    if (!targetShape || targetShape->size() != (size_t)valueTy.getRank())
      return failure();

    Type elemTy = valueTy.getElementType();
    ArrayRef<int64_t> shape = valueTy.getShape();
    auto layout = dyn_cast<xegpu::LayoutAttr>(op.getLayoutAttr());

    VectorType newValueTy = valueTy.cloneWith(*targetShape, elemTy);

    SmallVector<OpFoldResult> mixedOffsets = op.getMixedOffsets();
    SmallVector<SmallVector<OpFoldResult>> offsetsList;
    for (SmallVector<int64_t> offsets :
         StaticTileOffsetRange(shape, *targetShape)) {
      auto adds = xegpu::addElementwise(
          rewriter, loc, mixedOffsets,
          getAsIndexOpFoldResult(op.getContext(), offsets));
      offsetsList.push_back(adds);
    }
````
- **L673 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L673 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L674 EN**: Initializes variable `loc` from the right-hand expression.
  **L674 CN**: 使用右侧表达式初始化变量 `loc`。
- **L675 EN**: Initializes variable `valueTy` from the right-hand expression.
  **L675 CN**: 使用右侧表达式初始化变量 `valueTy`。
- **L676 EN**: Checks an internal invariant in debug builds.
  **L676 CN**: 在调试构建中检查内部不变式。
- **L677 EN**: Blank line separating nearby declarations or logic blocks.
  **L677 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L678 EN**: Initializes variable `targetShape` from the right-hand expression.
  **L678 CN**: 使用右侧表达式初始化变量 `targetShape`。
- **L679 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L679 CN**: 开始 `if` 控制流语句并计算其条件。
- **L680 EN**: Returns from the current function with `failure()`.
  **L680 CN**: 以 `failure()` 从当前函数返回。
- **L681 EN**: Blank line separating nearby declarations or logic blocks.
  **L681 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L682 EN**: Initializes variable `elemTy` from the right-hand expression.
  **L682 CN**: 使用右侧表达式初始化变量 `elemTy`。
- **L683 EN**: Initializes variable `shape` from the right-hand expression.
  **L683 CN**: 使用右侧表达式初始化变量 `shape`。
- **L684 EN**: Initializes variable `layout` from the right-hand expression.
  **L684 CN**: 使用右侧表达式初始化变量 `layout`。
- **L685 EN**: Blank line separating nearby declarations or logic blocks.
  **L685 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L686 EN**: Initializes variable `newValueTy` from the right-hand expression.
  **L686 CN**: 使用右侧表达式初始化变量 `newValueTy`。
- **L687 EN**: Blank line separating nearby declarations or logic blocks.
  **L687 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L688 EN**: Initializes variable `mixedOffsets` from the right-hand expression.
  **L688 CN**: 使用右侧表达式初始化变量 `mixedOffsets`。
- **L689 EN**: Executes a standalone statement or declaration: `SmallVector<SmallVector<OpFoldResult>> offsetsList;`.
  **L689 CN**: 执行一条独立语句或声明：`SmallVector<SmallVector<OpFoldResult>> offsetsList;`。
- **L690 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L690 CN**: 开始 `for` 控制流语句并计算其条件。
- **L691 EN**: Starts a function, method, lambda, or structured scope: `StaticTileOffsetRange(shape, *targetShape)) {`.
  **L691 CN**: 开始一个函数、方法、lambda 或结构化作用域：`StaticTileOffsetRange(shape, *targetShape)) {`。
- **L692 EN**: Continues logic associated with callable symbol `addElementwise`.
  **L692 CN**: 继续与可调用符号 `addElementwise` 相关的逻辑。
- **L693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, mixedOffsets,`.
  **L693 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, mixedOffsets,`。
- **L694 EN**: Executes a call or declaration centered on `getAsIndexOpFoldResult`.
  **L694 CN**: 执行以 `getAsIndexOpFoldResult` 为核心的调用或声明。
- **L695 EN**: Executes a call or declaration centered on `offsetsList.push_back`.
  **L695 CN**: 执行以 `offsetsList.push_back` 为核心的调用或声明。
- **L696 EN**: Closes the current lexical scope or compound statement.
  **L696 CN**: 结束当前词法作用域或复合语句块。

### Lines 697-720

````cpp

    SmallVector<Value> newOps;
    layout = layout.dropInstData();
    for (SmallVector<OpFoldResult> offsets : offsetsList) {
      auto newOp = xegpu::LoadMatrixOp::create(
          rewriter, op.getLoc(), newValueTy, op.getMemDesc(), offsets, layout);
      newOps.push_back(newOp);
    }
    Value castOp = unpack(newOps, op.getType(), *targetShape, loc, rewriter);
    rewriter.replaceOp(op, castOp);
    return success();
  }
};

struct UnrollStoreMatrixOp : public UnrollPattern<xegpu::StoreMatrixOp> {
  using UnrollPattern<xegpu::StoreMatrixOp>::UnrollPattern;
  LogicalResult matchAndRewrite(xegpu::StoreMatrixOp op,
                                PatternRewriter &rewriter) const override {
    std::optional<SmallVector<int64_t>> targetShape = getTargetShape(op);
    if (!targetShape)
      return failure();

    Location loc = op.getLoc();
    VectorType valueTy = llvm::dyn_cast<VectorType>(op.getData().getType());
````
- **L697 EN**: Blank line separating nearby declarations or logic blocks.
  **L697 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L698 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newOps;`.
  **L698 CN**: 执行一条独立语句或声明：`SmallVector<Value> newOps;`。
- **L699 EN**: Executes a call or declaration centered on `layout.dropInstData`.
  **L699 CN**: 执行以 `layout.dropInstData` 为核心的调用或声明。
- **L700 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L700 CN**: 开始 `for` 控制流语句并计算其条件。
- **L701 EN**: Continues logic associated with callable symbol `create`.
  **L701 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L702 EN**: Executes a call or declaration centered on `op.getLoc`.
  **L702 CN**: 执行以 `op.getLoc` 为核心的调用或声明。
- **L703 EN**: Executes a call or declaration centered on `newOps.push_back`.
  **L703 CN**: 执行以 `newOps.push_back` 为核心的调用或声明。
- **L704 EN**: Closes the current lexical scope or compound statement.
  **L704 CN**: 结束当前词法作用域或复合语句块。
- **L705 EN**: Initializes variable `castOp` from the right-hand expression.
  **L705 CN**: 使用右侧表达式初始化变量 `castOp`。
- **L706 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L706 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L707 EN**: Returns from the current function with `success()`.
  **L707 CN**: 以 `success()` 从当前函数返回。
- **L708 EN**: Closes the current lexical scope or compound statement.
  **L708 CN**: 结束当前词法作用域或复合语句块。
- **L709 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L709 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L710 EN**: Blank line separating nearby declarations or logic blocks.
  **L710 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L711 EN**: Declares struct `UnrollStoreMatrixOp`.
  **L711 CN**: 声明 struct `UnrollStoreMatrixOp`。
- **L712 EN**: Executes a standalone statement or declaration: `using UnrollPattern<xegpu::StoreMatrixOp>::UnrollPattern;`.
  **L712 CN**: 执行一条独立语句或声明：`using UnrollPattern<xegpu::StoreMatrixOp>::UnrollPattern;`。
- **L713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::StoreMatrixOp op,`.
  **L713 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::StoreMatrixOp op,`。
- **L714 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L714 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L715 EN**: Initializes variable `targetShape` from the right-hand expression.
  **L715 CN**: 使用右侧表达式初始化变量 `targetShape`。
- **L716 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L716 CN**: 开始 `if` 控制流语句并计算其条件。
- **L717 EN**: Returns from the current function with `failure()`.
  **L717 CN**: 以 `failure()` 从当前函数返回。
- **L718 EN**: Blank line separating nearby declarations or logic blocks.
  **L718 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L719 EN**: Initializes variable `loc` from the right-hand expression.
  **L719 CN**: 使用右侧表达式初始化变量 `loc`。
- **L720 EN**: Initializes variable `valueTy` from the right-hand expression.
  **L720 CN**: 使用右侧表达式初始化变量 `valueTy`。

### Lines 721-744

````cpp
    assert(valueTy && "the value type must be vector type!");
    ArrayRef<int64_t> shape = valueTy.getShape();
    auto layout = dyn_cast<xegpu::LayoutAttr>(op.getLayoutAttr());

    SmallVector<Type> convertedValTypes =
        getUnrolledTypes(valueTy, *targetShape);
    SmallVector<Value> convertedValues =
        pack(op.getData(), convertedValTypes, *targetShape, loc, rewriter);

    SmallVector<OpFoldResult> mixedOffsets = op.getMixedOffsets();
    SmallVector<SmallVector<OpFoldResult>> offsetsList;
    for (SmallVector<int64_t> offsets :
         StaticTileOffsetRange(shape, *targetShape)) {
      auto adds = xegpu::addElementwise(
          rewriter, loc, mixedOffsets,
          getAsIndexOpFoldResult(op.getContext(), offsets));
      offsetsList.push_back(adds);
    }

    for (auto [v, offsets] : llvm::zip_equal(convertedValues, offsetsList))
      xegpu::StoreMatrixOp::create(rewriter, loc, v, op.getMemDesc(), offsets,
                                   layout.dropInstData());

    rewriter.eraseOp(op);
````
- **L721 EN**: Checks an internal invariant in debug builds.
  **L721 CN**: 在调试构建中检查内部不变式。
- **L722 EN**: Initializes variable `shape` from the right-hand expression.
  **L722 CN**: 使用右侧表达式初始化变量 `shape`。
- **L723 EN**: Initializes variable `layout` from the right-hand expression.
  **L723 CN**: 使用右侧表达式初始化变量 `layout`。
- **L724 EN**: Blank line separating nearby declarations or logic blocks.
  **L724 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L725 EN**: Continues the surrounding expression or declaration: `SmallVector<Type> convertedValTypes =`.
  **L725 CN**: 继续构造周围的表达式或声明：`SmallVector<Type> convertedValTypes =`。
- **L726 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L726 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L727 EN**: Continues the surrounding expression or declaration: `SmallVector<Value> convertedValues =`.
  **L727 CN**: 继续构造周围的表达式或声明：`SmallVector<Value> convertedValues =`。
- **L728 EN**: Executes a call or declaration centered on `pack`.
  **L728 CN**: 执行以 `pack` 为核心的调用或声明。
- **L729 EN**: Blank line separating nearby declarations or logic blocks.
  **L729 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L730 EN**: Initializes variable `mixedOffsets` from the right-hand expression.
  **L730 CN**: 使用右侧表达式初始化变量 `mixedOffsets`。
- **L731 EN**: Executes a standalone statement or declaration: `SmallVector<SmallVector<OpFoldResult>> offsetsList;`.
  **L731 CN**: 执行一条独立语句或声明：`SmallVector<SmallVector<OpFoldResult>> offsetsList;`。
- **L732 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L732 CN**: 开始 `for` 控制流语句并计算其条件。
- **L733 EN**: Starts a function, method, lambda, or structured scope: `StaticTileOffsetRange(shape, *targetShape)) {`.
  **L733 CN**: 开始一个函数、方法、lambda 或结构化作用域：`StaticTileOffsetRange(shape, *targetShape)) {`。
- **L734 EN**: Continues logic associated with callable symbol `addElementwise`.
  **L734 CN**: 继续与可调用符号 `addElementwise` 相关的逻辑。
- **L735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, mixedOffsets,`.
  **L735 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, mixedOffsets,`。
- **L736 EN**: Executes a call or declaration centered on `getAsIndexOpFoldResult`.
  **L736 CN**: 执行以 `getAsIndexOpFoldResult` 为核心的调用或声明。
- **L737 EN**: Executes a call or declaration centered on `offsetsList.push_back`.
  **L737 CN**: 执行以 `offsetsList.push_back` 为核心的调用或声明。
- **L738 EN**: Closes the current lexical scope or compound statement.
  **L738 CN**: 结束当前词法作用域或复合语句块。
- **L739 EN**: Blank line separating nearby declarations or logic blocks.
  **L739 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L740 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L740 CN**: 开始 `for` 控制流语句并计算其条件。
- **L741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::StoreMatrixOp::create(rewriter, loc, v, op.getMemDesc(), offsets,`.
  **L741 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::StoreMatrixOp::create(rewriter, loc, v, op.getMemDesc(), offsets,`。
- **L742 EN**: Executes a call or declaration centered on `layout.dropInstData`.
  **L742 CN**: 执行以 `layout.dropInstData` 为核心的调用或声明。
- **L743 EN**: Blank line separating nearby declarations or logic blocks.
  **L743 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L744 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L744 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。

### Lines 745-768

````cpp
    return success();
  }
};

/// UnrollConvertLayoutOp pattern for unrolling xegpu::ConvertLayoutOp
/// operations. It first check whether the convert layout op has valid layouts
/// after inst_data stripped. If it does, it will unroll the vector into
/// multiple smaller vectors according to the target shape, and create multiple
/// ConvertLayoutOp with the unrolled vectors and the stripped layouts.
struct UnrollConvertLayoutOp : public UnrollPattern<xegpu::ConvertLayoutOp> {
  using UnrollPattern<xegpu::ConvertLayoutOp>::UnrollPattern;
  LogicalResult matchAndRewrite(xegpu::ConvertLayoutOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    Type valType = op.getType();

    xegpu::DistributeLayoutAttr inputLayout = op.getInputLayoutAttr();
    xegpu::DistributeLayoutAttr targetLayout = op.getTargetLayoutAttr();
    if (!inputLayout || !targetLayout)
      return rewriter.notifyMatchFailure(op, "missing layout attributes.");

    if (valType.isIntOrFloat()) {
      rewriter.replaceOp(op, op.getSource());
      assert(!inputLayout.dropInstData() && !targetLayout.dropInstData() &&
````
- **L745 EN**: Returns from the current function with `success()`.
  **L745 CN**: 以 `success()` 从当前函数返回。
- **L746 EN**: Closes the current lexical scope or compound statement.
  **L746 CN**: 结束当前词法作用域或复合语句块。
- **L747 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L747 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L748 EN**: Blank line separating nearby declarations or logic blocks.
  **L748 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L749 EN**: Comment explains nearby logic, invariants, or intent: `UnrollConvertLayoutOp pattern for unrolling xegpu::ConvertLayoutOp`.
  **L749 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`UnrollConvertLayoutOp pattern for unrolling xegpu::ConvertLayoutOp`。
- **L750 EN**: Comment explains nearby logic, invariants, or intent: `operations. It first check whether the convert layout op has valid layouts`.
  **L750 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`operations. It first check whether the convert layout op has valid layouts`。
- **L751 EN**: Comment explains nearby logic, invariants, or intent: `after inst_data stripped. If it does, it will unroll the vector into`.
  **L751 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`after inst_data stripped. If it does, it will unroll the vector into`。
- **L752 EN**: Comment explains nearby logic, invariants, or intent: `multiple smaller vectors according to the target shape, and create multiple`.
  **L752 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`multiple smaller vectors according to the target shape, and create multiple`。
- **L753 EN**: Comment explains nearby logic, invariants, or intent: `ConvertLayoutOp with the unrolled vectors and the stripped layouts.`.
  **L753 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ConvertLayoutOp with the unrolled vectors and the stripped layouts.`。
- **L754 EN**: Declares struct `UnrollConvertLayoutOp`.
  **L754 CN**: 声明 struct `UnrollConvertLayoutOp`。
- **L755 EN**: Executes a standalone statement or declaration: `using UnrollPattern<xegpu::ConvertLayoutOp>::UnrollPattern;`.
  **L755 CN**: 执行一条独立语句或声明：`using UnrollPattern<xegpu::ConvertLayoutOp>::UnrollPattern;`。
- **L756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::ConvertLayoutOp op,`.
  **L756 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::ConvertLayoutOp op,`。
- **L757 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L757 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L758 EN**: Initializes variable `loc` from the right-hand expression.
  **L758 CN**: 使用右侧表达式初始化变量 `loc`。
- **L759 EN**: Initializes variable `valType` from the right-hand expression.
  **L759 CN**: 使用右侧表达式初始化变量 `valType`。
- **L760 EN**: Blank line separating nearby declarations or logic blocks.
  **L760 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L761 EN**: Initializes variable `inputLayout` from the right-hand expression.
  **L761 CN**: 使用右侧表达式初始化变量 `inputLayout`。
- **L762 EN**: Initializes variable `targetLayout` from the right-hand expression.
  **L762 CN**: 使用右侧表达式初始化变量 `targetLayout`。
- **L763 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L763 CN**: 开始 `if` 控制流语句并计算其条件。
- **L764 EN**: Returns from the current function with `rewriter.notifyMatchFailure(op, "missing layout attributes.")`.
  **L764 CN**: 以 `rewriter.notifyMatchFailure(op, "missing layout attributes.")` 从当前函数返回。
- **L765 EN**: Blank line separating nearby declarations or logic blocks.
  **L765 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L766 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L766 CN**: 开始 `if` 控制流语句并计算其条件。
- **L767 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L767 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L768 EN**: Checks an internal invariant in debug builds.
  **L768 CN**: 在调试构建中检查内部不变式。

### Lines 769-792

````cpp
             "unexpected layout attributes for scalar type");
      return success();
    }

    if (inputLayout.getEffectiveInstDataAsInt().empty() ||
        targetLayout.getEffectiveInstDataAsInt().empty())
      return rewriter.notifyMatchFailure(op, "Not a target ConvertLayoutOp.");

    inputLayout = inputLayout.dropInstData();
    targetLayout = targetLayout.dropInstData();

    VectorType valueTy = llvm::dyn_cast<VectorType>(op.getType());
    assert(valueTy && "the value type must be vector type!");

    std::optional<SmallVector<int64_t>> targetShape = getTargetShape(op);
    if (!targetShape || targetShape->size() != (size_t)valueTy.getRank())
      return failure();

    Value newSource = op.getSource();
    SmallVector<Value> newOps;
    if (inputLayout && targetLayout) {
      SmallVector<Type> convertedValTypes =
          getUnrolledTypes(valueTy, *targetShape);
      SmallVector<Value> convertedValues =
````
- **L769 EN**: Executes a standalone statement or declaration: `"unexpected layout attributes for scalar type");`.
  **L769 CN**: 执行一条独立语句或声明：`"unexpected layout attributes for scalar type");`。
- **L770 EN**: Returns from the current function with `success()`.
  **L770 CN**: 以 `success()` 从当前函数返回。
- **L771 EN**: Closes the current lexical scope or compound statement.
  **L771 CN**: 结束当前词法作用域或复合语句块。
- **L772 EN**: Blank line separating nearby declarations or logic blocks.
  **L772 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L773 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L773 CN**: 开始 `if` 控制流语句并计算其条件。
- **L774 EN**: Continues logic associated with callable symbol `getEffectiveInstDataAsInt`.
  **L774 CN**: 继续与可调用符号 `getEffectiveInstDataAsInt` 相关的逻辑。
- **L775 EN**: Returns from the current function with `rewriter.notifyMatchFailure(op, "Not a target ConvertLayoutOp.")`.
  **L775 CN**: 以 `rewriter.notifyMatchFailure(op, "Not a target ConvertLayoutOp.")` 从当前函数返回。
- **L776 EN**: Blank line separating nearby declarations or logic blocks.
  **L776 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L777 EN**: Executes a call or declaration centered on `inputLayout.dropInstData`.
  **L777 CN**: 执行以 `inputLayout.dropInstData` 为核心的调用或声明。
- **L778 EN**: Executes a call or declaration centered on `targetLayout.dropInstData`.
  **L778 CN**: 执行以 `targetLayout.dropInstData` 为核心的调用或声明。
- **L779 EN**: Blank line separating nearby declarations or logic blocks.
  **L779 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L780 EN**: Initializes variable `valueTy` from the right-hand expression.
  **L780 CN**: 使用右侧表达式初始化变量 `valueTy`。
- **L781 EN**: Checks an internal invariant in debug builds.
  **L781 CN**: 在调试构建中检查内部不变式。
- **L782 EN**: Blank line separating nearby declarations or logic blocks.
  **L782 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L783 EN**: Initializes variable `targetShape` from the right-hand expression.
  **L783 CN**: 使用右侧表达式初始化变量 `targetShape`。
- **L784 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L784 CN**: 开始 `if` 控制流语句并计算其条件。
- **L785 EN**: Returns from the current function with `failure()`.
  **L785 CN**: 以 `failure()` 从当前函数返回。
- **L786 EN**: Blank line separating nearby declarations or logic blocks.
  **L786 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L787 EN**: Initializes variable `newSource` from the right-hand expression.
  **L787 CN**: 使用右侧表达式初始化变量 `newSource`。
- **L788 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newOps;`.
  **L788 CN**: 执行一条独立语句或声明：`SmallVector<Value> newOps;`。
- **L789 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L789 CN**: 开始 `if` 控制流语句并计算其条件。
- **L790 EN**: Continues the surrounding expression or declaration: `SmallVector<Type> convertedValTypes =`.
  **L790 CN**: 继续构造周围的表达式或声明：`SmallVector<Type> convertedValTypes =`。
- **L791 EN**: Executes a call or declaration centered on `getUnrolledTypes`.
  **L791 CN**: 执行以 `getUnrolledTypes` 为核心的调用或声明。
- **L792 EN**: Continues the surrounding expression or declaration: `SmallVector<Value> convertedValues =`.
  **L792 CN**: 继续构造周围的表达式或声明：`SmallVector<Value> convertedValues =`。

### Lines 793-816

````cpp
          pack(op.getOperand(), convertedValTypes, *targetShape, loc, rewriter);
      for (auto [v, t] : llvm::zip(convertedValues, convertedValTypes)) {
        auto newOp = xegpu::ConvertLayoutOp::create(rewriter, loc, t, v,
                                                    inputLayout, targetLayout);
        newOps.push_back(newOp);
      }
      newSource = unpack(newOps, op.getType(), *targetShape, loc, rewriter);
    }

    rewriter.replaceOp(op, newSource);
    return success();
  }
};

} // namespace

void mlir::xegpu::populateXeGPUUnrollPatterns(
    RewritePatternSet &patterns, const xegpu::UnrollOptions &options) {
  patterns.add<UnrollCreateNdOp, UnrollPrefetchNdOp, UnrollLoadNdOp,
               UnrollStoreNdOp, UnrollDpasOp, UnrollDpasMxOp,
               UnrollLoadMatrixOp, UnrollStoreMatrixOp, UnrollLoadGatherOp,
               UnrollStoreScatterOp, UnrollConvertLayoutOp>(
      patterns.getContext(), options);
}
````
- **L793 EN**: Executes a call or declaration centered on `pack`.
  **L793 CN**: 执行以 `pack` 为核心的调用或声明。
- **L794 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L794 CN**: 开始 `for` 控制流语句并计算其条件。
- **L795 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto newOp = xegpu::ConvertLayoutOp::create(rewriter, loc, t, v,`.
  **L795 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto newOp = xegpu::ConvertLayoutOp::create(rewriter, loc, t, v,`。
- **L796 EN**: Executes a standalone statement or declaration: `inputLayout, targetLayout);`.
  **L796 CN**: 执行一条独立语句或声明：`inputLayout, targetLayout);`。
- **L797 EN**: Executes a call or declaration centered on `newOps.push_back`.
  **L797 CN**: 执行以 `newOps.push_back` 为核心的调用或声明。
- **L798 EN**: Closes the current lexical scope or compound statement.
  **L798 CN**: 结束当前词法作用域或复合语句块。
- **L799 EN**: Executes a call or declaration centered on `unpack`.
  **L799 CN**: 执行以 `unpack` 为核心的调用或声明。
- **L800 EN**: Closes the current lexical scope or compound statement.
  **L800 CN**: 结束当前词法作用域或复合语句块。
- **L801 EN**: Blank line separating nearby declarations or logic blocks.
  **L801 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L802 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L802 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L803 EN**: Returns from the current function with `success()`.
  **L803 CN**: 以 `success()` 从当前函数返回。
- **L804 EN**: Closes the current lexical scope or compound statement.
  **L804 CN**: 结束当前词法作用域或复合语句块。
- **L805 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L805 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L806 EN**: Blank line separating nearby declarations or logic blocks.
  **L806 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L807 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L807 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L808 EN**: Blank line separating nearby declarations or logic blocks.
  **L808 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L809 EN**: Continues logic associated with callable symbol `populateXeGPUUnrollPatterns`.
  **L809 CN**: 继续与可调用符号 `populateXeGPUUnrollPatterns` 相关的逻辑。
- **L810 EN**: Continues the surrounding expression or declaration: `RewritePatternSet &patterns, const xegpu::UnrollOptions &options) {`.
  **L810 CN**: 继续构造周围的表达式或声明：`RewritePatternSet &patterns, const xegpu::UnrollOptions &options) {`。
- **L811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `patterns.add<UnrollCreateNdOp, UnrollPrefetchNdOp, UnrollLoadNdOp,`.
  **L811 CN**: 继续一个多行参数列表、初始化器或聚合项：`patterns.add<UnrollCreateNdOp, UnrollPrefetchNdOp, UnrollLoadNdOp,`。
- **L812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnrollStoreNdOp, UnrollDpasOp, UnrollDpasMxOp,`.
  **L812 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnrollStoreNdOp, UnrollDpasOp, UnrollDpasMxOp,`。
- **L813 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnrollLoadMatrixOp, UnrollStoreMatrixOp, UnrollLoadGatherOp,`.
  **L813 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnrollLoadMatrixOp, UnrollStoreMatrixOp, UnrollLoadGatherOp,`。
- **L814 EN**: Continues logic associated with callable symbol `UnrollConvertLayoutOp>`.
  **L814 CN**: 继续与可调用符号 `UnrollConvertLayoutOp>` 相关的逻辑。
- **L815 EN**: Executes a call or declaration centered on `patterns.getContext`.
  **L815 CN**: 执行以 `patterns.getContext` 为核心的调用或声明。
- **L816 EN**: Closes the current lexical scope or compound statement.
  **L816 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **XeGPU target integration / XeGPU 目标集成**
- **Rewrite-driven lowering / 基于重写的 lowering**
- **Dialect definition and registration / 方言定义与注册**
- **Operation semantics and verification / 操作语义与验证**
- **Pattern-based rewriting / 基于模式的重写**
- **Pattern collection management / 重写模式集合管理**
- **Failure-aware result propagation / 带失败语义的结果传播**
- **IR mutation through rewriter helpers / 通过重写器辅助工具进行 IR 变换**
- **Attribute representation / 属性表示**
- **Type-system modeling / 类型系统建模**

## Dependencies / 依赖关系

- `mlir/Dialect/Utils/IndexingUtils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/IR/XeGPU.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Transforms/Transforms.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Utils/XeGPUUtils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `llvm/ADT/STLExtras.h`: Provides LLVM ADT containers and low-level utility types. / 提供LLVM ADT 容器与底层工具类型。
- `llvm/Support/DebugLog.h`: Provides LLVM support-library facilities such as diagnostics and casting. / 提供LLVM 支持库设施，如诊断与类型转换。
- `mlir/Dialect/XeGPU/Transforms/Passes.h.inc`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
