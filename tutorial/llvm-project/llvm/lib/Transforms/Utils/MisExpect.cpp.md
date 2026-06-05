# MisExpect.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/MisExpect.cpp` | `llvm/lib/Transforms/Utils/MisExpect.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements check the use of llvm.expect with PGO data within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 MisExpect 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===--- MisExpect.cpp - Check the use of llvm.expect with PGO data -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This contains code to emit warnings for potentially incorrect usage of the
// llvm.expect intrinsic. This utility extracts the threshold values from
// metadata associated with the instrumented Branch or Switch instruction. The
// threshold values are then used to determine if a warning should be emmited.
//
// MisExpect's implementation relies on two assumptions about how branch weights
// are managed in LLVM.
//
// 1) Frontend profiling weights are always in place before llvm.expect is
// lowered in LowerExpectIntrinsic.cpp. Frontend based instrumentation therefore
// needs to extract the branch weights and then compare them to the weights
// being added by the llvm.expect intrinsic lowering.
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 21-40

```cpp
//
// 2) Sampling and IR based profiles will *only* have branch weight metadata
// before profiling data is consulted if they are from a lowered llvm.expect
// intrinsic. These profiles thus always extract the expected weights and then
// compare them to the weights collected during profiling to determine if a
// diagnostic message is warranted.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/MisExpect.h"
#include "llvm/ADT/Twine.h"
#include "llvm/Analysis/OptimizationRemarkEmitter.h"
#include "llvm/IR/DiagnosticInfo.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/ProfDataUtils.h"
#include "llvm/Support/BranchProbability.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/FormatVariadic.h"
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 41-63

```cpp
#include <algorithm>
#include <cstdint>
#include <functional>
#include <numeric>

#define DEBUG_TYPE "misexpect"

using namespace llvm;
using namespace misexpect;

// Command line option to enable/disable the warning when profile data suggests
// a mismatch with the use of the llvm.expect intrinsic
static cl::opt<bool> PGOWarnMisExpect(
    "pgo-warn-misexpect", cl::init(false), cl::Hidden,
    cl::desc("Use this option to turn on/off "
             "warnings about incorrect usage of llvm.expect intrinsics."));

// Command line option for setting the diagnostic tolerance threshold
static cl::opt<uint32_t> MisExpectTolerance(
    "misexpect-tolerance", cl::init(0),
    cl::desc("Prevents emitting diagnostics when profile counts are "
             "within N% of the threshold.."));

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 64-83

```cpp
static bool isMisExpectDiagEnabled(const LLVMContext &Ctx) {
  return PGOWarnMisExpect || Ctx.getMisExpectWarningRequested();
}

static uint32_t getMisExpectTolerance(const LLVMContext &Ctx) {
  return std::max(static_cast<uint32_t>(MisExpectTolerance),
                  Ctx.getDiagnosticsMisExpectTolerance());
}

static const Instruction *getInstCondition(const Instruction *I) {
  assert(I != nullptr && "MisExpect target Instruction cannot be nullptr");
  const Instruction *Ret = nullptr;
  if (auto *B = dyn_cast<CondBrInst>(I)) {
    Ret = dyn_cast<Instruction>(B->getCondition());
  }
  // TODO: Find a way to resolve condition location for switches
  // Using the condition of the switch seems to often resolve to an earlier
  // point in the program, i.e. the calculation of the switch condition, rather
  // than the switch's location in the source code. Thus, we should use the
  // instruction to get source code locations rather than the condition to
```
- EN: Core entities appearing here include isMisExpectDiagEnabled, getMisExpectTolerance, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 isMisExpectDiagEnabled, getMisExpectTolerance，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 84-103

```cpp
  // improve diagnostic output, such as the caret. If the same problem exists
  // for branch instructions, then we should remove this function and directly
  // use the instruction
  //
  else if (auto *S = dyn_cast<SwitchInst>(I)) {
    Ret = dyn_cast<Instruction>(S->getCondition());
  }
  return Ret ? Ret : I;
}

static void emitMisexpectDiagnostic(const Instruction *I, LLVMContext &Ctx,
                                    uint64_t ProfCount, uint64_t TotalCount) {
  double PercentageCorrect = (double)ProfCount / TotalCount;
  auto PerString =
      formatv("{0:P} ({1} / {2})", PercentageCorrect, ProfCount, TotalCount);
  auto RemStr = formatv(
      "Potential performance regression from use of the llvm.expect intrinsic: "
      "Annotation was correct on {0} of profiled executions.",
      PerString);
  const Instruction *Cond = getInstCondition(I);
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 104-123

```cpp
  if (isMisExpectDiagEnabled(Ctx))
    Ctx.diagnose(DiagnosticInfoMisExpect(Cond, Twine(PerString)));
  OptimizationRemarkEmitter ORE(I->getParent()->getParent());
  ORE.emit(OptimizationRemark(DEBUG_TYPE, "misexpect", Cond) << RemStr.str());
}

