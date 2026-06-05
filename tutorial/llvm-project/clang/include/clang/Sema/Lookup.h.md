# Lookup.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Sema/Lookup.h`
- Repository: `llvm-project`
- Purpose (EN): Classes for name lookup.
- 用途（中文）: 该文件为 Sema 子系统中的 Lookup 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48

```cpp
 1: //===- Lookup.h - Classes for name lookup -----------------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file defines the LookupResult class, which is integral to
10: // Sema's name-lookup subsystem.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #ifndef LLVM_CLANG_SEMA_LOOKUP_H
15: #define LLVM_CLANG_SEMA_LOOKUP_H
16: 
17: #include "clang/AST/Decl.h"
18: #include "clang/AST/DeclBase.h"
19: #include "clang/AST/DeclCXX.h"
20: #include "clang/AST/DeclarationName.h"
21: #include "clang/AST/Type.h"
22: #include "clang/AST/UnresolvedSet.h"
23: #include "clang/Basic/LLVM.h"
24: #include "clang/Basic/LangOptions.h"
25: #include "clang/Basic/SourceLocation.h"
26: #include "clang/Basic/Specifiers.h"
27: #include "clang/Sema/Sema.h"
28: #include "llvm/ADT/MapVector.h"
29: #include "llvm/ADT/STLExtras.h"
30: #include "llvm/Support/Casting.h"
31: #include <cassert>
32: #include <optional>
33: #include <utility>
34: 
35: namespace clang {
36: 
37: class CXXBasePaths;
38: 
39: enum class LookupResultKind {
40:   /// No entity found met the criteria.
41:   NotFound = 0,
42: 
43:   /// No entity found met the criteria within the current
44:   /// instantiation,, but there were dependent base classes of the
45:   /// current instantiation that could not be searched.
46:   NotFoundInCurrentInstantiation,
47: 
48:   /// Name lookup found a single declaration that met the
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/AST/Decl.h`, `clang/AST/DeclBase.h`, `clang/AST/DeclCXX.h` and 14 more. It opens, closes, or documents namespace scope for `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/AST/Decl.h`, `clang/AST/DeclBase.h`, `clang/AST/DeclCXX.h` 以及另外 14 项依赖。 它打开、关闭或说明了 `clang` 的命名空间作用域。

### Lines 49-96

```cpp
49:   /// criteria.  getFoundDecl() will return this declaration.
50:   Found,
51: 
52:   /// Name lookup found a set of overloaded functions that
53:   /// met the criteria.
54:   FoundOverloaded,
55: 
56:   /// Name lookup found an unresolvable value declaration
57:   /// and cannot yet complete.  This only happens in C++ dependent
58:   /// contexts with dependent using declarations.
59:   FoundUnresolvedValue,
60: 
61:   /// Name lookup results in an ambiguity; use
62:   /// getAmbiguityKind to figure out what kind of ambiguity
63:   /// we have.
64:   Ambiguous
65: };
66: 
67: enum class LookupAmbiguityKind {
68:   /// Name lookup results in an ambiguity because multiple
69:   /// entities that meet the lookup criteria were found in
70:   /// subobjects of different types. For example:
71:   /// @code
72:   /// struct A { void f(int); }
73:   /// struct B { void f(double); }
74:   /// struct C : A, B { };
75:   /// void test(C c) {
76:   ///   c.f(0); // error: A::f and B::f come from subobjects of different
77:   ///           // types. overload resolution is not performed.
78:   /// }
79:   /// @endcode
80:   AmbiguousBaseSubobjectTypes,
81: 
82:   /// Name lookup results in an ambiguity because multiple
83:   /// nonstatic entities that meet the lookup criteria were found
84:   /// in different subobjects of the same type. For example:
85:   /// @code
86:   /// struct A { int x; };
87:   /// struct B : A { };
88:   /// struct C : A { };
89:   /// struct D : B, C { };
90:   /// int test(D d) {
91:   ///   return d.x; // error: 'x' is found in two A subobjects (of B and C)
92:   /// }
93:   /// @endcode
94:   AmbiguousBaseSubobjects,
95: 
96:   /// Name lookup results in an ambiguity because multiple definitions
```
- EN: Key type declarations here include `LookupAmbiguityKind`. It introduces enum-based state or option sets such as `LookupAmbiguityKind`.
- 中文: 这里的重要类型声明包括 `LookupAmbiguityKind`。 它引入了 `LookupAmbiguityKind` 等基于枚举的状态或选项集合。

### Lines 97-144

