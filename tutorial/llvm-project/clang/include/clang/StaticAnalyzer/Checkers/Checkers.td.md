# Checkers.td — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/StaticAnalyzer/Checkers/Checkers.td`
- Repository: `llvm-project`
- Purpose (EN): Packages. The Alpha package is for checkers that have too many false positives to be turned on by default. The hierarchy under Alpha should be organized in the hierarchy checkers would have had if they were truly at the top level.
- 用途（中文）: 该文件为 StaticAnalyzer::Checkers 子系统中的 Checkers 提供 TableGen 声明式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48

```tablegen
 1: //===--- Checkers.td - Static Analyzer Checkers -===-----------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: include "CheckerBase.td"
10: 
11: //===----------------------------------------------------------------------===//
12: // Packages.
13: //===----------------------------------------------------------------------===//
14: 
15: // The Alpha package is for checkers that have too many false positives to be
16: // turned on by default. The hierarchy under Alpha should be organized in the
17: // hierarchy checkers would have had if they were truly at the top level.
18: // (For example, a Cocoa-specific checker that is alpha should be in
19: // alpha.osx.cocoa).
20: def Alpha : Package<"alpha">;
21: 
22: def Core : Package<"core">;
23: def CoreBuiltin : Package<"builtin">, ParentPackage<Core>, Hidden;
24: def CoreUninitialized  : Package<"uninitialized">, ParentPackage<Core>;
25: def CoreAlpha : Package<"core">, ParentPackage<Alpha>;
26: 
27: // The OptIn package is for checkers that are not alpha and that would normally
28: // be on by default but where the driver does not have enough information to
29: // determine when they are applicable. For example, localizability checkers fit
30: // this criterion because the driver cannot determine whether a project is
31: // localized or not -- this is best determined at the IDE or build-system level.
32: //
33: // The checker hierarchy under OptIn should mirror that in Alpha: checkers
34: // should be organized as if they were at the top level.
35: //
36: // Note: OptIn is *not* intended for checkers that are too noisy to be on by
37: // default. Such checkers belong in the alpha package.
38: def OptIn : Package<"optin">;
39: 
40: def CoreOptIn : Package<"core">, ParentPackage<OptIn>;
41: 
42: // In the Portability package reside checkers for finding code that relies on
43: // implementation-defined behavior. Such checks are wanted for cross-platform
44: // development, but unwanted for developers who target only a single platform.
45: def PortabilityOptIn : Package<"portability">, ParentPackage<OptIn>;
46: 
47: // Optional checkers related to taint security analysis.
48: def TaintOptIn : Package<"taint">, ParentPackage<OptIn>;
```
- EN: The TableGen records declared here include `Alpha`, `Core`, `CoreBuiltin`, `CoreUninitialized`.
- 中文: 这里声明的 TableGen 记录包括 `Alpha`, `Core`, `CoreBuiltin`, `CoreUninitialized`。

### Lines 49-96

```tablegen
49: 
50: def Nullability : Package<"nullability">,
51:   PackageOptions<[
52:     CmdLineOption<Boolean,
53:                   "NoDiagnoseCallsToSystemHeaders",
54:                   "Suppresses warnings for violating nullability annotations "
55:                   "of system header functions. This is useful if you are "
56:                   "concerned with your custom nullability annotations more "
57:                   "than with following nullability specifications of system "
58:                   "header functions.",
59:                   "false",
60:                   Released>
61:   ]>;
62: 
63: def Cplusplus : Package<"cplusplus">;
64: def CplusplusAlpha : Package<"cplusplus">, ParentPackage<Alpha>;
65: def CplusplusOptIn : Package<"cplusplus">, ParentPackage<OptIn>;
66: 
67: def DeadCode : Package<"deadcode">;
68: def DeadCodeAlpha : Package<"deadcode">, ParentPackage<Alpha>;
69: 
70: def Performance : Package<"performance">, ParentPackage<OptIn>;
71: 
72: def Security : Package <"security">;
73: def InsecureAPI : Package<"insecureAPI">, ParentPackage<Security>;
74: def SecurityAlpha : Package<"security">, ParentPackage<Alpha>;
75: 
76: def CERT : Package<"cert">, ParentPackage<Security>;
77: def ENV : Package<"env">, ParentPackage<CERT>;
78: 
79: def Unix : Package<"unix">;
80: def UnixAlpha : Package<"unix">, ParentPackage<Alpha>;
81: def CString : Package<"cstring">, ParentPackage<Unix>;
82: def CStringAlpha : Package<"cstring">, ParentPackage<UnixAlpha>;
83: 
84: def OSX : Package<"osx">;
85: def OSXAlpha : Package<"osx">, ParentPackage<Alpha>;
86: def OSXOptIn : Package<"osx">, ParentPackage<OptIn>;
87: 
88: def Cocoa : Package<"cocoa">, ParentPackage<OSX>;
89: def CocoaAlpha : Package<"cocoa">, ParentPackage<OSXAlpha>;
90: def CocoaOptIn : Package<"cocoa">, ParentPackage<OSXOptIn>;
91: 
92: def CoreFoundation : Package<"coreFoundation">, ParentPackage<OSX>;
93: def Containers : Package<"containers">, ParentPackage<CoreFoundation>;
94: 
95: def LocalizabilityAlpha : Package<"localizability">, ParentPackage<CocoaAlpha>;
96: def LocalizabilityOptIn : Package<"localizability">, ParentPackage<CocoaOptIn>;
```
- EN: The TableGen records declared here include `Nullability`, `Cplusplus`, `CplusplusAlpha`, `CplusplusOptIn`.
- 中文: 这里声明的 TableGen 记录包括 `Nullability`, `Cplusplus`, `CplusplusAlpha`, `CplusplusOptIn`。

### Lines 97-144

```tablegen
 97: 
 98: def MPI : Package<"mpi">, ParentPackage<OptIn>;
 99: 
100: def LLVM : Package<"llvm">;
101: def LLVMAlpha : Package<"llvm">, ParentPackage<Alpha>;
102: 
103: // The APIModeling package is for checkers that model APIs and don't perform
104: // any diagnostics. These checkers are always turned on; this package is
105: // intended for API modeling that is not controlled by the target triple.
106: def APIModeling : Package<"apiModeling">, Hidden;
107: def APIModelingAlpha : Package<"apiModeling">, ParentPackage<Alpha>, Hidden;
108: 
109: def GoogleAPIModeling : Package<"google">, ParentPackage<APIModeling>, Hidden;
110: def LLVMAPIModeling : Package<"llvm">, ParentPackage<APIModeling>, Hidden;
111: 
112: def Debug : Package<"debug">, Hidden;
113: 
114: def CloneDetectionAlpha : Package<"clone">, ParentPackage<Alpha>;
115: 
116: def Fuchsia : Package<"fuchsia">;
117: def FuchsiaAlpha : Package<"fuchsia">, ParentPackage<Alpha>;
118: 
119: def WebKit : Package<"webkit">;
120: def WebKitAlpha : Package<"webkit">, ParentPackage<Alpha>;
121: 
122: //===----------------------------------------------------------------------===//
123: // Core Checkers.
124: //===----------------------------------------------------------------------===//
125: 
126: let ParentPackage = Core in {
127: 
128: def BitwiseShiftChecker : Checker<"BitwiseShift">,
129:   HelpText<"Finds cases where bitwise shift operation causes undefined behaviour.">,
130:   CheckerOptions<[
131:     CmdLineOption<Boolean,
132:                   "Pedantic",
133:                   "If set to true, the checker reports undefined behavior even "
134:                   "if it is supported by most compilers. (This flag has no "
135:                   "effect in C++20 where these constructs are legal.)",
136:                   "false",
137:                   Released>,
138:   ]>,
139:   Documentation<HasDocumentation>;
140: 
141: def CallAndMessageChecker
142:     : Checker<"CallAndMessage">,
143:       HelpText<
144:           "Check for logical errors for function calls and Objective-C "
```
- EN: The TableGen records declared here include `MPI`, `LLVM`, `LLVMAlpha`, `APIModeling`.
- 中文: 这里声明的 TableGen 记录包括 `MPI`, `LLVM`, `LLVMAlpha`, `APIModeling`。

### Lines 145-192

```tablegen
145:           "message expressions (e.g., uninitialized arguments, null function "
146:           "pointers)">,
147:       CheckerOptions<
148:           [CmdLineOption<Boolean, "FunctionPointer",
149:                          "Check whether a called function pointer is null or "
150:                          "undefined",
151:                          "true", Released>,
152:            CmdLineOption<
153:                Boolean, "ParameterCount",
154:                "Check whether a function was called with the appropriate "
155:                "number of arguments",
156:                "true", Released>,
157:            CmdLineOption<Boolean, "CXXThisMethodCall",
158:                          "Check whether the implicit this parameter is null or "
159:                          "undefined upon a method call",
160:                          "true", Released>,
161:            CmdLineOption<
162:                Boolean, "CXXDeallocationArg",
163:                "Check whether the argument of operator delete is undefined",
164:                "true", Released>,
165:            CmdLineOption<Boolean, "ArgInitializedness",
166:                          "Check whether any of the pass-by-value parameters is "
167:                          "undefined",
168:                          "true", Released>,
169:            CmdLineOption<Boolean, "ArgPointeeInitializedness",
170:                          "Check whether the pointee of a pass-by-reference or "
171:                          "pass-by-pointer is undefined",
172:                          "false", InAlpha>,
173:            CmdLineOption<
174:                Boolean, "NilReceiver",
175:                "Check whether the receiver in the message expression is nil",
176:                "true", Released>,
177:            CmdLineOption<
178:                Boolean, "UndefReceiver",
179:                "Check whether the receiver in the message expression is "
180:                "undefined",
181:                "true", Released>,
182:            CmdLineOption<
183:                Boolean, "ArgPointeeInitializednessComplete",
184:                "If set to true, treat a struct as initialized when all of its "
185:                "members are initialized, otherwise when it has any initialized "
186:                "member (used only at ArgPointeeInitializedness)",
187:                "false", Released>,
188: ]>,
189:       Documentation<HasDocumentation>;
190: 
191: def NullDereferenceChecker
192:     : Checker<"NullDereference">,
```
- EN: Key type declarations here include `as`. The TableGen records declared here include `NullDereferenceChecker`.
- 中文: 这里的重要类型声明包括 `as`。 这里声明的 TableGen 记录包括 `NullDereferenceChecker`。

### Lines 193-240

