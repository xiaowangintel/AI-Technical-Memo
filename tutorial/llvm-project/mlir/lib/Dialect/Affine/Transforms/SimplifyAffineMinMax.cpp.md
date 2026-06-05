# SimplifyAffineMinMax.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Transforms/SimplifyAffineMinMax.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements a transform to simplify mix/max affine operations.
  - **CN**: 实现 Affine 方言与仿射循环推理 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- SimplifyAffineMinMax.cpp - Simplify affine min/max ops -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-14
```cpp
//
// This file implements a transform to simplify mix/max affine operations.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Affine/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Transforms/Passes.h`。

### Lines 15-24
```cpp
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/Transforms/Transforms.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Interfaces/FunctionInterfaces.h"
#include "mlir/Interfaces/ValueBoundsOpInterface.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/ADT/IntEqClasses.h"
#include "llvm/Support/DebugLog.h"
#include "llvm/Support/InterleavedRange.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`, `mlir/Interfaces/FunctionInterfaces.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`, `mlir/Interfaces/FunctionInterfaces.h`。

### Lines 25-31
```cpp
#define DEBUG_TYPE "affine-min-max"

using namespace mlir;
using namespace mlir::affine;

/// Simplifies an affine min/max operation by proving there's a lower or upper
/// bound.
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or generated-code switches.
- **CN**: 定义受预处理器控制的结构、特性开关或生成代码开关。

### Lines 32-40
```cpp
template <typename AffineOp>
static bool simplifyAffineMinMaxOp(RewriterBase &rewriter, AffineOp affineOp) {
  using Variable = ValueBoundsConstraintSet::Variable;
  using ComparisonOperator = ValueBoundsConstraintSet::ComparisonOperator;

  AffineMap affineMap = affineOp.getMap();
  ValueRange operands = affineOp.getOperands();
  static constexpr bool isMin = std::is_same_v<AffineOp, AffineMinOp>;

```
- **EN**: Implements logic around `simplifyAffineMinMaxOp`, `getMap`, `getOperands`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `simplifyAffineMinMaxOp`, `getMap`, `getOperands` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 41-53
```cpp
  LDBG() << "analyzing value: `" << affineOp;

  // Create a `Variable` list with values corresponding to each of the results
  // in the affine affineMap.
  SmallVector<Variable> variables = llvm::map_to_vector(
      llvm::iota_range<unsigned>(0u, affineMap.getNumResults(), false),
      [&](unsigned i) {
        return Variable(affineMap.getSliceMap(i, 1), operands);
      });
  LDBG() << "- constructed variables are: "
         << llvm::interleaved_array(llvm::map_range(
                variables, [](const Variable &v) { return v.getMap(); }));

```
- **EN**: Implements logic around `LDBG`, `map_to_vector`, `iota_range`, `Variable`, and 2 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `LDBG`, `map_to_vector`, `iota_range`, `Variable`, and 2 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 54-63
```cpp
  // Get the comparison operation.
  ComparisonOperator cmpOp =
      isMin ? ComparisonOperator::LT : ComparisonOperator::GT;

  // Find disjoint sets bounded by a common value.
  llvm::IntEqClasses boundedClasses(variables.size());
  DenseMap<unsigned, Variable *> bounds;
  for (auto &&[i, v] : llvm::enumerate(variables)) {
    unsigned eqClass = boundedClasses.findLeader(i);

```
- **EN**: Implements logic around `boundedClasses`, `enumerate`, `findLeader`.
- **CN**: 围绕 `boundedClasses`, `enumerate`, `findLeader` 实现具体逻辑。

### Lines 64-70
```cpp
    // If the class already has a bound continue.
    if (bounds.contains(eqClass))
      continue;

    // Initialize the bound.
    Variable *bound = &v;

```
- **EN**: Implements logic around `contains`.
- **CN**: 围绕 `contains` 实现具体逻辑。

### Lines 71-80
```cpp
    LDBG() << "- inspecting variable: #" << i << ", with map: `" << v.getMap()
           << "`\n";

    // Check against the other variables.
    for (size_t j = i + 1; j < variables.size(); ++j) {
      unsigned jEqClass = boundedClasses.findLeader(j);
      // Skip if the class is the same.
      if (jEqClass == eqClass)
        continue;

```
- **EN**: Implements logic around `LDBG`, `size`, `findLeader`.
- **CN**: 围绕 `LDBG`, `size`, `findLeader` 实现具体逻辑。

### Lines 81-90
```cpp
      // Get the bound of the equivalence class or itself.
      Variable *nv = bounds.lookup_or(jEqClass, &variables[j]);

      LDBG() << "- comparing with variable: #" << jEqClass
             << ", with map: " << nv->getMap();

      // Compare the variables.
      FailureOr<bool> cmpResult =
          ValueBoundsConstraintSet::strongCompare(*bound, cmpOp, *nv);

```
- **EN**: Implements logic around `lookup_or`, `LDBG`, `getMap`, `strongCompare`.
- **CN**: 围绕 `lookup_or`, `LDBG`, `getMap`, `strongCompare` 实现具体逻辑。

### Lines 91-97
```cpp
      // The variables cannot be compared.
      if (failed(cmpResult)) {
        LDBG() << "-- classes: #" << i << ", #" << jEqClass
               << " cannot be merged";
        continue;
      }

```
- **EN**: Implements logic around `failed`, `LDBG`.
- **CN**: 围绕 `failed`, `LDBG` 实现具体逻辑。

### Lines 98-111
```cpp
      // Join the equivalent classes and update the bound if necessary.
      LDBG() << "-- merging classes: #" << i << ", #" << jEqClass
             << ", is cmp(lhs, rhs): " << *cmpResult << "`";
      if (*cmpResult) {
        boundedClasses.join(eqClass, jEqClass);
      } else {
        // In this case we have lhs > rhs if isMin == true, or lhs < rhs if
        // isMin == false.
        bound = nv;
        boundedClasses.join(eqClass, jEqClass);
      }
    }
    bounds[boundedClasses.findLeader(i)] = bound;
  }
```
- **EN**: Implements logic around `LDBG`, `cmp`, `join`, `findLeader`.
- **CN**: 围绕 `LDBG`, `cmp`, `join`, `findLeader` 实现具体逻辑。

### Lines 112-118
```cpp

  // Return if there's no simplification.
  if (bounds.size() >= affineMap.getNumResults()) {
    LDBG() << "- the affine operation couldn't get simplified";
    return false;
  }

```
- **EN**: Implements logic around `size`, `LDBG`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `size`, `LDBG` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 119-130
```cpp
  // Construct the new affine affineMap.
  SmallVector<AffineExpr> results;
  results.reserve(bounds.size());
  for (auto [k, bound] : bounds)
    results.push_back(bound->getMap().getResult(0));

  LDBG() << "- starting from map: " << affineMap;
  LDBG() << "- creating new map with:";
  LDBG() << "--- dims: " << affineMap.getNumDims();
  LDBG() << "--- syms: " << affineMap.getNumSymbols();
  LDBG() << "--- res: " << llvm::interleaved_array(results);

```
- **EN**: Implements logic around `reserve`, `push_back`, `LDBG`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `reserve`, `push_back`, `LDBG` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 131-140
```cpp
  affineMap =
      AffineMap::get(0, affineMap.getNumSymbols() + affineMap.getNumDims(),
                     results, rewriter.getContext());

  // Update the affine op.
  rewriter.modifyOpInPlace(affineOp, [&]() { affineOp.setMap(affineMap); });
  LDBG() << "- simplified affine op: `" << affineOp << "`";
  return true;
}

