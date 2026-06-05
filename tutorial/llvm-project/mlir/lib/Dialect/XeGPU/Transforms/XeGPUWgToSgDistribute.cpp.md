# XeGPUWgToSgDistribute.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/XeGPU/Transforms/XeGPUWgToSgDistribute.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements XeGPU lowering, scheduling, and rewrite passes.
- **Purpose (CN)**: 实现 XeGPU lowering、调度与重写 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===- XeGPUWgToSgDistribute.cpp - XeGPU Workgroup to Subgroup Pass -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
#include "mlir/Dialect/XeGPU/Transforms/Passes.h"

#include "mlir/Dialect/Affine/Utils.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/Index/IR/IndexDialect.h"
#include "mlir/Dialect/Index/IR/IndexOps.h"
#include "mlir/Dialect/Math/IR/Math.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/SCF/Transforms/Patterns.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/Dialect/XeGPU/IR/XeGPU.h"
#include "mlir/Dialect/XeGPU/Transforms/Transforms.h"
#include "mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h"
#include "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h"
#include "mlir/Transforms/DialectConversion.h"
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
- **L8 EN**: Includes "mlir/Dialect/XeGPU/Transforms/Passes.h" to access dialect-specific IR, transforms, or shared utilities.
  **L8 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/Passes.h" 以使用方言专用 IR、变换或共享工具。
- **L9 EN**: Blank line separating nearby declarations or logic blocks.
  **L9 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L10 EN**: Includes "mlir/Dialect/Affine/Utils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L10 CN**: 引入 "mlir/Dialect/Affine/Utils.h" 以使用方言专用 IR、变换或共享工具。
- **L11 EN**: Includes "mlir/Dialect/Arith/IR/Arith.h" to access dialect-specific IR, transforms, or shared utilities.
  **L11 CN**: 引入 "mlir/Dialect/Arith/IR/Arith.h" 以使用方言专用 IR、变换或共享工具。
- **L12 EN**: Includes "mlir/Dialect/Arith/Utils/Utils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L12 CN**: 引入 "mlir/Dialect/Arith/Utils/Utils.h" 以使用方言专用 IR、变换或共享工具。
- **L13 EN**: Includes "mlir/Dialect/GPU/IR/GPUDialect.h" to access dialect-specific IR, transforms, or shared utilities.
  **L13 CN**: 引入 "mlir/Dialect/GPU/IR/GPUDialect.h" 以使用方言专用 IR、变换或共享工具。
- **L14 EN**: Includes "mlir/Dialect/Index/IR/IndexDialect.h" to access dialect-specific IR, transforms, or shared utilities.
  **L14 CN**: 引入 "mlir/Dialect/Index/IR/IndexDialect.h" 以使用方言专用 IR、变换或共享工具。
- **L15 EN**: Includes "mlir/Dialect/Index/IR/IndexOps.h" to access dialect-specific IR, transforms, or shared utilities.
  **L15 CN**: 引入 "mlir/Dialect/Index/IR/IndexOps.h" 以使用方言专用 IR、变换或共享工具。
- **L16 EN**: Includes "mlir/Dialect/Math/IR/Math.h" to access dialect-specific IR, transforms, or shared utilities.
  **L16 CN**: 引入 "mlir/Dialect/Math/IR/Math.h" 以使用方言专用 IR、变换或共享工具。
- **L17 EN**: Includes "mlir/Dialect/MemRef/IR/MemRef.h" to access dialect-specific IR, transforms, or shared utilities.
  **L17 CN**: 引入 "mlir/Dialect/MemRef/IR/MemRef.h" 以使用方言专用 IR、变换或共享工具。
- **L18 EN**: Includes "mlir/Dialect/SCF/Transforms/Patterns.h" to access dialect-specific IR, transforms, or shared utilities.
  **L18 CN**: 引入 "mlir/Dialect/SCF/Transforms/Patterns.h" 以使用方言专用 IR、变换或共享工具。
- **L19 EN**: Includes "mlir/Dialect/Utils/IndexingUtils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L19 CN**: 引入 "mlir/Dialect/Utils/IndexingUtils.h" 以使用方言专用 IR、变换或共享工具。
- **L20 EN**: Includes "mlir/Dialect/XeGPU/IR/XeGPU.h" to access dialect-specific IR, transforms, or shared utilities.
  **L20 CN**: 引入 "mlir/Dialect/XeGPU/IR/XeGPU.h" 以使用方言专用 IR、变换或共享工具。
- **L21 EN**: Includes "mlir/Dialect/XeGPU/Transforms/Transforms.h" to access dialect-specific IR, transforms, or shared utilities.
  **L21 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/Transforms.h" 以使用方言专用 IR、变换或共享工具。
- **L22 EN**: Includes "mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h" to access dialect-specific IR, transforms, or shared utilities.
  **L22 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h" 以使用方言专用 IR、变换或共享工具。
- **L23 EN**: Includes "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L23 CN**: 引入 "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h" 以使用方言专用 IR、变换或共享工具。
- **L24 EN**: Includes "mlir/Transforms/DialectConversion.h" to access generic transformation utilities and canonicalization helpers.
  **L24 CN**: 引入 "mlir/Transforms/DialectConversion.h" 以使用通用变换工具与规范化辅助逻辑。

### Lines 25-48

````cpp
#include <optional>

namespace mlir {
namespace xegpu {
#define GEN_PASS_DEF_XEGPUWGTOSGDISTRIBUTE
#include "mlir/Dialect/XeGPU/Transforms/Passes.h.inc"
} // namespace xegpu
} // namespace mlir

using namespace mlir;

namespace {

// Retrieve the RangeAttr if it is specified.
static xegpu::RangeAttr getRangeSpecAttr(Operation *op) {
  Operation *parent = op->getParentOfType<scf::IfOp>();
  while (parent) {
    if (auto attr = llvm::dyn_cast_if_present<xegpu::RangeAttr>(
            parent->getAttr("sg_id_range")))
      return attr;
    parent = parent->getParentOfType<scf::IfOp>();
  }
  return {};
}
````
- **L25 EN**: Includes <optional> to access supporting declarations used by the current translation unit.
  **L25 CN**: 引入 <optional> 以使用当前编译单元使用的辅助声明。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Opens namespace scope `mlir`.
  **L27 CN**: 打开命名空间作用域 `mlir`。
- **L28 EN**: Opens namespace scope `xegpu`.
  **L28 CN**: 打开命名空间作用域 `xegpu`。
- **L29 EN**: Defines macro `GEN_PASS_DEF_XEGPUWGTOSGDISTRIBUTE` for generated declarations, local shorthand, or conditional logic.
  **L29 CN**: 定义宏 `GEN_PASS_DEF_XEGPUWGTOSGDISTRIBUTE`，供生成式声明、本地简写或条件逻辑使用。
- **L30 EN**: Includes "mlir/Dialect/XeGPU/Transforms/Passes.h.inc" to access dialect-specific IR, transforms, or shared utilities.
  **L30 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/Passes.h.inc" 以使用方言专用 IR、变换或共享工具。
- **L31 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace xegpu`.
  **L31 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace xegpu`。
- **L32 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace mlir`.
  **L32 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace mlir`。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L34 EN**: Brings namespace `mlir` into local scope.
  **L34 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L36 EN**: Opens namespace scope ``.
  **L36 CN**: 打开命名空间作用域 ``。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Comment explains nearby logic, invariants, or intent: `Retrieve the RangeAttr if it is specified.`.
  **L38 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Retrieve the RangeAttr if it is specified.`。
- **L39 EN**: Starts a function, method, lambda, or structured scope: `static xegpu::RangeAttr getRangeSpecAttr(Operation *op) {`.
  **L39 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static xegpu::RangeAttr getRangeSpecAttr(Operation *op) {`。
- **L40 EN**: Executes a call or declaration centered on `op->getParentOfType<scf::IfOp>`.
  **L40 CN**: 执行以 `op->getParentOfType<scf::IfOp>` 为核心的调用或声明。
- **L41 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L41 CN**: 开始 `while` 控制流语句并计算其条件。
- **L42 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L42 CN**: 开始 `if` 控制流语句并计算其条件。
- **L43 EN**: Continues logic associated with callable symbol `getAttr`.
  **L43 CN**: 继续与可调用符号 `getAttr` 相关的逻辑。
- **L44 EN**: Returns from the current function with `attr`.
  **L44 CN**: 以 `attr` 从当前函数返回。
- **L45 EN**: Executes a call or declaration centered on `parent->getParentOfType<scf::IfOp>`.
  **L45 CN**: 执行以 `parent->getParentOfType<scf::IfOp>` 为核心的调用或声明。
- **L46 EN**: Closes the current lexical scope or compound statement.
  **L46 CN**: 结束当前词法作用域或复合语句块。
- **L47 EN**: Returns from the current function with `{}`.
  **L47 CN**: 以 `{}` 从当前函数返回。
- **L48 EN**: Closes the current lexical scope or compound statement.
  **L48 CN**: 结束当前词法作用域或复合语句块。

### Lines 49-72

````cpp

static std::pair<SmallVector<int64_t>, int>
getSgShapeAndCount(ArrayRef<int64_t> shape,
                   xegpu::DistributeLayoutAttr layout) {
  int count = 1;
  SmallVector<int64_t> sgShape(shape);
  auto distributedShape = layout.computeDistributedShape(
      SmallVector<int64_t>(shape.begin(), shape.end()));
  if (failed(distributedShape))
    return std::make_pair(sgShape, count);
  auto sgData = layout.getEffectiveSgDataAsInt();
  count = computeProduct(distributedShape.value()) / computeProduct(sgData);
  return std::make_pair(sgData, count);
}

/// Utility helper for deriving a list of offsets for each sub-TensorDescs
/// or sub-MemDescs to be accessed by current subgroup (sgId) based on the
/// associated distribute layout attribute, the shape, subgroup id and the
/// original offsets of the op
template <typename OpType,
          typename = std::enable_if_t<llvm::is_one_of<
              OpType, xegpu::LoadNdOp, xegpu::StoreNdOp, xegpu::PrefetchNdOp,
              xegpu::LoadMatrixOp, xegpu::StoreMatrixOp>::value>>
static LogicalResult
````
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L50 EN**: Continues the surrounding expression or declaration: `static std::pair<SmallVector<int64_t>, int>`.
  **L50 CN**: 继续构造周围的表达式或声明：`static std::pair<SmallVector<int64_t>, int>`。
- **L51 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getSgShapeAndCount(ArrayRef<int64_t> shape,`.
  **L51 CN**: 继续一个多行参数列表、初始化器或聚合项：`getSgShapeAndCount(ArrayRef<int64_t> shape,`。
- **L52 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout) {`.
  **L52 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout) {`。
- **L53 EN**: Initializes variable `count` from the right-hand expression.
  **L53 CN**: 使用右侧表达式初始化变量 `count`。
- **L54 EN**: Executes a call or declaration centered on `sgShape`.
  **L54 CN**: 执行以 `sgShape` 为核心的调用或声明。
- **L55 EN**: Continues logic associated with callable symbol `computeDistributedShape`.
  **L55 CN**: 继续与可调用符号 `computeDistributedShape` 相关的逻辑。
- **L56 EN**: Executes a call or declaration centered on `SmallVector<int64_t>`.
  **L56 CN**: 执行以 `SmallVector<int64_t>` 为核心的调用或声明。
- **L57 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L57 CN**: 开始 `if` 控制流语句并计算其条件。
- **L58 EN**: Returns from the current function with `std::make_pair(sgShape, count)`.
  **L58 CN**: 以 `std::make_pair(sgShape, count)` 从当前函数返回。
- **L59 EN**: Initializes variable `sgData` from the right-hand expression.
  **L59 CN**: 使用右侧表达式初始化变量 `sgData`。
- **L60 EN**: Executes a call or declaration centered on `computeProduct`.
  **L60 CN**: 执行以 `computeProduct` 为核心的调用或声明。
- **L61 EN**: Returns from the current function with `std::make_pair(sgData, count)`.
  **L61 CN**: 以 `std::make_pair(sgData, count)` 从当前函数返回。
- **L62 EN**: Closes the current lexical scope or compound statement.
  **L62 CN**: 结束当前词法作用域或复合语句块。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L64 EN**: Comment explains nearby logic, invariants, or intent: `Utility helper for deriving a list of offsets for each sub-TensorDescs`.
  **L64 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Utility helper for deriving a list of offsets for each sub-TensorDescs`。
- **L65 EN**: Comment explains nearby logic, invariants, or intent: `or sub-MemDescs to be accessed by current subgroup (sgId) based on the`.
  **L65 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`or sub-MemDescs to be accessed by current subgroup (sgId) based on the`。
- **L66 EN**: Comment explains nearby logic, invariants, or intent: `associated distribute layout attribute, the shape, subgroup id and the`.
  **L66 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`associated distribute layout attribute, the shape, subgroup id and the`。
- **L67 EN**: Comment explains nearby logic, invariants, or intent: `original offsets of the op`.
  **L67 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`original offsets of the op`。
- **L68 EN**: Introduces template parameters or specialization context: `template <typename OpType,`.
  **L68 CN**: 为后续声明引入模板参数或特化上下文：`template <typename OpType,`。
- **L69 EN**: Continues the surrounding expression or declaration: `typename = std::enable_if_t<llvm::is_one_of<`.
  **L69 CN**: 继续构造周围的表达式或声明：`typename = std::enable_if_t<llvm::is_one_of<`。
- **L70 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OpType, xegpu::LoadNdOp, xegpu::StoreNdOp, xegpu::PrefetchNdOp,`.
  **L70 CN**: 继续一个多行参数列表、初始化器或聚合项：`OpType, xegpu::LoadNdOp, xegpu::StoreNdOp, xegpu::PrefetchNdOp,`。
- **L71 EN**: Continues the surrounding expression or declaration: `xegpu::LoadMatrixOp, xegpu::StoreMatrixOp>::value>>`.
  **L71 CN**: 继续构造周围的表达式或声明：`xegpu::LoadMatrixOp, xegpu::StoreMatrixOp>::value>>`。
- **L72 EN**: Continues the surrounding expression or declaration: `static LogicalResult`.
  **L72 CN**: 继续构造周围的表达式或声明：`static LogicalResult`。

### Lines 73-96

````cpp
genOffsetsList(ConversionPatternRewriter &rewriter, OpType op,
               SmallVector<SmallVector<OpFoldResult>> &offsetsList) {
  Location loc = op.getLoc();
  SmallVector<OpFoldResult> origOffsets = op.getMixedOffsets();
  // not applicable to ops without offsets operands.
  if (origOffsets.empty())
    return failure();

  // if op is xegpu::CreateNdDescOp, call op.getDescLayoutAttr()
  xegpu::DistributeLayoutAttr layout;
  if constexpr (std::is_same_v<OpType, xegpu::LoadMatrixOp> ||
                std::is_same_v<OpType, xegpu::StoreMatrixOp>) {
    layout = op.getLayoutAttr();
  } else {
    layout = op.getDescLayoutAttr();
  }

  // not applicable to ops without workgroup layout attributes
  if (!layout || !layout.isForWorkgroup())
    return failure();

  Value sgId =
      gpu::SubgroupIdOp::create(rewriter, loc, /*upper_bound=*/nullptr);

````
- **L73 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genOffsetsList(ConversionPatternRewriter &rewriter, OpType op,`.
  **L73 CN**: 继续一个多行参数列表、初始化器或聚合项：`genOffsetsList(ConversionPatternRewriter &rewriter, OpType op,`。
- **L74 EN**: Continues the surrounding expression or declaration: `SmallVector<SmallVector<OpFoldResult>> &offsetsList) {`.
  **L74 CN**: 继续构造周围的表达式或声明：`SmallVector<SmallVector<OpFoldResult>> &offsetsList) {`。
- **L75 EN**: Initializes variable `loc` from the right-hand expression.
  **L75 CN**: 使用右侧表达式初始化变量 `loc`。
- **L76 EN**: Initializes variable `origOffsets` from the right-hand expression.
  **L76 CN**: 使用右侧表达式初始化变量 `origOffsets`。
- **L77 EN**: Comment explains nearby logic, invariants, or intent: `not applicable to ops without offsets operands.`.
  **L77 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`not applicable to ops without offsets operands.`。
- **L78 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L78 CN**: 开始 `if` 控制流语句并计算其条件。
- **L79 EN**: Returns from the current function with `failure()`.
  **L79 CN**: 以 `failure()` 从当前函数返回。
- **L80 EN**: Blank line separating nearby declarations or logic blocks.
  **L80 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L81 EN**: Comment explains nearby logic, invariants, or intent: `if op is xegpu::CreateNdDescOp, call op.getDescLayoutAttr()`.
  **L81 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if op is xegpu::CreateNdDescOp, call op.getDescLayoutAttr()`。
- **L82 EN**: Executes a standalone statement or declaration: `xegpu::DistributeLayoutAttr layout;`.
  **L82 CN**: 执行一条独立语句或声明：`xegpu::DistributeLayoutAttr layout;`。
- **L83 EN**: Continues logic associated with callable symbol `constexpr`.
  **L83 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L84 EN**: Continues the surrounding expression or declaration: `std::is_same_v<OpType, xegpu::StoreMatrixOp>) {`.
  **L84 CN**: 继续构造周围的表达式或声明：`std::is_same_v<OpType, xegpu::StoreMatrixOp>) {`。
- **L85 EN**: Executes a call or declaration centered on `op.getLayoutAttr`.
  **L85 CN**: 执行以 `op.getLayoutAttr` 为核心的调用或声明。
- **L86 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L86 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L87 EN**: Executes a call or declaration centered on `op.getDescLayoutAttr`.
  **L87 CN**: 执行以 `op.getDescLayoutAttr` 为核心的调用或声明。
- **L88 EN**: Closes the current lexical scope or compound statement.
  **L88 CN**: 结束当前词法作用域或复合语句块。
- **L89 EN**: Blank line separating nearby declarations or logic blocks.
  **L89 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L90 EN**: Comment explains nearby logic, invariants, or intent: `not applicable to ops without workgroup layout attributes`.
  **L90 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`not applicable to ops without workgroup layout attributes`。
- **L91 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L91 CN**: 开始 `if` 控制流语句并计算其条件。
- **L92 EN**: Returns from the current function with `failure()`.
  **L92 CN**: 以 `failure()` 从当前函数返回。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L94 EN**: Continues the surrounding expression or declaration: `Value sgId =`.
  **L94 CN**: 继续构造周围的表达式或声明：`Value sgId =`。
- **L95 EN**: Executes a call or declaration centered on `gpu::SubgroupIdOp::create`.
  **L95 CN**: 执行以 `gpu::SubgroupIdOp::create` 为核心的调用或声明。
- **L96 EN**: Blank line separating nearby declarations or logic blocks.
  **L96 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 97-120

````cpp
  // verify and adjust the sgId if the range specifier is present
  xegpu::RangeAttr sgIdRange = getRangeSpecAttr(op);
  if (sgIdRange) {
    int64_t startOfRange = sgIdRange.getStart().getInt();
    int64_t endOfRange = sgIdRange.getEnd().getInt();
    // verify the RangeAttr against the layout attribute
    if (layout.getNumSubgroups() != endOfRange - startOfRange)
      return rewriter.notifyMatchFailure(
          op, "sg_layout size must match the sg_id_range");
    // adjust the sgId if necessary
    if (startOfRange > 0) {
      Value startOfRangeVal =
          arith::ConstantIndexOp::create(rewriter, loc, startOfRange);
      sgId = index::SubOp::create(rewriter, loc, sgId, startOfRangeVal);
    }
  }

  // Compute the list of subgroup-relative offsets for sub-tensors or sub-memory
  // descriptors to be accessed, based on the layout information.
  ArrayRef<int64_t> wgShape = op.getDataShape();
  auto maybeDescOffsets =
      layout.computeDistributedCoords(rewriter, loc, sgId, wgShape);
  if (failed(maybeDescOffsets))
    return failure();
````
- **L97 EN**: Comment explains nearby logic, invariants, or intent: `verify and adjust the sgId if the range specifier is present`.
  **L97 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`verify and adjust the sgId if the range specifier is present`。
- **L98 EN**: Initializes variable `sgIdRange` from the right-hand expression.
  **L98 CN**: 使用右侧表达式初始化变量 `sgIdRange`。
- **L99 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L99 CN**: 开始 `if` 控制流语句并计算其条件。
- **L100 EN**: Initializes variable `startOfRange` from the right-hand expression.
  **L100 CN**: 使用右侧表达式初始化变量 `startOfRange`。
- **L101 EN**: Initializes variable `endOfRange` from the right-hand expression.
  **L101 CN**: 使用右侧表达式初始化变量 `endOfRange`。
- **L102 EN**: Comment explains nearby logic, invariants, or intent: `verify the RangeAttr against the layout attribute`.
  **L102 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`verify the RangeAttr against the layout attribute`。
- **L103 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L103 CN**: 开始 `if` 控制流语句并计算其条件。
- **L104 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L104 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L105 EN**: Executes a standalone statement or declaration: `op, "sg_layout size must match the sg_id_range");`.
  **L105 CN**: 执行一条独立语句或声明：`op, "sg_layout size must match the sg_id_range");`。
- **L106 EN**: Comment explains nearby logic, invariants, or intent: `adjust the sgId if necessary`.
  **L106 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`adjust the sgId if necessary`。
- **L107 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L107 CN**: 开始 `if` 控制流语句并计算其条件。
- **L108 EN**: Continues the surrounding expression or declaration: `Value startOfRangeVal =`.
  **L108 CN**: 继续构造周围的表达式或声明：`Value startOfRangeVal =`。
- **L109 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L109 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L110 EN**: Executes a call or declaration centered on `index::SubOp::create`.
  **L110 CN**: 执行以 `index::SubOp::create` 为核心的调用或声明。
- **L111 EN**: Closes the current lexical scope or compound statement.
  **L111 CN**: 结束当前词法作用域或复合语句块。
- **L112 EN**: Closes the current lexical scope or compound statement.
  **L112 CN**: 结束当前词法作用域或复合语句块。
- **L113 EN**: Blank line separating nearby declarations or logic blocks.
  **L113 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L114 EN**: Comment explains nearby logic, invariants, or intent: `Compute the list of subgroup-relative offsets for sub-tensors or sub-memory`.
  **L114 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Compute the list of subgroup-relative offsets for sub-tensors or sub-memory`。
- **L115 EN**: Comment explains nearby logic, invariants, or intent: `descriptors to be accessed, based on the layout information.`.
  **L115 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`descriptors to be accessed, based on the layout information.`。
- **L116 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L116 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L117 EN**: Continues the surrounding expression or declaration: `auto maybeDescOffsets =`.
  **L117 CN**: 继续构造周围的表达式或声明：`auto maybeDescOffsets =`。
- **L118 EN**: Executes a call or declaration centered on `layout.computeDistributedCoords`.
  **L118 CN**: 执行以 `layout.computeDistributedCoords` 为核心的调用或声明。
- **L119 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L119 CN**: 开始 `if` 控制流语句并计算其条件。
- **L120 EN**: Returns from the current function with `failure()`.
  **L120 CN**: 以 `failure()` 从当前函数返回。

### Lines 121-144

````cpp

  // Compute the final global offsets for each accessed sub-tensor
  // or sub-memory descriptor.
  for (const auto &sgOffsets : *maybeDescOffsets) {
    SmallVector<OpFoldResult> newOffsets = xegpu::addWithRightAligned(
        rewriter, loc, getAsOpFoldResult(sgOffsets), origOffsets);
    offsetsList.push_back(std::move(newOffsets));
  }

  // callback(offsetsList);
  return success();
}

/// This pattern transforms the CreateNdDescOp to create a subgroup descriptor
/// from a workgroup descriptor. It replaces the offsets and sizes with
/// appropriate values for the subgroup.
/// It uses round-robin assignment to distribute the work to the subgroups.
/// Following create_nd_desc operation:
///    %tdesc = xegpu.create_nd_tdesc %src : memref<24x24xf32>
///       -> !xegpu.tensor_desc<24x24xf32, #xegpu.layout<sg_layout = [4, 4],
///           sg_data = [2, 2], lane_layout = [2, 2], lane_data = [1, 1]>>
/// is converted to 9 subgroup level operations based on the sg_layout &
/// sg_data:
///    %tdesc = xegpu.create_nd_tdesc %src : memref<24x24xf32> ->
````
- **L121 EN**: Blank line separating nearby declarations or logic blocks.
  **L121 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L122 EN**: Comment explains nearby logic, invariants, or intent: `Compute the final global offsets for each accessed sub-tensor`.
  **L122 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Compute the final global offsets for each accessed sub-tensor`。
- **L123 EN**: Comment explains nearby logic, invariants, or intent: `or sub-memory descriptor.`.
  **L123 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`or sub-memory descriptor.`。
- **L124 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L124 CN**: 开始 `for` 控制流语句并计算其条件。
- **L125 EN**: Continues logic associated with callable symbol `addWithRightAligned`.
  **L125 CN**: 继续与可调用符号 `addWithRightAligned` 相关的逻辑。
- **L126 EN**: Executes a call or declaration centered on `getAsOpFoldResult`.
  **L126 CN**: 执行以 `getAsOpFoldResult` 为核心的调用或声明。
- **L127 EN**: Executes a call or declaration centered on `offsetsList.push_back`.
  **L127 CN**: 执行以 `offsetsList.push_back` 为核心的调用或声明。
- **L128 EN**: Closes the current lexical scope or compound statement.
  **L128 CN**: 结束当前词法作用域或复合语句块。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L130 EN**: Comment explains nearby logic, invariants, or intent: `callback(offsetsList);`.
  **L130 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`callback(offsetsList);`。
- **L131 EN**: Returns from the current function with `success()`.
  **L131 CN**: 以 `success()` 从当前函数返回。
- **L132 EN**: Closes the current lexical scope or compound statement.
  **L132 CN**: 结束当前词法作用域或复合语句块。
- **L133 EN**: Blank line separating nearby declarations or logic blocks.
  **L133 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L134 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms the CreateNdDescOp to create a subgroup descriptor`.
  **L134 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms the CreateNdDescOp to create a subgroup descriptor`。
- **L135 EN**: Comment explains nearby logic, invariants, or intent: `from a workgroup descriptor. It replaces the offsets and sizes with`.
  **L135 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`from a workgroup descriptor. It replaces the offsets and sizes with`。
- **L136 EN**: Comment explains nearby logic, invariants, or intent: `appropriate values for the subgroup.`.
  **L136 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`appropriate values for the subgroup.`。
- **L137 EN**: Comment explains nearby logic, invariants, or intent: `It uses round-robin assignment to distribute the work to the subgroups.`.
  **L137 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`It uses round-robin assignment to distribute the work to the subgroups.`。
- **L138 EN**: Comment explains nearby logic, invariants, or intent: `Following create_nd_desc operation:`.
  **L138 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Following create_nd_desc operation:`。
- **L139 EN**: Comment explains nearby logic, invariants, or intent: `%tdesc = xegpu.create_nd_tdesc %src : memref<24x24xf32>`.
  **L139 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%tdesc = xegpu.create_nd_tdesc %src : memref<24x24xf32>`。
- **L140 EN**: Comment explains nearby logic, invariants, or intent: `> !xegpu.tensor_desc<24x24xf32, #xegpu.layout<sg_layout = [4, 4],`.
  **L140 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`> !xegpu.tensor_desc<24x24xf32, #xegpu.layout<sg_layout = [4, 4],`。
- **L141 EN**: Comment explains nearby logic, invariants, or intent: `sg_data = [2, 2], lane_layout = [2, 2], lane_data = [1, 1]>>`.
  **L141 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sg_data = [2, 2], lane_layout = [2, 2], lane_data = [1, 1]>>`。
- **L142 EN**: Comment explains nearby logic, invariants, or intent: `is converted to 9 subgroup level operations based on the sg_layout &`.
  **L142 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`is converted to 9 subgroup level operations based on the sg_layout &`。
- **L143 EN**: Comment explains nearby logic, invariants, or intent: `sg_data:`.
  **L143 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sg_data:`。
- **L144 EN**: Comment explains nearby logic, invariants, or intent: `%tdesc = xegpu.create_nd_tdesc %src : memref<24x24xf32> ->`.
  **L144 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%tdesc = xegpu.create_nd_tdesc %src : memref<24x24xf32> ->`。

### Lines 145-168

````cpp
///           !xegpu.tensor_desc<2x2xf32, #xegpu.layout<lane_layout = [2, 2],
///           lane_data = [1, 1]>>
///
/// The sg_layout and sg_data attributes are dropped after the pass as they are
/// no longer needed.
///
/// 24x24 matrix distribution example:
/// sg_layout = [4, 4], sg_data = [2, 2]
/// Each 8x8 matrix within the 24x24 matrix is called a distribution unit.
/// dist_unit_shape = [8, 8] --> sg_layout[i] * sg_data[i]
///
/// +------------------------+
/// | 8x8 | 8x8 | 8x8 |      <- 3 tiles across
/// |-----+-----+-----|
/// | 8x8 | 8x8 | 8x8 |      <- 3 tiles down
/// |-----+-----+-----|
/// | 8x8 | 8x8 | 8x8 |
/// +------------------------+
///
/// Each 8x8 tile is further subdivided among subgroups:
/// +------------------------+
/// | 2x2 2x2 2x2 2x2 |  <- 4 subgroups across (each handles 2 columns)
/// | 2x2 2x2 2x2 2x2 |  <- 4 subgroups down (each handles 2 rows)
/// | 2x2 2x2 2x2 2x2 |
````
- **L145 EN**: Comment explains nearby logic, invariants, or intent: `xegpu.tensor_desc<2x2xf32, #xegpu.layout<lane_layout = [2, 2],`.
  **L145 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`xegpu.tensor_desc<2x2xf32, #xegpu.layout<lane_layout = [2, 2],`。
- **L146 EN**: Comment explains nearby logic, invariants, or intent: `lane_data = [1, 1]>>`.
  **L146 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`lane_data = [1, 1]>>`。
- **L147 EN**: Separator comment used for visual grouping.
  **L147 CN**: 用于视觉分组的分隔注释。
- **L148 EN**: Comment explains nearby logic, invariants, or intent: `The sg_layout and sg_data attributes are dropped after the pass as they are`.
  **L148 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The sg_layout and sg_data attributes are dropped after the pass as they are`。
- **L149 EN**: Comment explains nearby logic, invariants, or intent: `no longer needed.`.
  **L149 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`no longer needed.`。
- **L150 EN**: Separator comment used for visual grouping.
  **L150 CN**: 用于视觉分组的分隔注释。
- **L151 EN**: Comment explains nearby logic, invariants, or intent: `24x24 matrix distribution example:`.
  **L151 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`24x24 matrix distribution example:`。
- **L152 EN**: Comment explains nearby logic, invariants, or intent: `sg_layout = [4, 4], sg_data = [2, 2]`.
  **L152 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sg_layout = [4, 4], sg_data = [2, 2]`。
- **L153 EN**: Comment explains nearby logic, invariants, or intent: `Each 8x8 matrix within the 24x24 matrix is called a distribution unit.`.
  **L153 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Each 8x8 matrix within the 24x24 matrix is called a distribution unit.`。
- **L154 EN**: Comment explains nearby logic, invariants, or intent: `dist_unit_shape = [8, 8] --> sg_layout[i] * sg_data[i]`.
  **L154 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`dist_unit_shape = [8, 8] --> sg_layout[i] * sg_data[i]`。
- **L155 EN**: Separator comment used for visual grouping.
  **L155 CN**: 用于视觉分组的分隔注释。
- **L156 EN**: Comment explains nearby logic, invariants, or intent: `+------------------------+`.
  **L156 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`+------------------------+`。
- **L157 EN**: Comment explains nearby logic, invariants, or intent: `| 8x8 | 8x8 | 8x8 |      <- 3 tiles across`.
  **L157 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`| 8x8 | 8x8 | 8x8 |      <- 3 tiles across`。
- **L158 EN**: Comment explains nearby logic, invariants, or intent: `|-----+-----+-----|`.
  **L158 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`|-----+-----+-----|`。
- **L159 EN**: Comment explains nearby logic, invariants, or intent: `| 8x8 | 8x8 | 8x8 |      <- 3 tiles down`.
  **L159 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`| 8x8 | 8x8 | 8x8 |      <- 3 tiles down`。
- **L160 EN**: Comment explains nearby logic, invariants, or intent: `|-----+-----+-----|`.
  **L160 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`|-----+-----+-----|`。
- **L161 EN**: Comment explains nearby logic, invariants, or intent: `| 8x8 | 8x8 | 8x8 |`.
  **L161 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`| 8x8 | 8x8 | 8x8 |`。
- **L162 EN**: Comment explains nearby logic, invariants, or intent: `+------------------------+`.
  **L162 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`+------------------------+`。
- **L163 EN**: Separator comment used for visual grouping.
  **L163 CN**: 用于视觉分组的分隔注释。
- **L164 EN**: Comment explains nearby logic, invariants, or intent: `Each 8x8 tile is further subdivided among subgroups:`.
  **L164 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Each 8x8 tile is further subdivided among subgroups:`。
- **L165 EN**: Comment explains nearby logic, invariants, or intent: `+------------------------+`.
  **L165 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`+------------------------+`。
- **L166 EN**: Comment explains nearby logic, invariants, or intent: `| 2x2 2x2 2x2 2x2 |  <- 4 subgroups across (each handles 2 columns)`.
  **L166 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`| 2x2 2x2 2x2 2x2 |  <- 4 subgroups across (each handles 2 columns)`。
- **L167 EN**: Comment explains nearby logic, invariants, or intent: `| 2x2 2x2 2x2 2x2 |  <- 4 subgroups down (each handles 2 rows)`.
  **L167 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`| 2x2 2x2 2x2 2x2 |  <- 4 subgroups down (each handles 2 rows)`。
- **L168 EN**: Comment explains nearby logic, invariants, or intent: `| 2x2 2x2 2x2 2x2 |`.
  **L168 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`| 2x2 2x2 2x2 2x2 |`。

### Lines 169-192

````cpp
/// | 2x2 2x2 2x2 2x2 |
/// +------------------------+
///
/// Since the 24x24 matrix is divided into 8x8 distribution units, there will be
/// 9 distribution units (3x3) in total. Hence the 9 subgroup level operations.

/// The pass currently has entire distribution logic in the WgToSgCreateNdOp
/// pattern and all the other ops just follow.
/// TODO: Decouple the distribution logic from WgToSgCreateNdOp for all the
/// ops in the pass.
// This pattern transforms the CreateNdDescOp to create a
// subgroup descriptor from a workgroup descriptor.
struct WgToSgCreateNdOp : public OpConversionPattern<xegpu::CreateNdDescOp> {
  using OpConversionPattern<xegpu::CreateNdDescOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(xegpu::CreateNdDescOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {

    Location loc = op.getLoc();
    MLIRContext *ctx = op.getContext();
    xegpu::TensorDescType tdescTy = op.getType();
    auto layout = dyn_cast<xegpu::LayoutAttr>(tdescTy.getLayout());
    if (!layout || !layout.isForWorkgroup())
````
- **L169 EN**: Comment explains nearby logic, invariants, or intent: `| 2x2 2x2 2x2 2x2 |`.
  **L169 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`| 2x2 2x2 2x2 2x2 |`。
- **L170 EN**: Comment explains nearby logic, invariants, or intent: `+------------------------+`.
  **L170 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`+------------------------+`。
- **L171 EN**: Separator comment used for visual grouping.
  **L171 CN**: 用于视觉分组的分隔注释。
- **L172 EN**: Comment explains nearby logic, invariants, or intent: `Since the 24x24 matrix is divided into 8x8 distribution units, there will be`.
  **L172 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Since the 24x24 matrix is divided into 8x8 distribution units, there will be`。
- **L173 EN**: Comment explains nearby logic, invariants, or intent: `9 distribution units (3x3) in total. Hence the 9 subgroup level operations.`.
  **L173 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`9 distribution units (3x3) in total. Hence the 9 subgroup level operations.`。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L175 EN**: Comment explains nearby logic, invariants, or intent: `The pass currently has entire distribution logic in the WgToSgCreateNdOp`.
  **L175 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The pass currently has entire distribution logic in the WgToSgCreateNdOp`。
- **L176 EN**: Comment explains nearby logic, invariants, or intent: `pattern and all the other ops just follow.`.
  **L176 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`pattern and all the other ops just follow.`。
- **L177 EN**: Comment records a pending task or caution: `TODO: Decouple the distribution logic from WgToSgCreateNdOp for all the`.
  **L177 CN**: 注释记录了待办事项或注意点：`TODO: Decouple the distribution logic from WgToSgCreateNdOp for all the`。
- **L178 EN**: Comment explains nearby logic, invariants, or intent: `ops in the pass.`.
  **L178 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ops in the pass.`。
- **L179 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms the CreateNdDescOp to create a`.
  **L179 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms the CreateNdDescOp to create a`。
- **L180 EN**: Comment explains nearby logic, invariants, or intent: `subgroup descriptor from a workgroup descriptor.`.
  **L180 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`subgroup descriptor from a workgroup descriptor.`。
- **L181 EN**: Declares struct `WgToSgCreateNdOp`.
  **L181 CN**: 声明 struct `WgToSgCreateNdOp`。
- **L182 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::CreateNdDescOp>::OpConversionPattern;`.
  **L182 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::CreateNdDescOp>::OpConversionPattern;`。
- **L183 EN**: Blank line separating nearby declarations or logic blocks.
  **L183 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L184 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L184 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::CreateNdDescOp op, OneToNOpAdaptor adaptor,`.
  **L185 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::CreateNdDescOp op, OneToNOpAdaptor adaptor,`。
- **L186 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L186 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L187 EN**: Blank line separating nearby declarations or logic blocks.
  **L187 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L188 EN**: Initializes variable `loc` from the right-hand expression.
  **L188 CN**: 使用右侧表达式初始化变量 `loc`。
- **L189 EN**: Executes a call or declaration centered on `op.getContext`.
  **L189 CN**: 执行以 `op.getContext` 为核心的调用或声明。
- **L190 EN**: Initializes variable `tdescTy` from the right-hand expression.
  **L190 CN**: 使用右侧表达式初始化变量 `tdescTy`。
- **L191 EN**: Initializes variable `layout` from the right-hand expression.
  **L191 CN**: 使用右侧表达式初始化变量 `layout`。
- **L192 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L192 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 193-216

````cpp
      return failure();

    Type elemTy = tdescTy.getElementType();
    ArrayRef<int64_t> wgShape = tdescTy.getShape();

    SmallVector<int64_t> sgShape;
    int count;
    std::tie(sgShape, count) = getSgShapeAndCount(wgShape, layout);
    xegpu::TensorDescType newTdescTy =
        xegpu::TensorDescType::get(ctx, sgShape, elemTy, tdescTy.getEncoding(),
                                   layout.dropSgLayoutAndData());

    SmallVector<Value> newCreateNdOps(count);
    std::generate(newCreateNdOps.begin(), newCreateNdOps.end(), [&]() {
      return xegpu::CreateNdDescOp::create(rewriter, loc, newTdescTy,
                                           op.getSource(), op.getMixedSizes(),
                                           op.getMixedStrides());
    });

    rewriter.replaceOpWithMultiple(op, {newCreateNdOps});
    return success();
  }
};

````
- **L193 EN**: Returns from the current function with `failure()`.
  **L193 CN**: 以 `failure()` 从当前函数返回。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L195 EN**: Initializes variable `elemTy` from the right-hand expression.
  **L195 CN**: 使用右侧表达式初始化变量 `elemTy`。
- **L196 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L196 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L198 EN**: Executes a standalone statement or declaration: `SmallVector<int64_t> sgShape;`.
  **L198 CN**: 执行一条独立语句或声明：`SmallVector<int64_t> sgShape;`。
- **L199 EN**: Executes a standalone statement or declaration: `int count;`.
  **L199 CN**: 执行一条独立语句或声明：`int count;`。
- **L200 EN**: Executes a call or declaration centered on `std::tie`.
  **L200 CN**: 执行以 `std::tie` 为核心的调用或声明。
- **L201 EN**: Continues the surrounding expression or declaration: `xegpu::TensorDescType newTdescTy =`.
  **L201 CN**: 继续构造周围的表达式或声明：`xegpu::TensorDescType newTdescTy =`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::TensorDescType::get(ctx, sgShape, elemTy, tdescTy.getEncoding(),`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::TensorDescType::get(ctx, sgShape, elemTy, tdescTy.getEncoding(),`。
- **L203 EN**: Executes a call or declaration centered on `layout.dropSgLayoutAndData`.
  **L203 CN**: 执行以 `layout.dropSgLayoutAndData` 为核心的调用或声明。
- **L204 EN**: Blank line separating nearby declarations or logic blocks.
  **L204 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L205 EN**: Executes a call or declaration centered on `newCreateNdOps`.
  **L205 CN**: 执行以 `newCreateNdOps` 为核心的调用或声明。
- **L206 EN**: Starts a function, method, lambda, or structured scope: `std::generate(newCreateNdOps.begin(), newCreateNdOps.end(), [&]() {`.
  **L206 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::generate(newCreateNdOps.begin(), newCreateNdOps.end(), [&]() {`。
- **L207 EN**: Returns from the current function with `xegpu::CreateNdDescOp::create(rewriter, loc, newTdescTy,`.
  **L207 CN**: 以 `xegpu::CreateNdDescOp::create(rewriter, loc, newTdescTy,` 从当前函数返回。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op.getSource(), op.getMixedSizes(),`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`op.getSource(), op.getMixedSizes(),`。
- **L209 EN**: Executes a call or declaration centered on `op.getMixedStrides`.
  **L209 CN**: 执行以 `op.getMixedStrides` 为核心的调用或声明。
- **L210 EN**: Executes a standalone statement or declaration: `});`.
  **L210 CN**: 执行一条独立语句或声明：`});`。
- **L211 EN**: Blank line separating nearby declarations or logic blocks.
  **L211 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L212 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L212 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L213 EN**: Returns from the current function with `success()`.
  **L213 CN**: 以 `success()` 从当前函数返回。
- **L214 EN**: Closes the current lexical scope or compound statement.
  **L214 CN**: 结束当前词法作用域或复合语句块。
- **L215 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L215 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L216 EN**: Blank line separating nearby declarations or logic blocks.
  **L216 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 217-240

````cpp
/// This pattern transforms the LoadNdOp to load subgroup data.
struct WgToSgLoadNdOp : public OpConversionPattern<xegpu::LoadNdOp> {
  using OpConversionPattern<xegpu::LoadNdOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(xegpu::LoadNdOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {

    SmallVector<SmallVector<OpFoldResult>> offsetsList;
    if (failed(genOffsetsList(rewriter, op, offsetsList)))
      return failure();

    xegpu::DistributeLayoutAttr layout = op.getLayoutAttr();
    if (layout)
      layout = layout.dropSgLayoutAndData();
    SmallVector<Value> newOps;
    for (auto [tdesc, offsets] :
         llvm::zip(adaptor.getTensorDesc(), offsetsList)) {
      auto tdescTy = dyn_cast<xegpu::TensorDescType>(tdesc.getType());
      VectorType newResTy =
          VectorType::get(tdescTy.getShape(), tdescTy.getElementType());
      auto newOp = xegpu::LoadNdOp::create(
          rewriter, op.getLoc(), newResTy, tdesc, offsets,
          /*packed = */ nullptr, /*transpose = */ nullptr, op.getL1HintAttr(),
          op.getL2HintAttr(), op.getL3HintAttr(), layout);
````
- **L217 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms the LoadNdOp to load subgroup data.`.
  **L217 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms the LoadNdOp to load subgroup data.`。
- **L218 EN**: Declares struct `WgToSgLoadNdOp`.
  **L218 CN**: 声明 struct `WgToSgLoadNdOp`。
- **L219 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::LoadNdOp>::OpConversionPattern;`.
  **L219 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::LoadNdOp>::OpConversionPattern;`。
- **L220 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L220 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::LoadNdOp op, OneToNOpAdaptor adaptor,`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::LoadNdOp op, OneToNOpAdaptor adaptor,`。
- **L222 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L222 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L224 EN**: Executes a standalone statement or declaration: `SmallVector<SmallVector<OpFoldResult>> offsetsList;`.
  **L224 CN**: 执行一条独立语句或声明：`SmallVector<SmallVector<OpFoldResult>> offsetsList;`。
- **L225 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L225 CN**: 开始 `if` 控制流语句并计算其条件。
- **L226 EN**: Returns from the current function with `failure()`.
  **L226 CN**: 以 `failure()` 从当前函数返回。
- **L227 EN**: Blank line separating nearby declarations or logic blocks.
  **L227 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L228 EN**: Initializes variable `layout` from the right-hand expression.
  **L228 CN**: 使用右侧表达式初始化变量 `layout`。
- **L229 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L229 CN**: 开始 `if` 控制流语句并计算其条件。
- **L230 EN**: Executes a call or declaration centered on `layout.dropSgLayoutAndData`.
  **L230 CN**: 执行以 `layout.dropSgLayoutAndData` 为核心的调用或声明。
- **L231 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newOps;`.
  **L231 CN**: 执行一条独立语句或声明：`SmallVector<Value> newOps;`。
- **L232 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L232 CN**: 开始 `for` 控制流语句并计算其条件。
- **L233 EN**: Starts a function, method, lambda, or structured scope: `llvm::zip(adaptor.getTensorDesc(), offsetsList)) {`.
  **L233 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::zip(adaptor.getTensorDesc(), offsetsList)) {`。
- **L234 EN**: Initializes variable `tdescTy` from the right-hand expression.
  **L234 CN**: 使用右侧表达式初始化变量 `tdescTy`。
- **L235 EN**: Continues the surrounding expression or declaration: `VectorType newResTy =`.
  **L235 CN**: 继续构造周围的表达式或声明：`VectorType newResTy =`。
- **L236 EN**: Executes a call or declaration centered on `VectorType::get`.
  **L236 CN**: 执行以 `VectorType::get` 为核心的调用或声明。
- **L237 EN**: Continues logic associated with callable symbol `create`.
  **L237 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, op.getLoc(), newResTy, tdesc, offsets,`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, op.getLoc(), newResTy, tdesc, offsets,`。
- **L239 EN**: Comment explains nearby logic, invariants, or intent: `packed = */ nullptr, /*transpose = */ nullptr, op.getL1HintAttr(),`.
  **L239 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`packed = */ nullptr, /*transpose = */ nullptr, op.getL1HintAttr(),`。
- **L240 EN**: Executes a call or declaration centered on `op.getL2HintAttr`.
  **L240 CN**: 执行以 `op.getL2HintAttr` 为核心的调用或声明。

### Lines 241-264

````cpp
      newOps.push_back(newOp);
    }
    rewriter.replaceOpWithMultiple(op, {newOps});

    return success();
  }
};

/// This pattern transforms the StoreNdOp to store subgroup data.
struct WgToSgStoreNdOp : public OpConversionPattern<xegpu::StoreNdOp> {
  using OpConversionPattern<xegpu::StoreNdOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(xegpu::StoreNdOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    SmallVector<SmallVector<OpFoldResult>> offsetsList;
    if (failed(genOffsetsList(rewriter, op, offsetsList)))
      return failure();

    xegpu::DistributeLayoutAttr layout = op.getLayoutAttr();
    if (layout)
      layout = layout.dropSgLayoutAndData();
    for (auto [v, tdesc, offsets] :
         llvm::zip(adaptor.getValue(), adaptor.getTensorDesc(), offsetsList)) {
      xegpu::StoreNdOp::create(rewriter, op.getLoc(), v, tdesc, offsets,
````
- **L241 EN**: Executes a call or declaration centered on `newOps.push_back`.
  **L241 CN**: 执行以 `newOps.push_back` 为核心的调用或声明。
- **L242 EN**: Closes the current lexical scope or compound statement.
  **L242 CN**: 结束当前词法作用域或复合语句块。
- **L243 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L243 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L244 EN**: Blank line separating nearby declarations or logic blocks.
  **L244 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L245 EN**: Returns from the current function with `success()`.
  **L245 CN**: 以 `success()` 从当前函数返回。
- **L246 EN**: Closes the current lexical scope or compound statement.
  **L246 CN**: 结束当前词法作用域或复合语句块。
- **L247 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L247 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L249 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms the StoreNdOp to store subgroup data.`.
  **L249 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms the StoreNdOp to store subgroup data.`。
- **L250 EN**: Declares struct `WgToSgStoreNdOp`.
  **L250 CN**: 声明 struct `WgToSgStoreNdOp`。
- **L251 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::StoreNdOp>::OpConversionPattern;`.
  **L251 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::StoreNdOp>::OpConversionPattern;`。
- **L252 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L252 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::StoreNdOp op, OneToNOpAdaptor adaptor,`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::StoreNdOp op, OneToNOpAdaptor adaptor,`。
- **L254 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L254 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L255 EN**: Executes a standalone statement or declaration: `SmallVector<SmallVector<OpFoldResult>> offsetsList;`.
  **L255 CN**: 执行一条独立语句或声明：`SmallVector<SmallVector<OpFoldResult>> offsetsList;`。
- **L256 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L256 CN**: 开始 `if` 控制流语句并计算其条件。
- **L257 EN**: Returns from the current function with `failure()`.
  **L257 CN**: 以 `failure()` 从当前函数返回。
- **L258 EN**: Blank line separating nearby declarations or logic blocks.
  **L258 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L259 EN**: Initializes variable `layout` from the right-hand expression.
  **L259 CN**: 使用右侧表达式初始化变量 `layout`。
- **L260 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L260 CN**: 开始 `if` 控制流语句并计算其条件。
- **L261 EN**: Executes a call or declaration centered on `layout.dropSgLayoutAndData`.
  **L261 CN**: 执行以 `layout.dropSgLayoutAndData` 为核心的调用或声明。
- **L262 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L262 CN**: 开始 `for` 控制流语句并计算其条件。
- **L263 EN**: Starts a function, method, lambda, or structured scope: `llvm::zip(adaptor.getValue(), adaptor.getTensorDesc(), offsetsList)) {`.
  **L263 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::zip(adaptor.getValue(), adaptor.getTensorDesc(), offsetsList)) {`。
- **L264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::StoreNdOp::create(rewriter, op.getLoc(), v, tdesc, offsets,`.
  **L264 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::StoreNdOp::create(rewriter, op.getLoc(), v, tdesc, offsets,`。

### Lines 265-288

````cpp
                               op.getL1HintAttr(), op.getL2HintAttr(),
                               op.getL3HintAttr(), layout);
    }
    rewriter.eraseOp(op);

    return success();
  }
};

/// This pattern transforms the PrefetchNdOp to prefetch subgroup data.
struct WgToSgPrefetchNdOp : public OpConversionPattern<xegpu::PrefetchNdOp> {
  using OpConversionPattern<xegpu::PrefetchNdOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(xegpu::PrefetchNdOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    SmallVector<SmallVector<OpFoldResult>> offsetsList;
    if (failed(genOffsetsList(rewriter, op, offsetsList)))
      return failure();

    xegpu::DistributeLayoutAttr layout = op.getLayoutAttr();
    if (layout)
      layout = layout.dropSgLayoutAndData();
    for (auto [tdesc, offsets] :
         llvm::zip(adaptor.getTensorDesc(), offsetsList)) {
````
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op.getL1HintAttr(), op.getL2HintAttr(),`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`op.getL1HintAttr(), op.getL2HintAttr(),`。
- **L266 EN**: Executes a call or declaration centered on `op.getL3HintAttr`.
  **L266 CN**: 执行以 `op.getL3HintAttr` 为核心的调用或声明。
- **L267 EN**: Closes the current lexical scope or compound statement.
  **L267 CN**: 结束当前词法作用域或复合语句块。
- **L268 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L268 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L270 EN**: Returns from the current function with `success()`.
  **L270 CN**: 以 `success()` 从当前函数返回。
- **L271 EN**: Closes the current lexical scope or compound statement.
  **L271 CN**: 结束当前词法作用域或复合语句块。
- **L272 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L272 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L273 EN**: Blank line separating nearby declarations or logic blocks.
  **L273 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L274 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms the PrefetchNdOp to prefetch subgroup data.`.
  **L274 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms the PrefetchNdOp to prefetch subgroup data.`。
- **L275 EN**: Declares struct `WgToSgPrefetchNdOp`.
  **L275 CN**: 声明 struct `WgToSgPrefetchNdOp`。
- **L276 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::PrefetchNdOp>::OpConversionPattern;`.
  **L276 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::PrefetchNdOp>::OpConversionPattern;`。
- **L277 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L277 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::PrefetchNdOp op, OneToNOpAdaptor adaptor,`.
  **L278 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::PrefetchNdOp op, OneToNOpAdaptor adaptor,`。
- **L279 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L279 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L280 EN**: Executes a standalone statement or declaration: `SmallVector<SmallVector<OpFoldResult>> offsetsList;`.
  **L280 CN**: 执行一条独立语句或声明：`SmallVector<SmallVector<OpFoldResult>> offsetsList;`。
- **L281 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L281 CN**: 开始 `if` 控制流语句并计算其条件。
- **L282 EN**: Returns from the current function with `failure()`.
  **L282 CN**: 以 `failure()` 从当前函数返回。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L284 EN**: Initializes variable `layout` from the right-hand expression.
  **L284 CN**: 使用右侧表达式初始化变量 `layout`。
- **L285 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L285 CN**: 开始 `if` 控制流语句并计算其条件。
- **L286 EN**: Executes a call or declaration centered on `layout.dropSgLayoutAndData`.
  **L286 CN**: 执行以 `layout.dropSgLayoutAndData` 为核心的调用或声明。
- **L287 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L287 CN**: 开始 `for` 控制流语句并计算其条件。
- **L288 EN**: Starts a function, method, lambda, or structured scope: `llvm::zip(adaptor.getTensorDesc(), offsetsList)) {`.
  **L288 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::zip(adaptor.getTensorDesc(), offsetsList)) {`。

### Lines 289-312

````cpp
      xegpu::PrefetchNdOp::create(rewriter, op.getLoc(), tdesc, offsets,
                                  op.getL1HintAttr(), op.getL2HintAttr(),
                                  op.getL3HintAttr(), layout);
    }
    rewriter.eraseOp(op);

    return success();
  }
};

/// This pattern transforms the DpasOp to work at subgroup level.
struct WgToSgDpasOp : public OpConversionPattern<xegpu::DpasOp> {
  using OpConversionPattern<xegpu::DpasOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(xegpu::DpasOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    VectorType resultTy = op.getResult().getType();
    if (resultTy.getRank() != 2)
      return failure();

    auto layoutCd = op.getLayoutCdAttr();
    auto layoutA = op.getLayoutAAttr();
    auto layoutB = op.getLayoutBAttr();
````
- **L289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::PrefetchNdOp::create(rewriter, op.getLoc(), tdesc, offsets,`.
  **L289 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::PrefetchNdOp::create(rewriter, op.getLoc(), tdesc, offsets,`。
- **L290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op.getL1HintAttr(), op.getL2HintAttr(),`.
  **L290 CN**: 继续一个多行参数列表、初始化器或聚合项：`op.getL1HintAttr(), op.getL2HintAttr(),`。
- **L291 EN**: Executes a call or declaration centered on `op.getL3HintAttr`.
  **L291 CN**: 执行以 `op.getL3HintAttr` 为核心的调用或声明。
- **L292 EN**: Closes the current lexical scope or compound statement.
  **L292 CN**: 结束当前词法作用域或复合语句块。
- **L293 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L293 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L295 EN**: Returns from the current function with `success()`.
  **L295 CN**: 以 `success()` 从当前函数返回。
- **L296 EN**: Closes the current lexical scope or compound statement.
  **L296 CN**: 结束当前词法作用域或复合语句块。
- **L297 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L297 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L298 EN**: Blank line separating nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L299 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms the DpasOp to work at subgroup level.`.
  **L299 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms the DpasOp to work at subgroup level.`。
- **L300 EN**: Declares struct `WgToSgDpasOp`.
  **L300 CN**: 声明 struct `WgToSgDpasOp`。
- **L301 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::DpasOp>::OpConversionPattern;`.
  **L301 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::DpasOp>::OpConversionPattern;`。
- **L302 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L302 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::DpasOp op, OneToNOpAdaptor adaptor,`.
  **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::DpasOp op, OneToNOpAdaptor adaptor,`。
- **L304 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L304 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L305 EN**: Initializes variable `loc` from the right-hand expression.
  **L305 CN**: 使用右侧表达式初始化变量 `loc`。
- **L306 EN**: Initializes variable `resultTy` from the right-hand expression.
  **L306 CN**: 使用右侧表达式初始化变量 `resultTy`。
- **L307 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L307 CN**: 开始 `if` 控制流语句并计算其条件。
- **L308 EN**: Returns from the current function with `failure()`.
  **L308 CN**: 以 `failure()` 从当前函数返回。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L310 EN**: Initializes variable `layoutCd` from the right-hand expression.
  **L310 CN**: 使用右侧表达式初始化变量 `layoutCd`。
- **L311 EN**: Initializes variable `layoutA` from the right-hand expression.
  **L311 CN**: 使用右侧表达式初始化变量 `layoutA`。
- **L312 EN**: Initializes variable `layoutB` from the right-hand expression.
  **L312 CN**: 使用右侧表达式初始化变量 `layoutB`。

### Lines 313-336

````cpp
    if (!layoutCd || !layoutA || !layoutB)
      return failure();
    size_t i = 0;
    SmallVector<Value> newDpasOps;
    for (auto aVec : adaptor.getLhs()) {
      for (auto bVec : adaptor.getRhs()) {

        llvm::SmallVector<Value> operands({aVec, bVec});
        Value tmpC;
        if (op.getAcc()) {
          tmpC = adaptor.getAcc()[i++];
          operands.push_back(tmpC);
        }

        ArrayRef<int64_t> aVecShape =
            cast<VectorType>(aVec.getType()).getShape();
        ArrayRef<int64_t> bVecShape =
            cast<VectorType>(bVec.getType()).getShape();
        VectorType resTy = VectorType::get({aVecShape[0], bVecShape[1]},
                                           resultTy.getElementType());
        auto newDpasOp = xegpu::DpasOp::create(rewriter, loc, resTy, operands);
        newDpasOp.setLayoutCdAttr(layoutCd.dropSgLayoutAndData());
        newDpasOp.setLayoutAAttr(layoutA.dropSgLayoutAndData());
        newDpasOp.setLayoutBAttr(layoutB.dropSgLayoutAndData());
````
- **L313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L314 EN**: Returns from the current function with `failure()`.
  **L314 CN**: 以 `failure()` 从当前函数返回。
- **L315 EN**: Initializes variable `i` from the right-hand expression.
  **L315 CN**: 使用右侧表达式初始化变量 `i`。
- **L316 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newDpasOps;`.
  **L316 CN**: 执行一条独立语句或声明：`SmallVector<Value> newDpasOps;`。
- **L317 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L317 CN**: 开始 `for` 控制流语句并计算其条件。
- **L318 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L318 CN**: 开始 `for` 控制流语句并计算其条件。
- **L319 EN**: Blank line separating nearby declarations or logic blocks.
  **L319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L320 EN**: Executes a call or declaration centered on `operands`.
  **L320 CN**: 执行以 `operands` 为核心的调用或声明。
- **L321 EN**: Executes a standalone statement or declaration: `Value tmpC;`.
  **L321 CN**: 执行一条独立语句或声明：`Value tmpC;`。
- **L322 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L322 CN**: 开始 `if` 控制流语句并计算其条件。
- **L323 EN**: Executes a call or declaration centered on `adaptor.getAcc`.
  **L323 CN**: 执行以 `adaptor.getAcc` 为核心的调用或声明。
- **L324 EN**: Executes a call or declaration centered on `operands.push_back`.
  **L324 CN**: 执行以 `operands.push_back` 为核心的调用或声明。
- **L325 EN**: Closes the current lexical scope or compound statement.
  **L325 CN**: 结束当前词法作用域或复合语句块。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L327 EN**: Continues the surrounding expression or declaration: `ArrayRef<int64_t> aVecShape =`.
  **L327 CN**: 继续构造周围的表达式或声明：`ArrayRef<int64_t> aVecShape =`。
- **L328 EN**: Executes a call or declaration centered on `cast<VectorType>`.
  **L328 CN**: 执行以 `cast<VectorType>` 为核心的调用或声明。
- **L329 EN**: Continues the surrounding expression or declaration: `ArrayRef<int64_t> bVecShape =`.
  **L329 CN**: 继续构造周围的表达式或声明：`ArrayRef<int64_t> bVecShape =`。
- **L330 EN**: Executes a call or declaration centered on `cast<VectorType>`.
  **L330 CN**: 执行以 `cast<VectorType>` 为核心的调用或声明。
- **L331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VectorType resTy = VectorType::get({aVecShape[0], bVecShape[1]},`.
  **L331 CN**: 继续一个多行参数列表、初始化器或聚合项：`VectorType resTy = VectorType::get({aVecShape[0], bVecShape[1]},`。
- **L332 EN**: Executes a call or declaration centered on `resultTy.getElementType`.
  **L332 CN**: 执行以 `resultTy.getElementType` 为核心的调用或声明。
- **L333 EN**: Initializes variable `newDpasOp` from the right-hand expression.
  **L333 CN**: 使用右侧表达式初始化变量 `newDpasOp`。
- **L334 EN**: Executes a call or declaration centered on `newDpasOp.setLayoutCdAttr`.
  **L334 CN**: 执行以 `newDpasOp.setLayoutCdAttr` 为核心的调用或声明。
- **L335 EN**: Executes a call or declaration centered on `newDpasOp.setLayoutAAttr`.
  **L335 CN**: 执行以 `newDpasOp.setLayoutAAttr` 为核心的调用或声明。
- **L336 EN**: Executes a call or declaration centered on `newDpasOp.setLayoutBAttr`.
  **L336 CN**: 执行以 `newDpasOp.setLayoutBAttr` 为核心的调用或声明。

### Lines 337-360

````cpp

        newDpasOps.push_back(newDpasOp);
      }
    }
    rewriter.replaceOpWithMultiple(op, {newDpasOps});
    return success();
  }
};

/// This pattern transforms the DpasMxOp to work at subgroup level.
struct WgToSgDpasMxOp : public OpConversionPattern<xegpu::DpasMxOp> {
  using OpConversionPattern<xegpu::DpasMxOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(xegpu::DpasMxOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {

    Location loc = op.getLoc();
    VectorType resultTy = op.getResult().getType();

    if (resultTy.getRank() != 2)
      return failure();

    auto layoutCd = op.getLayoutCdAttr();
    auto layoutA = op.getLayoutAAttr();
````
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L338 EN**: Executes a call or declaration centered on `newDpasOps.push_back`.
  **L338 CN**: 执行以 `newDpasOps.push_back` 为核心的调用或声明。
- **L339 EN**: Closes the current lexical scope or compound statement.
  **L339 CN**: 结束当前词法作用域或复合语句块。
- **L340 EN**: Closes the current lexical scope or compound statement.
  **L340 CN**: 结束当前词法作用域或复合语句块。
- **L341 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L341 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L342 EN**: Returns from the current function with `success()`.
  **L342 CN**: 以 `success()` 从当前函数返回。
- **L343 EN**: Closes the current lexical scope or compound statement.
  **L343 CN**: 结束当前词法作用域或复合语句块。
- **L344 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L344 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L345 EN**: Blank line separating nearby declarations or logic blocks.
  **L345 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L346 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms the DpasMxOp to work at subgroup level.`.
  **L346 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms the DpasMxOp to work at subgroup level.`。
- **L347 EN**: Declares struct `WgToSgDpasMxOp`.
  **L347 CN**: 声明 struct `WgToSgDpasMxOp`。
- **L348 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::DpasMxOp>::OpConversionPattern;`.
  **L348 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::DpasMxOp>::OpConversionPattern;`。
- **L349 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L349 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::DpasMxOp op, OneToNOpAdaptor adaptor,`.
  **L350 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::DpasMxOp op, OneToNOpAdaptor adaptor,`。
- **L351 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L351 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L353 EN**: Initializes variable `loc` from the right-hand expression.
  **L353 CN**: 使用右侧表达式初始化变量 `loc`。
- **L354 EN**: Initializes variable `resultTy` from the right-hand expression.
  **L354 CN**: 使用右侧表达式初始化变量 `resultTy`。
- **L355 EN**: Blank line separating nearby declarations or logic blocks.
  **L355 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L356 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L356 CN**: 开始 `if` 控制流语句并计算其条件。
- **L357 EN**: Returns from the current function with `failure()`.
  **L357 CN**: 以 `failure()` 从当前函数返回。
- **L358 EN**: Blank line separating nearby declarations or logic blocks.
  **L358 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L359 EN**: Initializes variable `layoutCd` from the right-hand expression.
  **L359 CN**: 使用右侧表达式初始化变量 `layoutCd`。
- **L360 EN**: Initializes variable `layoutA` from the right-hand expression.
  **L360 CN**: 使用右侧表达式初始化变量 `layoutA`。

### Lines 361-384

````cpp
    auto layoutB = op.getLayoutBAttr();
    auto layoutAScale = op.getLayoutAScaleAttr();
    auto layoutBScale = op.getLayoutBScaleAttr();

    if (!layoutCd || !layoutA || !layoutB || !layoutAScale || !layoutBScale)
      return failure();

    size_t index_c = 0;
    SmallVector<Value> newDpasMxOps;
    for (auto [index_a, aVec] : llvm::enumerate(adaptor.getA())) {
      for (auto [index_b, bVec] : llvm::enumerate(adaptor.getB())) {
        Value accVal = (op.getAcc()) ? adaptor.getAcc()[index_c++] : Value();
        Value scaleAVal =
            (op.getScaleA()) ? adaptor.getScaleA()[index_a] : Value();
        Value scaleBVal =
            (op.getScaleB()) ? adaptor.getScaleB()[index_b] : Value();

        ArrayRef<int64_t> aVecShape =
            cast<VectorType>(aVec.getType()).getShape();
        ArrayRef<int64_t> bVecShape =
            cast<VectorType>(bVec.getType()).getShape();
        VectorType resTy = VectorType::get({aVecShape[0], bVecShape[1]},
                                           resultTy.getElementType());
        auto newDpasMxOp = xegpu::DpasMxOp::create(
````
- **L361 EN**: Initializes variable `layoutB` from the right-hand expression.
  **L361 CN**: 使用右侧表达式初始化变量 `layoutB`。
- **L362 EN**: Initializes variable `layoutAScale` from the right-hand expression.
  **L362 CN**: 使用右侧表达式初始化变量 `layoutAScale`。
- **L363 EN**: Initializes variable `layoutBScale` from the right-hand expression.
  **L363 CN**: 使用右侧表达式初始化变量 `layoutBScale`。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L365 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L365 CN**: 开始 `if` 控制流语句并计算其条件。
- **L366 EN**: Returns from the current function with `failure()`.
  **L366 CN**: 以 `failure()` 从当前函数返回。
- **L367 EN**: Blank line separating nearby declarations or logic blocks.
  **L367 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L368 EN**: Initializes variable `index_c` from the right-hand expression.
  **L368 CN**: 使用右侧表达式初始化变量 `index_c`。
- **L369 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newDpasMxOps;`.
  **L369 CN**: 执行一条独立语句或声明：`SmallVector<Value> newDpasMxOps;`。
- **L370 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L370 CN**: 开始 `for` 控制流语句并计算其条件。
- **L371 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L371 CN**: 开始 `for` 控制流语句并计算其条件。
- **L372 EN**: Initializes variable `accVal` from the right-hand expression.
  **L372 CN**: 使用右侧表达式初始化变量 `accVal`。
- **L373 EN**: Continues the surrounding expression or declaration: `Value scaleAVal =`.
  **L373 CN**: 继续构造周围的表达式或声明：`Value scaleAVal =`。
- **L374 EN**: Executes a call or declaration centered on `statement`.
  **L374 CN**: 执行以 `statement` 为核心的调用或声明。
- **L375 EN**: Continues the surrounding expression or declaration: `Value scaleBVal =`.
  **L375 CN**: 继续构造周围的表达式或声明：`Value scaleBVal =`。
- **L376 EN**: Executes a call or declaration centered on `statement`.
  **L376 CN**: 执行以 `statement` 为核心的调用或声明。
- **L377 EN**: Blank line separating nearby declarations or logic blocks.
  **L377 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L378 EN**: Continues the surrounding expression or declaration: `ArrayRef<int64_t> aVecShape =`.
  **L378 CN**: 继续构造周围的表达式或声明：`ArrayRef<int64_t> aVecShape =`。
- **L379 EN**: Executes a call or declaration centered on `cast<VectorType>`.
  **L379 CN**: 执行以 `cast<VectorType>` 为核心的调用或声明。
- **L380 EN**: Continues the surrounding expression or declaration: `ArrayRef<int64_t> bVecShape =`.
  **L380 CN**: 继续构造周围的表达式或声明：`ArrayRef<int64_t> bVecShape =`。
- **L381 EN**: Executes a call or declaration centered on `cast<VectorType>`.
  **L381 CN**: 执行以 `cast<VectorType>` 为核心的调用或声明。
- **L382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VectorType resTy = VectorType::get({aVecShape[0], bVecShape[1]},`.
  **L382 CN**: 继续一个多行参数列表、初始化器或聚合项：`VectorType resTy = VectorType::get({aVecShape[0], bVecShape[1]},`。
- **L383 EN**: Executes a call or declaration centered on `resultTy.getElementType`.
  **L383 CN**: 执行以 `resultTy.getElementType` 为核心的调用或声明。
- **L384 EN**: Continues logic associated with callable symbol `create`.
  **L384 CN**: 继续与可调用符号 `create` 相关的逻辑。

### Lines 385-408

````cpp
            rewriter, loc, resTy, aVec, bVec, accVal, scaleAVal, scaleBVal,
            layoutA.dropSgLayoutAndData(), layoutB.dropSgLayoutAndData(),
            layoutCd.dropSgLayoutAndData(), layoutAScale.dropSgLayoutAndData(),
            layoutBScale.dropSgLayoutAndData());

        newDpasMxOps.push_back(newDpasMxOp);
      }
    }
    rewriter.replaceOpWithMultiple(op, {newDpasMxOps});
    return success();
  }
};

/// This pattern transforms vector.broadcast ops to work at subgroup level.
struct WgToSgVectorBroadcastOp
    : public OpConversionPattern<vector::BroadcastOp> {
  using OpConversionPattern<vector::BroadcastOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(vector::BroadcastOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {

    VectorType resultType = op.getResult().getType();
    ArrayRef<int64_t> wgShape = resultType.getShape();
````
- **L385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, resTy, aVec, bVec, accVal, scaleAVal, scaleBVal,`.
  **L385 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, resTy, aVec, bVec, accVal, scaleAVal, scaleBVal,`。
- **L386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `layoutA.dropSgLayoutAndData(), layoutB.dropSgLayoutAndData(),`.
  **L386 CN**: 继续一个多行参数列表、初始化器或聚合项：`layoutA.dropSgLayoutAndData(), layoutB.dropSgLayoutAndData(),`。
- **L387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `layoutCd.dropSgLayoutAndData(), layoutAScale.dropSgLayoutAndData(),`.
  **L387 CN**: 继续一个多行参数列表、初始化器或聚合项：`layoutCd.dropSgLayoutAndData(), layoutAScale.dropSgLayoutAndData(),`。
- **L388 EN**: Executes a call or declaration centered on `layoutBScale.dropSgLayoutAndData`.
  **L388 CN**: 执行以 `layoutBScale.dropSgLayoutAndData` 为核心的调用或声明。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L390 EN**: Executes a call or declaration centered on `newDpasMxOps.push_back`.
  **L390 CN**: 执行以 `newDpasMxOps.push_back` 为核心的调用或声明。
- **L391 EN**: Closes the current lexical scope or compound statement.
  **L391 CN**: 结束当前词法作用域或复合语句块。
- **L392 EN**: Closes the current lexical scope or compound statement.
  **L392 CN**: 结束当前词法作用域或复合语句块。
- **L393 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L393 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L394 EN**: Returns from the current function with `success()`.
  **L394 CN**: 以 `success()` 从当前函数返回。
- **L395 EN**: Closes the current lexical scope or compound statement.
  **L395 CN**: 结束当前词法作用域或复合语句块。
- **L396 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L396 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L397 EN**: Blank line separating nearby declarations or logic blocks.
  **L397 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L398 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms vector.broadcast ops to work at subgroup level.`.
  **L398 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms vector.broadcast ops to work at subgroup level.`。
- **L399 EN**: Declares struct `WgToSgVectorBroadcastOp`.
  **L399 CN**: 声明 struct `WgToSgVectorBroadcastOp`。
- **L400 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<vector::BroadcastOp> {`.
  **L400 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<vector::BroadcastOp> {`。
- **L401 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<vector::BroadcastOp>::OpConversionPattern;`.
  **L401 CN**: 执行一条独立语句或声明：`using OpConversionPattern<vector::BroadcastOp>::OpConversionPattern;`。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L403 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L403 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(vector::BroadcastOp op, OneToNOpAdaptor adaptor,`.
  **L404 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(vector::BroadcastOp op, OneToNOpAdaptor adaptor,`。
- **L405 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L405 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L406 EN**: Blank line separating nearby declarations or logic blocks.
  **L406 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L407 EN**: Initializes variable `resultType` from the right-hand expression.
  **L407 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L408 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L408 CN**: 使用右侧表达式初始化变量 `wgShape`。

### Lines 409-432

````cpp

    xegpu::DistributeLayoutAttr layout =
        xegpu::getTemporaryLayout(llvm::cast<OpResult>(op.getResult()));
    if (!layout || !layout.isForWorkgroup())
      return failure();

    SmallVector<int64_t> sgShape;
    int count;
    std::tie(sgShape, count) = getSgShapeAndCount(wgShape, layout);
    VectorType newResultType =
        VectorType::get(sgShape, resultType.getElementType());

    SmallVector<Value> newBroadcastOps;
    auto distSource = adaptor.getOperands().front();
    int numDistributions = count / distSource.size();
    for (int i = 0; i < numDistributions; ++i) {
      for (auto operand : distSource) {
        auto newBroadcast = vector::BroadcastOp::create(rewriter, op.getLoc(),
                                                        newResultType, operand);

        newBroadcastOps.push_back(newBroadcast.getResult());
      }
    }
    rewriter.replaceOpWithMultiple(op, {newBroadcastOps});
````
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L410 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout =`.
  **L410 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout =`。
- **L411 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L411 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L412 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L412 CN**: 开始 `if` 控制流语句并计算其条件。
- **L413 EN**: Returns from the current function with `failure()`.
  **L413 CN**: 以 `failure()` 从当前函数返回。
- **L414 EN**: Blank line separating nearby declarations or logic blocks.
  **L414 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L415 EN**: Executes a standalone statement or declaration: `SmallVector<int64_t> sgShape;`.
  **L415 CN**: 执行一条独立语句或声明：`SmallVector<int64_t> sgShape;`。
- **L416 EN**: Executes a standalone statement or declaration: `int count;`.
  **L416 CN**: 执行一条独立语句或声明：`int count;`。
- **L417 EN**: Executes a call or declaration centered on `std::tie`.
  **L417 CN**: 执行以 `std::tie` 为核心的调用或声明。
- **L418 EN**: Continues the surrounding expression or declaration: `VectorType newResultType =`.
  **L418 CN**: 继续构造周围的表达式或声明：`VectorType newResultType =`。
- **L419 EN**: Executes a call or declaration centered on `VectorType::get`.
  **L419 CN**: 执行以 `VectorType::get` 为核心的调用或声明。
- **L420 EN**: Blank line separating nearby declarations or logic blocks.
  **L420 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L421 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newBroadcastOps;`.
  **L421 CN**: 执行一条独立语句或声明：`SmallVector<Value> newBroadcastOps;`。
- **L422 EN**: Initializes variable `distSource` from the right-hand expression.
  **L422 CN**: 使用右侧表达式初始化变量 `distSource`。
- **L423 EN**: Initializes variable `numDistributions` from the right-hand expression.
  **L423 CN**: 使用右侧表达式初始化变量 `numDistributions`。
- **L424 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L424 CN**: 开始 `for` 控制流语句并计算其条件。
- **L425 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L425 CN**: 开始 `for` 控制流语句并计算其条件。
- **L426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto newBroadcast = vector::BroadcastOp::create(rewriter, op.getLoc(),`.
  **L426 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto newBroadcast = vector::BroadcastOp::create(rewriter, op.getLoc(),`。
- **L427 EN**: Executes a standalone statement or declaration: `newResultType, operand);`.
  **L427 CN**: 执行一条独立语句或声明：`newResultType, operand);`。
- **L428 EN**: Blank line separating nearby declarations or logic blocks.
  **L428 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L429 EN**: Executes a call or declaration centered on `newBroadcastOps.push_back`.
  **L429 CN**: 执行以 `newBroadcastOps.push_back` 为核心的调用或声明。
- **L430 EN**: Closes the current lexical scope or compound statement.
  **L430 CN**: 结束当前词法作用域或复合语句块。
- **L431 EN**: Closes the current lexical scope or compound statement.
  **L431 CN**: 结束当前词法作用域或复合语句块。
- **L432 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L432 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。

### Lines 433-456

````cpp
    return success();
  }
};

// This pattern transforms elementwise ops to work at subgroup level.
struct WgToSgElementwiseOp : public ConversionPattern {
  WgToSgElementwiseOp(MLIRContext *ctx)
      : ConversionPattern(MatchAnyOpTypeTag(), /*benefit=*/1, ctx) {}

  LogicalResult
  matchAndRewrite(Operation *op, ArrayRef<ValueRange> operands,
                  ConversionPatternRewriter &rewriter) const override {
    // Only match ops with elementwise trait and single result.
    if (!OpTrait::hasElementwiseMappableTraits(op) || op->getNumResults() != 1)
      return failure();

    auto resultType = dyn_cast<VectorType>(op->getResult(0).getType());
    assert(resultType && "Expected result to be a VectorType");

    ArrayRef<int64_t> wgShape = resultType.getShape();

    xegpu::DistributeLayoutAttr layout =
        xegpu::getTemporaryLayout(llvm::cast<OpResult>(op->getResult(0)));
    if (!layout || !layout.isForWorkgroup())
````
- **L433 EN**: Returns from the current function with `success()`.
  **L433 CN**: 以 `success()` 从当前函数返回。
- **L434 EN**: Closes the current lexical scope or compound statement.
  **L434 CN**: 结束当前词法作用域或复合语句块。
- **L435 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L435 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L436 EN**: Blank line separating nearby declarations or logic blocks.
  **L436 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L437 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms elementwise ops to work at subgroup level.`.
  **L437 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms elementwise ops to work at subgroup level.`。
- **L438 EN**: Declares struct `WgToSgElementwiseOp`.
  **L438 CN**: 声明 struct `WgToSgElementwiseOp`。
- **L439 EN**: Continues logic associated with callable symbol `WgToSgElementwiseOp`.
  **L439 CN**: 继续与可调用符号 `WgToSgElementwiseOp` 相关的逻辑。
- **L440 EN**: Continues logic associated with callable symbol `ConversionPattern`.
  **L440 CN**: 继续与可调用符号 `ConversionPattern` 相关的逻辑。
- **L441 EN**: Blank line separating nearby declarations or logic blocks.
  **L441 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L442 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L442 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(Operation *op, ArrayRef<ValueRange> operands,`.
  **L443 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(Operation *op, ArrayRef<ValueRange> operands,`。
- **L444 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L444 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L445 EN**: Comment explains nearby logic, invariants, or intent: `Only match ops with elementwise trait and single result.`.
  **L445 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only match ops with elementwise trait and single result.`。
- **L446 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L446 CN**: 开始 `if` 控制流语句并计算其条件。
- **L447 EN**: Returns from the current function with `failure()`.
  **L447 CN**: 以 `failure()` 从当前函数返回。
- **L448 EN**: Blank line separating nearby declarations or logic blocks.
  **L448 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L449 EN**: Initializes variable `resultType` from the right-hand expression.
  **L449 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L450 EN**: Checks an internal invariant in debug builds.
  **L450 CN**: 在调试构建中检查内部不变式。
- **L451 EN**: Blank line separating nearby declarations or logic blocks.
  **L451 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L452 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L452 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L453 EN**: Blank line separating nearby declarations or logic blocks.
  **L453 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L454 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout =`.
  **L454 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout =`。
- **L455 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L455 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L456 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L456 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 457-480

````cpp
      return failure();

    SmallVector<int64_t> sgShape = getSgShapeAndCount(wgShape, layout).first;

    size_t numVariants = operands.empty() ? 0 : operands.front().size();

    if (llvm::any_of(operands, [&](const ValueRange &operandVec) {
          return operandVec.size() != numVariants;
        }))
      return failure();

    SmallVector<Value> newResults;
    VectorType newResultType =
        VectorType::get(sgShape, resultType.getElementType());

    for (size_t i = 0; i < numVariants; ++i) {
      SmallVector<Value> opOperands;
      for (auto &operandVec : operands)
        opOperands.push_back(operandVec[i]);

      OperationState state(op->getLoc(), op->getName());
      state.addOperands(opOperands);
      state.addTypes(newResultType);
      state.addAttributes(op->getAttrs());
````
- **L457 EN**: Returns from the current function with `failure()`.
  **L457 CN**: 以 `failure()` 从当前函数返回。
- **L458 EN**: Blank line separating nearby declarations or logic blocks.
  **L458 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L459 EN**: Initializes variable `sgShape` from the right-hand expression.
  **L459 CN**: 使用右侧表达式初始化变量 `sgShape`。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L461 EN**: Initializes variable `numVariants` from the right-hand expression.
  **L461 CN**: 使用右侧表达式初始化变量 `numVariants`。
- **L462 EN**: Blank line separating nearby declarations or logic blocks.
  **L462 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L463 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L463 CN**: 开始 `if` 控制流语句并计算其条件。
- **L464 EN**: Returns from the current function with `operandVec.size() != numVariants`.
  **L464 CN**: 以 `operandVec.size() != numVariants` 从当前函数返回。
- **L465 EN**: Continues the surrounding expression or declaration: `}))`.
  **L465 CN**: 继续构造周围的表达式或声明：`}))`。
- **L466 EN**: Returns from the current function with `failure()`.
  **L466 CN**: 以 `failure()` 从当前函数返回。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L468 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newResults;`.
  **L468 CN**: 执行一条独立语句或声明：`SmallVector<Value> newResults;`。
- **L469 EN**: Continues the surrounding expression or declaration: `VectorType newResultType =`.
  **L469 CN**: 继续构造周围的表达式或声明：`VectorType newResultType =`。
- **L470 EN**: Executes a call or declaration centered on `VectorType::get`.
  **L470 CN**: 执行以 `VectorType::get` 为核心的调用或声明。
- **L471 EN**: Blank line separating nearby declarations or logic blocks.
  **L471 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L472 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L472 CN**: 开始 `for` 控制流语句并计算其条件。
- **L473 EN**: Executes a standalone statement or declaration: `SmallVector<Value> opOperands;`.
  **L473 CN**: 执行一条独立语句或声明：`SmallVector<Value> opOperands;`。
- **L474 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L474 CN**: 开始 `for` 控制流语句并计算其条件。
- **L475 EN**: Executes a call or declaration centered on `opOperands.push_back`.
  **L475 CN**: 执行以 `opOperands.push_back` 为核心的调用或声明。
- **L476 EN**: Blank line separating nearby declarations or logic blocks.
  **L476 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L477 EN**: Executes a call or declaration centered on `state`.
  **L477 CN**: 执行以 `state` 为核心的调用或声明。
- **L478 EN**: Executes a call or declaration centered on `state.addOperands`.
  **L478 CN**: 执行以 `state.addOperands` 为核心的调用或声明。
- **L479 EN**: Executes a call or declaration centered on `state.addTypes`.
  **L479 CN**: 执行以 `state.addTypes` 为核心的调用或声明。
- **L480 EN**: Executes a call or declaration centered on `state.addAttributes`.
  **L480 CN**: 执行以 `state.addAttributes` 为核心的调用或声明。

### Lines 481-504

````cpp
      Operation *newOp = rewriter.create(state);
      xegpu::removeLayoutAttrs(newOp);
      newResults.push_back(newOp->getResult(0));
    }

    rewriter.replaceOpWithMultiple(op, {newResults});
    return success();
  }
};

// clang-format off
// Pattern for lowering ConvertLayoutOp based on sg_layout and sg_data.
// If input_layout and target_layout have identical sg_layout and sg_data,
// the op is rewritten to a subgroup-level ConvertLayoutOp with these fields
// dropped. For example:
//   #a = #xegpu.layout<sg_layout = [2, 2], sg_data = [16, 16], inst_data = [16, 16]>
//   #b = #xegpu.layout<sg_layout = [2, 2], sg_data = [16, 16], inst_data = [8, 16]>
//   xegpu.convert_layout %1 <{input_layout = #a, target_layout = #b}> : vector<32x64xf32>
// becomes:
//   #a = #xegpu.layout<inst_data = [16, 16]>
//   #b = #xegpu.layout<inst_data = [8, 16]>
//   xegpu.convert_layout %1 <{input_layout = #a, target_layout = #b}> : vector<16x16xf32>
// (vector<16x16xf32> is determined by sg_data = [16, 16])
//
````
- **L481 EN**: Executes a call or declaration centered on `rewriter.create`.
  **L481 CN**: 执行以 `rewriter.create` 为核心的调用或声明。
- **L482 EN**: Executes a call or declaration centered on `xegpu::removeLayoutAttrs`.
  **L482 CN**: 执行以 `xegpu::removeLayoutAttrs` 为核心的调用或声明。
- **L483 EN**: Executes a call or declaration centered on `newResults.push_back`.
  **L483 CN**: 执行以 `newResults.push_back` 为核心的调用或声明。
- **L484 EN**: Closes the current lexical scope or compound statement.
  **L484 CN**: 结束当前词法作用域或复合语句块。
- **L485 EN**: Blank line separating nearby declarations or logic blocks.
  **L485 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L486 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L486 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L487 EN**: Returns from the current function with `success()`.
  **L487 CN**: 以 `success()` 从当前函数返回。
- **L488 EN**: Closes the current lexical scope or compound statement.
  **L488 CN**: 结束当前词法作用域或复合语句块。
- **L489 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L489 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L490 EN**: Blank line separating nearby declarations or logic blocks.
  **L490 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L491 EN**: Comment explains nearby logic, invariants, or intent: `clang-format off`.
  **L491 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`clang-format off`。
- **L492 EN**: Comment explains nearby logic, invariants, or intent: `Pattern for lowering ConvertLayoutOp based on sg_layout and sg_data.`.
  **L492 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Pattern for lowering ConvertLayoutOp based on sg_layout and sg_data.`。
- **L493 EN**: Comment explains nearby logic, invariants, or intent: `If input_layout and target_layout have identical sg_layout and sg_data,`.
  **L493 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If input_layout and target_layout have identical sg_layout and sg_data,`。
- **L494 EN**: Comment explains nearby logic, invariants, or intent: `the op is rewritten to a subgroup-level ConvertLayoutOp with these fields`.
  **L494 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the op is rewritten to a subgroup-level ConvertLayoutOp with these fields`。
- **L495 EN**: Comment explains nearby logic, invariants, or intent: `dropped. For example:`.
  **L495 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`dropped. For example:`。
- **L496 EN**: Comment explains nearby logic, invariants, or intent: `#a = #xegpu.layout<sg_layout = [2, 2], sg_data = [16, 16], inst_data = [16, 16]>`.
  **L496 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`#a = #xegpu.layout<sg_layout = [2, 2], sg_data = [16, 16], inst_data = [16, 16]>`。
- **L497 EN**: Comment explains nearby logic, invariants, or intent: `#b = #xegpu.layout<sg_layout = [2, 2], sg_data = [16, 16], inst_data = [8, 16]>`.
  **L497 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`#b = #xegpu.layout<sg_layout = [2, 2], sg_data = [16, 16], inst_data = [8, 16]>`。
- **L498 EN**: Comment explains nearby logic, invariants, or intent: `xegpu.convert_layout %1 <{input_layout = #a, target_layout = #b}> : vector<32x64xf32>`.
  **L498 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`xegpu.convert_layout %1 <{input_layout = #a, target_layout = #b}> : vector<32x64xf32>`。
- **L499 EN**: Comment explains nearby logic, invariants, or intent: `becomes:`.
  **L499 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`becomes:`。
- **L500 EN**: Comment explains nearby logic, invariants, or intent: `#a = #xegpu.layout<inst_data = [16, 16]>`.
  **L500 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`#a = #xegpu.layout<inst_data = [16, 16]>`。
- **L501 EN**: Comment explains nearby logic, invariants, or intent: `#b = #xegpu.layout<inst_data = [8, 16]>`.
  **L501 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`#b = #xegpu.layout<inst_data = [8, 16]>`。
- **L502 EN**: Comment explains nearby logic, invariants, or intent: `xegpu.convert_layout %1 <{input_layout = #a, target_layout = #b}> : vector<16x16xf32>`.
  **L502 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`xegpu.convert_layout %1 <{input_layout = #a, target_layout = #b}> : vector<16x16xf32>`。
- **L503 EN**: Comment explains nearby logic, invariants, or intent: `(vector<16x16xf32> is determined by sg_data = [16, 16])`.
  **L503 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(vector<16x16xf32> is determined by sg_data = [16, 16])`。
- **L504 EN**: Separator comment used for visual grouping.
  **L504 CN**: 用于视觉分组的分隔注释。

### Lines 505-528

````cpp
// If sg_layout or sg_data differ, SLM is used to redistribute data across subgroups.
// For example:
//   #a = #xegpu.layout<sg_layout = [1, 4], sg_data = [32, 16], inst_data = [16, 16]>
//   #b = #xegpu.layout<sg_layout = [2, 2], sg_data = [16, 32], inst_data = [8, 16]>
//   xegpu.convert_layout %1 <{input_layout = #a, target_layout = #b}> : vector<32x64xf32>
// is lowered to:
//   #a = #xegpu.layout<inst_data = [16, 16]>
//   #b = #xegpu.layout<inst_data = [8, 16]>
//   store_matrix %1, %slm <{layout_input_0 = #a}> : vector<32x16>, mem_desc<32x64xf32>
//   %d = load_matrix %slm <{layout_result_0 = #a}> : mem_desc<32x64xf32> -> vector<16x32xf32>
//   xegpu.convert_layout %d <{input_layout = #a, target_layout = #b}> : vector<16x32xf32>
// clang-format on
struct WgToSgConvertLayoutOp
    : public OpConversionPattern<xegpu::ConvertLayoutOp> {
  using OpConversionPattern<xegpu::ConvertLayoutOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(xegpu::ConvertLayoutOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    auto inputLayout = op.getInputLayout();
    auto targetLayout = op.getTargetLayout();

    if (!inputLayout || !targetLayout || !inputLayout.isForWorkgroup() ||
````
- **L505 EN**: Comment explains nearby logic, invariants, or intent: `If sg_layout or sg_data differ, SLM is used to redistribute data across subgroups.`.
  **L505 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If sg_layout or sg_data differ, SLM is used to redistribute data across subgroups.`。
- **L506 EN**: Comment explains nearby logic, invariants, or intent: `For example:`.
  **L506 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For example:`。
- **L507 EN**: Comment explains nearby logic, invariants, or intent: `#a = #xegpu.layout<sg_layout = [1, 4], sg_data = [32, 16], inst_data = [16, 16]>`.
  **L507 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`#a = #xegpu.layout<sg_layout = [1, 4], sg_data = [32, 16], inst_data = [16, 16]>`。
- **L508 EN**: Comment explains nearby logic, invariants, or intent: `#b = #xegpu.layout<sg_layout = [2, 2], sg_data = [16, 32], inst_data = [8, 16]>`.
  **L508 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`#b = #xegpu.layout<sg_layout = [2, 2], sg_data = [16, 32], inst_data = [8, 16]>`。
- **L509 EN**: Comment explains nearby logic, invariants, or intent: `xegpu.convert_layout %1 <{input_layout = #a, target_layout = #b}> : vector<32x64xf32>`.
  **L509 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`xegpu.convert_layout %1 <{input_layout = #a, target_layout = #b}> : vector<32x64xf32>`。
- **L510 EN**: Comment explains nearby logic, invariants, or intent: `is lowered to:`.
  **L510 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`is lowered to:`。
- **L511 EN**: Comment explains nearby logic, invariants, or intent: `#a = #xegpu.layout<inst_data = [16, 16]>`.
  **L511 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`#a = #xegpu.layout<inst_data = [16, 16]>`。
- **L512 EN**: Comment explains nearby logic, invariants, or intent: `#b = #xegpu.layout<inst_data = [8, 16]>`.
  **L512 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`#b = #xegpu.layout<inst_data = [8, 16]>`。
- **L513 EN**: Comment explains nearby logic, invariants, or intent: `store_matrix %1, %slm <{layout_input_0 = #a}> : vector<32x16>, mem_desc<32x64xf32>`.
  **L513 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`store_matrix %1, %slm <{layout_input_0 = #a}> : vector<32x16>, mem_desc<32x64xf32>`。
- **L514 EN**: Comment explains nearby logic, invariants, or intent: `%d = load_matrix %slm <{layout_result_0 = #a}> : mem_desc<32x64xf32> -> vector<16x32xf32>`.
  **L514 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%d = load_matrix %slm <{layout_result_0 = #a}> : mem_desc<32x64xf32> -> vector<16x32xf32>`。
- **L515 EN**: Comment explains nearby logic, invariants, or intent: `xegpu.convert_layout %d <{input_layout = #a, target_layout = #b}> : vector<16x32xf32>`.
  **L515 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`xegpu.convert_layout %d <{input_layout = #a, target_layout = #b}> : vector<16x32xf32>`。
- **L516 EN**: Comment explains nearby logic, invariants, or intent: `clang-format on`.
  **L516 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`clang-format on`。
- **L517 EN**: Declares struct `WgToSgConvertLayoutOp`.
  **L517 CN**: 声明 struct `WgToSgConvertLayoutOp`。
- **L518 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<xegpu::ConvertLayoutOp> {`.
  **L518 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<xegpu::ConvertLayoutOp> {`。
- **L519 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::ConvertLayoutOp>::OpConversionPattern;`.
  **L519 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::ConvertLayoutOp>::OpConversionPattern;`。
- **L520 EN**: Blank line separating nearby declarations or logic blocks.
  **L520 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L521 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L521 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::ConvertLayoutOp op, OneToNOpAdaptor adaptor,`.
  **L522 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::ConvertLayoutOp op, OneToNOpAdaptor adaptor,`。
- **L523 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L523 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L524 EN**: Initializes variable `loc` from the right-hand expression.
  **L524 CN**: 使用右侧表达式初始化变量 `loc`。
- **L525 EN**: Initializes variable `inputLayout` from the right-hand expression.
  **L525 CN**: 使用右侧表达式初始化变量 `inputLayout`。
- **L526 EN**: Initializes variable `targetLayout` from the right-hand expression.
  **L526 CN**: 使用右侧表达式初始化变量 `targetLayout`。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L528 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L528 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 529-552

````cpp
        !targetLayout.isForWorkgroup())
      return rewriter.notifyMatchFailure(
          op, "Input and target layouts must have subgroup layout");

    Type resultType = op.getResult().getType();
    if (resultType.isIntOrFloat()) {
      rewriter.replaceOp(op, op.getSource());
      assert(!inputLayout.dropSgLayoutAndData() &&
             !targetLayout.dropSgLayoutAndData() &&
             "unexpected layout attributes for scalar type");
      return success();
    }

    ArrayRef<int64_t> wgShape = cast<VectorType>(resultType).getShape();
    SmallVector<int64_t> inputSgLayout =
        inputLayout.getEffectiveSgLayoutAsInt();
    SmallVector<int64_t> inputSgData = inputLayout.getEffectiveSgDataAsInt();
    SmallVector<int64_t> targetSgLayout =
        targetLayout.getEffectiveSgLayoutAsInt();
    SmallVector<int64_t> targetSgData = targetLayout.getEffectiveSgDataAsInt();

    // Fast path: if sg_layout and sg_data are identical, no SLM needed
    SmallVector<int64_t> wgShapeVec(wgShape.begin(), wgShape.end());
    if (inputLayout.isCompatibleWith(targetLayout, wgShapeVec,
````
- **L529 EN**: Continues logic associated with callable symbol `isForWorkgroup`.
  **L529 CN**: 继续与可调用符号 `isForWorkgroup` 相关的逻辑。
- **L530 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L530 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L531 EN**: Executes a standalone statement or declaration: `op, "Input and target layouts must have subgroup layout");`.
  **L531 CN**: 执行一条独立语句或声明：`op, "Input and target layouts must have subgroup layout");`。
- **L532 EN**: Blank line separating nearby declarations or logic blocks.
  **L532 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L533 EN**: Initializes variable `resultType` from the right-hand expression.
  **L533 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L534 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L534 CN**: 开始 `if` 控制流语句并计算其条件。
- **L535 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L535 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L536 EN**: Checks an internal invariant in debug builds.
  **L536 CN**: 在调试构建中检查内部不变式。
- **L537 EN**: Continues logic associated with callable symbol `dropSgLayoutAndData`.
  **L537 CN**: 继续与可调用符号 `dropSgLayoutAndData` 相关的逻辑。
- **L538 EN**: Executes a standalone statement or declaration: `"unexpected layout attributes for scalar type");`.
  **L538 CN**: 执行一条独立语句或声明：`"unexpected layout attributes for scalar type");`。
- **L539 EN**: Returns from the current function with `success()`.
  **L539 CN**: 以 `success()` 从当前函数返回。
- **L540 EN**: Closes the current lexical scope or compound statement.
  **L540 CN**: 结束当前词法作用域或复合语句块。
- **L541 EN**: Blank line separating nearby declarations or logic blocks.
  **L541 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L542 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L542 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L543 EN**: Continues the surrounding expression or declaration: `SmallVector<int64_t> inputSgLayout =`.
  **L543 CN**: 继续构造周围的表达式或声明：`SmallVector<int64_t> inputSgLayout =`。
- **L544 EN**: Executes a call or declaration centered on `inputLayout.getEffectiveSgLayoutAsInt`.
  **L544 CN**: 执行以 `inputLayout.getEffectiveSgLayoutAsInt` 为核心的调用或声明。
- **L545 EN**: Initializes variable `inputSgData` from the right-hand expression.
  **L545 CN**: 使用右侧表达式初始化变量 `inputSgData`。
- **L546 EN**: Continues the surrounding expression or declaration: `SmallVector<int64_t> targetSgLayout =`.
  **L546 CN**: 继续构造周围的表达式或声明：`SmallVector<int64_t> targetSgLayout =`。
- **L547 EN**: Executes a call or declaration centered on `targetLayout.getEffectiveSgLayoutAsInt`.
  **L547 CN**: 执行以 `targetLayout.getEffectiveSgLayoutAsInt` 为核心的调用或声明。
- **L548 EN**: Initializes variable `targetSgData` from the right-hand expression.
  **L548 CN**: 使用右侧表达式初始化变量 `targetSgData`。
- **L549 EN**: Blank line separating nearby declarations or logic blocks.
  **L549 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L550 EN**: Comment explains nearby logic, invariants, or intent: `Fast path: if sg_layout and sg_data are identical, no SLM needed`.
  **L550 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Fast path: if sg_layout and sg_data are identical, no SLM needed`。
- **L551 EN**: Executes a call or declaration centered on `wgShapeVec`.
  **L551 CN**: 执行以 `wgShapeVec` 为核心的调用或声明。
- **L552 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L552 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 553-576

````cpp
                                     xegpu::LayoutKind::Subgroup)) {
      inputLayout = inputLayout.dropSgLayoutAndData();
      targetLayout = targetLayout.dropSgLayoutAndData();

      SmallVector<Value> newOps(adaptor.getSource());
      if (inputLayout && targetLayout) {
        for (auto [i, src] : llvm::enumerate(adaptor.getSource())) {
          auto newOp = xegpu::ConvertLayoutOp::create(
              rewriter, loc, src.getType(), src, inputLayout, targetLayout);
          newOps[i] = newOp;
        }
      }
      rewriter.replaceOpWithMultiple(op, {newOps});
      return success();
    }

    // SLM path: layouts differ, need cross-subgroup data redistribution
    Type elemTy = cast<VectorType>(op.getSource().getType()).getElementType();

    SmallVector<int64_t> slmShape = llvm::to_vector(wgShape);

    // Calculate SLM size requirements
    auto bitWidth = elemTy.getIntOrFloatBitWidth();
    auto bytesPerElement = bitWidth / 8;
````
- **L553 EN**: Continues the surrounding expression or declaration: `xegpu::LayoutKind::Subgroup)) {`.
  **L553 CN**: 继续构造周围的表达式或声明：`xegpu::LayoutKind::Subgroup)) {`。
- **L554 EN**: Executes a call or declaration centered on `inputLayout.dropSgLayoutAndData`.
  **L554 CN**: 执行以 `inputLayout.dropSgLayoutAndData` 为核心的调用或声明。
- **L555 EN**: Executes a call or declaration centered on `targetLayout.dropSgLayoutAndData`.
  **L555 CN**: 执行以 `targetLayout.dropSgLayoutAndData` 为核心的调用或声明。
- **L556 EN**: Blank line separating nearby declarations or logic blocks.
  **L556 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L557 EN**: Executes a call or declaration centered on `newOps`.
  **L557 CN**: 执行以 `newOps` 为核心的调用或声明。
- **L558 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L558 CN**: 开始 `if` 控制流语句并计算其条件。
- **L559 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L559 CN**: 开始 `for` 控制流语句并计算其条件。
- **L560 EN**: Continues logic associated with callable symbol `create`.
  **L560 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L561 EN**: Executes a call or declaration centered on `src.getType`.
  **L561 CN**: 执行以 `src.getType` 为核心的调用或声明。
- **L562 EN**: Executes a standalone statement or declaration: `newOps[i] = newOp;`.
  **L562 CN**: 执行一条独立语句或声明：`newOps[i] = newOp;`。
- **L563 EN**: Closes the current lexical scope or compound statement.
  **L563 CN**: 结束当前词法作用域或复合语句块。
- **L564 EN**: Closes the current lexical scope or compound statement.
  **L564 CN**: 结束当前词法作用域或复合语句块。
- **L565 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L565 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L566 EN**: Returns from the current function with `success()`.
  **L566 CN**: 以 `success()` 从当前函数返回。
- **L567 EN**: Closes the current lexical scope or compound statement.
  **L567 CN**: 结束当前词法作用域或复合语句块。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L569 EN**: Comment explains nearby logic, invariants, or intent: `SLM path: layouts differ, need cross-subgroup data redistribution`.
  **L569 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SLM path: layouts differ, need cross-subgroup data redistribution`。
- **L570 EN**: Initializes variable `elemTy` from the right-hand expression.
  **L570 CN**: 使用右侧表达式初始化变量 `elemTy`。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L572 EN**: Initializes variable `slmShape` from the right-hand expression.
  **L572 CN**: 使用右侧表达式初始化变量 `slmShape`。
- **L573 EN**: Blank line separating nearby declarations or logic blocks.
  **L573 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L574 EN**: Comment explains nearby logic, invariants, or intent: `Calculate SLM size requirements`.
  **L574 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Calculate SLM size requirements`。
- **L575 EN**: Initializes variable `bitWidth` from the right-hand expression.
  **L575 CN**: 使用右侧表达式初始化变量 `bitWidth`。
- **L576 EN**: Initializes variable `bytesPerElement` from the right-hand expression.
  **L576 CN**: 使用右侧表达式初始化变量 `bytesPerElement`。

### Lines 577-600

````cpp
    auto slmSize = computeProduct(slmShape) * bytesPerElement;

    // Allocate SLM
    auto slmTy = MemRefType::get({slmSize}, rewriter.getI8Type(), {}, 3);
    auto slm = memref::AllocaOp::create(rewriter, loc, slmTy);

    auto memDescType = xegpu::MemDescType::get(rewriter.getContext(), slmShape,
                                               elemTy, nullptr);
    auto memDesc =
        xegpu::CreateMemDescOp::create(rewriter, loc, memDescType, slm);

    auto sgId = gpu::SubgroupIdOp::create(rewriter, loc,
                                          rewriter.getIndexType(), nullptr);

    // STORE PHASE: Each subgroup stores in SLM using input layout
    auto storeCoords = inputLayout.computeDistributedCoords(
        rewriter, loc, sgId.getResult(), wgShape);
    if (failed(storeCoords))
      return failure();

    // Store to SLM
    for (auto [src, coords] : llvm::zip(adaptor.getSource(), *storeCoords)) {
      SmallVector<OpFoldResult> storeMatrixOffsets;
      for (Value coord : coords) {
````
- **L577 EN**: Initializes variable `slmSize` from the right-hand expression.
  **L577 CN**: 使用右侧表达式初始化变量 `slmSize`。
- **L578 EN**: Blank line separating nearby declarations or logic blocks.
  **L578 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L579 EN**: Comment explains nearby logic, invariants, or intent: `Allocate SLM`.
  **L579 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Allocate SLM`。
- **L580 EN**: Initializes variable `slmTy` from the right-hand expression.
  **L580 CN**: 使用右侧表达式初始化变量 `slmTy`。
- **L581 EN**: Initializes variable `slm` from the right-hand expression.
  **L581 CN**: 使用右侧表达式初始化变量 `slm`。
- **L582 EN**: Blank line separating nearby declarations or logic blocks.
  **L582 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto memDescType = xegpu::MemDescType::get(rewriter.getContext(), slmShape,`.
  **L583 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto memDescType = xegpu::MemDescType::get(rewriter.getContext(), slmShape,`。
- **L584 EN**: Executes a standalone statement or declaration: `elemTy, nullptr);`.
  **L584 CN**: 执行一条独立语句或声明：`elemTy, nullptr);`。
- **L585 EN**: Continues the surrounding expression or declaration: `auto memDesc =`.
  **L585 CN**: 继续构造周围的表达式或声明：`auto memDesc =`。
- **L586 EN**: Executes a call or declaration centered on `xegpu::CreateMemDescOp::create`.
  **L586 CN**: 执行以 `xegpu::CreateMemDescOp::create` 为核心的调用或声明。
- **L587 EN**: Blank line separating nearby declarations or logic blocks.
  **L587 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto sgId = gpu::SubgroupIdOp::create(rewriter, loc,`.
  **L588 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto sgId = gpu::SubgroupIdOp::create(rewriter, loc,`。
- **L589 EN**: Executes a call or declaration centered on `rewriter.getIndexType`.
  **L589 CN**: 执行以 `rewriter.getIndexType` 为核心的调用或声明。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L591 EN**: Comment explains nearby logic, invariants, or intent: `STORE PHASE: Each subgroup stores in SLM using input layout`.
  **L591 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`STORE PHASE: Each subgroup stores in SLM using input layout`。
- **L592 EN**: Continues logic associated with callable symbol `computeDistributedCoords`.
  **L592 CN**: 继续与可调用符号 `computeDistributedCoords` 相关的逻辑。
- **L593 EN**: Executes a call or declaration centered on `sgId.getResult`.
  **L593 CN**: 执行以 `sgId.getResult` 为核心的调用或声明。
- **L594 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L594 CN**: 开始 `if` 控制流语句并计算其条件。
- **L595 EN**: Returns from the current function with `failure()`.
  **L595 CN**: 以 `failure()` 从当前函数返回。
- **L596 EN**: Blank line separating nearby declarations or logic blocks.
  **L596 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L597 EN**: Comment explains nearby logic, invariants, or intent: `Store to SLM`.
  **L597 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Store to SLM`。
- **L598 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L598 CN**: 开始 `for` 控制流语句并计算其条件。
- **L599 EN**: Executes a standalone statement or declaration: `SmallVector<OpFoldResult> storeMatrixOffsets;`.
  **L599 CN**: 执行一条独立语句或声明：`SmallVector<OpFoldResult> storeMatrixOffsets;`。
- **L600 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L600 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 601-624

````cpp
        storeMatrixOffsets.push_back(coord);
      }
      xegpu::StoreMatrixOp::create(rewriter, loc, src, memDesc.getResult(),
                                   storeMatrixOffsets, nullptr /*layout*/);
    }

    gpu::BarrierOp::create(rewriter, loc);

    // LOAD PHASE: Each target subgroup loads from SLM using target layout
    auto loadCoords = targetLayout.computeDistributedCoords(
        rewriter, loc, sgId.getResult(), wgShape);
    if (failed(loadCoords))
      return failure();

    VectorType loadType = VectorType::get(targetSgData, elemTy);

    // Load vectors from SLM
    SmallVector<Value> finalResults;
    for (auto coords : *loadCoords) {
      SmallVector<OpFoldResult> loadMatrixOffsets;
      for (Value coord : coords) {
        loadMatrixOffsets.push_back(coord);
      }
      auto loadOp = xegpu::LoadMatrixOp::create(
````
- **L601 EN**: Executes a call or declaration centered on `storeMatrixOffsets.push_back`.
  **L601 CN**: 执行以 `storeMatrixOffsets.push_back` 为核心的调用或声明。
- **L602 EN**: Closes the current lexical scope or compound statement.
  **L602 CN**: 结束当前词法作用域或复合语句块。
- **L603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::StoreMatrixOp::create(rewriter, loc, src, memDesc.getResult(),`.
  **L603 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::StoreMatrixOp::create(rewriter, loc, src, memDesc.getResult(),`。
- **L604 EN**: Executes a standalone statement or declaration: `storeMatrixOffsets, nullptr /*layout*/);`.
  **L604 CN**: 执行一条独立语句或声明：`storeMatrixOffsets, nullptr /*layout*/);`。
- **L605 EN**: Closes the current lexical scope or compound statement.
  **L605 CN**: 结束当前词法作用域或复合语句块。
- **L606 EN**: Blank line separating nearby declarations or logic blocks.
  **L606 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L607 EN**: Executes a call or declaration centered on `gpu::BarrierOp::create`.
  **L607 CN**: 执行以 `gpu::BarrierOp::create` 为核心的调用或声明。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L609 EN**: Comment explains nearby logic, invariants, or intent: `LOAD PHASE: Each target subgroup loads from SLM using target layout`.
  **L609 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LOAD PHASE: Each target subgroup loads from SLM using target layout`。
- **L610 EN**: Continues logic associated with callable symbol `computeDistributedCoords`.
  **L610 CN**: 继续与可调用符号 `computeDistributedCoords` 相关的逻辑。
- **L611 EN**: Executes a call or declaration centered on `sgId.getResult`.
  **L611 CN**: 执行以 `sgId.getResult` 为核心的调用或声明。
- **L612 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L612 CN**: 开始 `if` 控制流语句并计算其条件。
- **L613 EN**: Returns from the current function with `failure()`.
  **L613 CN**: 以 `failure()` 从当前函数返回。
- **L614 EN**: Blank line separating nearby declarations or logic blocks.
  **L614 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L615 EN**: Initializes variable `loadType` from the right-hand expression.
  **L615 CN**: 使用右侧表达式初始化变量 `loadType`。
- **L616 EN**: Blank line separating nearby declarations or logic blocks.
  **L616 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L617 EN**: Comment explains nearby logic, invariants, or intent: `Load vectors from SLM`.
  **L617 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Load vectors from SLM`。
- **L618 EN**: Executes a standalone statement or declaration: `SmallVector<Value> finalResults;`.
  **L618 CN**: 执行一条独立语句或声明：`SmallVector<Value> finalResults;`。
- **L619 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L619 CN**: 开始 `for` 控制流语句并计算其条件。
- **L620 EN**: Executes a standalone statement or declaration: `SmallVector<OpFoldResult> loadMatrixOffsets;`.
  **L620 CN**: 执行一条独立语句或声明：`SmallVector<OpFoldResult> loadMatrixOffsets;`。
- **L621 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L621 CN**: 开始 `for` 控制流语句并计算其条件。
- **L622 EN**: Executes a call or declaration centered on `loadMatrixOffsets.push_back`.
  **L622 CN**: 执行以 `loadMatrixOffsets.push_back` 为核心的调用或声明。
- **L623 EN**: Closes the current lexical scope or compound statement.
  **L623 CN**: 结束当前词法作用域或复合语句块。
- **L624 EN**: Continues logic associated with callable symbol `create`.
  **L624 CN**: 继续与可调用符号 `create` 相关的逻辑。

### Lines 625-648

````cpp
          rewriter, loc, loadType, memDesc.getResult(), loadMatrixOffsets,
          targetLayout.dropSgLayoutAndData());

      finalResults.push_back(loadOp.getResult());
    }

    rewriter.replaceOpWithMultiple(op, {finalResults});
    return success();
  }
};

// Handles UnrealizedConversionCastOp generated during
// SCFStructuralTypeConversions (step 1). This op may appear as either a
// target or source materialization for Vector values, e.g.:
// 1. unrealized_cast %1 : vector<256xf32> to vector<16xf32>, ...
// 2. unrealized_cast %1 : vector<16xf32>, ... to vector<256xf32>
// it could be either 1:N or N:1 cast. In both cases, the pattern
// simply forwards the inputs to the outputs using 1:1 or 1:N interface.
// for example, the following scf::forOp
// ```
// %for = scf.for ... iter_args(%arg1 = %0)->(vector<128x128xf16>) {
//     %n = use(%arg1): vector<128x128xf16>
//     scf.yield %n : vector<128x128xf16>
// }
````
- **L625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, loadType, memDesc.getResult(), loadMatrixOffsets,`.
  **L625 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, loadType, memDesc.getResult(), loadMatrixOffsets,`。
- **L626 EN**: Executes a call or declaration centered on `targetLayout.dropSgLayoutAndData`.
  **L626 CN**: 执行以 `targetLayout.dropSgLayoutAndData` 为核心的调用或声明。
- **L627 EN**: Blank line separating nearby declarations or logic blocks.
  **L627 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L628 EN**: Executes a call or declaration centered on `finalResults.push_back`.
  **L628 CN**: 执行以 `finalResults.push_back` 为核心的调用或声明。
- **L629 EN**: Closes the current lexical scope or compound statement.
  **L629 CN**: 结束当前词法作用域或复合语句块。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L631 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L631 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L632 EN**: Returns from the current function with `success()`.
  **L632 CN**: 以 `success()` 从当前函数返回。
- **L633 EN**: Closes the current lexical scope or compound statement.
  **L633 CN**: 结束当前词法作用域或复合语句块。
- **L634 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L634 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L635 EN**: Blank line separating nearby declarations or logic blocks.
  **L635 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L636 EN**: Comment explains nearby logic, invariants, or intent: `Handles UnrealizedConversionCastOp generated during`.
  **L636 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Handles UnrealizedConversionCastOp generated during`。
- **L637 EN**: Comment explains nearby logic, invariants, or intent: `SCFStructuralTypeConversions (step 1). This op may appear as either a`.
  **L637 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SCFStructuralTypeConversions (step 1). This op may appear as either a`。
- **L638 EN**: Comment explains nearby logic, invariants, or intent: `target or source materialization for Vector values, e.g.:`.
  **L638 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`target or source materialization for Vector values, e.g.:`。
- **L639 EN**: Comment explains nearby logic, invariants, or intent: `1. unrealized_cast %1 : vector<256xf32> to vector<16xf32>, ...`.
  **L639 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`1. unrealized_cast %1 : vector<256xf32> to vector<16xf32>, ...`。
- **L640 EN**: Comment explains nearby logic, invariants, or intent: `2. unrealized_cast %1 : vector<16xf32>, ... to vector<256xf32>`.
  **L640 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2. unrealized_cast %1 : vector<16xf32>, ... to vector<256xf32>`。
- **L641 EN**: Comment explains nearby logic, invariants, or intent: `it could be either 1:N or N:1 cast. In both cases, the pattern`.
  **L641 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`it could be either 1:N or N:1 cast. In both cases, the pattern`。
- **L642 EN**: Comment explains nearby logic, invariants, or intent: `simply forwards the inputs to the outputs using 1:1 or 1:N interface.`.
  **L642 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`simply forwards the inputs to the outputs using 1:1 or 1:N interface.`。
- **L643 EN**: Comment explains nearby logic, invariants, or intent: `for example, the following scf::forOp`.
  **L643 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for example, the following scf::forOp`。
- **L644 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L644 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L645 EN**: Comment explains nearby logic, invariants, or intent: `%for = scf.for ... iter_args(%arg1 = %0)->(vector<128x128xf16>) {`.
  **L645 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%for = scf.for ... iter_args(%arg1 = %0)->(vector<128x128xf16>) {`。
- **L646 EN**: Comment explains nearby logic, invariants, or intent: `%n = use(%arg1): vector<128x128xf16>`.
  **L646 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%n = use(%arg1): vector<128x128xf16>`。
- **L647 EN**: Comment explains nearby logic, invariants, or intent: `scf.yield %n : vector<128x128xf16>`.
  **L647 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scf.yield %n : vector<128x128xf16>`。
- **L648 EN**: Comment explains nearby logic, invariants, or intent: `}`.
  **L648 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`}`。

### Lines 649-672

````cpp
// ```
// Could be converted to:
// ```
// %1 = unrealized_conversion_cast %0
//          : vector<128x128xf16> to vector<16x16xf16>, vector<16x16xf16>
// %for:2 = scf.for ... iter_args(%arg1 = %1#1, %arg2 = %1#2)
//                    -> (vector<16x16xf16>, vector<16x16xf16) {
//     %m = unrealized_conversion_cast %arg1, %arg2
//            : vector<16x16xf16>, vector<16x16xf16> to vector<128x128xf16>
//     %n = use(%m): vector<128x128xf16>
//     %b = unrealized_conversion_cast %n
//            : vector<128x128xf16> to vector<16x16xf16>, vector<16x16xf16>
//     scf.yield %b#1, %b#2 : vector<16x16xf16>, vector<16x16xf16>
// }
// %cast = unrealized_conversion_cast %for:2
//          : vector<16x16xf16>, vector<16x16xf16> to vector<128x128xf16>
// ```
// TODO: remove it when context-aware type converter is ready.
struct UnrealizedConversionCastOpPattern
    : public OpConversionPattern<mlir::UnrealizedConversionCastOp> {
  using OpConversionPattern<
      mlir::UnrealizedConversionCastOp>::OpConversionPattern;

  mlir::LogicalResult
````
- **L649 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L649 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L650 EN**: Comment explains nearby logic, invariants, or intent: `Could be converted to:`.
  **L650 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Could be converted to:`。
- **L651 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L651 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L652 EN**: Comment explains nearby logic, invariants, or intent: `%1 = unrealized_conversion_cast %0`.
  **L652 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%1 = unrealized_conversion_cast %0`。
- **L653 EN**: Comment explains nearby logic, invariants, or intent: `: vector<128x128xf16> to vector<16x16xf16>, vector<16x16xf16>`.
  **L653 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`: vector<128x128xf16> to vector<16x16xf16>, vector<16x16xf16>`。
- **L654 EN**: Comment explains nearby logic, invariants, or intent: `%for:2 = scf.for ... iter_args(%arg1 = %1#1, %arg2 = %1#2)`.
  **L654 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%for:2 = scf.for ... iter_args(%arg1 = %1#1, %arg2 = %1#2)`。
- **L655 EN**: Comment explains nearby logic, invariants, or intent: `> (vector<16x16xf16>, vector<16x16xf16) {`.
  **L655 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`> (vector<16x16xf16>, vector<16x16xf16) {`。
- **L656 EN**: Comment explains nearby logic, invariants, or intent: `%m = unrealized_conversion_cast %arg1, %arg2`.
  **L656 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%m = unrealized_conversion_cast %arg1, %arg2`。
- **L657 EN**: Comment explains nearby logic, invariants, or intent: `: vector<16x16xf16>, vector<16x16xf16> to vector<128x128xf16>`.
  **L657 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`: vector<16x16xf16>, vector<16x16xf16> to vector<128x128xf16>`。
- **L658 EN**: Comment explains nearby logic, invariants, or intent: `%n = use(%m): vector<128x128xf16>`.
  **L658 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%n = use(%m): vector<128x128xf16>`。
- **L659 EN**: Comment explains nearby logic, invariants, or intent: `%b = unrealized_conversion_cast %n`.
  **L659 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%b = unrealized_conversion_cast %n`。
- **L660 EN**: Comment explains nearby logic, invariants, or intent: `: vector<128x128xf16> to vector<16x16xf16>, vector<16x16xf16>`.
  **L660 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`: vector<128x128xf16> to vector<16x16xf16>, vector<16x16xf16>`。
- **L661 EN**: Comment explains nearby logic, invariants, or intent: `scf.yield %b#1, %b#2 : vector<16x16xf16>, vector<16x16xf16>`.
  **L661 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scf.yield %b#1, %b#2 : vector<16x16xf16>, vector<16x16xf16>`。
- **L662 EN**: Comment explains nearby logic, invariants, or intent: `}`.
  **L662 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`}`。
- **L663 EN**: Comment explains nearby logic, invariants, or intent: `%cast = unrealized_conversion_cast %for:2`.
  **L663 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%cast = unrealized_conversion_cast %for:2`。
- **L664 EN**: Comment explains nearby logic, invariants, or intent: `: vector<16x16xf16>, vector<16x16xf16> to vector<128x128xf16>`.
  **L664 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`: vector<16x16xf16>, vector<16x16xf16> to vector<128x128xf16>`。
- **L665 EN**: Comment explains nearby logic, invariants, or intent: `````.
  **L665 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`````。
- **L666 EN**: Comment records a pending task or caution: `TODO: remove it when context-aware type converter is ready.`.
  **L666 CN**: 注释记录了待办事项或注意点：`TODO: remove it when context-aware type converter is ready.`。
- **L667 EN**: Declares struct `UnrealizedConversionCastOpPattern`.
  **L667 CN**: 声明 struct `UnrealizedConversionCastOpPattern`。
- **L668 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<mlir::UnrealizedConversionCastOp> {`.
  **L668 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<mlir::UnrealizedConversionCastOp> {`。
- **L669 EN**: Continues the surrounding expression or declaration: `using OpConversionPattern<`.
  **L669 CN**: 继续构造周围的表达式或声明：`using OpConversionPattern<`。
- **L670 EN**: Executes a standalone statement or declaration: `mlir::UnrealizedConversionCastOp>::OpConversionPattern;`.
  **L670 CN**: 执行一条独立语句或声明：`mlir::UnrealizedConversionCastOp>::OpConversionPattern;`。
- **L671 EN**: Blank line separating nearby declarations or logic blocks.
  **L671 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L672 EN**: Continues the surrounding expression or declaration: `mlir::LogicalResult`.
  **L672 CN**: 继续构造周围的表达式或声明：`mlir::LogicalResult`。

### Lines 673-696

````cpp
  matchAndRewrite(mlir::UnrealizedConversionCastOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    SmallVector<Value> inputs = xegpu::flattenValues(adaptor.getInputs());

    auto inputTy = dyn_cast<VectorType>(inputs[0].getType());
    auto outputTy = dyn_cast<VectorType>(op->getOpResult(0).getType());

    if (!inputTy || !outputTy || !llvm::all_equal(op->getResultTypes()) ||
        !llvm::all_equal(ValueRange(inputs).getTypes()))
      return failure();

    // Handles the case "cast %1 : vector<256xf32> to vector<16xf32>, ...".
    // It is generated by source materialization (e.g., inits to scf forOp).
    // The input values provided by the adaptor should already be distributed,
    // and their types should correspond exactly to the result types of the
    // operation.
    if (op.getNumOperands() == 1 &&
        llvm::equal(ValueRange(inputs).getTypes(), op->getResultTypes())) {
      rewriter.replaceOp(op, inputs);
      return success();
    }

    // Handles the case "cast %1 : vector<16xf32>, ... to vector<256xf32>".
    // It is generated by target materialization (e.g., arguments/results
````
- **L673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(mlir::UnrealizedConversionCastOp op, OneToNOpAdaptor adaptor,`.
  **L673 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(mlir::UnrealizedConversionCastOp op, OneToNOpAdaptor adaptor,`。
- **L674 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L674 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L675 EN**: Initializes variable `inputs` from the right-hand expression.
  **L675 CN**: 使用右侧表达式初始化变量 `inputs`。
- **L676 EN**: Blank line separating nearby declarations or logic blocks.
  **L676 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L677 EN**: Initializes variable `inputTy` from the right-hand expression.
  **L677 CN**: 使用右侧表达式初始化变量 `inputTy`。
- **L678 EN**: Initializes variable `outputTy` from the right-hand expression.
  **L678 CN**: 使用右侧表达式初始化变量 `outputTy`。
- **L679 EN**: Blank line separating nearby declarations or logic blocks.
  **L679 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L680 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L680 CN**: 开始 `if` 控制流语句并计算其条件。
- **L681 EN**: Continues logic associated with callable symbol `all_equal`.
  **L681 CN**: 继续与可调用符号 `all_equal` 相关的逻辑。
- **L682 EN**: Returns from the current function with `failure()`.
  **L682 CN**: 以 `failure()` 从当前函数返回。
- **L683 EN**: Blank line separating nearby declarations or logic blocks.
  **L683 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L684 EN**: Comment explains nearby logic, invariants, or intent: `Handles the case "cast %1 : vector<256xf32> to vector<16xf32>, ...".`.
  **L684 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Handles the case "cast %1 : vector<256xf32> to vector<16xf32>, ...".`。
- **L685 EN**: Comment explains nearby logic, invariants, or intent: `It is generated by source materialization (e.g., inits to scf forOp).`.
  **L685 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`It is generated by source materialization (e.g., inits to scf forOp).`。
- **L686 EN**: Comment explains nearby logic, invariants, or intent: `The input values provided by the adaptor should already be distributed,`.
  **L686 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The input values provided by the adaptor should already be distributed,`。
- **L687 EN**: Comment explains nearby logic, invariants, or intent: `and their types should correspond exactly to the result types of the`.
  **L687 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and their types should correspond exactly to the result types of the`。
- **L688 EN**: Comment explains nearby logic, invariants, or intent: `operation.`.
  **L688 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`operation.`。
- **L689 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L689 CN**: 开始 `if` 控制流语句并计算其条件。
- **L690 EN**: Starts a function, method, lambda, or structured scope: `llvm::equal(ValueRange(inputs).getTypes(), op->getResultTypes())) {`.
  **L690 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::equal(ValueRange(inputs).getTypes(), op->getResultTypes())) {`。
- **L691 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L691 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L692 EN**: Returns from the current function with `success()`.
  **L692 CN**: 以 `success()` 从当前函数返回。
- **L693 EN**: Closes the current lexical scope or compound statement.
  **L693 CN**: 结束当前词法作用域或复合语句块。
- **L694 EN**: Blank line separating nearby declarations or logic blocks.
  **L694 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L695 EN**: Comment explains nearby logic, invariants, or intent: `Handles the case "cast %1 : vector<16xf32>, ... to vector<256xf32>".`.
  **L695 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Handles the case "cast %1 : vector<16xf32>, ... to vector<256xf32>".`。
- **L696 EN**: Comment explains nearby logic, invariants, or intent: `It is generated by target materialization (e.g., arguments/results`.
  **L696 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`It is generated by target materialization (e.g., arguments/results`。

### Lines 697-720

````cpp
    // of scf forOp). All input values must have the same vector type, and
    // their shape must be evenly divisible by the output vector's shape
    // (determined by the nature of the workgroup to subgroup distribution).
    // TODO: it is not safe to do such forward, since such N:1 cast could be
    // from others.
    if (op.getNumResults() == 1 &&
        computeShapeRatio(outputTy.getShape(), inputTy.getShape())) {
      rewriter.replaceOpWithMultiple(op, {inputs});
      return success();
    }

    return mlir::failure();
  }
};

// This pattern distributes arith.constant op into subgroup-level constants
struct WgToSgArithConstantOp : public OpConversionPattern<arith::ConstantOp> {
  using OpConversionPattern<arith::ConstantOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(arith::ConstantOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto vecAttr = dyn_cast<DenseElementsAttr>(op.getValue());
    auto vecType = dyn_cast<VectorType>(op.getType());
````
- **L697 EN**: Comment explains nearby logic, invariants, or intent: `of scf forOp). All input values must have the same vector type, and`.
  **L697 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`of scf forOp). All input values must have the same vector type, and`。
- **L698 EN**: Comment explains nearby logic, invariants, or intent: `their shape must be evenly divisible by the output vector's shape`.
  **L698 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`their shape must be evenly divisible by the output vector's shape`。
- **L699 EN**: Comment explains nearby logic, invariants, or intent: `(determined by the nature of the workgroup to subgroup distribution).`.
  **L699 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(determined by the nature of the workgroup to subgroup distribution).`。
- **L700 EN**: Comment records a pending task or caution: `TODO: it is not safe to do such forward, since such N:1 cast could be`.
  **L700 CN**: 注释记录了待办事项或注意点：`TODO: it is not safe to do such forward, since such N:1 cast could be`。
- **L701 EN**: Comment explains nearby logic, invariants, or intent: `from others.`.
  **L701 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`from others.`。
- **L702 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L702 CN**: 开始 `if` 控制流语句并计算其条件。
- **L703 EN**: Starts a function, method, lambda, or structured scope: `computeShapeRatio(outputTy.getShape(), inputTy.getShape())) {`.
  **L703 CN**: 开始一个函数、方法、lambda 或结构化作用域：`computeShapeRatio(outputTy.getShape(), inputTy.getShape())) {`。
- **L704 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L704 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L705 EN**: Returns from the current function with `success()`.
  **L705 CN**: 以 `success()` 从当前函数返回。
- **L706 EN**: Closes the current lexical scope or compound statement.
  **L706 CN**: 结束当前词法作用域或复合语句块。
- **L707 EN**: Blank line separating nearby declarations or logic blocks.
  **L707 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L708 EN**: Returns from the current function with `mlir::failure()`.
  **L708 CN**: 以 `mlir::failure()` 从当前函数返回。
- **L709 EN**: Closes the current lexical scope or compound statement.
  **L709 CN**: 结束当前词法作用域或复合语句块。
- **L710 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L710 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L711 EN**: Blank line separating nearby declarations or logic blocks.
  **L711 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L712 EN**: Comment explains nearby logic, invariants, or intent: `This pattern distributes arith.constant op into subgroup-level constants`.
  **L712 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern distributes arith.constant op into subgroup-level constants`。
- **L713 EN**: Declares struct `WgToSgArithConstantOp`.
  **L713 CN**: 声明 struct `WgToSgArithConstantOp`。
- **L714 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<arith::ConstantOp>::OpConversionPattern;`.
  **L714 CN**: 执行一条独立语句或声明：`using OpConversionPattern<arith::ConstantOp>::OpConversionPattern;`。
- **L715 EN**: Blank line separating nearby declarations or logic blocks.
  **L715 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L716 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L716 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(arith::ConstantOp op, OneToNOpAdaptor adaptor,`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(arith::ConstantOp op, OneToNOpAdaptor adaptor,`。
- **L718 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L718 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L719 EN**: Initializes variable `vecAttr` from the right-hand expression.
  **L719 CN**: 使用右侧表达式初始化变量 `vecAttr`。
- **L720 EN**: Initializes variable `vecType` from the right-hand expression.
  **L720 CN**: 使用右侧表达式初始化变量 `vecType`。

### Lines 721-744

````cpp
    if (!vecAttr || !vecType)
      return failure();

    xegpu::DistributeLayoutAttr layout =
        xegpu::getTemporaryLayout(dyn_cast<OpResult>(op.getResult()));
    if (!layout || !layout.isForWorkgroup())
      return failure();

    ArrayRef<int64_t> wgShape = vecType.getShape();
    SmallVector<int64_t> sgShape;
    int count;
    std::tie(sgShape, count) = getSgShapeAndCount(wgShape, layout);

    auto newType = VectorType::get(sgShape, vecType.getElementType());
    Location loc = op.getLoc();
    auto eltType = vecType.getElementType();

    if (vecAttr.isSplat()) {
      // Splat: single value for all subgroups
      Attribute singleVal = vecAttr.getSplatValue<Attribute>();
      auto sgAttr = DenseElementsAttr::get(newType, singleVal);
      SmallVector<Value> newConstOps;
      for (int i = 0; i < count; ++i) {
        auto cstOp = arith::ConstantOp::create(rewriter, loc, newType, sgAttr);
````
- **L721 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L721 CN**: 开始 `if` 控制流语句并计算其条件。
- **L722 EN**: Returns from the current function with `failure()`.
  **L722 CN**: 以 `failure()` 从当前函数返回。
- **L723 EN**: Blank line separating nearby declarations or logic blocks.
  **L723 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L724 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout =`.
  **L724 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout =`。
- **L725 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L725 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L726 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L726 CN**: 开始 `if` 控制流语句并计算其条件。
- **L727 EN**: Returns from the current function with `failure()`.
  **L727 CN**: 以 `failure()` 从当前函数返回。
- **L728 EN**: Blank line separating nearby declarations or logic blocks.
  **L728 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L729 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L729 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L730 EN**: Executes a standalone statement or declaration: `SmallVector<int64_t> sgShape;`.
  **L730 CN**: 执行一条独立语句或声明：`SmallVector<int64_t> sgShape;`。
- **L731 EN**: Executes a standalone statement or declaration: `int count;`.
  **L731 CN**: 执行一条独立语句或声明：`int count;`。
- **L732 EN**: Executes a call or declaration centered on `std::tie`.
  **L732 CN**: 执行以 `std::tie` 为核心的调用或声明。
- **L733 EN**: Blank line separating nearby declarations or logic blocks.
  **L733 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L734 EN**: Initializes variable `newType` from the right-hand expression.
  **L734 CN**: 使用右侧表达式初始化变量 `newType`。
- **L735 EN**: Initializes variable `loc` from the right-hand expression.
  **L735 CN**: 使用右侧表达式初始化变量 `loc`。
- **L736 EN**: Initializes variable `eltType` from the right-hand expression.
  **L736 CN**: 使用右侧表达式初始化变量 `eltType`。
- **L737 EN**: Blank line separating nearby declarations or logic blocks.
  **L737 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L738 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L738 CN**: 开始 `if` 控制流语句并计算其条件。
- **L739 EN**: Comment explains nearby logic, invariants, or intent: `Splat: single value for all subgroups`.
  **L739 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Splat: single value for all subgroups`。
- **L740 EN**: Initializes variable `singleVal` from the right-hand expression.
  **L740 CN**: 使用右侧表达式初始化变量 `singleVal`。
- **L741 EN**: Initializes variable `sgAttr` from the right-hand expression.
  **L741 CN**: 使用右侧表达式初始化变量 `sgAttr`。
- **L742 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newConstOps;`.
  **L742 CN**: 执行一条独立语句或声明：`SmallVector<Value> newConstOps;`。
- **L743 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L743 CN**: 开始 `for` 控制流语句并计算其条件。
- **L744 EN**: Initializes variable `cstOp` from the right-hand expression.
  **L744 CN**: 使用右侧表达式初始化变量 `cstOp`。

### Lines 745-768

````cpp
        newConstOps.push_back(cstOp);
      }
      rewriter.replaceOpWithMultiple(op, {newConstOps});
      return success();
    } else if (sgShape == wgShape) { // if the entire vector is shared by all
                                     // subgroups, don't distribute
      auto newConstOp =
          arith::ConstantOp::create(rewriter, op.getLoc(), vecType, vecAttr);
      rewriter.replaceOp(op, newConstOp);
      return success();
    } else {
      // Non-splat constant
      // Only supports 1D & 2D
      // TODO: support other cases that require SLM access
      if (!eltType.isIndex())
        return rewriter.notifyMatchFailure(
            op, "Unsupported element type for non-splat constant op.");

      if (wgShape.size() > 2)
        return rewriter.notifyMatchFailure(
            op, "Only 1D & 2D vector constant supported");

      SmallVector<Attribute> values(vecAttr.getValues<Attribute>());
      int64_t rowStride = 0, colStride = 0;
````
- **L745 EN**: Executes a call or declaration centered on `newConstOps.push_back`.
  **L745 CN**: 执行以 `newConstOps.push_back` 为核心的调用或声明。
- **L746 EN**: Closes the current lexical scope or compound statement.
  **L746 CN**: 结束当前词法作用域或复合语句块。
- **L747 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L747 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L748 EN**: Returns from the current function with `success()`.
  **L748 CN**: 以 `success()` 从当前函数返回。
- **L749 EN**: Continues the surrounding expression or declaration: `} else if (sgShape == wgShape) { // if the entire vector is shared by all`.
  **L749 CN**: 继续构造周围的表达式或声明：`} else if (sgShape == wgShape) { // if the entire vector is shared by all`。
- **L750 EN**: Comment explains nearby logic, invariants, or intent: `subgroups, don't distribute`.
  **L750 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`subgroups, don't distribute`。
- **L751 EN**: Continues the surrounding expression or declaration: `auto newConstOp =`.
  **L751 CN**: 继续构造周围的表达式或声明：`auto newConstOp =`。
- **L752 EN**: Executes a call or declaration centered on `arith::ConstantOp::create`.
  **L752 CN**: 执行以 `arith::ConstantOp::create` 为核心的调用或声明。
- **L753 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L753 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L754 EN**: Returns from the current function with `success()`.
  **L754 CN**: 以 `success()` 从当前函数返回。
- **L755 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L755 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L756 EN**: Comment explains nearby logic, invariants, or intent: `Non-splat constant`.
  **L756 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Non-splat constant`。
- **L757 EN**: Comment explains nearby logic, invariants, or intent: `Only supports 1D & 2D`.
  **L757 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only supports 1D & 2D`。
- **L758 EN**: Comment records a pending task or caution: `TODO: support other cases that require SLM access`.
  **L758 CN**: 注释记录了待办事项或注意点：`TODO: support other cases that require SLM access`。
- **L759 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L759 CN**: 开始 `if` 控制流语句并计算其条件。
- **L760 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L760 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L761 EN**: Executes a standalone statement or declaration: `op, "Unsupported element type for non-splat constant op.");`.
  **L761 CN**: 执行一条独立语句或声明：`op, "Unsupported element type for non-splat constant op.");`。
- **L762 EN**: Blank line separating nearby declarations or logic blocks.
  **L762 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L763 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L763 CN**: 开始 `if` 控制流语句并计算其条件。
- **L764 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L764 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L765 EN**: Executes a standalone statement or declaration: `op, "Only 1D & 2D vector constant supported");`.
  **L765 CN**: 执行一条独立语句或声明：`op, "Only 1D & 2D vector constant supported");`。
- **L766 EN**: Blank line separating nearby declarations or logic blocks.
  **L766 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L767 EN**: Executes a call or declaration centered on `values`.
  **L767 CN**: 执行以 `values` 为核心的调用或声明。
- **L768 EN**: Initializes variable `rowStride` from the right-hand expression.
  **L768 CN**: 使用右侧表达式初始化变量 `rowStride`。

### Lines 769-792

````cpp
      int64_t rows = wgShape.size() == 1 ? 1 : wgShape[0];
      int64_t cols = wgShape.size() == 1 ? wgShape[0] : wgShape[1];

      // Compute colStride and rowStride, and check for constant strides.
      if (cols > 1) {
        colStride = cast<IntegerAttr>(values[1]).getInt() -
                    cast<IntegerAttr>(values[0]).getInt();
      }
      if (rows > 1) {
        rowStride = cast<IntegerAttr>(values[cols]).getInt() -
                    cast<IntegerAttr>(values[0]).getInt();
      }

      for (int64_t r = 0; r < rows; ++r) {
        for (int64_t c = 0; c < cols; ++c) {
          int64_t idx = r * cols + c;
          // Check column stride
          if (c > 0 && cols > 1) {
            int64_t prevIdx = r * cols + (c - 1);
            int64_t diff = cast<IntegerAttr>(values[idx]).getInt() -
                           cast<IntegerAttr>(values[prevIdx]).getInt();
            if (diff != colStride)
              return rewriter.notifyMatchFailure(
                  op, "Non-constant column stride in constant op.");
````
- **L769 EN**: Initializes variable `rows` from the right-hand expression.
  **L769 CN**: 使用右侧表达式初始化变量 `rows`。
- **L770 EN**: Initializes variable `cols` from the right-hand expression.
  **L770 CN**: 使用右侧表达式初始化变量 `cols`。
- **L771 EN**: Blank line separating nearby declarations or logic blocks.
  **L771 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L772 EN**: Comment explains nearby logic, invariants, or intent: `Compute colStride and rowStride, and check for constant strides.`.
  **L772 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Compute colStride and rowStride, and check for constant strides.`。
- **L773 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L773 CN**: 开始 `if` 控制流语句并计算其条件。
- **L774 EN**: Continues logic associated with callable symbol `cast<IntegerAttr>`.
  **L774 CN**: 继续与可调用符号 `cast<IntegerAttr>` 相关的逻辑。
- **L775 EN**: Executes a call or declaration centered on `cast<IntegerAttr>`.
  **L775 CN**: 执行以 `cast<IntegerAttr>` 为核心的调用或声明。
- **L776 EN**: Closes the current lexical scope or compound statement.
  **L776 CN**: 结束当前词法作用域或复合语句块。
- **L777 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L777 CN**: 开始 `if` 控制流语句并计算其条件。
- **L778 EN**: Continues logic associated with callable symbol `cast<IntegerAttr>`.
  **L778 CN**: 继续与可调用符号 `cast<IntegerAttr>` 相关的逻辑。
- **L779 EN**: Executes a call or declaration centered on `cast<IntegerAttr>`.
  **L779 CN**: 执行以 `cast<IntegerAttr>` 为核心的调用或声明。
- **L780 EN**: Closes the current lexical scope or compound statement.
  **L780 CN**: 结束当前词法作用域或复合语句块。
- **L781 EN**: Blank line separating nearby declarations or logic blocks.
  **L781 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L782 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L782 CN**: 开始 `for` 控制流语句并计算其条件。
- **L783 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L783 CN**: 开始 `for` 控制流语句并计算其条件。
- **L784 EN**: Initializes variable `idx` from the right-hand expression.
  **L784 CN**: 使用右侧表达式初始化变量 `idx`。
- **L785 EN**: Comment explains nearby logic, invariants, or intent: `Check column stride`.
  **L785 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check column stride`。
- **L786 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L786 CN**: 开始 `if` 控制流语句并计算其条件。
- **L787 EN**: Initializes variable `prevIdx` from the right-hand expression.
  **L787 CN**: 使用右侧表达式初始化变量 `prevIdx`。
- **L788 EN**: Continues logic associated with callable symbol `cast<IntegerAttr>`.
  **L788 CN**: 继续与可调用符号 `cast<IntegerAttr>` 相关的逻辑。
- **L789 EN**: Executes a call or declaration centered on `cast<IntegerAttr>`.
  **L789 CN**: 执行以 `cast<IntegerAttr>` 为核心的调用或声明。
- **L790 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L790 CN**: 开始 `if` 控制流语句并计算其条件。
- **L791 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L791 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L792 EN**: Executes a standalone statement or declaration: `op, "Non-constant column stride in constant op.");`.
  **L792 CN**: 执行一条独立语句或声明：`op, "Non-constant column stride in constant op.");`。

### Lines 793-816

````cpp
          }
          // Check row stride
          if (r > 0 && rows > 1) {
            int64_t prevIdx = (r - 1) * cols + c;
            int64_t diff = cast<IntegerAttr>(values[idx]).getInt() -
                           cast<IntegerAttr>(values[prevIdx]).getInt();
            if (diff != rowStride)
              return rewriter.notifyMatchFailure(
                  op, "Non-constant row stride in constant op.");
          }
        }
      }

      // Create a constant for the base tile.
      // For 2D case, extract the top-left sgShape[0] x sgShape[1] submatrix.
      // For 1D case, extract the first sgShape[0] elements.
      SmallVector<Attribute> baseTileValues;
      int baseTileCols = sgShape[sgShape.size() - 1];
      int64_t baseTileRows = sgShape.size() == 1 ? 1 : sgShape[0];
      for (int64_t r = 0; r < baseTileRows; ++r) {
        for (int64_t c = 0; c < baseTileCols; ++c) {
          baseTileValues.push_back(values[r * cols + c]);
        }
      }
````
- **L793 EN**: Closes the current lexical scope or compound statement.
  **L793 CN**: 结束当前词法作用域或复合语句块。
- **L794 EN**: Comment explains nearby logic, invariants, or intent: `Check row stride`.
  **L794 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check row stride`。
- **L795 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L795 CN**: 开始 `if` 控制流语句并计算其条件。
- **L796 EN**: Initializes variable `prevIdx` from the right-hand expression.
  **L796 CN**: 使用右侧表达式初始化变量 `prevIdx`。
- **L797 EN**: Continues logic associated with callable symbol `cast<IntegerAttr>`.
  **L797 CN**: 继续与可调用符号 `cast<IntegerAttr>` 相关的逻辑。
- **L798 EN**: Executes a call or declaration centered on `cast<IntegerAttr>`.
  **L798 CN**: 执行以 `cast<IntegerAttr>` 为核心的调用或声明。
- **L799 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L799 CN**: 开始 `if` 控制流语句并计算其条件。
- **L800 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L800 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L801 EN**: Executes a standalone statement or declaration: `op, "Non-constant row stride in constant op.");`.
  **L801 CN**: 执行一条独立语句或声明：`op, "Non-constant row stride in constant op.");`。
- **L802 EN**: Closes the current lexical scope or compound statement.
  **L802 CN**: 结束当前词法作用域或复合语句块。
- **L803 EN**: Closes the current lexical scope or compound statement.
  **L803 CN**: 结束当前词法作用域或复合语句块。
- **L804 EN**: Closes the current lexical scope or compound statement.
  **L804 CN**: 结束当前词法作用域或复合语句块。
- **L805 EN**: Blank line separating nearby declarations or logic blocks.
  **L805 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L806 EN**: Comment explains nearby logic, invariants, or intent: `Create a constant for the base tile.`.
  **L806 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create a constant for the base tile.`。
- **L807 EN**: Comment explains nearby logic, invariants, or intent: `For 2D case, extract the top-left sgShape[0] x sgShape[1] submatrix.`.
  **L807 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For 2D case, extract the top-left sgShape[0] x sgShape[1] submatrix.`。
- **L808 EN**: Comment explains nearby logic, invariants, or intent: `For 1D case, extract the first sgShape[0] elements.`.
  **L808 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For 1D case, extract the first sgShape[0] elements.`。
- **L809 EN**: Executes a standalone statement or declaration: `SmallVector<Attribute> baseTileValues;`.
  **L809 CN**: 执行一条独立语句或声明：`SmallVector<Attribute> baseTileValues;`。
- **L810 EN**: Initializes variable `baseTileCols` from the right-hand expression.
  **L810 CN**: 使用右侧表达式初始化变量 `baseTileCols`。
- **L811 EN**: Initializes variable `baseTileRows` from the right-hand expression.
  **L811 CN**: 使用右侧表达式初始化变量 `baseTileRows`。
- **L812 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L812 CN**: 开始 `for` 控制流语句并计算其条件。
- **L813 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L813 CN**: 开始 `for` 控制流语句并计算其条件。
- **L814 EN**: Executes a call or declaration centered on `baseTileValues.push_back`.
  **L814 CN**: 执行以 `baseTileValues.push_back` 为核心的调用或声明。
- **L815 EN**: Closes the current lexical scope or compound statement.
  **L815 CN**: 结束当前词法作用域或复合语句块。
- **L816 EN**: Closes the current lexical scope or compound statement.
  **L816 CN**: 结束当前词法作用域或复合语句块。

### Lines 817-840

````cpp

      auto tileAttr = DenseElementsAttr::get(VectorType::get(sgShape, eltType),
                                             baseTileValues);
      auto baseConstVec = arith::ConstantOp::create(rewriter, loc, tileAttr);

      // Get subgroup id
      Value sgId =
          gpu::SubgroupIdOp::create(rewriter, loc, /*upper_bound=*/nullptr);
      auto sgOffsets =
          layout.computeDistributedCoords(rewriter, loc, sgId, wgShape);
      if (failed(sgOffsets))
        return failure();

      SmallVector<Value, 2> strideConsts;
      strideConsts.push_back(
          arith::ConstantIndexOp::create(rewriter, loc, colStride));
      if (rows > 1)
        strideConsts.insert(
            strideConsts.begin(),
            arith::ConstantIndexOp::create(rewriter, loc, rowStride));

      SmallVector<Value> newConstOps;
      for (auto offsets : *sgOffsets) {
        // Multiply offset with stride, broadcast it and add to baseConstVec
````
- **L817 EN**: Blank line separating nearby declarations or logic blocks.
  **L817 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L818 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto tileAttr = DenseElementsAttr::get(VectorType::get(sgShape, eltType),`.
  **L818 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto tileAttr = DenseElementsAttr::get(VectorType::get(sgShape, eltType),`。
- **L819 EN**: Executes a standalone statement or declaration: `baseTileValues);`.
  **L819 CN**: 执行一条独立语句或声明：`baseTileValues);`。
- **L820 EN**: Initializes variable `baseConstVec` from the right-hand expression.
  **L820 CN**: 使用右侧表达式初始化变量 `baseConstVec`。
- **L821 EN**: Blank line separating nearby declarations or logic blocks.
  **L821 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L822 EN**: Comment explains nearby logic, invariants, or intent: `Get subgroup id`.
  **L822 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get subgroup id`。
- **L823 EN**: Continues the surrounding expression or declaration: `Value sgId =`.
  **L823 CN**: 继续构造周围的表达式或声明：`Value sgId =`。
- **L824 EN**: Executes a call or declaration centered on `gpu::SubgroupIdOp::create`.
  **L824 CN**: 执行以 `gpu::SubgroupIdOp::create` 为核心的调用或声明。
- **L825 EN**: Continues the surrounding expression or declaration: `auto sgOffsets =`.
  **L825 CN**: 继续构造周围的表达式或声明：`auto sgOffsets =`。
- **L826 EN**: Executes a call or declaration centered on `layout.computeDistributedCoords`.
  **L826 CN**: 执行以 `layout.computeDistributedCoords` 为核心的调用或声明。
- **L827 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L827 CN**: 开始 `if` 控制流语句并计算其条件。
- **L828 EN**: Returns from the current function with `failure()`.
  **L828 CN**: 以 `failure()` 从当前函数返回。
- **L829 EN**: Blank line separating nearby declarations or logic blocks.
  **L829 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L830 EN**: Executes a standalone statement or declaration: `SmallVector<Value, 2> strideConsts;`.
  **L830 CN**: 执行一条独立语句或声明：`SmallVector<Value, 2> strideConsts;`。
- **L831 EN**: Continues logic associated with callable symbol `push_back`.
  **L831 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L832 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L832 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L833 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L833 CN**: 开始 `if` 控制流语句并计算其条件。
- **L834 EN**: Continues logic associated with callable symbol `insert`.
  **L834 CN**: 继续与可调用符号 `insert` 相关的逻辑。
- **L835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `strideConsts.begin(),`.
  **L835 CN**: 继续一个多行参数列表、初始化器或聚合项：`strideConsts.begin(),`。
- **L836 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L836 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L837 EN**: Blank line separating nearby declarations or logic blocks.
  **L837 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L838 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newConstOps;`.
  **L838 CN**: 执行一条独立语句或声明：`SmallVector<Value> newConstOps;`。
- **L839 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L839 CN**: 开始 `for` 控制流语句并计算其条件。
- **L840 EN**: Comment explains nearby logic, invariants, or intent: `Multiply offset with stride, broadcast it and add to baseConstVec`.
  **L840 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multiply offset with stride, broadcast it and add to baseConstVec`。

### Lines 841-864

````cpp
        Value mulOffset = arith::ConstantIndexOp::create(rewriter, loc, 0);
        for (size_t i = 0; i < strideConsts.size(); ++i) {
          Value mul =
              arith::MulIOp::create(rewriter, loc, rewriter.getIndexType(),
                                    offsets[i], strideConsts[i]);
          mulOffset = arith::AddIOp::create(
              rewriter, loc, rewriter.getIndexType(), mulOffset, mul);
        }
        // Broadcast to baseConstVec size
        auto bcastOffset = vector::BroadcastOp::create(
            rewriter, loc, baseConstVec.getType(), mulOffset);
        auto finalConst =
            arith::AddIOp::create(rewriter, loc, baseConstVec, bcastOffset);
        newConstOps.push_back(finalConst);
      }
      rewriter.replaceOpWithMultiple(op, {newConstOps});
      return success();
    }
  }
};

// This pattern transforms the LoadGatherOp with explicit offsets to load
// subgroup data
struct WgToSgLoadGatherOp : public OpConversionPattern<xegpu::LoadGatherOp> {
````
- **L841 EN**: Initializes variable `mulOffset` from the right-hand expression.
  **L841 CN**: 使用右侧表达式初始化变量 `mulOffset`。
- **L842 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L842 CN**: 开始 `for` 控制流语句并计算其条件。
- **L843 EN**: Continues the surrounding expression or declaration: `Value mul =`.
  **L843 CN**: 继续构造周围的表达式或声明：`Value mul =`。
- **L844 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arith::MulIOp::create(rewriter, loc, rewriter.getIndexType(),`.
  **L844 CN**: 继续一个多行参数列表、初始化器或聚合项：`arith::MulIOp::create(rewriter, loc, rewriter.getIndexType(),`。
- **L845 EN**: Executes a standalone statement or declaration: `offsets[i], strideConsts[i]);`.
  **L845 CN**: 执行一条独立语句或声明：`offsets[i], strideConsts[i]);`。
- **L846 EN**: Continues logic associated with callable symbol `create`.
  **L846 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L847 EN**: Executes a call or declaration centered on `rewriter.getIndexType`.
  **L847 CN**: 执行以 `rewriter.getIndexType` 为核心的调用或声明。
- **L848 EN**: Closes the current lexical scope or compound statement.
  **L848 CN**: 结束当前词法作用域或复合语句块。
- **L849 EN**: Comment explains nearby logic, invariants, or intent: `Broadcast to baseConstVec size`.
  **L849 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Broadcast to baseConstVec size`。
- **L850 EN**: Continues logic associated with callable symbol `create`.
  **L850 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L851 EN**: Executes a call or declaration centered on `baseConstVec.getType`.
  **L851 CN**: 执行以 `baseConstVec.getType` 为核心的调用或声明。
- **L852 EN**: Continues the surrounding expression or declaration: `auto finalConst =`.
  **L852 CN**: 继续构造周围的表达式或声明：`auto finalConst =`。
- **L853 EN**: Executes a call or declaration centered on `arith::AddIOp::create`.
  **L853 CN**: 执行以 `arith::AddIOp::create` 为核心的调用或声明。
- **L854 EN**: Executes a call or declaration centered on `newConstOps.push_back`.
  **L854 CN**: 执行以 `newConstOps.push_back` 为核心的调用或声明。
- **L855 EN**: Closes the current lexical scope or compound statement.
  **L855 CN**: 结束当前词法作用域或复合语句块。
- **L856 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L856 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L857 EN**: Returns from the current function with `success()`.
  **L857 CN**: 以 `success()` 从当前函数返回。
- **L858 EN**: Closes the current lexical scope or compound statement.
  **L858 CN**: 结束当前词法作用域或复合语句块。
- **L859 EN**: Closes the current lexical scope or compound statement.
  **L859 CN**: 结束当前词法作用域或复合语句块。
- **L860 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L860 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L861 EN**: Blank line separating nearby declarations or logic blocks.
  **L861 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L862 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms the LoadGatherOp with explicit offsets to load`.
  **L862 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms the LoadGatherOp with explicit offsets to load`。
- **L863 EN**: Comment explains nearby logic, invariants, or intent: `subgroup data`.
  **L863 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`subgroup data`。
- **L864 EN**: Declares struct `WgToSgLoadGatherOp`.
  **L864 CN**: 声明 struct `WgToSgLoadGatherOp`。

### Lines 865-888

````cpp
  using OpConversionPattern<xegpu::LoadGatherOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(xegpu::LoadGatherOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {

    Location loc = op.getLoc();
    VectorType resultType = dyn_cast<VectorType>(op.getResult().getType());
    if (!resultType)
      return failure();
    ArrayRef<int64_t> wgShape = resultType.getShape();

    xegpu::DistributeLayoutAttr layout = op.getLayoutAttr();

    if (!layout || !layout.isForWorkgroup())
      return failure();

    SmallVector<int64_t> sgShape = getSgShapeAndCount(wgShape, layout).first;

    // The offsets need to be distributed
    auto offsetsVecType =
        dyn_cast<VectorType>(adaptor.getOffsets().front().getType());
    auto maskVecType =
        dyn_cast<VectorType>(adaptor.getMask().front().getType());
    if (!offsetsVecType || !maskVecType ||
````
- **L865 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::LoadGatherOp>::OpConversionPattern;`.
  **L865 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::LoadGatherOp>::OpConversionPattern;`。
- **L866 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L866 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::LoadGatherOp op, OneToNOpAdaptor adaptor,`.
  **L867 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::LoadGatherOp op, OneToNOpAdaptor adaptor,`。
- **L868 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L868 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L869 EN**: Blank line separating nearby declarations or logic blocks.
  **L869 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L870 EN**: Initializes variable `loc` from the right-hand expression.
  **L870 CN**: 使用右侧表达式初始化变量 `loc`。
- **L871 EN**: Initializes variable `resultType` from the right-hand expression.
  **L871 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L872 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L872 CN**: 开始 `if` 控制流语句并计算其条件。
- **L873 EN**: Returns from the current function with `failure()`.
  **L873 CN**: 以 `failure()` 从当前函数返回。
- **L874 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L874 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L875 EN**: Blank line separating nearby declarations or logic blocks.
  **L875 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L876 EN**: Initializes variable `layout` from the right-hand expression.
  **L876 CN**: 使用右侧表达式初始化变量 `layout`。
- **L877 EN**: Blank line separating nearby declarations or logic blocks.
  **L877 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L878 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L878 CN**: 开始 `if` 控制流语句并计算其条件。
- **L879 EN**: Returns from the current function with `failure()`.
  **L879 CN**: 以 `failure()` 从当前函数返回。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L881 EN**: Initializes variable `sgShape` from the right-hand expression.
  **L881 CN**: 使用右侧表达式初始化变量 `sgShape`。
- **L882 EN**: Blank line separating nearby declarations or logic blocks.
  **L882 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L883 EN**: Comment explains nearby logic, invariants, or intent: `The offsets need to be distributed`.
  **L883 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The offsets need to be distributed`。
- **L884 EN**: Continues the surrounding expression or declaration: `auto offsetsVecType =`.
  **L884 CN**: 继续构造周围的表达式或声明：`auto offsetsVecType =`。
- **L885 EN**: Executes a call or declaration centered on `dyn_cast<VectorType>`.
  **L885 CN**: 执行以 `dyn_cast<VectorType>` 为核心的调用或声明。
- **L886 EN**: Continues the surrounding expression or declaration: `auto maskVecType =`.
  **L886 CN**: 继续构造周围的表达式或声明：`auto maskVecType =`。
- **L887 EN**: Executes a call or declaration centered on `dyn_cast<VectorType>`.
  **L887 CN**: 执行以 `dyn_cast<VectorType>` 为核心的调用或声明。
- **L888 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L888 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 889-912

````cpp
        offsetsVecType.getShape() != maskVecType.getShape()) {
      return rewriter.notifyMatchFailure(op,
                                         "offsets have not been distributed");
    }

    SmallVector<Value> newLoadOps;
    auto chunkSizeAttr =
        rewriter.getI64IntegerAttr(op.getChunkSize().value_or(1));
    VectorType newTy = VectorType::get(sgShape, resultType.getElementType());
    for (auto [offsets, mask] :
         llvm::zip(adaptor.getOffsets(), adaptor.getMask())) {
      auto newLayout = layout.dropSgLayoutAndData();
      auto newLoadOp = xegpu::LoadGatherOp::create(
          rewriter, loc, newTy, op.getSource(), offsets, mask, chunkSizeAttr,
          op.getL1HintAttr(), op.getL2HintAttr(), op.getL3HintAttr(),
          newLayout);
      newLoadOps.push_back(newLoadOp);
    }
    rewriter.replaceOpWithMultiple(op, {newLoadOps});
    return success();
  }
};

// This pattern transforms the StoreScatterOp with explicit offsets to store
````
- **L889 EN**: Starts a function, method, lambda, or structured scope: `offsetsVecType.getShape() != maskVecType.getShape()) {`.
  **L889 CN**: 开始一个函数、方法、lambda 或结构化作用域：`offsetsVecType.getShape() != maskVecType.getShape()) {`。
- **L890 EN**: Returns from the current function with `rewriter.notifyMatchFailure(op,`.
  **L890 CN**: 以 `rewriter.notifyMatchFailure(op,` 从当前函数返回。
- **L891 EN**: Executes a standalone statement or declaration: `"offsets have not been distributed");`.
  **L891 CN**: 执行一条独立语句或声明：`"offsets have not been distributed");`。
- **L892 EN**: Closes the current lexical scope or compound statement.
  **L892 CN**: 结束当前词法作用域或复合语句块。
- **L893 EN**: Blank line separating nearby declarations or logic blocks.
  **L893 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L894 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newLoadOps;`.
  **L894 CN**: 执行一条独立语句或声明：`SmallVector<Value> newLoadOps;`。
- **L895 EN**: Continues the surrounding expression or declaration: `auto chunkSizeAttr =`.
  **L895 CN**: 继续构造周围的表达式或声明：`auto chunkSizeAttr =`。
- **L896 EN**: Executes a call or declaration centered on `rewriter.getI64IntegerAttr`.
  **L896 CN**: 执行以 `rewriter.getI64IntegerAttr` 为核心的调用或声明。
- **L897 EN**: Initializes variable `newTy` from the right-hand expression.
  **L897 CN**: 使用右侧表达式初始化变量 `newTy`。
- **L898 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L898 CN**: 开始 `for` 控制流语句并计算其条件。
- **L899 EN**: Starts a function, method, lambda, or structured scope: `llvm::zip(adaptor.getOffsets(), adaptor.getMask())) {`.
  **L899 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::zip(adaptor.getOffsets(), adaptor.getMask())) {`。
- **L900 EN**: Initializes variable `newLayout` from the right-hand expression.
  **L900 CN**: 使用右侧表达式初始化变量 `newLayout`。
- **L901 EN**: Continues logic associated with callable symbol `create`.
  **L901 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, newTy, op.getSource(), offsets, mask, chunkSizeAttr,`.
  **L902 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, newTy, op.getSource(), offsets, mask, chunkSizeAttr,`。
- **L903 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op.getL1HintAttr(), op.getL2HintAttr(), op.getL3HintAttr(),`.
  **L903 CN**: 继续一个多行参数列表、初始化器或聚合项：`op.getL1HintAttr(), op.getL2HintAttr(), op.getL3HintAttr(),`。
- **L904 EN**: Executes a standalone statement or declaration: `newLayout);`.
  **L904 CN**: 执行一条独立语句或声明：`newLayout);`。
- **L905 EN**: Executes a call or declaration centered on `newLoadOps.push_back`.
  **L905 CN**: 执行以 `newLoadOps.push_back` 为核心的调用或声明。
- **L906 EN**: Closes the current lexical scope or compound statement.
  **L906 CN**: 结束当前词法作用域或复合语句块。
- **L907 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L907 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L908 EN**: Returns from the current function with `success()`.
  **L908 CN**: 以 `success()` 从当前函数返回。
- **L909 EN**: Closes the current lexical scope or compound statement.
  **L909 CN**: 结束当前词法作用域或复合语句块。
- **L910 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L910 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L911 EN**: Blank line separating nearby declarations or logic blocks.
  **L911 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L912 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms the StoreScatterOp with explicit offsets to store`.
  **L912 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms the StoreScatterOp with explicit offsets to store`。

### Lines 913-936

````cpp
// subgroup data
struct WgToSgStoreScatterOp
    : public OpConversionPattern<xegpu::StoreScatterOp> {
  using OpConversionPattern<xegpu::StoreScatterOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(xegpu::StoreScatterOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {

    Location loc = op.getLoc();
    VectorType valueType = dyn_cast<VectorType>(op.getValue().getType());
    if (!valueType)
      return failure();

    xegpu::DistributeLayoutAttr layout = op.getLayoutAttr();

    if (!layout || !layout.isForWorkgroup())
      return failure();

    // The offsets need to be distributed
    auto offsetsVecType =
        dyn_cast<VectorType>(adaptor.getOffsets().front().getType());
    auto maskVecType =
        dyn_cast<VectorType>(adaptor.getMask().front().getType());
    if (!offsetsVecType || !maskVecType ||
````
- **L913 EN**: Comment explains nearby logic, invariants, or intent: `subgroup data`.
  **L913 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`subgroup data`。
- **L914 EN**: Declares struct `WgToSgStoreScatterOp`.
  **L914 CN**: 声明 struct `WgToSgStoreScatterOp`。
- **L915 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<xegpu::StoreScatterOp> {`.
  **L915 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<xegpu::StoreScatterOp> {`。
- **L916 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::StoreScatterOp>::OpConversionPattern;`.
  **L916 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::StoreScatterOp>::OpConversionPattern;`。
- **L917 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L917 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::StoreScatterOp op, OneToNOpAdaptor adaptor,`.
  **L918 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::StoreScatterOp op, OneToNOpAdaptor adaptor,`。
- **L919 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L919 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L920 EN**: Blank line separating nearby declarations or logic blocks.
  **L920 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L921 EN**: Initializes variable `loc` from the right-hand expression.
  **L921 CN**: 使用右侧表达式初始化变量 `loc`。
- **L922 EN**: Initializes variable `valueType` from the right-hand expression.
  **L922 CN**: 使用右侧表达式初始化变量 `valueType`。
- **L923 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L923 CN**: 开始 `if` 控制流语句并计算其条件。
- **L924 EN**: Returns from the current function with `failure()`.
  **L924 CN**: 以 `failure()` 从当前函数返回。
- **L925 EN**: Blank line separating nearby declarations or logic blocks.
  **L925 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L926 EN**: Initializes variable `layout` from the right-hand expression.
  **L926 CN**: 使用右侧表达式初始化变量 `layout`。
- **L927 EN**: Blank line separating nearby declarations or logic blocks.
  **L927 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L928 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L928 CN**: 开始 `if` 控制流语句并计算其条件。
- **L929 EN**: Returns from the current function with `failure()`.
  **L929 CN**: 以 `failure()` 从当前函数返回。
- **L930 EN**: Blank line separating nearby declarations or logic blocks.
  **L930 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L931 EN**: Comment explains nearby logic, invariants, or intent: `The offsets need to be distributed`.
  **L931 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The offsets need to be distributed`。
- **L932 EN**: Continues the surrounding expression or declaration: `auto offsetsVecType =`.
  **L932 CN**: 继续构造周围的表达式或声明：`auto offsetsVecType =`。
- **L933 EN**: Executes a call or declaration centered on `dyn_cast<VectorType>`.
  **L933 CN**: 执行以 `dyn_cast<VectorType>` 为核心的调用或声明。
- **L934 EN**: Continues the surrounding expression or declaration: `auto maskVecType =`.
  **L934 CN**: 继续构造周围的表达式或声明：`auto maskVecType =`。
- **L935 EN**: Executes a call or declaration centered on `dyn_cast<VectorType>`.
  **L935 CN**: 执行以 `dyn_cast<VectorType>` 为核心的调用或声明。
- **L936 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L936 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 937-960

````cpp
        offsetsVecType.getShape() != maskVecType.getShape()) {
      return rewriter.notifyMatchFailure(op,
                                         "offsets have not been distributed");
    }

    auto chunkSizeOpt = op.getChunkSize();
    int64_t chunkSize = chunkSizeOpt ? static_cast<int64_t>(*chunkSizeOpt) : 1;
    auto chunkSizeAttr = rewriter.getI64IntegerAttr(chunkSize);
    for (auto [val, offs, mask] : llvm::zip(
             adaptor.getValue(), adaptor.getOffsets(), adaptor.getMask())) {
      xegpu::StoreScatterOp::create(rewriter, loc, val, op.getDest(), offs,
                                    mask, chunkSizeAttr, op.getL1HintAttr(),
                                    op.getL2HintAttr(), op.getL3HintAttr(),
                                    layout.dropSgLayoutAndData());
    }
    rewriter.eraseOp(op);
    return success();
  }
};

struct WgToSgLoadMatrixOp : public OpConversionPattern<xegpu::LoadMatrixOp> {
  using OpConversionPattern<xegpu::LoadMatrixOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(xegpu::LoadMatrixOp op, OneToNOpAdaptor adaptor,
````
- **L937 EN**: Starts a function, method, lambda, or structured scope: `offsetsVecType.getShape() != maskVecType.getShape()) {`.
  **L937 CN**: 开始一个函数、方法、lambda 或结构化作用域：`offsetsVecType.getShape() != maskVecType.getShape()) {`。
- **L938 EN**: Returns from the current function with `rewriter.notifyMatchFailure(op,`.
  **L938 CN**: 以 `rewriter.notifyMatchFailure(op,` 从当前函数返回。
- **L939 EN**: Executes a standalone statement or declaration: `"offsets have not been distributed");`.
  **L939 CN**: 执行一条独立语句或声明：`"offsets have not been distributed");`。
- **L940 EN**: Closes the current lexical scope or compound statement.
  **L940 CN**: 结束当前词法作用域或复合语句块。
- **L941 EN**: Blank line separating nearby declarations or logic blocks.
  **L941 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L942 EN**: Initializes variable `chunkSizeOpt` from the right-hand expression.
  **L942 CN**: 使用右侧表达式初始化变量 `chunkSizeOpt`。
- **L943 EN**: Initializes variable `chunkSize` from the right-hand expression.
  **L943 CN**: 使用右侧表达式初始化变量 `chunkSize`。
- **L944 EN**: Initializes variable `chunkSizeAttr` from the right-hand expression.
  **L944 CN**: 使用右侧表达式初始化变量 `chunkSizeAttr`。
- **L945 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L945 CN**: 开始 `for` 控制流语句并计算其条件。
- **L946 EN**: Starts a function, method, lambda, or structured scope: `adaptor.getValue(), adaptor.getOffsets(), adaptor.getMask())) {`.
  **L946 CN**: 开始一个函数、方法、lambda 或结构化作用域：`adaptor.getValue(), adaptor.getOffsets(), adaptor.getMask())) {`。
- **L947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::StoreScatterOp::create(rewriter, loc, val, op.getDest(), offs,`.
  **L947 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::StoreScatterOp::create(rewriter, loc, val, op.getDest(), offs,`。
- **L948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mask, chunkSizeAttr, op.getL1HintAttr(),`.
  **L948 CN**: 继续一个多行参数列表、初始化器或聚合项：`mask, chunkSizeAttr, op.getL1HintAttr(),`。
- **L949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op.getL2HintAttr(), op.getL3HintAttr(),`.
  **L949 CN**: 继续一个多行参数列表、初始化器或聚合项：`op.getL2HintAttr(), op.getL3HintAttr(),`。
- **L950 EN**: Executes a call or declaration centered on `layout.dropSgLayoutAndData`.
  **L950 CN**: 执行以 `layout.dropSgLayoutAndData` 为核心的调用或声明。
- **L951 EN**: Closes the current lexical scope or compound statement.
  **L951 CN**: 结束当前词法作用域或复合语句块。
- **L952 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L952 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L953 EN**: Returns from the current function with `success()`.
  **L953 CN**: 以 `success()` 从当前函数返回。
- **L954 EN**: Closes the current lexical scope or compound statement.
  **L954 CN**: 结束当前词法作用域或复合语句块。
- **L955 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L955 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L956 EN**: Blank line separating nearby declarations or logic blocks.
  **L956 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L957 EN**: Declares struct `WgToSgLoadMatrixOp`.
  **L957 CN**: 声明 struct `WgToSgLoadMatrixOp`。
- **L958 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::LoadMatrixOp>::OpConversionPattern;`.
  **L958 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::LoadMatrixOp>::OpConversionPattern;`。
- **L959 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L959 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::LoadMatrixOp op, OneToNOpAdaptor adaptor,`.
  **L960 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::LoadMatrixOp op, OneToNOpAdaptor adaptor,`。

### Lines 961-984

````cpp
                  ConversionPatternRewriter &rewriter) const override {

    SmallVector<SmallVector<OpFoldResult>> offsetsList;
    if (failed(genOffsetsList(rewriter, op, offsetsList)))
      return failure();

    ArrayRef<int64_t> wgShape = op.getDataShape();
    VectorType valueTy = llvm::dyn_cast<VectorType>(op.getRes().getType());
    assert(valueTy && "the value type must be vector type!");
    Type elemTy = valueTy.getElementType();

    xegpu::DistributeLayoutAttr layout = op.getLayoutAttr();
    SmallVector<int64_t> sgShape = getSgShapeAndCount(wgShape, layout).first;
    VectorType newResTy = VectorType::get(sgShape, elemTy);
    SmallVector<Value> newOps;
    for (auto offsets : offsetsList) {
      auto newOp = xegpu::LoadMatrixOp::create(rewriter, op.getLoc(), newResTy,
                                               op.getMemDesc(), offsets,
                                               layout.dropSgLayoutAndData());
      newOps.push_back(newOp);
    }
    rewriter.replaceOpWithMultiple(op, {newOps});

    return success();
````
- **L961 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L961 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L963 EN**: Executes a standalone statement or declaration: `SmallVector<SmallVector<OpFoldResult>> offsetsList;`.
  **L963 CN**: 执行一条独立语句或声明：`SmallVector<SmallVector<OpFoldResult>> offsetsList;`。
- **L964 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L964 CN**: 开始 `if` 控制流语句并计算其条件。
- **L965 EN**: Returns from the current function with `failure()`.
  **L965 CN**: 以 `failure()` 从当前函数返回。
- **L966 EN**: Blank line separating nearby declarations or logic blocks.
  **L966 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L967 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L967 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L968 EN**: Initializes variable `valueTy` from the right-hand expression.
  **L968 CN**: 使用右侧表达式初始化变量 `valueTy`。
- **L969 EN**: Checks an internal invariant in debug builds.
  **L969 CN**: 在调试构建中检查内部不变式。
- **L970 EN**: Initializes variable `elemTy` from the right-hand expression.
  **L970 CN**: 使用右侧表达式初始化变量 `elemTy`。
- **L971 EN**: Blank line separating nearby declarations or logic blocks.
  **L971 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L972 EN**: Initializes variable `layout` from the right-hand expression.
  **L972 CN**: 使用右侧表达式初始化变量 `layout`。
- **L973 EN**: Initializes variable `sgShape` from the right-hand expression.
  **L973 CN**: 使用右侧表达式初始化变量 `sgShape`。
- **L974 EN**: Initializes variable `newResTy` from the right-hand expression.
  **L974 CN**: 使用右侧表达式初始化变量 `newResTy`。
- **L975 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newOps;`.
  **L975 CN**: 执行一条独立语句或声明：`SmallVector<Value> newOps;`。
- **L976 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L976 CN**: 开始 `for` 控制流语句并计算其条件。
- **L977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto newOp = xegpu::LoadMatrixOp::create(rewriter, op.getLoc(), newResTy,`.
  **L977 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto newOp = xegpu::LoadMatrixOp::create(rewriter, op.getLoc(), newResTy,`。
- **L978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op.getMemDesc(), offsets,`.
  **L978 CN**: 继续一个多行参数列表、初始化器或聚合项：`op.getMemDesc(), offsets,`。
- **L979 EN**: Executes a call or declaration centered on `layout.dropSgLayoutAndData`.
  **L979 CN**: 执行以 `layout.dropSgLayoutAndData` 为核心的调用或声明。
- **L980 EN**: Executes a call or declaration centered on `newOps.push_back`.
  **L980 CN**: 执行以 `newOps.push_back` 为核心的调用或声明。
- **L981 EN**: Closes the current lexical scope or compound statement.
  **L981 CN**: 结束当前词法作用域或复合语句块。
- **L982 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L982 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L983 EN**: Blank line separating nearby declarations or logic blocks.
  **L983 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L984 EN**: Returns from the current function with `success()`.
  **L984 CN**: 以 `success()` 从当前函数返回。

### Lines 985-1008

````cpp
  }
};

struct WgToSgStoreMatrixOp : public OpConversionPattern<xegpu::StoreMatrixOp> {
  using OpConversionPattern<xegpu::StoreMatrixOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(xegpu::StoreMatrixOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {

    SmallVector<SmallVector<OpFoldResult>> offsetsList;
    if (failed(genOffsetsList(rewriter, op, offsetsList)))
      return failure();

    xegpu::DistributeLayoutAttr layout = op.getLayoutAttr();
    for (auto [v, offsets] : llvm::zip(adaptor.getData(), offsetsList))
      xegpu::StoreMatrixOp::create(rewriter, op.getLoc(), v, op.getMemDesc(),
                                   offsets, layout.dropSgLayoutAndData());
    rewriter.eraseOp(op);
    return success();
  }
};

// This pattern distributes the vector.step ops to work at subgroup level
struct WgToSgVectorStepOp : public OpConversionPattern<vector::StepOp> {
````
- **L985 EN**: Closes the current lexical scope or compound statement.
  **L985 CN**: 结束当前词法作用域或复合语句块。
- **L986 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L986 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L987 EN**: Blank line separating nearby declarations or logic blocks.
  **L987 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L988 EN**: Declares struct `WgToSgStoreMatrixOp`.
  **L988 CN**: 声明 struct `WgToSgStoreMatrixOp`。
- **L989 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::StoreMatrixOp>::OpConversionPattern;`.
  **L989 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::StoreMatrixOp>::OpConversionPattern;`。
- **L990 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L990 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L991 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::StoreMatrixOp op, OneToNOpAdaptor adaptor,`.
  **L991 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::StoreMatrixOp op, OneToNOpAdaptor adaptor,`。
- **L992 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L992 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L993 EN**: Blank line separating nearby declarations or logic blocks.
  **L993 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L994 EN**: Executes a standalone statement or declaration: `SmallVector<SmallVector<OpFoldResult>> offsetsList;`.
  **L994 CN**: 执行一条独立语句或声明：`SmallVector<SmallVector<OpFoldResult>> offsetsList;`。
- **L995 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L995 CN**: 开始 `if` 控制流语句并计算其条件。
- **L996 EN**: Returns from the current function with `failure()`.
  **L996 CN**: 以 `failure()` 从当前函数返回。
- **L997 EN**: Blank line separating nearby declarations or logic blocks.
  **L997 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L998 EN**: Initializes variable `layout` from the right-hand expression.
  **L998 CN**: 使用右侧表达式初始化变量 `layout`。
- **L999 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L999 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::StoreMatrixOp::create(rewriter, op.getLoc(), v, op.getMemDesc(),`.
  **L1000 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::StoreMatrixOp::create(rewriter, op.getLoc(), v, op.getMemDesc(),`。
- **L1001 EN**: Executes a call or declaration centered on `layout.dropSgLayoutAndData`.
  **L1001 CN**: 执行以 `layout.dropSgLayoutAndData` 为核心的调用或声明。
- **L1002 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L1002 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L1003 EN**: Returns from the current function with `success()`.
  **L1003 CN**: 以 `success()` 从当前函数返回。
- **L1004 EN**: Closes the current lexical scope or compound statement.
  **L1004 CN**: 结束当前词法作用域或复合语句块。
- **L1005 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1005 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1006 EN**: Blank line separating nearby declarations or logic blocks.
  **L1006 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1007 EN**: Comment explains nearby logic, invariants, or intent: `This pattern distributes the vector.step ops to work at subgroup level`.
  **L1007 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern distributes the vector.step ops to work at subgroup level`。
- **L1008 EN**: Declares struct `WgToSgVectorStepOp`.
  **L1008 CN**: 声明 struct `WgToSgVectorStepOp`。

### Lines 1009-1032

````cpp
  using OpConversionPattern<vector::StepOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(vector::StepOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    xegpu::DistributeLayoutAttr layout =
        xegpu::getTemporaryLayout(dyn_cast<OpResult>(op.getResult()));
    if (!layout || !layout.isForWorkgroup())
      return failure();

    Location loc = op.getLoc();
    VectorType type = op.getResult().getType();
    auto wgShape = type.getShape();
    std::optional<SmallVector<int64_t>> sgShape =
        getSgShapeAndCount(wgShape, layout).first;
    if (!sgShape)
      return failure();

    Value sgId =
        gpu::SubgroupIdOp::create(rewriter, loc, /*upper_bound=*/nullptr);
    auto sgOffsets =
        layout.computeDistributedCoords(rewriter, loc, sgId, wgShape);
    if (failed(sgOffsets))
      return failure();

````
- **L1009 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<vector::StepOp>::OpConversionPattern;`.
  **L1009 CN**: 执行一条独立语句或声明：`using OpConversionPattern<vector::StepOp>::OpConversionPattern;`。
- **L1010 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L1010 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L1011 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(vector::StepOp op, OneToNOpAdaptor adaptor,`.
  **L1011 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(vector::StepOp op, OneToNOpAdaptor adaptor,`。
- **L1012 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L1012 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L1013 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout =`.
  **L1013 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout =`。
- **L1014 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L1014 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L1015 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1015 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1016 EN**: Returns from the current function with `failure()`.
  **L1016 CN**: 以 `failure()` 从当前函数返回。
- **L1017 EN**: Blank line separating nearby declarations or logic blocks.
  **L1017 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1018 EN**: Initializes variable `loc` from the right-hand expression.
  **L1018 CN**: 使用右侧表达式初始化变量 `loc`。
- **L1019 EN**: Initializes variable `type` from the right-hand expression.
  **L1019 CN**: 使用右侧表达式初始化变量 `type`。
- **L1020 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L1020 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L1021 EN**: Continues the surrounding expression or declaration: `std::optional<SmallVector<int64_t>> sgShape =`.
  **L1021 CN**: 继续构造周围的表达式或声明：`std::optional<SmallVector<int64_t>> sgShape =`。
- **L1022 EN**: Executes a call or declaration centered on `getSgShapeAndCount`.
  **L1022 CN**: 执行以 `getSgShapeAndCount` 为核心的调用或声明。
- **L1023 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1023 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1024 EN**: Returns from the current function with `failure()`.
  **L1024 CN**: 以 `failure()` 从当前函数返回。
- **L1025 EN**: Blank line separating nearby declarations or logic blocks.
  **L1025 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1026 EN**: Continues the surrounding expression or declaration: `Value sgId =`.
  **L1026 CN**: 继续构造周围的表达式或声明：`Value sgId =`。
- **L1027 EN**: Executes a call or declaration centered on `gpu::SubgroupIdOp::create`.
  **L1027 CN**: 执行以 `gpu::SubgroupIdOp::create` 为核心的调用或声明。
- **L1028 EN**: Continues the surrounding expression or declaration: `auto sgOffsets =`.
  **L1028 CN**: 继续构造周围的表达式或声明：`auto sgOffsets =`。
- **L1029 EN**: Executes a call or declaration centered on `layout.computeDistributedCoords`.
  **L1029 CN**: 执行以 `layout.computeDistributedCoords` 为核心的调用或声明。
- **L1030 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1030 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1031 EN**: Returns from the current function with `failure()`.
  **L1031 CN**: 以 `failure()` 从当前函数返回。
- **L1032 EN**: Blank line separating nearby declarations or logic blocks.
  **L1032 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1033-1056

````cpp
    VectorType newTy = type.cloneWith(*sgShape, type.getElementType());
    auto steps = vector::StepOp::create(rewriter, loc, newTy);
    SmallVector<Value> newOps;
    for (auto offsets : *sgOffsets) {
      // Broadcast the offset scalar to a vector & add to the base steps
      auto bcastOffset =
          vector::BroadcastOp::create(rewriter, loc, newTy, offsets[0]);
      auto finalSteps =
          arith::AddIOp::create(rewriter, loc, steps, bcastOffset);
      newOps.push_back(finalSteps);
    }

    rewriter.replaceOpWithMultiple(op, {newOps});
    return success();
  }
};

// This pattern transforms vector.shape_cast ops to work at subgroup level.
struct WgToSgVectorShapeCastOp
    : public OpConversionPattern<vector::ShapeCastOp> {
  using OpConversionPattern<vector::ShapeCastOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(vector::ShapeCastOp op, OneToNOpAdaptor adaptor,
````
- **L1033 EN**: Initializes variable `newTy` from the right-hand expression.
  **L1033 CN**: 使用右侧表达式初始化变量 `newTy`。
- **L1034 EN**: Initializes variable `steps` from the right-hand expression.
  **L1034 CN**: 使用右侧表达式初始化变量 `steps`。
- **L1035 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newOps;`.
  **L1035 CN**: 执行一条独立语句或声明：`SmallVector<Value> newOps;`。
- **L1036 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1036 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1037 EN**: Comment explains nearby logic, invariants, or intent: `Broadcast the offset scalar to a vector & add to the base steps`.
  **L1037 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Broadcast the offset scalar to a vector & add to the base steps`。
- **L1038 EN**: Continues the surrounding expression or declaration: `auto bcastOffset =`.
  **L1038 CN**: 继续构造周围的表达式或声明：`auto bcastOffset =`。
- **L1039 EN**: Executes a call or declaration centered on `vector::BroadcastOp::create`.
  **L1039 CN**: 执行以 `vector::BroadcastOp::create` 为核心的调用或声明。
- **L1040 EN**: Continues the surrounding expression or declaration: `auto finalSteps =`.
  **L1040 CN**: 继续构造周围的表达式或声明：`auto finalSteps =`。
- **L1041 EN**: Executes a call or declaration centered on `arith::AddIOp::create`.
  **L1041 CN**: 执行以 `arith::AddIOp::create` 为核心的调用或声明。
- **L1042 EN**: Executes a call or declaration centered on `newOps.push_back`.
  **L1042 CN**: 执行以 `newOps.push_back` 为核心的调用或声明。
- **L1043 EN**: Closes the current lexical scope or compound statement.
  **L1043 CN**: 结束当前词法作用域或复合语句块。
- **L1044 EN**: Blank line separating nearby declarations or logic blocks.
  **L1044 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1045 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L1045 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L1046 EN**: Returns from the current function with `success()`.
  **L1046 CN**: 以 `success()` 从当前函数返回。
- **L1047 EN**: Closes the current lexical scope or compound statement.
  **L1047 CN**: 结束当前词法作用域或复合语句块。
- **L1048 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1048 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1049 EN**: Blank line separating nearby declarations or logic blocks.
  **L1049 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1050 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms vector.shape_cast ops to work at subgroup level.`.
  **L1050 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms vector.shape_cast ops to work at subgroup level.`。
- **L1051 EN**: Declares struct `WgToSgVectorShapeCastOp`.
  **L1051 CN**: 声明 struct `WgToSgVectorShapeCastOp`。
- **L1052 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<vector::ShapeCastOp> {`.
  **L1052 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<vector::ShapeCastOp> {`。
- **L1053 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<vector::ShapeCastOp>::OpConversionPattern;`.
  **L1053 CN**: 执行一条独立语句或声明：`using OpConversionPattern<vector::ShapeCastOp>::OpConversionPattern;`。
- **L1054 EN**: Blank line separating nearby declarations or logic blocks.
  **L1054 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1055 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L1055 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L1056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(vector::ShapeCastOp op, OneToNOpAdaptor adaptor,`.
  **L1056 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(vector::ShapeCastOp op, OneToNOpAdaptor adaptor,`。

### Lines 1057-1080

````cpp
                  ConversionPatternRewriter &rewriter) const override {

    VectorType resultType = dyn_cast<VectorType>(op.getResult().getType());
    if (!resultType)
      return failure();

    ArrayRef<int64_t> wgShape = resultType.getShape();
    xegpu::DistributeLayoutAttr layout =
        xegpu::getTemporaryLayout(dyn_cast<OpResult>(op.getResult()));
    if (!layout || !layout.isForWorkgroup())
      return failure();

    // Check that srcShape and destShape, if they differ, only differ by
    // expand of unit dimensions.
    auto srcType = dyn_cast<VectorType>(op.getSource().getType());
    if (!srcType)
      return failure();

    ArrayRef<int64_t> srcShape = srcType.getShape();

    xegpu::DistributeLayoutAttr layoutToDistribute = layout;
    SmallVector<int64_t> expandedUnitDims;
    if (xegpu::matchUnitDimExpansion(srcShape, wgShape, expandedUnitDims)) {
      xegpu::DistributeLayoutAttr sourceLayout =
````
- **L1057 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L1057 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L1058 EN**: Blank line separating nearby declarations or logic blocks.
  **L1058 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1059 EN**: Initializes variable `resultType` from the right-hand expression.
  **L1059 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L1060 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1060 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1061 EN**: Returns from the current function with `failure()`.
  **L1061 CN**: 以 `failure()` 从当前函数返回。
- **L1062 EN**: Blank line separating nearby declarations or logic blocks.
  **L1062 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1063 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L1063 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L1064 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout =`.
  **L1064 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout =`。
- **L1065 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L1065 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L1066 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1066 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1067 EN**: Returns from the current function with `failure()`.
  **L1067 CN**: 以 `failure()` 从当前函数返回。
- **L1068 EN**: Blank line separating nearby declarations or logic blocks.
  **L1068 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1069 EN**: Comment explains nearby logic, invariants, or intent: `Check that srcShape and destShape, if they differ, only differ by`.
  **L1069 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check that srcShape and destShape, if they differ, only differ by`。
- **L1070 EN**: Comment explains nearby logic, invariants, or intent: `expand of unit dimensions.`.
  **L1070 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`expand of unit dimensions.`。
- **L1071 EN**: Initializes variable `srcType` from the right-hand expression.
  **L1071 CN**: 使用右侧表达式初始化变量 `srcType`。
- **L1072 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1072 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1073 EN**: Returns from the current function with `failure()`.
  **L1073 CN**: 以 `failure()` 从当前函数返回。
- **L1074 EN**: Blank line separating nearby declarations or logic blocks.
  **L1074 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1075 EN**: Initializes variable `srcShape` from the right-hand expression.
  **L1075 CN**: 使用右侧表达式初始化变量 `srcShape`。
- **L1076 EN**: Blank line separating nearby declarations or logic blocks.
  **L1076 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1077 EN**: Initializes variable `layoutToDistribute` from the right-hand expression.
  **L1077 CN**: 使用右侧表达式初始化变量 `layoutToDistribute`。
- **L1078 EN**: Executes a standalone statement or declaration: `SmallVector<int64_t> expandedUnitDims;`.
  **L1078 CN**: 执行一条独立语句或声明：`SmallVector<int64_t> expandedUnitDims;`。
- **L1079 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1079 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1080 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr sourceLayout =`.
  **L1080 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr sourceLayout =`。

### Lines 1081-1104

````cpp
          xegpu::getTemporaryLayout(op->getOpOperand(0));

      if (!sourceLayout.isSliceOf(layout))
        return rewriter.notifyMatchFailure(
            op, "The ShapeCast op only expands dimensions, the input layout "
                "must be a slice of the result layout.");

      assert(layoutToDistribute.isEqualTo(
                 layoutToDistribute.setUnitDimData(expandedUnitDims)) &&
             "The sg_data for unit dimensions should be set as 1");
    }

    SmallVector<int64_t> sgShape =
        getSgShapeAndCount(wgShape, layoutToDistribute).first;
    VectorType newResultType =
        VectorType::get(sgShape, resultType.getElementType());

    SmallVector<Value> newShapeCastOps;
    for (auto src : adaptor.getSource()) {
      auto newShapeCast = vector::ShapeCastOp::create(rewriter, op.getLoc(),
                                                      newResultType, src);
      newShapeCastOps.push_back(newShapeCast.getResult());
    }

````
- **L1081 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L1081 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L1082 EN**: Blank line separating nearby declarations or logic blocks.
  **L1082 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1083 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1083 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1084 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L1084 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L1085 EN**: Continues the surrounding expression or declaration: `op, "The ShapeCast op only expands dimensions, the input layout "`.
  **L1085 CN**: 继续构造周围的表达式或声明：`op, "The ShapeCast op only expands dimensions, the input layout "`。
- **L1086 EN**: Executes a standalone statement or declaration: `"must be a slice of the result layout.");`.
  **L1086 CN**: 执行一条独立语句或声明：`"must be a slice of the result layout.");`。
- **L1087 EN**: Blank line separating nearby declarations or logic blocks.
  **L1087 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1088 EN**: Checks an internal invariant in debug builds.
  **L1088 CN**: 在调试构建中检查内部不变式。
- **L1089 EN**: Continues logic associated with callable symbol `setUnitDimData`.
  **L1089 CN**: 继续与可调用符号 `setUnitDimData` 相关的逻辑。
- **L1090 EN**: Executes a standalone statement or declaration: `"The sg_data for unit dimensions should be set as 1");`.
  **L1090 CN**: 执行一条独立语句或声明：`"The sg_data for unit dimensions should be set as 1");`。
- **L1091 EN**: Closes the current lexical scope or compound statement.
  **L1091 CN**: 结束当前词法作用域或复合语句块。
- **L1092 EN**: Blank line separating nearby declarations or logic blocks.
  **L1092 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1093 EN**: Continues the surrounding expression or declaration: `SmallVector<int64_t> sgShape =`.
  **L1093 CN**: 继续构造周围的表达式或声明：`SmallVector<int64_t> sgShape =`。
- **L1094 EN**: Executes a call or declaration centered on `getSgShapeAndCount`.
  **L1094 CN**: 执行以 `getSgShapeAndCount` 为核心的调用或声明。
- **L1095 EN**: Continues the surrounding expression or declaration: `VectorType newResultType =`.
  **L1095 CN**: 继续构造周围的表达式或声明：`VectorType newResultType =`。
- **L1096 EN**: Executes a call or declaration centered on `VectorType::get`.
  **L1096 CN**: 执行以 `VectorType::get` 为核心的调用或声明。
- **L1097 EN**: Blank line separating nearby declarations or logic blocks.
  **L1097 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1098 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newShapeCastOps;`.
  **L1098 CN**: 执行一条独立语句或声明：`SmallVector<Value> newShapeCastOps;`。
- **L1099 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1099 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto newShapeCast = vector::ShapeCastOp::create(rewriter, op.getLoc(),`.
  **L1100 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto newShapeCast = vector::ShapeCastOp::create(rewriter, op.getLoc(),`。
- **L1101 EN**: Executes a standalone statement or declaration: `newResultType, src);`.
  **L1101 CN**: 执行一条独立语句或声明：`newResultType, src);`。
- **L1102 EN**: Executes a call or declaration centered on `newShapeCastOps.push_back`.
  **L1102 CN**: 执行以 `newShapeCastOps.push_back` 为核心的调用或声明。
- **L1103 EN**: Closes the current lexical scope or compound statement.
  **L1103 CN**: 结束当前词法作用域或复合语句块。
- **L1104 EN**: Blank line separating nearby declarations or logic blocks.
  **L1104 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1105-1128

````cpp
    rewriter.replaceOpWithMultiple(op, {newShapeCastOps});
    return success();
  }
};

/// This pattern transforms vector.multi_dim_reduction operations from
/// workgroup-level to subgroup-level execution with support for multiple
/// reduction dimensions.
///
/// Steps include:
/// 1. LOCAL REDUCTION :
///    - Each subgroup performs local reduction on its data slice
///    - Uses ZERO accumulator to avoid double-counting during cross-subgroup
///    phase
///
/// 2. CROSS-SUBGROUP :
///    - Determines if cross-subgroup reduction is needed (when sg_layout > 1 in
///      reduction dims & sgData[reduction dims] < wgData[reduction dims])
///    - If not needed, adds original accumulator and returns local results
///
/// 3. SHARED LOCAL MEMORY (SLM) PHASE (when cross-subgroup reduction needed):
///    a) SLM Layout Design:
///       - Rows: subgroups participating in reduction (product of sg_layout in
///       reduction dims)
````
- **L1105 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L1105 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L1106 EN**: Returns from the current function with `success()`.
  **L1106 CN**: 以 `success()` 从当前函数返回。
- **L1107 EN**: Closes the current lexical scope or compound statement.
  **L1107 CN**: 结束当前词法作用域或复合语句块。
- **L1108 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1108 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1109 EN**: Blank line separating nearby declarations or logic blocks.
  **L1109 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1110 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms vector.multi_dim_reduction operations from`.
  **L1110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms vector.multi_dim_reduction operations from`。
- **L1111 EN**: Comment explains nearby logic, invariants, or intent: `workgroup-level to subgroup-level execution with support for multiple`.
  **L1111 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`workgroup-level to subgroup-level execution with support for multiple`。
- **L1112 EN**: Comment explains nearby logic, invariants, or intent: `reduction dimensions.`.
  **L1112 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`reduction dimensions.`。
- **L1113 EN**: Separator comment used for visual grouping.
  **L1113 CN**: 用于视觉分组的分隔注释。
- **L1114 EN**: Comment explains nearby logic, invariants, or intent: `Steps include:`.
  **L1114 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Steps include:`。
- **L1115 EN**: Comment explains nearby logic, invariants, or intent: `1. LOCAL REDUCTION :`.
  **L1115 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`1. LOCAL REDUCTION :`。
- **L1116 EN**: Comment explains nearby logic, invariants, or intent: `Each subgroup performs local reduction on its data slice`.
  **L1116 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Each subgroup performs local reduction on its data slice`。
- **L1117 EN**: Comment explains nearby logic, invariants, or intent: `Uses ZERO accumulator to avoid double-counting during cross-subgroup`.
  **L1117 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Uses ZERO accumulator to avoid double-counting during cross-subgroup`。
- **L1118 EN**: Comment explains nearby logic, invariants, or intent: `phase`.
  **L1118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`phase`。
- **L1119 EN**: Separator comment used for visual grouping.
  **L1119 CN**: 用于视觉分组的分隔注释。
- **L1120 EN**: Comment explains nearby logic, invariants, or intent: `2. CROSS-SUBGROUP :`.
  **L1120 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2. CROSS-SUBGROUP :`。
- **L1121 EN**: Comment explains nearby logic, invariants, or intent: `Determines if cross-subgroup reduction is needed (when sg_layout > 1 in`.
  **L1121 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Determines if cross-subgroup reduction is needed (when sg_layout > 1 in`。
- **L1122 EN**: Comment explains nearby logic, invariants, or intent: `reduction dims & sgData[reduction dims] < wgData[reduction dims])`.
  **L1122 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`reduction dims & sgData[reduction dims] < wgData[reduction dims])`。
- **L1123 EN**: Comment explains nearby logic, invariants, or intent: `If not needed, adds original accumulator and returns local results`.
  **L1123 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If not needed, adds original accumulator and returns local results`。
- **L1124 EN**: Separator comment used for visual grouping.
  **L1124 CN**: 用于视觉分组的分隔注释。
- **L1125 EN**: Comment explains nearby logic, invariants, or intent: `3. SHARED LOCAL MEMORY (SLM) PHASE (when cross-subgroup reduction needed):`.
  **L1125 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`3. SHARED LOCAL MEMORY (SLM) PHASE (when cross-subgroup reduction needed):`。
- **L1126 EN**: Comment explains nearby logic, invariants, or intent: `a) SLM Layout Design:`.
  **L1126 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a) SLM Layout Design:`。
- **L1127 EN**: Comment explains nearby logic, invariants, or intent: `Rows: subgroups participating in reduction (product of sg_layout in`.
  **L1127 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Rows: subgroups participating in reduction (product of sg_layout in`。
- **L1128 EN**: Comment explains nearby logic, invariants, or intent: `reduction dims)`.
  **L1128 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`reduction dims)`。

### Lines 1129-1152

````cpp
///       - Cols: total result elements across non-reduction dimensions
///
///    b) Store Phase:
///       - Each subgroup stores its local reduction result to SLM
///       - Row offset: linearized index of subgroup in reduction dimensions
///       - Col offset: linearized index of subgroup in non-reduction dimensions
///
///    c) Load and Final Reduction Phase:
///       - Each subgroup loads a column of data (all reduction participants for
///       its position)
///       - Performs final reduction along the loaded dimension
///       - Adds original accumulator to get final result
///
struct WgToSgMultiDimReductionOp
    : public OpConversionPattern<vector::MultiDimReductionOp> {
  using OpConversionPattern<vector::MultiDimReductionOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(vector::MultiDimReductionOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op.getLoc();

    VectorType srcType = op.getSourceVectorType();
    Type resultTy = op.getResult().getType();
````
- **L1129 EN**: Comment explains nearby logic, invariants, or intent: `Cols: total result elements across non-reduction dimensions`.
  **L1129 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Cols: total result elements across non-reduction dimensions`。
- **L1130 EN**: Separator comment used for visual grouping.
  **L1130 CN**: 用于视觉分组的分隔注释。
- **L1131 EN**: Comment explains nearby logic, invariants, or intent: `b) Store Phase:`.
  **L1131 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`b) Store Phase:`。
- **L1132 EN**: Comment explains nearby logic, invariants, or intent: `Each subgroup stores its local reduction result to SLM`.
  **L1132 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Each subgroup stores its local reduction result to SLM`。
- **L1133 EN**: Comment explains nearby logic, invariants, or intent: `Row offset: linearized index of subgroup in reduction dimensions`.
  **L1133 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Row offset: linearized index of subgroup in reduction dimensions`。
- **L1134 EN**: Comment explains nearby logic, invariants, or intent: `Col offset: linearized index of subgroup in non-reduction dimensions`.
  **L1134 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Col offset: linearized index of subgroup in non-reduction dimensions`。
- **L1135 EN**: Separator comment used for visual grouping.
  **L1135 CN**: 用于视觉分组的分隔注释。
- **L1136 EN**: Comment explains nearby logic, invariants, or intent: `c) Load and Final Reduction Phase:`.
  **L1136 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`c) Load and Final Reduction Phase:`。
- **L1137 EN**: Comment explains nearby logic, invariants, or intent: `Each subgroup loads a column of data (all reduction participants for`.
  **L1137 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Each subgroup loads a column of data (all reduction participants for`。
- **L1138 EN**: Comment explains nearby logic, invariants, or intent: `its position)`.
  **L1138 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`its position)`。
- **L1139 EN**: Comment explains nearby logic, invariants, or intent: `Performs final reduction along the loaded dimension`.
  **L1139 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Performs final reduction along the loaded dimension`。
- **L1140 EN**: Comment explains nearby logic, invariants, or intent: `Adds original accumulator to get final result`.
  **L1140 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Adds original accumulator to get final result`。
- **L1141 EN**: Separator comment used for visual grouping.
  **L1141 CN**: 用于视觉分组的分隔注释。
- **L1142 EN**: Declares struct `WgToSgMultiDimReductionOp`.
  **L1142 CN**: 声明 struct `WgToSgMultiDimReductionOp`。
- **L1143 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<vector::MultiDimReductionOp> {`.
  **L1143 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<vector::MultiDimReductionOp> {`。
- **L1144 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<vector::MultiDimReductionOp>::OpConversionPattern;`.
  **L1144 CN**: 执行一条独立语句或声明：`using OpConversionPattern<vector::MultiDimReductionOp>::OpConversionPattern;`。
- **L1145 EN**: Blank line separating nearby declarations or logic blocks.
  **L1145 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1146 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L1146 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L1147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(vector::MultiDimReductionOp op, OneToNOpAdaptor adaptor,`.
  **L1147 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(vector::MultiDimReductionOp op, OneToNOpAdaptor adaptor,`。
- **L1148 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L1148 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L1149 EN**: Initializes variable `loc` from the right-hand expression.
  **L1149 CN**: 使用右侧表达式初始化变量 `loc`。
- **L1150 EN**: Blank line separating nearby declarations or logic blocks.
  **L1150 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1151 EN**: Initializes variable `srcType` from the right-hand expression.
  **L1151 CN**: 使用右侧表达式初始化变量 `srcType`。
- **L1152 EN**: Initializes variable `resultTy` from the right-hand expression.
  **L1152 CN**: 使用右侧表达式初始化变量 `resultTy`。

### Lines 1153-1176

````cpp
    VectorType dstVecType = dyn_cast<VectorType>(resultTy);
    bool isScalarResult = !dstVecType;

    auto originalSrcShape = srcType.getShape();
    Type elemTy = srcType.getElementType();

    xegpu::DistributeLayoutAttr layout =
        xegpu::getTemporaryLayout(dyn_cast<OpResult>(op.getResult()));
    if (!layout || !layout.isForWorkgroup())
      return failure();

    auto reductionDims = llvm::to_vector(op.getReductionDims());

    // Get sg_layout and sg_data from the parent layout
    SmallVector<int64_t> sgLayout;
    SmallVector<int64_t> sgData;
    xegpu::DistributeLayoutAttr parentLayout;
    if (auto sliceAttr = dyn_cast<xegpu::SliceAttr>(layout)) {
      parentLayout = sliceAttr.getParent();
      sgLayout = parentLayout.getEffectiveSgLayoutAsInt();
      sgData = parentLayout.getEffectiveSgDataAsInt();
    } else
      return rewriter.notifyMatchFailure(
          op, "Reduction should have SliceAttr layout");
````
- **L1153 EN**: Initializes variable `dstVecType` from the right-hand expression.
  **L1153 CN**: 使用右侧表达式初始化变量 `dstVecType`。
- **L1154 EN**: Initializes variable `isScalarResult` from the right-hand expression.
  **L1154 CN**: 使用右侧表达式初始化变量 `isScalarResult`。
- **L1155 EN**: Blank line separating nearby declarations or logic blocks.
  **L1155 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1156 EN**: Initializes variable `originalSrcShape` from the right-hand expression.
  **L1156 CN**: 使用右侧表达式初始化变量 `originalSrcShape`。
- **L1157 EN**: Initializes variable `elemTy` from the right-hand expression.
  **L1157 CN**: 使用右侧表达式初始化变量 `elemTy`。
- **L1158 EN**: Blank line separating nearby declarations or logic blocks.
  **L1158 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1159 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout =`.
  **L1159 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout =`。
- **L1160 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L1160 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L1161 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1161 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1162 EN**: Returns from the current function with `failure()`.
  **L1162 CN**: 以 `failure()` 从当前函数返回。
- **L1163 EN**: Blank line separating nearby declarations or logic blocks.
  **L1163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1164 EN**: Initializes variable `reductionDims` from the right-hand expression.
  **L1164 CN**: 使用右侧表达式初始化变量 `reductionDims`。
- **L1165 EN**: Blank line separating nearby declarations or logic blocks.
  **L1165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1166 EN**: Comment explains nearby logic, invariants, or intent: `Get sg_layout and sg_data from the parent layout`.
  **L1166 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get sg_layout and sg_data from the parent layout`。
- **L1167 EN**: Executes a standalone statement or declaration: `SmallVector<int64_t> sgLayout;`.
  **L1167 CN**: 执行一条独立语句或声明：`SmallVector<int64_t> sgLayout;`。
- **L1168 EN**: Executes a standalone statement or declaration: `SmallVector<int64_t> sgData;`.
  **L1168 CN**: 执行一条独立语句或声明：`SmallVector<int64_t> sgData;`。
- **L1169 EN**: Executes a standalone statement or declaration: `xegpu::DistributeLayoutAttr parentLayout;`.
  **L1169 CN**: 执行一条独立语句或声明：`xegpu::DistributeLayoutAttr parentLayout;`。
- **L1170 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1170 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1171 EN**: Executes a call or declaration centered on `sliceAttr.getParent`.
  **L1171 CN**: 执行以 `sliceAttr.getParent` 为核心的调用或声明。
- **L1172 EN**: Executes a call or declaration centered on `parentLayout.getEffectiveSgLayoutAsInt`.
  **L1172 CN**: 执行以 `parentLayout.getEffectiveSgLayoutAsInt` 为核心的调用或声明。
- **L1173 EN**: Executes a call or declaration centered on `parentLayout.getEffectiveSgDataAsInt`.
  **L1173 CN**: 执行以 `parentLayout.getEffectiveSgDataAsInt` 为核心的调用或声明。
- **L1174 EN**: Continues the surrounding expression or declaration: `} else`.
  **L1174 CN**: 继续构造周围的表达式或声明：`} else`。
- **L1175 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L1175 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L1176 EN**: Executes a standalone statement or declaration: `op, "Reduction should have SliceAttr layout");`.
  **L1176 CN**: 执行一条独立语句或声明：`op, "Reduction should have SliceAttr layout");`。

### Lines 1177-1200

````cpp

    // Step 1: perform local subgroup reductions with neutral accumulator
    SmallVector<Value> localReductions;
    auto sgSrcs = adaptor.getSource();
    auto sgSrcType = dyn_cast<VectorType>(sgSrcs.front().getType());
    SmallVector<int64_t> sgSrcShape(sgSrcType.getShape().begin(),
                                    sgSrcType.getShape().end());

    // Determine the SG-level destination type.
    // For scalar results (all dims reduced), the sg result is also scalar.
    // For vector results, compute the sg destination shape from layout.
    Type sgDstType;
    if (dstVecType) {
      auto originalDstShape = dstVecType.getShape();
      SmallVector<int64_t> sgDstShape =
          getSgShapeAndCount(originalDstShape, layout).first;
      sgDstType = VectorType::get(sgDstShape, elemTy);
    } else {
      sgDstType = elemTy;
    }

    for (auto sgSrc : sgSrcs) {
      // Create neutral accumulator for local reduction
      Value neutralLocalAcc = xegpu::createReductionNeutralValue(
````
- **L1177 EN**: Blank line separating nearby declarations or logic blocks.
  **L1177 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1178 EN**: Comment explains nearby logic, invariants, or intent: `Step 1: perform local subgroup reductions with neutral accumulator`.
  **L1178 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Step 1: perform local subgroup reductions with neutral accumulator`。
- **L1179 EN**: Executes a standalone statement or declaration: `SmallVector<Value> localReductions;`.
  **L1179 CN**: 执行一条独立语句或声明：`SmallVector<Value> localReductions;`。
- **L1180 EN**: Initializes variable `sgSrcs` from the right-hand expression.
  **L1180 CN**: 使用右侧表达式初始化变量 `sgSrcs`。
- **L1181 EN**: Initializes variable `sgSrcType` from the right-hand expression.
  **L1181 CN**: 使用右侧表达式初始化变量 `sgSrcType`。
- **L1182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<int64_t> sgSrcShape(sgSrcType.getShape().begin(),`.
  **L1182 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<int64_t> sgSrcShape(sgSrcType.getShape().begin(),`。
- **L1183 EN**: Executes a call or declaration centered on `sgSrcType.getShape`.
  **L1183 CN**: 执行以 `sgSrcType.getShape` 为核心的调用或声明。
- **L1184 EN**: Blank line separating nearby declarations or logic blocks.
  **L1184 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1185 EN**: Comment explains nearby logic, invariants, or intent: `Determine the SG-level destination type.`.
  **L1185 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Determine the SG-level destination type.`。
- **L1186 EN**: Comment explains nearby logic, invariants, or intent: `For scalar results (all dims reduced), the sg result is also scalar.`.
  **L1186 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For scalar results (all dims reduced), the sg result is also scalar.`。
- **L1187 EN**: Comment explains nearby logic, invariants, or intent: `For vector results, compute the sg destination shape from layout.`.
  **L1187 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For vector results, compute the sg destination shape from layout.`。
- **L1188 EN**: Executes a standalone statement or declaration: `Type sgDstType;`.
  **L1188 CN**: 执行一条独立语句或声明：`Type sgDstType;`。
- **L1189 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1189 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1190 EN**: Initializes variable `originalDstShape` from the right-hand expression.
  **L1190 CN**: 使用右侧表达式初始化变量 `originalDstShape`。
- **L1191 EN**: Continues the surrounding expression or declaration: `SmallVector<int64_t> sgDstShape =`.
  **L1191 CN**: 继续构造周围的表达式或声明：`SmallVector<int64_t> sgDstShape =`。
- **L1192 EN**: Executes a call or declaration centered on `getSgShapeAndCount`.
  **L1192 CN**: 执行以 `getSgShapeAndCount` 为核心的调用或声明。
- **L1193 EN**: Executes a call or declaration centered on `VectorType::get`.
  **L1193 CN**: 执行以 `VectorType::get` 为核心的调用或声明。
- **L1194 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L1194 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L1195 EN**: Executes a standalone statement or declaration: `sgDstType = elemTy;`.
  **L1195 CN**: 执行一条独立语句或声明：`sgDstType = elemTy;`。
- **L1196 EN**: Closes the current lexical scope or compound statement.
  **L1196 CN**: 结束当前词法作用域或复合语句块。
- **L1197 EN**: Blank line separating nearby declarations or logic blocks.
  **L1197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1198 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1198 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1199 EN**: Comment explains nearby logic, invariants, or intent: `Create neutral accumulator for local reduction`.
  **L1199 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create neutral accumulator for local reduction`。
- **L1200 EN**: Continues logic associated with callable symbol `createReductionNeutralValue`.
  **L1200 CN**: 继续与可调用符号 `createReductionNeutralValue` 相关的逻辑。

### Lines 1201-1224

````cpp
          rewriter, loc, sgDstType, op.getKind());
      // Local reduction with neutral accumulator
      auto localReduce = vector::MultiDimReductionOp::create(
          rewriter, loc, sgDstType, op.getKind(), sgSrc, neutralLocalAcc,
          reductionDims);
      localReductions.push_back(localReduce.getResult());
    }

    // Check if cross-subgroup reduction is needed for any reduction dimension
    SmallVector<int64_t> crossSgReductionDims;
    for (int64_t reductionDim : reductionDims) {
      bool needsCrossSubgroupReduction =
          (sgLayout[reductionDim] > 1) &&
          (sgData[reductionDim] < originalSrcShape[reductionDim]);

      if (needsCrossSubgroupReduction) {
        crossSgReductionDims.push_back(reductionDim);
      }
    }

    // If no cross-subgroup reduction needed, add accumulator and return
    if (crossSgReductionDims.empty()) {
      SmallVector<Value> results;
      for (auto localResult : localReductions) {
````
- **L1201 EN**: Executes a call or declaration centered on `op.getKind`.
  **L1201 CN**: 执行以 `op.getKind` 为核心的调用或声明。
- **L1202 EN**: Comment explains nearby logic, invariants, or intent: `Local reduction with neutral accumulator`.
  **L1202 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Local reduction with neutral accumulator`。
- **L1203 EN**: Continues logic associated with callable symbol `create`.
  **L1203 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, sgDstType, op.getKind(), sgSrc, neutralLocalAcc,`.
  **L1204 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, sgDstType, op.getKind(), sgSrc, neutralLocalAcc,`。
- **L1205 EN**: Executes a standalone statement or declaration: `reductionDims);`.
  **L1205 CN**: 执行一条独立语句或声明：`reductionDims);`。
- **L1206 EN**: Executes a call or declaration centered on `localReductions.push_back`.
  **L1206 CN**: 执行以 `localReductions.push_back` 为核心的调用或声明。
- **L1207 EN**: Closes the current lexical scope or compound statement.
  **L1207 CN**: 结束当前词法作用域或复合语句块。
- **L1208 EN**: Blank line separating nearby declarations or logic blocks.
  **L1208 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1209 EN**: Comment explains nearby logic, invariants, or intent: `Check if cross-subgroup reduction is needed for any reduction dimension`.
  **L1209 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check if cross-subgroup reduction is needed for any reduction dimension`。
- **L1210 EN**: Executes a standalone statement or declaration: `SmallVector<int64_t> crossSgReductionDims;`.
  **L1210 CN**: 执行一条独立语句或声明：`SmallVector<int64_t> crossSgReductionDims;`。
- **L1211 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1211 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1212 EN**: Continues the surrounding expression or declaration: `bool needsCrossSubgroupReduction =`.
  **L1212 CN**: 继续构造周围的表达式或声明：`bool needsCrossSubgroupReduction =`。
- **L1213 EN**: Continues the surrounding expression or declaration: `(sgLayout[reductionDim] > 1) &&`.
  **L1213 CN**: 继续构造周围的表达式或声明：`(sgLayout[reductionDim] > 1) &&`。
- **L1214 EN**: Executes a call or declaration centered on `statement`.
  **L1214 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1215 EN**: Blank line separating nearby declarations or logic blocks.
  **L1215 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1216 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1216 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1217 EN**: Executes a call or declaration centered on `crossSgReductionDims.push_back`.
  **L1217 CN**: 执行以 `crossSgReductionDims.push_back` 为核心的调用或声明。
- **L1218 EN**: Closes the current lexical scope or compound statement.
  **L1218 CN**: 结束当前词法作用域或复合语句块。
- **L1219 EN**: Closes the current lexical scope or compound statement.
  **L1219 CN**: 结束当前词法作用域或复合语句块。
- **L1220 EN**: Blank line separating nearby declarations or logic blocks.
  **L1220 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1221 EN**: Comment explains nearby logic, invariants, or intent: `If no cross-subgroup reduction needed, add accumulator and return`.
  **L1221 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If no cross-subgroup reduction needed, add accumulator and return`。
- **L1222 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1222 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1223 EN**: Executes a standalone statement or declaration: `SmallVector<Value> results;`.
  **L1223 CN**: 执行一条独立语句或声明：`SmallVector<Value> results;`。
- **L1224 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1224 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 1225-1248

````cpp
        auto finalResult = vector::makeArithReduction(
            rewriter, loc, op.getKind(), localResult, adaptor.getAcc()[0]);
        results.push_back(finalResult);
      }
      rewriter.replaceOpWithMultiple(op, {results});
      return success();
    }

    // Step 2: cross-subgroup reduction using SLM - allocating slm memory
    auto slmStoreDataShape = sgSrcShape;
    for (int64_t dim : reductionDims)
      slmStoreDataShape[dim] = 1;
    VectorType slmStoreDataType = VectorType::get(slmStoreDataShape, elemTy);
    SmallVector<Value> slmStoreData;
    for (auto localResult : localReductions) {
      if (isScalarResult) {
        // Scalar result: broadcast scalar to vector<1x...x1> for SLM store
        slmStoreData.push_back(vector::BroadcastOp::create(
            rewriter, loc, slmStoreDataType, localResult));
      } else {
        slmStoreData.push_back(vector::ShapeCastOp::create(
            rewriter, loc, slmStoreDataType, localResult));
      }
    }
````
- **L1225 EN**: Continues logic associated with callable symbol `makeArithReduction`.
  **L1225 CN**: 继续与可调用符号 `makeArithReduction` 相关的逻辑。
- **L1226 EN**: Executes a call or declaration centered on `op.getKind`.
  **L1226 CN**: 执行以 `op.getKind` 为核心的调用或声明。
- **L1227 EN**: Executes a call or declaration centered on `results.push_back`.
  **L1227 CN**: 执行以 `results.push_back` 为核心的调用或声明。
- **L1228 EN**: Closes the current lexical scope or compound statement.
  **L1228 CN**: 结束当前词法作用域或复合语句块。
- **L1229 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L1229 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L1230 EN**: Returns from the current function with `success()`.
  **L1230 CN**: 以 `success()` 从当前函数返回。
- **L1231 EN**: Closes the current lexical scope or compound statement.
  **L1231 CN**: 结束当前词法作用域或复合语句块。
- **L1232 EN**: Blank line separating nearby declarations or logic blocks.
  **L1232 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1233 EN**: Comment explains nearby logic, invariants, or intent: `Step 2: cross-subgroup reduction using SLM - allocating slm memory`.
  **L1233 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Step 2: cross-subgroup reduction using SLM - allocating slm memory`。
- **L1234 EN**: Initializes variable `slmStoreDataShape` from the right-hand expression.
  **L1234 CN**: 使用右侧表达式初始化变量 `slmStoreDataShape`。
- **L1235 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1235 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1236 EN**: Executes a standalone statement or declaration: `slmStoreDataShape[dim] = 1;`.
  **L1236 CN**: 执行一条独立语句或声明：`slmStoreDataShape[dim] = 1;`。
- **L1237 EN**: Initializes variable `slmStoreDataType` from the right-hand expression.
  **L1237 CN**: 使用右侧表达式初始化变量 `slmStoreDataType`。
- **L1238 EN**: Executes a standalone statement or declaration: `SmallVector<Value> slmStoreData;`.
  **L1238 CN**: 执行一条独立语句或声明：`SmallVector<Value> slmStoreData;`。
- **L1239 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1239 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1240 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1240 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1241 EN**: Comment explains nearby logic, invariants, or intent: `Scalar result: broadcast scalar to vector<1x...x1> for SLM store`.
  **L1241 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scalar result: broadcast scalar to vector<1x...x1> for SLM store`。
- **L1242 EN**: Continues logic associated with callable symbol `push_back`.
  **L1242 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L1243 EN**: Executes a standalone statement or declaration: `rewriter, loc, slmStoreDataType, localResult));`.
  **L1243 CN**: 执行一条独立语句或声明：`rewriter, loc, slmStoreDataType, localResult));`。
- **L1244 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L1244 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L1245 EN**: Continues logic associated with callable symbol `push_back`.
  **L1245 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L1246 EN**: Executes a standalone statement or declaration: `rewriter, loc, slmStoreDataType, localResult));`.
  **L1246 CN**: 执行一条独立语句或声明：`rewriter, loc, slmStoreDataType, localResult));`。
- **L1247 EN**: Closes the current lexical scope or compound statement.
  **L1247 CN**: 结束当前词法作用域或复合语句块。
- **L1248 EN**: Closes the current lexical scope or compound statement.
  **L1248 CN**: 结束当前词法作用域或复合语句块。

### Lines 1249-1272

````cpp
    // for reduction dimension, SLM stores partial results from each subgroup
    SmallVector<int64_t> slmShape(originalSrcShape.begin(),
                                  originalSrcShape.end());
    SmallVector<int> slmSgData(sgData.begin(), sgData.end());
    SmallVector<int> slmSgLayout(sgLayout.begin(), sgLayout.end());
    for (int dim : reductionDims) {
      slmShape[dim] = sgLayout[dim];
      slmSgData[dim] = 1;
    }
    xegpu::LayoutAttr slmStoreLayout =
        xegpu::LayoutAttr::get(rewriter.getContext(), slmSgLayout, slmSgData);

    // Allocate SLM
    auto bitWidth = elemTy.getIntOrFloatBitWidth();
    auto bytesPerElement = bitWidth / 8;
    auto slmSize = computeProduct(slmShape) * bytesPerElement;
    auto slmTy = MemRefType::get({slmSize}, rewriter.getI8Type(), {}, 3);
    auto slm = memref::AllocaOp::create(rewriter, loc, slmTy);

    auto memDescType = xegpu::MemDescType::get(rewriter.getContext(), slmShape,
                                               elemTy, nullptr);
    auto memDesc =
        xegpu::CreateMemDescOp::create(rewriter, loc, memDescType, slm);

````
- **L1249 EN**: Comment explains nearby logic, invariants, or intent: `for reduction dimension, SLM stores partial results from each subgroup`.
  **L1249 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for reduction dimension, SLM stores partial results from each subgroup`。
- **L1250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<int64_t> slmShape(originalSrcShape.begin(),`.
  **L1250 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<int64_t> slmShape(originalSrcShape.begin(),`。
- **L1251 EN**: Executes a call or declaration centered on `originalSrcShape.end`.
  **L1251 CN**: 执行以 `originalSrcShape.end` 为核心的调用或声明。
- **L1252 EN**: Executes a call or declaration centered on `slmSgData`.
  **L1252 CN**: 执行以 `slmSgData` 为核心的调用或声明。
- **L1253 EN**: Executes a call or declaration centered on `slmSgLayout`.
  **L1253 CN**: 执行以 `slmSgLayout` 为核心的调用或声明。
- **L1254 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1254 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1255 EN**: Executes a standalone statement or declaration: `slmShape[dim] = sgLayout[dim];`.
  **L1255 CN**: 执行一条独立语句或声明：`slmShape[dim] = sgLayout[dim];`。
- **L1256 EN**: Executes a standalone statement or declaration: `slmSgData[dim] = 1;`.
  **L1256 CN**: 执行一条独立语句或声明：`slmSgData[dim] = 1;`。
- **L1257 EN**: Closes the current lexical scope or compound statement.
  **L1257 CN**: 结束当前词法作用域或复合语句块。
- **L1258 EN**: Continues the surrounding expression or declaration: `xegpu::LayoutAttr slmStoreLayout =`.
  **L1258 CN**: 继续构造周围的表达式或声明：`xegpu::LayoutAttr slmStoreLayout =`。
- **L1259 EN**: Executes a call or declaration centered on `xegpu::LayoutAttr::get`.
  **L1259 CN**: 执行以 `xegpu::LayoutAttr::get` 为核心的调用或声明。
- **L1260 EN**: Blank line separating nearby declarations or logic blocks.
  **L1260 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1261 EN**: Comment explains nearby logic, invariants, or intent: `Allocate SLM`.
  **L1261 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Allocate SLM`。
- **L1262 EN**: Initializes variable `bitWidth` from the right-hand expression.
  **L1262 CN**: 使用右侧表达式初始化变量 `bitWidth`。
- **L1263 EN**: Initializes variable `bytesPerElement` from the right-hand expression.
  **L1263 CN**: 使用右侧表达式初始化变量 `bytesPerElement`。
- **L1264 EN**: Initializes variable `slmSize` from the right-hand expression.
  **L1264 CN**: 使用右侧表达式初始化变量 `slmSize`。
- **L1265 EN**: Initializes variable `slmTy` from the right-hand expression.
  **L1265 CN**: 使用右侧表达式初始化变量 `slmTy`。
- **L1266 EN**: Initializes variable `slm` from the right-hand expression.
  **L1266 CN**: 使用右侧表达式初始化变量 `slm`。
- **L1267 EN**: Blank line separating nearby declarations or logic blocks.
  **L1267 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto memDescType = xegpu::MemDescType::get(rewriter.getContext(), slmShape,`.
  **L1268 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto memDescType = xegpu::MemDescType::get(rewriter.getContext(), slmShape,`。
- **L1269 EN**: Executes a standalone statement or declaration: `elemTy, nullptr);`.
  **L1269 CN**: 执行一条独立语句或声明：`elemTy, nullptr);`。
- **L1270 EN**: Continues the surrounding expression or declaration: `auto memDesc =`.
  **L1270 CN**: 继续构造周围的表达式或声明：`auto memDesc =`。
- **L1271 EN**: Executes a call or declaration centered on `xegpu::CreateMemDescOp::create`.
  **L1271 CN**: 执行以 `xegpu::CreateMemDescOp::create` 为核心的调用或声明。
- **L1272 EN**: Blank line separating nearby declarations or logic blocks.
  **L1272 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1273-1296

````cpp
    // Step 3: Store local results to SLM
    auto sgId = gpu::SubgroupIdOp::create(rewriter, loc,
                                          rewriter.getIndexType(), nullptr);

    auto slmStoreCoords =
        slmStoreLayout.computeDistributedCoords(rewriter, loc, sgId, slmShape);
    if (failed(slmStoreCoords))
      return failure();
    for (auto [data, coord] : llvm::zip(slmStoreData, *slmStoreCoords)) {
      SmallVector<OpFoldResult> coordOfr(coord.begin(), coord.end());
      xegpu::StoreMatrixOp::create(rewriter, loc, data, memDesc.getResult(),
                                   coordOfr,
                                   /*layout=*/nullptr);
    }

    gpu::BarrierOp::create(rewriter, loc);

    // Step 4: Load from SLM for final reduction
    SmallVector<int64_t> slmLoadDataShape(sgSrcShape.begin(), sgSrcShape.end());
    for (int64_t dim : reductionDims) {
      slmLoadDataShape[dim] = slmShape[dim];
      slmSgData[dim] = slmShape[dim];
    }
    xegpu::LayoutAttr slmLoadLayout =
````
- **L1273 EN**: Comment explains nearby logic, invariants, or intent: `Step 3: Store local results to SLM`.
  **L1273 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Step 3: Store local results to SLM`。
- **L1274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto sgId = gpu::SubgroupIdOp::create(rewriter, loc,`.
  **L1274 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto sgId = gpu::SubgroupIdOp::create(rewriter, loc,`。
- **L1275 EN**: Executes a call or declaration centered on `rewriter.getIndexType`.
  **L1275 CN**: 执行以 `rewriter.getIndexType` 为核心的调用或声明。
- **L1276 EN**: Blank line separating nearby declarations or logic blocks.
  **L1276 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1277 EN**: Continues the surrounding expression or declaration: `auto slmStoreCoords =`.
  **L1277 CN**: 继续构造周围的表达式或声明：`auto slmStoreCoords =`。
- **L1278 EN**: Executes a call or declaration centered on `slmStoreLayout.computeDistributedCoords`.
  **L1278 CN**: 执行以 `slmStoreLayout.computeDistributedCoords` 为核心的调用或声明。
- **L1279 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1279 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1280 EN**: Returns from the current function with `failure()`.
  **L1280 CN**: 以 `failure()` 从当前函数返回。
- **L1281 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1281 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1282 EN**: Executes a call or declaration centered on `coordOfr`.
  **L1282 CN**: 执行以 `coordOfr` 为核心的调用或声明。
- **L1283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::StoreMatrixOp::create(rewriter, loc, data, memDesc.getResult(),`.
  **L1283 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::StoreMatrixOp::create(rewriter, loc, data, memDesc.getResult(),`。
- **L1284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `coordOfr,`.
  **L1284 CN**: 继续一个多行参数列表、初始化器或聚合项：`coordOfr,`。
- **L1285 EN**: Comment explains nearby logic, invariants, or intent: `layout=*/nullptr);`.
  **L1285 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`layout=*/nullptr);`。
- **L1286 EN**: Closes the current lexical scope or compound statement.
  **L1286 CN**: 结束当前词法作用域或复合语句块。
- **L1287 EN**: Blank line separating nearby declarations or logic blocks.
  **L1287 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1288 EN**: Executes a call or declaration centered on `gpu::BarrierOp::create`.
  **L1288 CN**: 执行以 `gpu::BarrierOp::create` 为核心的调用或声明。
- **L1289 EN**: Blank line separating nearby declarations or logic blocks.
  **L1289 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1290 EN**: Comment explains nearby logic, invariants, or intent: `Step 4: Load from SLM for final reduction`.
  **L1290 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Step 4: Load from SLM for final reduction`。
- **L1291 EN**: Executes a call or declaration centered on `slmLoadDataShape`.
  **L1291 CN**: 执行以 `slmLoadDataShape` 为核心的调用或声明。
- **L1292 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1292 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1293 EN**: Executes a standalone statement or declaration: `slmLoadDataShape[dim] = slmShape[dim];`.
  **L1293 CN**: 执行一条独立语句或声明：`slmLoadDataShape[dim] = slmShape[dim];`。
- **L1294 EN**: Executes a standalone statement or declaration: `slmSgData[dim] = slmShape[dim];`.
  **L1294 CN**: 执行一条独立语句或声明：`slmSgData[dim] = slmShape[dim];`。
- **L1295 EN**: Closes the current lexical scope or compound statement.
  **L1295 CN**: 结束当前词法作用域或复合语句块。
- **L1296 EN**: Continues the surrounding expression or declaration: `xegpu::LayoutAttr slmLoadLayout =`.
  **L1296 CN**: 继续构造周围的表达式或声明：`xegpu::LayoutAttr slmLoadLayout =`。

### Lines 1297-1320

````cpp
        xegpu::LayoutAttr::get(rewriter.getContext(), slmSgLayout, slmSgData);
    auto slmLoadCoords =
        slmLoadLayout.computeDistributedCoords(rewriter, loc, sgId, slmShape);
    if (failed(slmLoadCoords))
      return failure();

    VectorType slmLoadType = VectorType::get(slmLoadDataShape, elemTy);
    SmallVector<Value> slmLoadData;
    for (auto coord : *slmLoadCoords) {
      SmallVector<OpFoldResult> coordOfr(coord.begin(), coord.end());
      slmLoadData.push_back(xegpu::LoadMatrixOp::create(
          rewriter, loc, slmLoadType, memDesc.getResult(), coordOfr,
          /*layout=*/nullptr));
    }

    // Step 5: Perform final reduction with neutral accumulator and add the
    // original accumulator at the end
    Value neutralFinalAcc = xegpu::createReductionNeutralValue(
        rewriter, loc, sgDstType, op.getKind());

    SmallVector<Value> finalResults;
    for (size_t i = 0; i < slmLoadData.size(); ++i) {
      auto loaded = slmLoadData[i];
      auto finalReduce = vector::MultiDimReductionOp::create(
````
- **L1297 EN**: Executes a call or declaration centered on `xegpu::LayoutAttr::get`.
  **L1297 CN**: 执行以 `xegpu::LayoutAttr::get` 为核心的调用或声明。
- **L1298 EN**: Continues the surrounding expression or declaration: `auto slmLoadCoords =`.
  **L1298 CN**: 继续构造周围的表达式或声明：`auto slmLoadCoords =`。
- **L1299 EN**: Executes a call or declaration centered on `slmLoadLayout.computeDistributedCoords`.
  **L1299 CN**: 执行以 `slmLoadLayout.computeDistributedCoords` 为核心的调用或声明。
- **L1300 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1300 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1301 EN**: Returns from the current function with `failure()`.
  **L1301 CN**: 以 `failure()` 从当前函数返回。
- **L1302 EN**: Blank line separating nearby declarations or logic blocks.
  **L1302 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1303 EN**: Initializes variable `slmLoadType` from the right-hand expression.
  **L1303 CN**: 使用右侧表达式初始化变量 `slmLoadType`。
- **L1304 EN**: Executes a standalone statement or declaration: `SmallVector<Value> slmLoadData;`.
  **L1304 CN**: 执行一条独立语句或声明：`SmallVector<Value> slmLoadData;`。
- **L1305 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1305 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1306 EN**: Executes a call or declaration centered on `coordOfr`.
  **L1306 CN**: 执行以 `coordOfr` 为核心的调用或声明。
- **L1307 EN**: Continues logic associated with callable symbol `push_back`.
  **L1307 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L1308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, slmLoadType, memDesc.getResult(), coordOfr,`.
  **L1308 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, slmLoadType, memDesc.getResult(), coordOfr,`。
- **L1309 EN**: Comment explains nearby logic, invariants, or intent: `layout=*/nullptr));`.
  **L1309 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`layout=*/nullptr));`。
- **L1310 EN**: Closes the current lexical scope or compound statement.
  **L1310 CN**: 结束当前词法作用域或复合语句块。
- **L1311 EN**: Blank line separating nearby declarations or logic blocks.
  **L1311 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1312 EN**: Comment explains nearby logic, invariants, or intent: `Step 5: Perform final reduction with neutral accumulator and add the`.
  **L1312 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Step 5: Perform final reduction with neutral accumulator and add the`。
- **L1313 EN**: Comment explains nearby logic, invariants, or intent: `original accumulator at the end`.
  **L1313 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`original accumulator at the end`。
- **L1314 EN**: Continues logic associated with callable symbol `createReductionNeutralValue`.
  **L1314 CN**: 继续与可调用符号 `createReductionNeutralValue` 相关的逻辑。
- **L1315 EN**: Executes a call or declaration centered on `op.getKind`.
  **L1315 CN**: 执行以 `op.getKind` 为核心的调用或声明。
- **L1316 EN**: Blank line separating nearby declarations or logic blocks.
  **L1316 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1317 EN**: Executes a standalone statement or declaration: `SmallVector<Value> finalResults;`.
  **L1317 CN**: 执行一条独立语句或声明：`SmallVector<Value> finalResults;`。
- **L1318 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1318 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1319 EN**: Initializes variable `loaded` from the right-hand expression.
  **L1319 CN**: 使用右侧表达式初始化变量 `loaded`。
- **L1320 EN**: Continues logic associated with callable symbol `create`.
  **L1320 CN**: 继续与可调用符号 `create` 相关的逻辑。

### Lines 1321-1344

````cpp
          rewriter, loc, sgDstType, op.getKind(), loaded, neutralFinalAcc,
          reductionDims);
      finalResults.push_back(vector::makeArithReduction(
          rewriter, loc, op.getKind(), finalReduce.getResult(),
          adaptor.getAcc()[i]));
    }
    rewriter.replaceOpWithMultiple(op, {finalResults});
    return success();
  }
};

// This pattern transforms vector.transpose ops to work at subgroup level.
struct WgToSgVectorTransposeOp
    : public OpConversionPattern<vector::TransposeOp> {
  using OpConversionPattern<vector::TransposeOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(vector::TransposeOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    VectorType resultType = op.getResultVectorType();

    ArrayRef<int64_t> wgShape = resultType.getShape();
    xegpu::DistributeLayoutAttr layout =
        xegpu::getTemporaryLayout(dyn_cast<OpResult>(op.getResult()));
````
- **L1321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, sgDstType, op.getKind(), loaded, neutralFinalAcc,`.
  **L1321 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, sgDstType, op.getKind(), loaded, neutralFinalAcc,`。
- **L1322 EN**: Executes a standalone statement or declaration: `reductionDims);`.
  **L1322 CN**: 执行一条独立语句或声明：`reductionDims);`。
- **L1323 EN**: Continues logic associated with callable symbol `push_back`.
  **L1323 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L1324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, op.getKind(), finalReduce.getResult(),`.
  **L1324 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, op.getKind(), finalReduce.getResult(),`。
- **L1325 EN**: Executes a call or declaration centered on `adaptor.getAcc`.
  **L1325 CN**: 执行以 `adaptor.getAcc` 为核心的调用或声明。
- **L1326 EN**: Closes the current lexical scope or compound statement.
  **L1326 CN**: 结束当前词法作用域或复合语句块。
- **L1327 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L1327 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L1328 EN**: Returns from the current function with `success()`.
  **L1328 CN**: 以 `success()` 从当前函数返回。
- **L1329 EN**: Closes the current lexical scope or compound statement.
  **L1329 CN**: 结束当前词法作用域或复合语句块。
- **L1330 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1330 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1331 EN**: Blank line separating nearby declarations or logic blocks.
  **L1331 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1332 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms vector.transpose ops to work at subgroup level.`.
  **L1332 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms vector.transpose ops to work at subgroup level.`。
- **L1333 EN**: Declares struct `WgToSgVectorTransposeOp`.
  **L1333 CN**: 声明 struct `WgToSgVectorTransposeOp`。
- **L1334 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<vector::TransposeOp> {`.
  **L1334 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<vector::TransposeOp> {`。
- **L1335 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<vector::TransposeOp>::OpConversionPattern;`.
  **L1335 CN**: 执行一条独立语句或声明：`using OpConversionPattern<vector::TransposeOp>::OpConversionPattern;`。
- **L1336 EN**: Blank line separating nearby declarations or logic blocks.
  **L1336 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1337 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L1337 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L1338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(vector::TransposeOp op, OneToNOpAdaptor adaptor,`.
  **L1338 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(vector::TransposeOp op, OneToNOpAdaptor adaptor,`。
- **L1339 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L1339 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L1340 EN**: Initializes variable `resultType` from the right-hand expression.
  **L1340 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L1341 EN**: Blank line separating nearby declarations or logic blocks.
  **L1341 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1342 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L1342 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L1343 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout =`.
  **L1343 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout =`。
- **L1344 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L1344 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。

### Lines 1345-1368

````cpp
    if (!layout || !layout.isForWorkgroup())
      return failure();
    // TODO-LayoutRefactor: handle the case using getTemporaryLayout
    xegpu::DistributeLayoutAttr sourceLayout =
        xegpu::getDistributeLayoutAttr(op.getVector());
    if (!sourceLayout || !sourceLayout.isForWorkgroup())
      return failure();

    SmallVector<int64_t> sourceSgLayout =
        sourceLayout.getEffectiveSgLayoutAsInt();
    SmallVector<int64_t> resultSgLayout = layout.getEffectiveSgLayoutAsInt();

    ArrayRef<int64_t> permutation = op.getPermutation();
    size_t permutationSize = permutation.size();
    if (sourceSgLayout.size() != permutationSize ||
        resultSgLayout.size() != permutationSize) {
      return rewriter.notifyMatchFailure(
          op, "Layouts and permutation must have the same rank");
    }

    // Check that sgLayout, sgData & order are properly transposed for source
    // and result
    if (!layout.isTransposeOf(sourceLayout, permutation,
                              xegpu::LayoutKind::Subgroup))
````
- **L1345 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1345 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1346 EN**: Returns from the current function with `failure()`.
  **L1346 CN**: 以 `failure()` 从当前函数返回。
- **L1347 EN**: Comment records a pending task or caution: `TODO-LayoutRefactor: handle the case using getTemporaryLayout`.
  **L1347 CN**: 注释记录了待办事项或注意点：`TODO-LayoutRefactor: handle the case using getTemporaryLayout`。
- **L1348 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr sourceLayout =`.
  **L1348 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr sourceLayout =`。
- **L1349 EN**: Executes a call or declaration centered on `xegpu::getDistributeLayoutAttr`.
  **L1349 CN**: 执行以 `xegpu::getDistributeLayoutAttr` 为核心的调用或声明。
- **L1350 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1350 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1351 EN**: Returns from the current function with `failure()`.
  **L1351 CN**: 以 `failure()` 从当前函数返回。
- **L1352 EN**: Blank line separating nearby declarations or logic blocks.
  **L1352 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1353 EN**: Continues the surrounding expression or declaration: `SmallVector<int64_t> sourceSgLayout =`.
  **L1353 CN**: 继续构造周围的表达式或声明：`SmallVector<int64_t> sourceSgLayout =`。
- **L1354 EN**: Executes a call or declaration centered on `sourceLayout.getEffectiveSgLayoutAsInt`.
  **L1354 CN**: 执行以 `sourceLayout.getEffectiveSgLayoutAsInt` 为核心的调用或声明。
- **L1355 EN**: Initializes variable `resultSgLayout` from the right-hand expression.
  **L1355 CN**: 使用右侧表达式初始化变量 `resultSgLayout`。
- **L1356 EN**: Blank line separating nearby declarations or logic blocks.
  **L1356 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1357 EN**: Initializes variable `permutation` from the right-hand expression.
  **L1357 CN**: 使用右侧表达式初始化变量 `permutation`。
- **L1358 EN**: Initializes variable `permutationSize` from the right-hand expression.
  **L1358 CN**: 使用右侧表达式初始化变量 `permutationSize`。
- **L1359 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1359 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1360 EN**: Starts a function, method, lambda, or structured scope: `resultSgLayout.size() != permutationSize) {`.
  **L1360 CN**: 开始一个函数、方法、lambda 或结构化作用域：`resultSgLayout.size() != permutationSize) {`。
- **L1361 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L1361 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L1362 EN**: Executes a standalone statement or declaration: `op, "Layouts and permutation must have the same rank");`.
  **L1362 CN**: 执行一条独立语句或声明：`op, "Layouts and permutation must have the same rank");`。
- **L1363 EN**: Closes the current lexical scope or compound statement.
  **L1363 CN**: 结束当前词法作用域或复合语句块。
- **L1364 EN**: Blank line separating nearby declarations or logic blocks.
  **L1364 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1365 EN**: Comment explains nearby logic, invariants, or intent: `Check that sgLayout, sgData & order are properly transposed for source`.
  **L1365 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check that sgLayout, sgData & order are properly transposed for source`。
- **L1366 EN**: Comment explains nearby logic, invariants, or intent: `and result`.
  **L1366 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and result`。
- **L1367 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1367 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1368 EN**: Continues the surrounding expression or declaration: `xegpu::LayoutKind::Subgroup))`.
  **L1368 CN**: 继续构造周围的表达式或声明：`xegpu::LayoutKind::Subgroup))`。

### Lines 1369-1392

````cpp
      return rewriter.notifyMatchFailure(
          op, "Result layout is not a valid transpose of source layout "
              "according to permutation");

    SmallVector<int64_t> sgShape = getSgShapeAndCount(wgShape, layout).first;
    VectorType newResultType =
        VectorType::get(sgShape, resultType.getElementType());

    SmallVector<Value> newTransposeOps;
    for (auto src : adaptor.getVector()) {
      auto newTranspose = vector::TransposeOp::create(
          rewriter, op.getLoc(), newResultType, src, permutation);
      newTransposeOps.push_back(newTranspose.getResult());
    }
    rewriter.replaceOpWithMultiple(op, {newTransposeOps});
    return success();
  }
};

// Distribute vector mask ops to work at subgroup level.
template <typename MaskOpType>
struct WgToSgVectorMaskOp : public OpConversionPattern<MaskOpType> {
  using OpConversionPattern<MaskOpType>::OpConversionPattern;

````
- **L1369 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L1369 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L1370 EN**: Continues the surrounding expression or declaration: `op, "Result layout is not a valid transpose of source layout "`.
  **L1370 CN**: 继续构造周围的表达式或声明：`op, "Result layout is not a valid transpose of source layout "`。
- **L1371 EN**: Executes a standalone statement or declaration: `"according to permutation");`.
  **L1371 CN**: 执行一条独立语句或声明：`"according to permutation");`。
- **L1372 EN**: Blank line separating nearby declarations or logic blocks.
  **L1372 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1373 EN**: Initializes variable `sgShape` from the right-hand expression.
  **L1373 CN**: 使用右侧表达式初始化变量 `sgShape`。
- **L1374 EN**: Continues the surrounding expression or declaration: `VectorType newResultType =`.
  **L1374 CN**: 继续构造周围的表达式或声明：`VectorType newResultType =`。
- **L1375 EN**: Executes a call or declaration centered on `VectorType::get`.
  **L1375 CN**: 执行以 `VectorType::get` 为核心的调用或声明。
- **L1376 EN**: Blank line separating nearby declarations or logic blocks.
  **L1376 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1377 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newTransposeOps;`.
  **L1377 CN**: 执行一条独立语句或声明：`SmallVector<Value> newTransposeOps;`。
- **L1378 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1378 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1379 EN**: Continues logic associated with callable symbol `create`.
  **L1379 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1380 EN**: Executes a call or declaration centered on `op.getLoc`.
  **L1380 CN**: 执行以 `op.getLoc` 为核心的调用或声明。
- **L1381 EN**: Executes a call or declaration centered on `newTransposeOps.push_back`.
  **L1381 CN**: 执行以 `newTransposeOps.push_back` 为核心的调用或声明。
- **L1382 EN**: Closes the current lexical scope or compound statement.
  **L1382 CN**: 结束当前词法作用域或复合语句块。
- **L1383 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L1383 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L1384 EN**: Returns from the current function with `success()`.
  **L1384 CN**: 以 `success()` 从当前函数返回。
- **L1385 EN**: Closes the current lexical scope or compound statement.
  **L1385 CN**: 结束当前词法作用域或复合语句块。
- **L1386 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1386 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1387 EN**: Blank line separating nearby declarations or logic blocks.
  **L1387 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1388 EN**: Comment explains nearby logic, invariants, or intent: `Distribute vector mask ops to work at subgroup level.`.
  **L1388 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Distribute vector mask ops to work at subgroup level.`。
- **L1389 EN**: Introduces template parameters or specialization context: `template <typename MaskOpType>`.
  **L1389 CN**: 为后续声明引入模板参数或特化上下文：`template <typename MaskOpType>`。
- **L1390 EN**: Declares struct `WgToSgVectorMaskOp`.
  **L1390 CN**: 声明 struct `WgToSgVectorMaskOp`。
- **L1391 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<MaskOpType>::OpConversionPattern;`.
  **L1391 CN**: 执行一条独立语句或声明：`using OpConversionPattern<MaskOpType>::OpConversionPattern;`。
- **L1392 EN**: Blank line separating nearby declarations or logic blocks.
  **L1392 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1393-1416

````cpp
  LogicalResult matchAndRewrite(
      MaskOpType op,
      typename OpConversionPattern<MaskOpType>::OneToNOpAdaptor adaptor,
      ConversionPatternRewriter &rewriter) const override {
    xegpu::DistributeLayoutAttr layout =
        xegpu::getTemporaryLayout(dyn_cast<OpResult>(op.getResult()));
    if (!layout || !layout.isForWorkgroup())
      return failure();

    Location loc = op.getLoc();
    VectorType type = op.getResult().getType();
    auto wgShape = type.getShape();

    SmallVector<Value> wgMaskDimSizes;
    if constexpr (std::is_same_v<MaskOpType, vector::ConstantMaskOp>) {
      for (int64_t maskSize : op.getMaskDimSizes()) {
        wgMaskDimSizes.push_back(
            arith::ConstantIndexOp::create(rewriter, loc, maskSize));
      }
    } else if constexpr (std::is_same_v<MaskOpType, vector::CreateMaskOp>) {
      wgMaskDimSizes = llvm::to_vector(op.getOperands());
    }

    Value sgId =
````
- **L1393 EN**: Continues logic associated with callable symbol `matchAndRewrite`.
  **L1393 CN**: 继续与可调用符号 `matchAndRewrite` 相关的逻辑。
- **L1394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskOpType op,`.
  **L1394 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskOpType op,`。
- **L1395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `typename OpConversionPattern<MaskOpType>::OneToNOpAdaptor adaptor,`.
  **L1395 CN**: 继续一个多行参数列表、初始化器或聚合项：`typename OpConversionPattern<MaskOpType>::OneToNOpAdaptor adaptor,`。
- **L1396 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L1396 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L1397 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout =`.
  **L1397 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout =`。
- **L1398 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L1398 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L1399 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1399 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1400 EN**: Returns from the current function with `failure()`.
  **L1400 CN**: 以 `failure()` 从当前函数返回。
- **L1401 EN**: Blank line separating nearby declarations or logic blocks.
  **L1401 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1402 EN**: Initializes variable `loc` from the right-hand expression.
  **L1402 CN**: 使用右侧表达式初始化变量 `loc`。
- **L1403 EN**: Initializes variable `type` from the right-hand expression.
  **L1403 CN**: 使用右侧表达式初始化变量 `type`。
- **L1404 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L1404 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L1405 EN**: Blank line separating nearby declarations or logic blocks.
  **L1405 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1406 EN**: Executes a standalone statement or declaration: `SmallVector<Value> wgMaskDimSizes;`.
  **L1406 CN**: 执行一条独立语句或声明：`SmallVector<Value> wgMaskDimSizes;`。
- **L1407 EN**: Continues logic associated with callable symbol `constexpr`.
  **L1407 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L1408 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1408 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1409 EN**: Continues logic associated with callable symbol `push_back`.
  **L1409 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L1410 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L1410 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L1411 EN**: Closes the current lexical scope or compound statement.
  **L1411 CN**: 结束当前词法作用域或复合语句块。
- **L1412 EN**: Starts a function, method, lambda, or structured scope: `} else if constexpr (std::is_same_v<MaskOpType, vector::CreateMaskOp>) {`.
  **L1412 CN**: 开始一个函数、方法、lambda 或结构化作用域：`} else if constexpr (std::is_same_v<MaskOpType, vector::CreateMaskOp>) {`。
- **L1413 EN**: Executes a call or declaration centered on `llvm::to_vector`.
  **L1413 CN**: 执行以 `llvm::to_vector` 为核心的调用或声明。
- **L1414 EN**: Closes the current lexical scope or compound statement.
  **L1414 CN**: 结束当前词法作用域或复合语句块。
- **L1415 EN**: Blank line separating nearby declarations or logic blocks.
  **L1415 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1416 EN**: Continues the surrounding expression or declaration: `Value sgId =`.
  **L1416 CN**: 继续构造周围的表达式或声明：`Value sgId =`。

### Lines 1417-1440

````cpp
        gpu::SubgroupIdOp::create(rewriter, loc, /*upper_bound=*/nullptr);
    auto sgOffsets =
        layout.computeDistributedCoords(rewriter, loc, sgId, wgShape);
    if (failed(sgOffsets))
      return failure();

    SmallVector<int64_t> sgShape = getSgShapeAndCount(wgShape, layout).first;
    VectorType resultType = VectorType::get(sgShape, type.getElementType());

    // In each dimension, each subgroup computes its local mask size as:
    // min(max(wgMaskDimSize[d] - offset[d], 0), sgDimSize[d])
    SmallVector<Value> newCreateMaskOps;
    for (auto offsetSet : *sgOffsets) {
      SmallVector<Value> maskOperands;

      for (auto [i, wgMaskDimSize] : llvm::enumerate(wgMaskDimSizes)) {
        Value dimSizeVal =
            arith::ConstantIndexOp::create(rewriter, loc, sgShape[i]);
        Value offset = offsetSet[i];
        Value adjustedMaskSize =
            arith::SubIOp::create(rewriter, loc, wgMaskDimSize, offset);
        Value zero = arith::ConstantIndexOp::create(rewriter, loc, 0);
        Value nonNegative =
            arith::MaxSIOp::create(rewriter, loc, adjustedMaskSize, zero);
````
- **L1417 EN**: Executes a call or declaration centered on `gpu::SubgroupIdOp::create`.
  **L1417 CN**: 执行以 `gpu::SubgroupIdOp::create` 为核心的调用或声明。
- **L1418 EN**: Continues the surrounding expression or declaration: `auto sgOffsets =`.
  **L1418 CN**: 继续构造周围的表达式或声明：`auto sgOffsets =`。
- **L1419 EN**: Executes a call or declaration centered on `layout.computeDistributedCoords`.
  **L1419 CN**: 执行以 `layout.computeDistributedCoords` 为核心的调用或声明。
- **L1420 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1420 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1421 EN**: Returns from the current function with `failure()`.
  **L1421 CN**: 以 `failure()` 从当前函数返回。
- **L1422 EN**: Blank line separating nearby declarations or logic blocks.
  **L1422 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1423 EN**: Initializes variable `sgShape` from the right-hand expression.
  **L1423 CN**: 使用右侧表达式初始化变量 `sgShape`。
- **L1424 EN**: Initializes variable `resultType` from the right-hand expression.
  **L1424 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L1425 EN**: Blank line separating nearby declarations or logic blocks.
  **L1425 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1426 EN**: Comment explains nearby logic, invariants, or intent: `In each dimension, each subgroup computes its local mask size as:`.
  **L1426 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`In each dimension, each subgroup computes its local mask size as:`。
- **L1427 EN**: Comment explains nearby logic, invariants, or intent: `min(max(wgMaskDimSize[d] - offset[d], 0), sgDimSize[d])`.
  **L1427 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`min(max(wgMaskDimSize[d] - offset[d], 0), sgDimSize[d])`。
- **L1428 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newCreateMaskOps;`.
  **L1428 CN**: 执行一条独立语句或声明：`SmallVector<Value> newCreateMaskOps;`。
- **L1429 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1429 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1430 EN**: Executes a standalone statement or declaration: `SmallVector<Value> maskOperands;`.
  **L1430 CN**: 执行一条独立语句或声明：`SmallVector<Value> maskOperands;`。
- **L1431 EN**: Blank line separating nearby declarations or logic blocks.
  **L1431 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1432 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1432 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1433 EN**: Continues the surrounding expression or declaration: `Value dimSizeVal =`.
  **L1433 CN**: 继续构造周围的表达式或声明：`Value dimSizeVal =`。
- **L1434 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L1434 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L1435 EN**: Initializes variable `offset` from the right-hand expression.
  **L1435 CN**: 使用右侧表达式初始化变量 `offset`。
- **L1436 EN**: Continues the surrounding expression or declaration: `Value adjustedMaskSize =`.
  **L1436 CN**: 继续构造周围的表达式或声明：`Value adjustedMaskSize =`。
- **L1437 EN**: Executes a call or declaration centered on `arith::SubIOp::create`.
  **L1437 CN**: 执行以 `arith::SubIOp::create` 为核心的调用或声明。
- **L1438 EN**: Initializes variable `zero` from the right-hand expression.
  **L1438 CN**: 使用右侧表达式初始化变量 `zero`。
- **L1439 EN**: Continues the surrounding expression or declaration: `Value nonNegative =`.
  **L1439 CN**: 继续构造周围的表达式或声明：`Value nonNegative =`。
- **L1440 EN**: Executes a call or declaration centered on `arith::MaxSIOp::create`.
  **L1440 CN**: 执行以 `arith::MaxSIOp::create` 为核心的调用或声明。

### Lines 1441-1464

````cpp
        Value sgMaskSize =
            arith::MinSIOp::create(rewriter, loc, nonNegative, dimSizeVal);
        maskOperands.push_back(sgMaskSize);
      }

      auto newCreateMaskOp =
          vector::CreateMaskOp::create(rewriter, loc, resultType, maskOperands);
      newCreateMaskOps.push_back(newCreateMaskOp.getResult());
    }

    rewriter.replaceOpWithMultiple(op, {newCreateMaskOps});
    return success();
  }
};

using WgToSgVectorConstantMaskOp = WgToSgVectorMaskOp<vector::ConstantMaskOp>;
using WgToSgVectorCreateMaskOp = WgToSgVectorMaskOp<vector::CreateMaskOp>;

// This pattern transforms vector.bitcast ops to work at subgroup level.
struct WgToSgVectorBitCastOp : public OpConversionPattern<vector::BitCastOp> {
  using OpConversionPattern<vector::BitCastOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(vector::BitCastOp op, OneToNOpAdaptor adaptor,
````
- **L1441 EN**: Continues the surrounding expression or declaration: `Value sgMaskSize =`.
  **L1441 CN**: 继续构造周围的表达式或声明：`Value sgMaskSize =`。
- **L1442 EN**: Executes a call or declaration centered on `arith::MinSIOp::create`.
  **L1442 CN**: 执行以 `arith::MinSIOp::create` 为核心的调用或声明。
- **L1443 EN**: Executes a call or declaration centered on `maskOperands.push_back`.
  **L1443 CN**: 执行以 `maskOperands.push_back` 为核心的调用或声明。
- **L1444 EN**: Closes the current lexical scope or compound statement.
  **L1444 CN**: 结束当前词法作用域或复合语句块。
- **L1445 EN**: Blank line separating nearby declarations or logic blocks.
  **L1445 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1446 EN**: Continues the surrounding expression or declaration: `auto newCreateMaskOp =`.
  **L1446 CN**: 继续构造周围的表达式或声明：`auto newCreateMaskOp =`。
- **L1447 EN**: Executes a call or declaration centered on `vector::CreateMaskOp::create`.
  **L1447 CN**: 执行以 `vector::CreateMaskOp::create` 为核心的调用或声明。
- **L1448 EN**: Executes a call or declaration centered on `newCreateMaskOps.push_back`.
  **L1448 CN**: 执行以 `newCreateMaskOps.push_back` 为核心的调用或声明。
- **L1449 EN**: Closes the current lexical scope or compound statement.
  **L1449 CN**: 结束当前词法作用域或复合语句块。
- **L1450 EN**: Blank line separating nearby declarations or logic blocks.
  **L1450 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1451 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L1451 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L1452 EN**: Returns from the current function with `success()`.
  **L1452 CN**: 以 `success()` 从当前函数返回。
- **L1453 EN**: Closes the current lexical scope or compound statement.
  **L1453 CN**: 结束当前词法作用域或复合语句块。
- **L1454 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1454 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1455 EN**: Blank line separating nearby declarations or logic blocks.
  **L1455 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1456 EN**: Defines alias `WgToSgVectorConstantMaskOp` to simplify later code.
  **L1456 CN**: 定义别名 `WgToSgVectorConstantMaskOp` 以简化后续代码。
- **L1457 EN**: Defines alias `WgToSgVectorCreateMaskOp` to simplify later code.
  **L1457 CN**: 定义别名 `WgToSgVectorCreateMaskOp` 以简化后续代码。
- **L1458 EN**: Blank line separating nearby declarations or logic blocks.
  **L1458 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1459 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms vector.bitcast ops to work at subgroup level.`.
  **L1459 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms vector.bitcast ops to work at subgroup level.`。
- **L1460 EN**: Declares struct `WgToSgVectorBitCastOp`.
  **L1460 CN**: 声明 struct `WgToSgVectorBitCastOp`。
- **L1461 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<vector::BitCastOp>::OpConversionPattern;`.
  **L1461 CN**: 执行一条独立语句或声明：`using OpConversionPattern<vector::BitCastOp>::OpConversionPattern;`。
- **L1462 EN**: Blank line separating nearby declarations or logic blocks.
  **L1462 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1463 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L1463 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L1464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(vector::BitCastOp op, OneToNOpAdaptor adaptor,`.
  **L1464 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(vector::BitCastOp op, OneToNOpAdaptor adaptor,`。

### Lines 1465-1488

````cpp
                  ConversionPatternRewriter &rewriter) const override {
    VectorType resultType = op.getResultVectorType();

    ArrayRef<int64_t> wgShape = resultType.getShape();
    xegpu::DistributeLayoutAttr layout =
        xegpu::getTemporaryLayout(dyn_cast<OpResult>(op.getResult()));
    if (!layout || !layout.isForWorkgroup())
      return failure();

    SmallVector<int64_t> sgShape = getSgShapeAndCount(wgShape, layout).first;
    VectorType newResultType =
        VectorType::get(sgShape, resultType.getElementType());

    SmallVector<Value> newBitCastOps;
    for (auto src : adaptor.getSource()) {
      auto newBitCast =
          vector::BitCastOp::create(rewriter, op.getLoc(), newResultType, src);
      newBitCastOps.push_back(newBitCast.getResult());
    }

    rewriter.replaceOpWithMultiple(op, {newBitCastOps});
    return success();
  }
};
````
- **L1465 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L1465 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L1466 EN**: Initializes variable `resultType` from the right-hand expression.
  **L1466 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L1467 EN**: Blank line separating nearby declarations or logic blocks.
  **L1467 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1468 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L1468 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L1469 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout =`.
  **L1469 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout =`。
- **L1470 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L1470 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L1471 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1471 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1472 EN**: Returns from the current function with `failure()`.
  **L1472 CN**: 以 `failure()` 从当前函数返回。
- **L1473 EN**: Blank line separating nearby declarations or logic blocks.
  **L1473 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1474 EN**: Initializes variable `sgShape` from the right-hand expression.
  **L1474 CN**: 使用右侧表达式初始化变量 `sgShape`。
- **L1475 EN**: Continues the surrounding expression or declaration: `VectorType newResultType =`.
  **L1475 CN**: 继续构造周围的表达式或声明：`VectorType newResultType =`。
- **L1476 EN**: Executes a call or declaration centered on `VectorType::get`.
  **L1476 CN**: 执行以 `VectorType::get` 为核心的调用或声明。
- **L1477 EN**: Blank line separating nearby declarations or logic blocks.
  **L1477 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1478 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newBitCastOps;`.
  **L1478 CN**: 执行一条独立语句或声明：`SmallVector<Value> newBitCastOps;`。
- **L1479 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1479 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1480 EN**: Continues the surrounding expression or declaration: `auto newBitCast =`.
  **L1480 CN**: 继续构造周围的表达式或声明：`auto newBitCast =`。
- **L1481 EN**: Executes a call or declaration centered on `vector::BitCastOp::create`.
  **L1481 CN**: 执行以 `vector::BitCastOp::create` 为核心的调用或声明。
- **L1482 EN**: Executes a call or declaration centered on `newBitCastOps.push_back`.
  **L1482 CN**: 执行以 `newBitCastOps.push_back` 为核心的调用或声明。
- **L1483 EN**: Closes the current lexical scope or compound statement.
  **L1483 CN**: 结束当前词法作用域或复合语句块。
- **L1484 EN**: Blank line separating nearby declarations or logic blocks.
  **L1484 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1485 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L1485 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L1486 EN**: Returns from the current function with `success()`.
  **L1486 CN**: 以 `success()` 从当前函数返回。
- **L1487 EN**: Closes the current lexical scope or compound statement.
  **L1487 CN**: 结束当前词法作用域或复合语句块。
- **L1488 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1488 CN**: 结束当前声明作用域，例如类、结构体或枚举。

### Lines 1489-1512

````cpp

// This pattern transforms vector.interleave ops to work at subgroup level.
struct WgToSgVectorInterleaveOp
    : public OpConversionPattern<vector::InterleaveOp> {
  using OpConversionPattern<vector::InterleaveOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(vector::InterleaveOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    VectorType resultType = op.getResultVectorType();

    ArrayRef<int64_t> wgShape = resultType.getShape();
    xegpu::DistributeLayoutAttr layout =
        xegpu::getTemporaryLayout(dyn_cast<OpResult>(op.getResult()));
    if (!layout || !layout.isForWorkgroup())
      return failure();

    SmallVector<int64_t> sgShape = getSgShapeAndCount(wgShape, layout).first;
    VectorType newResultType =
        VectorType::get(sgShape, resultType.getElementType());

    SmallVector<Value> newInterleaveOps;
    // Interleave operates pairwise: each lhs value is interleaved with
    // corresponding rhs value
````
- **L1489 EN**: Blank line separating nearby declarations or logic blocks.
  **L1489 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1490 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms vector.interleave ops to work at subgroup level.`.
  **L1490 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms vector.interleave ops to work at subgroup level.`。
- **L1491 EN**: Declares struct `WgToSgVectorInterleaveOp`.
  **L1491 CN**: 声明 struct `WgToSgVectorInterleaveOp`。
- **L1492 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<vector::InterleaveOp> {`.
  **L1492 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<vector::InterleaveOp> {`。
- **L1493 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<vector::InterleaveOp>::OpConversionPattern;`.
  **L1493 CN**: 执行一条独立语句或声明：`using OpConversionPattern<vector::InterleaveOp>::OpConversionPattern;`。
- **L1494 EN**: Blank line separating nearby declarations or logic blocks.
  **L1494 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1495 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L1495 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L1496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(vector::InterleaveOp op, OneToNOpAdaptor adaptor,`.
  **L1496 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(vector::InterleaveOp op, OneToNOpAdaptor adaptor,`。
- **L1497 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L1497 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L1498 EN**: Initializes variable `resultType` from the right-hand expression.
  **L1498 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L1499 EN**: Blank line separating nearby declarations or logic blocks.
  **L1499 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1500 EN**: Initializes variable `wgShape` from the right-hand expression.
  **L1500 CN**: 使用右侧表达式初始化变量 `wgShape`。
- **L1501 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout =`.
  **L1501 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout =`。
- **L1502 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L1502 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L1503 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1503 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1504 EN**: Returns from the current function with `failure()`.
  **L1504 CN**: 以 `failure()` 从当前函数返回。
- **L1505 EN**: Blank line separating nearby declarations or logic blocks.
  **L1505 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1506 EN**: Initializes variable `sgShape` from the right-hand expression.
  **L1506 CN**: 使用右侧表达式初始化变量 `sgShape`。
- **L1507 EN**: Continues the surrounding expression or declaration: `VectorType newResultType =`.
  **L1507 CN**: 继续构造周围的表达式或声明：`VectorType newResultType =`。
- **L1508 EN**: Executes a call or declaration centered on `VectorType::get`.
  **L1508 CN**: 执行以 `VectorType::get` 为核心的调用或声明。
- **L1509 EN**: Blank line separating nearby declarations or logic blocks.
  **L1509 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1510 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newInterleaveOps;`.
  **L1510 CN**: 执行一条独立语句或声明：`SmallVector<Value> newInterleaveOps;`。
- **L1511 EN**: Comment explains nearby logic, invariants, or intent: `Interleave operates pairwise: each lhs value is interleaved with`.
  **L1511 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Interleave operates pairwise: each lhs value is interleaved with`。
- **L1512 EN**: Comment explains nearby logic, invariants, or intent: `corresponding rhs value`.
  **L1512 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`corresponding rhs value`。

### Lines 1513-1536

````cpp
    for (auto [lhs, rhs] : llvm::zip(adaptor.getLhs(), adaptor.getRhs())) {
      auto newInterleave = vector::InterleaveOp::create(
          rewriter, op.getLoc(), newResultType, lhs, rhs);
      newInterleaveOps.push_back(newInterleave.getResult());
    }

    rewriter.replaceOpWithMultiple(op, {newInterleaveOps});
    return success();
  }
};

// This pattern transforms vector.deinterleave ops to work at subgroup level.
struct WgToSgVectorDeinterleaveOp
    : public OpConversionPattern<vector::DeinterleaveOp> {
  using OpConversionPattern<vector::DeinterleaveOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(vector::DeinterleaveOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    SmallVector<Value> newRes1Ops;
    SmallVector<Value> newRes2Ops;

    for (auto src : adaptor.getSource()) {
      auto newDeinterleave =
````
- **L1513 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1513 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1514 EN**: Continues logic associated with callable symbol `create`.
  **L1514 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1515 EN**: Executes a call or declaration centered on `op.getLoc`.
  **L1515 CN**: 执行以 `op.getLoc` 为核心的调用或声明。
- **L1516 EN**: Executes a call or declaration centered on `newInterleaveOps.push_back`.
  **L1516 CN**: 执行以 `newInterleaveOps.push_back` 为核心的调用或声明。
- **L1517 EN**: Closes the current lexical scope or compound statement.
  **L1517 CN**: 结束当前词法作用域或复合语句块。
- **L1518 EN**: Blank line separating nearby declarations or logic blocks.
  **L1518 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1519 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L1519 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L1520 EN**: Returns from the current function with `success()`.
  **L1520 CN**: 以 `success()` 从当前函数返回。
- **L1521 EN**: Closes the current lexical scope or compound statement.
  **L1521 CN**: 结束当前词法作用域或复合语句块。
- **L1522 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1522 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1523 EN**: Blank line separating nearby declarations or logic blocks.
  **L1523 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1524 EN**: Comment explains nearby logic, invariants, or intent: `This pattern transforms vector.deinterleave ops to work at subgroup level.`.
  **L1524 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pattern transforms vector.deinterleave ops to work at subgroup level.`。
- **L1525 EN**: Declares struct `WgToSgVectorDeinterleaveOp`.
  **L1525 CN**: 声明 struct `WgToSgVectorDeinterleaveOp`。
- **L1526 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<vector::DeinterleaveOp> {`.
  **L1526 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<vector::DeinterleaveOp> {`。
- **L1527 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<vector::DeinterleaveOp>::OpConversionPattern;`.
  **L1527 CN**: 执行一条独立语句或声明：`using OpConversionPattern<vector::DeinterleaveOp>::OpConversionPattern;`。
- **L1528 EN**: Blank line separating nearby declarations or logic blocks.
  **L1528 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1529 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L1529 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L1530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(vector::DeinterleaveOp op, OneToNOpAdaptor adaptor,`.
  **L1530 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(vector::DeinterleaveOp op, OneToNOpAdaptor adaptor,`。
- **L1531 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L1531 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L1532 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newRes1Ops;`.
  **L1532 CN**: 执行一条独立语句或声明：`SmallVector<Value> newRes1Ops;`。
- **L1533 EN**: Executes a standalone statement or declaration: `SmallVector<Value> newRes2Ops;`.
  **L1533 CN**: 执行一条独立语句或声明：`SmallVector<Value> newRes2Ops;`。
- **L1534 EN**: Blank line separating nearby declarations or logic blocks.
  **L1534 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1535 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1535 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1536 EN**: Continues the surrounding expression or declaration: `auto newDeinterleave =`.
  **L1536 CN**: 继续构造周围的表达式或声明：`auto newDeinterleave =`。

### Lines 1537-1560

````cpp
          vector::DeinterleaveOp::create(rewriter, op.getLoc(), src);
      newRes1Ops.push_back(newDeinterleave.getRes1());
      newRes2Ops.push_back(newDeinterleave.getRes2());
    }

    SmallVector<SmallVector<Value>> results = {newRes1Ops, newRes2Ops};
    rewriter.replaceOpWithMultiple(op, results);
    return success();
  }
};

} // namespace

namespace mlir {
namespace xegpu {
void populateXeGPUWgToSgDistributePatterns(RewritePatternSet &patterns) {
  patterns.add<WgToSgCreateNdOp, WgToSgLoadNdOp, WgToSgStoreNdOp, WgToSgDpasOp,
               WgToSgDpasMxOp, WgToSgPrefetchNdOp,
               UnrealizedConversionCastOpPattern, WgToSgElementwiseOp,
               WgToSgVectorBroadcastOp, WgToSgConvertLayoutOp,
               WgToSgArithConstantOp, WgToSgLoadGatherOp, WgToSgStoreScatterOp,
               WgToSgLoadMatrixOp, WgToSgStoreMatrixOp, WgToSgVectorStepOp,
               WgToSgVectorShapeCastOp, WgToSgMultiDimReductionOp,
               WgToSgVectorTransposeOp, WgToSgVectorConstantMaskOp,
````
- **L1537 EN**: Executes a call or declaration centered on `vector::DeinterleaveOp::create`.
  **L1537 CN**: 执行以 `vector::DeinterleaveOp::create` 为核心的调用或声明。
- **L1538 EN**: Executes a call or declaration centered on `newRes1Ops.push_back`.
  **L1538 CN**: 执行以 `newRes1Ops.push_back` 为核心的调用或声明。
- **L1539 EN**: Executes a call or declaration centered on `newRes2Ops.push_back`.
  **L1539 CN**: 执行以 `newRes2Ops.push_back` 为核心的调用或声明。
- **L1540 EN**: Closes the current lexical scope or compound statement.
  **L1540 CN**: 结束当前词法作用域或复合语句块。
- **L1541 EN**: Blank line separating nearby declarations or logic blocks.
  **L1541 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1542 EN**: Initializes variable `results` from the right-hand expression.
  **L1542 CN**: 使用右侧表达式初始化变量 `results`。
- **L1543 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L1543 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L1544 EN**: Returns from the current function with `success()`.
  **L1544 CN**: 以 `success()` 从当前函数返回。
- **L1545 EN**: Closes the current lexical scope or compound statement.
  **L1545 CN**: 结束当前词法作用域或复合语句块。
- **L1546 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1546 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1547 EN**: Blank line separating nearby declarations or logic blocks.
  **L1547 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1548 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L1548 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L1549 EN**: Blank line separating nearby declarations or logic blocks.
  **L1549 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1550 EN**: Opens namespace scope `mlir`.
  **L1550 CN**: 打开命名空间作用域 `mlir`。
- **L1551 EN**: Opens namespace scope `xegpu`.
  **L1551 CN**: 打开命名空间作用域 `xegpu`。
- **L1552 EN**: Starts a function, method, lambda, or structured scope: `void populateXeGPUWgToSgDistributePatterns(RewritePatternSet &patterns) {`.
  **L1552 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void populateXeGPUWgToSgDistributePatterns(RewritePatternSet &patterns) {`。
- **L1553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `patterns.add<WgToSgCreateNdOp, WgToSgLoadNdOp, WgToSgStoreNdOp, WgToSgDpasOp,`.
  **L1553 CN**: 继续一个多行参数列表、初始化器或聚合项：`patterns.add<WgToSgCreateNdOp, WgToSgLoadNdOp, WgToSgStoreNdOp, WgToSgDpasOp,`。
- **L1554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WgToSgDpasMxOp, WgToSgPrefetchNdOp,`.
  **L1554 CN**: 继续一个多行参数列表、初始化器或聚合项：`WgToSgDpasMxOp, WgToSgPrefetchNdOp,`。
- **L1555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnrealizedConversionCastOpPattern, WgToSgElementwiseOp,`.
  **L1555 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnrealizedConversionCastOpPattern, WgToSgElementwiseOp,`。
- **L1556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WgToSgVectorBroadcastOp, WgToSgConvertLayoutOp,`.
  **L1556 CN**: 继续一个多行参数列表、初始化器或聚合项：`WgToSgVectorBroadcastOp, WgToSgConvertLayoutOp,`。
- **L1557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WgToSgArithConstantOp, WgToSgLoadGatherOp, WgToSgStoreScatterOp,`.
  **L1557 CN**: 继续一个多行参数列表、初始化器或聚合项：`WgToSgArithConstantOp, WgToSgLoadGatherOp, WgToSgStoreScatterOp,`。
- **L1558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WgToSgLoadMatrixOp, WgToSgStoreMatrixOp, WgToSgVectorStepOp,`.
  **L1558 CN**: 继续一个多行参数列表、初始化器或聚合项：`WgToSgLoadMatrixOp, WgToSgStoreMatrixOp, WgToSgVectorStepOp,`。
- **L1559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WgToSgVectorShapeCastOp, WgToSgMultiDimReductionOp,`.
  **L1559 CN**: 继续一个多行参数列表、初始化器或聚合项：`WgToSgVectorShapeCastOp, WgToSgMultiDimReductionOp,`。
- **L1560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WgToSgVectorTransposeOp, WgToSgVectorConstantMaskOp,`.
  **L1560 CN**: 继续一个多行参数列表、初始化器或聚合项：`WgToSgVectorTransposeOp, WgToSgVectorConstantMaskOp,`。

### Lines 1561-1584

````cpp
               WgToSgVectorCreateMaskOp, WgToSgVectorBitCastOp,
               WgToSgVectorInterleaveOp, WgToSgVectorDeinterleaveOp>(
      patterns.getContext());
}
} // namespace xegpu
} // namespace mlir

namespace {
struct XeGPUWgToSgDistributePass
    : public xegpu::impl::XeGPUWgToSgDistributeBase<XeGPUWgToSgDistributePass> {
  void runOnOperation() override;
};
} // namespace

void XeGPUWgToSgDistributePass::runOnOperation() {

  Operation *op = getOperation();
  if (!xegpu::recoverTemporaryLayouts(op)) {
    signalPassFailure();
    return;
  }

  // Track existing UnrealizedConversionCastOps
  SmallVector<Operation *> existingCastOps;
````
- **L1561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WgToSgVectorCreateMaskOp, WgToSgVectorBitCastOp,`.
  **L1561 CN**: 继续一个多行参数列表、初始化器或聚合项：`WgToSgVectorCreateMaskOp, WgToSgVectorBitCastOp,`。
- **L1562 EN**: Continues logic associated with callable symbol `WgToSgVectorDeinterleaveOp>`.
  **L1562 CN**: 继续与可调用符号 `WgToSgVectorDeinterleaveOp>` 相关的逻辑。
- **L1563 EN**: Executes a call or declaration centered on `patterns.getContext`.
  **L1563 CN**: 执行以 `patterns.getContext` 为核心的调用或声明。
- **L1564 EN**: Closes the current lexical scope or compound statement.
  **L1564 CN**: 结束当前词法作用域或复合语句块。
- **L1565 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace xegpu`.
  **L1565 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace xegpu`。
- **L1566 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace mlir`.
  **L1566 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace mlir`。
- **L1567 EN**: Blank line separating nearby declarations or logic blocks.
  **L1567 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1568 EN**: Opens namespace scope ``.
  **L1568 CN**: 打开命名空间作用域 ``。
- **L1569 EN**: Declares struct `XeGPUWgToSgDistributePass`.
  **L1569 CN**: 声明 struct `XeGPUWgToSgDistributePass`。
- **L1570 EN**: Continues the surrounding expression or declaration: `: public xegpu::impl::XeGPUWgToSgDistributeBase<XeGPUWgToSgDistributePass> {`.
  **L1570 CN**: 继续构造周围的表达式或声明：`: public xegpu::impl::XeGPUWgToSgDistributeBase<XeGPUWgToSgDistributePass> {`。
- **L1571 EN**: Executes a call or declaration centered on `runOnOperation`.
  **L1571 CN**: 执行以 `runOnOperation` 为核心的调用或声明。
- **L1572 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1572 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1573 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L1573 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L1574 EN**: Blank line separating nearby declarations or logic blocks.
  **L1574 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1575 EN**: Starts a function, method, lambda, or structured scope: `void XeGPUWgToSgDistributePass::runOnOperation() {`.
  **L1575 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void XeGPUWgToSgDistributePass::runOnOperation() {`。
- **L1576 EN**: Blank line separating nearby declarations or logic blocks.
  **L1576 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1577 EN**: Executes a call or declaration centered on `getOperation`.
  **L1577 CN**: 执行以 `getOperation` 为核心的调用或声明。
- **L1578 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1578 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1579 EN**: Executes a call or declaration centered on `signalPassFailure`.
  **L1579 CN**: 执行以 `signalPassFailure` 为核心的调用或声明。
- **L1580 EN**: Returns from the current function with `void`.
  **L1580 CN**: 以 `void` 从当前函数返回。
- **L1581 EN**: Closes the current lexical scope or compound statement.
  **L1581 CN**: 结束当前词法作用域或复合语句块。
- **L1582 EN**: Blank line separating nearby declarations or logic blocks.
  **L1582 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1583 EN**: Comment explains nearby logic, invariants, or intent: `Track existing UnrealizedConversionCastOps`.
  **L1583 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Track existing UnrealizedConversionCastOps`。
- **L1584 EN**: Executes a standalone statement or declaration: `SmallVector<Operation *> existingCastOps;`.
  **L1584 CN**: 执行一条独立语句或声明：`SmallVector<Operation *> existingCastOps;`。

### Lines 1585-1608

````cpp
  getOperation()->walk([&](UnrealizedConversionCastOp castOp) {
    existingCastOps.push_back(castOp.getOperation());
  });

  {
    // Step 1: Apply SCFStructuralTypeConversions to SCF operations with
    // VectorType operands. This first converts such operands to
    // RankedTensorType, propagates the layout attribute into the encoding
    // attribute, and finally converts the RankedTensorType to VectorType based
    // on the encoding.

    TypeConverter converter;
    converter.addConversion([&](Type type) -> Type { return type; });
    converter.addConversion(
        [&](RankedTensorType type,
            SmallVectorImpl<Type> &result) -> std::optional<LogicalResult> {
          // Only convert RankedTensorTypes that carry an XeGPU layout encoding.
          // Plain tensors (e.g. tensor<?xi32>) have no XeGPU encoding and must
          // not be converted: VectorType does not support dynamic dimensions.
          auto encoding = dyn_cast_if_present<xegpu::DistributeLayoutAttr>(
              type.getEncoding());
          if (!encoding)
            return std::nullopt;

````
- **L1585 EN**: Starts a function, method, lambda, or structured scope: `getOperation()->walk([&](UnrealizedConversionCastOp castOp) {`.
  **L1585 CN**: 开始一个函数、方法、lambda 或结构化作用域：`getOperation()->walk([&](UnrealizedConversionCastOp castOp) {`。
- **L1586 EN**: Executes a call or declaration centered on `existingCastOps.push_back`.
  **L1586 CN**: 执行以 `existingCastOps.push_back` 为核心的调用或声明。
- **L1587 EN**: Executes a standalone statement or declaration: `});`.
  **L1587 CN**: 执行一条独立语句或声明：`});`。
- **L1588 EN**: Blank line separating nearby declarations or logic blocks.
  **L1588 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1589 EN**: Opens a new lexical scope or compound statement.
  **L1589 CN**: 打开一个新的词法作用域或复合语句块。
- **L1590 EN**: Comment explains nearby logic, invariants, or intent: `Step 1: Apply SCFStructuralTypeConversions to SCF operations with`.
  **L1590 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Step 1: Apply SCFStructuralTypeConversions to SCF operations with`。
- **L1591 EN**: Comment explains nearby logic, invariants, or intent: `VectorType operands. This first converts such operands to`.
  **L1591 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VectorType operands. This first converts such operands to`。
- **L1592 EN**: Comment explains nearby logic, invariants, or intent: `RankedTensorType, propagates the layout attribute into the encoding`.
  **L1592 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RankedTensorType, propagates the layout attribute into the encoding`。
- **L1593 EN**: Comment explains nearby logic, invariants, or intent: `attribute, and finally converts the RankedTensorType to VectorType based`.
  **L1593 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`attribute, and finally converts the RankedTensorType to VectorType based`。
- **L1594 EN**: Comment explains nearby logic, invariants, or intent: `on the encoding.`.
  **L1594 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`on the encoding.`。
- **L1595 EN**: Blank line separating nearby declarations or logic blocks.
  **L1595 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1596 EN**: Executes a standalone statement or declaration: `TypeConverter converter;`.
  **L1596 CN**: 执行一条独立语句或声明：`TypeConverter converter;`。
- **L1597 EN**: Executes a call or declaration centered on `converter.addConversion`.
  **L1597 CN**: 执行以 `converter.addConversion` 为核心的调用或声明。
- **L1598 EN**: Continues logic associated with callable symbol `addConversion`.
  **L1598 CN**: 继续与可调用符号 `addConversion` 相关的逻辑。
- **L1599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](RankedTensorType type,`.
  **L1599 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](RankedTensorType type,`。
- **L1600 EN**: Continues the surrounding expression or declaration: `SmallVectorImpl<Type> &result) -> std::optional<LogicalResult> {`.
  **L1600 CN**: 继续构造周围的表达式或声明：`SmallVectorImpl<Type> &result) -> std::optional<LogicalResult> {`。
- **L1601 EN**: Comment explains nearby logic, invariants, or intent: `Only convert RankedTensorTypes that carry an XeGPU layout encoding.`.
  **L1601 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only convert RankedTensorTypes that carry an XeGPU layout encoding.`。
- **L1602 EN**: Comment explains nearby logic, invariants, or intent: `Plain tensors (e.g. tensor<?xi32>) have no XeGPU encoding and must`.
  **L1602 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Plain tensors (e.g. tensor<?xi32>) have no XeGPU encoding and must`。
- **L1603 EN**: Comment explains nearby logic, invariants, or intent: `not be converted: VectorType does not support dynamic dimensions.`.
  **L1603 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`not be converted: VectorType does not support dynamic dimensions.`。
- **L1604 EN**: Continues logic associated with callable symbol `DistributeLayoutAttr>`.
  **L1604 CN**: 继续与可调用符号 `DistributeLayoutAttr>` 相关的逻辑。
- **L1605 EN**: Executes a call or declaration centered on `type.getEncoding`.
  **L1605 CN**: 执行以 `type.getEncoding` 为核心的调用或声明。
- **L1606 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1606 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1607 EN**: Returns from the current function with `std::nullopt`.
  **L1607 CN**: 以 `std::nullopt` 从当前函数返回。
- **L1608 EN**: Blank line separating nearby declarations or logic blocks.
  **L1608 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1609-1632

````cpp
          Type elemTy = type.getElementType();
          ArrayRef<int64_t> shape = type.getShape();

          int count;
          SmallVector<int64_t> subShape;
          std::tie(subShape, count) = getSgShapeAndCount(shape, encoding);

          auto newTy = VectorType::get(subShape, elemTy);
          result.append(count, newTy);
          return success();
        });

    xegpu::doSCFStructuralTypeConversionWithTensorType(getOperation(),
                                                       converter);
  }

  // Step 2: Perform workgroup to subgroup distribution for TensorDesc values,
  // as well as XeGPU, Arith, and Vector operations.
  MLIRContext *ctx = &getContext();
  RewritePatternSet patterns(ctx);
  ConversionTarget target(*ctx);
  TypeConverter converter;
  converter.addConversion([&](Type type) -> Type { return type; });
  converter.addConversion(
````
- **L1609 EN**: Initializes variable `elemTy` from the right-hand expression.
  **L1609 CN**: 使用右侧表达式初始化变量 `elemTy`。
- **L1610 EN**: Initializes variable `shape` from the right-hand expression.
  **L1610 CN**: 使用右侧表达式初始化变量 `shape`。
- **L1611 EN**: Blank line separating nearby declarations or logic blocks.
  **L1611 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1612 EN**: Executes a standalone statement or declaration: `int count;`.
  **L1612 CN**: 执行一条独立语句或声明：`int count;`。
- **L1613 EN**: Executes a standalone statement or declaration: `SmallVector<int64_t> subShape;`.
  **L1613 CN**: 执行一条独立语句或声明：`SmallVector<int64_t> subShape;`。
- **L1614 EN**: Executes a call or declaration centered on `std::tie`.
  **L1614 CN**: 执行以 `std::tie` 为核心的调用或声明。
- **L1615 EN**: Blank line separating nearby declarations or logic blocks.
  **L1615 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1616 EN**: Initializes variable `newTy` from the right-hand expression.
  **L1616 CN**: 使用右侧表达式初始化变量 `newTy`。
- **L1617 EN**: Executes a call or declaration centered on `result.append`.
  **L1617 CN**: 执行以 `result.append` 为核心的调用或声明。
- **L1618 EN**: Returns from the current function with `success()`.
  **L1618 CN**: 以 `success()` 从当前函数返回。
- **L1619 EN**: Executes a standalone statement or declaration: `});`.
  **L1619 CN**: 执行一条独立语句或声明：`});`。
- **L1620 EN**: Blank line separating nearby declarations or logic blocks.
  **L1620 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::doSCFStructuralTypeConversionWithTensorType(getOperation(),`.
  **L1621 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::doSCFStructuralTypeConversionWithTensorType(getOperation(),`。
- **L1622 EN**: Executes a standalone statement or declaration: `converter);`.
  **L1622 CN**: 执行一条独立语句或声明：`converter);`。
- **L1623 EN**: Closes the current lexical scope or compound statement.
  **L1623 CN**: 结束当前词法作用域或复合语句块。
- **L1624 EN**: Blank line separating nearby declarations or logic blocks.
  **L1624 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1625 EN**: Comment explains nearby logic, invariants, or intent: `Step 2: Perform workgroup to subgroup distribution for TensorDesc values,`.
  **L1625 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Step 2: Perform workgroup to subgroup distribution for TensorDesc values,`。
- **L1626 EN**: Comment explains nearby logic, invariants, or intent: `as well as XeGPU, Arith, and Vector operations.`.
  **L1626 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`as well as XeGPU, Arith, and Vector operations.`。
- **L1627 EN**: Executes a call or declaration centered on `&getContext`.
  **L1627 CN**: 执行以 `&getContext` 为核心的调用或声明。
- **L1628 EN**: Executes a call or declaration centered on `patterns`.
  **L1628 CN**: 执行以 `patterns` 为核心的调用或声明。
- **L1629 EN**: Executes a call or declaration centered on `target`.
  **L1629 CN**: 执行以 `target` 为核心的调用或声明。
- **L1630 EN**: Executes a standalone statement or declaration: `TypeConverter converter;`.
  **L1630 CN**: 执行一条独立语句或声明：`TypeConverter converter;`。
- **L1631 EN**: Executes a call or declaration centered on `converter.addConversion`.
  **L1631 CN**: 执行以 `converter.addConversion` 为核心的调用或声明。
- **L1632 EN**: Continues logic associated with callable symbol `addConversion`.
  **L1632 CN**: 继续与可调用符号 `addConversion` 相关的逻辑。

### Lines 1633-1656

````cpp
      [&](xegpu::TensorDescType type,
          SmallVectorImpl<Type> &result) -> std::optional<LogicalResult> {
        xegpu::DistributeLayoutAttr layout = type.getLayoutAttr();
        // Only convert WG-level tensor descs. SG-level or layout-less types
        // are already legal and should pass through unchanged.
        if (!layout || !layout.isForWorkgroup())
          return std::nullopt;

        Type elemTy = type.getElementType();
        ArrayRef<int64_t> shape = type.getShape();

        int count;
        SmallVector<int64_t> subShape;
        std::tie(subShape, count) = getSgShapeAndCount(shape, layout);

        layout = layout.dropSgLayoutAndData();

        auto newTy = xegpu::TensorDescType::get(
            type.getContext(), subShape, elemTy, type.getEncoding(), layout);
        result.append(count, newTy);
        return success();
      });

  auto getTensorDescType = [](Operation *op) -> xegpu::TensorDescType {
````
- **L1633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](xegpu::TensorDescType type,`.
  **L1633 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](xegpu::TensorDescType type,`。
- **L1634 EN**: Continues the surrounding expression or declaration: `SmallVectorImpl<Type> &result) -> std::optional<LogicalResult> {`.
  **L1634 CN**: 继续构造周围的表达式或声明：`SmallVectorImpl<Type> &result) -> std::optional<LogicalResult> {`。
- **L1635 EN**: Initializes variable `layout` from the right-hand expression.
  **L1635 CN**: 使用右侧表达式初始化变量 `layout`。
- **L1636 EN**: Comment explains nearby logic, invariants, or intent: `Only convert WG-level tensor descs. SG-level or layout-less types`.
  **L1636 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only convert WG-level tensor descs. SG-level or layout-less types`。
- **L1637 EN**: Comment explains nearby logic, invariants, or intent: `are already legal and should pass through unchanged.`.
  **L1637 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`are already legal and should pass through unchanged.`。
- **L1638 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1638 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1639 EN**: Returns from the current function with `std::nullopt`.
  **L1639 CN**: 以 `std::nullopt` 从当前函数返回。
- **L1640 EN**: Blank line separating nearby declarations or logic blocks.
  **L1640 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1641 EN**: Initializes variable `elemTy` from the right-hand expression.
  **L1641 CN**: 使用右侧表达式初始化变量 `elemTy`。
- **L1642 EN**: Initializes variable `shape` from the right-hand expression.
  **L1642 CN**: 使用右侧表达式初始化变量 `shape`。
- **L1643 EN**: Blank line separating nearby declarations or logic blocks.
  **L1643 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1644 EN**: Executes a standalone statement or declaration: `int count;`.
  **L1644 CN**: 执行一条独立语句或声明：`int count;`。
- **L1645 EN**: Executes a standalone statement or declaration: `SmallVector<int64_t> subShape;`.
  **L1645 CN**: 执行一条独立语句或声明：`SmallVector<int64_t> subShape;`。
- **L1646 EN**: Executes a call or declaration centered on `std::tie`.
  **L1646 CN**: 执行以 `std::tie` 为核心的调用或声明。
- **L1647 EN**: Blank line separating nearby declarations or logic blocks.
  **L1647 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1648 EN**: Executes a call or declaration centered on `layout.dropSgLayoutAndData`.
  **L1648 CN**: 执行以 `layout.dropSgLayoutAndData` 为核心的调用或声明。
- **L1649 EN**: Blank line separating nearby declarations or logic blocks.
  **L1649 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1650 EN**: Continues logic associated with callable symbol `get`.
  **L1650 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L1651 EN**: Executes a call or declaration centered on `type.getContext`.
  **L1651 CN**: 执行以 `type.getContext` 为核心的调用或声明。
- **L1652 EN**: Executes a call or declaration centered on `result.append`.
  **L1652 CN**: 执行以 `result.append` 为核心的调用或声明。
- **L1653 EN**: Returns from the current function with `success()`.
  **L1653 CN**: 以 `success()` 从当前函数返回。
- **L1654 EN**: Executes a standalone statement or declaration: `});`.
  **L1654 CN**: 执行一条独立语句或声明：`});`。
- **L1655 EN**: Blank line separating nearby declarations or logic blocks.
  **L1655 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1656 EN**: Starts a function, method, lambda, or structured scope: `auto getTensorDescType = [](Operation *op) -> xegpu::TensorDescType {`.
  **L1656 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto getTensorDescType = [](Operation *op) -> xegpu::TensorDescType {`。

### Lines 1657-1680

````cpp
    if (auto createOp = dyn_cast<xegpu::CreateNdDescOp>(op))
      return createOp.getType();
    if (auto loadOp = dyn_cast<xegpu::LoadNdOp>(op))
      return loadOp.getTensorDescType();
    if (auto storeOp = dyn_cast<xegpu::StoreNdOp>(op))
      return storeOp.getTensorDescType();
    if (auto prefetchOp = dyn_cast<xegpu::PrefetchNdOp>(op))
      return prefetchOp.getTensorDescType();
    return xegpu::TensorDescType();
  };

  auto isLegal = [&](xegpu::DistributeLayoutAttr layout) -> bool {
    return !layout || !layout.isForWorkgroup();
  };

  target.addDynamicallyLegalOp<xegpu::CreateNdDescOp, xegpu::LoadNdOp,
                               xegpu::StoreNdOp, xegpu::PrefetchNdOp>(
      [=](Operation *op) -> bool {
        auto tdescTy = getTensorDescType(op);
        auto layout =
            dyn_cast_if_present<xegpu::LayoutAttr>(tdescTy.getLayout());
        return isLegal(layout);
      });

````
- **L1657 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1657 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1658 EN**: Returns from the current function with `createOp.getType()`.
  **L1658 CN**: 以 `createOp.getType()` 从当前函数返回。
- **L1659 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1659 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1660 EN**: Returns from the current function with `loadOp.getTensorDescType()`.
  **L1660 CN**: 以 `loadOp.getTensorDescType()` 从当前函数返回。
- **L1661 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1661 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1662 EN**: Returns from the current function with `storeOp.getTensorDescType()`.
  **L1662 CN**: 以 `storeOp.getTensorDescType()` 从当前函数返回。
- **L1663 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1663 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1664 EN**: Returns from the current function with `prefetchOp.getTensorDescType()`.
  **L1664 CN**: 以 `prefetchOp.getTensorDescType()` 从当前函数返回。
- **L1665 EN**: Returns from the current function with `xegpu::TensorDescType()`.
  **L1665 CN**: 以 `xegpu::TensorDescType()` 从当前函数返回。
- **L1666 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1666 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1667 EN**: Blank line separating nearby declarations or logic blocks.
  **L1667 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1668 EN**: Starts a function, method, lambda, or structured scope: `auto isLegal = [&](xegpu::DistributeLayoutAttr layout) -> bool {`.
  **L1668 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto isLegal = [&](xegpu::DistributeLayoutAttr layout) -> bool {`。
- **L1669 EN**: Returns from the current function with `!layout || !layout.isForWorkgroup()`.
  **L1669 CN**: 以 `!layout || !layout.isForWorkgroup()` 从当前函数返回。
- **L1670 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1670 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1671 EN**: Blank line separating nearby declarations or logic blocks.
  **L1671 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `target.addDynamicallyLegalOp<xegpu::CreateNdDescOp, xegpu::LoadNdOp,`.
  **L1672 CN**: 继续一个多行参数列表、初始化器或聚合项：`target.addDynamicallyLegalOp<xegpu::CreateNdDescOp, xegpu::LoadNdOp,`。
- **L1673 EN**: Continues logic associated with callable symbol `PrefetchNdOp>`.
  **L1673 CN**: 继续与可调用符号 `PrefetchNdOp>` 相关的逻辑。
- **L1674 EN**: Starts a function, method, lambda, or structured scope: `[=](Operation *op) -> bool {`.
  **L1674 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[=](Operation *op) -> bool {`。
- **L1675 EN**: Initializes variable `tdescTy` from the right-hand expression.
  **L1675 CN**: 使用右侧表达式初始化变量 `tdescTy`。
- **L1676 EN**: Continues the surrounding expression or declaration: `auto layout =`.
  **L1676 CN**: 继续构造周围的表达式或声明：`auto layout =`。
- **L1677 EN**: Executes a call or declaration centered on `dyn_cast_if_present<xegpu::LayoutAttr>`.
  **L1677 CN**: 执行以 `dyn_cast_if_present<xegpu::LayoutAttr>` 为核心的调用或声明。
- **L1678 EN**: Returns from the current function with `isLegal(layout)`.
  **L1678 CN**: 以 `isLegal(layout)` 从当前函数返回。
- **L1679 EN**: Executes a standalone statement or declaration: `});`.
  **L1679 CN**: 执行一条独立语句或声明：`});`。
- **L1680 EN**: Blank line separating nearby declarations or logic blocks.
  **L1680 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1681-1704

````cpp
  target.addDynamicallyLegalOp<xegpu::DpasOp>([=](xegpu::DpasOp op) -> bool {
    auto layout = op.getLayoutCdAttr();
    return isLegal(layout);
  });

  target.addDynamicallyLegalOp<xegpu::DpasMxOp>(
      [=](xegpu::DpasMxOp op) -> bool {
        auto layout = op.getLayoutCdAttr();
        return isLegal(layout);
      });

  target.addDynamicallyLegalOp<xegpu::LoadMatrixOp>(
      [=](xegpu::LoadMatrixOp op) -> bool {
        return isLegal(op.getLayoutAttr());
      });

  target.addDynamicallyLegalOp<xegpu::StoreMatrixOp>(
      [=](xegpu::StoreMatrixOp op) -> bool {
        return isLegal(op.getLayoutAttr());
      });

  target.addDynamicallyLegalOp<arith::ConstantOp>(
      [=](arith::ConstantOp op) -> bool {
        auto vecType = dyn_cast<VectorType>(op.getType());
````
- **L1681 EN**: Starts a function, method, lambda, or structured scope: `target.addDynamicallyLegalOp<xegpu::DpasOp>([=](xegpu::DpasOp op) -> bool {`.
  **L1681 CN**: 开始一个函数、方法、lambda 或结构化作用域：`target.addDynamicallyLegalOp<xegpu::DpasOp>([=](xegpu::DpasOp op) -> bool {`。
- **L1682 EN**: Initializes variable `layout` from the right-hand expression.
  **L1682 CN**: 使用右侧表达式初始化变量 `layout`。
- **L1683 EN**: Returns from the current function with `isLegal(layout)`.
  **L1683 CN**: 以 `isLegal(layout)` 从当前函数返回。
- **L1684 EN**: Executes a standalone statement or declaration: `});`.
  **L1684 CN**: 执行一条独立语句或声明：`});`。
- **L1685 EN**: Blank line separating nearby declarations or logic blocks.
  **L1685 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1686 EN**: Continues logic associated with callable symbol `DpasMxOp>`.
  **L1686 CN**: 继续与可调用符号 `DpasMxOp>` 相关的逻辑。
- **L1687 EN**: Starts a function, method, lambda, or structured scope: `[=](xegpu::DpasMxOp op) -> bool {`.
  **L1687 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[=](xegpu::DpasMxOp op) -> bool {`。
- **L1688 EN**: Initializes variable `layout` from the right-hand expression.
  **L1688 CN**: 使用右侧表达式初始化变量 `layout`。
- **L1689 EN**: Returns from the current function with `isLegal(layout)`.
  **L1689 CN**: 以 `isLegal(layout)` 从当前函数返回。
- **L1690 EN**: Executes a standalone statement or declaration: `});`.
  **L1690 CN**: 执行一条独立语句或声明：`});`。
- **L1691 EN**: Blank line separating nearby declarations or logic blocks.
  **L1691 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1692 EN**: Continues logic associated with callable symbol `LoadMatrixOp>`.
  **L1692 CN**: 继续与可调用符号 `LoadMatrixOp>` 相关的逻辑。
- **L1693 EN**: Starts a function, method, lambda, or structured scope: `[=](xegpu::LoadMatrixOp op) -> bool {`.
  **L1693 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[=](xegpu::LoadMatrixOp op) -> bool {`。
- **L1694 EN**: Returns from the current function with `isLegal(op.getLayoutAttr())`.
  **L1694 CN**: 以 `isLegal(op.getLayoutAttr())` 从当前函数返回。
- **L1695 EN**: Executes a standalone statement or declaration: `});`.
  **L1695 CN**: 执行一条独立语句或声明：`});`。
- **L1696 EN**: Blank line separating nearby declarations or logic blocks.
  **L1696 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1697 EN**: Continues logic associated with callable symbol `StoreMatrixOp>`.
  **L1697 CN**: 继续与可调用符号 `StoreMatrixOp>` 相关的逻辑。
- **L1698 EN**: Starts a function, method, lambda, or structured scope: `[=](xegpu::StoreMatrixOp op) -> bool {`.
  **L1698 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[=](xegpu::StoreMatrixOp op) -> bool {`。
- **L1699 EN**: Returns from the current function with `isLegal(op.getLayoutAttr())`.
  **L1699 CN**: 以 `isLegal(op.getLayoutAttr())` 从当前函数返回。
- **L1700 EN**: Executes a standalone statement or declaration: `});`.
  **L1700 CN**: 执行一条独立语句或声明：`});`。
- **L1701 EN**: Blank line separating nearby declarations or logic blocks.
  **L1701 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1702 EN**: Continues logic associated with callable symbol `ConstantOp>`.
  **L1702 CN**: 继续与可调用符号 `ConstantOp>` 相关的逻辑。
- **L1703 EN**: Starts a function, method, lambda, or structured scope: `[=](arith::ConstantOp op) -> bool {`.
  **L1703 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[=](arith::ConstantOp op) -> bool {`。
- **L1704 EN**: Initializes variable `vecType` from the right-hand expression.
  **L1704 CN**: 使用右侧表达式初始化变量 `vecType`。

### Lines 1705-1728

````cpp
        if (!vecType)
          return true;

        auto layout =
            xegpu::getTemporaryLayout(dyn_cast<OpResult>(op.getResult()));
        return isLegal(layout);
      });

  target.addDynamicallyLegalOp<
      vector::ShapeCastOp, vector::StepOp, vector::TransposeOp,
      vector::BroadcastOp, vector::MultiDimReductionOp, vector::ConstantMaskOp,
      vector::CreateMaskOp, vector::BitCastOp, vector::InterleaveOp,
      vector::DeinterleaveOp>([=](Operation *op) -> bool {
    // Check for either a SliceAttr or LayoutAttr on the result.
    auto layout =
        xegpu::getTemporaryLayout(dyn_cast<OpResult>(op->getResult(0)));
    return isLegal(layout);
  });

  target.addDynamicallyLegalOp<xegpu::LoadGatherOp>(
      [=](xegpu::LoadGatherOp op) -> bool {
        auto layout = op.getLayoutAttr();
        return isLegal(layout);
      });
````
- **L1705 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1705 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1706 EN**: Returns from the current function with `true`.
  **L1706 CN**: 以 `true` 从当前函数返回。
- **L1707 EN**: Blank line separating nearby declarations or logic blocks.
  **L1707 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1708 EN**: Continues the surrounding expression or declaration: `auto layout =`.
  **L1708 CN**: 继续构造周围的表达式或声明：`auto layout =`。
- **L1709 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L1709 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L1710 EN**: Returns from the current function with `isLegal(layout)`.
  **L1710 CN**: 以 `isLegal(layout)` 从当前函数返回。
- **L1711 EN**: Executes a standalone statement or declaration: `});`.
  **L1711 CN**: 执行一条独立语句或声明：`});`。
- **L1712 EN**: Blank line separating nearby declarations or logic blocks.
  **L1712 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1713 EN**: Continues the surrounding expression or declaration: `target.addDynamicallyLegalOp<`.
  **L1713 CN**: 继续构造周围的表达式或声明：`target.addDynamicallyLegalOp<`。
- **L1714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::ShapeCastOp, vector::StepOp, vector::TransposeOp,`.
  **L1714 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::ShapeCastOp, vector::StepOp, vector::TransposeOp,`。
- **L1715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::BroadcastOp, vector::MultiDimReductionOp, vector::ConstantMaskOp,`.
  **L1715 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::BroadcastOp, vector::MultiDimReductionOp, vector::ConstantMaskOp,`。
- **L1716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `vector::CreateMaskOp, vector::BitCastOp, vector::InterleaveOp,`.
  **L1716 CN**: 继续一个多行参数列表、初始化器或聚合项：`vector::CreateMaskOp, vector::BitCastOp, vector::InterleaveOp,`。
- **L1717 EN**: Starts a function, method, lambda, or structured scope: `vector::DeinterleaveOp>([=](Operation *op) -> bool {`.
  **L1717 CN**: 开始一个函数、方法、lambda 或结构化作用域：`vector::DeinterleaveOp>([=](Operation *op) -> bool {`。
- **L1718 EN**: Comment explains nearby logic, invariants, or intent: `Check for either a SliceAttr or LayoutAttr on the result.`.
  **L1718 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check for either a SliceAttr or LayoutAttr on the result.`。
- **L1719 EN**: Continues the surrounding expression or declaration: `auto layout =`.
  **L1719 CN**: 继续构造周围的表达式或声明：`auto layout =`。
- **L1720 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L1720 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L1721 EN**: Returns from the current function with `isLegal(layout)`.
  **L1721 CN**: 以 `isLegal(layout)` 从当前函数返回。
- **L1722 EN**: Executes a standalone statement or declaration: `});`.
  **L1722 CN**: 执行一条独立语句或声明：`});`。
- **L1723 EN**: Blank line separating nearby declarations or logic blocks.
  **L1723 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1724 EN**: Continues logic associated with callable symbol `LoadGatherOp>`.
  **L1724 CN**: 继续与可调用符号 `LoadGatherOp>` 相关的逻辑。
- **L1725 EN**: Starts a function, method, lambda, or structured scope: `[=](xegpu::LoadGatherOp op) -> bool {`.
  **L1725 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[=](xegpu::LoadGatherOp op) -> bool {`。
- **L1726 EN**: Initializes variable `layout` from the right-hand expression.
  **L1726 CN**: 使用右侧表达式初始化变量 `layout`。
- **L1727 EN**: Returns from the current function with `isLegal(layout)`.
  **L1727 CN**: 以 `isLegal(layout)` 从当前函数返回。
- **L1728 EN**: Executes a standalone statement or declaration: `});`.
  **L1728 CN**: 执行一条独立语句或声明：`});`。

### Lines 1729-1752

````cpp

  target.addDynamicallyLegalOp<xegpu::StoreScatterOp>(
      [=](xegpu::StoreScatterOp op) -> bool {
        auto layout = op.getLayoutAttr();
        return isLegal(layout);
      });

  target.addDynamicallyLegalOp<xegpu::ConvertLayoutOp>(
      [=](xegpu::ConvertLayoutOp op) -> bool {
        return isLegal(op.getInputLayout()) && isLegal(op.getTargetLayout());
      });

  target.addDynamicallyLegalDialect<math::MathDialect, arith::ArithDialect>(
      [=](Operation *op) -> std::optional<bool> {
        // Only handle elementwise mappable ops
        if (!OpTrait::hasElementwiseMappableTraits(op))
          return true;

        VectorType resultType =
            dyn_cast<VectorType>(op->getResult(0).getType());
        if (!resultType)
          return true;

        // Check if all operands are vectors of the same shape
````
- **L1729 EN**: Blank line separating nearby declarations or logic blocks.
  **L1729 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1730 EN**: Continues logic associated with callable symbol `StoreScatterOp>`.
  **L1730 CN**: 继续与可调用符号 `StoreScatterOp>` 相关的逻辑。
- **L1731 EN**: Starts a function, method, lambda, or structured scope: `[=](xegpu::StoreScatterOp op) -> bool {`.
  **L1731 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[=](xegpu::StoreScatterOp op) -> bool {`。
- **L1732 EN**: Initializes variable `layout` from the right-hand expression.
  **L1732 CN**: 使用右侧表达式初始化变量 `layout`。
- **L1733 EN**: Returns from the current function with `isLegal(layout)`.
  **L1733 CN**: 以 `isLegal(layout)` 从当前函数返回。
- **L1734 EN**: Executes a standalone statement or declaration: `});`.
  **L1734 CN**: 执行一条独立语句或声明：`});`。
- **L1735 EN**: Blank line separating nearby declarations or logic blocks.
  **L1735 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1736 EN**: Continues logic associated with callable symbol `ConvertLayoutOp>`.
  **L1736 CN**: 继续与可调用符号 `ConvertLayoutOp>` 相关的逻辑。
- **L1737 EN**: Starts a function, method, lambda, or structured scope: `[=](xegpu::ConvertLayoutOp op) -> bool {`.
  **L1737 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[=](xegpu::ConvertLayoutOp op) -> bool {`。
- **L1738 EN**: Returns from the current function with `isLegal(op.getInputLayout()) && isLegal(op.getTargetLayout())`.
  **L1738 CN**: 以 `isLegal(op.getInputLayout()) && isLegal(op.getTargetLayout())` 从当前函数返回。
- **L1739 EN**: Executes a standalone statement or declaration: `});`.
  **L1739 CN**: 执行一条独立语句或声明：`});`。
- **L1740 EN**: Blank line separating nearby declarations or logic blocks.
  **L1740 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1741 EN**: Continues logic associated with callable symbol `ArithDialect>`.
  **L1741 CN**: 继续与可调用符号 `ArithDialect>` 相关的逻辑。
- **L1742 EN**: Starts a function, method, lambda, or structured scope: `[=](Operation *op) -> std::optional<bool> {`.
  **L1742 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[=](Operation *op) -> std::optional<bool> {`。
- **L1743 EN**: Comment explains nearby logic, invariants, or intent: `Only handle elementwise mappable ops`.
  **L1743 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only handle elementwise mappable ops`。
- **L1744 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1744 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1745 EN**: Returns from the current function with `true`.
  **L1745 CN**: 以 `true` 从当前函数返回。
- **L1746 EN**: Blank line separating nearby declarations or logic blocks.
  **L1746 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1747 EN**: Continues the surrounding expression or declaration: `VectorType resultType =`.
  **L1747 CN**: 继续构造周围的表达式或声明：`VectorType resultType =`。
- **L1748 EN**: Executes a call or declaration centered on `dyn_cast<VectorType>`.
  **L1748 CN**: 执行以 `dyn_cast<VectorType>` 为核心的调用或声明。
- **L1749 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1749 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1750 EN**: Returns from the current function with `true`.
  **L1750 CN**: 以 `true` 从当前函数返回。
- **L1751 EN**: Blank line separating nearby declarations or logic blocks.
  **L1751 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1752 EN**: Comment explains nearby logic, invariants, or intent: `Check if all operands are vectors of the same shape`.
  **L1752 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check if all operands are vectors of the same shape`。

### Lines 1753-1776

````cpp
        // TODO: Support other types.
        for (Value operand : op->getOperands()) {
          VectorType operandType = dyn_cast<VectorType>(operand.getType());
          if (!operandType || operandType.getShape() != resultType.getShape()) {
            return true;
          }
        }

        xegpu::DistributeLayoutAttr layout =
            xegpu::getTemporaryLayout(op->getResult(0));
        return isLegal(layout);
      });

  target.addDynamicallyLegalOp<UnrealizedConversionCastOp>(
      [=](UnrealizedConversionCastOp op) {
        return llvm::is_contained(existingCastOps, op.getOperation());
      });

  target.markUnknownOpDynamicallyLegal([](Operation *) { return true; });

  scf::populateSCFStructuralTypeConversionsAndLegality(converter, patterns,
                                                       target);
  xegpu::populateXeGPUWgToSgDistributePatterns(patterns);
  if (failed(
````
- **L1753 EN**: Comment records a pending task or caution: `TODO: Support other types.`.
  **L1753 CN**: 注释记录了待办事项或注意点：`TODO: Support other types.`。
- **L1754 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1754 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1755 EN**: Initializes variable `operandType` from the right-hand expression.
  **L1755 CN**: 使用右侧表达式初始化变量 `operandType`。
- **L1756 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1756 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1757 EN**: Returns from the current function with `true`.
  **L1757 CN**: 以 `true` 从当前函数返回。
- **L1758 EN**: Closes the current lexical scope or compound statement.
  **L1758 CN**: 结束当前词法作用域或复合语句块。
- **L1759 EN**: Closes the current lexical scope or compound statement.
  **L1759 CN**: 结束当前词法作用域或复合语句块。
- **L1760 EN**: Blank line separating nearby declarations or logic blocks.
  **L1760 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1761 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout =`.
  **L1761 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout =`。
- **L1762 EN**: Executes a call or declaration centered on `xegpu::getTemporaryLayout`.
  **L1762 CN**: 执行以 `xegpu::getTemporaryLayout` 为核心的调用或声明。
- **L1763 EN**: Returns from the current function with `isLegal(layout)`.
  **L1763 CN**: 以 `isLegal(layout)` 从当前函数返回。
- **L1764 EN**: Executes a standalone statement or declaration: `});`.
  **L1764 CN**: 执行一条独立语句或声明：`});`。
- **L1765 EN**: Blank line separating nearby declarations or logic blocks.
  **L1765 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1766 EN**: Continues logic associated with callable symbol `addDynamicallyLegalOp<UnrealizedConversionCastOp>`.
  **L1766 CN**: 继续与可调用符号 `addDynamicallyLegalOp<UnrealizedConversionCastOp>` 相关的逻辑。
- **L1767 EN**: Starts a function, method, lambda, or structured scope: `[=](UnrealizedConversionCastOp op) {`.
  **L1767 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[=](UnrealizedConversionCastOp op) {`。
- **L1768 EN**: Returns from the current function with `llvm::is_contained(existingCastOps, op.getOperation())`.
  **L1768 CN**: 以 `llvm::is_contained(existingCastOps, op.getOperation())` 从当前函数返回。
- **L1769 EN**: Executes a standalone statement or declaration: `});`.
  **L1769 CN**: 执行一条独立语句或声明：`});`。
- **L1770 EN**: Blank line separating nearby declarations or logic blocks.
  **L1770 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1771 EN**: Executes a call or declaration centered on `target.markUnknownOpDynamicallyLegal`.
  **L1771 CN**: 执行以 `target.markUnknownOpDynamicallyLegal` 为核心的调用或声明。
- **L1772 EN**: Blank line separating nearby declarations or logic blocks.
  **L1772 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `scf::populateSCFStructuralTypeConversionsAndLegality(converter, patterns,`.
  **L1773 CN**: 继续一个多行参数列表、初始化器或聚合项：`scf::populateSCFStructuralTypeConversionsAndLegality(converter, patterns,`。
- **L1774 EN**: Executes a standalone statement or declaration: `target);`.
  **L1774 CN**: 执行一条独立语句或声明：`target);`。
- **L1775 EN**: Executes a call or declaration centered on `xegpu::populateXeGPUWgToSgDistributePatterns`.
  **L1775 CN**: 执行以 `xegpu::populateXeGPUWgToSgDistributePatterns` 为核心的调用或声明。
- **L1776 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1776 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1777-1781

````cpp
          applyPartialConversion(getOperation(), target, std::move(patterns))))
    return signalPassFailure();

  xegpu::removeTemporaryLayoutAttrs(getOperation());
}
````
- **L1777 EN**: Continues logic associated with callable symbol `applyPartialConversion`.
  **L1777 CN**: 继续与可调用符号 `applyPartialConversion` 相关的逻辑。
- **L1778 EN**: Returns from the current function with `signalPassFailure()`.
  **L1778 CN**: 以 `signalPassFailure()` 从当前函数返回。
- **L1779 EN**: Blank line separating nearby declarations or logic blocks.
  **L1779 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1780 EN**: Executes a call or declaration centered on `xegpu::removeTemporaryLayoutAttrs`.
  **L1780 CN**: 执行以 `xegpu::removeTemporaryLayoutAttrs` 为核心的调用或声明。
- **L1781 EN**: Closes the current lexical scope or compound statement.
  **L1781 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **XeGPU target integration / XeGPU 目标集成**
- **Rewrite-driven lowering / 基于重写的 lowering**
- **Dialect definition and registration / 方言定义与注册**
- **Operation semantics and verification / 操作语义与验证**
- **Pattern collection management / 重写模式集合管理**
- **Dialect conversion patterns / 方言转换模式**
- **Legality modeling for conversion / 转换合法性建模**
- **Type conversion rules / 类型转换规则**
- **Failure-aware result propagation / 带失败语义的结果传播**
- **IR mutation through rewriter helpers / 通过重写器辅助工具进行 IR 变换**

## Dependencies / 依赖关系

- `mlir/Dialect/XeGPU/Transforms/Passes.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Affine/Utils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Arith/IR/Arith.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Arith/Utils/Utils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/GPU/IR/GPUDialect.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Index/IR/IndexDialect.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Index/IR/IndexOps.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Math/IR/Math.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/MemRef/IR/MemRef.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/SCF/Transforms/Patterns.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Utils/IndexingUtils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/IR/XeGPU.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Transforms/Transforms.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Utils/XeGPUUtils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Transforms/DialectConversion.h`: Provides generic transformation utilities and canonicalization helpers. / 提供通用变换工具与规范化辅助逻辑。
- `optional`: Provides supporting declarations used by the current translation unit. / 提供当前编译单元使用的辅助声明。
- `mlir/Dialect/XeGPU/Transforms/Passes.h.inc`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
