# TemplateDeduction.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Sema/TemplateDeduction.h`
- Repository: `llvm-project`
- Purpose (EN): C++ template argument deduction.
- 用途（中文）: 该文件为 Sema 子系统中的 Template Deduction 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-33

```cpp
 1: //===- TemplateDeduction.h - C++ template argument deduction ----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file provides types used with Sema's template argument deduction
10: // routines.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #ifndef LLVM_CLANG_SEMA_TEMPLATEDEDUCTION_H
15: #define LLVM_CLANG_SEMA_TEMPLATEDEDUCTION_H
16: 
17: #include "clang/Sema/Ownership.h"
18: #include "clang/Sema/SemaConcept.h"
19: #include "clang/AST/ASTConcept.h"
20: #include "clang/AST/DeclAccessPair.h"
21: #include "clang/AST/DeclTemplate.h"
22: #include "clang/AST/TemplateBase.h"
23: #include "clang/Basic/PartialDiagnostic.h"
24: #include "clang/Basic/SourceLocation.h"
25: #include "llvm/ADT/SmallVector.h"
26: #include <cassert>
27: #include <cstddef>
28: #include <optional>
29: #include <utility>
30: 
31: namespace clang {
32: 
33: class Decl;
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/Sema/Ownership.h`, `clang/Sema/SemaConcept.h`, `clang/AST/ASTConcept.h` and 10 more. It opens, closes, or documents namespace scope for `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/Sema/Ownership.h`, `clang/Sema/SemaConcept.h`, `clang/AST/ASTConcept.h` 以及另外 10 项依赖。 它打开、关闭或说明了 `clang` 的命名空间作用域。

### Lines 34-66

```cpp
34: struct DeducedPack;
35: class Sema;
36: enum class TemplateDeductionResult;
37: 
38: namespace sema {
39: 
40: /// Provides information about an attempted template argument
41: /// deduction, whose success or failure was described by a
42: /// TemplateDeductionResult value.
43: class TemplateDeductionInfo {
44:   /// The deduced template argument list.
45:   TemplateArgumentList *DeducedSugared = nullptr, *DeducedCanonical = nullptr;
46: 
47:   /// The source location at which template argument
48:   /// deduction is occurring.
49:   SourceLocation Loc;
50: 
51:   /// Have we suppressed an error during deduction?
52:   bool HasSFINAEDiagnostic = false;
53: 
54:   /// Have we matched any packs on the parameter side, versus any non-packs on
55:   /// the argument side, in a context where the opposite matching is also
56:   /// allowed?
57:   bool StrictPackMatch = false;
58: 
59:   /// The template parameter depth for which we're performing deduction.
60:   unsigned DeducedDepth;
61: 
62:   /// The number of parameters with explicitly-specified template arguments,
63:   /// up to and including the partially-specified pack (if any).
64:   unsigned ExplicitArgs = 0;
65: 
66:   /// Warnings (and follow-on notes) that were suppressed due to
```
- EN: It opens, closes, or documents namespace scope for `sema`. Key type declarations here include `DeducedPack`, `Sema`, `TemplateDeductionResult`, `TemplateDeductionInfo`. It introduces enum-based state or option sets such as `TemplateDeductionResult`.
- 中文: 它打开、关闭或说明了 `sema` 的命名空间作用域。 这里的重要类型声明包括 `DeducedPack`, `Sema`, `TemplateDeductionResult`, `TemplateDeductionInfo`。 它引入了 `TemplateDeductionResult` 等基于枚举的状态或选项集合。

### Lines 67-99

