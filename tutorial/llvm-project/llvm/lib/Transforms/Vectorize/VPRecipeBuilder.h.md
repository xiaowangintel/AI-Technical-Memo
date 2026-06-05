# VPRecipeBuilder.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPRecipeBuilder.h` | `llvm/lib/Transforms/Vectorize/VPRecipeBuilder.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This header defines the public interface for helper class to build recipes within LLVM's vectorization and VPlan infrastructure layer. | 该头文件定义了对外接口 LLVM 向量化与 VPlan 基础设施层中的 VPRecipeBuilder 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-17

```cpp
//===- VPRecipeBuilder.h - Helper class to build recipes --------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_TRANSFORMS_VECTORIZE_VPRECIPEBUILDER_H
#define LLVM_TRANSFORMS_VECTORIZE_VPRECIPEBUILDER_H

#include "LoopVectorizationPlanner.h"
#include "VPlan.h"
#include "llvm/Analysis/ScalarEvolutionExpressions.h"

namespace llvm {

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. The header guard ensures the declarations are safe to include from multiple compilation units. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 头文件保护宏保证这些声明可以被多个编译单元安全地重复包含。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

### Lines 18-33

```cpp
class LoopVectorizationLegality;
class LoopVectorizationCostModel;
struct HistogramInfo;
struct VFRange;

/// Helper class to create VPRecipies from IR instructions.
class VPRecipeBuilder {
  /// The VPlan new recipes are added to.
  VPlan &Plan;

  /// The legality analysis.
  LoopVectorizationLegality *Legal;

  /// The profitablity analysis.
  LoopVectorizationCostModel &CM;

```
- EN: Core entities appearing here include LoopVectorizationLegality, LoopVectorizationCostModel, HistogramInfo, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 LoopVectorizationLegality, LoopVectorizationCostModel, HistogramInfo，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 34-50

```cpp
  VPBuilder &Builder;

  /// Check if \p I can be widened at the start of \p Range and possibly
  /// decrease the range such that the returned value holds for the entire \p
  /// Range. The function should not be called for memory instructions or calls.
  bool shouldWiden(Instruction *I, VFRange &Range) const;

  /// Optimize the special case where the operand of \p VPI is a constant
  /// integer induction variable.
  VPWidenIntOrFpInductionRecipe *
  tryToOptimizeInductionTruncate(VPInstruction *VPI, VFRange &Range);

  /// Check if \p VPI has an opcode that can be widened and return a
  /// widened recipe if it can. The function should only be called if the
  /// cost-model indicates that widening should be performed.
  VPRecipeWithIRFlags *tryToWiden(VPInstruction *VPI);

```
- EN: The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 51-65

```cpp
public:
  VPRecipeBuilder(VPlan &Plan, LoopVectorizationLegality *Legal,
                  LoopVectorizationCostModel &CM, VPBuilder &Builder)
      : Plan(Plan), Legal(Legal), CM(CM), Builder(Builder) {}

  /// Create and return a widened recipe for a non-phi recipe \p R if one can be
  /// created within the given VF \p Range.
  VPRecipeBase *tryToCreateWidenNonPhiRecipe(VPSingleDefRecipe *R,
                                             VFRange &Range);

  /// Check if the load or store instruction \p VPI should widened for \p
  /// Range.Start and potentially masked. Such instructions are handled by a
  /// recipe that takes an additional VPInstruction for the mask.
  VPRecipeBase *tryToWidenMemory(VPInstruction *VPI, VFRange &Range);

```
- EN: The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 66-80

```cpp
  /// If \p VPI represents a histogram operation (as determined by
  /// LoopVectorizationLegality) make that safe for vectorization, by emitting a
  /// llvm.experimental.vector.histogram.add intrinsic in place of the Load +
  /// Add|Sub + Store operations that perform the histogram in the original
  /// scalar loop.
  VPHistogramRecipe *widenIfHistogram(VPInstruction *VPI);

  /// If \p VPI is a store of a reduction into an invariant address, delete it.
  /// If it is the final store of a reduction result, a uniform store recipe
  /// will be created for it in the middle block. Returns `true` if replacement
  /// took place. The order of stores must be preserved, hence \p
  /// FinalRedStoresBuidler.
  bool replaceWithFinalIfReductionStore(VPInstruction *VPI,
                                        VPBuilder &FinalRedStoresBuilder);

```
- EN: The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 81-88

```cpp
  /// Build a VPReplicationRecipe for \p VPI. If it is predicated, add the mask
  /// as last operand. Range.End may be decreased to ensure same recipe behavior
  /// from \p Range.Start to \p Range.End.
  VPReplicateRecipe *handleReplication(VPInstruction *VPI, VFRange &Range);
};
} // end namespace llvm

#endif // LLVM_TRANSFORMS_VECTORIZE_VPRECIPEBUILDER_H
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `LoopVectorizationLegality, LoopVectorizationCostModel, HistogramInfo, VFRange, VPRecipeBuilder` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`LoopVectorizationLegality, LoopVectorizationCostModel, HistogramInfo, VFRange, VPRecipeBuilder` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `ScalarEvolution` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `ScalarEvolution` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Header role: it mostly exposes declarations, contracts, inline helpers, and reusable utilities rather than driving a standalone transform by itself.
  - CN: 头文件角色：它主要暴露声明、约定、内联辅助函数和可复用工具，而不是独立执行某个完整变换。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/ScalarEvolutionExpressions.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/ScalarEvolutionExpressions.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: Standard/other headers: `LoopVectorizationPlanner.h`, `VPlan.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`LoopVectorizationPlanner.h`, `VPlan.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `ScalarEvolution` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`ScalarEvolution` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
