# Mips.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/Mips.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for Mips.
- **Purpose (CN) / 目的（中文）**: 实现 Mips 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: //===- Mips.cpp -----------------------------------------------------------===//
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
15: //===----------------------------------------------------------------------===//
16: // MIPS ABI Implementation.  This works for both little-endian and
```
- **EN**: This block imports local CodeGen headers `ABIInfoImpl.h`, `TargetInfo.h`; opens or references namespaces `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfoImpl.h`, `TargetInfo.h`；打开或引用命名空间 `clang`；包含影响本编译单元构建方式的预处理结构。

### Lines 17-32
```cpp
17: // big-endian variants.
18: //===----------------------------------------------------------------------===//
19: 
20: namespace {
21: class MipsABIInfo : public ABIInfo {
22:   bool IsO32;
23:   const unsigned MinABIStackAlignInBytes, StackAlignInBytes;
24:   void CoerceToIntArgs(uint64_t TySize,
25:                        SmallVectorImpl<llvm::Type *> &ArgList) const;
26:   llvm::Type* HandleAggregates(QualType Ty, uint64_t TySize) const;
27:   llvm::Type* returnAggregateInRegs(QualType RetTy, uint64_t Size) const;
28:   llvm::Type* getPaddingType(uint64_t Align, uint64_t Offset) const;
29: public:
30:   MipsABIInfo(CodeGenTypes &CGT, bool _IsO32) :
31:     ABIInfo(CGT), IsO32(_IsO32), MinABIStackAlignInBytes(IsO32 ? 4 : 8),
32:     StackAlignInBytes(IsO32 ? 8 : 16) {}
```
- **EN**: This block introduces declarations such as `MipsABIInfo`; defines callable entry points like `CoerceToIntArgs`, `HandleAggregates`, `returnAggregateInRegs`, `getPaddingType`, `MipsABIInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块给出诸如 `MipsABIInfo` 的声明；定义可调用入口，例如 `CoerceToIntArgs`, `HandleAggregates`, `returnAggregateInRegs`, `getPaddingType`, `MipsABIInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 33-48
```cpp
33: 
34:   ABIArgInfo classifyReturnType(QualType RetTy) const;
35:   ABIArgInfo classifyArgumentType(QualType RetTy, uint64_t &Offset) const;
36:   void computeInfo(CGFunctionInfo &FI) const override;
37:   RValue EmitVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
38:                    AggValueSlot Slot) const override;
39:   ABIArgInfo extendType(QualType Ty) const;
40: };
41: 
42: class MIPSTargetCodeGenInfo : public TargetCodeGenInfo {
43:   unsigned SizeOfUnwindException;
44: public:
45:   MIPSTargetCodeGenInfo(CodeGenTypes &CGT, bool IsO32)
46:       : TargetCodeGenInfo(std::make_unique<MipsABIInfo>(CGT, IsO32)),
47:         SizeOfUnwindException(IsO32 ? 24 : 32) {}
48: 
```
- **EN**: This block introduces declarations such as `MIPSTargetCodeGenInfo`; defines callable entry points like `classifyReturnType`, `classifyArgumentType`, `computeInfo`, `EmitVAArg`, `extendType`.
- **CN**: 该代码块给出诸如 `MIPSTargetCodeGenInfo` 的声明；定义可调用入口，例如 `classifyReturnType`, `classifyArgumentType`, `computeInfo`, `EmitVAArg`, `extendType`。

### Lines 49-64
```cpp
49:   int getDwarfEHStackPointer(CodeGen::CodeGenModule &CGM) const override {
50:     return 29;
51:   }
52: 
53:   void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
54:                            CodeGen::CodeGenModule &CGM) const override {
55:     const FunctionDecl *FD = dyn_cast_or_null<FunctionDecl>(D);
56:     if (!FD) return;
57:     llvm::Function *Fn = cast<llvm::Function>(GV);
58: 
59:     if (FD->hasAttr<MipsLongCallAttr>())
60:       Fn->addFnAttr("long-call");
61:     else if (FD->hasAttr<MipsShortCallAttr>())
62:       Fn->addFnAttr("short-call");
63: 
64:     // Other attributes do not have a meaning for declarations.
```
- **EN**: This block defines callable entry points like `getDwarfEHStackPointer`, `setTargetAttributes`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDwarfEHStackPointer`, `setTargetAttributes`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 65-80
```cpp
65:     if (GV->isDeclaration())
66:       return;
67: 
68:     if (FD->hasAttr<Mips16Attr>()) {
69:       Fn->addFnAttr("mips16");
70:     }
71:     else if (FD->hasAttr<NoMips16Attr>()) {
72:       Fn->addFnAttr("nomips16");
73:     }
74: 
75:     if (FD->hasAttr<MicroMipsAttr>())
76:       Fn->addFnAttr("micromips");
77:     else if (FD->hasAttr<NoMicroMipsAttr>())
78:       Fn->addFnAttr("nomicromips");
79: 
80:     const MipsInterruptAttr *Attr = FD->getAttr<MipsInterruptAttr>();
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 81-96
```cpp
81:     if (!Attr)
82:       return;
83: 
84:     const char *Kind;
85:     switch (Attr->getInterrupt()) {
86:     case MipsInterruptAttr::eic:     Kind = "eic"; break;
87:     case MipsInterruptAttr::sw0:     Kind = "sw0"; break;
88:     case MipsInterruptAttr::sw1:     Kind = "sw1"; break;
89:     case MipsInterruptAttr::hw0:     Kind = "hw0"; break;
90:     case MipsInterruptAttr::hw1:     Kind = "hw1"; break;
91:     case MipsInterruptAttr::hw2:     Kind = "hw2"; break;
92:     case MipsInterruptAttr::hw3:     Kind = "hw3"; break;
93:     case MipsInterruptAttr::hw4:     Kind = "hw4"; break;
94:     case MipsInterruptAttr::hw5:     Kind = "hw5"; break;
95:     }
96: 
```
- **EN**: This block uses control flow (if, switch, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 97-112
```cpp
 97:     Fn->addFnAttr("interrupt", Kind);
 98: 
 99:   }
