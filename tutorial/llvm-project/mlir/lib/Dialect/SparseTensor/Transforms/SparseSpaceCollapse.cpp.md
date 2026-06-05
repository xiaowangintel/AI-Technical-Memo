# SparseSpaceCollapse.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SparseTensor/Transforms/SparseSpaceCollapse.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements sparse tensor optimization, lowering, and rewrite passes.
- **Purpose (CN)**: 实现稀疏张量优化、lowering 与重写 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===--------- SparseSpaceCollapse.cpp - Collapse Sparse Space Pass -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/Transforms/Passes.h"

#include "mlir/Dialect/SparseTensor/IR/SparseTensor.h"
#include "mlir/Dialect/SparseTensor/Transforms/Passes.h"

namespace mlir {
#define GEN_PASS_DEF_SPARSESPACECOLLAPSE
#include "mlir/Dialect/SparseTensor/Transforms/Passes.h.inc"
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
- **L9 EN**: Includes "mlir/Dialect/Func/IR/FuncOps.h" to access dialect-specific IR, transforms, or shared utilities.
  **L9 CN**: 引入 "mlir/Dialect/Func/IR/FuncOps.h" 以使用方言专用 IR、变换或共享工具。
- **L10 EN**: Includes "mlir/IR/IRMapping.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L10 CN**: 引入 "mlir/IR/IRMapping.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L11 EN**: Includes "mlir/Transforms/Passes.h" to access generic transformation utilities and canonicalization helpers.
  **L11 CN**: 引入 "mlir/Transforms/Passes.h" 以使用通用变换工具与规范化辅助逻辑。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Includes "mlir/Dialect/SparseTensor/IR/SparseTensor.h" to access dialect-specific IR, transforms, or shared utilities.
  **L13 CN**: 引入 "mlir/Dialect/SparseTensor/IR/SparseTensor.h" 以使用方言专用 IR、变换或共享工具。
- **L14 EN**: Includes "mlir/Dialect/SparseTensor/Transforms/Passes.h" to access dialect-specific IR, transforms, or shared utilities.
  **L14 CN**: 引入 "mlir/Dialect/SparseTensor/Transforms/Passes.h" 以使用方言专用 IR、变换或共享工具。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L16 EN**: Opens namespace scope `mlir`.
  **L16 CN**: 打开命名空间作用域 `mlir`。
- **L17 EN**: Defines macro `GEN_PASS_DEF_SPARSESPACECOLLAPSE` for generated declarations, local shorthand, or conditional logic.
  **L17 CN**: 定义宏 `GEN_PASS_DEF_SPARSESPACECOLLAPSE`，供生成式声明、本地简写或条件逻辑使用。
- **L18 EN**: Includes "mlir/Dialect/SparseTensor/Transforms/Passes.h.inc" to access dialect-specific IR, transforms, or shared utilities.
  **L18 CN**: 引入 "mlir/Dialect/SparseTensor/Transforms/Passes.h.inc" 以使用方言专用 IR、变换或共享工具。

### Lines 19-36

````cpp
} // namespace mlir

#define DEBUG_TYPE "sparse-space-collapse"

using namespace mlir;
using namespace sparse_tensor;