```cpp
67:   /// SFINAE while performing template argument deduction.
68:   SmallVector<PartialDiagnosticAt, 4> SuppressedDiagnostics;
69: 
70: public:
71:   TemplateDeductionInfo(SourceLocation Loc, unsigned DeducedDepth = 0)
72:       : Loc(Loc), DeducedDepth(DeducedDepth) {}
73:   TemplateDeductionInfo(const TemplateDeductionInfo &) = delete;
74:   TemplateDeductionInfo &operator=(const TemplateDeductionInfo &) = delete;
75: 
76:   enum ForBaseTag { ForBase };
77:   /// Create temporary template deduction info for speculatively deducing
78:   /// against a base class of an argument's type.
79:   TemplateDeductionInfo(ForBaseTag, const TemplateDeductionInfo &Info)
80:       : DeducedSugared(Info.DeducedSugared), Loc(Info.Loc),
81:         DeducedDepth(Info.DeducedDepth), ExplicitArgs(Info.ExplicitArgs) {}
82: 
83:   /// Returns the location at which template argument is
84:   /// occurring.
85:   SourceLocation getLocation() const {
86:     return Loc;
87:   }
88: 
89:   /// The depth of template parameters for which deduction is being
90:   /// performed.
91:   unsigned getDeducedDepth() const {
92:     return DeducedDepth;
93:   }
94: 
95:   bool hasStrictPackMatch() const { return StrictPackMatch; }
96: 
97:   void setStrictPackMatch() { StrictPackMatch = true; }
98: 
99:   /// Get the number of explicitly-specified arguments.
```
- EN: It introduces enum-based state or option sets such as `ForBaseTag`. It exposes API surface such as `Loc`, `TemplateDeductionInfo`, `DeducedDepth`, `getLocation`.
- 中文: 它引入了 `ForBaseTag` 等基于枚举的状态或选项集合。 它暴露了 `Loc`, `TemplateDeductionInfo`, `DeducedDepth`, `getLocation` 等接口。

### Lines 100-132

```cpp
100:   unsigned getNumExplicitArgs() const {
101:     return ExplicitArgs;
102:   }
103: 
104:   /// Take ownership of the deduced template argument lists.
105:   TemplateArgumentList *takeSugared() {
106:     TemplateArgumentList *Result = DeducedSugared;
107:     DeducedSugared = nullptr;
108:     return Result;
109:   }
110:   TemplateArgumentList *takeCanonical() {
111:     TemplateArgumentList *Result = DeducedCanonical;
112:     DeducedCanonical = nullptr;
113:     return Result;
114:   }
115: 
116:   /// Take ownership of the SFINAE diagnostic.
117:   void takeSFINAEDiagnostic(PartialDiagnosticAt &PD) {
118:     assert(HasSFINAEDiagnostic);
119:     PD.first = SuppressedDiagnostics.front().first;
120:     PD.second.swap(SuppressedDiagnostics.front().second);
121:     clearSFINAEDiagnostic();
122:   }
123: 
124:   /// Discard any SFINAE diagnostics.
125:   void clearSFINAEDiagnostic() {
126:     SuppressedDiagnostics.clear();
127:     HasSFINAEDiagnostic = false;
128:   }
129: 
130:   /// Peek at the SFINAE diagnostic.
131:   const PartialDiagnosticAt &peekSFINAEDiagnostic() const {
132:     assert(HasSFINAEDiagnostic);
```
- EN: It exposes API surface such as `getNumExplicitArgs`, `takeSugared`, `takeCanonical`, `takeSFINAEDiagnostic`.
- 中文: 它暴露了 `getNumExplicitArgs`, `takeSugared`, `takeCanonical`, `takeSFINAEDiagnostic` 等接口。

### Lines 133-165

```cpp
133:     return SuppressedDiagnostics.front();
134:   }
135: 
136:   /// Provide an initial template argument list that contains the
137:   /// explicitly-specified arguments.
138:   void setExplicitArgs(TemplateArgumentList *NewDeducedSugared,
139:                        TemplateArgumentList *NewDeducedCanonical) {
140:     assert(NewDeducedSugared->size() == NewDeducedCanonical->size());
141:     DeducedSugared = NewDeducedSugared;
142:     DeducedCanonical = NewDeducedCanonical;
143:     ExplicitArgs = DeducedSugared->size();
144:   }
145: 
146:   /// Provide a new template argument list that contains the
147:   /// results of template argument deduction.
148:   void reset(TemplateArgumentList *NewDeducedSugared,
149:              TemplateArgumentList *NewDeducedCanonical) {
150:     DeducedSugared = NewDeducedSugared;
151:     DeducedCanonical = NewDeducedCanonical;
152:   }
153: 
154:   /// Is a SFINAE diagnostic available?
155:   bool hasSFINAEDiagnostic() const {
156:     return HasSFINAEDiagnostic;
157:   }
158: 
159:   /// Set the diagnostic which caused the SFINAE failure.
160:   void addSFINAEDiagnostic(SourceLocation Loc, PartialDiagnostic PD) {
161:     // Only collect the first diagnostic.
162:     if (HasSFINAEDiagnostic)
163:       return;
164:     SuppressedDiagnostics.clear();
165:     SuppressedDiagnostics.emplace_back(Loc, std::move(PD));
```
- EN: It exposes API surface such as `front`, `assert`, `size`, `hasSFINAEDiagnostic`.
- 中文: 它暴露了 `front`, `assert`, `size`, `hasSFINAEDiagnostic` 等接口。

