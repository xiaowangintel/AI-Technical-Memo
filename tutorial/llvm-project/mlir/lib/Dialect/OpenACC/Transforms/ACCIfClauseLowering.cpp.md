# ACCIfClauseLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenACC/Transforms/ACCIfClauseLowering.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This pass lowers OpenACC compute constructs (parallel, kernels, serial) with `if` clauses using region specialization. It creates two execution paths: device execution when the condition is true, host execution when false.
  - **CN**: 该文件位于 `mlir/lib/Dialect/OpenACC/Transforms`，围绕 OpenACC 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ACCIfClauseLowering.cpp - Lower ACC compute construct if clauses --===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-21
```cpp
//
// This pass lowers OpenACC compute constructs (parallel, kernels, serial) with
// `if` clauses using region specialization. It creates two execution paths:
// device execution when the condition is true, host execution when false.
//
// Overview:
// ---------
// When an ACC compute construct has an `if` clause, the construct should only
// execute on the device when the condition is true. If the condition is false,
// the code should execute on the host instead. This pass transforms:
//
//   acc.parallel if(%cond) { ... }
//
// Into:
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 22-35
```cpp
//
//   scf.if %cond {
//     // Device path: clone data ops, compute construct without if, exit ops
//     acc.parallel { ... }
//   } else {
//     // Host path: original region body with ACC ops converted to host
//   }
//
// Transformations:
// ----------------
// For each compute construct with an `if` clause:
//
// 1. Device Path (true branch):
//    - Clone data entry operations (acc.copyin, acc.create, etc.)
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 36-49
```cpp
//    - Clone the compute construct without the `if` clause
//    - Clone data exit operations (acc.copyout, acc.delete, etc.)
//
// 2. Host Path (false branch):
//    - Move the original region body to the else branch
//    - Apply host fallback patterns to convert ACC ops to host equivalents
//
// 3. Cleanup:
//    - Erase the original compute construct and data operations
//    - Replace uses of ACC variables with host variables in the else branch
//
// Requirements:
// -------------
// To use this pass in a pipeline, the following requirements exist:
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 50-56
```cpp
//
// 1. Analysis Registration (Optional): If custom behavior is needed for
//    emitting not-yet-implemented messages for unsupported cases, the pipeline
//    should pre-register the `acc::OpenACCSupport` analysis.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 57-70
```cpp
#include "mlir/Dialect/OpenACC/Transforms/Passes.h"

#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h"
#include "mlir/Dialect/OpenACC/OpenACC.h"
#include "mlir/Dialect/OpenACC/OpenACCUtilsLoop.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`。

### Lines 71-77
```cpp
namespace mlir {
namespace acc {
#define GEN_PASS_DEF_ACCIFCLAUSELOWERING
#include "mlir/Dialect/OpenACC/Transforms/Passes.h.inc"
} // namespace acc
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`。

### Lines 78-84
```cpp
#define DEBUG_TYPE "acc-if-clause-lowering"

using namespace mlir;
using namespace mlir::acc;

namespace {

```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 85-91
```cpp
class ACCIfClauseLowering
    : public acc::impl::ACCIfClauseLoweringBase<ACCIfClauseLowering> {
  using ACCIfClauseLoweringBase<ACCIfClauseLowering>::ACCIfClauseLoweringBase;

private:
  OpenACCSupport *accSupport = nullptr;

```
- **EN**: Introduces declarations for `ACCIfClauseLowering`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ACCIfClauseLowering` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 92-98
```cpp
  void convertHostRegion(Operation *computeOp, Region &region);

  template <typename OpTy>
  void lowerIfClauseForComputeConstruct(OpTy computeConstructOp,
                                        SmallVector<Operation *> &eraseOps);

public:
```
- **EN**: Implements logic around `convertHostRegion`, `lowerIfClauseForComputeConstruct`.
- **CN**: 围绕 `convertHostRegion`, `lowerIfClauseForComputeConstruct` 实现具体逻辑。

### Lines 99-110
```cpp
  void runOnOperation() override;
};