namespace {

struct CollapseSpaceInfo {
  ExtractIterSpaceOp space;
  IterateOp loop;
};

bool isCollapsableLoops(LoopLikeOpInterface parent, LoopLikeOpInterface node) {
  auto pIterArgs = parent.getRegionIterArgs();
  auto nInitArgs = node.getInits();
  if (pIterArgs.size() != nInitArgs.size())
````
- **L19 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace mlir`.
  **L19 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace mlir`。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L21 EN**: Defines macro `DEBUG_TYPE` for generated declarations, local shorthand, or conditional logic.
  **L21 CN**: 定义宏 `DEBUG_TYPE`，供生成式声明、本地简写或条件逻辑使用。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L23 EN**: Brings namespace `mlir` into local scope.
  **L23 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L24 EN**: Brings namespace `sparse_tensor` into local scope.
  **L24 CN**: 将命名空间 `sparse_tensor` 引入当前作用域。
- **L25 EN**: Blank line separating nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L26 EN**: Opens namespace scope ``.
  **L26 CN**: 打开命名空间作用域 ``。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Declares struct `CollapseSpaceInfo`.
  **L28 CN**: 声明 struct `CollapseSpaceInfo`。
- **L29 EN**: Executes a standalone statement or declaration: `ExtractIterSpaceOp space;`.
  **L29 CN**: 执行一条独立语句或声明：`ExtractIterSpaceOp space;`。
- **L30 EN**: Executes a standalone statement or declaration: `IterateOp loop;`.
  **L30 CN**: 执行一条独立语句或声明：`IterateOp loop;`。
- **L31 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L31 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L33 EN**: Starts a function, method, lambda, or structured scope: `bool isCollapsableLoops(LoopLikeOpInterface parent, LoopLikeOpInterface node) {`.
  **L33 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool isCollapsableLoops(LoopLikeOpInterface parent, LoopLikeOpInterface node) {`。
- **L34 EN**: Initializes variable `pIterArgs` from the right-hand expression.
  **L34 CN**: 使用右侧表达式初始化变量 `pIterArgs`。
- **L35 EN**: Initializes variable `nInitArgs` from the right-hand expression.
  **L35 CN**: 使用右侧表达式初始化变量 `nInitArgs`。
- **L36 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L36 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 37-54

````cpp
    return false;

  // Two loops are collapsable if they are perfectly nested.
  auto pYields = parent.getYieldedValues();
  auto nResult = node.getLoopResults().value();

  bool yieldEq =
      llvm::all_of(llvm::zip_equal(pYields, nResult), [](auto zipped) {
        return std::get<0>(zipped) == std::get<1>(zipped);
      });

  // Parent iter_args should be passed directly to the node's init_args.
  bool iterArgEq =
      llvm::all_of(llvm::zip_equal(pIterArgs, nInitArgs), [](auto zipped) {
        return std::get<0>(zipped) == std::get<1>(zipped);
      });

  return yieldEq && iterArgEq;
````
- **L37 EN**: Returns from the current function with `false`.
  **L37 CN**: 以 `false` 从当前函数返回。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L39 EN**: Comment explains nearby logic, invariants, or intent: `Two loops are collapsable if they are perfectly nested.`.
  **L39 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Two loops are collapsable if they are perfectly nested.`。
- **L40 EN**: Initializes variable `pYields` from the right-hand expression.
  **L40 CN**: 使用右侧表达式初始化变量 `pYields`。
- **L41 EN**: Initializes variable `nResult` from the right-hand expression.
  **L41 CN**: 使用右侧表达式初始化变量 `nResult`。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Continues the surrounding expression or declaration: `bool yieldEq =`.
  **L43 CN**: 继续构造周围的表达式或声明：`bool yieldEq =`。
- **L44 EN**: Starts a function, method, lambda, or structured scope: `llvm::all_of(llvm::zip_equal(pYields, nResult), [](auto zipped) {`.
  **L44 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::all_of(llvm::zip_equal(pYields, nResult), [](auto zipped) {`。
- **L45 EN**: Returns from the current function with `std::get<0>(zipped) == std::get<1>(zipped)`.
  **L45 CN**: 以 `std::get<0>(zipped) == std::get<1>(zipped)` 从当前函数返回。
- **L46 EN**: Executes a standalone statement or declaration: `});`.
  **L46 CN**: 执行一条独立语句或声明：`});`。
- **L47 EN**: Blank line separating nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L48 EN**: Comment explains nearby logic, invariants, or intent: `Parent iter_args should be passed directly to the node's init_args.`.
  **L48 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Parent iter_args should be passed directly to the node's init_args.`。
- **L49 EN**: Continues the surrounding expression or declaration: `bool iterArgEq =`.
  **L49 CN**: 继续构造周围的表达式或声明：`bool iterArgEq =`。
- **L50 EN**: Starts a function, method, lambda, or structured scope: `llvm::all_of(llvm::zip_equal(pIterArgs, nInitArgs), [](auto zipped) {`.
  **L50 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::all_of(llvm::zip_equal(pIterArgs, nInitArgs), [](auto zipped) {`。
- **L51 EN**: Returns from the current function with `std::get<0>(zipped) == std::get<1>(zipped)`.
  **L51 CN**: 以 `std::get<0>(zipped) == std::get<1>(zipped)` 从当前函数返回。
- **L52 EN**: Executes a standalone statement or declaration: `});`.
  **L52 CN**: 执行一条独立语句或声明：`});`。
- **L53 EN**: Blank line separating nearby declarations or logic blocks.
  **L53 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L54 EN**: Returns from the current function with `yieldEq && iterArgEq`.
  **L54 CN**: 以 `yieldEq && iterArgEq` 从当前函数返回。

### Lines 55-72

````cpp
}

bool legalToCollapse(SmallVectorImpl<CollapseSpaceInfo> &toCollapse,
                     ExtractIterSpaceOp curSpace) {

  auto getIterateOpOverSpace = [](ExtractIterSpaceOp space) -> IterateOp {
    Value spaceVal = space.getExtractedSpace();
    if (spaceVal.hasOneUse())
      return llvm::dyn_cast<IterateOp>(*spaceVal.getUsers().begin());
    return nullptr;
  };

  if (toCollapse.empty()) {
    // Collapse root.
    if (auto itOp = getIterateOpOverSpace(curSpace)) {
      CollapseSpaceInfo &info = toCollapse.emplace_back();
      info.space = curSpace;
      info.loop = itOp;
````
- **L55 EN**: Closes the current lexical scope or compound statement.
  **L55 CN**: 结束当前词法作用域或复合语句块。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bool legalToCollapse(SmallVectorImpl<CollapseSpaceInfo> &toCollapse,`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`bool legalToCollapse(SmallVectorImpl<CollapseSpaceInfo> &toCollapse,`。
