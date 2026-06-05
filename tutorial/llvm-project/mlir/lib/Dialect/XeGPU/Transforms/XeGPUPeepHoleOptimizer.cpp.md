# XeGPUPeepHoleOptimizer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/XeGPU/Transforms/XeGPUPeepHoleOptimizer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements XeGPU lowering, scheduling, and rewrite passes.
- **Purpose (CN)**: 实现 XeGPU lowering、调度与重写 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- XeGPUPeepHoleOptimizer.cpp - XeGPU optimize block loads -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/SCF/Transforms/Patterns.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/Dialect/XeGPU/IR/XeGPU.h"
#include "mlir/Dialect/XeGPU/Transforms/Passes.h"
#include "mlir/Dialect/XeGPU/Transforms/Transforms.h"
#include "mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h"
#include "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h"
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
- **L9 EN**: Includes "mlir/Dialect/Arith/IR/Arith.h" to access dialect-specific IR, transforms, or shared utilities.
  **L9 CN**: 引入 "mlir/Dialect/Arith/IR/Arith.h" 以使用方言专用 IR、变换或共享工具。
- **L10 EN**: Includes "mlir/Dialect/GPU/IR/GPUDialect.h" to access dialect-specific IR, transforms, or shared utilities.
  **L10 CN**: 引入 "mlir/Dialect/GPU/IR/GPUDialect.h" 以使用方言专用 IR、变换或共享工具。
- **L11 EN**: Includes "mlir/Dialect/MemRef/IR/MemRef.h" to access dialect-specific IR, transforms, or shared utilities.
  **L11 CN**: 引入 "mlir/Dialect/MemRef/IR/MemRef.h" 以使用方言专用 IR、变换或共享工具。
- **L12 EN**: Includes "mlir/Dialect/SCF/Transforms/Patterns.h" to access dialect-specific IR, transforms, or shared utilities.
  **L12 CN**: 引入 "mlir/Dialect/SCF/Transforms/Patterns.h" 以使用方言专用 IR、变换或共享工具。
- **L13 EN**: Includes "mlir/Dialect/Utils/IndexingUtils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L13 CN**: 引入 "mlir/Dialect/Utils/IndexingUtils.h" 以使用方言专用 IR、变换或共享工具。
- **L14 EN**: Includes "mlir/Dialect/Utils/StaticValueUtils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L14 CN**: 引入 "mlir/Dialect/Utils/StaticValueUtils.h" 以使用方言专用 IR、变换或共享工具。
- **L15 EN**: Includes "mlir/Dialect/Vector/IR/VectorOps.h" to access dialect-specific IR, transforms, or shared utilities.
  **L15 CN**: 引入 "mlir/Dialect/Vector/IR/VectorOps.h" 以使用方言专用 IR、变换或共享工具。
- **L16 EN**: Includes "mlir/Dialect/XeGPU/IR/XeGPU.h" to access dialect-specific IR, transforms, or shared utilities.
  **L16 CN**: 引入 "mlir/Dialect/XeGPU/IR/XeGPU.h" 以使用方言专用 IR、变换或共享工具。
- **L17 EN**: Includes "mlir/Dialect/XeGPU/Transforms/Passes.h" to access dialect-specific IR, transforms, or shared utilities.
  **L17 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/Passes.h" 以使用方言专用 IR、变换或共享工具。
- **L18 EN**: Includes "mlir/Dialect/XeGPU/Transforms/Transforms.h" to access dialect-specific IR, transforms, or shared utilities.
  **L18 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/Transforms.h" 以使用方言专用 IR、变换或共享工具。
- **L19 EN**: Includes "mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h" to access dialect-specific IR, transforms, or shared utilities.
  **L19 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h" 以使用方言专用 IR、变换或共享工具。
- **L20 EN**: Includes "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L20 CN**: 引入 "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h" 以使用方言专用 IR、变换或共享工具。

### Lines 21-40

````cpp
#include "mlir/Dialect/XeGPU/uArch/IntelGpuXe2.h"
#include "mlir/Dialect/XeGPU/uArch/uArchBase.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/OpDefinition.h"
#include "mlir/IR/Types.h"
#include "mlir/IR/Value.h"
#include "mlir/Transforms/DialectConversion.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include <optional>

namespace mlir {
namespace xegpu {
#define GEN_PASS_DEF_XEGPUPEEPHOLEOPTIMIZER
#include "mlir/Dialect/XeGPU/Transforms/Passes.h.inc"
} // namespace xegpu
} // namespace mlir

#define DEBUG_TYPE "xegpu-optimize-peephole"
````
- **L21 EN**: Includes "mlir/Dialect/XeGPU/uArch/IntelGpuXe2.h" to access dialect-specific IR, transforms, or shared utilities.
  **L21 CN**: 引入 "mlir/Dialect/XeGPU/uArch/IntelGpuXe2.h" 以使用方言专用 IR、变换或共享工具。
- **L22 EN**: Includes "mlir/Dialect/XeGPU/uArch/uArchBase.h" to access dialect-specific IR, transforms, or shared utilities.
  **L22 CN**: 引入 "mlir/Dialect/XeGPU/uArch/uArchBase.h" 以使用方言专用 IR、变换或共享工具。
- **L23 EN**: Includes "mlir/IR/BuiltinTypes.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L23 CN**: 引入 "mlir/IR/BuiltinTypes.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L24 EN**: Includes "mlir/IR/OpDefinition.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L24 CN**: 引入 "mlir/IR/OpDefinition.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L25 EN**: Includes "mlir/IR/Types.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L25 CN**: 引入 "mlir/IR/Types.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L26 EN**: Includes "mlir/IR/Value.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L26 CN**: 引入 "mlir/IR/Value.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L27 EN**: Includes "mlir/Transforms/DialectConversion.h" to access generic transformation utilities and canonicalization helpers.
  **L27 CN**: 引入 "mlir/Transforms/DialectConversion.h" 以使用通用变换工具与规范化辅助逻辑。
- **L28 EN**: Includes "mlir/Transforms/GreedyPatternRewriteDriver.h" to access generic transformation utilities and canonicalization helpers.
  **L28 CN**: 引入 "mlir/Transforms/GreedyPatternRewriteDriver.h" 以使用通用变换工具与规范化辅助逻辑。
- **L29 EN**: Includes "llvm/ADT/STLExtras.h" to access LLVM ADT containers and low-level utility types.
  **L29 CN**: 引入 "llvm/ADT/STLExtras.h" 以使用LLVM ADT 容器与底层工具类型。
- **L30 EN**: Includes "llvm/ADT/SmallVector.h" to access LLVM ADT containers and low-level utility types.
  **L30 CN**: 引入 "llvm/ADT/SmallVector.h" 以使用LLVM ADT 容器与底层工具类型。
- **L31 EN**: Includes <optional> to access supporting declarations used by the current translation unit.
  **L31 CN**: 引入 <optional> 以使用当前编译单元使用的辅助声明。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L33 EN**: Opens namespace scope `mlir`.
  **L33 CN**: 打开命名空间作用域 `mlir`。
- **L34 EN**: Opens namespace scope `xegpu`.
  **L34 CN**: 打开命名空间作用域 `xegpu`。
- **L35 EN**: Defines macro `GEN_PASS_DEF_XEGPUPEEPHOLEOPTIMIZER` for generated declarations, local shorthand, or conditional logic.
  **L35 CN**: 定义宏 `GEN_PASS_DEF_XEGPUPEEPHOLEOPTIMIZER`，供生成式声明、本地简写或条件逻辑使用。
- **L36 EN**: Includes "mlir/Dialect/XeGPU/Transforms/Passes.h.inc" to access dialect-specific IR, transforms, or shared utilities.
  **L36 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/Passes.h.inc" 以使用方言专用 IR、变换或共享工具。
- **L37 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace xegpu`.
  **L37 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace xegpu`。
- **L38 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace mlir`.
  **L38 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace mlir`。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L40 EN**: Defines macro `DEBUG_TYPE` for generated declarations, local shorthand, or conditional logic.
  **L40 CN**: 定义宏 `DEBUG_TYPE`，供生成式声明、本地简写或条件逻辑使用。

### Lines 41-60

````cpp
#define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")

using namespace mlir;

