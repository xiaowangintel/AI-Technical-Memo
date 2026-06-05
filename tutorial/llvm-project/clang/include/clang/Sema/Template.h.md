# Template.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Sema/Template.h`
- Repository: `llvm-project`
- Purpose (EN): C++ Templates.
- 用途（中文）: 该文件为 Sema 子系统中的 Template 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48

```cpp
 1: //===- SemaTemplate.h - C++ Templates ---------------------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //===----------------------------------------------------------------------===//
 7: //
 8: // This file provides types used in the semantic analysis of C++ templates.
 9: //
10: //===----------------------------------------------------------------------===//
11: 
12: #ifndef LLVM_CLANG_SEMA_TEMPLATE_H
13: #define LLVM_CLANG_SEMA_TEMPLATE_H
14: 
15: #include "clang/AST/DeclTemplate.h"
16: #include "clang/AST/DeclVisitor.h"
17: #include "clang/AST/TemplateBase.h"
18: #include "clang/AST/Type.h"
19: #include "clang/Basic/LLVM.h"
20: #include "clang/Sema/Sema.h"
21: #include "llvm/ADT/ArrayRef.h"
22: #include "llvm/ADT/DenseMap.h"
23: #include "llvm/ADT/PointerUnion.h"
24: #include "llvm/ADT/SmallVector.h"
25: #include <cassert>
26: #include <optional>
27: #include <utility>
28: 
29: namespace clang {
30: 
31: class ASTContext;
32: class BindingDecl;
33: class CXXMethodDecl;
34: class Decl;
35: class DeclaratorDecl;
36: class DeclContext;
37: class EnumDecl;
38: class FunctionDecl;
39: class NamedDecl;
40: class ParmVarDecl;
41: class TagDecl;
42: class TypedefNameDecl;
43: class TypeSourceInfo;
44: class VarDecl;
45: 
46: /// The kind of template substitution being performed.
47: enum class TemplateSubstitutionKind : char {
48:   /// We are substituting template parameters for template arguments in order
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/AST/DeclTemplate.h`, `clang/AST/DeclVisitor.h`, `clang/AST/TemplateBase.h` and 10 more. It opens, closes, or documents namespace scope for `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/AST/DeclTemplate.h`, `clang/AST/DeclVisitor.h`, `clang/AST/TemplateBase.h` 以及另外 10 项依赖。 它打开、关闭或说明了 `clang` 的命名空间作用域。

### Lines 49-96

```cpp
49:   /// to form a template specialization.
50:   Specialization,
51:   /// We are substituting template parameters for (typically) other template
52:   /// parameters in order to rewrite a declaration as a different declaration
53:   /// (for example, when forming a deduction guide from a constructor).
54:   Rewrite,
55: };
56: 
57:   /// Data structure that captures multiple levels of template argument
58:   /// lists for use in template instantiation.
59:   ///
60:   /// Multiple levels of template arguments occur when instantiating the
61:   /// definitions of member templates. For example:
62:   ///
63:   /// \code
64:   /// template<typename T>
65:   /// struct X {
66:   ///   template<T Value>
67:   ///   struct Y {
68:   ///     void f();
69:   ///   };
70:   /// };
71:   /// \endcode
72:   ///
73:   /// When instantiating X<int>::Y<17>::f, the multi-level template argument
74:   /// list will contain a template argument list (int) at depth 0 and a
75:   /// template argument list (17) at depth 1.
76:   class MultiLevelTemplateArgumentList {
77:     /// The template argument list at a certain template depth
78: 
79:     using ArgList = ArrayRef<TemplateArgument>;
80:     struct ArgumentListLevel {
81:       llvm::PointerIntPair<Decl *, 1, bool> AssociatedDeclAndFinal;
82:       ArgList Args;
83:     };
84:     using ContainerType = SmallVector<ArgumentListLevel, 4>;
85: 
86:     using ArgListsIterator = ContainerType::iterator;
87:     using ConstArgListsIterator = ContainerType::const_iterator;
88: 
89:     /// The template argument lists, stored from the innermost template
90:     /// argument list (first) to the outermost template argument list (last).
91:     ContainerType TemplateArgumentLists;
92: 
93:     /// The number of outer levels of template arguments that are not
94:     /// being substituted.
95:     unsigned NumRetainedOuterLevels = 0;
96: 
```
- EN: Key type declarations here include `MultiLevelTemplateArgumentList`, `ArgumentListLevel`. It defines convenient aliases such as `ArgList`, `ContainerType`, `ArgListsIterator`, `ConstArgListsIterator`.
- 中文: 这里的重要类型声明包括 `MultiLevelTemplateArgumentList`, `ArgumentListLevel`。 它定义了 `ArgList`, `ContainerType`, `ArgListsIterator`, `ConstArgListsIterator` 等便捷别名。

### Lines 97-144