```tablegen
193:       HelpText<"Check for dereferences of null pointers">,
194:       Documentation<HasDocumentation>;
195: 
196: def NullPointerArithmChecker
197:     : Checker<"NullPointerArithm">,
198:       HelpText<"Check for undefined arithmetic operations on null pointers">,
199:       Documentation<HasDocumentation>;
200: 
201: def NonNullParamChecker : Checker<"NonNullParamChecker">,
202:   HelpText<"Check for null pointers passed as arguments to a function whose "
203:            "arguments are references or marked with the 'nonnull' attribute">,
204:   Documentation<HasDocumentation>;
205: 
206: def VLASizeChecker : Checker<"VLASize">,
207:   HelpText<"Check for declarations of VLA of undefined or zero size">,
208:   Documentation<HasDocumentation>;
209: 
210: def DivZeroChecker : Checker<"DivideZero">,
211:   HelpText<"Check for division by zero">,
212:   Documentation<HasDocumentation>;
213: 
214: def UndefResultChecker : Checker<"UndefinedBinaryOperatorResult">,
215:   HelpText<"Check for undefined results of binary operators">,
216:   Documentation<HasDocumentation>;
217: 
218: def StackAddrEscapeChecker
219:     : Checker<"StackAddressEscape">,
220:       HelpText<
221:           "Check that addresses to stack memory do not escape the function">,
222:       Documentation<HasDocumentation>;
223: 
224: def DynamicTypePropagation : Checker<"DynamicTypePropagation">,
225:   HelpText<"Generate dynamic type information">,
226:   Documentation<NotDocumented>,
227:   Hidden;
228: 
229: def NonnullGlobalConstantsChecker: Checker<"NonnilStringConstants">,
230:   HelpText<"Assume that const string-like globals are non-null">,
231:   Documentation<NotDocumented>,
232:   Hidden;
233: 
234: } // end "core"
235: 
236: let ParentPackage = CoreAlpha in {
237: 
238: def BoolAssignmentChecker : Checker<"BoolAssignment">,
239:   HelpText<"Warn about assigning non-{0,1} values to Boolean variables">,
240:   Documentation<HasDocumentation>;
```
- EN: The TableGen records declared here include `NullPointerArithmChecker`, `NonNullParamChecker`, `VLASizeChecker`, `DivZeroChecker`.
- 中文: 这里声明的 TableGen 记录包括 `NullPointerArithmChecker`, `NonNullParamChecker`, `VLASizeChecker`, `DivZeroChecker`。

### Lines 241-288

```tablegen
241: 
242: def CastToStructChecker : Checker<"CastToStruct">,
243:   HelpText<"Check for cast from non-struct pointer to struct pointer">,
244:   Documentation<HasDocumentation>;
245: 
246: def ConversionChecker : Checker<"Conversion">,
247:   HelpText<"Loss of sign/precision in implicit conversions">,
248:   Documentation<HasDocumentation>;
249: 
250: def PointerArithChecker : Checker<"PointerArithm">,
251:   HelpText<"Check for pointer arithmetic on locations other than array "
252:            "elements">,
253:   Documentation<HasDocumentation>;
254: 
255: def TestAfterDivZeroChecker : Checker<"TestAfterDivZero">,
256:   HelpText<"Check for division by variable that is later compared against 0. "
257:            "Either the comparison is useless or there is division by zero.">,
258:   Documentation<HasDocumentation>;
259: 
260: def DynamicTypeChecker
261:     : Checker<"DynamicTypeChecker">,
262:       HelpText<"Check for cases where the dynamic and the static type of an "
263:                "object are unrelated.">,
264:       Dependencies<[DynamicTypePropagation]>,
265:       Documentation<HasDocumentation>;
266: 
267: def StackAddrAsyncEscapeChecker
268:     : Checker<"StackAddressAsyncEscape">,
269:       HelpText<
270:           "Check that addresses to stack memory do not escape the function">,
271:       Documentation<HasDocumentation>;
272: 
273: def StoreToImmutableChecker : Checker<"StoreToImmutable">,
274:   HelpText<"Check for writes to immutable memory regions. "
275:            "This implements part of SEI CERT Rule ENV30-C.">,
276:   Documentation<HasDocumentation>;
277: 
278: def PthreadLockBase : Checker<"PthreadLockBase">,
279:   HelpText<"Helper registering multiple checks.">,
280:   Documentation<NotDocumented>,
281:   Hidden;
282: 
283: def C11LockChecker : Checker<"C11Lock">,
284:   HelpText<"Simple lock -> unlock checker">,
285:   Dependencies<[PthreadLockBase]>,
286:   Documentation<HasDocumentation>;
287: 
288: def StdVariantChecker : Checker<"StdVariant">,
```
- EN: Key type declarations here include `pointer`. The TableGen records declared here include `CastToStructChecker`, `ConversionChecker`, `PointerArithChecker`, `TestAfterDivZeroChecker`.
- 中文: 这里的重要类型声明包括 `pointer`。 这里声明的 TableGen 记录包括 `CastToStructChecker`, `ConversionChecker`, `PointerArithChecker`, `TestAfterDivZeroChecker`。

### Lines 289-336

```tablegen
289:   HelpText<"Check for bad type access for std::variant.">,
290:   Documentation<HasDocumentation>;
291: 
292: } // end "alpha.core"
293: 
294: //===----------------------------------------------------------------------===//
295: // Nullability checkers.
296: //===----------------------------------------------------------------------===//
297: 
298: let ParentPackage = Nullability in {
299: 
300:   def NullPassedToNonnullChecker
301:       : Checker<"NullPassedToNonnull">,
302:         HelpText<"Warns when a null pointer is passed to a pointer which has a "
303:                  "_Nonnull type.">,
304:         Documentation<HasDocumentation>;
305: 
306:   def NullReturnedFromNonnullChecker
307:       : Checker<"NullReturnedFromNonnull">,
308:         HelpText<"Warns when a null pointer is returned from a function that "
309:                  "has _Nonnull return type.">,
310:         Documentation<HasDocumentation>;
311: 
312:   def NullableDereferencedChecker
313:       : Checker<"NullableDereferenced">,
314:         HelpText<"Warns when a nullable pointer is dereferenced.">,
315:         Documentation<HasDocumentation>;
316: 
317:   def NullablePassedToNonnullChecker
318:       : Checker<"NullablePassedToNonnull">,
319:         HelpText<"Warns when a nullable pointer is passed to a pointer which "
320:                  "has a _Nonnull type.">,
321:         Documentation<HasDocumentation>;
322: 
323:   def NullableReturnedFromNonnullChecker
324:       : Checker<"NullableReturnedFromNonnull">,
325:         HelpText<"Warns when a nullable pointer is returned from a function "
326:                  "that has _Nonnull return type.">,
327:         Documentation<NotDocumented>;
328: 
329: } // end "nullability"
330: 
331: //===----------------------------------------------------------------------===//
332: // APIModeling.
333: //===----------------------------------------------------------------------===//
334: 
335: let ParentPackage = APIModeling in {
336: 
```
- EN: The TableGen records declared here include `NullPassedToNonnullChecker`, `NullReturnedFromNonnullChecker`, `NullableDereferencedChecker`, `NullablePassedToNonnullChecker`.
- 中文: 这里声明的 TableGen 记录包括 `NullPassedToNonnullChecker`, `NullReturnedFromNonnullChecker`, `NullableDereferencedChecker`, `NullablePassedToNonnullChecker`。

### Lines 337-384

```tablegen
337: def ErrnoModeling : Checker<"Errno">,
338:   HelpText<"Make the special value 'errno' available to other checkers.">,
339:   Documentation<NotDocumented>;
340: 
341: def OpaqueSTLFunctionsModeling : Checker<"OpaqueSTLFunctionsModeling">,
342:   HelpText<"Model opaque, conservative evaluation of some STL functions">,
343:   Documentation<NotDocumented>;
344: 
345: def TrustNonnullChecker : Checker<"TrustNonnull">,
346:   HelpText<"Trust that returns from framework methods annotated with _Nonnull "
347:            "are not null">,
348:   Documentation<NotDocumented>;
349: 
350: def TrustReturnsNonnullChecker : Checker<"TrustReturnsNonnull">,
351:   HelpText<"Trust that returns from methods annotated with returns_nonnull "
352:            "are not null">,
353:   Documentation<NotDocumented>;
354: 
355: } // end "apiModeling"
356: 
357: //===----------------------------------------------------------------------===//
358: // Evaluate "builtin" functions and assumptions.
359: //===----------------------------------------------------------------------===//
360: 
361: let ParentPackage = CoreBuiltin in {
362: 
363: def NoReturnFunctionChecker : Checker<"NoReturnFunctions">,
364:   HelpText<"Evaluate \"panic\" functions that are known to not return to the "
365:            "caller">,
366:   Documentation<NotDocumented>;
367: 
368: def BuiltinFunctionChecker : Checker<"BuiltinFunctions">,
369:   HelpText<"Evaluate compiler builtin functions (e.g., alloca())">,
370:   Documentation<NotDocumented>;
371: 
372: def AssumeModeling : Checker<"AssumeModeling">,
373:   HelpText<"Model compiler builtin assume functions and the assume attribute">,
374:   Documentation<NotDocumented>;
375: 
376: } // end "core.builtin"
377: 
378: //===----------------------------------------------------------------------===//
379: // Uninitialized values checkers.
380: //===----------------------------------------------------------------------===//
381: 
382: let ParentPackage = CoreUninitialized in {
383: 
384: def UndefinedArraySubscriptChecker : Checker<"ArraySubscript">,
```
- EN: The TableGen records declared here include `ErrnoModeling`, `OpaqueSTLFunctionsModeling`, `TrustNonnullChecker`, `TrustReturnsNonnullChecker`.
- 中文: 这里声明的 TableGen 记录包括 `ErrnoModeling`, `OpaqueSTLFunctionsModeling`, `TrustNonnullChecker`, `TrustReturnsNonnullChecker`。

### Lines 385-432

```tablegen
385:   HelpText<"Check for uninitialized values used as array subscripts">,
386:   Documentation<HasDocumentation>;
387: 
388: def UndefinedAssignmentChecker : Checker<"Assign">,
389:   HelpText<"Check for assigning uninitialized values">,
390:   Documentation<HasDocumentation>;
391: 
392: def UndefBranchChecker : Checker<"Branch">,
393:   HelpText<"Check for uninitialized values used as branch conditions">,
394:   Documentation<HasDocumentation>;
395: 
396: def UndefCapturedBlockVarChecker : Checker<"CapturedBlockVariable">,
397:   HelpText<"Check for blocks that capture uninitialized values">,
398:   Documentation<NotDocumented>;
399: 
400: def ReturnUndefChecker : Checker<"UndefReturn">,
401:   HelpText<"Check for uninitialized values being returned to the caller">,
402:   Documentation<HasDocumentation>;
403: 
404: def UndefinedNewArraySizeChecker : Checker<"NewArraySize">,
405:   HelpText<"Check if the size of the array in a new[] expression is undefined">,
406:   Documentation<HasDocumentation>;
407: 
408: } // end "core.uninitialized"
409: 
410: //===----------------------------------------------------------------------===//
411: // Optin checkers for core language features
412: //===----------------------------------------------------------------------===//
413: 
414: let ParentPackage = CoreOptIn in {
415: 
416: def EnumCastOutOfRangeChecker : Checker<"EnumCastOutOfRange">,
417:   HelpText<"Check integer to enumeration casts for out of range values">,
418:   Documentation<HasDocumentation>;
419: 
420: def FixedAddressDereferenceChecker
421:     : Checker<"FixedAddressDereference">,
422:       HelpText<"Check for dereferences of fixed addresses">,
423:       Documentation<HasDocumentation>;
424: 
425: def UnconditionalVAArgChecker
426:     : Checker<"UnconditionalVAArg">,
427:       HelpText<"Check variadic functions unconditionally using va_arg">,
428:       Documentation<HasDocumentation>;
429: 
430: } // end "optin.core"
431: 
432: //===----------------------------------------------------------------------===//
```
- EN: The TableGen records declared here include `UndefinedAssignmentChecker`, `UndefBranchChecker`, `UndefCapturedBlockVarChecker`, `ReturnUndefChecker`.
- 中文: 这里声明的 TableGen 记录包括 `UndefinedAssignmentChecker`, `UndefBranchChecker`, `UndefCapturedBlockVarChecker`, `ReturnUndefChecker`。

### Lines 433-480

