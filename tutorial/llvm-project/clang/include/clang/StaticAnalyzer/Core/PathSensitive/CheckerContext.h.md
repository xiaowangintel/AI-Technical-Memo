# CheckerContext.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/StaticAnalyzer/Core/PathSensitive/CheckerContext.h`
- Repository: `llvm-project`
- Purpose (EN): CheckerContext.h - Context info for path-sensitive checkers--*- C++ -*--=// This file defines CheckerContext that provides contextual info for path-sensitive checkers.
- 用途（中文）: 该文件为 StaticAnalyzer::Core::PathSensitive 子系统中的 Checker Context 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
 1: //== CheckerContext.h - Context info for path-sensitive checkers--*- C++ -*--=//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //  This file defines CheckerContext that provides contextual info for
10: //  path-sensitive checkers.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #ifndef LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_CHECKERCONTEXT_H
15: #define LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_CHECKERCONTEXT_H
16: 
17: #include "clang/StaticAnalyzer/Core/PathSensitive/ExprEngine.h"
18: #include "clang/StaticAnalyzer/Core/PathSensitive/ProgramStateTrait.h"
19: #include <optional>
20: 
21: namespace clang {
22: namespace ento {
23: 
24: class CheckerContext {
25:   ExprEngine &Eng;
26:   /// The current exploded(symbolic execution) graph node.
27:   ExplodedNode *Pred;
28:   /// The flag is true if the (state of the execution) has been modified
29:   /// by the checker using this context. For example, a new transition has been
30:   /// added or a bug report issued.
31:   bool Changed;
32:   /// The tagged location, which is used to generate all new nodes.
33:   const ProgramPoint Location;
34:   NodeBuilder &NB;
35: 
36: public:
37:   /// If we are post visiting a call, this flag will be set if the
38:   /// call was inlined.  In all other cases it will be false.
39:   const bool wasInlined;
40: 
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/StaticAnalyzer/Core/PathSensitive/ExprEngine.h`, `clang/StaticAnalyzer/Core/PathSensitive/ProgramStateTrait.h`, `optional`. It opens, closes, or documents namespace scope for `clang`, `ento`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/StaticAnalyzer/Core/PathSensitive/ExprEngine.h`, `clang/StaticAnalyzer/Core/PathSensitive/ProgramStateTrait.h`, `optional` 等依赖。 它打开、关闭或说明了 `clang`, `ento` 的命名空间作用域。

### Lines 41-80

```cpp
41:   CheckerContext(NodeBuilder &builder,
42:                  ExprEngine &eng,
43:                  ExplodedNode *pred,
44:                  const ProgramPoint &loc,
45:                  bool wasInlined = false)
46:     : Eng(eng),
47:       Pred(pred),
48:       Changed(false),
49:       Location(loc),
50:       NB(builder),
51:       wasInlined(wasInlined) {
52:     assert(Pred->getState() &&
53:            "We should not call the checkers on an empty state.");
54:     assert(loc.getTag() && "The ProgramPoint associated with CheckerContext "
55:                            "must be tagged with the active checker.");
56:   }
57: 
58:   AnalysisManager &getAnalysisManager() {
59:     return Eng.getAnalysisManager();
60:   }
61:   const AnalysisManager &getAnalysisManager() const {
62:     return Eng.getAnalysisManager();
63:   }
64: 
65:   ConstraintManager &getConstraintManager() {
66:     return Eng.getConstraintManager();
67:   }
68:   const ConstraintManager &getConstraintManager() const {
69:     return Eng.getConstraintManager();
70:   }
71: 
72:   StoreManager &getStoreManager() {
73:     return Eng.getStoreManager();
74:   }
75:   const StoreManager &getStoreManager() const { return Eng.getStoreManager(); }
76: 
77:   /// Returns the previous node in the exploded graph, which includes
78:   /// the state of the program before the checker ran. Note, checkers should
79:   /// not retain the node in their state since the nodes might get invalidated.
80:   ExplodedNode *getPredecessor() { return Pred; }
```
- EN: It exposes API surface such as `wasInlined`, `getAnalysisManager`, `getConstraintManager`, `getStoreManager`.
- 中文: 它暴露了 `wasInlined`, `getAnalysisManager`, `getConstraintManager`, `getStoreManager` 等接口。

### Lines 81-120

```cpp
 81:   const ExplodedNode *getPredecessor() const { return Pred; }
 82:   const ProgramPoint getLocation() const { return Location; }
 83:   const ProgramStateRef &getState() const { return Pred->getState(); }
 84: 
 85:   /// Check if the checker changed the state of the execution; ex: added
 86:   /// a new transition or a bug report.
 87:   bool isDifferent() { return Changed; }
 88:   bool isDifferent() const { return Changed; }
 89: 
 90:   /// Returns the number of times the current block has been visited
 91:   /// along the analyzed path.
 92:   unsigned blockCount() const { return Eng.getNumVisitedCurrent(); }
 93: 
 94:   ASTContext &getASTContext() {
 95:     return Eng.getContext();
 96:   }
 97: 
 98:   const ASTContext &getASTContext() const { return Eng.getContext(); }
 99: 
100:   const LangOptions &getLangOpts() const {
101:     return Eng.getContext().getLangOpts();
102:   }
103: 
104:   const LocationContext *getLocationContext() const {
105:     return Pred->getLocationContext();
106:   }
107: 
108:   const StackFrame *getStackFrame() const { return Pred->getStackFrame(); }
109: 
110:   /// Return true if the current LocationContext has no caller context.
111:   bool inTopFrame() const { return getLocationContext()->inTopFrame();  }
112: 
113:   BugReporter &getBugReporter() {
114:     return Eng.getBugReporter();
115:   }
116:   const BugReporter &getBugReporter() const { return Eng.getBugReporter(); }
117: 
118:   const SourceManager &getSourceManager() {
119:     return getBugReporter().getSourceManager();
120:   }
```
- EN: It exposes API surface such as `getPredecessor`, `getLocation`, `getState`, `isDifferent`.
- 中文: 它暴露了 `getPredecessor`, `getLocation`, `getState`, `isDifferent` 等接口。

### Lines 121-160

```cpp
121:   const SourceManager &getSourceManager() const {
122:     return getBugReporter().getSourceManager();
123:   }
124: 
125:   Preprocessor &getPreprocessor() { return getBugReporter().getPreprocessor(); }
126:   const Preprocessor &getPreprocessor() const {
127:     return getBugReporter().getPreprocessor();
128:   }
129: 
130:   SValBuilder &getSValBuilder() {
131:     return Eng.getSValBuilder();
132:   }
133:   const SValBuilder &getSValBuilder() const { return Eng.getSValBuilder(); }
134: 
135:   SymbolManager &getSymbolManager() {
136:     return getSValBuilder().getSymbolManager();
137:   }
138:   const SymbolManager &getSymbolManager() const {
139:     return getSValBuilder().getSymbolManager();
140:   }
141: 
142:   ProgramStateManager &getStateManager() {
143:     return Eng.getStateManager();
144:   }
145:   const ProgramStateManager &getStateManager() const {
146:     return Eng.getStateManager();
147:   }
148: 
149:   AnalysisDeclContext *getCurrentAnalysisDeclContext() const {
150:     return Pred->getLocationContext()->getAnalysisDeclContext();
151:   }
152: 
153:   /// Get the blockID.
154:   unsigned getBlockID() const { return Eng.getCurrBlock()->getBlockID(); }
155: 
156:   /// If the given node corresponds to a PostStore program point,
157:   /// retrieve the location region as it was uttered in the code.
158:   ///
159:   /// This utility can be useful for generating extensive diagnostics, for
160:   /// example, for finding variables that the given symbol was assigned to.
```
- EN: It exposes API surface such as `getSourceManager`, `getBugReporter`, `getPreprocessor`, `getSValBuilder`.
- 中文: 它暴露了 `getSourceManager`, `getBugReporter`, `getPreprocessor`, `getSValBuilder` 等接口。

### Lines 161-200

```cpp
161:   static const MemRegion *getLocationRegionIfPostStore(const ExplodedNode *N) {
162:     ProgramPoint L = N->getLocation();
163:     if (std::optional<PostStore> PSL = L.getAs<PostStore>())
164:       return reinterpret_cast<const MemRegion*>(PSL->getLocationValue());
165:     return nullptr;
166:   }
167: 
168:   /// Get the value of arbitrary expressions at this point in the path.
169:   SVal getSVal(const Expr *E) const { return Pred->getSVal(E); }
170: 
171:   ConstCFGElementRef getCFGElementRef() const { return Eng.getCFGElementRef(); }
172: 
173:   /// Returns true if the value of \p E is greater than or equal to \p
174:   /// Val under unsigned comparison.
175:   bool isGreaterOrEqual(const Expr *E, unsigned long long Val);
176: 
177:   /// Returns true if the value of \p E is negative.
178:   bool isNegative(const Expr *E);
179: 
180:   /// Generates a new transition in the program state graph
181:   /// (ExplodedGraph). Uses the default CheckerContext predecessor node.
182:   ///
183:   /// @param State The state of the generated node. If not specified, the state
184:   ///        will not be changed, but the new node will have the checker's tag.
185:   /// @param Tag The tag is used to uniquely identify the creation site. If no
186:   ///        tag is specified, a default tag, unique to the given checker,
187:   ///        will be used. Tags are used to prevent states generated at
188:   ///        different sites from caching out.
189:   /// NOTE: If the State is unchanged and the Tag is nullptr, this may return a
190:   /// node which is not tagged (instead of using the default tag corresponding
191:   /// to the active checker). This is arguably a bug and should be fixed.
192:   ExplodedNode *addTransition(ProgramStateRef State = nullptr,
193:                               const ProgramPointTag *Tag = nullptr) {
194:     return addTransitionImpl(State ? State : getState(), false, nullptr, Tag);
195:   }
196: 
197:   /// Generates a new transition with the given predecessor.
198:   /// Allows checkers to generate a chain of nodes.
199:   ///
200:   /// @param State The state of the generated node.
```
- EN: It exposes API surface such as `getLocationRegionIfPostStore`, `getLocation`, `getLocationValue`, `getSVal`.
- 中文: 它暴露了 `getLocationRegionIfPostStore`, `getLocation`, `getLocationValue`, `getSVal` 等接口。

### Lines 201-240

```cpp
201:   /// @param Pred The transition will be generated from the specified Pred node
202:   ///             to the newly generated node.
203:   /// @param Tag The tag to uniquely identify the creation site.
204:   /// NOTE: If the State is unchanged and the Tag is nullptr, this may return a
205:   /// node which is not tagged (instead of using the default tag corresponding
206:   /// to the active checker). This is arguably a bug and should be fixed.
207:   ExplodedNode *addTransition(ProgramStateRef State, ExplodedNode *Pred,
208:                               const ProgramPointTag *Tag = nullptr) {
209:     return addTransitionImpl(State, false, Pred, Tag);
210:   }
211: 
212:   /// Generate a sink node. Generating a sink stops exploration of the
213:   /// given path. To create a sink node for the purpose of reporting an error,
214:   /// checkers should use generateErrorNode() instead.
215:   ExplodedNode *generateSink(ProgramStateRef State, ExplodedNode *Pred,
216:                              const ProgramPointTag *Tag = nullptr) {
217:     return addTransitionImpl(State ? State : getState(), true, Pred, Tag);
218:   }
219: 
220:   /// Add a sink node to the current path of execution, halting analysis.
221:   void addSink(ProgramStateRef State = nullptr,
222:                const ProgramPointTag *Tag = nullptr) {
223:     if (!State)
224:       State = getState();
225:     addTransition(State, generateSink(State, getPredecessor()));
226:   }
227: 
228:   /// Generate a transition to a node that will be used to report
229:   /// an error. This node will be a sink. That is, it will stop exploration of
230:   /// the given path.
231:   ///
232:   /// @param State The state of the generated node.
233:   /// @param Tag The tag to uniquely identify the creation site. If null,
234:   ///        the default tag for the checker will be used.
235:   ExplodedNode *generateErrorNode(ProgramStateRef State = nullptr,
236:                                   const ProgramPointTag *Tag = nullptr) {
237:     return generateSink(State, Pred,
238:                        (Tag ? Tag : Location.getTag()));
239:   }
240: 
```
- EN: It exposes API surface such as `addTransitionImpl`, `getState`, `addTransition`, `getTag`.
- 中文: 它暴露了 `addTransitionImpl`, `getState`, `addTransition`, `getTag` 等接口。

### Lines 241-280

```cpp
241:   /// Generate a transition to a node that will be used to report
242:   /// an error. This node will be a sink. That is, it will stop exploration of
243:   /// the given path.
244:   ///
245:   /// @param State The state of the generated node.
246:   /// @param Pred The transition will be generated from the specified Pred node
247:   ///             to the newly generated node.
248:   /// @param Tag The tag to uniquely identify the creation site. If null,
249:   ///        the default tag for the checker will be used.
250:   ExplodedNode *generateErrorNode(ProgramStateRef State,
251:                                   ExplodedNode *Pred,
252:                                   const ProgramPointTag *Tag = nullptr) {
253:     return generateSink(State, Pred,
254:                        (Tag ? Tag : Location.getTag()));
255:   }
256: 
257:   /// Generate a transition to a node that will be used to report
258:   /// an error. This node will not be a sink. That is, exploration will
259:   /// continue along this path.
260:   ///
261:   /// @param State The state of the generated node.
262:   /// @param Tag The tag to uniquely identify the creation site. If null,
263:   ///        the default tag for the checker will be used.
264:   ExplodedNode *
265:   generateNonFatalErrorNode(ProgramStateRef State = nullptr,
266:                             const ProgramPointTag *Tag = nullptr) {
267:     return addTransition(State, (Tag ? Tag : Location.getTag()));
268:   }
269: 
270:   /// Generate a transition to a node that will be used to report
271:   /// an error. This node will not be a sink. That is, exploration will
272:   /// continue along this path.
273:   ///
274:   /// @param State The state of the generated node.
275:   /// @param Pred The transition will be generated from the specified Pred node
276:   ///             to the newly generated node.
277:   /// @param Tag The tag to uniquely identify the creation site. If null,
278:   ///        the default tag for the checker will be used.
279:   ExplodedNode *
280:   generateNonFatalErrorNode(ProgramStateRef State,
```
- EN: It exposes API surface such as `getTag`, `addTransition`.
- 中文: 它暴露了 `getTag`, `addTransition` 等接口。

### Lines 281-320

```cpp
281:                             ExplodedNode *Pred,
282:                             const ProgramPointTag *Tag = nullptr) {
283:     return addTransition(State, Pred, (Tag ? Tag : Location.getTag()));
284:   }
285: 
286:   /// Emit the diagnostics report.
287:   void emitReport(std::unique_ptr<BugReport> R) {
288:     Changed = true;
289:     Eng.getBugReporter().emitReport(std::move(R));
290:   }
291: 
292:   /// Produce a program point tag that displays an additional path note
293:   /// to the user. This is a lightweight alternative to the
294:   /// BugReporterVisitor mechanism: instead of visiting the bug report
295:   /// node-by-node to restore the sequence of events that led to discovering
296:   /// a bug, you can add notes as you add your transitions.
297:   ///
298:   /// @param Cb Callback with 'BugReporterContext &, BugReport &' parameters.
299:   /// @param IsPrunable Whether the note is prunable. It allows BugReporter
300:   ///        to omit the note from the report if it would make the displayed
301:   ///        bug path significantly shorter.
302:   LLVM_ATTRIBUTE_RETURNS_NONNULL
303:   const NoteTag *getNoteTag(NoteTag::Callback &&Cb, bool IsPrunable = false) {
304:     return Eng.getDataTags().make<NoteTag>(std::move(Cb), IsPrunable);
305:   }
306: 
307:   /// A shorthand version of getNoteTag that doesn't require you to accept
308:   /// the 'BugReporterContext' argument when you don't need it.
309:   ///
310:   /// @param Cb Callback only with 'BugReport &' parameter.
311:   /// @param IsPrunable Whether the note is prunable. It allows BugReporter
312:   ///        to omit the note from the report if it would make the displayed
313:   ///        bug path significantly shorter.
314:   const NoteTag
315:   *getNoteTag(std::function<std::string(PathSensitiveBugReport &)> &&Cb,
316:               bool IsPrunable = false) {
317:     return getNoteTag(
318:         [Cb](BugReporterContext &,
319:              PathSensitiveBugReport &BR) { return Cb(BR); },
320:         IsPrunable);
```
- EN: It exposes API surface such as `addTransition`, `emitReport`, `getBugReporter`, `getNoteTag`.
- 中文: 它暴露了 `addTransition`, `emitReport`, `getBugReporter`, `getNoteTag` 等接口。

### Lines 321-360

```cpp
321:   }
322: 
323:   /// A shorthand version of getNoteTag that doesn't require you to accept
324:   /// the arguments when you don't need it.
325:   ///
326:   /// @param Cb Callback without parameters.
327:   /// @param IsPrunable Whether the note is prunable. It allows BugReporter
328:   ///        to omit the note from the report if it would make the displayed
329:   ///        bug path significantly shorter.
330:   const NoteTag *getNoteTag(std::function<std::string()> &&Cb,
331:                             bool IsPrunable = false) {
332:     return getNoteTag([Cb](BugReporterContext &,
333:                            PathSensitiveBugReport &) { return Cb(); },
334:                       IsPrunable);
335:   }
336: 
337:   /// A shorthand version of getNoteTag that accepts a plain note.
338:   ///
339:   /// @param Note The note.
340:   /// @param IsPrunable Whether the note is prunable. It allows BugReporter
341:   ///        to omit the note from the report if it would make the displayed
342:   ///        bug path significantly shorter.
343:   const NoteTag *getNoteTag(StringRef Note, bool IsPrunable = false) {
344:     return getNoteTag(
345:         [Note = std::string(Note)](BugReporterContext &,
346:                PathSensitiveBugReport &) { return Note; },
347:         IsPrunable);
348:   }
349: 
350:   /// A shorthand version of getNoteTag that accepts a lambda with stream for
351:   /// note.
352:   ///
353:   /// @param Cb Callback with 'BugReport &' and 'llvm::raw_ostream &'.
354:   /// @param IsPrunable Whether the note is prunable. It allows BugReporter
355:   ///        to omit the note from the report if it would make the displayed
356:   ///        bug path significantly shorter.
357:   const NoteTag *getNoteTag(
358:       std::function<void(PathSensitiveBugReport &BR, llvm::raw_ostream &OS)> &&Cb,
359:       bool IsPrunable = false) {
360:     return getNoteTag(
```
- EN: It exposes API surface such as `Cb`, `getNoteTag`.
- 中文: 它暴露了 `Cb`, `getNoteTag` 等接口。

### Lines 361-400

```cpp
361:         [Cb](PathSensitiveBugReport &BR) -> std::string {
362:           llvm::SmallString<128> Str;
363:           llvm::raw_svector_ostream OS(Str);
364:           Cb(BR, OS);
365:           return std::string(OS.str());
366:         },
367:         IsPrunable);
368:   }
369: 
370:   /// Returns the word that should be used to refer to the declaration
371:   /// in the report.
372:   StringRef getDeclDescription(const Decl *D);
373: 
374:   /// Get the declaration of the called function (path-sensitive).
375:   const FunctionDecl *getCalleeDecl(const CallExpr *CE) const;
376: 
377:   /// Get the name of the called function (path-sensitive).
378:   StringRef getCalleeName(const FunctionDecl *FunDecl) const;
379: 
380:   /// Get the identifier of the called function (path-sensitive).
381:   const IdentifierInfo *getCalleeIdentifier(const CallExpr *CE) const {
382:     const FunctionDecl *FunDecl = getCalleeDecl(CE);
383:     if (FunDecl)
384:       return FunDecl->getIdentifier();
385:     else
386:       return nullptr;
387:   }
388: 
389:   /// Get the name of the called function (path-sensitive).
390:   StringRef getCalleeName(const CallExpr *CE) const {
391:     const FunctionDecl *FunDecl = getCalleeDecl(CE);
392:     return getCalleeName(FunDecl);
393:   }
394: 
395:   /// Returns true if the given function is an externally-visible function in
396:   /// the top-level namespace, such as \c malloc.
397:   ///
398:   /// If a name is provided, the function must additionally match the given
399:   /// name.
400:   ///
```
- EN: It exposes API surface such as `OS`, `Cb`, `string`, `getDeclDescription`.
- 中文: 它暴露了 `OS`, `Cb`, `string`, `getDeclDescription` 等接口。

### Lines 401-440

```cpp
401:   /// Note that this also accepts functions from the \c std namespace (because
402:   /// headers like <cstdlib> declare them there) and does not check if the
403:   /// function is declared as 'extern "C"' or if it uses C++ name mangling.
404:   static bool isCLibraryFunction(const FunctionDecl *FD,
405:                                  StringRef Name = StringRef());
406: 
407:   /// In builds that use source hardening (-D_FORTIFY_SOURCE), many standard
408:   /// functions are implemented as macros that expand to calls of hardened
409:   /// functions that take additional arguments compared to the "usual"
410:   /// variant and perform additional input validation. For example, a `memcpy`
411:   /// call may expand to `__memcpy_chk()` or `__builtin___memcpy_chk()`.
412:   ///
413:   /// This method returns true if `FD` declares a fortified variant of the
414:   /// standard library function `Name`.
415:   ///
416:   /// NOTE: This method relies on heuristics; extend it if you need to handle a
417:   /// hardened variant that's not yet covered by it.
418:   static bool isHardenedVariantOf(const FunctionDecl *FD, StringRef Name);
419: 
420:   /// Depending on whether the location corresponds to a macro, return
421:   /// either the macro name or the token spelling.
422:   ///
423:   /// This could be useful when checkers' logic depends on whether a function
424:   /// is called with a given macro argument. For example:
425:   ///   s = socket(AF_INET,..)
426:   /// If AF_INET is a macro, the result should be treated as a source of taint.
427:   ///
428:   /// \sa clang::Lexer::getSpelling(), clang::Lexer::getImmediateMacroName().
429:   std::string getMacroNameOrSpelling(SourceLocation &Loc);
430: 
431: private:
432:   ExplodedNode *addTransitionImpl(ProgramStateRef State,
433:                                  bool MarkAsSink,
434:                                  ExplodedNode *P = nullptr,
435:                                  const ProgramPointTag *Tag = nullptr) {
436:     // The analyzer may stop exploring if it sees a state it has previously
437:     // visited ("cache out"). The early return here is a defensive check to
438:     // prevent accidental caching out by checker API clients. Unless there is a
439:     // tag or the client checker has requested that the generated node be
440:     // marked as a sink, we assume that a client requesting a transition to a
```
- EN: It exposes API surface such as `StringRef`, `isHardenedVariantOf`, `getMacroNameOrSpelling`.
- 中文: 它暴露了 `StringRef`, `isHardenedVariantOf`, `getMacroNameOrSpelling` 等接口。

### Lines 441-469

```cpp
441:     // state that is the same as the predecessor state has made a mistake. We
442:     // return the predecessor rather than cache out.
443:     //
444:     // TODO: We could potentially change the return to an assertion to alert
445:     // clients to their mistake, but several checkers (including
446:     // DereferenceChecker, CallAndMessageChecker, and DynamicTypePropagation)
447:     // rely upon the defensive behavior and would need to be updated.
448:     if (!State || (State == Pred->getState() && !Tag && !MarkAsSink))
449:       return Pred;
450: 
451:     Changed = true;
452:     const ProgramPoint &LocalLoc = (Tag ? Location.withTag(Tag) : Location);
453:     if (!P)
454:       P = Pred;
455: 
456:     ExplodedNode *node;
457:     if (MarkAsSink)
458:       node = NB.generateSink(LocalLoc, State, P);
459:     else
460:       node = NB.generateNode(LocalLoc, State, P);
461:     return node;
462:   }
463: };
464: 
465: } // end GR namespace
466: 
467: } // end clang namespace
468: 
469: #endif
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It exposes API surface such as `withTag`, `generateSink`, `generateNode`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它暴露了 `withTag`, `generateSink`, `generateNode` 等接口。

## Key Concepts / 关键概念

- `CheckerContext`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `wasInlined`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getAnalysisManager`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getConstraintManager`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getStoreManager`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getPredecessor`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getLocation`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getState`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/StaticAnalyzer/Core/PathSensitive/ExprEngine.h`, `clang/StaticAnalyzer/Core/PathSensitive/ProgramStateTrait.h`, `optional`
- Forward declarations / 前向声明: None / 无
- Namespace context / 命名空间上下文: `clang`, `ento`
- Macro-style dependencies / 宏式依赖: `NB`