```cpp
 97:     /// The kind of substitution described by this argument list.
 98:     TemplateSubstitutionKind Kind = TemplateSubstitutionKind::Specialization;
 99: 
100:   public:
101:     /// Construct an empty set of template argument lists.
102:     MultiLevelTemplateArgumentList() = default;
103: 
104:     /// Construct a single-level template argument list.
105:     MultiLevelTemplateArgumentList(Decl *D, ArgList Args, bool Final) {
106:       addOuterTemplateArguments(D, Args, Final);
107:     }
108: 
109:     void setKind(TemplateSubstitutionKind K) { Kind = K; }
110: 
111:     /// Determine the kind of template substitution being performed.
112:     TemplateSubstitutionKind getKind() const { return Kind; }
113: 
114:     /// Determine whether we are rewriting template parameters rather than
115:     /// substituting for them. If so, we should not leave references to the
116:     /// original template parameters behind.
117:     bool isRewrite() const {
118:       return Kind == TemplateSubstitutionKind::Rewrite;
119:     }
120: 
121:     /// Determine the number of levels in this template argument
122:     /// list.
123:     unsigned getNumLevels() const {
124:       return TemplateArgumentLists.size() + NumRetainedOuterLevels;
125:     }
126: 
127:     /// Determine the number of substituted levels in this template
128:     /// argument list.
129:     unsigned getNumSubstitutedLevels() const {
130:       return TemplateArgumentLists.size();
131:     }
132: 
133:     // Determine the number of substituted args at 'Depth'.
134:     unsigned getNumSubsitutedArgs(unsigned Depth) const {
135:       assert(NumRetainedOuterLevels <= Depth && Depth < getNumLevels());
136:       return TemplateArgumentLists[getNumLevels() - Depth - 1].Args.size();
137:     }
138: 
139:     unsigned getNumRetainedOuterLevels() const {
140:       return NumRetainedOuterLevels;
141:     }
142: 
143:     /// Determine how many of the \p OldDepth outermost template parameter
144:     /// lists would be removed by substituting these arguments.
```
- EN: It exposes API surface such as `MultiLevelTemplateArgumentList`, `addOuterTemplateArguments`, `setKind`, `getKind`.
- 中文: 它暴露了 `MultiLevelTemplateArgumentList`, `addOuterTemplateArguments`, `setKind`, `getKind` 等接口。

### Lines 145-192

```cpp
145:     unsigned getNewDepth(unsigned OldDepth) const {
146:       if (OldDepth < NumRetainedOuterLevels)
147:         return OldDepth;
148:       if (OldDepth < getNumLevels())
149:         return NumRetainedOuterLevels;
150:       return OldDepth - TemplateArgumentLists.size();
151:     }
152: 
153:     /// Retrieve the template argument at a given depth and index.
154:     const TemplateArgument &operator()(unsigned Depth, unsigned Index) const {
155:       assert(NumRetainedOuterLevels <= Depth && Depth < getNumLevels());
156:       assert(Index <
157:              TemplateArgumentLists[getNumLevels() - Depth - 1].Args.size());
158:       return TemplateArgumentLists[getNumLevels() - Depth - 1].Args[Index];
159:     }
160: 
161:     /// A template-like entity which owns the whole pattern being substituted.
162:     /// This will usually own a set of template parameters, or in some
163:     /// cases might even be a template parameter itself.
164:     std::pair<Decl *, bool> getAssociatedDecl(unsigned Depth) const {
165:       assert(NumRetainedOuterLevels <= Depth && Depth < getNumLevels());
166:       auto AD = TemplateArgumentLists[getNumLevels() - Depth - 1]
167:                     .AssociatedDeclAndFinal;
168:       return {AD.getPointer(), AD.getInt()};
169:     }
170: 
171:     /// Determine whether there is a non-NULL template argument at the
172:     /// given depth and index.
173:     ///
174:     /// There must exist a template argument list at the given depth.
175:     bool hasTemplateArgument(unsigned Depth, unsigned Index) const {
176:       assert(Depth < getNumLevels());
177: 
178:       if (Depth < NumRetainedOuterLevels)
179:         return false;
180: 
181:       if (Index >=
182:           TemplateArgumentLists[getNumLevels() - Depth - 1].Args.size())
183:         return false;
184: 
185:       return !(*this)(Depth, Index).isNull();
186:     }
187: 
188:     bool isAnyArgInstantiationDependent() const {
189:       for (ArgumentListLevel ListLevel : TemplateArgumentLists)
190:         for (const TemplateArgument &TA : ListLevel.Args)
191:           // There might be null template arguments representing unused template
192:           // parameter mappings in an MLTAL during concept checking.
```
- EN: It exposes API surface such as `getNewDepth`, `size`, `operator`, `assert`.
- 中文: 它暴露了 `getNewDepth`, `size`, `operator`, `assert` 等接口。

### Lines 193-240

