# SeedCollector.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/SeedCollector.cpp` | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/SeedCollector.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Seed Collector within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 SeedCollector 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

```cpp
//===- SeedCollector.cpp  -------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Vectorize/SandboxVectorizer/SeedCollector.h"
#include "llvm/Analysis/LoopAccessAnalysis.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/Type.h"
#include "llvm/SandboxIR/Instruction.h"
#include "llvm/SandboxIR/Utils.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Debug.h"

using namespace llvm;
namespace llvm::sandboxir {

static cl::opt<unsigned> SeedBundleSizeLimit(
    "sbvec-seed-bundle-size-limit", cl::init(32), cl::Hidden,
    cl::desc("Limit the size of the seed bundle to cap compilation time."));

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 25-46

```cpp
static cl::opt<unsigned> SeedGroupsLimit(
    "sbvec-seed-groups-limit", cl::init(256), cl::Hidden,
    cl::desc("Limit the number of collected seeds groups in a BB to "
             "cap compilation time."));

ArrayRef<Instruction *> SeedBundle::getSlice(unsigned StartIdx,
                                             unsigned MaxVecRegBits,
                                             bool ForcePowerOf2) {
  // Use uint32_t here for compatibility with IsPowerOf2_32

  // BitCount tracks the size of the working slice. From that we can tell
  // when the working slice's size is a power-of-two and when it exceeds
  // the legal size in MaxVecBits.
  uint32_t BitCount = 0;
  uint32_t NumElements = 0;
  // Tracks the most recent slice where NumElements gave a power-of-2 BitCount
  uint32_t NumElementsPowerOfTwo = 0;
  uint32_t BitCountPowerOfTwo = 0;
  // Can't start a slice with a used instruction.
  assert(!isUsed(StartIdx) && "Expected unused at StartIdx");
  for (Instruction *S : drop_begin(Seeds, StartIdx)) {
    // Stop if this instruction is used. This needs to be done before
```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 47-65

```cpp
    // getNumBits() because a "used" instruction may have been erased.
    if (isUsed(StartIdx + NumElements))
      break;
    uint32_t InstBits = Utils::getNumBits(S);
    // Stop if adding it puts the slice over the limit.
    if (BitCount + InstBits > MaxVecRegBits)
      break;
    NumElements++;
    BitCount += InstBits;
    if (ForcePowerOf2 && isPowerOf2_32(BitCount)) {
      NumElementsPowerOfTwo = NumElements;
      BitCountPowerOfTwo = BitCount;
    }
  }
  if (ForcePowerOf2) {
    NumElements = NumElementsPowerOfTwo;
    BitCount = BitCountPowerOfTwo;
  }

```
- EN: This region continues the SeedCollector implementation with local helper logic centered on StartIdx, NumElements, InstBits, Utils.
- CN: 这一段延续了 SeedCollector 的主体实现，围绕 StartIdx, NumElements, InstBits, Utils 等局部辅助逻辑展开。

### Lines 66-87

```cpp
  // Return any non-empty slice
  if (NumElements > 1) {
    assert((!ForcePowerOf2 || isPowerOf2_32(BitCount)) &&
           "Must be a power of two");
    return ArrayRef<Instruction *>(&Seeds[StartIdx], NumElements);
  }
  return {};
}

template <typename LoadOrStoreT>
SeedContainer::KeyT SeedContainer::getKey(LoadOrStoreT *LSI,
                                          bool AllowDiffTypes) const {
  assert((isa<LoadInst>(LSI) || isa<StoreInst>(LSI)) &&
         "Expected Load or Store!");
  Value *Ptr = Utils::getMemInstructionBase(LSI);
  Instruction::Opcode Op = LSI->getOpcode();
  Type *Ty;
  if (AllowDiffTypes) {
    Ty = nullptr;
  } else {
    Ty = Utils::getExpectedType(LSI);
    if (auto *VTy = dyn_cast<VectorType>(Ty))
```
- EN: This region continues the SeedCollector implementation with local helper logic centered on Return, NumElements, ForcePowerOf2, BitCount.
- CN: 这一段延续了 SeedCollector 的主体实现，围绕 Return, NumElements, ForcePowerOf2, BitCount 等局部辅助逻辑展开。

### Lines 88-108

```cpp
      Ty = VTy->getElementType();
  }
  return {Ptr, Ty, Op};
}

// Explicit instantiations
template SeedContainer::KeyT
SeedContainer::getKey<LoadInst>(LoadInst *LSI, bool AllowDiffTypes) const;
template SeedContainer::KeyT
SeedContainer::getKey<StoreInst>(StoreInst *LSI, bool AllowDiffTypes) const;

bool SeedContainer::erase(Instruction *I) {
  assert((isa<LoadInst>(I) || isa<StoreInst>(I)) && "Expected Load or Store!");
  auto It = SeedLookupMap.find(I);
  if (It == SeedLookupMap.end())
    return false;
  SeedBundle *Bndl = It->second;
  Bndl->setUsed(I);
  return true;
}

```
- EN: Core entities appearing here include erase, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 erase，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 109-129

```cpp
template <typename LoadOrStoreT>
void SeedContainer::insert(LoadOrStoreT *LSI, bool AllowDiffTypes) {
  // Find the bundle containing seeds for this symbol and type-of-access.
  auto &BundleVec = Bundles[getKey(LSI, AllowDiffTypes)];
  // Fill this vector of bundles front to back so that only the last bundle in
  // the vector may have available space. This avoids iteration to find one with
  // space.
  if (BundleVec.empty() || BundleVec.back()->size() == SeedBundleSizeLimit)
    BundleVec.emplace_back(std::make_unique<MemSeedBundle<LoadOrStoreT>>(LSI));
  else
    BundleVec.back()->insert(LSI, SE);

  SeedLookupMap[LSI] = BundleVec.back().get();
}