- **L58 EN**: Continues the surrounding expression or declaration: `ExtractIterSpaceOp curSpace) {`.
  **L58 CN**: 继续构造周围的表达式或声明：`ExtractIterSpaceOp curSpace) {`。
- **L59 EN**: Blank line separating nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L60 EN**: Starts a function, method, lambda, or structured scope: `auto getIterateOpOverSpace = [](ExtractIterSpaceOp space) -> IterateOp {`.
  **L60 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto getIterateOpOverSpace = [](ExtractIterSpaceOp space) -> IterateOp {`。
- **L61 EN**: Initializes variable `spaceVal` from the right-hand expression.
  **L61 CN**: 使用右侧表达式初始化变量 `spaceVal`。
- **L62 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L62 CN**: 开始 `if` 控制流语句并计算其条件。
- **L63 EN**: Returns from the current function with `llvm::dyn_cast<IterateOp>(*spaceVal.getUsers().begin())`.
  **L63 CN**: 以 `llvm::dyn_cast<IterateOp>(*spaceVal.getUsers().begin())` 从当前函数返回。
- **L64 EN**: Returns from the current function with `nullptr`.
  **L64 CN**: 以 `nullptr` 从当前函数返回。
- **L65 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L65 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L67 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L67 CN**: 开始 `if` 控制流语句并计算其条件。
- **L68 EN**: Comment explains nearby logic, invariants, or intent: `Collapse root.`.
  **L68 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Collapse root.`。
- **L69 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L69 CN**: 开始 `if` 控制流语句并计算其条件。
- **L70 EN**: Executes a call or declaration centered on `toCollapse.emplace_back`.
  **L70 CN**: 执行以 `toCollapse.emplace_back` 为核心的调用或声明。
- **L71 EN**: Executes a standalone statement or declaration: `info.space = curSpace;`.
  **L71 CN**: 执行一条独立语句或声明：`info.space = curSpace;`。
- **L72 EN**: Executes a standalone statement or declaration: `info.loop = itOp;`.
  **L72 CN**: 执行一条独立语句或声明：`info.loop = itOp;`。

### Lines 73-90

````cpp
      return true;
    }
    return false;
  }

  auto parent = toCollapse.back().space;
  auto pItOp = toCollapse.back().loop;
  auto nItOp = getIterateOpOverSpace(curSpace);

  // Can only collapse spaces extracted from the same tensor.
  if (parent.getTensor() != curSpace.getTensor()) {
    LLVM_DEBUG({
      llvm::dbgs()
          << "failed to collpase spaces extracted from different tensors.";
    });
    return false;
  }

````
- **L73 EN**: Returns from the current function with `true`.
  **L73 CN**: 以 `true` 从当前函数返回。
- **L74 EN**: Closes the current lexical scope or compound statement.
  **L74 CN**: 结束当前词法作用域或复合语句块。
- **L75 EN**: Returns from the current function with `false`.
  **L75 CN**: 以 `false` 从当前函数返回。
- **L76 EN**: Closes the current lexical scope or compound statement.
  **L76 CN**: 结束当前词法作用域或复合语句块。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L78 EN**: Initializes variable `parent` from the right-hand expression.
  **L78 CN**: 使用右侧表达式初始化变量 `parent`。
- **L79 EN**: Initializes variable `pItOp` from the right-hand expression.
  **L79 CN**: 使用右侧表达式初始化变量 `pItOp`。
