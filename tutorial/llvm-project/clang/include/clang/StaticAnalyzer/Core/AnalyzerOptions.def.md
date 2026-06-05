# AnalyzerOptions.def — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/StaticAnalyzer/Core/AnalyzerOptions.def`
- Repository: `llvm-project`
- Purpose (EN): Metadata about Static Analyses.
- 用途（中文）: 该文件为 StaticAnalyzer::Core 子系统中的 Analyzer Options 提供可重复展开的宏定义列表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48

```cpp
 1: //===-- AnalyzerOptions.def - Metadata about Static Analyses ----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //  This file defines the analyzer options avaible with -analyzer-config.
10: //  Note that clang/docs/tools/generate_analyzer_options_docs.py relies on the
11: //  structure of this file, so if this file is refactored, then make sure to
12: //  update that script as well.
13: //
14: //===----------------------------------------------------------------------===//
15: 
16: #ifndef LLVM_CLANG_STATICANALYZER_CORE_ANALYZEROPTIONS_H
17: #error This .def file is expected to be included in translation units where \
18: "clang/StaticAnalyzer/Core/AnalyzerOptions.h" is already included!
19: #endif
20: 
21: #ifdef ANALYZER_OPTION
22: #ifndef ANALYZER_OPTION_DEPENDS_ON_USER_MODE
23: #error If you didnt include this file with the intent of generating methods, \
24: define both 'ANALYZER_OPTION' and 'ANALYZER_OPTION_DEPENDS_ON_USER_MODE' macros!
25: #endif
26: #endif
27: 
28: #ifndef ANALYZER_OPTION_DEPENDS_ON_USER_MODE
29: #ifdef ANALYZER_OPTION
30: #error If you didnt include this file with the intent of generating methods, \
31: define both 'ANALYZER_OPTION' and 'ANALYZER_OPTION_DEPENDS_ON_USER_MODE' macros!
32: #endif
33: #endif
34: 
35: #ifndef ANALYZER_OPTION
36: /// Create a new analyzer option, but dont generate a method for it in
37: /// AnalyzerOptions.
38: ///
39: ///   TYPE - The type of the option object that will be stored in
40: ///          AnalyzerOptions. This file is expected to be icluded in translation
41: ///          units where AnalyzerOptions.h is included, so types from that
42: ///          header should be used.
43: ///   NAME - The name of the option object.
44: ///   CMDFLAG - The command line flag for the option.
45: ///             (-analyzer-config CMDFLAG=VALUE)
46: ///   DESC - Description of the flag.
47: ///   DEFAULT_VAL - The default value for CMDFLAG.
48: #define ANALYZER_OPTION(TYPE, NAME, CMDFLAG, DESC, DEFAULT_VAL)
```
- EN: This range establishes include guards or other file-scope compilation boundaries.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。

### Lines 49-96