### Lines 166-198

```cpp
166:     HasSFINAEDiagnostic = true;
167:   }
168: 
169:   /// Add a new diagnostic to the set of diagnostics
170:   void addSuppressedDiagnostic(SourceLocation Loc,
171:                                PartialDiagnostic PD) {
172:     if (HasSFINAEDiagnostic)
173:       return;
174:     SuppressedDiagnostics.emplace_back(Loc, std::move(PD));
175:   }
176: 
177:   /// Iterator over the set of suppressed diagnostics.
178:   using diag_iterator = SmallVectorImpl<PartialDiagnosticAt>::const_iterator;
179: 
180:   /// Returns an iterator at the beginning of the sequence of suppressed
181:   /// diagnostics.
182:   diag_iterator diag_begin() const { return SuppressedDiagnostics.begin(); }
183: 
184:   /// Returns an iterator at the end of the sequence of suppressed
185:   /// diagnostics.
186:   diag_iterator diag_end() const { return SuppressedDiagnostics.end(); }
187: 
188:   /// The template parameter to which a template argument
189:   /// deduction failure refers.
190:   ///
191:   /// Depending on the result of template argument deduction, this
192:   /// template parameter may have different meanings:
193:   ///
194:   ///   TDK_Incomplete: this is the first template parameter whose
195:   ///   corresponding template argument was not deduced.
196:   ///
197:   ///   TDK_IncompletePack: this is the expanded parameter pack for
198:   ///   which we deduced too few arguments.
```
- EN: It defines convenient aliases such as `diag_iterator`. It exposes API surface such as `emplace_back`, `diag_begin`, `diag_end`.
- 中文: 它定义了 `diag_iterator` 等便捷别名。 它暴露了 `emplace_back`, `diag_begin`, `diag_end` 等接口。

### Lines 199-231