```tablegen
433: // Unix API checkers.
434: //===----------------------------------------------------------------------===//
435: 
436: let ParentPackage = CString in {
437: 
438: def CStringModeling : Checker<"CStringModeling">,
439:   HelpText<"The base of several CString related checkers. On it's own it emits "
440:            "no reports, but adds valuable information to the analysis when "
441:            "enabled.">,
442:   Documentation<NotDocumented>,
443:   Hidden;
444: 
445: def CStringNotNullTerm : Checker<"NotNullTerminated">,
446:   HelpText<"Check for arguments passed to C string functions which are not "
447:            "null-terminated strings">,
448:   Dependencies<[CStringModeling]>,
449:   Documentation<HasDocumentation>;
450: 
451: def CStringNullArg : Checker<"NullArg">,
452:   HelpText<"Check for null pointers being passed as arguments to C string "
453:            "functions">,
454:   Dependencies<[CStringModeling]>,
455:   Documentation<HasDocumentation>;
456: 
457: def CStringSyntaxChecker : Checker<"BadSizeArg">,
458:   HelpText<"Check the size argument passed into C string functions for common "
459:            "erroneous patterns">,
460:   Dependencies<[CStringModeling]>,
461:   Documentation<HasDocumentation>;
462: 
463: } // end "unix.cstring"
464: 
465: let ParentPackage = CStringAlpha in {
466: 
467: def CStringOutOfBounds : Checker<"OutOfBounds">,
468:   HelpText<"Check for out-of-bounds access in string functions">,
469:   Dependencies<[CStringModeling]>,
470:   Documentation<HasDocumentation>;
471: 
472: def CStringBufferOverlap : Checker<"BufferOverlap">,
473:   HelpText<"Checks for overlap in two buffer arguments">,
474:   Dependencies<[CStringModeling]>,
475:   Documentation<HasDocumentation>;
476: 
477: def CStringUninitializedRead : Checker<"UninitializedRead">,
478:   HelpText<"Checks if the string manipulation function would read uninitialized bytes">,
479:   Dependencies<[CStringModeling]>,
480:   Documentation<HasDocumentation>;
```
- EN: The TableGen records declared here include `CStringModeling`, `CStringNotNullTerm`, `CStringNullArg`, `CStringSyntaxChecker`.
- 中文: 这里声明的 TableGen 记录包括 `CStringModeling`, `CStringNotNullTerm`, `CStringNullArg`, `CStringSyntaxChecker`。

### Lines 481-528

```tablegen
481: 
482: } // end "alpha.unix.cstring"
483: 
484: let ParentPackage = Unix in {
485: 
486: def UnixAPIMisuseChecker : Checker<"API">,
487:   HelpText<"Check calls to various UNIX/Posix functions">,
488:   Documentation<HasDocumentation>;
489: 
490: def BlockInCriticalSectionChecker : Checker<"BlockInCriticalSection">,
491:   HelpText<"Check for calls to blocking functions inside a critical section">,
492:   Documentation<HasDocumentation>;
493: 
494: def DynamicMemoryModeling: Checker<"DynamicMemoryModeling">,
495:   HelpText<"The base of several malloc() related checkers. On it's own it "
496:            "emits no reports, but adds valuable information to the analysis "
497:            "when enabled.">,
498:   CheckerOptions<[
499:     CmdLineOption<Boolean,
500:                   "Optimistic",
501:                   "If set to true, the checker assumes that all the "
502:                   "allocating and deallocating functions are annotated with "
503:                   "ownership_holds, ownership_takes and ownership_returns.",
504:                   "false",
505:                   InAlpha>,
506:     CmdLineOption<Boolean,
507:                   "AddNoOwnershipChangeNotes",
508:                   "Add an additional note to the bug report for leak-like "
509:                   "bugs. Dynamically allocated objects passed to functions "
510:                   "that neither deallocated it, or have taken responsibility "
511:                   "of the ownership are noted, similarly to "
512:                   "NoStoreFuncVisitor.",
513:                   "true",
514:                   Released,
515:                   Hide>
516:   ]>,
517:   Dependencies<[CStringModeling]>,
518:   Documentation<NotDocumented>,
519:   Hidden;
520: 
521: def ErrnoChecker : Checker<"Errno">,
522:   HelpText<"Check for improper use of 'errno'">,
523:   Dependencies<[ErrnoModeling]>,
524:   CheckerOptions<[
525:     CmdLineOption<Boolean,
526:                   "AllowErrnoReadOutsideConditionExpressions",
527:                   "Allow read of undefined value from errno outside of conditions",
528:                   "true",
```
- EN: The TableGen records declared here include `UnixAPIMisuseChecker`, `BlockInCriticalSectionChecker`, `DynamicMemoryModeling`, `ErrnoChecker`.
- 中文: 这里声明的 TableGen 记录包括 `UnixAPIMisuseChecker`, `BlockInCriticalSectionChecker`, `DynamicMemoryModeling`, `ErrnoChecker`。

### Lines 529-576

```tablegen
529:                   InAlpha>,
530:   ]>,
531:   Documentation<HasDocumentation>;
532: 
533: def MallocChecker: Checker<"Malloc">,
534:   HelpText<"Check for memory leaks, double free, and use-after-free problems. "
535:            "Traces memory managed by malloc()/free().">,
536:   Dependencies<[DynamicMemoryModeling]>,
537:   Documentation<HasDocumentation>;
538: 
539: def MallocSizeofChecker : Checker<"MallocSizeof">,
540:   HelpText<"Check for dubious malloc arguments involving sizeof">,
541:   Documentation<HasDocumentation>;
542: 
543: def MismatchedDeallocatorChecker : Checker<"MismatchedDeallocator">,
544:   HelpText<"Check for mismatched deallocators.">,
545:   Dependencies<[DynamicMemoryModeling]>,
546:   Documentation<HasDocumentation>;
547: 
548: // This must appear before StdCLibraryFunctionsChecker because a dependency.
549: def StreamChecker : Checker<"Stream">,
550:   HelpText<"Check stream handling functions">,
551:   WeakDependencies<[NonNullParamChecker]>,
552:   CheckerOptions<[
553:     CmdLineOption<Boolean,
554:                   "Pedantic",
555:                   "If false, assume that stream operations which are often not "
556:                   "checked for error do not fail.",
557:                   "false",
558:                   InAlpha>
559:   ]>,
560:   Documentation<HasDocumentation>;
561: 
562: def StdCLibraryFunctionsChecker : Checker<"StdCLibraryFunctions">,
563:   HelpText<"Check for invalid arguments of C standard library functions, "
564:            "and apply relations between arguments and return value">,
565:   CheckerOptions<[
566:     CmdLineOption<Boolean,
567:                   "DisplayLoadedSummaries",
568:                   "If set to true, the checker displays the found summaries "
569:                   "for the given translation unit.",
570:                   "false",
571:                   Released,
572:                   Hide>,
573:     CmdLineOption<Boolean,
574:                   "ModelPOSIX",
575:                   "If set to true, the checker models additional functions "
576:                   "from the POSIX standard.",
```
- EN: The TableGen records declared here include `MallocChecker`, `MallocSizeofChecker`, `MismatchedDeallocatorChecker`, `StreamChecker`.
- 中文: 这里声明的 TableGen 记录包括 `MallocChecker`, `MallocSizeofChecker`, `MismatchedDeallocatorChecker`, `StreamChecker`。

### Lines 577-624

```tablegen
577:                   "true",
578:                   InAlpha>
579:   ]>,
580:   WeakDependencies<[CallAndMessageChecker, NonNullParamChecker, StreamChecker]>,
581:   Documentation<HasDocumentation>;
582: 
583: def VforkChecker : Checker<"Vfork">,
584:   HelpText<"Check for proper usage of vfork">,
585:   Documentation<HasDocumentation>;
586: 
587: def ChrootChecker : Checker<"Chroot">,
588:   HelpText<"Check improper use of chroot">,
589:   Documentation<HasDocumentation>;
590: 
591: } // end "unix"
592: 
593: let ParentPackage = UnixAlpha in {
594: 
595: def PthreadLockChecker : Checker<"PthreadLock">,
596:   HelpText<"Simple lock -> unlock checker">,
597:   Dependencies<[PthreadLockBase]>,
598:   Documentation<HasDocumentation>;
599: 
600: def SimpleStreamChecker : Checker<"SimpleStream">,
601:   HelpText<"Check for misuses of stream APIs">,
602:   Documentation<HasDocumentation>;
603: 
604: } // end "alpha.unix"
605: 
606: //===----------------------------------------------------------------------===//
607: // C++ checkers.
608: //===----------------------------------------------------------------------===//
609: 
610: let ParentPackage = Cplusplus in {
611: 
612: def ArrayDeleteChecker : Checker<"ArrayDelete">,
613:   HelpText<"Reports destructions of arrays of polymorphic objects that are "
614:            "destructed as their base class.">,
615:   Documentation<HasDocumentation>;
616: 
617: def InnerPointerChecker : Checker<"InnerPointer">,
618:   HelpText<"Check for inner pointers of C++ containers used after "
619:            "re/deallocation">,
620:   Dependencies<[DynamicMemoryModeling]>,
621:   Documentation<NotDocumented>;
622: 
623: def NewDeleteChecker : Checker<"NewDelete">,
624:   HelpText<"Check for double-free and use-after-free problems. Traces memory "
```
- EN: The TableGen records declared here include `VforkChecker`, `ChrootChecker`, `PthreadLockChecker`, `SimpleStreamChecker`.
- 中文: 这里声明的 TableGen 记录包括 `VforkChecker`, `ChrootChecker`, `PthreadLockChecker`, `SimpleStreamChecker`。

### Lines 625-672

```tablegen
625:            "managed by new/delete.">,
626:   Dependencies<[DynamicMemoryModeling]>,
627:   Documentation<HasDocumentation>;
628: 
629: def NewDeleteLeaksChecker : Checker<"NewDeleteLeaks">,
630:   HelpText<"Check for memory leaks. Traces memory managed by new/delete.">,
631:   Dependencies<[DynamicMemoryModeling]>,
632:   Documentation<HasDocumentation>;
633: 
634: def PlacementNewChecker : Checker<"PlacementNew">,
635:   HelpText<"Check if default placement new is provided with pointers to "
636:            "sufficient storage capacity">,
637:   Dependencies<[DynamicMemoryModeling]>,
638:   Documentation<HasDocumentation>;
639: 
640: def CXXSelfAssignmentChecker : Checker<"SelfAssignment">,
641:   HelpText<"Checks C++ copy and move assignment operators for self assignment">,
642:   Documentation<NotDocumented>,
643:   Hidden;
644: 
645: def SmartPtrModeling: Checker<"SmartPtrModeling">,
646:   HelpText<"Model behavior of C++ smart pointers">,
647:   Documentation<NotDocumented>,
648:     CheckerOptions<[
649:     CmdLineOption<Boolean,
650:                   "ModelSmartPtrDereference",
651:                   "Enable modeling for SmartPtr null dereferences",
652:                   "false",
653:                   InAlpha,
654:                   Hide>,
655:   ]>,
656:   Hidden;
657: 
658: def StringChecker: Checker<"StringChecker">,
659:   HelpText<"Checks C++ std::string bugs">,
660:   Documentation<HasDocumentation>;
661: 
662: def MoveChecker: Checker<"Move">,
663:   HelpText<"Find use-after-move bugs in C++">,
664:   CheckerOptions<[
665:     CmdLineOption<String,
666:                   "WarnOn",
667:                   "With setting \"KnownsOnly\" warn only on objects with known "
668:                   "move semantics like smart pointers and other STL objects. "
669:                   "With setting \"KnownsAndLocals\" warn additionally on local "
670:                   "variables (or rvalue references). With setting \"All\" warn "
671:                   "on all variables (excluding global variables).",
672:                   "KnownsAndLocals",
```
- EN: The TableGen records declared here include `NewDeleteLeaksChecker`, `PlacementNewChecker`, `CXXSelfAssignmentChecker`, `SmartPtrModeling`.
- 中文: 这里声明的 TableGen 记录包括 `NewDeleteLeaksChecker`, `PlacementNewChecker`, `CXXSelfAssignmentChecker`, `SmartPtrModeling`。