```
- **EN**: Implements logic around `get`, `getContext`, `modifyOpInPlace`, `LDBG`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `get`, `getContext`, `modifyOpInPlace`, `LDBG` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 141-148
```cpp
bool mlir::affine::simplifyAffineMinOp(RewriterBase &rewriter, AffineMinOp op) {
  return simplifyAffineMinMaxOp(rewriter, op);
}

bool mlir::affine::simplifyAffineMaxOp(RewriterBase &rewriter, AffineMaxOp op) {
  return simplifyAffineMinMaxOp(rewriter, op);
}

```
- **EN**: Implements logic around `simplifyAffineMinOp`, `simplifyAffineMinMaxOp`, `simplifyAffineMaxOp`; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `simplifyAffineMinOp`, `simplifyAffineMinMaxOp`, `simplifyAffineMaxOp` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 149-162
```cpp
LogicalResult mlir::affine::simplifyAffineMinMaxOps(RewriterBase &rewriter,
                                                    ArrayRef<Operation *> ops,
                                                    bool *modified) {
  bool changed = false;
  for (Operation *op : ops) {
    if (auto minOp = dyn_cast<AffineMinOp>(op)) {
      changed = simplifyAffineMinOp(rewriter, minOp) || changed;
      continue;
    }
    auto maxOp = cast<AffineMaxOp>(op);
    changed = simplifyAffineMaxOp(rewriter, maxOp) || changed;
  }
  RewritePatternSet patterns(rewriter.getContext());
  AffineMaxOp::getCanonicalizationPatterns(patterns, rewriter.getContext());
```
- **EN**: Implements logic around `simplifyAffineMinMaxOps`, `dyn_cast`, `simplifyAffineMinOp`, `cast`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; registers dialect entities or dialect-level hooks; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `simplifyAffineMinMaxOps`, `dyn_cast`, `simplifyAffineMinOp`, `cast`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并注册方言实体或方言级钩子，并执行仿射推理或形状/边界处理。

### Lines 163-176
```cpp
  AffineMinOp::getCanonicalizationPatterns(patterns, rewriter.getContext());
  FrozenRewritePatternSet frozenPatterns(std::move(patterns));
  if (modified)
    *modified = changed;
  // Canonicalize to a fixpoint.
  if (failed(applyOpPatternsGreedily(
          ops, frozenPatterns,
          GreedyRewriteConfig()
              .setListener(
                  static_cast<RewriterBase::Listener *>(rewriter.getListener()))
              .setStrictness(GreedyRewriteStrictness::ExistingAndNewOps),
          &changed))) {
    return failure();
  }
```
- **EN**: Implements logic around `getCanonicalizationPatterns`, `frozenPatterns`, `failed`, `GreedyRewriteConfig`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `getCanonicalizationPatterns`, `frozenPatterns`, `failed`, `GreedyRewriteConfig`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并注册方言实体或方言级钩子。

### Lines 177-183
```cpp
  if (modified)
    *modified = changed;
  return success();
}

