# SystemZ.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/SystemZ.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for SystemZ.
- **Purpose (CN) / 目的（中文）**: 实现 SystemZ 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: //===- SystemZ.cpp --------------------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: #include "ABIInfoImpl.h"
10: #include "TargetInfo.h"
11: #include "clang/Basic/Builtins.h"
12: #include "llvm/IR/IntrinsicsS390.h"
13: 
14: using namespace clang;
15: using namespace clang::CodeGen;
16: 
```
- **EN**: This block imports local CodeGen headers `ABIInfoImpl.h`, `TargetInfo.h`; Clang headers `clang/Basic/Builtins.h`; LLVM headers `llvm/IR/IntrinsicsS390.h`; opens or references namespaces `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfoImpl.h`, `TargetInfo.h`；Clang 头文件 `clang/Basic/Builtins.h`；LLVM 头文件 `llvm/IR/IntrinsicsS390.h`；打开或引用命名空间 `clang`；包含影响本编译单元构建方式的预处理结构。

### Lines 17-32
```cpp
17: //===----------------------------------------------------------------------===//
18: // SystemZ ABI Implementation
19: //===----------------------------------------------------------------------===//
20: 
21: namespace {
22: 
23: class SystemZABIInfo : public ABIInfo {
24:   bool HasVector;
25:   bool IsSoftFloatABI;
26: 
27: public:
28:   SystemZABIInfo(CodeGenTypes &CGT, bool HV, bool SF)
29:       : ABIInfo(CGT), HasVector(HV), IsSoftFloatABI(SF) {}
30: 
31:   bool isPromotableIntegerTypeForABI(QualType Ty) const;
32:   bool isCompoundType(QualType Ty) const;
```
- **EN**: This block introduces declarations such as `SystemZABIInfo`; defines callable entry points like `SystemZABIInfo`, `isPromotableIntegerTypeForABI`, `isCompoundType`.
- **CN**: 该代码块给出诸如 `SystemZABIInfo` 的声明；定义可调用入口，例如 `SystemZABIInfo`, `isPromotableIntegerTypeForABI`, `isCompoundType`。

### Lines 33-48
```cpp
33:   bool isVectorArgumentType(QualType Ty) const;
34:   llvm::Type *getFPArgumentType(QualType Ty, uint64_t Size) const;
35:   QualType GetSingleElementType(QualType Ty) const;
36: 
37:   ABIArgInfo classifyReturnType(QualType RetTy) const;
38:   ABIArgInfo classifyArgumentType(QualType ArgTy) const;
39: 
40:   void computeInfo(CGFunctionInfo &FI) const override;
41:   RValue EmitVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
42:                    AggValueSlot Slot) const override;
43: };
44: 
45: class SystemZTargetCodeGenInfo : public TargetCodeGenInfo {
46:   ASTContext &Ctx;
47: 
48:   // These are used for speeding up the search for a visible vector ABI.
```
- **EN**: This block introduces declarations such as `SystemZTargetCodeGenInfo`; defines callable entry points like `isVectorArgumentType`, `GetSingleElementType`, `classifyReturnType`, `classifyArgumentType`, `computeInfo`.
- **CN**: 该代码块给出诸如 `SystemZTargetCodeGenInfo` 的声明；定义可调用入口，例如 `isVectorArgumentType`, `GetSingleElementType`, `classifyReturnType`, `classifyArgumentType`, `computeInfo`。

### Lines 49-64
```cpp
49:   mutable bool HasVisibleVecABIFlag = false;
50:   mutable std::set<const Type *> SeenTypes;
51: 
52:   // Returns true (the first time) if Ty is, or is found to include, a vector
53:   // type that exposes the vector ABI. This is any vector >=16 bytes which
54:   // with vector support are aligned to only 8 bytes. When IsParam is true,
55:   // the type belongs to a value as passed between functions. If it is a
56:   // vector <=16 bytes it will be passed in a vector register (if supported).
57:   bool isVectorTypeBased(const Type *Ty, bool IsParam) const;
58: 
59: public:
60:   SystemZTargetCodeGenInfo(CodeGenTypes &CGT, bool HasVector, bool SoftFloatABI)
61:       : TargetCodeGenInfo(
62:             std::make_unique<SystemZABIInfo>(CGT, HasVector, SoftFloatABI)),
63:             Ctx(CGT.getContext()) {
64:     SwiftInfo =
```
- **EN**: This block defines callable entry points like `isVectorTypeBased`, `SystemZTargetCodeGenInfo`.
- **CN**: 该代码块定义可调用入口，例如 `isVectorTypeBased`, `SystemZTargetCodeGenInfo`。

### Lines 65-80
```cpp
65:         std::make_unique<SwiftABIInfo>(CGT, /*SwiftErrorInRegister=*/false);
66:   }
67: 
68:   // The vector ABI is different when the vector facility is present and when
69:   // a module e.g. defines an externally visible vector variable, a flag
70:   // indicating a visible vector ABI is added. Eventually this will result in
71:   // a GNU attribute indicating the vector ABI of the module.  Ty is the type
72:   // of a variable or function parameter that is globally visible.
73:   void handleExternallyVisibleObjABI(const Type *Ty, CodeGen::CodeGenModule &M,
74:                                      bool IsParam) const {
75:     if (!HasVisibleVecABIFlag && isVectorTypeBased(Ty, IsParam)) {
76:       M.getModule().addModuleFlag(llvm::Module::Warning,
77:                                   "s390x-visible-vector-ABI", 1);
78:       HasVisibleVecABIFlag = true;
79:     }
80:   }
```
- **EN**: This block defines callable entry points like `handleExternallyVisibleObjABI`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `handleExternallyVisibleObjABI`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 81-96
```cpp
81: 
82:   void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
83:                            CodeGen::CodeGenModule &M) const override {
84:     if (!D)
85:       return;
86: 
87:     // Check if the vector ABI becomes visible by an externally visible
88:     // variable or function.
89:     if (const auto *VD = dyn_cast<VarDecl>(D)) {
90:       if (VD->isExternallyVisible())
91:         handleExternallyVisibleObjABI(VD->getType().getTypePtr(), M,
92:                                       /*IsParam*/false);
93:     }
94:     else if (const FunctionDecl *FD = dyn_cast<FunctionDecl>(D)) {
95:       if (FD->isExternallyVisible())
96:         handleExternallyVisibleObjABI(FD->getType().getTypePtr(), M,
```
- **EN**: This block defines callable entry points like `setTargetAttributes`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `setTargetAttributes`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 97-112
```cpp
 97:                                       /*IsParam*/false);
 98:     }
 99:   }
