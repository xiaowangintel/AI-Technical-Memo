# XeGPUArrayLengthOptimization.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/XeGPU/Transforms/XeGPUArrayLengthOptimization.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements XeGPU lowering, scheduling, and rewrite passes.
- **Purpose (CN)**: 实现 XeGPU lowering、调度与重写 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===- XeGPUArrayLengthOptimization.cpp - Array Length Opt -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/Dialect/XeGPU/IR/XeGPU.h"
#include "mlir/Dialect/XeGPU/Transforms/Transforms.h"
#include "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h"
#include "mlir/Dialect/XeGPU/uArch/IntelGpuXe2.h"
#include "mlir/Dialect/XeGPU/uArch/uArchBase.h"
#include "mlir/IR/PatternMatch.h"
#include "llvm/ADT/SmallVector.h"

#define DEBUG_TYPE "xegpu-array-length-optimization"
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
- **L10 EN**: Includes "mlir/Dialect/XeGPU/IR/XeGPU.h" to access dialect-specific IR, transforms, or shared utilities.
  **L10 CN**: 引入 "mlir/Dialect/XeGPU/IR/XeGPU.h" 以使用方言专用 IR、变换或共享工具。
- **L11 EN**: Includes "mlir/Dialect/XeGPU/Transforms/Transforms.h" to access dialect-specific IR, transforms, or shared utilities.
  **L11 CN**: 引入 "mlir/Dialect/XeGPU/Transforms/Transforms.h" 以使用方言专用 IR、变换或共享工具。
- **L12 EN**: Includes "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L12 CN**: 引入 "mlir/Dialect/XeGPU/Utils/XeGPUUtils.h" 以使用方言专用 IR、变换或共享工具。
- **L13 EN**: Includes "mlir/Dialect/XeGPU/uArch/IntelGpuXe2.h" to access dialect-specific IR, transforms, or shared utilities.
  **L13 CN**: 引入 "mlir/Dialect/XeGPU/uArch/IntelGpuXe2.h" 以使用方言专用 IR、变换或共享工具。
- **L14 EN**: Includes "mlir/Dialect/XeGPU/uArch/uArchBase.h" to access dialect-specific IR, transforms, or shared utilities.
  **L14 CN**: 引入 "mlir/Dialect/XeGPU/uArch/uArchBase.h" 以使用方言专用 IR、变换或共享工具。
- **L15 EN**: Includes "mlir/IR/PatternMatch.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L15 CN**: 引入 "mlir/IR/PatternMatch.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L16 EN**: Includes "llvm/ADT/SmallVector.h" to access LLVM ADT containers and low-level utility types.
  **L16 CN**: 引入 "llvm/ADT/SmallVector.h" 以使用LLVM ADT 容器与底层工具类型。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L18 EN**: Defines macro `DEBUG_TYPE` for generated declarations, local shorthand, or conditional logic.
  **L18 CN**: 定义宏 `DEBUG_TYPE`，供生成式声明、本地简写或条件逻辑使用。

### Lines 19-36

````cpp

using namespace mlir;

