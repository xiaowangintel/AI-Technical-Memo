# DXILForwardHandleAccesses.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILForwardHandleAccesses.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILForwardHandleAccesses support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- DXILForwardHandleAccesses.cpp - Cleanup Handles --------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: #include "DXILForwardHandleAccesses.h"
10: #include "DXILShaderFlags.h"
11: #include "DirectX.h"
12: #include "llvm/ADT/STLExtras.h"
13: #include "llvm/Analysis/DXILResource.h"
14: #include "llvm/Analysis/Loads.h"
15: #include "llvm/IR/DiagnosticInfo.h"
16: #include "llvm/IR/Dominators.h"
17: #include "llvm/IR/InstrTypes.h"
18: #include "llvm/IR/Instructions.h"
19: #include "llvm/IR/IntrinsicInst.h"
20: #include "llvm/IR/Intrinsics.h"
21: #include "llvm/IR/IntrinsicsDirectX.h"
22: #include "llvm/IR/Module.h"
23: #include "llvm/InitializePasses.h"
24: #include "llvm/Pass.h"
25: #include "llvm/Transforms/Utils/Local.h"
26:
27: #define DEBUG_TYPE "dxil-forward-handle-accesses"
28:
29: using namespace llvm;
30:
31: static void diagnoseAmbiguousHandle(IntrinsicInst *NewII,
32:                                     IntrinsicInst *PrevII) {
33:   Function *F = NewII->getFunction();
34:   LLVMContext &Context = F->getParent()->getContext();
35:   Context.diagnose(DiagnosticInfoGeneric(
36:       Twine("Handle at \"") + NewII->getName() + "\" overwrites handle at \"" +
37:       PrevII->getName() + "\""));
38: }
39:
40: static void diagnoseHandleNotFound(LoadInst *LI) {
```
- EN: This range implements operational logic in helpers such as getFunction, getParent, getName, diagnoseHandleNotFound, translating backend policy into executable code.
- CN: 这一段实现了 getFunction、getParent、getName、diagnoseHandleNotFound 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:   Function *F = LI->getFunction();
42:   LLVMContext &Context = F->getParent()->getContext();
43:   Context.diagnose(DiagnosticInfoGeneric(
44:       LI, Twine("Load of \"") + LI->getPointerOperand()->getName() +
45:               "\" is not a global resource handle"));
46: }
47:
48: static void diagnoseUndominatedLoad(LoadInst *LI, IntrinsicInst *Handle) {
49:   Function *F = LI->getFunction();
50:   LLVMContext &Context = F->getParent()->getContext();
51:   Context.diagnose(DiagnosticInfoGeneric(
52:       LI, Twine("Load at \"") + LI->getName() +
53:               "\" is not dominated by handle creation at \"" +
54:               Handle->getName() + "\""));
55: }
56:
57: static void
58: processHandle(IntrinsicInst *II,
59:               DenseMap<GlobalVariable *, IntrinsicInst *> &HandleMap) {
60:   for (User *U : II->users())
61:     if (auto *SI = dyn_cast<StoreInst>(U))
62:       if (auto *GV = dyn_cast<GlobalVariable>(SI->getPointerOperand())) {
63:         auto Entry = HandleMap.try_emplace(GV, II);
64:         if (Entry.second)
65:           LLVM_DEBUG(dbgs() << "Added " << GV->getName() << " to handle map\n");
66:         else
67:           diagnoseAmbiguousHandle(II, Entry.first->second);
68:       }
69: }
70:
71: static bool forwardHandleAccesses(Function &F, DominatorTree &DT) {
72:   bool Changed = false;
73:
74:   DenseMap<GlobalVariable *, IntrinsicInst *> HandleMap;
75:   SmallVector<LoadInst *> LoadsToProcess;
76:   DenseMap<AllocaInst *, SmallVector<IntrinsicInst *>> LifeTimeIntrinsicMap;
77:   for (BasicBlock &BB : F)
78:     for (Instruction &Inst : BB)
79:       if (auto *II = dyn_cast<IntrinsicInst>(&Inst)) {
80:         switch (II->getIntrinsicID()) {
```
- EN: This range implements operational logic in helpers such as getFunction, getParent, diagnoseUndominatedLoad, getName, translating backend policy into executable code.
- CN: 这一段实现了 getFunction、getParent、diagnoseUndominatedLoad、getName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:         case Intrinsic::dx_resource_handlefrombinding:
 82:         case Intrinsic::dx_resource_handlefromimplicitbinding:
 83:           processHandle(II, HandleMap);
 84:           break;
 85:         case Intrinsic::lifetime_start:
 86:         case Intrinsic::lifetime_end:
 87:           if (II->arg_size() >= 1) {
 88:             Value *Ptr = II->getArgOperand(0);
 89:             if (auto *Alloca = dyn_cast<AllocaInst>(Ptr))
 90:               LifeTimeIntrinsicMap[Alloca].push_back(II);
 91:           }
 92:           break;
 93:         default:
 94:           continue;
 95:         }
 96:       } else if (auto *LI = dyn_cast<LoadInst>(&Inst))
 97:         if (isa<dxil::AnyResourceExtType>(LI->getType()))
 98:           LoadsToProcess.push_back(LI);
 99:
