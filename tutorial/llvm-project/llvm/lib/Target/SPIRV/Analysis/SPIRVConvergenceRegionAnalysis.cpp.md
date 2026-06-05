# SPIRVConvergenceRegionAnalysis.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/Analysis/SPIRVConvergenceRegionAnalysis.cpp`
- Repository: `llvm-project`
- Purpose (EN): The analysis determines the convergence region for each basic block of the module, and provides a tree-like structure describing the region hierarchy.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===----------------------------------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // The analysis determines the convergence region for each basic block of
10: // the module, and provides a tree-like structure describing the region
11: // hierarchy.
12: //
13: //===----------------------------------------------------------------------===//
14:
15: #include "SPIRVConvergenceRegionAnalysis.h"
16: #include "SPIRV.h"
17: #include "llvm/Analysis/LoopInfo.h"
18: #include "llvm/IR/Dominators.h"
19: #include "llvm/IR/IntrinsicInst.h"
20: #include "llvm/InitializePasses.h"
21: #include "llvm/Transforms/Utils/LoopSimplify.h"
22: #include <optional>
23: #include <queue>
24: #include <unordered_set>
25:
26: #define DEBUG_TYPE "spirv-convergence-region-analysis"
27:
28: using namespace llvm;
29: using namespace SPIRV;
30:
31: INITIALIZE_PASS_BEGIN(SPIRVConvergenceRegionAnalysisWrapperPass,
32:                       "convergence-region",
33:                       "SPIRV convergence regions analysis", true, true)
34: INITIALIZE_PASS_DEPENDENCY(LoopSimplify)
35: INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
36: INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)
37: INITIALIZE_PASS_END(SPIRVConvergenceRegionAnalysisWrapperPass,
38:                     "convergence-region", "SPIRV convergence regions analysis",
39:                     true, true)
40:
41: namespace {
42:
43: template <typename BasicBlockType, typename IntrinsicInstType>
44: std::optional<IntrinsicInstType *>
45: getConvergenceTokenInternal(BasicBlockType *BB) {
46:   static_assert(std::is_const_v<IntrinsicInstType> ==
47:                     std::is_const_v<BasicBlockType>,
48:                 "Constness must match between input and output.");
49:   static_assert(std::is_same_v<BasicBlock, std::remove_const_t<BasicBlockType>>,
50:                 "Input must be a basic block.");
51:   static_assert(
52:       std::is_same_v<IntrinsicInst, std::remove_const_t<IntrinsicInstType>>,
53:       "Output type must be an intrinsic instruction.");
54:
55:   for (auto &I : *BB) {
56:     if (auto *CI = dyn_cast<ConvergenceControlInst>(&I)) {
57:       // Make sure that the anchor or entry intrinsics did not reach here with a
58:       // parent token. This should have failed the verifier.
59:       assert(CI->isLoop() ||
60:              !CI->getOperandBundle(LLVMContext::OB_convergencectrl));
```
- EN: This range implements operational logic in helpers such as INITIALIZE_PASS_DEPENDENCY, getConvergenceTokenInternal, getOperandBundle, translating backend policy into executable code.
- CN: 这一段实现了 INITIALIZE_PASS_DEPENDENCY、getConvergenceTokenInternal、getOperandBundle 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:       return CI;
 62:     }
 63:
 64:     if (auto *CI = dyn_cast<CallInst>(&I)) {
 65:       auto OB = CI->getOperandBundle(LLVMContext::OB_convergencectrl);
 66:       if (!OB.has_value())
 67:         continue;
 68:       return dyn_cast<IntrinsicInst>(OB.value().Inputs[0]);
 69:     }
 70:   }
 71:
 72:   return std::nullopt;
 73: }
 74: } // anonymous namespace
 75:
 76: // Given a ConvergenceRegion tree with |Start| as its root, finds the smallest
 77: // region |Entry| belongs to. If |Entry| does not belong to the region defined
 78: // by |Start|, this function returns |nullptr|.
 79: static ConvergenceRegion *findParentRegion(ConvergenceRegion *Start,
 80:                                            BasicBlock *Entry) {
 81:   ConvergenceRegion *Candidate = nullptr;
 82:   ConvergenceRegion *NextCandidate = Start;
 83:
 84:   while (Candidate != NextCandidate && NextCandidate != nullptr) {
 85:     Candidate = NextCandidate;
 86:     NextCandidate = nullptr;
 87:
 88:     // End of the search, we can return.
 89:     if (Candidate->Children.size() == 0)
 90:       return Candidate;
 91:
 92:     for (auto *Child : Candidate->Children) {
 93:       if (Child->Blocks.count(Entry) != 0) {
 94:         NextCandidate = Child;
 95:         break;
 96:       }
 97:     }
 98:   }
 99:
100:   return Candidate;
101: }
102:
103: std::optional<IntrinsicInst *>
104: llvm::SPIRV::getConvergenceToken(BasicBlock *BB) {
105:   return getConvergenceTokenInternal<BasicBlock, IntrinsicInst>(BB);
106: }
107:
108: std::optional<const IntrinsicInst *>
109: llvm::SPIRV::getConvergenceToken(const BasicBlock *BB) {
110:   return getConvergenceTokenInternal<const BasicBlock, const IntrinsicInst>(BB);
111: }
112:
113: ConvergenceRegion::ConvergenceRegion(DominatorTree &DT, LoopInfo &LI,
114:                                      Function &F)
115:     : DT(DT), LI(LI), Parent(nullptr) {
116:   Entry = &F.getEntryBlock();
117:   ConvergenceToken = getConvergenceToken(Entry);
118:   for (auto &B : F) {
119:     Blocks.insert(&B);
120:     if (isa<ReturnInst>(B.getTerminator()))
```
- EN: This range implements operational logic in helpers such as getOperandBundle, value, llvm::SPIRV::getConvergenceToken, DT, translating backend policy into executable code.
- CN: 这一段实现了 getOperandBundle、value、llvm::SPIRV::getConvergenceToken、DT 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:       Exits.insert(&B);
122:   }
123: }
124:
125: ConvergenceRegion::ConvergenceRegion(
126:     DominatorTree &DT, LoopInfo &LI,
127:     std::optional<IntrinsicInst *> ConvergenceToken, BasicBlock *Entry,
128:     SmallPtrSet<BasicBlock *, 8> &&Blocks, SmallPtrSet<BasicBlock *, 2> &&Exits)
129:     : DT(DT), LI(LI), ConvergenceToken(ConvergenceToken), Entry(Entry),
130:       Exits(std::move(Exits)), Blocks(std::move(Blocks)) {
131:   for ([[maybe_unused]] auto *BB : this->Exits)
132:     assert(this->Blocks.count(BB) != 0);
133:   assert(this->Blocks.count(this->Entry) != 0);
134: }
135:
136: void ConvergenceRegion::releaseMemory() {
137:   // Parent memory is owned by the parent.
138:   Parent = nullptr;
139:   for (auto *Child : Children) {
140:     Child->releaseMemory();
141:     delete Child;
142:   }
143:   Children.resize(0);
144: }
145:
146: void ConvergenceRegion::dump(const unsigned IndentSize) const {
147:   const std::string Indent(IndentSize, '\t');
148:   dbgs() << Indent << this << ": {\n";
149:   dbgs() << Indent << "	Parent: " << Parent << "\n";
150:
151:   if (ConvergenceToken.value_or(nullptr)) {
152:     dbgs() << Indent
153:            << "	ConvergenceToken: " << ConvergenceToken.value()->getName()
154:            << "\n";
155:   }
156:
157:   if (Entry->getName() != "")
158:     dbgs() << Indent << "	Entry: " << Entry->getName() << "\n";
159:   else
160:     dbgs() << Indent << "	Entry: " << Entry << "\n";
161:
162:   dbgs() << Indent << "	Exits: { ";
163:   for (const auto &Exit : Exits) {
164:     if (Exit->getName() != "")
165:       dbgs() << Exit->getName() << ", ";
166:     else
167:       dbgs() << Exit << ", ";
168:   }
169:   dbgs() << "	}\n";
170:
171:   dbgs() << Indent << "	Blocks: { ";
172:   for (const auto &Block : Blocks) {
173:     if (Block->getName() != "")
174:       dbgs() << Block->getName() << ", ";
175:     else
176:       dbgs() << Block << ", ";
177:   }
178:   dbgs() << "	}\n";
179:
180:   dbgs() << Indent << "	Children: {\n";
```
- EN: This range implements operational logic in helpers such as insert, Exits, assert, ConvergenceRegion::releaseMemory, translating backend policy into executable code.
- CN: 这一段实现了 insert、Exits、assert、ConvergenceRegion::releaseMemory 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:   for (const auto Child : Children)
182:     Child->dump(IndentSize + 2);
183:   dbgs() << Indent << "	}\n";
184:
185:   dbgs() << Indent << "}\n";
186: }
187:
188: namespace {
189: class ConvergenceRegionAnalyzer {
190: public:
191:   ConvergenceRegionAnalyzer(Function &F, DominatorTree &DT, LoopInfo &LI)
192:       : DT(DT), LI(LI), F(F) {}
193:
194: private:
195:   bool isBackEdge(const BasicBlock *From, const BasicBlock *To) const {
196:     if (From == To)
197:       return true;
198:
199:     // We only handle loop in the simplified form. This means:
200:     // - a single back-edge, a single latch.
201:     // - meaning the back-edge target can only be the loop header.
202:     // - meaning the From can only be the loop latch.
203:     if (!LI.isLoopHeader(To))
204:       return false;
205:
206:     auto *L = LI.getLoopFor(To);
207:     if (L->contains(From) && L->isLoopLatch(From))
208:       return true;
209:
210:     return false;
211:   }
212:
213:   std::unordered_set<BasicBlock *>
214:   findPathsToMatch(LoopInfo &LI, BasicBlock *From,
215:                    std::function<bool(const BasicBlock *)> isMatch) const {
216:     std::unordered_set<BasicBlock *> Output;
217:
218:     if (isMatch(From))
219:       Output.insert(From);
220:
221:     auto *Terminator = From->getTerminator();
222:     for (unsigned i = 0; i < Terminator->getNumSuccessors(); ++i) {
223:       auto *To = Terminator->getSuccessor(i);
224:       // Ignore back edges.
225:       if (isBackEdge(From, To))
226:         continue;
227:
228:       auto ChildSet = findPathsToMatch(LI, To, isMatch);
229:       if (ChildSet.size() == 0)
230:         continue;
231:
232:       Output.insert(ChildSet.begin(), ChildSet.end());
233:       Output.insert(From);
234:       if (LI.isLoopHeader(From)) {
235:         auto *L = LI.getLoopFor(From);
236:         for (auto *BB : L->getBlocks()) {
237:           Output.insert(BB);
238:         }
239:       }
240:     }
```
- EN: This range defines or declares important types such as dump, ConvergenceRegionAnalyzer, DT, isBackEdge, shaping the data model used by SPIRVConvergenceRegionAnalysis.cpp.
- CN: 这一段定义或声明了 dump、ConvergenceRegionAnalyzer、DT、isBackEdge 等关键类型，构成 SPIRVConvergenceRegionAnalysis.cpp 使用的数据模型。

### Lines 241-300
```cpp
241:
242:     return Output;
243:   }
244:
245:   SmallPtrSet<BasicBlock *, 2>
246:   findExitNodes(const SmallPtrSetImpl<BasicBlock *> &RegionBlocks) {
247:     SmallPtrSet<BasicBlock *, 2> Exits;
248:
249:     for (auto *B : RegionBlocks) {
250:       auto *Terminator = B->getTerminator();
251:       for (unsigned i = 0; i < Terminator->getNumSuccessors(); ++i) {
252:         auto *Child = Terminator->getSuccessor(i);
253:         if (RegionBlocks.count(Child) == 0)
254:           Exits.insert(B);
255:       }
256:     }
257:
258:     return Exits;
259:   }
260:
261: public:
262:   ConvergenceRegionInfo analyze() {
263:     ConvergenceRegion *TopLevelRegion = new ConvergenceRegion(DT, LI, F);
264:     std::queue<Loop *> ToProcess;
265:     for (auto *L : LI.getLoopsInPreorder())
266:       ToProcess.push(L);
267:
268:     while (ToProcess.size() != 0) {
269:       auto *L = ToProcess.front();
270:       ToProcess.pop();
271:
272:       auto CT = getConvergenceToken(L->getHeader());
273:       SmallPtrSet<BasicBlock *, 8> RegionBlocks(llvm::from_range, L->blocks());
274:       SmallVector<BasicBlock *> LoopExits;
275:       L->getExitingBlocks(LoopExits);
276:       if (CT.has_value()) {
277:         for (auto *Exit : LoopExits) {
278:           auto N = findPathsToMatch(LI, Exit, [&CT](const BasicBlock *block) {
279:             auto Token = getConvergenceToken(block);
280:             if (Token == std::nullopt)
281:               return false;
282:             return Token.value() == CT.value();
283:           });
284:           RegionBlocks.insert_range(N);
285:         }
286:       }
287:
288:       auto RegionExits = findExitNodes(RegionBlocks);
289:       ConvergenceRegion *Region = new ConvergenceRegion(
290:           DT, LI, CT, L->getHeader(), std::move(RegionBlocks),
291:           std::move(RegionExits));
292:       Region->Parent = findParentRegion(TopLevelRegion, Region->Entry);
293:       assert(Region->Parent != nullptr && "This is impossible.");
294:       Region->Parent->Children.push_back(Region);
295:     }
296:
297:     return ConvergenceRegionInfo(TopLevelRegion);
298:   }
299:
300: private:
```
- EN: This range implements operational logic in helpers such as findExitNodes, getTerminator, getNumSuccessors, getSuccessor, translating backend policy into executable code.
- CN: 这一段实现了 findExitNodes、getTerminator、getNumSuccessors、getSuccessor 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-338
```cpp
301:   DominatorTree &DT;
302:   LoopInfo &LI;
303:   Function &F;
304: };
305: } // anonymous namespace
306:
307: ConvergenceRegionInfo llvm::SPIRV::getConvergenceRegions(Function &F,
308:                                                          DominatorTree &DT,
309:                                                          LoopInfo &LI) {
310:   ConvergenceRegionAnalyzer Analyzer(F, DT, LI);
311:   return Analyzer.analyze();
312: }
313:
314: char SPIRVConvergenceRegionAnalysisWrapperPass::ID = 0;
315:
316: SPIRVConvergenceRegionAnalysisWrapperPass::
317:     SPIRVConvergenceRegionAnalysisWrapperPass()
318:     : FunctionPass(ID) {}
319:
320: bool SPIRVConvergenceRegionAnalysisWrapperPass::runOnFunction(Function &F) {
321:   DominatorTree &DT = getAnalysis<DominatorTreeWrapperPass>().getDomTree();
322:   LoopInfo &LI = getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
323:
324:   CRI = SPIRV::getConvergenceRegions(F, DT, LI);
325:   // Nothing was modified.
326:   return false;
327: }
328:
329: SPIRVConvergenceRegionAnalysis::Result
330: SPIRVConvergenceRegionAnalysis::run(Function &F, FunctionAnalysisManager &AM) {
331:   Result CRI;
332:   auto &DT = AM.getResult<DominatorTreeAnalysis>(F);
333:   auto &LI = AM.getResult<LoopAnalysis>(F);
334:   CRI = SPIRV::getConvergenceRegions(F, DT, LI);
335:   return CRI;
336: }
337:
338: AnalysisKey SPIRVConvergenceRegionAnalysis::Key;
```
- EN: This range implements operational logic in helpers such as Analyzer, analyze, SPIRVConvergenceRegionAnalysisWrapperPass, FunctionPass, translating backend policy into executable code.
- CN: 这一段实现了 Analyzer、analyze、SPIRVConvergenceRegionAnalysisWrapperPass、FunctionPass 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include INITIALIZE_PASS_DEPENDENCY, getConvergenceTokenInternal, getOperandBundle, value, llvm::SPIRV::getConvergenceToken, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 INITIALIZE_PASS_DEPENDENCY, getConvergenceTokenInternal, getOperandBundle, value, llvm::SPIRV::getConvergenceToken，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVConvergenceRegionAnalysis.h`
  - `SPIRV.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/LoopInfo.h`
  - `llvm/IR/Dominators.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/InitializePasses.h`
  - `llvm/Transforms/Utils/LoopSimplify.h`
- System/standard headers / 系统或标准头文件:
  - `optional`
  - `queue`
  - `unordered_set`