```cpp
193:           if (!TA.isNull() && TA.isInstantiationDependent())
194:             return true;
195:       return false;
196:     }
197: 
198:     /// Clear out a specific template argument.
199:     void setArgument(unsigned Depth, unsigned Index,
200:                      TemplateArgument Arg) {
201:       assert(NumRetainedOuterLevels <= Depth && Depth < getNumLevels());
202:       assert(Index <
203:              TemplateArgumentLists[getNumLevels() - Depth - 1].Args.size());
204:       const_cast<TemplateArgument &>(
205:           TemplateArgumentLists[getNumLevels() - Depth - 1].Args[Index]) = Arg;
206:     }
207: 
208:     /// Add a new outmost level to the multi-level template argument
209:     /// list.
210:     /// A 'Final' substitution means that these Args don't need to be
211:     /// resugared later.
212:     void addOuterTemplateArguments(Decl *AssociatedDecl, ArgList Args,
213:                                    bool Final) {
214:       assert(!NumRetainedOuterLevels &&
215:              "substituted args outside retained args?");
216:       assert(getKind() == TemplateSubstitutionKind::Specialization);
217:       TemplateArgumentLists.push_back(
218:           {{AssociatedDecl ? AssociatedDecl->getCanonicalDecl() : nullptr,
219:             Final},
220:            Args});
221:     }
222: 
223:     void addOuterTemplateArguments(ArgList Args) {
224:       assert(!NumRetainedOuterLevels &&
225:              "substituted args outside retained args?");
226:       assert(getKind() == TemplateSubstitutionKind::Rewrite);
227:       TemplateArgumentLists.push_back({{}, Args});
228:     }
229: 
230:     void addOuterTemplateArguments(std::nullopt_t) {
231:       assert(!NumRetainedOuterLevels &&
232:              "substituted args outside retained args?");
233:       TemplateArgumentLists.push_back({});
234:     }
235: 
236:     /// Replaces the current 'innermost' level with the provided argument list.
237:     /// This is useful for type deduction cases where we need to get the entire
238:     /// list from the AST, but then add the deduced innermost list.
239:     void replaceInnermostTemplateArguments(Decl *AssociatedDecl, ArgList Args,
240:                                            bool Final = false) {
```
- EN: It exposes API surface such as `assert`, `getNumLevels`, `addOuterTemplateArguments`.
- 中文: 它暴露了 `assert`, `getNumLevels`, `addOuterTemplateArguments` 等接口。

### Lines 241-288

```cpp
241:       assert((!TemplateArgumentLists.empty() || NumRetainedOuterLevels) &&
242:              "Replacing in an empty list?");
243: 
244:       if (!TemplateArgumentLists.empty()) {
245:         TemplateArgumentLists[0].Args = Args;
246:         return;
247:       }
248:       --NumRetainedOuterLevels;
249:       TemplateArgumentLists.push_back(
250:           {{AssociatedDecl, /*Final=*/Final}, Args});
251:     }
252: 
253:     void replaceOutermostTemplateArguments(Decl *AssociatedDecl, ArgList Args) {
254:       assert((!TemplateArgumentLists.empty()) && "Replacing in an empty list?");
255:       TemplateArgumentLists.back().AssociatedDeclAndFinal.setPointer(
256:           AssociatedDecl);
257:       TemplateArgumentLists.back().Args = Args;
258:     }
259: 
260:     /// Add an outermost level that we are not substituting. We have no
261:     /// arguments at this level, and do not remove it from the depth of inner
262:     /// template parameters that we instantiate.
263:     void addOuterRetainedLevel() {
264:       ++NumRetainedOuterLevels;
265:     }
266:     void addOuterRetainedLevels(unsigned Num) {
267:       NumRetainedOuterLevels += Num;
268:     }
269: 
270:     /// Retrieve the innermost template argument list.
271:     const ArgList &getInnermost() const {
272:       return TemplateArgumentLists.front().Args;
273:     }
274:     /// Retrieve the outermost template argument list.
275:     const ArgList &getOutermost() const {
276:       return TemplateArgumentLists.back().Args;
277:     }
278:     ArgListsIterator begin() { return TemplateArgumentLists.begin(); }
279:     ConstArgListsIterator begin() const {
280:       return TemplateArgumentLists.begin();
281:     }
282:     ArgListsIterator end() { return TemplateArgumentLists.end(); }
283:     ConstArgListsIterator end() const { return TemplateArgumentLists.end(); }
284: 
285:     LLVM_DUMP_METHOD void dump() const {
286:       LangOptions LO;
287:       LO.CPlusPlus = true;
288:       LO.Bool = true;
```
- EN: It exposes API surface such as `replaceOutermostTemplateArguments`, `assert`, `addOuterRetainedLevel`, `addOuterRetainedLevels`.
- 中文: 它暴露了 `replaceOutermostTemplateArguments`, `assert`, `addOuterRetainedLevel`, `addOuterRetainedLevels` 等接口。

### Lines 289-336

