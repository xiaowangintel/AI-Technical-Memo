# SPIRVCBufferAccess.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVCBufferAccess.cpp`
- Repository: `llvm-project`
- Purpose (EN): This pass replaces all accesses to constant buffer global variables with accesses to the proper SPIR-V resource.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- SPIRVCBufferAccess.cpp - Translate CBuffer Loads ---------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This pass replaces all accesses to constant buffer global variables with
10: // accesses to the proper SPIR-V resource.
11: //
12: // The pass operates as follows:
13: // 1. It finds all constant buffers by looking for the `!hlsl.cbs` metadata.
14: // 2. For each cbuffer, it finds the global variable holding the resource handle
15: //    and the global variables for each of the cbuffer's members.
16: // 3. For each member variable, it creates a call to the
17: //    `llvm.spv.resource.getpointer` intrinsic. This intrinsic takes the
18: //    resource handle and the member's index within the cbuffer as arguments.
19: //    The result is a pointer to that member within the SPIR-V resource.
20: // 4. It then replaces all uses of the original member global variable with the
21: //    pointer returned by the `getpointer` intrinsic. This effectively retargets
22: //    all loads and GEPs to the new resource pointer.
23: // 5. Finally, it cleans up by deleting the original global variables and the
24: //    `!hlsl.cbs` metadata.
25: //
26: // This approach allows subsequent passes, like SPIRVEmitIntrinsics, to
27: // correctly handle GEPs that operate on the result of the `getpointer` call,
28: // folding them into a single OpAccessChain instruction.
29: //
30: //===----------------------------------------------------------------------===//
31:
32: #include "SPIRVCBufferAccess.h"
33: #include "SPIRV.h"
34: #include "llvm/Frontend/HLSL/CBuffer.h"
35: #include "llvm/IR/IRBuilder.h"
36: #include "llvm/IR/IntrinsicsSPIRV.h"
37: #include "llvm/IR/Module.h"
38: #include "llvm/IR/ReplaceConstant.h"
39:
40: #define DEBUG_TYPE "spirv-cbuffer-access"
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 41-80
```cpp
41: using namespace llvm;
42:
43: // Finds the single instruction that defines the resource handle. This is
44: // typically a call to `llvm.spv.resource.handlefrombinding`.
45: static Instruction *findHandleDef(GlobalVariable *HandleVar) {
46:   for (User *U : HandleVar->users()) {
47:     if (auto *SI = dyn_cast<StoreInst>(U)) {
48:       if (auto *I = dyn_cast<Instruction>(SI->getValueOperand())) {
49:         return I;
50:       }
51:     }
52:   }
53:   return nullptr;
54: }
55:
56: static bool replaceCBufferAccesses(Module &M) {
57:   std::optional<hlsl::CBufferMetadata> CBufMD =
58:       hlsl::CBufferMetadata::get(M, [](Type *Ty) {
59:         if (auto *TET = dyn_cast<TargetExtType>(Ty))
60:           return TET->getName() == "spirv.Padding";
61:         return false;
62:       });
63:   if (!CBufMD)
64:     return false;
65:
66:   SmallVector<Constant *> CBufferGlobals;
67:   for (const hlsl::CBufferMapping &Mapping : *CBufMD)
68:     for (const hlsl::CBufferMember &Member : Mapping.Members)
69:       CBufferGlobals.push_back(Member.GV);
70:   convertUsersOfConstantsToInstructions(CBufferGlobals);
71:
72:   for (const hlsl::CBufferMapping &Mapping : *CBufMD) {
73:     Instruction *HandleDef = findHandleDef(Mapping.Handle);
74:     if (!HandleDef) {
75:       report_fatal_error("Could not find handle definition for cbuffer: " +
76:                          Mapping.Handle->getName());
77:     }
78:
79:     // The handle definition should dominate all uses of the cbuffer members.
80:     // We'll insert our getpointer calls right after it.
```
- EN: This range implements operational logic in helpers such as findHandleDef, replaceCBufferAccesses, hlsl::CBufferMetadata::get, push_back, translating backend policy into executable code.
- CN: 这一段实现了 findHandleDef、replaceCBufferAccesses、hlsl::CBufferMetadata::get、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:     IRBuilder<> Builder(HandleDef->getNextNode());
 82:     auto *HandleTy = cast<TargetExtType>(Mapping.Handle->getValueType());
 83:     auto *LayoutTy = cast<StructType>(HandleTy->getTypeParameter(0));
 84:     const StructLayout *SL = M.getDataLayout().getStructLayout(LayoutTy);
 85:
 86:     for (const hlsl::CBufferMember &Member : Mapping.Members) {
 87:       GlobalVariable *MemberGV = Member.GV;
 88:       if (MemberGV->use_empty()) {
 89:         continue;
 90:       }
 91:
 92:       uint32_t IndexInStruct = SL->getElementContainingOffset(Member.Offset);
 93:
 94:       // Create the getpointer intrinsic call.
 95:       Value *IndexVal = Builder.getInt32(IndexInStruct);
 96:       Type *PtrType = MemberGV->getType();
 97:       Value *GetPointerCall = Builder.CreateIntrinsic(
 98:           PtrType, Intrinsic::spv_resource_getpointer, {HandleDef, IndexVal});
 99:
100:       MemberGV->replaceAllUsesWith(GetPointerCall);
101:     }
102:   }
103:
104:   // Now that all uses are replaced, clean up the globals and metadata.
105:   for (const hlsl::CBufferMapping &Mapping : *CBufMD) {
106:     for (const auto &Member : Mapping.Members) {
107:       Member.GV->eraseFromParent();
108:     }
109:     // Erase the stores to the handle variable before erasing the handle itself.
110:     SmallVector<Instruction *, 4> HandleStores;
111:     for (User *U : Mapping.Handle->users()) {
112:       if (auto *SI = dyn_cast<StoreInst>(U)) {
113:         HandleStores.push_back(SI);
114:       }
115:     }
116:     for (Instruction *I : HandleStores) {
117:       I->eraseFromParent();
118:     }
119:     Mapping.Handle->eraseFromParent();
120:   }
```
- EN: This range implements operational logic in helpers such as Builder, getValueType, getTypeParameter, getDataLayout, translating backend policy into executable code.
- CN: 这一段实现了 Builder、getValueType、getTypeParameter、getDataLayout 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-151
```cpp
121:
122:   CBufMD->eraseFromModule();
123:   return true;
124: }
125:
126: PreservedAnalyses SPIRVCBufferAccess::run(Module &M,
127:                                           ModuleAnalysisManager &AM) {
128:   if (replaceCBufferAccesses(M)) {
129:     return PreservedAnalyses::none();
130:   }
131:   return PreservedAnalyses::all();
132: }
133:
134: namespace {
135: class SPIRVCBufferAccessLegacy : public ModulePass {
136: public:
137:   bool runOnModule(Module &M) override { return replaceCBufferAccesses(M); }
138:   StringRef getPassName() const override { return "SPIRV CBuffer Access"; }
139:   SPIRVCBufferAccessLegacy() : ModulePass(ID) {}
140:
141:   static char ID; // Pass identification.
142: };
143: char SPIRVCBufferAccessLegacy::ID = 0;
144: } // end anonymous namespace
145:
146: INITIALIZE_PASS(SPIRVCBufferAccessLegacy, DEBUG_TYPE, "SPIRV CBuffer Access",
147:                 false, false)
148:
149: ModulePass *llvm::createSPIRVCBufferAccessLegacyPass() {
150:   return new SPIRVCBufferAccessLegacy();
151: }
```
- EN: This range defines or declares important types such as eraseFromModule, PreservedAnalyses::none, PreservedAnalyses::all, SPIRVCBufferAccessLegacy, shaping the data model used by SPIRVCBufferAccess.cpp.
- CN: 这一段定义或声明了 eraseFromModule、PreservedAnalyses::none、PreservedAnalyses::all、SPIRVCBufferAccessLegacy 等关键类型，构成 SPIRVCBufferAccess.cpp 使用的数据模型。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include findHandleDef, replaceCBufferAccesses, hlsl::CBufferMetadata::get, push_back, convertUsersOfConstantsToInstructions, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 findHandleDef, replaceCBufferAccesses, hlsl::CBufferMetadata::get, push_back, convertUsersOfConstantsToInstructions，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVCBufferAccess.h`
  - `SPIRV.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Frontend/HLSL/CBuffer.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/IR/Module.h`
  - `llvm/IR/ReplaceConstant.h`