### Lines 673-720

```tablegen
673:                   Released>
674:   ]>,
675:   Documentation<HasDocumentation>;
676: 
677: def PureVirtualCallChecker
678:     : Checker<"PureVirtualCall">,
679:       HelpText<
680:           "Check pure virtual function calls during construction/destruction">,
681:       Documentation<HasDocumentation>;
682: } // end: "cplusplus"
683: 
684: let ParentPackage = CplusplusOptIn in {
685: 
686: def UninitializedObjectChecker: Checker<"UninitializedObject">,
687:   HelpText<"Reports uninitialized fields after object construction">,
688:   CheckerOptions<[
689:     CmdLineOption<Boolean,
690:                   "Pedantic",
691:                   "If set to false, the checker won't emit warnings "
692:                   "for objects that don't have at least one initialized "
693:                   "field.",
694:                   "false",
695:                   Released>,
696:     CmdLineOption<Boolean,
697:                   "NotesAsWarnings",
698:                   "If set to true, the checker will emit a warning "
699:                   "for each uninitalized field, as opposed to emitting one "
700:                   "warning per constructor call, and listing the uninitialized "
701:                   "fields that belongs to it in notes.",
702:                   "false",
703:                   Released,
704:                   Hide>,
705:     CmdLineOption<Boolean,
706:                   "CheckPointeeInitialization",
707:                   "If set to false, the checker will not analyze "
708:                   "the pointee of pointer/reference fields, and will only "
709:                   "check whether the object itself is initialized.",
710:                   "false",
711:                   InAlpha>,
712:     CmdLineOption<String,
713:                   "IgnoreRecordsWithField",
714:                   "If supplied, the checker will not analyze "
715:                   "structures that have a field with a name or type name that "
716:                   "matches the given pattern.",
717:                   "\"\"",
718:                   Released>,
719:     CmdLineOption<Boolean,
720:                   "IgnoreGuardedFields",
```
- EN: The TableGen records declared here include `PureVirtualCallChecker`, `UninitializedObjectChecker`.
- 中文: 这里声明的 TableGen 记录包括 `PureVirtualCallChecker`, `UninitializedObjectChecker`。

### Lines 721-768

```tablegen
721:                   "If set to true, the checker will analyze _syntactically_ "
722:                   "whether the found uninitialized object is used without a "
723:                   "preceding assert call. Defaults to false.",
724:                   "false",
725:                   InAlpha>
726:   ]>,
727:   Documentation<HasDocumentation>;
728: 
729: def VirtualCallChecker
730:     : Checker<"VirtualCall">,
731:       HelpText<"Check virtual function calls during construction/destruction">,
732:       CheckerOptions<[CmdLineOption<Boolean, "ShowFixIts",
733:                                     "Enable fix-it hints for this checker",
734:                                     "false", InAlpha>]>,
735:       Documentation<HasDocumentation>;
736: 
737: } // end: "optin.cplusplus"
738: 
739: let ParentPackage = CplusplusAlpha in {
740: 
741: def ContainerModeling : Checker<"ContainerModeling">,
742:   HelpText<"Models C++ containers">,
743:   Documentation<NotDocumented>,
744:   Hidden;
745: 
746: def DeleteWithNonVirtualDtorChecker : Checker<"DeleteWithNonVirtualDtor">,
747:   HelpText<"Reports destructions of polymorphic objects with a non-virtual "
748:            "destructor in their base class">,
749:   Documentation<HasDocumentation>;
750: 
751: def IteratorModeling : Checker<"IteratorModeling">,
752:   HelpText<"Models iterators of C++ containers">,
753:   Dependencies<[ContainerModeling]>,
754:   Documentation<NotDocumented>,
755:   Hidden;
756: 
757: def STLAlgorithmModeling : Checker<"STLAlgorithmModeling">,
758:   HelpText<"Models the algorithm library of the C++ STL.">,
759:   CheckerOptions<[
760:     CmdLineOption<Boolean,
761:                   "AggressiveStdFindModeling",
762:                   "Enables exploration of the failure branch in std::find-like "
763:                   "functions.",
764:                   "false",
765:                   Released>
766:   ]>,
767:   Dependencies<[ContainerModeling]>,
768:   Documentation<NotDocumented>;
```
- EN: The TableGen records declared here include `VirtualCallChecker`, `ContainerModeling`, `DeleteWithNonVirtualDtorChecker`, `IteratorModeling`.
- 中文: 这里声明的 TableGen 记录包括 `VirtualCallChecker`, `ContainerModeling`, `DeleteWithNonVirtualDtorChecker`, `IteratorModeling`。

### Lines 769-816

```tablegen
769: 
770: def InvalidatedIteratorChecker : Checker<"InvalidatedIterator">,
771:   HelpText<"Check for use of invalidated iterators">,
772:   Dependencies<[IteratorModeling]>,
773:   Documentation<HasDocumentation>;
774: 
775: def IteratorRangeChecker : Checker<"IteratorRange">,
776:   HelpText<"Check for iterators used outside their valid ranges">,
777:   Dependencies<[IteratorModeling]>,
778:   Documentation<HasDocumentation>;
779: 
780: def MismatchedIteratorChecker : Checker<"MismatchedIterator">,
781:   HelpText<"Check for use of iterators of different containers where iterators "
782:            "of the same container are expected">,
783:   Dependencies<[IteratorModeling]>,
784:   Documentation<HasDocumentation>;
785: 
786: def SmartPtrChecker: Checker<"SmartPtr">,
787:   HelpText<"Find the dereference of null SmrtPtr">,
788:   Dependencies<[SmartPtrModeling]>,
789:   Documentation<HasDocumentation>;
790: 
791: } // end: "alpha.cplusplus"
792: 
793: //===----------------------------------------------------------------------===//
794: // Deadcode checkers.
795: //===----------------------------------------------------------------------===//
796: 
797: let ParentPackage = DeadCode in {
798: 
799: def DeadStoresChecker : Checker<"DeadStores">,
800:   HelpText<"Check for values stored to variables that are never read "
801:            "afterwards">,
802:   CheckerOptions<[
803:     CmdLineOption<Boolean,
804:                   "WarnForDeadNestedAssignments",
805:                   "Warns for deadstores in nested assignments."
806:                   "E.g.: if ((P = f())) where P is unused.",
807:                   "true",
808:                   Released>,
809:     CmdLineOption<Boolean,
810:                   "ShowFixIts",
811:                   "Enable fix-it hints for this checker",
812:                   "false",
813:                   InAlpha>
814:   ]>,
815:   Documentation<HasDocumentation>;
816: 
```
- EN: The TableGen records declared here include `InvalidatedIteratorChecker`, `IteratorRangeChecker`, `MismatchedIteratorChecker`, `SmartPtrChecker`.
- 中文: 这里声明的 TableGen 记录包括 `InvalidatedIteratorChecker`, `IteratorRangeChecker`, `MismatchedIteratorChecker`, `SmartPtrChecker`。

### Lines 817-864

```tablegen
817: } // end DeadCode
818: 
819: let ParentPackage = DeadCodeAlpha in {
820: 
821: def UnreachableCodeChecker : Checker<"UnreachableCode">,
822:   HelpText<"Check unreachable code">,
823:   Documentation<HasDocumentation>;
824: 
825: } // end "alpha.deadcode"
826: 
827: //===----------------------------------------------------------------------===//
828: // Performance checkers.
829: //===----------------------------------------------------------------------===//
830: 
831: let ParentPackage = Performance in {
832: 
833: def PaddingChecker : Checker<"Padding">,
834:   HelpText<"Check for excessively padded structs.">,
835:   CheckerOptions<[
836:     CmdLineOption<Integer,
837:                   "AllowedPad",
838:                   "Reports are only generated if the excessive padding exceeds "
839:                   "'AllowedPad' in bytes.",
840:                   "24",
841:                   Released>
842:   ]>,
843:   Documentation<HasDocumentation>;
844: 
845: } // end: "padding"
846: 
847: //===----------------------------------------------------------------------===//
848: // Security checkers.
849: //===----------------------------------------------------------------------===//
850: 
851: let ParentPackage = InsecureAPI in {
852: 
853: def SecuritySyntaxChecker : Checker<"SecuritySyntaxChecker">,
854:   HelpText<"Base of various security function related checkers">,
855:   Documentation<NotDocumented>,
856:   Hidden;
857: 
858: def bcmp : Checker<"bcmp">,
859:   HelpText<"Warn on uses of the 'bcmp' function">,
860:   Dependencies<[SecuritySyntaxChecker]>,
861:   Documentation<HasDocumentation>;
862: 
863: def bcopy : Checker<"bcopy">,
864:   HelpText<"Warn on uses of the 'bcopy' function">,
```
- EN: The TableGen records declared here include `UnreachableCodeChecker`, `PaddingChecker`, `SecuritySyntaxChecker`, `bcmp`.
- 中文: 这里声明的 TableGen 记录包括 `UnreachableCodeChecker`, `PaddingChecker`, `SecuritySyntaxChecker`, `bcmp`。

### Lines 865-912

```tablegen
865:   Dependencies<[SecuritySyntaxChecker]>,
866:   Documentation<HasDocumentation>;
867: 
868: def bzero : Checker<"bzero">,
869:   HelpText<"Warn on uses of the 'bzero' function">,
870:   Dependencies<[SecuritySyntaxChecker]>,
871:   Documentation<HasDocumentation>;
872: 
873: def gets : Checker<"gets">,
874:   HelpText<"Warn on uses of the 'gets' function">,
875:   Dependencies<[SecuritySyntaxChecker]>,
876:   Documentation<HasDocumentation>;
877: 
878: def getpw : Checker<"getpw">,
879:   HelpText<"Warn on uses of the 'getpw' function">,
880:   Dependencies<[SecuritySyntaxChecker]>,
881:   Documentation<HasDocumentation>;
882: 
883: def mktemp : Checker<"mktemp">,
884:   HelpText<"Warn on uses of the 'mktemp' function">,
885:   Dependencies<[SecuritySyntaxChecker]>,
886:   Documentation<HasDocumentation>;
887: 
888: def mkstemp : Checker<"mkstemp">,
889:   HelpText<"Warn when 'mkstemp' is passed fewer than 6 X's in the format "
890:            "string">,
891:   Dependencies<[SecuritySyntaxChecker]>,
892:   Documentation<HasDocumentation>;
893: 
894: def rand : Checker<"rand">,
895:   HelpText<"Warn on uses of the 'rand', 'random', and related functions">,
896:   Dependencies<[SecuritySyntaxChecker]>,
897:   Documentation<HasDocumentation>;
898: 
899: def strcpy : Checker<"strcpy">,
900:   HelpText<"Warn on uses of the 'strcpy' and 'strcat' functions">,
901:   Dependencies<[SecuritySyntaxChecker]>,
902:   Documentation<HasDocumentation>;
903: 
904: def vfork : Checker<"vfork">,
905:   HelpText<"Warn on uses of the 'vfork' function">,
906:   Dependencies<[SecuritySyntaxChecker]>,
907:   Documentation<HasDocumentation>;
908: 
909: def UncheckedReturn : Checker<"UncheckedReturn">,
910:   HelpText<"Warn on uses of functions whose return values must be always "
911:            "checked">,
912:   Dependencies<[SecuritySyntaxChecker]>,
```
- EN: The TableGen records declared here include `bzero`, `gets`, `getpw`, `mktemp`.
- 中文: 这里声明的 TableGen 记录包括 `bzero`, `gets`, `getpw`, `mktemp`。

