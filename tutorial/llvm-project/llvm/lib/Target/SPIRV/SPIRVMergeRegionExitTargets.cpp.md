# SPIRVMergeRegionExitTargets.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVMergeRegionExitTargets.cpp`
- Repository: `llvm-project`
- Purpose (EN): Merge the multiple exit targets of a convergence region into a single block.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVMergeRegionExitTargets.cpp ----------------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // Merge the multiple exit targets of a convergence region into a single block.
10: // Each exit target will be assigned a constant value, and a phi node + switch
11: // will allow the new exit target to re-route to the correct basic block.
12: //
13: //===----------------------------------------------------------------------===//
14:
15: #include "SPIRVMergeRegionExitTargets.h"
16: #include "Analysis/SPIRVConvergenceRegionAnalysis.h"
17: #include "SPIRV.h"
18: #include "SPIRVSubtarget.h"
19: #include "SPIRVUtils.h"
20: #include "llvm/ADT/DenseMap.h"
21: #include "llvm/ADT/SmallPtrSet.h"
22: #include "llvm/Analysis/LoopInfo.h"
23: #include "llvm/IR/Dominators.h"
24: #include "llvm/IR/IRBuilder.h"
25: #include "llvm/IR/Intrinsics.h"
26: #include "llvm/InitializePasses.h"
27: #include "llvm/Transforms/Utils/Cloning.h"
28: #include "llvm/Transforms/Utils/LoopSimplify.h"
29: #include "llvm/Transforms/Utils/LowerMemIntrinsics.h"
30:
31: using namespace llvm;
32:
33: namespace {
34:
35: /// Create a value in BB set to the value associated with the branch the block
36: /// terminator will take.
37: static llvm::Value *
38: createExitVariable(BasicBlock *BB,
39:                    const DenseMap<BasicBlock *, ConstantInt *> &TargetToValue) {
40:   auto *T = BB->getTerminator();
```
- EN: This range implements operational logic in helpers such as getTerminator, translating backend policy into executable code.
- CN: 这一段实现了 getTerminator 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:   if (isa<ReturnInst>(T))
42:     return nullptr;
43:   if (auto *BI = dyn_cast<UncondBrInst>(T))
44:     return TargetToValue.lookup(BI->getSuccessor());
45:
46:   IRBuilder<> Builder(BB);
47:   Builder.SetInsertPoint(T);
48:
49:   if (auto *BI = dyn_cast<CondBrInst>(T)) {
50:     Value *LHS = TargetToValue.lookup(BI->getSuccessor(0));
51:     Value *RHS = TargetToValue.lookup(BI->getSuccessor(1));
52:
53:     if (LHS == nullptr || RHS == nullptr)
54:       return LHS == nullptr ? RHS : LHS;
55:     return Builder.CreateSelect(BI->getCondition(), LHS, RHS);
56:   }
57:
58:   // TODO: add support for switch cases.
59:   llvm_unreachable("Unhandled terminator type.");
60: }
61:
62: static AllocaInst *createVariable(Function &F, Type *Type,
63:                                   BasicBlock::iterator Position) {
64:   const DataLayout &DL = F.getDataLayout();
65:   return new AllocaInst(Type, DL.getAllocaAddrSpace(), nullptr, "reg",
66:                         Position);
67: }
68:
69: // Run the pass on the given convergence region, ignoring the sub-regions.
70: // Returns true if the CFG changed, false otherwise.
71: static bool runOnConvergenceRegionNoRecurse(LoopInfo &LI,
72:                                             SPIRV::ConvergenceRegion *CR) {
73:   // Gather all the exit targets for this region.
74:   SmallPtrSet<BasicBlock *, 4> ExitTargets;
75:   for (BasicBlock *Exit : CR->Exits) {
76:     for (BasicBlock *Target : successors(Exit)) {
77:       if (CR->Blocks.count(Target) == 0)
78:         ExitTargets.insert(Target);
79:     }
80:   }
```
- EN: This range implements operational logic in helpers such as lookup, Builder, SetInsertPoint, CreateSelect, translating backend policy into executable code.
- CN: 这一段实现了 lookup、Builder、SetInsertPoint、CreateSelect 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:
 82:   // If we have zero or one exit target, nothing do to.
 83:   if (ExitTargets.size() <= 1)
 84:     return false;
 85:
 86:   // Create the new single exit target.
 87:   auto F = CR->Entry->getParent();
 88:   auto NewExitTarget = BasicBlock::Create(F->getContext(), "new.exit", F);
 89:   IRBuilder<> Builder(NewExitTarget);
 90:
 91:   AllocaInst *Variable = createVariable(*F, Builder.getInt32Ty(),
 92:                                         F->begin()->getFirstInsertionPt());
 93:
 94:   // CodeGen output needs to be stable. Using the set as-is would order
 95:   // the targets differently depending on the allocation pattern.
 96:   // Sorting per basic-block ordering in the function.
 97:   std::vector<BasicBlock *> SortedExitTargets;
 98:   std::vector<BasicBlock *> SortedExits;
 99:   for (BasicBlock &BB : *F) {
100:     if (ExitTargets.count(&BB) != 0)
101:       SortedExitTargets.push_back(&BB);
102:     if (CR->Exits.count(&BB) != 0)
103:       SortedExits.push_back(&BB);
104:   }
105:
106:   // Creating one constant per distinct exit target. This will be route to the
107:   // correct target.
108:   DenseMap<BasicBlock *, ConstantInt *> TargetToValue;
109:   for (BasicBlock *Target : SortedExitTargets)
110:     TargetToValue.insert(
111:         std::make_pair(Target, Builder.getInt32(TargetToValue.size())));
112:
113:   // Creating one variable per exit node, set to the constant matching the
114:   // targeted external block.
115:   std::vector<std::pair<BasicBlock *, Value *>> ExitToVariable;
116:   for (auto Exit : SortedExits) {
117:     llvm::Value *Value = createExitVariable(Exit, TargetToValue);
118:     IRBuilder<> B2(Exit);
119:     B2.SetInsertPoint(Exit->getFirstInsertionPt());
120:     B2.CreateStore(Value, Variable);
```
- EN: This range implements operational logic in helpers such as getParent, BasicBlock::Create, Builder, begin, translating backend policy into executable code.
- CN: 这一段实现了 getParent、BasicBlock::Create、Builder、begin 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:     ExitToVariable.emplace_back(std::make_pair(Exit, Value));
122:   }
123:
124:   llvm::Value *Load = Builder.CreateLoad(Builder.getInt32Ty(), Variable);
125:
126:   // Creating the switch to jump to the correct exit target.
127:   llvm::SwitchInst *Sw = Builder.CreateSwitch(Load, SortedExitTargets[0],
128:                                               SortedExitTargets.size() - 1);
129:   for (size_t i = 1; i < SortedExitTargets.size(); i++) {
130:     BasicBlock *BB = SortedExitTargets[i];
131:     Sw->addCase(TargetToValue[BB], BB);
132:   }
133:
134:   // Fix exit branches to redirect to the new exit.
135:   for (auto Exit : CR->Exits) {
136:     Instruction *T = Exit->getTerminator();
137:     for (auto I = succ_begin(T), E = succ_end(T); I != E; ++I)
138:       if (ExitTargets.contains(*I))
139:         I.getUse()->set(NewExitTarget);
140:   }
141:
142:   CR = CR->Parent;
143:   while (CR) {
144:     CR->Blocks.insert(NewExitTarget);
145:     CR = CR->Parent;
146:   }
147:
148:   return true;
149: }
150:
151: /// Run the pass on the given convergence region and sub-regions (DFS).
152: /// Returns true if a region/sub-region was modified, false otherwise.
153: /// This returns as soon as one region/sub-region has been modified.
154: static bool runOnConvergenceRegion(LoopInfo &LI, SPIRV::ConvergenceRegion *CR) {
155:   for (auto *Child : CR->Children)
156:     if (runOnConvergenceRegion(LI, Child))
157:       return true;
158:
159:   return runOnConvergenceRegionNoRecurse(LI, CR);
160: }
```
- EN: This range implements operational logic in helpers such as emplace_back, CreateLoad, size, addCase, translating backend policy into executable code.
- CN: 这一段实现了 emplace_back、CreateLoad、size、addCase 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:
162: #if !NDEBUG
163: /// Validates each edge exiting the region has the same destination basic
164: /// block.
165: static void validateRegionExits(const SPIRV::ConvergenceRegion *CR) {
166:   for (auto *Child : CR->Children)
167:     validateRegionExits(Child);
168:
169:   std::unordered_set<BasicBlock *> ExitTargets;
170:   for (auto *Exit : CR->Exits) {
171:     for (auto *BB : successors(Exit)) {
172:       if (CR->Blocks.count(BB) == 0)
173:         ExitTargets.insert(BB);
174:     }
175:   }
176:
177:   assert(ExitTargets.size() <= 1);
178: }
179: #endif
180:
181: static bool runImpl(Function &F, LoopInfo &LI,
182:                     SPIRV::ConvergenceRegionInfo &RegionInfo) {
183:   auto *TopLevelRegion = RegionInfo.getWritableTopLevelRegion();
184:
185:   // FIXME: very inefficient method: each time a region is modified, we bubble
186:   // back up, and recompute the whole convergence region tree. Once the
187:   // algorithm is completed and test coverage good enough, rewrite this pass
188:   // to be efficient instead of simple.
189:   bool Modified = false;
190:   while (runOnConvergenceRegion(LI, TopLevelRegion)) {
191:     Modified = true;
192:   }
193:
194: #if !defined(NDEBUG) || defined(EXPENSIVE_CHECKS)
195:   validateRegionExits(TopLevelRegion);
196: #endif
197:   return Modified;
198: }
199:
200: class SPIRVMergeRegionExitTargetsLegacy : public FunctionPass {
```
- EN: This range defines or declares important types such as validateRegionExits, insert, assert, getWritableTopLevelRegion, shaping the data model used by SPIRVMergeRegionExitTargets.cpp.
- CN: 这一段定义或声明了 validateRegionExits、insert、assert、getWritableTopLevelRegion 等关键类型，构成 SPIRVMergeRegionExitTargets.cpp 使用的数据模型。

### Lines 201-240
```cpp
201: public:
202:   static char ID;
203:
204:   SPIRVMergeRegionExitTargetsLegacy() : FunctionPass(ID) {}
205:
206:   bool runOnFunction(Function &F) override {
207:     LoopInfo &LI = getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
208:     auto &RegionInfo = getAnalysis<SPIRVConvergenceRegionAnalysisWrapperPass>()
209:                            .getRegionInfo();
210:     return runImpl(F, LI, RegionInfo);
211:   }
212:
213:   void getAnalysisUsage(AnalysisUsage &AU) const override {
214:     AU.addRequired<DominatorTreeWrapperPass>();
215:     AU.addRequired<LoopInfoWrapperPass>();
216:     AU.addRequired<SPIRVConvergenceRegionAnalysisWrapperPass>();
217:
218:     AU.addPreserved<SPIRVConvergenceRegionAnalysisWrapperPass>();
219:     FunctionPass::getAnalysisUsage(AU);
220:   }
221: };
222: } // namespace
223:
224: PreservedAnalyses
225: SPIRVMergeRegionExitTargets::run(Function &F, FunctionAnalysisManager &AM) {
226:   auto &LI = AM.getResult<LoopAnalysis>(F);
227:   auto &RegionInfo = AM.getResult<SPIRVConvergenceRegionAnalysis>(F);
228:   return runImpl(F, LI, RegionInfo) ? PreservedAnalyses::none()
229:                                     : PreservedAnalyses::all();
230: }
231:
232: char SPIRVMergeRegionExitTargetsLegacy::ID = 0;
233:
234: INITIALIZE_PASS_BEGIN(SPIRVMergeRegionExitTargetsLegacy,
235:                       "split-region-exit-blocks",
236:                       "SPIRV split region exit blocks", false, false)
237: INITIALIZE_PASS_DEPENDENCY(LoopSimplify)
238: INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
239: INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)
240: INITIALIZE_PASS_DEPENDENCY(SPIRVConvergenceRegionAnalysisWrapperPass)
```
- EN: This range implements operational logic in helpers such as SPIRVMergeRegionExitTargetsLegacy, runOnFunction, getLoopInfo, getRegionInfo, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVMergeRegionExitTargetsLegacy、runOnFunction、getLoopInfo、getRegionInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-248
```cpp
241:
242: INITIALIZE_PASS_END(SPIRVMergeRegionExitTargetsLegacy,
243:                     "split-region-exit-blocks",
244:                     "SPIRV split region exit blocks", false, false)
245:
246: FunctionPass *llvm::createSPIRVMergeRegionExitTargetsPass() {
247:   return new SPIRVMergeRegionExitTargetsLegacy();
248: }
```
- EN: This range implements operational logic in helpers such as llvm::createSPIRVMergeRegionExitTargetsPass, SPIRVMergeRegionExitTargetsLegacy, translating backend policy into executable code.
- CN: 这一段实现了 llvm::createSPIRVMergeRegionExitTargetsPass、SPIRVMergeRegionExitTargetsLegacy 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include getTerminator, lookup, Builder, SetInsertPoint, CreateSelect, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getTerminator, lookup, Builder, SetInsertPoint, CreateSelect，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVMergeRegionExitTargets.h`
  - `Analysis/SPIRVConvergenceRegionAnalysis.h`
  - `SPIRV.h`
  - `SPIRVSubtarget.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/SmallPtrSet.h`
  - `llvm/Analysis/LoopInfo.h`
  - `llvm/IR/Dominators.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/InitializePasses.h`
  - `llvm/Transforms/Utils/Cloning.h`
  - `llvm/Transforms/Utils/LoopSimplify.h`
  - `llvm/Transforms/Utils/LowerMemIntrinsics.h`