void ACCIfClauseLowering::convertHostRegion(Operation *computeOp,
                                            Region &region) {
  // Only collect ACC dialect operations - other ops don't need conversion
  SmallVector<Operation *> hostOps;
  region.walk<WalkOrder::PreOrder>([&](Operation *op) {
    if (isa<acc::OpenACCDialect>(op->getDialect()))
      hostOps.push_back(op);
  });

```
- **EN**: Implements logic around `runOnOperation`, `convertHostRegion`, `PreOrder>`, `OpenACCDialect>`, and 1 more symbols; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `runOnOperation`, `convertHostRegion`, `PreOrder>`, `OpenACCDialect>`, and 1 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 111-120
```cpp
  RewritePatternSet patterns(computeOp->getContext());
  populateACCHostFallbackPatterns(patterns, *accSupport);

  GreedyRewriteConfig config;
  config.setUseTopDownTraversal(true);
  config.setStrictness(GreedyRewriteStrictness::ExistingOps);
  if (failed(applyOpPatternsGreedily(hostOps, std::move(patterns), config)))
    accSupport->emitNYI(computeOp->getLoc(), "failed to convert host region");
}

```
- **EN**: Implements logic around `patterns`, `populateACCHostFallbackPatterns`, `setUseTopDownTraversal`, `setStrictness`, and 2 more symbols; this block coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `patterns`, `populateACCHostFallbackPatterns`, `setUseTopDownTraversal`, `setStrictness`, and 2 more symbols 实现具体逻辑；该代码块协调方言转换或 lowering 决策。

### Lines 121-129
```cpp
// Template function to handle if condition conversion for ACC compute
// constructs
template <typename OpTy>
void ACCIfClauseLowering::lowerIfClauseForComputeConstruct(
    OpTy computeConstructOp, SmallVector<Operation *> &eraseOps) {
  Value ifCond = computeConstructOp.getIfCond();
  if (!ifCond)
    return;

```
- **EN**: Implements logic around `lowerIfClauseForComputeConstruct`, `getIfCond`.
- **CN**: 围绕 `lowerIfClauseForComputeConstruct`, `getIfCond` 实现具体逻辑。

### Lines 130-143
```cpp
  IRRewriter rewriter(computeConstructOp);

  LLVM_DEBUG(llvm::dbgs() << "Converting " << computeConstructOp->getName()
                          << " with if condition: " << computeConstructOp
                          << "\n");

  // Collect data clause operations that need to be recreated in the if
  // condition
  SmallVector<Operation *> dataEntryOps;
  SmallVector<Operation *> dataExitOps;
  SmallVector<Operation *> firstprivateOps;
  SmallVector<Operation *> privateOps;
  SmallVector<Operation *> reductionOps;

```
- **EN**: Implements logic around `rewriter`, `dbgs`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `rewriter`, `dbgs` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 144-156
```cpp
  // Collect data entry operations
  for (Value operand : computeConstructOp.getDataClauseOperands())
    if (Operation *defOp = operand.getDefiningOp())
      if (isa<ACC_DATA_ENTRY_OPS>(defOp))
        dataEntryOps.push_back(defOp);

  // Find corresponding exit operations for each entry operation.
  // Iterate backwards through entry ops since exit ops appear in reverse order.
  for (Operation *dataEntryOp : llvm::reverse(dataEntryOps))
    for (Operation *user : dataEntryOp->getUsers())
      if (isa<ACC_DATA_EXIT_OPS>(user))
        dataExitOps.push_back(user);

```
- **EN**: Implements logic around `getDataClauseOperands`, `getDefiningOp`, `push_back`, `reverse`, and 1 more symbols.
- **CN**: 围绕 `getDataClauseOperands`, `getDefiningOp`, `push_back`, `reverse`, and 1 more symbols 实现具体逻辑。

### Lines 157-166
```cpp
  // Collect firstprivate, private, and reduction operations
  auto collectOps = [&](SmallVector<Operation *> &ops, OperandRange operands) {
    for (Value operand : operands)
      if (Operation *defOp = operand.getDefiningOp())
        ops.push_back(defOp);
  };
  collectOps(firstprivateOps, computeConstructOp.getFirstprivateOperands());
  collectOps(privateOps, computeConstructOp.getPrivateOperands());
  collectOps(reductionOps, computeConstructOp.getReductionOperands());

```
- **EN**: Implements logic around `getDefiningOp`, `push_back`, `collectOps`.
- **CN**: 围绕 `getDefiningOp`, `push_back`, `collectOps` 实现具体逻辑。

### Lines 167-173
```cpp
  // Create scf.if with device and host execution paths
  auto ifOp = scf::IfOp::create(rewriter, computeConstructOp.getLoc(),
                                TypeRange{}, ifCond, /*withElseRegion=*/true);

  LLVM_DEBUG(llvm::dbgs() << "Cloning " << dataEntryOps.size()
                          << " data entry operations for device path\n");

```
- **EN**: Implements logic around `create`, `dbgs`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `dbgs` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 174-183
```cpp
  // Device execution path (true branch)
  Block &thenBlock = ifOp.getThenRegion().front();
  rewriter.setInsertionPointToStart(&thenBlock);

  // Clone data entry operations
  SmallVector<Value> deviceDataOperands;
  SmallVector<Value> firstprivateOperands;
  SmallVector<Value> privateOperands;
  SmallVector<Value> reductionOperands;

```
- **EN**: Implements logic around `getThenRegion`, `setInsertionPointToStart`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getThenRegion`, `setInsertionPointToStart` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 184-197
```cpp
  // Map the data entry and firstprivate ops for the cloned region
  IRMapping deviceMapping;
  auto cloneAndMapOps = [&](SmallVector<Operation *> &ops,
                            SmallVector<Value> &operands) {
    for (Operation *op : ops) {
      Operation *clonedOp = rewriter.clone(*op, deviceMapping);
      operands.push_back(clonedOp->getResult(0));
      deviceMapping.map(op->getResult(0), clonedOp->getResult(0));
    }
  };
  cloneAndMapOps(dataEntryOps, deviceDataOperands);
  cloneAndMapOps(firstprivateOps, firstprivateOperands);
  cloneAndMapOps(privateOps, privateOperands);
  cloneAndMapOps(reductionOps, reductionOperands);
```
- **EN**: Implements logic around `clone`, `push_back`, `map`, `cloneAndMapOps`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `clone`, `push_back`, `map`, `cloneAndMapOps` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 198-208
```cpp

  // Create new compute op without if condition for device execution by
  // cloning
  OpTy newComputeOp = cast<OpTy>(
      rewriter.clone(*computeConstructOp.getOperation(), deviceMapping));
  newComputeOp.getIfCondMutable().clear();
  newComputeOp.getDataClauseOperandsMutable().assign(deviceDataOperands);
  newComputeOp.getFirstprivateOperandsMutable().assign(firstprivateOperands);
  newComputeOp.getPrivateOperandsMutable().assign(privateOperands);
  newComputeOp.getReductionOperandsMutable().assign(reductionOperands);

```
- **EN**: Implements logic around `clone`, `getIfCondMutable`, `getDataClauseOperandsMutable`, `getFirstprivateOperandsMutable`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `clone`, `getIfCondMutable`, `getDataClauseOperandsMutable`, `getFirstprivateOperandsMutable`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 209-217
```cpp
  // Clone data exit operations
  rewriter.setInsertionPointAfter(newComputeOp);
  for (Operation *dataOp : dataExitOps)
    rewriter.clone(*dataOp, deviceMapping);

  rewriter.setInsertionPointToEnd(&thenBlock);
  if (!thenBlock.getTerminator())
    scf::YieldOp::create(rewriter, computeConstructOp.getLoc());

```
- **EN**: Implements logic around `setInsertionPointAfter`, `clone`, `setInsertionPointToEnd`, `getTerminator`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `setInsertionPointAfter`, `clone`, `setInsertionPointToEnd`, `getTerminator`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 218-229
```cpp
  // Host execution path (false branch)
  Region &hostRegion = computeConstructOp.getRegion();
  if (hostRegion.hasOneBlock()) {
    // Don't need to clone original ops, just take them and legalize for host.
    ifOp.getElseRegion().takeBody(hostRegion);

    // Swap acc yield for scf yield.
    Block &elseBlock = ifOp.getElseRegion().front();
    elseBlock.getTerminator()->erase();
    rewriter.setInsertionPointToEnd(&elseBlock);
    scf::YieldOp::create(rewriter, computeConstructOp.getLoc());

```
- **EN**: Implements logic around `getRegion`, `hasOneBlock`, `getElseRegion`, `getTerminator`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getRegion`, `hasOneBlock`, `getElseRegion`, `getTerminator`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 230-241
```cpp
    convertHostRegion(computeConstructOp, ifOp.getElseRegion());
  } else {
    // scf.if regions must stay single-block. Wrap the original multi-block ACC
    // body in scf.execute_region so it can be hosted in the else branch.
    Block &elseBlock = ifOp.getElseRegion().front();
    rewriter.setInsertionPoint(elseBlock.getTerminator());
    IRMapping hostMapping;
    auto hostExecuteRegion = wrapMultiBlockRegionWithSCFExecuteRegion(
        hostRegion, hostMapping, computeConstructOp.getLoc(), rewriter);
    convertHostRegion(computeConstructOp, hostExecuteRegion.getRegion());
  }