### Lines 913-960

```tablegen
913:   Documentation<HasDocumentation>;
914: 
915: def DeprecatedOrUnsafeBufferHandling
916:     : Checker<"DeprecatedOrUnsafeBufferHandling">,
917:       HelpText<"Warn on uses of unsecure or deprecated buffer manipulating "
918:                "functions">,
919:       Dependencies<[SecuritySyntaxChecker]>,
920:       CheckerOptions<
921:           [CmdLineOption<
922:                String, "ReportMode",
923:                "Controls when warnings are reported. \"all\" reports all "
924:                "unsafe functions regardless of C standard or Annex K "
925:                "availability. \"actionable\" only reports when Annex K is "
926:                "available (C11 with __STDC_LIB_EXT1__ and "
927:                "__STDC_WANT_LIB_EXT1__=1). \"c11-only\" reports when C11 "
928:                "standard is enabled (does not take Annex K availability into "
929:                "account).",
930:                "c11-only",
931:                Released>,
932:       ]>,
933:       Documentation<HasDocumentation>;
934: 
935: def decodeValueOfObjCType : Checker<"decodeValueOfObjCType">,
936:   HelpText<"Warn on uses of the '-decodeValueOfObjCType:at:' method">,
937:   Dependencies<[SecuritySyntaxChecker]>,
938:   Documentation<HasDocumentation>;
939: 
940: } // end "security.insecureAPI"
941: 
942: let ParentPackage = Security in {
943: 
944:   def ArrayBoundChecker : Checker<"ArrayBound">,
945:                           HelpText<"Warn about out of bounds access to memory">,
946:                           Documentation<HasDocumentation>;
947: 
948:   def FloatLoopCounter
949:       : Checker<"FloatLoopCounter">,
950:         HelpText<
951:             "Warn on using a floating point value as a loop counter (CERT: "
952:             "FLP30-C, FLP30-CPP)">,
953:         Dependencies<[SecuritySyntaxChecker]>,
954:         Documentation<HasDocumentation>;
955: 
956:   def MmapWriteExecChecker
957:       : Checker<"MmapWriteExec">,
958:         HelpText<
959:             "Warn on mmap() calls with both writable and executable access">,
960:         Documentation<HasDocumentation>;
```
- EN: The TableGen records declared here include `DeprecatedOrUnsafeBufferHandling`, `decodeValueOfObjCType`, `ArrayBoundChecker`, `FloatLoopCounter`.
- 中文: 这里声明的 TableGen 记录包括 `DeprecatedOrUnsafeBufferHandling`, `decodeValueOfObjCType`, `ArrayBoundChecker`, `FloatLoopCounter`。

### Lines 961-1008

```tablegen
 961: 
 962:   def PointerSubChecker
 963:       : Checker<"PointerSub">,
 964:         HelpText<"Check for pointer subtractions on two pointers pointing to "
 965:                  "different memory chunks">,
 966:         Documentation<HasDocumentation>;
 967: 
 968:   def PutenvStackArray
 969:       : Checker<"PutenvStackArray">,
 970:         HelpText<"Finds calls to the function 'putenv' which pass a pointer to "
 971:                  "an automatic (stack-allocated) array as the argument.">,
 972:         Documentation<HasDocumentation>;
 973: 
 974:   def SetgidSetuidOrderChecker
 975:       : Checker<"SetgidSetuidOrder">,
 976:         HelpText<"Warn on possible reversed order of 'setgid(getgid()))' and "
 977:                  "'setuid(getuid())' (CERT: POS36-C)">,
 978:         Documentation<HasDocumentation>;
 979: 
 980:   def VAListChecker : Checker<"VAList">,
 981:                       HelpText<"Warn on misuse of va_list objects">,
 982:                       Documentation<HasDocumentation>;
 983: 
 984: } // end "security"
 985: 
 986: let ParentPackage = ENV in {
 987: 
 988:   def InvalidPtrChecker : Checker<"InvalidPtr">,
 989:   HelpText<"Finds usages of possibly invalidated pointers">,
 990:   CheckerOptions<[
 991:     CmdLineOption<Boolean,
 992:                   "InvalidatingGetEnv",
 993:                   "Regard getenv as an invalidating call (as per POSIX "
 994:                   "standard), which can lead to false positives depending on "
 995:                   "implementation.",
 996:                   "false",
 997:                   Released>,
 998:   ]>,
 999:   Documentation<HasDocumentation>;
1000: 
1001: } // end "security.cert.env"
1002: 
1003: let ParentPackage = SecurityAlpha in {
1004: 
1005: def ReturnPointerRangeChecker : Checker<"ReturnPtrRange">,
1006:   HelpText<"Check for an out-of-bound pointer being returned to callers">,
1007:   Documentation<HasDocumentation>;
1008: 
```
- EN: The TableGen records declared here include `PointerSubChecker`, `PutenvStackArray`, `SetgidSetuidOrderChecker`, `VAListChecker`.
- 中文: 这里声明的 TableGen 记录包括 `PointerSubChecker`, `PutenvStackArray`, `SetgidSetuidOrderChecker`, `VAListChecker`。

### Lines 1009-1056

```tablegen
1009: } // end "alpha.security"
1010: 
1011: 
1012: //===----------------------------------------------------------------------===//
1013: // Mac OS X, Cocoa, and Core Foundation checkers.
1014: //===----------------------------------------------------------------------===//
1015: 
1016: let ParentPackage = Cocoa in {
1017: 
1018: def RetainCountBase : Checker<"RetainCountBase">,
1019:   HelpText<"Common base of various retain count related checkers">,
1020:   Documentation<NotDocumented>,
1021:   Hidden;
1022: 
1023: } // end "osx.cocoa"
1024: 
1025: let ParentPackage = OSX in {
1026: 
1027: def NumberObjectConversionChecker : Checker<"NumberObjectConversion">,
1028:   HelpText<"Check for erroneous conversions of objects representing numbers "
1029:            "into numbers">,
1030:   CheckerOptions<[
1031:     CmdLineOption<Boolean,
1032:                   "Pedantic",
1033:                   "Enables detection of more conversion patterns (which are "
1034:                   "most likely more harmless, and therefore are more likely to "
1035:                   "produce false positives).",
1036:                   "false",
1037:                   Released>
1038:   ]>,
1039:   Documentation<NotDocumented>;
1040: 
1041: def MacOSXAPIChecker : Checker<"API">,
1042:   HelpText<"Check for proper uses of various Apple APIs">,
1043:   Documentation<HasDocumentation>;
1044: 
1045: def MacOSKeychainAPIChecker : Checker<"SecKeychainAPI">,
1046:   HelpText<"Check for proper uses of Secure Keychain APIs">,
1047:   Documentation<HasDocumentation>;
1048: 
1049: def MIGChecker : Checker<"MIG">,
1050:   HelpText<"Find violations of the Mach Interface Generator "
1051:            "calling convention">,
1052:   Documentation<NotDocumented>;
1053: 
1054: def ObjCPropertyChecker : Checker<"ObjCProperty">,
1055:   HelpText<"Check for proper uses of Objective-C properties">,
1056:   Documentation<NotDocumented>;
```
- EN: The TableGen records declared here include `RetainCountBase`, `NumberObjectConversionChecker`, `MacOSXAPIChecker`, `MacOSKeychainAPIChecker`.
- 中文: 这里声明的 TableGen 记录包括 `RetainCountBase`, `NumberObjectConversionChecker`, `MacOSXAPIChecker`, `MacOSKeychainAPIChecker`。

### Lines 1057-1104

```tablegen
1057: 
1058: def OSObjectRetainCountChecker : Checker<"OSObjectRetainCount">,
1059:   HelpText<"Check for leaks and improper reference count management for "
1060:            "OSObject">,
1061:   Dependencies<[RetainCountBase]>,
1062:   Documentation<NotDocumented>;
1063: 
1064: } // end "osx"
1065: 
1066: let ParentPackage = Cocoa in {
1067: 
1068: def RunLoopAutoreleaseLeakChecker : Checker<"RunLoopAutoreleaseLeak">,
1069:   HelpText<"Check for leaked memory in autorelease pools that will never be "
1070:            "drained">,
1071:   Documentation<NotDocumented>;
1072: 
1073: def ObjCAtSyncChecker : Checker<"AtSync">,
1074:   HelpText<"Check for nil pointers used as mutexes for @synchronized">,
1075:   Documentation<HasDocumentation>;
1076: 
1077: def NilArgChecker : Checker<"NilArg">,
1078:   HelpText<"Check for prohibited nil arguments to ObjC method calls">,
1079:   Documentation<HasDocumentation>;
1080: 
1081: def ClassReleaseChecker : Checker<"ClassRelease">,
1082:   HelpText<"Check for sending 'retain', 'release', or 'autorelease' directly "
1083:            "to a Class">,
1084:   Documentation<HasDocumentation>;
1085: 
1086: def VariadicMethodTypeChecker : Checker<"VariadicMethodTypes">,
1087:   HelpText<"Check for passing non-Objective-C types to variadic collection "
1088:            "initialization methods that expect only Objective-C types">,
1089:   Documentation<HasDocumentation>;
1090: 
1091: def NSAutoreleasePoolChecker : Checker<"NSAutoreleasePool">,
1092:   HelpText<"Warn for suboptimal uses of NSAutoreleasePool in Objective-C GC "
1093:            "mode">,
1094:   Documentation<HasDocumentation>;
1095: 
1096: def ObjCMethSigsChecker : Checker<"IncompatibleMethodTypes">,
1097:   HelpText<"Warn about Objective-C method signatures with type "
1098:            "incompatibilities">,
1099:   Documentation<HasDocumentation>;
1100: 
1101: def ObjCUnusedIvarsChecker : Checker<"UnusedIvars">,
1102:   HelpText<"Warn about private ivars that are never used">,
1103:   Documentation<HasDocumentation>;
1104: 
```
- EN: The TableGen records declared here include `OSObjectRetainCountChecker`, `RunLoopAutoreleaseLeakChecker`, `ObjCAtSyncChecker`, `NilArgChecker`.
- 中文: 这里声明的 TableGen 记录包括 `OSObjectRetainCountChecker`, `RunLoopAutoreleaseLeakChecker`, `ObjCAtSyncChecker`, `NilArgChecker`。

### Lines 1105-1152

