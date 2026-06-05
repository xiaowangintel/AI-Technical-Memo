# AMDGPU.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/Targets/AMDGPU.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements target-specific ABI rules and lowering details for AMDGPU.
- **Purpose (CN) / 目的（中文）**: 实现 AMDGPU 目标相关的 ABI 规则与降级细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: //===- AMDGPU.cpp ---------------------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: #include "ABIInfoImpl.h"
10: #include "TargetInfo.h"
11: #include "clang/AST/DeclCXX.h"
12: #include "llvm/ADT/StringExtras.h"
13: #include "llvm/Support/AMDGPUAddrSpace.h"
14: 
15: using namespace clang;
16: using namespace clang::CodeGen;
17: 
18: //===----------------------------------------------------------------------===//
19: // AMDGPU ABI Implementation
20: //===----------------------------------------------------------------------===//
```
- **EN**: This block imports local CodeGen headers `ABIInfoImpl.h`, `TargetInfo.h`; Clang headers `clang/AST/DeclCXX.h`; LLVM headers `llvm/ADT/StringExtras.h`, `llvm/Support/AMDGPUAddrSpace.h`; opens or references namespaces `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfoImpl.h`, `TargetInfo.h`；Clang 头文件 `clang/AST/DeclCXX.h`；LLVM 头文件 `llvm/ADT/StringExtras.h`, `llvm/Support/AMDGPUAddrSpace.h`；打开或引用命名空间 `clang`；包含影响本编译单元构建方式的预处理结构。

### Lines 21-40
```cpp
21: 
22: namespace {
23: 
24: class AMDGPUABIInfo final : public DefaultABIInfo {
25: private:
26:   static const unsigned MaxNumRegsForArgsRet = 16;
27: 
28:   uint64_t numRegsForType(QualType Ty) const;
29: 
30:   bool isHomogeneousAggregateBaseType(QualType Ty) const override;
31:   bool isHomogeneousAggregateSmallEnough(const Type *Base,
32:                                          uint64_t Members) const override;
33: 
34:   // Coerce HIP scalar pointer arguments from generic pointers to global ones.
35:   llvm::Type *coerceKernelArgumentType(llvm::Type *Ty, unsigned FromAS,
36:                                        unsigned ToAS) const {
37:     // Single value types.
38:     auto *PtrTy = llvm::dyn_cast<llvm::PointerType>(Ty);
39:     if (PtrTy && PtrTy->getAddressSpace() == FromAS)
40:       return llvm::PointerType::get(Ty->getContext(), ToAS);
```
- **EN**: This block introduces declarations such as `AMDGPUABIInfo`; defines callable entry points like `numRegsForType`, `isHomogeneousAggregateBaseType`, `isHomogeneousAggregateSmallEnough`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出诸如 `AMDGPUABIInfo` 的声明；定义可调用入口，例如 `numRegsForType`, `isHomogeneousAggregateBaseType`, `isHomogeneousAggregateSmallEnough`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 41-60
```cpp
41:     return Ty;
42:   }
43: 
44: public:
45:   explicit AMDGPUABIInfo(CodeGen::CodeGenTypes &CGT) :
46:     DefaultABIInfo(CGT) {}
47: 
48:   ABIArgInfo classifyReturnType(QualType RetTy) const;
49:   ABIArgInfo classifyKernelArgumentType(QualType Ty) const;
50:   ABIArgInfo classifyArgumentType(QualType Ty, bool Variadic,
51:                                   unsigned &NumRegsLeft) const;
52: 
53:   void computeInfo(CGFunctionInfo &FI) const override;
54:   RValue EmitVAArg(CodeGenFunction &CGF, Address VAListAddr, QualType Ty,
55:                    AggValueSlot Slot) const override;
56: 
57:   llvm::FixedVectorType *
58:   getOptimalVectorMemoryType(llvm::FixedVectorType *T,
59:                              const LangOptions &Opt) const override {
60:     // We have legal instructions for 96-bit so 3x32 can be supported.
```
- **EN**: This block defines callable entry points like `AMDGPUABIInfo`, `classifyReturnType`, `classifyKernelArgumentType`, `classifyArgumentType`, `computeInfo`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `AMDGPUABIInfo`, `classifyReturnType`, `classifyKernelArgumentType`, `classifyArgumentType`, `computeInfo`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 61-80
```cpp
61:     // FIXME: This check should be a subtarget feature as technically SI doesn't
62:     // support it.
63:     if (T->getNumElements() == 3 && getDataLayout().getTypeSizeInBits(T) == 96)
64:       return T;
65:     return DefaultABIInfo::getOptimalVectorMemoryType(T, Opt);
66:   }
67: };
68: 
69: bool AMDGPUABIInfo::isHomogeneousAggregateBaseType(QualType Ty) const {
70:   return true;
71: }
72: 
73: bool AMDGPUABIInfo::isHomogeneousAggregateSmallEnough(
74:   const Type *Base, uint64_t Members) const {
75:   uint32_t NumRegs = (getContext().getTypeSize(Base) + 31) / 32;
76: 
77:   // Homogeneous Aggregates may occupy at most 16 registers.
78:   return Members * NumRegs <= MaxNumRegsForArgsRet;
79: }
80: 
```
- **EN**: This block defines callable entry points like `getOptimalVectorMemoryType`, `isHomogeneousAggregateBaseType`, `isHomogeneousAggregateSmallEnough`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getOptimalVectorMemoryType`, `isHomogeneousAggregateBaseType`, `isHomogeneousAggregateSmallEnough`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 81-100
```cpp
 81: /// Check if all fields in an aggregate type contain only sub-32-bit integer
 82: /// types. Such aggregates should be packed into i32 registers rather than
 83: /// passed as individual elements. Aggregates containing floats or full-sized
 84: /// integer types (i32, i64) should preserve their original types.
 85: static bool containsOnlyPackableIntegerTypes(const RecordDecl *RD,
 86:                                              const ASTContext &Context) {
 87:   for (const FieldDecl *Field : RD->fields()) {
 88:     QualType FieldTy = Field->getType();
 89: 
 90:     // For bitfields, they are always integer types so they're always packable.
 91:     // A bitfield like "unsigned a : 4" should be packable even though
 92:     // 'unsigned' is 32 bits. Similarly, larger bitfields that fill into
 93:     // wider ints (like i64) should also be packed.
 94:     if (Field->isBitField()) {
 95:       continue;
 96:     }
 97: 
 98:     // Recursively check nested structs
 99:     if (const RecordDecl *NestedRD = FieldTy->getAsRecordDecl()) {
100:       if (!containsOnlyPackableIntegerTypes(NestedRD, Context))
```
- **EN**: This block defines callable entry points like `containsOnlyPackableIntegerTypes`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `containsOnlyPackableIntegerTypes`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 101-120
```cpp
101:         return false;
102:       continue;
103:     }
104: 
105:     // Arrays - check the element type
106:     if (const ConstantArrayType *AT = Context.getAsConstantArrayType(FieldTy)) {
107:       QualType EltTy = AT->getElementType();
108:       if (const RecordDecl *NestedRD = EltTy->getAsRecordDecl()) {
109:         if (!containsOnlyPackableIntegerTypes(NestedRD, Context))
110:           return false;
111:         continue;
112:       }
113:       // For non-struct array elements, check if they're packable integers
114:       if (!EltTy->isIntegerType())
115:         return false;
116:       uint64_t EltSize = Context.getTypeSize(EltTy);
117:       if (EltSize >= 32)
118:         return false;
119:       continue;
120:     }
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 121-140
```cpp
121: 
122:     // Floating point types should not be packed into integers
123:     if (FieldTy->isFloatingType())
124:       return false;
125: 
126:     // Only integer types that are smaller than 32 bits should be packed
127:     if (!FieldTy->isIntegerType())
128:       return false;
129: 
130:     uint64_t FieldSize = Context.getTypeSize(FieldTy);
131:     if (FieldSize >= 32)
132:       return false;
133:   }
134:   return true;
135: }
136: 
137: /// Estimate number of registers the type will use when passed in registers.
138: uint64_t AMDGPUABIInfo::numRegsForType(QualType Ty) const {
139:   uint64_t NumRegs = 0;
140: 
```
- **EN**: This block defines callable entry points like `numRegsForType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `numRegsForType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 141-160
```cpp
141:   if (const VectorType *VT = Ty->getAs<VectorType>()) {
142:     // Compute from the number of elements. The reported size is based on the
143:     // in-memory size, which includes the padding 4th element for 3-vectors.
144:     QualType EltTy = VT->getElementType();
145:     uint64_t EltSize = getContext().getTypeSize(EltTy);
146: 
147:     // 16-bit element vectors should be passed as packed.
148:     if (EltSize == 16)
149:       return (VT->getNumElements() + 1) / 2;
150: 
151:     uint64_t EltNumRegs = (EltSize + 31) / 32;
152:     return EltNumRegs * VT->getNumElements();
153:   }
154: 
155:   if (const auto *RD = Ty->getAsRecordDecl()) {
156:     assert(!RD->hasFlexibleArrayMember());
157: 
158:     for (const FieldDecl *Field : RD->fields()) {
159:       QualType FieldTy = Field->getType();
160:       NumRegs += numRegsForType(FieldTy);
```
- **EN**: This block uses control flow (if, for) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 161-180
```cpp
161:     }
162: 
163:     return NumRegs;
164:   }
165: 
166:   return (getContext().getTypeSize(Ty) + 31) / 32;
167: }
168: 
169: void AMDGPUABIInfo::computeInfo(CGFunctionInfo &FI) const {
170:   llvm::CallingConv::ID CC = FI.getCallingConvention();
171: 
172:   if (!getCXXABI().classifyReturnType(FI))
173:     FI.getReturnInfo() = classifyReturnType(FI.getReturnType());
174: 
175:   unsigned ArgumentIndex = 0;
176:   const unsigned numFixedArguments = FI.getNumRequiredArgs();
177: 
178:   unsigned NumRegsLeft = MaxNumRegsForArgsRet;
179:   for (auto &Arg : FI.arguments()) {
180:     if (CC == llvm::CallingConv::AMDGPU_KERNEL) {
```
- **EN**: This block defines callable entry points like `computeInfo`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `computeInfo`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 181-200
```cpp
181:       Arg.info = classifyKernelArgumentType(Arg.type);
182:     } else {
183:       bool FixedArgument = ArgumentIndex++ < numFixedArguments;
184:       Arg.info = classifyArgumentType(Arg.type, !FixedArgument, NumRegsLeft);
185:     }
186:   }
187: }
188: 
189: RValue AMDGPUABIInfo::EmitVAArg(CodeGenFunction &CGF, Address VAListAddr,
190:                                 QualType Ty, AggValueSlot Slot) const {
191:   const bool IsIndirect = false;
192:   const bool AllowHigherAlign = false;
193:   return emitVoidPtrVAArg(CGF, VAListAddr, Ty, IsIndirect,
194:                           getContext().getTypeInfoInChars(Ty),
195:                           CharUnits::fromQuantity(4), AllowHigherAlign, Slot);
196: }
197: 
198: ABIArgInfo AMDGPUABIInfo::classifyReturnType(QualType RetTy) const {
199:   if (isAggregateTypeForABI(RetTy)) {
200:     // Records with non-trivial destructors/copy-constructors should not be
```
- **EN**: This block defines callable entry points like `EmitVAArg`, `emitVoidPtrVAArg`, `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `EmitVAArg`, `emitVoidPtrVAArg`, `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 201-220
```cpp
201:     // returned by value.
202:     if (!getRecordArgABI(RetTy, getCXXABI())) {
203:       // Ignore empty structs/unions.
204:       if (isEmptyRecord(getContext(), RetTy, true))
205:         return ABIArgInfo::getIgnore();
206: 
207:       // Lower single-element structs to just return a regular value.
208:       if (const Type *SeltTy = isSingleElementStruct(RetTy, getContext()))
209:         return ABIArgInfo::getDirect(CGT.ConvertType(QualType(SeltTy, 0)));
210: 
211:       if (const auto *RD = RetTy->getAsRecordDecl();
212:           RD && RD->hasFlexibleArrayMember())
213:         return DefaultABIInfo::classifyReturnType(RetTy);
214: 
215:       // Pack aggregates <= 8 bytes into single VGPR or pair, but only if they
216:       // contain sub-32-bit integer types. Aggregates with floats or full-sized
217:       // integers should preserve their original types.
218:       uint64_t Size = getContext().getTypeSize(RetTy);
219:       if (Size <= 64) {
220:         const RecordDecl *RD = RetTy->getAsRecordDecl();
```
- **EN**: This block defines callable entry points like `classifyReturnType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyReturnType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 221-240
```cpp
221:         bool ShouldPackToInt =
222:             RD && containsOnlyPackableIntegerTypes(RD, getContext());
223: 
224:         if (ShouldPackToInt) {
225:           if (Size <= 16)
226:             return ABIArgInfo::getDirect(
227:                 llvm::Type::getInt16Ty(getVMContext()));
228: 
229:           if (Size <= 32)
230:             return ABIArgInfo::getDirect(
231:                 llvm::Type::getInt32Ty(getVMContext()));
232: 
233:           llvm::Type *I32Ty = llvm::Type::getInt32Ty(getVMContext());
234:           return ABIArgInfo::getDirect(llvm::ArrayType::get(I32Ty, 2));
235:         }
236:       }
237: 
238:       if (numRegsForType(RetTy) <= MaxNumRegsForArgsRet)
239:         return ABIArgInfo::getDirect();
240:     }
```
- **EN**: This block defines callable entry points like `containsOnlyPackableIntegerTypes`, `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `containsOnlyPackableIntegerTypes`, `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 241-260
```cpp
241:   }
242: 
243:   // Otherwise just do the default thing.
244:   return DefaultABIInfo::classifyReturnType(RetTy);
245: }
246: 
247: /// For kernels all parameters are really passed in a special buffer. It doesn't
248: /// make sense to pass anything byval, so everything must be direct.
249: ABIArgInfo AMDGPUABIInfo::classifyKernelArgumentType(QualType Ty) const {
250:   Ty = useFirstFieldIfTransparentUnion(Ty);
251: 
252:   // TODO: Can we omit empty structs?
253: 
254:   if (const Type *SeltTy = isSingleElementStruct(Ty, getContext()))
255:     Ty = QualType(SeltTy, 0);
256: 
257:   llvm::Type *OrigLTy = CGT.ConvertType(Ty);
258:   llvm::Type *LTy = OrigLTy;
259:   if (getContext().getLangOpts().HIP) {
260:     LTy = coerceKernelArgumentType(
```
- **EN**: This block defines callable entry points like `classifyReturnType`, `classifyKernelArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyReturnType`, `classifyKernelArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 261-280
```cpp
261:         OrigLTy, /*FromAS=*/getContext().getTargetAddressSpace(LangAS::Default),
262:         /*ToAS=*/getContext().getTargetAddressSpace(LangAS::cuda_device));
263:   }
264: 
265:   // FIXME: This doesn't apply the optimization of coercing pointers in structs
266:   // to global address space when using byref. This would require implementing a
267:   // new kind of coercion of the in-memory type when for indirect arguments.
268:   if (LTy == OrigLTy && isAggregateTypeForABI(Ty)) {
269:     return ABIArgInfo::getIndirectAliased(
270:         getContext().getTypeAlignInChars(Ty),
271:         getContext().getTargetAddressSpace(LangAS::opencl_constant),
272:         false /*Realign*/, nullptr /*Padding*/);
273:   }
274: 
275:   // If we set CanBeFlattened to true, CodeGen will expand the struct to its
276:   // individual elements, which confuses the Clover OpenCL backend; therefore we
277:   // have to set it to false here. Other args of getDirect() are just defaults.
278:   return ABIArgInfo::getDirect(LTy, 0, nullptr, false);
279: }
280: 
```
- **EN**: This block defines callable entry points like `getIndirectAliased`, `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getIndirectAliased`, `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 281-300
```cpp
281: ABIArgInfo AMDGPUABIInfo::classifyArgumentType(QualType Ty, bool Variadic,
282:                                                unsigned &NumRegsLeft) const {
283:   assert(NumRegsLeft <= MaxNumRegsForArgsRet && "register estimate underflow");
284: 
285:   Ty = useFirstFieldIfTransparentUnion(Ty);
286: 
287:   if (Variadic) {
288:     return ABIArgInfo::getDirect(/*T=*/nullptr,
289:                                  /*Offset=*/0,
290:                                  /*Padding=*/nullptr,
291:                                  /*CanBeFlattened=*/false,
292:                                  /*Align=*/0);
293:   }
294: 
295:   if (isAggregateTypeForABI(Ty)) {
296:     // Records with non-trivial destructors/copy-constructors should not be
297:     // passed by value.
298:     if (auto RAA = getRecordArgABI(Ty, getCXXABI()))
299:       return getNaturalAlignIndirect(Ty, getDataLayout().getAllocaAddrSpace(),
300:                                      RAA == CGCXXABI::RAA_DirectInMemory);
```
- **EN**: This block defines callable entry points like `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 301-320
```cpp
301: 
302:     // Ignore empty structs/unions.
303:     if (isEmptyRecord(getContext(), Ty, true))
304:       return ABIArgInfo::getIgnore();
305: 
306:     // Lower single-element structs to just pass a regular value. TODO: We
307:     // could do reasonable-size multiple-element structs too, using getExpand(),
308:     // though watch out for things like bitfields.
309:     if (const Type *SeltTy = isSingleElementStruct(Ty, getContext()))
310:       return ABIArgInfo::getDirect(CGT.ConvertType(QualType(SeltTy, 0)));
311: 
312:     if (const auto *RD = Ty->getAsRecordDecl();
313:         RD && RD->hasFlexibleArrayMember())
314:       return DefaultABIInfo::classifyArgumentType(Ty);
315: 
316:     // Pack aggregates <= 8 bytes into single VGPR or pair, but only if they
317:     // contain sub-32-bit integer types. Aggregates with floats or full-sized
318:     // integers (i32, i64) should preserve their original types.
319:     uint64_t Size = getContext().getTypeSize(Ty);
320:     if (Size <= 64) {
```
- **EN**: This block defines callable entry points like `classifyArgumentType`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `classifyArgumentType`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 321-340
```cpp
321:       const RecordDecl *RD = Ty->getAsRecordDecl();
322:       bool ShouldPackToInt =
323:           RD && containsOnlyPackableIntegerTypes(RD, getContext());
324: 
325:       if (ShouldPackToInt) {
326:         unsigned NumRegs = (Size + 31) / 32;
327:         NumRegsLeft -= std::min(NumRegsLeft, NumRegs);
328: 
329:         if (Size <= 16)
330:           return ABIArgInfo::getDirect(llvm::Type::getInt16Ty(getVMContext()));
331: 
332:         if (Size <= 32)
333:           return ABIArgInfo::getDirect(llvm::Type::getInt32Ty(getVMContext()));
334: 
335:         llvm::Type *I32Ty = llvm::Type::getInt32Ty(getVMContext());
336:         return ABIArgInfo::getDirect(llvm::ArrayType::get(I32Ty, 2));
337:       }
338:     }
339: 
340:     if (NumRegsLeft > 0) {
```
- **EN**: This block defines callable entry points like `containsOnlyPackableIntegerTypes`, `getDirect`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `containsOnlyPackableIntegerTypes`, `getDirect`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 341-360
```cpp
341:       uint64_t NumRegs = numRegsForType(Ty);
342:       if (NumRegsLeft >= NumRegs) {
343:         NumRegsLeft -= NumRegs;
344:         return ABIArgInfo::getDirect();
345:       }
346:     }
347: 
348:     // Use pass-by-reference in stead of pass-by-value for struct arguments in
349:     // function ABI.
350:     return ABIArgInfo::getIndirectAliased(
351:         getContext().getTypeAlignInChars(Ty),
352:         getContext().getTargetAddressSpace(LangAS::opencl_private));
353:   }
354: 
355:   // Otherwise just do the default thing.
356:   ABIArgInfo ArgInfo = DefaultABIInfo::classifyArgumentType(Ty);
357:   if (!ArgInfo.isIndirect()) {
358:     uint64_t NumRegs = numRegsForType(Ty);
359:     NumRegsLeft -= std::min(NumRegs, uint64_t{NumRegsLeft});
360:   }
```
- **EN**: This block defines callable entry points like `getDirect`, `getIndirectAliased`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDirect`, `getIndirectAliased`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 361-380
```cpp
361: 
362:   return ArgInfo;
363: }
364: 
365: class AMDGPUTargetCodeGenInfo : public TargetCodeGenInfo {
366: public:
367:   AMDGPUTargetCodeGenInfo(CodeGenTypes &CGT)
368:       : TargetCodeGenInfo(std::make_unique<AMDGPUABIInfo>(CGT)) {}
369: 
370:   bool supportsLibCall() const override { return false; }
371:   void setFunctionDeclAttributes(const FunctionDecl *FD, llvm::Function *F,
372:                                  CodeGenModule &CGM) const;
373: 
374:   void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
375:                            CodeGen::CodeGenModule &M) const override;
376:   unsigned getDeviceKernelCallingConv() const override;
377: 
378:   llvm::Constant *getNullPointer(const CodeGen::CodeGenModule &CGM,
379:       llvm::PointerType *T, QualType QT) const override;
380: 
```
- **EN**: This block introduces declarations such as `AMDGPUTargetCodeGenInfo`; defines callable entry points like `AMDGPUTargetCodeGenInfo`, `supportsLibCall`, `setFunctionDeclAttributes`, `setTargetAttributes`, `getDeviceKernelCallingConv`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块给出诸如 `AMDGPUTargetCodeGenInfo` 的声明；定义可调用入口，例如 `AMDGPUTargetCodeGenInfo`, `supportsLibCall`, `setFunctionDeclAttributes`, `setTargetAttributes`, `getDeviceKernelCallingConv`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 381-400
```cpp
381:   LangAS getSRetAddrSpace(const CXXRecordDecl *RD) const override;
382: 
383:   LangAS getGlobalVarAddressSpace(CodeGenModule &CGM,
384:                                   const VarDecl *D) const override;
385:   StringRef getLLVMSyncScopeStr(const LangOptions &LangOpts, SyncScope Scope,
386:                                 llvm::AtomicOrdering Ordering) const override;
387:   void setTargetAtomicMetadata(CodeGenFunction &CGF,
388:                                llvm::Instruction &AtomicInst,
389:                                const AtomicExpr *Expr = nullptr) const override;
390:   llvm::Value *createEnqueuedBlockKernel(CodeGenFunction &CGF,
391:                                          llvm::Function *BlockInvokeFunc,
392:                                          llvm::Type *BlockTy) const override;
393:   bool shouldEmitStaticExternCAliases() const override;
394:   bool shouldEmitDWARFBitFieldSeparators() const override;
395:   void setCUDAKernelCallingConvention(const FunctionType *&FT) const override;
396: };
397: }
398: 
399: static bool requiresAMDGPUProtectedVisibility(const Decl *D,
400:                                               llvm::GlobalValue *GV) {
```
- **EN**: This block defines callable entry points like `getSRetAddrSpace`, `getGlobalVarAddressSpace`, `getLLVMSyncScopeStr`, `setTargetAtomicMetadata`, `shouldEmitStaticExternCAliases`.
- **CN**: 该代码块定义可调用入口，例如 `getSRetAddrSpace`, `getGlobalVarAddressSpace`, `getLLVMSyncScopeStr`, `setTargetAtomicMetadata`, `shouldEmitStaticExternCAliases`。