```
- **EN**: Implements logic around `convertHostRegion`, `getElseRegion`, `setInsertionPoint`, `wrapMultiBlockRegionWithSCFExecuteRegion`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `convertHostRegion`, `getElseRegion`, `setInsertionPoint`, `wrapMultiBlockRegionWithSCFExecuteRegion`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 242-249
```cpp
  // The original op is now empty and can be erased
  eraseOps.push_back(computeConstructOp);

  // TODO: Can probably 'move' the data ops instead of cloning them
  // which would eliminate need to explicitly erase
  for (Operation *dataOp : dataExitOps)
    eraseOps.push_back(dataOp);

```
- **EN**: Implements logic around `push_back`.
- **CN**: 围绕 `push_back` 实现具体逻辑。

### Lines 250-263
```cpp
  // The new host code may contain uses of the acc variables. Replace them by
  // the host values.
  auto replaceAndEraseOps = [&](SmallVector<Operation *> &ops) {
    for (Operation *op : ops) {
      getAccVar(op).replaceAllUsesWith(getVar(op));
      eraseOps.push_back(op);
    }
  };
  replaceAndEraseOps(dataEntryOps);
  replaceAndEraseOps(firstprivateOps);
  replaceAndEraseOps(privateOps);
  replaceAndEraseOps(reductionOps);
}