void misexpect::verifyMisExpect(const Instruction &I,
                                ArrayRef<uint32_t> RealWeights,
                                ArrayRef<uint32_t> ExpectedWeights) {
  // To determine if we emit a diagnostic, we need to compare the branch weights
  // from the profile to those added by the llvm.expect intrinsic.
  // So first, we extract the "likely" and "unlikely" weights from
  // ExpectedWeights And determine the correct weight in the profile to compare
  // against.
  uint64_t LikelyBranchWeight = 0,
           UnlikelyBranchWeight = std::numeric_limits<uint32_t>::max();
  size_t MaxIndex = 0;
  for (const auto &[Idx, V] : enumerate(ExpectedWeights)) {
    if (LikelyBranchWeight < V) {
      LikelyBranchWeight = V;
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 124-142

```cpp
      MaxIndex = Idx;
    }
    if (UnlikelyBranchWeight > V)
      UnlikelyBranchWeight = V;
  }

  const uint64_t ProfiledWeight = RealWeights[MaxIndex];
  const uint64_t RealWeightsTotal =
      std::accumulate(RealWeights.begin(), RealWeights.end(), (uint64_t)0,
                      std::plus<uint64_t>());
  const uint64_t NumUnlikelyTargets = RealWeights.size() - 1;

  uint64_t TotalBranchWeight =
      LikelyBranchWeight + (UnlikelyBranchWeight * NumUnlikelyTargets);

  // Failing this assert means that we have corrupted metadata.
  assert((TotalBranchWeight >= LikelyBranchWeight) && (TotalBranchWeight > 0) &&
         "TotalBranchWeight is less than the Likely branch weight");

```
- EN: This region continues the MisExpect implementation with local helper logic centered on MaxIndex, Idx, UnlikelyBranchWeight, ProfiledWeight.
- CN: 这一段延续了 MisExpect 的主体实现，围绕 MaxIndex, Idx, UnlikelyBranchWeight, ProfiledWeight 等局部辅助逻辑展开。

### Lines 143-165

```cpp
  // To determine our threshold value we need to obtain the branch probability
  // for the weights added by llvm.expect and use that proportion to calculate
  // our threshold based on the collected profile data.
  auto LikelyProbablilty = BranchProbability::getBranchProbability(
      LikelyBranchWeight, TotalBranchWeight);

  uint64_t ScaledThreshold = LikelyProbablilty.scale(RealWeightsTotal);

  // clamp tolerance range to [0, 100)
  uint32_t Tolerance = getMisExpectTolerance(I.getContext());
  Tolerance = std::clamp(Tolerance, 0u, 99u);

  // Allow users to relax checking by N%  i.e., if they use a 5% tolerance,
  // then we check against 0.95*ScaledThreshold
  if (Tolerance > 0)
    ScaledThreshold *= (1.0 - Tolerance / 100.0);

  // When the profile weight is below the threshold, we emit the diagnostic
  if (ProfiledWeight < ScaledThreshold)
    emitMisexpectDiagnostic(&I, I.getContext(), ProfiledWeight,
                            RealWeightsTotal);
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 166-188

```cpp
void misexpect::checkBackendInstrumentation(const Instruction &I,
                                            ArrayRef<uint32_t> RealWeights) {
  // Backend checking assumes any existing weight comes from an `llvm.expect`
  // intrinsic. However, SampleProfiling + ThinLTO add branch weights  multiple
  // times, leading to an invalid assumption in our checking. Backend checks
  // should only operate on branch weights that carry the "!expected" field,
  // since they are guaranteed to be added by the LowerExpectIntrinsic pass.
  if (!hasBranchWeightOrigin(I))
    return;
  SmallVector<uint32_t> ExpectedWeights;
  if (!extractBranchWeights(I, ExpectedWeights))
    return;
  verifyMisExpect(I, RealWeights, ExpectedWeights);
}

void misexpect::checkFrontendInstrumentation(
    const Instruction &I, ArrayRef<uint32_t> ExpectedWeights) {
  SmallVector<uint32_t> RealWeights;
  if (!extractBranchWeights(I, RealWeights))
    return;
  verifyMisExpect(I, RealWeights, ExpectedWeights);
}

```
- EN: This region continues the MisExpect implementation with local helper logic centered on Instruction, ArrayRef, RealWeights, Backend.
- CN: 这一段延续了 MisExpect 的主体实现，围绕 Instruction, ArrayRef, RealWeights, Backend 等局部辅助逻辑展开。

### Lines 189-196

```cpp
void misexpect::checkExpectAnnotations(const Instruction &I,
                                       ArrayRef<uint32_t> ExistingWeights,
                                       bool IsFrontend) {
  if (IsFrontend)
    checkFrontendInstrumentation(I, ExistingWeights);
  else
    checkBackendInstrumentation(I, ExistingWeights);
}
```
- EN: This region continues the MisExpect implementation with local helper logic centered on Instruction, ArrayRef, ExistingWeights, IsFrontend.
- CN: 这一段延续了 MisExpect 的主体实现，围绕 Instruction, ArrayRef, ExistingWeights, IsFrontend 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `isMisExpectDiagEnabled, getMisExpectTolerance` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`isMisExpectDiagEnabled, getMisExpectTolerance` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `OptimizationRemarkEmitter` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `OptimizationRemarkEmitter` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `PGOWarnMisExpect, MisExpectTolerance` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `PGOWarnMisExpect, MisExpectTolerance` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/OptimizationRemarkEmitter.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/OptimizationRemarkEmitter.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/DiagnosticInfo.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/MisExpect.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DiagnosticInfo.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/ProfDataUtils.h`, `llvm/Transforms/Utils/MisExpect.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/BranchProbability.h`, `llvm/Support/CommandLine.h`, `llvm/Support/FormatVariadic.h`, `llvm/ADT/Twine.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/BranchProbability.h`, `llvm/Support/CommandLine.h`, `llvm/Support/FormatVariadic.h`, `llvm/ADT/Twine.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cstdint`, `functional`, `numeric` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cstdint`, `functional`, `numeric` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `OptimizationRemarkEmitter` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`OptimizationRemarkEmitter` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
