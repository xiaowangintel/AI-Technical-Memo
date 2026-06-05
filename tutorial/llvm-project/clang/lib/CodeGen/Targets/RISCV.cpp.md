# RISCV.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/RISCV.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for RISCV.
- **Purpose (CN) / 目的（中文）**: 实现 RISCV 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: //===- RISCV.cpp ----------------------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: #include "ABIInfoImpl.h"
10: #include "TargetInfo.h"
11: #include "llvm/IR/IntrinsicsRISCV.h"
12: #include "llvm/TargetParser/RISCVTargetParser.h"
13: 
14: using namespace clang;
15: using namespace clang::CodeGen;
16: 
17: //===----------------------------------------------------------------------===//
18: // RISC-V ABI Implementation
19: //===----------------------------------------------------------------------===//
20: 
```
- **EN**: This block imports local CodeGen headers `ABIInfoImpl.h`, `TargetInfo.h`; LLVM headers `llvm/IR/IntrinsicsRISCV.h`, `llvm/TargetParser/RISCVTargetParser.h`; opens or references namespaces `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfoImpl.h`, `TargetInfo.h`；LLVM 头文件 `llvm/IR/IntrinsicsRISCV.h`, `llvm/TargetParser/RISCVTargetParser.h`；打开或引用命名空间 `clang`；包含影响本编译单元构建方式的预处理结构。

### Lines 21-40
```cpp
21: namespace {
22: class RISCVABIInfo : public DefaultABIInfo {
23: private:
24:   // Size of the integer ('x') registers in bits.
25:   unsigned XLen;
26:   // Size of the floating point ('f') registers in bits. Note that the target
27:   // ISA might have a wider FLen than the selected ABI (e.g. an RV32IF target
28:   // with soft float ABI has FLen==0).
29:   unsigned FLen;
30:   const int NumArgGPRs;
31:   const int NumArgFPRs;
32:   const bool EABI;
33:   bool detectFPCCEligibleStructHelper(QualType Ty, CharUnits CurOff,
34:                                       llvm::Type *&Field1Ty,
35:                                       CharUnits &Field1Off,
36:                                       llvm::Type *&Field2Ty,
37:                                       CharUnits &Field2Off) const;
38: 
39:   bool detectVLSCCEligibleStruct(QualType Ty, unsigned ABIVLen,
40:                                  llvm::Type *&VLSType) const;
```
- **EN**: This block introduces declarations such as `RISCVABIInfo`; defines callable entry points like `detectFPCCEligibleStructHelper`, `detectVLSCCEligibleStruct`.
- **CN**: 该代码块给出诸如 `RISCVABIInfo` 的声明；定义可调用入口，例如 `detectFPCCEligibleStructHelper`, `detectVLSCCEligibleStruct`。

### Lines 41-60
```cpp
41: 
42: public:
43:   RISCVABIInfo(CodeGen::CodeGenTypes &CGT, unsigned XLen, unsigned FLen,
44:                bool EABI)
45:       : DefaultABIInfo(CGT), XLen(XLen), FLen(FLen), NumArgGPRs(EABI ? 6 : 8),
46:         NumArgFPRs(FLen != 0 ? 8 : 0), EABI(EABI) {}
47: 
48:   // DefaultABIInfo's classifyReturnType and classifyArgumentType are
49:   // non-virtual, but computeInfo is virtual, so we overload it.
50:   void computeInfo(CGFunctionInfo &FI) const override;
51: 
52:   ABIArgInfo classifyArgumentType(QualType Ty, bool IsFixed, int &ArgGPRsLeft,
53:                                   int &ArgFPRsLeft, unsigned ABIVLen) const;
54:   ABIArgInfo classifyReturnType(QualType RetTy, unsigned ABIVLen) const;
55: 
56:   RValue EmitVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
57:                    AggValueSlot Slot) const override;
58: 
59:   ABIArgInfo extendType(QualType Ty, llvm::Type *CoerceTy = nullptr) const;
60: 
```
- **EN**: This block defines callable entry points like `RISCVABIInfo`, `computeInfo`, `classifyArgumentType`, `classifyReturnType`, `EmitVAArg`.
- **CN**: 该代码块定义可调用入口，例如 `RISCVABIInfo`, `computeInfo`, `classifyArgumentType`, `classifyReturnType`, `EmitVAArg`。

### Lines 61-80
```cpp
61:   bool detectFPCCEligibleStruct(QualType Ty, llvm::Type *&Field1Ty,
62:                                 CharUnits &Field1Off, llvm::Type *&Field2Ty,
63:                                 CharUnits &Field2Off, int &NeededArgGPRs,
64:                                 int &NeededArgFPRs) const;
65:   ABIArgInfo coerceAndExpandFPCCEligibleStruct(llvm::Type *Field1Ty,
66:                                                CharUnits Field1Off,
67:                                                llvm::Type *Field2Ty,
68:                                                CharUnits Field2Off) const;
69: 
70:   ABIArgInfo coerceVLSVector(QualType Ty, unsigned ABIVLen = 0) const;
71: 
72:   using ABIInfo::appendAttributeMangling;
73:   void appendAttributeMangling(TargetClonesAttr *Attr, unsigned Index,
74:                                raw_ostream &Out) const override;
75:   void appendAttributeMangling(StringRef AttrStr,
76:                                raw_ostream &Out) const override;
77:   llvm::Value *createCoercedLoad(Address SrcAddr, const ABIArgInfo &AI,
78:                                  CodeGenFunction &CGF) const override;
79:   void createCoercedStore(llvm::Value *Val, Address DstAddr,
80:                           const ABIArgInfo &AI, bool DestIsVolatile,
```
- **EN**: This block spells out callable entry points like `detectFPCCEligibleStruct`, `coerceAndExpandFPCCEligibleStruct`, `coerceVLSVector`, `appendAttributeMangling`.
- **CN**: 该代码块给出可调用入口的声明，例如 `detectFPCCEligibleStruct`, `coerceAndExpandFPCCEligibleStruct`, `coerceVLSVector`, `appendAttributeMangling`。

### Lines 81-100
```cpp
 81:                           CodeGenFunction &CGF) const override;
 82: };
 83: } // end anonymous namespace
 84: 
 85: void RISCVABIInfo::appendAttributeMangling(TargetClonesAttr *Attr,
 86:                                            unsigned Index,
 87:                                            raw_ostream &Out) const {
 88:   appendAttributeMangling(Attr->getFeatureStr(Index), Out);
 89: }
 90: 
 91: void RISCVABIInfo::appendAttributeMangling(StringRef AttrStr,
 92:                                            raw_ostream &Out) const {
 93:   if (AttrStr == "default") {
 94:     Out << ".default";
 95:     return;
 96:   }
 97: 
 98:   Out << '.';
 99: 
100:   SmallVector<StringRef, 8> Attrs;
```
- **EN**: This block opens or references namespaces `void`; defines callable entry points like `appendAttributeMangling`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块打开或引用命名空间 `void`；定义可调用入口，例如 `appendAttributeMangling`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 101-120
```cpp
101:   AttrStr.split(Attrs, ';');
102: 
103:   // Only consider the arch string.
104:   StringRef ArchStr;
105:   for (auto &Attr : Attrs) {
106:     if (Attr.starts_with("arch="))
107:       ArchStr = Attr;
108:   }
109: 
110:   // Extract features string.
111:   SmallVector<StringRef, 8> Features;
112:   ArchStr.consume_front("arch=");
113:   ArchStr.split(Features, ',');
114: 
115:   llvm::stable_sort(Features);
116: 
117:   for (auto Feat : Features) {
118:     Feat.consume_front("+");
119:     Out << "_" << Feat;
120:   }
```
- **EN**: This block defines callable entry points like `stable_sort`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `stable_sort`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 121-140
```cpp
121: }
122: 
123: void RISCVABIInfo::computeInfo(CGFunctionInfo &FI) const {
124:   unsigned ABIVLen;
125:   switch (FI.getExtInfo().getCC()) {
126:   default:
127:     ABIVLen = 0;
128:     break;
129: #define CC_VLS_CASE(ABI_VLEN)                                                  \
130:   case CallingConv::CC_RISCVVLSCall_##ABI_VLEN:                                \
131:     ABIVLen = ABI_VLEN;                                                        \
132:     break;
133:     CC_VLS_CASE(32)
134:     CC_VLS_CASE(64)
135:     CC_VLS_CASE(128)
136:     CC_VLS_CASE(256)
137:     CC_VLS_CASE(512)
138:     CC_VLS_CASE(1024)
139:     CC_VLS_CASE(2048)
140:     CC_VLS_CASE(4096)
```
- **EN**: This block defines callable entry points like `computeInfo`; uses control flow (switch, case) to specialize target-specific ABI and code generation; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块定义可调用入口，例如 `computeInfo`；通过控制流（switch, case）细化 目标相关的 ABI 与代码生成 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 141-160
```cpp
141:     CC_VLS_CASE(8192)
142:     CC_VLS_CASE(16384)
143:     CC_VLS_CASE(32768)
144:     CC_VLS_CASE(65536)
145: #undef CC_VLS_CASE
146:   }
147:   QualType RetTy = FI.getReturnType();
148:   if (!getCXXABI().classifyReturnType(FI))
149:     FI.getReturnInfo() = classifyReturnType(RetTy, ABIVLen);
150: 
151:   // IsRetIndirect is true if classifyArgumentType indicated the value should
152:   // be passed indirect, or if the type size is a scalar greater than 2*XLen
153:   // and not a complex type with elements <= FLen. e.g. fp128 is passed direct
154:   // in LLVM IR, relying on the backend lowering code to rewrite the argument
155:   // list and pass indirectly on RV32.
156:   bool IsRetIndirect = FI.getReturnInfo().getKind() == ABIArgInfo::Indirect;
157:   if (!IsRetIndirect && RetTy->isScalarType() &&
158:       getContext().getTypeSize(RetTy) > (2 * XLen)) {
159:     if (RetTy->isComplexType() && FLen) {
160:       QualType EltTy = RetTy->castAs<ComplexType>()->getElementType();
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 161-180
```cpp
161:       IsRetIndirect = getContext().getTypeSize(EltTy) > FLen;
162:     } else {
163:       // This is a normal scalar > 2*XLen, such as fp128 on RV32.
164:       IsRetIndirect = true;
165:     }
166:   }
167: 
168:   int ArgGPRsLeft = IsRetIndirect ? NumArgGPRs - 1 : NumArgGPRs;
169:   int ArgFPRsLeft = NumArgFPRs;
170:   int NumFixedArgs = FI.getNumRequiredArgs();
171: 
172:   int ArgNum = 0;
173:   for (auto &ArgInfo : FI.arguments()) {
174:     bool IsFixed = ArgNum < NumFixedArgs;
175:     ArgInfo.info = classifyArgumentType(ArgInfo.type, IsFixed, ArgGPRsLeft,
176:                                         ArgFPRsLeft, ABIVLen);
177:     ArgNum++;
178:   }
179: }
180: 
```
- **EN**: This block uses control flow (for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 181-200
```cpp
181: // Returns true if the struct is a potential candidate for the floating point
182: // calling convention. If this function returns true, the caller is
183: // responsible for checking that if there is only a single field then that
184: // field is a float.
185: bool RISCVABIInfo::detectFPCCEligibleStructHelper(QualType Ty, CharUnits CurOff,
186:                                                   llvm::Type *&Field1Ty,
187:                                                   CharUnits &Field1Off,
188:                                                   llvm::Type *&Field2Ty,
189:                                                   CharUnits &Field2Off) const {
190:   bool IsInt = Ty->isIntegralOrEnumerationType();
191:   bool IsFloat = Ty->isRealFloatingType();
192: 
193:   if (IsInt || IsFloat) {
194:     uint64_t Size = getContext().getTypeSize(Ty);
195:     if (IsInt && Size > XLen)
196:       return false;
197:     // Can't be eligible if larger than the FP registers. Handling of half
198:     // precision values has been specified in the ABI, so don't block those.
199:     if (IsFloat && Size > FLen)
200:       return false;
```
- **EN**: This block defines callable entry points like `detectFPCCEligibleStructHelper`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `detectFPCCEligibleStructHelper`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 201-220
```cpp
201:     // Can't be eligible if an integer type was already found (int+int pairs
202:     // are not eligible).
203:     if (IsInt && Field1Ty && Field1Ty->isIntegerTy())
204:       return false;
205:     if (!Field1Ty) {
206:       Field1Ty = CGT.ConvertType(Ty);
207:       Field1Off = CurOff;
208:       return true;
209:     }
210:     if (!Field2Ty) {
211:       Field2Ty = CGT.ConvertType(Ty);
212:       Field2Off = CurOff;
213:       return true;
214:     }
215:     return false;
216:   }
217: 
218:   if (auto CTy = Ty->getAs<ComplexType>()) {
219:     if (Field1Ty)
220:       return false;
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 221-240
```cpp
221:     QualType EltTy = CTy->getElementType();
222:     if (getContext().getTypeSize(EltTy) > FLen)
223:       return false;
224:     Field1Ty = CGT.ConvertType(EltTy);
225:     Field1Off = CurOff;
226:     Field2Ty = Field1Ty;
227:     Field2Off = Field1Off + getContext().getTypeSizeInChars(EltTy);
228:     return true;
229:   }
230: 
231:   if (const ConstantArrayType *ATy = getContext().getAsConstantArrayType(Ty)) {
232:     uint64_t ArraySize = ATy->getZExtSize();
233:     QualType EltTy = ATy->getElementType();
234:     // Non-zero-length arrays of empty records make the struct ineligible for
235:     // the FP calling convention in C++.
236:     if (const auto *RTy = EltTy->getAsCanonical<RecordType>()) {
237:       if (ArraySize != 0 && isa<CXXRecordDecl>(RTy->getDecl()) &&
238:           isEmptyRecord(getContext(), EltTy, true, true))
239:         return false;
240:     }
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 241-260
```cpp
241:     CharUnits EltSize = getContext().getTypeSizeInChars(EltTy);
242:     for (uint64_t i = 0; i < ArraySize; ++i) {
243:       bool Ret = detectFPCCEligibleStructHelper(EltTy, CurOff, Field1Ty,
244:                                                 Field1Off, Field2Ty, Field2Off);
245:       if (!Ret)
246:         return false;
247:       CurOff += EltSize;
248:     }
249:     return true;
250:   }
251: 
252:   if (const auto *RTy = Ty->getAsCanonical<RecordType>()) {
253:     // Structures with either a non-trivial destructor or a non-trivial
254:     // copy constructor are not eligible for the FP calling convention.
255:     if (getRecordArgABI(Ty, CGT.getCXXABI()))
256:       return false;
257:     if (isEmptyRecord(getContext(), Ty, true, true))
258:       return true;
259:     const RecordDecl *RD = RTy->getDecl()->getDefinitionOrSelf();
260:     // Unions aren't eligible unless they're empty (which is caught above).
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 261-280
```cpp
261:     if (RD->isUnion())
262:       return false;
263:     const ASTRecordLayout &Layout = getContext().getASTRecordLayout(RD);
264:     // If this is a C++ record, check the bases first.
265:     if (const CXXRecordDecl *CXXRD = dyn_cast<CXXRecordDecl>(RD)) {
266:       for (const CXXBaseSpecifier &B : CXXRD->bases()) {
267:         const auto *BDecl = B.getType()->castAsCXXRecordDecl();
268:         CharUnits BaseOff = Layout.getBaseClassOffset(BDecl);
269:         bool Ret = detectFPCCEligibleStructHelper(B.getType(), CurOff + BaseOff,
270:                                                   Field1Ty, Field1Off, Field2Ty,
271:                                                   Field2Off);
272:         if (!Ret)
273:           return false;
274:       }
275:     }
276:     int ZeroWidthBitFieldCount = 0;
277:     for (const FieldDecl *FD : RD->fields()) {
278:       uint64_t FieldOffInBits = Layout.getFieldOffset(FD->getFieldIndex());
279:       QualType QTy = FD->getType();
280:       if (FD->isBitField()) {
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 281-300
```cpp
281:         unsigned BitWidth = FD->getBitWidthValue();
282:         // Allow a bitfield with a type greater than XLen as long as the
283:         // bitwidth is XLen or less.
284:         if (getContext().getTypeSize(QTy) > XLen && BitWidth <= XLen)
285:           QTy = getContext().getIntTypeForBitwidth(XLen, false);
286:         // Trim type to bitwidth if possible
287:         else if (getContext().getTypeSize(QTy) > BitWidth) {
288:           bool IsSigned =
289:               FD->getType().getTypePtr()->hasSignedIntegerRepresentation();
290:           unsigned Bits = std::max(8U, (unsigned)llvm::PowerOf2Ceil(BitWidth));
291:           QTy = getContext().getIntTypeForBitwidth(Bits, IsSigned);
292:         }
293:         if (BitWidth == 0) {
294:           ZeroWidthBitFieldCount++;
295:           continue;
296:         }
297:       }
298: 
299:       bool Ret = detectFPCCEligibleStructHelper(
300:           QTy, CurOff + getContext().toCharUnitsFromBits(FieldOffInBits),
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 301-320
```cpp
301:           Field1Ty, Field1Off, Field2Ty, Field2Off);
302:       if (!Ret)
303:         return false;
304: 
305:       // As a quirk of the ABI, zero-width bitfields aren't ignored for fp+fp
306:       // or int+fp structs, but are ignored for a struct with an fp field and
307:       // any number of zero-width bitfields.
308:       if (Field2Ty && ZeroWidthBitFieldCount > 0)
309:         return false;
310:     }
311:     return Field1Ty != nullptr;
312:   }
313: 
314:   return false;
315: }
316: 
317: // Determine if a struct is eligible for passing according to the floating
318: // point calling convention (i.e., when flattened it contains a single fp
319: // value, fp+fp, or int+fp of appropriate size). If so, NeededArgFPRs and
320: // NeededArgGPRs are incremented appropriately.
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 321-340
```cpp
321: bool RISCVABIInfo::detectFPCCEligibleStruct(QualType Ty, llvm::Type *&Field1Ty,
322:                                             CharUnits &Field1Off,
323:                                             llvm::Type *&Field2Ty,
324:                                             CharUnits &Field2Off,
325:                                             int &NeededArgGPRs,
326:                                             int &NeededArgFPRs) const {
327:   Field1Ty = nullptr;
328:   Field2Ty = nullptr;
329:   NeededArgGPRs = 0;
330:   NeededArgFPRs = 0;
331:   bool IsCandidate = detectFPCCEligibleStructHelper(
332:       Ty, CharUnits::Zero(), Field1Ty, Field1Off, Field2Ty, Field2Off);
333:   if (!Field1Ty)
334:     return false;
335:   // Not really a candidate if we have a single int but no float.
336:   if (Field1Ty && !Field2Ty && !Field1Ty->isFloatingPointTy())
337:     return false;
338:   if (!IsCandidate)
339:     return false;
340:   if (Field1Ty && Field1Ty->isFloatingPointTy())
```
- **EN**: This block defines callable entry points like `detectFPCCEligibleStruct`, `Zero`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `detectFPCCEligibleStruct`, `Zero`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 341-360
```cpp
341:     NeededArgFPRs++;
342:   else if (Field1Ty)
343:     NeededArgGPRs++;
344:   if (Field2Ty && Field2Ty->isFloatingPointTy())
345:     NeededArgFPRs++;
346:   else if (Field2Ty)
347:     NeededArgGPRs++;
348:   return true;
349: }
350: 
351: // Call getCoerceAndExpand for the two-element flattened struct described by
352: // Field1Ty, Field1Off, Field2Ty, Field2Off. This method will create an
353: // appropriate coerceToType and unpaddedCoerceToType.
354: ABIArgInfo RISCVABIInfo::coerceAndExpandFPCCEligibleStruct(
355:     llvm::Type *Field1Ty, CharUnits Field1Off, llvm::Type *Field2Ty,
356:     CharUnits Field2Off) const {
357:   SmallVector<llvm::Type *, 3> CoerceElts;
358:   SmallVector<llvm::Type *, 2> UnpaddedCoerceElts;
359:   if (!Field1Off.isZero())
360:     CoerceElts.push_back(llvm::ArrayType::get(
```
- **EN**: This block defines callable entry points like `coerceAndExpandFPCCEligibleStruct`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `coerceAndExpandFPCCEligibleStruct`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 361-380
```cpp
361:         llvm::Type::getInt8Ty(getVMContext()), Field1Off.getQuantity()));
362: 
363:   CoerceElts.push_back(Field1Ty);
364:   UnpaddedCoerceElts.push_back(Field1Ty);
365: 
366:   if (!Field2Ty) {
367:     return ABIArgInfo::getCoerceAndExpand(
368:         llvm::StructType::get(getVMContext(), CoerceElts, !Field1Off.isZero()),
369:         UnpaddedCoerceElts[0]);
370:   }
371: 
372:   CharUnits Field2Align =
373:       CharUnits::fromQuantity(getDataLayout().getABITypeAlign(Field2Ty));
374:   CharUnits Field1End = Field1Off +
375:       CharUnits::fromQuantity(getDataLayout().getTypeStoreSize(Field1Ty));
376:   CharUnits Field2OffNoPadNoPack = Field1End.alignTo(Field2Align);
377: 
378:   CharUnits Padding = CharUnits::Zero();
379:   if (Field2Off > Field2OffNoPadNoPack)
380:     Padding = Field2Off - Field2OffNoPadNoPack;
```
- **EN**: This block defines callable entry points like `getInt8Ty`, `getCoerceAndExpand`, `fromQuantity`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getInt8Ty`, `getCoerceAndExpand`, `fromQuantity`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 381-400
```cpp
381:   else if (Field2Off != Field2Align && Field2Off > Field1End)
382:     Padding = Field2Off - Field1End;
383: 
384:   bool IsPacked = !Field2Off.isMultipleOf(Field2Align);
385: 
386:   if (!Padding.isZero())
387:     CoerceElts.push_back(llvm::ArrayType::get(
388:         llvm::Type::getInt8Ty(getVMContext()), Padding.getQuantity()));
389: 
390:   CoerceElts.push_back(Field2Ty);
391:   UnpaddedCoerceElts.push_back(Field2Ty);
392: 
393:   auto CoerceToType =
394:       llvm::StructType::get(getVMContext(), CoerceElts, IsPacked);
395:   auto UnpaddedCoerceToType =
396:       llvm::StructType::get(getVMContext(), UnpaddedCoerceElts, IsPacked);
397: 
398:   return ABIArgInfo::getCoerceAndExpand(CoerceToType, UnpaddedCoerceToType);
399: }
400: 
```
- **EN**: This block spells out callable entry points like `get`, `getCoerceAndExpand`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`, `getCoerceAndExpand`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 401-420
```cpp
401: bool RISCVABIInfo::detectVLSCCEligibleStruct(QualType Ty, unsigned ABIVLen,
402:                                              llvm::Type *&VLSType) const {
403:   // No riscv_vls_cc attribute.
404:   if (ABIVLen == 0)
405:     return false;
406: 
407:   // Legal struct for VLS calling convention should fulfill following rules:
408:   // 1. Struct element should be either "homogeneous fixed-length vectors" or "a
409:   //    fixed-length vector array".
410:   // 2. Number of struct elements or array elements should be greater or equal
411:   //    to 1 and less or equal to 8
412:   // 3. Total number of vector registers needed should not exceed 8.
413:   //
414:   // Examples: Assume ABI_VLEN = 128.
415:   // These are legal structs:
416:   //   a. Structs with 1~8 "same" fixed-length vectors, e.g.
417:   //   struct {
418:   //     __attribute__((vector_size(16))) int a;
419:   //     __attribute__((vector_size(16))) int b;
420:   //   }
```
- **EN**: This block defines callable entry points like `detectVLSCCEligibleStruct`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `detectVLSCCEligibleStruct`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 421-440
```cpp
421:   //
422:   //   b. Structs with "single" fixed-length vector array with lengh 1~8, e.g.
423:   //   struct {
424:   //     __attribute__((vector_size(16))) int a[3];
425:   //   }
426:   // These are illegal structs:
427:   //   a. Structs with 9 fixed-length vectors, e.g.
428:   //   struct {
429:   //     __attribute__((vector_size(16))) int a;
430:   //     __attribute__((vector_size(16))) int b;
431:   //     __attribute__((vector_size(16))) int c;
432:   //     __attribute__((vector_size(16))) int d;
433:   //     __attribute__((vector_size(16))) int e;
434:   //     __attribute__((vector_size(16))) int f;
435:   //     __attribute__((vector_size(16))) int g;
436:   //     __attribute__((vector_size(16))) int h;
437:   //     __attribute__((vector_size(16))) int i;
438:   //   }
439:   //
440:   //   b. Structs with "multiple" fixed-length vector array, e.g.
```
- **EN**: This block documents intent or context for the surrounding target-specific ABI and code generation code.
- **CN**: 该代码块说明周围 目标相关的 ABI 与代码生成 代码的意图或上下文。

### Lines 441-460
```cpp
441:   //   struct {
442:   //     __attribute__((vector_size(16))) int a[2];
443:   //     __attribute__((vector_size(16))) int b[2];
444:   //   }
445:   //
446:   //   c. Vector registers needed exceeds 8, e.g.
447:   //   struct {
448:   //     // Registers needed for single fixed-length element:
449:   //     // 64 * 8 / ABI_VLEN = 4
450:   //     __attribute__((vector_size(64))) int a;
451:   //     __attribute__((vector_size(64))) int b;
452:   //     __attribute__((vector_size(64))) int c;
453:   //     __attribute__((vector_size(64))) int d;
454:   //   }
455:   //
456:   // 1. Struct of 1 fixed-length vector is passed as a scalable vector.
457:   // 2. Struct of >1 fixed-length vectors are passed as vector tuple.
458:   // 3. Struct of an array with 1 element of fixed-length vectors is passed as a
459:   //    scalable vector.
460:   // 4. Struct of an array with >1 elements of fixed-length vectors is passed as
```
- **EN**: This block documents intent or context for the surrounding target-specific ABI and code generation code.
- **CN**: 该代码块说明周围 目标相关的 ABI 与代码生成 代码的意图或上下文。

### Lines 461-480
```cpp
461:   //    vector tuple.
462:   // 5. Otherwise, pass the struct indirectly.
463: 
464:   llvm::StructType *STy = dyn_cast<llvm::StructType>(CGT.ConvertType(Ty));
465:   if (!STy)
466:     return false;
467: 
468:   unsigned NumElts = STy->getStructNumElements();
469:   if (NumElts > 8)
470:     return false;
471: 
472:   auto *FirstEltTy = STy->getElementType(0);
473:   if (!STy->containsHomogeneousTypes())
474:     return false;
475: 
476:   if (auto *ArrayTy = dyn_cast<llvm::ArrayType>(FirstEltTy)) {
477:     // Only struct of single array is accepted
478:     if (NumElts != 1)
479:       return false;
480:     FirstEltTy = ArrayTy->getArrayElementType();
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 481-500
```cpp
481:     NumElts = ArrayTy->getNumElements();
482:   }
483: 
484:   auto *FixedVecTy = dyn_cast<llvm::FixedVectorType>(FirstEltTy);
485:   if (!FixedVecTy)
486:     return false;
487: 
488:   // Check registers needed <= 8.
489:   if (NumElts * llvm::divideCeil(
490:                     FixedVecTy->getNumElements() *
491:                         FixedVecTy->getElementType()->getScalarSizeInBits(),
492:                     ABIVLen) >
493:       8)
494:     return false;
495: 
496:   // Turn them into scalable vector type or vector tuple type if legal.
497:   if (NumElts == 1) {
498:     // Handle single fixed-length vector.
499:     VLSType = llvm::ScalableVectorType::get(
500:         FixedVecTy->getElementType(),
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 501-520
```cpp
501:         llvm::divideCeil(FixedVecTy->getNumElements() *
502:                              llvm::RISCV::RVVBitsPerBlock,
503:                          ABIVLen));
504:     return true;
505:   }
506: 
507:   // LMUL
508:   // = fixed-length vector size / ABIVLen
509:   // = 8 * I8EltCount / RVVBitsPerBlock
510:   // =>
511:   // I8EltCount
512:   // = (fixed-length vector size * RVVBitsPerBlock) / (ABIVLen * 8)
513:   unsigned I8EltCount =
514:       llvm::divideCeil(FixedVecTy->getNumElements() *
515:                            FixedVecTy->getElementType()->getScalarSizeInBits() *
516:                            llvm::RISCV::RVVBitsPerBlock,
517:                        ABIVLen * 8);
518:   VLSType = llvm::TargetExtType::get(
519:       getVMContext(), "riscv.vector.tuple",
520:       llvm::ScalableVectorType::get(llvm::Type::getInt8Ty(getVMContext()),
```
- **EN**: This block spells out callable entry points like `divideCeil`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块给出可调用入口的声明，例如 `divideCeil`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 521-540
```cpp
521:                                     I8EltCount),
522:       NumElts);
523:   return true;
524: }
525: 
526: // Fixed-length RVV vectors are represented as scalable vectors in function
527: // args/return and must be coerced from fixed vectors.
528: ABIArgInfo RISCVABIInfo::coerceVLSVector(QualType Ty, unsigned ABIVLen) const {
529:   assert(Ty->isVectorType() && "expected vector type!");
530: 
531:   const auto *VT = Ty->castAs<VectorType>();
532:   assert(VT->getElementType()->isBuiltinType() && "expected builtin type!");
533: 
534:   auto VScale = getContext().getTargetInfo().getVScaleRange(
535:       getContext().getLangOpts(), TargetInfo::ArmStreamingKind::NotStreaming);
536: 
537:   unsigned NumElts = VT->getNumElements();
538:   llvm::Type *EltType = llvm::Type::getInt1Ty(getVMContext());
539:   switch (VT->getVectorKind()) {
540:   case VectorKind::RVVFixedLengthMask_1:
```
- **EN**: This block defines callable entry points like `coerceVLSVector`, `getContext`; uses control flow (switch, case) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `coerceVLSVector`, `getContext`；通过控制流（switch, case）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 541-560
```cpp
541:     break;
542:   case VectorKind::RVVFixedLengthMask_2:
543:     NumElts *= 2;
544:     break;
545:   case VectorKind::RVVFixedLengthMask_4:
546:     NumElts *= 4;
547:     break;
548:   case VectorKind::RVVFixedLengthMask:
549:     NumElts *= 8;
550:     break;
551:   default:
552:     assert((VT->getVectorKind() == VectorKind::Generic ||
553:             VT->getVectorKind() == VectorKind::RVVFixedLengthData) &&
554:            "Unexpected vector kind");
555:     EltType = CGT.ConvertType(VT->getElementType());
556:   }
557: 
558:   llvm::ScalableVectorType *ResType;
559: 
560:   if (ABIVLen == 0) {
```
- **EN**: This block uses control flow (if, case) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 561-580
```cpp
561:     // The MinNumElts is simplified from equation:
562:     // NumElts / VScale =
563:     //  (EltSize * NumElts / (VScale * RVVBitsPerBlock))
564:     //    * (RVVBitsPerBlock / EltSize)
565:     ResType = llvm::ScalableVectorType::get(EltType, NumElts / VScale->first);
566:   } else {
567:     // Check registers needed <= 8.
568:     if ((EltType->getScalarSizeInBits() * NumElts / ABIVLen) > 8)
569:       return getNaturalAlignIndirect(
570:           Ty, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
571:           /*ByVal=*/false);
572: 
573:     // Generic vector
574:     // The number of elements needs to be at least 1.
575:     ResType = llvm::ScalableVectorType::get(
576:         EltType,
577:         llvm::divideCeil(NumElts * llvm::RISCV::RVVBitsPerBlock, ABIVLen));
578: 
579:     // If the corresponding extension is not supported, just make it an i8
580:     // vector with same LMUL.
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 581-600
```cpp
581:     const TargetInfo &TI = getContext().getTargetInfo();
582:     if ((EltType->isHalfTy() && !TI.hasFeature("zvfhmin")) ||
583:         (EltType->isBFloatTy() && !TI.hasFeature("zvfbfmin")) ||
584:         (EltType->isFloatTy() && !TI.hasFeature("zve32f")) ||
585:         (EltType->isDoubleTy() && !TI.hasFeature("zve64d")) ||
586:         (EltType->isIntegerTy(64) && !TI.hasFeature("zve64x")) ||
587:         EltType->isIntegerTy(128)) {
588:       // The number of elements needs to be at least 1.
589:       ResType = llvm::ScalableVectorType::get(
590:           llvm::Type::getInt8Ty(getVMContext()),
591:           llvm::divideCeil(EltType->getScalarSizeInBits() * NumElts *
592:                                llvm::RISCV::RVVBitsPerBlock,
593:                            8 * ABIVLen));
594:     }
595:   }
596: 
597:   return ABIArgInfo::getDirect(ResType);
598: }
599: 
600: ABIArgInfo RISCVABIInfo::classifyArgumentType(QualType Ty, bool IsFixed,
```
- **EN**: This block defines callable entry points like `getInt8Ty`, `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getInt8Ty`, `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 601-620
```cpp
601:                                               int &ArgGPRsLeft,
602:                                               int &ArgFPRsLeft,
603:                                               unsigned ABIVLen) const {
604:   assert(ArgGPRsLeft <= NumArgGPRs && "Arg GPR tracking underflow");
605:   Ty = useFirstFieldIfTransparentUnion(Ty);
606: 
607:   // Structures with either a non-trivial destructor or a non-trivial
608:   // copy constructor are always passed indirectly.
609:   if (CGCXXABI::RecordArgABI RAA = getRecordArgABI(Ty, getCXXABI())) {
610:     if (ArgGPRsLeft)
611:       ArgGPRsLeft -= 1;
612:     return getNaturalAlignIndirect(
613:         Ty, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
614:         /*ByVal=*/RAA == CGCXXABI::RAA_DirectInMemory);
615:   }
616: 
617:   uint64_t Size = getContext().getTypeSize(Ty);
618: 
619:   // Ignore empty structs/unions whose size is zero. According to the calling
620:   // convention empty structs/unions are required to be sized types in C++.
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 621-640
```cpp
621:   if (isEmptyRecord(getContext(), Ty, true) && Size == 0)
622:     return ABIArgInfo::getIgnore();
623: 
624:   // Pass floating point values via FPRs if possible.
625:   if (IsFixed && Ty->isFloatingType() && !Ty->isComplexType() &&
626:       FLen >= Size && ArgFPRsLeft) {
627:     ArgFPRsLeft--;
628:     return ABIArgInfo::getDirect();
629:   }
630: 
631:   // Complex types for the hard float ABI must be passed direct rather than
632:   // using CoerceAndExpand.
633:   if (IsFixed && Ty->isComplexType() && FLen && ArgFPRsLeft >= 2) {
634:     QualType EltTy = Ty->castAs<ComplexType>()->getElementType();
635:     if (getContext().getTypeSize(EltTy) <= FLen) {
636:       ArgFPRsLeft -= 2;
637:       return ABIArgInfo::getDirect();
638:     }
639:   }
640: 
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 641-660
```cpp
641:   if (IsFixed && FLen && Ty->isStructureOrClassType()) {
642:     llvm::Type *Field1Ty = nullptr;
643:     llvm::Type *Field2Ty = nullptr;
644:     CharUnits Field1Off = CharUnits::Zero();
645:     CharUnits Field2Off = CharUnits::Zero();
646:     int NeededArgGPRs = 0;
647:     int NeededArgFPRs = 0;
648:     bool IsCandidate =
649:         detectFPCCEligibleStruct(Ty, Field1Ty, Field1Off, Field2Ty, Field2Off,
650:                                  NeededArgGPRs, NeededArgFPRs);
651:     if (IsCandidate && NeededArgGPRs <= ArgGPRsLeft &&
652:         NeededArgFPRs <= ArgFPRsLeft) {
653:       ArgGPRsLeft -= NeededArgGPRs;
654:       ArgFPRsLeft -= NeededArgFPRs;
655:       return coerceAndExpandFPCCEligibleStruct(Field1Ty, Field1Off, Field2Ty,
656:                                                Field2Off);
657:     }
658:   }
659: 
660:   if (IsFixed && Ty->isStructureOrClassType()) {
```
- **EN**: This block defines callable entry points like `detectFPCCEligibleStruct`, `coerceAndExpandFPCCEligibleStruct`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `detectFPCCEligibleStruct`, `coerceAndExpandFPCCEligibleStruct`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 661-680
```cpp
661:     llvm::Type *VLSType = nullptr;
662:     if (detectVLSCCEligibleStruct(Ty, ABIVLen, VLSType))
663:       return ABIArgInfo::getTargetSpecific(VLSType);
664:   }
665: 
666:   uint64_t NeededAlign = getContext().getTypeAlign(Ty);
667:   // Determine the number of GPRs needed to pass the current argument
668:   // according to the ABI. 2*XLen-aligned varargs are passed in "aligned"
669:   // register pairs, so may consume 3 registers.
670:   // TODO: To be compatible with GCC's behaviors, we don't align registers
671:   // currently if we are using ILP32E calling convention. This behavior may be
672:   // changed when RV32E/ILP32E is ratified.
673:   int NeededArgGPRs = 1;
674:   if (!IsFixed && NeededAlign == 2 * XLen)
675:     NeededArgGPRs = 2 + (EABI && XLen == 32 ? 0 : (ArgGPRsLeft % 2));
676:   else if (Size > XLen && Size <= 2 * XLen)
677:     NeededArgGPRs = 2;
678: 
679:   if (NeededArgGPRs > ArgGPRsLeft) {
680:     NeededArgGPRs = ArgGPRsLeft;
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 681-700
```cpp
681:   }
682: 
683:   ArgGPRsLeft -= NeededArgGPRs;
684: 
685:   if (!isAggregateTypeForABI(Ty) && !Ty->isVectorType()) {
686:     // Treat an enum type as its underlying type.
687:     if (const auto *ED = Ty->getAsEnumDecl())
688:       Ty = ED->getIntegerType();
689: 
690:     if (const auto *EIT = Ty->getAs<BitIntType>()) {
691: 
692:       if (XLen == 64 && EIT->getNumBits() == 32)
693:         return extendType(Ty, CGT.ConvertType(Ty));
694: 
695:       if (EIT->getNumBits() <= 2 * XLen)
696:         return ABIArgInfo::getExtend(Ty, CGT.ConvertType(Ty));
697:       return getNaturalAlignIndirect(
698:           Ty, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
699:           /*ByVal=*/false);
700:     }
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 701-720
```cpp
701: 
702:     // All integral types are promoted to XLen width
703:     if (Size < XLen && Ty->isIntegralOrEnumerationType())
704:       return extendType(Ty, CGT.ConvertType(Ty));
705: 
706:     return ABIArgInfo::getDirect();
707:   }
708: 
709:   // TODO: _BitInt is not handled yet in VLS calling convention since _BitInt
710:   // ABI is also not merged yet in RISC-V:
711:   // https://github.com/riscv-non-isa/riscv-elf-psabi-doc/pull/419
712:   if (const VectorType *VT = Ty->getAs<VectorType>();
713:       VT && !VT->getElementType()->isBitIntType()) {
714:     if (VT->getVectorKind() == VectorKind::RVVFixedLengthData ||
715:         VT->getVectorKind() == VectorKind::RVVFixedLengthMask ||
716:         VT->getVectorKind() == VectorKind::RVVFixedLengthMask_1 ||
717:         VT->getVectorKind() == VectorKind::RVVFixedLengthMask_2 ||
718:         VT->getVectorKind() == VectorKind::RVVFixedLengthMask_4)
719:       return coerceVLSVector(Ty);
720:     if (VT->getVectorKind() == VectorKind::Generic && ABIVLen != 0)
```
- **EN**: This block defines callable entry points like `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 721-740
```cpp
721:       // Generic vector without riscv_vls_cc should fall through and pass by
722:       // reference.
723:       return coerceVLSVector(Ty, ABIVLen);
724:   }
725: 
726:   // Aggregates which are <= 2*XLen will be passed in registers if possible,
727:   // so coerce to integers.
728:   if (Size <= 2 * XLen) {
729:     unsigned Alignment = getContext().getTypeAlign(Ty);
730: 
731:     if (Size <= XLen) {
732:       // Use the smallest integer type we can.
733:       return ABIArgInfo::getDirect(
734:           llvm::IntegerType::get(getVMContext(), Size));
735:     }
736:     // Use 2*XLen if 2*XLen alignment is required.
737:     if (Alignment == 2 * XLen)
738:       return ABIArgInfo::getDirect(
739:           llvm::IntegerType::get(getVMContext(), 2 * XLen));
740:     // Use 2-element XLen array if only XLen alignment is required.
```
- **EN**: This block defines callable entry points like `coerceVLSVector`, `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `coerceVLSVector`, `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 741-760
```cpp
741:     return ABIArgInfo::getDirect(
742:         llvm::ArrayType::get(llvm::IntegerType::get(getVMContext(), XLen), 2));
743:   }
744:   return getNaturalAlignIndirect(
745:       Ty, /*AddrSpace=*/getDataLayout().getAllocaAddrSpace(),
746:       /*ByVal=*/false);
747: }
748: 
749: ABIArgInfo RISCVABIInfo::classifyReturnType(QualType RetTy,
750:                                             unsigned ABIVLen) const {
751:   if (RetTy->isVoidType())
752:     return ABIArgInfo::getIgnore();
753: 
754:   int ArgGPRsLeft = 2;
755:   int ArgFPRsLeft = FLen ? 2 : 0;
756: 
757:   // The rules for return and argument types are the same, so defer to
758:   // classifyArgumentType.
759:   return classifyArgumentType(RetTy, /*IsFixed=*/true, ArgGPRsLeft, ArgFPRsLeft,
760:                               ABIVLen);
```
- **EN**: This block defines callable entry points like `getDirect`, `classifyReturnType`, `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `classifyReturnType`, `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 761-780
```cpp
761: }
762: 
763: RValue RISCVABIInfo::EmitVAArg(CodeGenFunction &CGF, Address VAListAddr,
764:                                QualType Ty, AggValueSlot Slot) const {
765:   CharUnits SlotSize = CharUnits::fromQuantity(XLen / 8);
766: 
767:   // Empty records are ignored for parameter passing purposes.
768:   if (isEmptyRecord(getContext(), Ty, true))
769:     return Slot.asRValue();
770: 
771:   auto TInfo = getContext().getTypeInfoInChars(Ty);
772: 
773:   // TODO: To be compatible with GCC's behaviors, we force arguments with
774:   // 2×XLEN-bit alignment and size at most 2×XLEN bits like `long long`,
775:   // `unsigned long long` and `double` to have 4-byte alignment. This
776:   // behavior may be changed when RV32E/ILP32E is ratified.
777:   if (EABI && XLen == 32)
778:     TInfo.Align = std::min(TInfo.Align, CharUnits::fromQuantity(4));
779: 
780:   // Arguments bigger than 2*Xlen bytes are passed indirectly.
```
- **EN**: This block defines callable entry points like `EmitVAArg`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `EmitVAArg`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 781-800
```cpp
781:   bool IsIndirect = TInfo.Width > 2 * SlotSize;
782: 
783:   return emitVoidPtrVAArg(CGF, VAListAddr, Ty, IsIndirect, TInfo, SlotSize,
784:                           /*AllowHigherAlign=*/true, Slot);
785: }
786: 
787: ABIArgInfo RISCVABIInfo::extendType(QualType Ty, llvm::Type *CoerceTy) const {
788:   int TySize = getContext().getTypeSize(Ty);
789:   // RV64 ABI requires unsigned 32 bit integers to be sign extended.
790:   if (XLen == 64 && Ty->isUnsignedIntegerOrEnumerationType() && TySize == 32)
791:     return ABIArgInfo::getSignExtend(Ty, CoerceTy);
792:   return ABIArgInfo::getExtend(Ty, CoerceTy);
793: }
794: 
795: llvm::Value *RISCVABIInfo::createCoercedLoad(Address Src, const ABIArgInfo &AI,
796:                                              CodeGenFunction &CGF) const {
797:   llvm::Type *Ty = AI.getCoerceToType();
798:   llvm::Type *SrcTy = Src.getElementType();
799:   llvm::StructType *SrcSTy = cast<llvm::StructType>(SrcTy);
800:   assert((Ty->isScalableTy() || Ty->isTargetExtTy()) &&
```
- **EN**: This block defines callable entry points like `extendType`, `getExtend`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `extendType`, `getExtend`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 801-820
```cpp
801:          "Only scalable vector type and vector tuple type are allowed for load "
802:          "type.");
803:   if (llvm::TargetExtType *TupTy = dyn_cast<llvm::TargetExtType>(Ty)) {
804:     // In RISC-V VLS calling convention, struct of fixed vectors or struct of
805:     // array of fixed vector of length >1 might be lowered using vector tuple
806:     // type, we consider it as a valid load, e.g.
807:     // struct i32x4x2 {
808:     //     __attribute__((vector_size(16))) int i;
809:     //     __attribute__((vector_size(16))) int i;
810:     // };
811:     // or
812:     // struct i32x4 {
813:     //     __attribute__((vector_size(16))) int i[2];
814:     // };
815:     // is lowered to target("riscv.vector.tuple", <vscale x 8 x i8>, 2)
816:     // when ABI_VLEN = 128 bits, please checkout
817:     // clang/test/CodeGen/RISCV/riscv-vector-callingconv-llvm-ir.c
818:     // for more information.
819:     assert(TupTy->getName() == "riscv.vector.tuple");
820:     llvm::Type *EltTy = TupTy->getTypeParameter(0);
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 821-840
```cpp
821:     unsigned NumElts = TupTy->getIntParameter(0);
822: 
823:     if (auto *ArrayTy = dyn_cast<llvm::ArrayType>(SrcSTy->getElementType(0)))
824:       Src = Src.withElementType(ArrayTy);
825: 
826:     // Perform extract element and load
827:     llvm::Value *TupleVal = llvm::PoisonValue::get(Ty);
828:     auto *Load = CGF.Builder.CreateLoad(Src);
829:     for (unsigned i = 0; i < NumElts; ++i) {
830:       // Extract from struct
831:       llvm::Value *ExtractFromLoad = CGF.Builder.CreateExtractValue(Load, i);
832:       // Element in vector tuple type is always i8, so we need to cast back to
833:       // it's original element type.
834:       EltTy =
835:           cast<llvm::ScalableVectorType>(llvm::VectorType::getWithSizeAndScalar(
836:               cast<llvm::VectorType>(EltTy), ExtractFromLoad->getType()));
837:       llvm::Value *VectorVal = llvm::PoisonValue::get(EltTy);
838:       // Insert to scalable vector
839:       VectorVal = CGF.Builder.CreateInsertVector(
840:           EltTy, VectorVal, ExtractFromLoad, uint64_t(0), "cast.scalable");
```
- **EN**: This block defines callable entry points like `uint64_t`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `uint64_t`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 841-860
```cpp
841:       // Insert scalable vector to vector tuple
842:       llvm::Value *Idx = CGF.Builder.getInt32(i);
843:       TupleVal =
844:           CGF.Builder.CreateIntrinsic(llvm::Intrinsic::riscv_tuple_insert,
845:                                       {Ty, EltTy}, {TupleVal, VectorVal, Idx});
846:     }
847:     return TupleVal;
848:   }
849: 
850:   // In RISC-V VLS calling convention, struct of fixed vector or struct of
851:   // fixed vector array of length 1 might be lowered using scalable vector,
852:   // we consider it as a valid load, e.g.
853:   // struct i32x4 {
854:   //     __attribute__((vector_size(16))) int i;
855:   // };
856:   // or
857:   // struct i32x4 {
858:   //     __attribute__((vector_size(16))) int i[1];
859:   // };
860:   // is lowered to <vscale x 2 x i32>
```
- **EN**: This block documents intent or context for the surrounding target-specific ABI and code generation code.
- **CN**: 该代码块说明周围 目标相关的 ABI 与代码生成 代码的意图或上下文。

### Lines 861-880
```cpp
861:   // when ABI_VLEN = 128 bits, please checkout
862:   // clang/test/CodeGen/RISCV/riscv-vector-callingconv-llvm-ir.c
863:   // for more information.
864:   auto *ScalableDstTy = cast<llvm::ScalableVectorType>(Ty);
865:   SrcTy = SrcSTy->getElementType(0);
866:   if (auto *ArrayTy = dyn_cast<llvm::ArrayType>(SrcTy))
867:     SrcTy = ArrayTy->getElementType();
868:   Src = Src.withElementType(SrcTy);
869:   [[maybe_unused]] auto *FixedSrcTy = cast<llvm::FixedVectorType>(SrcTy);
870:   assert(ScalableDstTy->getElementType() == FixedSrcTy->getElementType());
871:   auto *Load = CGF.Builder.CreateLoad(Src);
872:   auto *VectorVal = llvm::PoisonValue::get(ScalableDstTy);
873:   llvm::Value *Result = CGF.Builder.CreateInsertVector(
874:       ScalableDstTy, VectorVal, Load, uint64_t(0), "cast.scalable");
875:   return Result;
876: }
877: 
878: void RISCVABIInfo::createCoercedStore(llvm::Value *Val, Address Dst,
879:                                       const ABIArgInfo &AI, bool DestIsVolatile,
880:                                       CodeGenFunction &CGF) const {
```
- **EN**: This block defines callable entry points like `uint64_t`, `createCoercedStore`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `uint64_t`, `createCoercedStore`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 881-900
```cpp
881:   llvm::Type *SrcTy = Val->getType();
882:   llvm::StructType *DstSTy = cast<llvm::StructType>(Dst.getElementType());
883:   assert((SrcTy->isScalableTy() || SrcTy->isTargetExtTy()) &&
884:          "Only scalable vector type and vector tuple type are allowed for "
885:          "store value.");
886:   if (llvm::TargetExtType *TupTy = dyn_cast<llvm::TargetExtType>(SrcTy)) {
887:     // In RISC-V VLS calling convention, struct of fixed vectors or struct
888:     // of array of fixed vector of length >1 might be lowered using vector
889:     // tuple type, we consider it as a valid load, e.g.
890:     // struct i32x4x2 {
891:     //     __attribute__((vector_size(16))) int i;
892:     //     __attribute__((vector_size(16))) int i;
893:     // };
894:     // or
895:     // struct i32x4 {
896:     //     __attribute__((vector_size(16))) int i[2];
897:     // };
898:     // is lowered to target("riscv.vector.tuple", <vscale x 8 x i8>, 2)
899:     // when ABI_VLEN = 128 bits, please checkout
900:     // clang/test/CodeGen/RISCV/riscv-vector-callingconv-llvm-ir.c
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 901-920
```cpp
901:     // for more information.
902:     assert(TupTy->getName() == "riscv.vector.tuple");
903:     llvm::Type *EltTy = TupTy->getTypeParameter(0);
904:     unsigned NumElts = TupTy->getIntParameter(0);
905: 
906:     llvm::Type *FixedVecTy = DstSTy->getElementType(0);
907:     if (auto *ArrayTy = dyn_cast<llvm::ArrayType>(DstSTy->getElementType(0))) {
908:       Dst = Dst.withElementType(ArrayTy);
909:       FixedVecTy = ArrayTy->getArrayElementType();
910:     }
911: 
912:     // Perform extract element and store
913:     for (unsigned i = 0; i < NumElts; ++i) {
914:       // Element in vector tuple type is always i8, so we need to cast back
915:       // to it's original element type.
916:       EltTy =
917:           cast<llvm::ScalableVectorType>(llvm::VectorType::getWithSizeAndScalar(
918:               cast<llvm::VectorType>(EltTy), FixedVecTy));
919:       // Extract scalable vector from tuple
920:       llvm::Value *Idx = CGF.Builder.getInt32(i);
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 921-940
```cpp
921:       auto *TupleElement = CGF.Builder.CreateIntrinsic(
922:           llvm::Intrinsic::riscv_tuple_extract, {EltTy, TupTy}, {Val, Idx});
923: 
924:       // Extract fixed vector from scalable vector
925:       auto *ExtractVec = CGF.Builder.CreateExtractVector(
926:           FixedVecTy, TupleElement, uint64_t(0));
927:       // Store fixed vector to corresponding address
928:       Address EltPtr = Address::invalid();
929:       if (Dst.getElementType()->isStructTy())
930:         EltPtr = CGF.Builder.CreateStructGEP(Dst, i);
931:       else
932:         EltPtr = CGF.Builder.CreateConstArrayGEP(Dst, i);
933:       auto *I = CGF.Builder.CreateStore(ExtractVec, EltPtr, DestIsVolatile);
934:       CGF.addInstToCurrentSourceAtom(I, ExtractVec);
935:     }
936:     return;
937:   }
938: 
939:   // In RISC-V VLS calling convention, struct of fixed vector or struct of
940:   // fixed vector array of length 1 might be lowered using scalable
```
- **EN**: This block defines callable entry points like `uint64_t`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `uint64_t`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 941-960
```cpp
941:   // vector, we consider it as a valid load, e.g.
942:   // struct i32x4 {
943:   //     __attribute__((vector_size(16))) int i;
944:   // };
945:   // or
946:   // struct i32x4 {
947:   //     __attribute__((vector_size(16))) int i[1];
948:   // };
949:   // is lowered to <vscale x 2 x i32>
950:   // when ABI_VLEN = 128 bits, please checkout
951:   // clang/test/CodeGen/RISCV/riscv-vector-callingconv-llvm-ir.c
952:   // for more information.
953:   llvm::Type *EltTy = DstSTy->getElementType(0);
954:   if (auto *ArrayTy = dyn_cast<llvm::ArrayType>(EltTy)) {
955:     assert(ArrayTy->getNumElements() == 1);
956:     EltTy = ArrayTy->getElementType();
957:   }
958:   auto *Coerced = CGF.Builder.CreateExtractVector(
959:       cast<llvm::FixedVectorType>(EltTy), Val, uint64_t(0));
960:   auto *I = CGF.Builder.CreateStore(Coerced, Dst, DestIsVolatile);
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 961-980
```cpp
961:   CGF.addInstToCurrentSourceAtom(I, Val);
962: }
963: 
964: namespace {
965: class RISCVTargetCodeGenInfo : public TargetCodeGenInfo {
966: public:
967:   RISCVTargetCodeGenInfo(CodeGen::CodeGenTypes &CGT, unsigned XLen,
968:                          unsigned FLen, bool EABI)
969:       : TargetCodeGenInfo(
970:             std::make_unique<RISCVABIInfo>(CGT, XLen, FLen, EABI)) {
971:     SwiftInfo =
972:         std::make_unique<SwiftABIInfo>(CGT, /*SwiftErrorInRegister=*/false);
973:   }
974: 
975:   void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
976:                            CodeGen::CodeGenModule &CGM) const override {
977:     const auto *FD = dyn_cast_or_null<FunctionDecl>(D);
978:     if (!FD) return;
979: 
980:     auto *Fn = cast<llvm::Function>(GV);
```
- **EN**: This block introduces declarations such as `RISCVTargetCodeGenInfo`; defines callable entry points like `RISCVTargetCodeGenInfo`, `setTargetAttributes`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出诸如 `RISCVTargetCodeGenInfo` 的声明；定义可调用入口，例如 `RISCVTargetCodeGenInfo`, `setTargetAttributes`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 981-1000
```cpp
 981: 
 982:     if (CGM.getCodeGenOpts().CFProtectionReturn)
 983:       Fn->addFnAttr("hw-shadow-stack");
 984: 
 985:     const auto *Attr = FD->getAttr<RISCVInterruptAttr>();
 986:     if (!Attr)
 987:       return;
 988: 
 989:     StringRef Kind = "machine";
 990:     bool HasSiFiveCLICPreemptible = false;
 991:     bool HasSiFiveCLICStackSwap = false;
 992:     for (RISCVInterruptAttr::InterruptType type : Attr->interrupt()) {
 993:       switch (type) {
 994:       case RISCVInterruptAttr::machine:
 995:         // Do not update `Kind` because `Kind` is already "machine", or the
 996:         // kinds also contains SiFive types which need to be applied.
 997:         break;
 998:       case RISCVInterruptAttr::supervisor:
 999:         Kind = "supervisor";
1000:         break;
```
- **EN**: This block uses control flow (if, switch, for, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, switch, for, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1001-1020
```cpp
1001:       case RISCVInterruptAttr::rnmi:
1002:         Kind = "rnmi";
1003:         break;
1004:       case RISCVInterruptAttr::qcinest:
1005:         Kind = "qci-nest";
1006:         break;
1007:       case RISCVInterruptAttr::qcinonest:
1008:         Kind = "qci-nonest";
1009:         break;
1010:       // There are three different LLVM IR attribute values for SiFive CLIC
1011:       // interrupt kinds, one for each kind and one extra for their combination.
1012:       case RISCVInterruptAttr::SiFiveCLICPreemptible: {
1013:         HasSiFiveCLICPreemptible = true;
1014:         Kind = HasSiFiveCLICStackSwap ? "SiFive-CLIC-preemptible-stack-swap"
1015:                                       : "SiFive-CLIC-preemptible";
1016:         break;
1017:       }
1018:       case RISCVInterruptAttr::SiFiveCLICStackSwap: {
1019:         HasSiFiveCLICStackSwap = true;
1020:         Kind = HasSiFiveCLICPreemptible ? "SiFive-CLIC-preemptible-stack-swap"
```
- **EN**: This block uses control flow (case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 1021-1037
```cpp
1021:                                         : "SiFive-CLIC-stack-swap";
1022:         break;
1023:       }
1024:       }
1025:     }
1026: 
1027:     Fn->addFnAttr("interrupt", Kind);
1028:   }
1029: };
1030: } // namespace
1031: 
1032: std::unique_ptr<TargetCodeGenInfo>
1033: CodeGen::createRISCVTargetCodeGenInfo(CodeGenModule &CGM, unsigned XLen,
1034:                                       unsigned FLen, bool EABI) {
1035:   return std::make_unique<RISCVTargetCodeGenInfo>(CGM.getTypes(), XLen, FLen,
1036:                                                   EABI);
1037: }
```
- **EN**: This block opens or references namespaces `std`; defines callable entry points like `createRISCVTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块打开或引用命名空间 `std`；定义可调用入口，例如 `createRISCVTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CharUnits**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Field1Ty**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **XLen**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **EltTy**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Field2Ty**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getContext**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `TargetInfo.h`
- **LLVM libraries / LLVM 库**: `llvm/IR/IntrinsicsRISCV.h`, `llvm/TargetParser/RISCVTargetParser.h`