namespace {

/// Get the 2D lane data from a tensor desc type if it exists.
static std::optional<SmallVector<int64_t>>
getMaybeLaneData(xegpu::TensorDescType tdescType) {
  auto layout = tdescType.getLayoutAttr();
  if (!layout)
    return std::nullopt;
  auto laneData = layout.getEffectiveLaneDataAsInt();
  if (laneData.size() != 2)
    return std::nullopt;
  return laneData;
}

/// Get the 2D lane layout from a tensor desc type if it exists.
static std::optional<SmallVector<int64_t>>
````
- **L41 EN**: Defines macro `DBGS()` for generated declarations, local shorthand, or conditional logic.
  **L41 CN**: 定义宏 `DBGS()`，供生成式声明、本地简写或条件逻辑使用。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Brings namespace `mlir` into local scope.
  **L43 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L45 EN**: Opens namespace scope ``.
  **L45 CN**: 打开命名空间作用域 ``。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L47 EN**: Comment explains nearby logic, invariants, or intent: `Get the 2D lane data from a tensor desc type if it exists.`.
  **L47 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get the 2D lane data from a tensor desc type if it exists.`。
- **L48 EN**: Continues the surrounding expression or declaration: `static std::optional<SmallVector<int64_t>>`.
  **L48 CN**: 继续构造周围的表达式或声明：`static std::optional<SmallVector<int64_t>>`。
- **L49 EN**: Starts a function, method, lambda, or structured scope: `getMaybeLaneData(xegpu::TensorDescType tdescType) {`.
  **L49 CN**: 开始一个函数、方法、lambda 或结构化作用域：`getMaybeLaneData(xegpu::TensorDescType tdescType) {`。
- **L50 EN**: Initializes variable `layout` from the right-hand expression.
  **L50 CN**: 使用右侧表达式初始化变量 `layout`。
- **L51 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L51 CN**: 开始 `if` 控制流语句并计算其条件。
- **L52 EN**: Returns from the current function with `std::nullopt`.
  **L52 CN**: 以 `std::nullopt` 从当前函数返回。
- **L53 EN**: Initializes variable `laneData` from the right-hand expression.
  **L53 CN**: 使用右侧表达式初始化变量 `laneData`。
- **L54 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L54 CN**: 开始 `if` 控制流语句并计算其条件。
- **L55 EN**: Returns from the current function with `std::nullopt`.
  **L55 CN**: 以 `std::nullopt` 从当前函数返回。
- **L56 EN**: Returns from the current function with `laneData`.
  **L56 CN**: 以 `laneData` 从当前函数返回。
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L59 EN**: Comment explains nearby logic, invariants, or intent: `Get the 2D lane layout from a tensor desc type if it exists.`.
  **L59 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get the 2D lane layout from a tensor desc type if it exists.`。
- **L60 EN**: Continues the surrounding expression or declaration: `static std::optional<SmallVector<int64_t>>`.
  **L60 CN**: 继续构造周围的表达式或声明：`static std::optional<SmallVector<int64_t>>`。

### Lines 61-80

````cpp
getMaybeLaneLayout(xegpu::TensorDescType tdescType) {
  auto layout = tdescType.getLayoutAttr();
  if (!layout)
    return std::nullopt;
  auto laneLayout = layout.getEffectiveLaneLayoutAsInt();
  if (laneLayout.size() != 2)
    return std::nullopt;
  return laneLayout;
}

/// A layout can be optimized if its lane layout is transposed (lane[0] != 1 &&
/// lane[1] == 1), but inner lane data is not equal to [1, 1].
/// Example:
///     !xegpu.tensor_desc<16x16xf16,
///         #xegpu.layout<lane_layout = [16, 1], lane_data = [1, 2]>>
/// In this case, lane layout is transposed (from the usual [1, SG_SIZE] form)
/// indicating that this is a load that requires transpose effect. However,
/// lane data is [1, 2], meaning that each lane must grab 2 f16 elements from
/// the inner dimension. We convert this to a optimized form by converting the
/// tensor_desc to i32 type such that lane data becomes [1, 1]. This makes the
````
- **L61 EN**: Starts a function, method, lambda, or structured scope: `getMaybeLaneLayout(xegpu::TensorDescType tdescType) {`.
  **L61 CN**: 开始一个函数、方法、lambda 或结构化作用域：`getMaybeLaneLayout(xegpu::TensorDescType tdescType) {`。
- **L62 EN**: Initializes variable `layout` from the right-hand expression.
  **L62 CN**: 使用右侧表达式初始化变量 `layout`。
- **L63 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L63 CN**: 开始 `if` 控制流语句并计算其条件。
- **L64 EN**: Returns from the current function with `std::nullopt`.
  **L64 CN**: 以 `std::nullopt` 从当前函数返回。
- **L65 EN**: Initializes variable `laneLayout` from the right-hand expression.
  **L65 CN**: 使用右侧表达式初始化变量 `laneLayout`。
- **L66 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L66 CN**: 开始 `if` 控制流语句并计算其条件。
- **L67 EN**: Returns from the current function with `std::nullopt`.
  **L67 CN**: 以 `std::nullopt` 从当前函数返回。
- **L68 EN**: Returns from the current function with `laneLayout`.
  **L68 CN**: 以 `laneLayout` 从当前函数返回。
- **L69 EN**: Closes the current lexical scope or compound statement.
  **L69 CN**: 结束当前词法作用域或复合语句块。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L71 EN**: Comment explains nearby logic, invariants, or intent: `A layout can be optimized if its lane layout is transposed (lane[0] != 1 &&`.
  **L71 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A layout can be optimized if its lane layout is transposed (lane[0] != 1 &&`。
- **L72 EN**: Comment explains nearby logic, invariants, or intent: `lane[1] == 1), but inner lane data is not equal to [1, 1].`.
  **L72 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`lane[1] == 1), but inner lane data is not equal to [1, 1].`。
- **L73 EN**: Comment explains nearby logic, invariants, or intent: `Example:`.
  **L73 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Example:`。
- **L74 EN**: Comment explains nearby logic, invariants, or intent: `xegpu.tensor_desc<16x16xf16,`.
  **L74 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`xegpu.tensor_desc<16x16xf16,`。
- **L75 EN**: Comment explains nearby logic, invariants, or intent: `#xegpu.layout<lane_layout = [16, 1], lane_data = [1, 2]>>`.
  **L75 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`#xegpu.layout<lane_layout = [16, 1], lane_data = [1, 2]>>`。
- **L76 EN**: Comment explains nearby logic, invariants, or intent: `In this case, lane layout is transposed (from the usual [1, SG_SIZE] form)`.
  **L76 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`In this case, lane layout is transposed (from the usual [1, SG_SIZE] form)`。
- **L77 EN**: Comment explains nearby logic, invariants, or intent: `indicating that this is a load that requires transpose effect. However,`.
  **L77 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`indicating that this is a load that requires transpose effect. However,`。
- **L78 EN**: Comment explains nearby logic, invariants, or intent: `lane data is [1, 2], meaning that each lane must grab 2 f16 elements from`.
  **L78 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`lane data is [1, 2], meaning that each lane must grab 2 f16 elements from`。
- **L79 EN**: Comment explains nearby logic, invariants, or intent: `the inner dimension. We convert this to a optimized form by converting the`.
  **L79 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the inner dimension. We convert this to a optimized form by converting the`。
- **L80 EN**: Comment explains nearby logic, invariants, or intent: `tensor_desc to i32 type such that lane data becomes [1, 1]. This makes the`.
  **L80 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tensor_desc to i32 type such that lane data becomes [1, 1]. This makes the`。

### Lines 81-100

````cpp
/// later lowering easily use the load with transpose instruction.
static bool canBeOptimizedForTranspose(ArrayRef<int64_t> laneLayout,
                                       ArrayRef<int64_t> laneData) {
  if (laneLayout.size() != 2 || laneData.size() != 2)
    return false;
  if (laneLayout[0] == 1 || laneLayout[1] != 1)
    return false;
  if (laneData[0] != 1 || laneData[1] == 1)
    return false;
  return true;
}

/// A tensor desc type can be optimized if its element type is less than 32 bits
/// and its layout can be optimized.
static bool canBeOptimizedForTranspose(xegpu::TensorDescType tdescType) {
  // If the dtype is greater or equal to 32 bits, layout must be valid.
  int elementTyBitwidth = tdescType.getElementType().getIntOrFloatBitWidth();
  if (elementTyBitwidth >= 32)
    return false;
  auto maybeLaneLayout = getMaybeLaneLayout(tdescType);
````
- **L81 EN**: Comment explains nearby logic, invariants, or intent: `later lowering easily use the load with transpose instruction.`.
  **L81 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`later lowering easily use the load with transpose instruction.`。
- **L82 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static bool canBeOptimizedForTranspose(ArrayRef<int64_t> laneLayout,`.
  **L82 CN**: 继续一个多行参数列表、初始化器或聚合项：`static bool canBeOptimizedForTranspose(ArrayRef<int64_t> laneLayout,`。
- **L83 EN**: Continues the surrounding expression or declaration: `ArrayRef<int64_t> laneData) {`.
  **L83 CN**: 继续构造周围的表达式或声明：`ArrayRef<int64_t> laneData) {`。
- **L84 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L84 CN**: 开始 `if` 控制流语句并计算其条件。
- **L85 EN**: Returns from the current function with `false`.
  **L85 CN**: 以 `false` 从当前函数返回。
- **L86 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L86 CN**: 开始 `if` 控制流语句并计算其条件。
- **L87 EN**: Returns from the current function with `false`.
  **L87 CN**: 以 `false` 从当前函数返回。
- **L88 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L88 CN**: 开始 `if` 控制流语句并计算其条件。
- **L89 EN**: Returns from the current function with `false`.
  **L89 CN**: 以 `false` 从当前函数返回。
- **L90 EN**: Returns from the current function with `true`.
  **L90 CN**: 以 `true` 从当前函数返回。
- **L91 EN**: Closes the current lexical scope or compound statement.
  **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L93 EN**: Comment explains nearby logic, invariants, or intent: `A tensor desc type can be optimized if its element type is less than 32 bits`.
  **L93 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A tensor desc type can be optimized if its element type is less than 32 bits`。
- **L94 EN**: Comment explains nearby logic, invariants, or intent: `and its layout can be optimized.`.
  **L94 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and its layout can be optimized.`。
- **L95 EN**: Starts a function, method, lambda, or structured scope: `static bool canBeOptimizedForTranspose(xegpu::TensorDescType tdescType) {`.
  **L95 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool canBeOptimizedForTranspose(xegpu::TensorDescType tdescType) {`。
- **L96 EN**: Comment explains nearby logic, invariants, or intent: `If the dtype is greater or equal to 32 bits, layout must be valid.`.
  **L96 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If the dtype is greater or equal to 32 bits, layout must be valid.`。
- **L97 EN**: Initializes variable `elementTyBitwidth` from the right-hand expression.
  **L97 CN**: 使用右侧表达式初始化变量 `elementTyBitwidth`。
- **L98 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L98 CN**: 开始 `if` 控制流语句并计算其条件。
- **L99 EN**: Returns from the current function with `false`.
  **L99 CN**: 以 `false` 从当前函数返回。
- **L100 EN**: Initializes variable `maybeLaneLayout` from the right-hand expression.
  **L100 CN**: 使用右侧表达式初始化变量 `maybeLaneLayout`。

### Lines 101-120

````cpp
  auto maybeLaneData = getMaybeLaneData(tdescType);
  if (!maybeLaneData || !maybeLaneLayout)
    return false;
  return canBeOptimizedForTranspose(*maybeLaneLayout, *maybeLaneData);
}

/// Check if a tensor desc type can be optimized for transpose, if so return the
/// new optimized tensor desc type with a valid transpose layout.
static xegpu::TensorDescType tryOptimize(xegpu::TensorDescType tdescType,
                                         const uArch *targetuArch) {
  if (!canBeOptimizedForTranspose(tdescType))
    return tdescType;
  auto laneData = getMaybeLaneData(tdescType)
                      .value(); // Lane data must exist if we reach here.
  int64_t innerLaneData = laneData[1];
  int elementTyBitwidth = tdescType.getElementType().getIntOrFloatBitWidth();
  // Required shape is total shape of the vector result that this tensor desc
  // must eventually load after adjusting for the new bitwidth and array
  // length.
  SmallVector<int64_t> requiredShape(tdescType.getShape());
````
- **L101 EN**: Initializes variable `maybeLaneData` from the right-hand expression.
  **L101 CN**: 使用右侧表达式初始化变量 `maybeLaneData`。
- **L102 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L102 CN**: 开始 `if` 控制流语句并计算其条件。
- **L103 EN**: Returns from the current function with `false`.
  **L103 CN**: 以 `false` 从当前函数返回。
- **L104 EN**: Returns from the current function with `canBeOptimizedForTranspose(*maybeLaneLayout, *maybeLaneData)`.
  **L104 CN**: 以 `canBeOptimizedForTranspose(*maybeLaneLayout, *maybeLaneData)` 从当前函数返回。
- **L105 EN**: Closes the current lexical scope or compound statement.
  **L105 CN**: 结束当前词法作用域或复合语句块。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L107 EN**: Comment explains nearby logic, invariants, or intent: `Check if a tensor desc type can be optimized for transpose, if so return the`.
  **L107 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check if a tensor desc type can be optimized for transpose, if so return the`。
- **L108 EN**: Comment explains nearby logic, invariants, or intent: `new optimized tensor desc type with a valid transpose layout.`.
  **L108 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`new optimized tensor desc type with a valid transpose layout.`。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static xegpu::TensorDescType tryOptimize(xegpu::TensorDescType tdescType,`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`static xegpu::TensorDescType tryOptimize(xegpu::TensorDescType tdescType,`。
- **L110 EN**: Continues the surrounding expression or declaration: `const uArch *targetuArch) {`.
  **L110 CN**: 继续构造周围的表达式或声明：`const uArch *targetuArch) {`。
- **L111 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L111 CN**: 开始 `if` 控制流语句并计算其条件。
- **L112 EN**: Returns from the current function with `tdescType`.
  **L112 CN**: 以 `tdescType` 从当前函数返回。
- **L113 EN**: Continues logic associated with callable symbol `getMaybeLaneData`.
  **L113 CN**: 继续与可调用符号 `getMaybeLaneData` 相关的逻辑。
- **L114 EN**: Continues logic associated with callable symbol `value`.
  **L114 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L115 EN**: Initializes variable `innerLaneData` from the right-hand expression.
  **L115 CN**: 使用右侧表达式初始化变量 `innerLaneData`。
- **L116 EN**: Initializes variable `elementTyBitwidth` from the right-hand expression.
  **L116 CN**: 使用右侧表达式初始化变量 `elementTyBitwidth`。
- **L117 EN**: Comment explains nearby logic, invariants, or intent: `Required shape is total shape of the vector result that this tensor desc`.
  **L117 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Required shape is total shape of the vector result that this tensor desc`。
- **L118 EN**: Comment explains nearby logic, invariants, or intent: `must eventually load after adjusting for the new bitwidth and array`.
  **L118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`must eventually load after adjusting for the new bitwidth and array`。
- **L119 EN**: Comment explains nearby logic, invariants, or intent: `length.`.
  **L119 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`length.`。
- **L120 EN**: Executes a call or declaration centered on `requiredShape`.
  **L120 CN**: 执行以 `requiredShape` 为核心的调用或声明。

### Lines 121-140

````cpp
  requiredShape.back() =
      requiredShape.back() * tdescType.getArrayLength() / innerLaneData;
  int newBitWidth = elementTyBitwidth * innerLaneData;
  Type newElemTy = IntegerType::get(tdescType.getContext(), newBitWidth);
  // Supported shape is the max transpose shape that can be supported by
  // hardware that is less than or equal to required shape.
  auto *blockLoadTarget = dyn_cast<Subgroup2DBlockLoadInstruction>(
      targetuArch->getInstruction(InstructionKind::Subgroup2DBlockLoad));
  auto maybeHWParams = blockLoadTarget->getBlockWidthHeightCount(
      newElemTy, /** has transform */ false, /** has transpose */ true);
  // If no HW params found, return the original type.
  if (!maybeHWParams)
    return tdescType;
  auto [widths, heights, counts] = maybeHWParams.value();
  // TODO: Currently we expect array length to be 1 for transpose case.
  if (counts.size() != 1 || counts[0] != 1)
    return tdescType;
  int arrayLen = counts[0];
  int supportedHeight =
      xegpu::getLargestDivisor(static_cast<int>(requiredShape[0]), heights);
````
- **L121 EN**: Continues logic associated with callable symbol `back`.
  **L121 CN**: 继续与可调用符号 `back` 相关的逻辑。
- **L122 EN**: Executes a call or declaration centered on `requiredShape.back`.
  **L122 CN**: 执行以 `requiredShape.back` 为核心的调用或声明。
- **L123 EN**: Initializes variable `newBitWidth` from the right-hand expression.
  **L123 CN**: 使用右侧表达式初始化变量 `newBitWidth`。
- **L124 EN**: Initializes variable `newElemTy` from the right-hand expression.
  **L124 CN**: 使用右侧表达式初始化变量 `newElemTy`。
- **L125 EN**: Comment explains nearby logic, invariants, or intent: `Supported shape is the max transpose shape that can be supported by`.
  **L125 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Supported shape is the max transpose shape that can be supported by`。
- **L126 EN**: Comment explains nearby logic, invariants, or intent: `hardware that is less than or equal to required shape.`.
  **L126 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`hardware that is less than or equal to required shape.`。
- **L127 EN**: Continues logic associated with callable symbol `dyn_cast<Subgroup2DBlockLoadInstruction>`.
  **L127 CN**: 继续与可调用符号 `dyn_cast<Subgroup2DBlockLoadInstruction>` 相关的逻辑。
- **L128 EN**: Executes a call or declaration centered on `targetuArch->getInstruction`.
  **L128 CN**: 执行以 `targetuArch->getInstruction` 为核心的调用或声明。
- **L129 EN**: Continues logic associated with callable symbol `getBlockWidthHeightCount`.
  **L129 CN**: 继续与可调用符号 `getBlockWidthHeightCount` 相关的逻辑。
- **L130 EN**: Executes a standalone statement or declaration: `newElemTy, /** has transform */ false, /** has transpose */ true);`.
  **L130 CN**: 执行一条独立语句或声明：`newElemTy, /** has transform */ false, /** has transpose */ true);`。
- **L131 EN**: Comment explains nearby logic, invariants, or intent: `If no HW params found, return the original type.`.
  **L131 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If no HW params found, return the original type.`。
- **L132 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L132 CN**: 开始 `if` 控制流语句并计算其条件。
- **L133 EN**: Returns from the current function with `tdescType`.
  **L133 CN**: 以 `tdescType` 从当前函数返回。
- **L134 EN**: Executes a call or declaration centered on `maybeHWParams.value`.
  **L134 CN**: 执行以 `maybeHWParams.value` 为核心的调用或声明。
- **L135 EN**: Comment records a pending task or caution: `TODO: Currently we expect array length to be 1 for transpose case.`.
  **L135 CN**: 注释记录了待办事项或注意点：`TODO: Currently we expect array length to be 1 for transpose case.`。
- **L136 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L136 CN**: 开始 `if` 控制流语句并计算其条件。
- **L137 EN**: Returns from the current function with `tdescType`.
  **L137 CN**: 以 `tdescType` 从当前函数返回。
- **L138 EN**: Initializes variable `arrayLen` from the right-hand expression.
  **L138 CN**: 使用右侧表达式初始化变量 `arrayLen`。
- **L139 EN**: Continues the surrounding expression or declaration: `int supportedHeight =`.
  **L139 CN**: 继续构造周围的表达式或声明：`int supportedHeight =`。
- **L140 EN**: Executes a call or declaration centered on `xegpu::getLargestDivisor`.
  **L140 CN**: 执行以 `xegpu::getLargestDivisor` 为核心的调用或声明。

### Lines 141-160

````cpp
  int supportedWidth =
      xegpu::getLargestDivisor(static_cast<int>(requiredShape[1]), widths);
  // If no supported height or width found, return the original type.
  if (supportedHeight == -1 || supportedWidth == -1)
    return tdescType;

  SmallVector<int64_t> supportedShape = {supportedHeight, supportedWidth};
  auto ctx = tdescType.getContext();
  auto origLayout = tdescType.getLayoutAttr();
  auto laneLayoutI64 = origLayout.getEffectiveLaneLayoutAsInt();
  SmallVector<int32_t> laneLayoutI32(laneLayoutI64.begin(),
                                     laneLayoutI64.end());

  xegpu::LayoutAttr newLayout = xegpu::LayoutAttr::get(
      ctx, /*lane_layout=*/DenseI32ArrayAttr::get(ctx, laneLayoutI32),
      /*lane_data=*/DenseI32ArrayAttr::get(ctx, {1, 1}),
      /*order=*/origLayout.getOrder());

  // Array length can not be larger than 1 for transpose case.
  return xegpu::TensorDescType::get(supportedShape, newElemTy, arrayLen,
````
- **L141 EN**: Continues the surrounding expression or declaration: `int supportedWidth =`.
  **L141 CN**: 继续构造周围的表达式或声明：`int supportedWidth =`。
- **L142 EN**: Executes a call or declaration centered on `xegpu::getLargestDivisor`.
  **L142 CN**: 执行以 `xegpu::getLargestDivisor` 为核心的调用或声明。
- **L143 EN**: Comment explains nearby logic, invariants, or intent: `If no supported height or width found, return the original type.`.
  **L143 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If no supported height or width found, return the original type.`。
- **L144 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L144 CN**: 开始 `if` 控制流语句并计算其条件。
- **L145 EN**: Returns from the current function with `tdescType`.
  **L145 CN**: 以 `tdescType` 从当前函数返回。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L147 EN**: Initializes variable `supportedShape` from the right-hand expression.
  **L147 CN**: 使用右侧表达式初始化变量 `supportedShape`。
- **L148 EN**: Initializes variable `ctx` from the right-hand expression.
  **L148 CN**: 使用右侧表达式初始化变量 `ctx`。
- **L149 EN**: Initializes variable `origLayout` from the right-hand expression.
  **L149 CN**: 使用右侧表达式初始化变量 `origLayout`。
- **L150 EN**: Initializes variable `laneLayoutI64` from the right-hand expression.
  **L150 CN**: 使用右侧表达式初始化变量 `laneLayoutI64`。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<int32_t> laneLayoutI32(laneLayoutI64.begin(),`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<int32_t> laneLayoutI32(laneLayoutI64.begin(),`。
- **L152 EN**: Executes a call or declaration centered on `laneLayoutI64.end`.
  **L152 CN**: 执行以 `laneLayoutI64.end` 为核心的调用或声明。
- **L153 EN**: Blank line separating nearby declarations or logic blocks.
  **L153 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L154 EN**: Continues logic associated with callable symbol `get`.
  **L154 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ctx, /*lane_layout=*/DenseI32ArrayAttr::get(ctx, laneLayoutI32),`.
  **L155 CN**: 继续一个多行参数列表、初始化器或聚合项：`ctx, /*lane_layout=*/DenseI32ArrayAttr::get(ctx, laneLayoutI32),`。
- **L156 EN**: Comment explains nearby logic, invariants, or intent: `lane_data=*/DenseI32ArrayAttr::get(ctx, {1, 1}),`.
  **L156 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`lane_data=*/DenseI32ArrayAttr::get(ctx, {1, 1}),`。
- **L157 EN**: Comment explains nearby logic, invariants, or intent: `order=*/origLayout.getOrder());`.
  **L157 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`order=*/origLayout.getOrder());`。
- **L158 EN**: Blank line separating nearby declarations or logic blocks.
  **L158 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L159 EN**: Comment explains nearby logic, invariants, or intent: `Array length can not be larger than 1 for transpose case.`.
  **L159 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Array length can not be larger than 1 for transpose case.`。
- **L160 EN**: Returns from the current function with `xegpu::TensorDescType::get(supportedShape, newElemTy, arrayLen,`.
  **L160 CN**: 以 `xegpu::TensorDescType::get(supportedShape, newElemTy, arrayLen,` 从当前函数返回。

### Lines 161-180

````cpp
                                    tdescType.getBoundaryCheck(),
                                    tdescType.getMemorySpace(), newLayout);
}

/// Helper to convert an OpFoldResult to Value.
static Value convertToValue(ConversionPatternRewriter &rewriter, Location loc,
                            OpFoldResult ofr) {
  std::optional<int64_t> mayBeInt = getConstantIntValue(ofr);
  if (mayBeInt)
    return arith::ConstantIndexOp::create(rewriter, loc, *mayBeInt).getResult();
  return llvm::cast<Value>(ofr);
}

/// Helper to divide a Value by a constant integer.
static Value divideByConstant(ConversionPatternRewriter &rewriter, Location loc,
                              Value val, int64_t constant) {
  // If the constant is a power of 2, use right shift for division.
  if (llvm::isPowerOf2_64(constant)) {
    int64_t shiftAmount = llvm::Log2_64(constant);
    return arith::ShRUIOp::create(
````
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tdescType.getBoundaryCheck(),`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`tdescType.getBoundaryCheck(),`。
- **L162 EN**: Executes a call or declaration centered on `tdescType.getMemorySpace`.
  **L162 CN**: 执行以 `tdescType.getMemorySpace` 为核心的调用或声明。
- **L163 EN**: Closes the current lexical scope or compound statement.
  **L163 CN**: 结束当前词法作用域或复合语句块。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L165 EN**: Comment explains nearby logic, invariants, or intent: `Helper to convert an OpFoldResult to Value.`.
  **L165 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper to convert an OpFoldResult to Value.`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value convertToValue(ConversionPatternRewriter &rewriter, Location loc,`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value convertToValue(ConversionPatternRewriter &rewriter, Location loc,`。
- **L167 EN**: Continues the surrounding expression or declaration: `OpFoldResult ofr) {`.
  **L167 CN**: 继续构造周围的表达式或声明：`OpFoldResult ofr) {`。
- **L168 EN**: Initializes variable `mayBeInt` from the right-hand expression.
  **L168 CN**: 使用右侧表达式初始化变量 `mayBeInt`。
- **L169 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L169 CN**: 开始 `if` 控制流语句并计算其条件。
- **L170 EN**: Returns from the current function with `arith::ConstantIndexOp::create(rewriter, loc, *mayBeInt).getResult()`.
  **L170 CN**: 以 `arith::ConstantIndexOp::create(rewriter, loc, *mayBeInt).getResult()` 从当前函数返回。
- **L171 EN**: Returns from the current function with `llvm::cast<Value>(ofr)`.
  **L171 CN**: 以 `llvm::cast<Value>(ofr)` 从当前函数返回。
- **L172 EN**: Closes the current lexical scope or compound statement.
  **L172 CN**: 结束当前词法作用域或复合语句块。
- **L173 EN**: Blank line separating nearby declarations or logic blocks.
  **L173 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L174 EN**: Comment explains nearby logic, invariants, or intent: `Helper to divide a Value by a constant integer.`.
  **L174 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper to divide a Value by a constant integer.`。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value divideByConstant(ConversionPatternRewriter &rewriter, Location loc,`.
  **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value divideByConstant(ConversionPatternRewriter &rewriter, Location loc,`。
- **L176 EN**: Continues the surrounding expression or declaration: `Value val, int64_t constant) {`.
  **L176 CN**: 继续构造周围的表达式或声明：`Value val, int64_t constant) {`。
- **L177 EN**: Comment explains nearby logic, invariants, or intent: `If the constant is a power of 2, use right shift for division.`.
  **L177 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If the constant is a power of 2, use right shift for division.`。
- **L178 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L178 CN**: 开始 `if` 控制流语句并计算其条件。
- **L179 EN**: Initializes variable `shiftAmount` from the right-hand expression.
  **L179 CN**: 使用右侧表达式初始化变量 `shiftAmount`。
- **L180 EN**: Returns from the current function with `arith::ShRUIOp::create(`.
  **L180 CN**: 以 `arith::ShRUIOp::create(` 从当前函数返回。

### Lines 181-200

````cpp
               rewriter, loc, val,
               arith::ConstantIndexOp::create(rewriter, loc, shiftAmount)
                   .getResult())
        .getResult();
  }
  auto constantOp =
      arith::ConstantIndexOp::create(rewriter, loc, constant).getResult();
  return arith::DivUIOp::create(rewriter, loc, val, constantOp).getResult();
}

/// This function takes a larger register block `data` and generates multiple
/// smaller loads (size given by `newTensorDesc`) to fill in the `data` block
/// starting from `offsets`.
static Value generateLoads(ConversionPatternRewriter &rewriter,
                           TypedValue<VectorType> data,
                           SmallVector<OpFoldResult> offsets,
                           TypedValue<xegpu::TensorDescType> newTensorDesc,
                           xegpu::LoadNdOp origLoadOp) {
  Location loc = data.getLoc();
  assert(offsets.size() >= 2 && "Expecting at least 2 offsets for 2D LoadNdOp");
````
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, val,`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, val,`。
- **L182 EN**: Continues logic associated with callable symbol `create`.
  **L182 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L183 EN**: Continues logic associated with callable symbol `getResult`.
  **L183 CN**: 继续与可调用符号 `getResult` 相关的逻辑。
- **L184 EN**: Executes a call or declaration centered on `.getResult`.
  **L184 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L185 EN**: Closes the current lexical scope or compound statement.
  **L185 CN**: 结束当前词法作用域或复合语句块。
- **L186 EN**: Continues the surrounding expression or declaration: `auto constantOp =`.
  **L186 CN**: 继续构造周围的表达式或声明：`auto constantOp =`。
- **L187 EN**: Executes a call or declaration centered on `arith::ConstantIndexOp::create`.
  **L187 CN**: 执行以 `arith::ConstantIndexOp::create` 为核心的调用或声明。
- **L188 EN**: Returns from the current function with `arith::DivUIOp::create(rewriter, loc, val, constantOp).getResult()`.
  **L188 CN**: 以 `arith::DivUIOp::create(rewriter, loc, val, constantOp).getResult()` 从当前函数返回。
- **L189 EN**: Closes the current lexical scope or compound statement.
  **L189 CN**: 结束当前词法作用域或复合语句块。
- **L190 EN**: Blank line separating nearby declarations or logic blocks.
  **L190 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L191 EN**: Comment explains nearby logic, invariants, or intent: `This function takes a larger register block `data` and generates multiple`.
  **L191 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This function takes a larger register block `data` and generates multiple`。
- **L192 EN**: Comment explains nearby logic, invariants, or intent: `smaller loads (size given by `newTensorDesc`) to fill in the `data` block`.
  **L192 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`smaller loads (size given by `newTensorDesc`) to fill in the `data` block`。
- **L193 EN**: Comment explains nearby logic, invariants, or intent: `starting from `offsets`.`.
  **L193 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`starting from `offsets`.`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value generateLoads(ConversionPatternRewriter &rewriter,`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value generateLoads(ConversionPatternRewriter &rewriter,`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypedValue<VectorType> data,`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypedValue<VectorType> data,`。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<OpFoldResult> offsets,`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<OpFoldResult> offsets,`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TypedValue<xegpu::TensorDescType> newTensorDesc,`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`TypedValue<xegpu::TensorDescType> newTensorDesc,`。
- **L198 EN**: Continues the surrounding expression or declaration: `xegpu::LoadNdOp origLoadOp) {`.
  **L198 CN**: 继续构造周围的表达式或声明：`xegpu::LoadNdOp origLoadOp) {`。
- **L199 EN**: Initializes variable `loc` from the right-hand expression.
  **L199 CN**: 使用右侧表达式初始化变量 `loc`。
- **L200 EN**: Checks an internal invariant in debug builds.
  **L200 CN**: 在调试构建中检查内部不变式。

### Lines 201-220

````cpp
  Value offsetDim0 = convertToValue(rewriter, loc, offsets[offsets.size() - 2]);
  Value offsetDim1 = convertToValue(rewriter, loc, offsets[offsets.size() - 1]);
  SmallVector<int64_t> supportedShape(newTensorDesc.getType().getShape());
  // Compute the ratio between original shape and supported shape. We need to
  // generate loads in this ratio arrangement.
  auto shapeRatio = computeShapeRatio(data.getType().getShape(),
                                      supportedShape)
                        .value(); // `ratio` must be defined if we reach here.
  for (int64_t h = 0; h < shapeRatio[0]; ++h) {
    for (int64_t w = 0; w < shapeRatio[1]; ++w) {
      int64_t localOffsetDim0 = h * supportedShape[0];
      int64_t localOffsetDim1 = w * supportedShape[1];
      Value loadOffsetX = arith::AddIOp::create(
          rewriter, loc, offsetDim0,
          arith::ConstantIndexOp::create(rewriter, loc, localOffsetDim0)
              .getResult());
      Value loadOffsetY = arith::AddIOp::create(
          rewriter, loc, offsetDim1,
          arith::ConstantIndexOp::create(rewriter, loc, localOffsetDim1)
              .getResult());
````
- **L201 EN**: Initializes variable `offsetDim0` from the right-hand expression.
  **L201 CN**: 使用右侧表达式初始化变量 `offsetDim0`。
- **L202 EN**: Initializes variable `offsetDim1` from the right-hand expression.
  **L202 CN**: 使用右侧表达式初始化变量 `offsetDim1`。
- **L203 EN**: Executes a call or declaration centered on `supportedShape`.
  **L203 CN**: 执行以 `supportedShape` 为核心的调用或声明。
- **L204 EN**: Comment explains nearby logic, invariants, or intent: `Compute the ratio between original shape and supported shape. We need to`.
  **L204 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Compute the ratio between original shape and supported shape. We need to`。
- **L205 EN**: Comment explains nearby logic, invariants, or intent: `generate loads in this ratio arrangement.`.
  **L205 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`generate loads in this ratio arrangement.`。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto shapeRatio = computeShapeRatio(data.getType().getShape(),`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto shapeRatio = computeShapeRatio(data.getType().getShape(),`。
- **L207 EN**: Continues the surrounding expression or declaration: `supportedShape)`.
  **L207 CN**: 继续构造周围的表达式或声明：`supportedShape)`。
- **L208 EN**: Continues logic associated with callable symbol `value`.
  **L208 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L209 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L209 CN**: 开始 `for` 控制流语句并计算其条件。
- **L210 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L210 CN**: 开始 `for` 控制流语句并计算其条件。
- **L211 EN**: Initializes variable `localOffsetDim0` from the right-hand expression.
  **L211 CN**: 使用右侧表达式初始化变量 `localOffsetDim0`。
- **L212 EN**: Initializes variable `localOffsetDim1` from the right-hand expression.
  **L212 CN**: 使用右侧表达式初始化变量 `localOffsetDim1`。
- **L213 EN**: Continues logic associated with callable symbol `create`.
  **L213 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, offsetDim0,`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, offsetDim0,`。
- **L215 EN**: Continues logic associated with callable symbol `create`.
  **L215 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L216 EN**: Executes a call or declaration centered on `.getResult`.
  **L216 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L217 EN**: Continues logic associated with callable symbol `create`.
  **L217 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, offsetDim1,`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, offsetDim1,`。
- **L219 EN**: Continues logic associated with callable symbol `create`.
  **L219 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L220 EN**: Executes a call or declaration centered on `.getResult`.
  **L220 CN**: 执行以 `.getResult` 为核心的调用或声明。

### Lines 221-240

````cpp
      auto loadOp = xegpu::LoadNdOp::create(
          rewriter, loc,
          VectorType::get(supportedShape, data.getType().getElementType()),
          newTensorDesc, ArrayRef<OpFoldResult>{loadOffsetX, loadOffsetY},
          origLoadOp.getPackedAttr(), origLoadOp.getTransposeAttr(),
          origLoadOp.getL1HintAttr(), origLoadOp.getL2HintAttr(),
          origLoadOp.getL3HintAttr(), origLoadOp.getLayoutAttr());
      // Set the layout for the loadOp.
      auto layoutAttr = newTensorDesc.getType().getLayoutAttr();
      loadOp.setAnchorLayout(layoutAttr);
      // Insert the loaded block into the right position in data.
      auto insertOp = vector::InsertStridedSliceOp::create(
          rewriter, loc, loadOp.getResult(), data,
          ArrayRef<int64_t>{localOffsetDim0, localOffsetDim1},
          ArrayRef<int64_t>{1, 1});
      // InsertOp must have the same layout as newTensorDesc.
      xegpu::setTemporaryLayout(insertOp->getOpResult(0), layoutAttr);
      data = insertOp.getResult();
    }
  }
````
- **L221 EN**: Continues logic associated with callable symbol `create`.
  **L221 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc,`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc,`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VectorType::get(supportedShape, data.getType().getElementType()),`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`VectorType::get(supportedShape, data.getType().getElementType()),`。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `newTensorDesc, ArrayRef<OpFoldResult>{loadOffsetX, loadOffsetY},`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`newTensorDesc, ArrayRef<OpFoldResult>{loadOffsetX, loadOffsetY},`。
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `origLoadOp.getPackedAttr(), origLoadOp.getTransposeAttr(),`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`origLoadOp.getPackedAttr(), origLoadOp.getTransposeAttr(),`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `origLoadOp.getL1HintAttr(), origLoadOp.getL2HintAttr(),`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`origLoadOp.getL1HintAttr(), origLoadOp.getL2HintAttr(),`。
- **L227 EN**: Executes a call or declaration centered on `origLoadOp.getL3HintAttr`.
  **L227 CN**: 执行以 `origLoadOp.getL3HintAttr` 为核心的调用或声明。
- **L228 EN**: Comment explains nearby logic, invariants, or intent: `Set the layout for the loadOp.`.
  **L228 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Set the layout for the loadOp.`。
- **L229 EN**: Initializes variable `layoutAttr` from the right-hand expression.
  **L229 CN**: 使用右侧表达式初始化变量 `layoutAttr`。
- **L230 EN**: Executes a call or declaration centered on `loadOp.setAnchorLayout`.
  **L230 CN**: 执行以 `loadOp.setAnchorLayout` 为核心的调用或声明。
- **L231 EN**: Comment explains nearby logic, invariants, or intent: `Insert the loaded block into the right position in data.`.
  **L231 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Insert the loaded block into the right position in data.`。
- **L232 EN**: Continues logic associated with callable symbol `create`.
  **L232 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, loadOp.getResult(), data,`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, loadOp.getResult(), data,`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<int64_t>{localOffsetDim0, localOffsetDim1},`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<int64_t>{localOffsetDim0, localOffsetDim1},`。
- **L235 EN**: Executes a standalone statement or declaration: `ArrayRef<int64_t>{1, 1});`.
  **L235 CN**: 执行一条独立语句或声明：`ArrayRef<int64_t>{1, 1});`。
- **L236 EN**: Comment explains nearby logic, invariants, or intent: `InsertOp must have the same layout as newTensorDesc.`.
  **L236 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`InsertOp must have the same layout as newTensorDesc.`。
- **L237 EN**: Executes a call or declaration centered on `xegpu::setTemporaryLayout`.
  **L237 CN**: 执行以 `xegpu::setTemporaryLayout` 为核心的调用或声明。
- **L238 EN**: Executes a call or declaration centered on `insertOp.getResult`.
  **L238 CN**: 执行以 `insertOp.getResult` 为核心的调用或声明。
- **L239 EN**: Closes the current lexical scope or compound statement.
  **L239 CN**: 结束当前词法作用域或复合语句块。
- **L240 EN**: Closes the current lexical scope or compound statement.
  **L240 CN**: 结束当前词法作用域或复合语句块。

### Lines 241-260

````cpp
  return data;
}

/// Checks if a CreateNdDescOp can be optimized for transpose, if so creates a
/// new CreateNdDescOp with optimized tensor desc type. This involves extracting
/// the base pointer from the original memory source and adjusting the shape and
/// strides of the tensor desc to fit with the new optimized transpose layout.
class XeGPUCreateNdDescOpPattern final
    : public OpConversionPattern<xegpu::CreateNdDescOp> {
public:
  using OpConversionPattern<xegpu::CreateNdDescOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(xegpu::CreateNdDescOp createNdOp, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto tdescTy = createNdOp.getType();
    // Get the target uArch info.
    auto chipStr = xegpu::getChipStr(createNdOp);
    // Check if the chip is supported.
    assert(chipStr &&
           (chipStr.value() == "pvc" || chipStr.value() == "bmg" ||
````
- **L241 EN**: Returns from the current function with `data`.
  **L241 CN**: 以 `data` 从当前函数返回。
- **L242 EN**: Closes the current lexical scope or compound statement.
  **L242 CN**: 结束当前词法作用域或复合语句块。
- **L243 EN**: Blank line separating nearby declarations or logic blocks.
  **L243 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L244 EN**: Comment explains nearby logic, invariants, or intent: `Checks if a CreateNdDescOp can be optimized for transpose, if so creates a`.
  **L244 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Checks if a CreateNdDescOp can be optimized for transpose, if so creates a`。
- **L245 EN**: Comment explains nearby logic, invariants, or intent: `new CreateNdDescOp with optimized tensor desc type. This involves extracting`.
  **L245 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`new CreateNdDescOp with optimized tensor desc type. This involves extracting`。
- **L246 EN**: Comment explains nearby logic, invariants, or intent: `the base pointer from the original memory source and adjusting the shape and`.
  **L246 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the base pointer from the original memory source and adjusting the shape and`。
- **L247 EN**: Comment explains nearby logic, invariants, or intent: `strides of the tensor desc to fit with the new optimized transpose layout.`.
  **L247 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`strides of the tensor desc to fit with the new optimized transpose layout.`。
- **L248 EN**: Declares class `XeGPUCreateNdDescOpPattern`.
  **L248 CN**: 声明 class `XeGPUCreateNdDescOpPattern`。
- **L249 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<xegpu::CreateNdDescOp> {`.
  **L249 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<xegpu::CreateNdDescOp> {`。
- **L250 EN**: Sets the following members to `public` access.
  **L250 CN**: 将后续成员的访问级别设为 `public`。
- **L251 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::CreateNdDescOp>::OpConversionPattern;`.
  **L251 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::CreateNdDescOp>::OpConversionPattern;`。
- **L252 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L252 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::CreateNdDescOp createNdOp, OpAdaptor adaptor,`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::CreateNdDescOp createNdOp, OpAdaptor adaptor,`。
- **L254 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L254 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L255 EN**: Initializes variable `tdescTy` from the right-hand expression.
  **L255 CN**: 使用右侧表达式初始化变量 `tdescTy`。
- **L256 EN**: Comment explains nearby logic, invariants, or intent: `Get the target uArch info.`.
  **L256 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get the target uArch info.`。
- **L257 EN**: Initializes variable `chipStr` from the right-hand expression.
  **L257 CN**: 使用右侧表达式初始化变量 `chipStr`。
- **L258 EN**: Comment explains nearby logic, invariants, or intent: `Check if the chip is supported.`.
  **L258 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check if the chip is supported.`。
- **L259 EN**: Checks an internal invariant in debug builds.
  **L259 CN**: 在调试构建中检查内部不变式。
- **L260 EN**: Continues logic associated with callable symbol `value`.
  **L260 CN**: 继续与可调用符号 `value` 相关的逻辑。

### Lines 261-280

````cpp
            chipStr.value() == "cri") &&
           "Expecting target chip to be pvc, bmg or cri for transpose "
           "optimization.");
    const uArch *targetuArch = xegpu::uArch::getUArch(chipStr.value());

    auto convertType = tryOptimize(tdescTy, targetuArch);
    if (convertType == tdescTy)
      return failure();
    auto strides = createNdOp.getMixedStrides();
    auto maybeConstInnerStride = getConstantIntValue(strides.back());
    // Only row-major memrefs are expected for now.
    if (!maybeConstInnerStride || *maybeConstInnerStride != 1)
      return rewriter.notifyMatchFailure(
          createNdOp, "Expecting row-major memref for transpose optimization.");
    Value source = createNdOp.getSource();
    auto optionalLaneData = getMaybeLaneData(tdescTy);
    assert(optionalLaneData && "Expected 2D lane data");
    auto laneData = optionalLaneData.value();
    int64_t innerLaneData = laneData[1];
    auto memrefType = dyn_cast<MemRefType>(source.getType());
````
- **L261 EN**: Continues logic associated with callable symbol `value`.
  **L261 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L262 EN**: Continues the surrounding expression or declaration: `"Expecting target chip to be pvc, bmg or cri for transpose "`.
  **L262 CN**: 继续构造周围的表达式或声明：`"Expecting target chip to be pvc, bmg or cri for transpose "`。
- **L263 EN**: Executes a standalone statement or declaration: `"optimization.");`.
  **L263 CN**: 执行一条独立语句或声明：`"optimization.");`。
- **L264 EN**: Executes a call or declaration centered on `xegpu::uArch::getUArch`.
  **L264 CN**: 执行以 `xegpu::uArch::getUArch` 为核心的调用或声明。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L266 EN**: Initializes variable `convertType` from the right-hand expression.
  **L266 CN**: 使用右侧表达式初始化变量 `convertType`。
- **L267 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L267 CN**: 开始 `if` 控制流语句并计算其条件。
- **L268 EN**: Returns from the current function with `failure()`.
  **L268 CN**: 以 `failure()` 从当前函数返回。
- **L269 EN**: Initializes variable `strides` from the right-hand expression.
  **L269 CN**: 使用右侧表达式初始化变量 `strides`。
- **L270 EN**: Initializes variable `maybeConstInnerStride` from the right-hand expression.
  **L270 CN**: 使用右侧表达式初始化变量 `maybeConstInnerStride`。
- **L271 EN**: Comment explains nearby logic, invariants, or intent: `Only row-major memrefs are expected for now.`.
  **L271 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only row-major memrefs are expected for now.`。
- **L272 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L272 CN**: 开始 `if` 控制流语句并计算其条件。
- **L273 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L273 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L274 EN**: Executes a standalone statement or declaration: `createNdOp, "Expecting row-major memref for transpose optimization.");`.
  **L274 CN**: 执行一条独立语句或声明：`createNdOp, "Expecting row-major memref for transpose optimization.");`。
- **L275 EN**: Initializes variable `source` from the right-hand expression.
  **L275 CN**: 使用右侧表达式初始化变量 `source`。
- **L276 EN**: Initializes variable `optionalLaneData` from the right-hand expression.
  **L276 CN**: 使用右侧表达式初始化变量 `optionalLaneData`。
- **L277 EN**: Checks an internal invariant in debug builds.
  **L277 CN**: 在调试构建中检查内部不变式。
- **L278 EN**: Initializes variable `laneData` from the right-hand expression.
  **L278 CN**: 使用右侧表达式初始化变量 `laneData`。
- **L279 EN**: Initializes variable `innerLaneData` from the right-hand expression.
  **L279 CN**: 使用右侧表达式初始化变量 `innerLaneData`。
- **L280 EN**: Initializes variable `memrefType` from the right-hand expression.
  **L280 CN**: 使用右侧表达式初始化变量 `memrefType`。

### Lines 281-300

````cpp
    // Inner dimension of the shape must be adjusted based on innerLaneData.
    SmallVector<OpFoldResult> modifiedShape(createNdOp.getMixedSizes());
    modifiedShape.back() = divideByConstant(
        rewriter, createNdOp.getLoc(),
        convertToValue(rewriter, createNdOp.getLoc(), modifiedShape.back()),
        innerLaneData);
    // Similarly, second to last stride must be adjusted.
    assert(strides.size() >= 2 &&
           "Expected at least 2 strides for CreateNdDescOp");
    SmallVector<OpFoldResult> modifiedStrides(strides);
    modifiedStrides[modifiedStrides.size() - 2] = divideByConstant(
        rewriter, createNdOp.getLoc(),
        convertToValue(rewriter, createNdOp.getLoc(),
                       modifiedStrides[modifiedStrides.size() - 2]),
        innerLaneData);

    // If the source is a static memref, we need to extract the pointer to
    // base address.
    if (memrefType && memrefType.hasStaticShape()) {
      auto extractOp = memref::ExtractAlignedPointerAsIndexOp::create(
````
- **L281 EN**: Comment explains nearby logic, invariants, or intent: `Inner dimension of the shape must be adjusted based on innerLaneData.`.
  **L281 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Inner dimension of the shape must be adjusted based on innerLaneData.`。
- **L282 EN**: Executes a call or declaration centered on `modifiedShape`.
  **L282 CN**: 执行以 `modifiedShape` 为核心的调用或声明。
- **L283 EN**: Continues logic associated with callable symbol `back`.
  **L283 CN**: 继续与可调用符号 `back` 相关的逻辑。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, createNdOp.getLoc(),`.
  **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, createNdOp.getLoc(),`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `convertToValue(rewriter, createNdOp.getLoc(), modifiedShape.back()),`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`convertToValue(rewriter, createNdOp.getLoc(), modifiedShape.back()),`。
- **L286 EN**: Executes a standalone statement or declaration: `innerLaneData);`.
  **L286 CN**: 执行一条独立语句或声明：`innerLaneData);`。
- **L287 EN**: Comment explains nearby logic, invariants, or intent: `Similarly, second to last stride must be adjusted.`.
  **L287 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Similarly, second to last stride must be adjusted.`。
- **L288 EN**: Checks an internal invariant in debug builds.
  **L288 CN**: 在调试构建中检查内部不变式。
- **L289 EN**: Executes a standalone statement or declaration: `"Expected at least 2 strides for CreateNdDescOp");`.
  **L289 CN**: 执行一条独立语句或声明：`"Expected at least 2 strides for CreateNdDescOp");`。
- **L290 EN**: Executes a call or declaration centered on `modifiedStrides`.
  **L290 CN**: 执行以 `modifiedStrides` 为核心的调用或声明。
- **L291 EN**: Continues logic associated with callable symbol `size`.
  **L291 CN**: 继续与可调用符号 `size` 相关的逻辑。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, createNdOp.getLoc(),`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, createNdOp.getLoc(),`。
- **L293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `convertToValue(rewriter, createNdOp.getLoc(),`.
  **L293 CN**: 继续一个多行参数列表、初始化器或聚合项：`convertToValue(rewriter, createNdOp.getLoc(),`。
- **L294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `modifiedStrides[modifiedStrides.size() - 2]),`.
  **L294 CN**: 继续一个多行参数列表、初始化器或聚合项：`modifiedStrides[modifiedStrides.size() - 2]),`。
- **L295 EN**: Executes a standalone statement or declaration: `innerLaneData);`.
  **L295 CN**: 执行一条独立语句或声明：`innerLaneData);`。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L297 EN**: Comment explains nearby logic, invariants, or intent: `If the source is a static memref, we need to extract the pointer to`.
  **L297 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If the source is a static memref, we need to extract the pointer to`。
- **L298 EN**: Comment explains nearby logic, invariants, or intent: `base address.`.
  **L298 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`base address.`。
- **L299 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L299 CN**: 开始 `if` 控制流语句并计算其条件。
- **L300 EN**: Continues logic associated with callable symbol `create`.
  **L300 CN**: 继续与可调用符号 `create` 相关的逻辑。

### Lines 301-320

````cpp
          rewriter, createNdOp.getLoc(), source);
      source = arith::IndexCastOp::create(rewriter, createNdOp.getLoc(),
                                          rewriter.getI64Type(),
                                          extractOp.getResult())
                   .getResult();
    }
    // Create a new CreateNdDescOp with the modified shape and converted type.
    auto newCreateNdDescOp = xegpu::CreateNdDescOp::create(
        rewriter, createNdOp.getLoc(), convertType, source, modifiedShape,
        modifiedStrides);
    rewriter.replaceOp(createNdOp, newCreateNdDescOp.getResult());
    return success();
  }
};

/// Checks if a LoadNdOp consumes a tensor desc type that was rewritten for
/// tranpose optimization. If so, rewrites the LoadNdOp to to align with the
/// adjusted tensor desc type. This can result in multiple LoadNdOps being
/// generated to fill in the original load shape.
class XeGPULoadNdDescOpPattern final
````
- **L301 EN**: Executes a call or declaration centered on `createNdOp.getLoc`.
  **L301 CN**: 执行以 `createNdOp.getLoc` 为核心的调用或声明。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `source = arith::IndexCastOp::create(rewriter, createNdOp.getLoc(),`.
  **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`source = arith::IndexCastOp::create(rewriter, createNdOp.getLoc(),`。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter.getI64Type(),`.
  **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter.getI64Type(),`。
- **L304 EN**: Continues logic associated with callable symbol `getResult`.
  **L304 CN**: 继续与可调用符号 `getResult` 相关的逻辑。
- **L305 EN**: Executes a call or declaration centered on `.getResult`.
  **L305 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L306 EN**: Closes the current lexical scope or compound statement.
  **L306 CN**: 结束当前词法作用域或复合语句块。
- **L307 EN**: Comment explains nearby logic, invariants, or intent: `Create a new CreateNdDescOp with the modified shape and converted type.`.
  **L307 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create a new CreateNdDescOp with the modified shape and converted type.`。
- **L308 EN**: Continues logic associated with callable symbol `create`.
  **L308 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, createNdOp.getLoc(), convertType, source, modifiedShape,`.
  **L309 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, createNdOp.getLoc(), convertType, source, modifiedShape,`。
- **L310 EN**: Executes a standalone statement or declaration: `modifiedStrides);`.
  **L310 CN**: 执行一条独立语句或声明：`modifiedStrides);`。
- **L311 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L311 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L312 EN**: Returns from the current function with `success()`.
  **L312 CN**: 以 `success()` 从当前函数返回。
- **L313 EN**: Closes the current lexical scope or compound statement.
  **L313 CN**: 结束当前词法作用域或复合语句块。
- **L314 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L314 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L315 EN**: Blank line separating nearby declarations or logic blocks.
  **L315 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L316 EN**: Comment explains nearby logic, invariants, or intent: `Checks if a LoadNdOp consumes a tensor desc type that was rewritten for`.
  **L316 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Checks if a LoadNdOp consumes a tensor desc type that was rewritten for`。
- **L317 EN**: Comment explains nearby logic, invariants, or intent: `tranpose optimization. If so, rewrites the LoadNdOp to to align with the`.
  **L317 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tranpose optimization. If so, rewrites the LoadNdOp to to align with the`。
- **L318 EN**: Comment explains nearby logic, invariants, or intent: `adjusted tensor desc type. This can result in multiple LoadNdOps being`.
  **L318 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`adjusted tensor desc type. This can result in multiple LoadNdOps being`。
- **L319 EN**: Comment explains nearby logic, invariants, or intent: `generated to fill in the original load shape.`.
  **L319 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`generated to fill in the original load shape.`。
- **L320 EN**: Declares class `XeGPULoadNdDescOpPattern`.
  **L320 CN**: 声明 class `XeGPULoadNdDescOpPattern`。

### Lines 321-340

````cpp
    : public OpConversionPattern<xegpu::LoadNdOp> {
public:
  using OpConversionPattern<xegpu::LoadNdOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(xegpu::LoadNdOp loadNdOp, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto origTensorDescType = loadNdOp.getTensorDescType();
    auto adaptorType =
        cast<xegpu::TensorDescType>(adaptor.getTensorDesc().getType());
    if (adaptorType == origTensorDescType)
      return failure();
    // Offsets must be adjusted based on innerLaneData.
    auto laneData = getMaybeLaneData(loadNdOp.getTensorDescType()).value();
    int64_t innerLaneData = laneData[1];
    auto offsets = loadNdOp.getMixedOffsets();
    if (offsets.empty())
      return rewriter.notifyMatchFailure(loadNdOp,
                                         "Expecting offsets in LoadNd");
    SmallVector<OpFoldResult> modifiedOffsets(offsets);
    modifiedOffsets.back() = divideByConstant(
````
- **L321 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<xegpu::LoadNdOp> {`.
  **L321 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<xegpu::LoadNdOp> {`。
- **L322 EN**: Sets the following members to `public` access.
  **L322 CN**: 将后续成员的访问级别设为 `public`。
- **L323 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<xegpu::LoadNdOp>::OpConversionPattern;`.
  **L323 CN**: 执行一条独立语句或声明：`using OpConversionPattern<xegpu::LoadNdOp>::OpConversionPattern;`。
- **L324 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L324 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(xegpu::LoadNdOp loadNdOp, OpAdaptor adaptor,`.
  **L325 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(xegpu::LoadNdOp loadNdOp, OpAdaptor adaptor,`。
- **L326 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L326 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L327 EN**: Initializes variable `origTensorDescType` from the right-hand expression.
  **L327 CN**: 使用右侧表达式初始化变量 `origTensorDescType`。
- **L328 EN**: Continues the surrounding expression or declaration: `auto adaptorType =`.
  **L328 CN**: 继续构造周围的表达式或声明：`auto adaptorType =`。
- **L329 EN**: Executes a call or declaration centered on `cast<xegpu::TensorDescType>`.
  **L329 CN**: 执行以 `cast<xegpu::TensorDescType>` 为核心的调用或声明。
- **L330 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L330 CN**: 开始 `if` 控制流语句并计算其条件。
- **L331 EN**: Returns from the current function with `failure()`.
  **L331 CN**: 以 `failure()` 从当前函数返回。
- **L332 EN**: Comment explains nearby logic, invariants, or intent: `Offsets must be adjusted based on innerLaneData.`.
  **L332 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Offsets must be adjusted based on innerLaneData.`。
- **L333 EN**: Initializes variable `laneData` from the right-hand expression.
  **L333 CN**: 使用右侧表达式初始化变量 `laneData`。
- **L334 EN**: Initializes variable `innerLaneData` from the right-hand expression.
  **L334 CN**: 使用右侧表达式初始化变量 `innerLaneData`。
- **L335 EN**: Initializes variable `offsets` from the right-hand expression.
  **L335 CN**: 使用右侧表达式初始化变量 `offsets`。
- **L336 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L336 CN**: 开始 `if` 控制流语句并计算其条件。
- **L337 EN**: Returns from the current function with `rewriter.notifyMatchFailure(loadNdOp,`.
  **L337 CN**: 以 `rewriter.notifyMatchFailure(loadNdOp,` 从当前函数返回。
- **L338 EN**: Executes a standalone statement or declaration: `"Expecting offsets in LoadNd");`.
  **L338 CN**: 执行一条独立语句或声明：`"Expecting offsets in LoadNd");`。
- **L339 EN**: Executes a call or declaration centered on `modifiedOffsets`.
  **L339 CN**: 执行以 `modifiedOffsets` 为核心的调用或声明。
- **L340 EN**: Continues logic associated with callable symbol `back`.
  **L340 CN**: 继续与可调用符号 `back` 相关的逻辑。

### Lines 341-360

````cpp
        rewriter, loadNdOp.getLoc(),
        convertToValue(rewriter, loadNdOp.getLoc(), modifiedOffsets.back()),
        innerLaneData);
    // Get the 2D data shape of this loadNdOp in its original type including
    // array length.
    SmallVector<int64_t> origDataShape(origTensorDescType.getShape());
    // Adjust the data shape based on innerLaneData.
    origDataShape.back() /= innerLaneData;
    // HW supported shape is the new tensor desc shape after conversion.
    SmallVector<int64_t> hwSupportedShape(adaptorType.getShape());
    VectorType origVectorType =
        VectorType::get(origDataShape, adaptorType.getElementType());
    Value data;
    // Orig data shape is 3D for the array length case.
    if (origTensorDescType.getArrayLength() > 1) {
      SmallVector<Value> arraySlices;
      for (int64_t i = 0; i < origTensorDescType.getArrayLength(); ++i) {
        Value slice = arith::ConstantOp::create(
            rewriter, loadNdOp->getLoc(), origVectorType,
            rewriter.getZeroAttr(origVectorType));
````
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loadNdOp.getLoc(),`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loadNdOp.getLoc(),`。
- **L342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `convertToValue(rewriter, loadNdOp.getLoc(), modifiedOffsets.back()),`.
  **L342 CN**: 继续一个多行参数列表、初始化器或聚合项：`convertToValue(rewriter, loadNdOp.getLoc(), modifiedOffsets.back()),`。
- **L343 EN**: Executes a standalone statement or declaration: `innerLaneData);`.
  **L343 CN**: 执行一条独立语句或声明：`innerLaneData);`。
- **L344 EN**: Comment explains nearby logic, invariants, or intent: `Get the 2D data shape of this loadNdOp in its original type including`.
  **L344 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get the 2D data shape of this loadNdOp in its original type including`。
- **L345 EN**: Comment explains nearby logic, invariants, or intent: `array length.`.
  **L345 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`array length.`。
- **L346 EN**: Executes a call or declaration centered on `origDataShape`.
  **L346 CN**: 执行以 `origDataShape` 为核心的调用或声明。
- **L347 EN**: Comment explains nearby logic, invariants, or intent: `Adjust the data shape based on innerLaneData.`.
  **L347 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Adjust the data shape based on innerLaneData.`。
- **L348 EN**: Executes a call or declaration centered on `origDataShape.back`.
  **L348 CN**: 执行以 `origDataShape.back` 为核心的调用或声明。
- **L349 EN**: Comment explains nearby logic, invariants, or intent: `HW supported shape is the new tensor desc shape after conversion.`.
  **L349 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`HW supported shape is the new tensor desc shape after conversion.`。
- **L350 EN**: Executes a call or declaration centered on `hwSupportedShape`.
  **L350 CN**: 执行以 `hwSupportedShape` 为核心的调用或声明。
- **L351 EN**: Continues the surrounding expression or declaration: `VectorType origVectorType =`.
  **L351 CN**: 继续构造周围的表达式或声明：`VectorType origVectorType =`。
- **L352 EN**: Executes a call or declaration centered on `VectorType::get`.
  **L352 CN**: 执行以 `VectorType::get` 为核心的调用或声明。
- **L353 EN**: Executes a standalone statement or declaration: `Value data;`.
  **L353 CN**: 执行一条独立语句或声明：`Value data;`。
- **L354 EN**: Comment explains nearby logic, invariants, or intent: `Orig data shape is 3D for the array length case.`.
  **L354 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Orig data shape is 3D for the array length case.`。
- **L355 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L355 CN**: 开始 `if` 控制流语句并计算其条件。
- **L356 EN**: Executes a standalone statement or declaration: `SmallVector<Value> arraySlices;`.
  **L356 CN**: 执行一条独立语句或声明：`SmallVector<Value> arraySlices;`。
- **L357 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L357 CN**: 开始 `for` 控制流语句并计算其条件。
- **L358 EN**: Continues logic associated with callable symbol `create`.
  **L358 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loadNdOp->getLoc(), origVectorType,`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loadNdOp->getLoc(), origVectorType,`。
- **L360 EN**: Executes a call or declaration centered on `rewriter.getZeroAttr`.
  **L360 CN**: 执行以 `rewriter.getZeroAttr` 为核心的调用或声明。

### Lines 361-380

````cpp
        // Increase the Y offset for each array slice.
        Value offsetY = convertToValue(rewriter, loadNdOp->getLoc(),
                                       modifiedOffsets.back());
        modifiedOffsets.back() =
            arith::AddIOp::create(
                rewriter, loadNdOp->getLoc(), offsetY,
                arith::ConstantIndexOp::create(rewriter, loadNdOp->getLoc(),
                                               i * origDataShape[1])
                    .getResult())
                .getResult();
        slice = generateLoads(
            rewriter, cast<TypedValue<VectorType>>(slice), modifiedOffsets,
            cast<TypedValue<xegpu::TensorDescType>>(adaptor.getTensorDesc()),
            loadNdOp);
        // BitCast back to original load shape without array length.
        auto bitcastType = VectorType::get(origTensorDescType.getShape(),
                                           origTensorDescType.getElementType());
        auto bitCastOp = vector::BitCastOp::create(rewriter, loadNdOp->getLoc(),
                                                   bitcastType, slice);
        // BitCastOp must have the same layout as the original loadNdOp.
````
- **L361 EN**: Comment explains nearby logic, invariants, or intent: `Increase the Y offset for each array slice.`.
  **L361 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Increase the Y offset for each array slice.`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value offsetY = convertToValue(rewriter, loadNdOp->getLoc(),`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value offsetY = convertToValue(rewriter, loadNdOp->getLoc(),`。
- **L363 EN**: Executes a call or declaration centered on `modifiedOffsets.back`.
  **L363 CN**: 执行以 `modifiedOffsets.back` 为核心的调用或声明。
- **L364 EN**: Continues logic associated with callable symbol `back`.
  **L364 CN**: 继续与可调用符号 `back` 相关的逻辑。
- **L365 EN**: Continues logic associated with callable symbol `create`.
  **L365 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loadNdOp->getLoc(), offsetY,`.
  **L366 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loadNdOp->getLoc(), offsetY,`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arith::ConstantIndexOp::create(rewriter, loadNdOp->getLoc(),`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`arith::ConstantIndexOp::create(rewriter, loadNdOp->getLoc(),`。
- **L368 EN**: Continues the surrounding expression or declaration: `i * origDataShape[1])`.
  **L368 CN**: 继续构造周围的表达式或声明：`i * origDataShape[1])`。
- **L369 EN**: Continues logic associated with callable symbol `getResult`.
  **L369 CN**: 继续与可调用符号 `getResult` 相关的逻辑。
- **L370 EN**: Executes a call or declaration centered on `.getResult`.
  **L370 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L371 EN**: Continues logic associated with callable symbol `generateLoads`.
  **L371 CN**: 继续与可调用符号 `generateLoads` 相关的逻辑。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, cast<TypedValue<VectorType>>(slice), modifiedOffsets,`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, cast<TypedValue<VectorType>>(slice), modifiedOffsets,`。
- **L373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `cast<TypedValue<xegpu::TensorDescType>>(adaptor.getTensorDesc()),`.
  **L373 CN**: 继续一个多行参数列表、初始化器或聚合项：`cast<TypedValue<xegpu::TensorDescType>>(adaptor.getTensorDesc()),`。
- **L374 EN**: Executes a standalone statement or declaration: `loadNdOp);`.
  **L374 CN**: 执行一条独立语句或声明：`loadNdOp);`。
- **L375 EN**: Comment explains nearby logic, invariants, or intent: `BitCast back to original load shape without array length.`.
  **L375 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BitCast back to original load shape without array length.`。
- **L376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto bitcastType = VectorType::get(origTensorDescType.getShape(),`.
  **L376 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto bitcastType = VectorType::get(origTensorDescType.getShape(),`。
- **L377 EN**: Executes a call or declaration centered on `origTensorDescType.getElementType`.
  **L377 CN**: 执行以 `origTensorDescType.getElementType` 为核心的调用或声明。
- **L378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto bitCastOp = vector::BitCastOp::create(rewriter, loadNdOp->getLoc(),`.
  **L378 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto bitCastOp = vector::BitCastOp::create(rewriter, loadNdOp->getLoc(),`。
- **L379 EN**: Executes a standalone statement or declaration: `bitcastType, slice);`.
  **L379 CN**: 执行一条独立语句或声明：`bitcastType, slice);`。
- **L380 EN**: Comment explains nearby logic, invariants, or intent: `BitCastOp must have the same layout as the original loadNdOp.`.
  **L380 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BitCastOp must have the same layout as the original loadNdOp.`。

### Lines 381-400

````cpp
        xegpu::setTemporaryLayout(bitCastOp->getOpResult(0),
                                  origTensorDescType.getLayoutAttr());
        arraySlices.push_back(bitCastOp.getResult());
      }
      rewriter.replaceOpWithMultiple(loadNdOp, {arraySlices});
      return success();
    }
    data = arith::ConstantOp::create(
        rewriter, loadNdOp->getLoc(),
        VectorType::get(origDataShape, adaptorType.getElementType()),
        rewriter.getZeroAttr(origVectorType));
    data = generateLoads(
        rewriter, cast<TypedValue<VectorType>>(data), modifiedOffsets,
        cast<TypedValue<xegpu::TensorDescType>>(adaptor.getTensorDesc()),
        loadNdOp);
    auto bitCastOp = vector::BitCastOp::create(rewriter, loadNdOp->getLoc(),
                                               loadNdOp.getType(), data);
    // BitCastOp must have the same layout as the original loadNdOp.
    xegpu::setTemporaryLayout(bitCastOp->getOpResult(0),
                              origTensorDescType.getLayoutAttr());
````
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::setTemporaryLayout(bitCastOp->getOpResult(0),`.
  **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::setTemporaryLayout(bitCastOp->getOpResult(0),`。
- **L382 EN**: Executes a call or declaration centered on `origTensorDescType.getLayoutAttr`.
  **L382 CN**: 执行以 `origTensorDescType.getLayoutAttr` 为核心的调用或声明。
- **L383 EN**: Executes a call or declaration centered on `arraySlices.push_back`.
  **L383 CN**: 执行以 `arraySlices.push_back` 为核心的调用或声明。
- **L384 EN**: Closes the current lexical scope or compound statement.
  **L384 CN**: 结束当前词法作用域或复合语句块。
- **L385 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithMultiple`.
  **L385 CN**: 执行以 `rewriter.replaceOpWithMultiple` 为核心的调用或声明。
- **L386 EN**: Returns from the current function with `success()`.
  **L386 CN**: 以 `success()` 从当前函数返回。
- **L387 EN**: Closes the current lexical scope or compound statement.
  **L387 CN**: 结束当前词法作用域或复合语句块。
- **L388 EN**: Continues logic associated with callable symbol `create`.
  **L388 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loadNdOp->getLoc(),`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loadNdOp->getLoc(),`。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VectorType::get(origDataShape, adaptorType.getElementType()),`.
  **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`VectorType::get(origDataShape, adaptorType.getElementType()),`。
- **L391 EN**: Executes a call or declaration centered on `rewriter.getZeroAttr`.
  **L391 CN**: 执行以 `rewriter.getZeroAttr` 为核心的调用或声明。
- **L392 EN**: Continues logic associated with callable symbol `generateLoads`.
  **L392 CN**: 继续与可调用符号 `generateLoads` 相关的逻辑。
- **L393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, cast<TypedValue<VectorType>>(data), modifiedOffsets,`.
  **L393 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, cast<TypedValue<VectorType>>(data), modifiedOffsets,`。
- **L394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `cast<TypedValue<xegpu::TensorDescType>>(adaptor.getTensorDesc()),`.
  **L394 CN**: 继续一个多行参数列表、初始化器或聚合项：`cast<TypedValue<xegpu::TensorDescType>>(adaptor.getTensorDesc()),`。
- **L395 EN**: Executes a standalone statement or declaration: `loadNdOp);`.
  **L395 CN**: 执行一条独立语句或声明：`loadNdOp);`。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto bitCastOp = vector::BitCastOp::create(rewriter, loadNdOp->getLoc(),`.
  **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto bitCastOp = vector::BitCastOp::create(rewriter, loadNdOp->getLoc(),`。
- **L397 EN**: Executes a call or declaration centered on `loadNdOp.getType`.
  **L397 CN**: 执行以 `loadNdOp.getType` 为核心的调用或声明。
- **L398 EN**: Comment explains nearby logic, invariants, or intent: `BitCastOp must have the same layout as the original loadNdOp.`.
  **L398 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BitCastOp must have the same layout as the original loadNdOp.`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `xegpu::setTemporaryLayout(bitCastOp->getOpResult(0),`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`xegpu::setTemporaryLayout(bitCastOp->getOpResult(0),`。
- **L400 EN**: Executes a call or declaration centered on `origTensorDescType.getLayoutAttr`.
  **L400 CN**: 执行以 `origTensorDescType.getLayoutAttr` 为核心的调用或声明。

### Lines 401-420

````cpp
    rewriter.replaceOp(loadNdOp, bitCastOp);
    return success();
  }
};

/// Vector ExtractOp must be processed if the original tensor desc type has
/// array length greater than 1. In this case, the LoadNdOp is replaced with
/// multiple LoadNdOps for each array slice making the extraction unnecessary.
/// In this case, we simply remove the ExtractOp.
class VectorExtractOpPattern final
    : public OpConversionPattern<vector::ExtractOp> {
public:
  using OpConversionPattern<vector::ExtractOp>::OpConversionPattern;
  LogicalResult
  matchAndRewrite(vector::ExtractOp extractOp, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    // Check if the source of the extraction is split to multiple values.
    if (adaptor.getSource().size() == 1)
      return failure();
    auto mixedPos = extractOp.getMixedPosition();
````
- **L401 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L401 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L402 EN**: Returns from the current function with `success()`.
  **L402 CN**: 以 `success()` 从当前函数返回。
- **L403 EN**: Closes the current lexical scope or compound statement.
  **L403 CN**: 结束当前词法作用域或复合语句块。
- **L404 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L404 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L405 EN**: Blank line separating nearby declarations or logic blocks.
  **L405 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L406 EN**: Comment explains nearby logic, invariants, or intent: `Vector ExtractOp must be processed if the original tensor desc type has`.
  **L406 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector ExtractOp must be processed if the original tensor desc type has`。
- **L407 EN**: Comment explains nearby logic, invariants, or intent: `array length greater than 1. In this case, the LoadNdOp is replaced with`.
  **L407 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`array length greater than 1. In this case, the LoadNdOp is replaced with`。
- **L408 EN**: Comment explains nearby logic, invariants, or intent: `multiple LoadNdOps for each array slice making the extraction unnecessary.`.
  **L408 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`multiple LoadNdOps for each array slice making the extraction unnecessary.`。
- **L409 EN**: Comment explains nearby logic, invariants, or intent: `In this case, we simply remove the ExtractOp.`.
  **L409 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`In this case, we simply remove the ExtractOp.`。
- **L410 EN**: Declares class `VectorExtractOpPattern`.
  **L410 CN**: 声明 class `VectorExtractOpPattern`。
- **L411 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<vector::ExtractOp> {`.
  **L411 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<vector::ExtractOp> {`。
- **L412 EN**: Sets the following members to `public` access.
  **L412 CN**: 将后续成员的访问级别设为 `public`。
- **L413 EN**: Executes a standalone statement or declaration: `using OpConversionPattern<vector::ExtractOp>::OpConversionPattern;`.
  **L413 CN**: 执行一条独立语句或声明：`using OpConversionPattern<vector::ExtractOp>::OpConversionPattern;`。
- **L414 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L414 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(vector::ExtractOp extractOp, OneToNOpAdaptor adaptor,`.
  **L415 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(vector::ExtractOp extractOp, OneToNOpAdaptor adaptor,`。
- **L416 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L416 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L417 EN**: Comment explains nearby logic, invariants, or intent: `Check if the source of the extraction is split to multiple values.`.
  **L417 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check if the source of the extraction is split to multiple values.`。
- **L418 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L418 CN**: 开始 `if` 控制流语句并计算其条件。
- **L419 EN**: Returns from the current function with `failure()`.
  **L419 CN**: 以 `failure()` 从当前函数返回。
- **L420 EN**: Initializes variable `mixedPos` from the right-hand expression.
  **L420 CN**: 使用右侧表达式初始化变量 `mixedPos`。

### Lines 421-440

````cpp
    if (mixedPos.size() != 1)
      return failure();
    auto mayBeInt = getConstantIntValue(mixedPos[0]);
    if (!mayBeInt)
      return failure();
    rewriter.replaceOp(extractOp, adaptor.getSource()[*mayBeInt]);
    return success();
  }
};

/// Performs a reduction over 2 dimensions by decomposing it into two 1D
/// reductions ordered based on layout to minimize cross-lane communication.
class MultiRed2dOpPattern
    : public OpConversionPattern<vector::MultiDimReductionOp> {
  using OpConversionPattern::OpConversionPattern;
  LogicalResult
  matchAndRewrite(vector::MultiDimReductionOp reductionOp, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    auto sourceVecType = reductionOp.getSourceVectorType();
    if (reductionOp.getReductionDims().size() != 2)
````
- **L421 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L421 CN**: 开始 `if` 控制流语句并计算其条件。
- **L422 EN**: Returns from the current function with `failure()`.
  **L422 CN**: 以 `failure()` 从当前函数返回。
- **L423 EN**: Initializes variable `mayBeInt` from the right-hand expression.
  **L423 CN**: 使用右侧表达式初始化变量 `mayBeInt`。
- **L424 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L424 CN**: 开始 `if` 控制流语句并计算其条件。
- **L425 EN**: Returns from the current function with `failure()`.
  **L425 CN**: 以 `failure()` 从当前函数返回。
- **L426 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L426 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L427 EN**: Returns from the current function with `success()`.
  **L427 CN**: 以 `success()` 从当前函数返回。
- **L428 EN**: Closes the current lexical scope or compound statement.
  **L428 CN**: 结束当前词法作用域或复合语句块。
- **L429 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L429 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L430 EN**: Blank line separating nearby declarations or logic blocks.
  **L430 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L431 EN**: Comment explains nearby logic, invariants, or intent: `Performs a reduction over 2 dimensions by decomposing it into two 1D`.
  **L431 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Performs a reduction over 2 dimensions by decomposing it into two 1D`。
- **L432 EN**: Comment explains nearby logic, invariants, or intent: `reductions ordered based on layout to minimize cross-lane communication.`.
  **L432 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`reductions ordered based on layout to minimize cross-lane communication.`。
- **L433 EN**: Declares class `MultiRed2dOpPattern`.
  **L433 CN**: 声明 class `MultiRed2dOpPattern`。
- **L434 EN**: Continues the surrounding expression or declaration: `: public OpConversionPattern<vector::MultiDimReductionOp> {`.
  **L434 CN**: 继续构造周围的表达式或声明：`: public OpConversionPattern<vector::MultiDimReductionOp> {`。
- **L435 EN**: Executes a standalone statement or declaration: `using OpConversionPattern::OpConversionPattern;`.
  **L435 CN**: 执行一条独立语句或声明：`using OpConversionPattern::OpConversionPattern;`。
- **L436 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L436 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(vector::MultiDimReductionOp reductionOp, OpAdaptor adaptor,`.
  **L437 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(vector::MultiDimReductionOp reductionOp, OpAdaptor adaptor,`。
- **L438 EN**: Continues the surrounding expression or declaration: `ConversionPatternRewriter &rewriter) const override {`.
  **L438 CN**: 继续构造周围的表达式或声明：`ConversionPatternRewriter &rewriter) const override {`。
- **L439 EN**: Initializes variable `sourceVecType` from the right-hand expression.
  **L439 CN**: 使用右侧表达式初始化变量 `sourceVecType`。
- **L440 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L440 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 441-460

````cpp
      return rewriter.notifyMatchFailure(reductionOp, "Expected 2D reduction");
    auto resLayout = xegpu::getDistributeLayoutAttr(reductionOp.getResult());
    // Retrieve and order dims for 1D decomposition (prefer intra-lane first).
    auto dims = llvm::to_vector(reductionOp.getReductionDims());
    auto [intraLaneDim, crossLaneDim] = getReductionDimOrder(dims, resLayout);
    // Order does not matter
    if (intraLaneDim == -1 || crossLaneDim == -1) {
      intraLaneDim = dims[0];
      crossLaneDim = dims[1];
    }
    auto loc = reductionOp.getLoc();
    auto acc = reductionOp.getAcc();

    // The decomposition below splits the 2D reduction into an intra-lane
    // then a cross-lane 1D reduction. The natural result layout of the
    // decomposed sequence (a doubly-sliced layout) differs from the
    // original 2D reduction's result layout that the rest of the IR was
    // written/propagated against. To keep the post-peephole IR
    // self-consistent without depending on a follow-up layout
    // propagation pass, we always insert a bridge xegpu.convert_layout
````
- **L441 EN**: Returns from the current function with `rewriter.notifyMatchFailure(reductionOp, "Expected 2D reduction")`.
  **L441 CN**: 以 `rewriter.notifyMatchFailure(reductionOp, "Expected 2D reduction")` 从当前函数返回。
- **L442 EN**: Initializes variable `resLayout` from the right-hand expression.
  **L442 CN**: 使用右侧表达式初始化变量 `resLayout`。
- **L443 EN**: Comment explains nearby logic, invariants, or intent: `Retrieve and order dims for 1D decomposition (prefer intra-lane first).`.
  **L443 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Retrieve and order dims for 1D decomposition (prefer intra-lane first).`。
- **L444 EN**: Initializes variable `dims` from the right-hand expression.
  **L444 CN**: 使用右侧表达式初始化变量 `dims`。
- **L445 EN**: Executes a call or declaration centered on `getReductionDimOrder`.
  **L445 CN**: 执行以 `getReductionDimOrder` 为核心的调用或声明。
- **L446 EN**: Comment explains nearby logic, invariants, or intent: `Order does not matter`.
  **L446 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Order does not matter`。
- **L447 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L447 CN**: 开始 `if` 控制流语句并计算其条件。
- **L448 EN**: Executes a standalone statement or declaration: `intraLaneDim = dims[0];`.
  **L448 CN**: 执行一条独立语句或声明：`intraLaneDim = dims[0];`。
- **L449 EN**: Executes a standalone statement or declaration: `crossLaneDim = dims[1];`.
  **L449 CN**: 执行一条独立语句或声明：`crossLaneDim = dims[1];`。
- **L450 EN**: Closes the current lexical scope or compound statement.
  **L450 CN**: 结束当前词法作用域或复合语句块。
- **L451 EN**: Initializes variable `loc` from the right-hand expression.
  **L451 CN**: 使用右侧表达式初始化变量 `loc`。
- **L452 EN**: Initializes variable `acc` from the right-hand expression.
  **L452 CN**: 使用右侧表达式初始化变量 `acc`。
- **L453 EN**: Blank line separating nearby declarations or logic blocks.
  **L453 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L454 EN**: Comment explains nearby logic, invariants, or intent: `The decomposition below splits the 2D reduction into an intra-lane`.
  **L454 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The decomposition below splits the 2D reduction into an intra-lane`。
- **L455 EN**: Comment explains nearby logic, invariants, or intent: `then a cross-lane 1D reduction. The natural result layout of the`.
  **L455 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`then a cross-lane 1D reduction. The natural result layout of the`。
- **L456 EN**: Comment explains nearby logic, invariants, or intent: `decomposed sequence (a doubly-sliced layout) differs from the`.
  **L456 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`decomposed sequence (a doubly-sliced layout) differs from the`。
- **L457 EN**: Comment explains nearby logic, invariants, or intent: `original 2D reduction's result layout that the rest of the IR was`.
  **L457 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`original 2D reduction's result layout that the rest of the IR was`。
- **L458 EN**: Comment explains nearby logic, invariants, or intent: `written/propagated against. To keep the post-peephole IR`.
  **L458 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`written/propagated against. To keep the post-peephole IR`。
- **L459 EN**: Comment explains nearby logic, invariants, or intent: `self-consistent without depending on a follow-up layout`.
  **L459 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`self-consistent without depending on a follow-up layout`。
- **L460 EN**: Comment explains nearby logic, invariants, or intent: `propagation pass, we always insert a bridge xegpu.convert_layout`.
  **L460 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`propagation pass, we always insert a bridge xegpu.convert_layout`。

### Lines 461-480

````cpp
    // from the natural post-decomposition layout to the original
    // reduction's result layout. Trivial bridges fold away in
    // canonicalization.
    xegpu::DistributeLayoutAttr postDecompLayout;
    if (resLayout) {
      // Derive the source vector's layout.
      xegpu::DistributeLayoutAttr srcLayoutForCvt;
      if (auto resSlice = dyn_cast_if_present<xegpu::SliceAttr>(resLayout))
        srcLayoutForCvt = resSlice.getParent();
      if (!srcLayoutForCvt)
        srcLayoutForCvt =
            xegpu::getDistributeLayoutAttr(reductionOp.getSource());
      if (srcLayoutForCvt) {
        // The natural layout of the post-decomposition reduction result
        // is a nested SliceAttr: REDUCE_1 (reduces `intraLaneDim` from
        // the source) yields `slice<src, [intraLaneDim]>`; REDUCE_2
        // then reduces `adjCrossLaneDim` from that intermediate, giving
        // `slice<slice<src, [intraLaneDim]>, [adjCrossLaneDim]>`.
        MLIRContext *ctx = reductionOp.getContext();
        int64_t adjCrossLaneDim =
````
- **L461 EN**: Comment explains nearby logic, invariants, or intent: `from the natural post-decomposition layout to the original`.
  **L461 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`from the natural post-decomposition layout to the original`。
- **L462 EN**: Comment explains nearby logic, invariants, or intent: `reduction's result layout. Trivial bridges fold away in`.
  **L462 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`reduction's result layout. Trivial bridges fold away in`。
- **L463 EN**: Comment explains nearby logic, invariants, or intent: `canonicalization.`.
  **L463 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`canonicalization.`。
- **L464 EN**: Executes a standalone statement or declaration: `xegpu::DistributeLayoutAttr postDecompLayout;`.
  **L464 CN**: 执行一条独立语句或声明：`xegpu::DistributeLayoutAttr postDecompLayout;`。
- **L465 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L465 CN**: 开始 `if` 控制流语句并计算其条件。
- **L466 EN**: Comment explains nearby logic, invariants, or intent: `Derive the source vector's layout.`.
  **L466 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Derive the source vector's layout.`。
- **L467 EN**: Executes a standalone statement or declaration: `xegpu::DistributeLayoutAttr srcLayoutForCvt;`.
  **L467 CN**: 执行一条独立语句或声明：`xegpu::DistributeLayoutAttr srcLayoutForCvt;`。
- **L468 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L468 CN**: 开始 `if` 控制流语句并计算其条件。
- **L469 EN**: Executes a call or declaration centered on `resSlice.getParent`.
  **L469 CN**: 执行以 `resSlice.getParent` 为核心的调用或声明。
- **L470 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L470 CN**: 开始 `if` 控制流语句并计算其条件。
- **L471 EN**: Continues the surrounding expression or declaration: `srcLayoutForCvt =`.
  **L471 CN**: 继续构造周围的表达式或声明：`srcLayoutForCvt =`。
- **L472 EN**: Executes a call or declaration centered on `xegpu::getDistributeLayoutAttr`.
  **L472 CN**: 执行以 `xegpu::getDistributeLayoutAttr` 为核心的调用或声明。
- **L473 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L473 CN**: 开始 `if` 控制流语句并计算其条件。
- **L474 EN**: Comment explains nearby logic, invariants, or intent: `The natural layout of the post-decomposition reduction result`.
  **L474 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The natural layout of the post-decomposition reduction result`。
- **L475 EN**: Comment explains nearby logic, invariants, or intent: `is a nested SliceAttr: REDUCE_1 (reduces `intraLaneDim` from`.
  **L475 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`is a nested SliceAttr: REDUCE_1 (reduces `intraLaneDim` from`。
- **L476 EN**: Comment explains nearby logic, invariants, or intent: `the source) yields `slice<src, [intraLaneDim]>`; REDUCE_2`.
  **L476 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the source) yields `slice<src, [intraLaneDim]>`; REDUCE_2`。
- **L477 EN**: Comment explains nearby logic, invariants, or intent: `then reduces `adjCrossLaneDim` from that intermediate, giving`.
  **L477 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`then reduces `adjCrossLaneDim` from that intermediate, giving`。
- **L478 EN**: Comment explains nearby logic, invariants, or intent: ``slice<slice<src, [intraLaneDim]>, [adjCrossLaneDim]>`.`.
  **L478 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：``slice<slice<src, [intraLaneDim]>, [adjCrossLaneDim]>`.`。
- **L479 EN**: Executes a call or declaration centered on `reductionOp.getContext`.
  **L479 CN**: 执行以 `reductionOp.getContext` 为核心的调用或声明。
- **L480 EN**: Continues the surrounding expression or declaration: `int64_t adjCrossLaneDim =`.
  **L480 CN**: 继续构造周围的表达式或声明：`int64_t adjCrossLaneDim =`。

### Lines 481-500

````cpp
            crossLaneDim > intraLaneDim ? crossLaneDim - 1 : crossLaneDim;
        auto intermediateLayout = xegpu::SliceAttr::get(
            ctx, srcLayoutForCvt, DenseI64ArrayAttr::get(ctx, {intraLaneDim}));
        postDecompLayout = xegpu::SliceAttr::get(
            ctx, intermediateLayout,
            DenseI64ArrayAttr::get(ctx, {adjCrossLaneDim}));
      }
    }

    SmallVector<int64_t> accShape(sourceVecType.getShape());
    accShape.erase(accShape.begin() + intraLaneDim);
    Type eTy = sourceVecType.getElementType();
    Value constNeutralVal = xegpu::createReductionNeutralValue(
        rewriter, loc, VectorType::get(accShape, eTy), reductionOp.getKind());

    Value intraLaneReduced = vector::MultiDimReductionOp::create(
        rewriter, loc, reductionOp.getKind(), reductionOp.getSource(),
        constNeutralVal, ArrayRef<int64_t>(intraLaneDim));

    // Adjust crossLaneDim after the first reduction.
````
- **L481 EN**: Executes a standalone statement or declaration: `crossLaneDim > intraLaneDim ? crossLaneDim - 1 : crossLaneDim;`.
  **L481 CN**: 执行一条独立语句或声明：`crossLaneDim > intraLaneDim ? crossLaneDim - 1 : crossLaneDim;`。
- **L482 EN**: Continues logic associated with callable symbol `get`.
  **L482 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L483 EN**: Executes a call or declaration centered on `DenseI64ArrayAttr::get`.
  **L483 CN**: 执行以 `DenseI64ArrayAttr::get` 为核心的调用或声明。
- **L484 EN**: Continues logic associated with callable symbol `get`.
  **L484 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ctx, intermediateLayout,`.
  **L485 CN**: 继续一个多行参数列表、初始化器或聚合项：`ctx, intermediateLayout,`。
- **L486 EN**: Executes a call or declaration centered on `DenseI64ArrayAttr::get`.
  **L486 CN**: 执行以 `DenseI64ArrayAttr::get` 为核心的调用或声明。
- **L487 EN**: Closes the current lexical scope or compound statement.
  **L487 CN**: 结束当前词法作用域或复合语句块。
- **L488 EN**: Closes the current lexical scope or compound statement.
  **L488 CN**: 结束当前词法作用域或复合语句块。
- **L489 EN**: Blank line separating nearby declarations or logic blocks.
  **L489 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L490 EN**: Executes a call or declaration centered on `accShape`.
  **L490 CN**: 执行以 `accShape` 为核心的调用或声明。
- **L491 EN**: Executes a call or declaration centered on `accShape.erase`.
  **L491 CN**: 执行以 `accShape.erase` 为核心的调用或声明。
- **L492 EN**: Initializes variable `eTy` from the right-hand expression.
  **L492 CN**: 使用右侧表达式初始化变量 `eTy`。
- **L493 EN**: Continues logic associated with callable symbol `createReductionNeutralValue`.
  **L493 CN**: 继续与可调用符号 `createReductionNeutralValue` 相关的逻辑。
- **L494 EN**: Executes a call or declaration centered on `VectorType::get`.
  **L494 CN**: 执行以 `VectorType::get` 为核心的调用或声明。
- **L495 EN**: Blank line separating nearby declarations or logic blocks.
  **L495 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L496 EN**: Continues logic associated with callable symbol `create`.
  **L496 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, reductionOp.getKind(), reductionOp.getSource(),`.
  **L497 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, reductionOp.getKind(), reductionOp.getSource(),`。
- **L498 EN**: Executes a call or declaration centered on `ArrayRef<int64_t>`.
  **L498 CN**: 执行以 `ArrayRef<int64_t>` 为核心的调用或声明。
- **L499 EN**: Blank line separating nearby declarations or logic blocks.
  **L499 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L500 EN**: Comment explains nearby logic, invariants, or intent: `Adjust crossLaneDim after the first reduction.`.
  **L500 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Adjust crossLaneDim after the first reduction.`。

### Lines 501-520

````cpp
    if (crossLaneDim > intraLaneDim)
      crossLaneDim -= 1;
    Value crossLaneReduced = vector::MultiDimReductionOp::create(
        rewriter, loc, reductionOp.getKind(), intraLaneReduced, acc,
        ArrayRef<int64_t>(crossLaneDim));
    assert(crossLaneReduced.getType() == reductionOp.getResult().getType() &&
           "Type mismatch");

    Value replacement = crossLaneReduced;
    if (resLayout && postDecompLayout) {
      // Bridge from the natural post-decomposition layout to the
      // original reduction's result layout. This preserves the contract
      // any consumer (convert_layout, anchor op, or otherwise) was
      // written against, so the rewrite is correct independent of
      // whether layout propagation runs afterwards.
      auto bridgeOp = xegpu::ConvertLayoutOp::create(
          rewriter, loc, crossLaneReduced.getType(), crossLaneReduced,
          postDecompLayout, resLayout);
      replacement = bridgeOp.getResult();
    }
````
- **L501 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L501 CN**: 开始 `if` 控制流语句并计算其条件。
- **L502 EN**: Executes a standalone statement or declaration: `crossLaneDim -= 1;`.
  **L502 CN**: 执行一条独立语句或声明：`crossLaneDim -= 1;`。
- **L503 EN**: Continues logic associated with callable symbol `create`.
  **L503 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, reductionOp.getKind(), intraLaneReduced, acc,`.
  **L504 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, reductionOp.getKind(), intraLaneReduced, acc,`。
- **L505 EN**: Executes a call or declaration centered on `ArrayRef<int64_t>`.
  **L505 CN**: 执行以 `ArrayRef<int64_t>` 为核心的调用或声明。
- **L506 EN**: Checks an internal invariant in debug builds.
  **L506 CN**: 在调试构建中检查内部不变式。
- **L507 EN**: Executes a standalone statement or declaration: `"Type mismatch");`.
  **L507 CN**: 执行一条独立语句或声明：`"Type mismatch");`。
- **L508 EN**: Blank line separating nearby declarations or logic blocks.
  **L508 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L509 EN**: Initializes variable `replacement` from the right-hand expression.
  **L509 CN**: 使用右侧表达式初始化变量 `replacement`。
- **L510 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L510 CN**: 开始 `if` 控制流语句并计算其条件。
- **L511 EN**: Comment explains nearby logic, invariants, or intent: `Bridge from the natural post-decomposition layout to the`.
  **L511 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Bridge from the natural post-decomposition layout to the`。
- **L512 EN**: Comment explains nearby logic, invariants, or intent: `original reduction's result layout. This preserves the contract`.
  **L512 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`original reduction's result layout. This preserves the contract`。
- **L513 EN**: Comment explains nearby logic, invariants, or intent: `any consumer (convert_layout, anchor op, or otherwise) was`.
  **L513 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`any consumer (convert_layout, anchor op, or otherwise) was`。
- **L514 EN**: Comment explains nearby logic, invariants, or intent: `written against, so the rewrite is correct independent of`.
  **L514 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`written against, so the rewrite is correct independent of`。
- **L515 EN**: Comment explains nearby logic, invariants, or intent: `whether layout propagation runs afterwards.`.
  **L515 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`whether layout propagation runs afterwards.`。
- **L516 EN**: Continues logic associated with callable symbol `create`.
  **L516 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, crossLaneReduced.getType(), crossLaneReduced,`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, crossLaneReduced.getType(), crossLaneReduced,`。
- **L518 EN**: Executes a standalone statement or declaration: `postDecompLayout, resLayout);`.
  **L518 CN**: 执行一条独立语句或声明：`postDecompLayout, resLayout);`。
- **L519 EN**: Executes a call or declaration centered on `bridgeOp.getResult`.
  **L519 CN**: 执行以 `bridgeOp.getResult` 为核心的调用或声明。
- **L520 EN**: Closes the current lexical scope or compound statement.
  **L520 CN**: 结束当前词法作用域或复合语句块。

### Lines 521-540

````cpp

    rewriter.replaceOp(reductionOp, replacement);
    return success();
  }

private:
  std::pair<int64_t, int64_t>
  getReductionDimOrder(ArrayRef<int64_t> reductionDims,
                       xegpu::DistributeLayoutAttr layout) const {
    assert(layout.isForSubgroup() && "Must know the lane layout");
    assert(reductionDims.size() == 2 && "Expected 2D reduction");
    int64_t intra, cross = -1;
    xegpu::LayoutAttr layoutAttr = dyn_cast<xegpu::LayoutAttr>(layout);
    if (auto layoutSliceAttr = dyn_cast<xegpu::SliceAttr>(layout))
      layoutAttr =
          dyn_cast<xegpu::LayoutAttr>(layoutSliceAttr.flatten().getParent());
    assert(layoutAttr);
    SmallVector<int64_t> laneLayout = layoutAttr.getEffectiveLaneLayoutAsInt();

    assert(laneLayout.size() && "Expected a non-empty layout");
````
- **L521 EN**: Blank line separating nearby declarations or logic blocks.
  **L521 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L522 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L522 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L523 EN**: Returns from the current function with `success()`.
  **L523 CN**: 以 `success()` 从当前函数返回。
- **L524 EN**: Closes the current lexical scope or compound statement.
  **L524 CN**: 结束当前词法作用域或复合语句块。
- **L525 EN**: Blank line separating nearby declarations or logic blocks.
  **L525 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L526 EN**: Sets the following members to `private` access.
  **L526 CN**: 将后续成员的访问级别设为 `private`。
- **L527 EN**: Continues the surrounding expression or declaration: `std::pair<int64_t, int64_t>`.
  **L527 CN**: 继续构造周围的表达式或声明：`std::pair<int64_t, int64_t>`。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getReductionDimOrder(ArrayRef<int64_t> reductionDims,`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`getReductionDimOrder(ArrayRef<int64_t> reductionDims,`。
- **L529 EN**: Continues the surrounding expression or declaration: `xegpu::DistributeLayoutAttr layout) const {`.
  **L529 CN**: 继续构造周围的表达式或声明：`xegpu::DistributeLayoutAttr layout) const {`。
- **L530 EN**: Checks an internal invariant in debug builds.
  **L530 CN**: 在调试构建中检查内部不变式。
- **L531 EN**: Checks an internal invariant in debug builds.
  **L531 CN**: 在调试构建中检查内部不变式。
- **L532 EN**: Initializes variable `cross` from the right-hand expression.
  **L532 CN**: 使用右侧表达式初始化变量 `cross`。
- **L533 EN**: Initializes variable `layoutAttr` from the right-hand expression.
  **L533 CN**: 使用右侧表达式初始化变量 `layoutAttr`。
- **L534 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L534 CN**: 开始 `if` 控制流语句并计算其条件。
- **L535 EN**: Continues the surrounding expression or declaration: `layoutAttr =`.
  **L535 CN**: 继续构造周围的表达式或声明：`layoutAttr =`。
- **L536 EN**: Executes a call or declaration centered on `dyn_cast<xegpu::LayoutAttr>`.
  **L536 CN**: 执行以 `dyn_cast<xegpu::LayoutAttr>` 为核心的调用或声明。
- **L537 EN**: Checks an internal invariant in debug builds.
  **L537 CN**: 在调试构建中检查内部不变式。
- **L538 EN**: Initializes variable `laneLayout` from the right-hand expression.
  **L538 CN**: 使用右侧表达式初始化变量 `laneLayout`。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L540 EN**: Checks an internal invariant in debug builds.
  **L540 CN**: 在调试构建中检查内部不变式。

### Lines 541-560

````cpp
    // try to pick a dim that does not communicate
    for (auto dim : reductionDims) {
      if (laneLayout[dim] == 1)
        intra = dim;
      else
        cross = dim;
    }
    return {intra, cross};
  }
};

} // namespace

void xegpu::populateXeGPUPeepHoleOptimizerPatterns(
    RewritePatternSet &patterns) {
  patterns.add<XeGPUCreateNdDescOpPattern, XeGPULoadNdDescOpPattern,
               VectorExtractOpPattern, MultiRed2dOpPattern>(
      patterns.getContext());
}

````
- **L541 EN**: Comment explains nearby logic, invariants, or intent: `try to pick a dim that does not communicate`.
  **L541 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`try to pick a dim that does not communicate`。
- **L542 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L542 CN**: 开始 `for` 控制流语句并计算其条件。
- **L543 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L543 CN**: 开始 `if` 控制流语句并计算其条件。
- **L544 EN**: Executes a standalone statement or declaration: `intra = dim;`.
  **L544 CN**: 执行一条独立语句或声明：`intra = dim;`。
- **L545 EN**: Starts the alternative branch of the preceding conditional.
  **L545 CN**: 开始前一个条件语句的备选分支。
- **L546 EN**: Executes a standalone statement or declaration: `cross = dim;`.
  **L546 CN**: 执行一条独立语句或声明：`cross = dim;`。
- **L547 EN**: Closes the current lexical scope or compound statement.
  **L547 CN**: 结束当前词法作用域或复合语句块。
- **L548 EN**: Returns from the current function with `{intra, cross}`.
  **L548 CN**: 以 `{intra, cross}` 从当前函数返回。
- **L549 EN**: Closes the current lexical scope or compound statement.
  **L549 CN**: 结束当前词法作用域或复合语句块。
- **L550 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L550 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L551 EN**: Blank line separating nearby declarations or logic blocks.
  **L551 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L552 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L552 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L553 EN**: Blank line separating nearby declarations or logic blocks.
  **L553 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L554 EN**: Continues logic associated with callable symbol `populateXeGPUPeepHoleOptimizerPatterns`.
  **L554 CN**: 继续与可调用符号 `populateXeGPUPeepHoleOptimizerPatterns` 相关的逻辑。
- **L555 EN**: Continues the surrounding expression or declaration: `RewritePatternSet &patterns) {`.
  **L555 CN**: 继续构造周围的表达式或声明：`RewritePatternSet &patterns) {`。
- **L556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `patterns.add<XeGPUCreateNdDescOpPattern, XeGPULoadNdDescOpPattern,`.
  **L556 CN**: 继续一个多行参数列表、初始化器或聚合项：`patterns.add<XeGPUCreateNdDescOpPattern, XeGPULoadNdDescOpPattern,`。
- **L557 EN**: Continues logic associated with callable symbol `MultiRed2dOpPattern>`.
  **L557 CN**: 继续与可调用符号 `MultiRed2dOpPattern>` 相关的逻辑。
- **L558 EN**: Executes a call or declaration centered on `patterns.getContext`.
  **L558 CN**: 执行以 `patterns.getContext` 为核心的调用或声明。
- **L559 EN**: Closes the current lexical scope or compound statement.
  **L559 CN**: 结束当前词法作用域或复合语句块。
- **L560 EN**: Blank line separating nearby declarations or logic blocks.
  **L560 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 561-580

````cpp
namespace {

struct XeGPUPeepHoleOptimizerPass final
    : public xegpu::impl::XeGPUPeepHoleOptimizerBase<
          XeGPUPeepHoleOptimizerPass> {
  void runOnOperation() override {
    MLIRContext &context = getContext();
    TypeConverter converter;
    RewritePatternSet patterns(&context);
    ConversionTarget target(context);

    // This pass is only meant for PVC and BMG targets. If unsupported target
    // is found, exit early.
    bool isTargetSupported = false;
    getOperation()->walk([&](gpu::GPUFuncOp funcOp) {
      auto chipStr = xegpu::getChipStr(funcOp);
      if (chipStr && (chipStr.value() == "pvc" || chipStr.value() == "bmg" ||
                      chipStr.value() == "cri"))
        isTargetSupported = true;
    });
````
- **L561 EN**: Opens namespace scope ``.
  **L561 CN**: 打开命名空间作用域 ``。
- **L562 EN**: Blank line separating nearby declarations or logic blocks.
  **L562 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L563 EN**: Declares struct `XeGPUPeepHoleOptimizerPass`.
  **L563 CN**: 声明 struct `XeGPUPeepHoleOptimizerPass`。
- **L564 EN**: Continues the surrounding expression or declaration: `: public xegpu::impl::XeGPUPeepHoleOptimizerBase<`.
  **L564 CN**: 继续构造周围的表达式或声明：`: public xegpu::impl::XeGPUPeepHoleOptimizerBase<`。
- **L565 EN**: Continues the surrounding expression or declaration: `XeGPUPeepHoleOptimizerPass> {`.
  **L565 CN**: 继续构造周围的表达式或声明：`XeGPUPeepHoleOptimizerPass> {`。
- **L566 EN**: Starts a function, method, lambda, or structured scope: `void runOnOperation() override {`.
  **L566 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void runOnOperation() override {`。
- **L567 EN**: Executes a call or declaration centered on `getContext`.
  **L567 CN**: 执行以 `getContext` 为核心的调用或声明。
- **L568 EN**: Executes a standalone statement or declaration: `TypeConverter converter;`.
  **L568 CN**: 执行一条独立语句或声明：`TypeConverter converter;`。
- **L569 EN**: Executes a call or declaration centered on `patterns`.
  **L569 CN**: 执行以 `patterns` 为核心的调用或声明。
- **L570 EN**: Executes a call or declaration centered on `target`.
  **L570 CN**: 执行以 `target` 为核心的调用或声明。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L572 EN**: Comment explains nearby logic, invariants, or intent: `This pass is only meant for PVC and BMG targets. If unsupported target`.
  **L572 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This pass is only meant for PVC and BMG targets. If unsupported target`。
- **L573 EN**: Comment explains nearby logic, invariants, or intent: `is found, exit early.`.
  **L573 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`is found, exit early.`。
- **L574 EN**: Initializes variable `isTargetSupported` from the right-hand expression.
  **L574 CN**: 使用右侧表达式初始化变量 `isTargetSupported`。
- **L575 EN**: Starts a function, method, lambda, or structured scope: `getOperation()->walk([&](gpu::GPUFuncOp funcOp) {`.
  **L575 CN**: 开始一个函数、方法、lambda 或结构化作用域：`getOperation()->walk([&](gpu::GPUFuncOp funcOp) {`。
- **L576 EN**: Initializes variable `chipStr` from the right-hand expression.
  **L576 CN**: 使用右侧表达式初始化变量 `chipStr`。
- **L577 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L577 CN**: 开始 `if` 控制流语句并计算其条件。
- **L578 EN**: Continues logic associated with callable symbol `value`.
  **L578 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L579 EN**: Executes a standalone statement or declaration: `isTargetSupported = true;`.
  **L579 CN**: 执行一条独立语句或声明：`isTargetSupported = true;`。
- **L580 EN**: Executes a standalone statement or declaration: `});`.
  **L580 CN**: 执行一条独立语句或声明：`});`。

### Lines 581-600

````cpp

    if (!isTargetSupported) {
      DBGS() << "XeGPUPeepHoleOptimizerPass only supports PVC, BMG and CRI "
                "targets."
             << "\n";
      return;
    }

    // Run array length optimization patterns first so that subsequent transpose
    // peephole patterns operate on the array-length-optimized tensor descs.
    {
      RewritePatternSet arrayLenPatterns(&context);
      xegpu::populateXeGPUArrayLengthOptimizationPatterns(arrayLenPatterns);
      if (failed(applyPatternsGreedily(getOperation(),
                                       std::move(arrayLenPatterns)))) {
        DBGS() << "Array length optimization patterns failed.\n";
        return signalPassFailure();
      }
    }

````
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L582 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L582 CN**: 开始 `if` 控制流语句并计算其条件。
- **L583 EN**: Continues logic associated with callable symbol `DBGS`.
  **L583 CN**: 继续与可调用符号 `DBGS` 相关的逻辑。
- **L584 EN**: Continues the surrounding expression or declaration: `"targets."`.
  **L584 CN**: 继续构造周围的表达式或声明：`"targets."`。
- **L585 EN**: Executes a standalone statement or declaration: `<< "\n";`.
  **L585 CN**: 执行一条独立语句或声明：`<< "\n";`。
- **L586 EN**: Returns from the current function with `void`.
  **L586 CN**: 以 `void` 从当前函数返回。
- **L587 EN**: Closes the current lexical scope or compound statement.
  **L587 CN**: 结束当前词法作用域或复合语句块。
- **L588 EN**: Blank line separating nearby declarations or logic blocks.
  **L588 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L589 EN**: Comment explains nearby logic, invariants, or intent: `Run array length optimization patterns first so that subsequent transpose`.
  **L589 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Run array length optimization patterns first so that subsequent transpose`。
- **L590 EN**: Comment explains nearby logic, invariants, or intent: `peephole patterns operate on the array-length-optimized tensor descs.`.
  **L590 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`peephole patterns operate on the array-length-optimized tensor descs.`。
- **L591 EN**: Opens a new lexical scope or compound statement.
  **L591 CN**: 打开一个新的词法作用域或复合语句块。
- **L592 EN**: Executes a call or declaration centered on `arrayLenPatterns`.
  **L592 CN**: 执行以 `arrayLenPatterns` 为核心的调用或声明。
- **L593 EN**: Executes a call or declaration centered on `xegpu::populateXeGPUArrayLengthOptimizationPatterns`.
  **L593 CN**: 执行以 `xegpu::populateXeGPUArrayLengthOptimizationPatterns` 为核心的调用或声明。
- **L594 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L594 CN**: 开始 `if` 控制流语句并计算其条件。
- **L595 EN**: Starts a function, method, lambda, or structured scope: `std::move(arrayLenPatterns)))) {`.
  **L595 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::move(arrayLenPatterns)))) {`。
- **L596 EN**: Executes a call or declaration centered on `DBGS`.
  **L596 CN**: 执行以 `DBGS` 为核心的调用或声明。
- **L597 EN**: Returns from the current function with `signalPassFailure()`.
  **L597 CN**: 以 `signalPassFailure()` 从当前函数返回。
- **L598 EN**: Closes the current lexical scope or compound statement.
  **L598 CN**: 结束当前词法作用域或复合语句块。
- **L599 EN**: Closes the current lexical scope or compound statement.
  **L599 CN**: 结束当前词法作用域或复合语句块。
- **L600 EN**: Blank line separating nearby declarations or logic blocks.
  **L600 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 601-620

````cpp
    // CreateNdDescOp and LoadNdOp with optimizable tensor desc types must be
    // converted.
    target.addDynamicallyLegalOp<xegpu::CreateNdDescOp>(
        [&](xegpu::CreateNdDescOp createNdOp) {
          return !canBeOptimizedForTranspose(createNdOp.getType());
        });
    target.addDynamicallyLegalOp<xegpu::LoadNdOp>(
        [&](xegpu::LoadNdOp loadNdOp) {
          return !canBeOptimizedForTranspose(loadNdOp.getTensorDescType());
        });
    // Vector ExtractOps can have optimizable layouts if they extract from
    // LoadNdOps with array length greater than 1. These ExtractOps must be
    // converted.
    target.addDynamicallyLegalOp<vector::ExtractOp>(
        [&](vector::ExtractOp extractOp) {
          auto layout = xegpu::getTemporaryLayout(
              dyn_cast<OpResult>(extractOp.getResult()));
          if (!layout)
            return true;
          auto laneLayout = layout.getEffectiveLaneLayoutAsInt();
````
- **L601 EN**: Comment explains nearby logic, invariants, or intent: `CreateNdDescOp and LoadNdOp with optimizable tensor desc types must be`.
  **L601 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`CreateNdDescOp and LoadNdOp with optimizable tensor desc types must be`。
- **L602 EN**: Comment explains nearby logic, invariants, or intent: `converted.`.
  **L602 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`converted.`。
- **L603 EN**: Continues logic associated with callable symbol `CreateNdDescOp>`.
  **L603 CN**: 继续与可调用符号 `CreateNdDescOp>` 相关的逻辑。
- **L604 EN**: Starts a function, method, lambda, or structured scope: `[&](xegpu::CreateNdDescOp createNdOp) {`.
  **L604 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](xegpu::CreateNdDescOp createNdOp) {`。
- **L605 EN**: Returns from the current function with `!canBeOptimizedForTranspose(createNdOp.getType())`.
  **L605 CN**: 以 `!canBeOptimizedForTranspose(createNdOp.getType())` 从当前函数返回。
- **L606 EN**: Executes a standalone statement or declaration: `});`.
  **L606 CN**: 执行一条独立语句或声明：`});`。
- **L607 EN**: Continues logic associated with callable symbol `LoadNdOp>`.
  **L607 CN**: 继续与可调用符号 `LoadNdOp>` 相关的逻辑。
- **L608 EN**: Starts a function, method, lambda, or structured scope: `[&](xegpu::LoadNdOp loadNdOp) {`.
  **L608 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](xegpu::LoadNdOp loadNdOp) {`。
- **L609 EN**: Returns from the current function with `!canBeOptimizedForTranspose(loadNdOp.getTensorDescType())`.
  **L609 CN**: 以 `!canBeOptimizedForTranspose(loadNdOp.getTensorDescType())` 从当前函数返回。
- **L610 EN**: Executes a standalone statement or declaration: `});`.
  **L610 CN**: 执行一条独立语句或声明：`});`。
- **L611 EN**: Comment explains nearby logic, invariants, or intent: `Vector ExtractOps can have optimizable layouts if they extract from`.
  **L611 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector ExtractOps can have optimizable layouts if they extract from`。
- **L612 EN**: Comment explains nearby logic, invariants, or intent: `LoadNdOps with array length greater than 1. These ExtractOps must be`.
  **L612 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LoadNdOps with array length greater than 1. These ExtractOps must be`。
- **L613 EN**: Comment explains nearby logic, invariants, or intent: `converted.`.
  **L613 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`converted.`。
- **L614 EN**: Continues logic associated with callable symbol `ExtractOp>`.
  **L614 CN**: 继续与可调用符号 `ExtractOp>` 相关的逻辑。
- **L615 EN**: Starts a function, method, lambda, or structured scope: `[&](vector::ExtractOp extractOp) {`.
  **L615 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](vector::ExtractOp extractOp) {`。
- **L616 EN**: Continues logic associated with callable symbol `getTemporaryLayout`.
  **L616 CN**: 继续与可调用符号 `getTemporaryLayout` 相关的逻辑。
- **L617 EN**: Executes a call or declaration centered on `dyn_cast<OpResult>`.
  **L617 CN**: 执行以 `dyn_cast<OpResult>` 为核心的调用或声明。
- **L618 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L618 CN**: 开始 `if` 控制流语句并计算其条件。
- **L619 EN**: Returns from the current function with `true`.
  **L619 CN**: 以 `true` 从当前函数返回。
- **L620 EN**: Initializes variable `laneLayout` from the right-hand expression.
  **L620 CN**: 使用右侧表达式初始化变量 `laneLayout`。

### Lines 621-640

````cpp
          auto laneData = layout.getEffectiveLaneDataAsInt();
          return !canBeOptimizedForTranspose(laneLayout, laneData);
        });

    target.addDynamicallyLegalOp<vector::MultiDimReductionOp>(
        [=](Operation *op) -> bool {
          auto layout = xegpu::getDistributeLayoutAttr(op->getResult(0));
          if (!layout || !layout.isForSubgroup())
            return true;
          if (auto reductionOp = dyn_cast<vector::MultiDimReductionOp>(op))
            return reductionOp.getReductionDims().size() != 2;
          return true;
        });

    converter.addConversion([](Type type) { return type; });

    target.addLegalDialect<arith::ArithDialect, memref::MemRefDialect,
                           vector::VectorDialect>();
    // xegpu.convert_layout is left untouched by this pass; mark it legal
    // so in-place updates don't trigger re-legalization failures.
````
- **L621 EN**: Initializes variable `laneData` from the right-hand expression.
  **L621 CN**: 使用右侧表达式初始化变量 `laneData`。
- **L622 EN**: Returns from the current function with `!canBeOptimizedForTranspose(laneLayout, laneData)`.
  **L622 CN**: 以 `!canBeOptimizedForTranspose(laneLayout, laneData)` 从当前函数返回。
- **L623 EN**: Executes a standalone statement or declaration: `});`.
  **L623 CN**: 执行一条独立语句或声明：`});`。
- **L624 EN**: Blank line separating nearby declarations or logic blocks.
  **L624 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L625 EN**: Continues logic associated with callable symbol `MultiDimReductionOp>`.
  **L625 CN**: 继续与可调用符号 `MultiDimReductionOp>` 相关的逻辑。
- **L626 EN**: Starts a function, method, lambda, or structured scope: `[=](Operation *op) -> bool {`.
  **L626 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[=](Operation *op) -> bool {`。
- **L627 EN**: Initializes variable `layout` from the right-hand expression.
  **L627 CN**: 使用右侧表达式初始化变量 `layout`。
- **L628 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L628 CN**: 开始 `if` 控制流语句并计算其条件。
- **L629 EN**: Returns from the current function with `true`.
  **L629 CN**: 以 `true` 从当前函数返回。
- **L630 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L630 CN**: 开始 `if` 控制流语句并计算其条件。
- **L631 EN**: Returns from the current function with `reductionOp.getReductionDims().size() != 2`.
  **L631 CN**: 以 `reductionOp.getReductionDims().size() != 2` 从当前函数返回。
- **L632 EN**: Returns from the current function with `true`.
  **L632 CN**: 以 `true` 从当前函数返回。
- **L633 EN**: Executes a standalone statement or declaration: `});`.
  **L633 CN**: 执行一条独立语句或声明：`});`。
- **L634 EN**: Blank line separating nearby declarations or logic blocks.
  **L634 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L635 EN**: Executes a call or declaration centered on `converter.addConversion`.
  **L635 CN**: 执行以 `converter.addConversion` 为核心的调用或声明。
- **L636 EN**: Blank line separating nearby declarations or logic blocks.
  **L636 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `target.addLegalDialect<arith::ArithDialect, memref::MemRefDialect,`.
  **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`target.addLegalDialect<arith::ArithDialect, memref::MemRefDialect,`。
- **L638 EN**: Executes a call or declaration centered on `vector::VectorDialect>`.
  **L638 CN**: 执行以 `vector::VectorDialect>` 为核心的调用或声明。
- **L639 EN**: Comment explains nearby logic, invariants, or intent: `xegpu.convert_layout is left untouched by this pass; mark it legal`.
  **L639 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`xegpu.convert_layout is left untouched by this pass; mark it legal`。
- **L640 EN**: Comment explains nearby logic, invariants, or intent: `so in-place updates don't trigger re-legalization failures.`.
  **L640 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`so in-place updates don't trigger re-legalization failures.`。

### Lines 641-660

````cpp
    target.addLegalOp<xegpu::ConvertLayoutOp>();
    scf::populateSCFStructuralTypeConversionsAndLegality(converter, patterns,
                                                         target);
    xegpu::populateXeGPUPeepHoleOptimizerPatterns(patterns);
    if (failed(applyPartialConversion(getOperation(), target,
                                      std::move(patterns)))) {
      DBGS() << "Optimize block loads pass failed.\n";
      return signalPassFailure();
    }

    // Apply folding for cleaning up IR.
    MLIRContext *ctx = &getContext();
    RewritePatternSet emptyPatterns(ctx);
    (void)applyPatternsGreedily(getOperation(), std::move(emptyPatterns));

    xegpu::removeTemporaryLayoutAttrs(getOperation());
  }
};

} // namespace
````
- **L641 EN**: Executes a call or declaration centered on `target.addLegalOp<xegpu::ConvertLayoutOp>`.
  **L641 CN**: 执行以 `target.addLegalOp<xegpu::ConvertLayoutOp>` 为核心的调用或声明。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `scf::populateSCFStructuralTypeConversionsAndLegality(converter, patterns,`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`scf::populateSCFStructuralTypeConversionsAndLegality(converter, patterns,`。
- **L643 EN**: Executes a standalone statement or declaration: `target);`.
  **L643 CN**: 执行一条独立语句或声明：`target);`。
- **L644 EN**: Executes a call or declaration centered on `xegpu::populateXeGPUPeepHoleOptimizerPatterns`.
  **L644 CN**: 执行以 `xegpu::populateXeGPUPeepHoleOptimizerPatterns` 为核心的调用或声明。
- **L645 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L645 CN**: 开始 `if` 控制流语句并计算其条件。
- **L646 EN**: Starts a function, method, lambda, or structured scope: `std::move(patterns)))) {`.
  **L646 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::move(patterns)))) {`。
- **L647 EN**: Executes a call or declaration centered on `DBGS`.
  **L647 CN**: 执行以 `DBGS` 为核心的调用或声明。
- **L648 EN**: Returns from the current function with `signalPassFailure()`.
  **L648 CN**: 以 `signalPassFailure()` 从当前函数返回。
- **L649 EN**: Closes the current lexical scope or compound statement.
  **L649 CN**: 结束当前词法作用域或复合语句块。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L651 EN**: Comment explains nearby logic, invariants, or intent: `Apply folding for cleaning up IR.`.
  **L651 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Apply folding for cleaning up IR.`。
- **L652 EN**: Executes a call or declaration centered on `&getContext`.
  **L652 CN**: 执行以 `&getContext` 为核心的调用或声明。
- **L653 EN**: Executes a call or declaration centered on `emptyPatterns`.
  **L653 CN**: 执行以 `emptyPatterns` 为核心的调用或声明。
- **L654 EN**: Executes a call or declaration centered on `statement`.
  **L654 CN**: 执行以 `statement` 为核心的调用或声明。
- **L655 EN**: Blank line separating nearby declarations or logic blocks.
  **L655 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L656 EN**: Executes a call or declaration centered on `xegpu::removeTemporaryLayoutAttrs`.
  **L656 CN**: 执行以 `xegpu::removeTemporaryLayoutAttrs` 为核心的调用或声明。
- **L657 EN**: Closes the current lexical scope or compound statement.
  **L657 CN**: 结束当前词法作用域或复合语句块。
- **L658 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L658 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L659 EN**: Blank line separating nearby declarations or logic blocks.
  **L659 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L660 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L660 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。

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

- `mlir/Dialect/Arith/IR/Arith.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/GPU/IR/GPUDialect.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/MemRef/IR/MemRef.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/SCF/Transforms/Patterns.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Utils/IndexingUtils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Utils/StaticValueUtils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Vector/IR/VectorOps.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/IR/XeGPU.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Transforms/Passes.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Transforms/Transforms.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Transforms/XeGPULayoutImpl.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Utils/XeGPUUtils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/uArch/IntelGpuXe2.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/uArch/uArchBase.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/IR/BuiltinTypes.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `mlir/IR/OpDefinition.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `mlir/IR/Types.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `mlir/IR/Value.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `mlir/Transforms/DialectConversion.h`: Provides generic transformation utilities and canonicalization helpers. / 提供通用变换工具与规范化辅助逻辑。
- `mlir/Transforms/GreedyPatternRewriteDriver.h`: Provides generic transformation utilities and canonicalization helpers. / 提供通用变换工具与规范化辅助逻辑。
- `llvm/ADT/STLExtras.h`: Provides LLVM ADT containers and low-level utility types. / 提供LLVM ADT 容器与底层工具类型。
- `llvm/ADT/SmallVector.h`: Provides LLVM ADT containers and low-level utility types. / 提供LLVM ADT 容器与底层工具类型。
- `optional`: Provides supporting declarations used by the current translation unit. / 提供当前编译单元使用的辅助声明。
- `mlir/Dialect/XeGPU/Transforms/Passes.h.inc`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