namespace {

// Fallback subgroup size used when the target uArch cannot be resolved from
// the op (e.g. standalone unit tests with no chip attribute attached).
constexpr int64_t DEFAULT_SUBGROUP_SIZE = 16;

/// Return the subgroup size for `op`'s target uArch, falling back to
/// DEFAULT_SUBGROUP_SIZE if no chip attribute is attached or the chip is not
/// recognized.
static int64_t getSubgroupSize(Operation *op) {
  auto chipStr = xegpu::getChipStr(op);
  if (!chipStr)
    return DEFAULT_SUBGROUP_SIZE;
  const xegpu::uArch::uArch *targetUArch =
      xegpu::uArch::getUArch(chipStr.value());
````
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Brings namespace `mlir` into local scope.
  **L20 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Opens namespace scope ``.
  **L22 CN**: 打开命名空间作用域 ``。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Comment explains nearby logic, invariants, or intent: `Fallback subgroup size used when the target uArch cannot be resolved from`.
  **L24 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Fallback subgroup size used when the target uArch cannot be resolved from`。
- **L25 EN**: Comment explains nearby logic, invariants, or intent: `the op (e.g. standalone unit tests with no chip attribute attached).`.
  **L25 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the op (e.g. standalone unit tests with no chip attribute attached).`。
- **L26 EN**: Initializes variable `DEFAULT_SUBGROUP_SIZE` from the right-hand expression.
  **L26 CN**: 使用右侧表达式初始化变量 `DEFAULT_SUBGROUP_SIZE`。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Comment explains nearby logic, invariants, or intent: `Return the subgroup size for `op`'s target uArch, falling back to`.
  **L28 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return the subgroup size for `op`'s target uArch, falling back to`。
- **L29 EN**: Comment explains nearby logic, invariants, or intent: `DEFAULT_SUBGROUP_SIZE if no chip attribute is attached or the chip is not`.
  **L29 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`DEFAULT_SUBGROUP_SIZE if no chip attribute is attached or the chip is not`。
- **L30 EN**: Comment explains nearby logic, invariants, or intent: `recognized.`.
  **L30 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`recognized.`。
- **L31 EN**: Starts a function, method, lambda, or structured scope: `static int64_t getSubgroupSize(Operation *op) {`.
  **L31 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static int64_t getSubgroupSize(Operation *op) {`。
- **L32 EN**: Initializes variable `chipStr` from the right-hand expression.
  **L32 CN**: 使用右侧表达式初始化变量 `chipStr`。
- **L33 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L33 CN**: 开始 `if` 控制流语句并计算其条件。
- **L34 EN**: Returns from the current function with `DEFAULT_SUBGROUP_SIZE`.
  **L34 CN**: 以 `DEFAULT_SUBGROUP_SIZE` 从当前函数返回。
- **L35 EN**: Continues the surrounding expression or declaration: `const xegpu::uArch::uArch *targetUArch =`.
  **L35 CN**: 继续构造周围的表达式或声明：`const xegpu::uArch::uArch *targetUArch =`。
- **L36 EN**: Executes a call or declaration centered on `xegpu::uArch::getUArch`.
  **L36 CN**: 执行以 `xegpu::uArch::getUArch` 为核心的调用或声明。

### Lines 37-54

````cpp
  if (!targetUArch)
    return DEFAULT_SUBGROUP_SIZE;
  return targetUArch->getSubgroupSize();
}

/// Helper to compute array_length from FCD and subgroup size.
/// TODO: Currently, we are only allowing subgroupSize as our new FCD for LANE
/// level distribution simplicity. But it can be different, and in the future,
/// we can add that support.
static int64_t computeArrayLength(int64_t fcdSize, int64_t subgroupSize) {
  if (fcdSize <= subgroupSize)
    return 1;
  return fcdSize / subgroupSize;
}

/// Check if a 2D `xegpu.create_nd_tdesc` can be optimized into an
/// array-length-enabled descriptor. Applies only when the FCD is an integer
/// multiple of the subgroup size larger than the subgroup size itself and the
````
- **L37 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L37 CN**: 开始 `if` 控制流语句并计算其条件。
- **L38 EN**: Returns from the current function with `DEFAULT_SUBGROUP_SIZE`.
  **L38 CN**: 以 `DEFAULT_SUBGROUP_SIZE` 从当前函数返回。
- **L39 EN**: Returns from the current function with `targetUArch->getSubgroupSize()`.
  **L39 CN**: 以 `targetUArch->getSubgroupSize()` 从当前函数返回。
- **L40 EN**: Closes the current lexical scope or compound statement.
  **L40 CN**: 结束当前词法作用域或复合语句块。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L42 EN**: Comment explains nearby logic, invariants, or intent: `Helper to compute array_length from FCD and subgroup size.`.
  **L42 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper to compute array_length from FCD and subgroup size.`。
- **L43 EN**: Comment records a pending task or caution: `TODO: Currently, we are only allowing subgroupSize as our new FCD for LANE`.
  **L43 CN**: 注释记录了待办事项或注意点：`TODO: Currently, we are only allowing subgroupSize as our new FCD for LANE`。
- **L44 EN**: Comment explains nearby logic, invariants, or intent: `level distribution simplicity. But it can be different, and in the future,`.
  **L44 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`level distribution simplicity. But it can be different, and in the future,`。
- **L45 EN**: Comment explains nearby logic, invariants, or intent: `we can add that support.`.
  **L45 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`we can add that support.`。
- **L46 EN**: Starts a function, method, lambda, or structured scope: `static int64_t computeArrayLength(int64_t fcdSize, int64_t subgroupSize) {`.
  **L46 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static int64_t computeArrayLength(int64_t fcdSize, int64_t subgroupSize) {`。
- **L47 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L47 CN**: 开始 `if` 控制流语句并计算其条件。
- **L48 EN**: Returns from the current function with `1`.
  **L48 CN**: 以 `1` 从当前函数返回。
- **L49 EN**: Returns from the current function with `fcdSize / subgroupSize`.
  **L49 CN**: 以 `fcdSize / subgroupSize` 从当前函数返回。
- **L50 EN**: Closes the current lexical scope or compound statement.
  **L50 CN**: 结束当前词法作用域或复合语句块。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Comment explains nearby logic, invariants, or intent: `Check if a 2D `xegpu.create_nd_tdesc` can be optimized into an`.
  **L52 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check if a 2D `xegpu.create_nd_tdesc` can be optimized into an`。
- **L53 EN**: Comment explains nearby logic, invariants, or intent: `array-length-enabled descriptor. Applies only when the FCD is an integer`.
  **L53 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`array-length-enabled descriptor. Applies only when the FCD is an integer`。
- **L54 EN**: Comment explains nearby logic, invariants, or intent: `multiple of the subgroup size larger than the subgroup size itself and the`.
  **L54 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`multiple of the subgroup size larger than the subgroup size itself and the`。

### Lines 55-72

````cpp
/// tensor desc does not already carry an array_length.
static bool needsOptimization(xegpu::TensorDescType tdescType,
                              int64_t subgroupSize) {
  auto shape = tdescType.getShape();
  if (shape.size() != 2)
    return false;

  int64_t fcd = shape[1];
  if (fcd % subgroupSize != 0)
    return false;

  return fcd > subgroupSize && tdescType.getArrayLength() == 1;
}

/// Returns true if `loadOp` carries a non-identity transpose attribute. A
/// transpose of `[0, 1]` is the identity and is therefore treated as absent.
static bool hasNonIdentityTranspose(xegpu::LoadNdOp loadOp) {
  auto transpose = loadOp.getTranspose();
````
- **L55 EN**: Comment explains nearby logic, invariants, or intent: `tensor desc does not already carry an array_length.`.
  **L55 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tensor desc does not already carry an array_length.`。
- **L56 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static bool needsOptimization(xegpu::TensorDescType tdescType,`.
  **L56 CN**: 继续一个多行参数列表、初始化器或聚合项：`static bool needsOptimization(xegpu::TensorDescType tdescType,`。
- **L57 EN**: Continues the surrounding expression or declaration: `int64_t subgroupSize) {`.
  **L57 CN**: 继续构造周围的表达式或声明：`int64_t subgroupSize) {`。
- **L58 EN**: Initializes variable `shape` from the right-hand expression.
  **L58 CN**: 使用右侧表达式初始化变量 `shape`。
- **L59 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L59 CN**: 开始 `if` 控制流语句并计算其条件。
- **L60 EN**: Returns from the current function with `false`.
  **L60 CN**: 以 `false` 从当前函数返回。
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L62 EN**: Initializes variable `fcd` from the right-hand expression.
  **L62 CN**: 使用右侧表达式初始化变量 `fcd`。
- **L63 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L63 CN**: 开始 `if` 控制流语句并计算其条件。
- **L64 EN**: Returns from the current function with `false`.
  **L64 CN**: 以 `false` 从当前函数返回。
- **L65 EN**: Blank line separating nearby declarations or logic blocks.
  **L65 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L66 EN**: Returns from the current function with `fcd > subgroupSize && tdescType.getArrayLength() == 1`.
  **L66 CN**: 以 `fcd > subgroupSize && tdescType.getArrayLength() == 1` 从当前函数返回。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L69 EN**: Comment explains nearby logic, invariants, or intent: `Returns true if `loadOp` carries a non-identity transpose attribute. A`.
  **L69 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns true if `loadOp` carries a non-identity transpose attribute. A`。
- **L70 EN**: Comment explains nearby logic, invariants, or intent: `transpose of `[0, 1]` is the identity and is therefore treated as absent.`.
  **L70 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`transpose of `[0, 1]` is the identity and is therefore treated as absent.`。
- **L71 EN**: Starts a function, method, lambda, or structured scope: `static bool hasNonIdentityTranspose(xegpu::LoadNdOp loadOp) {`.
  **L71 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool hasNonIdentityTranspose(xegpu::LoadNdOp loadOp) {`。
- **L72 EN**: Initializes variable `transpose` from the right-hand expression.
  **L72 CN**: 使用右侧表达式初始化变量 `transpose`。

### Lines 73-90

````cpp
  if (!transpose)
    return false;
  ArrayRef<int64_t> perm = *transpose;
  return !(perm.size() == 2 && perm[0] == 0 && perm[1] == 1);
}

/// Returns true if `tdescType` carries a lane layout that signals a
/// transpose-intent load (lane_layout = `[SG, 1]`). Such descriptors are
/// rewritten by the transpose peephole optimization and must not be touched
/// here, since stacking the array blocks along the non-FCD dimension would
/// invalidate that rewrite.
static bool hasTransposeLaneLayout(xegpu::TensorDescType tdescType) {
  auto layout = tdescType.getLayoutAttr();
  if (!layout)
    return false;
  SmallVector<int64_t> laneLayout = layout.getEffectiveLaneLayoutAsInt();
  if (laneLayout.size() != 2)
    return false;
````
- **L73 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L73 CN**: 开始 `if` 控制流语句并计算其条件。
- **L74 EN**: Returns from the current function with `false`.
  **L74 CN**: 以 `false` 从当前函数返回。
- **L75 EN**: Initializes variable `perm` from the right-hand expression.
  **L75 CN**: 使用右侧表达式初始化变量 `perm`。
- **L76 EN**: Returns from the current function with `!(perm.size() == 2 && perm[0] == 0 && perm[1] == 1)`.
  **L76 CN**: 以 `!(perm.size() == 2 && perm[0] == 0 && perm[1] == 1)` 从当前函数返回。
- **L77 EN**: Closes the current lexical scope or compound statement.
  **L77 CN**: 结束当前词法作用域或复合语句块。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L79 EN**: Comment explains nearby logic, invariants, or intent: `Returns true if `tdescType` carries a lane layout that signals a`.
  **L79 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns true if `tdescType` carries a lane layout that signals a`。
- **L80 EN**: Comment explains nearby logic, invariants, or intent: `transpose-intent load (lane_layout = `[SG, 1]`). Such descriptors are`.
  **L80 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`transpose-intent load (lane_layout = `[SG, 1]`). Such descriptors are`。
- **L81 EN**: Comment explains nearby logic, invariants, or intent: `rewritten by the transpose peephole optimization and must not be touched`.
  **L81 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`rewritten by the transpose peephole optimization and must not be touched`。
- **L82 EN**: Comment explains nearby logic, invariants, or intent: `here, since stacking the array blocks along the non-FCD dimension would`.
  **L82 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`here, since stacking the array blocks along the non-FCD dimension would`。
- **L83 EN**: Comment explains nearby logic, invariants, or intent: `invalidate that rewrite.`.
  **L83 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`invalidate that rewrite.`。
- **L84 EN**: Starts a function, method, lambda, or structured scope: `static bool hasTransposeLaneLayout(xegpu::TensorDescType tdescType) {`.
  **L84 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool hasTransposeLaneLayout(xegpu::TensorDescType tdescType) {`。
- **L85 EN**: Initializes variable `layout` from the right-hand expression.
  **L85 CN**: 使用右侧表达式初始化变量 `layout`。
- **L86 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L86 CN**: 开始 `if` 控制流语句并计算其条件。
- **L87 EN**: Returns from the current function with `false`.
  **L87 CN**: 以 `false` 从当前函数返回。
- **L88 EN**: Initializes variable `laneLayout` from the right-hand expression.
  **L88 CN**: 使用右侧表达式初始化变量 `laneLayout`。
- **L89 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L89 CN**: 开始 `if` 控制流语句并计算其条件。
- **L90 EN**: Returns from the current function with `false`.
  **L90 CN**: 以 `false` 从当前函数返回。

### Lines 91-108

````cpp
  return laneLayout[0] != 1 && laneLayout[1] == 1;
}

/// Rewrite `xegpu.create_nd_tdesc` to fold an array_length attribute into the
/// resulting tensor descriptor type. Supports static memref, dynamic-shape
/// memref, and raw-pointer (integer) sources — the memory region described by
/// `shape`/`strides` is unchanged; only the tensor_desc view is narrowed along
/// the FCD and tagged with `array_length`. Skipped if any consumer load_nd
/// carries a non-identity transpose, since stacking the array blocks along the
/// non-FCD dimension would invalidate that load.
class OptimizeCreateNdDescOp : public OpRewritePattern<xegpu::CreateNdDescOp> {
public:
  using OpRewritePattern<xegpu::CreateNdDescOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(xegpu::CreateNdDescOp op,
                                PatternRewriter &rewriter) const override {
    int64_t subgroupSize = getSubgroupSize(op);
    auto tdescType = op.getType();
````
- **L91 EN**: Returns from the current function with `laneLayout[0] != 1 && laneLayout[1] == 1`.
  **L91 CN**: 以 `laneLayout[0] != 1 && laneLayout[1] == 1` 从当前函数返回。
- **L92 EN**: Closes the current lexical scope or compound statement.
  **L92 CN**: 结束当前词法作用域或复合语句块。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L94 EN**: Comment explains nearby logic, invariants, or intent: `Rewrite `xegpu.create_nd_tdesc` to fold an array_length attribute into the`.
  **L94 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Rewrite `xegpu.create_nd_tdesc` to fold an array_length attribute into the`。
- **L95 EN**: Comment explains nearby logic, invariants, or intent: `resulting tensor descriptor type. Supports static memref, dynamic-shape`.
  **L95 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`resulting tensor descriptor type. Supports static memref, dynamic-shape`。
- **L96 EN**: Comment explains nearby logic, invariants, or intent: `memref, and raw-pointer (integer) sources — the memory region described by`.
  **L96 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`memref, and raw-pointer (integer) sources — the memory region described by`。
- **L97 EN**: Comment explains nearby logic, invariants, or intent: ``shape`/`strides` is unchanged; only the tensor_desc view is narrowed along`.
  **L97 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：``shape`/`strides` is unchanged; only the tensor_desc view is narrowed along`。
- **L98 EN**: Comment explains nearby logic, invariants, or intent: `the FCD and tagged with `array_length`. Skipped if any consumer load_nd`.
  **L98 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the FCD and tagged with `array_length`. Skipped if any consumer load_nd`。
- **L99 EN**: Comment explains nearby logic, invariants, or intent: `carries a non-identity transpose, since stacking the array blocks along the`.
  **L99 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`carries a non-identity transpose, since stacking the array blocks along the`。
- **L100 EN**: Comment explains nearby logic, invariants, or intent: `non-FCD dimension would invalidate that load.`.
  **L100 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`non-FCD dimension would invalidate that load.`。
- **L101 EN**: Declares class `OptimizeCreateNdDescOp`.
  **L101 CN**: 声明 class `OptimizeCreateNdDescOp`。
- **L102 EN**: Sets the following members to `public` access.
  **L102 CN**: 将后续成员的访问级别设为 `public`。
- **L103 EN**: Executes a standalone statement or declaration: `using OpRewritePattern<xegpu::CreateNdDescOp>::OpRewritePattern;`.
  **L103 CN**: 执行一条独立语句或声明：`using OpRewritePattern<xegpu::CreateNdDescOp>::OpRewritePattern;`。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::CreateNdDescOp op,`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::CreateNdDescOp op,`。
- **L106 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L106 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L107 EN**: Initializes variable `subgroupSize` from the right-hand expression.
  **L107 CN**: 使用右侧表达式初始化变量 `subgroupSize`。
- **L108 EN**: Initializes variable `tdescType` from the right-hand expression.
  **L108 CN**: 使用右侧表达式初始化变量 `tdescType`。

### Lines 109-126

````cpp
    if (!needsOptimization(tdescType, subgroupSize))
      return failure();

    // A transpose lane layout marks this descriptor as a candidate for the
    // separate transpose peephole; stacking the array blocks would break it.
    if (hasTransposeLaneLayout(tdescType))
      return failure();

    Value source = op.getSource();
    if (!isa<MemRefType, IntegerType>(source.getType()))
      return failure();

    // Bail out if any consumer is a transposing load_nd.
    for (Operation *user : op.getResult().getUsers()) {
      if (auto loadOp = dyn_cast<xegpu::LoadNdOp>(user))
        if (hasNonIdentityTranspose(loadOp))
          return failure();
    }
````
- **L109 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L109 CN**: 开始 `if` 控制流语句并计算其条件。
- **L110 EN**: Returns from the current function with `failure()`.
  **L110 CN**: 以 `failure()` 从当前函数返回。
- **L111 EN**: Blank line separating nearby declarations or logic blocks.
  **L111 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L112 EN**: Comment explains nearby logic, invariants, or intent: `A transpose lane layout marks this descriptor as a candidate for the`.
  **L112 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A transpose lane layout marks this descriptor as a candidate for the`。
- **L113 EN**: Comment explains nearby logic, invariants, or intent: `separate transpose peephole; stacking the array blocks would break it.`.
  **L113 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`separate transpose peephole; stacking the array blocks would break it.`。
- **L114 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L114 CN**: 开始 `if` 控制流语句并计算其条件。
- **L115 EN**: Returns from the current function with `failure()`.
  **L115 CN**: 以 `failure()` 从当前函数返回。
- **L116 EN**: Blank line separating nearby declarations or logic blocks.
  **L116 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L117 EN**: Initializes variable `source` from the right-hand expression.
  **L117 CN**: 使用右侧表达式初始化变量 `source`。
- **L118 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L118 CN**: 开始 `if` 控制流语句并计算其条件。
- **L119 EN**: Returns from the current function with `failure()`.
  **L119 CN**: 以 `failure()` 从当前函数返回。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L121 EN**: Comment explains nearby logic, invariants, or intent: `Bail out if any consumer is a transposing load_nd.`.
  **L121 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Bail out if any consumer is a transposing load_nd.`。
- **L122 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L122 CN**: 开始 `for` 控制流语句并计算其条件。
- **L123 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L123 CN**: 开始 `if` 控制流语句并计算其条件。
- **L124 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L124 CN**: 开始 `if` 控制流语句并计算其条件。
- **L125 EN**: Returns from the current function with `failure()`.
  **L125 CN**: 以 `failure()` 从当前函数返回。
- **L126 EN**: Closes the current lexical scope or compound statement.
  **L126 CN**: 结束当前词法作用域或复合语句块。

### Lines 127-144

````cpp

    auto shape = tdescType.getShape();
    int64_t arrayLength = computeArrayLength(shape[1], subgroupSize);
    SmallVector<int64_t> newShape = {shape[0], shape[1] / arrayLength};

    auto newTdescType = xegpu::TensorDescType::get(
        newShape, tdescType.getElementType(), arrayLength,
        tdescType.getBoundaryCheck(), tdescType.getMemorySpace(),
        tdescType.getLayout());

    // The memory region is unchanged; pass through the existing shape/strides.
    // The general builder recognizes the static-memref case and drops the
    // redundant attributes.
    auto newOp = xegpu::CreateNdDescOp::create(
        rewriter, op.getLoc(), newTdescType, source, op.getMixedSizes(),
        op.getMixedStrides());
    rewriter.replaceOp(op, newOp.getResult());
    return success();
````
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L128 EN**: Initializes variable `shape` from the right-hand expression.
  **L128 CN**: 使用右侧表达式初始化变量 `shape`。
- **L129 EN**: Initializes variable `arrayLength` from the right-hand expression.
  **L129 CN**: 使用右侧表达式初始化变量 `arrayLength`。
- **L130 EN**: Initializes variable `newShape` from the right-hand expression.
  **L130 CN**: 使用右侧表达式初始化变量 `newShape`。
- **L131 EN**: Blank line separating nearby declarations or logic blocks.
  **L131 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L132 EN**: Continues logic associated with callable symbol `get`.
  **L132 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `newShape, tdescType.getElementType(), arrayLength,`.
  **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`newShape, tdescType.getElementType(), arrayLength,`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tdescType.getBoundaryCheck(), tdescType.getMemorySpace(),`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`tdescType.getBoundaryCheck(), tdescType.getMemorySpace(),`。
- **L135 EN**: Executes a call or declaration centered on `tdescType.getLayout`.
  **L135 CN**: 执行以 `tdescType.getLayout` 为核心的调用或声明。
- **L136 EN**: Blank line separating nearby declarations or logic blocks.
  **L136 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L137 EN**: Comment explains nearby logic, invariants, or intent: `The memory region is unchanged; pass through the existing shape/strides.`.
  **L137 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The memory region is unchanged; pass through the existing shape/strides.`。
- **L138 EN**: Comment explains nearby logic, invariants, or intent: `The general builder recognizes the static-memref case and drops the`.
  **L138 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The general builder recognizes the static-memref case and drops the`。
- **L139 EN**: Comment explains nearby logic, invariants, or intent: `redundant attributes.`.
  **L139 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`redundant attributes.`。
- **L140 EN**: Continues logic associated with callable symbol `create`.
  **L140 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, op.getLoc(), newTdescType, source, op.getMixedSizes(),`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, op.getLoc(), newTdescType, source, op.getMixedSizes(),`。
- **L142 EN**: Executes a call or declaration centered on `op.getMixedStrides`.
  **L142 CN**: 执行以 `op.getMixedStrides` 为核心的调用或声明。
- **L143 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L143 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L144 EN**: Returns from the current function with `success()`.
  **L144 CN**: 以 `success()` 从当前函数返回。

### Lines 145-162

````cpp
  }
};

/// Pattern to rewrite xegpu.load_nd operations
class OptimizeLoadNdOp : public OpRewritePattern<xegpu::LoadNdOp> {
public:
  using OpRewritePattern<xegpu::LoadNdOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(xegpu::LoadNdOp op,
                                PatternRewriter &rewriter) const override {
    auto tdescType = op.getTensorDescType();
    int64_t arrayLength = tdescType.getArrayLength();

    if (arrayLength <= 1)
      return failure();

    // Transposing loads are not compatible with the stacked-on-non-FCD layout
    // that this pass produces.
````
- **L145 EN**: Closes the current lexical scope or compound statement.
  **L145 CN**: 结束当前词法作用域或复合语句块。
- **L146 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L146 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L148 EN**: Comment explains nearby logic, invariants, or intent: `Pattern to rewrite xegpu.load_nd operations`.
  **L148 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Pattern to rewrite xegpu.load_nd operations`。
- **L149 EN**: Declares class `OptimizeLoadNdOp`.
  **L149 CN**: 声明 class `OptimizeLoadNdOp`。
- **L150 EN**: Sets the following members to `public` access.
  **L150 CN**: 将后续成员的访问级别设为 `public`。
- **L151 EN**: Executes a standalone statement or declaration: `using OpRewritePattern<xegpu::LoadNdOp>::OpRewritePattern;`.
  **L151 CN**: 执行一条独立语句或声明：`using OpRewritePattern<xegpu::LoadNdOp>::OpRewritePattern;`。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(xegpu::LoadNdOp op,`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(xegpu::LoadNdOp op,`。
- **L154 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L154 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L155 EN**: Initializes variable `tdescType` from the right-hand expression.
  **L155 CN**: 使用右侧表达式初始化变量 `tdescType`。
- **L156 EN**: Initializes variable `arrayLength` from the right-hand expression.
  **L156 CN**: 使用右侧表达式初始化变量 `arrayLength`。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L158 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L158 CN**: 开始 `if` 控制流语句并计算其条件。
- **L159 EN**: Returns from the current function with `failure()`.
  **L159 CN**: 以 `failure()` 从当前函数返回。
- **L160 EN**: Blank line separating nearby declarations or logic blocks.
  **L160 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L161 EN**: Comment explains nearby logic, invariants, or intent: `Transposing loads are not compatible with the stacked-on-non-FCD layout`.
  **L161 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Transposing loads are not compatible with the stacked-on-non-FCD layout`。
- **L162 EN**: Comment explains nearby logic, invariants, or intent: `that this pass produces.`.
  **L162 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`that this pass produces.`。

### Lines 163-180

````cpp
    if (hasNonIdentityTranspose(op) || hasTransposeLaneLayout(tdescType))
      return failure();

    auto origVectorType = op.getType();
    auto origShape = origVectorType.getShape();
    if (origShape.size() != 2)
      return failure();

    // The expected vector shape is: [tdesc_non_FCD * array_length, tdesc_FCD]
    int64_t expectedNonFCD = tdescType.getShape()[0] * arrayLength;
    int64_t expectedFCD = tdescType.getShape()[1];

    // If already matches expected shape, skip
    if (origShape[0] == expectedNonFCD && origShape[1] == expectedFCD)
      return failure();

    // Compute new vector shape for register layout
    SmallVector<int64_t> newShape = {expectedNonFCD, expectedFCD};
````
- **L163 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L163 CN**: 开始 `if` 控制流语句并计算其条件。
- **L164 EN**: Returns from the current function with `failure()`.
  **L164 CN**: 以 `failure()` 从当前函数返回。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L166 EN**: Initializes variable `origVectorType` from the right-hand expression.
  **L166 CN**: 使用右侧表达式初始化变量 `origVectorType`。
- **L167 EN**: Initializes variable `origShape` from the right-hand expression.
  **L167 CN**: 使用右侧表达式初始化变量 `origShape`。
- **L168 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L168 CN**: 开始 `if` 控制流语句并计算其条件。
- **L169 EN**: Returns from the current function with `failure()`.
  **L169 CN**: 以 `failure()` 从当前函数返回。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L171 EN**: Comment explains nearby logic, invariants, or intent: `The expected vector shape is: [tdesc_non_FCD * array_length, tdesc_FCD]`.
  **L171 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The expected vector shape is: [tdesc_non_FCD * array_length, tdesc_FCD]`。
- **L172 EN**: Initializes variable `expectedNonFCD` from the right-hand expression.
  **L172 CN**: 使用右侧表达式初始化变量 `expectedNonFCD`。
- **L173 EN**: Initializes variable `expectedFCD` from the right-hand expression.
  **L173 CN**: 使用右侧表达式初始化变量 `expectedFCD`。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L175 EN**: Comment explains nearby logic, invariants, or intent: `If already matches expected shape, skip`.
  **L175 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If already matches expected shape, skip`。
- **L176 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L176 CN**: 开始 `if` 控制流语句并计算其条件。
- **L177 EN**: Returns from the current function with `failure()`.
  **L177 CN**: 以 `failure()` 从当前函数返回。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L179 EN**: Comment explains nearby logic, invariants, or intent: `Compute new vector shape for register layout`.
  **L179 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Compute new vector shape for register layout`。
- **L180 EN**: Initializes variable `newShape` from the right-hand expression.
  **L180 CN**: 使用右侧表达式初始化变量 `newShape`。

### Lines 181-198

````cpp
    auto newVectorType =
        VectorType::get(newShape, origVectorType.getElementType());

    // Create new LoadNdOp with updated result type
    auto newLoadOp = xegpu::LoadNdOp::create(
        rewriter, op.getLoc(), newVectorType, op.getTensorDesc(),
        op.getMixedOffsets(), op.getPackedAttr(), op.getTransposeAttr(),
        op.getL1HintAttr(), op.getL2HintAttr(), op.getL3HintAttr(),
        op.getLayoutAttr());

    rewriter.replaceOp(op, newLoadOp.getResult());
    return success();
  }
};

/// Rewrite `vector.extract_strided_slice` offsets so they index into the
/// stacked register layout produced by `OptimizeLoadNdOp`.
///
````
- **L181 EN**: Continues the surrounding expression or declaration: `auto newVectorType =`.
  **L181 CN**: 继续构造周围的表达式或声明：`auto newVectorType =`。
- **L182 EN**: Executes a call or declaration centered on `VectorType::get`.
  **L182 CN**: 执行以 `VectorType::get` 为核心的调用或声明。
- **L183 EN**: Blank line separating nearby declarations or logic blocks.
  **L183 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L184 EN**: Comment explains nearby logic, invariants, or intent: `Create new LoadNdOp with updated result type`.
  **L184 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create new LoadNdOp with updated result type`。
- **L185 EN**: Continues logic associated with callable symbol `create`.
  **L185 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, op.getLoc(), newVectorType, op.getTensorDesc(),`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, op.getLoc(), newVectorType, op.getTensorDesc(),`。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op.getMixedOffsets(), op.getPackedAttr(), op.getTransposeAttr(),`.
  **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`op.getMixedOffsets(), op.getPackedAttr(), op.getTransposeAttr(),`。
- **L188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op.getL1HintAttr(), op.getL2HintAttr(), op.getL3HintAttr(),`.
  **L188 CN**: 继续一个多行参数列表、初始化器或聚合项：`op.getL1HintAttr(), op.getL2HintAttr(), op.getL3HintAttr(),`。
- **L189 EN**: Executes a call or declaration centered on `op.getLayoutAttr`.
  **L189 CN**: 执行以 `op.getLayoutAttr` 为核心的调用或声明。
- **L190 EN**: Blank line separating nearby declarations or logic blocks.
  **L190 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L191 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L191 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L192 EN**: Returns from the current function with `success()`.
  **L192 CN**: 以 `success()` 从当前函数返回。
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L194 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L195 EN**: Blank line separating nearby declarations or logic blocks.
  **L195 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L196 EN**: Comment explains nearby logic, invariants, or intent: `Rewrite `vector.extract_strided_slice` offsets so they index into the`.
  **L196 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Rewrite `vector.extract_strided_slice` offsets so they index into the`。
- **L197 EN**: Comment explains nearby logic, invariants, or intent: `stacked register layout produced by `OptimizeLoadNdOp`.`.
  **L197 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`stacked register layout produced by `OptimizeLoadNdOp`.`。
- **L198 EN**: Separator comment used for visual grouping.
  **L198 CN**: 用于视觉分组的分隔注释。

### Lines 199-216

````cpp
/// The optimized load places `arrayLength` blocks side-by-side in memory
/// but stacks them along the non-FCD dimension in registers. Given a
/// tensor desc of shape `[H, W]` with array_length = A:
///
///   memory layout (what the extract offsets refer to): `[H, W * A]`
///   register layout (what the new load returns):       `[H * A, W]`
///
/// An extract at memory offset `[r, c]` therefore maps to register offset
/// `[r + (c / W) * H, 0]` — provided the extract is block-aligned in the
/// FCD dimension, i.e. `c % W == 0`.
///
/// Example (`A = 2`, `H = 32`, `W = 16`):
///
///   // before
///   %v = xegpu.load_nd %t : ... -> vector<32x32xf16>
///   %e = vector.extract_strided_slice %v
///          {offsets = [0, 16], sizes = [16, 16], strides = [1, 1]}
///          : vector<32x32xf16> to vector<16x16xf16>
````
- **L199 EN**: Comment explains nearby logic, invariants, or intent: `The optimized load places `arrayLength` blocks side-by-side in memory`.
  **L199 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The optimized load places `arrayLength` blocks side-by-side in memory`。
- **L200 EN**: Comment explains nearby logic, invariants, or intent: `but stacks them along the non-FCD dimension in registers. Given a`.
  **L200 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`but stacks them along the non-FCD dimension in registers. Given a`。
- **L201 EN**: Comment explains nearby logic, invariants, or intent: `tensor desc of shape `[H, W]` with array_length = A:`.
  **L201 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tensor desc of shape `[H, W]` with array_length = A:`。
- **L202 EN**: Separator comment used for visual grouping.
  **L202 CN**: 用于视觉分组的分隔注释。
- **L203 EN**: Comment explains nearby logic, invariants, or intent: `memory layout (what the extract offsets refer to): `[H, W * A]``.
  **L203 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`memory layout (what the extract offsets refer to): `[H, W * A]``。
- **L204 EN**: Comment explains nearby logic, invariants, or intent: `register layout (what the new load returns):       `[H * A, W]``.
  **L204 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`register layout (what the new load returns):       `[H * A, W]``。
- **L205 EN**: Separator comment used for visual grouping.
  **L205 CN**: 用于视觉分组的分隔注释。
- **L206 EN**: Comment explains nearby logic, invariants, or intent: `An extract at memory offset `[r, c]` therefore maps to register offset`.
  **L206 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`An extract at memory offset `[r, c]` therefore maps to register offset`。
- **L207 EN**: Comment explains nearby logic, invariants, or intent: ``[r + (c / W) * H, 0]` — provided the extract is block-aligned in the`.
  **L207 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：``[r + (c / W) * H, 0]` — provided the extract is block-aligned in the`。
- **L208 EN**: Comment explains nearby logic, invariants, or intent: `FCD dimension, i.e. `c % W == 0`.`.
  **L208 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FCD dimension, i.e. `c % W == 0`.`。
- **L209 EN**: Separator comment used for visual grouping.
  **L209 CN**: 用于视觉分组的分隔注释。
- **L210 EN**: Comment explains nearby logic, invariants, or intent: `Example (`A = 2`, `H = 32`, `W = 16`):`.
  **L210 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Example (`A = 2`, `H = 32`, `W = 16`):`。
- **L211 EN**: Separator comment used for visual grouping.
  **L211 CN**: 用于视觉分组的分隔注释。
- **L212 EN**: Comment explains nearby logic, invariants, or intent: `// before`.
  **L212 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`// before`。
- **L213 EN**: Comment explains nearby logic, invariants, or intent: `%v = xegpu.load_nd %t : ... -> vector<32x32xf16>`.
  **L213 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%v = xegpu.load_nd %t : ... -> vector<32x32xf16>`。
- **L214 EN**: Comment explains nearby logic, invariants, or intent: `%e = vector.extract_strided_slice %v`.
  **L214 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%e = vector.extract_strided_slice %v`。
- **L215 EN**: Comment explains nearby logic, invariants, or intent: `{offsets = [0, 16], sizes = [16, 16], strides = [1, 1]}`.
  **L215 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`{offsets = [0, 16], sizes = [16, 16], strides = [1, 1]}`。
- **L216 EN**: Comment explains nearby logic, invariants, or intent: `: vector<32x32xf16> to vector<16x16xf16>`.
  **L216 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`: vector<32x32xf16> to vector<16x16xf16>`。

### Lines 217-234

````cpp
///
///   // after (load rewritten to vector<64x16>, extract offset remapped)
///   %v = xegpu.load_nd %t : ... -> vector<64x16xf16>
///   %e = vector.extract_strided_slice %v
///          {offsets = [32, 0], sizes = [16, 16], strides = [1, 1]}
///          : vector<64x16xf16> to vector<16x16xf16>
class UpdateExtractStridedSliceOp
    : public OpRewritePattern<vector::ExtractStridedSliceOp> {
public:
  using OpRewritePattern<vector::ExtractStridedSliceOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(vector::ExtractStridedSliceOp op,
                                PatternRewriter &rewriter) const override {
    auto sourceType = dyn_cast<VectorType>(op.getSource().getType());
    if (!sourceType || sourceType.getRank() != 2)
      return failure();

    auto loadOp = op.getSource().getDefiningOp<xegpu::LoadNdOp>();
````
- **L217 EN**: Separator comment used for visual grouping.
  **L217 CN**: 用于视觉分组的分隔注释。
- **L218 EN**: Comment explains nearby logic, invariants, or intent: `// after (load rewritten to vector<64x16>, extract offset remapped)`.
  **L218 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`// after (load rewritten to vector<64x16>, extract offset remapped)`。
- **L219 EN**: Comment explains nearby logic, invariants, or intent: `%v = xegpu.load_nd %t : ... -> vector<64x16xf16>`.
  **L219 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%v = xegpu.load_nd %t : ... -> vector<64x16xf16>`。
- **L220 EN**: Comment explains nearby logic, invariants, or intent: `%e = vector.extract_strided_slice %v`.
  **L220 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%e = vector.extract_strided_slice %v`。
- **L221 EN**: Comment explains nearby logic, invariants, or intent: `{offsets = [32, 0], sizes = [16, 16], strides = [1, 1]}`.
  **L221 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`{offsets = [32, 0], sizes = [16, 16], strides = [1, 1]}`。
- **L222 EN**: Comment explains nearby logic, invariants, or intent: `: vector<64x16xf16> to vector<16x16xf16>`.
  **L222 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`: vector<64x16xf16> to vector<16x16xf16>`。
- **L223 EN**: Declares class `UpdateExtractStridedSliceOp`.
  **L223 CN**: 声明 class `UpdateExtractStridedSliceOp`。
- **L224 EN**: Continues the surrounding expression or declaration: `: public OpRewritePattern<vector::ExtractStridedSliceOp> {`.
  **L224 CN**: 继续构造周围的表达式或声明：`: public OpRewritePattern<vector::ExtractStridedSliceOp> {`。
- **L225 EN**: Sets the following members to `public` access.
  **L225 CN**: 将后续成员的访问级别设为 `public`。
- **L226 EN**: Executes a standalone statement or declaration: `using OpRewritePattern<vector::ExtractStridedSliceOp>::OpRewritePattern;`.
  **L226 CN**: 执行一条独立语句或声明：`using OpRewritePattern<vector::ExtractStridedSliceOp>::OpRewritePattern;`。
- **L227 EN**: Blank line separating nearby declarations or logic blocks.
  **L227 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(vector::ExtractStridedSliceOp op,`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(vector::ExtractStridedSliceOp op,`。
- **L229 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L229 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L230 EN**: Initializes variable `sourceType` from the right-hand expression.
  **L230 CN**: 使用右侧表达式初始化变量 `sourceType`。
- **L231 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L231 CN**: 开始 `if` 控制流语句并计算其条件。
- **L232 EN**: Returns from the current function with `failure()`.
  **L232 CN**: 以 `failure()` 从当前函数返回。
- **L233 EN**: Blank line separating nearby declarations or logic blocks.
  **L233 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L234 EN**: Initializes variable `loadOp` from the right-hand expression.
  **L234 CN**: 使用右侧表达式初始化变量 `loadOp`。

### Lines 235-252

````cpp
    if (!loadOp)
      return failure();

    auto tdescType = loadOp.getTensorDescType();
    int64_t arrayLength = tdescType.getArrayLength();
    if (arrayLength <= 1)
      return failure();

    auto offsets = op.getOffsets().getValue();
    auto sizes = op.getSizes().getValue();
    auto strides = op.getStrides().getValue();

    if (offsets.size() != 2 || sizes.size() != 2 || strides.size() != 2)
      return failure();

    int64_t origOffset0 = cast<IntegerAttr>(offsets[0]).getInt();
    int64_t origOffset1 = cast<IntegerAttr>(offsets[1]).getInt();

````
- **L235 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L235 CN**: 开始 `if` 控制流语句并计算其条件。
- **L236 EN**: Returns from the current function with `failure()`.
  **L236 CN**: 以 `failure()` 从当前函数返回。
- **L237 EN**: Blank line separating nearby declarations or logic blocks.
  **L237 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L238 EN**: Initializes variable `tdescType` from the right-hand expression.
  **L238 CN**: 使用右侧表达式初始化变量 `tdescType`。
- **L239 EN**: Initializes variable `arrayLength` from the right-hand expression.
  **L239 CN**: 使用右侧表达式初始化变量 `arrayLength`。
- **L240 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L240 CN**: 开始 `if` 控制流语句并计算其条件。
- **L241 EN**: Returns from the current function with `failure()`.
  **L241 CN**: 以 `failure()` 从当前函数返回。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L243 EN**: Initializes variable `offsets` from the right-hand expression.
  **L243 CN**: 使用右侧表达式初始化变量 `offsets`。
- **L244 EN**: Initializes variable `sizes` from the right-hand expression.
  **L244 CN**: 使用右侧表达式初始化变量 `sizes`。
- **L245 EN**: Initializes variable `strides` from the right-hand expression.
  **L245 CN**: 使用右侧表达式初始化变量 `strides`。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L247 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L247 CN**: 开始 `if` 控制流语句并计算其条件。
- **L248 EN**: Returns from the current function with `failure()`.
  **L248 CN**: 以 `failure()` 从当前函数返回。
- **L249 EN**: Blank line separating nearby declarations or logic blocks.
  **L249 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L250 EN**: Initializes variable `origOffset0` from the right-hand expression.
  **L250 CN**: 使用右侧表达式初始化变量 `origOffset0`。
- **L251 EN**: Initializes variable `origOffset1` from the right-hand expression.
  **L251 CN**: 使用右侧表达式初始化变量 `origOffset1`。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 253-270

````cpp
    int64_t blockHeight = tdescType.getShape()[0];
    int64_t arrayWidth = tdescType.getShape()[1];

    // Skip extracts that already live entirely inside block 0: their offsets
    // are identical in the memory and register layouts, so there is nothing
    // to rewrite.
    if (origOffset1 < arrayWidth)
      return failure();

    // The remap is only well-defined when the extract is aligned to an array
    // block along the FCD.
    assert(origOffset1 % arrayWidth == 0 &&
           "extract offset along FCD must be a multiple of the array width");

    int64_t arrayIndex = origOffset1 / arrayWidth;
    SmallVector<int64_t> newOffsets = {origOffset0 + arrayIndex * blockHeight,
                                       /*offset1=*/0};

````
- **L253 EN**: Initializes variable `blockHeight` from the right-hand expression.
  **L253 CN**: 使用右侧表达式初始化变量 `blockHeight`。
- **L254 EN**: Initializes variable `arrayWidth` from the right-hand expression.
  **L254 CN**: 使用右侧表达式初始化变量 `arrayWidth`。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L256 EN**: Comment explains nearby logic, invariants, or intent: `Skip extracts that already live entirely inside block 0: their offsets`.
  **L256 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Skip extracts that already live entirely inside block 0: their offsets`。
- **L257 EN**: Comment explains nearby logic, invariants, or intent: `are identical in the memory and register layouts, so there is nothing`.
  **L257 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`are identical in the memory and register layouts, so there is nothing`。
- **L258 EN**: Comment explains nearby logic, invariants, or intent: `to rewrite.`.
  **L258 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to rewrite.`。
- **L259 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L259 CN**: 开始 `if` 控制流语句并计算其条件。
- **L260 EN**: Returns from the current function with `failure()`.
  **L260 CN**: 以 `failure()` 从当前函数返回。
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L262 EN**: Comment explains nearby logic, invariants, or intent: `The remap is only well-defined when the extract is aligned to an array`.
  **L262 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The remap is only well-defined when the extract is aligned to an array`。
- **L263 EN**: Comment explains nearby logic, invariants, or intent: `block along the FCD.`.
  **L263 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`block along the FCD.`。
- **L264 EN**: Checks an internal invariant in debug builds.
  **L264 CN**: 在调试构建中检查内部不变式。
- **L265 EN**: Executes a standalone statement or declaration: `"extract offset along FCD must be a multiple of the array width");`.
  **L265 CN**: 执行一条独立语句或声明：`"extract offset along FCD must be a multiple of the array width");`。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L267 EN**: Initializes variable `arrayIndex` from the right-hand expression.
  **L267 CN**: 使用右侧表达式初始化变量 `arrayIndex`。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVector<int64_t> newOffsets = {origOffset0 + arrayIndex * blockHeight,`.
  **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVector<int64_t> newOffsets = {origOffset0 + arrayIndex * blockHeight,`。
- **L269 EN**: Comment explains nearby logic, invariants, or intent: `offset1=*/0};`.
  **L269 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`offset1=*/0};`。
- **L270 EN**: Blank line separating nearby declarations or logic blocks.
  **L270 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 271-288

````cpp
    auto toInts = [](ArrayAttr arr) {
      return llvm::to_vector(llvm::map_range(
          arr, [](Attribute a) { return cast<IntegerAttr>(a).getInt(); }));
    };
    SmallVector<int64_t> sliceSizes = toInts(op.getSizes());
    SmallVector<int64_t> sliceStrides = toInts(op.getStrides());

    auto newOp = vector::ExtractStridedSliceOp::create(
        rewriter, op.getLoc(), op.getSource(), newOffsets, sliceSizes,
        sliceStrides);

    rewriter.replaceOp(op, newOp.getResult());
    return success();
  }
};

} // namespace

````
- **L271 EN**: Starts a function, method, lambda, or structured scope: `auto toInts = [](ArrayAttr arr) {`.
  **L271 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto toInts = [](ArrayAttr arr) {`。
- **L272 EN**: Returns from the current function with `llvm::to_vector(llvm::map_range(`.
  **L272 CN**: 以 `llvm::to_vector(llvm::map_range(` 从当前函数返回。
- **L273 EN**: Executes a call or declaration centered on `[]`.
  **L273 CN**: 执行以 `[]` 为核心的调用或声明。
- **L274 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L274 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L275 EN**: Initializes variable `sliceSizes` from the right-hand expression.
  **L275 CN**: 使用右侧表达式初始化变量 `sliceSizes`。
- **L276 EN**: Initializes variable `sliceStrides` from the right-hand expression.
  **L276 CN**: 使用右侧表达式初始化变量 `sliceStrides`。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L278 EN**: Continues logic associated with callable symbol `create`.
  **L278 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, op.getLoc(), op.getSource(), newOffsets, sliceSizes,`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, op.getLoc(), op.getSource(), newOffsets, sliceSizes,`。
- **L280 EN**: Executes a standalone statement or declaration: `sliceStrides);`.
  **L280 CN**: 执行一条独立语句或声明：`sliceStrides);`。
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L282 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L282 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L283 EN**: Returns from the current function with `success()`.
  **L283 CN**: 以 `success()` 从当前函数返回。
- **L284 EN**: Closes the current lexical scope or compound statement.
  **L284 CN**: 结束当前词法作用域或复合语句块。
- **L285 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L285 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L286 EN**: Blank line separating nearby declarations or logic blocks.
  **L286 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L287 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L287 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 289-293

````cpp
void xegpu::populateXeGPUArrayLengthOptimizationPatterns(
    RewritePatternSet &patterns) {
  patterns.add<OptimizeCreateNdDescOp, OptimizeLoadNdOp,
               UpdateExtractStridedSliceOp>(patterns.getContext());
}
````
- **L289 EN**: Continues logic associated with callable symbol `populateXeGPUArrayLengthOptimizationPatterns`.
  **L289 CN**: 继续与可调用符号 `populateXeGPUArrayLengthOptimizationPatterns` 相关的逻辑。
- **L290 EN**: Continues the surrounding expression or declaration: `RewritePatternSet &patterns) {`.
  **L290 CN**: 继续构造周围的表达式或声明：`RewritePatternSet &patterns) {`。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `patterns.add<OptimizeCreateNdDescOp, OptimizeLoadNdOp,`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`patterns.add<OptimizeCreateNdDescOp, OptimizeLoadNdOp,`。
- **L292 EN**: Executes a call or declaration centered on `UpdateExtractStridedSliceOp>`.
  **L292 CN**: 执行以 `UpdateExtractStridedSliceOp>` 为核心的调用或声明。
- **L293 EN**: Closes the current lexical scope or compound statement.
  **L293 CN**: 结束当前词法作用域或复合语句块。

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

- `mlir/Dialect/Vector/IR/VectorOps.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/IR/XeGPU.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Transforms/Transforms.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/Utils/XeGPUUtils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/uArch/IntelGpuXe2.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/XeGPU/uArch/uArchBase.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/IR/PatternMatch.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `llvm/ADT/SmallVector.h`: Provides LLVM ADT containers and low-level utility types. / 提供LLVM ADT 容器与底层工具类型。
