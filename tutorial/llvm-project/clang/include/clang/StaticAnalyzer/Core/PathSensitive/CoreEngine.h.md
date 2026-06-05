# CoreEngine.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/StaticAnalyzer/Core/PathSensitive/CoreEngine.h`
- Repository: `llvm-project`
- Purpose (EN): Path-Sensitive Dataflow Engine.
- 用途（中文）: 该文件为 StaticAnalyzer::Core::PathSensitive 子系统中的 Core Engine 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-29

```cpp
 1: //===- CoreEngine.h - Path-Sensitive Dataflow Engine ------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //  This file defines a generic engine for intraprocedural, path-sensitive,
10: //  dataflow analysis via graph reachability.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #ifndef LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_COREENGINE_H
15: #define LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_COREENGINE_H
16: 
17: #include "clang/AST/Stmt.h"
18: #include "clang/Analysis/AnalysisDeclContext.h"
19: #include "clang/Analysis/CFG.h"
20: #include "clang/Analysis/ProgramPoint.h"
21: #include "clang/Basic/LLVM.h"
22: #include "clang/StaticAnalyzer/Core/BugReporter/BugReporter.h"
23: #include "clang/StaticAnalyzer/Core/PathSensitive/BlockCounter.h"
24: #include "clang/StaticAnalyzer/Core/PathSensitive/ExplodedGraph.h"
25: #include "clang/StaticAnalyzer/Core/PathSensitive/ProgramState_Fwd.h"
26: #include "clang/StaticAnalyzer/Core/PathSensitive/WorkList.h"
27: #include "llvm/ADT/SmallVector.h"
28: #include "llvm/ADT/iterator_range.h"
29: #include "llvm/Support/Casting.h"
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/AST/Stmt.h`, `clang/Analysis/AnalysisDeclContext.h`, `clang/Analysis/CFG.h` and 10 more.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/AST/Stmt.h`, `clang/Analysis/AnalysisDeclContext.h`, `clang/Analysis/CFG.h` 以及另外 10 项依赖。

### Lines 30-58

```cpp
30: #include <cassert>
31: #include <memory>
32: #include <utility>
33: #include <vector>
34: 
35: namespace clang {
36: 
37: class AnalyzerOptions;
38: class CXXBindTemporaryExpr;
39: class Expr;
40: class LabelDecl;
41: 
42: namespace ento {
43: 
44: class FunctionSummariesTy;
45: class ExprEngine;
46: 
47: //===----------------------------------------------------------------------===//
48: /// CoreEngine - Implements the core logic of the graph-reachability analysis.
49: /// It traverses the CFG and generates the ExplodedGraph.
50: class CoreEngine {
51:   friend class ExprEngine;
52:   friend class NodeBuilder;
53:   friend class NodeBuilderContext;
54: 
55: public:
56:   using BlocksExhausted =
57:       std::vector<std::pair<BlockEdge, const ExplodedNode *>>;
58: 
```
- EN: This block imports dependencies such as `cassert`, `memory`, `utility` and 1 more. It opens, closes, or documents namespace scope for `clang`, `ento`. Key type declarations here include `AnalyzerOptions`, `CXXBindTemporaryExpr`, `Expr`, `LabelDecl`.
- 中文: 这一块引入了 `cassert`, `memory`, `utility` 以及另外 1 项依赖。 它打开、关闭或说明了 `clang`, `ento` 的命名空间作用域。 这里的重要类型声明包括 `AnalyzerOptions`, `CXXBindTemporaryExpr`, `Expr`, `LabelDecl`。

### Lines 59-87

```cpp
59:   using BlocksAborted =
60:       std::vector<std::pair<const CFGBlock *, const ExplodedNode *>>;
61: 
62: private:
63:   ExprEngine &ExprEng;
64: 
65:   /// G - The simulation graph.  Each node is a (location,state) pair.
66:   mutable ExplodedGraph G;
67: 
68:   /// WList - A set of queued nodes that need to be processed by the
69:   ///  worklist algorithm.  It is up to the implementation of WList to decide
70:   ///  the order that nodes are processed.
71:   std::unique_ptr<WorkList> WList;
72:   std::unique_ptr<WorkList> CTUWList;
73: 
74:   /// BCounterFactory - A factory object for created BlockCounter objects.
75:   ///   These are used to record for key nodes in the ExplodedGraph the
76:   ///   number of times different CFGBlocks have been visited along a path.
77:   BlockCounter::Factory BCounterFactory;
78: 
79:   /// The locations where we stopped doing work because we visited a location
80:   ///  too many times.
81:   BlocksExhausted blocksExhausted;
82: 
83:   /// The locations where we stopped because the engine aborted analysis,
84:   /// usually because it could not reason about something.
85:   BlocksAborted blocksAborted;
86: 
87:   /// The information about functions shared by the whole translation unit.
```
- EN: It defines convenient aliases such as `BlocksAborted`.
- 中文: 它定义了 `BlocksAborted` 等便捷别名。

### Lines 88-116

```cpp
 88:   /// (This data is owned by AnalysisConsumer.)
 89:   FunctionSummariesTy *FunctionSummaries;
 90: 
 91:   /// Add path tags with some useful data along the path when we see that
 92:   /// something interesting is happening. This field is the allocator for such
 93:   /// tags.
 94:   DataTag::Factory DataTags;
 95: 
 96:   void setBlockCounter(BlockCounter C);
 97: 
 98:   void HandleBlockEdge(const BlockEdge &E, ExplodedNode *Pred);
 99:   void HandleBlockEntrance(const BlockEntrance &E, ExplodedNode *Pred);
100:   void HandleBlockExit(const CFGBlock *B, ExplodedNode *Pred);
101: 
102:   void HandleCallEnter(const CallEnter &CE, ExplodedNode *Pred);
103: 
104:   void HandlePostStmt(const CFGBlock *B, unsigned StmtIdx, ExplodedNode *Pred);
105: 
106:   void HandleBranch(const Stmt *Cond, const Stmt *Term, const CFGBlock *B,
107:                     ExplodedNode *Pred);
108:   void HandleCleanupTemporaryBranch(const CXXBindTemporaryExpr *BTE,
109:                                     const CFGBlock *B, ExplodedNode *Pred);
110: 
111:   /// Handle conditional logic for running static initializers.
112:   void HandleStaticInit(const DeclStmt *DS, const CFGBlock *B,
113:                         ExplodedNode *Pred);
114: 
115:   void HandleVirtualBaseBranch(const CFGBlock *B, ExplodedNode *Pred);
116: 
```
- EN: It exposes API surface such as `setBlockCounter`, `HandleBlockEdge`, `HandleBlockEntrance`, `HandleBlockExit`.
- 中文: 它暴露了 `setBlockCounter`, `HandleBlockEdge`, `HandleBlockEntrance`, `HandleBlockExit` 等接口。

### Lines 117-145

```cpp
117: private:
118:   /// Helper function called by `HandleBranch()`. If the currently handled
119:   /// branch corresponds to a loop, this returns the number of already
120:   /// completed iterations in that loop, otherwise the return value is
121:   /// `std::nullopt`. Note that this counts _all_ earlier iterations, including
122:   /// ones that were performed within an earlier iteration of an outer loop.
123:   std::optional<unsigned> getCompletedIterationCount(const CFGBlock *B,
124:                                                      ExplodedNode *Pred) const;
125: 
126: public:
127:   /// Construct a CoreEngine object to analyze the provided CFG.
128:   CoreEngine(ExprEngine &exprengine,
129:              FunctionSummariesTy *FS,
130:              AnalyzerOptions &Opts);
131: 
132:   CoreEngine(const CoreEngine &) = delete;
133:   CoreEngine &operator=(const CoreEngine &) = delete;
134: 
135:   /// getGraph - Returns the exploded graph.
136:   ExplodedGraph &getGraph() { return G; }
137: 
138:   /// ExecuteWorkList - Run the worklist algorithm for a maximum number of
139:   ///  steps.  Returns true if there is still simulation state on the worklist.
140:   bool ExecuteWorkList(const LocationContext *L, unsigned Steps,
141:                        ProgramStateRef InitState);
142: 
143:   /// Dispatch the work list item based on the given location information.
144:   /// Use Pred parameter as the predecessor state.
145:   void dispatchWorkItem(ExplodedNode* Pred, ProgramPoint Loc,
```
- EN: It exposes API surface such as `CoreEngine`, `getGraph`.
- 中文: 它暴露了 `CoreEngine`, `getGraph` 等接口。

### Lines 146-174

```cpp
146:                         const WorkListUnit& WU);
147: 
148:   // Functions for external checking of whether we have unfinished work.
149:   bool wasBlockAborted() const { return !blocksAborted.empty(); }
150:   bool wasBlocksExhausted() const { return !blocksExhausted.empty(); }
151:   bool hasWorkRemaining() const { return wasBlocksExhausted() ||
152:                                          WList->hasWork() ||
153:                                          wasBlockAborted(); }
154: 
155:   /// Inform the CoreEngine that a basic block was aborted because
156:   /// it could not be completely analyzed.
157:   void addAbortedBlock(const ExplodedNode *node, const CFGBlock *block) {
158:     blocksAborted.push_back(std::make_pair(block, node));
159:   }
160: 
161:   WorkList *getWorkList() const { return WList.get(); }
162:   WorkList *getCTUWorkList() const { return CTUWList.get(); }
163: 
164:   auto exhausted_blocks() const {
165:     return llvm::iterator_range(blocksExhausted);
166:   }
167: 
168:   auto aborted_blocks() const { return llvm::iterator_range(blocksAborted); }
169: 
170:   ExplodedNode *makeNode(const ProgramPoint &Loc, ProgramStateRef State,
171:                          ExplodedNode *Pred, bool MarkAsSink = false) const;
172: 
173:   ExplodedNode *makePostStmtNode(const Stmt *S, ProgramStateRef State,
174:                                  ExplodedNode *Pred,
```
- EN: It exposes API surface such as `wasBlockAborted`, `wasBlocksExhausted`, `hasWorkRemaining`, `addAbortedBlock`.
- 中文: 它暴露了 `wasBlockAborted`, `wasBlocksExhausted`, `hasWorkRemaining`, `addAbortedBlock` 等接口。

### Lines 175-203

```cpp
175:                                  bool MarkAsSink = false) const {
176:     PostStmt Loc(S, Pred->getLocationContext(), /*tag=*/nullptr);
177:     return makeNode(Loc, State, Pred, MarkAsSink);
178:   }
179: 
180:   ExplodedNode *
181:   makeNodeWithBinding(ExplodedNode *Pred, const Expr *E, SVal V,
182:                       ProgramStateRef State,
183:                       ProgramPoint::Kind K = ProgramPoint::PostStmtKind) const {
184:     const LocationContext *LC = Pred->getLocationContext();
185:     State = State->BindExpr(E, LC, V);
186:     const auto &L = ProgramPoint::getProgramPoint(E, K, LC, /*tag=*/nullptr);
187:     return makeNode(L, State, Pred);
188:   }
189: 
190:   ExplodedNode *
191:   makeNodeWithBinding(ExplodedNode *Pred, const Expr *E, SVal V,
192:                       ProgramPoint::Kind K = ProgramPoint::PostStmtKind) const {
193:     return makeNodeWithBinding(Pred, E, V, Pred->getState(), K);
194:   }
195: 
196:   /// Enqueue the given set of nodes onto the work list.
197:   void enqueue(ExplodedNodeSet &Set);
198: 
199:   /// Enqueue nodes that were created as a result of processing
200:   /// a statement onto the work list.
201:   void enqueueStmtNodes(ExplodedNodeSet &Set, const CFGBlock *Block,
202:                         unsigned Idx);
203: 
```
- EN: It exposes API surface such as `Loc`, `makeNode`, `getLocationContext`, `BindExpr`.
- 中文: 它暴露了 `Loc`, `makeNode`, `getLocationContext`, `BindExpr` 等接口。

### Lines 204-232

```cpp
204:   /// enqueue the nodes corresponding to the end of function onto the
205:   /// end of path / work list.
206:   void enqueueEndOfFunction(ExplodedNodeSet &Set, const ReturnStmt *RS);
207: 
208:   /// Enqueue a single node created as a result of statement processing.
209:   void enqueueStmtNode(ExplodedNode *N, const CFGBlock *Block, unsigned Idx);
210: 
211:   DataTag::Factory &getDataTags() { return DataTags; }
212: };
213: 
214: class NodeBuilderContext {
215:   const CoreEngine &Eng;
216:   const CFGBlock *Block;
217:   const LocationContext *LC;
218: 
219: public:
220:   NodeBuilderContext(const CoreEngine &E, const CFGBlock *B,
221:                      const LocationContext *L)
222:       : Eng(E), Block(B), LC(L) {
223:     assert(B);
224:   }
225: 
226:   NodeBuilderContext(const CoreEngine &E, const CFGBlock *B, ExplodedNode *N)
227:       : NodeBuilderContext(E, B, N->getLocationContext()) {}
228: 
229:   /// Return the CoreEngine associated with this builder.
230:   const CoreEngine &getEngine() const { return Eng; }
231: 
232:   /// Return the CFGBlock associated with this builder.
```
- EN: Key type declarations here include `NodeBuilderContext`. It exposes API surface such as `enqueueEndOfFunction`, `enqueueStmtNode`, `getDataTags`, `Eng`.
- 中文: 这里的重要类型声明包括 `NodeBuilderContext`。 它暴露了 `enqueueEndOfFunction`, `enqueueStmtNode`, `getDataTags`, `Eng` 等接口。

### Lines 233-261

```cpp
233:   const CFGBlock *getBlock() const { return Block; }
234: 
235:   /// Return the location context associated with this builder.
236:   const LocationContext *getLocationContext() const { return LC; }
237: 
238:   /// Returns the number of times the current basic block has been
239:   /// visited on the exploded graph path.
240:   unsigned blockCount() const {
241:     return Eng.WList->getBlockCounter().getNumVisited(
242:                     LC->getStackFrame(),
243:                     Block->getBlockID());
244:   }
245: };
246: 
247: /// \class NodeBuilder
248: /// This is the simplest builder which generates nodes in the
249: /// ExplodedGraph.
250: ///
251: /// The main benefit of the builder is that it automatically tracks the
252: /// frontier nodes (or destination set). This is the set of nodes which should
253: /// be propagated to the next step / builder. They are the nodes which have been
254: /// added to the builder (either as the input node set or as the newly
255: /// constructed nodes) but did not have any outgoing transitions added.
256: ///
257: /// TODO: This "main benefit" is often useless, in fact the only significant
258: /// use is within `CheckerManager::ExpandGraphWithCheckers`. There this logic
259: /// ensures that if a checker performs multiple transitions on the same path,
260: /// then only the last of them is "built upon" by other checkers or the engine.
261: ///
```
- EN: It exposes API surface such as `getBlock`, `getLocationContext`, `blockCount`, `getBlockID`.
- 中文: 它暴露了 `getBlock`, `getLocationContext`, `blockCount`, `getBlockID` 等接口。

### Lines 262-290

```cpp
262: /// However, there are also many short-lived temporary `NodeBuilder` instances
263: /// where the `generateNode` is called in a very predictable manner (once, or
264: /// once for each source node) and the frontier management is overkill.
265: /// These locations should be gradually simplified by using the method
266: /// `CoreEngine::makeNode()` instead of the temporary `NodeBuilder`s.
267: class NodeBuilder {
268: protected:
269:   const NodeBuilderContext &C;
270: 
271:   bool HasGeneratedNodes = false;
272: 
273:   /// The frontier set - a set of nodes which need to be propagated after
274:   /// the builder dies.
275:   ExplodedNodeSet &Frontier;
276: 
277: public:
278:   NodeBuilder(ExplodedNodeSet &DstSet, const NodeBuilderContext &Ctx)
279:       : C(Ctx), Frontier(DstSet) {}
280: 
281:   NodeBuilder(ExplodedNode *SrcNode, ExplodedNodeSet &DstSet,
282:               const NodeBuilderContext &Ctx)
283:       : NodeBuilder(DstSet, Ctx) {
284:     Frontier.insert(SrcNode);
285:   }
286: 
287:   NodeBuilder(const ExplodedNodeSet &SrcSet, ExplodedNodeSet &DstSet,
288:               const NodeBuilderContext &Ctx)
289:       : NodeBuilder(DstSet, Ctx) {
290:     Frontier.insert(SrcSet);
```
- EN: Key type declarations here include `NodeBuilder`. It exposes API surface such as `C`, `NodeBuilder`, `insert`.
- 中文: 这里的重要类型声明包括 `NodeBuilder`。 它暴露了 `C`, `NodeBuilder`, `insert` 等接口。

### Lines 291-319

```cpp
291:   }
292: 
293:   /// Generates a node in the ExplodedGraph.
294:   ExplodedNode *generateNode(const ProgramPoint &PP, ProgramStateRef State,
295:                              ExplodedNode *Pred, bool MarkAsSink = false);
296: 
297:   /// Generates a sink in the ExplodedGraph.
298:   ///
299:   /// When a node is marked as sink, the exploration from the node is stopped -
300:   /// the node becomes the last node on the path and certain kinds of bugs are
301:   /// suppressed.
302:   ExplodedNode *generateSink(const ProgramPoint &PP,
303:                              ProgramStateRef State,
304:                              ExplodedNode *Pred) {
305:     return generateNode(PP, State, Pred, true);
306:   }
307: 
308:   ExplodedNode *generateNode(const Stmt *S,
309:                              ExplodedNode *Pred,
310:                              ProgramStateRef St,
311:                              const ProgramPointTag *tag = nullptr,
312:                              ProgramPoint::Kind K = ProgramPoint::PostStmtKind){
313:     const ProgramPoint &L = ProgramPoint::getProgramPoint(S, K,
314:                                   Pred->getLocationContext(), tag);
315:     return generateNode(L, St, Pred);
316:   }
317: 
318:   ExplodedNode *generateSink(const Stmt *S,
319:                              ExplodedNode *Pred,
```
- EN: It exposes API surface such as `generateNode`, `getLocationContext`.
- 中文: 它暴露了 `generateNode`, `getLocationContext` 等接口。

### Lines 320-346

```cpp
320:                              ProgramStateRef St,
321:                              const ProgramPointTag *tag = nullptr,
322:                              ProgramPoint::Kind K = ProgramPoint::PostStmtKind){
323:     const ProgramPoint &L = ProgramPoint::getProgramPoint(S, K,
324:                                   Pred->getLocationContext(), tag);
325:     return generateSink(L, St, Pred);
326:   }
327: 
328:   const ExplodedNodeSet &getResults() const { return Frontier; }
329: 
330:   bool hasGeneratedNodes() const { return HasGeneratedNodes; }
331: 
332:   void takeNodes(const ExplodedNodeSet &S) {
333:     for (const auto I : S)
334:       Frontier.erase(I);
335:   }
336: 
337:   void takeNodes(ExplodedNode *N) { Frontier.erase(N); }
338:   void addNodes(const ExplodedNodeSet &S) { Frontier.insert(S); }
339:   void addNodes(ExplodedNode *N) { Frontier.insert(N); }
340: };
341: 
342: } // namespace ento
343: 
344: } // namespace clang
345: 
346: #endif // LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_COREENGINE_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `ento`, `clang`. It exposes API surface such as `getLocationContext`, `generateSink`, `getResults`, `hasGeneratedNodes`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `ento`, `clang` 的命名空间作用域。 它暴露了 `getLocationContext`, `generateSink`, `getResults`, `hasGeneratedNodes` 等接口。

