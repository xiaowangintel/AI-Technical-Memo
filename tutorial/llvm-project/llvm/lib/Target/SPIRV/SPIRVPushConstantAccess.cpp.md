# SPIRVPushConstantAccess.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVPushConstantAccess.cpp`
- Repository: `llvm-project`
- Purpose (EN): This pass changes the types of all the globals in the PushConstant address space into a target extension type, and makes all references to this global go though a custom SPIR-V intrinsic.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- SPIRVPushConstantAccess.cpp - Translate CBuffer Loads ----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This pass changes the types of all the globals in the PushConstant
10: // address space into a target extension type, and makes all references
11: // to this global go though a custom SPIR-V intrinsic.
12: //
13: // This allows the backend to properly lower the push constant struct type
14: // to a fully laid out type, and generate the proper OpAccessChain.
15: //
16: //===----------------------------------------------------------------------===//
17:
18: #include "SPIRVPushConstantAccess.h"
19: #include "SPIRV.h"
20: #include "SPIRVSubtarget.h"
21: #include "SPIRVTargetMachine.h"
22: #include "SPIRVUtils.h"
23: #include "llvm/Frontend/HLSL/CBuffer.h"
24: #include "llvm/IR/IRBuilder.h"
25: #include "llvm/IR/IntrinsicsSPIRV.h"
26: #include "llvm/IR/Module.h"
27: #include "llvm/IR/ReplaceConstant.h"
28:
29: #define DEBUG_TYPE "spirv-pushconstant-access"
30: using namespace llvm;
31:
32: static bool replacePushConstantAccesses(Module &M, SPIRVGlobalRegistry *GR) {
33:   bool Changed = false;
34:   for (GlobalVariable &GV : make_early_inc_range(M.globals())) {
35:     if (GV.getAddressSpace() !=
36:         storageClassToAddressSpace(SPIRV::StorageClass::PushConstant))
37:       continue;
38:
39:     convertUsersOfConstantsToInstructions(
40:         llvm::SmallVector<Constant *, 1>(1, &GV));
```
- EN: This range implements operational logic in helpers such as replacePushConstantAccesses, storageClassToAddressSpace, translating backend policy into executable code.
- CN: 这一段实现了 replacePushConstantAccesses、storageClassToAddressSpace 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:
42:     Type *PCType = llvm::TargetExtType::get(
43:         M.getContext(), "spirv.PushConstant", {GV.getValueType()});
44:     GlobalVariable *NewGV =
45:         new GlobalVariable(M, PCType, GV.isConstant(), GV.getLinkage(),
46:                            /* initializer= */ nullptr, GV.getName(),
47:                            /* InsertBefore= */ &GV, GV.getThreadLocalMode(),
48:                            GV.getAddressSpace(), GV.isExternallyInitialized());
49:     NewGV->setVisibility(GV.getVisibility());
50:
51:     for (User *U : make_early_inc_range(GV.users())) {
52:       Instruction *I = cast<Instruction>(U);
53:       IRBuilder<> Builder(I);
54:       Value *GetPointerCall = Builder.CreateIntrinsic(
55:           NewGV->getType(), Intrinsic::spv_pushconstant_getpointer, {NewGV});
56:       GR->buildAssignPtr(Builder, GV.getValueType(), GetPointerCall);
57:
58:       I->replaceUsesOfWith(&GV, GetPointerCall);
59:     }
60:
61:     GV.eraseFromParent();
62:     Changed = true;
63:   }
64:
65:   return Changed;
66: }
67:
68: PreservedAnalyses SPIRVPushConstantAccess::run(Module &M,
69:                                                ModuleAnalysisManager &AM) {
70:   const SPIRVSubtarget *ST = TM.getSubtargetImpl();
71:   SPIRVGlobalRegistry *GR = ST->getSPIRVGlobalRegistry();
72:   return replacePushConstantAccesses(M, GR) ? PreservedAnalyses::none()
73:                                             : PreservedAnalyses::all();
74: }
75:
76: namespace {
77: class SPIRVPushConstantAccessLegacy : public ModulePass {
78:   SPIRVTargetMachine *TM = nullptr;
79:
80: public:
```
- EN: This range defines or declares important types such as getContext, getAddressSpace, setVisibility, Builder, shaping the data model used by SPIRVPushConstantAccess.cpp.
- CN: 这一段定义或声明了 getContext、getAddressSpace、setVisibility、Builder 等关键类型，构成 SPIRVPushConstantAccess.cpp 使用的数据模型。

### Lines 81-103
```cpp
 81:   bool runOnModule(Module &M) override {
 82:     const SPIRVSubtarget *ST = TM->getSubtargetImpl();
 83:     SPIRVGlobalRegistry *GR = ST->getSPIRVGlobalRegistry();
 84:     return replacePushConstantAccesses(M, GR);
 85:   }
 86:   StringRef getPassName() const override {
 87:     return "SPIRV push constant Access";
 88:   }
 89:   SPIRVPushConstantAccessLegacy(SPIRVTargetMachine *TM)
 90:       : ModulePass(ID), TM(TM) {}
 91:
 92:   static char ID; // Pass identification.
 93: };
 94: char SPIRVPushConstantAccessLegacy::ID = 0;
 95: } // end anonymous namespace
 96:
 97: INITIALIZE_PASS(SPIRVPushConstantAccessLegacy, DEBUG_TYPE,
 98:                 "SPIRV push constant Access", false, false)
 99:
100: ModulePass *
101: llvm::createSPIRVPushConstantAccessLegacyPass(SPIRVTargetMachine *TM) {
102:   return new SPIRVPushConstantAccessLegacy(TM);
103: }
```
- EN: This range implements operational logic in helpers such as runOnModule, getSubtargetImpl, getSPIRVGlobalRegistry, replacePushConstantAccesses, translating backend policy into executable code.
- CN: 这一段实现了 runOnModule、getSubtargetImpl、getSPIRVGlobalRegistry、replacePushConstantAccesses 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include replacePushConstantAccesses, storageClassToAddressSpace, getContext, getAddressSpace, setVisibility, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 replacePushConstantAccesses, storageClassToAddressSpace, getContext, getAddressSpace, setVisibility，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVPushConstantAccess.h`
  - `SPIRV.h`
  - `SPIRVSubtarget.h`
  - `SPIRVTargetMachine.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Frontend/HLSL/CBuffer.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/IR/Module.h`
  - `llvm/IR/ReplaceConstant.h`
