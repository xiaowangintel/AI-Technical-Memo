# MemoryOpRemark.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/MemoryOpRemark.cpp` | `llvm/lib/Transforms/Utils/MemoryOpRemark.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements auto-init remark analysis within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 MemoryOpRemark 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-35

```cpp
//===-- MemoryOpRemark.cpp - Auto-init remark analysis---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Implementation of the analysis for the "auto-init" remark.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/MemoryOpRemark.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/Analysis/OptimizationRemarkEmitter.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/DebugInfo.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include <optional>

using namespace llvm;
using namespace llvm::ore;

MemoryOpRemark::~MemoryOpRemark() = default;

bool MemoryOpRemark::canHandle(const Instruction *I, const TargetLibraryInfo &TLI) {
  if (isa<StoreInst>(I))
    return true;

  if (auto *II = dyn_cast<IntrinsicInst>(I)) {
    switch (II->getIntrinsicID()) {
    case Intrinsic::memcpy_inline:
    case Intrinsic::memcpy:
    case Intrinsic::memmove:
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include canHandle, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 canHandle，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 36-75

```cpp
    case Intrinsic::memset:
    case Intrinsic::memcpy_element_unordered_atomic:
    case Intrinsic::memmove_element_unordered_atomic:
    case Intrinsic::memset_element_unordered_atomic:
      return true;
    default:
      return false;
    }
  }

  if (auto *CI = dyn_cast<CallInst>(I)) {
    auto *CF = CI->getCalledFunction();
    if (!CF)
      return false;

    if (!CF->hasName())
      return false;

    LibFunc LF;
    bool KnownLibCall = TLI.getLibFunc(*CF, LF) && TLI.has(LF);
    if (!KnownLibCall)
      return false;

    switch (LF) {
    case LibFunc_memcpy_chk:
    case LibFunc_mempcpy_chk:
    case LibFunc_memset_chk:
    case LibFunc_memmove_chk:
    case LibFunc_memcpy:
    case LibFunc_mempcpy:
    case LibFunc_memset:
    case LibFunc_memmove:
    case LibFunc_bzero:
    case LibFunc_bcopy:
      return true;
    default:
      return false;
    }
  }

```
- EN: This region continues the MemoryOpRemark implementation with local helper logic centered on Intrinsic, CallInst, LibFunc, KnownLibCall.
- CN: 这一段延续了 MemoryOpRemark 的主体实现，围绕 Intrinsic, CallInst, LibFunc, KnownLibCall 等局部辅助逻辑展开。

### Lines 76-113

```cpp
  return false;
}

void MemoryOpRemark::visit(const Instruction *I) {
  // For some of them, we can provide more information:

  // For stores:
  // * size
  // * volatile / atomic
  if (auto *SI = dyn_cast<StoreInst>(I)) {
    visitStore(*SI);
    return;
  }

  // For intrinsics:
  // * user-friendly name
  // * size
  if (auto *II = dyn_cast<IntrinsicInst>(I)) {
    visitIntrinsicCall(*II);
    return;
  }

  // For calls:
  // * known/unknown function (e.g. the compiler knows bzero, but it doesn't
  //                                know my_bzero)
  // * memory operation size
  if (auto *CI = dyn_cast<CallInst>(I)) {
    visitCall(*CI);
    return;
  }

  visitUnknown(*I);
}

std::string MemoryOpRemark::explainSource(StringRef Type) const {
  return (Type + ".").str();
}

```
- EN: Core entities appearing here include visit, explainSource, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 visit, explainSource，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 114-148

```cpp
StringRef MemoryOpRemark::remarkName(RemarkKind RK) const {
  switch (RK) {
  case RK_Store:
    return "MemoryOpStore";
  case RK_Unknown:
    return "MemoryOpUnknown";
  case RK_IntrinsicCall:
    return "MemoryOpIntrinsicCall";
  case RK_Call:
    return "MemoryOpCall";
  }
  llvm_unreachable("missing RemarkKind case");
}

