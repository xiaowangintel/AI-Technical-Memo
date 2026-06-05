# SeedCollection.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/Passes/SeedCollection.cpp` | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/Passes/SeedCollection.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements seed collection pass within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 SeedCollection 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

```cpp
//===- SeedCollection.cpp - Seed collection pass --------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Vectorize/SandboxVectorizer/Passes/SeedCollection.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/SandboxIR/Module.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/RegionWithScore.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/SandboxVectorizerPassBuilder.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/SeedCollector.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h"

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 17-33

```cpp
namespace llvm {

static cl::opt<unsigned>
    OverrideVecRegBits("sbvec-vec-reg-bits", cl::init(0), cl::Hidden,
                       cl::desc("Override the vector register size in bits, "
                                "which is otherwise found by querying TTI."));
static cl::opt<bool>
    AllowNonPow2("sbvec-allow-non-pow2", cl::init(false), cl::Hidden,
                 cl::desc("Allow non-power-of-2 vectorization."));

#define LoadSeedsDef "loads"
#define StoreSeedsDef "stores"
cl::opt<std::string> CollectSeeds(
    "sbvec-collect-seeds", cl::init(StoreSeedsDef), cl::Hidden,
    cl::desc("Collect these seeds. Use empty for none or a comma-separated "
             "list of '" StoreSeedsDef "' and '" LoadSeedsDef "'."));

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 34-50

```cpp
namespace sandboxir {

SeedCollection::SeedCollection(StringRef Pipeline, StringRef AuxArg)
    : FunctionPass("seed-collection"),
      RPM("rpm", Pipeline, SandboxVectorizerPassBuilder::createRegionPass) {
  if (!AuxArg.empty()) {
    if (AuxArg != DiffTypesArgStr) {
      std::string ErrStr;
      raw_string_ostream ErrSS(ErrStr);
      ErrSS << "SeedCollection only supports '" << DiffTypesArgStr
            << "' aux argument!\n";
      reportFatalUsageError(ErrStr.c_str());
    }
    AllowDiffTypes = true;
  }
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include RPM, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 RPM，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 51-65

```cpp
bool SeedCollection::runOnFunction(Function &F, const Analyses &A) {
  bool Change = false;
  const auto &DL = F.getParent()->getDataLayout();
  bool CollectStores = CollectSeeds.find(StoreSeedsDef) != std::string::npos;
  bool CollectLoads = CollectSeeds.find(LoadSeedsDef) != std::string::npos;

  // TODO: Start from innermost BBs first
  for (auto &BB : F) {
    SeedCollector SC(&BB, A.getScalarEvolution(), CollectStores, CollectLoads,
                     AllowDiffTypes);
    for (SeedBundle &Seeds : SC.getStoreSeeds()) {
      unsigned ElmBits =
          Utils::getNumBits(VecUtils::getElementType(Utils::getExpectedType(
                                Seeds[Seeds.getFirstUnusedElementIdx()])),
                            DL);
```
- EN: Core entities appearing here include runOnFunction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, ScalarEvolution.
- CN: 此处出现的核心实体包括 runOnFunction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, ScalarEvolution 等分析结果。

### Lines 66-80

```cpp
      unsigned AS = getLoadStoreAddressSpace(Seeds[0]);
      unsigned VecRegBits = OverrideVecRegBits != 0
                                ? OverrideVecRegBits
                                : A.getTTI().getLoadStoreVecRegBitWidth(AS);

      auto DivideBy2 = [](unsigned Num) {
        auto Floor = VecUtils::getFloorPowerOf2(Num);
        if (Floor == Num)
          return Floor / 2;
        return Floor;
      };
      // Try to create the largest vector supported by the target. If it fails
      // reduce the vector size by half.
      for (unsigned SliceElms = std::min(VecRegBits / ElmBits,
                                         Seeds.getNumUnusedBits() / ElmBits);
```
- EN: This region continues the SeedCollection implementation with local helper logic centered on Seeds, VecRegBits, OverrideVecRegBits, DivideBy2.
- CN: 这一段延续了 SeedCollection 的主体实现，围绕 Seeds, VecRegBits, OverrideVecRegBits, DivideBy2 等局部辅助逻辑展开。

### Lines 81-99

```cpp
           SliceElms >= 2u; SliceElms = DivideBy2(SliceElms)) {
        if (Seeds.allUsed())
          break;
        // Keep trying offsets after FirstUnusedElementIdx, until we vectorize
        // the slice. This could be quite expensive, so we enforce a limit.
        for (unsigned Offset = Seeds.getFirstUnusedElementIdx(),
                      OE = Seeds.size();
             Offset + 1 < OE; Offset += 1) {
          // Seeds are getting used as we vectorize, so skip them.
          if (Seeds.isUsed(Offset))
            continue;
          if (Seeds.allUsed())
            break;

          auto SeedSlice =
              Seeds.getSlice(Offset, SliceElms * ElmBits, !AllowNonPow2);
          if (SeedSlice.empty())
            continue;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 100-114

```cpp
          assert(SeedSlice.size() >= 2 && "Should have been rejected!");

          // Create a region containing the seed slice.
          auto &Ctx = F.getContext();
          RegionWithScore Rgn(Ctx, A.getTTI());
          Rgn.setAux(SeedSlice);
          // Run the region pass pipeline.
          Change |= RPM.runOnRegion(Rgn, A);
          Rgn.clearAux();
        }
      }
    }
  }
  return Change;
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 115-116

```cpp
} // namespace sandboxir
} // namespace llvm
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `RPM, runOnFunction` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`RPM, runOnFunction` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, ScalarEvolution, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, ScalarEvolution, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `OverrideVecRegBits, AllowNonPow2` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `OverrideVecRegBits, AllowNonPow2` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/Transforms/Vectorize/SandboxVectorizer/Passes/SeedCollection.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/RegionWithScore.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/SandboxVectorizerPassBuilder.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/SeedCollector.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/Transforms/Vectorize/SandboxVectorizer/Passes/SeedCollection.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/RegionWithScore.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/SandboxVectorizerPassBuilder.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/SeedCollector.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/SandboxIR/Module.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/SandboxIR/Module.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DataLayout`, `ScalarEvolution`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `ScalarEvolution`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