```cpp
 97:   /// of entity that meet the lookup criteria were found in different
 98:   /// declaration contexts.
 99:   /// @code
100:   /// namespace A {
101:   ///   int i;
102:   ///   namespace B { int i; }
103:   ///   int test() {
104:   ///     using namespace B;
105:   ///     return i; // error 'i' is found in namespace A and A::B
106:   ///    }
107:   /// }
108:   /// @endcode
109:   AmbiguousReference,
110: 
111:   /// Name lookup results in an ambiguity because multiple placeholder
112:   /// variables were found in the same scope.
113:   /// @code
114:   /// void f() {
115:   ///    int _ = 0;
116:   ///    int _ = 0;
117:   ///    return _; // ambiguous use of placeholder variable
118:   /// }
119:   /// @endcode
120:   AmbiguousReferenceToPlaceholderVariable,
121: 
122:   /// Name lookup results in an ambiguity because an entity with a
123:   /// tag name was hidden by an entity with an ordinary name from
124:   /// a different context.
125:   /// @code
126:   /// namespace A { struct Foo {}; }
127:   /// namespace B { void Foo(); }
128:   /// namespace C {
129:   ///   using namespace A;
130:   ///   using namespace B;
131:   /// }
132:   /// void test() {
133:   ///   C::Foo(); // error: tag 'A::Foo' is hidden by an object in a
134:   ///             // different namespace
135:   /// }
136:   /// @endcode
137:   AmbiguousTagHiding
138: };
139: 
140: /// Represents the results of name lookup.
141: ///
142: /// An instance of the LookupResult class captures the results of a
143: /// single name lookup, which can return no result (nothing found),
144: /// a single declaration, a set of overloaded functions, or an
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 145-192

```cpp
145: /// ambiguity. Use the getKind() method to determine which of these
146: /// results occurred for a given lookup.
147: class LookupResult {
148: public:
149:   /// A little identifier for flagging temporary lookup results.
150:   enum TemporaryToken {
151:     Temporary
152:   };
153: 
154:   using iterator = UnresolvedSetImpl::iterator;
155: 
156:   LookupResult(
157:       Sema &SemaRef, const DeclarationNameInfo &NameInfo,
158:       Sema::LookupNameKind LookupKind,
159:       RedeclarationKind Redecl = RedeclarationKind::NotForRedeclaration)
160:       : SemaPtr(&SemaRef), NameInfo(NameInfo), LookupKind(LookupKind),
161:         Redecl(Redecl != RedeclarationKind::NotForRedeclaration),
162:         ExternalRedecl(Redecl == RedeclarationKind::ForExternalRedeclaration),
163:         DiagnoseAccess(Redecl == RedeclarationKind::NotForRedeclaration),
164:         DiagnoseAmbiguous(Redecl == RedeclarationKind::NotForRedeclaration) {
165:     configure();
166:   }
167: 
168:   // TODO: consider whether this constructor should be restricted to take
169:   // as input a const IdentifierInfo* (instead of Name),
170:   // forcing other cases towards the constructor taking a DNInfo.
171:   LookupResult(
172:       Sema &SemaRef, DeclarationName Name, SourceLocation NameLoc,
173:       Sema::LookupNameKind LookupKind,
174:       RedeclarationKind Redecl = RedeclarationKind::NotForRedeclaration)
175:       : SemaPtr(&SemaRef), NameInfo(Name, NameLoc), LookupKind(LookupKind),
176:         Redecl(Redecl != RedeclarationKind::NotForRedeclaration),
177:         ExternalRedecl(Redecl == RedeclarationKind::ForExternalRedeclaration),
178:         DiagnoseAccess(Redecl == RedeclarationKind::NotForRedeclaration),
179:         DiagnoseAmbiguous(Redecl == RedeclarationKind::NotForRedeclaration) {
180:     configure();
181:   }
182: 
183:   /// Creates a temporary lookup result, initializing its core data
184:   /// using the information from another result.  Diagnostics are always
185:   /// disabled.
186:   LookupResult(TemporaryToken _, const LookupResult &Other)
187:       : SemaPtr(Other.SemaPtr), NameInfo(Other.NameInfo),
188:         LookupKind(Other.LookupKind), IDNS(Other.IDNS), Redecl(Other.Redecl),
189:         ExternalRedecl(Other.ExternalRedecl), HideTags(Other.HideTags),
190:         AllowHidden(Other.AllowHidden),
191:         TemplateNameLookup(Other.TemplateNameLookup) {}
192: 
```
- EN: Key type declarations here include `LookupResult`. It introduces enum-based state or option sets such as `TemporaryToken`. It defines convenient aliases such as `iterator`.
- 中文: 这里的重要类型声明包括 `LookupResult`。 它引入了 `TemporaryToken` 等基于枚举的状态或选项集合。 它定义了 `iterator` 等便捷别名。

### Lines 193-240

```cpp
193:   // FIXME: Remove these deleted methods once the default build includes
194:   // -Wdeprecated.
195:   LookupResult(const LookupResult &) = delete;
196:   LookupResult &operator=(const LookupResult &) = delete;
197: 
198:   LookupResult(LookupResult &&Other)
199:       : ResultKind(std::move(Other.ResultKind)),
200:         Ambiguity(std::move(Other.Ambiguity)), Decls(std::move(Other.Decls)),
201:         Paths(std::move(Other.Paths)),
202:         NamingClass(std::move(Other.NamingClass)),
203:         BaseObjectType(std::move(Other.BaseObjectType)),
204:         SemaPtr(std::move(Other.SemaPtr)), NameInfo(std::move(Other.NameInfo)),
205:         NameContextRange(std::move(Other.NameContextRange)),
206:         LookupKind(std::move(Other.LookupKind)), IDNS(std::move(Other.IDNS)),
207:         Redecl(std::move(Other.Redecl)),
208:         ExternalRedecl(std::move(Other.ExternalRedecl)),
209:         HideTags(std::move(Other.HideTags)),
210:         DiagnoseAccess(std::move(Other.DiagnoseAccess)),
211:         DiagnoseAmbiguous(std::move(Other.DiagnoseAmbiguous)),
212:         AllowHidden(std::move(Other.AllowHidden)),
213:         Shadowed(std::move(Other.Shadowed)),
214:         TemplateNameLookup(std::move(Other.TemplateNameLookup)) {
215:     Other.Paths = nullptr;
216:     Other.DiagnoseAccess = false;
217:     Other.DiagnoseAmbiguous = false;
218:   }
219: 
220:   LookupResult &operator=(LookupResult &&Other) {
221:     ResultKind = std::move(Other.ResultKind);
222:     Ambiguity = std::move(Other.Ambiguity);
223:     Decls = std::move(Other.Decls);
224:     Paths = std::move(Other.Paths);
225:     NamingClass = std::move(Other.NamingClass);
226:     BaseObjectType = std::move(Other.BaseObjectType);
227:     SemaPtr = std::move(Other.SemaPtr);
228:     NameInfo = std::move(Other.NameInfo);
229:     NameContextRange = std::move(Other.NameContextRange);
230:     LookupKind = std::move(Other.LookupKind);
231:     IDNS = std::move(Other.IDNS);
232:     Redecl = std::move(Other.Redecl);
233:     ExternalRedecl = std::move(Other.ExternalRedecl);
234:     HideTags = std::move(Other.HideTags);
235:     DiagnoseAccess = std::move(Other.DiagnoseAccess);
236:     DiagnoseAmbiguous = std::move(Other.DiagnoseAmbiguous);
237:     AllowHidden = std::move(Other.AllowHidden);
238:     Shadowed = std::move(Other.Shadowed);
239:     TemplateNameLookup = std::move(Other.TemplateNameLookup);
240:     Other.Paths = nullptr;
```
- EN: It exposes API surface such as `LookupResult`, `TemplateNameLookup`, `move`.
- 中文: 它暴露了 `LookupResult`, `TemplateNameLookup`, `move` 等接口。

### Lines 241-288

```cpp
241:     Other.DiagnoseAccess = false;
242:     Other.DiagnoseAmbiguous = false;
243:     return *this;
244:   }
245: 
246:   ~LookupResult() {
247:     if (DiagnoseAccess)
248:       diagnoseAccess();
249:     if (DiagnoseAmbiguous)
250:       diagnoseAmbiguous();
251:     if (Paths) deletePaths(Paths);
252:   }
253: 
254:   /// Gets the name info to look up.
255:   const DeclarationNameInfo &getLookupNameInfo() const {
256:     return NameInfo;
257:   }
258: 
259:   /// Sets the name info to look up.
260:   void setLookupNameInfo(const DeclarationNameInfo &NameInfo) {
261:     this->NameInfo = NameInfo;
262:   }
263: 
264:   /// Gets the name to look up.
265:   DeclarationName getLookupName() const {
266:     return NameInfo.getName();
267:   }
268: 
269:   /// Sets the name to look up.
270:   void setLookupName(DeclarationName Name) {
271:     NameInfo.setName(Name);
272:   }
273: 
274:   /// Gets the kind of lookup to perform.
275:   Sema::LookupNameKind getLookupKind() const {
276:     return LookupKind;
277:   }
278: 
279:   /// True if this lookup is just looking for an existing declaration.
280:   bool isForRedeclaration() const {
281:     return Redecl;
282:   }
283: 
284:   /// True if this lookup is just looking for an existing declaration to link
285:   /// against a declaration with external linkage.
286:   bool isForExternalRedeclaration() const {
287:     return ExternalRedecl;
288:   }
```
- EN: It exposes API surface such as `~LookupResult`, `diagnoseAccess`, `diagnoseAmbiguous`, `getLookupNameInfo`.
- 中文: 它暴露了 `~LookupResult`, `diagnoseAccess`, `diagnoseAmbiguous`, `getLookupNameInfo` 等接口。

### Lines 289-336

```cpp
289: 
290:   RedeclarationKind redeclarationKind() const {
291:     return ExternalRedecl ? RedeclarationKind::ForExternalRedeclaration
292:            : Redecl       ? RedeclarationKind::ForVisibleRedeclaration
293:                           : RedeclarationKind::NotForRedeclaration;
294:   }
295: 
296:   /// Specify whether hidden declarations are visible, e.g.,
297:   /// for recovery reasons.
298:   void setAllowHidden(bool AH) {
299:     AllowHidden = AH;
300:   }
301: 
302:   /// Determine whether this lookup is permitted to see hidden
303:   /// declarations, such as those in modules that have not yet been imported.
304:   bool isHiddenDeclarationVisible(NamedDecl *ND) const {
305:     return AllowHidden ||
306:            (isForExternalRedeclaration() && ND->isExternallyDeclarable());
307:   }
308: 
309:   /// Sets whether tag declarations should be hidden by non-tag
310:   /// declarations during resolution.  The default is true.
311:   void setHideTags(bool Hide) {
312:     HideTags = Hide;
313:   }
314: 
315:   /// Sets whether this is a template-name lookup. For template-name lookups,
316:   /// injected-class-names are treated as naming a template rather than a
317:   /// template specialization.
318:   void setTemplateNameLookup(bool TemplateName) {
319:     TemplateNameLookup = TemplateName;
320:   }
321: 
322:   bool isTemplateNameLookup() const { return TemplateNameLookup; }
323: 
324:   bool isAmbiguous() const {
325:     return getResultKind() == LookupResultKind::Ambiguous;
326:   }
327: 
328:   /// Determines if this names a single result which is not an
329:   /// unresolved value using decl.  If so, it is safe to call
330:   /// getFoundDecl().
331:   bool isSingleResult() const {
332:     return getResultKind() == LookupResultKind::Found;
333:   }
334: 
335:   /// Determines if the results are overloaded.
336:   bool isOverloadedResult() const {
```
- EN: It exposes API surface such as `redeclarationKind`, `setAllowHidden`, `isHiddenDeclarationVisible`, `isForExternalRedeclaration`.
- 中文: 它暴露了 `redeclarationKind`, `setAllowHidden`, `isHiddenDeclarationVisible`, `isForExternalRedeclaration` 等接口。

### Lines 337-384

```cpp
337:     return getResultKind() == LookupResultKind::FoundOverloaded;
338:   }
339: 
340:   bool isUnresolvableResult() const {
341:     return getResultKind() == LookupResultKind::FoundUnresolvedValue;
342:   }
343: 
344:   LookupResultKind getResultKind() const {
345:     assert(checkDebugAssumptions());
346:     return ResultKind;
347:   }
348: 
349:   LookupAmbiguityKind getAmbiguityKind() const {
350:     assert(isAmbiguous());
351:     return Ambiguity;
352:   }
353: 
354:   const UnresolvedSetImpl &asUnresolvedSet() const {
355:     return Decls;
356:   }
357: 
358:   iterator begin() const { return iterator(Decls.begin()); }
359:   iterator end() const { return iterator(Decls.end()); }
360: 
361:   /// Return true if no decls were found
362:   bool empty() const { return Decls.empty(); }
363: 
364:   /// Return the base paths structure that's associated with
365:   /// these results, or null if none is.
366:   CXXBasePaths *getBasePaths() const {
367:     return Paths;
368:   }
369: 
370:   /// Determine whether the given declaration is visible to the
371:   /// program.
372:   static bool isVisible(Sema &SemaRef, NamedDecl *D);
373: 
374:   static bool isReachable(Sema &SemaRef, NamedDecl *D);
375: 
376:   static bool isAcceptable(Sema &SemaRef, NamedDecl *D,
377:                            Sema::AcceptableKind Kind) {
378:     return Kind == Sema::AcceptableKind::Visible ? isVisible(SemaRef, D)
379:                                                  : isReachable(SemaRef, D);
380:   }
381: 
382:   /// Determine whether this lookup is permitted to see the declaration.
383:   /// Note that a reachable but not visible declaration inhabiting a namespace
384:   /// is not allowed to be seen during name lookup.
```
- EN: It exposes API surface such as `isUnresolvableResult`, `getResultKind`, `assert`, `getAmbiguityKind`.
- 中文: 它暴露了 `isUnresolvableResult`, `getResultKind`, `assert`, `getAmbiguityKind` 等接口。

### Lines 385-432

```cpp
385:   ///
386:   /// For example:
387:   /// ```
388:   /// // m.cppm
389:   /// export module m;
390:   /// struct reachable { int v; }
391:   /// export auto func() { return reachable{43}; }
392:   /// // Use.cpp
393:   /// import m;
394:   /// auto Use() {
395:   ///   // Not valid. We couldn't see reachable here.
396:   ///   // So isAvailableForLookup would return false when we look
397:   ///   up 'reachable' here.
398:   ///   // return reachable(43).v;
399:   ///   // Valid. The field name 'v' is allowed during name lookup.
400:   ///   // So isAvailableForLookup would return true when we look up 'v' here.
401:   ///   return func().v;
402:   /// }
403:   /// ```
404:   static bool isAvailableForLookup(Sema &SemaRef, NamedDecl *ND);
405: 
406:   /// Retrieve the accepted (re)declaration of the given declaration,
407:   /// if there is one.
408:   NamedDecl *getAcceptableDecl(NamedDecl *D) const {
409:     if (!D->isInIdentifierNamespace(IDNS))
410:       return nullptr;
411: 
412:     if (isAvailableForLookup(getSema(), D) || isHiddenDeclarationVisible(D))
413:       return D;
414: 
415:     return getAcceptableDeclSlow(D);
416:   }
417: 
418: private:
419:   static bool isAcceptableSlow(Sema &SemaRef, NamedDecl *D,
420:                                Sema::AcceptableKind Kind);
421:   static bool isReachableSlow(Sema &SemaRef, NamedDecl *D);
422:   NamedDecl *getAcceptableDeclSlow(NamedDecl *D) const;
423: 
424: public:
425:   /// Returns the identifier namespace mask for this lookup.
426:   unsigned getIdentifierNamespace() const {
427:     return IDNS;
428:   }
429: 
430:   /// Returns whether these results arose from performing a
431:   /// lookup into a class.
432:   bool isClassLookup() const {
```
- EN: It exposes API surface such as `isAvailableForLookup`, `getAcceptableDecl`, `getAcceptableDeclSlow`, `isReachableSlow`.
- 中文: 它暴露了 `isAvailableForLookup`, `getAcceptableDecl`, `getAcceptableDeclSlow`, `isReachableSlow` 等接口。

### Lines 433-480

```cpp
433:     return NamingClass != nullptr;
434:   }
435: 
436:   /// Returns the 'naming class' for this lookup, i.e. the
437:   /// class which was looked into to find these results.
438:   ///
439:   /// C++0x [class.access.base]p5:
440:   ///   The access to a member is affected by the class in which the
441:   ///   member is named. This naming class is the class in which the
442:   ///   member name was looked up and found. [Note: this class can be
443:   ///   explicit, e.g., when a qualified-id is used, or implicit,
444:   ///   e.g., when a class member access operator (5.2.5) is used
445:   ///   (including cases where an implicit "this->" is added). If both
446:   ///   a class member access operator and a qualified-id are used to
447:   ///   name the member (as in p->T::m), the class naming the member
448:   ///   is the class named by the nested-name-specifier of the
449:   ///   qualified-id (that is, T). -- end note ]
450:   ///
451:   /// This is set by the lookup routines when they find results in a class.
452:   CXXRecordDecl *getNamingClass() const {
453:     return NamingClass;
454:   }
455: 
456:   /// Sets the 'naming class' for this lookup.
457:   void setNamingClass(CXXRecordDecl *Record) {
458:     NamingClass = Record;
459:   }
460: 
461:   /// Returns the base object type associated with this lookup;
462:   /// important for [class.protected].  Most lookups do not have an
463:   /// associated base object.
464:   QualType getBaseObjectType() const {
465:     return BaseObjectType;
466:   }
467: 
468:   /// Sets the base object type for this lookup.
469:   void setBaseObjectType(QualType T) {
470:     BaseObjectType = T;
471:   }
472: 
473:   /// Add a declaration to these results with its natural access.
474:   /// Does not test the acceptance criteria.
475:   void addDecl(NamedDecl *D) {
476:     addDecl(D, D->getAccess());
477:   }
478: 
479:   /// Add a declaration to these results with the given access.
480:   /// Does not test the acceptance criteria.
```
- EN: It exposes API surface such as `getNamingClass`, `setNamingClass`, `getBaseObjectType`, `setBaseObjectType`.
- 中文: 它暴露了 `getNamingClass`, `setNamingClass`, `getBaseObjectType`, `setBaseObjectType` 等接口。

### Lines 481-528

```cpp
481:   void addDecl(NamedDecl *D, AccessSpecifier AS) {
482:     Decls.addDecl(D, AS);
483:     ResultKind = LookupResultKind::Found;
484:   }
485: 
486:   /// Add all the declarations from another set of lookup
487:   /// results.
488:   void addAllDecls(const LookupResult &Other) {
489:     Decls.append(Other.Decls.begin(), Other.Decls.end());
490:     ResultKind = LookupResultKind::Found;
491:   }
492: 
493:   /// Determine whether no result was found because we could not
494:   /// search into dependent base classes of the current instantiation.
495:   bool wasNotFoundInCurrentInstantiation() const {
496:     return ResultKind == LookupResultKind::NotFoundInCurrentInstantiation;
497:   }
498: 
499:   /// Note that while no result was found in the current instantiation,
500:   /// there were dependent base classes that could not be searched.
501:   void setNotFoundInCurrentInstantiation() {
502:     assert((ResultKind == LookupResultKind::NotFound ||
503:             ResultKind == LookupResultKind::NotFoundInCurrentInstantiation) &&
504:            Decls.empty());
505:     ResultKind = LookupResultKind::NotFoundInCurrentInstantiation;
506:   }
507: 
508:   /// Determine whether the lookup result was shadowed by some other
509:   /// declaration that lookup ignored.
510:   bool isShadowed() const { return Shadowed; }
511: 
512:   /// Note that we found and ignored a declaration while performing
513:   /// lookup.
514:   void setShadowed() { Shadowed = true; }
515: 
516:   /// Resolves the result kind of the lookup, possibly hiding
517:   /// decls.
518:   ///
519:   /// This should be called in any environment where lookup might
520:   /// generate multiple lookup results.
521:   void resolveKind();
522: 
523:   /// Re-resolves the result kind of the lookup after a set of
524:   /// removals has been performed.
525:   void resolveKindAfterFilter() {
526:     if (Decls.empty()) {
527:       if (ResultKind != LookupResultKind::NotFoundInCurrentInstantiation)
528:         ResultKind = LookupResultKind::NotFound;
```
- EN: It exposes API surface such as `addDecl`, `addAllDecls`, `append`, `wasNotFoundInCurrentInstantiation`.
- 中文: 它暴露了 `addDecl`, `addAllDecls`, `append`, `wasNotFoundInCurrentInstantiation` 等接口。

### Lines 529-576

```cpp
529: 
530:       if (Paths) {
531:         deletePaths(Paths);
532:         Paths = nullptr;
533:       }
534:     } else {
535:       std::optional<LookupAmbiguityKind> SavedAK;
536:       bool WasAmbiguous = false;
537:       if (ResultKind == LookupResultKind::Ambiguous) {
538:         SavedAK = Ambiguity;
539:         WasAmbiguous = true;
540:       }
541:       ResultKind = LookupResultKind::Found;
542:       resolveKind();
543: 
544:       // If we didn't make the lookup unambiguous, restore the old
545:       // ambiguity kind.
546:       if (ResultKind == LookupResultKind::Ambiguous) {
547:         (void)WasAmbiguous;
548:         assert(WasAmbiguous);
549:         Ambiguity = *SavedAK;
550:       } else if (Paths) {
551:         deletePaths(Paths);
552:         Paths = nullptr;
553:       }
554:     }
555:   }
556: 
557:   template <class DeclClass>
558:   DeclClass *getAsSingle() const {
559:     if (getResultKind() != LookupResultKind::Found)
560:       return nullptr;
561:     return dyn_cast<DeclClass>(getFoundDecl());
562:   }
563: 
564:   /// Fetch the unique decl found by this lookup.  Asserts
565:   /// that one was found.
566:   ///
567:   /// This is intended for users who have examined the result kind
568:   /// and are certain that there is only one result.
569:   NamedDecl *getFoundDecl() const {
570:     assert(getResultKind() == LookupResultKind::Found &&
571:            "getFoundDecl called on non-unique result");
572:     return (*begin())->getUnderlyingDecl();
573:   }
574: 
575:   /// Fetches a representative decl.  Useful for lazy diagnostics.
576:   NamedDecl *getRepresentativeDecl() const {
```
- EN: Key type declarations here include `DeclClass`. It exposes API surface such as `deletePaths`, `resolveKind`, `assert`, `getAsSingle`.
- 中文: 这里的重要类型声明包括 `DeclClass`。 它暴露了 `deletePaths`, `resolveKind`, `assert`, `getAsSingle` 等接口。

### Lines 577-624

```cpp
577:     assert(!Decls.empty() && "cannot get representative of empty set");
578:     return *begin();
579:   }
580: 
581:   /// Asks if the result is a single tag decl.
582:   bool isSingleTagDecl() const {
583:     return getResultKind() == LookupResultKind::Found &&
584:            isa<TagDecl>(getFoundDecl());
585:   }
586: 
587:   /// Make these results show that the name was found in
588:   /// base classes of different types.
589:   ///
590:   /// The given paths object is copied and invalidated.
591:   void setAmbiguousBaseSubobjectTypes(CXXBasePaths &P);
592: 
593:   /// Make these results show that the name was found in
594:   /// distinct base classes of the same type.
595:   ///
596:   /// The given paths object is copied and invalidated.
597:   void setAmbiguousBaseSubobjects(CXXBasePaths &P);
598: 
599:   /// Make these results show that the name was found in
600:   /// different contexts and a tag decl was hidden by an ordinary
601:   /// decl in a different context.
602:   void setAmbiguousQualifiedTagHiding() {
603:     setAmbiguous(LookupAmbiguityKind::AmbiguousTagHiding);
604:   }
605: 
606:   /// Clears out any current state.
607:   LLVM_ATTRIBUTE_REINITIALIZES void clear() {
608:     ResultKind = LookupResultKind::NotFound;
609:     Decls.clear();
610:     if (Paths) deletePaths(Paths);
611:     Paths = nullptr;
612:     NamingClass = nullptr;
613:     Shadowed = false;
614:   }
615: 
616:   /// Clears out any current state and re-initializes for a
617:   /// different kind of lookup.
618:   void clear(Sema::LookupNameKind Kind) {
619:     clear();
620:     LookupKind = Kind;
621:     configure();
622:   }
623: 
624:   /// Change this lookup's redeclaration kind.
```
- EN: It exposes API surface such as `assert`, `begin`, `isSingleTagDecl`, `isa`.
- 中文: 它暴露了 `assert`, `begin`, `isSingleTagDecl`, `isa` 等接口。

### Lines 625-672

```cpp
625:   void setRedeclarationKind(RedeclarationKind RK) {
626:     Redecl = (RK != RedeclarationKind::NotForRedeclaration);
627:     ExternalRedecl = (RK == RedeclarationKind::ForExternalRedeclaration);
628:     configure();
629:   }
630: 
631:   void dump();
632:   void print(raw_ostream &);
633: 
634:   /// Suppress the diagnostics that would normally fire because of this
635:   /// lookup.  This happens during (e.g.) redeclaration lookups.
636:   void suppressDiagnostics() {
637:     DiagnoseAccess = false;
638:     DiagnoseAmbiguous = false;
639:   }
640: 
641:   /// Suppress the diagnostics that would normally fire because of this
642:   /// lookup due to access control violations.
643:   void suppressAccessDiagnostics() { DiagnoseAccess = false; }
644: 
645:   /// Determines whether this lookup is suppressing access control diagnostics.
646:   bool isSuppressingAccessDiagnostics() const { return !DiagnoseAccess; }
647: 
648:   /// Determines whether this lookup is suppressing ambiguous lookup
649:   /// diagnostics.
650:   bool isSuppressingAmbiguousDiagnostics() const { return !DiagnoseAmbiguous; }
651: 
652:   /// Sets a 'context' source range.
653:   void setContextRange(SourceRange SR) {
654:     NameContextRange = SR;
655:   }
656: 
657:   /// Gets the source range of the context of this name; for C++
658:   /// qualified lookups, this is the source range of the scope
659:   /// specifier.
660:   SourceRange getContextRange() const {
661:     return NameContextRange;
662:   }
663: 
664:   /// Gets the location of the identifier.  This isn't always defined:
665:   /// sometimes we're doing lookups on synthesized names.
666:   SourceLocation getNameLoc() const {
667:     return NameInfo.getLoc();
668:   }
669: 
670:   /// Get the Sema object that this lookup result is searching
671:   /// with.
672:   Sema &getSema() const { return *SemaPtr; }
```
- EN: It exposes API surface such as `setRedeclarationKind`, `configure`, `dump`, `print`.
- 中文: 它暴露了 `setRedeclarationKind`, `configure`, `dump`, `print` 等接口。

### Lines 673-720

```cpp
673: 
674:   /// A class for iterating through a result set and possibly
675:   /// filtering out results.  The results returned are possibly
676:   /// sugared.
677:   class Filter {
678:     friend class LookupResult;
679: 
680:     LookupResult &Results;
681:     LookupResult::iterator I;
682:     bool Changed = false;
683:     bool CalledDone = false;
684: 
685:     Filter(LookupResult &Results) : Results(Results), I(Results.begin()) {}
686: 
687:   public:
688:     Filter(Filter &&F)
689:         : Results(F.Results), I(F.I), Changed(F.Changed),
690:           CalledDone(F.CalledDone) {
691:       F.CalledDone = true;
692:     }
693: 
694:     // The move assignment operator is defined as deleted pending
695:     // further motivation.
696:     Filter &operator=(Filter &&) = delete;
697: 
698:     // The copy constrcutor and copy assignment operator is defined as deleted
699:     // pending further motivation.
700:     Filter(const Filter &) = delete;
701:     Filter &operator=(const Filter &) = delete;
702: 
703:     ~Filter() {
704:       assert(CalledDone &&
705:              "LookupResult::Filter destroyed without done() call");
706:     }
707: 
708:     bool hasNext() const {
709:       return I != Results.end();
710:     }
711: 
712:     NamedDecl *next() {
713:       assert(I != Results.end() && "next() called on empty filter");
714:       return *I++;
715:     }
716: 
717:     /// Restart the iteration.
718:     void restart() {
719:       I = Results.begin();
720:     }
```
- EN: Key type declarations here include `Filter`, `LookupResult`. It exposes API surface such as `Filter`, `CalledDone`, `~Filter`, `done`.
- 中文: 这里的重要类型声明包括 `Filter`, `LookupResult`。 它暴露了 `Filter`, `CalledDone`, `~Filter`, `done` 等接口。

### Lines 721-768

```cpp
721: 
722:     /// Erase the last element returned from this iterator.
723:     void erase() {
724:       Results.Decls.erase(--I);
725:       Changed = true;
726:     }
727: 
728:     /// Replaces the current entry with the given one, preserving the
729:     /// access bits.
730:     void replace(NamedDecl *D) {
731:       Results.Decls.replace(I-1, D);
732:       Changed = true;
733:     }
734: 
735:     /// Replaces the current entry with the given one.
736:     void replace(NamedDecl *D, AccessSpecifier AS) {
737:       Results.Decls.replace(I-1, D, AS);
738:       Changed = true;
739:     }
740: 
741:     void done() {
742:       assert(!CalledDone && "done() called twice");
743:       CalledDone = true;
744: 
745:       if (Changed)
746:         Results.resolveKindAfterFilter();
747:     }
748:   };
749: 
750:   /// Create a filter for this result set.
751:   Filter makeFilter() {
752:     return Filter(*this);
753:   }
754: 
755:   void setFindLocalExtern(bool FindLocalExtern) {
756:     if (FindLocalExtern)
757:       IDNS |= Decl::IDNS_LocalExtern;
758:     else
759:       IDNS &= ~Decl::IDNS_LocalExtern;
760:   }
761: 
762: private:
763:   void diagnoseAccess() {
764:     if (!isAmbiguous() && isClassLookup() &&
765:         getSema().getLangOpts().AccessControl)
766:       getSema().CheckLookupAccess(*this);
767:   }
768: 
```
- EN: It exposes API surface such as `erase`, `replace`, `done`, `assert`.
- 中文: 它暴露了 `erase`, `replace`, `done`, `assert` 等接口。

### Lines 769-816

```cpp
769:   void diagnoseAmbiguous() {
770:     if (isAmbiguous())
771:       getSema().DiagnoseAmbiguousLookup(*this);
772:   }
773: 
774:   void setAmbiguous(LookupAmbiguityKind AK) {
775:     ResultKind = LookupResultKind::Ambiguous;
776:     Ambiguity = AK;
777:   }
778: 
779:   void addDeclsFromBasePaths(const CXXBasePaths &P);
780:   void configure();
781: 
782:   bool checkDebugAssumptions() const;
783: 
784:   bool checkUnresolved() const {
785:     for (iterator I = begin(), E = end(); I != E; ++I)
786:       if (isa<UnresolvedUsingValueDecl>((*I)->getUnderlyingDecl()))
787:         return true;
788:     return false;
789:   }
790: 
791:   static void deletePaths(CXXBasePaths *);
792: 
793:   // Results.
794:   LookupResultKind ResultKind = LookupResultKind::NotFound;
795:   // ill-defined unless ambiguous. Still need to be initialized it will be
796:   // copied/moved.
797:   LookupAmbiguityKind Ambiguity = {};
798:   UnresolvedSet<8> Decls;
799:   CXXBasePaths *Paths = nullptr;
800:   CXXRecordDecl *NamingClass = nullptr;
801:   QualType BaseObjectType;
802: 
803:   // Parameters.
804:   Sema *SemaPtr;
805:   DeclarationNameInfo NameInfo;
806:   SourceRange NameContextRange;
807:   Sema::LookupNameKind LookupKind;
808:   unsigned IDNS = 0; // set by configure()
809: 
810:   bool Redecl;
811:   bool ExternalRedecl;
812: 
813:   /// True if tag declarations should be hidden if non-tags
814:   ///   are present
815:   bool HideTags = true;
816: 
```
- EN: It exposes API surface such as `diagnoseAmbiguous`, `getSema`, `setAmbiguous`, `addDeclsFromBasePaths`.
- 中文: 它暴露了 `diagnoseAmbiguous`, `getSema`, `setAmbiguous`, `addDeclsFromBasePaths` 等接口。

### Lines 817-864

```cpp
817:   bool DiagnoseAccess = false;
818:   bool DiagnoseAmbiguous = false;
819: 
820:   /// True if we should allow hidden declarations to be 'visible'.
821:   bool AllowHidden = false;
822: 
823:   /// True if the found declarations were shadowed by some other
824:   /// declaration that we skipped. This only happens when \c LookupKind
825:   /// is \c LookupRedeclarationWithLinkage.
826:   bool Shadowed = false;
827: 
828:   /// True if we're looking up a template-name.
829:   bool TemplateNameLookup = false;
830: };
831: 
832: /// Consumes visible declarations found when searching for
833: /// all visible names within a given scope or context.
834: ///
835: /// This abstract class is meant to be subclassed by clients of \c
836: /// Sema::LookupVisibleDecls(), each of which should override the \c
837: /// FoundDecl() function to process declarations as they are found.
838: class VisibleDeclConsumer {
839: public:
840:   /// Destroys the visible declaration consumer.
841:   virtual ~VisibleDeclConsumer();
842: 
843:   /// Determine whether hidden declarations (from unimported
844:   /// modules) should be given to this consumer. By default, they
845:   /// are not included.
846:   virtual bool includeHiddenDecls() const;
847: 
848:   /// Invoked each time \p Sema::LookupVisibleDecls() finds a
849:   /// declaration visible from the current scope or context.
850:   ///
851:   /// \param ND the declaration found.
852:   ///
853:   /// \param Hiding a declaration that hides the declaration \p ND,
854:   /// or NULL if no such declaration exists.
855:   ///
856:   /// \param Ctx the original context from which the lookup started.
857:   ///
858:   /// \param InBaseClass whether this declaration was found in base
859:   /// class of the context we searched.
860:   virtual void FoundDecl(NamedDecl *ND, NamedDecl *Hiding, DeclContext *Ctx,
861:                          bool InBaseClass) = 0;
862: 
863:   /// Callback to inform the client that Sema entered into a new context
864:   /// to find a visible declaration.
```
- EN: Key type declarations here include `VisibleDeclConsumer`. It exposes API surface such as `~VisibleDeclConsumer`, `includeHiddenDecls`.
- 中文: 这里的重要类型声明包括 `VisibleDeclConsumer`。 它暴露了 `~VisibleDeclConsumer`, `includeHiddenDecls` 等接口。

### Lines 865-900

```cpp
865:   //
866:   /// \param Ctx the context which Sema entered.
867:   virtual void EnteredContext(DeclContext *Ctx) {}
868: };
869: 
870: /// A class for storing results from argument-dependent lookup.
871: class ADLResult {
872: private:
873:   /// A map from canonical decls to the 'most recent' decl.
874:   llvm::MapVector<NamedDecl*, NamedDecl*> Decls;
875: 
876:   struct select_second {
877:     NamedDecl *operator()(std::pair<NamedDecl*, NamedDecl*> P) const {
878:       return P.second;
879:     }
880:   };
881: 
882: public:
883:   /// Adds a new ADL candidate to this map.
884:   void insert(NamedDecl *D);
885: 
886:   /// Removes any data associated with a given decl.
887:   void erase(NamedDecl *D) {
888:     Decls.erase(cast<NamedDecl>(D->getCanonicalDecl()));
889:   }
890: 
891:   using iterator =
892:       llvm::mapped_iterator<decltype(Decls)::iterator, select_second>;
893: 
894:   iterator begin() { return iterator(Decls.begin(), select_second()); }
895:   iterator end() { return iterator(Decls.end(), select_second()); }
896: };
897: 
898: } // namespace clang
899: 
900: #endif // LLVM_CLANG_SEMA_LOOKUP_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `clang`. Key type declarations here include `ADLResult`, `select_second`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `clang` 的命名空间作用域。 这里的重要类型声明包括 `ADLResult`, `select_second`。

## Key Concepts / 关键概念

- `CXXBasePaths`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `LookupResultKind`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `LookupAmbiguityKind`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `LookupResult`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `TemporaryToken`: An enumeration that captures a bounded set of modes, states, or categories. / 用于表达有限模式、状态或类别的枚举。
- `iterator`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。
- `DeclClass`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Filter`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/AST/Decl.h`, `clang/AST/DeclBase.h`, `clang/AST/DeclCXX.h`, `clang/AST/DeclarationName.h`, `clang/AST/Type.h`, `clang/AST/UnresolvedSet.h`, `clang/Basic/LLVM.h`, `clang/Basic/LangOptions.h`, `clang/Basic/SourceLocation.h`, `clang/Basic/Specifiers.h`, `clang/Sema/Sema.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/STLExtras.h`, `llvm/Support/Casting.h`, `cassert`
- Forward declarations / 前向声明: `CXXBasePaths`
- Namespace context / 命名空间上下文: `clang`
- Macro-style dependencies / 宏式依赖: None / 无