static void inlineVolatileOrAtomicWithExtraArgs(bool *Inline, bool Volatile,
                                                bool Atomic,
                                                DiagnosticInfoIROptimization &R) {
  if (Inline && *Inline)
    R << " Inlined: " << NV("StoreInlined", true) << ".";
  if (Volatile)
    R << " Volatile: " << NV("StoreVolatile", true) << ".";
  if (Atomic)
    R << " Atomic: " << NV("StoreAtomic", true) << ".";
  // Emit the false cases under ExtraArgs. This won't show them in the remark
  // message but will end up in the serialized remarks.
  if ((Inline && !*Inline) || !Volatile || !Atomic)
    R << setExtraArgs();
  if (Inline && !*Inline)
    R << " Inlined: " << NV("StoreInlined", false) << ".";
  if (!Volatile)
    R << " Volatile: " << NV("StoreVolatile", false) << ".";
  if (!Atomic)
    R << " Atomic: " << NV("StoreAtomic", false) << ".";
}

```
- EN: Core entities appearing here include remarkName, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 remarkName，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 149-187

```cpp
static std::optional<uint64_t>
getSizeInBytes(std::optional<uint64_t> SizeInBits) {
  if (!SizeInBits || *SizeInBits % 8 != 0)
    return std::nullopt;
  return *SizeInBits / 8;
}

template<typename ...Ts>
std::unique_ptr<DiagnosticInfoIROptimization>
MemoryOpRemark::makeRemark(Ts... Args) {
  switch (diagnosticKind()) {
  case DK_OptimizationRemarkAnalysis:
    return std::make_unique<OptimizationRemarkAnalysis>(Args...);
  case DK_OptimizationRemarkMissed:
    return std::make_unique<OptimizationRemarkMissed>(Args...);
  default:
    llvm_unreachable("unexpected DiagnosticKind");
  }
}

void MemoryOpRemark::visitStore(const StoreInst &SI) {
  bool Volatile = SI.isVolatile();
  bool Atomic = SI.isAtomic();
  int64_t Size = DL.getTypeStoreSize(SI.getOperand(0)->getType());

  auto R = makeRemark(RemarkPass.data(), remarkName(RK_Store), &SI);
  *R << explainSource("Store") << "\nStore size: " << NV("StoreSize", Size)
     << " bytes.";
  visitPtr(SI.getOperand(1), /*IsRead=*/false, *R);
  inlineVolatileOrAtomicWithExtraArgs(nullptr, Volatile, Atomic, *R);
  ORE.emit(*R);
}

void MemoryOpRemark::visitUnknown(const Instruction &I) {
  auto R = makeRemark(RemarkPass.data(), remarkName(RK_Unknown), &I);
  *R << explainSource("Initialization");
  ORE.emit(*R);
}

```
- EN: Core entities appearing here include getSizeInBytes, makeRemark, visitStore, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getSizeInBytes, makeRemark, visitStore，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 188-225

```cpp
void MemoryOpRemark::visitIntrinsicCall(const IntrinsicInst &II) {
  SmallString<32> CallTo;
  bool Atomic = false;
  bool Inline = false;
  switch (II.getIntrinsicID()) {
  case Intrinsic::memcpy_inline:
    CallTo = "memcpy";
    Inline = true;
    break;
  case Intrinsic::memcpy:
    CallTo = "memcpy";
    break;
  case Intrinsic::memmove:
    CallTo = "memmove";
    break;
  case Intrinsic::memset:
    CallTo = "memset";
    break;
  case Intrinsic::memcpy_element_unordered_atomic:
    CallTo = "memcpy";
    Atomic = true;
    break;
  case Intrinsic::memmove_element_unordered_atomic:
    CallTo = "memmove";
    Atomic = true;
    break;
  case Intrinsic::memset_element_unordered_atomic:
    CallTo = "memset";
    Atomic = true;
    break;
  default:
    return visitUnknown(II);
  }

  auto R = makeRemark(RemarkPass.data(), remarkName(RK_IntrinsicCall), &II);
  visitCallee(CallTo.str(), /*KnownLibCall=*/true, *R);
  visitSizeOperand(II.getOperand(2), *R);

```
- EN: Core entities appearing here include visitIntrinsicCall, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 visitIntrinsicCall，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 226-258

```cpp
  auto *CIVolatile = dyn_cast<ConstantInt>(II.getOperand(3));
  // No such thing as a memory intrinsic that is both atomic and volatile.
  bool Volatile = !Atomic && CIVolatile && CIVolatile->getZExtValue();
  switch (II.getIntrinsicID()) {
  case Intrinsic::memcpy_inline:
  case Intrinsic::memcpy:
  case Intrinsic::memmove:
  case Intrinsic::memcpy_element_unordered_atomic:
    visitPtr(II.getOperand(1), /*IsRead=*/true, *R);
    visitPtr(II.getOperand(0), /*IsRead=*/false, *R);
    break;
  case Intrinsic::memset:
  case Intrinsic::memset_element_unordered_atomic:
    visitPtr(II.getOperand(0), /*IsRead=*/false, *R);
    break;
  }
  inlineVolatileOrAtomicWithExtraArgs(&Inline, Volatile, Atomic, *R);
  ORE.emit(*R);
}