100: 
101:   bool initDwarfEHRegSizeTable(CodeGen::CodeGenFunction &CGF,
102:                                llvm::Value *Address) const override;
103: 
104:   unsigned getSizeOfUnwindException() const override {
105:     return SizeOfUnwindException;
106:   }
107: };
108: 
109: class WindowsMIPSTargetCodeGenInfo : public MIPSTargetCodeGenInfo {
110: public:
111:   WindowsMIPSTargetCodeGenInfo(CodeGenTypes &CGT, bool IsO32)
112:       : MIPSTargetCodeGenInfo(CGT, IsO32) {}
```
- **EN**: This block introduces declarations such as `WindowsMIPSTargetCodeGenInfo`; defines callable entry points like `initDwarfEHRegSizeTable`, `getSizeOfUnwindException`, `WindowsMIPSTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块给出诸如 `WindowsMIPSTargetCodeGenInfo` 的声明；定义可调用入口，例如 `initDwarfEHRegSizeTable`, `getSizeOfUnwindException`, `WindowsMIPSTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 113-128
```cpp
113: 
114:   void getDependentLibraryOption(llvm::StringRef Lib,
115:                                  llvm::SmallString<24> &Opt) const override {
116:     Opt = "/DEFAULTLIB:";
117:     Opt += qualifyWindowsLibrary(Lib);
118:   }
119: 
120:   void getDetectMismatchOption(llvm::StringRef Name, llvm::StringRef Value,
121:                                llvm::SmallString<32> &Opt) const override {
122:     Opt = "/FAILIFMISMATCH:\"" + Name.str() + "=" + Value.str() + "\"";
123:   }
124: };
125: }
126: 
127: void MipsABIInfo::CoerceToIntArgs(
128:     uint64_t TySize, SmallVectorImpl<llvm::Type *> &ArgList) const {
```
- **EN**: This block defines callable entry points like `getDependentLibraryOption`, `getDetectMismatchOption`, `CoerceToIntArgs`.
- **CN**: 该代码块定义可调用入口，例如 `getDependentLibraryOption`, `getDetectMismatchOption`, `CoerceToIntArgs`。

### Lines 129-144
```cpp
129:   llvm::IntegerType *IntTy =
130:     llvm::IntegerType::get(getVMContext(), MinABIStackAlignInBytes * 8);
131: 
132:   // Add (TySize / MinABIStackAlignInBytes) args of IntTy.
133:   for (unsigned N = TySize / (MinABIStackAlignInBytes * 8); N; --N)
134:     ArgList.push_back(IntTy);
135: 
136:   // If necessary, add one more integer type to ArgList.
137:   unsigned R = TySize % (MinABIStackAlignInBytes * 8);
138: 
139:   if (R)
140:     ArgList.push_back(llvm::IntegerType::get(getVMContext(), R));
141: }
142: 
143: // In N32/64, an aligned double precision floating point field is passed in
144: // a register.
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 145-160
```cpp
145: llvm::Type* MipsABIInfo::HandleAggregates(QualType Ty, uint64_t TySize) const {
146:   SmallVector<llvm::Type*, 8> ArgList, IntArgList;
147: 
148:   if (IsO32) {
149:     CoerceToIntArgs(TySize, ArgList);
150:     return llvm::StructType::get(getVMContext(), ArgList);
151:   }
152: 
153:   if (Ty->isComplexType())
154:     return CGT.ConvertType(Ty);
155: 
156:   const RecordType *RT = Ty->getAsCanonical<RecordType>();
157: 
158:   // Unions/vectors are passed in integer registers.
159:   if (!RT || !RT->isStructureOrClassType()) {
160:     CoerceToIntArgs(TySize, ArgList);
```
- **EN**: This block defines callable entry points like `HandleAggregates`, `CoerceToIntArgs`, `get`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `HandleAggregates`, `CoerceToIntArgs`, `get`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 161-176
```cpp
161:     return llvm::StructType::get(getVMContext(), ArgList);
162:   }
163: 
164:   const RecordDecl *RD = RT->getDecl()->getDefinitionOrSelf();
165:   const ASTRecordLayout &Layout = getContext().getASTRecordLayout(RD);
166:   assert(!(TySize % 8) && "Size of structure must be multiple of 8.");
167: 
168:   uint64_t LastOffset = 0;
169:   unsigned idx = 0;
170:   llvm::IntegerType *I64 = llvm::IntegerType::get(getVMContext(), 64);
171: 
172:   // Iterate over fields in the struct/class and check if there are any aligned
173:   // double fields.
174:   for (RecordDecl::field_iterator i = RD->field_begin(), e = RD->field_end();
175:        i != e; ++i, ++idx) {
176:     const QualType Ty = i->getType();
```
- **EN**: This block defines callable entry points like `get`; uses control flow (for) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（for）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 177-192
```cpp
177:     const BuiltinType *BT = Ty->getAs<BuiltinType>();
178: 
179:     if (!BT || BT->getKind() != BuiltinType::Double)
180:       continue;
181: 
182:     uint64_t Offset = Layout.getFieldOffset(idx);
183:     if (Offset % 64) // Ignore doubles that are not aligned.
184:       continue;
185: 
186:     // Add ((Offset - LastOffset) / 64) args of type i64.
187:     for (unsigned j = (Offset - LastOffset) / 64; j > 0; --j)
188:       ArgList.push_back(I64);
189: 
190:     // Add double type.
191:     ArgList.push_back(llvm::Type::getDoubleTy(getVMContext()));
192:     LastOffset = Offset + 64;
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 193-208
```cpp
193:   }
194: 
195:   CoerceToIntArgs(TySize - LastOffset, IntArgList);
196:   ArgList.append(IntArgList.begin(), IntArgList.end());
197: 
198:   return llvm::StructType::get(getVMContext(), ArgList);
199: }
200: 
201: llvm::Type *MipsABIInfo::getPaddingType(uint64_t OrigOffset,
202:                                         uint64_t Offset) const {
203:   if (OrigOffset + MinABIStackAlignInBytes > Offset)
204:     return nullptr;
205: 
206:   return llvm::IntegerType::get(getVMContext(), (Offset - OrigOffset) * 8);
207: }
208: 
```
- **EN**: This block defines callable entry points like `CoerceToIntArgs`, `get`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `CoerceToIntArgs`, `get`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 209-224
```cpp
209: ABIArgInfo
210: MipsABIInfo::classifyArgumentType(QualType Ty, uint64_t &Offset) const {
211:   Ty = useFirstFieldIfTransparentUnion(Ty);
212: 
213:   uint64_t OrigOffset = Offset;
214:   uint64_t TySize = getContext().getTypeSize(Ty);
215:   uint64_t Align = getContext().getTypeAlign(Ty) / 8;
216: 
217:   Align = std::clamp(Align, (uint64_t)MinABIStackAlignInBytes,
218:                      (uint64_t)StackAlignInBytes);
219:   unsigned CurrOffset = llvm::alignTo(Offset, Align);
220:   Offset = CurrOffset + llvm::alignTo(TySize, Align * 8) / 8;
221: 
222:   if (isAggregateTypeForABI(Ty) || Ty->isVectorType()) {
223:     // Ignore empty aggregates.
224:     if (TySize == 0)
```
- **EN**: This block defines callable entry points like `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 225-240
```cpp
225:       return ABIArgInfo::getIgnore();
226: 
227:     if (CGCXXABI::RecordArgABI RAA = getRecordArgABI(Ty, getCXXABI())) {
228:       Offset = OrigOffset + MinABIStackAlignInBytes;
229:       return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
230:                                      RAA == CGCXXABI::RAA_DirectInMemory);
231:     }
232: 
233:     // If we have reached here, aggregates are passed directly by coercing to
234:     // another structure type. Padding is inserted if the offset of the
235:     // aggregate is unaligned.
236:     ABIArgInfo ArgInfo =
237:         ABIArgInfo::getDirect(HandleAggregates(Ty, TySize), 0,
238:                               getPaddingType(OrigOffset, CurrOffset));
239:     ArgInfo.setInReg(true);
240:     return ArgInfo;
```
- **EN**: This block defines callable entry points like `getIgnore`, `getNaturalAlignIndirect`, `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getIgnore`, `getNaturalAlignIndirect`, `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 241-256
```cpp
241:   }
242: 
243:   // Treat an enum type as its underlying type.
244:   if (const auto *ED = Ty->getAsEnumDecl())
245:     Ty = ED->getIntegerType();
246: 
247:   // Make sure we pass indirectly things that are too large.
248:   if (const auto *EIT = Ty->getAs<BitIntType>())
249:     if (EIT->getNumBits() > 128 ||
250:         (EIT->getNumBits() > 64 &&
251:          !getContext().getTargetInfo().hasInt128Type()))
252:       return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace());
253: 
254:   // All integral types are promoted to the GPR width.
255:   if (Ty->isIntegralOrEnumerationType())
256:     return extendType(Ty);
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 257-272
```cpp
257: 
258:   return ABIArgInfo::getDirect(
259:       nullptr, 0, IsO32 ? nullptr : getPaddingType(OrigOffset, CurrOffset));
260: }
261: 
262: llvm::Type*
263: MipsABIInfo::returnAggregateInRegs(QualType RetTy, uint64_t Size) const {
264:   const RecordType *RT = RetTy->getAsCanonical<RecordType>();
265:   SmallVector<llvm::Type*, 8> RTList;
266: 
267:   if (RT && RT->isStructureOrClassType()) {
268:     const RecordDecl *RD = RT->getDecl()->getDefinitionOrSelf();
269:     const ASTRecordLayout &Layout = getContext().getASTRecordLayout(RD);
270:     unsigned FieldCnt = Layout.getFieldCount();
271: 
272:     // N32/64 returns struct/classes in floating point registers if the
```
- **EN**: This block defines callable entry points like `getDirect`, `returnAggregateInRegs`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `returnAggregateInRegs`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 273-288
```cpp
273:     // following conditions are met:
274:     // 1. The size of the struct/class is no larger than 128-bit.
275:     // 2. The struct/class has one or two fields all of which are floating
276:     //    point types.
277:     // 3. The offset of the first field is zero (this follows what gcc does).
278:     //
279:     // Any other composite results are returned in integer registers.
280:     //
281:     if (FieldCnt && (FieldCnt <= 2) && !Layout.getFieldOffset(0)) {
282:       RecordDecl::field_iterator b = RD->field_begin(), e = RD->field_end();
283:       for (; b != e; ++b) {
284:         const BuiltinType *BT = b->getType()->getAs<BuiltinType>();
285: 
286:         if (!BT || !BT->isFloatingPoint())
287:           break;
288: 
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 289-304
```cpp
289:         RTList.push_back(CGT.ConvertType(b->getType()));
290:       }
291: 
292:       if (b == e)
293:         return llvm::StructType::get(getVMContext(), RTList,
294:                                      RD->hasAttr<PackedAttr>());
295: 
296:       RTList.clear();
297:     }
298:   }
299: 
300:   CoerceToIntArgs(Size, RTList);
301:   return llvm::StructType::get(getVMContext(), RTList);
302: }
303: 
304: ABIArgInfo MipsABIInfo::classifyReturnType(QualType RetTy) const {
```
- **EN**: This block defines callable entry points like `CoerceToIntArgs`, `get`, `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `CoerceToIntArgs`, `get`, `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 305-320
```cpp
305:   uint64_t Size = getContext().getTypeSize(RetTy);
306: 
307:   if (RetTy->isVoidType())
308:     return ABIArgInfo::getIgnore();
309: 
310:   // O32 doesn't treat zero-sized structs differently from other structs.
311:   // However, N32/N64 ignores zero sized return values.
312:   if (!IsO32 && Size == 0)
313:     return ABIArgInfo::getIgnore();
314: 
315:   if (isAggregateTypeForABI(RetTy) || RetTy->isVectorType()) {
316:     if (Size <= 128) {
317:       if (RetTy->isAnyComplexType())
318:         return ABIArgInfo::getDirect();
319: 
320:       // O32 returns integer vectors in registers and N32/N64 returns all small
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 321-336
```cpp
321:       // aggregates in registers.
322:       if (!IsO32 ||
323:           (RetTy->isVectorType() && !RetTy->hasFloatingRepresentation())) {
324:         ABIArgInfo ArgInfo =
325:             ABIArgInfo::getDirect(returnAggregateInRegs(RetTy, Size));
326:         ArgInfo.setInReg(true);
327:         return ArgInfo;
328:       }
329:     }
330: 
331:     return getNaturalAlignIndirect(RetTy, getDataLayout().getAllocaAddrSpace());
332:   }
333: 
334:   // Treat an enum type as its underlying type.
335:   if (const auto *ED = RetTy->getAsEnumDecl())
336:     RetTy = ED->getIntegerType();
```
- **EN**: This block defines callable entry points like `getDirect`, `getNaturalAlignIndirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `getNaturalAlignIndirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 337-352
```cpp
337: 
338:   // Make sure we pass indirectly things that are too large.
339:   if (const auto *EIT = RetTy->getAs<BitIntType>())
340:     if (EIT->getNumBits() > 128 ||
341:         (EIT->getNumBits() > 64 &&
342:          !getContext().getTargetInfo().hasInt128Type()))
343:       return getNaturalAlignIndirect(RetTy,
344:                                      getDataLayout().getAllocaAddrSpace());
345: 
346:   if (isPromotableIntegerTypeForABI(RetTy))
347:     return ABIArgInfo::getExtend(RetTy);
348: 
349:   if ((RetTy->isUnsignedIntegerOrEnumerationType() ||
350:       RetTy->isSignedIntegerOrEnumerationType()) && Size == 32 && !IsO32)
351:     return ABIArgInfo::getSignExtend(RetTy);
352: 
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 353-368
```cpp
353:   return ABIArgInfo::getDirect();
354: }
355: 
356: void MipsABIInfo::computeInfo(CGFunctionInfo &FI) const {
357:   ABIArgInfo &RetInfo = FI.getReturnInfo();
358:   if (!getCXXABI().classifyReturnType(FI))
359:     RetInfo = classifyReturnType(FI.getReturnType());
360: 
361:   // Check if a pointer to an aggregate is passed as a hidden argument.
362:   uint64_t Offset = RetInfo.isIndirect() ? MinABIStackAlignInBytes : 0;
363: 
364:   for (auto &I : FI.arguments())
365:     I.info = classifyArgumentType(I.type, Offset);
366: }
367: 
368: RValue MipsABIInfo::EmitVAArg(CodeGenFunction &CGF, Address VAListAddr,
```
- **EN**: This block defines callable entry points like `getDirect`, `computeInfo`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `computeInfo`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 369-384
```cpp
369:                               QualType OrigTy, AggValueSlot Slot) const {
370:   QualType Ty = OrigTy;
371: 
372:   // Integer arguments are promoted to 32-bit on O32 and 64-bit on N32/N64.
373:   // Pointers are also promoted in the same way but this only matters for N32.
374:   unsigned SlotSizeInBits = IsO32 ? 32 : 64;
375:   unsigned PtrWidth = getTarget().getPointerWidth(LangAS::Default);
376:   bool DidPromote = false;
377:   if ((Ty->isIntegerType() &&
378:           getContext().getIntWidth(Ty) < SlotSizeInBits) ||
379:       (Ty->isPointerType() && PtrWidth < SlotSizeInBits)) {
380:     DidPromote = true;
381:     Ty = getContext().getIntTypeForBitwidth(SlotSizeInBits,
382:                                             Ty->isSignedIntegerType());
383:   }
384: 
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 385-400
```cpp
385:   auto TyInfo = getContext().getTypeInfoInChars(Ty);
386: 
387:   // The alignment of things in the argument area is never larger than
388:   // StackAlignInBytes.
389:   TyInfo.Align =
390:     std::min(TyInfo.Align, CharUnits::fromQuantity(StackAlignInBytes));
391: 
392:   // MinABIStackAlignInBytes is the size of argument slots on the stack.
393:   CharUnits ArgSlotSize = CharUnits::fromQuantity(MinABIStackAlignInBytes);
394: 
395:   RValue Res = emitVoidPtrVAArg(CGF, VAListAddr, Ty, /*indirect*/ false, TyInfo,
396:                                 ArgSlotSize, /*AllowHigherAlign*/ true, Slot);
397: 
398:   // If there was a promotion, "unpromote".
399:   // TODO: can we just use a pointer into a subset of the original slot?
400:   if (DidPromote) {
```
- **EN**: This block defines callable entry points like `min`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `min`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 401-416
```cpp
401:     llvm::Type *ValTy = CGF.ConvertType(OrigTy);
402:     llvm::Value *Promoted = Res.getScalarVal();
403: 
404:     // Truncate down to the right width.
405:     llvm::Type *IntTy = (OrigTy->isIntegerType() ? ValTy : CGF.IntPtrTy);
406:     llvm::Value *V = CGF.Builder.CreateTrunc(Promoted, IntTy);
407:     if (OrigTy->isPointerType())
408:       V = CGF.Builder.CreateIntToPtr(V, ValTy);
409: 
410:     return RValue::get(V);
411:   }
412: 
413:   return Res;
414: }
415: 
416: ABIArgInfo MipsABIInfo::extendType(QualType Ty) const {
```
- **EN**: This block defines callable entry points like `get`, `extendType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `get`, `extendType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 417-432
```cpp
417:   int TySize = getContext().getTypeSize(Ty);
418: 
419:   // MIPS64 ABI requires unsigned 32 bit integers to be sign extended.
420:   if (Ty->isUnsignedIntegerOrEnumerationType() && TySize == 32)
421:     return ABIArgInfo::getSignExtend(Ty);
422: 
423:   return ABIArgInfo::getExtend(Ty);
424: }
425: 
426: bool
427: MIPSTargetCodeGenInfo::initDwarfEHRegSizeTable(CodeGen::CodeGenFunction &CGF,
428:                                                llvm::Value *Address) const {
429:   // This information comes from gcc's implementation, which seems to
430:   // as canonical as it gets.
431: 
432:   // Everything on MIPS is 4 bytes.  Double-precision FP registers
```
- **EN**: This block defines callable entry points like `getExtend`, `initDwarfEHRegSizeTable`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getExtend`, `initDwarfEHRegSizeTable`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 433-448
```cpp
433:   // are aliased to pairs of single-precision FP registers.
434:   llvm::Value *Four8 = llvm::ConstantInt::get(CGF.Int8Ty, 4);
435: 
436:   // 0-31 are the general purpose registers, $0 - $31.
437:   // 32-63 are the floating-point registers, $f0 - $f31.
438:   // 64 and 65 are the multiply/divide registers, $hi and $lo.
439:   // 66 is the (notional, I think) register for signal-handler return.
440:   AssignToArrayRange(CGF.Builder, Address, Four8, 0, 65);
441: 
442:   // 67-74 are the floating-point status registers, $fcc0 - $fcc7.
443:   // They are one bit wide and ignored here.
444: 
445:   // 80-111 are the coprocessor 0 registers, $c0r0 - $c0r31.
446:   // (coprocessor 1 is the FP unit)
447:   // 112-143 are the coprocessor 2 registers, $c2r0 - $c2r31.
448:   // 144-175 are the coprocessor 3 registers, $c3r0 - $c3r31.
```
- **EN**: This block spells out callable entry points like `AssignToArrayRange`.
- **CN**: 该代码块给出可调用入口的声明，例如 `AssignToArrayRange`。

### Lines 449-462
```cpp
449:   // 176-181 are the DSP accumulator registers.
450:   AssignToArrayRange(CGF.Builder, Address, Four8, 80, 181);
451:   return false;
452: }
453: 
454: std::unique_ptr<TargetCodeGenInfo>
455: CodeGen::createMIPSTargetCodeGenInfo(CodeGenModule &CGM, bool IsOS32) {
456:   return std::make_unique<MIPSTargetCodeGenInfo>(CGM.getTypes(), IsOS32);
457: }
458: 
459: std::unique_ptr<TargetCodeGenInfo>
460: CodeGen::createWindowsMIPSTargetCodeGenInfo(CodeGenModule &CGM, bool IsOS32) {
461:   return std::make_unique<WindowsMIPSTargetCodeGenInfo>(CGM.getTypes(), IsOS32);
462: }
```
- **EN**: This block defines callable entry points like `AssignToArrayRange`, `createMIPSTargetCodeGenInfo`, `createWindowsMIPSTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `AssignToArrayRange`, `createMIPSTargetCodeGenInfo`, `createWindowsMIPSTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **RetTy**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **TySize**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Offset**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **IsO32**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **QualType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ArgList**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `TargetInfo.h`