- **L80 EN**: Initializes variable `nItOp` from the right-hand expression.
  **L80 CN**: 使用右侧表达式初始化变量 `nItOp`。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L82 EN**: Comment explains nearby logic, invariants, or intent: `Can only collapse spaces extracted from the same tensor.`.
  **L82 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Can only collapse spaces extracted from the same tensor.`。
- **L83 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L83 CN**: 开始 `if` 控制流语句并计算其条件。
- **L84 EN**: Starts a function, method, lambda, or structured scope: `LLVM_DEBUG({`.
  **L84 CN**: 开始一个函数、方法、lambda 或结构化作用域：`LLVM_DEBUG({`。
- **L85 EN**: Continues logic associated with callable symbol `dbgs`.
  **L85 CN**: 继续与可调用符号 `dbgs` 相关的逻辑。
- **L86 EN**: Executes a standalone statement or declaration: `<< "failed to collpase spaces extracted from different tensors.";`.
  **L86 CN**: 执行一条独立语句或声明：`<< "failed to collpase spaces extracted from different tensors.";`。
- **L87 EN**: Executes a standalone statement or declaration: `});`.
  **L87 CN**: 执行一条独立语句或声明：`});`。
- **L88 EN**: Returns from the current function with `false`.
  **L88 CN**: 以 `false` 从当前函数返回。
- **L89 EN**: Closes the current lexical scope or compound statement.
  **L89 CN**: 结束当前词法作用域或复合语句块。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 91-108

````cpp
  // Can only collapse consecutive simple iteration on one tensor (i.e., no
  // coiteration).
  if (!nItOp || nItOp->getBlock() != curSpace->getBlock() ||
      pItOp.getIterator() != curSpace.getParentIter() ||
      curSpace->getParentOp() != pItOp.getOperation()) {
    LLVM_DEBUG(
        { llvm::dbgs() << "failed to collapse non-consecutive IterateOps."; });
    return false;
  }

  if (pItOp && !isCollapsableLoops(pItOp, nItOp)) {
    LLVM_DEBUG({
      llvm::dbgs()
          << "failed to collapse IterateOps that are not perfectly nested.";
    });
    return false;
  }

````
- **L91 EN**: Comment explains nearby logic, invariants, or intent: `Can only collapse consecutive simple iteration on one tensor (i.e., no`.
  **L91 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Can only collapse consecutive simple iteration on one tensor (i.e., no`。
- **L92 EN**: Comment explains nearby logic, invariants, or intent: `coiteration).`.
  **L92 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`coiteration).`。
- **L93 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L93 CN**: 开始 `if` 控制流语句并计算其条件。
- **L94 EN**: Continues logic associated with callable symbol `getIterator`.
  **L94 CN**: 继续与可调用符号 `getIterator` 相关的逻辑。
- **L95 EN**: Starts a function, method, lambda, or structured scope: `curSpace->getParentOp() != pItOp.getOperation()) {`.
  **L95 CN**: 开始一个函数、方法、lambda 或结构化作用域：`curSpace->getParentOp() != pItOp.getOperation()) {`。
- **L96 EN**: Continues logic associated with callable symbol `LLVM_DEBUG`.
  **L96 CN**: 继续与可调用符号 `LLVM_DEBUG` 相关的逻辑。
- **L97 EN**: Executes a call or declaration centered on `llvm::dbgs`.
  **L97 CN**: 执行以 `llvm::dbgs` 为核心的调用或声明。
- **L98 EN**: Returns from the current function with `false`.
  **L98 CN**: 以 `false` 从当前函数返回。
- **L99 EN**: Closes the current lexical scope or compound statement.
  **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L101 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L101 CN**: 开始 `if` 控制流语句并计算其条件。
- **L102 EN**: Starts a function, method, lambda, or structured scope: `LLVM_DEBUG({`.
  **L102 CN**: 开始一个函数、方法、lambda 或结构化作用域：`LLVM_DEBUG({`。
- **L103 EN**: Continues logic associated with callable symbol `dbgs`.
  **L103 CN**: 继续与可调用符号 `dbgs` 相关的逻辑。
- **L104 EN**: Executes a standalone statement or declaration: `<< "failed to collapse IterateOps that are not perfectly nested.";`.
  **L104 CN**: 执行一条独立语句或声明：`<< "failed to collapse IterateOps that are not perfectly nested.";`。
- **L105 EN**: Executes a standalone statement or declaration: `});`.
  **L105 CN**: 执行一条独立语句或声明：`});`。
- **L106 EN**: Returns from the current function with `false`.
  **L106 CN**: 以 `false` 从当前函数返回。
- **L107 EN**: Closes the current lexical scope or compound statement.
  **L107 CN**: 结束当前词法作用域或复合语句块。
- **L108 EN**: Blank line separating nearby declarations or logic blocks.
  **L108 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 109-126

````cpp
  CollapseSpaceInfo &info = toCollapse.emplace_back();
  info.space = curSpace;
  info.loop = nItOp;
  return true;
}