```cpp
289:       PrintingPolicy PP(LO);
290:       llvm::errs() << "NumRetainedOuterLevels: " << NumRetainedOuterLevels
291:                    << "\n";
292:       for (unsigned Depth = NumRetainedOuterLevels; Depth < getNumLevels();
293:            ++Depth) {
294:         llvm::errs() << Depth << ": ";
295:         printTemplateArgumentList(
296:             llvm::errs(),
297:             TemplateArgumentLists[getNumLevels() - Depth - 1].Args, PP);
298:         llvm::errs() << "\n";
299:       }
300:     }
301:   };
302: 
303:   /// The context in which partial ordering of function templates occurs.
304:   enum TPOC {
305:     /// Partial ordering of function templates for a function call.
306:     TPOC_Call,
307: 
308:     /// Partial ordering of function templates for a call to a
309:     /// conversion function.
310:     TPOC_Conversion,
311: 
312:     /// Partial ordering of function templates in other contexts, e.g.,
313:     /// taking the address of a function template or matching a function
314:     /// template specialization to a function template.
315:     TPOC_Other
316:   };
317: 
318:   // This is lame but unavoidable in a world without forward
319:   // declarations of enums.  The alternatives are to either pollute
320:   // Sema.h (by including this file) or sacrifice type safety (by
321:   // making Sema.h declare things as enums).
322:   class TemplatePartialOrderingContext {
323:     TPOC Value;
324: 
325:   public:
326:     TemplatePartialOrderingContext(TPOC Value) : Value(Value) {}
327: 
328:     operator TPOC() const { return Value; }
329:   };
330: 
331:   /// Captures a template argument whose value has been deduced
332:   /// via c++ template argument deduction.
333:   class DeducedTemplateArgument : public TemplateArgument {
334:     /// For a non-type template argument, whether the value was
335:     /// deduced from an array bound.
336:     bool DeducedFromArrayBound = false;
```
- EN: Key type declarations here include `TemplatePartialOrderingContext`, `DeducedTemplateArgument`. It introduces enum-based state or option sets such as `TPOC`. It exposes API surface such as `PP`, `getNumLevels`, `TemplatePartialOrderingContext`, `TPOC`.
- 中文: 这里的重要类型声明包括 `TemplatePartialOrderingContext`, `DeducedTemplateArgument`。 它引入了 `TPOC` 等基于枚举的状态或选项集合。 它暴露了 `PP`, `getNumLevels`, `TemplatePartialOrderingContext`, `TPOC` 等接口。

### Lines 337-384

```cpp
337: 
338:   public:
339:     DeducedTemplateArgument() = default;
340: 
341:     DeducedTemplateArgument(const TemplateArgument &Arg,
342:                             bool DeducedFromArrayBound = false)
343:         : TemplateArgument(Arg), DeducedFromArrayBound(DeducedFromArrayBound) {}
344: 
345:     /// Construct an integral non-type template argument that
346:     /// has been deduced, possibly from an array bound.
347:     DeducedTemplateArgument(ASTContext &Ctx,
348:                             const llvm::APSInt &Value,
349:                             QualType ValueType,
350:                             bool DeducedFromArrayBound)
351:         : TemplateArgument(Ctx, Value, ValueType),
352:           DeducedFromArrayBound(DeducedFromArrayBound) {}
353: 
354:     /// For a non-type template argument, determine whether the
355:     /// template argument was deduced from an array bound.
356:     bool wasDeducedFromArrayBound() const { return DeducedFromArrayBound; }
357: 
358:     /// Specify whether the given non-type template argument
359:     /// was deduced from an array bound.
360:     void setDeducedFromArrayBound(bool Deduced) {
361:       DeducedFromArrayBound = Deduced;
362:     }
363:   };
364: 
365:   /// A stack-allocated class that identifies which local
366:   /// variable declaration instantiations are present in this scope.
367:   ///
368:   /// A new instance of this class type will be created whenever we
369:   /// instantiate a new function declaration, which will have its own
370:   /// set of parameter declarations.
371:   class LocalInstantiationScope {
372:   public:
373:     /// A set of declarations.
374:     using DeclArgumentPack = SmallVector<ValueDecl *, 4>;
375: 
376:   private:
377:     /// Reference to the semantic analysis that is performing
378:     /// this template instantiation.
379:     Sema &SemaRef;
380: 
381:     using LocalDeclsMap =
382:         llvm::SmallDenseMap<const Decl *,
383:                             llvm::PointerUnion<Decl *, DeclArgumentPack *>, 4>;
384: 
```
- EN: Key type declarations here include `LocalInstantiationScope`. It defines convenient aliases such as `DeclArgumentPack`, `LocalDeclsMap`. It exposes API surface such as `DeducedTemplateArgument`, `TemplateArgument`, `DeducedFromArrayBound`, `wasDeducedFromArrayBound`.
- 中文: 这里的重要类型声明包括 `LocalInstantiationScope`。 它定义了 `DeclArgumentPack`, `LocalDeclsMap` 等便捷别名。 它暴露了 `DeducedTemplateArgument`, `TemplateArgument`, `DeducedFromArrayBound`, `wasDeducedFromArrayBound` 等接口。

### Lines 385-432