```cpp
49: #endif
50: 
51: #ifndef ANALYZER_OPTION_DEPENDS_ON_USER_MODE
52: /// Create a new analyzer option, but dont generate a method for it in
53: /// AnalyzerOptions. It's value depends on the option "user-mode".
54: ///
55: ///   TYPE - The type of the option object that will be stored in
56: ///          AnalyzerOptions. This file is expected to be icluded in translation
57: ///          units where AnalyzerOptions.h is included, so types from that
58: ///          header should be used.
59: ///   NAME - The name of the option object.
60: ///   CMDFLAG - The command line flag for the option.
61: ///             (-analyzer-config CMDFLAG=VALUE)
62: ///   DESC - Description of the flag.
63: ///   SHALLOW_VAL - The default value for CMDFLAG, when "user-mode" was set to
64: ///                 "shallow".
65: ///   DEEP_VAL - The default value for CMDFLAG, when "user-mode" was set to
66: ///              "deep".
67: #define ANALYZER_OPTION_DEPENDS_ON_USER_MODE(TYPE, NAME, CMDFLAG, DESC,        \
68:                                              SHALLOW_VAL, DEEP_VAL)
69: #endif
70: 
71: //===----------------------------------------------------------------------===//
72: // The "mode" option. Since some options depend on this, we list it on top of
73: // this file in order to make sure that the generated field for it is
74: // initialized before the rest.
75: //===----------------------------------------------------------------------===//
76: 
77: ANALYZER_OPTION(
78:     StringRef, UserMode, "mode",
79:     "(string) Controls the high-level analyzer mode, which influences the "
80:     "default settings for some of the lower-level config options (such as "
81:     "IPAMode). Value: \"deep\", \"shallow\".",
82:     "deep")
83: 
84: //===----------------------------------------------------------------------===//
85: // Boolean analyzer options.
86: //===----------------------------------------------------------------------===//
87: 
88: ANALYZER_OPTION(bool, ShouldIncludeImplicitDtorsInCFG, "cfg-implicit-dtors",
89:                 "Whether or not implicit destructors for C++ objects "
90:                 "should be included in the CFG.",
91:                 true)
92: 
93: ANALYZER_OPTION(bool, ShouldIncludeTemporaryDtorsInCFG, "cfg-temporary-dtors",
94:                 "Whether or not the destructors for C++ temporary "
95:                 "objects should be included in the CFG.",
96:                 true)
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This section enumerates macro-driven entries through `ANALYZER_OPTION`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一段通过 `ANALYZER_OPTION` 等宏列出可重复展开的条目。

### Lines 97-144

```cpp
 97: 
 98: ANALYZER_OPTION(
 99:     bool, ShouldIncludeLifetimeInCFG, "cfg-lifetime",
100:     "Whether or not end-of-lifetime information should be included in the CFG.",
101:     false)
102: 
103: ANALYZER_OPTION(bool, ShouldIncludeLoopExitInCFG, "cfg-loopexit",
104:                 "Whether or not the end of the loop information should "
105:                 "be included in the CFG.",
106:                 false)
107: 
108: ANALYZER_OPTION(bool, ShouldIncludeRichConstructorsInCFG,
109:                 "cfg-rich-constructors",
110:                 "Whether or not construction site information should be "
111:                 "included in the CFG C++ constructor elements.",
112:                 true)
113: 
114: ANALYZER_OPTION(
115:     bool, ShouldIncludeScopesInCFG, "cfg-scopes",
116:     "Whether or not scope information should be included in the CFG.", false)
117: 
118: ANALYZER_OPTION(bool, ShouldIncludeDefaultInitForAggregates,
119:                 "cfg-expand-default-aggr-inits",
120:                 "Whether or not inline CXXDefaultInitializers for aggregate "
121:                 "initialization in the CFG.",
122:                 false)
123: 
124: ANALYZER_OPTION(
125:     bool, MayInlineTemplateFunctions, "c++-template-inlining",
126:     "Whether or not templated functions may be considered for inlining.", true)
127: 
128: ANALYZER_OPTION(bool, MayInlineCXXStandardLibrary, "c++-stdlib-inlining",
129:                 "Whether or not C++ standard library functions may be "
130:                 "considered for inlining.",
131:                 true)
132: 
133: ANALYZER_OPTION(bool, MayInlineCXXAllocator, "c++-allocator-inlining",
134:                 "Whether or not allocator and deallocator calls may be "
135:                 "considered for inlining.",
136:                 true)
137: 
138: ANALYZER_OPTION(
139:     bool, MayInlineCXXSharedPtrDtor, "c++-shared_ptr-inlining",
140:     "Whether or not the destructor of C++ 'shared_ptr' may be considered for "
141:     "inlining. This covers std::shared_ptr, std::tr1::shared_ptr, and "
142:     "boost::shared_ptr, and indeed any destructor named '~shared_ptr'.",
143:     false)
144: 
```
- EN: This section enumerates macro-driven entries through `ANALYZER_OPTION`.
- 中文: 这一段通过 `ANALYZER_OPTION` 等宏列出可重复展开的条目。

### Lines 145-192

```cpp
145: ANALYZER_OPTION(bool, MayInlineCXXTemporaryDtors, "c++-temp-dtor-inlining",
146:                 "Whether C++ temporary destructors should be inlined "
147:                 "during analysis. If temporary destructors are disabled "
148:                 "in the CFG via the 'cfg-temporary-dtors' option, "
149:                 "temporary destructors would not be inlined anyway.",
150:                 true)
151: 
152: ANALYZER_OPTION(
153:     bool, ShouldSuppressNullReturnPaths, "suppress-null-return-paths",
154:     "Whether or not paths that go through null returns should be suppressed. "
155:     "This is a heuristic for avoiding bug reports with paths that go through "
156:     "inlined functions that are more defensive than their callers.",
157:     true)
158: 
159: ANALYZER_OPTION(
160:     bool, ShouldAvoidSuppressingNullArgumentPaths,
161:     "avoid-suppressing-null-argument-paths",
162:     "Whether a bug report should not be suppressed if its path includes a call "
163:     "with a null argument, even if that call has a null return. This option "
164:     "has no effect when ShouldSuppressNullReturnPaths is false. This is a "
165:     "counter-heuristic to avoid false negatives.",
166:     false)
167: 
168: ANALYZER_OPTION(bool, ShouldSuppressInlinedDefensiveChecks,
169:                 "suppress-inlined-defensive-checks",
170:                 "Whether or not diagnostics containing inlined "
171:                 "defensive NULL checks should be suppressed.",
172:                 true)
173: 
174: ANALYZER_OPTION(bool, MayInlineCXXContainerMethods, "c++-container-inlining",
175:                 "Whether or not methods of C++ container objects may be "
176:                 "considered for inlining.",
177:                 false)
178: 
179: ANALYZER_OPTION(bool, ShouldSuppressFromCXXStandardLibrary,
180:                 "suppress-c++-stdlib",
181:                 "Whether or not diagnostics reported within the C++ "
182:                 "standard library should be suppressed.",
183:                 true)
184: 
185: ANALYZER_OPTION(bool, ShouldCrosscheckWithZ3, "crosscheck-with-z3",
186:                 "Whether bug reports should be crosschecked with the Z3 "
187:                 "constraint manager backend.",
188:                 false)
189: 
190: ANALYZER_OPTION(
191:     unsigned, Z3CrosscheckEQClassTimeoutThreshold,
192:     "crosscheck-with-z3-eqclass-timeout-threshold",
```
- EN: This section enumerates macro-driven entries through `ANALYZER_OPTION`.
- 中文: 这一段通过 `ANALYZER_OPTION` 等宏列出可重复展开的条目。

### Lines 193-240

```cpp
193:     "Set a timeout for bug report equivalence classes in milliseconds. "
194:     "If we exhaust this threshold, we will drop the bug report eqclass "
195:     "instead of doing more Z3 queries. Setting this to 700 ms in conjunction "
196:     "with \"crosscheck-with-z3-timeout-threshold\" of 300 ms, would nicely "
197:     "guarantee that no bug report equivalence class can take longer than "
198:     "1 second, effectively mitigating Z3 hangs during refutation. "
199:     "If there were Z3 retries, only the minimum query time is considered "
200:     "when accumulating query times within a report equivalence class. "
201:     "Set 0 for no timeout.", 0)
202: 
203: ANALYZER_OPTION(
204:     unsigned, Z3CrosscheckTimeoutThreshold,
205:     "crosscheck-with-z3-timeout-threshold",
206:     "Set a timeout for individual Z3 queries in milliseconds. "
207:     "On fast machines, 300 worked well in some cases. "
208:     "The lower it is, the higher the chances of having flaky issues. "
209:     "Having no timeout may hang the analyzer indefinitely. "
210:     "Set 0 for no timeout.", 15'000)
211: 
212: ANALYZER_OPTION(
213:     unsigned, Z3CrosscheckRLimitThreshold,
214:     "crosscheck-with-z3-rlimit-threshold",
215:     "Set the Z3 resource limit threshold. This sets a supposedly deterministic "
216:     "cutoff point for Z3 queries, as longer queries usually consume more "
217:     "resources. "
218:     "400'000 should on average make Z3 queries run for up to 100ms on modern "
219:     "hardware. Set 0 for unlimited.", 0)
220: 
221: ANALYZER_OPTION(
222:     PositiveAnalyzerOption, Z3CrosscheckMaxAttemptsPerQuery,
223:     "crosscheck-with-z3-max-attempts-per-query",
224:     "Set how many times the oracle is allowed to run a Z3 query. "
225:     "This must be a positive value. Set 1 to not allow any retry attempts. "
226:     "Increasing the number of attempts is often more effective at reducing "
227:     "the number of nondeterministic diagnostics than "
228:     "\"crosscheck-with-z3-timeout-threshold\" in practice.", 3)
229: 
230: ANALYZER_OPTION(bool, ShouldReportIssuesInMainSourceFile,
231:                 "report-in-main-source-file",
232:                 "Whether or not the diagnostic report should be always "
233:                 "reported in the main source file and not the headers.",
234:                 false)
235: 
236: ANALYZER_OPTION(bool, ShouldWriteStableReportFilename, "stable-report-filename",
237:                 "Deprecated: report filenames are now always stable. "
238:                 "See also 'verbose-report-filename'.",
239:                 false)
240: 
```
- EN: Key type declarations here include `can`. This section enumerates macro-driven entries through `ANALYZER_OPTION`.
- 中文: 这里的重要类型声明包括 `can`。 这一段通过 `ANALYZER_OPTION` 等宏列出可重复展开的条目。

### Lines 241-288

```cpp
241: ANALYZER_OPTION(bool, ShouldWriteVerboseReportFilename, "verbose-report-filename",
242:                 "Whether or not the report filename should contain extra "
243:                 "information about the issue.",
244:                 false)
245: 
246: ANALYZER_OPTION(
247:     bool, ShouldSerializeStats, "serialize-stats",
248:     "Whether the analyzer should serialize statistics to plist output. "
249:     "Statistics would be serialized in JSON format inside the main dictionary "
250:     "under the statistics key. Available only if compiled in assert mode or "
251:     "with LLVM statistics explicitly enabled.",
252:     false)
253: 
254: ANALYZER_OPTION(bool, MayInlineObjCMethod, "objc-inlining",
255:                 "Whether ObjectiveC inlining is enabled, false otherwise.",
256:                 true)
257: 
258: ANALYZER_OPTION(bool, ShouldPrunePaths, "prune-paths",
259:                 "Whether irrelevant parts of a bug report path should "
260:                 "be pruned out of the final output.",
261:                 true)
262: 
263: ANALYZER_OPTION(bool, ShouldAddPopUpNotes, "add-pop-up-notes",
264:                 "Whether pop-up notes should be added to the final output.",
265:                 true)
266: 
267: ANALYZER_OPTION(
268:     bool, ShouldConditionalizeStaticInitializers,
269:     "cfg-conditional-static-initializers",
270:     "Whether 'static' initializers should be in conditional logic in the CFG.",
271:     true)
272: 
273: ANALYZER_OPTION(bool, ShouldSynthesizeBodies, "faux-bodies",
274:                 "Whether the analyzer engine should synthesize fake "
275:                 "bodies for well-known functions.",
276:                 true)
277: 
278: ANALYZER_OPTION(
279:     bool, ShouldElideConstructors, "elide-constructors",
280:     "Whether elidable C++ copy-constructors and move-constructors should be "
281:     "actually elided during analysis. Both behaviors are allowed by the C++ "
282:     "standard, and the analyzer, like CodeGen, defaults to eliding. Starting "
283:     "with C++17 some elisions become mandatory, and in these cases the option "
284:     "will be ignored.",
285:     true)
286: 
287: ANALYZER_OPTION(
288:     bool, ShouldInlineLambdas, "inline-lambdas",
```
- EN: This section enumerates macro-driven entries through `ANALYZER_OPTION`.
- 中文: 这一段通过 `ANALYZER_OPTION` 等宏列出可重复展开的条目。

### Lines 289-336

```cpp
289:     "Whether lambdas should be inlined. Otherwise a sink node will be "
290:     "generated each time a LambdaExpr is visited.",
291:     true)
292: 
293: ANALYZER_OPTION(bool, ShouldWidenLoops, "widen-loops",
294:                 "Whether the analysis should try to widen loops.", false)
295: 
296: ANALYZER_OPTION(
297:     bool, ShouldUnrollLoops, "unroll-loops",
298:     "Whether the analysis should try to unroll loops with known bounds.", false)
299: 
300: ANALYZER_OPTION(
301:     bool, ShouldAssumeAtLeastOneIteration, "assume-at-least-one-iteration",
302:     "Whether the analyzer should always assume at least one iteration in "
303:     "loops where the loop condition is opaque (i.e. the analyzer cannot "
304:     "determine if it's true or false). Setting this to true eliminates some "
305:     "false positives (where e.g. a structure is nonempty, but the analyzer "
306:     "does not notice this); but it also eliminates some true positives (e.g. "
307:     "cases where a structure can be empty and this causes buggy behavior).",
308:     false)
309: 
310: ANALYZER_OPTION(
311:     bool, ShouldDisplayNotesAsEvents, "notes-as-events",
312:     "Whether the bug reporter should transparently treat extra note diagnostic "
313:     "pieces as event diagnostic pieces. Useful when the diagnostic consumer "
314:     "doesn't support the extra note pieces.",
315:     false)
316: 
317: ANALYZER_OPTION(
318:     bool, ShouldAggressivelySimplifyBinaryOperation,
319:     "aggressive-binary-operation-simplification",
320:     "Whether SValBuilder should rearrange comparisons and additive operations "
321:     "of symbolic expressions which consist of a sum of a symbol and a concrete "
322:     "integer into the format where symbols are on the left-hand side and the "
323:     "integer is on the right. This is only done if both symbols and both "
324:     "concrete integers are signed, greater than or equal to the quarter of the "
325:     "minimum value of the type and less than or equal to the quarter of the "
326:     "maximum value of that type. A + n <OP> B + m becomes A - B <OP> m - n, "
327:     "where A and B symbolic, n and m are integers. <OP> is any of '==', '!=', "
328:     "'<', '<=', '>', '>=', '+' or '-'. The rearrangement also happens with '-' "
329:     "instead of '+' on either or both side and also if any or both integers "
330:     "are missing.",
331:     false)
332: 
333: ANALYZER_OPTION(
334:     bool, ShouldEagerlyAssume, "eagerly-assume",
335:     "If this is enabled (the default behavior), when the analyzer encounters "
336:     "a comparison operator or logical negation, it immediately splits the "
```
- EN: This section enumerates macro-driven entries through `ANALYZER_OPTION`.
- 中文: 这一段通过 `ANALYZER_OPTION` 等宏列出可重复展开的条目。

### Lines 337-384

```cpp
337:     "state to separate the case when the expression is true and the case when "
338:     "it's false. The upside is that this can increase analysis precision until "
339:     "we have a better way to lazily evaluate such logic; the downside is that "
340:     "it eagerly bifurcates paths.",
341:     true)
342: 
343: ANALYZER_OPTION(
344:     bool, IsNaiveCTUEnabled, "experimental-enable-naive-ctu-analysis",
345:     "Whether naive cross translation unit analysis is enabled. This is an "
346:     "experimental feature to inline functions from other translation units.",
347:     false)
348: 
349: ANALYZER_OPTION(bool, ShouldDisplayMacroExpansions, "expand-macros",
350:                 "Whether macros related to the bugpath should be "
351:                 "expanded and included in the plist output.",
352:                 false)
353: 
354: ANALYZER_OPTION(bool, DisplayCTUProgress, "display-ctu-progress",
355:                 "Whether to emit verbose output about "
356:                 "the analyzer's progress related to ctu.",
357:                 false)
358: 
359: ANALYZER_OPTION(
360:     StringRef, DumpEntryPointStatsToCSV, "dump-entry-point-stats-to-csv",
361:     "If provided, the analyzer will dump statistics per entry point "
362:     "into the specified CSV file.",
363:     "")
364: 
365: ANALYZER_OPTION(bool, ShouldTrackConditions, "track-conditions",
366:                 "Whether to track conditions that are a control dependency of "
367:                 "an already tracked variable.",
368:                 true)
369: 
370: ANALYZER_OPTION(bool, ShouldTrackConditionsDebug, "track-conditions-debug",
371:                 "Whether to place an event at each tracked condition.",
372:                 false)
373: 
374: ANALYZER_OPTION(bool, ShouldApplyFixIts, "apply-fixits",
375:                 "Apply the fix-it hints to the files",
376:                 false)
377: 
378: ANALYZER_OPTION(bool, ShouldDisplayCheckerNameForText, "display-checker-name",
379:                 "Display the checker name for textual outputs",
380:                 true)
381: 
382: ANALYZER_OPTION(bool, ShouldSupportSymbolicIntegerCasts,
383:                 "support-symbolic-integer-casts",
384:                 "Produce cast symbols for integral types.",
```
- EN: This section enumerates macro-driven entries through `ANALYZER_OPTION`.
- 中文: 这一段通过 `ANALYZER_OPTION` 等宏列出可重复展开的条目。

### Lines 385-432

```cpp
385:                 false)
386: 
387: ANALYZER_OPTION(
388:     bool, ShouldAssumeControlledEnvironment, "assume-controlled-environment",
389:     "Whether the analyzed application runs in a controlled environment. "
390:     "We will assume that environment variables exist in queries and they hold "
391:     "no malicious data. For instance, if this option is enabled, 'getenv()' "
392:     "might be modeled by the analyzer to never return NULL.",
393:     false)
394: 
395: ANALYZER_OPTION(
396:     bool, ShouldIgnoreBisonGeneratedFiles, "ignore-bison-generated-files",
397:     "If enabled, any files containing the \"/* A Bison parser, made by\" "
398:     "won't be analyzed.",
399:     true)
400: 
401: ANALYZER_OPTION(
402:     bool, ShouldIgnoreFlexGeneratedFiles, "ignore-flex-generated-files",
403:     "If enabled, any files containing the \"/* A lexical scanner generated by "
404:     "flex\" won't be analyzed.",
405:     true)
406: 
407: ANALYZER_OPTION(
408:     bool, ShouldSuppressAddressSpaceDereferences, "suppress-dereferences-from-any-address-space",
409:     "The analyzer does not report dereferences on memory that use "
410:     "address space #256, #257, and #258. Those address spaces are used when "
411:     "dereferencing address spaces relative to the GS, FS, and SS segments on "
412:     "x86/x86-64 targets. Dereferencing a null pointer in these address spaces "
413:     "is not defined as an error. All other null dereferences in other address "
414:     "spaces are defined as an error unless explicitly defined. "
415:     "When this option is turned on, the special behavior of address spaces "
416:     "#256, #257, #258 is extended to all pointers with address spaces and on "
417:     "any target.",
418:     true)
419: 
420: ANALYZER_OPTION(
421:     bool, InlineFunctionsWithAmbiguousLoops, "inline-functions-with-ambiguous-loops",
422:     "If disabled (the default), the analyzer puts functions on a \"do not "
423:     "inline this\" list if it finds an execution path within that function "
424:     "that may potentially perform 'analyzer-max-loop' (= 4 by default) "
425:     "iterations in a loop. (Note that functions that _definitely_ reach the "
426:     "loop limit on some execution path are currently marked as \"do not "
427:     "inline\" even if this option is enabled.) Enabling this option "
428:     "eliminates this (somewhat arbitrary) restriction from the analysis "
429:     "scope, which increases the analysis runtime (on average by ~10%, but "
430:     "a few translation units may see much larger slowdowns).",
431:     false)
432: 
```
- EN: This section enumerates macro-driven entries through `ANALYZER_OPTION`.
- 中文: 这一段通过 `ANALYZER_OPTION` 等宏列出可重复展开的条目。

### Lines 433-480

```cpp
433: //===----------------------------------------------------------------------===//
434: // Unsigned analyzer options.
435: //===----------------------------------------------------------------------===//
436: 
437: ANALYZER_OPTION(unsigned, CTUImportThreshold, "ctu-import-threshold",
438:                 "The maximal amount of translation units that is considered "
439:                 "for import when inlining functions during CTU analysis. "
440:                 "Lowering this threshold can alleviate the memory burden of "
441:                 "analysis with many interdependent definitions located in "
442:                 "various translation units. This is valid only for non C++ "
443:                 "source files.",
444:                 24u)
445: 
446: ANALYZER_OPTION(unsigned, CTUImportCppThreshold, "ctu-import-cpp-threshold",
447:                 "The maximal amount of translation units that is considered "
448:                 "for import when inlining functions during CTU analysis of C++ "
449:                 "source files.",
450:                 8u)
451: 
452: ANALYZER_OPTION(
453:     unsigned, AlwaysInlineSize, "ipa-always-inline-size",
454:     "The size of the functions (in basic blocks), which should be considered "
455:     "to be small enough to always inline.",
456:     3)
457: 
458: ANALYZER_OPTION(
459:     unsigned, GraphTrimInterval, "graph-trim-interval",
460:     "How often nodes in the ExplodedGraph should be recycled to save memory. "
461:     "To disable node reclamation, set the option to 0.",
462:     1000)
463: 
464: ANALYZER_OPTION(
465:     unsigned, MinCFGSizeTreatFunctionsAsLarge,
466:     "min-cfg-size-treat-functions-as-large",
467:     "The number of basic blocks a function needs to have to be considered "
468:     "large for the 'max-times-inline-large' config option.",
469:     14)
470: 
471: ANALYZER_OPTION(unsigned, MaxSymbolComplexity, "max-symbol-complexity",
472:                 "The maximum complexity of symbolic constraint.", 35)
473: 
474: // HACK:https://discourse.llvm.org/t/rfc-make-istainted-and-complex-symbols-friends/79570
475: // Ideally, we should get rid of this option soon.
476: ANALYZER_OPTION(unsigned, MaxTaintedSymbolComplexity, "max-tainted-symbol-complexity",
477:                 "[DEPRECATED] The maximum complexity of a symbol to carry taint", 9)
478: 
479: ANALYZER_OPTION(unsigned, MaxTimesInlineLarge, "max-times-inline-large",
480:                 "The maximum times a large function could be inlined.", 32)
```
- EN: This section enumerates macro-driven entries through `ANALYZER_OPTION`.
- 中文: 这一段通过 `ANALYZER_OPTION` 等宏列出可重复展开的条目。

### Lines 481-528

```cpp
481: 
482: ANALYZER_OPTION_DEPENDS_ON_USER_MODE(
483:     unsigned, MaxInlinableSize, "max-inlinable-size",
484:     "The bound on the number of basic blocks in an inlined function.",
485:     /* SHALLOW_VAL */ 4, /* DEEP_VAL */ 100)
486: 
487: ANALYZER_OPTION_DEPENDS_ON_USER_MODE(
488:     unsigned, MaxNodesPerTopLevelFunction, "max-nodes",
489:     "The maximum number of nodes the analyzer can generate while exploring a "
490:     "top level function (for each exploded graph). 0 means no limit.",
491:     /* SHALLOW_VAL */ 75000, /* DEEP_VAL */ 225000)
492: 
493: ANALYZER_OPTION(
494:     unsigned, CTUMaxNodesPercentage, "ctu-max-nodes-pct",
495:     "The percentage of single-TU analysed nodes that the CTU analysis is "
496:     "allowed to visit.", 50)
497: 
498: ANALYZER_OPTION(
499:     unsigned, CTUMaxNodesMin, "ctu-max-nodes-min",
500:     "The maximum number of nodes in CTU mode is determinded by "
501:     "'ctu-max-nodes-pct'. However, if the number of nodes in single-TU "
502:     "analysis is too low, it is meaningful to provide a minimum value that "
503:     "serves as an upper bound instead.", 10000)
504: 
505: ANALYZER_OPTION(
506:     unsigned, RegionStoreSmallStructLimit, "region-store-small-struct-limit",
507:     "The largest number of fields a struct can have and still be considered "
508:     "small. This is currently used to decide whether or not it is worth forcing "
509:     "a LazyCompoundVal on bind. To disable all small-struct-dependent "
510:     "behavior, set the option to 0.",
511:     2)
512: 
513: ANALYZER_OPTION(
514:     unsigned, RegionStoreSmallArrayLimit, "region-store-small-array-limit",
515:     "The largest number of elements an array can have and still be considered "
516:     "small. This is currently used to decide whether or not it is worth forcing "
517:     "a LazyCompoundVal on bind. To disable all small-array-dependent "
518:     "behavior, set the option to 0.",
519:     5)
520: 
521: ANALYZER_OPTION(
522:     unsigned, RegionStoreMaxBindingFanOut, "region-store-max-binding-fanout",
523:     "This option limits how many sub-bindings a single binding operation can "
524:     "scatter into. For example, binding an array would scatter into binding "
525:     "each individual element. Setting this to zero means unlimited, but then "
526:     "modelling large array initializers may take proportional time to their "
527:     "size.", 128)
528: 
```
- EN: Key type declarations here include `can`. This section enumerates macro-driven entries through `ANALYZER_OPTION_DEPENDS_ON_USER_MODE`, `ANALYZER_OPTION`.
- 中文: 这里的重要类型声明包括 `can`。 这一段通过 `ANALYZER_OPTION_DEPENDS_ON_USER_MODE`, `ANALYZER_OPTION` 等宏列出可重复展开的条目。

### Lines 529-576

```cpp
529: //===----------------------------------------------------------------------===//
530: // String analyzer options.
531: //===----------------------------------------------------------------------===//
532: 
533: ANALYZER_OPTION(StringRef, CTUDir, "ctu-dir",
534:                 "The directory containing the CTU related files.", "")
535: 
536: ANALYZER_OPTION(StringRef, CTUIndexName, "ctu-index-name",
537:                 "The name of the file containing the CTU index of definitions. "
538:                 "The index file maps USR-names to identifiers. An identifier "
539:                 "can end with an '.ast' suffix, indicating the indentifier is "
540:                 "a path to a pch-dump. Otherwise the identifier is regarded as "
541:                 "path to a source file which is parsed on-demand. Relative "
542:                 "paths are prefixed with ctu-dir, absolute paths are used "
543:                 "unmodified during lookup.",
544:                 "externalDefMap.txt")
545: 
546: ANALYZER_OPTION(
547:     StringRef, CTUInvocationList, "ctu-invocation-list",
548:     "The path to the YAML format file containing a mapping from source file "
549:     "paths to command-line invocations represented as a list of arguments. "
550:     "This invocation is used produce the source-file's AST in case on-demand "
551:     "loading is performed. Example file-content: "
552:     "{/main.cpp: [clang++, /main.cpp], other.cpp: [clang++, /other.cpp]}",
553:     "invocations.yaml")
554: 
555: ANALYZER_OPTION(
556:     StringRef, ModelPath, "model-path",
557:     "The analyzer can inline an alternative implementation written in C at the "
558:     "call site if the called function's body is not available. This is a path "
559:     "where to look for those alternative implementations (called models).",
560:     "")
561: 
562: ANALYZER_OPTION(
563:     StringRef, CTUPhase1InliningMode, "ctu-phase1-inlining",
564:     "Controls which functions will be inlined during the first phase of the ctu "
565:     "analysis. "
566:     "If the value is set to 'all' then all foreign functions are inlinied "
567:     "immediately during the first phase, thus rendering the second phase a noop. "
568:     "The 'ctu-max-nodes-*' budge has no effect in this case. "
569:     "If the value is 'small' then only functions with a linear CFG and with a "
570:     "limited number of statements would be inlined during the first phase. The "
571:     "long and/or nontrivial functions are handled in the second phase and are "
572:     "controlled by the 'ctu-max-nodes-*' budge. "
573:     "The value 'none' means that all foreign functions are inlined only in the "
574:     "second phase, 'ctu-max-nodes-*' budge limits the second phase. "
575:     "Value: \"none\", \"small\", \"all\".",
576:     "small")
```
- EN: This section enumerates macro-driven entries through `ANALYZER_OPTION`.
- 中文: 这一段通过 `ANALYZER_OPTION` 等宏列出可重复展开的条目。

### Lines 577-604

```cpp
577: 
578: ANALYZER_OPTION(
579:     StringRef, CXXMemberInliningMode, "c++-inlining",
580:     "Controls which C++ member functions will be considered for inlining. "
581:     "Value: \"constructors\", \"destructors\", \"methods\".",
582:     "destructors")
583: 
584: ANALYZER_OPTION(
585:     StringRef, ExplorationStrategy, "exploration_strategy",
586:     "Value: \"dfs\", \"bfs\", \"unexplored_first\", "
587:     "\"unexplored_first_queue\", \"unexplored_first_location_queue\", "
588:     "\"bfs_block_dfs_contents\".",
589:     "unexplored_first_queue")
590: 
591: ANALYZER_OPTION(
592:     StringRef, RawSilencedCheckersAndPackages, "silence-checkers",
593:     "A semicolon separated list of checker and package names to silence. "
594:     "Silenced checkers will not emit reports, but the modeling remain enabled.",
595:     "")
596: 
597: ANALYZER_OPTION_DEPENDS_ON_USER_MODE(
598:     StringRef, IPAMode, "ipa",
599:     "Controls the mode of inter-procedural analysis. Value: \"none\", "
600:     "\"basic-inlining\", \"inlining\", \"dynamic\", \"dynamic-bifurcate\".",
601:     /* SHALLOW_VAL */ "inlining", /* DEEP_VAL */ "dynamic-bifurcate")
602: 
603: #undef ANALYZER_OPTION_DEPENDS_ON_USER_MODE
604: #undef ANALYZER_OPTION
```
- EN: This section enumerates macro-driven entries through `ANALYZER_OPTION`, `ANALYZER_OPTION_DEPENDS_ON_USER_MODE`.
- 中文: 这一段通过 `ANALYZER_OPTION`, `ANALYZER_OPTION_DEPENDS_ON_USER_MODE` 等宏列出可重复展开的条目。

## Key Concepts / 关键概念

- `ANALYZER_OPTION`: A macro hook that lets this definition list be reused in multiple expansion contexts. / 允许该定义列表在多个展开上下文中复用的宏钩子。
- `can`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ANALYZER_OPTION_DEPENDS_ON_USER_MODE`: A macro hook that lets this definition list be reused in multiple expansion contexts. / 允许该定义列表在多个展开上下文中复用的宏钩子。

## Dependencies / 依赖关系

- Direct includes / 直接包含: None / 无
- Forward declarations / 前向声明: None / 无
- Namespace context / 命名空间上下文: None / 无
- Macro-style dependencies / 宏式依赖: `ANALYZER_OPTION`, `ANALYZER_OPTION_DEPENDS_ON_USER_MODE`
