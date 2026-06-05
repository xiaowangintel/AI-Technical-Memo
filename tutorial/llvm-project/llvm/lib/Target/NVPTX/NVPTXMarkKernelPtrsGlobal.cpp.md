# NVPTXMarkKernelPtrsGlobal.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXMarkKernelPtrsGlobal.cpp`
- Repository: `llvm-project`
- Purpose (EN): For CUDA kernels, pointers loaded from byval parameters are known to be in global address space.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXMarkKernelPtrsGlobal.cpp - Mark kernel pointers as global ----===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // For CUDA kernels, pointers loaded from byval parameters are known to be in
10: // global address space. This pass inserts addrspacecast pairs to make that
11: // explicit, enabling later address-space inference to propagate the global AS.
12: // It also handles the pattern where a pointer is loaded as an integer and then
13: // converted via inttoptr.
14: //
15: //===----------------------------------------------------------------------===//
16:
17: #include "NVPTX.h"
18: #include "NVVMProperties.h"
19: #include "llvm/Analysis/ValueTracking.h"
20: #include "llvm/IR/InstIterator.h"
21: #include "llvm/IR/Instructions.h"
22: #include "llvm/InitializePasses.h"
23: #include "llvm/Pass.h"
24: #include "llvm/Support/NVPTXAddrSpace.h"
25:
26: using namespace llvm;
27: using namespace NVPTXAS;
28:
29: static void markPointerAsAS(Value *Ptr, unsigned AS) {
30:   if (Ptr->getType()->getPointerAddressSpace() != ADDRESS_SPACE_GENERIC)
31:     return;
32:
33:   BasicBlock::iterator InsertPt;
34:   if (auto *Arg = dyn_cast<Argument>(Ptr)) {
35:     InsertPt = Arg->getParent()->getEntryBlock().begin();
36:   } else {
37:     InsertPt = ++cast<Instruction>(Ptr)->getIterator();
38:     assert(InsertPt != InsertPt->getParent()->end() &&
39:            "We don't call this function with Ptr being a terminator.");
40:   }
```
- EN: This range implements operational logic in helpers such as markPointerAsAS, getParent, getIterator, translating backend policy into executable code.
- CN: 这一段实现了 markPointerAsAS、getParent、getIterator 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:
42:   Instruction *PtrInGlobal = new AddrSpaceCastInst(
43:       Ptr, PointerType::get(Ptr->getContext(), AS), Ptr->getName(), InsertPt);
44:   Value *PtrInGeneric = new AddrSpaceCastInst(PtrInGlobal, Ptr->getType(),
45:                                               Ptr->getName(), InsertPt);
46:   Ptr->replaceAllUsesWith(PtrInGeneric);
47:   PtrInGlobal->setOperand(0, Ptr);
48: }
49:
50: static void markPointerAsGlobal(Value *Ptr) {
51:   markPointerAsAS(Ptr, ADDRESS_SPACE_GLOBAL);
52: }
53:
54: static void handleIntToPtr(Value &V) {
55:   if (!all_of(V.users(), [](User *U) { return isa<IntToPtrInst>(U); }))
56:     return;
57:
58:   SmallVector<User *, 16> UsersToUpdate(V.users());
59:   for (User *U : UsersToUpdate)
60:     markPointerAsGlobal(U);
61: }
62:
63: static bool markKernelPtrsGlobal(Function &F) {
64:   if (!isKernelFunction(F))
65:     return false;
66:
67:   // Copying of byval aggregates + SROA may result in pointers being loaded as
68:   // integers, followed by inttoptr. We mark those as global too, but only if
69:   // the loaded integer is used exclusively for conversion to a pointer.
70:   for (auto &I : instructions(F)) {
71:     auto *LI = dyn_cast<LoadInst>(&I);
72:     if (!LI)
73:       continue;
74:
75:     if (LI->getType()->isPointerTy() || LI->getType()->isIntegerTy()) {
76:       Value *UO = getUnderlyingObject(LI->getPointerOperand());
77:       if (auto *Arg = dyn_cast<Argument>(UO)) {
78:         if (Arg->hasByValAttr()) {
79:           if (LI->getType()->isPointerTy())
80:             markPointerAsGlobal(LI);
```
- EN: This range implements operational logic in helpers such as PointerType::get, getName, replaceAllUsesWith, setOperand, translating backend policy into executable code.
- CN: 这一段实现了 PointerType::get、getName、replaceAllUsesWith、setOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:           else
 82:             handleIntToPtr(*LI);
 83:         }
 84:       }
 85:     }
 86:   }
 87:
 88:   for (Argument &Arg : F.args())
 89:     if (Arg.getType()->isIntegerTy())
 90:       handleIntToPtr(Arg);
 91:
 92:   return true;
 93: }
 94:
 95: namespace {
 96:
 97: class NVPTXMarkKernelPtrsGlobalLegacyPass : public FunctionPass {
 98: public:
 99:   static char ID;
100:   NVPTXMarkKernelPtrsGlobalLegacyPass() : FunctionPass(ID) {}
101:   bool runOnFunction(Function &F) override;
102: };
103:
104: } // namespace
105:
106: INITIALIZE_PASS(NVPTXMarkKernelPtrsGlobalLegacyPass,
107:                 "nvptx-mark-kernel-ptrs-global",
108:                 "NVPTX Mark Kernel Pointers Global", false, false)
109:
110: bool NVPTXMarkKernelPtrsGlobalLegacyPass::runOnFunction(Function &F) {
111:   return markKernelPtrsGlobal(F);
112: }
113:
114: char NVPTXMarkKernelPtrsGlobalLegacyPass::ID = 0;
115:
116: FunctionPass *llvm::createNVPTXMarkKernelPtrsGlobalPass() {
117:   return new NVPTXMarkKernelPtrsGlobalLegacyPass();
118: }
119:
120: PreservedAnalyses
```
- EN: This range defines or declares important types such as handleIntToPtr, NVPTXMarkKernelPtrsGlobalLegacyPass, runOnFunction, NVPTXMarkKernelPtrsGlobalLegacyPass::runOnFunction, shaping the data model used by NVPTXMarkKernelPtrsGlobal.cpp.
- CN: 这一段定义或声明了 handleIntToPtr、NVPTXMarkKernelPtrsGlobalLegacyPass、runOnFunction、NVPTXMarkKernelPtrsGlobalLegacyPass::runOnFunction 等关键类型，构成 NVPTXMarkKernelPtrsGlobal.cpp 使用的数据模型。

### Lines 121-124
```cpp
121: NVPTXMarkKernelPtrsGlobalPass::run(Function &F, FunctionAnalysisManager &) {
122:   return markKernelPtrsGlobal(F) ? PreservedAnalyses::none()
123:                                  : PreservedAnalyses::all();
124: }
```
- EN: This range implements operational logic in helpers such as NVPTXMarkKernelPtrsGlobalPass::run, markKernelPtrsGlobal, PreservedAnalyses::all, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXMarkKernelPtrsGlobalPass::run、markKernelPtrsGlobal、PreservedAnalyses::all 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include markPointerAsAS, getParent, getIterator, PointerType::get, getName, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 markPointerAsAS, getParent, getIterator, PointerType::get, getName，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
  - `NVVMProperties.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/ValueTracking.h`
  - `llvm/IR/InstIterator.h`
  - `llvm/IR/Instructions.h`
  - `llvm/InitializePasses.h`
  - `llvm/Pass.h`
  - `llvm/Support/NVPTXAddrSpace.h`