```tablegen
1105: def ObjCSelfInitChecker : Checker<"SelfInit">,
1106:   HelpText<"Check that 'self' is properly initialized inside an initializer "
1107:            "method">,
1108:   Documentation<HasDocumentation>;
1109: 
1110: def ObjCLoopChecker : Checker<"Loops">,
1111:   HelpText<"Improved modeling of loops using Cocoa collection types">,
1112:   Documentation<NotDocumented>;
1113: 
1114: def ObjCNonNilReturnValueChecker : Checker<"NonNilReturnValue">,
1115:   HelpText<"Model the APIs that are guaranteed to return a non-nil value">,
1116:   Documentation<NotDocumented>;
1117: 
1118: def ObjCSuperCallChecker : Checker<"MissingSuperCall">,
1119:   HelpText<"Warn about Objective-C methods that lack a necessary call to "
1120:            "super">,
1121:   Documentation<NotDocumented>;
1122: 
1123: def NSErrorChecker : Checker<"NSError">,
1124:                      HelpText<"Check usage of NSError** parameters">,
1125:                      Documentation<HasDocumentation>;
1126: 
1127: def RetainCountChecker : Checker<"RetainCount">,
1128:   HelpText<"Check for leaks and improper reference count management">,
1129:   CheckerOptions<[
1130:     CmdLineOption<Boolean,
1131:                   "TrackNSCFStartParam",
1132:                   "Check not only that the code follows retain-release rules "
1133:                   "with respect to objects it allocates or borrows from "
1134:                   "elsewhere, but also that it fulfills its own retain count "
1135:                   "specification with respect to objects that it receives as "
1136:                   "arguments.",
1137:                   "false",
1138:                   Released>
1139:   ]>,
1140:   Dependencies<[RetainCountBase]>,
1141:   Documentation<HasDocumentation>;
1142: 
1143: def ObjCGenericsChecker : Checker<"ObjCGenerics">,
1144:   HelpText<"Check for type errors when using Objective-C generics">,
1145:   Dependencies<[DynamicTypePropagation]>,
1146:   Documentation<HasDocumentation>;
1147: 
1148: def ObjCDeallocChecker : Checker<"Dealloc">,
1149:   HelpText<"Warn about Objective-C classes that lack a correct implementation "
1150:            "of -dealloc">,
1151:   Documentation<HasDocumentation>;
1152: 
```
- EN: The TableGen records declared here include `ObjCSelfInitChecker`, `ObjCLoopChecker`, `ObjCNonNilReturnValueChecker`, `ObjCSuperCallChecker`.
- 中文: 这里声明的 TableGen 记录包括 `ObjCSelfInitChecker`, `ObjCLoopChecker`, `ObjCNonNilReturnValueChecker`, `ObjCSuperCallChecker`。

### Lines 1153-1200

```tablegen
1153: def ObjCSuperDeallocChecker : Checker<"SuperDealloc">,
1154:   HelpText<"Warn about improper use of '[super dealloc]' in Objective-C">,
1155:   Documentation<HasDocumentation>;
1156: 
1157: def AutoreleaseWriteChecker : Checker<"AutoreleaseWrite">,
1158:   HelpText<"Warn about potentially crashing writes to autoreleasing objects "
1159:            "from different autoreleasing pools in Objective-C">,
1160:   Documentation<NotDocumented>;
1161: 
1162: } // end "osx.cocoa"
1163: 
1164: let ParentPackage = Performance in {
1165: 
1166: def GCDAntipattern : Checker<"GCDAntipattern">,
1167:   HelpText<"Check for performance anti-patterns when using Grand Central "
1168:            "Dispatch">,
1169:   Documentation<NotDocumented>;
1170: } // end "optin.performance"
1171: 
1172: let ParentPackage = OSXOptIn in {
1173: 
1174: def OSObjectCStyleCast : Checker<"OSObjectCStyleCast">,
1175:   HelpText<"Checker for C-style casts of OSObjects">,
1176:   Documentation<NotDocumented>;
1177: 
1178: } // end "optin.osx"
1179: 
1180: let ParentPackage = CocoaAlpha in {
1181: 
1182: def IvarInvalidationModeling : Checker<"IvarInvalidationModeling">,
1183:   HelpText<"Gathers information for annotation driven invalidation checking "
1184:            "for classes that contains a method annotated with "
1185:            "'objc_instance_variable_invalidator'">,
1186:   Documentation<NotDocumented>,
1187:   Hidden;
1188: 
1189: def InstanceVariableInvalidation : Checker<"InstanceVariableInvalidation">,
1190:   HelpText<"Check that the invalidatable instance variables are invalidated in "
1191:            "the methods annotated with objc_instance_variable_invalidator">,
1192:   Dependencies<[IvarInvalidationModeling]>,
1193:   Documentation<HasDocumentation>;
1194: 
1195: def MissingInvalidationMethod : Checker<"MissingInvalidationMethod">,
1196:   HelpText<"Check that the invalidation methods are present in classes that "
1197:            "contain invalidatable instance variables">,
1198:   Dependencies<[IvarInvalidationModeling]>,
1199:   Documentation<HasDocumentation>;
1200: 
```
- EN: The TableGen records declared here include `ObjCSuperDeallocChecker`, `AutoreleaseWriteChecker`, `GCDAntipattern`, `OSObjectCStyleCast`.
- 中文: 这里声明的 TableGen 记录包括 `ObjCSuperDeallocChecker`, `AutoreleaseWriteChecker`, `GCDAntipattern`, `OSObjectCStyleCast`。

### Lines 1201-1248

```tablegen
1201: def DirectIvarAssignment : Checker<"DirectIvarAssignment">,
1202:   HelpText<"Check for direct assignments to instance variables">,
1203:   CheckerOptions<[
1204:     CmdLineOption<Boolean,
1205:                   "AnnotatedFunctions",
1206:                   "Check for direct assignments to instance variables in the "
1207:                   "methods annotated with "
1208:                   "objc_no_direct_instance_variable_assignment",
1209:                   "false",
1210:                   InAlpha>
1211:   ]>,
1212:   Documentation<HasDocumentation>;
1213: 
1214: } // end "alpha.osx.cocoa"
1215: 
1216: let ParentPackage = CoreFoundation in {
1217: 
1218: def CFNumberChecker : Checker<"CFNumber">,
1219:   HelpText<"Check for proper uses of CFNumber APIs">,
1220:   Documentation<HasDocumentation>;
1221: 
1222: def CFRetainReleaseChecker : Checker<"CFRetainRelease">,
1223:   HelpText<"Check for null arguments to CFRetain/CFRelease/CFMakeCollectable">,
1224:   Documentation<HasDocumentation>;
1225: 
1226: def CFErrorChecker : Checker<"CFError">,
1227:                      HelpText<"Check usage of CFErrorRef* parameters">,
1228:                      Documentation<HasDocumentation>;
1229: 
1230: } // end "osx.coreFoundation"
1231: 
1232: let ParentPackage = Containers in {
1233: 
1234: def ObjCContainersASTChecker : Checker<"PointerSizedValues">,
1235:   HelpText<"Warns if 'CFArray', 'CFDictionary', 'CFSet' are created with "
1236:            "non-pointer-size values">,
1237:   Documentation<HasDocumentation>;
1238: 
1239: def ObjCContainersChecker : Checker<"OutOfBounds">,
1240:   HelpText<"Checks for index out-of-bounds when using 'CFArray' API">,
1241:   Documentation<HasDocumentation>;
1242: 
1243: } // end "osx.coreFoundation.containers"
1244: 
1245: let ParentPackage = LocalizabilityOptIn in {
1246: 
1247: def NonLocalizedStringChecker : Checker<"NonLocalizedStringChecker">,
1248:   HelpText<"Warns about uses of non-localized NSStrings passed to UI methods "
```
- EN: The TableGen records declared here include `DirectIvarAssignment`, `CFNumberChecker`, `CFRetainReleaseChecker`, `CFErrorChecker`.
- 中文: 这里声明的 TableGen 记录包括 `DirectIvarAssignment`, `CFNumberChecker`, `CFRetainReleaseChecker`, `CFErrorChecker`。

### Lines 1249-1296

```tablegen
1249:            "expecting localized NSStrings">,
1250:   CheckerOptions<[
1251:     CmdLineOption<Boolean,
1252:                   "AggressiveReport",
1253:                   "Marks a string being returned by any call as localized if "
1254:                   "it is in LocStringFunctions (LSF) or the function is "
1255:                   "annotated. Otherwise, we mark it as NonLocalized "
1256:                   "(Aggressive) or NonLocalized only if it is not backed by a "
1257:                   "SymRegion (Non-Aggressive), basically leaving only string "
1258:                   "literals as NonLocalized.",
1259:                   "false",
1260:                   InAlpha,
1261:                   Hide>
1262:   ]>,
1263:   Documentation<HasDocumentation>;
1264: 
1265: def EmptyLocalizationContextChecker :
1266:   Checker<"EmptyLocalizationContextChecker">,
1267:   HelpText<"Check that NSLocalizedString macros include a comment for context">,
1268:   Documentation<HasDocumentation>;
1269: 
1270: } // end "optin.osx.cocoa.localizability"
1271: 
1272: let ParentPackage = LocalizabilityAlpha in {
1273: 
1274: def PluralMisuseChecker : Checker<"PluralMisuseChecker">,
1275:   HelpText<"Warns against using one vs. many plural pattern in code when "
1276:            "generating localized strings.">,
1277:   Documentation<HasDocumentation>;
1278: 
1279: } // end "alpha.osx.cocoa.localizability"
1280: 
1281: let ParentPackage = MPI in {
1282: 
1283: def MPIChecker : Checker<"MPI-Checker">,
1284:   HelpText<"Checks MPI code">,
1285:   Documentation<HasDocumentation>;
1286: 
1287: } // end "optin.mpi"
1288: 
1289: //===----------------------------------------------------------------------===//
1290: // Checkers for LLVM development.
1291: //===----------------------------------------------------------------------===//
1292: 
1293: let ParentPackage = LLVMAlpha in {
1294: 
1295: def LLVMConventionsChecker : Checker<"Conventions">,
1296:   HelpText<"Check code for LLVM codebase conventions">,
```
- EN: The TableGen records declared here include `EmptyLocalizationContextChecker`, `PluralMisuseChecker`, `MPIChecker`, `LLVMConventionsChecker`.
- 中文: 这里声明的 TableGen 记录包括 `EmptyLocalizationContextChecker`, `PluralMisuseChecker`, `MPIChecker`, `LLVMConventionsChecker`。

### Lines 1297-1344

```tablegen
1297:   Documentation<HasDocumentation>;
1298: 
1299: } // end "llvm"
1300: 
1301: let ParentPackage = LLVMAPIModeling in {
1302: 
1303: def CastValueChecker : Checker<"CastValue">,
1304:   HelpText<"Model implementation of custom RTTIs">,
1305:   Documentation<NotDocumented>;
1306: 
1307: def ReturnValueChecker : Checker<"ReturnValue">,
1308:   HelpText<"Model certain Error() methods that always return true by convention">,
1309:   Documentation<NotDocumented>;
1310: 
1311: } // end "apiModeling.llvm"
1312: 
1313: //===----------------------------------------------------------------------===//
1314: // Checkers modeling Google APIs.
1315: //===----------------------------------------------------------------------===//
1316: 
1317: let ParentPackage = GoogleAPIModeling in {
1318: 
1319: def GTestChecker : Checker<"GTest">,
1320:   HelpText<"Model gtest assertion APIs">,
1321:   Documentation<NotDocumented>;
1322: 
1323: } // end "apiModeling.google"
1324: 
1325: //===----------------------------------------------------------------------===//
1326: // Debugging checkers (for analyzer development).
1327: //===----------------------------------------------------------------------===//
1328: 
1329: let ParentPackage = Debug in {
1330: 
1331: def AnalysisOrderChecker : Checker<"AnalysisOrder">,
1332:   HelpText<"Print callbacks that are called during analysis in order">,
1333:   CheckerOptions<[
1334:     CmdLineOption<Boolean,
1335:                   "PreStmtCastExpr",
1336:                   "",
1337:                   "false",
1338:                   Released,
1339:                   Hide>,
1340:     CmdLineOption<Boolean,
1341:                   "PostStmtCastExpr",
1342:                   "",
1343:                   "false",
1344:                   Released,
```
- EN: The TableGen records declared here include `CastValueChecker`, `ReturnValueChecker`, `GTestChecker`, `AnalysisOrderChecker`.
- 中文: 这里声明的 TableGen 记录包括 `CastValueChecker`, `ReturnValueChecker`, `GTestChecker`, `AnalysisOrderChecker`。

