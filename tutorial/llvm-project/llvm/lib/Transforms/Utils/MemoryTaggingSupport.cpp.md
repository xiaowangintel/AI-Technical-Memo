# MemoryTaggingSupport.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/MemoryTaggingSupport.cpp` | `llvm/lib/Transforms/Utils/MemoryTaggingSupport.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements == MemoryTaggingSupport.cpp - helpers for memory tagging implementations === within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 MemoryTaggingSupport 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-36

```cpp
//== MemoryTaggingSupport.cpp - helpers for memory tagging implementations ===//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares common infrastructure for HWAddressSanitizer and
// Aarch64StackTagging.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/MemoryTaggingSupport.h"

#include "llvm/ADT/STLExtras.h"
#include "llvm/Analysis/CFG.h"
#include "llvm/Analysis/PostDominators.h"
#include "llvm/Analysis/StackSafetyAnalysis.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/BinaryFormat/Dwarf.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/TargetParser/Triple.h"
#include "llvm/Transforms/Utils/PromoteMemToReg.h"
#include <utility>

namespace llvm {
namespace memtag {

void forAllReachableExits(const DominatorTree &DT, const PostDominatorTree &PDT,
                          const LoopInfo &LI, const AllocaInfo &AInfo,
                          const SmallVectorImpl<Instruction *> &RetVec,
                          llvm::function_ref<void(Instruction *)> Callback) {
  if (AInfo.LifetimeEnd.size() == 1 && AInfo.LifetimeStart.size() == 1 &&
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, PostDominatorTree, ValueTracking.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, PostDominatorTree, ValueTracking 等分析结果。

### Lines 37-72

```cpp
      PDT.dominates(AInfo.LifetimeEnd[0], AInfo.LifetimeStart[0])) {
    Callback(AInfo.LifetimeEnd[0]);
    return;
  }
  SmallPtrSet<BasicBlock *, 2> EndBlocks;
  SmallVector<BasicBlock *, 2> StartBlocks;
  for (const auto &[BB, BBInfo] : AInfo.BBInfos) {
    if (BBInfo.Last == Intrinsic::lifetime_end)
      EndBlocks.insert(BB);
    else
      StartBlocks.push_back(BB);
  }

  if (!StartBlocks.empty()) {
    for (auto *RI : RetVec) {
      auto WL = StartBlocks;
      // If the block with the return is an EndBlock (i.e. a block where the
      // last relevant lifetime intrinsic is an end), we don't have to run a
      // complicated algorithm to know that the RetInst is never reachable
      // without going through an end.
      if (!EndBlocks.contains(RI->getParent()) &&
          isPotentiallyReachableFromMany(WL, RI->getParent(), &EndBlocks, &DT,
                                         &LI)) {
        Callback(RI);
      }
    }
  }
  for_each(AInfo.LifetimeEnd, Callback);
}