void collapseSparseSpace(MutableArrayRef<CollapseSpaceInfo> toCollapse) {
  if (toCollapse.size() < 2)
    return;

  ExtractIterSpaceOp root = toCollapse.front().space;
  ExtractIterSpaceOp leaf = toCollapse.back().space;
  Location loc = root.getLoc();

  assert(root->hasOneUse() && leaf->hasOneUse());

  // Insert collapsed operation at the same scope as root operation.
  OpBuilder builder(root);
````
- **L109 EN**: Executes a call or declaration centered on `toCollapse.emplace_back`.
  **L109 CN**: 执行以 `toCollapse.emplace_back` 为核心的调用或声明。
- **L110 EN**: Executes a standalone statement or declaration: `info.space = curSpace;`.
  **L110 CN**: 执行一条独立语句或声明：`info.space = curSpace;`。
- **L111 EN**: Executes a standalone statement or declaration: `info.loop = nItOp;`.
  **L111 CN**: 执行一条独立语句或声明：`info.loop = nItOp;`。
- **L112 EN**: Returns from the current function with `true`.
  **L112 CN**: 以 `true` 从当前函数返回。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L115 EN**: Starts a function, method, lambda, or structured scope: `void collapseSparseSpace(MutableArrayRef<CollapseSpaceInfo> toCollapse) {`.
  **L115 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void collapseSparseSpace(MutableArrayRef<CollapseSpaceInfo> toCollapse) {`。
- **L116 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L116 CN**: 开始 `if` 控制流语句并计算其条件。
- **L117 EN**: Returns from the current function with `void`.
  **L117 CN**: 以 `void` 从当前函数返回。
- **L118 EN**: Blank line separating nearby declarations or logic blocks.
  **L118 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L119 EN**: Initializes variable `root` from the right-hand expression.
  **L119 CN**: 使用右侧表达式初始化变量 `root`。
- **L120 EN**: Initializes variable `leaf` from the right-hand expression.
  **L120 CN**: 使用右侧表达式初始化变量 `leaf`。
- **L121 EN**: Initializes variable `loc` from the right-hand expression.
  **L121 CN**: 使用右侧表达式初始化变量 `loc`。
- **L122 EN**: Blank line separating nearby declarations or logic blocks.
  **L122 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L123 EN**: Checks an internal invariant in debug builds.
  **L123 CN**: 在调试构建中检查内部不变式。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L125 EN**: Comment explains nearby logic, invariants, or intent: `Insert collapsed operation at the same scope as root operation.`.
  **L125 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Insert collapsed operation at the same scope as root operation.`。
- **L126 EN**: Executes a call or declaration centered on `builder`.
  **L126 CN**: 执行以 `builder` 为核心的调用或声明。

### Lines 127-144

````cpp

  // Construct the collapsed iteration space.
  auto collapsedSpace = ExtractIterSpaceOp::create(
      builder, loc, root.getTensor(), root.getParentIter(), root.getLoLvl(),
      leaf.getHiLvl());

  auto rItOp = llvm::cast<IterateOp>(*root->getUsers().begin());
  auto innermost = toCollapse.back().loop;

  IRMapping mapper;
  mapper.map(leaf, collapsedSpace.getExtractedSpace());
  for (auto z : llvm::zip_equal(innermost.getInitArgs(), rItOp.getInitArgs()))
    mapper.map(std::get<0>(z), std::get<1>(z));

  auto cloned = llvm::cast<IterateOp>(builder.clone(*innermost, mapper));
  builder.setInsertionPointToStart(cloned.getBody());

  I64BitSet crdUsedLvls;
````
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L128 EN**: Comment explains nearby logic, invariants, or intent: `Construct the collapsed iteration space.`.
  **L128 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Construct the collapsed iteration space.`。
- **L129 EN**: Continues logic associated with callable symbol `create`.
  **L129 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, root.getTensor(), root.getParentIter(), root.getLoLvl(),`.
  **L130 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, root.getTensor(), root.getParentIter(), root.getLoLvl(),`。
- **L131 EN**: Executes a call or declaration centered on `leaf.getHiLvl`.
  **L131 CN**: 执行以 `leaf.getHiLvl` 为核心的调用或声明。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L133 EN**: Initializes variable `rItOp` from the right-hand expression.
  **L133 CN**: 使用右侧表达式初始化变量 `rItOp`。
- **L134 EN**: Initializes variable `innermost` from the right-hand expression.
  **L134 CN**: 使用右侧表达式初始化变量 `innermost`。
- **L135 EN**: Blank line separating nearby declarations or logic blocks.
  **L135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L136 EN**: Executes a standalone statement or declaration: `IRMapping mapper;`.
  **L136 CN**: 执行一条独立语句或声明：`IRMapping mapper;`。
- **L137 EN**: Executes a call or declaration centered on `mapper.map`.
  **L137 CN**: 执行以 `mapper.map` 为核心的调用或声明。
- **L138 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L138 CN**: 开始 `for` 控制流语句并计算其条件。
- **L139 EN**: Executes a call or declaration centered on `mapper.map`.
  **L139 CN**: 执行以 `mapper.map` 为核心的调用或声明。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L141 EN**: Initializes variable `cloned` from the right-hand expression.
  **L141 CN**: 使用右侧表达式初始化变量 `cloned`。
- **L142 EN**: Executes a call or declaration centered on `builder.setInsertionPointToStart`.
  **L142 CN**: 执行以 `builder.setInsertionPointToStart` 为核心的调用或声明。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L144 EN**: Executes a standalone statement or declaration: `I64BitSet crdUsedLvls;`.
  **L144 CN**: 执行一条独立语句或声明：`I64BitSet crdUsedLvls;`。

### Lines 145-162

````cpp
  unsigned shift = 0, argIdx = 1;
  for (auto info : toCollapse.drop_back()) {
    I64BitSet set = info.loop.getCrdUsedLvls();
    crdUsedLvls |= set.lshift(shift);
    shift += info.loop.getSpaceDim();
    for (BlockArgument crd : info.loop.getCrds()) {
      BlockArgument collapsedCrd = cloned.getBody()->insertArgument(
          argIdx++, builder.getIndexType(), crd.getLoc());
      crd.replaceAllUsesWith(collapsedCrd);
    }
  }
  crdUsedLvls |= innermost.getCrdUsedLvls().lshift(shift);
  cloned.getIterator().setType(collapsedSpace.getType().getIteratorType());
  cloned.setCrdUsedLvls(crdUsedLvls);

  rItOp.replaceAllUsesWith(cloned.getResults());
  // Erase collapsed loops.
  rItOp.erase();
````
- **L145 EN**: Initializes variable `shift` from the right-hand expression.
  **L145 CN**: 使用右侧表达式初始化变量 `shift`。
- **L146 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L146 CN**: 开始 `for` 控制流语句并计算其条件。
- **L147 EN**: Initializes variable `set` from the right-hand expression.
  **L147 CN**: 使用右侧表达式初始化变量 `set`。
- **L148 EN**: Executes a call or declaration centered on `set.lshift`.
  **L148 CN**: 执行以 `set.lshift` 为核心的调用或声明。
- **L149 EN**: Executes a call or declaration centered on `info.loop.getSpaceDim`.
  **L149 CN**: 执行以 `info.loop.getSpaceDim` 为核心的调用或声明。
- **L150 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L150 CN**: 开始 `for` 控制流语句并计算其条件。
- **L151 EN**: Continues logic associated with callable symbol `getBody`.
  **L151 CN**: 继续与可调用符号 `getBody` 相关的逻辑。
- **L152 EN**: Executes a call or declaration centered on `builder.getIndexType`.
  **L152 CN**: 执行以 `builder.getIndexType` 为核心的调用或声明。
- **L153 EN**: Executes a call or declaration centered on `crd.replaceAllUsesWith`.
  **L153 CN**: 执行以 `crd.replaceAllUsesWith` 为核心的调用或声明。
- **L154 EN**: Closes the current lexical scope or compound statement.
  **L154 CN**: 结束当前词法作用域或复合语句块。
- **L155 EN**: Closes the current lexical scope or compound statement.
  **L155 CN**: 结束当前词法作用域或复合语句块。
- **L156 EN**: Executes a call or declaration centered on `innermost.getCrdUsedLvls`.
  **L156 CN**: 执行以 `innermost.getCrdUsedLvls` 为核心的调用或声明。
- **L157 EN**: Executes a call or declaration centered on `cloned.getIterator`.
  **L157 CN**: 执行以 `cloned.getIterator` 为核心的调用或声明。
- **L158 EN**: Executes a call or declaration centered on `cloned.setCrdUsedLvls`.
  **L158 CN**: 执行以 `cloned.setCrdUsedLvls` 为核心的调用或声明。
- **L159 EN**: Blank line separating nearby declarations or logic blocks.
  **L159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L160 EN**: Executes a call or declaration centered on `rItOp.replaceAllUsesWith`.
  **L160 CN**: 执行以 `rItOp.replaceAllUsesWith` 为核心的调用或声明。
- **L161 EN**: Comment explains nearby logic, invariants, or intent: `Erase collapsed loops.`.
  **L161 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Erase collapsed loops.`。
- **L162 EN**: Executes a call or declaration centered on `rItOp.erase`.
  **L162 CN**: 执行以 `rItOp.erase` 为核心的调用或声明。

### Lines 163-180

````cpp
  root.erase();
}

struct SparseSpaceCollapsePass
    : public impl::SparseSpaceCollapseBase<SparseSpaceCollapsePass> {
  SparseSpaceCollapsePass() = default;

  void runOnOperation() override {
    func::FuncOp func = getOperation();

    // A naive (experimental) implementation to collapse consecutive sparse
    // spaces. It does NOT handle complex cases where multiple spaces are
    // extracted in the same basic block. E.g.,
    //
    // %space1 = extract_space %t1 ...
    // %space2 = extract_space %t2 ...
    // sparse_tensor.iterate(%sp1) ...
    //
````
- **L163 EN**: Executes a call or declaration centered on `root.erase`.
  **L163 CN**: 执行以 `root.erase` 为核心的调用或声明。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L166 EN**: Declares struct `SparseSpaceCollapsePass`.
  **L166 CN**: 声明 struct `SparseSpaceCollapsePass`。
- **L167 EN**: Continues the surrounding expression or declaration: `: public impl::SparseSpaceCollapseBase<SparseSpaceCollapsePass> {`.
  **L167 CN**: 继续构造周围的表达式或声明：`: public impl::SparseSpaceCollapseBase<SparseSpaceCollapsePass> {`。
- **L168 EN**: Executes a call or declaration centered on `SparseSpaceCollapsePass`.
  **L168 CN**: 执行以 `SparseSpaceCollapsePass` 为核心的调用或声明。
- **L169 EN**: Blank line separating nearby declarations or logic blocks.
  **L169 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L170 EN**: Starts a function, method, lambda, or structured scope: `void runOnOperation() override {`.
  **L170 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void runOnOperation() override {`。
- **L171 EN**: Initializes variable `func` from the right-hand expression.
  **L171 CN**: 使用右侧表达式初始化变量 `func`。
- **L172 EN**: Blank line separating nearby declarations or logic blocks.
  **L172 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L173 EN**: Comment explains nearby logic, invariants, or intent: `A naive (experimental) implementation to collapse consecutive sparse`.
  **L173 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A naive (experimental) implementation to collapse consecutive sparse`。
- **L174 EN**: Comment explains nearby logic, invariants, or intent: `spaces. It does NOT handle complex cases where multiple spaces are`.
  **L174 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`spaces. It does NOT handle complex cases where multiple spaces are`。
- **L175 EN**: Comment explains nearby logic, invariants, or intent: `extracted in the same basic block. E.g.,`.
  **L175 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`extracted in the same basic block. E.g.,`。
- **L176 EN**: Separator comment used for visual grouping.
  **L176 CN**: 用于视觉分组的分隔注释。
- **L177 EN**: Comment explains nearby logic, invariants, or intent: `%space1 = extract_space %t1 ...`.
  **L177 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%space1 = extract_space %t1 ...`。
- **L178 EN**: Comment explains nearby logic, invariants, or intent: `%space2 = extract_space %t2 ...`.
  **L178 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`%space2 = extract_space %t2 ...`。
- **L179 EN**: Comment explains nearby logic, invariants, or intent: `sparse_tensor.iterate(%sp1) ...`.
  **L179 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sparse_tensor.iterate(%sp1) ...`。
- **L180 EN**: Separator comment used for visual grouping.
  **L180 CN**: 用于视觉分组的分隔注释。

### Lines 181-198

````cpp
    // Collect all groups to collapse before performing any IR mutations.
    // Mutating (erasing) ops during the walk would invalidate the walk's
    // internal iterator and cause use-after-free crashes.
    SmallVector<SmallVector<CollapseSpaceInfo>> groups;
    SmallVector<CollapseSpaceInfo> toCollapse;
    func->walk([&](ExtractIterSpaceOp op) {
      if (!legalToCollapse(toCollapse, op)) {
        // Save the current group and start a new one.
        groups.push_back(std::move(toCollapse));
        toCollapse.clear();
        // Try to start a new group with the current op.
        legalToCollapse(toCollapse, op);
      }
    });
    groups.push_back(std::move(toCollapse));

    // Apply all collapse transformations after the walk is complete.
    for (auto &group : groups)
````
- **L181 EN**: Comment explains nearby logic, invariants, or intent: `Collect all groups to collapse before performing any IR mutations.`.
  **L181 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Collect all groups to collapse before performing any IR mutations.`。
- **L182 EN**: Comment explains nearby logic, invariants, or intent: `Mutating (erasing) ops during the walk would invalidate the walk's`.
  **L182 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Mutating (erasing) ops during the walk would invalidate the walk's`。
- **L183 EN**: Comment explains nearby logic, invariants, or intent: `internal iterator and cause use-after-free crashes.`.
  **L183 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`internal iterator and cause use-after-free crashes.`。
- **L184 EN**: Executes a standalone statement or declaration: `SmallVector<SmallVector<CollapseSpaceInfo>> groups;`.
  **L184 CN**: 执行一条独立语句或声明：`SmallVector<SmallVector<CollapseSpaceInfo>> groups;`。
- **L185 EN**: Executes a standalone statement or declaration: `SmallVector<CollapseSpaceInfo> toCollapse;`.
  **L185 CN**: 执行一条独立语句或声明：`SmallVector<CollapseSpaceInfo> toCollapse;`。
- **L186 EN**: Starts a function, method, lambda, or structured scope: `func->walk([&](ExtractIterSpaceOp op) {`.
  **L186 CN**: 开始一个函数、方法、lambda 或结构化作用域：`func->walk([&](ExtractIterSpaceOp op) {`。
- **L187 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L187 CN**: 开始 `if` 控制流语句并计算其条件。
- **L188 EN**: Comment explains nearby logic, invariants, or intent: `Save the current group and start a new one.`.
  **L188 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Save the current group and start a new one.`。
- **L189 EN**: Executes a call or declaration centered on `groups.push_back`.
  **L189 CN**: 执行以 `groups.push_back` 为核心的调用或声明。
- **L190 EN**: Executes a call or declaration centered on `toCollapse.clear`.
  **L190 CN**: 执行以 `toCollapse.clear` 为核心的调用或声明。
- **L191 EN**: Comment explains nearby logic, invariants, or intent: `Try to start a new group with the current op.`.
  **L191 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Try to start a new group with the current op.`。
- **L192 EN**: Executes a call or declaration centered on `legalToCollapse`.
  **L192 CN**: 执行以 `legalToCollapse` 为核心的调用或声明。
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Executes a standalone statement or declaration: `});`.
  **L194 CN**: 执行一条独立语句或声明：`});`。
- **L195 EN**: Executes a call or declaration centered on `groups.push_back`.
  **L195 CN**: 执行以 `groups.push_back` 为核心的调用或声明。
- **L196 EN**: Blank line separating nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L197 EN**: Comment explains nearby logic, invariants, or intent: `Apply all collapse transformations after the walk is complete.`.
  **L197 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Apply all collapse transformations after the walk is complete.`。
- **L198 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L198 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 199-207

````cpp
      collapseSparseSpace(group);
  }
};

} // namespace

std::unique_ptr<Pass> mlir::createSparseSpaceCollapsePass() {
  return std::make_unique<SparseSpaceCollapsePass>();
}
````
- **L199 EN**: Executes a call or declaration centered on `collapseSparseSpace`.
  **L199 CN**: 执行以 `collapseSparseSpace` 为核心的调用或声明。
- **L200 EN**: Closes the current lexical scope or compound statement.
  **L200 CN**: 结束当前词法作用域或复合语句块。
- **L201 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L201 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L202 EN**: Blank line separating nearby declarations or logic blocks.
  **L202 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L203 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L203 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L204 EN**: Blank line separating nearby declarations or logic blocks.
  **L204 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L205 EN**: Starts a function, method, lambda, or structured scope: `std::unique_ptr<Pass> mlir::createSparseSpaceCollapsePass() {`.
  **L205 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::unique_ptr<Pass> mlir::createSparseSpaceCollapsePass() {`。
- **L206 EN**: Returns from the current function with `std::make_unique<SparseSpaceCollapsePass>()`.
  **L206 CN**: 以 `std::make_unique<SparseSpaceCollapsePass>()` 从当前函数返回。
- **L207 EN**: Closes the current lexical scope or compound statement.
  **L207 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Sparse tensor compilation / 稀疏张量编译**
- **Rewrite-driven lowering / 基于重写的 lowering**
- **Dialect definition and registration / 方言定义与注册**
- **Operation semantics and verification / 操作语义与验证**
- **Type-system modeling / 类型系统建模**
- **Region-based control structure / 基于 Region 的控制结构**
- **Basic block ownership and traversal / 基本块拥有关系与遍历**
- **SSA value representation / SSA 值表示**
- **Tensor-level abstraction / 张量层抽象**
- **Transform dialect orchestration / Transform 方言编排**

## Dependencies / 依赖关系

- `mlir/Dialect/Func/IR/FuncOps.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/IR/IRMapping.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `mlir/Transforms/Passes.h`: Provides generic transformation utilities and canonicalization helpers. / 提供通用变换工具与规范化辅助逻辑。
- `mlir/Dialect/SparseTensor/IR/SparseTensor.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/SparseTensor/Transforms/Passes.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/SparseTensor/Transforms/Passes.h.inc`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
