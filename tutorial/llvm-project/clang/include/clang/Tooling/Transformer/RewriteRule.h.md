# RewriteRule.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Tooling/Transformer/RewriteRule.h`
- Repository: `llvm-project`
- Purpose (EN): RewriteRule class.
- 用途（中文）: 该文件为 Tooling::Transformer 子系统中的 Rewrite Rule 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-45

```cpp
 1: //===--- RewriteRule.h - RewriteRule class ----------------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: ///  \file
10: ///  Defines the RewriteRule class and related functions for creating,
11: ///  modifying and interpreting RewriteRules.
12: ///
13: //===----------------------------------------------------------------------===//
14: 
15: #ifndef LLVM_CLANG_TOOLING_TRANSFORMER_REWRITERULE_H
16: #define LLVM_CLANG_TOOLING_TRANSFORMER_REWRITERULE_H
17: 
18: #include "clang/ASTMatchers/ASTMatchFinder.h"
19: #include "clang/ASTMatchers/ASTMatchers.h"
20: #include "clang/ASTMatchers/ASTMatchersInternal.h"
21: #include "clang/Tooling/Refactoring/AtomicChange.h"
22: #include "clang/Tooling/Transformer/MatchConsumer.h"
23: #include "clang/Tooling/Transformer/RangeSelector.h"
24: #include "llvm/ADT/Any.h"
25: #include "llvm/ADT/STLExtras.h"
26: #include "llvm/ADT/SmallVector.h"
27: #include "llvm/Support/Error.h"
28: #include <functional>
29: #include <string>
30: #include <utility>
31: 
32: namespace clang {
33: namespace transformer {
34: // Specifies how to interpret an edit.
35: enum class EditKind {
36:   // Edits a source range in the file.
37:   Range,
38:   // Inserts an include in the file. The `Replacement` field is the name of the
39:   // newly included file.
40:   AddInclude,
41: };
42: 
43: /// A concrete description of a source edit, represented by a character range in
44: /// the source to be replaced and a corresponding replacement string.
45: struct Edit {
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/ASTMatchers/ASTMatchFinder.h`, `clang/ASTMatchers/ASTMatchers.h`, `clang/ASTMatchers/ASTMatchersInternal.h` and 10 more. It opens, closes, or documents namespace scope for `clang`, `transformer`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/ASTMatchers/ASTMatchFinder.h`, `clang/ASTMatchers/ASTMatchers.h`, `clang/ASTMatchers/ASTMatchersInternal.h` 以及另外 10 项依赖。 它打开、关闭或说明了 `clang`, `transformer` 的命名空间作用域。

### Lines 46-90

```cpp
46:   EditKind Kind = EditKind::Range;
47:   CharSourceRange Range;
48:   std::string Replacement;
49:   std::string Note;
50:   llvm::Any Metadata;
51: };
52: 
53: /// Format of the path in an include directive -- angle brackets or quotes.
54: enum class IncludeFormat {
55:   Quoted,
56:   Angled,
57: };
58: 
59: /// Maps a match result to a list of concrete edits (with possible
60: /// failure). This type is a building block of rewrite rules, but users will
61: /// generally work in terms of `ASTEdit`s (below) rather than directly in terms
62: /// of `EditGenerator`.
63: using EditGenerator = MatchConsumer<llvm::SmallVector<Edit, 1>>;
64: 
65: template <typename T> using Generator = std::shared_ptr<MatchComputation<T>>;
66: 
67: using TextGenerator = Generator<std::string>;
68: 
69: using AnyGenerator = MatchConsumer<llvm::Any>;
70: 
71: // Description of a source-code edit, expressed in terms of an AST node.
72: // Includes: an ID for the (bound) node, a selector for source related to the
73: // node, a replacement and, optionally, an explanation for the edit.
74: //
75: // * Target: the source code impacted by the rule. This identifies an AST node,
76: //   or part thereof (\c Part), whose source range indicates the extent of the
77: //   replacement applied by the replacement term.  By default, the extent is the
78: //   node matched by the pattern term (\c NodePart::Node). Target's are typed
79: //   (\c Kind), which guides the determination of the node extent.
80: //
81: // * Replacement: a function that produces a replacement string for the target,
82: //   based on the match result.
83: //
84: // * Note: (optional) a note specifically for this edit, potentially referencing
85: //   elements of the match.  This will be displayed to the user, where possible;
86: //   for example, in clang-tidy diagnostics.  Use of notes should be rare --
87: //   explanations of the entire rewrite should be set in the rule
88: //   (`RewriteRule::Explanation`) instead.  Notes serve the rare cases wherein
89: //   edit-specific diagnostics are required.
90: //
```
- EN: Key type declarations here include `IncludeFormat`. It introduces enum-based state or option sets such as `IncludeFormat`. It defines convenient aliases such as `EditGenerator`, `Generator`, `TextGenerator`, `AnyGenerator`.
- 中文: 这里的重要类型声明包括 `IncludeFormat`。 它引入了 `IncludeFormat` 等基于枚举的状态或选项集合。 它定义了 `EditGenerator`, `Generator`, `TextGenerator`, `AnyGenerator` 等便捷别名。

### Lines 91-135

```cpp
 91: // `ASTEdit` should be built using the `change` convenience functions. For
 92: // example,
 93: // \code
 94: //   changeTo(name(fun), cat("Frodo"))
 95: // \endcode
 96: // Or, if we use Stencil for the TextGenerator:
 97: // \code
 98: //   using stencil::cat;
 99: //   changeTo(statement(thenNode), cat("{", thenNode, "}"))