bool isSupportedLifetime(const AllocaInfo &AInfo, const DominatorTree *DT,
                         const LoopInfo *LI) {
  if (AInfo.LifetimeStart.empty())
    return false;
  SmallVector<BasicBlock *, 2> LastEndBlocks;
  SmallPtrSet<const BasicBlock *, 2> FirstEndBlocks;
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。

### Lines 73-108

```cpp
  SmallPtrSet<BasicBlock *, 2> StartBlocks;
  for_each(AInfo.BBInfos, [&](const auto &It) {
    const auto &[BB, BBI] = It;
    if (BBI.Last == Intrinsic::lifetime_end)
      LastEndBlocks.append(succ_begin(BB), succ_end(BB));
    else
      StartBlocks.insert(BB);
    if (BBI.First == Intrinsic::lifetime_end)
      FirstEndBlocks.insert(BB);
  });
  if (LastEndBlocks.empty() || FirstEndBlocks.empty())
    return true;
  return !isManyPotentiallyReachableFromMany(LastEndBlocks, FirstEndBlocks,
                                             &StartBlocks, DT, LI);
}

Instruction *getUntagLocationIfFunctionExit(Instruction &Inst) {
  if (isa<ReturnInst>(Inst)) {
    if (CallInst *CI = Inst.getParent()->getTerminatingMustTailCall())
      return CI;
    return &Inst;
  }
  if (isa<ResumeInst, CleanupReturnInst>(Inst)) {
    return &Inst;
  }
  return nullptr;
}

void StackInfoBuilder::visit(OptimizationRemarkEmitter &ORE,
                             Instruction &Inst) {
  // Visit non-intrinsic debug-info records attached to Inst.
  for (DbgVariableRecord &DVR : filterDbgVars(Inst.getDbgRecordRange())) {
    auto AddIfInteresting = [&](Value *V) {
      if (auto *AI = dyn_cast_or_null<AllocaInst>(V)) {
        if (getAllocaInterestingness(*AI) !=
            AllocaInterestingness::kInteresting)
```
- EN: Core entities appearing here include for_each, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 for_each，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 109-144

```cpp
          return;
        AllocaInfo &AInfo = Info.AllocasToInstrument[AI];
        auto &DVRVec = AInfo.DbgVariableRecords;
        if (DVRVec.empty() || DVRVec.back() != &DVR)
          DVRVec.push_back(&DVR);
      }
    };

    for_each(DVR.location_ops(), AddIfInteresting);
    if (DVR.isDbgAssign())
      AddIfInteresting(DVR.getAddress());
  }

  if (CallInst *CI = dyn_cast<CallInst>(&Inst)) {
    if (CI->canReturnTwice()) {
      Info.CallsReturnTwice = true;
    }
  }
  if (AllocaInst *AI = dyn_cast<AllocaInst>(&Inst)) {
    switch (getAllocaInterestingness(*AI)) {
    case AllocaInterestingness::kInteresting:
      Info.AllocasToInstrument[AI].AI = AI;
      ORE.emit([&]() {
        return OptimizationRemarkMissed(DebugType, "safeAlloca", &Inst);
      });
      break;
    case AllocaInterestingness::kSafe:
      ORE.emit(
          [&]() { return OptimizationRemark(DebugType, "safeAlloca", &Inst); });
      break;
    case AllocaInterestingness::kUninteresting:
      break;
    }
    return;
  }
  if (auto *II = dyn_cast<LifetimeIntrinsic>(&Inst)) {
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 145-180

```cpp
    AllocaInst *AI = dyn_cast<AllocaInst>(II->getArgOperand(0));
    if (!AI ||
        getAllocaInterestingness(*AI) != AllocaInterestingness::kInteresting)
      return;
    auto &AInfo = Info.AllocasToInstrument[AI];
    auto &BBInfo = AInfo.BBInfos[II->getParent()];

    if (II->getIntrinsicID() == Intrinsic::lifetime_start)
      AInfo.LifetimeStart.push_back(II);
    else if (BBInfo.Last != Intrinsic::lifetime_end)
      AInfo.LifetimeEnd.push_back(II);

    BBInfo.Last = II->getIntrinsicID();
    if (BBInfo.First == Intrinsic::not_intrinsic)
      BBInfo.First = II->getIntrinsicID();

    return;
  }

  Instruction *ExitUntag = getUntagLocationIfFunctionExit(Inst);
  if (ExitUntag)
    Info.RetVec.push_back(ExitUntag);
}

AllocaInterestingness
StackInfoBuilder::getAllocaInterestingness(const AllocaInst &AI) {
  std::optional<TypeSize> Size = AI.getAllocationSize(AI.getDataLayout());
  if (Size &&
      // FIXME: support vscale.
      !Size->isScalable() &&
      // FIXME: instrument dynamic allocas, too
      AI.isStaticAlloca() &&
      // alloca() may be called with 0 size, ignore it.
      !Size->isZero() &&
      // We are only interested in allocas not promotable to registers.
      // Promotable allocas are common under -O0.
```
- EN: Core entities appearing here include getAllocaInterestingness, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getAllocaInterestingness，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 181-216

```cpp
      !isAllocaPromotable(&AI) &&
      // inalloca allocas are not treated as static, and we don't want
      // dynamic alloca instrumentation for them as well.
      !AI.isUsedWithInAlloca() &&
      // swifterror allocas are register promoted by ISel
      !AI.isSwiftError()) {
    if (!(SSI && SSI->isSafe(AI))) {
      return AllocaInterestingness::kInteresting;
    }
    // safe allocas are not interesting
    return AllocaInterestingness::kSafe;
  }
  return AllocaInterestingness::kUninteresting;
}

uint64_t getAllocaSizeInBytes(const AllocaInst &AI) {
  auto DL = AI.getDataLayout();
  return *AI.getAllocationSize(DL);
}