## Key Concepts / 关键概念

- `AnalyzerOptions`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CXXBindTemporaryExpr`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Expr`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `LabelDecl`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `FunctionSummariesTy`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ExprEngine`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CoreEngine`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `NodeBuilder`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/AST/Stmt.h`, `clang/Analysis/AnalysisDeclContext.h`, `clang/Analysis/CFG.h`, `clang/Analysis/ProgramPoint.h`, `clang/Basic/LLVM.h`, `clang/StaticAnalyzer/Core/BugReporter/BugReporter.h`, `clang/StaticAnalyzer/Core/PathSensitive/BlockCounter.h`, `clang/StaticAnalyzer/Core/PathSensitive/ExplodedGraph.h`, `clang/StaticAnalyzer/Core/PathSensitive/ProgramState_Fwd.h`, `clang/StaticAnalyzer/Core/PathSensitive/WorkList.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/iterator_range.h`, `llvm/Support/Casting.h`, `cassert`, `memory`
- Forward declarations / 前向声明: `AnalyzerOptions`, `CXXBindTemporaryExpr`, `Expr`, `LabelDecl`, `FunctionSummariesTy`, `ExprEngine`
- Namespace context / 命名空间上下文: `clang`, `ento`
- Macro-style dependencies / 宏式依赖: None / 无