void MemoryOpRemark::visitCall(const CallInst &CI) {
  Function *F = CI.getCalledFunction();
  if (!F)
    return visitUnknown(CI);

  LibFunc LF;
  bool KnownLibCall = TLI.getLibFunc(*F, LF) && TLI.has(LF);
  auto R = makeRemark(RemarkPass.data(), remarkName(RK_Call), &CI);
  visitCallee(F, KnownLibCall, *R);
  visitKnownLibCall(CI, LF, *R);
  ORE.emit(*R);
}

```
- EN: Core entities appearing here include visitCall, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 visitCall，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 259-295

```cpp
template <typename FTy>
void MemoryOpRemark::visitCallee(FTy F, bool KnownLibCall,
                                 DiagnosticInfoIROptimization &R) {
  R << "Call to ";
  if (!KnownLibCall)
    R << NV("UnknownLibCall", "unknown") << " function ";
  R << NV("Callee", F) << explainSource("");
}

void MemoryOpRemark::visitKnownLibCall(const CallInst &CI, LibFunc LF,
                                       DiagnosticInfoIROptimization &R) {
  switch (LF) {
  default:
    return;
  case LibFunc_memset_chk:
  case LibFunc_memset:
    visitSizeOperand(CI.getOperand(2), R);
    visitPtr(CI.getOperand(0), /*IsRead=*/false, R);
    break;
  case LibFunc_bzero:
    visitSizeOperand(CI.getOperand(1), R);
    visitPtr(CI.getOperand(0), /*IsRead=*/false, R);
    break;
  case LibFunc_memcpy_chk:
  case LibFunc_mempcpy_chk:
  case LibFunc_memmove_chk:
  case LibFunc_memcpy:
  case LibFunc_mempcpy:
  case LibFunc_memmove:
  case LibFunc_bcopy:
    visitSizeOperand(CI.getOperand(2), R);
    visitPtr(CI.getOperand(1), /*IsRead=*/true, R);
    visitPtr(CI.getOperand(0), /*IsRead=*/false, R);
    break;
  }
}

```
- EN: This region continues the MemoryOpRemark implementation with local helper logic centered on FTy, MemoryOpRemark, KnownLibCall, DiagnosticInfoIROptimization.
- CN: 这一段延续了 MemoryOpRemark 的主体实现，围绕 FTy, MemoryOpRemark, KnownLibCall, DiagnosticInfoIROptimization 等局部辅助逻辑展开。

### Lines 296-335

```cpp
void MemoryOpRemark::visitSizeOperand(Value *V, DiagnosticInfoIROptimization &R) {
  if (auto *Len = dyn_cast<ConstantInt>(V)) {
    uint64_t Size = Len->getZExtValue();
    R << " Memory operation size: " << NV("StoreSize", Size) << " bytes.";
  }
}

static std::optional<StringRef> nameOrNone(const Value *V) {
  if (V->hasName())
    return V->getName();
  return std::nullopt;
}

void MemoryOpRemark::visitVariable(const Value *V,
                                   SmallVectorImpl<VariableInfo> &Result) {
  if (auto *GV = dyn_cast<GlobalVariable>(V)) {
    auto *Ty = GV->getValueType();
    uint64_t Size = DL.getTypeSizeInBits(Ty).getFixedValue();
    VariableInfo Var{nameOrNone(GV), Size};
    if (!Var.isEmpty())
      Result.push_back(std::move(Var));
    return;
  }

  // If we find some information in the debug info, take that.
  bool FoundDI = false;
  // Try to get a dbg.declare, which has a DILocalVariable giving us the
  // real debug info name and size of the variable.
  auto FindDI = [&](const DbgVariableRecord *DVI) {
    if (DILocalVariable *DILV = DVI->getVariable()) {
      std::optional<uint64_t> DISize = getSizeInBytes(DILV->getSizeInBits());
      VariableInfo Var{DILV->getName(), DISize};
      if (!Var.isEmpty()) {
        Result.push_back(std::move(Var));
        FoundDI = true;
      }
    }
  };
  for_each(findDVRDeclares(const_cast<Value *>(V)), FindDI);

```
- EN: Core entities appearing here include visitSizeOperand, nameOrNone, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 visitSizeOperand, nameOrNone，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 336-370

```cpp
  if (FoundDI) {
    assert(!Result.empty());
    return;
  }

  const auto *AI = dyn_cast<AllocaInst>(V);
  if (!AI)
    return;

  // If not, get it from the alloca.
  std::optional<TypeSize> TySize = AI->getAllocationSize(DL);
  std::optional<uint64_t> Size =
      TySize ? std::optional(TySize->getFixedValue()) : std::nullopt;
  VariableInfo Var{nameOrNone(AI), Size};
  if (!Var.isEmpty())
    Result.push_back(std::move(Var));
}

