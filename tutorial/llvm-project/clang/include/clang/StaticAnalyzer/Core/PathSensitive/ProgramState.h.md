# ProgramState.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/StaticAnalyzer/Core/PathSensitive/ProgramState.h`
- Repository: `llvm-project`
- Purpose (EN): ProgramState.h - Path-sensitive "State" for tracking values -*- C++ -*--=// This file defines the state of the program along the analysis path.
- 用途（中文）: 该文件为 StaticAnalyzer::Core::PathSensitive 子系统中的 Program State 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48

```cpp
 1: //== ProgramState.h - Path-sensitive "State" for tracking values -*- C++ -*--=//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file defines the state of the program along the analysis path.
10: //
11: //===----------------------------------------------------------------------===//
12: 
13: #ifndef LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_PROGRAMSTATE_H
14: #define LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_PROGRAMSTATE_H
15: 
16: #include "clang/Basic/LLVM.h"
17: #include "clang/StaticAnalyzer/Core/PathSensitive/ConstraintManager.h"
18: #include "clang/StaticAnalyzer/Core/PathSensitive/DynamicTypeInfo.h"
19: #include "clang/StaticAnalyzer/Core/PathSensitive/Environment.h"
20: #include "clang/StaticAnalyzer/Core/PathSensitive/ProgramState_Fwd.h"
21: #include "clang/StaticAnalyzer/Core/PathSensitive/SValBuilder.h"
22: #include "clang/StaticAnalyzer/Core/PathSensitive/Store.h"
23: #include "llvm/ADT/FoldingSet.h"
24: #include "llvm/ADT/ImmutableMap.h"
25: #include "llvm/Support/Allocator.h"
26: #include <optional>
27: #include <utility>
28: 
29: namespace llvm {
30: class APSInt;
31: }
32: 
33: namespace clang {
34: class ASTContext;
35: 
36: namespace ento {
37: 
38: class AnalysisManager;
39: class CallEvent;
40: class CallEventManager;
41: 
42: typedef std::unique_ptr<ConstraintManager>(*ConstraintManagerCreator)(
43:     ProgramStateManager &, ExprEngine *);
44: typedef std::unique_ptr<StoreManager>(*StoreManagerCreator)(
45:     ProgramStateManager &);
46: 
47: //===----------------------------------------------------------------------===//
48: // ProgramStateTrait - Traits used by the Generic Data Map of a ProgramState.
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/Basic/LLVM.h`, `clang/StaticAnalyzer/Core/PathSensitive/ConstraintManager.h`, `clang/StaticAnalyzer/Core/PathSensitive/DynamicTypeInfo.h` and 9 more. It opens, closes, or documents namespace scope for `llvm`, `clang`, `ento`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/Basic/LLVM.h`, `clang/StaticAnalyzer/Core/PathSensitive/ConstraintManager.h`, `clang/StaticAnalyzer/Core/PathSensitive/DynamicTypeInfo.h` 以及另外 9 项依赖。 它打开、关闭或说明了 `llvm`, `clang`, `ento` 的命名空间作用域。

### Lines 49-96

```cpp
49: //===----------------------------------------------------------------------===//
50: 
51: template <typename T> struct ProgramStateTrait {
52:   typedef typename T::data_type data_type;
53:   static inline void *MakeVoidPtr(data_type D) { return (void*) D; }
54:   static inline data_type MakeData(void *const* P) {
55:     return P ? (data_type) *P : (data_type) 0;
56:   }
57: };
58: 
59: /// \class ProgramState
60: /// ProgramState - This class encapsulates:
61: ///
62: ///    1. A mapping from expressions to values (Environment)
63: ///    2. A mapping from locations to values (Store)
64: ///    3. Constraints on symbolic values (GenericDataMap)
65: ///
66: ///  Together these represent the "abstract state" of a program.
67: ///
68: ///  ProgramState is intended to be used as a functional object; that is,
69: ///  once it is created and made "persistent" in a FoldingSet, its
70: ///  values will never change.
71: class ProgramState : public llvm::FoldingSetNode {
72: public:
73:   typedef llvm::ImmutableMap<const void *, void *> GenericDataMap;
74: 
75: private:
76:   void operator=(const ProgramState& R) = delete;
77: 
78:   friend class ProgramStateManager;
79:   friend class ExplodedGraph;
80:   friend class ExplodedNode;
81: 
82:   ProgramStateManager *stateMgr;
83:   Environment Env;           // Maps a Stmt to its current SVal.
84:   Store store;               // Maps a location to its current value.
85:   GenericDataMap   GDM;      // Custom data stored by a client of this class.
86: 
87:   // A state is infeasible if there is a contradiction among the constraints.
88:   // An infeasible state is represented by a `nullptr`.
89:   // In the sense of `assumeDual`, a state can have two children by adding a
90:   // new constraint and the negation of that new constraint. A parent state is
91:   // over-constrained if both of its children are infeasible. In the
92:   // mathematical sense, it means that the parent is infeasible and we should
93:   // have realized that at the moment when we have created it. However, we
94:   // could not recognize that because of the imperfection of the underlying
95:   // constraint solver. We say it is posteriorly over-constrained because we
96:   // recognize that a parent is infeasible only *after* a new and more specific
```
- EN: Key type declarations here include `ProgramStateTrait`, `ProgramState`, `ProgramStateManager`, `ExplodedGraph`. It exposes API surface such as `MakeVoidPtr`, `MakeData`.
- 中文: 这里的重要类型声明包括 `ProgramStateTrait`, `ProgramState`, `ProgramStateManager`, `ExplodedGraph`。 它暴露了 `MakeVoidPtr`, `MakeData` 等接口。

### Lines 97-144

```cpp
 97:   // constraint and its negation are evaluated.
 98:   //
 99:   // Example:
100:   //
101:   // x * x = 4 and x is in the range [0, 1]
102:   // This is an already infeasible state, but the constraint solver is not
103:   // capable of handling sqrt, thus we don't know it yet.
104:   //
105:   // Then a new constraint `x = 0` is added. At this moment the constraint
106:   // solver re-evaluates the existing constraints and realizes the
107:   // contradiction `0 * 0 = 4`.
108:   // We also evaluate the negated constraint `x != 0`;  the constraint solver
109:   // deduces `x = 1` and then realizes the contradiction `1 * 1 = 4`.
110:   // Both children are infeasible, thus the parent state is marked as
111:   // posteriorly over-constrained. These parents are handled with special care:
112:   // we do not allow transitions to exploded nodes with such states.
113:   bool PosteriorlyOverconstrained = false;
114:   // Make internal constraint solver entities friends so they can access the
115:   // overconstrained-related functions. We want to keep this API inaccessible
116:   // for Checkers.
117:   friend class ConstraintManager;
118:   // The CoreEngine also needs to be a friend to mark nodes as sinks if they
119:   // are generated with a PosteriorlyOverconstrained state.
120:   // FIXME: Perform this check in the relevant methods of `ExplodedGraph` and
121:   // remove this `friend` declaration.
122:   friend class CoreEngine;
123:   bool isPosteriorlyOverconstrained() const {
124:     return PosteriorlyOverconstrained;
125:   }
126:   ProgramStateRef cloneAsPosteriorlyOverconstrained() const;
127: 
128:   unsigned refCount;
129: 
130:   /// makeWithStore - Return a ProgramState with the same values as the current
131:   ///  state with the exception of using the specified Store.
132:   ProgramStateRef makeWithStore(const StoreRef &store) const;
133:   ProgramStateRef makeWithStore(const BindResult &BindRes) const;
134: 
135:   void setStore(const StoreRef &storeRef);
136: 
137: public:
138:   /// This ctor is used when creating the first ProgramState object.
139:   ProgramState(ProgramStateManager *mgr, const Environment& env,
140:           StoreRef st, GenericDataMap gdm);
141: 
142:   /// Copy ctor - We must explicitly define this or else the "Next" ptr
143:   ///  in FoldingSetNode will also get copied.
144:   ProgramState(const ProgramState &RHS);
```
- EN: Key type declarations here include `ConstraintManager`, `CoreEngine`. It exposes API surface such as `isPosteriorlyOverconstrained`, `cloneAsPosteriorlyOverconstrained`, `makeWithStore`, `setStore`.
- 中文: 这里的重要类型声明包括 `ConstraintManager`, `CoreEngine`。 它暴露了 `isPosteriorlyOverconstrained`, `cloneAsPosteriorlyOverconstrained`, `makeWithStore`, `setStore` 等接口。

### Lines 145-192

```cpp
145: 
146:   ~ProgramState();
147: 
148:   int64_t getID() const;
149: 
150:   /// Return the ProgramStateManager associated with this state.
151:   ProgramStateManager &getStateManager() const {
152:     return *stateMgr;
153:   }
154: 
155:   AnalysisManager &getAnalysisManager() const;
156: 
157:   /// Return the ConstraintManager.
158:   ConstraintManager &getConstraintManager() const;
159: 
160:   /// getEnvironment - Return the environment associated with this state.
161:   ///  The environment is the mapping from expressions to values.
162:   const Environment& getEnvironment() const { return Env; }
163: 
164:   /// Return the store associated with this state.  The store
165:   ///  is a mapping from locations to values.
166:   Store getStore() const { return store; }
167: 
168:   /// getGDM - Return the generic data map associated with this state.
169:   GenericDataMap getGDM() const { return GDM; }
170: 
171:   void setGDM(GenericDataMap gdm) { GDM = gdm; }
172: 
173:   /// Profile - Profile the contents of a ProgramState object for use in a
174:   ///  FoldingSet.  Two ProgramState objects are considered equal if they
175:   ///  have the same Environment, Store, and GenericDataMap.
176:   static void Profile(llvm::FoldingSetNodeID& ID, const ProgramState *V) {
177:     V->Env.Profile(ID);
178:     ID.AddPointer(V->store);
179:     V->GDM.Profile(ID);
180:     ID.AddBoolean(V->PosteriorlyOverconstrained);
181:   }
182: 
183:   /// Profile - Used to profile the contents of this object for inclusion
184:   ///  in a FoldingSet.
185:   void Profile(llvm::FoldingSetNodeID& ID) const {
186:     Profile(ID, this);
187:   }
188: 
189:   BasicValueFactory &getBasicVals() const;
190:   SymbolManager &getSymbolManager() const;
191: 
192:   //==---------------------------------------------------------------------==//
```
- EN: It exposes API surface such as `~ProgramState`, `getID`, `getStateManager`, `getAnalysisManager`.
- 中文: 它暴露了 `~ProgramState`, `getID`, `getStateManager`, `getAnalysisManager` 等接口。

### Lines 193-240

```cpp
193:   // Constraints on values.
194:   //==---------------------------------------------------------------------==//
195:   //
196:   // Each ProgramState records constraints on symbolic values.  These constraints
197:   // are managed using the ConstraintManager associated with a ProgramStateManager.
198:   // As constraints gradually accrue on symbolic values, added constraints
199:   // may conflict and indicate that a state is infeasible (as no real values
200:   // could satisfy all the constraints).  This is the principal mechanism
201:   // for modeling path-sensitivity in ExprEngine/ProgramState.
202:   //
203:   // Various "assume" methods form the interface for adding constraints to
204:   // symbolic values.  A call to 'assume' indicates an assumption being placed
205:   // on one or symbolic values.  'assume' methods take the following inputs:
206:   //
207:   //  (1) A ProgramState object representing the current state.
208:   //
209:   //  (2) The assumed constraint (which is specific to a given "assume" method).
210:   //
211:   //  (3) A binary value "Assumption" that indicates whether the constraint is
212:   //      assumed to be true or false.
213:   //
214:   // The output of "assume*" is a new ProgramState object with the added constraints.
215:   // If no new state is feasible, NULL is returned.
216:   //
217: 
218:   /// Assumes that the value of \p cond is zero (if \p assumption is "false")
219:   /// or non-zero (if \p assumption is "true").
220:   ///
221:   /// This returns a new state with the added constraint on \p cond.
222:   /// If no new state is feasible, NULL is returned.
223:   [[nodiscard]] ProgramStateRef assume(DefinedOrUnknownSVal cond,
224:                                        bool assumption) const;
225: 
226:   /// Assumes both "true" and "false" for \p cond, and returns both
227:   /// corresponding states (respectively).
228:   ///
229:   /// This is more efficient than calling assume() twice. Note that one (but not
230:   /// both) of the returned states may be NULL.
231:   [[nodiscard]] std::pair<ProgramStateRef, ProgramStateRef>
232:   assume(DefinedOrUnknownSVal cond) const;
233: 
234:   [[nodiscard]] std::pair<ProgramStateRef, ProgramStateRef>
235:   assumeInBoundDual(DefinedOrUnknownSVal idx, DefinedOrUnknownSVal upperBound,
236:                     QualType IndexType = QualType()) const;
237: 
238:   [[nodiscard]] ProgramStateRef
239:   assumeInBound(DefinedOrUnknownSVal idx, DefinedOrUnknownSVal upperBound,
240:                 bool assumption, QualType IndexType = QualType()) const;
```
- EN: It exposes API surface such as `assume`, `QualType`.
- 中文: 它暴露了 `assume`, `QualType` 等接口。

### Lines 241-288

```cpp
241: 
242:   /// Assumes that the value of \p Val is bounded with [\p From; \p To]
243:   /// (if \p assumption is "true") or it is fully out of this range
244:   /// (if \p assumption is "false").
245:   ///
246:   /// This returns a new state with the added constraint on \p cond.
247:   /// If no new state is feasible, NULL is returned.
248:   [[nodiscard]] ProgramStateRef assumeInclusiveRange(DefinedOrUnknownSVal Val,
249:                                                      const llvm::APSInt &From,
250:                                                      const llvm::APSInt &To,
251:                                                      bool assumption) const;
252: 
253:   /// Assumes given range both "true" and "false" for \p Val, and returns both
254:   /// corresponding states (respectively).
255:   ///
256:   /// This is more efficient than calling assume() twice. Note that one (but not
257:   /// both) of the returned states may be NULL.
258:   [[nodiscard]] std::pair<ProgramStateRef, ProgramStateRef>
259:   assumeInclusiveRange(DefinedOrUnknownSVal Val, const llvm::APSInt &From,
260:                        const llvm::APSInt &To) const;
261: 
262:   /// Check if the given SVal is not constrained to zero and is not
263:   ///        a zero constant.
264:   ConditionTruthVal isNonNull(SVal V) const;
265: 
266:   /// Check if the given SVal is constrained to zero or is a zero
267:   ///        constant.
268:   ConditionTruthVal isNull(SVal V) const;
269: 
270:   /// \return Whether values \p Lhs and \p Rhs are equal.
271:   ConditionTruthVal areEqual(SVal Lhs, SVal Rhs) const;
272: 
273:   /// Utility method for getting regions.
274:   LLVM_ATTRIBUTE_RETURNS_NONNULL
275:   const VarRegion* getRegion(const VarDecl *D, const LocationContext *LC) const;
276: 
277:   //==---------------------------------------------------------------------==//
278:   // Binding and retrieving values to/from the environment and symbolic store.
279:   //==---------------------------------------------------------------------==//
280: 
281:   /// Create a new state by binding the value \p V to the expression \p E in
282:   /// the state's environment.
283:   [[nodiscard]] ProgramStateRef BindExpr(const Expr *E,
284:                                          const LocationContext *LCtx, SVal V,
285:                                          bool Invalidate = true) const;
286: 
287:   [[nodiscard]] ProgramStateRef bindLoc(Loc location, SVal V,
288:                                         const LocationContext *LCtx,
```
- EN: It exposes API surface such as `isNonNull`, `isNull`, `areEqual`, `getRegion`.
- 中文: 它暴露了 `isNonNull`, `isNull`, `areEqual`, `getRegion` 等接口。

### Lines 289-336

```cpp
289:                                         bool notifyChanges = true) const;
290: 
291:   [[nodiscard]] ProgramStateRef bindLoc(SVal location, SVal V,
292:                                         const LocationContext *LCtx) const;
293: 
294:   /// Initializes the region of memory represented by \p loc with an initial
295:   /// value. Once initialized, all values loaded from any sub-regions of that
296:   /// region will be equal to \p V, unless overwritten later by the program.
297:   /// This method should not be used on regions that are already initialized.
298:   /// If you need to indicate that memory contents have suddenly become unknown
299:   /// within a certain region of memory, consider invalidateRegions().
300:   [[nodiscard]] ProgramStateRef
301:   bindDefaultInitial(SVal loc, SVal V, const LocationContext *LCtx) const;
302: 
303:   /// Performs C++ zero-initialization procedure on the region of memory
304:   /// represented by \p loc.
305:   [[nodiscard]] ProgramStateRef
306:   bindDefaultZero(SVal loc, const LocationContext *LCtx) const;
307: 
308:   [[nodiscard]] ProgramStateRef killBinding(Loc LV) const;
309: 
310:   /// Returns the state with bindings for the given regions cleared from the
311:   /// store. If \p Call is non-null, also invalidates global regions (but if
312:   /// \p Call is from a system header, then this is limited to globals declared
313:   /// in system headers).
314:   ///
315:   /// This calls the lower-level method \c StoreManager::invalidateRegions to
316:   /// do the actual invalidation, then calls the checker callbacks which should
317:   /// be triggered by this event.
318:   ///
319:   /// \param Regions the set of regions to be invalidated.
320:   /// \param Elem The CFG Element that caused the invalidation.
321:   /// \param BlockCount The number of times the current basic block has been
322:   ///        visited.
323:   /// \param CausesPointerEscape the flag is set to true when the invalidation
324:   ///        entails escape of a symbol (representing a pointer). For example,
325:   ///        due to it being passed as an argument in a call.
326:   /// \param IS the set of invalidated symbols.
327:   /// \param Call if non-null, the invalidated regions represent parameters to
328:   ///        the call and should be considered directly invalidated.
329:   /// \param ITraits information about special handling for particular regions
330:   ///        or symbols.
331:   [[nodiscard]] ProgramStateRef
332:   invalidateRegions(ArrayRef<const MemRegion *> Regions,
333:                     ConstCFGElementRef Elem, unsigned BlockCount,
334:                     const LocationContext *LCtx, bool CausesPointerEscape,
335:                     InvalidatedSymbols *IS = nullptr,
336:                     const CallEvent *Call = nullptr,
```
- EN: It exposes API surface such as `bindDefaultInitial`, `bindDefaultZero`, `killBinding`.
- 中文: 它暴露了 `bindDefaultInitial`, `bindDefaultZero`, `killBinding` 等接口。

### Lines 337-384

```cpp
337:                     RegionAndSymbolInvalidationTraits *ITraits = nullptr) const;
338: 
339:   [[nodiscard]] ProgramStateRef
340:   invalidateRegions(ArrayRef<SVal> Values, ConstCFGElementRef Elem,
341:                     unsigned BlockCount, const LocationContext *LCtx,
342:                     bool CausesPointerEscape, InvalidatedSymbols *IS = nullptr,
343:                     const CallEvent *Call = nullptr,
344:                     RegionAndSymbolInvalidationTraits *ITraits = nullptr) const;
345: 
346:   /// enterStackFrame - Returns the state for entry to the given stack frame,
347:   ///  preserving the current state.
348:   [[nodiscard]] ProgramStateRef
349:   enterStackFrame(const CallEvent &Call, const StackFrame *CalleeSF) const;
350: 
351:   /// Return the value of 'self' if available in the given context.
352:   SVal getSelfSVal(const LocationContext *LC) const;
353: 
354:   /// Get the lvalue for a base class object reference.
355:   Loc getLValue(const CXXBaseSpecifier &BaseSpec, const SubRegion *Super) const;
356: 
357:   /// Get the lvalue for a base class object reference.
358:   Loc getLValue(const CXXRecordDecl *BaseClass, const SubRegion *Super,
359:                 bool IsVirtual) const;
360: 
361:   /// Get the lvalue for a variable reference.
362:   Loc getLValue(const VarDecl *D, const LocationContext *LC) const;
363: 
364:   Loc getLValue(const CompoundLiteralExpr *literal,
365:                 const LocationContext *LC) const;
366: 
367:   /// Get the lvalue for an ivar reference.
368:   SVal getLValue(const ObjCIvarDecl *decl, SVal base) const;
369: 
370:   /// Get the lvalue for a field reference.
371:   SVal getLValue(const FieldDecl *decl, SVal Base) const;
372: 
373:   /// Get the lvalue for an indirect field reference.
374:   SVal getLValue(const IndirectFieldDecl *decl, SVal Base) const;
375: 
376:   /// Get the lvalue for an array index.
377:   SVal getLValue(QualType ElementType, SVal Idx, SVal Base) const;
378: 
379:   /// Returns the SVal bound to the expression \p E in the state's environment.
380:   SVal getSVal(const Expr *E, const LocationContext *LCtx) const;
381: 
382:   SVal getSValAsScalarOrLoc(const Expr *E, const LocationContext *LCtx) const;
383: 
384:   /// Return the value bound to the specified location.
```
- EN: It exposes API surface such as `enterStackFrame`, `getSelfSVal`, `getLValue`, `getSVal`.
- 中文: 它暴露了 `enterStackFrame`, `getSelfSVal`, `getLValue`, `getSVal` 等接口。

### Lines 385-432

```cpp
385:   /// Returns UnknownVal() if none found.
386:   SVal getSVal(Loc LV, QualType T = QualType()) const;
387: 
388:   /// Returns the "raw" SVal bound to LV before any value simplification.
389:   SVal getRawSVal(Loc LV, QualType T= QualType()) const;
390: 
391:   /// Return the value bound to the specified location.
392:   /// Returns UnknownVal() if none found.
393:   SVal getSVal(const MemRegion* R, QualType T = QualType()) const;
394: 
395:   /// Return the value bound to the specified location, assuming
396:   /// that the value is a scalar integer or an enumeration or a pointer.
397:   /// Returns UnknownVal() if none found or the region is not known to hold
398:   /// a value of such type.
399:   SVal getSValAsScalarOrLoc(const MemRegion *R) const;
400: 
401:   using region_iterator = const MemRegion **;
402: 
403:   /// Visits the symbols reachable from the given SVal using the provided
404:   /// SymbolVisitor.
405:   ///
406:   /// This is a convenience API. Consider using ScanReachableSymbols class
407:   /// directly when making multiple scans on the same state with the same
408:   /// visitor to avoid repeated initialization cost.
409:   /// \sa ScanReachableSymbols
410:   bool scanReachableSymbols(SVal val, SymbolVisitor& visitor) const;
411: 
412:   /// Visits the symbols reachable from the regions in the given
413:   /// MemRegions range using the provided SymbolVisitor.
414:   bool scanReachableSymbols(llvm::iterator_range<region_iterator> Reachable,
415:                             SymbolVisitor &visitor) const;
416: 
417:   template <typename CB> CB scanReachableSymbols(SVal val) const;
418:   template <typename CB> CB
419:   scanReachableSymbols(llvm::iterator_range<region_iterator> Reachable) const;
420: 
421:   //==---------------------------------------------------------------------==//
422:   // Accessing the Generic Data Map (GDM).
423:   //==---------------------------------------------------------------------==//
424: 
425:   void *const *FindGDM(const void *K) const;
426: 
427:   template <typename T>
428:   [[nodiscard]] ProgramStateRef
429:   add(typename ProgramStateTrait<T>::key_type K) const;
430: 
431:   template <typename T>
432:   typename ProgramStateTrait<T>::data_type
```
- EN: It defines convenient aliases such as `region_iterator`. It exposes API surface such as `getSVal`, `getRawSVal`, `getSValAsScalarOrLoc`, `scanReachableSymbols`.
- 中文: 它定义了 `region_iterator` 等便捷别名。 它暴露了 `getSVal`, `getRawSVal`, `getSValAsScalarOrLoc`, `scanReachableSymbols` 等接口。

### Lines 433-480

```cpp
433:   get() const {
434:     return ProgramStateTrait<T>::MakeData(FindGDM(ProgramStateTrait<T>::GDMIndex()));
435:   }
436: 
437:   template<typename T>
438:   typename ProgramStateTrait<T>::lookup_type
439:   get(typename ProgramStateTrait<T>::key_type key) const {
440:     void *const* d = FindGDM(ProgramStateTrait<T>::GDMIndex());
441:     return ProgramStateTrait<T>::Lookup(ProgramStateTrait<T>::MakeData(d), key);
442:   }
443: 
444:   template <typename T>
445:   typename ProgramStateTrait<T>::context_type get_context() const;
446: 
447:   template <typename T>
448:   [[nodiscard]] ProgramStateRef
449:   remove(typename ProgramStateTrait<T>::key_type K) const;
450: 
451:   template <typename T>
452:   [[nodiscard]] ProgramStateRef
453:   remove(typename ProgramStateTrait<T>::key_type K,
454:          typename ProgramStateTrait<T>::context_type C) const;
455: 
456:   template <typename T> [[nodiscard]] ProgramStateRef remove() const;
457: 
458:   template <typename T>
459:   [[nodiscard]] ProgramStateRef
460:   set(typename ProgramStateTrait<T>::data_type D) const;
461: 
462:   template <typename T>
463:   [[nodiscard]] ProgramStateRef
464:   set(typename ProgramStateTrait<T>::key_type K,
465:       typename ProgramStateTrait<T>::value_type E) const;
466: 
467:   template <typename T>
468:   [[nodiscard]] ProgramStateRef
469:   set(typename ProgramStateTrait<T>::key_type K,
470:       typename ProgramStateTrait<T>::value_type E,
471:       typename ProgramStateTrait<T>::context_type C) const;
472: 
473:   template<typename T>
474:   bool contains(typename ProgramStateTrait<T>::key_type key) const {
475:     void *const* d = FindGDM(ProgramStateTrait<T>::GDMIndex());
476:     return ProgramStateTrait<T>::Contains(ProgramStateTrait<T>::MakeData(d), key);
477:   }
478: 
479:   // Pretty-printing.
480:   void printJson(raw_ostream &Out, const LocationContext *LCtx = nullptr,
```
- EN: It exposes API surface such as `get`, `MakeData`, `FindGDM`, `Lookup`.
- 中文: 它暴露了 `get`, `MakeData`, `FindGDM`, `Lookup` 等接口。

### Lines 481-528

```cpp
481:                  const char *NL = "\n", unsigned int Space = 0,
482:                  bool IsDot = false) const;
483: 
484:   void printDOT(raw_ostream &Out, const LocationContext *LCtx = nullptr,
485:                 unsigned int Space = 0) const;
486: 
487:   void dump() const;
488: 
489: private:
490:   friend void ProgramStateRetain(const ProgramState *state);
491:   friend void ProgramStateRelease(const ProgramState *state);
492: 
493:   SVal desugarReference(SVal Val) const;
494:   SVal wrapSymbolicRegion(SVal Base) const;
495: };
496: 
497: //===----------------------------------------------------------------------===//
498: // ProgramStateManager - Factory object for ProgramStates.
499: //===----------------------------------------------------------------------===//
500: 
501: class ProgramStateManager {
502:   friend class ProgramState;
503:   friend void ProgramStateRelease(const ProgramState *state);
504: private:
505:   /// Eng - The ExprEngine that owns this state manager.
506:   ExprEngine *Eng; /* Can be null. */
507: 
508:   EnvironmentManager                   EnvMgr;
509:   std::unique_ptr<StoreManager>        StoreMgr;
510:   std::unique_ptr<ConstraintManager>   ConstraintMgr;
511: 
512:   ProgramState::GenericDataMap::Factory     GDMFactory;
513: 
514:   typedef llvm::DenseMap<const void *, std::pair<void *, void (*)(void *)>>
515:       GDMContextsTy;
516:   GDMContextsTy GDMContexts;
517: 
518:   /// StateSet - FoldingSet containing all the states created for analyzing
519:   ///  a particular function.  This is used to unique states.
520:   llvm::FoldingSet<ProgramState> StateSet;
521: 
522:   /// Object that manages the data for all created SVals.
523:   std::unique_ptr<SValBuilder> svalBuilder;
524: 
525:   /// Manages memory for created CallEvents.
526:   std::unique_ptr<CallEventManager> CallEventMgr;
527: 
528:   /// A BumpPtrAllocator to allocate states.
```
- EN: Key type declarations here include `ProgramStateManager`, `ProgramState`. It exposes API surface such as `dump`, `ProgramStateRetain`, `ProgramStateRelease`, `desugarReference`.
- 中文: 这里的重要类型声明包括 `ProgramStateManager`, `ProgramState`。 它暴露了 `dump`, `ProgramStateRetain`, `ProgramStateRelease`, `desugarReference` 等接口。

### Lines 529-576

```cpp
529:   llvm::BumpPtrAllocator &Alloc;
530: 
531:   /// A vector of ProgramStates that we can reuse.
532:   std::vector<ProgramState *> freeStates;
533: 
534: public:
535:   ProgramStateManager(ASTContext &Ctx,
536:                  StoreManagerCreator CreateStoreManager,
537:                  ConstraintManagerCreator CreateConstraintManager,
538:                  llvm::BumpPtrAllocator& alloc,
539:                  ExprEngine *expreng);
540: 
541:   ~ProgramStateManager();
542: 
543:   ProgramStateRef getInitialState(const LocationContext *InitLoc);
544: 
545:   ASTContext &getContext() { return svalBuilder->getContext(); }
546:   const ASTContext &getContext() const { return svalBuilder->getContext(); }
547: 
548:   BasicValueFactory &getBasicVals() {
549:     return svalBuilder->getBasicValueFactory();
550:   }
551: 
552:   SValBuilder &getSValBuilder() {
553:     return *svalBuilder;
554:   }
555: 
556:   const SValBuilder &getSValBuilder() const {
557:     return *svalBuilder;
558:   }
559: 
560:   SymbolManager &getSymbolManager() {
561:     return svalBuilder->getSymbolManager();
562:   }
563:   const SymbolManager &getSymbolManager() const {
564:     return svalBuilder->getSymbolManager();
565:   }
566: 
567:   llvm::BumpPtrAllocator& getAllocator() { return Alloc; }
568: 
569:   MemRegionManager& getRegionManager() {
570:     return svalBuilder->getRegionManager();
571:   }
572:   const MemRegionManager &getRegionManager() const {
573:     return svalBuilder->getRegionManager();
574:   }
575: 
576:   CallEventManager &getCallEventManager() { return *CallEventMgr; }
```
- EN: It exposes API surface such as `~ProgramStateManager`, `getInitialState`, `getContext`, `getBasicVals`.
- 中文: 它暴露了 `~ProgramStateManager`, `getInitialState`, `getContext`, `getBasicVals` 等接口。

### Lines 577-624

```cpp
577: 
578:   StoreManager &getStoreManager() { return *StoreMgr; }
579:   const StoreManager &getStoreManager() const { return *StoreMgr; }
580:   ConstraintManager &getConstraintManager() { return *ConstraintMgr; }
581:   const ConstraintManager &getConstraintManager() const {
582:     return *ConstraintMgr;
583:   }
584:   ExprEngine &getOwningEngine() { return *Eng; }
585: 
586:   ProgramStateRef removeDeadBindingsFromEnvironmentAndStore(
587:       ProgramStateRef St, const StackFrame *SF, SymbolReaper &SymReaper);
588: 
589: public:
590: 
591:   SVal ArrayToPointer(Loc Array, QualType ElementTy) {
592:     return StoreMgr->ArrayToPointer(Array, ElementTy);
593:   }
594: 
595:   // Methods that manipulate the GDM.
596:   ProgramStateRef addGDM(ProgramStateRef St, const void *Key, void *Data);
597:   ProgramStateRef removeGDM(ProgramStateRef state, const void *Key);
598: 
599:   // Methods that query & manipulate the Store.
600: 
601:   void iterBindings(ProgramStateRef state, StoreManager::BindingsHandler& F) {
602:     StoreMgr->iterBindings(state->getStore(), F);
603:   }
604: 
605:   ProgramStateRef getPersistentState(ProgramState &Impl);
606:   ProgramStateRef getPersistentStateWithGDM(ProgramStateRef FromState,
607:                                            ProgramStateRef GDMState);
608: 
609:   bool haveEqualConstraints(ProgramStateRef S1, ProgramStateRef S2) const {
610:     return ConstraintMgr->haveEqualConstraints(S1, S2);
611:   }
612: 
613:   bool haveEqualEnvironments(ProgramStateRef S1, ProgramStateRef S2) const {
614:     return S1->Env == S2->Env;
615:   }
616: 
617:   bool haveEqualStores(ProgramStateRef S1, ProgramStateRef S2) const {
618:     return S1->store == S2->store;
619:   }
620: 
621:   //==---------------------------------------------------------------------==//
622:   // Generic Data Map methods.
623:   //==---------------------------------------------------------------------==//
624:   //
```
- EN: It exposes API surface such as `getStoreManager`, `getConstraintManager`, `getOwningEngine`, `ArrayToPointer`.
- 中文: 它暴露了 `getStoreManager`, `getConstraintManager`, `getOwningEngine`, `ArrayToPointer` 等接口。

### Lines 625-672

```cpp
625:   // ProgramStateManager and ProgramState support a "generic data map" that allows
626:   // different clients of ProgramState objects to embed arbitrary data within a
627:   // ProgramState object.  The generic data map is essentially an immutable map
628:   // from a "tag" (that acts as the "key" for a client) and opaque values.
629:   // Tags/keys and values are simply void* values.  The typical way that clients
630:   // generate unique tags are by taking the address of a static variable.
631:   // Clients are responsible for ensuring that data values referred to by a
632:   // the data pointer are immutable (and thus are essentially purely functional
633:   // data).
634:   //
635:   // The templated methods below use the ProgramStateTrait<T> class
636:   // to resolve keys into the GDM and to return data values to clients.
637:   //
638: 
639:   // Trait based GDM dispatch.
640:   template <typename T>
641:   ProgramStateRef set(ProgramStateRef st, typename ProgramStateTrait<T>::data_type D) {
642:     return addGDM(st, ProgramStateTrait<T>::GDMIndex(),
643:                   ProgramStateTrait<T>::MakeVoidPtr(D));
644:   }
645: 
646:   template<typename T>
647:   ProgramStateRef set(ProgramStateRef st,
648:                      typename ProgramStateTrait<T>::key_type K,
649:                      typename ProgramStateTrait<T>::value_type V,
650:                      typename ProgramStateTrait<T>::context_type C) {
651: 
652:     return addGDM(st, ProgramStateTrait<T>::GDMIndex(),
653:      ProgramStateTrait<T>::MakeVoidPtr(ProgramStateTrait<T>::Set(st->get<T>(), K, V, C)));
654:   }
655: 
656:   template <typename T>
657:   ProgramStateRef add(ProgramStateRef st,
658:                      typename ProgramStateTrait<T>::key_type K,
659:                      typename ProgramStateTrait<T>::context_type C) {
660:     return addGDM(st, ProgramStateTrait<T>::GDMIndex(),
661:         ProgramStateTrait<T>::MakeVoidPtr(ProgramStateTrait<T>::Add(st->get<T>(), K, C)));
662:   }
663: 
664:   template <typename T>
665:   ProgramStateRef remove(ProgramStateRef st,
666:                         typename ProgramStateTrait<T>::key_type K,
667:                         typename ProgramStateTrait<T>::context_type C) {
668: 
669:     return addGDM(st, ProgramStateTrait<T>::GDMIndex(),
670:      ProgramStateTrait<T>::MakeVoidPtr(ProgramStateTrait<T>::Remove(st->get<T>(), K, C)));
671:   }
672: 
```
- EN: It exposes API surface such as `set`, `MakeVoidPtr`.
- 中文: 它暴露了 `set`, `MakeVoidPtr` 等接口。

### Lines 673-720

```cpp
673:   template <typename T>
674:   ProgramStateRef remove(ProgramStateRef st) {
675:     return removeGDM(st, ProgramStateTrait<T>::GDMIndex());
676:   }
677: 
678:   void *FindGDMContext(const void *index,
679:                        void *(*CreateContext)(llvm::BumpPtrAllocator &),
680:                        void (*DeleteContext)(void *));
681: 
682:   template <typename T>
683:   typename ProgramStateTrait<T>::context_type get_context() {
684:     void *p = FindGDMContext(ProgramStateTrait<T>::GDMIndex(),
685:                              ProgramStateTrait<T>::CreateContext,
686:                              ProgramStateTrait<T>::DeleteContext);
687: 
688:     return ProgramStateTrait<T>::MakeContext(p);
689:   }
690: };
691: 
692: 
693: //===----------------------------------------------------------------------===//
694: // Out-of-line method definitions for ProgramState.
695: //===----------------------------------------------------------------------===//
696: 
697: inline ConstraintManager &ProgramState::getConstraintManager() const {
698:   return stateMgr->getConstraintManager();
699: }
700: 
701: inline const VarRegion* ProgramState::getRegion(const VarDecl *D,
702:                                                 const LocationContext *LC) const
703: {
704:   return getStateManager().getRegionManager().getVarRegion(D, LC);
705: }
706: 
707: inline ProgramStateRef ProgramState::assume(DefinedOrUnknownSVal Cond,
708:                                       bool Assumption) const {
709:   if (Cond.isUnknown())
710:     return this;
711: 
712:   return getStateManager().ConstraintMgr
713:       ->assume(this, Cond.castAs<DefinedSVal>(), Assumption);
714: }
715: 
716: inline std::pair<ProgramStateRef , ProgramStateRef >
717: ProgramState::assume(DefinedOrUnknownSVal Cond) const {
718:   if (Cond.isUnknown())
719:     return std::make_pair(this, this);
720: 
```
- EN: It exposes API surface such as `remove`, `removeGDM`, `void`, `get_context`.
- 中文: 它暴露了 `remove`, `removeGDM`, `void`, `get_context` 等接口。

### Lines 721-768

```cpp
721:   return getStateManager().ConstraintMgr
722:       ->assumeDual(this, Cond.castAs<DefinedSVal>());
723: }
724: 
725: inline ProgramStateRef ProgramState::assumeInclusiveRange(
726:     DefinedOrUnknownSVal Val, const llvm::APSInt &From, const llvm::APSInt &To,
727:     bool Assumption) const {
728:   if (Val.isUnknown())
729:     return this;
730: 
731:   assert(isa<NonLoc>(Val) && "Only NonLocs are supported!");
732: 
733:   return getStateManager().ConstraintMgr->assumeInclusiveRange(
734:       this, Val.castAs<NonLoc>(), From, To, Assumption);
735: }
736: 
737: inline std::pair<ProgramStateRef, ProgramStateRef>
738: ProgramState::assumeInclusiveRange(DefinedOrUnknownSVal Val,
739:                                    const llvm::APSInt &From,
740:                                    const llvm::APSInt &To) const {
741:   if (Val.isUnknown())
742:     return std::make_pair(this, this);
743: 
744:   assert(isa<NonLoc>(Val) && "Only NonLocs are supported!");
745: 
746:   return getStateManager().ConstraintMgr->assumeInclusiveRangeDual(
747:       this, Val.castAs<NonLoc>(), From, To);
748: }
749: 
750: inline ProgramStateRef ProgramState::bindLoc(SVal LV, SVal V, const LocationContext *LCtx) const {
751:   if (std::optional<Loc> L = LV.getAs<Loc>())
752:     return bindLoc(*L, V, LCtx);
753:   return this;
754: }
755: 
756: inline Loc ProgramState::getLValue(const CXXBaseSpecifier &BaseSpec,
757:                                    const SubRegion *Super) const {
758:   const auto *Base = BaseSpec.getType()->getAsCXXRecordDecl();
759:   return loc::MemRegionVal(
760:            getStateManager().getRegionManager().getCXXBaseObjectRegion(
761:                                             Base, Super, BaseSpec.isVirtual()));
762: }
763: 
764: inline Loc ProgramState::getLValue(const CXXRecordDecl *BaseClass,
765:                                    const SubRegion *Super,
766:                                    bool IsVirtual) const {
767:   return loc::MemRegionVal(
768:            getStateManager().getRegionManager().getCXXBaseObjectRegion(
```
- EN: It exposes API surface such as `assumeDual`, `assert`, `castAs`, `make_pair`.
- 中文: 它暴露了 `assumeDual`, `assert`, `castAs`, `make_pair` 等接口。

### Lines 769-816

```cpp
769:                                                   BaseClass, Super, IsVirtual));
770: }
771: 
772: inline Loc ProgramState::getLValue(const VarDecl *VD,
773:                                const LocationContext *LC) const {
774:   return getStateManager().StoreMgr->getLValueVar(VD, LC);
775: }
776: 
777: inline Loc ProgramState::getLValue(const CompoundLiteralExpr *literal,
778:                                const LocationContext *LC) const {
779:   return getStateManager().StoreMgr->getLValueCompoundLiteral(literal, LC);
780: }
781: 
782: inline SVal ProgramState::getLValue(const ObjCIvarDecl *D, SVal Base) const {
783:   return getStateManager().StoreMgr->getLValueIvar(D, Base);
784: }
785: 
786: inline SVal ProgramState::getLValue(QualType ElementType, SVal Idx, SVal Base) const{
787:   if (std::optional<NonLoc> N = Idx.getAs<NonLoc>())
788:     return getStateManager().StoreMgr->getLValueElement(ElementType, *N, Base);
789:   return UnknownVal();
790: }
791: 
792: inline SVal ProgramState::getSVal(const Expr *E,
793:                                   const LocationContext *LCtx) const {
794:   return Env.getSVal(EnvironmentEntry(E, LCtx), *getStateManager().svalBuilder);
795: }
796: 
797: inline SVal
798: ProgramState::getSValAsScalarOrLoc(const Expr *E,
799:                                    const LocationContext *LCtx) const {
800:   QualType T = E->getType();
801:   if (E->isGLValue() || Loc::isLocType(T) || T->isIntegralOrEnumerationType())
802:     return getSVal(E, LCtx);
803:   return UnknownVal();
804: }
805: 
806: inline SVal ProgramState::getRawSVal(Loc LV, QualType T) const {
807:   return getStateManager().StoreMgr->getBinding(getStore(), LV, T);
808: }
809: 
810: inline SVal ProgramState::getSVal(const MemRegion* R, QualType T) const {
811:   return getStateManager().StoreMgr->getBinding(getStore(),
812:                                                 loc::MemRegionVal(R),
813:                                                 T);
814: }
815: 
816: inline BasicValueFactory &ProgramState::getBasicVals() const {
```
- EN: It exposes API surface such as `getStateManager`, `getLValue`, `UnknownVal`, `getSVal`.
- 中文: 它暴露了 `getStateManager`, `getLValue`, `UnknownVal`, `getSVal` 等接口。

### Lines 817-864

```cpp
817:   return getStateManager().getBasicVals();
818: }
819: 
820: inline SymbolManager &ProgramState::getSymbolManager() const {
821:   return getStateManager().getSymbolManager();
822: }
823: 
824: template<typename T>
825: ProgramStateRef ProgramState::add(typename ProgramStateTrait<T>::key_type K) const {
826:   return getStateManager().add<T>(this, K, get_context<T>());
827: }
828: 
829: template <typename T>
830: typename ProgramStateTrait<T>::context_type ProgramState::get_context() const {
831:   return getStateManager().get_context<T>();
832: }
833: 
834: template<typename T>
835: ProgramStateRef ProgramState::remove(typename ProgramStateTrait<T>::key_type K) const {
836:   return getStateManager().remove<T>(this, K, get_context<T>());
837: }
838: 
839: template<typename T>
840: ProgramStateRef ProgramState::remove(typename ProgramStateTrait<T>::key_type K,
841:                                typename ProgramStateTrait<T>::context_type C) const {
842:   return getStateManager().remove<T>(this, K, C);
843: }
844: 
845: template <typename T>
846: ProgramStateRef ProgramState::remove() const {
847:   return getStateManager().remove<T>(this);
848: }
849: 
850: template<typename T>
851: ProgramStateRef ProgramState::set(typename ProgramStateTrait<T>::data_type D) const {
852:   return getStateManager().set<T>(this, D);
853: }
854: 
855: template<typename T>
856: ProgramStateRef ProgramState::set(typename ProgramStateTrait<T>::key_type K,
857:                             typename ProgramStateTrait<T>::value_type E) const {
858:   return getStateManager().set<T>(this, K, E, get_context<T>());
859: }
860: 
861: template<typename T>
862: ProgramStateRef ProgramState::set(typename ProgramStateTrait<T>::key_type K,
863:                             typename ProgramStateTrait<T>::value_type E,
864:                             typename ProgramStateTrait<T>::context_type C) const {
```
- EN: It exposes API surface such as `getStateManager`, `getSymbolManager`, `add`, `get_context`.
- 中文: 它暴露了 `getStateManager`, `getSymbolManager`, `add`, `get_context` 等接口。

### Lines 865-908

```cpp
865:   return getStateManager().set<T>(this, K, E, C);
866: }
867: 
868: template <typename CB>
869: CB ProgramState::scanReachableSymbols(SVal val) const {
870:   CB cb(this);
871:   scanReachableSymbols(val, cb);
872:   return cb;
873: }
874: 
875: template <typename CB>
876: CB ProgramState::scanReachableSymbols(
877:     llvm::iterator_range<region_iterator> Reachable) const {
878:   CB cb(this);
879:   scanReachableSymbols(Reachable, cb);
880:   return cb;
881: }
882: 
883: /// \class ScanReachableSymbols
884: /// A utility class that visits the reachable symbols using a custom
885: /// SymbolVisitor. Terminates recursive traversal when the visitor function
886: /// returns false.
887: class ScanReachableSymbols {
888:   typedef llvm::DenseSet<const void*> VisitedItems;
889: 
890:   VisitedItems visited;
891:   ProgramStateRef state;
892:   SymbolVisitor &visitor;
893: public:
894:   ScanReachableSymbols(ProgramStateRef st, SymbolVisitor &v)
895:       : state(std::move(st)), visitor(v) {}
896: 
897:   bool scan(nonloc::LazyCompoundVal val);
898:   bool scan(nonloc::CompoundVal val);
899:   bool scan(SVal val);
900:   bool scan(const MemRegion *R);
901:   bool scan(const SymExpr *sym);
902: };
903: 
904: } // end ento namespace
905: 
906: } // end clang namespace
907: 
908: #endif
```
- EN: This range establishes include guards or other file-scope compilation boundaries. Key type declarations here include `ScanReachableSymbols`. It exposes API surface such as `getStateManager`, `scanReachableSymbols`, `cb`, `state`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这里的重要类型声明包括 `ScanReachableSymbols`。 它暴露了 `getStateManager`, `scanReachableSymbols`, `cb`, `state` 等接口。

## Key Concepts / 关键概念

- `APSInt`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ASTContext`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `AnalysisManager`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CallEvent`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CallEventManager`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ProgramStateTrait`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ProgramState`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ProgramStateManager`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/Basic/LLVM.h`, `clang/StaticAnalyzer/Core/PathSensitive/ConstraintManager.h`, `clang/StaticAnalyzer/Core/PathSensitive/DynamicTypeInfo.h`, `clang/StaticAnalyzer/Core/PathSensitive/Environment.h`, `clang/StaticAnalyzer/Core/PathSensitive/ProgramState_Fwd.h`, `clang/StaticAnalyzer/Core/PathSensitive/SValBuilder.h`, `clang/StaticAnalyzer/Core/PathSensitive/Store.h`, `llvm/ADT/FoldingSet.h`, `llvm/ADT/ImmutableMap.h`, `llvm/Support/Allocator.h`, `optional`, `utility`
- Forward declarations / 前向声明: `APSInt`, `ASTContext`, `AnalysisManager`, `CallEvent`, `CallEventManager`
- Namespace context / 命名空间上下文: `llvm`, `clang`, `ento`
- Macro-style dependencies / 宏式依赖: None / 无