100: //   changeTo(callArgs(call), cat(x, ",", y))
101: // \endcode
102: // Or, if you are changing the node corresponding to the rule's matcher, you can
103: // use the single-argument override of \c change:
104: // \code
105: //   changeTo(cat("different_expr"))
106: // \endcode
107: struct ASTEdit {
108:   EditKind Kind = EditKind::Range;
109:   RangeSelector TargetRange;
110:   TextGenerator Replacement;
111:   TextGenerator Note;
112:   // Not all transformations will want or need to attach metadata and therefore
113:   // should not be required to do so.
114:   AnyGenerator Metadata = [](const ast_matchers::MatchFinder::MatchResult &)
115:       -> llvm::Expected<llvm::Any> {
116:     return llvm::Expected<llvm::Any>(llvm::Any());
117:   };
118: };
119: 
120: /// Generates a single (specified) edit.
121: EditGenerator edit(ASTEdit E);
122: 
123: /// Lifts a list of `ASTEdit`s into an `EditGenerator`.
124: ///
125: /// The `EditGenerator` will return an empty vector if any of the edits apply to
126: /// portions of the source that are ineligible for rewriting (certain
127: /// interactions with macros, for example) and it will fail if any invariants
128: /// are violated relating to bound nodes in the match.  However, it does not
129: /// fail in the case of conflicting edits -- conflict handling is left to
130: /// clients.  We recommend use of the \c AtomicChange or \c Replacements classes
131: /// for assistance in detecting such conflicts.
132: EditGenerator editList(llvm::SmallVector<ASTEdit, 1> Edits);
133: 
134: /// Generates no edits.
135: inline EditGenerator noEdits() { return editList({}); }
```
- EN: Key type declarations here include `ASTEdit`. It exposes API surface such as `Any>`, `edit`, `editList`, `noEdits`.
- 中文: 这里的重要类型声明包括 `ASTEdit`。 它暴露了 `Any>`, `edit`, `editList`, `noEdits` 等接口。

### Lines 136-180

```cpp
136: 
137: /// Generates a single, no-op edit anchored at the start location of the
138: /// specified range. A `noopEdit` may be preferred over `noEdits` to associate a
139: /// diagnostic `Explanation` with the rule.
140: EditGenerator noopEdit(RangeSelector Anchor);
141: 
142: /// Generates a single, no-op edit with the associated note anchored at the
143: /// start location of the specified range.
144: ASTEdit note(RangeSelector Anchor, TextGenerator Note);
145: 
146: /// Version of `ifBound` specialized to `ASTEdit`.
147: inline EditGenerator ifBound(std::string ID, ASTEdit TrueEdit,
148:                              ASTEdit FalseEdit) {
149:   return ifBound(std::move(ID), edit(std::move(TrueEdit)),
150:                  edit(std::move(FalseEdit)));
151: }
152: 
153: /// Version of `ifBound` that has no "False" branch. If the node is not bound,
154: /// then no edits are produced.
155: inline EditGenerator ifBound(std::string ID, ASTEdit TrueEdit) {
156:   return ifBound(std::move(ID), edit(std::move(TrueEdit)), noEdits());
157: }
158: 
159: /// Flattens a list of generators into a single generator whose elements are the
160: /// concatenation of the results of the argument generators.
161: EditGenerator flattenVector(SmallVector<EditGenerator, 2> Generators);
162: 
163: namespace detail {
164: /// Helper function to construct an \c EditGenerator. Overloaded for common
165: /// cases so that user doesn't need to specify which factory function to
166: /// use. This pattern gives benefits similar to implicit constructors, while
167: /// maintaing a higher degree of explicitness.
168: inline EditGenerator injectEdits(ASTEdit E) { return edit(std::move(E)); }
169: inline EditGenerator injectEdits(EditGenerator G) { return G; }
170: } // namespace detail
171: 
172: template <typename... Ts> EditGenerator flatten(Ts &&...Edits) {
173:   return flattenVector({detail::injectEdits(std::forward<Ts>(Edits))...});
174: }
175: 
176: // Every rewrite rule is triggered by a match against some AST node.
177: // Transformer guarantees that this ID is bound to the triggering node whenever
178: // a rewrite rule is applied.
179: extern const char RootID[];
180: 
```
- EN: It opens, closes, or documents namespace scope for `detail`. It exposes API surface such as `noopEdit`, `note`, `edit`, `ifBound`.
- 中文: 它打开、关闭或说明了 `detail` 的命名空间作用域。 它暴露了 `noopEdit`, `note`, `edit`, `ifBound` 等接口。

### Lines 181-225

```cpp
181: /// Replaces a portion of the source text with \p Replacement.
182: ASTEdit changeTo(RangeSelector Target, TextGenerator Replacement);
183: /// DEPRECATED: use \c changeTo.
184: inline ASTEdit change(RangeSelector Target, TextGenerator Replacement) {
185:   return changeTo(std::move(Target), std::move(Replacement));
186: }
187: 
188: /// Replaces the entirety of a RewriteRule's match with \p Replacement.  For
189: /// example, to replace a function call, one could write:
190: /// \code
191: ///   makeRule(callExpr(callee(functionDecl(hasName("foo")))),
192: ///            changeTo(cat("bar()")))
193: /// \endcode
194: inline ASTEdit changeTo(TextGenerator Replacement) {
195:   return changeTo(node(RootID), std::move(Replacement));
196: }
197: /// DEPRECATED: use \c changeTo.
198: inline ASTEdit change(TextGenerator Replacement) {
199:   return changeTo(std::move(Replacement));
200: }
201: 
202: /// Inserts \p Replacement before \p S, leaving the source selected by \S
203: /// unchanged.
204: inline ASTEdit insertBefore(RangeSelector S, TextGenerator Replacement) {
205:   return changeTo(before(std::move(S)), std::move(Replacement));
206: }
207: 
208: /// Inserts \p Replacement after \p S, leaving the source selected by \S
209: /// unchanged.
210: inline ASTEdit insertAfter(RangeSelector S, TextGenerator Replacement) {
211:   return changeTo(after(std::move(S)), std::move(Replacement));
212: }
213: 
214: /// Removes the source selected by \p S.
215: ASTEdit remove(RangeSelector S);
216: 
217: /// Adds an include directive for the given header to the file of `Target`. The
218: /// particular location specified by `Target` is ignored.
219: ASTEdit addInclude(RangeSelector Target, StringRef Header,
220:                    IncludeFormat Format = IncludeFormat::Quoted);
221: 
222: /// Adds an include directive for the given header to the file associated with
223: /// `RootID`. If `RootID` matches inside a macro expansion, will add the
224: /// directive to the file in which the macro was expanded (as opposed to the
225: /// file in which the macro is defined).
```
- EN: It exposes API surface such as `changeTo`, `change`, `insertBefore`, `insertAfter`.
- 中文: 它暴露了 `changeTo`, `change`, `insertBefore`, `insertAfter` 等接口。

### Lines 226-270

```cpp
226: inline ASTEdit addInclude(StringRef Header,
227:                           IncludeFormat Format = IncludeFormat::Quoted) {
228:   return addInclude(expansion(node(RootID)), Header, Format);
229: }
230: 
231: // FIXME: If `Metadata` returns an `llvm::Expected<T>` the `AnyGenerator` will
232: // construct an `llvm::Expected<llvm::Any>` where no error is present but the
233: // `llvm::Any` holds the error. This is unlikely but potentially surprising.
234: // Perhaps the `llvm::Expected` should be unwrapped, or perhaps this should be a
235: // compile-time error. No solution here is perfect.
236: //
237: // Note: This function template accepts any type callable with a MatchResult
238: // rather than a `std::function` because the return-type needs to be deduced. If
239: // it accepted a `std::function<R(MatchResult)>`, lambdas or other callable
240: // types would not be able to deduce `R`, and users would be forced to specify
241: // explicitly the type they intended to return by wrapping the lambda at the
242: // call-site.
243: template <typename Callable>
244: inline ASTEdit withMetadata(ASTEdit Edit, Callable Metadata) {
245:   Edit.Metadata =
246:       [Gen = std::move(Metadata)](
247:           const ast_matchers::MatchFinder::MatchResult &R) -> llvm::Any {
248:     return Gen(R);
249:   };
250: 
251:   return Edit;
252: }
253: 
254: /// Assuming that the inner range is enclosed by the outer range, creates
255: /// precision edits to remove the parts of the outer range that are not included
256: /// in the inner range.
257: inline EditGenerator shrinkTo(RangeSelector outer, RangeSelector inner) {
258:   return editList({remove(enclose(before(outer), before(inner))),
259:                    remove(enclose(after(inner), after(outer)))});
260: }
261: 
262: /// Description of a source-code transformation.
263: //
264: // A *rewrite rule* describes a transformation of source code. A simple rule
265: // contains each of the following components:
266: //
267: // * Matcher: the pattern term, expressed as clang matchers (with Transformer
268: //   extensions).
269: //
270: // * Edits: a set of Edits to the source code, described with ASTEdits.
```
- EN: It exposes API surface such as `addInclude`, `withMetadata`, `Gen`, `shrinkTo`.
- 中文: 它暴露了 `addInclude`, `withMetadata`, `Gen`, `shrinkTo` 等接口。

### Lines 271-315

```cpp
271: //
272: // However, rules can also consist of (sub)rules, where the first that matches
273: // is applied and the rest are ignored.  So, the above components together form
274: // a logical "case" and a rule is a sequence of cases.
275: //
276: // Rule cases have an additional, implicit, component: the parameters. These are
277: // portions of the pattern which are left unspecified, yet bound in the pattern
278: // so that we can reference them in the edits.
279: //
280: // The \c Transformer class can be used to apply the rewrite rule and obtain the
281: // corresponding replacements.
282: struct RewriteRuleBase {
283:   struct Case {
284:     ast_matchers::internal::DynTypedMatcher Matcher;
285:     EditGenerator Edits;
286:   };
287:   // We expect RewriteRules will most commonly include only one case.
288:   SmallVector<Case, 1> Cases;
289: };
290: 
291: /// A source-code transformation with accompanying metadata.
292: ///
293: /// When a case of the rule matches, the \c Transformer invokes the
294: /// corresponding metadata generator and provides it alongside the edits.
295: template <typename MetadataT> struct RewriteRuleWith : RewriteRuleBase {
296:   SmallVector<Generator<MetadataT>, 1> Metadata;
297: };
298: 
299: template <> struct RewriteRuleWith<void> : RewriteRuleBase {};
300: 
301: using RewriteRule = RewriteRuleWith<void>;
302: 
303: namespace detail {
304: 
305: RewriteRule makeRule(ast_matchers::internal::DynTypedMatcher M,
306:                      EditGenerator Edits);
307: 
308: template <typename MetadataT>
309: RewriteRuleWith<MetadataT> makeRule(ast_matchers::internal::DynTypedMatcher M,
310:                                     EditGenerator Edits,
311:                                     Generator<MetadataT> Metadata) {
312:   RewriteRuleWith<MetadataT> R;
313:   R.Cases = {{std::move(M), std::move(Edits)}};
314:   R.Metadata = {std::move(Metadata)};
315:   return R;
```
- EN: It opens, closes, or documents namespace scope for `detail`. Key type declarations here include `RewriteRuleBase`, `Case`, `RewriteRuleWith`. It defines convenient aliases such as `RewriteRule`.
- 中文: 它打开、关闭或说明了 `detail` 的命名空间作用域。 这里的重要类型声明包括 `RewriteRuleBase`, `Case`, `RewriteRuleWith`。 它定义了 `RewriteRule` 等便捷别名。

### Lines 316-360

```cpp
316: }
317: 
318: inline EditGenerator makeEditGenerator(EditGenerator Edits) { return Edits; }
319: EditGenerator makeEditGenerator(llvm::SmallVector<ASTEdit, 1> Edits);
320: EditGenerator makeEditGenerator(ASTEdit Edit);
321: 
322: } // namespace detail
323: 
324: /// Constructs a simple \c RewriteRule. \c Edits can be an \c EditGenerator,
325: /// multiple \c ASTEdits, or a single \c ASTEdit.
326: /// @{
327: template <int &..., typename EditsT>
328: RewriteRule makeRule(ast_matchers::internal::DynTypedMatcher M,
329:                      EditsT &&Edits) {
330:   return detail::makeRule(
331:       std::move(M), detail::makeEditGenerator(std::forward<EditsT>(Edits)));
332: }
333: 
334: RewriteRule makeRule(ast_matchers::internal::DynTypedMatcher M,
335:                      std::initializer_list<ASTEdit> Edits);
336: /// @}
337: 
338: /// Overloads of \c makeRule that also generate metadata when matching.
339: /// @{
340: template <typename MetadataT, int &..., typename EditsT>
341: RewriteRuleWith<MetadataT> makeRule(ast_matchers::internal::DynTypedMatcher M,
342:                                     EditsT &&Edits,
343:                                     Generator<MetadataT> Metadata) {
344:   return detail::makeRule(
345:       std::move(M), detail::makeEditGenerator(std::forward<EditsT>(Edits)),
346:       std::move(Metadata));
347: }
348: 
349: template <typename MetadataT>
350: RewriteRuleWith<MetadataT> makeRule(ast_matchers::internal::DynTypedMatcher M,
351:                                     std::initializer_list<ASTEdit> Edits,
352:                                     Generator<MetadataT> Metadata) {
353:   return detail::makeRule(std::move(M),
354:                           detail::makeEditGenerator(std::move(Edits)),
355:                           std::move(Metadata));
356: }
357: /// @}
358: 
359: /// For every case in Rule, adds an include directive for the given header. The
360: /// common use is assumed to be a rule with only one case. For example, to
```
- EN: It opens, closes, or documents namespace scope for `detail`. It exposes API surface such as `makeEditGenerator`, `move`.
- 中文: 它打开、关闭或说明了 `detail` 的命名空间作用域。 它暴露了 `makeEditGenerator`, `move` 等接口。

### Lines 361-405

```cpp
361: /// replace a function call and add headers corresponding to the new code, one
362: /// could write:
363: /// \code
364: ///   auto R = makeRule(callExpr(callee(functionDecl(hasName("foo")))),
365: ///            changeTo(cat("bar()")));
366: ///   addInclude(R, "path/to/bar_header.h");
367: ///   addInclude(R, "vector", IncludeFormat::Angled);
368: /// \endcode
369: void addInclude(RewriteRuleBase &Rule, llvm::StringRef Header,
370:                 IncludeFormat Format = IncludeFormat::Quoted);
371: 
372: /// Applies the first rule whose pattern matches; other rules are ignored.  If
373: /// the matchers are independent then order doesn't matter. In that case,
374: /// `applyFirst` is simply joining the set of rules into one.
375: //
376: // `applyFirst` is like an `anyOf` matcher with an edit action attached to each
377: // of its cases. Anywhere you'd use `anyOf(m1.bind("id1"), m2.bind("id2"))` and
378: // then dispatch on those ids in your code for control flow, `applyFirst` lifts
379: // that behavior to the rule level.  So, you can write `applyFirst({makeRule(m1,
380: // action1), makeRule(m2, action2), ...});`
381: //
382: // For example, consider a type `T` with a deterministic serialization function,
383: // `serialize()`.  For performance reasons, we would like to make it
384: // non-deterministic.  Therefore, we want to drop the expectation that
385: // `a.serialize() = b.serialize() iff a = b` (although we'll maintain
386: // `deserialize(a.serialize()) = a`).
387: //
388: // We have three cases to consider (for some equality function, `eq`):
389: // ```
390: // eq(a.serialize(), b.serialize()) --> eq(a,b)
391: // eq(a, b.serialize())             --> eq(deserialize(a), b)
392: // eq(a.serialize(), b)             --> eq(a, deserialize(b))
393: // ```
394: //
395: // `applyFirst` allows us to specify each independently:
396: // ```
397: // auto eq_fun = functionDecl(...);
398: // auto method_call = cxxMemberCallExpr(...);
399: //
400: // auto two_calls = callExpr(callee(eq_fun), hasArgument(0, method_call),
401: //                           hasArgument(1, method_call));
402: // auto left_call =
403: //     callExpr(callee(eq_fun), callExpr(hasArgument(0, method_call)));
404: // auto right_call =
405: //     callExpr(callee(eq_fun), callExpr(hasArgument(1, method_call)));
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 406-450