```
- **EN**: Implements logic around `getAccVar`, `push_back`, `replaceAndEraseOps`.
- **CN**: 围绕 `getAccVar`, `push_back`, `replaceAndEraseOps` 实现具体逻辑。

### Lines 264-277
```cpp
void ACCIfClauseLowering::runOnOperation() {
  func::FuncOp funcOp = getOperation();
  accSupport = &getAnalysis<OpenACCSupport>();

  SmallVector<Operation *> eraseOps;
  funcOp.walk([&](Operation *op) {
    if (auto parallelOp = dyn_cast<acc::ParallelOp>(op))
      lowerIfClauseForComputeConstruct(parallelOp, eraseOps);
    else if (auto kernelsOp = dyn_cast<acc::KernelsOp>(op))
      lowerIfClauseForComputeConstruct(kernelsOp, eraseOps);
    else if (auto serialOp = dyn_cast<acc::SerialOp>(op))
      lowerIfClauseForComputeConstruct(serialOp, eraseOps);
  });

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `getAnalysis`, `walk`, and 4 more symbols; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `getAnalysis`, `walk`, and 4 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 278-282
```cpp
  for (Operation *op : eraseOps)
    op->erase();
}

} // namespace
```
- **EN**: Implements logic around `erase`.
- **CN**: 围绕 `erase` 实现具体逻辑。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/OpenACC/OpenACCUtilsLoop.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/IR/Builders.h`, `mlir/IR/IRMapping.h`, `mlir/IR/PatternMatch.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h` ... (+3 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (7), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1), LLVM support-library facilities / LLVM Support 库设施 (1)