### Lines 1345-1392

```tablegen
1345:                   Hide>,
1346:     CmdLineOption<Boolean,
1347:                   "PreStmtArraySubscriptExpr",
1348:                   "",
1349:                   "false",
1350:                   Released,
1351:                   Hide>,
1352:     CmdLineOption<Boolean,
1353:                   "PostStmtArraySubscriptExpr",
1354:                   "",
1355:                   "false",
1356:                   Released,
1357:                   Hide>,
1358:     CmdLineOption<Boolean,
1359:                   "PreStmtCXXNewExpr",
1360:                   "",
1361:                   "false",
1362:                   Released,
1363:                   Hide>,
1364:     CmdLineOption<Boolean,
1365:                   "PostStmtCXXNewExpr",
1366:                   "",
1367:                   "false",
1368:                   Released,
1369:                   Hide>,
1370:     CmdLineOption<Boolean,
1371:                   "PreStmtCXXDeleteExpr",
1372:                   "",
1373:                   "false",
1374:                   Released,
1375:                   Hide>,
1376:     CmdLineOption<Boolean,
1377:                   "PostStmtCXXDeleteExpr",
1378:                   "",
1379:                   "false",
1380:                   Released,
1381:                   Hide>,
1382:     CmdLineOption<Boolean,
1383:                   "PreStmtCXXConstructExpr",
1384:                   "",
1385:                   "false",
1386:                   Released,
1387:                   Hide>,
1388:     CmdLineOption<Boolean,
1389:                   "PostStmtCXXConstructExpr",
1390:                   "",
1391:                   "false",
1392:                   Released,
```
- EN: The lines continue connective syntax and structural scaffolding for the surrounding definition.
- 中文: 这些行延续了周围定义所需的连接语法和结构骨架。

### Lines 1393-1440

```tablegen
1393:                   Hide>,
1394:     CmdLineOption<Boolean,
1395:                   "PreStmtOffsetOfExpr",
1396:                   "",
1397:                   "false",
1398:                   Released,
1399:                   Hide>,
1400:     CmdLineOption<Boolean,
1401:                   "PostStmtOffsetOfExpr",
1402:                   "",
1403:                   "false",
1404:                   Released,
1405:                   Hide>,
1406:     CmdLineOption<Boolean,
1407:                   "EvalCall",
1408:                   "",
1409:                   "false",
1410:                   Released,
1411:                   Hide>,
1412:     CmdLineOption<Boolean,
1413:                   "PreCall",
1414:                   "",
1415:                   "false",
1416:                   Released,
1417:                   Hide>,
1418:     CmdLineOption<Boolean,
1419:                   "PostCall",
1420:                   "",
1421:                   "false",
1422:                   Released,
1423:                   Hide>,
1424:     CmdLineOption<Boolean,
1425:                   "EndFunction",
1426:                   "",
1427:                   "false",
1428:                   Released,
1429:                   Hide>,
1430:     CmdLineOption<Boolean,
1431:                   "EndAnalysis",
1432:                   "",
1433:                   "false",
1434:                   Released,
1435:                   Hide>,
1436:     CmdLineOption<Boolean,
1437:                   "NewAllocator",
1438:                   "",
1439:                   "false",
1440:                   Released,
```
- EN: The lines continue connective syntax and structural scaffolding for the surrounding definition.
- 中文: 这些行延续了周围定义所需的连接语法和结构骨架。

### Lines 1441-1488

```tablegen
1441:                   Hide>,
1442:     CmdLineOption<Boolean,
1443:                   "Bind",
1444:                   "",
1445:                   "false",
1446:                   Released,
1447:                   Hide>,
1448:     CmdLineOption<Boolean,
1449:                   "LiveSymbols",
1450:                   "",
1451:                   "false",
1452:                   Released,
1453:                   Hide>,
1454:     CmdLineOption<Boolean,
1455:                   "RegionChanges",
1456:                   "",
1457:                   "false",
1458:                   Released,
1459:                   Hide>,
1460:     CmdLineOption<Boolean,
1461:                   "PointerEscape",
1462:                   "",
1463:                   "false",
1464:                   Released,
1465:                   Hide>,
1466:     CmdLineOption<Boolean,
1467:                   "*",
1468:                   "Enables all callbacks.",
1469:                   "false",
1470:                   Released,
1471:                   Hide>
1472:   ]>,
1473:   Documentation<NotDocumented>;
1474: 
1475: def DominatorsTreeDumper : Checker<"DumpDominators">,
1476:   HelpText<"Print the dominance tree for a given CFG">,
1477:   Documentation<NotDocumented>;
1478: 
1479: def PostDominatorsTreeDumper : Checker<"DumpPostDominators">,
1480:   HelpText<"Print the post dominance tree for a given CFG">,
1481:   Documentation<NotDocumented>;
1482: 
1483: def ControlDependencyTreeDumper : Checker<"DumpControlDependencies">,
1484:   HelpText<"Print the post control dependency tree for a given CFG">,
1485:   Documentation<NotDocumented>;
1486: 
1487: def LiveVariablesDumper : Checker<"DumpLiveVars">,
1488:   HelpText<"Print results of live variable analysis">,
```
- EN: The TableGen records declared here include `DominatorsTreeDumper`, `PostDominatorsTreeDumper`, `ControlDependencyTreeDumper`, `LiveVariablesDumper`.
- 中文: 这里声明的 TableGen 记录包括 `DominatorsTreeDumper`, `PostDominatorsTreeDumper`, `ControlDependencyTreeDumper`, `LiveVariablesDumper`。

### Lines 1489-1536

```tablegen
1489:   Documentation<NotDocumented>;
1490: 
1491: def LiveExpressionsDumper : Checker<"DumpLiveExprs">,
1492:   HelpText<"Print results of live expression analysis">,
1493:   Documentation<NotDocumented>;
1494: 
1495: def CFGViewer : Checker<"ViewCFG">,
1496:   HelpText<"View Control-Flow Graphs using GraphViz">,
1497:   Documentation<NotDocumented>;
1498: 
1499: def CFGDumper : Checker<"DumpCFG">,
1500:   HelpText<"Display Control-Flow Graphs">,
1501:   Documentation<NotDocumented>;
1502: 
1503: def CallGraphViewer : Checker<"ViewCallGraph">,
1504:   HelpText<"View Call Graph using GraphViz">,
1505:   Documentation<NotDocumented>;
1506: 
1507: def CallGraphDumper : Checker<"DumpCallGraph">,
1508:   HelpText<"Display Call Graph">,
1509:   Documentation<NotDocumented>;
1510: 
1511: def ConfigDumper : Checker<"ConfigDumper">,
1512:   HelpText<"Dump config table">,
1513:   Documentation<NotDocumented>;
1514: 
1515: def TraversalDumper : Checker<"DumpTraversal">,
1516:   HelpText<"Print branch conditions as they are traversed by the engine">,
1517:   Documentation<NotDocumented>;
1518: 
1519: def CallDumper : Checker<"DumpCalls">,
1520:   HelpText<"Print calls as they are traversed by the engine">,
1521:   Documentation<NotDocumented>;
1522: 
1523: def AnalyzerStatsChecker : Checker<"Stats">,
1524:   HelpText<"Emit warnings with analyzer statistics">,
1525:   Documentation<NotDocumented>;
1526: 
1527: def TaintTesterChecker : Checker<"TaintTest">,
1528:   HelpText<"Mark tainted symbols as such.">,
1529:   Documentation<NotDocumented>;
1530: 
1531: // This checker *technically* depends on SteamChecker, but we don't allow
1532: // dependency checkers to emit diagnostics, and a debug checker isn't worth
1533: // the chore needed to create a modeling portion on its own. Since this checker
1534: // is for development purposes only anyways, make sure that StreamChecker is
1535: // also enabled, at least for the time being.
1536: def StreamTesterChecker : Checker<"StreamTester">,
```
- EN: The TableGen records declared here include `LiveExpressionsDumper`, `CFGViewer`, `CFGDumper`, `CallGraphViewer`.
- 中文: 这里声明的 TableGen 记录包括 `LiveExpressionsDumper`, `CFGViewer`, `CFGDumper`, `CallGraphViewer`。

### Lines 1537-1584

```tablegen
1537:   HelpText<"Add test functions to StreamChecker for test and debugging "
1538:            "purposes.">,
1539:   WeakDependencies<[StreamChecker]>,
1540:   Documentation<NotDocumented>;
1541: 
1542: def ErrnoTesterChecker : Checker<"ErrnoTest">,
1543:   HelpText<"Check modeling aspects of 'errno'.">,
1544:   Dependencies<[ErrnoModeling]>,
1545:   Documentation<NotDocumented>;
1546: 
1547: def ExprInspectionChecker : Checker<"ExprInspection">,
1548:   HelpText<"Check the analyzer's understanding of expressions">,
1549:   Documentation<NotDocumented>;
1550: 
1551: def ExplodedGraphViewer : Checker<"ViewExplodedGraph">,
1552:   HelpText<"View Exploded Graphs using GraphViz">,
1553:   Documentation<NotDocumented>;
1554: 
1555: def ReportStmts : Checker<"ReportStmts">,
1556:   HelpText<"Emits a warning for every statement.">,
1557:   Documentation<NotDocumented>;
1558: 
1559: def DebugContainerModeling : Checker<"DebugContainerModeling">,
1560:   HelpText<"Check the analyzer's understanding of C++ containers">,
1561:   Dependencies<[ContainerModeling]>,
1562:   Documentation<NotDocumented>;
1563: 
1564: def DebugIteratorModeling : Checker<"DebugIteratorModeling">,
1565:   HelpText<"Check the analyzer's understanding of C++ iterators">,
1566:   Dependencies<[DebugContainerModeling, IteratorModeling]>,
1567:   Documentation<NotDocumented>;
1568: 
1569: def StdCLibraryFunctionsTesterChecker : Checker<"StdCLibraryFunctionsTester">,
1570:   HelpText<"Add test functions to the summary map, so testing of individual "
1571:            "summary constituents becomes possible.">,
1572:   WeakDependencies<[StdCLibraryFunctionsChecker]>,
1573:   Documentation<NotDocumented>;
1574: 
1575: def CheckerDocumentationChecker : Checker<"CheckerDocumentation">,
1576:   HelpText<"Defines an empty checker callback for all possible handlers.">,
1577:   Documentation<NotDocumented>;
1578: 
1579: } // end "debug"
1580: 
1581: 
1582: //===----------------------------------------------------------------------===//
1583: // Clone Detection
1584: //===----------------------------------------------------------------------===//
```
- EN: The TableGen records declared here include `ErrnoTesterChecker`, `ExprInspectionChecker`, `ExplodedGraphViewer`, `ReportStmts`.
- 中文: 这里声明的 TableGen 记录包括 `ErrnoTesterChecker`, `ExprInspectionChecker`, `ExplodedGraphViewer`, `ReportStmts`。

### Lines 1585-1632