namespace {

```
- **EN**: Implements logic around `success`.
- **CN**: 围绕 `success` 实现具体逻辑。

### Lines 184-192
```cpp
struct SimplifyAffineMaxOp : public OpRewritePattern<AffineMaxOp> {
  using OpRewritePattern<AffineMaxOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(AffineMaxOp affineOp,
                                PatternRewriter &rewriter) const override {
    return success(simplifyAffineMaxOp(rewriter, affineOp));
  }
};

```
- **EN**: Introduces declarations for `SimplifyAffineMaxOp`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SimplifyAffineMaxOp` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 193-201
```cpp
struct SimplifyAffineMinOp : public OpRewritePattern<AffineMinOp> {
  using OpRewritePattern<AffineMinOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(AffineMinOp affineOp,
                                PatternRewriter &rewriter) const override {
    return success(simplifyAffineMinOp(rewriter, affineOp));
  }
};

```
- **EN**: Introduces declarations for `SimplifyAffineMinOp`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SimplifyAffineMinOp` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 202-212
```cpp
struct SimplifyAffineApplyOp : public OpRewritePattern<AffineApplyOp> {
  using OpRewritePattern<AffineApplyOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(AffineApplyOp affineOp,
                                PatternRewriter &rewriter) const override {
    AffineMap map = affineOp.getAffineMap();
    SmallVector<Value> operands{affineOp->getOperands().begin(),
                                affineOp->getOperands().end()};
    fullyComposeAffineMapAndOperands(&map, &operands,
                                     /*composeAffineMin=*/true);

```
- **EN**: Introduces declarations for `SimplifyAffineApplyOp`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SimplifyAffineApplyOp` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 213-224
```cpp
    // No change => failure to apply.
    if (map == affineOp.getAffineMap())
      return failure();

    rewriter.modifyOpInPlace(affineOp, [&]() {
      affineOp.setMap(map);
      affineOp->setOperands(operands);
    });
    return success();
  }
};

```
- **EN**: Implements logic around `getAffineMap`, `failure`, `modifyOpInPlace`, `setMap`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getAffineMap`, `failure`, `modifyOpInPlace`, `setMap`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 225-233
```cpp
} // namespace

namespace mlir {
namespace affine {
#define GEN_PASS_DEF_SIMPLIFYAFFINEMINMAXPASS
#include "mlir/Dialect/Affine/Transforms/Passes.h.inc"
} // namespace affine
} // namespace mlir

```
- **EN**: Introduces declarations for `mlir`, `affine`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `affine` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 234-240
```cpp
/// Creates a simplification pass for affine min/max/apply.
struct SimplifyAffineMinMaxPass
    : public affine::impl::SimplifyAffineMinMaxPassBase<
          SimplifyAffineMinMaxPass> {
  void runOnOperation() override;
};

```
- **EN**: Introduces declarations for `SimplifyAffineMinMaxPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SimplifyAffineMinMaxPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 241-251
```cpp
void SimplifyAffineMinMaxPass::runOnOperation() {
  FunctionOpInterface func = getOperation();
  RewritePatternSet patterns(func.getContext());
  AffineMaxOp::getCanonicalizationPatterns(patterns, func.getContext());
  AffineMinOp::getCanonicalizationPatterns(patterns, func.getContext());
  patterns.add<SimplifyAffineMaxOp, SimplifyAffineMinOp, SimplifyAffineApplyOp>(
      func.getContext());
  FrozenRewritePatternSet frozenPatterns(std::move(patterns));
  if (failed(applyPatternsGreedily(func, frozenPatterns)))
    return signalPassFailure();
}
```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `patterns`, `getCanonicalizationPatterns`, and 5 more symbols; this block participates in pass execution or pass construction; registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `patterns`, `getCanonicalizationPatterns`, and 5 more symbols 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并注册方言实体或方言级钩子。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Affine reasoning / 仿射推理**:
  - **EN**: Works with affine maps, bounds, dependence reasoning, and loop structure.
  - **CN**: 处理仿射映射、边界、依赖推理与循环结构。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`, `mlir/Interfaces/FunctionInterfaces.h`, `mlir/Interfaces/ValueBoundsOpInterface.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `llvm/ADT/IntEqClasses.h`, `llvm/Support/DebugLog.h`, `llvm/Support/InterleavedRange.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (4), MLIR interface declarations / MLIR 接口声明 (2), LLVM support-library helpers / LLVM Support 库辅助功能 (2), MLIR IR core abstractions / MLIR IR 核心抽象 (1), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