```cpp
406: //
407: // RewriteRule R = applyFirst({makeRule(two_calls, two_calls_action),
408: //                             makeRule(left_call, left_call_action),
409: //                             makeRule(right_call, right_call_action)});
410: // ```
411: /// @{
412: template <typename MetadataT>
413: RewriteRuleWith<MetadataT>
414: applyFirst(ArrayRef<RewriteRuleWith<MetadataT>> Rules) {
415:   RewriteRuleWith<MetadataT> R;
416:   for (auto &Rule : Rules) {
417:     assert(Rule.Cases.size() == Rule.Metadata.size() &&
418:            "mis-match in case and metadata array size");
419:     R.Cases.append(Rule.Cases.begin(), Rule.Cases.end());
420:     R.Metadata.append(Rule.Metadata.begin(), Rule.Metadata.end());
421:   }
422:   return R;
423: }
424: 
425: template <>
426: RewriteRuleWith<void> applyFirst(ArrayRef<RewriteRuleWith<void>> Rules);
427: 
428: template <typename MetadataT>
429: RewriteRuleWith<MetadataT>
430: applyFirst(const std::vector<RewriteRuleWith<MetadataT>> &Rules) {
431:   return applyFirst(llvm::ArrayRef(Rules));
432: }
433: 
434: template <typename MetadataT>
435: RewriteRuleWith<MetadataT>
436: applyFirst(std::initializer_list<RewriteRuleWith<MetadataT>> Rules) {
437:   return applyFirst(llvm::ArrayRef(Rules.begin(), Rules.end()));
438: }
439: /// @}
440: 
441: /// Converts a \c RewriteRuleWith<T> to a \c RewriteRule by stripping off the
442: /// metadata generators.
443: template <int &..., typename MetadataT>
444: std::enable_if_t<!std::is_same<MetadataT, void>::value, RewriteRule>
445: stripMetadata(RewriteRuleWith<MetadataT> Rule) {
446:   RewriteRule R;
447:   R.Cases = std::move(Rule.Cases);
448:   return R;
449: }
450: 
```
- EN: It exposes API surface such as `applyFirst`, `append`, `stripMetadata`, `move`.
- 中文: 它暴露了 `applyFirst`, `append`, `stripMetadata`, `move` 等接口。

### Lines 451-495

```cpp
451: /// Applies `Rule` to all descendants of the node bound to `NodeId`. `Rule` can
452: /// refer to nodes bound by the calling rule. `Rule` is not applied to the node
453: /// itself.
454: ///
455: /// For example,
456: /// ```
457: /// auto InlineX =
458: ///     makeRule(declRefExpr(to(varDecl(hasName("x")))), changeTo(cat("3")));
459: /// makeRule(functionDecl(hasName("f"), hasBody(stmt().bind("body"))).bind("f"),
460: ///          flatten(
461: ///            changeTo(name("f"), cat("newName")),
462: ///            rewriteDescendants("body", InlineX)));
463: /// ```
464: /// Here, we find the function `f`, change its name to `newName` and change all
465: /// appearances of `x` in its body to `3`.
466: EditGenerator rewriteDescendants(std::string NodeId, RewriteRule Rule);
467: 
468: /// The following three functions are a low-level part of the RewriteRule
469: /// API. We expose them for use in implementing the fixtures that interpret
470: /// RewriteRule, like Transformer and TransfomerTidy, or for more advanced
471: /// users.
472: //
473: // FIXME: These functions are really public, if advanced, elements of the
474: // RewriteRule API.  Recast them as such.  Or, just declare these functions
475: // public and well-supported and move them out of `detail`.
476: namespace detail {
477: /// The following overload set is a version of `rewriteDescendants` that
478: /// operates directly on the AST, rather than generating a Transformer
479: /// combinator. It applies `Rule` to all descendants of `Node`, although not
480: /// `Node` itself. `Rule` can refer to nodes bound in `Result`.
481: ///
482: /// For example, assuming that "body" is bound to a function body in MatchResult
483: /// `Results`, this will produce edits to change all appearances of `x` in that
484: /// body to `3`.
485: /// ```
486: /// auto InlineX =
487: ///     makeRule(declRefExpr(to(varDecl(hasName("x")))), changeTo(cat("3")));
488: /// const auto *Node = Results.Nodes.getNodeAs<Stmt>("body");
489: /// auto Edits = rewriteDescendants(*Node, InlineX, Results);
490: /// ```
491: /// @{
492: llvm::Expected<SmallVector<Edit, 1>>
493: rewriteDescendants(const Decl &Node, RewriteRule Rule,
494:                    const ast_matchers::MatchFinder::MatchResult &Result);
495: 
```
- EN: It opens, closes, or documents namespace scope for `detail`. It exposes API surface such as `rewriteDescendants`.
- 中文: 它打开、关闭或说明了 `detail` 的命名空间作用域。 它暴露了 `rewriteDescendants` 等接口。

### Lines 496-540

```cpp
496: llvm::Expected<SmallVector<Edit, 1>>
497: rewriteDescendants(const Stmt &Node, RewriteRule Rule,
498:                    const ast_matchers::MatchFinder::MatchResult &Result);
499: 
500: llvm::Expected<SmallVector<Edit, 1>>
501: rewriteDescendants(const TypeLoc &Node, RewriteRule Rule,
502:                    const ast_matchers::MatchFinder::MatchResult &Result);
503: 
504: llvm::Expected<SmallVector<Edit, 1>>
505: rewriteDescendants(const DynTypedNode &Node, RewriteRule Rule,
506:                    const ast_matchers::MatchFinder::MatchResult &Result);
507: /// @}
508: 
509: /// Builds a single matcher for the rule, covering all of the rule's cases.
510: /// Only supports Rules whose cases' matchers share the same base "kind"
511: /// (`Stmt`, `Decl`, etc.)  Deprecated: use `buildMatchers` instead, which
512: /// supports mixing matchers of different kinds.
513: ast_matchers::internal::DynTypedMatcher
514: buildMatcher(const RewriteRuleBase &Rule);
515: 
516: /// Builds a set of matchers that cover the rule.
517: ///
518: /// One matcher is built for each distinct node matcher base kind: Stmt, Decl,
519: /// etc. Node-matchers for `QualType` and `Type` are not permitted, since such
520: /// nodes carry no source location information and are therefore not relevant
521: /// for rewriting. If any such matchers are included, will return an empty
522: /// vector.
523: std::vector<ast_matchers::internal::DynTypedMatcher>
524: buildMatchers(const RewriteRuleBase &Rule);
525: 
526: /// Gets the beginning location of the source matched by a rewrite rule. If the
527: /// match occurs within a macro expansion, returns the beginning of the
528: /// expansion point. `Result` must come from the matching of a rewrite rule.
529: SourceLocation
530: getRuleMatchLoc(const ast_matchers::MatchFinder::MatchResult &Result);
531: 
532: /// Returns the index of the \c Case of \c Rule that was selected in the match
533: /// result. Assumes a matcher built with \c buildMatcher.
534: size_t findSelectedCase(const ast_matchers::MatchFinder::MatchResult &Result,
535:                         const RewriteRuleBase &Rule);
536: } // namespace detail
537: } // namespace transformer
538: } // namespace clang
539: 
540: #endif // LLVM_CLANG_TOOLING_TRANSFORMER_REWRITERULE_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `detail`, `transformer`, `clang`. It exposes API surface such as `buildMatcher`, `buildMatchers`, `getRuleMatchLoc`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `detail`, `transformer`, `clang` 的命名空间作用域。 它暴露了 `buildMatcher`, `buildMatchers`, `getRuleMatchLoc` 等接口。

## Key Concepts / 关键概念

- `EditKind`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Edit`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `IncludeFormat`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `EditGenerator`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。
- `Generator`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。
- `TextGenerator`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。
- `AnyGenerator`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。
- `ASTEdit`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/ASTMatchers/ASTMatchFinder.h`, `clang/ASTMatchers/ASTMatchers.h`, `clang/ASTMatchers/ASTMatchersInternal.h`, `clang/Tooling/Refactoring/AtomicChange.h`, `clang/Tooling/Transformer/MatchConsumer.h`, `clang/Tooling/Transformer/RangeSelector.h`, `llvm/ADT/Any.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/Support/Error.h`, `functional`, `string`, `utility`
- Forward declarations / 前向声明: None / 无
- Namespace context / 命名空间上下文: `clang`, `transformer`, `detail`
- Macro-style dependencies / 宏式依赖: None / 无