void MemoryOpRemark::visitPtr(Value *Ptr, bool IsRead, DiagnosticInfoIROptimization &R) {
  // Find if Ptr is a known variable we can give more information on.
  SmallVector<Value *, 2> Objects;
  getUnderlyingObjectsForCodeGen(Ptr, Objects);
  SmallVector<VariableInfo, 2> VIs;
  for (const Value *V : Objects)
    visitVariable(V, VIs);

  if (VIs.empty()) {
    bool CanBeNull;
    bool CanBeFreed;
    uint64_t Size = Ptr->getPointerDereferenceableBytes(DL, CanBeNull, CanBeFreed);
    if (!Size)
      return;
    VIs.push_back({std::nullopt, Size});
  }

```
- EN: Core entities appearing here include visitPtr, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 visitPtr，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 371-405

```cpp
  R << (IsRead ? "\n Read Variables: " : "\n Written Variables: ");
  for (unsigned i = 0; i < VIs.size(); ++i) {
    const VariableInfo &VI = VIs[i];
    assert(!VI.isEmpty() && "No extra content to display.");
    if (i != 0)
      R << ", ";
    if (VI.Name)
      R << NV(IsRead ? "RVarName" : "WVarName", *VI.Name);
    else
      R << NV(IsRead ? "RVarName" : "WVarName", "<unknown>");
    if (VI.Size)
      R << " (" << NV(IsRead ? "RVarSize" : "WVarSize", *VI.Size) << " bytes)";
  }
  R << ".";
}

bool AutoInitRemark::canHandle(const Instruction *I) {
  if (!I->hasMetadata(LLVMContext::MD_annotation))
    return false;
  return any_of(I->getMetadata(LLVMContext::MD_annotation)->operands(),
                [](const MDOperand &Op) {
                  return isa<MDString>(Op.get()) &&
                         cast<MDString>(Op.get())->getString() == "auto-init";
                });
}

std::string AutoInitRemark::explainSource(StringRef Type) const {
  return (Type + " inserted by -ftrivial-auto-var-init.").str();
}

StringRef AutoInitRemark::remarkName(RemarkKind RK) const {
  switch (RK) {
  case RK_Store:
    return "AutoInitStore";
  case RK_Unknown:
```
- EN: Core entities appearing here include canHandle, explainSource, remarkName, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 canHandle, explainSource, remarkName，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 406-413

```cpp
    return "AutoInitUnknownInstruction";
  case RK_IntrinsicCall:
    return "AutoInitIntrinsicCall";
  case RK_Call:
    return "AutoInitCall";
  }
  llvm_unreachable("missing RemarkKind case");
}
```
- EN: This region continues the MemoryOpRemark implementation with local helper logic centered on AutoInitUnknownInstruction, RK_IntrinsicCall, AutoInitIntrinsicCall, RK_Call.
- CN: 这一段延续了 MemoryOpRemark 的主体实现，围绕 AutoInitUnknownInstruction, RK_IntrinsicCall, AutoInitIntrinsicCall, RK_Call 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `canHandle, visit, explainSource, remarkName, getSizeInBytes, makeRemark, visitStore, visitUnknown` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`canHandle, visit, explainSource, remarkName, getSizeInBytes, makeRemark, visitStore, visitUnknown` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `OptimizationRemarkEmitter, TargetLibraryInfo, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `OptimizationRemarkEmitter, TargetLibraryInfo, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/DebugInfo.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/Transforms/Utils/MemoryOpRemark.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DebugInfo.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/Transforms/Utils/MemoryOpRemark.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/SmallString.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/SmallString.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `optional` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`optional` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `OptimizationRemarkEmitter`, `TargetLibraryInfo`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`OptimizationRemarkEmitter`, `TargetLibraryInfo`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