```tablegen
1585: 
1586: let ParentPackage = CloneDetectionAlpha in {
1587: 
1588: def CloneChecker : Checker<"CloneChecker">,
1589:   HelpText<"Reports similar pieces of code.">,
1590:   CheckerOptions<[
1591:     CmdLineOption<Integer,
1592:                   "MinimumCloneComplexity",
1593:                   "Ensures that every clone has at least the given complexity. "
1594:                   "Complexity is here defined as the total amount of children "
1595:                   "of a statement. This constraint assumes the first statement "
1596:                   "in the group is representative for all other statements in "
1597:                   "the group in terms of complexity.",
1598:                   "50",
1599:                   Released>,
1600:     CmdLineOption<Boolean,
1601:                   "ReportNormalClones",
1602:                   "Report all clones, even less suspicious ones.",
1603:                   "true",
1604:                   Released>,
1605:     CmdLineOption<String,
1606:                   "IgnoredFilesPattern",
1607:                   "If supplied, the checker wont analyze files with a filename "
1608:                   "that matches the given pattern.",
1609:                   "\"\"",
1610:                   Released>
1611:   ]>,
1612:   Documentation<HasDocumentation>;
1613: 
1614: } // end "clone"
1615: 
1616: //===----------------------------------------------------------------------===//
1617: // Portability checkers.
1618: //===----------------------------------------------------------------------===//
1619: 
1620: let ParentPackage = PortabilityOptIn in {
1621: 
1622: def UnixAPIPortabilityChecker : Checker<"UnixAPI">,
1623:   HelpText<"Finds dynamic memory allocation with size zero">,
1624:   Documentation<HasDocumentation>;
1625: 
1626: } // end optin.portability
1627: 
1628: 
1629: //===----------------------------------------------------------------------===//
1630: // Taint checkers.
1631: //===----------------------------------------------------------------------===//
1632: 
```
- EN: The TableGen records declared here include `CloneChecker`, `UnixAPIPortabilityChecker`.
- 中文: 这里声明的 TableGen 记录包括 `CloneChecker`, `UnixAPIPortabilityChecker`。

### Lines 1633-1680

```tablegen
1633: let ParentPackage = TaintOptIn in {
1634: 
1635: def TaintPropagationChecker : Checker<"TaintPropagation">, // Modelling checker
1636:   HelpText<"Generate taint information used by other checkers">,
1637:   CheckerOptions<[
1638:     CmdLineOption<String,
1639:                   "Config",
1640:                   "Specifies the name of the configuration file.",
1641:                   "",
1642:                   Released>,
1643:     CmdLineOption<Boolean,
1644:                   "EnableDefaultConfig",
1645:                   "If set to false, the default source, "
1646:                   "sink and propagation rules are not loaded."
1647:                   "This way, advanced users can fully customize "
1648:                   "their taint configuration model.",
1649:                   "true",
1650:                   Released>
1651:   ]>,
1652:   Documentation<NotDocumented>,
1653:   Hidden;
1654: 
1655: def GenericTaintChecker : Checker<"GenericTaint">,
1656:   HelpText<"Reports potential injection vulnerabilities">,
1657:   Dependencies<[TaintPropagationChecker]>,
1658:   Documentation<HasDocumentation>;
1659: 
1660: 
1661: def TaintedAllocChecker: Checker<"TaintedAlloc">,
1662:   HelpText<"Check for memory allocations, where the size parameter "
1663:            "might be a tainted (attacker controlled) value.">,
1664:   Dependencies<[DynamicMemoryModeling, TaintPropagationChecker]>,
1665:   Documentation<HasDocumentation>;
1666: 
1667: def TaintedDivChecker: Checker<"TaintedDiv">,
1668:   HelpText<"Check for divisions where the denominator is tainted "
1669:            "(attacker controlled) and might be 0.">,
1670:   Dependencies<[TaintPropagationChecker]>,
1671:   Documentation<HasDocumentation>;
1672: 
1673: } // end "optin.taint"
1674: 
1675: //===----------------------------------------------------------------------===//
1676: // Fuchsia checkers.
1677: //===----------------------------------------------------------------------===//
1678: 
1679: let ParentPackage = Fuchsia in {
1680: 
```
- EN: The TableGen records declared here include `TaintPropagationChecker`, `GenericTaintChecker`, `TaintedAllocChecker`, `TaintedDivChecker`.
- 中文: 这里声明的 TableGen 记录包括 `TaintPropagationChecker`, `GenericTaintChecker`, `TaintedAllocChecker`, `TaintedDivChecker`。

### Lines 1681-1728

```tablegen
1681: def FuchsiaHandleChecker : Checker<"HandleChecker">,
1682:   HelpText<"A Checker that detect leaks related to Fuchsia handles">,
1683:   Documentation<HasDocumentation>;
1684: 
1685: }
1686: 
1687: let ParentPackage = FuchsiaAlpha in {
1688: 
1689: def FuchsiaLockChecker : Checker<"Lock">,
1690:   HelpText<"Check for the correct usage of locking APIs.">,
1691:   Dependencies<[PthreadLockBase]>,
1692:   Documentation<HasDocumentation>;
1693: 
1694: } // end fuchsia
1695: 
1696: //===----------------------------------------------------------------------===//
1697: // WebKit checkers.
1698: //===----------------------------------------------------------------------===//
1699: 
1700: let ParentPackage = WebKit in {
1701: 
1702: def RefCntblBaseVirtualDtorChecker : Checker<"RefCntblBaseVirtualDtor">,
1703:   HelpText<"Check for any ref-countable base class having virtual destructor.">,
1704:   Documentation<HasDocumentation>;
1705: 
1706: def NoUncountedMemberChecker : Checker<"NoUncountedMemberChecker">,
1707:   HelpText<"Check for no uncounted member variables.">,
1708:   Documentation<HasDocumentation>;
1709: 
1710: def UncountedLambdaCapturesChecker : Checker<"UncountedLambdaCapturesChecker">,
1711:   HelpText<"Check uncounted lambda captures.">,
1712:   Documentation<HasDocumentation>;
1713: 
1714: } // end webkit
1715: 
1716: let ParentPackage = WebKitAlpha in {
1717: 
1718: def ForwardDeclChecker : Checker<"ForwardDeclChecker">,
1719:   HelpText<"Check for forward declared local or member variables and function arguments">,
1720:   Documentation<HasDocumentation>;
1721: 
1722: def MemoryUnsafeCastChecker : Checker<"MemoryUnsafeCastChecker">,
1723:   HelpText<"Check for memory unsafe casts from base type to derived type.">,
1724:   Documentation<HasDocumentation>;
1725: 
1726: def NoDeleteChecker : Checker<"NoDeleteChecker">,
1727:   HelpText<"Check for [[clang::annotate_type(\"webkit.nodelete\")]] annotation">,
1728:   Documentation<HasDocumentation>;
```
- EN: Key type declarations here include `having`. The TableGen records declared here include `FuchsiaHandleChecker`, `FuchsiaLockChecker`, `RefCntblBaseVirtualDtorChecker`, `NoUncountedMemberChecker`.
- 中文: 这里的重要类型声明包括 `having`。 这里声明的 TableGen 记录包括 `FuchsiaHandleChecker`, `FuchsiaLockChecker`, `RefCntblBaseVirtualDtorChecker`, `NoUncountedMemberChecker`。

### Lines 1729-1770

```tablegen
1729: 
1730: def NoUncheckedPtrMemberChecker : Checker<"NoUncheckedPtrMemberChecker">,
1731:   HelpText<"Check for no unchecked member variables.">,
1732:   Documentation<HasDocumentation>;
1733: 
1734: def NoUnretainedMemberChecker : Checker<"NoUnretainedMemberChecker">,
1735:   HelpText<"Check for no unretained member variables.">,
1736:   Documentation<HasDocumentation>;
1737: 
1738: def UnretainedLambdaCapturesChecker : Checker<"UnretainedLambdaCapturesChecker">,
1739:   HelpText<"Check unretained lambda captures.">,
1740:   Documentation<HasDocumentation>;
1741: 
1742: def UncountedCallArgsChecker : Checker<"UncountedCallArgsChecker">,
1743:   HelpText<"Check uncounted call arguments.">,
1744:   Documentation<HasDocumentation>;
1745: 
1746: def UncheckedCallArgsChecker : Checker<"UncheckedCallArgsChecker">,
1747:   HelpText<"Check unchecked call arguments.">,
1748:   Documentation<HasDocumentation>;
1749: 
1750: def UnretainedCallArgsChecker : Checker<"UnretainedCallArgsChecker">,
1751:   HelpText<"Check unretained call arguments.">,
1752:   Documentation<HasDocumentation>;
1753: 
1754: def UncountedLocalVarsChecker : Checker<"UncountedLocalVarsChecker">,
1755:   HelpText<"Check uncounted local variables.">,
1756:   Documentation<HasDocumentation>;
1757: 
1758: def UncheckedLocalVarsChecker : Checker<"UncheckedLocalVarsChecker">,
1759:   HelpText<"Check unchecked local variables.">,
1760:   Documentation<HasDocumentation>;
1761: 
1762: def UnretainedLocalVarsChecker : Checker<"UnretainedLocalVarsChecker">,
1763:   HelpText<"Check unretained local variables.">,
1764:   Documentation<HasDocumentation>;
1765: 
1766: def RetainPtrCtorAdoptChecker : Checker<"RetainPtrCtorAdoptChecker">,
1767:   HelpText<"Check for correct use of RetainPtr/OSObjectPtr constructor, adoptNS, adoptCF, and adoptOSObject">,
1768:   Documentation<HasDocumentation>;
1769: 
1770: } // end alpha.webkit
```
- EN: The TableGen records declared here include `NoUncheckedPtrMemberChecker`, `NoUnretainedMemberChecker`, `UnretainedLambdaCapturesChecker`, `UncountedCallArgsChecker`.
- 中文: 这里声明的 TableGen 记录包括 `NoUncheckedPtrMemberChecker`, `NoUnretainedMemberChecker`, `UnretainedLambdaCapturesChecker`, `UncountedCallArgsChecker`。

## Key Concepts / 关键概念

- `Alpha`: A TableGen record used for declarative code generation or metadata description. / 用于声明式代码生成或元数据描述的 TableGen 记录。
- `Core`: A TableGen record used for declarative code generation or metadata description. / 用于声明式代码生成或元数据描述的 TableGen 记录。
- `CoreBuiltin`: A TableGen record used for declarative code generation or metadata description. / 用于声明式代码生成或元数据描述的 TableGen 记录。
- `CoreUninitialized`: A TableGen record used for declarative code generation or metadata description. / 用于声明式代码生成或元数据描述的 TableGen 记录。
- `CoreAlpha`: A TableGen record used for declarative code generation or metadata description. / 用于声明式代码生成或元数据描述的 TableGen 记录。
- `OptIn`: A TableGen record used for declarative code generation or metadata description. / 用于声明式代码生成或元数据描述的 TableGen 记录。
- `CoreOptIn`: A TableGen record used for declarative code generation or metadata description. / 用于声明式代码生成或元数据描述的 TableGen 记录。
- `PortabilityOptIn`: A TableGen record used for declarative code generation or metadata description. / 用于声明式代码生成或元数据描述的 TableGen 记录。

## Dependencies / 依赖关系

- Direct includes / 直接包含: None / 无
- Forward declarations / 前向声明: None / 无
- Namespace context / 命名空间上下文: None / 无
- Macro-style dependencies / 宏式依赖: None / 无