100: 
101:   llvm::Value *testFPKind(llvm::Value *V, unsigned BuiltinID,
102:                           CGBuilderTy &Builder,
103:                           CodeGenModule &CGM) const override {
104:     assert(V->getType()->isFloatingPointTy() && "V should have an FP type.");
105:     // Only use TDC in constrained FP mode.
106:     if (!Builder.getIsFPConstrained())
107:       return nullptr;
108: 
109:     llvm::Type *Ty = V->getType();
110:     if (Ty->isHalfTy() || Ty->isFloatTy() || Ty->isDoubleTy() ||
111:         Ty->isFP128Ty()) {
112:       llvm::Module &M = CGM.getModule();
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 113-128
```cpp
113:       auto &Ctx = M.getContext();
114:       llvm::Function *TDCFunc = llvm::Intrinsic::getOrInsertDeclaration(
115:           &M, llvm::Intrinsic::s390_tdc, Ty);
116:       unsigned TDCBits = 0;
117:       switch (BuiltinID) {
118:       case Builtin::BI__builtin_isnan:
119:         TDCBits = 0xf;
120:         break;
121:       case Builtin::BIfinite:
122:       case Builtin::BI__finite:
123:       case Builtin::BIfinitef:
124:       case Builtin::BI__finitef:
125:       case Builtin::BIfinitel:
126:       case Builtin::BI__finitel:
127:       case Builtin::BI__builtin_isfinite:
128:         TDCBits = 0xfc0;
```
- **EN**: This block uses control flow (switch, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（switch, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 129-144
```cpp
129:         break;
130:       case Builtin::BI__builtin_isinf:
131:         TDCBits = 0x30;
132:         break;
133:       default:
134:         break;
135:       }
136:       if (TDCBits)
137:         return Builder.CreateCall(
138:             TDCFunc,
139:             {V, llvm::ConstantInt::get(llvm::Type::getInt64Ty(Ctx), TDCBits)});
140:     }
141:     return nullptr;
142:   }
143: };
144: }
```
- **EN**: This block uses control flow (if, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 145-160
```cpp
145: 
146: bool SystemZABIInfo::isPromotableIntegerTypeForABI(QualType Ty) const {
147:   // Treat an enum type as its underlying type.
148:   if (const auto *ED = Ty->getAsEnumDecl())
149:     Ty = ED->getIntegerType();
150: 
151:   // Promotable integer types are required to be promoted by the ABI.
152:   if (ABIInfo::isPromotableIntegerTypeForABI(Ty))
153:     return true;
154: 
155:   if (const auto *EIT = Ty->getAs<BitIntType>())
156:     if (EIT->getNumBits() < 64)
157:       return true;
158: 
159:   // 32-bit values must also be promoted.
160:   if (const BuiltinType *BT = Ty->getAs<BuiltinType>())
```
- **EN**: This block defines callable entry points like `isPromotableIntegerTypeForABI`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isPromotableIntegerTypeForABI`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 161-176
```cpp
161:     switch (BT->getKind()) {
162:     case BuiltinType::Int:
163:     case BuiltinType::UInt:
164:       return true;
165:     default:
166:       return false;
167:     }
168:   return false;
169: }
170: 
171: bool SystemZABIInfo::isCompoundType(QualType Ty) const {
172:   return (Ty->isAnyComplexType() ||
173:           Ty->isVectorType() ||
174:           isAggregateTypeForABI(Ty));
175: }
176: 
```
- **EN**: This block defines callable entry points like `isCompoundType`; uses control flow (switch, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isCompoundType`；通过控制流（switch, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 177-192
```cpp
177: bool SystemZABIInfo::isVectorArgumentType(QualType Ty) const {
178:   return (HasVector &&
179:           Ty->isVectorType() &&
180:           getContext().getTypeSize(Ty) <= 128);
181: }
182: 
183: // The Size argument will in case of af an overaligned single element struct
184: // reflect the overalignment value. In such a case the argument will be
185: // passed using the type matching Size.
186: llvm::Type *SystemZABIInfo::getFPArgumentType(QualType Ty,
187:                                               uint64_t Size) const {
188:   if (IsSoftFloatABI)
189:     return nullptr;
190: 
191:   if (const BuiltinType *BT = Ty->getAs<BuiltinType>())
192:     switch (BT->getKind()) {
```
- **EN**: This block defines callable entry points like `isVectorArgumentType`; uses control flow (if, switch) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isVectorArgumentType`；通过控制流（if, switch）细化 目标相关的 ABI 与代码生成 行为。

### Lines 193-208
```cpp
193:     case BuiltinType::Float16:
194:       if (Size == 16)
195:         return llvm::Type::getHalfTy(getVMContext());
196:       [[fallthrough]];
197:     case BuiltinType::Float:
198:       if (Size == 32)
199:         return llvm::Type::getFloatTy(getVMContext());
200:       [[fallthrough]];
201:     case BuiltinType::Double:
202:       return llvm::Type::getDoubleTy(getVMContext());
203:     default:
204:       return nullptr;
205:     }
206: 
207:   return nullptr;
208: }
```
- **EN**: This block spells out callable entry points like `getDoubleTy`; uses control flow (if, case) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出可调用入口的声明，例如 `getDoubleTy`；通过控制流（if, case）细化 目标相关的 ABI 与代码生成 行为。

### Lines 209-224
```cpp
209: 
210: QualType SystemZABIInfo::GetSingleElementType(QualType Ty) const {
211:   const auto *RD = Ty->getAsRecordDecl();
212:   if (RD && RD->isStructureOrClass()) {
213:     QualType Found;
214: 
215:     // If this is a C++ record, check the bases first.
216:     if (const CXXRecordDecl *CXXRD = dyn_cast<CXXRecordDecl>(RD))
217:       if (CXXRD->hasDefinition())
218:         for (const auto &I : CXXRD->bases()) {
219:           QualType Base = I.getType();
220: 
221:           // Empty bases don't affect things either way.
222:           if (isEmptyRecord(getContext(), Base, true))
223:             continue;
224: 
```
- **EN**: This block defines callable entry points like `GetSingleElementType`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `GetSingleElementType`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 225-240
```cpp
225:           if (!Found.isNull())
226:             return Ty;
227:           Found = GetSingleElementType(Base);
228:         }
229: 
230:     // Check the fields.
231:     for (const auto *FD : RD->fields()) {
232:       // Unlike isSingleElementStruct(), empty structure and array fields
233:       // do count.  So do anonymous bitfields that aren't zero-sized.
234: 
235:       // Like isSingleElementStruct(), ignore C++20 empty data members.
236:       if (FD->hasAttr<NoUniqueAddressAttr>() &&
237:           isEmptyRecord(getContext(), FD->getType(), true))
238:         continue;
239: 
240:       // Unlike isSingleElementStruct(), arrays do not count.
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 241-256
```cpp
241:       // Nested structures still do though.
242:       if (!Found.isNull())
243:         return Ty;
244:       Found = GetSingleElementType(FD->getType());
245:     }
246: 
247:     // Unlike isSingleElementStruct(), trailing padding is allowed.
248:     // An 8-byte aligned struct s { float f; } is passed as a double.
249:     if (!Found.isNull())
250:       return Found;
251:   }
252: 
253:   return Ty;
254: }
255: 
256: RValue SystemZABIInfo::EmitVAArg(CodeGenFunction &CGF, Address VAListAddr,
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 257-272
```cpp
257:                                  QualType Ty, AggValueSlot Slot) const {
258:   // Assume that va_list type is correct; should be pointer to LLVM type:
259:   // struct {
260:   //   i64 __gpr;
261:   //   i64 __fpr;
262:   //   i8 *__overflow_arg_area;
263:   //   i8 *__reg_save_area;
264:   // };
265: 
266:   // Every non-vector argument occupies 8 bytes and is passed by preference
267:   // in either GPRs or FPRs.  Vector arguments occupy 8 or 16 bytes and are
268:   // always passed on the stack.
269:   const SystemZTargetCodeGenInfo &SZCGI =
270:       static_cast<const SystemZTargetCodeGenInfo &>(
271:           CGT.getCGM().getTargetCodeGenInfo());
272:   Ty = getContext().getCanonicalType(Ty);
```
- **EN**: This block documents intent or context for the surrounding target-specific ABI and code generation code.
- **CN**: 该代码块说明周围 目标相关的 ABI 与代码生成 代码的意图或上下文。

### Lines 273-288
```cpp
273:   auto TyInfo = getContext().getTypeInfoInChars(Ty);
274:   llvm::Type *ArgTy = CGF.ConvertTypeForMem(Ty);
275:   llvm::Type *DirectTy = ArgTy;
276:   ABIArgInfo AI = classifyArgumentType(Ty);
277:   bool IsIndirect = AI.isIndirect();
278:   bool InFPRs = false;
279:   bool IsVector = false;
280:   CharUnits UnpaddedSize;
281:   CharUnits DirectAlign;
282:   SZCGI.handleExternallyVisibleObjABI(Ty.getTypePtr(), CGT.getCGM(),
283:                                       /*IsParam*/true);
284:   if (IsIndirect) {
285:     DirectTy = llvm::PointerType::getUnqual(DirectTy->getContext());
286:     UnpaddedSize = DirectAlign = CharUnits::fromQuantity(8);
287:   } else {
288:     if (AI.getCoerceToType())
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 289-304
```cpp
289:       ArgTy = AI.getCoerceToType();
290:     InFPRs = (!IsSoftFloatABI &&
291:               (ArgTy->isHalfTy() || ArgTy->isFloatTy() || ArgTy->isDoubleTy()));
292:     IsVector = ArgTy->isVectorTy();
293:     UnpaddedSize = TyInfo.Width;
294:     DirectAlign = TyInfo.Align;
295:   }
296:   CharUnits PaddedSize = CharUnits::fromQuantity(8);
297:   if (IsVector && UnpaddedSize > PaddedSize)
298:     PaddedSize = CharUnits::fromQuantity(16);
299:   assert((UnpaddedSize <= PaddedSize) && "Invalid argument size.");
300: 
301:   CharUnits Padding = (PaddedSize - UnpaddedSize);
302: 
303:   llvm::Type *IndexTy = CGF.Int64Ty;
304:   llvm::Value *PaddedSizeV =
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 305-320
```cpp
305:     llvm::ConstantInt::get(IndexTy, PaddedSize.getQuantity());
306: 
307:   if (IsVector) {
308:     // Work out the address of a vector argument on the stack.
309:     // Vector arguments are always passed in the high bits of a
310:     // single (8 byte) or double (16 byte) stack slot.
311:     Address OverflowArgAreaPtr =
312:         CGF.Builder.CreateStructGEP(VAListAddr, 2, "overflow_arg_area_ptr");
313:     Address OverflowArgArea =
314:         Address(CGF.Builder.CreateLoad(OverflowArgAreaPtr, "overflow_arg_area"),
315:                 CGF.Int8Ty, TyInfo.Align);
316:     Address MemAddr = OverflowArgArea.withElementType(DirectTy);
317: 
318:     // Update overflow_arg_area_ptr pointer
319:     llvm::Value *NewOverflowArgArea = CGF.Builder.CreateGEP(
320:         OverflowArgArea.getElementType(), OverflowArgArea.emitRawPointer(CGF),
```
- **EN**: This block defines callable entry points like `get`, `Address`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `get`, `Address`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 321-336
```cpp
321:         PaddedSizeV, "overflow_arg_area");
322:     CGF.Builder.CreateStore(NewOverflowArgArea, OverflowArgAreaPtr);
323: 
324:     return CGF.EmitLoadOfAnyValue(CGF.MakeAddrLValue(MemAddr, Ty), Slot);
325:   }
326: 
327:   assert(PaddedSize.getQuantity() == 8);
328: 
329:   unsigned MaxRegs, RegCountField, RegSaveIndex;
330:   CharUnits RegPadding;
331:   if (InFPRs) {
332:     MaxRegs = 4; // Maximum of 4 FPR arguments
333:     RegCountField = 1; // __fpr
334:     RegSaveIndex = 16; // save offset for f0
335:     RegPadding = CharUnits(); // floats are passed in the high bits of an FPR
336:   } else {
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 337-352
```cpp
337:     MaxRegs = 5; // Maximum of 5 GPR arguments
338:     RegCountField = 0; // __gpr
339:     RegSaveIndex = 2; // save offset for r2
340:     RegPadding = Padding; // values are passed in the low bits of a GPR
341:   }
342: 
343:   Address RegCountPtr =
344:       CGF.Builder.CreateStructGEP(VAListAddr, RegCountField, "reg_count_ptr");
345:   llvm::Value *RegCount = CGF.Builder.CreateLoad(RegCountPtr, "reg_count");
346:   llvm::Value *MaxRegsV = llvm::ConstantInt::get(IndexTy, MaxRegs);
347:   llvm::Value *InRegs = CGF.Builder.CreateICmpULT(RegCount, MaxRegsV,
348:                                                  "fits_in_regs");
349: 
350:   llvm::BasicBlock *InRegBlock = CGF.createBasicBlock("vaarg.in_reg");
351:   llvm::BasicBlock *InMemBlock = CGF.createBasicBlock("vaarg.in_mem");
352:   llvm::BasicBlock *ContBlock = CGF.createBasicBlock("vaarg.end");
```
- **EN**: This block uses control flow (for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 353-368
```cpp
353:   CGF.Builder.CreateCondBr(InRegs, InRegBlock, InMemBlock);
354: 
355:   // Emit code to load the value if it was passed in registers.
356:   CGF.EmitBlock(InRegBlock);
357: 
358:   // Work out the address of an argument register.
359:   llvm::Value *ScaledRegCount =
360:     CGF.Builder.CreateMul(RegCount, PaddedSizeV, "scaled_reg_count");
361:   llvm::Value *RegBase =
362:     llvm::ConstantInt::get(IndexTy, RegSaveIndex * PaddedSize.getQuantity()
363:                                       + RegPadding.getQuantity());
364:   llvm::Value *RegOffset =
365:     CGF.Builder.CreateAdd(ScaledRegCount, RegBase, "reg_offset");
366:   Address RegSaveAreaPtr =
367:       CGF.Builder.CreateStructGEP(VAListAddr, 3, "reg_save_area_ptr");
368:   llvm::Value *RegSaveArea =
```
- **EN**: This block spells out callable entry points like `get`.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`。

### Lines 369-384
```cpp
369:       CGF.Builder.CreateLoad(RegSaveAreaPtr, "reg_save_area");
370:   Address RawRegAddr(
371:       CGF.Builder.CreateGEP(CGF.Int8Ty, RegSaveArea, RegOffset, "raw_reg_addr"),
372:       CGF.Int8Ty, PaddedSize);
373:   Address RegAddr = RawRegAddr.withElementType(DirectTy);
374: 
375:   // Update the register count
376:   llvm::Value *One = llvm::ConstantInt::get(IndexTy, 1);
377:   llvm::Value *NewRegCount =
378:     CGF.Builder.CreateAdd(RegCount, One, "reg_count");
379:   CGF.Builder.CreateStore(NewRegCount, RegCountPtr);
380:   CGF.EmitBranch(ContBlock);
381: 
382:   // Emit code to load the value if it was passed in memory.
383:   CGF.EmitBlock(InMemBlock);
384: 
```
- **EN**: This block spells out callable entry points like `RawRegAddr`.
- **CN**: 该代码块给出可调用入口的声明，例如 `RawRegAddr`。

### Lines 385-400
```cpp
385:   // Work out the address of a stack argument.
386:   Address OverflowArgAreaPtr =
387:       CGF.Builder.CreateStructGEP(VAListAddr, 2, "overflow_arg_area_ptr");
388:   Address OverflowArgArea =
389:       Address(CGF.Builder.CreateLoad(OverflowArgAreaPtr, "overflow_arg_area"),
390:               CGF.Int8Ty, PaddedSize);
391:   Address RawMemAddr =
392:       CGF.Builder.CreateConstByteGEP(OverflowArgArea, Padding, "raw_mem_addr");
393:   Address MemAddr = RawMemAddr.withElementType(DirectTy);
394: 
395:   // Update overflow_arg_area_ptr pointer
396:   llvm::Value *NewOverflowArgArea = CGF.Builder.CreateGEP(
397:       OverflowArgArea.getElementType(), OverflowArgArea.emitRawPointer(CGF),
398:       PaddedSizeV, "overflow_arg_area");
399:   CGF.Builder.CreateStore(NewOverflowArgArea, OverflowArgAreaPtr);
400:   CGF.EmitBranch(ContBlock);
```
- **EN**: This block spells out callable entry points like `Address`.
- **CN**: 该代码块给出可调用入口的声明，例如 `Address`。

### Lines 401-416
```cpp
401: 
402:   // Return the appropriate result.
403:   CGF.EmitBlock(ContBlock);
404:   Address ResAddr = emitMergePHI(CGF, RegAddr, InRegBlock, MemAddr, InMemBlock,
405:                                  "va_arg.addr");
406: 
407:   if (IsIndirect)
408:     ResAddr = Address(CGF.Builder.CreateLoad(ResAddr, "indirect_arg"), ArgTy,
409:                       TyInfo.Align);
410: 
411:   return CGF.EmitLoadOfAnyValue(CGF.MakeAddrLValue(ResAddr, Ty), Slot);
412: }
413: 
414: ABIArgInfo SystemZABIInfo::classifyReturnType(QualType RetTy) const {
415:   if (RetTy->isVoidType())
416:     return ABIArgInfo::getIgnore();
```
- **EN**: This block defines callable entry points like `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 417-432
```cpp
417:   if (isVectorArgumentType(RetTy))
418:     return ABIArgInfo::getDirect();
419:   if (isCompoundType(RetTy) || getContext().getTypeSize(RetTy) > 64)
420:     return getNaturalAlignIndirect(RetTy, getDataLayout().getAllocaAddrSpace());
421:   return (isPromotableIntegerTypeForABI(RetTy) ? ABIArgInfo::getExtend(RetTy)
422:                                                : ABIArgInfo::getDirect());
423: }
424: 
425: ABIArgInfo SystemZABIInfo::classifyArgumentType(QualType Ty) const {
426:   // Handle transparent union types.
427:   Ty = useFirstFieldIfTransparentUnion(Ty);
428: 
429:   // Handle the generic C++ ABI.
430:   if (CGCXXABI::RecordArgABI RAA = getRecordArgABI(Ty, getCXXABI()))
431:     return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
432:                                    RAA == CGCXXABI::RAA_DirectInMemory);
```
- **EN**: This block defines callable entry points like `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 433-448
```cpp
433: 
434:   // Integers and enums are extended to full register width.
435:   if (isPromotableIntegerTypeForABI(Ty))
436:     return ABIArgInfo::getExtend(Ty, CGT.ConvertType(Ty));
437: 
438:   // Handle vector types and vector-like structure types.  Note that
439:   // as opposed to float-like structure types, we do not allow any
440:   // padding for vector-like structures, so verify the sizes match.
441:   uint64_t Size = getContext().getTypeSize(Ty);
442:   QualType SingleElementTy = GetSingleElementType(Ty);
443:   if (isVectorArgumentType(SingleElementTy) &&
444:       getContext().getTypeSize(SingleElementTy) == Size)
445:     return ABIArgInfo::getDirect(CGT.ConvertType(SingleElementTy));
446: 
447:   // Values that are not 1, 2, 4 or 8 bytes in size are passed indirectly.
448:   if (Size != 8 && Size != 16 && Size != 32 && Size != 64)
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 449-464
```cpp
449:     return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
450:                                    /*ByVal=*/false);
451: 
452:   // Handle small structures.
453:   if (const auto *RD = Ty->getAsRecordDecl()) {
454:     // Structures with flexible arrays have variable length, so really
455:     // fail the size test above.
456:     if (RD->hasFlexibleArrayMember())
457:       return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
458:                                      /*ByVal=*/false);
459: 
460:     // The structure is passed as an unextended integer, a half, a float,
461:     // or a double.
462:     if (llvm::Type *FPArgTy = getFPArgumentType(SingleElementTy, Size)) {
463:       assert(Size == 16 || Size == 32 || Size == 64);
464:       return ABIArgInfo::getDirect(FPArgTy);
```
- **EN**: This block defines callable entry points like `getNaturalAlignIndirect`, `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getNaturalAlignIndirect`, `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 465-480
```cpp
465:     } else {
466:       llvm::IntegerType *PassTy = llvm::IntegerType::get(getVMContext(), Size);
467:       return Size <= 32 ? ABIArgInfo::getNoExtend(PassTy)
468:                         : ABIArgInfo::getDirect(PassTy);
469:     }
470:   }
471: 
472:   // Non-structure compounds are passed indirectly.
473:   if (isCompoundType(Ty))
474:     return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
475:                                    /*ByVal=*/false);
476: 
477:   return ABIArgInfo::getDirect(nullptr);
478: }
479: 
480: void SystemZABIInfo::computeInfo(CGFunctionInfo &FI) const {
```
- **EN**: This block defines callable entry points like `getDirect`, `computeInfo`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `computeInfo`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 481-496
```cpp
481:   const SystemZTargetCodeGenInfo &SZCGI =
482:       static_cast<const SystemZTargetCodeGenInfo &>(
483:           CGT.getCGM().getTargetCodeGenInfo());
484:   if (!getCXXABI().classifyReturnType(FI))
485:     FI.getReturnInfo() = classifyReturnType(FI.getReturnType());
486:   unsigned Idx = 0;
487:   for (auto &I : FI.arguments()) {
488:     I.info = classifyArgumentType(I.type);
489:     if (FI.isVariadic() && Idx++ >= FI.getNumRequiredArgs())
490:       // Check if a vararg vector argument is passed, in which case the
491:       // vector ABI becomes visible as the va_list could be passed on to
492:       // other functions.
493:       SZCGI.handleExternallyVisibleObjABI(I.type.getTypePtr(), CGT.getCGM(),
494:                                           /*IsParam*/true);
495:   }
496: }
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 497-512
```cpp
497: 
498: bool SystemZTargetCodeGenInfo::isVectorTypeBased(const Type *Ty,
499:                                                  bool IsParam) const {
500:   if (!SeenTypes.insert(Ty).second)
501:     return false;
502: 
503:   if (IsParam) {
504:     // A narrow (<16 bytes) vector will as a parameter also expose the ABI as
505:     // it will be passed in a vector register. A wide (>16 bytes) vector will
506:     // be passed via "hidden" pointer where any extra alignment is not
507:     // required (per GCC).
508:     const Type *SingleEltTy = getABIInfo<SystemZABIInfo>()
509:                                   .GetSingleElementType(QualType(Ty, 0))
510:                                   .getTypePtr();
511:     bool SingleVecEltStruct = SingleEltTy != Ty && SingleEltTy->isVectorType() &&
512:       Ctx.getTypeSize(SingleEltTy) == Ctx.getTypeSize(Ty);
```
- **EN**: This block defines callable entry points like `isVectorTypeBased`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `isVectorTypeBased`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 513-528
```cpp
513:     if (Ty->isVectorType() || SingleVecEltStruct)
514:       return Ctx.getTypeSize(Ty) / 8 <= 16;
515:   }
516: 
517:   // Assume pointers are dereferenced.
518:   while (Ty->isPointerType() || Ty->isArrayType())
519:     Ty = Ty->getPointeeOrArrayElementType();
520: 
521:   // Vectors >= 16 bytes expose the ABI through alignment requirements.
522:   if (Ty->isVectorType() && Ctx.getTypeSize(Ty) / 8 >= 16)
523:       return true;
524: 
525:   if (const auto *RD = Ty->getAsRecordDecl()) {
526:     if (const CXXRecordDecl *CXXRD = dyn_cast<CXXRecordDecl>(RD))
527:       if (CXXRD->hasDefinition())
528:         for (const auto &I : CXXRD->bases())
```
- **EN**: This block uses control flow (if, for, while) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for, while）细化 目标相关的 ABI 与代码生成 行为。

### Lines 529-544
```cpp
529:           if (isVectorTypeBased(I.getType().getTypePtr(), /*IsParam*/false))
530:             return true;
531:     for (const auto *FD : RD->fields())
532:       if (isVectorTypeBased(FD->getType().getTypePtr(), /*IsParam*/false))
533:         return true;
534:   }
535: 
536:   if (const auto *FT = Ty->getAs<FunctionType>())
537:     if (isVectorTypeBased(FT->getReturnType().getTypePtr(), /*IsParam*/true))
538:       return true;
539:   if (const FunctionProtoType *Proto = Ty->getAs<FunctionProtoType>())
540:     for (const auto &ParamType : Proto->getParamTypes())
541:       if (isVectorTypeBased(ParamType.getTypePtr(), /*IsParam*/true))
542:         return true;
543: 
544:   return false;
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 545-552
```cpp
545: }
546: 
547: std::unique_ptr<TargetCodeGenInfo>
548: CodeGen::createSystemZTargetCodeGenInfo(CodeGenModule &CGM, bool HasVector,
549:                                         bool SoftFloatABI) {
550:   return std::make_unique<SystemZTargetCodeGenInfo>(CGM.getTypes(), HasVector,
551:                                                     SoftFloatABI);
552: }
```
- **EN**: This block defines callable entry points like `createSystemZTargetCodeGenInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `createSystemZTargetCodeGenInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

## Key Concepts / 关键概念

- **CGF**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。
- **Builder**: Acts as a construction helper that incrementally assembles target-specific ABI and code generation state. / 充当构建辅助器，逐步组装 目标相关的 ABI 与代码生成 状态。
- **QualType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Address**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Size**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `TargetInfo.h`
- **Clang libraries / Clang 库**: `clang/Basic/Builtins.h`
- **LLVM libraries / LLVM 库**: `llvm/IR/IntrinsicsS390.h`