```cpp
199:   ///
200:   ///   TDK_Inconsistent: this is the template parameter for which
201:   ///   two different template argument values were deduced.
202:   TemplateParameter Param;
203: 
204:   /// The first template argument to which the template
205:   /// argument deduction failure refers.
206:   ///
207:   /// Depending on the result of the template argument deduction,
208:   /// this template argument may have different meanings:
209:   ///
210:   ///   TDK_IncompletePack: this is the number of arguments we deduced
211:   ///   for the pack.
212:   ///
213:   ///   TDK_Inconsistent: this argument is the first value deduced
214:   ///   for the corresponding template parameter.
215:   ///
216:   ///   TDK_SubstitutionFailure: this argument is the template
217:   ///   argument we were instantiating when we encountered an error.
218:   ///
219:   ///   TDK_DeducedMismatch: this is the parameter type, after substituting
220:   ///   deduced arguments.
221:   ///
222:   ///   TDK_NonDeducedMismatch: this is the component of the 'parameter'
223:   ///   of the deduction, directly provided in the source code.
224:   TemplateArgument FirstArg;
225: 
226:   /// The second template argument to which the template
227:   /// argument deduction failure refers.
228:   ///
229:   ///   TDK_Inconsistent: this argument is the second value deduced
230:   ///   for the corresponding template parameter.
231:   ///
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 232-264

```cpp
232:   ///   TDK_DeducedMismatch: this is the (adjusted) call argument type.
233:   ///
234:   ///   TDK_NonDeducedMismatch: this is the mismatching component of the
235:   ///   'argument' of the deduction, from which we are deducing arguments.
236:   ///
237:   /// FIXME: Finish documenting this.
238:   TemplateArgument SecondArg;
239: 
240:   /// The index of the function argument that caused a deduction
241:   /// failure.
242:   ///
243:   ///   TDK_DeducedMismatch: this is the index of the argument that had a
244:   ///   different argument type from its substituted parameter type.
245:   unsigned CallArgIndex = 0;
246: 
247:   // C++20 [over.match.class.deduct]p5.2:
248:   //   During template argument deduction for the aggregate deduction
249:   //   candidate, the number of elements in a trailing parameter pack is only
250:   //   deduced from the number of remaining function arguments if it is not
251:   //   otherwise deduced.
252:   bool AggregateDeductionCandidateHasMismatchedArity = false;
253: 
254:   /// Information on packs that we're currently expanding.
255:   ///
256:   /// FIXME: This should be kept internal to SemaTemplateDeduction.
257:   SmallVector<DeducedPack *, 8> PendingDeducedPacks;
258: 
259:   /// \brief The constraint satisfaction details resulting from the associated
260:   /// constraints satisfaction tests.
261:   ConstraintSatisfaction AssociatedConstraintsSatisfaction;
262: };
263: 
264: } // namespace sema
```
- EN: It opens, closes, or documents namespace scope for `sema`.
- 中文: 它打开、关闭或说明了 `sema` 的命名空间作用域。

### Lines 265-297

```cpp
265: 
266: /// A structure used to record information about a failed
267: /// template argument deduction, for diagnosis.
268: struct DeductionFailureInfo {
269:   /// A Sema::TemplateDeductionResult.
270:   unsigned Result : 8;
271: 
272:   /// Indicates whether a diagnostic is stored in Diagnostic.
273:   unsigned HasDiagnostic : 1;
274: 
275:   /// Opaque pointer containing additional data about
276:   /// this deduction failure.
277:   void *Data;
278: 
279:   /// A diagnostic indicating why deduction failed.
280:   alignas(PartialDiagnosticAt) char Diagnostic[sizeof(PartialDiagnosticAt)];
281: 
282:   /// Retrieve the diagnostic which caused this deduction failure,
283:   /// if any.
284:   PartialDiagnosticAt *getSFINAEDiagnostic();
285: 
286:   /// Retrieve the template parameter this deduction failure
287:   /// refers to, if any.
288:   TemplateParameter getTemplateParameter();
289: 
290:   /// Retrieve the template argument list associated with this
291:   /// deduction failure, if any.
292:   TemplateArgumentList *getTemplateArgumentList();
293: 
294:   /// Return the first template argument this deduction failure
295:   /// refers to, if any.
296:   const TemplateArgument *getFirstArg();
297: 
```
- EN: Key type declarations here include `DeductionFailureInfo`. It exposes API surface such as `getSFINAEDiagnostic`, `getTemplateParameter`, `getTemplateArgumentList`, `getFirstArg`.
- 中文: 这里的重要类型声明包括 `DeductionFailureInfo`。 它暴露了 `getSFINAEDiagnostic`, `getTemplateParameter`, `getTemplateArgumentList`, `getFirstArg` 等接口。

### Lines 298-330

```cpp
298:   /// Return the second template argument this deduction failure
299:   /// refers to, if any.
300:   const TemplateArgument *getSecondArg();
301: 
302:   /// Return the index of the call argument that this deduction
303:   /// failure refers to, if any.
304:   UnsignedOrNone getCallArgIndex();
305: 
306:   /// Free any memory associated with this deduction failure.
307:   void Destroy();
308: 
309:   TemplateDeductionResult getResult() const {
310:     return static_cast<TemplateDeductionResult>(Result);
311:   }
312: };
313: 
314: /// TemplateSpecCandidate - This is a generalization of OverloadCandidate
315: /// which keeps track of template argument deduction failure info, when
316: /// handling explicit specializations (and instantiations) of templates
317: /// beyond function overloading.
318: /// For now, assume that the candidates are non-matching specializations.
319: /// TODO: In the future, we may need to unify/generalize this with
320: /// OverloadCandidate.
321: struct TemplateSpecCandidate {
322:   /// The declaration that was looked up, together with its access.
323:   /// Might be a UsingShadowDecl, but usually a FunctionTemplateDecl.
324:   DeclAccessPair FoundDecl;
325: 
326:   /// Specialization - The actual specialization that this candidate
327:   /// represents. When NULL, this may be a built-in candidate.
328:   Decl *Specialization;
329: 
330:   /// Template argument deduction info
```
- EN: Key type declarations here include `TemplateSpecCandidate`. It exposes API surface such as `getSecondArg`, `getCallArgIndex`, `Destroy`, `getResult`.
- 中文: 这里的重要类型声明包括 `TemplateSpecCandidate`。 它暴露了 `getSecondArg`, `getCallArgIndex`, `Destroy`, `getResult` 等接口。

### Lines 331-363

```cpp
331:   DeductionFailureInfo DeductionFailure;
332: 
333:   void set(DeclAccessPair Found, Decl *Spec, DeductionFailureInfo Info) {
334:     FoundDecl = Found;
335:     Specialization = Spec;
336:     DeductionFailure = Info;
337:   }
338: 
339:   /// Diagnose a template argument deduction failure.
340:   void NoteDeductionFailure(Sema &S, bool ForTakingAddress);
341: };
342: 
343: /// TemplateSpecCandidateSet - A set of generalized overload candidates,
344: /// used in template specializations.
345: /// TODO: In the future, we may need to unify/generalize this with
346: /// OverloadCandidateSet.
347: class TemplateSpecCandidateSet {
348:   SmallVector<TemplateSpecCandidate, 16> Candidates;
349:   SourceLocation Loc;
350: 
351:   // Stores whether we're taking the address of these candidates. This helps us
352:   // produce better error messages when dealing with the pass_object_size
353:   // attribute on parameters.
354:   bool ForTakingAddress;
355: 
356:   void destroyCandidates();
357: 
358: public:
359:   TemplateSpecCandidateSet(SourceLocation Loc, bool ForTakingAddress = false)
360:       : Loc(Loc), ForTakingAddress(ForTakingAddress) {}
361:   TemplateSpecCandidateSet(const TemplateSpecCandidateSet &) = delete;
362:   TemplateSpecCandidateSet &
363:   operator=(const TemplateSpecCandidateSet &) = delete;
```
- EN: Key type declarations here include `TemplateSpecCandidateSet`. It exposes API surface such as `set`, `NoteDeductionFailure`, `destroyCandidates`, `Loc`.
- 中文: 这里的重要类型声明包括 `TemplateSpecCandidateSet`。 它暴露了 `set`, `NoteDeductionFailure`, `destroyCandidates`, `Loc` 等接口。

### Lines 364-396

```cpp
364:   ~TemplateSpecCandidateSet() { destroyCandidates(); }
365: 
366:   SourceLocation getLocation() const { return Loc; }
367: 
368:   /// Clear out all of the candidates.
369:   /// TODO: This may be unnecessary.
370:   void clear();
371: 
372:   using iterator = SmallVector<TemplateSpecCandidate, 16>::iterator;
373: 
374:   iterator begin() { return Candidates.begin(); }
375:   iterator end() { return Candidates.end(); }
376: 
377:   size_t size() const { return Candidates.size(); }
378:   bool empty() const { return Candidates.empty(); }
379: 
380:   /// Add a new candidate with NumConversions conversion sequence slots
381:   /// to the overload set.
382:   TemplateSpecCandidate &addCandidate() {
383:     Candidates.emplace_back();
384:     return Candidates.back();
385:   }
386: 
387:   void NoteCandidates(Sema &S, SourceLocation Loc);
388: 
389:   void NoteCandidates(Sema &S, SourceLocation Loc) const {
390:     const_cast<TemplateSpecCandidateSet *>(this)->NoteCandidates(S, Loc);
391:   }
392: };
393: 
394: } // namespace clang
395: 
396: #endif // LLVM_CLANG_SEMA_TEMPLATEDEDUCTION_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `clang`. It defines convenient aliases such as `iterator`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `clang` 的命名空间作用域。 它定义了 `iterator` 等便捷别名。

## Key Concepts / 关键概念

- `Decl`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `DeducedPack`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Sema`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `TemplateDeductionResult`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `TemplateDeductionInfo`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ForBaseTag`: An enumeration that captures a bounded set of modes, states, or categories. / 用于表达有限模式、状态或类别的枚举。
- `diag_iterator`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。
- `DeductionFailureInfo`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/Sema/Ownership.h`, `clang/Sema/SemaConcept.h`, `clang/AST/ASTConcept.h`, `clang/AST/DeclAccessPair.h`, `clang/AST/DeclTemplate.h`, `clang/AST/TemplateBase.h`, `clang/Basic/PartialDiagnostic.h`, `clang/Basic/SourceLocation.h`, `llvm/ADT/SmallVector.h`, `cassert`, `cstddef`, `optional`, `utility`
- Forward declarations / 前向声明: `Decl`, `DeducedPack`, `Sema`, `TemplateDeductionResult`
- Namespace context / 命名空间上下文: `clang`, `sema`
- Macro-style dependencies / 宏式依赖: None / 无