// Explicit instantiations
template LLVM_EXPORT_TEMPLATE void SeedContainer::insert<LoadInst>(LoadInst *,
                                                                   bool);
template LLVM_EXPORT_TEMPLATE void SeedContainer::insert<StoreInst>(StoreInst *,
                                                                    bool);

```
- EN: Core entities appearing here include insert, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 insert，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 130-149

```cpp
#ifndef NDEBUG
void SeedContainer::print(raw_ostream &OS) const {
  for (const auto &Pair : Bundles) {
    auto [I, Ty, Opc] = Pair.first;
    const auto &SeedsVec = Pair.second;
    std::string RefType = dyn_cast<LoadInst>(I)    ? "Load"
                          : dyn_cast<StoreInst>(I) ? "Store"
                                                   : "Other";
    OS << "[Inst=" << *I << " Ty=" << Ty << " " << RefType << "]\n";
    for (const auto &SeedPtr : SeedsVec) {
      SeedPtr->dump(OS);
      OS << "\n";
    }
  }
  OS << "\n";
}

LLVM_DUMP_METHOD void SeedContainer::dump() const { print(dbgs()); }
#endif // NDEBUG

```
- EN: Core entities appearing here include print, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 print，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 150-172

```cpp
template <typename LoadOrStoreT> static bool isValidMemSeed(LoadOrStoreT *LSI) {
  if (!LSI->isSimple())
    return false;
  auto *Ty = Utils::getExpectedType(LSI);
  // Omit types that are architecturally unvectorizable
  if (Ty->isX86_FP80Ty() || Ty->isPPC_FP128Ty())
    return false;
  // Omit vector types without compile-time-known lane counts
  if (isa<ScalableVectorType>(Ty))
    return false;
  if (auto *VTy = dyn_cast<FixedVectorType>(Ty))
    return VectorType::isValidElementType(VTy->getElementType());
  return VectorType::isValidElementType(Ty);
}

template bool isValidMemSeed<LoadInst>(LoadInst *LSI);
template bool isValidMemSeed<StoreInst>(StoreInst *LSI);

SeedCollector::SeedCollector(BasicBlock *BB, ScalarEvolution &SE,
                             bool CollectStores, bool CollectLoads,
                             bool AllowDiffTypes)
    : StoreSeeds(SE), LoadSeeds(SE), Ctx(BB->getContext()) {

```
- EN: Core entities appearing here include d, StoreSeeds, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution.
- CN: 此处出现的核心实体包括 d, StoreSeeds，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution 等分析结果。

### Lines 173-196

```cpp
  if (!CollectStores && !CollectLoads)
    return;

  EraseCallbackID = Ctx.registerEraseInstrCallback([this](Instruction *I) {
    if (auto SI = dyn_cast<StoreInst>(I))
      StoreSeeds.erase(SI);
    else if (auto LI = dyn_cast<LoadInst>(I))
      LoadSeeds.erase(LI);
  });

  // Actually collect the seeds.
  for (auto &I : *BB) {
    if (StoreInst *SI = dyn_cast<StoreInst>(&I))
      if (CollectStores && isValidMemSeed(SI))
        StoreSeeds.insert(SI, AllowDiffTypes);
    if (LoadInst *LI = dyn_cast<LoadInst>(&I))
      if (CollectLoads && isValidMemSeed(LI))
        LoadSeeds.insert(LI, AllowDiffTypes);
    // Cap compilation time.
    if (totalNumSeedGroups() > SeedGroupsLimit)
      break;
  }
}

```
- EN: This region continues the SeedCollector implementation with local helper logic centered on CollectStores, CollectLoads, EraseCallbackID, Ctx.
- CN: 这一段延续了 SeedCollector 的主体实现，围绕 CollectStores, CollectLoads, EraseCallbackID, Ctx 等局部辅助逻辑展开。

### Lines 197-212

```cpp
SeedCollector::~SeedCollector() {
  Ctx.unregisterEraseInstrCallback(EraseCallbackID);
}

#ifndef NDEBUG
void SeedCollector::print(raw_ostream &OS) const {
  OS << "=== StoreSeeds ===\n";
  StoreSeeds.print(OS);
  OS << "=== LoadSeeds ===\n";
  LoadSeeds.print(OS);
}

void SeedCollector::dump() const { print(dbgs()); }
#endif

} // namespace llvm::sandboxir
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include ~SeedCollector, print, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 ~SeedCollector, print，它们承载了该区间涉及的局部状态、声明或变换入口。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `erase, insert, print, d, StoreSeeds, ~SeedCollector` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`erase, insert, print, d, StoreSeeds, ~SeedCollector` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `ScalarEvolution, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `ScalarEvolution, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `SeedBundleSizeLimit, SeedGroupsLimit` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `SeedBundleSizeLimit, SeedGroupsLimit` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/LoopAccessAnalysis.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/LoopAccessAnalysis.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Type.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/SeedCollector.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Type.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/SeedCollector.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/SandboxIR/Instruction.h`, `llvm/SandboxIR/Utils.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/SandboxIR/Instruction.h`, `llvm/SandboxIR/Utils.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `ScalarEvolution`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`ScalarEvolution`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