100:   for (LoadInst *LI : LoadsToProcess) {
101:     Value *V = LI->getPointerOperand();
102:     auto *GV = dyn_cast<GlobalVariable>(V);
103:
104:     // If we didn't find the global, we may need to walk through a level of
105:     // indirection. This generally happens at -O0.
106:     if (!GV) {
107:       if (auto *NestedLI = dyn_cast<LoadInst>(V)) {
108:         BasicBlock::iterator BBI(NestedLI);
109:         Value *Loaded = FindAvailableLoadedValue(
110:             NestedLI, NestedLI->getParent(), BBI, 0, nullptr, nullptr);
111:         GV = dyn_cast_or_null<GlobalVariable>(Loaded);
112:       } else if (auto *NestedAlloca = dyn_cast<AllocaInst>(V)) {
113:
114:         if (auto It = LifeTimeIntrinsicMap.find(NestedAlloca);
115:             It != LifeTimeIntrinsicMap.end()) {
116:           llvm::for_each(It->second,
117:                          [](IntrinsicInst *II) { II->eraseFromParent(); });
118:           LifeTimeIntrinsicMap.erase(It);
119:         }
120:
```
- EN: This range implements operational logic in helpers such as processHandle, getArgOperand, push_back, getPointerOperand, translating backend policy into executable code.
- CN: 这一段实现了 processHandle、getArgOperand、push_back、getPointerOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:         for (auto *User : NestedAlloca->users()) {
122:           auto *Store = dyn_cast<StoreInst>(User);
123:           if (!Store)
124:             continue;
125:
126:           Value *StoredVal = Store->getValueOperand();
127:           if (!StoredVal)
128:             continue;
129:
130:           // Try direct global match
131:           GV = dyn_cast<GlobalVariable>(StoredVal);
132:           if (GV)
133:             break;
134:
135:           // If it's a load, check its source
136:           if (auto *Load = dyn_cast<LoadInst>(StoredVal)) {
137:             GV = dyn_cast<GlobalVariable>(Load->getPointerOperand());
138:             if (GV)
139:               break;
140:
141:             // If loading from an unmodified stack copy of the global, reuse the
142:             // global's value. Note: we are just repeating what we are doing for
143:             // the load case for the alloca store pattern.
144:             BasicBlock::iterator BBI(Load);
145:             Value *Loaded = FindAvailableLoadedValue(Load, Load->getParent(),
146:                                                      BBI, 0, nullptr, nullptr);
147:             GV = dyn_cast<GlobalVariable>(Loaded);
148:             if (GV)
149:               break;
150:           }
151:         }
152:       }
153:     }
154:
155:     auto It = HandleMap.find(GV);
156:     if (It == HandleMap.end()) {
157:       diagnoseHandleNotFound(LI);
158:       continue;
159:     }
160:     Changed = true;
```
- EN: This range implements operational logic in helpers such as getValueOperand, getPointerOperand, BBI, find, translating backend policy into executable code.
- CN: 这一段实现了 getValueOperand、getPointerOperand、BBI、find 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:
162:     if (!DT.dominates(It->second, LI)) {
163:       diagnoseUndominatedLoad(LI, It->second);
164:       continue;
165:     }
166:
167:     LLVM_DEBUG(dbgs() << "Replacing uses of " << GV->getName() << " at "
168:                       << LI->getName() << " with " << It->second->getName()
169:                       << "\n");
170:     LI->replaceAllUsesWith(It->second);
171:     LI->eraseFromParent();
172:   }
173:
174:   return Changed;
175: }
176:
177: PreservedAnalyses DXILForwardHandleAccesses::run(Function &F,
178:                                                  FunctionAnalysisManager &AM) {
179:   PreservedAnalyses PA;
180:
181:   DominatorTree *DT = &AM.getResult<DominatorTreeAnalysis>(F);
182:   bool Changed = forwardHandleAccesses(F, *DT);
183:
184:   if (!Changed)
185:     return PreservedAnalyses::all();
186:   return PA;
187: }
188:
189: namespace {
190: class DXILForwardHandleAccessesLegacy : public FunctionPass {
191: public:
192:   bool runOnFunction(Function &F) override {
193:     DominatorTree *DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();
194:     return forwardHandleAccesses(F, *DT);
195:   }
196:   StringRef getPassName() const override {
197:     return "DXIL Forward Handle Accesses";
198:   }
199:
200:   void getAnalysisUsage(AnalysisUsage &AU) const override {
```
- EN: This range defines or declares important types such as diagnoseUndominatedLoad, getName, replaceAllUsesWith, eraseFromParent, shaping the data model used by DXILForwardHandleAccesses.cpp.
- CN: 这一段定义或声明了 diagnoseUndominatedLoad、getName、replaceAllUsesWith、eraseFromParent 等关键类型，构成 DXILForwardHandleAccesses.cpp 使用的数据模型。

### Lines 201-219
```cpp
201:     AU.addRequired<DominatorTreeWrapperPass>();
202:   }
203:
204:   DXILForwardHandleAccessesLegacy() : FunctionPass(ID) {}
205:
206:   static char ID; // Pass identification.
207: };
208: char DXILForwardHandleAccessesLegacy::ID = 0;
209: } // end anonymous namespace
210:
211: INITIALIZE_PASS_BEGIN(DXILForwardHandleAccessesLegacy, DEBUG_TYPE,
212:                       "DXIL Forward Handle Accesses", false, false)
213: INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
214: INITIALIZE_PASS_END(DXILForwardHandleAccessesLegacy, DEBUG_TYPE,
215:                     "DXIL Forward Handle Accesses", false, false)
216:
217: FunctionPass *llvm::createDXILForwardHandleAccessesLegacyPass() {
218:   return new DXILForwardHandleAccessesLegacy();
219: }
```
- EN: This range implements operational logic in helpers such as DXILForwardHandleAccessesLegacy, INITIALIZE_PASS_DEPENDENCY, llvm::createDXILForwardHandleAccessesLegacyPass, translating backend policy into executable code.
- CN: 这一段实现了 DXILForwardHandleAccessesLegacy、INITIALIZE_PASS_DEPENDENCY、llvm::createDXILForwardHandleAccessesLegacyPass 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include getFunction, getParent, getName, diagnoseHandleNotFound, diagnoseUndominatedLoad, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getFunction, getParent, getName, diagnoseHandleNotFound, diagnoseUndominatedLoad，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILForwardHandleAccesses.h`
  - `DXILShaderFlags.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/STLExtras.h`
  - `llvm/Analysis/DXILResource.h`
  - `llvm/Analysis/Loads.h`
  - `llvm/IR/DiagnosticInfo.h`
  - `llvm/IR/Dominators.h`
  - `llvm/IR/InstrTypes.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsDirectX.h`
  - `llvm/IR/Module.h`
  - `llvm/InitializePasses.h`
  - `llvm/Pass.h`
  - `llvm/Transforms/Utils/Local.h`