### Lines 401-420
```cpp
401:   if (GV->getVisibility() != llvm::GlobalValue::HiddenVisibility)
402:     return false;
403: 
404:   return !D->hasAttr<OMPDeclareTargetDeclAttr>() &&
405:          (D->hasAttr<DeviceKernelAttr>() ||
406:           (isa<FunctionDecl>(D) && D->hasAttr<CUDAGlobalAttr>()) ||
407:           (isa<VarDecl>(D) &&
408:            (D->hasAttr<CUDADeviceAttr>() || D->hasAttr<CUDAConstantAttr>() ||
409:             cast<VarDecl>(D)->getType()->isCUDADeviceBuiltinSurfaceType() ||
410:             cast<VarDecl>(D)->getType()->isCUDADeviceBuiltinTextureType())));
411: }
412: 
413: void AMDGPUTargetCodeGenInfo::setFunctionDeclAttributes(
414:     const FunctionDecl *FD, llvm::Function *F, CodeGenModule &M) const {
415:   const auto *ReqdWGS =
416:       M.getLangOpts().OpenCL ? FD->getAttr<ReqdWorkGroupSizeAttr>() : nullptr;
417:   const bool IsOpenCLKernel =
418:       M.getLangOpts().OpenCL && FD->hasAttr<DeviceKernelAttr>();
419:   const bool IsHIPKernel = M.getLangOpts().HIP && FD->hasAttr<CUDAGlobalAttr>();
420: 
```
- **EN**: This block defines callable entry points like `setFunctionDeclAttributes`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `setFunctionDeclAttributes`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 421-440
```cpp
421:   const auto *FlatWGS = FD->getAttr<AMDGPUFlatWorkGroupSizeAttr>();
422:   if (ReqdWGS || FlatWGS) {
423:     M.handleAMDGPUFlatWorkGroupSizeAttr(F, FlatWGS, ReqdWGS);
424:   } else if (IsOpenCLKernel || IsHIPKernel) {
425:     // By default, restrict the maximum size to a value specified by
426:     // --gpu-max-threads-per-block=n or its default value for HIP.
427:     const unsigned OpenCLDefaultMaxWorkGroupSize = 256;
428:     const unsigned DefaultMaxWorkGroupSize =
429:         IsOpenCLKernel ? OpenCLDefaultMaxWorkGroupSize
430:                        : M.getLangOpts().GPUMaxThreadsPerBlock;
431:     std::string AttrVal =
432:         std::string("1,") + llvm::utostr(DefaultMaxWorkGroupSize);
433:     F->addFnAttr("amdgpu-flat-work-group-size", AttrVal);
434:   }
435: 
436:   if (const auto *Attr = FD->getAttr<AMDGPUWavesPerEUAttr>())
437:     M.handleAMDGPUWavesPerEUAttr(F, Attr);
438: 
439:   if (const auto *Attr = FD->getAttr<AMDGPUNumSGPRAttr>()) {
440:     unsigned NumSGPR = Attr->getNumSGPR();
```
- **EN**: This block defines callable entry points like `string`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `string`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 441-460
```cpp
441: 
442:     if (NumSGPR != 0)
443:       F->addFnAttr("amdgpu-num-sgpr", llvm::utostr(NumSGPR));
444:   }
445: 
446:   if (const auto *Attr = FD->getAttr<AMDGPUNumVGPRAttr>()) {
447:     uint32_t NumVGPR = Attr->getNumVGPR();
448: 
449:     if (NumVGPR != 0)
450:       F->addFnAttr("amdgpu-num-vgpr", llvm::utostr(NumVGPR));
451:   }
452: 
453:   if (const auto *Attr = FD->getAttr<AMDGPUMaxNumWorkGroupsAttr>()) {
454:     uint32_t X = Attr->getMaxNumWorkGroupsX()
455:                      ->EvaluateKnownConstInt(M.getContext())
456:                      .getExtValue();
457:     // Y and Z dimensions default to 1 if not specified
458:     uint32_t Y = Attr->getMaxNumWorkGroupsY()
459:                      ? Attr->getMaxNumWorkGroupsY()
460:                            ->EvaluateKnownConstInt(M.getContext())
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 461-480
```cpp
461:                            .getExtValue()
462:                      : 1;
463:     uint32_t Z = Attr->getMaxNumWorkGroupsZ()
464:                      ? Attr->getMaxNumWorkGroupsZ()
465:                            ->EvaluateKnownConstInt(M.getContext())
466:                            .getExtValue()
467:                      : 1;
468: 
469:     llvm::SmallString<32> AttrVal;
470:     llvm::raw_svector_ostream OS(AttrVal);
471:     OS << X << ',' << Y << ',' << Z;
472: 
473:     F->addFnAttr("amdgpu-max-num-workgroups", AttrVal.str());
474:   }
475: 
476:   if (auto *Attr = FD->getAttr<CUDAClusterDimsAttr>()) {
477:     auto GetExprVal = [&](const auto &E) {
478:       return E ? E->EvaluateKnownConstInt(M.getContext()).getExtValue() : 1;
479:     };
480:     unsigned X = GetExprVal(Attr->getX());
```
- **EN**: This block defines callable entry points like `OS`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `OS`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 481-500
```cpp
481:     unsigned Y = GetExprVal(Attr->getY());
482:     unsigned Z = GetExprVal(Attr->getZ());
483:     llvm::SmallString<32> AttrVal;
484:     llvm::raw_svector_ostream OS(AttrVal);
485:     OS << X << ',' << Y << ',' << Z;
486:     F->addFnAttr("amdgpu-cluster-dims", AttrVal.str());
487:   }
488: 
489:   // OpenCL doesn't support cluster feature.
490:   const TargetInfo &TTI = M.getContext().getTargetInfo();
491:   if ((IsOpenCLKernel &&
492:        TTI.hasFeatureEnabled(TTI.getTargetOpts().FeatureMap, "clusters")) ||
493:       FD->hasAttr<CUDANoClusterAttr>())
494:     F->addFnAttr("amdgpu-cluster-dims", "0,0,0");
495: }
496: 
497: void AMDGPUTargetCodeGenInfo::setTargetAttributes(
498:     const Decl *D, llvm::GlobalValue *GV, CodeGen::CodeGenModule &M) const {
499:   if (requiresAMDGPUProtectedVisibility(D, GV)) {
500:     GV->setVisibility(llvm::GlobalValue::ProtectedVisibility);
```
- **EN**: This block defines callable entry points like `OS`, `setTargetAttributes`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `OS`, `setTargetAttributes`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 501-520
```cpp
501:     GV->setDSOLocal(true);
502:   }
503: 
504:   if (GV->isDeclaration())
505:     return;
506: 
507:   llvm::Function *F = dyn_cast<llvm::Function>(GV);
508:   if (!F)
509:     return;
510: 
511:   const FunctionDecl *FD = dyn_cast_or_null<FunctionDecl>(D);
512:   if (FD)
513:     setFunctionDeclAttributes(FD, F, M);
514:   if (!getABIInfo().getCodeGenOpts().EmitIEEENaNCompliantInsts)
515:     F->addFnAttr("amdgpu-ieee", "false");
516:   if (getABIInfo().getCodeGenOpts().AMDGPUExpandWaitcntProfiling)
517:     F->addFnAttr("amdgpu-expand-waitcnt-profiling");
518: }
519: 
520: unsigned AMDGPUTargetCodeGenInfo::getDeviceKernelCallingConv() const {
```
- **EN**: This block defines callable entry points like `getDeviceKernelCallingConv`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getDeviceKernelCallingConv`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 521-540
```cpp
521:   return llvm::CallingConv::AMDGPU_KERNEL;
522: }
523: 
524: // Currently LLVM assumes null pointers always have value 0,
525: // which results in incorrectly transformed IR. Therefore, instead of
526: // emitting null pointers in private and local address spaces, a null
527: // pointer in generic address space is emitted which is casted to a
528: // pointer in local or private address space.
529: llvm::Constant *AMDGPUTargetCodeGenInfo::getNullPointer(
530:     const CodeGen::CodeGenModule &CGM, llvm::PointerType *PT,
531:     QualType QT) const {
532:   if (CGM.getContext().getTargetNullPointerValue(QT) == 0)
533:     return llvm::ConstantPointerNull::get(PT);
534: 
535:   auto &Ctx = CGM.getContext();
536:   auto NPT = llvm::PointerType::get(
537:       PT->getContext(), Ctx.getTargetAddressSpace(LangAS::opencl_generic));
538:   return llvm::ConstantExpr::getAddrSpaceCast(
539:       llvm::ConstantPointerNull::get(NPT), PT);
540: }
```
- **EN**: This block defines callable entry points like `getAddrSpaceCast`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getAddrSpaceCast`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 541-560
```cpp
541: 
542: LangAS
543: AMDGPUTargetCodeGenInfo::getSRetAddrSpace(const CXXRecordDecl *RD) const {
544:   // Types with no viable copy/move must be constructed in-place , use the
545:   // default AS so the sret pointer matches the "this" convention.
546:   if (RD && !RD->canPassInRegisters())
547:     return LangAS::Default;
548:   return getLangASFromTargetAS(
549:       getABIInfo().getDataLayout().getAllocaAddrSpace());
550: }
551: 
552: LangAS
553: AMDGPUTargetCodeGenInfo::getGlobalVarAddressSpace(CodeGenModule &CGM,
554:                                                   const VarDecl *D) const {
555:   assert(!CGM.getLangOpts().OpenCL &&
556:          !(CGM.getLangOpts().CUDA && CGM.getLangOpts().CUDAIsDevice) &&
557:          "Address space agnostic languages only");
558:   LangAS DefaultGlobalAS = getLangASFromTargetAS(
559:       CGM.getContext().getTargetAddressSpace(LangAS::opencl_global));
560:   if (!D)
```
- **EN**: This block defines callable entry points like `getSRetAddrSpace`, `getLangASFromTargetAS`, `getGlobalVarAddressSpace`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getSRetAddrSpace`, `getLangASFromTargetAS`, `getGlobalVarAddressSpace`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 561-580
```cpp
561:     return DefaultGlobalAS;
562: 
563:   LangAS AddrSpace = D->getType().getAddressSpace();
564:   if (AddrSpace != LangAS::Default)
565:     return AddrSpace;
566: 
567:   // Only promote to address space 4 if VarDecl has constant initialization.
568:   if (D->getType().isConstantStorage(CGM.getContext(), false, false) &&
569:       D->hasConstantInitialization()) {
570:     if (auto ConstAS = CGM.getTarget().getConstantAddressSpace())
571:       return *ConstAS;
572:   }
573:   return DefaultGlobalAS;
574: }
575: 
576: StringRef AMDGPUTargetCodeGenInfo::getLLVMSyncScopeStr(
577:     const LangOptions &LangOpts, SyncScope Scope,
578:     llvm::AtomicOrdering Ordering) const {
579: 
580:   // OpenCL assumes by default that atomic scopes are per-address space for
```
- **EN**: This block defines callable entry points like `getLLVMSyncScopeStr`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `getLLVMSyncScopeStr`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 581-600
```cpp
581:   // non-sequentially consistent operations.
582:   bool IsOneAs = (Scope >= SyncScope::OpenCLWorkGroup &&
583:                   Scope <= SyncScope::OpenCLSubGroup &&
584:                   Ordering != llvm::AtomicOrdering::SequentiallyConsistent);
585: 
586:   switch (Scope) {
587:   case SyncScope::HIPSingleThread:
588:   case SyncScope::SingleScope:
589:     return IsOneAs ? "singlethread-one-as" : "singlethread";
590:   case SyncScope::HIPWavefront:
591:   case SyncScope::OpenCLSubGroup:
592:   case SyncScope::WavefrontScope:
593:     return IsOneAs ? "wavefront-one-as" : "wavefront";
594:   case SyncScope::HIPCluster:
595:   case SyncScope::ClusterScope:
596:     assert(!IsOneAs && "OpenCL does not have cluster scope");
597:     return "cluster";
598:   case SyncScope::HIPWorkgroup:
599:   case SyncScope::OpenCLWorkGroup:
600:   case SyncScope::WorkgroupScope:
```
- **EN**: This block uses control flow (switch, case) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 601-620
```cpp
601:     return IsOneAs ? "workgroup-one-as" : "workgroup";
602:   case SyncScope::HIPAgent:
603:   case SyncScope::OpenCLDevice:
604:   case SyncScope::DeviceScope:
605:     return IsOneAs ? "agent-one-as" : "agent";
606:   case SyncScope::SystemScope:
607:   case SyncScope::HIPSystem:
608:   case SyncScope::OpenCLAllSVMDevices:
609:     return IsOneAs ? "one-as" : "";
610:   }
611:   llvm_unreachable("Unknown SyncScope enum");
612: }
613: 
614: void AMDGPUTargetCodeGenInfo::setTargetAtomicMetadata(
615:     CodeGenFunction &CGF, llvm::Instruction &AtomicInst,
616:     const AtomicExpr *AE) const {
617:   auto *RMW = dyn_cast<llvm::AtomicRMWInst>(&AtomicInst);
618:   auto *CmpX = dyn_cast<llvm::AtomicCmpXchgInst>(&AtomicInst);
619: 
620:   // OpenCL and old style HIP atomics consider atomics targeting thread private
```
- **EN**: This block defines callable entry points like `setTargetAtomicMetadata`; uses control flow (case) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `setTargetAtomicMetadata`；通过控制流（case）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 621-640
```cpp
621:   // memory to be undefined.
622:   //
623:   // TODO: This is probably undefined for atomic load/store, but there's not
624:   // much direct codegen benefit to knowing this.
625:   if (((RMW && RMW->getPointerAddressSpace() == llvm::AMDGPUAS::FLAT_ADDRESS) ||
626:        (CmpX &&
627:         CmpX->getPointerAddressSpace() == llvm::AMDGPUAS::FLAT_ADDRESS)) &&
628:       AE && AE->threadPrivateMemoryAtomicsAreUndefined()) {
629:     llvm::MDBuilder MDHelper(CGF.getLLVMContext());
630:     llvm::MDNode *ASRange = MDHelper.createRange(
631:         llvm::APInt(32, llvm::AMDGPUAS::PRIVATE_ADDRESS),
632:         llvm::APInt(32, llvm::AMDGPUAS::PRIVATE_ADDRESS + 1));
633:     AtomicInst.setMetadata(llvm::LLVMContext::MD_noalias_addrspace, ASRange);
634:   }
635: 
636:   if (!RMW)
637:     return;
638: 
639:   AtomicOptions AO = CGF.CGM.getAtomicOpts();
640:   llvm::MDNode *Empty = llvm::MDNode::get(CGF.getLLVMContext(), {});
```
- **EN**: This block defines callable entry points like `MDHelper`, `APInt`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `MDHelper`, `APInt`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 641-660
```cpp
641:   if (!AO.getOption(clang::AtomicOptionKind::FineGrainedMemory))
642:     RMW->setMetadata("amdgpu.no.fine.grained.memory", Empty);
643:   if (!AO.getOption(clang::AtomicOptionKind::RemoteMemory))
644:     RMW->setMetadata("amdgpu.no.remote.memory", Empty);
645:   if (AO.getOption(clang::AtomicOptionKind::IgnoreDenormalMode) &&
646:       RMW->getOperation() == llvm::AtomicRMWInst::FAdd &&
647:       RMW->getType()->isFloatTy())
648:     RMW->setMetadata("amdgpu.ignore.denormal.mode", Empty);
649: }
650: 
651: bool AMDGPUTargetCodeGenInfo::shouldEmitStaticExternCAliases() const {
652:   return false;
653: }
654: 
655: bool AMDGPUTargetCodeGenInfo::shouldEmitDWARFBitFieldSeparators() const {
656:   return true;
657: }
658: 
659: void AMDGPUTargetCodeGenInfo::setCUDAKernelCallingConvention(
660:     const FunctionType *&FT) const {
```
- **EN**: This block defines callable entry points like `shouldEmitStaticExternCAliases`, `shouldEmitDWARFBitFieldSeparators`, `setCUDAKernelCallingConvention`; uses control flow (if) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `shouldEmitStaticExternCAliases`, `shouldEmitDWARFBitFieldSeparators`, `setCUDAKernelCallingConvention`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为。

### Lines 661-680
```cpp
661:   FT = getABIInfo().getContext().adjustFunctionType(
662:       FT, FT->getExtInfo().withCallingConv(CC_DeviceKernel));
663: }
664: 
665: /// Return IR struct type for rtinfo struct in rocm-device-libs used for device
666: /// enqueue.
667: ///
668: /// ptr addrspace(1) kernel_object, i32 private_segment_size,
669: /// i32 group_segment_size
670: 
671: static llvm::StructType *
672: getAMDGPURuntimeHandleType(llvm::LLVMContext &C,
673:                            llvm::Type *KernelDescriptorPtrTy) {
674:   llvm::Type *Int32 = llvm::Type::getInt32Ty(C);
675:   return llvm::StructType::create(C, {KernelDescriptorPtrTy, Int32, Int32},
676:                                   "block.runtime.handle.t");
677: }
678: 
679: /// Create an OpenCL kernel for an enqueued block.
680: ///
```
- **EN**: This block defines callable entry points like `getAMDGPURuntimeHandleType`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块定义可调用入口，例如 `getAMDGPURuntimeHandleType`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 681-700
```cpp
681: /// The type of the first argument (the block literal) is the struct type
682: /// of the block literal instead of a pointer type. The first argument
683: /// (block literal) is passed directly by value to the kernel. The kernel
684: /// allocates the same type of struct on stack and stores the block literal
685: /// to it and passes its pointer to the block invoke function. The kernel
686: /// has "enqueued-block" function attribute and kernel argument metadata.
687: llvm::Value *AMDGPUTargetCodeGenInfo::createEnqueuedBlockKernel(
688:     CodeGenFunction &CGF, llvm::Function *Invoke, llvm::Type *BlockTy) const {
689:   auto &Builder = CGF.Builder;
690:   auto &C = CGF.getLLVMContext();
691: 
692:   auto *InvokeFT = Invoke->getFunctionType();
693:   llvm::SmallVector<llvm::Type *, 2> ArgTys;
694:   llvm::SmallVector<llvm::Metadata *, 8> AddressQuals;
695:   llvm::SmallVector<llvm::Metadata *, 8> AccessQuals;
696:   llvm::SmallVector<llvm::Metadata *, 8> ArgTypeNames;
697:   llvm::SmallVector<llvm::Metadata *, 8> ArgBaseTypeNames;
698:   llvm::SmallVector<llvm::Metadata *, 8> ArgTypeQuals;
699:   llvm::SmallVector<llvm::Metadata *, 8> ArgNames;
700: 
```
- **EN**: This block documents intent or context for the surrounding target-specific ABI and code generation code.
- **CN**: 该代码块说明周围 目标相关的 ABI 与代码生成 代码的意图或上下文。

### Lines 701-720
```cpp
701:   ArgTys.push_back(BlockTy);
702:   ArgTypeNames.push_back(llvm::MDString::get(C, "__block_literal"));
703:   AddressQuals.push_back(llvm::ConstantAsMetadata::get(Builder.getInt32(0)));
704:   ArgBaseTypeNames.push_back(llvm::MDString::get(C, "__block_literal"));
705:   ArgTypeQuals.push_back(llvm::MDString::get(C, ""));
706:   AccessQuals.push_back(llvm::MDString::get(C, "none"));
707:   ArgNames.push_back(llvm::MDString::get(C, "block_literal"));
708:   for (unsigned I = 1, E = InvokeFT->getNumParams(); I < E; ++I) {
709:     ArgTys.push_back(InvokeFT->getParamType(I));
710:     ArgTypeNames.push_back(llvm::MDString::get(C, "void*"));
711:     AddressQuals.push_back(llvm::ConstantAsMetadata::get(Builder.getInt32(3)));
712:     AccessQuals.push_back(llvm::MDString::get(C, "none"));
713:     ArgBaseTypeNames.push_back(llvm::MDString::get(C, "void*"));
714:     ArgTypeQuals.push_back(llvm::MDString::get(C, ""));
715:     ArgNames.push_back(
716:         llvm::MDString::get(C, (Twine("local_arg") + Twine(I)).str()));
717:   }
718: 
719:   llvm::Module &Mod = CGF.CGM.getModule();
720:   const llvm::DataLayout &DL = Mod.getDataLayout();
```
- **EN**: This block defines callable entry points like `get`; uses control flow (for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 721-740
```cpp
721: 
722:   llvm::Twine Name = Invoke->getName() + "_kernel";
723:   auto *FT = llvm::FunctionType::get(llvm::Type::getVoidTy(C), ArgTys, false);
724: 
725:   // The kernel itself can be internal, the runtime does not directly access the
726:   // kernel address (only the kernel descriptor).
727:   auto *F = llvm::Function::Create(FT, llvm::GlobalValue::InternalLinkage, Name,
728:                                    &Mod);
729:   F->setCallingConv(getDeviceKernelCallingConv());
730: 
731:   llvm::AttrBuilder KernelAttrs(C);
732:   // FIXME: The invoke isn't applying the right attributes either
733:   // FIXME: This is missing setTargetAttributes
734:   CGF.CGM.addDefaultFunctionDefinitionAttributes(KernelAttrs);
735:   F->addFnAttrs(KernelAttrs);
736: 
737:   auto IP = CGF.Builder.saveIP();
738:   auto *BB = llvm::BasicBlock::Create(C, "entry", F);
739:   Builder.SetInsertPoint(BB);
740:   const auto BlockAlign = DL.getPrefTypeAlign(BlockTy);
```
- **EN**: This block spells out callable entry points like `KernelAttrs`.
- **CN**: 该代码块给出可调用入口的声明，例如 `KernelAttrs`。

### Lines 741-760
```cpp
741:   auto *BlockPtr = Builder.CreateAlloca(BlockTy, nullptr);
742:   BlockPtr->setAlignment(BlockAlign);
743:   Builder.CreateAlignedStore(F->arg_begin(), BlockPtr, BlockAlign);
744:   auto *Cast = Builder.CreatePointerCast(BlockPtr, InvokeFT->getParamType(0));
745:   llvm::SmallVector<llvm::Value *, 2> Args;
746:   Args.push_back(Cast);
747:   for (llvm::Argument &A : llvm::drop_begin(F->args()))
748:     Args.push_back(&A);
749:   llvm::CallInst *call = Builder.CreateCall(Invoke, Args);
750:   call->setCallingConv(Invoke->getCallingConv());
751:   Builder.CreateRetVoid();
752:   Builder.restoreIP(IP);
753: 
754:   F->setMetadata("kernel_arg_addr_space", llvm::MDNode::get(C, AddressQuals));
755:   F->setMetadata("kernel_arg_access_qual", llvm::MDNode::get(C, AccessQuals));
756:   F->setMetadata("kernel_arg_type", llvm::MDNode::get(C, ArgTypeNames));
757:   F->setMetadata("kernel_arg_base_type",
758:                  llvm::MDNode::get(C, ArgBaseTypeNames));
759:   F->setMetadata("kernel_arg_type_qual", llvm::MDNode::get(C, ArgTypeQuals));
760:   if (CGF.CGM.getCodeGenOpts().EmitOpenCLArgMetadata)
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (if, for) to specialize target-specific ABI and code generation.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（if, for）细化 目标相关的 ABI 与代码生成 行为。

### Lines 761-780
```cpp
761:     F->setMetadata("kernel_arg_name", llvm::MDNode::get(C, ArgNames));
762: 
763:   llvm::StructType *HandleTy = getAMDGPURuntimeHandleType(
764:       C, llvm::PointerType::get(C, DL.getDefaultGlobalsAddressSpace()));
765:   llvm::Constant *RuntimeHandleInitializer =
766:       llvm::ConstantAggregateZero::get(HandleTy);
767: 
768:   llvm::Twine RuntimeHandleName = F->getName() + ".runtime.handle";
769: 
770:   // The runtime needs access to the runtime handle as an external symbol. The
771:   // runtime handle will need to be made external later, in
772:   // AMDGPUExportOpenCLEnqueuedBlocks. The kernel itself has a hidden reference
773:   // inside the runtime handle, and is not directly referenced.
774: 
775:   // TODO: We would initialize the first field by declaring F->getName() + ".kd"
776:   // to reference the kernel descriptor. The runtime wouldn't need to bother
777:   // setting it. We would need to have a final symbol name though.
778:   // TODO: Can we directly use an external symbol with getGlobalIdentifier?
779:   auto *RuntimeHandle = new llvm::GlobalVariable(
780:       Mod, HandleTy,
```
- **EN**: This block spells out callable entry points like `get`.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`。

### Lines 781-800
```cpp
781:       /*isConstant=*/true, llvm::GlobalValue::InternalLinkage,
782:       /*Initializer=*/RuntimeHandleInitializer, RuntimeHandleName,
783:       /*InsertBefore=*/nullptr, llvm::GlobalValue::NotThreadLocal,
784:       DL.getDefaultGlobalsAddressSpace(),
785:       /*isExternallyInitialized=*/true);
786: 
787:   llvm::MDNode *HandleAsMD =
788:       llvm::MDNode::get(C, llvm::ValueAsMetadata::get(RuntimeHandle));
789:   F->setMetadata(llvm::LLVMContext::MD_associated, HandleAsMD);
790: 
791:   RuntimeHandle->setSection(".amdgpu.kernel.runtime.handle");
792: 
793:   CGF.CGM.addUsedGlobal(F);
794:   CGF.CGM.addUsedGlobal(RuntimeHandle);
795:   return RuntimeHandle;
796: }
797: 
798: void CodeGenModule::handleAMDGPUFlatWorkGroupSizeAttr(
799:     llvm::Function *F, const AMDGPUFlatWorkGroupSizeAttr *FlatWGS,
800:     const ReqdWorkGroupSizeAttr *ReqdWGS, int32_t *MinThreadsVal,
```
- **EN**: This block spells out callable entry points like `get`; returns or forwards computed values for the surrounding target-specific ABI and code generation logic.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；为周围的 目标相关的 ABI 与代码生成 逻辑返回或转发计算结果。

### Lines 801-820
```cpp
801:     int32_t *MaxThreadsVal) {
802:   unsigned Min = 0;
803:   unsigned Max = 0;
804:   auto Eval = [&](Expr *E) {
805:     return E->EvaluateKnownConstInt(getContext()).getExtValue();
806:   };
807:   if (FlatWGS) {
808:     Min = Eval(FlatWGS->getMin());
809:     Max = Eval(FlatWGS->getMax());
810:   }
811:   if (ReqdWGS && Min == 0 && Max == 0)
812:     Min = Max = Eval(ReqdWGS->getXDim()) * Eval(ReqdWGS->getYDim()) *
813:                 Eval(ReqdWGS->getZDim());
814: 
815:   if (Min != 0) {
816:     assert(Min <= Max && "Min must be less than or equal Max");
817: 
818:     if (MinThreadsVal)
819:       *MinThreadsVal = Min;
820:     if (MaxThreadsVal)
```
- **EN**: This block uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 821-840
```cpp
821:       *MaxThreadsVal = Max;
822:     std::string AttrVal = llvm::utostr(Min) + "," + llvm::utostr(Max);
823:     if (F)
824:       F->addFnAttr("amdgpu-flat-work-group-size", AttrVal);
825:   } else
826:     assert(Max == 0 && "Max must be zero");
827: }
828: 
829: void CodeGenModule::handleAMDGPUWavesPerEUAttr(
830:     llvm::Function *F, const AMDGPUWavesPerEUAttr *Attr) {
831:   unsigned Min =
832:       Attr->getMin()->EvaluateKnownConstInt(getContext()).getExtValue();
833:   unsigned Max =
834:       Attr->getMax()
835:           ? Attr->getMax()->EvaluateKnownConstInt(getContext()).getExtValue()
836:           : 0;
837: 
838:   if (Min != 0) {
839:     assert((Max == 0 || Min <= Max) && "Min must be less than or equal Max");
840: 
```
- **EN**: This block defines callable entry points like `handleAMDGPUWavesPerEUAttr`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `handleAMDGPUWavesPerEUAttr`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 841-852
```cpp
841:     std::string AttrVal = llvm::utostr(Min);
842:     if (Max != 0)
843:       AttrVal = AttrVal + "," + llvm::utostr(Max);
844:     F->addFnAttr("amdgpu-waves-per-eu", AttrVal);
845:   } else
846:     assert(Max == 0 && "Max must be zero");
847: }
848: 
849: std::unique_ptr<TargetCodeGenInfo>
850: CodeGen::createAMDGPUTargetCodeGenInfo(CodeGenModule &CGM) {
851:   return std::make_unique<AMDGPUTargetCodeGenInfo>(CGM.getTypes());
852: }
```
- **EN**: This block defines callable entry points like `createAMDGPUTargetCodeGenInfo`; uses control flow (if) to specialize target-specific ABI and code generation; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `createAMDGPUTargetCodeGenInfo`；通过控制流（if）细化 目标相关的 ABI 与代码生成 行为；使用断言或不可达标记保护关键不变量。

## Key Concepts / 关键概念

- **getContext**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ABIArgInfo**: Likely stores or computes descriptive metadata that drives target-specific ABI and code generation. / 很可能用于保存或计算驱动 目标相关的 ABI 与代码生成 的描述性元数据。
- **QualType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGM**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。
- **SyncScope**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Attr**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGF**: Central symbol in this file's implementation of target-specific ABI and code generation. / 是该文件实现 目标相关的 ABI 与代码生成 时的核心符号。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfoImpl.h`, `TargetInfo.h`
- **Clang libraries / Clang 库**: `clang/AST/DeclCXX.h`
- **LLVM libraries / LLVM 库**: `llvm/ADT/StringExtras.h`, `llvm/Support/AMDGPUAddrSpace.h`
