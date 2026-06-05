# SampleProfileLoaderBaseUtil.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/SampleProfileLoaderBaseUtil.cpp` | `llvm/lib/Transforms/Utils/SampleProfileLoaderBaseUtil.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements profile loader Util func within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 SampleProfileLoaderBaseUtil 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===- SampleProfileLoaderBaseUtil.cpp - Profile loader Util func ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the SampleProfileLoader base utility functions.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/SampleProfileLoaderBaseUtil.h"
#include "llvm/Analysis/ProfileSummaryInfo.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Module.h"
#include "llvm/Transforms/Utils/ModuleUtils.h"

namespace llvm {

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Legality, profitability, or canonical-form checks in this region depend on analyses such as ProfileSummaryInfo.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段中的合法性、收益性或规范形态检查依赖 ProfileSummaryInfo 等分析结果。

### Lines 21-40

```cpp
cl::opt<unsigned> SampleProfileMaxPropagateIterations(
    "sample-profile-max-propagate-iterations", cl::init(100),
    cl::desc("Maximum number of iterations to go through when propagating "
             "sample block/edge weights through the CFG."));

cl::opt<unsigned> SampleProfileRecordCoverage(
    "sample-profile-check-record-coverage", cl::init(0), cl::value_desc("N"),
    cl::desc("Emit a warning if less than N% of records in the input profile "
             "are matched to the IR."));

cl::opt<unsigned> SampleProfileSampleCoverage(
    "sample-profile-check-sample-coverage", cl::init(0), cl::value_desc("N"),
    cl::desc("Emit a warning if less than N% of samples in the input profile "
             "are matched to the IR."));

cl::opt<bool> NoWarnSampleUnused(
    "no-warn-sample-unused", cl::init(false), cl::Hidden,
    cl::desc("Use this option to turn off/on warnings about function with "
             "samples but without debug information to use those samples. "));

```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 41-59

```cpp
cl::opt<bool> SampleProfileUseProfi(
    "sample-profile-use-profi", cl::Hidden,
    cl::desc("Use profi to infer block and edge counts."));

namespace sampleprofutil {

/// Return true if the given callsite is hot wrt to hot cutoff threshold.
///
/// Functions that were inlined in the original binary will be represented
/// in the inline stack in the sample profile. If the profile shows that
/// the original inline decision was "good" (i.e., the callsite is executed
/// frequently), then we will recreate the inline decision and apply the
/// profile from the inlined callsite.
///
/// To decide whether an inlined callsite is hot, we compare the callsite
/// sample count with the hot cutoff computed by ProfileSummaryInfo, it is
/// regarded as hot if the count is above the cutoff value.
///
/// When ProfileAccurateForSymsInList is enabled and profile symbol list
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Legality, profitability, or canonical-form checks in this region depend on analyses such as ProfileSummaryInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 这一段中的合法性、收益性或规范形态检查依赖 ProfileSummaryInfo 等分析结果。

### Lines 60-76

```cpp
/// is present, functions in the profile symbol list but without profile will
/// be regarded as cold and much less inlining will happen in CGSCC inlining
/// pass, so we tend to lower the hot criteria here to allow more early
/// inlining to happen for warm callsites and it is helpful for performance.
bool callsiteIsHot(const FunctionSamples *CallsiteFS, ProfileSummaryInfo *PSI,
                   bool ProfAccForSymsInList) {
  if (!CallsiteFS)
    return false; // The callsite was not inlined in the original binary.

  assert(PSI && "PSI is expected to be non null");
  uint64_t CallsiteTotalSamples = CallsiteFS->getTotalSamples();
  if (ProfAccForSymsInList)
    return !PSI->isColdCount(CallsiteTotalSamples);
  else
    return PSI->isHotCount(CallsiteTotalSamples);
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ProfileSummaryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ProfileSummaryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 77-92

```cpp
/// Mark as used the sample record for the given function samples at
/// (LineOffset, Discriminator).
///
/// \returns true if this is the first time we mark the given record.
bool SampleCoverageTracker::markSamplesUsed(const FunctionSamples *FS,
                                            uint32_t LineOffset,
                                            uint32_t Discriminator,
                                            uint64_t Samples) {
  LineLocation Loc(LineOffset, Discriminator);
  unsigned &Count = SampleCoverage[FS][Loc];
  bool FirstTime = (++Count == 1);
  if (FirstTime)
    TotalUsedSamples += Samples;
  return FirstTime;
}

```
- EN: This region continues the SampleProfileLoaderBaseUtil implementation with local helper logic centered on Mark, LineOffset, Discriminator, SampleCoverageTracker.
- CN: 这一段延续了 SampleProfileLoaderBaseUtil 的主体实现，围绕 Mark, LineOffset, Discriminator, SampleCoverageTracker 等局部辅助逻辑展开。

### Lines 93-114

```cpp
/// Return the number of sample records that were applied from this profile.
///
/// This count does not include records from cold inlined callsites.
unsigned
SampleCoverageTracker::countUsedRecords(const FunctionSamples *FS,
                                        ProfileSummaryInfo *PSI) const {
  auto I = SampleCoverage.find(FS);

  // The size of the coverage map for FS represents the number of records
  // that were marked used at least once.
  unsigned Count = (I != SampleCoverage.end()) ? I->second.size() : 0;

  // If there are inlined callsites in this function, count the samples found
  // in the respective bodies. However, do not bother counting callees with 0
  // total samples, these are callees that were never invoked at runtime.
  for (const auto &I : FS->getCallsiteSamples())
    for (const auto &J : I.second) {
      const FunctionSamples *CalleeSamples = &J.second;
      if (callsiteIsHot(CalleeSamples, PSI, ProfAccForSymsInList))
        Count += countUsedRecords(CalleeSamples, PSI);
    }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ProfileSummaryInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ProfileSummaryInfo 等分析结果。

### Lines 115-136

```cpp
  return Count;
}

/// Return the number of sample records in the body of this profile.
///
/// This count does not include records from cold inlined callsites.
unsigned
SampleCoverageTracker::countBodyRecords(const FunctionSamples *FS,
                                        ProfileSummaryInfo *PSI) const {
  unsigned Count = FS->getBodySamples().size();

  // Only count records in hot callsites.
  for (const auto &I : FS->getCallsiteSamples())
    for (const auto &J : I.second) {
      const FunctionSamples *CalleeSamples = &J.second;
      if (callsiteIsHot(CalleeSamples, PSI, ProfAccForSymsInList))
        Count += countBodyRecords(CalleeSamples, PSI);
    }

  return Count;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ProfileSummaryInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ProfileSummaryInfo 等分析结果。

### Lines 137-157

```cpp
/// Return the number of samples collected in the body of this profile.
///
/// This count does not include samples from cold inlined callsites.
uint64_t
SampleCoverageTracker::countBodySamples(const FunctionSamples *FS,
                                        ProfileSummaryInfo *PSI) const {
  uint64_t Total = 0;
  for (const auto &I : FS->getBodySamples())
    Total += I.second.getSamples();

  // Only count samples in hot callsites.
  for (const auto &I : FS->getCallsiteSamples())
    for (const auto &J : I.second) {
      const FunctionSamples *CalleeSamples = &J.second;
      if (callsiteIsHot(CalleeSamples, PSI, ProfAccForSymsInList))
        Total += countBodySamples(CalleeSamples, PSI);
    }

  return Total;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ProfileSummaryInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ProfileSummaryInfo 等分析结果。

### Lines 158-175

```cpp
/// Return the fraction of sample records used in this profile.
///
/// The returned value is an unsigned integer in the range 0-100 indicating
/// the percentage of sample records that were used while applying this
/// profile to the associated function.
unsigned SampleCoverageTracker::computeCoverage(unsigned Used,
                                                unsigned Total) const {
  assert(Used <= Total &&
         "number of used records cannot exceed the total number of records");
  return Total > 0 ? Used * 100 / Total : 100;
}

/// Create a global variable to flag FSDiscriminators are used.
void createFSDiscriminatorVariable(Module *M) {
  const char *FSDiscriminatorVar = "__llvm_fs_discriminator__";
  if (M->getGlobalVariable(FSDiscriminatorVar))
    return;

```
- EN: Core entities appearing here include createFSDiscriminatorVariable, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 createFSDiscriminatorVariable，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 176-185

```cpp
  auto &Context = M->getContext();
  // Place this variable to llvm.used so it won't be GC'ed.
  appendToUsed(*M, {new GlobalVariable(*M, Type::getInt1Ty(Context), true,
                                       GlobalValue::WeakODRLinkage,
                                       ConstantInt::getTrue(Context),
                                       FSDiscriminatorVar)});
}

} // end of namespace sampleprofutil
} // end of namespace llvm
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `createFSDiscriminatorVariable` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`createFSDiscriminatorVariable` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `ProfileSummaryInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `ProfileSummaryInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/ProfileSummaryInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/ProfileSummaryInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Constants.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/SampleProfileLoaderBaseUtil.h`, `llvm/Transforms/Utils/ModuleUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Constants.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/SampleProfileLoaderBaseUtil.h`, `llvm/Transforms/Utils/ModuleUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Runtime analysis dependencies: `ProfileSummaryInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`ProfileSummaryInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
