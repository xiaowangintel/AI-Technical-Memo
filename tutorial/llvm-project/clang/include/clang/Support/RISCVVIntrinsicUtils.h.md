# RISCVVIntrinsicUtils.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Support/RISCVVIntrinsicUtils.h`
- Repository: `llvm-project`
- Purpose (EN): RISC-V Vector Intrinsic Utils.
- 用途（中文）: 该文件为 Support 子系统中的 RISCVV Intrinsic Utils 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-47

```cpp
 1: //===--- RISCVVIntrinsicUtils.h - RISC-V Vector Intrinsic Utils -*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: #ifndef CLANG_SUPPORT_RISCVVINTRINSICUTILS_H
10: #define CLANG_SUPPORT_RISCVVINTRINSICUTILS_H
11: 
12: #include "llvm/ADT/ArrayRef.h"
13: #include "llvm/ADT/BitmaskEnum.h"
14: #include "llvm/ADT/SmallVector.h"
15: #include "llvm/ADT/StringRef.h"
16: #include <cstdint>
17: #include <optional>
18: #include <set>
19: #include <string>
20: #include <unordered_map>
21: #include <vector>
22: 
23: namespace llvm {
24: class raw_ostream;
25: } // end namespace llvm
26: 
27: namespace clang {
28: namespace RISCV {
29: 
30: using VScaleVal = std::optional<unsigned>;
31: 
32: // Modifier for vector type.
33: enum class VectorTypeModifier : uint8_t {
34:   NoModifier,
35:   Widening2XVector,
36:   Widening4XVector,
37:   Widening8XVector,
38:   DoubleLMULVector,
39:   MaskVector,
40:   Log2EEW3,
41:   Log2EEW4,
42:   Log2EEW5,
43:   Log2EEW6,
44:   FixedSEW8,
45:   FixedSEW16,
46:   FixedSEW32,
47:   FixedSEW64,
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `llvm/ADT/ArrayRef.h`, `llvm/ADT/BitmaskEnum.h`, `llvm/ADT/SmallVector.h` and 7 more. It opens, closes, or documents namespace scope for `llvm`, `clang`, `RISCV`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `llvm/ADT/ArrayRef.h`, `llvm/ADT/BitmaskEnum.h`, `llvm/ADT/SmallVector.h` 以及另外 7 项依赖。 它打开、关闭或说明了 `llvm`, `clang`, `RISCV` 的命名空间作用域。

### Lines 48-94

```cpp
48:   LFixedLog2LMULN3,
49:   LFixedLog2LMULN2,
50:   LFixedLog2LMULN1,
51:   LFixedLog2LMUL0,
52:   LFixedLog2LMUL1,
53:   LFixedLog2LMUL2,
54:   LFixedLog2LMUL3,
55:   SFixedLog2LMULN3,
56:   SFixedLog2LMULN2,
57:   SFixedLog2LMULN1,
58:   SFixedLog2LMUL0,
59:   SFixedLog2LMUL1,
60:   SFixedLog2LMUL2,
61:   SFixedLog2LMUL3,
62:   SEFixedLog2LMULN3,
63:   SEFixedLog2LMULN2,
64:   SEFixedLog2LMULN1,
65:   SEFixedLog2LMUL0,
66:   SEFixedLog2LMUL1,
67:   SEFixedLog2LMUL2,
68:   SEFixedLog2LMUL3,
69:   Tuple2,
70:   Tuple3,
71:   Tuple4,
72:   Tuple5,
73:   Tuple6,
74:   Tuple7,
75:   Tuple8,
76: };
77: 
78: // Similar to basic type but used to describe what's kind of type related to
79: // basic vector type, used to compute type info of arguments.
80: enum class BaseTypeModifier : uint8_t {
81:   Invalid,
82:   Scalar,
83:   Vector,
84:   Void,
85:   SizeT,
86:   Ptrdiff,
87:   UnsignedLong,
88:   SignedLong,
89:   Float32
90: };
91: 
92: // Modifier for type, used for both scalar and vector types.
93: enum class TypeModifier : uint8_t {
94:   NoModifier = 0,
```
- EN: Key type declarations here include `BaseTypeModifier`, `TypeModifier`. It introduces enum-based state or option sets such as `BaseTypeModifier`, `TypeModifier`.
- 中文: 这里的重要类型声明包括 `BaseTypeModifier`, `TypeModifier`。 它引入了 `BaseTypeModifier`, `TypeModifier` 等基于枚举的状态或选项集合。

### Lines 95-141

```cpp
 95:   Pointer = 1 << 0,
 96:   Const = 1 << 1,
 97:   Immediate = 1 << 2,
 98:   UnsignedInteger = 1 << 3,
 99:   SignedInteger = 1 << 4,
100:   Float = 1 << 5,
101:   BFloat = 1 << 6,
102:   // LMUL1 should be kind of VectorTypeModifier, but that might come with
103:   // Widening2XVector for widening reduction.
104:   // However that might require VectorTypeModifier become bitmask rather than
105:   // simple enum, so we decide keek LMUL1 in TypeModifier for code size
106:   // optimization of clang binary size.
107:   LMUL1 = 1 << 7,
108:   MaxOffset = 7,
109:   LLVM_MARK_AS_BITMASK_ENUM(LMUL1),
110: };
111: 
112: class Policy {
113: public:
114:   enum PolicyType {
115:     Undisturbed,
116:     Agnostic,
117:   };
118: 
119: private:
120:   // The default assumption for an RVV instruction is TAMA, as an undisturbed
121:   // policy generally will affect the performance of an out-of-order core.
122:   const PolicyType TailPolicy = Agnostic;
123:   const PolicyType MaskPolicy = Agnostic;
124: 
125: public:
126:   Policy() = default;
127:   Policy(PolicyType TailPolicy) : TailPolicy(TailPolicy) {}
128:   Policy(PolicyType TailPolicy, PolicyType MaskPolicy)
129:       : TailPolicy(TailPolicy), MaskPolicy(MaskPolicy) {}
130: 
131:   bool isTAMAPolicy() const {
132:     return TailPolicy == Agnostic && MaskPolicy == Agnostic;
133:   }
134: 
135:   bool isTAMUPolicy() const {
136:     return TailPolicy == Agnostic && MaskPolicy == Undisturbed;
137:   }
138: 
139:   bool isTUMAPolicy() const {
140:     return TailPolicy == Undisturbed && MaskPolicy == Agnostic;
141:   }
```
- EN: Key type declarations here include `Policy`. It introduces enum-based state or option sets such as `PolicyType`. It exposes API surface such as `Policy`, `TailPolicy`, `isTAMAPolicy`, `isTAMUPolicy`.
- 中文: 这里的重要类型声明包括 `Policy`。 它引入了 `PolicyType` 等基于枚举的状态或选项集合。 它暴露了 `Policy`, `TailPolicy`, `isTAMAPolicy`, `isTAMUPolicy` 等接口。

### Lines 142-188

```cpp
142: 
143:   bool isTUMUPolicy() const {
144:     return TailPolicy == Undisturbed && MaskPolicy == Undisturbed;
145:   }
146: 
147:   bool isTAPolicy() const { return TailPolicy == Agnostic; }
148: 
149:   bool isTUPolicy() const { return TailPolicy == Undisturbed; }
150: 
151:   bool isMAPolicy() const { return MaskPolicy == Agnostic; }
152: 
153:   bool isMUPolicy() const { return MaskPolicy == Undisturbed; }
154: 
155:   bool operator==(const Policy &Other) const {
156:     return TailPolicy == Other.TailPolicy && MaskPolicy == Other.MaskPolicy;
157:   }
158: 
159:   bool operator!=(const Policy &Other) const { return !(*this == Other); }
160: 
161:   bool operator<(const Policy &Other) const {
162:     // Just for maintain the old order for quick test.
163:     if (MaskPolicy != Other.MaskPolicy)
164:       return Other.MaskPolicy < MaskPolicy;
165:     return TailPolicy < Other.TailPolicy;
166:   }
167: };
168: 
169: // PrototypeDescriptor is used to compute type info of arguments or return
170: // value.
171: struct PrototypeDescriptor {
172:   constexpr PrototypeDescriptor() = default;
173:   constexpr PrototypeDescriptor(
174:       BaseTypeModifier PT,
175:       VectorTypeModifier VTM = VectorTypeModifier::NoModifier,
176:       TypeModifier TM = TypeModifier::NoModifier)
177:       : PT(PT), VTM(VTM), TM(TM) {}
178:   constexpr PrototypeDescriptor(uint8_t PT, uint8_t VTM, uint8_t TM)
179:       : PT(static_cast<BaseTypeModifier>(PT)),
180:         VTM(static_cast<VectorTypeModifier>(VTM)),
181:         TM(static_cast<TypeModifier>(TM)) {}
182: 
183:   BaseTypeModifier PT = BaseTypeModifier::Invalid;
184:   VectorTypeModifier VTM = VectorTypeModifier::NoModifier;
185:   TypeModifier TM = TypeModifier::NoModifier;
186: 
187:   bool operator!=(const PrototypeDescriptor &PD) const {
188:     return !(*this == PD);
```
- EN: Key type declarations here include `PrototypeDescriptor`. It exposes API surface such as `isTUMUPolicy`, `isTAPolicy`, `isTUPolicy`, `isMAPolicy`.
- 中文: 这里的重要类型声明包括 `PrototypeDescriptor`。 它暴露了 `isTUMUPolicy`, `isTAPolicy`, `isTUPolicy`, `isMAPolicy` 等接口。

### Lines 189-235

```cpp
189:   }
190:   bool operator==(const PrototypeDescriptor &PD) const {
191:     return PD.PT == PT && PD.VTM == VTM && PD.TM == TM;
192:   }
193:   bool operator<(const PrototypeDescriptor &PD) const {
194:     return std::tie(PT, VTM, TM) < std::tie(PD.PT, PD.VTM, PD.TM);
195:   }
196:   static const PrototypeDescriptor Mask;
197:   static const PrototypeDescriptor Vector;
198:   static const PrototypeDescriptor VL;
199:   static std::optional<PrototypeDescriptor>
200:   parsePrototypeDescriptor(llvm::StringRef PrototypeStr);
201: };
202: 
203: llvm::SmallVector<PrototypeDescriptor>
204: parsePrototypes(llvm::StringRef Prototypes);
205: 
206: // Basic type of vector type.
207: enum class BasicType : uint16_t {
208:   Unknown = 0,
209:   Int8 = 1 << 0,
210:   Int16 = 1 << 1,
211:   Int32 = 1 << 2,
212:   Int64 = 1 << 3,
213:   BFloat16 = 1 << 4,
214:   Float16 = 1 << 5,
215:   Float32 = 1 << 6,
216:   Float64 = 1 << 7,
217:   F8E4M3 = 1 << 8,
218:   F8E5M2 = 1 << 9,
219:   MaxOffset = 9,
220:   LLVM_MARK_AS_BITMASK_ENUM(F8E5M2),
221: };
222: 
223: // Type of vector type.
224: enum ScalarTypeKind : uint8_t {
225:   Void,
226:   Size_t,
227:   Ptrdiff_t,
228:   UnsignedLong,
229:   SignedLong,
230:   Boolean,
231:   SignedInteger,
232:   UnsignedInteger,
233:   Float,
234:   BFloat,
235:   FloatE4M3,
```
- EN: Key type declarations here include `BasicType`. It introduces enum-based state or option sets such as `BasicType`, `ScalarTypeKind`. It exposes API surface such as `operator<`, `tie`, `parsePrototypeDescriptor`, `parsePrototypes`.
- 中文: 这里的重要类型声明包括 `BasicType`。 它引入了 `BasicType`, `ScalarTypeKind` 等基于枚举的状态或选项集合。 它暴露了 `operator<`, `tie`, `parsePrototypeDescriptor`, `parsePrototypes` 等接口。

### Lines 236-282

```cpp
236:   FloatE5M2,
237:   Invalid,
238:   Undefined,
239: };
240: 
241: // Exponential LMUL
242: struct LMULType {
243:   int Log2LMUL;
244:   LMULType(int Log2LMUL);
245:   // Return the C/C++ string representation of LMUL
246:   std::string str() const;
247:   std::optional<unsigned> getScale(unsigned ElementBitwidth) const;
248:   void MulLog2LMUL(int Log2LMUL);
249: };
250: 
251: class RVVType;
252: using RVVTypePtr = RVVType *;
253: using RVVTypes = std::vector<RVVTypePtr>;
254: class RVVTypeCache;
255: 
256: // This class is compact representation of a valid and invalid RVVType.
257: class RVVType {
258:   friend class RVVTypeCache;
259: 
260:   BasicType BT;
261:   ScalarTypeKind ScalarType = Undefined;
262:   LMULType LMUL;
263:   bool IsPointer = false;
264:   // IsConstant indices are "int", but have the constant expression.
265:   bool IsImmediate = false;
266:   // Const qualifier for pointer to const object or object of const type.
267:   bool IsConstant = false;
268:   unsigned ElementBitwidth = 0;
269:   VScaleVal Scale = 0;
270:   bool Valid;
271:   bool IsTuple = false;
272:   unsigned NF = 0;
273: 
274:   std::string BuiltinStr;
275:   std::string ClangBuiltinStr;
276:   std::string Str;
277:   std::string ShortStr;
278: 
279:   enum class FixedLMULType { LargerThan, SmallerThan, SmallerOrEqual };
280: 
281:   RVVType(BasicType BT, int Log2LMUL, const PrototypeDescriptor &Profile);
282: 
```
- EN: Key type declarations here include `LMULType`, `RVVType`, `RVVTypeCache`, `FixedLMULType`. It introduces enum-based state or option sets such as `FixedLMULType`. It defines convenient aliases such as `RVVTypePtr`, `RVVTypes`.
- 中文: 这里的重要类型声明包括 `LMULType`, `RVVType`, `RVVTypeCache`, `FixedLMULType`。 它引入了 `FixedLMULType` 等基于枚举的状态或选项集合。 它定义了 `RVVTypePtr`, `RVVTypes` 等便捷别名。

### Lines 283-329

```cpp
283: public:
284:   // Return the string representation of a type, which is an encoded string for
285:   // passing to the BUILTIN() macro in Builtins.def.
286:   const std::string &getBuiltinStr() const { return BuiltinStr; }
287: 
288:   // Return the clang builtin type for RVV vector type which are used in the
289:   // riscv_vector.h header file.
290:   const std::string &getClangBuiltinStr() const { return ClangBuiltinStr; }
291: 
292:   // Return the C/C++ string representation of a type for use in the
293:   // riscv_vector.h header file.
294:   const std::string &getTypeStr() const { return Str; }
295: 
296:   // Return the short name of a type for C/C++ name suffix.
297:   const std::string &getShortStr() {
298:     // Not all types are used in short name, so compute the short name by
299:     // demanded.
300:     if (ShortStr.empty())
301:       initShortStr();
302:     return ShortStr;
303:   }
304: 
305:   bool isValid() const { return Valid; }
306:   bool isScalar() const { return Scale && *Scale == 0; }
307:   bool isVector() const { return Scale && *Scale != 0; }
308:   bool isVector(unsigned Width) const {
309:     return isVector() && ElementBitwidth == Width;
310:   }
311:   bool isFloat() const { return ScalarType == ScalarTypeKind::Float; }
312:   bool isBFloat() const { return ScalarType == ScalarTypeKind::BFloat; }
313:   bool isSignedInteger() const {
314:     return ScalarType == ScalarTypeKind::SignedInteger;
315:   }
316:   bool isFloatVector(unsigned Width) const {
317:     return isVector() && isFloat() && ElementBitwidth == Width;
318:   }
319:   bool isFloat(unsigned Width) const {
320:     return isFloat() && ElementBitwidth == Width;
321:   }
322:   bool isConstant() const { return IsConstant; }
323:   bool isPointer() const { return IsPointer; }
324:   bool isTuple() const { return IsTuple; }
325:   unsigned getElementBitwidth() const { return ElementBitwidth; }
326: 
327:   ScalarTypeKind getScalarType() const { return ScalarType; }
328:   VScaleVal getScale() const { return Scale; }
329:   unsigned getNF() const {
```
- EN: It exposes API surface such as `getBuiltinStr`, `getClangBuiltinStr`, `getTypeStr`, `getShortStr`.
- 中文: 它暴露了 `getBuiltinStr`, `getClangBuiltinStr`, `getTypeStr`, `getShortStr` 等接口。

### Lines 330-376

```cpp
330:     assert(NF > 1 && NF <= 8 && "Only legal NF should be fetched");
331:     return NF;
332:   }
333: 
334: private:
335:   // Verify RVV vector type and set Valid.
336:   bool verifyType() const;
337: 
338:   // Creates a type based on basic types of TypeRange
339:   void applyBasicType();
340: 
341:   // Applies a prototype modifier to the current type. The result maybe an
342:   // invalid type.
343:   void applyModifier(const PrototypeDescriptor &prototype);
344: 
345:   void applyLog2EEW(unsigned Log2EEW);
346:   void applyFixedSEW(unsigned NewSEW);
347:   void applyFixedLog2LMUL(int Log2LMUL, enum FixedLMULType Type);
348: 
349:   // Compute and record a string for legal type.
350:   void initBuiltinStr();
351:   // Compute and record a builtin RVV vector type string.
352:   void initClangBuiltinStr();
353:   // Compute and record a type string for used in the header.
354:   void initTypeStr();
355:   // Compute and record a short name of a type for C/C++ name suffix.
356:   void initShortStr();
357: };
358: 
359: // This class is used to manage RVVType, RVVType should only created by this
360: // class, also provided thread-safe cache capability.
361: class RVVTypeCache {
362: private:
363:   std::unordered_map<uint64_t, RVVType> LegalTypes;
364:   std::set<uint64_t> IllegalTypes;
365: 
366: public:
367:   /// Compute output and input types by applying different config (basic type
368:   /// and LMUL with type transformers). It also record result of type in legal
369:   /// or illegal set to avoid compute the same config again. The result maybe
370:   /// have illegal RVVType.
371:   std::optional<RVVTypes>
372:   computeTypes(BasicType BT, int Log2LMUL, unsigned NF,
373:                llvm::ArrayRef<PrototypeDescriptor> Prototype);
374:   std::optional<RVVTypePtr> computeType(BasicType BT, int Log2LMUL,
375:                                         PrototypeDescriptor Proto);
376: };
```
- EN: Key type declarations here include `RVVTypeCache`. It introduces enum-based state or option sets such as `FixedLMULType`. It exposes API surface such as `assert`, `verifyType`, `applyBasicType`, `applyModifier`.
- 中文: 这里的重要类型声明包括 `RVVTypeCache`。 它引入了 `FixedLMULType` 等基于枚举的状态或选项集合。 它暴露了 `assert`, `verifyType`, `applyBasicType`, `applyModifier` 等接口。

### Lines 377-423

```cpp
377: 
378: enum PolicyScheme : uint8_t {
379:   SchemeNone,
380:   // Passthru operand is at first parameter in C builtin.
381:   HasPassthruOperand,
382:   HasPolicyOperand,
383: };
384: 
385: llvm::raw_ostream &operator<<(llvm::raw_ostream &OS, enum PolicyScheme PS);
386: 
387: // TODO refactor RVVIntrinsic class design after support all intrinsic
388: // combination. This represents an instantiation of an intrinsic with a
389: // particular type and prototype
390: class RVVIntrinsic {
391: 
392: private:
393:   std::string BuiltinName; // Builtin name
394:   std::string Name;        // C intrinsic name.
395:   std::string OverloadedName;
396:   std::string IRName;
397:   bool IsMasked;
398:   bool HasMaskedOffOperand;
399:   bool HasVL;
400:   PolicyScheme Scheme;
401:   bool SupportOverloading;
402:   bool HasBuiltinAlias;
403:   std::string ManualCodegen;
404:   RVVTypePtr OutputType; // Builtin output type
405:   RVVTypes InputTypes;   // Builtin input types
406:   // The types we use to obtain the specific LLVM intrinsic. They are index of
407:   // InputTypes. -1 means the return type.
408:   std::vector<int64_t> IntrinsicTypes;
409:   unsigned NF = 1;
410:   Policy PolicyAttrs;
411:   unsigned TWiden = 0;
412: 
413: public:
414:   RVVIntrinsic(llvm::StringRef Name, llvm::StringRef Suffix,
415:                llvm::StringRef OverloadedName, llvm::StringRef OverloadedSuffix,
416:                llvm::StringRef IRName, bool IsMasked, bool HasMaskedOffOperand,
417:                bool HasVL, PolicyScheme Scheme, bool SupportOverloading,
418:                bool HasBuiltinAlias, llvm::StringRef ManualCodegen,
419:                const RVVTypes &Types,
420:                const std::vector<int64_t> &IntrinsicTypes, unsigned NF,
421:                Policy PolicyAttrs, bool HasFRMRoundModeOp, unsigned TWiden,
422:                bool AltFmt);
423:   ~RVVIntrinsic() = default;
```
- EN: Key type declarations here include `RVVIntrinsic`. It introduces enum-based state or option sets such as `PolicyScheme`. It exposes API surface such as `operator<<`, `~RVVIntrinsic`.
- 中文: 这里的重要类型声明包括 `RVVIntrinsic`。 它引入了 `PolicyScheme` 等基于枚举的状态或选项集合。 它暴露了 `operator<<`, `~RVVIntrinsic` 等接口。

### Lines 424-470

```cpp
424: 
425:   RVVTypePtr getOutputType() const { return OutputType; }
426:   const RVVTypes &getInputTypes() const { return InputTypes; }
427:   llvm::StringRef getBuiltinName() const { return BuiltinName; }
428:   bool hasMaskedOffOperand() const { return HasMaskedOffOperand; }
429:   bool hasVL() const { return HasVL; }
430:   bool hasPolicy() const { return Scheme != PolicyScheme::SchemeNone; }
431:   bool hasPassthruOperand() const {
432:     return Scheme == PolicyScheme::HasPassthruOperand;
433:   }
434:   bool hasPolicyOperand() const {
435:     return Scheme == PolicyScheme::HasPolicyOperand;
436:   }
437:   bool supportOverloading() const { return SupportOverloading; }
438:   bool hasBuiltinAlias() const { return HasBuiltinAlias; }
439:   bool hasManualCodegen() const { return !ManualCodegen.empty(); }
440:   bool isMasked() const { return IsMasked; }
441:   llvm::StringRef getOverloadedName() const { return OverloadedName; }
442:   llvm::StringRef getIRName() const { return IRName; }
443:   llvm::StringRef getManualCodegen() const { return ManualCodegen; }
444:   PolicyScheme getPolicyScheme() const { return Scheme; }
445:   unsigned getNF() const { return NF; }
446:   unsigned getTWiden() const { return TWiden; }
447:   const std::vector<int64_t> &getIntrinsicTypes() const {
448:     return IntrinsicTypes;
449:   }
450:   Policy getPolicyAttrs() const {
451:     return PolicyAttrs;
452:   }
453:   unsigned getPolicyAttrsBits() const {
454:     // CGBuiltin.cpp
455:     // The 0th bit simulates the `vta` of RVV
456:     // The 1st bit simulates the `vma` of RVV
457:     // int PolicyAttrs = 0;
458: 
459:     if (PolicyAttrs.isTUMAPolicy())
460:       return 2;
461:     if (PolicyAttrs.isTAMAPolicy())
462:       return 3;
463:     if (PolicyAttrs.isTUMUPolicy())
464:       return 0;
465:     if (PolicyAttrs.isTAMUPolicy())
466:       return 1;
467: 
468:     llvm_unreachable("unsupport policy");
469:     return 0;
470:   }
```
- EN: It exposes API surface such as `getOutputType`, `getInputTypes`, `getBuiltinName`, `hasMaskedOffOperand`.
- 中文: 它暴露了 `getOutputType`, `getInputTypes`, `getBuiltinName`, `hasMaskedOffOperand` 等接口。

### Lines 471-517

```cpp
471: 
472:   // Return the type string for a BUILTIN() macro in Builtins.def.
473:   std::string getBuiltinTypeStr() const;
474: 
475:   static std::string
476:   getSuffixStr(RVVTypeCache &TypeCache, BasicType Type, int Log2LMUL,
477:                llvm::ArrayRef<PrototypeDescriptor> PrototypeDescriptors);
478: 
479:   static llvm::SmallVector<PrototypeDescriptor>
480:   computeBuiltinTypes(llvm::ArrayRef<PrototypeDescriptor> Prototype,
481:                       bool IsMasked, bool HasMaskedOffOperand, bool HasVL,
482:                       unsigned NF, PolicyScheme DefaultScheme,
483:                       Policy PolicyAttrs, bool IsTuple);
484: 
485:   static llvm::SmallVector<Policy> getSupportedUnMaskedPolicies();
486:   static llvm::SmallVector<Policy>
487:       getSupportedMaskedPolicies(bool HasTailPolicy, bool HasMaskPolicy);
488: 
489:   static void updateNamesAndPolicy(bool IsMasked, bool HasPolicy,
490:                                    std::string &Name, std::string &BuiltinName,
491:                                    std::string &OverloadedName,
492:                                    Policy &PolicyAttrs, bool HasFRMRoundModeOp,
493:                                    bool AltFmt);
494: };
495: 
496: // Raw RVV intrinsic info, used to expand later.
497: // This struct is highly compact for minimized code size.
498: struct RVVIntrinsicRecord {
499:   // Intrinsic name, e.g. vadd_vv
500:   const char *Name;
501: 
502:   // Overloaded intrinsic name, could be empty if it can be computed from Name.
503:   // e.g. vadd
504:   const char *OverloadedName;
505: 
506:   // Required target features for this intrinsic.
507:   const char *RequiredExtensions;
508: 
509:   // Prototype for this intrinsic, index of RVVSignatureTable.
510:   uint16_t PrototypeIndex;
511: 
512:   // Suffix of intrinsic name, index of RVVSignatureTable.
513:   uint16_t SuffixIndex;
514: 
515:   // Suffix of overloaded intrinsic name, index of RVVSignatureTable.
516:   uint16_t OverloadedSuffixIndex;
517: 
```
- EN: Key type declarations here include `RVVIntrinsicRecord`. It exposes API surface such as `getBuiltinTypeStr`, `getSupportedUnMaskedPolicies`, `getSupportedMaskedPolicies`.
- 中文: 这里的重要类型声明包括 `RVVIntrinsicRecord`。 它暴露了 `getBuiltinTypeStr`, `getSupportedUnMaskedPolicies`, `getSupportedMaskedPolicies` 等接口。

### Lines 518-558

```cpp
518:   // Length of the prototype.
519:   uint8_t PrototypeLength;
520: 
521:   // Length of intrinsic name suffix.
522:   uint8_t SuffixLength;
523: 
524:   // Length of overloaded intrinsic suffix.
525:   uint8_t OverloadedSuffixSize;
526: 
527:   // Supported type, mask of BasicType.
528:   uint16_t TypeRangeMask;
529: 
530:   // Supported LMUL.
531:   uint8_t Log2LMULMask;
532: 
533:   // Number of fields, greater than 1 if it's segment load/store.
534:   uint8_t NF;
535: 
536:   bool HasMasked : 1;
537:   bool HasVL : 1;
538:   bool HasMaskedOffOperand : 1;
539:   bool HasTailPolicy : 1;
540:   bool HasMaskPolicy : 1;
541:   bool HasFRMRoundModeOp : 1;
542:   bool AltFmt : 1;
543:   bool IsTuple : 1;
544:   LLVM_PREFERRED_TYPE(PolicyScheme)
545:   uint8_t UnMaskedPolicyScheme : 2;
546:   LLVM_PREFERRED_TYPE(PolicyScheme)
547:   uint8_t MaskedPolicyScheme : 2;
548: };
549: 
550: llvm::raw_ostream &operator<<(llvm::raw_ostream &OS,
551:                               const RVVIntrinsicRecord &RVVInstrRecord);
552: 
553: LLVM_ENABLE_BITMASK_ENUMS_IN_NAMESPACE();
554: } // end namespace RISCV
555: 
556: } // end namespace clang
557: 
558: #endif // CLANG_SUPPORT_RISCVVINTRINSICUTILS_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `RISCV`, `clang`. It exposes API surface such as `LLVM_ENABLE_BITMASK_ENUMS_IN_NAMESPACE`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `RISCV`, `clang` 的命名空间作用域。 它暴露了 `LLVM_ENABLE_BITMASK_ENUMS_IN_NAMESPACE` 等接口。

## Key Concepts / 关键概念

- `raw_ostream`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `VScaleVal`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。
- `VectorTypeModifier`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `BaseTypeModifier`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `TypeModifier`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Policy`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `PolicyType`: An enumeration that captures a bounded set of modes, states, or categories. / 用于表达有限模式、状态或类别的枚举。
- `PrototypeDescriptor`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/ADT/ArrayRef.h`, `llvm/ADT/BitmaskEnum.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `cstdint`, `optional`, `set`, `string`, `unordered_map`, `vector`
- Forward declarations / 前向声明: `raw_ostream`, `RVVType`, `RVVTypeCache`
- Namespace context / 命名空间上下文: `llvm`, `clang`, `RISCV`
- Macro-style dependencies / 宏式依赖: `LLVM_MARK_AS_BITMASK_ENUM`, `VTM`, `TM`, `LLVM_PREFERRED_TYPE`, `LLVM_ENABLE_BITMASK_ENUMS_IN_NAMESPACE`
