# LoongArch.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/LoongArch.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for LoongArch.
- **Purpose (CN) / 目的（中文）**: 实现 LoongArch 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: //===- LoongArch.cpp ------------------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: #include "ABIInfoImpl.h"
10: #include "TargetInfo.h"
11: 
12: using namespace clang;
13: using namespace clang::CodeGen;
14: 
15: // LoongArch ABI Implementation. Documented at
16: // https://loongson.github.io/LoongArch-Documentation/LoongArch-ELF-ABI-EN.html
```
- **EN**: This block imports local CodeGen headers `ABIInfoImpl.h`, `TargetInfo.h`; opens or references namespaces `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfoImpl.h`, `TargetInfo.h`；打开或引用命名空间 `clang`；包含影响本编译单元构建方式的预处理结构。

### Lines 17-32
```cpp
17: //
18: //===----------------------------------------------------------------------===//
19: 
20: namespace {
21: class LoongArchABIInfo : public DefaultABIInfo {
22: private:
23:   // Size of the integer ('r') registers in bits.
24:   unsigned GRLen;
25:   // Size of the floating point ('f') registers in bits.
26:   unsigned FRLen;
27:   // Number of general-purpose argument registers.
28:   static const int NumGARs = 8;
29:   // Number of floating-point argument registers.
30:   static const int NumFARs = 8;
31:   bool detectFARsEligibleStructHelper(QualType Ty, CharUnits CurOff,
32:                                       llvm::Type *&Field1Ty,
```
- **EN**: This block introduces declarations such as `LoongArchABIInfo`.
- **CN**: 该代码块给出诸如 `LoongArchABIInfo` 的声明。

### Lines 33-48
```cpp
33:                                       CharUnits &Field1Off,
34:                                       llvm::Type *&Field2Ty,
35:                                       CharUnits &Field2Off) const;
36: 
37: public:
38:   LoongArchABIInfo(CodeGen::CodeGenTypes &CGT, unsigned GRLen, unsigned FRLen)
39:       : DefaultABIInfo(CGT), GRLen(GRLen), FRLen(FRLen) {}
40: 
41:   void computeInfo(CGFunctionInfo &FI) const override;
42: 
43:   ABIArgInfo classifyArgumentType(QualType Ty, bool IsFixed, int &GARsLeft,
44:                                   int &FARsLeft) const;
45:   ABIArgInfo classifyReturnType(QualType RetTy) const;
46: 
47:   RValue EmitVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
48:                    AggValueSlot Slot) const override;
```
- **EN**: This block defines callable entry points like `LoongArchABIInfo`, `computeInfo`, `classifyArgumentType`, `classifyReturnType`, `EmitVAArg`.
- **CN**: 该代码块定义可调用入口，例如 `LoongArchABIInfo`, `computeInfo`, `classifyArgumentType`, `classifyReturnType`, `EmitVAArg`。

### Lines 49-64
```cpp
49: 
50:   ABIArgInfo extendType(QualType Ty) const;
51: 
52:   bool detectFARsEligibleStruct(QualType Ty, llvm::Type *&Field1Ty,
53:                                 CharUnits &Field1Off, llvm::Type *&Field2Ty,
54:                                 CharUnits &Field2Off, int &NeededArgGPRs,
55:                                 int &NeededArgFPRs) const;
56:   ABIArgInfo coerceAndExpandFARsEligibleStruct(llvm::Type *Field1Ty,
57:                                                CharUnits Field1Off,
58:                                                llvm::Type *Field2Ty,
59:                                                CharUnits Field2Off) const;
60: };
61: } // end anonymous namespace
62: 
63: void LoongArchABIInfo::computeInfo(CGFunctionInfo &FI) const {
64:   QualType RetTy = FI.getReturnType();
```
- **EN**: This block opens or references namespaces `void`; defines callable entry points like `extendType`, `detectFARsEligibleStruct`, `coerceAndExpandFARsEligibleStruct`, `computeInfo`.
- **CN**: 该代码块打开或引用命名空间 `void`；定义可调用入口，例如 `extendType`, `detectFARsEligibleStruct`, `coerceAndExpandFARsEligibleStruct`, `computeInfo`。

### Lines 65-80
```cpp
65:   if (!getCXXABI().classifyReturnType(FI))
66:     FI.getReturnInfo() = classifyReturnType(RetTy);
67: 
68:   // IsRetIndirect is true if classifyArgumentType indicated the value should
69:   // be passed indirect, or if the type size is a scalar greater than 2*GRLen
70:   // and not a complex type with elements <= FRLen. e.g. fp128 is passed direct
71:   // in LLVM IR, relying on the backend lowering code to rewrite the argument
72:   // list and pass indirectly on LA32.
73:   bool IsRetIndirect = FI.getReturnInfo().getKind() == ABIArgInfo::Indirect;
74:   if (!IsRetIndirect && RetTy->isScalarType() &&
75:       getContext().getTypeSize(RetTy) > (2 * GRLen)) {
76:     if (RetTy->isComplexType() && FRLen) {
77:       QualType EltTy = RetTy->castAs<ComplexType>()->getElementType();
78:       IsRetIndirect = getContext().getTypeSize(EltTy) > FRLen;
79:     } else {
80:       // This is a normal scalar > 2*GRLen, such as fp128 on LA32.
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 81-96
```cpp
81:       IsRetIndirect = true;
82:     }
83:   }
84: 
85:   // We must track the number of GARs and FARs used in order to conform to the
86:   // LoongArch ABI. As GAR usage is different for variadic arguments, we must
87:   // also track whether we are examining a vararg or not.
88:   int GARsLeft = IsRetIndirect ? NumGARs - 1 : NumGARs;
89:   int FARsLeft = FRLen ? NumFARs : 0;
90:   int NumFixedArgs = FI.getNumRequiredArgs();
91: 
92:   int ArgNum = 0;
93:   for (auto &ArgInfo : FI.arguments()) {
94:     ArgInfo.info = classifyArgumentType(
95:         ArgInfo.type, /*IsFixed=*/ArgNum < NumFixedArgs, GARsLeft, FARsLeft);
96:     ArgNum++;
```
- **EN**: This block uses control flow (for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 97-112
```cpp
 97:   }
 98: }
 99: 
100: // Returns true if the struct is a potential candidate to be passed in FARs (and
101: // GARs). If this function returns true, the caller is responsible for checking
102: // that if there is only a single field then that field is a float.
103: bool LoongArchABIInfo::detectFARsEligibleStructHelper(
104:     QualType Ty, CharUnits CurOff, llvm::Type *&Field1Ty, CharUnits &Field1Off,
105:     llvm::Type *&Field2Ty, CharUnits &Field2Off) const {
106:   bool IsInt = Ty->isIntegralOrEnumerationType();
107:   bool IsFloat = Ty->isRealFloatingType();
108: 
109:   if (IsInt || IsFloat) {
110:     uint64_t Size = getContext().getTypeSize(Ty);
111:     if (IsInt && Size > GRLen)
112:       return false;
```
- **EN**: This block defines callable entry points like `detectFARsEligibleStructHelper`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `detectFARsEligibleStructHelper`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 113-128
```cpp
113:     // Can't be eligible if larger than the FP registers. Handling of half
114:     // precision values has been specified in the ABI, so don't block those.
115:     if (IsFloat && Size > FRLen)
116:       return false;
117:     // Can't be eligible if an integer type was already found (int+int pairs
118:     // are not eligible).
119:     if (IsInt && Field1Ty && Field1Ty->isIntegerTy())
120:       return false;
121:     if (!Field1Ty) {
122:       Field1Ty = CGT.ConvertType(Ty);
123:       Field1Off = CurOff;
124:       return true;
125:     }
126:     if (!Field2Ty) {
127:       Field2Ty = CGT.ConvertType(Ty);
128:       Field2Off = CurOff;
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 129-144
```cpp
129:       return true;
130:     }
131:     return false;
132:   }
133: 
134:   if (auto CTy = Ty->getAs<ComplexType>()) {
135:     if (Field1Ty)
136:       return false;
137:     QualType EltTy = CTy->getElementType();
138:     if (getContext().getTypeSize(EltTy) > FRLen)
139:       return false;
140:     Field1Ty = CGT.ConvertType(EltTy);
141:     Field1Off = CurOff;
142:     Field2Ty = Field1Ty;
143:     Field2Off = Field1Off + getContext().getTypeSizeInChars(EltTy);
144:     return true;
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 145-160
```cpp
145:   }
146: 
147:   if (const ConstantArrayType *ATy = getContext().getAsConstantArrayType(Ty)) {
148:     uint64_t ArraySize = ATy->getZExtSize();
149:     QualType EltTy = ATy->getElementType();
150:     // Non-zero-length arrays of empty records make the struct ineligible to be
151:     // passed via FARs in C++.
152:     if (const auto *RTy = EltTy->getAsCanonical<RecordType>()) {
153:       if (ArraySize != 0 && isa<CXXRecordDecl>(RTy->getDecl()) &&
154:           isEmptyRecord(getContext(), EltTy, true, true))
155:         return false;
156:     }
157:     CharUnits EltSize = getContext().getTypeSizeInChars(EltTy);
158:     for (uint64_t i = 0; i < ArraySize; ++i) {
159:       if (!detectFARsEligibleStructHelper(EltTy, CurOff, Field1Ty, Field1Off,
160:                                           Field2Ty, Field2Off))
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 161-176
```cpp
161:         return false;
162:       CurOff += EltSize;
163:     }
164:     return true;
165:   }
166: 
167:   if (const auto *RTy = Ty->getAsCanonical<RecordType>()) {
168:     // Structures with either a non-trivial destructor or a non-trivial
169:     // copy constructor are not eligible for the FP calling convention.
170:     if (getRecordArgABI(Ty, CGT.getCXXABI()))
171:       return false;
172:     const RecordDecl *RD = RTy->getDecl()->getDefinitionOrSelf();
173:     if (isEmptyRecord(getContext(), Ty, true, true) &&
174:         (!RD->isUnion() || !isa<CXXRecordDecl>(RD)))
175:       return true;
176:     // Unions aren't eligible unless they're empty in C (which is caught above).
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 177-192
```cpp
177:     if (RD->isUnion())
178:       return false;
179:     const ASTRecordLayout &Layout = getContext().getASTRecordLayout(RD);
180:     // If this is a C++ record, check the bases first.
181:     if (const CXXRecordDecl *CXXRD = dyn_cast<CXXRecordDecl>(RD)) {
182:       for (const CXXBaseSpecifier &B : CXXRD->bases()) {
183:         const auto *BDecl = B.getType()->castAsCXXRecordDecl();
184:         if (!detectFARsEligibleStructHelper(
185:                 B.getType(), CurOff + Layout.getBaseClassOffset(BDecl),
186:                 Field1Ty, Field1Off, Field2Ty, Field2Off))
187:           return false;
188:       }
189:     }
190:     for (const FieldDecl *FD : RD->fields()) {
191:       QualType QTy = FD->getType();
192:       if (FD->isBitField()) {
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 193-208
```cpp
193:         unsigned BitWidth = FD->getBitWidthValue();
194:         // Zero-width bitfields are ignored.
195:         if (BitWidth == 0)
196:           continue;
197:         // Allow a bitfield with a type greater than GRLen as long as the
198:         // bitwidth is GRLen or less.
199:         if (getContext().getTypeSize(QTy) > GRLen && BitWidth <= GRLen) {
200:           QTy = getContext().getIntTypeForBitwidth(GRLen, false);
201:         }
202:       }
203: 
204:       if (!detectFARsEligibleStructHelper(
205:               QTy,
206:               CurOff + getContext().toCharUnitsFromBits(
207:                            Layout.getFieldOffset(FD->getFieldIndex())),
208:               Field1Ty, Field1Off, Field2Ty, Field2Off))
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 209-224
```cpp
209:         return false;
210:     }
211:     return Field1Ty != nullptr;
212:   }
213: 
214:   return false;
215: }
216: 
217: // Determine if a struct is eligible to be passed in FARs (and GARs) (i.e., when
218: // flattened it contains a single fp value, fp+fp, or int+fp of appropriate
219: // size). If so, NeededFARs and NeededGARs are incremented appropriately.
220: bool LoongArchABIInfo::detectFARsEligibleStruct(
221:     QualType Ty, llvm::Type *&Field1Ty, CharUnits &Field1Off,
222:     llvm::Type *&Field2Ty, CharUnits &Field2Off, int &NeededGARs,
223:     int &NeededFARs) const {
224:   Field1Ty = nullptr;
```
- **EN**: This block defines callable entry points like `detectFARsEligibleStruct`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `detectFARsEligibleStruct`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 225-240
```cpp
225:   Field2Ty = nullptr;
226:   NeededGARs = 0;
227:   NeededFARs = 0;
228:   if (!detectFARsEligibleStructHelper(Ty, CharUnits::Zero(), Field1Ty,
229:                                       Field1Off, Field2Ty, Field2Off))
230:     return false;
231:   if (!Field1Ty)
232:     return false;
233:   // Not really a candidate if we have a single int but no float.
234:   if (Field1Ty && !Field2Ty && !Field1Ty->isFloatingPointTy())
235:     return false;
236:   if (Field1Ty && Field1Ty->isFloatingPointTy())
237:     NeededFARs++;
238:   else if (Field1Ty)
239:     NeededGARs++;
240:   if (Field2Ty && Field2Ty->isFloatingPointTy())
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 241-256
```cpp
241:     NeededFARs++;
242:   else if (Field2Ty)
243:     NeededGARs++;
244:   return true;
245: }
246: 
247: // Call getCoerceAndExpand for the two-element flattened struct described by
248: // Field1Ty, Field1Off, Field2Ty, Field2Off. This method will create an
249: // appropriate coerceToType and unpaddedCoerceToType.
250: ABIArgInfo LoongArchABIInfo::coerceAndExpandFARsEligibleStruct(
251:     llvm::Type *Field1Ty, CharUnits Field1Off, llvm::Type *Field2Ty,
252:     CharUnits Field2Off) const {
253:   SmallVector<llvm::Type *, 3> CoerceElts;
254:   SmallVector<llvm::Type *, 2> UnpaddedCoerceElts;
255:   if (!Field1Off.isZero())
256:     CoerceElts.push_back(llvm::ArrayType::get(
```
- **EN**: This block defines callable entry points like `coerceAndExpandFARsEligibleStruct`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `coerceAndExpandFARsEligibleStruct`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 257-272
```cpp
257:         llvm::Type::getInt8Ty(getVMContext()), Field1Off.getQuantity()));
258: 
259:   CoerceElts.push_back(Field1Ty);
260:   UnpaddedCoerceElts.push_back(Field1Ty);
261: 
262:   if (!Field2Ty) {
263:     return ABIArgInfo::getCoerceAndExpand(
264:         llvm::StructType::get(getVMContext(), CoerceElts, !Field1Off.isZero()),
265:         UnpaddedCoerceElts[0]);
266:   }
267: 
268:   CharUnits Field2Align =
269:       CharUnits::fromQuantity(getDataLayout().getABITypeAlign(Field2Ty));
270:   CharUnits Field1End =
271:       Field1Off +
272:       CharUnits::fromQuantity(getDataLayout().getTypeStoreSize(Field1Ty));
```
- **EN**: This block defines callable entry points like `getInt8Ty`, `getCoerceAndExpand`, `fromQuantity`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getInt8Ty`, `getCoerceAndExpand`, `fromQuantity`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 273-288
```cpp
273:   CharUnits Field2OffNoPadNoPack = Field1End.alignTo(Field2Align);
274: 
275:   CharUnits Padding = CharUnits::Zero();
276:   if (Field2Off > Field2OffNoPadNoPack)
277:     Padding = Field2Off - Field2OffNoPadNoPack;
278:   else if (Field2Off != Field2Align && Field2Off > Field1End)
279:     Padding = Field2Off - Field1End;
280: 
281:   bool IsPacked = !Field2Off.isMultipleOf(Field2Align);
282: 
283:   if (!Padding.isZero())
284:     CoerceElts.push_back(llvm::ArrayType::get(
285:         llvm::Type::getInt8Ty(getVMContext()), Padding.getQuantity()));
286: 
287:   CoerceElts.push_back(Field2Ty);
288:   UnpaddedCoerceElts.push_back(Field2Ty);
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 289-304
```cpp
289: 
290:   return ABIArgInfo::getCoerceAndExpand(
291:       llvm::StructType::get(getVMContext(), CoerceElts, IsPacked),
292:       llvm::StructType::get(getVMContext(), UnpaddedCoerceElts, IsPacked));
293: }
294: 
295: ABIArgInfo LoongArchABIInfo::classifyArgumentType(QualType Ty, bool IsFixed,
296:                                                   int &GARsLeft,
297:                                                   int &FARsLeft) const {
298:   assert(GARsLeft <= NumGARs && "GAR tracking underflow");
299:   Ty = useFirstFieldIfTransparentUnion(Ty);
300: 
301:   // Structures with either a non-trivial destructor or a non-trivial
302:   // copy constructor are always passed indirectly.
303:   if (CGCXXABI::RecordArgABI RAA = getRecordArgABI(Ty, getCXXABI())) {
304:     if (GARsLeft)
```
- **EN**: This block defines callable entry points like `getCoerceAndExpand`, `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getCoerceAndExpand`, `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 305-320
```cpp
305:       GARsLeft -= 1;
306:     return getNaturalAlignIndirect(
307:         Ty, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
308:         /*ByVal=*/RAA == CGCXXABI::RAA_DirectInMemory);
309:   }
310: 
311:   uint64_t Size = getContext().getTypeSize(Ty);
312: 
313:   // Ignore empty struct or union whose size is zero, e.g. `struct { }` in C or
314:   // `struct { int a[0]; }` in C++. In C++, `struct { }` is empty but it's size
315:   // is 1 byte and g++ doesn't ignore it; clang++ matches this behaviour.
316:   if (isEmptyRecord(getContext(), Ty, true) && Size == 0)
317:     return ABIArgInfo::getIgnore();
318: 
319:   // Pass floating point values via FARs if possible.
320:   if (IsFixed && Ty->isFloatingType() && !Ty->isComplexType() &&
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 321-336
```cpp
321:       FRLen >= Size && FARsLeft) {
322:     FARsLeft--;
323:     return ABIArgInfo::getDirect();
324:   }
325: 
326:   // Complex types for the *f or *d ABI must be passed directly rather than
327:   // using CoerceAndExpand.
328:   if (IsFixed && Ty->isComplexType() && FRLen && FARsLeft >= 2) {
329:     QualType EltTy = Ty->castAs<ComplexType>()->getElementType();
330:     if (getContext().getTypeSize(EltTy) <= FRLen) {
331:       FARsLeft -= 2;
332:       return ABIArgInfo::getDirect();
333:     }
334:   }
335: 
336:   if (IsFixed && FRLen && Ty->isStructureOrClassType()) {
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 337-352
```cpp
337:     llvm::Type *Field1Ty = nullptr;
338:     llvm::Type *Field2Ty = nullptr;
339:     CharUnits Field1Off = CharUnits::Zero();
340:     CharUnits Field2Off = CharUnits::Zero();
341:     int NeededGARs = 0;
342:     int NeededFARs = 0;
343:     bool IsCandidate = detectFARsEligibleStruct(
344:         Ty, Field1Ty, Field1Off, Field2Ty, Field2Off, NeededGARs, NeededFARs);
345:     if (IsCandidate && NeededGARs <= GARsLeft && NeededFARs <= FARsLeft) {
346:       GARsLeft -= NeededGARs;
347:       FARsLeft -= NeededFARs;
348:       return coerceAndExpandFARsEligibleStruct(Field1Ty, Field1Off, Field2Ty,
349:                                                Field2Off);
350:     }
351:   }
352: 
```
- **EN**: This block defines callable entry points like `coerceAndExpandFARsEligibleStruct`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `coerceAndExpandFARsEligibleStruct`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 353-368
```cpp
353:   uint64_t NeededAlign = getContext().getTypeAlign(Ty);
354:   // Determine the number of GARs needed to pass the current argument
355:   // according to the ABI. 2*GRLen-aligned varargs are passed in "aligned"
356:   // register pairs, so may consume 3 registers.
357:   int NeededGARs = 1;
358:   if (!IsFixed && NeededAlign == 2 * GRLen)
359:     NeededGARs = 2 + (GARsLeft % 2);
360:   else if (Size > GRLen && Size <= 2 * GRLen)
361:     NeededGARs = 2;
362: 
363:   if (NeededGARs > GARsLeft)
364:     NeededGARs = GARsLeft;
365: 
366:   GARsLeft -= NeededGARs;
367: 
368:   if (!isAggregateTypeForABI(Ty) && !Ty->isVectorType()) {
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 369-384
```cpp
369:     // Treat an enum type as its underlying type.
370:     if (const auto *ED = Ty->getAsEnumDecl())
371:       Ty = ED->getIntegerType();
372: 
373:     // All integral types are promoted to GRLen width.
374:     if (Size < GRLen && Ty->isIntegralOrEnumerationType())
375:       return extendType(Ty);
376: 
377:     if (const auto *EIT = Ty->getAs<BitIntType>()) {
378:       if (EIT->getNumBits() < GRLen)
379:         return extendType(Ty);
380:       if (EIT->getNumBits() > 128 ||
381:           (!getContext().getTargetInfo().hasInt128Type() &&
382:            EIT->getNumBits() > 64))
383:         return getNaturalAlignIndirect(
384:             Ty, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 385-400
```cpp
385:             /*ByVal=*/false);
386:     }
387: 
388:     return ABIArgInfo::getDirect();
389:   }
390: 
391:   // Aggregates which are <= 2*GRLen will be passed in registers if possible,
392:   // so coerce to integers.
393:   if (Size <= 2 * GRLen) {
394:     // Use a single GRLen int if possible, 2*GRLen if 2*GRLen alignment is
395:     // required, and a 2-element GRLen array if only GRLen alignment is
396:     // required.
397:     if (Size <= GRLen) {
398:       return ABIArgInfo::getDirect(
399:           llvm::IntegerType::get(getVMContext(), GRLen));
400:     }
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 401-416
```cpp
401:     if (getContext().getTypeAlign(Ty) == 2 * GRLen) {
402:       return ABIArgInfo::getDirect(
403:           llvm::IntegerType::get(getVMContext(), 2 * GRLen));
404:     }
405:     return ABIArgInfo::getDirect(
406:         llvm::ArrayType::get(llvm::IntegerType::get(getVMContext(), GRLen), 2));
407:   }
408:   return getNaturalAlignIndirect(
409:       Ty, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
410:       /*ByVal=*/false);
411: }
412: 
413: ABIArgInfo LoongArchABIInfo::classifyReturnType(QualType RetTy) const {
414:   if (RetTy->isVoidType())
415:     return ABIArgInfo::getIgnore();
416:   // The rules for return and argument types are the same, so defer to
```
- **EN**: This block defines callable entry points like `getDirect`, `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 417-432
```cpp
417:   // classifyArgumentType.
418:   int GARsLeft = 2;
419:   int FARsLeft = FRLen ? 2 : 0;
420:   return classifyArgumentType(RetTy, /*IsFixed=*/true, GARsLeft, FARsLeft);
421: }
422: 
423: RValue LoongArchABIInfo::EmitVAArg(CodeGenFunction &CGF, Address VAListAddr,
424:                                    QualType Ty, AggValueSlot Slot) const {
425:   CharUnits SlotSize = CharUnits::fromQuantity(GRLen / 8);
426: 
427:   // Empty records are ignored for parameter passing purposes.
428:   if (isEmptyRecord(getContext(), Ty, true))
429:     return Slot.asRValue();
430: 
431:   auto TInfo = getContext().getTypeInfoInChars(Ty);
432: 
```
- **EN**: This block defines callable entry points like `classifyArgumentType`, `EmitVAArg`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyArgumentType`, `EmitVAArg`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 433-448
```cpp
433:   // Arguments bigger than 2*GRLen bytes are passed indirectly.
434:   return emitVoidPtrVAArg(CGF, VAListAddr, Ty,
435:                           /*IsIndirect=*/TInfo.Width > 2 * SlotSize, TInfo,
436:                           SlotSize,
437:                           /*AllowHigherAlign=*/true, Slot);
438: }
439: 
440: ABIArgInfo LoongArchABIInfo::extendType(QualType Ty) const {
441:   int TySize = getContext().getTypeSize(Ty);
442:   // LA64 ABI requires unsigned 32 bit integers to be sign extended.
443:   if (GRLen == 64 && Ty->isUnsignedIntegerOrEnumerationType() && TySize == 32)
444:     return ABIArgInfo::getSignExtend(Ty);
445:   return ABIArgInfo::getExtend(Ty);
446: }
447: 
448: namespace {
```
- **EN**: This block defines callable entry points like `extendType`, `getExtend`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `extendType`, `getExtend`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 449-463
```cpp
449: class LoongArchTargetCodeGenInfo : public TargetCodeGenInfo {
450: public:
451:   LoongArchTargetCodeGenInfo(CodeGen::CodeGenTypes &CGT, unsigned GRLen,
452:                              unsigned FRLen)
453:       : TargetCodeGenInfo(
454:             std::make_unique<LoongArchABIInfo>(CGT, GRLen, FRLen)) {}
455: };
456: } // namespace
457: 
458: std::unique_ptr<TargetCodeGenInfo>
459: CodeGen::createLoongArchTargetCodeGenInfo(CodeGenModule &CGM, unsigned GRLen,
460:                                           unsigned FLen) {
461:   return std::make_unique<LoongArchTargetCodeGenInfo>(CGM.getTypes(), GRLen,
462:                                                       FLen);
463: }
```
- **EN**: This block opens or references namespaces `std`; introduces declarations such as `LoongArchTargetCodeGenInfo`; defines callable entry points like `LoongArchTargetCodeGenInfo`, `createLoongArchTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块打开或引用命名空间 `std`；给出诸如 `LoongArchTargetCodeGenInfo` 的声明；定义可调用入口，例如 `LoongArchTargetCodeGenInfo`, `createLoongArchTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **Field1Ty**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CharUnits**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **GRLen**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Field2Ty**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getContext**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Field2Off**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **Field1Off**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `TargetInfo.h`