void alignAndPadAlloca(memtag::AllocaInfo &Info, llvm::Align Alignment) {
  const Align NewAlignment = std::max(Info.AI->getAlign(), Alignment);
  Info.AI->setAlignment(NewAlignment);
  auto &Ctx = Info.AI->getFunction()->getContext();

  uint64_t Size = getAllocaSizeInBytes(*Info.AI);
  uint64_t AlignedSize = alignTo(Size, Alignment);
  if (Size == AlignedSize)
    return;

  // Add padding to the alloca.
  Type *AllocatedType =
      Info.AI->isArrayAllocation()
          ? ArrayType::get(
                Info.AI->getAllocatedType(),
                cast<ConstantInt>(Info.AI->getArraySize())->getZExtValue())
```
- EN: Core entities appearing here include getAllocaSizeInBytes, alignAndPadAlloca, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 此处出现的核心实体包括 getAllocaSizeInBytes, alignAndPadAlloca，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 217-249

```cpp
          : Info.AI->getAllocatedType();
  Type *PaddingType = ArrayType::get(Type::getInt8Ty(Ctx), AlignedSize - Size);
  Type *TypeWithPadding = StructType::get(AllocatedType, PaddingType);
  auto *NewAI = new AllocaInst(TypeWithPadding, Info.AI->getAddressSpace(),
                               nullptr, "", Info.AI->getIterator());
  NewAI->takeName(Info.AI);
  NewAI->setAlignment(Info.AI->getAlign());
  NewAI->setUsedWithInAlloca(Info.AI->isUsedWithInAlloca());
  NewAI->setSwiftError(Info.AI->isSwiftError());
  NewAI->copyMetadata(*Info.AI);

  Info.AI->replaceAllUsesWith(NewAI);
  Info.AI->eraseFromParent();
  Info.AI = NewAI;
}

Value *readRegister(IRBuilder<> &IRB, StringRef Name) {
  Module *M = IRB.GetInsertBlock()->getParent()->getParent();
  MDNode *MD =
      MDNode::get(M->getContext(), {MDString::get(M->getContext(), Name)});
  Value *Args[] = {MetadataAsValue::get(M->getContext(), MD)};
  return IRB.CreateIntrinsic(Intrinsic::read_register,
                             IRB.getIntPtrTy(M->getDataLayout()), Args);
}

