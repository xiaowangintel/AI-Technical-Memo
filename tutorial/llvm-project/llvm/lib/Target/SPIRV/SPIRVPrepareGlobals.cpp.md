# SPIRVPrepareGlobals.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVPrepareGlobals.cpp`
- Repository: `llvm-project`
- Purpose (EN): The pass: transforms IR globals that cannot be trivially mapped to SPIRV into something that is trival to lower; for AMDGCN flavoured SPIRV, it assigns unique IDs to the specialisation constants associated with feature predicates, which were inserted by the FE when expanding calls to __builtin_amdgcn_processor_is or __builtin_amdgcn_is_invocable.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVPrepareGlobals.cpp - Prepare IR SPIRV globals ------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // The pass:
10: //   - transforms IR globals that cannot be trivially mapped to SPIRV into
11: //     something that is trival to lower;
12: //   - for AMDGCN flavoured SPIRV, it assigns unique IDs to the specialisation
13: //     constants associated with feature predicates, which were inserted by the
14: //     FE when expanding calls to __builtin_amdgcn_processor_is or
15: //     __builtin_amdgcn_is_invocable
16: //
17: //===----------------------------------------------------------------------===//
18:
19: #include "SPIRVPrepareGlobals.h"
20: #include "SPIRV.h"
21: #include "SPIRVUtils.h"
22:
23: #include "llvm/ADT/STLExtras.h"
24: #include "llvm/ADT/StringExtras.h"
25: #include "llvm/ADT/StringMap.h"
26: #include "llvm/IR/IntrinsicsSPIRV.h"
27: #include "llvm/IR/Module.h"
28: #include "llvm/Support/Debug.h"
29:
30: #include <climits>
31: #include <string>
32:
33: #define DEBUG_TYPE "spirv-prepare-globals"
34:
35: using namespace llvm;
36:
37: namespace {
38:
39: struct SPIRVPrepareGlobalsImpl {
40:   bool runOnModule(Module &M);
```
- EN: This range defines or declares important types such as SPIRVPrepareGlobalsImpl, runOnModule, shaping the data model used by SPIRVPrepareGlobals.cpp.
- CN: 这一段定义或声明了 SPIRVPrepareGlobalsImpl、runOnModule 等关键类型，构成 SPIRVPrepareGlobals.cpp 使用的数据模型。

### Lines 41-80
```cpp
41: };
42:
43: struct SPIRVPrepareGlobalsLegacy : public ModulePass {
44:   static char ID;
45:   SPIRVPrepareGlobalsLegacy() : ModulePass(ID) {}
46:
47:   StringRef getPassName() const override {
48:     return "SPIRV prepare global variables";
49:   }
50:
51:   bool runOnModule(Module &M) override {
52:     return SPIRVPrepareGlobalsImpl().runOnModule(M);
53:   }
54: };
55:
56: // The backend does not support GlobalAlias. Replace aliases with their aliasees
57: // when possible and remove them from the module.
58: bool tryReplaceAliasWithAliasee(GlobalAlias &GA) {
59:   // According to the lang ref, aliases cannot be replaced if either the alias
60:   // or the aliasee are interposable. We only replace in the case that both
61:   // are not interposable.
62:   if (GA.isInterposable()) {
63:     LLVM_DEBUG(dbgs() << "Skipping interposable alias: " << GA.getName()
64:                       << "\n");
65:     return false;
66:   }
67:
68:   auto *AO = dyn_cast<GlobalObject>(GA.getAliasee());
69:   if (!AO) {
70:     LLVM_DEBUG(dbgs() << "Skipping alias whose aliasee is not a GlobalObject: "
71:                       << GA.getName() << "\n");
72:     return false;
73:   }
74:
75:   if (AO->isInterposable()) {
76:     LLVM_DEBUG(dbgs() << "Skipping interposable aliasee: " << AO->getName()
77:                       << "\n");
78:     return false;
79:   }
80:
```
- EN: This range defines or declares important types such as SPIRVPrepareGlobalsLegacy, getPassName, runOnModule, SPIRVPrepareGlobalsImpl, shaping the data model used by SPIRVPrepareGlobals.cpp.
- CN: 这一段定义或声明了 SPIRVPrepareGlobalsLegacy、getPassName、runOnModule、SPIRVPrepareGlobalsImpl 等关键类型，构成 SPIRVPrepareGlobals.cpp 使用的数据模型。

### Lines 81-120
```cpp
 81:   LLVM_DEBUG(dbgs() << "Replacing alias " << GA.getName()
 82:                     << " with aliasee: " << AO->getName() << "\n");
 83:
 84:   GA.replaceAllUsesWith(AO);
 85:   if (GA.isDiscardableIfUnused()) {
 86:     GA.eraseFromParent();
 87:   }
 88:
 89:   return true;
 90: }
 91:
 92: bool tryAssignPredicateSpecConstIDs(Module &M, Function *F) {
 93:   StringMap<unsigned> IDs;
 94:   for (auto &&U : F->users()) {
 95:     auto *CI = dyn_cast<CallInst>(U);
 96:     if (!CI)
 97:       continue;
 98:
 99:     auto *SpecID = dyn_cast<ConstantInt>(CI->getArgOperand(0));
100:     if (!SpecID)
101:       continue;
102:
103:     unsigned ID = SpecID->getZExtValue();
104:     if (ID != UINT32_MAX)
105:       continue;
106:
107:     // Replace placeholder Specialisation Constant IDs with unique IDs
108:     // associated with the predicate being evaluated, which is encoded via
109:     // spv_assign_name.
110:     auto *MD =
111:         cast<MDNode>(cast<MetadataAsValue>(CI->getOperand(2))->getMetadata());
112:     auto *P = cast<MDString>(MD->getOperand(0));
113:
114:     ID = IDs.try_emplace(P->getString(), IDs.size()).first->second;
115:     CI->setArgOperand(0, ConstantInt::get(CI->getArgOperand(0)->getType(), ID));
116:   }
117:
118:   if (IDs.empty())
119:     return false;
120:
```
- EN: This range implements operational logic in helpers such as LLVM_DEBUG, getName, replaceAllUsesWith, eraseFromParent, translating backend policy into executable code.
- CN: 这一段实现了 LLVM_DEBUG、getName、replaceAllUsesWith、eraseFromParent 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:   // Store the predicate -> ID mapping as a fixed format string
122:   // (predicate ID\0...), for later use during SPIR-V consumption.
123:   std::string Tmp;
124:   for (auto &&[Predicate, SpecID] : IDs)
125:     Tmp.append(Predicate).append(" ").append(utostr(SpecID)).push_back('\0');
126:
127:   Constant *PredSpecIDStr =
128:       ConstantDataArray::getString(M.getContext(), Tmp, false);
129:
130:   new GlobalVariable(M, PredSpecIDStr->getType(), true,
131:                      GlobalVariable::LinkageTypes::ExternalLinkage,
132:                      PredSpecIDStr, "llvm.amdgcn.feature.predicate.ids");
133:
134:   return true;
135: }
136:
137: bool SPIRVPrepareGlobalsImpl::runOnModule(Module &M) {
138:   bool Changed = false;
139:
140:   for (GlobalAlias &GA : make_early_inc_range(M.aliases())) {
141:     Changed |= tryReplaceAliasWithAliasee(GA);
142:   }
143:
144:   if (M.getTargetTriple().getVendor() != Triple::AMD)
145:     return Changed;
146:
147:   // TODO: Currently, for AMDGCN flavoured SPIR-V, the symbol can only be
148:   //       inserted via feature predicate use, but in the future this will need
149:   //       revisiting if we start making more liberal use of the intrinsic.
150:   if (Function *F = Intrinsic::getDeclarationIfExists(
151:           &M, Intrinsic::spv_named_boolean_spec_constant))
152:     Changed |= tryAssignPredicateSpecConstIDs(M, F);
153:
154:   return Changed;
155: }
156: char SPIRVPrepareGlobalsLegacy::ID = 0;
157:
158: } // namespace
159:
160: INITIALIZE_PASS(SPIRVPrepareGlobalsLegacy, "spirv-prepare-globals",
```
- EN: This range implements operational logic in helpers such as append, ConstantDataArray::getString, SPIRVPrepareGlobalsImpl::runOnModule, tryReplaceAliasWithAliasee, translating backend policy into executable code.
- CN: 这一段实现了 append、ConstantDataArray::getString、SPIRVPrepareGlobalsImpl::runOnModule、tryReplaceAliasWithAliasee 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-173
```cpp
161:                 "SPIRV prepare global variables", false, false)
162:
163: PreservedAnalyses SPIRVPrepareGlobals::run(Module &M,
164:                                            ModuleAnalysisManager &AM) {
165:   return SPIRVPrepareGlobalsImpl().runOnModule(M) ? PreservedAnalyses::none()
166:                                                   : PreservedAnalyses::all();
167: }
168:
169: namespace llvm {
170: ModulePass *createSPIRVPrepareGlobalsPass() {
171:   return new SPIRVPrepareGlobalsLegacy();
172: }
173: } // namespace llvm
```
- EN: This range implements operational logic in helpers such as SPIRVPrepareGlobalsImpl, PreservedAnalyses::all, createSPIRVPrepareGlobalsPass, SPIRVPrepareGlobalsLegacy, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVPrepareGlobalsImpl、PreservedAnalyses::all、createSPIRVPrepareGlobalsPass、SPIRVPrepareGlobalsLegacy 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVPrepareGlobalsImpl, runOnModule, SPIRVPrepareGlobalsLegacy, getPassName, tryReplaceAliasWithAliasee, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVPrepareGlobalsImpl, runOnModule, SPIRVPrepareGlobalsLegacy, getPassName, tryReplaceAliasWithAliasee，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVPrepareGlobals.h`
  - `SPIRV.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/STLExtras.h`
  - `llvm/ADT/StringExtras.h`
  - `llvm/ADT/StringMap.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/IR/Module.h`
  - `llvm/Support/Debug.h`
- System/standard headers / 系统或标准头文件:
  - `climits`
  - `string`
