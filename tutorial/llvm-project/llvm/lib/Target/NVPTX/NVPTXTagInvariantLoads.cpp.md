# NVPTXTagInvariantLoads.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXTagInvariantLoads.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements invaraint load tagging.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===------ NVPTXTagInvariantLoads.cpp - Tag invariant loads --------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file implements invaraint load tagging. It traverses load instructions
10: // in a function, and determines if each load can be tagged as invariant.
11: //
12: // We currently infer invariance for loads from
13: //  - constant global variables, and
14: //  - kernel function pointer params that are noalias (i.e. __restrict) and
15: //    never written to.
16: //
17: // TODO: Perform a more powerful invariance analysis (ideally IPO).
18: //
19: //===----------------------------------------------------------------------===//
20:
21: #include "NVPTX.h"
22: #include "NVVMProperties.h"
23: #include "llvm/Analysis/ValueTracking.h"
24: #include "llvm/IR/GlobalVariable.h"
25: #include "llvm/IR/InstIterator.h"
26: #include "llvm/IR/Instructions.h"
27: #include "llvm/IR/IntrinsicInst.h"
28: #include "llvm/IR/Metadata.h"
29: #include "llvm/Support/NVPTXAddrSpace.h"
30:
31: using namespace llvm;
32:
33: static bool isInvariantLoad(const Instruction *I, const Value *Ptr,
34:                             const bool IsKernelFn) {
35:   // Don't bother with non-global loads
36:   if (Ptr->getType()->getPointerAddressSpace() != NVPTXAS::ADDRESS_SPACE_GLOBAL)
37:     return false;
38:
39:   // If the load is already marked as invariant, we don't need to do anything
40:   if (I->getMetadata(LLVMContext::MD_invariant_load))
```
- EN: This range implements operational logic in helpers such as backend logic, translating backend policy into executable code.
- CN: 这一段实现了 后端逻辑 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:     return false;
42:
43:   // We use getUnderlyingObjects() here instead of getUnderlyingObject()
44:   // mainly because the former looks through phi nodes while the latter does
45:   // not. We need to look through phi nodes to handle pointer induction
46:   // variables.
47:   SmallVector<const Value *, 8> Objs;
48:   getUnderlyingObjects(Ptr, Objs);
49:
50:   return all_of(Objs, [&](const Value *V) {
51:     if (const auto *A = dyn_cast<const Argument>(V))
52:       return IsKernelFn && ((A->onlyReadsMemory() && A->hasNoAliasAttr()) ||
53:                             isParamGridConstant(*A));
54:     if (const auto *GV = dyn_cast<const GlobalVariable>(V))
55:       return GV->isConstant();
56:     return false;
57:   });
58: }
59:
60: static void markLoadsAsInvariant(Instruction *I) {
61:   I->setMetadata(LLVMContext::MD_invariant_load,
62:                  MDNode::get(I->getContext(), {}));
63: }
64:
65: static bool tagInvariantLoads(Function &F) {
66:   const bool IsKernelFn = isKernelFunction(F);
67:
68:   bool Changed = false;
69:   for (auto &I : instructions(F)) {
70:     if (auto *LI = dyn_cast<LoadInst>(&I))
71:       if (isInvariantLoad(LI, LI->getPointerOperand(), IsKernelFn)) {
72:         markLoadsAsInvariant(LI);
73:         Changed = true;
74:       }
75:     if (auto *II = dyn_cast<IntrinsicInst>(&I))
76:       if (II->getIntrinsicID() == Intrinsic::masked_load &&
77:           isInvariantLoad(II, II->getOperand(0), IsKernelFn)) {
78:         markLoadsAsInvariant(II);
79:         Changed = true;
80:       }
```
- EN: This range implements operational logic in helpers such as getUnderlyingObjects, all_of, isParamGridConstant, isConstant, translating backend policy into executable code.
- CN: 这一段实现了 getUnderlyingObjects、all_of、isParamGridConstant、isConstant 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-113
```cpp
 81:   }
 82:   return Changed;
 83: }
 84:
 85: namespace {
 86:
 87: struct NVPTXTagInvariantLoadLegacyPass : public FunctionPass {
 88:   static char ID;
 89:
 90:   NVPTXTagInvariantLoadLegacyPass() : FunctionPass(ID) {}
 91:   bool runOnFunction(Function &F) override;
 92: };
 93:
 94: } // namespace
 95:
 96: INITIALIZE_PASS(NVPTXTagInvariantLoadLegacyPass, "nvptx-tag-invariant-loads",
 97:                 "NVPTX Tag Invariant Loads", false, false)
 98:
 99: bool NVPTXTagInvariantLoadLegacyPass::runOnFunction(Function &F) {
100:   return tagInvariantLoads(F);
101: }
102:
103: char NVPTXTagInvariantLoadLegacyPass::ID = 0;
104:
105: FunctionPass *llvm::createNVPTXTagInvariantLoadsPass() {
106:   return new NVPTXTagInvariantLoadLegacyPass();
107: }
108:
109: PreservedAnalyses NVPTXTagInvariantLoadsPass::run(Function &F,
110:                                                   FunctionAnalysisManager &) {
111:   return tagInvariantLoads(F) ? PreservedAnalyses::none()
112:                               : PreservedAnalyses::all();
113: }
```
- EN: This range defines or declares important types such as NVPTXTagInvariantLoadLegacyPass, runOnFunction, NVPTXTagInvariantLoadLegacyPass::runOnFunction, tagInvariantLoads, shaping the data model used by NVPTXTagInvariantLoads.cpp.
- CN: 这一段定义或声明了 NVPTXTagInvariantLoadLegacyPass、runOnFunction、NVPTXTagInvariantLoadLegacyPass::runOnFunction、tagInvariantLoads 等关键类型，构成 NVPTXTagInvariantLoads.cpp 使用的数据模型。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include getUnderlyingObjects, all_of, isParamGridConstant, isConstant, markLoadsAsInvariant, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getUnderlyingObjects, all_of, isParamGridConstant, isConstant, markLoadsAsInvariant，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
  - `NVVMProperties.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/ValueTracking.h`
  - `llvm/IR/GlobalVariable.h`
  - `llvm/IR/InstIterator.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Metadata.h`
  - `llvm/Support/NVPTXAddrSpace.h`