Value *getPC(const Triple &TargetTriple, IRBuilder<> &IRB) {
  Module *M = IRB.GetInsertBlock()->getParent()->getParent();
  if (TargetTriple.getArch() == Triple::aarch64)
    return memtag::readRegister(IRB, "pc");
  return IRB.CreatePtrToInt(IRB.GetInsertBlock()->getParent(),
                            IRB.getIntPtrTy(M->getDataLayout()));
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 250-289

```cpp
Value *getFP(IRBuilder<> &IRB) {
  Function *F = IRB.GetInsertBlock()->getParent();
  Module *M = F->getParent();
  return IRB.CreatePtrToInt(
      IRB.CreateIntrinsic(Intrinsic::frameaddress,
                          IRB.getPtrTy(M->getDataLayout().getAllocaAddrSpace()),
                          {Constant::getNullValue(IRB.getInt32Ty())}),
      IRB.getIntPtrTy(M->getDataLayout()));
}

Value *getDarwinSlotPtr(IRBuilder<> &IRB, int Slot) {
  Module *M = IRB.GetInsertBlock()->getParent()->getParent();
  // FIXME: This should use the thread_pointer intrinsic. However, the
  // intrinsic is not currently implemented on Darwin correctly. The
  // TPIDRRO_EL0 register became part of the ABI to access TSD on recent
  // versions of OS and so it is safe to use here
  // Darwin provides a fixed slot for sanitizers at offset 231.
  MDNode *MD = MDNode::get(M->getContext(),
                           {MDString::get(M->getContext(), "TPIDRRO_EL0")});
  Value *Args[] = {MetadataAsValue::get(M->getContext(), MD)};
  return IRB.CreateConstGEP1_32(
      IRB.getInt8Ty(),
      IRB.CreateIntToPtr(
          IRB.CreateIntrinsic(Intrinsic::read_register,
                              {IRB.getIntPtrTy(M->getDataLayout())}, Args),
          IRB.getPtrTy()),
      8 * Slot);
}

Value *getAndroidSlotPtr(IRBuilder<> &IRB, int Slot) {
  Module *M = IRB.GetInsertBlock()->getParent()->getParent();
  // Android provides a fixed TLS slot for sanitizers. See TLS_SLOT_SANITIZER
  // in Bionic's libc/private/bionic_tls.h.
  Function *ThreadPointerFunc = Intrinsic::getOrInsertDeclaration(
      M, Intrinsic::thread_pointer,
      IRB.getPtrTy(M->getDataLayout().getDefaultGlobalsAddressSpace()));
  return IRB.CreateConstGEP1_32(IRB.getInt8Ty(),
                                IRB.CreateCall(ThreadPointerFunc), 8 * Slot);
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 290-325

```cpp
static DbgVariableRecord *DynCastToDbgAssign(DbgVariableRecord *DVR) {
  return DVR->isDbgAssign() ? DVR : nullptr;
}

void annotateDebugRecords(AllocaInfo &Info, unsigned int Tag) {
  auto AnnotateDbgRecord = [&](DbgVariableRecord *DPtr) {
    // Prepend "tag_offset, N" to the dwarf expression.
    // Tag offset logically applies to the alloca pointer, and it makes sense
    // to put it at the beginning of the expression.
    SmallVector<uint64_t, 8> NewOps = {dwarf::DW_OP_LLVM_tag_offset, Tag};
    for (size_t LocNo = 0; LocNo < DPtr->getNumVariableLocationOps(); ++LocNo)
      if (DPtr->getVariableLocationOp(LocNo) == Info.AI)
        DPtr->setExpression(
            DIExpression::appendOpsToArg(DPtr->getExpression(), NewOps, LocNo));
    if (auto *DAI = DynCastToDbgAssign(DPtr)) {
      if (DAI->getAddress() == Info.AI)
        DAI->setAddressExpression(
            DIExpression::prependOpcodes(DAI->getAddressExpression(), NewOps));
    }
  };

  llvm::for_each(Info.DbgVariableRecords, AnnotateDbgRecord);
}

Value *incrementThreadLong(IRBuilder<> &IRB, Value *ThreadLong,
                           unsigned int Inc, bool IsMemtagDarwin) {
  // Update the ring buffer. Top byte of ThreadLong defines the size of the
  // buffer in pages, it must be a power of two, and the start of the buffer
  // must be aligned by twice that much. Therefore wrap around of the ring
  // buffer is simply Addr &= ~((ThreadLong >> 56) << 12).
  // The use of AShr instead of LShr is due to
  //   https://bugs.llvm.org/show_bug.cgi?id=39030
  // Runtime library makes sure not to use the highest bit.
  //
  // Mechanical proof of this address calculation can be found at:
  // https://github.com/google/sanitizers/blob/master/hwaddress-sanitizer/prove_hwasanwrap.smt2
```
- EN: Core entities appearing here include annotateDebugRecords, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 annotateDebugRecords，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 326-356

```cpp
  //
  // Example of the wrap case for N = 1
  // Pointer:   0x01AAAAAAAAAAAFF8
  //                     +
  //            0x0000000000000008
  //                     =
  //            0x01AAAAAAAAAAB000
  //                     &
  // WrapMask:  0xFFFFFFFFFFFFF000
  //                     =
  //            0x01AAAAAAAAAAA000
  //
  // Then the WrapMask will be a no-op until the next wrap case.
  //
  // Darwin relies on bit 60-62 to store the size of the buffer in pages. This
  // limits N to [0,2] while the rest of the proof remains unchanged. Bits
  // 56-59 are avoided in order to prevent MTE Canonical Tag Faults while
  // accessing the ring buffer. Bit 63 is avoided to prevent unintentional
  // signed extension by AShr.
  assert((4096 % Inc) == 0);
  Value *WrapMask = IRB.CreateXor(
      IRB.CreateShl(IRB.CreateAShr(ThreadLong, IsMemtagDarwin ? 60 : 56), 12,
                    "", true, true),
      ConstantInt::get(ThreadLong->getType(), (uint64_t)-1));
  return IRB.CreateAnd(
      IRB.CreateAdd(ThreadLong, ConstantInt::get(ThreadLong->getType(), Inc)),
      WrapMask);
}

} // namespace memtag
} // namespace llvm
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `for_each, getAllocaInterestingness, getAllocaSizeInBytes, alignAndPadAlloca, annotateDebugRecords` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`for_each, getAllocaInterestingness, getAllocaSizeInBytes, alignAndPadAlloca, annotateDebugRecords` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, LoopInfo, OptimizationRemarkEmitter, PostDominatorTree, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, LoopInfo, OptimizationRemarkEmitter, PostDominatorTree, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/CFG.h`, `llvm/Analysis/PostDominators.h`, `llvm/Analysis/StackSafetyAnalysis.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/CFG.h`, `llvm/Analysis/PostDominators.h`, `llvm/Analysis/StackSafetyAnalysis.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Intrinsics.h`, `llvm/Transforms/Utils/MemoryTaggingSupport.h`, `llvm/Transforms/Utils/PromoteMemToReg.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Intrinsics.h`, `llvm/Transforms/Utils/MemoryTaggingSupport.h`, `llvm/Transforms/Utils/PromoteMemToReg.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/STLExtras.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/TargetParser/Triple.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/STLExtras.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/TargetParser/Triple.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `utility` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`utility` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `LoopInfo`, `OptimizationRemarkEmitter`, `PostDominatorTree`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `LoopInfo`, `OptimizationRemarkEmitter`, `PostDominatorTree`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