```cpp
385:     /// A mapping from local declarations that occur
386:     /// within a template to their instantiations.
387:     ///
388:     /// This mapping is used during instantiation to keep track of,
389:     /// e.g., function parameter and variable declarations. For example,
390:     /// given:
391:     ///
392:     /// \code
393:     ///   template<typename T> T add(T x, T y) { return x + y; }
394:     /// \endcode
395:     ///
396:     /// when we instantiate add<int>, we will introduce a mapping from
397:     /// the ParmVarDecl for 'x' that occurs in the template to the
398:     /// instantiated ParmVarDecl for 'x'.
399:     ///
400:     /// For a parameter pack, the local instantiation scope may contain a
401:     /// set of instantiated parameters. This is stored as a DeclArgumentPack
402:     /// pointer.
403:     LocalDeclsMap LocalDecls;
404: 
405:     /// The set of argument packs we've allocated.
406:     SmallVector<DeclArgumentPack *, 1> ArgumentPacks;
407: 
408:     /// The outer scope, which contains local variable
409:     /// definitions from some other instantiation (that may not be
410:     /// relevant to this particular scope).
411:     LocalInstantiationScope *Outer;
412: 
413:     /// Whether we have already exited this scope.
414:     bool Exited = false;
415: 
416:     /// Whether to combine this scope with the outer scope, such that
417:     /// lookup will search our outer scope.
418:     bool CombineWithOuterScope;
419: 
420:     /// Whether this scope is being used to instantiate a lambda or block
421:     /// expression, in which case it should be reused for instantiating the
422:     /// lambda's FunctionProtoType.
423:     bool InstantiatingLambdaOrBlock = false;
424: 
425:     /// If non-NULL, the template parameter pack that has been
426:     /// partially substituted per C++0x [temp.arg.explicit]p9.
427:     NamedDecl *PartiallySubstitutedPack = nullptr;
428: 
429:     /// If \c PartiallySubstitutedPack is non-null, the set of
430:     /// explicitly-specified template arguments in that pack.
431:     const TemplateArgument *ArgsInPartiallySubstitutedPack;
432: 
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 433-480

```cpp
433:     /// If \c PartiallySubstitutedPack, the number of
434:     /// explicitly-specified template arguments in
435:     /// ArgsInPartiallySubstitutedPack.
436:     unsigned NumArgsInPartiallySubstitutedPack;
437: 
438:   public:
439:     LocalInstantiationScope(Sema &SemaRef, bool CombineWithOuterScope = false,
440:                             bool InstantiatingLambdaOrBlock = false)
441:         : SemaRef(SemaRef), Outer(SemaRef.CurrentInstantiationScope),
442:           CombineWithOuterScope(CombineWithOuterScope),
443:           InstantiatingLambdaOrBlock(InstantiatingLambdaOrBlock) {
444:       SemaRef.CurrentInstantiationScope = this;
445:     }
446: 
447:     LocalInstantiationScope(const LocalInstantiationScope &) = delete;
448:     LocalInstantiationScope &
449:     operator=(const LocalInstantiationScope &) = delete;
450: 
451:     ~LocalInstantiationScope() {
452:       Exit();
453:     }
454: 
455:     const Sema &getSema() const { return SemaRef; }
456: 
457:     /// Exit this local instantiation scope early.
458:     void Exit() {
459:       if (Exited)
460:         return;
461: 
462:       for (unsigned I = 0, N = ArgumentPacks.size(); I != N; ++I)
463:         delete ArgumentPacks[I];
464: 
465:       SemaRef.CurrentInstantiationScope = Outer;
466:       Exited = true;
467:     }
468: 
469:     /// Clone this scope, and all outer scopes, down to the given
470:     /// outermost scope.
471:     LocalInstantiationScope *cloneScopes(LocalInstantiationScope *Outermost) {
472:       if (this == Outermost) return this;
473: 
474:       // Save the current scope from SemaRef since the LocalInstantiationScope
475:       // will overwrite it on construction
476:       LocalInstantiationScope *oldScope = SemaRef.CurrentInstantiationScope;
477: 
478:       LocalInstantiationScope *newScope =
479:         new LocalInstantiationScope(SemaRef, CombineWithOuterScope);
480: 
```
- EN: It exposes API surface such as `InstantiatingLambdaOrBlock`, `LocalInstantiationScope`, `~LocalInstantiationScope`, `Exit`.
- 中文: 它暴露了 `InstantiatingLambdaOrBlock`, `LocalInstantiationScope`, `~LocalInstantiationScope`, `Exit` 等接口。

### Lines 481-528

```cpp
481:       newScope->Outer = nullptr;
482:       if (Outer)
483:         newScope->Outer = Outer->cloneScopes(Outermost);
484: 
485:       newScope->PartiallySubstitutedPack = PartiallySubstitutedPack;
486:       newScope->ArgsInPartiallySubstitutedPack = ArgsInPartiallySubstitutedPack;
487:       newScope->NumArgsInPartiallySubstitutedPack =
488:         NumArgsInPartiallySubstitutedPack;
489: 
490:       for (LocalDeclsMap::iterator I = LocalDecls.begin(), E = LocalDecls.end();
491:            I != E; ++I) {
492:         const Decl *D = I->first;
493:         llvm::PointerUnion<Decl *, DeclArgumentPack *> &Stored =
494:           newScope->LocalDecls[D];
495:         if (auto *D2 = dyn_cast<Decl *>(I->second)) {
496:           Stored = D2;
497:         } else {
498:           DeclArgumentPack *OldPack = cast<DeclArgumentPack *>(I->second);
499:           DeclArgumentPack *NewPack = new DeclArgumentPack(*OldPack);
500:           Stored = NewPack;
501:           newScope->ArgumentPacks.push_back(NewPack);
502:         }
503:       }
504:       // Restore the saved scope to SemaRef
505:       SemaRef.CurrentInstantiationScope = oldScope;
506:       return newScope;
507:     }
508: 
509:     /// deletes the given scope, and all outer scopes, down to the
510:     /// given outermost scope.
511:     static void deleteScopes(LocalInstantiationScope *Scope,
512:                              LocalInstantiationScope *Outermost) {
513:       while (Scope && Scope != Outermost) {
514:         LocalInstantiationScope *Out = Scope->Outer;
515:         delete Scope;
516:         Scope = Out;
517:       }
518:     }
519: 
520:     /// Find the instantiation of the declaration D within the current
521:     /// instantiation scope.
522:     ///
523:     /// \param D The declaration whose instantiation we are searching for.
524:     ///
525:     /// \returns A pointer to the declaration or argument pack of declarations
526:     /// to which the declaration \c D is instantiated, if found. Otherwise,
527:     /// returns NULL.
528:     llvm::PointerUnion<Decl *, DeclArgumentPack *> *
```
- EN: It exposes API surface such as `cloneScopes`, `DeclArgumentPack`, `push_back`.
- 中文: 它暴露了 `cloneScopes`, `DeclArgumentPack`, `push_back` 等接口。

### Lines 529-576

```cpp
529:     findInstantiationOf(const Decl *D);
530: 
531:     /// Similar to \p findInstantiationOf(), but it wouldn't assert if the
532:     /// instantiation was not found within the current instantiation scope. This
533:     /// is helpful for on-demand declaration instantiation.
534:     llvm::PointerUnion<Decl *, DeclArgumentPack *> *
535:     getInstantiationOfIfExists(const Decl *D);
536: 
537:     void InstantiatedLocal(const Decl *D, Decl *Inst);
538:     void InstantiatedLocalPackArg(const Decl *D, VarDecl *Inst);
539:     void MakeInstantiatedLocalArgPack(const Decl *D);
540: 
541:     /// Note that the given parameter pack has been partially substituted
542:     /// via explicit specification of template arguments
543:     /// (C++0x [temp.arg.explicit]p9).
544:     ///
545:     /// \param Pack The parameter pack, which will always be a template
546:     /// parameter pack.
547:     ///
548:     /// \param ExplicitArgs The explicitly-specified template arguments provided
549:     /// for this parameter pack.
550:     ///
551:     /// \param NumExplicitArgs The number of explicitly-specified template
552:     /// arguments provided for this parameter pack.
553:     void SetPartiallySubstitutedPack(NamedDecl *Pack,
554:                                      const TemplateArgument *ExplicitArgs,
555:                                      unsigned NumExplicitArgs);
556: 
557:     /// Reset the partially-substituted pack when it is no longer of
558:     /// interest.
559:     void ResetPartiallySubstitutedPack() {
560:       assert(PartiallySubstitutedPack && "No partially-substituted pack");
561:       PartiallySubstitutedPack = nullptr;
562:       ArgsInPartiallySubstitutedPack = nullptr;
563:       NumArgsInPartiallySubstitutedPack = 0;
564:     }
565: 
566:     /// Retrieve the partially-substitued template parameter pack.
567:     ///
568:     /// If there is no partially-substituted parameter pack, returns NULL.
569:     NamedDecl *
570:     getPartiallySubstitutedPack(const TemplateArgument **ExplicitArgs = nullptr,
571:                                 unsigned *NumExplicitArgs = nullptr) const;
572: 
573:     /// Determine whether D is a pack expansion created in this scope.
574:     bool isLocalPackExpansion(const Decl *D);
575: 
576:     /// Determine whether this scope is for instantiating a lambda or block.
```
- EN: It exposes API surface such as `findInstantiationOf`, `getInstantiationOfIfExists`, `InstantiatedLocal`, `InstantiatedLocalPackArg`.
- 中文: 它暴露了 `findInstantiationOf`, `getInstantiationOfIfExists`, `InstantiatedLocal`, `InstantiatedLocalPackArg` 等接口。

### Lines 577-624

```cpp
577:     bool isLambdaOrBlock() const { return InstantiatingLambdaOrBlock; }
578:   };
579: 
580:   class TemplateDeclInstantiator
581:     : public DeclVisitor<TemplateDeclInstantiator, Decl *>
582:   {
583:     Sema &SemaRef;
584:     Sema::ArgPackSubstIndexRAII SubstIndex;
585:     DeclContext *Owner;
586:     const MultiLevelTemplateArgumentList &TemplateArgs;
587:     Sema::LateInstantiatedAttrVec* LateAttrs = nullptr;
588:     LocalInstantiationScope *StartingScope = nullptr;
589:     // Whether to evaluate the C++20 constraints or simply substitute into them.
590:     bool EvaluateConstraints = true;
591: 
592:     /// A list of out-of-line class template partial
593:     /// specializations that will need to be instantiated after the
594:     /// enclosing class's instantiation is complete.
595:     SmallVector<std::pair<ClassTemplateDecl *,
596:                           ClassTemplatePartialSpecializationDecl *>,
597:                 1>
598:         OutOfLinePartialSpecs;
599: 
600:     /// A list of out-of-line variable template partial
601:     /// specializations that will need to be instantiated after the
602:     /// enclosing variable's instantiation is complete.
603:     /// FIXME: Verify that this is needed.
604:     SmallVector<
605:         std::pair<VarTemplateDecl *, VarTemplatePartialSpecializationDecl *>, 1>
606:         OutOfLineVarPartialSpecs;
607: 
608:   public:
609:     TemplateDeclInstantiator(Sema &SemaRef, DeclContext *Owner,
610:                              const MultiLevelTemplateArgumentList &TemplateArgs)
611:         : SemaRef(SemaRef), SubstIndex(SemaRef, SemaRef.ArgPackSubstIndex),
612:           Owner(Owner), TemplateArgs(TemplateArgs) {}
613: 
614:     void setEvaluateConstraints(bool B) {
615:       EvaluateConstraints = B;
616:     }
617:     bool getEvaluateConstraints() {
618:       return EvaluateConstraints;
619:     }
620: 
621: // Define all the decl visitors using DeclNodes.inc
622: #define DECL(DERIVED, BASE) \
623:     Decl *Visit ## DERIVED ## Decl(DERIVED ## Decl *D);
624: #define ABSTRACT_DECL(DECL)
```
- EN: This range establishes include guards or other file-scope compilation boundaries. Key type declarations here include `TemplateDeclInstantiator`. It exposes API surface such as `isLambdaOrBlock`, `Owner`, `setEvaluateConstraints`, `getEvaluateConstraints`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这里的重要类型声明包括 `TemplateDeclInstantiator`。 它暴露了 `isLambdaOrBlock`, `Owner`, `setEvaluateConstraints`, `getEvaluateConstraints` 等接口。

### Lines 625-672

```cpp
625: 
626: // Decls which never appear inside a class or function.
627: #define OBJCCONTAINER(DERIVED, BASE)
628: #define FILESCOPEASM(DERIVED, BASE)
629: #define TOPLEVELSTMT(DERIVED, BASE)
630: #define IMPORT(DERIVED, BASE)
631: #define EXPORT(DERIVED, BASE)
632: #define LINKAGESPEC(DERIVED, BASE)
633: #define OBJCCOMPATIBLEALIAS(DERIVED, BASE)
634: #define OBJCMETHOD(DERIVED, BASE)
635: #define OBJCTYPEPARAM(DERIVED, BASE)
636: #define OBJCIVAR(DERIVED, BASE)
637: #define OBJCPROPERTY(DERIVED, BASE)
638: #define OBJCPROPERTYIMPL(DERIVED, BASE)
639: #define EMPTY(DERIVED, BASE)
640: #define LIFETIMEEXTENDEDTEMPORARY(DERIVED, BASE)
641: 
642: // Decls which never appear inside a template.
643: #define OUTLINEDFUNCTION(DERIVED, BASE)
644: 
645: // Decls which use special-case instantiation code.
646: #define BLOCK(DERIVED, BASE)
647: #define CAPTURED(DERIVED, BASE)
648: #define IMPLICITPARAM(DERIVED, BASE)
649: 
650: #include "clang/AST/DeclNodes.inc"
651: 
652:     enum class RewriteKind { None, RewriteSpaceshipAsEqualEqual };
653: 
654:     void adjustForRewrite(RewriteKind RK, FunctionDecl *Orig, QualType &T,
655:                           TypeSourceInfo *&TInfo,
656:                           DeclarationNameInfo &NameInfo);
657: 
658:     // A few supplemental visitor functions.
659:     Decl *VisitCXXMethodDecl(CXXMethodDecl *D,
660:                              TemplateParameterList *TemplateParams,
661:                              RewriteKind RK = RewriteKind::None);
662:     Decl *VisitFunctionDecl(FunctionDecl *D,
663:                             TemplateParameterList *TemplateParams,
664:                             RewriteKind RK = RewriteKind::None);
665:     Decl *VisitDecl(Decl *D);
666:     Decl *VisitVarDecl(VarDecl *D, bool InstantiatingVarTemplate,
667:                        ArrayRef<BindingDecl *> *Bindings = nullptr);
668:     Decl *VisitBaseUsingDecls(BaseUsingDecl *D, BaseUsingDecl *Inst,
669:                               LookupResult *Lookup);
670: 
671:     // Enable late instantiation of attributes.  Late instantiated attributes
672:     // will be stored in LA.
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/AST/DeclNodes.inc`. Key type declarations here include `RewriteKind`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/AST/DeclNodes.inc` 等依赖。 这里的重要类型声明包括 `RewriteKind`。

### Lines 673-720

```cpp
673:     void enableLateAttributeInstantiation(Sema::LateInstantiatedAttrVec *LA) {
674:       LateAttrs = LA;
675:       StartingScope = SemaRef.CurrentInstantiationScope;
676:     }
677: 
678:     // Disable late instantiation of attributes.
679:     void disableLateAttributeInstantiation() {
680:       LateAttrs = nullptr;
681:       StartingScope = nullptr;
682:     }
683: 
684:     LocalInstantiationScope *getStartingScope() const { return StartingScope; }
685: 
686:     using delayed_partial_spec_iterator = SmallVectorImpl<std::pair<
687:       ClassTemplateDecl *, ClassTemplatePartialSpecializationDecl *>>::iterator;
688: 
689:     using delayed_var_partial_spec_iterator = SmallVectorImpl<std::pair<
690:         VarTemplateDecl *, VarTemplatePartialSpecializationDecl *>>::iterator;
691: 
692:     /// Return an iterator to the beginning of the set of
693:     /// "delayed" partial specializations, which must be passed to
694:     /// InstantiateClassTemplatePartialSpecialization once the class
695:     /// definition has been completed.
696:     delayed_partial_spec_iterator delayed_partial_spec_begin() {
697:       return OutOfLinePartialSpecs.begin();
698:     }
699: 
700:     delayed_var_partial_spec_iterator delayed_var_partial_spec_begin() {
701:       return OutOfLineVarPartialSpecs.begin();
702:     }
703: 
704:     /// Return an iterator to the end of the set of
705:     /// "delayed" partial specializations, which must be passed to
706:     /// InstantiateClassTemplatePartialSpecialization once the class
707:     /// definition has been completed.
708:     delayed_partial_spec_iterator delayed_partial_spec_end() {
709:       return OutOfLinePartialSpecs.end();
710:     }
711: 
712:     delayed_var_partial_spec_iterator delayed_var_partial_spec_end() {
713:       return OutOfLineVarPartialSpecs.end();
714:     }
715: 
716:     // Helper functions for instantiating methods.
717:     TypeSourceInfo *SubstFunctionType(FunctionDecl *D,
718:                              SmallVectorImpl<ParmVarDecl *> &Params);
719:     bool InitFunctionInstantiation(FunctionDecl *New, FunctionDecl *Tmpl);
720:     bool InitMethodInstantiation(CXXMethodDecl *New, CXXMethodDecl *Tmpl);
```
- EN: It defines convenient aliases such as `delayed_partial_spec_iterator`, `delayed_var_partial_spec_iterator`. It exposes API surface such as `enableLateAttributeInstantiation`, `disableLateAttributeInstantiation`, `getStartingScope`, `delayed_partial_spec_begin`.
- 中文: 它定义了 `delayed_partial_spec_iterator`, `delayed_var_partial_spec_iterator` 等便捷别名。 它暴露了 `enableLateAttributeInstantiation`, `disableLateAttributeInstantiation`, `getStartingScope`, `delayed_partial_spec_begin` 等接口。

### Lines 721-757

```cpp
721: 
722:     bool SubstDefaultedFunction(FunctionDecl *New, FunctionDecl *Tmpl);
723: 
724:     TemplateParameterList *
725:       SubstTemplateParams(TemplateParameterList *List);
726: 
727:     bool SubstQualifier(const DeclaratorDecl *OldDecl,
728:                         DeclaratorDecl *NewDecl);
729:     bool SubstQualifier(const TagDecl *OldDecl,
730:                         TagDecl *NewDecl);
731: 
732:     VarTemplateSpecializationDecl *VisitVarTemplateSpecializationDecl(
733:         VarTemplateDecl *VarTemplate, VarDecl *FromVar,
734:         ArrayRef<TemplateArgument> Converted,
735:         VarTemplateSpecializationDecl *PrevDecl = nullptr);
736: 
737:     Decl *InstantiateTypedefNameDecl(TypedefNameDecl *D, bool IsTypeAlias);
738:     Decl *InstantiateTypeAliasTemplateDecl(TypeAliasTemplateDecl *D);
739:     ClassTemplatePartialSpecializationDecl *
740:     InstantiateClassTemplatePartialSpecialization(
741:                                               ClassTemplateDecl *ClassTemplate,
742:                            ClassTemplatePartialSpecializationDecl *PartialSpec);
743:     VarTemplatePartialSpecializationDecl *
744:     InstantiateVarTemplatePartialSpecialization(
745:         VarTemplateDecl *VarTemplate,
746:         VarTemplatePartialSpecializationDecl *PartialSpec);
747:     void InstantiateEnumDefinition(EnumDecl *Enum, EnumDecl *Pattern);
748: 
749:   private:
750:     template<typename T>
751:     Decl *instantiateUnresolvedUsingDecl(T *D,
752:                                          bool InstantiatingPackElement = false);
753:   };
754: 
755: } // namespace clang
756: 
757: #endif // LLVM_CLANG_SEMA_TEMPLATE_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `clang`. It exposes API surface such as `SubstDefaultedFunction`, `SubstTemplateParams`, `InstantiateTypedefNameDecl`, `InstantiateTypeAliasTemplateDecl`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `clang` 的命名空间作用域。 它暴露了 `SubstDefaultedFunction`, `SubstTemplateParams`, `InstantiateTypedefNameDecl`, `InstantiateTypeAliasTemplateDecl` 等接口。

## Key Concepts / 关键概念

- `ASTContext`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `BindingDecl`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CXXMethodDecl`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Decl`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `DeclaratorDecl`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `DeclContext`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `EnumDecl`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `FunctionDecl`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/AST/DeclTemplate.h`, `clang/AST/DeclVisitor.h`, `clang/AST/TemplateBase.h`, `clang/AST/Type.h`, `clang/Basic/LLVM.h`, `clang/Sema/Sema.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/PointerUnion.h`, `llvm/ADT/SmallVector.h`, `cassert`, `optional`, `utility`, `clang/AST/DeclNodes.inc`
- Forward declarations / 前向声明: `ASTContext`, `BindingDecl`, `CXXMethodDecl`, `Decl`, `DeclaratorDecl`, `DeclContext`, `EnumDecl`, `FunctionDecl`, `NamedDecl`, `ParmVarDecl`, `TagDecl`, `TypedefNameDecl`, `TypeSourceInfo`, `VarDecl`
- Namespace context / 命名空间上下文: `clang`
- Macro-style dependencies / 宏式依赖: None / 无
