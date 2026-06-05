# CGHLSLBuiltins.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/CGHLSLBuiltins.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements the CGHLSLBuiltins portion of Clang's CodeGen pipeline.
- **Purpose (CN) / 目的（中文）**: 实现 Clang CodeGen 流水线中的 CGHLSLBuiltins 部分。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: //===------- CGHLSLBuiltins.cpp - Emit LLVM Code for HLSL builtins --------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This contains code to emit HLSL Builtin calls as LLVM code.
10: //
11: //===----------------------------------------------------------------------===//
12: 
13: #include "CGBuiltin.h"
14: #include "CGHLSLRuntime.h"
15: #include "CodeGenFunction.h"
16: #include "llvm/IR/MatrixBuilder.h"
17: 
18: using namespace clang;
19: using namespace CodeGen;
20: using namespace llvm;
```
- **EN**: This block imports local CodeGen headers `CGBuiltin.h`, `CGHLSLRuntime.h`, `CodeGenFunction.h`; LLVM headers `llvm/IR/MatrixBuilder.h`; opens or references namespaces `clang`, `CodeGen`, `llvm`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGBuiltin.h`, `CGHLSLRuntime.h`, `CodeGenFunction.h`；LLVM 头文件 `llvm/IR/MatrixBuilder.h`；打开或引用命名空间 `clang`, `CodeGen`, `llvm`；包含影响本编译单元构建方式的预处理结构。

### Lines 21-40
```cpp
21: 
22: static Value *handleAsDoubleBuiltin(CodeGenFunction &CGF, const CallExpr *E) {
23:   assert((E->getArg(0)->getType()->hasUnsignedIntegerRepresentation() &&
24:           E->getArg(1)->getType()->hasUnsignedIntegerRepresentation()) &&
25:          "asdouble operands types mismatch");
26:   Value *OpLowBits = CGF.EmitScalarExpr(E->getArg(0));
27:   Value *OpHighBits = CGF.EmitScalarExpr(E->getArg(1));
28: 
29:   llvm::Type *ResultType = CGF.DoubleTy;
30:   int N = 1;
31:   if (auto *VTy = E->getArg(0)->getType()->getAs<clang::VectorType>()) {
32:     N = VTy->getNumElements();
33:     ResultType = llvm::FixedVectorType::get(CGF.DoubleTy, N);
34:   }
35: 
36:   if (CGF.CGM.getTarget().getTriple().isDXIL())
37:     return CGF.Builder.CreateIntrinsic(
38:         /*ReturnType=*/ResultType, Intrinsic::dx_asdouble,
39:         {OpLowBits, OpHighBits}, nullptr, "hlsl.asdouble");
40: 
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 41-60
```cpp
41:   if (!E->getArg(0)->getType()->isVectorType()) {
42:     OpLowBits = CGF.Builder.CreateVectorSplat(1, OpLowBits);
43:     OpHighBits = CGF.Builder.CreateVectorSplat(1, OpHighBits);
44:   }
45: 
46:   llvm::SmallVector<int> Mask;
47:   for (int i = 0; i < N; i++) {
48:     Mask.push_back(i);
49:     Mask.push_back(i + N);
50:   }
51: 
52:   Value *BitVec = CGF.Builder.CreateShuffleVector(OpLowBits, OpHighBits, Mask);
53: 
54:   return CGF.Builder.CreateBitCast(BitVec, ResultType);
55: }
56: 
57: static Value *handleHlslClip(const CallExpr *E, CodeGenFunction *CGF) {
58:   Value *Op0 = CGF->EmitScalarExpr(E->getArg(0));
59: 
60:   Constant *FZeroConst = ConstantFP::getZero(CGF->FloatTy);
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 61-80
```cpp
61:   Value *CMP;
62:   Value *LastInstr;
63: 
64:   if (const auto *VecTy = E->getArg(0)->getType()->getAs<clang::VectorType>()) {
65:     FZeroConst = ConstantVector::getSplat(
66:         ElementCount::getFixed(VecTy->getNumElements()), FZeroConst);
67:     auto *FCompInst = CGF->Builder.CreateFCmpOLT(Op0, FZeroConst);
68:     CMP = CGF->Builder.CreateIntrinsic(
69:         CGF->Builder.getInt1Ty(), CGF->CGM.getHLSLRuntime().getAnyIntrinsic(),
70:         {FCompInst});
71:   } else {
72:     CMP = CGF->Builder.CreateFCmpOLT(Op0, FZeroConst);
73:   }
74: 
75:   if (CGF->CGM.getTarget().getTriple().isDXIL()) {
76:     LastInstr = CGF->Builder.CreateIntrinsic(Intrinsic::dx_discard, {CMP});
77:   } else if (CGF->CGM.getTarget().getTriple().isSPIRV()) {
78:     BasicBlock *LT0 = CGF->createBasicBlock("lt0", CGF->CurFn);
79:     BasicBlock *End = CGF->createBasicBlock("end", CGF->CurFn);
80: 
```
- **EN**: This block defines callable entry points like `getFixed`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getFixed`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 81-100
```cpp
 81:     CGF->Builder.CreateCondBr(CMP, LT0, End);
 82: 
 83:     CGF->Builder.SetInsertPoint(LT0);
 84: 
 85:     CGF->Builder.CreateIntrinsic(Intrinsic::spv_discard, {});
 86: 
 87:     LastInstr = CGF->Builder.CreateBr(End);
 88:     CGF->Builder.SetInsertPoint(End);
 89:   } else {
 90:     llvm_unreachable("Backend Codegen not supported.");
 91:   }
 92: 
 93:   return LastInstr;
 94: }
 95: 
 96: static Value *handleHlslSplitdouble(const CallExpr *E, CodeGenFunction *CGF) {
 97:   Value *Op0 = CGF->EmitScalarExpr(E->getArg(0));
 98:   const auto *OutArg1 = dyn_cast<HLSLOutArgExpr>(E->getArg(1));
 99:   const auto *OutArg2 = dyn_cast<HLSLOutArgExpr>(E->getArg(2));
100: 
```
- **EN**: This block guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块使用断言或不可达标记保护关键不变量。

### Lines 101-120
```cpp
101:   CallArgList Args;
102:   LValue Op1TmpLValue =
103:       CGF->EmitHLSLOutArgExpr(OutArg1, Args, OutArg1->getType());
104:   LValue Op2TmpLValue =
105:       CGF->EmitHLSLOutArgExpr(OutArg2, Args, OutArg2->getType());
106: 
107:   if (CGF->getTarget().getCXXABI().areArgsDestroyedLeftToRightInCallee())
108:     Args.reverseWritebacks();
109: 
110:   Value *LowBits = nullptr;
111:   Value *HighBits = nullptr;
112: 
113:   if (CGF->CGM.getTarget().getTriple().isDXIL()) {
114:     llvm::Type *RetElementTy = CGF->Int32Ty;
115:     if (auto *Op0VecTy = E->getArg(0)->getType()->getAs<clang::VectorType>())
116:       RetElementTy = llvm::VectorType::get(
117:           CGF->Int32Ty, ElementCount::getFixed(Op0VecTy->getNumElements()));
118:     auto *RetTy = llvm::StructType::get(RetElementTy, RetElementTy);
119: 
120:     CallInst *CI = CGF->Builder.CreateIntrinsic(
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 121-140
```cpp
121:         RetTy, Intrinsic::dx_splitdouble, {Op0}, nullptr, "hlsl.splitdouble");
122: 
123:     LowBits = CGF->Builder.CreateExtractValue(CI, 0);
124:     HighBits = CGF->Builder.CreateExtractValue(CI, 1);
125:   } else {
126:     // For Non DXIL targets we generate the instructions.
127: 
128:     if (!Op0->getType()->isVectorTy()) {
129:       FixedVectorType *DestTy = FixedVectorType::get(CGF->Int32Ty, 2);
130:       Value *Bitcast = CGF->Builder.CreateBitCast(Op0, DestTy);
131: 
132:       LowBits = CGF->Builder.CreateExtractElement(Bitcast, (uint64_t)0);
133:       HighBits = CGF->Builder.CreateExtractElement(Bitcast, 1);
134:     } else {
135:       int NumElements = 1;
136:       if (const auto *VecTy =
137:               E->getArg(0)->getType()->getAs<clang::VectorType>())
138:         NumElements = VecTy->getNumElements();
139: 
140:       FixedVectorType *Uint32VecTy =
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 141-160
```cpp
141:           FixedVectorType::get(CGF->Int32Ty, NumElements * 2);
142:       Value *Uint32Vec = CGF->Builder.CreateBitCast(Op0, Uint32VecTy);
143:       if (NumElements == 1) {
144:         LowBits = CGF->Builder.CreateExtractElement(Uint32Vec, (uint64_t)0);
145:         HighBits = CGF->Builder.CreateExtractElement(Uint32Vec, 1);
146:       } else {
147:         SmallVector<int> EvenMask, OddMask;
148:         for (int I = 0, E = NumElements; I != E; ++I) {
149:           EvenMask.push_back(I * 2);
150:           OddMask.push_back(I * 2 + 1);
151:         }
152:         LowBits = CGF->Builder.CreateShuffleVector(Uint32Vec, EvenMask);
153:         HighBits = CGF->Builder.CreateShuffleVector(Uint32Vec, OddMask);
154:       }
155:     }
156:   }
157:   CGF->Builder.CreateStore(LowBits, Op1TmpLValue.getAddress());
158:   auto *LastInst =
159:       CGF->Builder.CreateStore(HighBits, Op2TmpLValue.getAddress());
160:   CGF->EmitWritebacks(Args);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 161-180
```cpp
161:   return LastInst;
162: }
163: 
164: static Value *handleHlslWaveActiveBallot(CodeGenFunction &CGF,
165:                                          const CallExpr *E) {
166:   Value *Cond = CGF.EmitScalarExpr(E->getArg(0));
167:   llvm::Type *I32 = CGF.Int32Ty;
168: 
169:   llvm::Type *Vec4I32 = llvm::FixedVectorType::get(I32, 4);
170:   [[maybe_unused]] llvm::StructType *Struct4I32 =
171:       llvm::StructType::get(CGF.getLLVMContext(), {I32, I32, I32, I32});
172: 
173:   if (CGF.CGM.getTarget().getTriple().isDXIL()) {
174:     // Call DXIL intrinsic: returns { i32, i32, i32, i32 }
175:     Value *StructVal =
176:         CGF.EmitIntrinsicCall(Intrinsic::dx_wave_ballot, {I32}, {Cond});
177:     assert(StructVal->getType() == Struct4I32 &&
178:            "dx.wave.ballot must return {i32,i32,i32,i32}");
179: 
180:     // Reassemble struct to <4 x i32>
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 181-200
```cpp
181:     llvm::Value *VecVal = llvm::PoisonValue::get(Vec4I32);
182:     for (unsigned I = 0; I < 4; ++I) {
183:       Value *Elt = CGF.Builder.CreateExtractValue(StructVal, I);
184:       VecVal =
185:           CGF.Builder.CreateInsertElement(VecVal, Elt, CGF.Builder.getInt32(I));
186:     }
187: 
188:     return VecVal;
189:   }
190: 
191:   if (CGF.CGM.getTarget().getTriple().isSPIRV())
192:     return CGF.EmitIntrinsicCall(Intrinsic::spv_subgroup_ballot, {Cond});
193: 
194:   llvm_unreachable(
195:       "WaveActiveBallot is only supported for DXIL and SPIRV targets");
196: }
197: 
198: static Value *handleElementwiseF16ToF32(CodeGenFunction &CGF,
199:                                         const CallExpr *E) {
200:   Value *Op0 = CGF.EmitScalarExpr(E->getArg(0));
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 201-220
```cpp
201:   QualType Op0Ty = E->getArg(0)->getType();
202:   llvm::Type *ResType = CGF.FloatTy;
203:   uint64_t NumElements = 0;
204:   if (Op0->getType()->isVectorTy()) {
205:     NumElements =
206:         E->getArg(0)->getType()->castAs<clang::VectorType>()->getNumElements();
207:     ResType =
208:         llvm::VectorType::get(ResType, ElementCount::getFixed(NumElements));
209:   }
210:   if (!Op0Ty->hasUnsignedIntegerRepresentation())
211:     llvm_unreachable(
212:         "f16tof32 operand must have an unsigned int representation");
213: 
214:   if (CGF.CGM.getTriple().isDXIL())
215:     return CGF.Builder.CreateIntrinsic(ResType, Intrinsic::dx_legacyf16tof32,
216:                                        ArrayRef<Value *>{Op0}, nullptr,
217:                                        "hlsl.f16tof32");
218: 
219:   if (CGF.CGM.getTriple().isSPIRV()) {
220:     // We use the SPIRV UnpackHalf2x16 operation to avoid the need for the
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 221-240
```cpp
221:     // Int16 and Float16 capabilities
222:     auto *UnpackType =
223:         llvm::VectorType::get(CGF.FloatTy, ElementCount::getFixed(2));
224: 
225:     if (NumElements == 0) {
226:       // a scalar input - simply extract the first element of the unpacked
227:       // vector
228:       Value *Unpack = CGF.Builder.CreateIntrinsic(
229:           UnpackType, Intrinsic::spv_unpackhalf2x16, ArrayRef<Value *>{Op0});
230:       return CGF.Builder.CreateExtractElement(Unpack, (uint64_t)0);
231:     }
232: 
233:     // a vector input - build a congruent output vector by iterating through
234:     // the input vector calling unpackhalf2x16 for each element
235:     Value *Result = PoisonValue::get(ResType);
236:     for (uint64_t I = 0; I < NumElements; I++) {
237:       Value *InVal = CGF.Builder.CreateExtractElement(Op0, I);
238:       Value *Unpack = CGF.Builder.CreateIntrinsic(
239:           UnpackType, Intrinsic::spv_unpackhalf2x16, ArrayRef<Value *>{InVal});
240:       Value *Res = CGF.Builder.CreateExtractElement(Unpack, (uint64_t)0);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 241-260
```cpp
241:       Result = CGF.Builder.CreateInsertElement(Result, Res, I);
242:     }
243:     return Result;
244:   }
245: 
246:   llvm_unreachable("Intrinsic F16ToF32 not supported by target architecture");
247: }
248: 
249: static Value *handleElementwiseF32ToF16(CodeGenFunction &CGF,
250:                                         const CallExpr *E) {
251:   Value *Op0 = CGF.EmitScalarExpr(E->getArg(0));
252:   QualType Op0Ty = E->getArg(0)->getType();
253:   llvm::Type *ResType = CGF.IntTy;
254:   uint64_t NumElements = 0;
255:   if (Op0->getType()->isVectorTy()) {
256:     NumElements =
257:         E->getArg(0)->getType()->castAs<clang::VectorType>()->getNumElements();
258:     ResType =
259:         llvm::VectorType::get(ResType, ElementCount::getFixed(NumElements));
260:   }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 261-280
```cpp
261:   if (!Op0Ty->hasFloatingRepresentation())
262:     llvm_unreachable("f32tof16 operand must have a float representation");
263: 
264:   if (CGF.CGM.getTriple().isDXIL())
265:     return CGF.Builder.CreateIntrinsic(ResType, Intrinsic::dx_legacyf32tof16,
266:                                        ArrayRef<Value *>{Op0}, nullptr,
267:                                        "hlsl.f32tof16");
268: 
269:   if (CGF.CGM.getTriple().isSPIRV()) {
270:     // We use the SPIRV PackHalf2x16 operation to avoid the need for the
271:     // Int16 and Float16 capabilities
272:     auto *PackType =
273:         llvm::VectorType::get(CGF.FloatTy, ElementCount::getFixed(2));
274: 
275:     if (NumElements == 0) {
276:       // a scalar input - simply insert the scalar in the first element
277:       // of the 2 element float vector
278:       Value *Float2 = Constant::getNullValue(PackType);
279:       Float2 = CGF.Builder.CreateInsertElement(Float2, Op0, (uint64_t)0);
280:       Value *Result = CGF.Builder.CreateIntrinsic(
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 281-300
```cpp
281:           ResType, Intrinsic::spv_packhalf2x16, ArrayRef<Value *>{Float2});
282:       return Result;
283:     }
284: 
285:     // a vector input - build a congruent output vector by iterating through
286:     // the input vector calling packhalf2x16 for each element
287:     Value *Result = PoisonValue::get(ResType);
288:     for (uint64_t I = 0; I < NumElements; I++) {
289:       Value *Float2 = Constant::getNullValue(PackType);
290:       Value *InVal = CGF.Builder.CreateExtractElement(Op0, I);
291:       Float2 = CGF.Builder.CreateInsertElement(Float2, InVal, (uint64_t)0);
292:       Value *Res = CGF.Builder.CreateIntrinsic(
293:           CGF.IntTy, Intrinsic::spv_packhalf2x16, ArrayRef<Value *>{Float2});
294:       Result = CGF.Builder.CreateInsertElement(Result, Res, I);
295:     }
296:     return Result;
297:   }
298: 
299:   llvm_unreachable("Intrinsic F32ToF16 not supported by target architecture");
300: }
```
- **EN**: This block uses control flow (for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 301-320
```cpp
301: 
302: static Value *emitBufferStride(CodeGenFunction *CGF, const Expr *HandleExpr,
303:                                LValue &Stride) {
304:   // Figure out the stride of the buffer elements from the handle type.
305:   auto *HandleTy =
306:       cast<HLSLAttributedResourceType>(HandleExpr->getType().getTypePtr());
307:   QualType ElementTy = HandleTy->getContainedType();
308:   Value *StrideValue = CGF->getTypeSize(ElementTy);
309:   return CGF->Builder.CreateStore(StrideValue, Stride.getAddress());
310: }
311: 
312: // Return dot product intrinsic that corresponds to the QT scalar type
313: static Intrinsic::ID getDotProductIntrinsic(CGHLSLRuntime &RT, QualType QT) {
314:   if (QT->isFloatingType())
315:     return RT.getFDotIntrinsic();
316:   if (QT->isSignedIntegerType())
317:     return RT.getSDotIntrinsic();
318:   assert(QT->isUnsignedIntegerType());
319:   return RT.getUDotIntrinsic();
320: }
```
- **EN**: This block defines callable entry points like `getDotProductIntrinsic`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getDotProductIntrinsic`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 321-340
```cpp
321: 
322: static Intrinsic::ID getFirstBitHighIntrinsic(CGHLSLRuntime &RT, QualType QT) {
323:   if (QT->hasSignedIntegerRepresentation()) {
324:     return RT.getFirstBitSHighIntrinsic();
325:   }
326: 
327:   assert(QT->hasUnsignedIntegerRepresentation());
328:   return RT.getFirstBitUHighIntrinsic();
329: }
330: 
331: // Return wave active sum that corresponds to the QT scalar type
332: static Intrinsic::ID getWaveActiveSumIntrinsic(llvm::Triple::ArchType Arch,
333:                                                QualType QT) {
334:   switch (Arch) {
335:   case llvm::Triple::spirv:
336:     return Intrinsic::spv_wave_reduce_sum;
337:   case llvm::Triple::dxil: {
338:     if (QT->isUnsignedIntegerType())
339:       return Intrinsic::dx_wave_reduce_usum;
340:     return Intrinsic::dx_wave_reduce_sum;
```
- **EN**: This block defines callable entry points like `getFirstBitHighIntrinsic`, `getWaveActiveSumIntrinsic`; uses control flow (if, switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getFirstBitHighIntrinsic`, `getWaveActiveSumIntrinsic`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 341-360
```cpp
341:   }
342:   default:
343:     llvm_unreachable("Intrinsic WaveActiveSum"
344:                      " not supported by target architecture");
345:   }
346: }
347: 
348: // Return wave active product that corresponds to the QT scalar type
349: static Intrinsic::ID getWaveActiveProductIntrinsic(llvm::Triple::ArchType Arch,
350:                                                    QualType QT) {
351:   switch (Arch) {
352:   case llvm::Triple::spirv:
353:     return Intrinsic::spv_wave_product;
354:   case llvm::Triple::dxil: {
355:     if (QT->isUnsignedIntegerType())
356:       return Intrinsic::dx_wave_uproduct;
357:     return Intrinsic::dx_wave_product;
358:   }
359:   default:
360:     llvm_unreachable("Intrinsic WaveActiveProduct"
```
- **EN**: This block defines callable entry points like `getWaveActiveProductIntrinsic`; uses control flow (if, switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getWaveActiveProductIntrinsic`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 361-380
```cpp
361:                      " not supported by target architecture");
362:   }
363: }
364: 
365: static Intrinsic::ID getPrefixCountBitsIntrinsic(llvm::Triple::ArchType Arch) {
366:   switch (Arch) {
367:   case llvm::Triple::spirv:
368:     return Intrinsic::spv_subgroup_prefix_bit_count;
369:   case llvm::Triple::dxil: {
370:     return Intrinsic::dx_wave_prefix_bit_count;
371:   }
372:   default:
373:     llvm_unreachable(
374:         "WavePrefixOp instruction not supported by target architecture");
375:   }
376: }
377: 
378: // Return wave prefix sum that corresponds to the QT scalar type
379: static Intrinsic::ID getWavePrefixSumIntrinsic(llvm::Triple::ArchType Arch,
380:                                                QualType QT) {
```
- **EN**: This block defines callable entry points like `getPrefixCountBitsIntrinsic`, `getWavePrefixSumIntrinsic`; uses control flow (switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getPrefixCountBitsIntrinsic`, `getWavePrefixSumIntrinsic`；通过控制流（switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 381-400
```cpp
381:   switch (Arch) {
382:   case llvm::Triple::spirv:
383:     return Intrinsic::spv_wave_prefix_sum;
384:   case llvm::Triple::dxil: {
385:     if (QT->isUnsignedIntegerType())
386:       return Intrinsic::dx_wave_prefix_usum;
387:     return Intrinsic::dx_wave_prefix_sum;
388:   }
389:   default:
390:     llvm_unreachable("Intrinsic WavePrefixSum"
391:                      " not supported by target architecture");
392:   }
393: }
394: 
395: // Return wave prefix product that corresponds to the QT scalar type
396: static Intrinsic::ID getWavePrefixProductIntrinsic(llvm::Triple::ArchType Arch,
397:                                                    QualType QT) {
398:   switch (Arch) {
399:   case llvm::Triple::spirv:
400:     return Intrinsic::spv_wave_prefix_product;
```
- **EN**: This block defines callable entry points like `getWavePrefixProductIntrinsic`; uses control flow (if, switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getWavePrefixProductIntrinsic`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 401-420
```cpp
401:   case llvm::Triple::dxil: {
402:     if (QT->isUnsignedIntegerType())
403:       return Intrinsic::dx_wave_prefix_uproduct;
404:     return Intrinsic::dx_wave_prefix_product;
405:   }
406:   default:
407:     llvm_unreachable("Intrinsic WavePrefixProduct"
408:                      " not supported by target architecture");
409:   }
410: }
411: 
412: // Returns the mangled name for a builtin function that the SPIR-V backend
413: // will expand into a spec Constant.
414: static std::string getSpecConstantFunctionName(clang::QualType SpecConstantType,
415:                                                ASTContext &Context) {
416:   // The parameter types for our conceptual intrinsic function.
417:   QualType ClangParamTypes[] = {Context.IntTy, SpecConstantType};
418: 
419:   // Create a temporary FunctionDecl for the builtin fuction. It won't be
420:   // added to the AST.
```
- **EN**: This block defines callable entry points like `getSpecConstantFunctionName`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getSpecConstantFunctionName`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 421-440
```cpp
421:   FunctionProtoType::ExtProtoInfo EPI;
422:   QualType FnType =
423:       Context.getFunctionType(SpecConstantType, ClangParamTypes, EPI);
424:   DeclarationName FuncName = &Context.Idents.get("__spirv_SpecConstant");
425:   FunctionDecl *FnDeclForMangling = FunctionDecl::Create(
426:       Context, Context.getTranslationUnitDecl(), SourceLocation(),
427:       SourceLocation(), FuncName, FnType, /*TSI=*/nullptr, SC_Extern);
428: 
429:   // Attach the created parameter declarations to the function declaration.
430:   SmallVector<ParmVarDecl *, 2> ParamDecls;
431:   for (QualType ParamType : ClangParamTypes) {
432:     ParmVarDecl *PD = ParmVarDecl::Create(
433:         Context, FnDeclForMangling, SourceLocation(), SourceLocation(),
434:         /*IdentifierInfo*/ nullptr, ParamType, /*TSI*/ nullptr, SC_None,
435:         /*DefaultArg*/ nullptr);
436:     ParamDecls.push_back(PD);
437:   }
438:   FnDeclForMangling->setParams(ParamDecls);
439: 
440:   // Get the mangled name.
```
- **EN**: This block defines callable entry points like `SourceLocation`; uses control flow (for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `SourceLocation`；通过控制流（for）细化 LLVM IR 生成 行为。

### Lines 441-460
```cpp
441:   std::string Name;
442:   llvm::raw_string_ostream MangledNameStream(Name);
443:   std::unique_ptr<MangleContext> Mangler(Context.createMangleContext());
444:   Mangler->mangleName(FnDeclForMangling, MangledNameStream);
445:   MangledNameStream.flush();
446: 
447:   return Name;
448: }
449: 
450: static llvm::Type *getOffsetType(CodeGenModule &CGM, llvm::Type *CoordTy) {
451:   llvm::Type *Int32Ty = CGM.Int32Ty;
452:   if (auto *VT = dyn_cast<llvm::FixedVectorType>(CoordTy))
453:     return llvm::FixedVectorType::get(Int32Ty, VT->getNumElements());
454:   return Int32Ty;
455: }
456: 
457: static Value *emitHlslOffset(CodeGenFunction &CGF, const CallExpr *E,
458:                              unsigned OffsetArgIndex, llvm::Type *OffsetTy) {
459:   if (E->getNumArgs() > OffsetArgIndex)
460:     return CGF.EmitScalarExpr(E->getArg(OffsetArgIndex));
```
- **EN**: This block defines callable entry points like `MangledNameStream`, `Mangler`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `MangledNameStream`, `Mangler`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 461-480
```cpp
461: 
462:   return llvm::Constant::getNullValue(OffsetTy);
463: }
464: 
465: static Value *emitHlslClamp(CodeGenFunction &CGF, const CallExpr *E,
466:                             unsigned ClampArgIndex) {
467:   Value *Clamp = CGF.EmitScalarExpr(E->getArg(ClampArgIndex));
468:   // The builtin is defined with variadic arguments, so the clamp parameter
469:   // might have been promoted to double. The intrinsic requires a 32-bit
470:   // float.
471:   if (Clamp->getType() != CGF.Builder.getFloatTy())
472:     Clamp = CGF.Builder.CreateFPCast(Clamp, CGF.Builder.getFloatTy());
473:   return Clamp;
474: }
475: 
476: static Value *emitGetDimensions(CodeGenFunction &CGF, const CallExpr *E,
477:                                 unsigned IntrinsicID, unsigned NumRetComps,
478:                                 bool HasLod) {
479:   Value *Handle = CGF.EmitScalarExpr(E->getArg(0));
480: 
```
- **EN**: This block defines callable entry points like `getNullValue`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getNullValue`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 481-500
```cpp
481:   SmallVector<Value *> Args{Handle};
482:   if (HasLod)
483:     Args.push_back(CGF.EmitScalarExpr(E->getArg(1)));
484: 
485:   Value *DimValue =
486:       CGF.Builder.CreateIntrinsic(IntrinsicID, {Handle->getType()}, Args);
487: 
488:   Value *LastStore = nullptr;
489:   unsigned ArgIndex = HasLod ? 2 : 1;
490:   for (unsigned i = 0; i < NumRetComps; ++i) {
491:     const Expr *Arg = E->getArg(ArgIndex++);
492:     LValue DimOut = CGF.EmitLValue(Arg);
493:     Value *Elem = DimValue;
494:     if (NumRetComps > 1)
495:       Elem = CGF.Builder.CreateExtractElement(DimValue, i);
496: 
497:     // Handle float casting if needed
498:     if (Arg->getType()->isFloatingType())
499:       Elem = CGF.Builder.CreateUIToFP(
500:           Elem, llvm::Type::getFloatTy(CGF.getLLVMContext()));
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 501-520
```cpp
501: 
502:     LastStore = CGF.Builder.CreateStore(Elem, DimOut.getAddress());
503:   }
504:   return LastStore;
505: }
506: 
507: Value *CodeGenFunction::EmitHLSLBuiltinExpr(unsigned BuiltinID,
508:                                             const CallExpr *E,
509:                                             ReturnValueSlot ReturnValue) {
510:   if (!getLangOpts().HLSL)
511:     return nullptr;
512: 
513:   switch (BuiltinID) {
514:   case Builtin::BI__builtin_hlsl_adduint64: {
515:     Value *OpA = EmitScalarExpr(E->getArg(0));
516:     Value *OpB = EmitScalarExpr(E->getArg(1));
517:     QualType Arg0Ty = E->getArg(0)->getType();
518:     uint64_t NumElements = Arg0Ty->castAs<VectorType>()->getNumElements();
519:     assert(Arg0Ty == E->getArg(1)->getType() &&
520:            "AddUint64 operand types must match");
```
- **EN**: This block uses control flow (if, switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 521-540
```cpp
521:     assert(Arg0Ty->hasIntegerRepresentation() &&
522:            "AddUint64 operands must have an integer representation");
523:     assert((NumElements == 2 || NumElements == 4) &&
524:            "AddUint64 operands must have 2 or 4 elements");
525: 
526:     llvm::Value *LowA;
527:     llvm::Value *HighA;
528:     llvm::Value *LowB;
529:     llvm::Value *HighB;
530: 
531:     // Obtain low and high words of inputs A and B
532:     if (NumElements == 2) {
533:       LowA = Builder.CreateExtractElement(OpA, (uint64_t)0, "LowA");
534:       HighA = Builder.CreateExtractElement(OpA, (uint64_t)1, "HighA");
535:       LowB = Builder.CreateExtractElement(OpB, (uint64_t)0, "LowB");
536:       HighB = Builder.CreateExtractElement(OpB, (uint64_t)1, "HighB");
537:     } else {
538:       LowA = Builder.CreateShuffleVector(OpA, {0, 2}, "LowA");
539:       HighA = Builder.CreateShuffleVector(OpA, {1, 3}, "HighA");
540:       LowB = Builder.CreateShuffleVector(OpB, {0, 2}, "LowB");
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 541-560
```cpp
541:       HighB = Builder.CreateShuffleVector(OpB, {1, 3}, "HighB");
542:     }
543: 
544:     // Use an uadd_with_overflow to compute the sum of low words and obtain a
545:     // carry value
546:     llvm::Value *Carry;
547:     llvm::Value *LowSum = EmitOverflowIntrinsic(
548:         *this, Intrinsic::uadd_with_overflow, LowA, LowB, Carry);
549:     llvm::Value *ZExtCarry =
550:         Builder.CreateZExt(Carry, HighA->getType(), "CarryZExt");
551: 
552:     // Sum the high words and the carry
553:     llvm::Value *HighSum = Builder.CreateAdd(HighA, HighB, "HighSum");
554:     llvm::Value *HighSumPlusCarry =
555:         Builder.CreateAdd(HighSum, ZExtCarry, "HighSumPlusCarry");
556: 
557:     if (NumElements == 4) {
558:       return Builder.CreateShuffleVector(LowSum, HighSumPlusCarry, {0, 2, 1, 3},
559:                                          "hlsl.AddUint64");
560:     }
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 561-580
```cpp
561: 
562:     llvm::Value *Result = PoisonValue::get(OpA->getType());
563:     Result = Builder.CreateInsertElement(Result, LowSum, (uint64_t)0,
564:                                          "hlsl.AddUint64.upto0");
565:     Result = Builder.CreateInsertElement(Result, HighSumPlusCarry, (uint64_t)1,
566:                                          "hlsl.AddUint64");
567:     return Result;
568:   }
569:   case Builtin::BI__builtin_hlsl_resource_getpointer:
570:   case Builtin::BI__builtin_hlsl_resource_getpointer_typed: {
571:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
572:     bool IsIndexed =
573:         BuiltinID == Builtin::BI__builtin_hlsl_resource_getpointer_typed ||
574:         E->getNumArgs() > 1;
575: 
576:     llvm::Type *RetTy = ConvertType(E->getType());
577:     if (IsIndexed) {
578:       Value *IndexOp = EmitScalarExpr(E->getArg(1));
579:       return Builder.CreateIntrinsic(
580:           RetTy, CGM.getHLSLRuntime().getCreateResourceGetPointerIntrinsic(),
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 581-600
```cpp
581:           ArrayRef<Value *>{HandleOp, IndexOp});
582:     }
583:     return Builder.CreateIntrinsic(
584:         RetTy, CGM.getHLSLRuntime().getCreateResourceGetBasePointerIntrinsic(),
585:         ArrayRef<Value *>{HandleOp});
586:   }
587:   case Builtin::BI__builtin_hlsl_resource_sample: {
588:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
589:     Value *SamplerOp = EmitScalarExpr(E->getArg(1));
590:     Value *CoordOp = EmitScalarExpr(E->getArg(2));
591: 
592:     SmallVector<Value *, 4> Args;
593:     Args.push_back(HandleOp);
594:     Args.push_back(SamplerOp);
595:     Args.push_back(CoordOp);
596:     Args.push_back(
597:         emitHlslOffset(*this, E, 3, getOffsetType(CGM, CoordOp->getType())));
598: 
599:     llvm::Type *RetTy = ConvertType(E->getType());
600:     if (E->getNumArgs() <= 4) {
```
- **EN**: This block defines callable entry points like `emitHlslOffset`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitHlslOffset`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 601-620
```cpp
601:       return Builder.CreateIntrinsic(
602:           RetTy, CGM.getHLSLRuntime().getSampleIntrinsic(), Args);
603:     }
604: 
605:     Args.push_back(emitHlslClamp(*this, E, 4));
606:     return Builder.CreateIntrinsic(
607:         RetTy, CGM.getHLSLRuntime().getSampleClampIntrinsic(), Args);
608:   }
609:   case Builtin::BI__builtin_hlsl_resource_sample_bias: {
610:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
611:     Value *SamplerOp = EmitScalarExpr(E->getArg(1));
612:     Value *CoordOp = EmitScalarExpr(E->getArg(2));
613:     Value *BiasOp = EmitScalarExpr(E->getArg(3));
614:     if (BiasOp->getType() != Builder.getFloatTy())
615:       BiasOp = Builder.CreateFPCast(BiasOp, Builder.getFloatTy());
616: 
617:     SmallVector<Value *, 6> Args; // Max 6 arguments for SampleBias
618:     Args.push_back(HandleOp);
619:     Args.push_back(SamplerOp);
620:     Args.push_back(CoordOp);
```
- **EN**: This block uses control flow (if, for, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for, case）细化 LLVM IR 生成 行为。

### Lines 621-640
```cpp
621:     Args.push_back(BiasOp);
622:     Args.push_back(
623:         emitHlslOffset(*this, E, 4, getOffsetType(CGM, CoordOp->getType())));
624: 
625:     llvm::Type *RetTy = ConvertType(E->getType());
626:     if (E->getNumArgs() <= 5)
627:       return Builder.CreateIntrinsic(
628:           RetTy, CGM.getHLSLRuntime().getSampleBiasIntrinsic(), Args);
629: 
630:     Args.push_back(emitHlslClamp(*this, E, 5));
631:     return Builder.CreateIntrinsic(
632:         RetTy, CGM.getHLSLRuntime().getSampleBiasClampIntrinsic(), Args);
633:   }
634:   case Builtin::BI__builtin_hlsl_resource_sample_grad: {
635:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
636:     Value *SamplerOp = EmitScalarExpr(E->getArg(1));
637:     Value *CoordOp = EmitScalarExpr(E->getArg(2));
638:     Value *DDXOp = EmitScalarExpr(E->getArg(3));
639:     Value *DDYOp = EmitScalarExpr(E->getArg(4));
640: 
```
- **EN**: This block defines callable entry points like `emitHlslOffset`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitHlslOffset`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 641-660
```cpp
641:     SmallVector<Value *, 7> Args;
642:     Args.push_back(HandleOp);
643:     Args.push_back(SamplerOp);
644:     Args.push_back(CoordOp);
645:     Args.push_back(DDXOp);
646:     Args.push_back(DDYOp);
647:     Args.push_back(
648:         emitHlslOffset(*this, E, 5, getOffsetType(CGM, CoordOp->getType())));
649: 
650:     llvm::Type *RetTy = ConvertType(E->getType());
651: 
652:     if (E->getNumArgs() <= 6) {
653:       return Builder.CreateIntrinsic(
654:           RetTy, CGM.getHLSLRuntime().getSampleGradIntrinsic(), Args);
655:     }
656: 
657:     Args.push_back(emitHlslClamp(*this, E, 6));
658:     return Builder.CreateIntrinsic(
659:         RetTy, CGM.getHLSLRuntime().getSampleGradClampIntrinsic(), Args);
660:   }
```
- **EN**: This block defines callable entry points like `emitHlslOffset`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitHlslOffset`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 661-680
```cpp
661:   case Builtin::BI__builtin_hlsl_resource_sample_level: {
662:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
663:     Value *SamplerOp = EmitScalarExpr(E->getArg(1));
664:     Value *CoordOp = EmitScalarExpr(E->getArg(2));
665:     Value *LODOp = EmitScalarExpr(E->getArg(3));
666:     if (LODOp->getType() != Builder.getFloatTy())
667:       LODOp = Builder.CreateFPCast(LODOp, Builder.getFloatTy());
668: 
669:     SmallVector<Value *, 5> Args; // Max 5 arguments for SampleLevel
670:     Args.push_back(HandleOp);
671:     Args.push_back(SamplerOp);
672:     Args.push_back(CoordOp);
673:     Args.push_back(LODOp);
674:     Args.push_back(
675:         emitHlslOffset(*this, E, 4, getOffsetType(CGM, CoordOp->getType())));
676: 
677:     llvm::Type *RetTy = ConvertType(E->getType());
678:     return Builder.CreateIntrinsic(
679:         RetTy, CGM.getHLSLRuntime().getSampleLevelIntrinsic(), Args);
680:   }
```
- **EN**: This block defines callable entry points like `emitHlslOffset`; uses control flow (if, for, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitHlslOffset`；通过控制流（if, for, case）细化 LLVM IR 生成 行为。

### Lines 681-700
```cpp
681:   case Builtin::BI__builtin_hlsl_resource_load_level: {
682:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
683:     Value *CoordLODOp = EmitScalarExpr(E->getArg(1));
684: 
685:     auto *CoordLODVecTy = cast<llvm::FixedVectorType>(CoordLODOp->getType());
686:     unsigned NumElts = CoordLODVecTy->getNumElements();
687:     assert(NumElts >= 2 && "CoordLOD must have at least 2 elements");
688: 
689:     // Split CoordLOD into Coord and LOD
690:     SmallVector<int, 4> Mask;
691:     for (unsigned I = 0; I < NumElts - 1; ++I)
692:       Mask.push_back(I);
693: 
694:     Value *CoordOp =
695:         Builder.CreateShuffleVector(CoordLODOp, Mask, "hlsl.load.coord");
696:     Value *LODOp =
697:         Builder.CreateExtractElement(CoordLODOp, NumElts - 1, "hlsl.load.lod");
698: 
699:     SmallVector<Value *, 4> Args;
700:     Args.push_back(HandleOp);
```
- **EN**: This block uses control flow (for, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（for, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 701-720
```cpp
701:     Args.push_back(CoordOp);
702:     Args.push_back(LODOp);
703:     Args.push_back(
704:         emitHlslOffset(*this, E, 2, getOffsetType(CGM, CoordOp->getType())));
705: 
706:     llvm::Type *RetTy = ConvertType(E->getType());
707:     return Builder.CreateIntrinsic(
708:         RetTy, CGM.getHLSLRuntime().getLoadLevelIntrinsic(), Args);
709:   }
710:   case Builtin::BI__builtin_hlsl_resource_sample_cmp: {
711:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
712:     Value *SamplerOp = EmitScalarExpr(E->getArg(1));
713:     Value *CoordOp = EmitScalarExpr(E->getArg(2));
714:     Value *CmpOp = EmitScalarExpr(E->getArg(3));
715:     if (CmpOp->getType() != Builder.getFloatTy())
716:       CmpOp = Builder.CreateFPCast(CmpOp, Builder.getFloatTy());
717: 
718:     SmallVector<Value *, 6> Args; // Max 6 arguments for SampleCmp
719:     Args.push_back(HandleOp);
720:     Args.push_back(SamplerOp);
```
- **EN**: This block defines callable entry points like `emitHlslOffset`; uses control flow (if, for, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitHlslOffset`；通过控制流（if, for, case）细化 LLVM IR 生成 行为。

### Lines 721-740
```cpp
721:     Args.push_back(CoordOp);
722:     Args.push_back(CmpOp);
723:     Args.push_back(
724:         emitHlslOffset(*this, E, 4, getOffsetType(CGM, CoordOp->getType())));
725: 
726:     llvm::Type *RetTy = ConvertType(E->getType());
727:     if (E->getNumArgs() <= 5) {
728:       return Builder.CreateIntrinsic(
729:           RetTy, CGM.getHLSLRuntime().getSampleCmpIntrinsic(), Args);
730:     }
731: 
732:     Args.push_back(emitHlslClamp(*this, E, 5));
733:     return Builder.CreateIntrinsic(
734:         RetTy, CGM.getHLSLRuntime().getSampleCmpClampIntrinsic(), Args);
735:   }
736:   case Builtin::BI__builtin_hlsl_resource_sample_cmp_level_zero: {
737:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
738:     Value *SamplerOp = EmitScalarExpr(E->getArg(1));
739:     Value *CoordOp = EmitScalarExpr(E->getArg(2));
740:     Value *CmpOp = EmitScalarExpr(E->getArg(3));
```
- **EN**: This block defines callable entry points like `emitHlslOffset`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitHlslOffset`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 741-760
```cpp
741:     if (CmpOp->getType() != Builder.getFloatTy())
742:       CmpOp = Builder.CreateFPCast(CmpOp, Builder.getFloatTy());
743: 
744:     SmallVector<Value *, 5> Args;
745:     Args.push_back(HandleOp);
746:     Args.push_back(SamplerOp);
747:     Args.push_back(CoordOp);
748:     Args.push_back(CmpOp);
749: 
750:     Args.push_back(
751:         emitHlslOffset(*this, E, 4, getOffsetType(CGM, CoordOp->getType())));
752: 
753:     llvm::Type *RetTy = ConvertType(E->getType());
754:     return Builder.CreateIntrinsic(
755:         RetTy, CGM.getHLSLRuntime().getSampleCmpLevelZeroIntrinsic(), Args);
756:   }
757:   case Builtin::BI__builtin_hlsl_resource_calculate_lod: {
758:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
759:     Value *SamplerOp = EmitScalarExpr(E->getArg(1));
760:     Value *CoordOp = EmitScalarExpr(E->getArg(2));
```
- **EN**: This block defines callable entry points like `emitHlslOffset`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitHlslOffset`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 761-780
```cpp
761: 
762:     return Builder.CreateIntrinsic(
763:         ConvertType(E->getType()),
764:         CGM.getHLSLRuntime().getCalculateLodIntrinsic(),
765:         {HandleOp, SamplerOp, CoordOp});
766:   }
767:   case Builtin::BI__builtin_hlsl_resource_calculate_lod_unclamped: {
768:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
769:     Value *SamplerOp = EmitScalarExpr(E->getArg(1));
770:     Value *CoordOp = EmitScalarExpr(E->getArg(2));
771: 
772:     return Builder.CreateIntrinsic(
773:         ConvertType(E->getType()),
774:         CGM.getHLSLRuntime().getCalculateLodUnclampedIntrinsic(),
775:         {HandleOp, SamplerOp, CoordOp});
776:   }
777:   case Builtin::BI__builtin_hlsl_resource_gather: {
778:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
779:     Value *SamplerOp = EmitScalarExpr(E->getArg(1));
780:     Value *CoordOp = EmitScalarExpr(E->getArg(2));
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 781-800
```cpp
781:     Value *ComponentOp = EmitScalarExpr(E->getArg(3));
782:     if (ComponentOp->getType() != Builder.getInt32Ty())
783:       ComponentOp = Builder.CreateIntCast(ComponentOp, Builder.getInt32Ty(),
784:                                           /*isSigned=*/false);
785: 
786:     SmallVector<Value *, 5> Args;
787:     Args.push_back(HandleOp);
788:     Args.push_back(SamplerOp);
789:     Args.push_back(CoordOp);
790:     Args.push_back(ComponentOp);
791:     Args.push_back(
792:         emitHlslOffset(*this, E, 4, getOffsetType(CGM, CoordOp->getType())));
793: 
794:     llvm::Type *RetTy = ConvertType(E->getType());
795:     return Builder.CreateIntrinsic(
796:         RetTy, CGM.getHLSLRuntime().getGatherIntrinsic(), Args);
797:   }
798:   case Builtin::BI__builtin_hlsl_resource_gather_cmp: {
799:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
800:     Value *SamplerOp = EmitScalarExpr(E->getArg(1));
```
- **EN**: This block defines callable entry points like `emitHlslOffset`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitHlslOffset`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 801-820
```cpp
801:     Value *CoordOp = EmitScalarExpr(E->getArg(2));
802:     Value *CompareOp = EmitScalarExpr(E->getArg(3));
803:     if (CompareOp->getType() != Builder.getFloatTy())
804:       CompareOp = Builder.CreateFPCast(CompareOp, Builder.getFloatTy());
805: 
806:     SmallVector<Value *, 6> Args;
807:     Args.push_back(HandleOp);
808:     Args.push_back(SamplerOp);
809:     Args.push_back(CoordOp);
810:     Args.push_back(CompareOp);
811: 
812:     if (CGM.getTarget().getTriple().isDXIL()) {
813:       Value *ComponentOp = EmitScalarExpr(E->getArg(4));
814:       if (ComponentOp->getType() != Builder.getInt32Ty())
815:         ComponentOp = Builder.CreateIntCast(ComponentOp, Builder.getInt32Ty(),
816:                                             /*isSigned=*/false);
817:       Args.push_back(ComponentOp);
818:     }
819: 
820:     Args.push_back(
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 821-840
```cpp
821:         emitHlslOffset(*this, E, 5, getOffsetType(CGM, CoordOp->getType())));
822: 
823:     llvm::Type *RetTy = ConvertType(E->getType());
824:     return Builder.CreateIntrinsic(
825:         RetTy, CGM.getHLSLRuntime().getGatherCmpIntrinsic(), Args);
826:   }
827:   case Builtin::BI__builtin_hlsl_resource_load_with_status:
828:   case Builtin::BI__builtin_hlsl_resource_load_with_status_typed: {
829:     Value *HandleOp = EmitScalarExpr(E->getArg(0));
830:     Value *IndexOp = EmitScalarExpr(E->getArg(1));
831: 
832:     // Get the *address* of the status argument to write to it by reference
833:     LValue StatusLVal = EmitLValue(E->getArg(2));
834:     Address StatusAddr = StatusLVal.getAddress();
835: 
836:     QualType HandleTy = E->getArg(0)->getType();
837:     const HLSLAttributedResourceType *RT =
838:         HandleTy->getAs<HLSLAttributedResourceType>();
839:     assert(CGM.getTarget().getTriple().getArch() == llvm::Triple::dxil &&
840:            "Only DXIL currently implements load with status");
```
- **EN**: This block defines callable entry points like `emitHlslOffset`; uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitHlslOffset`；通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 841-860
```cpp
841: 
842:     Intrinsic::ID IntrID = RT->getAttrs().RawBuffer
843:                                ? llvm::Intrinsic::dx_resource_load_rawbuffer
844:                                : llvm::Intrinsic::dx_resource_load_typedbuffer;
845: 
846:     llvm::Type *DataTy = ConvertType(E->getType());
847:     llvm::Type *RetTy = llvm::StructType::get(Builder.getContext(),
848:                                               {DataTy, Builder.getInt1Ty()});
849: 
850:     SmallVector<Value *, 3> Args;
851:     Args.push_back(HandleOp);
852:     Args.push_back(IndexOp);
853: 
854:     if (RT->isRaw()) {
855:       Value *Offset = Builder.getInt32(0);
856:       // The offset parameter needs to be poison for ByteAddressBuffer
857:       if (!RT->isStructured())
858:         Offset = llvm::PoisonValue::get(Builder.getInt32Ty());
859:       Args.push_back(Offset);
860:     }
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 861-880
```cpp
861: 
862:     // The load intrinsics give us a (T value, i1 status) pair -
863:     // shepherd these into the return value and out reference respectively.
864:     Value *ResRet =
865:         Builder.CreateIntrinsic(RetTy, IntrID, Args, {}, "ld.struct");
866:     Value *LoadedValue = Builder.CreateExtractValue(ResRet, {0}, "ld.value");
867:     Value *StatusBit = Builder.CreateExtractValue(ResRet, {1}, "ld.status");
868:     Value *ExtendedStatus =
869:         Builder.CreateZExt(StatusBit, Builder.getInt32Ty(), "ld.status.ext");
870:     Builder.CreateStore(ExtendedStatus, StatusAddr);
871: 
872:     return LoadedValue;
873:   }
874:   case Builtin::BI__builtin_hlsl_resource_uninitializedhandle: {
875:     llvm::Type *HandleTy = CGM.getTypes().ConvertType(E->getType());
876:     return llvm::PoisonValue::get(HandleTy);
877:   }
878:   case Builtin::BI__builtin_hlsl_resource_handlefrombinding: {
879:     llvm::Type *HandleTy = CGM.getTypes().ConvertType(E->getType());
880:     Value *RegisterOp = EmitScalarExpr(E->getArg(1));
```
- **EN**: This block defines callable entry points like `get`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 881-900
```cpp
881:     Value *SpaceOp = EmitScalarExpr(E->getArg(2));
882:     Value *RangeOp = EmitScalarExpr(E->getArg(3));
883:     Value *IndexOp = EmitScalarExpr(E->getArg(4));
884:     Value *Name = EmitScalarExpr(E->getArg(5));
885:     llvm::Intrinsic::ID IntrinsicID =
886:         CGM.getHLSLRuntime().getCreateHandleFromBindingIntrinsic();
887:     SmallVector<Value *> Args{SpaceOp, RegisterOp, RangeOp, IndexOp, Name};
888:     return Builder.CreateIntrinsic(HandleTy, IntrinsicID, Args);
889:   }
890:   case Builtin::BI__builtin_hlsl_resource_handlefromimplicitbinding: {
891:     llvm::Type *HandleTy = CGM.getTypes().ConvertType(E->getType());
892:     Value *OrderID = EmitScalarExpr(E->getArg(1));
893:     Value *SpaceOp = EmitScalarExpr(E->getArg(2));
894:     Value *RangeOp = EmitScalarExpr(E->getArg(3));
895:     Value *IndexOp = EmitScalarExpr(E->getArg(4));
896:     Value *Name = EmitScalarExpr(E->getArg(5));
897:     llvm::Intrinsic::ID IntrinsicID =
898:         CGM.getHLSLRuntime().getCreateHandleFromImplicitBindingIntrinsic();
899:     SmallVector<Value *> Args{OrderID, SpaceOp, RangeOp, IndexOp, Name};
900:     return Builder.CreateIntrinsic(HandleTy, IntrinsicID, Args);
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 901-920
```cpp
901:   }
902:   case Builtin::BI__builtin_hlsl_resource_counterhandlefromimplicitbinding: {
903:     Value *MainHandle = EmitScalarExpr(E->getArg(0));
904:     if (!CGM.getTriple().isSPIRV())
905:       return MainHandle;
906: 
907:     llvm::Type *HandleTy = CGM.getTypes().ConvertType(E->getType());
908:     Value *OrderID = EmitScalarExpr(E->getArg(1));
909:     Value *SpaceOp = EmitScalarExpr(E->getArg(2));
910:     llvm::Intrinsic::ID IntrinsicID =
911:         llvm::Intrinsic::spv_resource_counterhandlefromimplicitbinding;
912:     SmallVector<Value *> Args{MainHandle, OrderID, SpaceOp};
913:     return Builder.CreateIntrinsic(HandleTy, IntrinsicID, Args);
914:   }
915:   case Builtin::BI__builtin_hlsl_resource_nonuniformindex: {
916:     Value *IndexOp = EmitScalarExpr(E->getArg(0));
917:     llvm::Type *RetTy = ConvertType(E->getType());
918:     return Builder.CreateIntrinsic(
919:         RetTy, CGM.getHLSLRuntime().getNonUniformResourceIndexIntrinsic(),
920:         ArrayRef<Value *>{IndexOp});
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 921-940
```cpp
921:   }
922:   case Builtin::BI__builtin_hlsl_resource_getdimensions_x:
923:   case Builtin::BI__builtin_hlsl_resource_getdimensions_x_float:
924:     return emitGetDimensions(*this, E,
925:                              CGM.getHLSLRuntime().getGetDimensionsXIntrinsic(),
926:                              1, /*HasLod=*/false);
927:   case Builtin::BI__builtin_hlsl_resource_getdimensions_xy:
928:   case Builtin::BI__builtin_hlsl_resource_getdimensions_xy_float:
929:     return emitGetDimensions(*this, E,
930:                              CGM.getHLSLRuntime().getGetDimensionsXYIntrinsic(),
931:                              2, /*HasLod=*/false);
932:   case Builtin::BI__builtin_hlsl_resource_getdimensions_levels_xy:
933:   case Builtin::BI__builtin_hlsl_resource_getdimensions_levels_xy_float:
934:     return emitGetDimensions(
935:         *this, E, CGM.getHLSLRuntime().getGetDimensionsLevelsXYIntrinsic(), 3,
936:         /*HasLod=*/true);
937:   case Builtin::BI__builtin_hlsl_resource_getstride: {
938:     LValue Stride = EmitLValue(E->getArg(1));
939:     return emitBufferStride(this, E->getArg(0), Stride);
940:   }
```
- **EN**: This block defines callable entry points like `emitGetDimensions`, `emitBufferStride`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitGetDimensions`, `emitBufferStride`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 941-960
```cpp
941:   case Builtin::BI__builtin_hlsl_all: {
942:     Value *Op0 = EmitScalarExpr(E->getArg(0));
943:     return Builder.CreateIntrinsic(
944:         /*ReturnType=*/llvm::Type::getInt1Ty(getLLVMContext()),
945:         CGM.getHLSLRuntime().getAllIntrinsic(), ArrayRef<Value *>{Op0}, nullptr,
946:         "hlsl.all");
947:   }
948:   case Builtin::BI__builtin_hlsl_and: {
949:     Value *Op0 = EmitScalarExpr(E->getArg(0));
950:     Value *Op1 = EmitScalarExpr(E->getArg(1));
951:     return Builder.CreateAnd(Op0, Op1, "hlsl.and");
952:   }
953:   case Builtin::BI__builtin_hlsl_or: {
954:     Value *Op0 = EmitScalarExpr(E->getArg(0));
955:     Value *Op1 = EmitScalarExpr(E->getArg(1));
956:     return Builder.CreateOr(Op0, Op1, "hlsl.or");
957:   }
958:   case Builtin::BI__builtin_hlsl_any: {
959:     Value *Op0 = EmitScalarExpr(E->getArg(0));
960:     return Builder.CreateIntrinsic(
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 961-980
```cpp
961:         /*ReturnType=*/llvm::Type::getInt1Ty(getLLVMContext()),
962:         CGM.getHLSLRuntime().getAnyIntrinsic(), ArrayRef<Value *>{Op0}, nullptr,
963:         "hlsl.any");
964:   }
965:   case Builtin::BI__builtin_hlsl_asdouble:
966:     return handleAsDoubleBuiltin(*this, E);
967:   case Builtin::BI__builtin_hlsl_elementwise_clamp: {
968:     Value *OpX = EmitScalarExpr(E->getArg(0));
969:     Value *OpMin = EmitScalarExpr(E->getArg(1));
970:     Value *OpMax = EmitScalarExpr(E->getArg(2));
971: 
972:     QualType Ty = E->getArg(0)->getType();
973:     if (auto *VecTy = Ty->getAs<VectorType>())
974:       Ty = VecTy->getElementType();
975: 
976:     Intrinsic::ID Intr;
977:     if (Ty->isFloatingType()) {
978:       Intr = CGM.getHLSLRuntime().getNClampIntrinsic();
979:     } else if (Ty->isUnsignedIntegerType()) {
980:       Intr = CGM.getHLSLRuntime().getUClampIntrinsic();
```
- **EN**: This block defines callable entry points like `handleAsDoubleBuiltin`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `handleAsDoubleBuiltin`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 981-1000
```cpp
 981:     } else {
 982:       assert(Ty->isSignedIntegerType());
 983:       Intr = CGM.getHLSLRuntime().getSClampIntrinsic();
 984:     }
 985:     return Builder.CreateIntrinsic(
 986:         /*ReturnType=*/OpX->getType(), Intr,
 987:         ArrayRef<Value *>{OpX, OpMin, OpMax}, nullptr, "hlsl.clamp");
 988:   }
 989:   case Builtin::BI__builtin_hlsl_crossf16:
 990:   case Builtin::BI__builtin_hlsl_crossf32: {
 991:     Value *Op0 = EmitScalarExpr(E->getArg(0));
 992:     Value *Op1 = EmitScalarExpr(E->getArg(1));
 993:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
 994:            E->getArg(1)->getType()->hasFloatingRepresentation() &&
 995:            "cross operands must have a float representation");
 996:     // make sure each vector has exactly 3 elements
 997:     assert(
 998:         E->getArg(0)->getType()->castAs<VectorType>()->getNumElements() == 3 &&
 999:         E->getArg(1)->getType()->castAs<VectorType>()->getNumElements() == 3 &&
1000:         "input vectors must have 3 elements each");
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1001-1020
```cpp
1001:     return Builder.CreateIntrinsic(
1002:         /*ReturnType=*/Op0->getType(), CGM.getHLSLRuntime().getCrossIntrinsic(),
1003:         ArrayRef<Value *>{Op0, Op1}, nullptr, "hlsl.cross");
1004:   }
1005:   case Builtin::BI__builtin_hlsl_dot: {
1006:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1007:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1008:     llvm::Type *T0 = Op0->getType();
1009:     llvm::Type *T1 = Op1->getType();
1010: 
1011:     // If the arguments are scalars, just emit a multiply
1012:     if (!T0->isVectorTy() && !T1->isVectorTy()) {
1013:       if (T0->isFloatingPointTy())
1014:         return Builder.CreateFMul(Op0, Op1, "hlsl.dot");
1015: 
1016:       if (T0->isIntegerTy())
1017:         return Builder.CreateMul(Op0, Op1, "hlsl.dot");
1018: 
1019:       llvm_unreachable(
1020:           "Scalar dot product is only supported on ints and floats.");
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1021-1040
```cpp
1021:     }
1022:     // For vectors, validate types and emit the appropriate intrinsic
1023:     assert(CGM.getContext().hasSameUnqualifiedType(E->getArg(0)->getType(),
1024:                                                    E->getArg(1)->getType()) &&
1025:            "Dot product operands must have the same type.");
1026: 
1027:     auto *VecTy0 = E->getArg(0)->getType()->castAs<VectorType>();
1028:     assert(VecTy0 && "Dot product argument must be a vector.");
1029: 
1030:     return Builder.CreateIntrinsic(
1031:         /*ReturnType=*/T0->getScalarType(),
1032:         getDotProductIntrinsic(CGM.getHLSLRuntime(), VecTy0->getElementType()),
1033:         ArrayRef<Value *>{Op0, Op1}, nullptr, "hlsl.dot");
1034:   }
1035:   case Builtin::BI__builtin_hlsl_dot4add_i8packed: {
1036:     Value *X = EmitScalarExpr(E->getArg(0));
1037:     Value *Y = EmitScalarExpr(E->getArg(1));
1038:     Value *Acc = EmitScalarExpr(E->getArg(2));
1039: 
1040:     Intrinsic::ID ID = CGM.getHLSLRuntime().getDot4AddI8PackedIntrinsic();
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1041-1060
```cpp
1041:     // Note that the argument order disagrees between the builtin and the
1042:     // intrinsic here.
1043:     return Builder.CreateIntrinsic(
1044:         /*ReturnType=*/Acc->getType(), ID, ArrayRef<Value *>{Acc, X, Y},
1045:         nullptr, "hlsl.dot4add.i8packed");
1046:   }
1047:   case Builtin::BI__builtin_hlsl_dot4add_u8packed: {
1048:     Value *X = EmitScalarExpr(E->getArg(0));
1049:     Value *Y = EmitScalarExpr(E->getArg(1));
1050:     Value *Acc = EmitScalarExpr(E->getArg(2));
1051: 
1052:     Intrinsic::ID ID = CGM.getHLSLRuntime().getDot4AddU8PackedIntrinsic();
1053:     // Note that the argument order disagrees between the builtin and the
1054:     // intrinsic here.
1055:     return Builder.CreateIntrinsic(
1056:         /*ReturnType=*/Acc->getType(), ID, ArrayRef<Value *>{Acc, X, Y},
1057:         nullptr, "hlsl.dot4add.u8packed");
1058:   }
1059:   case Builtin::BI__builtin_hlsl_elementwise_firstbithigh: {
1060:     Value *X = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 1061-1080
```cpp
1061: 
1062:     return Builder.CreateIntrinsic(
1063:         /*ReturnType=*/ConvertType(E->getType()),
1064:         getFirstBitHighIntrinsic(CGM.getHLSLRuntime(), E->getArg(0)->getType()),
1065:         ArrayRef<Value *>{X}, nullptr, "hlsl.firstbithigh");
1066:   }
1067:   case Builtin::BI__builtin_hlsl_elementwise_firstbitlow: {
1068:     Value *X = EmitScalarExpr(E->getArg(0));
1069: 
1070:     return Builder.CreateIntrinsic(
1071:         /*ReturnType=*/ConvertType(E->getType()),
1072:         CGM.getHLSLRuntime().getFirstBitLowIntrinsic(), ArrayRef<Value *>{X},
1073:         nullptr, "hlsl.firstbitlow");
1074:   }
1075:   case Builtin::BI__builtin_hlsl_lerp: {
1076:     Value *X = EmitScalarExpr(E->getArg(0));
1077:     Value *Y = EmitScalarExpr(E->getArg(1));
1078:     Value *S = EmitScalarExpr(E->getArg(2));
1079:     if (!E->getArg(0)->getType()->hasFloatingRepresentation())
1080:       llvm_unreachable("lerp operand must have a float representation");
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1081-1100
```cpp
1081:     return Builder.CreateIntrinsic(
1082:         /*ReturnType=*/X->getType(), CGM.getHLSLRuntime().getLerpIntrinsic(),
1083:         ArrayRef<Value *>{X, Y, S}, nullptr, "hlsl.lerp");
1084:   }
1085:   case Builtin::BI__builtin_hlsl_normalize: {
1086:     Value *X = EmitScalarExpr(E->getArg(0));
1087: 
1088:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
1089:            "normalize operand must have a float representation");
1090: 
1091:     return Builder.CreateIntrinsic(
1092:         /*ReturnType=*/X->getType(),
1093:         CGM.getHLSLRuntime().getNormalizeIntrinsic(), ArrayRef<Value *>{X},
1094:         nullptr, "hlsl.normalize");
1095:   }
1096:   case Builtin::BI__builtin_hlsl_elementwise_degrees: {
1097:     Value *X = EmitScalarExpr(E->getArg(0));
1098: 
1099:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
1100:            "degree operand must have a float representation");
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1101-1120
```cpp
1101: 
1102:     return Builder.CreateIntrinsic(
1103:         /*ReturnType=*/X->getType(), CGM.getHLSLRuntime().getDegreesIntrinsic(),
1104:         ArrayRef<Value *>{X}, nullptr, "hlsl.degrees");
1105:   }
1106:   case Builtin::BI__builtin_hlsl_elementwise_f16tof32: {
1107:     return handleElementwiseF16ToF32(*this, E);
1108:   }
1109:   case Builtin::BI__builtin_hlsl_elementwise_f32tof16: {
1110:     return handleElementwiseF32ToF16(*this, E);
1111:   }
1112:   case Builtin::BI__builtin_hlsl_elementwise_frac: {
1113:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1114:     if (!E->getArg(0)->getType()->hasFloatingRepresentation())
1115:       llvm_unreachable("frac operand must have a float representation");
1116:     return Builder.CreateIntrinsic(
1117:         /*ReturnType=*/Op0->getType(), CGM.getHLSLRuntime().getFracIntrinsic(),
1118:         ArrayRef<Value *>{Op0}, nullptr, "hlsl.frac");
1119:   }
1120:   case Builtin::BI__builtin_hlsl_elementwise_isinf: {
```
- **EN**: This block defines callable entry points like `handleElementwiseF16ToF32`, `handleElementwiseF32ToF16`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `handleElementwiseF16ToF32`, `handleElementwiseF32ToF16`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1121-1140
```cpp
1121:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1122:     llvm::Type *Xty = Op0->getType();
1123:     llvm::Type *retType = llvm::Type::getInt1Ty(this->getLLVMContext());
1124:     if (Xty->isVectorTy()) {
1125:       auto *XVecTy = E->getArg(0)->getType()->castAs<VectorType>();
1126:       retType = llvm::VectorType::get(
1127:           retType, ElementCount::getFixed(XVecTy->getNumElements()));
1128:     }
1129:     if (!E->getArg(0)->getType()->hasFloatingRepresentation())
1130:       llvm_unreachable("isinf operand must have a float representation");
1131:     return Builder.CreateIntrinsic(
1132:         retType, CGM.getHLSLRuntime().getIsInfIntrinsic(),
1133:         ArrayRef<Value *>{Op0}, nullptr, "hlsl.isinf");
1134:   }
1135:   case Builtin::BI__builtin_hlsl_elementwise_isnan: {
1136:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1137:     llvm::Type *Xty = Op0->getType();
1138:     llvm::Type *retType = llvm::Type::getInt1Ty(this->getLLVMContext());
1139:     if (Xty->isVectorTy()) {
1140:       auto *XVecTy = E->getArg(0)->getType()->castAs<VectorType>();
```
- **EN**: This block defines callable entry points like `getFixed`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getFixed`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1141-1160
```cpp
1141:       retType = llvm::VectorType::get(
1142:           retType, ElementCount::getFixed(XVecTy->getNumElements()));
1143:     }
1144:     if (!E->getArg(0)->getType()->hasFloatingRepresentation())
1145:       llvm_unreachable("isnan operand must have a float representation");
1146:     return Builder.CreateIntrinsic(
1147:         retType, CGM.getHLSLRuntime().getIsNaNIntrinsic(),
1148:         ArrayRef<Value *>{Op0}, nullptr, "hlsl.isnan");
1149:   }
1150:   case Builtin::BI__builtin_hlsl_mad: {
1151:     Value *M = EmitScalarExpr(E->getArg(0));
1152:     Value *A = EmitScalarExpr(E->getArg(1));
1153:     Value *B = EmitScalarExpr(E->getArg(2));
1154:     if (E->getArg(0)->getType()->hasFloatingRepresentation())
1155:       return Builder.CreateIntrinsic(
1156:           /*ReturnType*/ M->getType(), Intrinsic::fmuladd,
1157:           ArrayRef<Value *>{M, A, B}, nullptr, "hlsl.fmad");
1158: 
1159:     if (E->getArg(0)->getType()->hasSignedIntegerRepresentation()) {
1160:       if (CGM.getTarget().getTriple().getArch() == llvm::Triple::dxil)
```
- **EN**: This block defines callable entry points like `getFixed`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getFixed`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1161-1180
```cpp
1161:         return Builder.CreateIntrinsic(
1162:             /*ReturnType*/ M->getType(), Intrinsic::dx_imad,
1163:             ArrayRef<Value *>{M, A, B}, nullptr, "dx.imad");
1164: 
1165:       Value *Mul = Builder.CreateNSWMul(M, A);
1166:       return Builder.CreateNSWAdd(Mul, B);
1167:     }
1168:     assert(E->getArg(0)->getType()->hasUnsignedIntegerRepresentation());
1169:     if (CGM.getTarget().getTriple().getArch() == llvm::Triple::dxil)
1170:       return Builder.CreateIntrinsic(
1171:           /*ReturnType=*/M->getType(), Intrinsic::dx_umad,
1172:           ArrayRef<Value *>{M, A, B}, nullptr, "dx.umad");
1173: 
1174:     Value *Mul = Builder.CreateNUWMul(M, A);
1175:     return Builder.CreateNUWAdd(Mul, B);
1176:   }
1177:   case Builtin::BI__builtin_hlsl_mul: {
1178:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1179:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1180:     QualType QTy0 = E->getArg(0)->getType();
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1181-1200
```cpp
1181:     QualType QTy1 = E->getArg(1)->getType();
1182: 
1183:     bool IsVec0 = QTy0->isVectorType();
1184:     bool IsVec1 = QTy1->isVectorType();
1185:     bool IsMat0 = QTy0->isConstantMatrixType();
1186:     bool IsMat1 = QTy1->isConstantMatrixType();
1187: 
1188:     // The matrix multiply intrinsic only operates on column-major order
1189:     // matrices. Therefore matrix memory layout transforms must be inserted
1190:     // before and after matrix multiply intrinsics.
1191:     bool IsRowMajor = getLangOpts().getDefaultMatrixMemoryLayout() ==
1192:                       LangOptions::MatrixMemoryLayout::MatrixRowMajor;
1193: 
1194:     llvm::MatrixBuilder MB(Builder);
1195:     if (IsVec0 && IsMat1) {
1196:       unsigned N = QTy0->castAs<VectorType>()->getNumElements();
1197:       auto *MatTy = QTy1->castAs<ConstantMatrixType>();
1198:       unsigned Rows = MatTy->getNumRows();
1199:       unsigned Cols = MatTy->getNumColumns();
1200:       assert(N == Rows && "vector length must match matrix row count");
```
- **EN**: This block defines callable entry points like `MB`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `MB`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1201-1220
```cpp
1201:       if (IsRowMajor)
1202:         Op1 = MB.CreateRowMajorToColumnMajorTransform(Op1, Rows, Cols);
1203:       return MB.CreateMatrixMultiply(Op0, Op1, 1, N, Cols, "hlsl.mul");
1204:     }
1205:     if (IsMat0 && IsVec1) {
1206:       auto *MatTy = QTy0->castAs<ConstantMatrixType>();
1207:       unsigned Rows = MatTy->getNumRows();
1208:       unsigned Cols = MatTy->getNumColumns();
1209:       assert(QTy1->castAs<VectorType>()->getNumElements() == Cols &&
1210:              "vector length must match matrix column count");
1211:       if (IsRowMajor)
1212:         Op0 = MB.CreateRowMajorToColumnMajorTransform(Op0, Rows, Cols);
1213:       return MB.CreateMatrixMultiply(Op0, Op1, Rows, Cols, 1, "hlsl.mul");
1214:     }
1215:     assert(IsMat0 && IsMat1);
1216:     auto *MatTy0 = QTy0->castAs<ConstantMatrixType>();
1217:     auto *MatTy1 = QTy1->castAs<ConstantMatrixType>();
1218:     unsigned Rows0 = MatTy0->getNumRows();
1219:     unsigned Rows1 = MatTy1->getNumRows();
1220:     unsigned Cols0 = MatTy0->getNumColumns();
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1221-1240
```cpp
1221:     unsigned Cols1 = MatTy1->getNumColumns();
1222:     assert(Cols0 == Rows1 &&
1223:            "inner matrix dimensions must match for multiplication");
1224:     if (IsRowMajor) {
1225:       Op0 = MB.CreateRowMajorToColumnMajorTransform(Op0, Rows0, Cols0);
1226:       Op1 = MB.CreateRowMajorToColumnMajorTransform(Op1, Rows1, Cols1);
1227:     }
1228:     Value *Result =
1229:         MB.CreateMatrixMultiply(Op0, Op1, Rows0, Cols0, Cols1, "hlsl.mul");
1230:     if (IsRowMajor)
1231:       Result = MB.CreateColumnMajorToRowMajorTransform(Result, Rows0, Cols1);
1232:     return Result;
1233:   }
1234:   case Builtin::BI__builtin_hlsl_transpose: {
1235:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1236:     auto *MatTy = E->getArg(0)->getType()->castAs<ConstantMatrixType>();
1237:     unsigned Rows = MatTy->getNumRows();
1238:     unsigned Cols = MatTy->getNumColumns();
1239:     llvm::MatrixBuilder MB(Builder);
1240:     // The matrix transpose intrinsic operates on column-major matrices.
```
- **EN**: This block defines callable entry points like `MB`; uses control flow (if, for, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `MB`；通过控制流（if, for, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1241-1260
```cpp
1241:     // For row-major, a row-major RxC matrix is equivalent to a column-major
1242:     // CxR matrix, so transposing with swapped dimensions produces the correct
1243:     // row-major CxR result directly.
1244:     bool IsRowMajor = getLangOpts().getDefaultMatrixMemoryLayout() ==
1245:                       LangOptions::MatrixMemoryLayout::MatrixRowMajor;
1246:     if (IsRowMajor)
1247:       return MB.CreateMatrixTranspose(Op0, Cols, Rows);
1248:     return MB.CreateMatrixTranspose(Op0, Rows, Cols);
1249:   }
1250:   case Builtin::BI__builtin_hlsl_elementwise_rcp: {
1251:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1252:     if (!E->getArg(0)->getType()->hasFloatingRepresentation())
1253:       llvm_unreachable("rcp operand must have a float representation");
1254:     llvm::Type *Ty = Op0->getType();
1255:     llvm::Type *EltTy = Ty->getScalarType();
1256:     Constant *One = Ty->isVectorTy()
1257:                         ? ConstantVector::getSplat(
1258:                               ElementCount::getFixed(
1259:                                   cast<FixedVectorType>(Ty)->getNumElements()),
1260:                               ConstantFP::get(EltTy, 1.0))
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1261-1280
```cpp
1261:                         : ConstantFP::get(EltTy, 1.0);
1262:     return Builder.CreateFDiv(One, Op0, "hlsl.rcp");
1263:   }
1264:   case Builtin::BI__builtin_hlsl_elementwise_rsqrt: {
1265:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1266:     if (!E->getArg(0)->getType()->hasFloatingRepresentation())
1267:       llvm_unreachable("rsqrt operand must have a float representation");
1268:     return Builder.CreateIntrinsic(
1269:         /*ReturnType=*/Op0->getType(), CGM.getHLSLRuntime().getRsqrtIntrinsic(),
1270:         ArrayRef<Value *>{Op0}, nullptr, "hlsl.rsqrt");
1271:   }
1272:   case Builtin::BI__builtin_hlsl_elementwise_saturate: {
1273:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1274:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
1275:            "saturate operand must have a float representation");
1276:     return Builder.CreateIntrinsic(
1277:         /*ReturnType=*/Op0->getType(),
1278:         CGM.getHLSLRuntime().getSaturateIntrinsic(), ArrayRef<Value *>{Op0},
1279:         nullptr, "hlsl.saturate");
1280:   }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1281-1300
```cpp
1281:   case Builtin::BI__builtin_hlsl_wave_prefix_count_bits: {
1282:     Value *Op = EmitScalarExpr(E->getArg(0));
1283:     assert(Op->getType()->isIntegerTy(1) &&
1284:            "WavePrefixBitCount operand must be a boolean type");
1285: 
1286:     Intrinsic::ID IID =
1287:         getPrefixCountBitsIntrinsic(getTarget().getTriple().getArch());
1288: 
1289:     return EmitIntrinsicCall(IID, ArrayRef{Op}, "hlsl.wave.prefix.bit.count");
1290:   }
1291:   case Builtin::BI__builtin_hlsl_select: {
1292:     Value *OpCond = EmitScalarExpr(E->getArg(0));
1293:     RValue RValTrue = EmitAnyExpr(E->getArg(1));
1294:     Value *OpTrue =
1295:         RValTrue.isScalar()
1296:             ? RValTrue.getScalarVal()
1297:             : Builder.CreateLoad(RValTrue.getAggregateAddress(), "true_val");
1298:     RValue RValFalse = EmitAnyExpr(E->getArg(2));
1299:     Value *OpFalse =
1300:         RValFalse.isScalar()
```
- **EN**: This block defines callable entry points like `getPrefixCountBitsIntrinsic`; uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getPrefixCountBitsIntrinsic`；通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1301-1320
```cpp
1301:             ? RValFalse.getScalarVal()
1302:             : Builder.CreateLoad(RValFalse.getAggregateAddress(), "false_val");
1303:     if (auto *VTy = E->getType()->getAs<VectorType>()) {
1304:       if (!OpTrue->getType()->isVectorTy())
1305:         OpTrue =
1306:             Builder.CreateVectorSplat(VTy->getNumElements(), OpTrue, "splat");
1307:       if (!OpFalse->getType()->isVectorTy())
1308:         OpFalse =
1309:             Builder.CreateVectorSplat(VTy->getNumElements(), OpFalse, "splat");
1310:     }
1311: 
1312:     Value *SelectVal =
1313:         Builder.CreateSelect(OpCond, OpTrue, OpFalse, "hlsl.select");
1314:     if (!RValTrue.isScalar())
1315:       Builder.CreateStore(SelectVal, ReturnValue.getAddress(),
1316:                           ReturnValue.isVolatile());
1317: 
1318:     return SelectVal;
1319:   }
1320:   case Builtin::BI__builtin_hlsl_step: {
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 1321-1340
```cpp
1321:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1322:     Value *Op1 = EmitScalarExpr(E->getArg(1));
1323:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
1324:            E->getArg(1)->getType()->hasFloatingRepresentation() &&
1325:            "step operands must have a float representation");
1326:     return Builder.CreateIntrinsic(
1327:         /*ReturnType=*/Op0->getType(), CGM.getHLSLRuntime().getStepIntrinsic(),
1328:         ArrayRef<Value *>{Op0, Op1}, nullptr, "hlsl.step");
1329:   }
1330:   case Builtin::BI__builtin_hlsl_wave_active_all_equal: {
1331:     Value *Op = EmitScalarExpr(E->getArg(0));
1332: 
1333:     Intrinsic::ID ID = CGM.getHLSLRuntime().getWaveActiveAllEqualIntrinsic();
1334:     return EmitIntrinsicCall(ID, {Op->getType()}, {Op});
1335:   }
1336:   case Builtin::BI__builtin_hlsl_wave_active_all_true: {
1337:     Value *Op = EmitScalarExpr(E->getArg(0));
1338:     assert(Op->getType()->isIntegerTy(1) &&
1339:            "Intrinsic WaveActiveAllTrue operand must be a bool");
1340: 
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1341-1360
```cpp
1341:     Intrinsic::ID ID = CGM.getHLSLRuntime().getWaveActiveAllTrueIntrinsic();
1342:     return EmitIntrinsicCall(ID, {Op});
1343:   }
1344:   case Builtin::BI__builtin_hlsl_wave_active_any_true: {
1345:     Value *Op = EmitScalarExpr(E->getArg(0));
1346:     assert(Op->getType()->isIntegerTy(1) &&
1347:            "Intrinsic WaveActiveAnyTrue operand must be a bool");
1348: 
1349:     Intrinsic::ID ID = CGM.getHLSLRuntime().getWaveActiveAnyTrueIntrinsic();
1350:     return EmitIntrinsicCall(ID, {Op});
1351:   }
1352:   case Builtin::BI__builtin_hlsl_wave_active_bit_or: {
1353:     Value *Op = EmitScalarExpr(E->getArg(0));
1354:     assert(E->getArg(0)->getType()->hasUnsignedIntegerRepresentation() &&
1355:            "Intrinsic WaveActiveBitOr operand must have an unsigned integer "
1356:            "representation");
1357: 
1358:     Intrinsic::ID ID = CGM.getHLSLRuntime().getWaveActiveBitOrIntrinsic();
1359:     return EmitIntrinsicCall(ID, {Op->getType()}, ArrayRef{Op},
1360:                              "hlsl.wave.active.bit.or");
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1361-1380
```cpp
1361:   }
1362:   case Builtin::BI__builtin_hlsl_wave_active_bit_xor: {
1363:     Value *Op = EmitScalarExpr(E->getArg(0));
1364:     assert(E->getArg(0)->getType()->hasUnsignedIntegerRepresentation() &&
1365:            "Intrinsic WaveActiveBitXor operand must have an unsigned integer "
1366:            "representation");
1367: 
1368:     Intrinsic::ID ID = CGM.getHLSLRuntime().getWaveActiveBitXorIntrinsic();
1369:     return EmitIntrinsicCall(ID, {Op->getType()}, ArrayRef{Op},
1370:                              "hlsl.wave.active.bit.xor");
1371:   }
1372:   case Builtin::BI__builtin_hlsl_wave_active_bit_and: {
1373:     Value *Op = EmitScalarExpr(E->getArg(0));
1374:     assert(E->getArg(0)->getType()->hasUnsignedIntegerRepresentation() &&
1375:            "Intrinsic WaveActiveBitAnd operand must have an unsigned integer "
1376:            "representation");
1377: 
1378:     Intrinsic::ID ID = CGM.getHLSLRuntime().getWaveActiveBitAndIntrinsic();
1379:     return EmitIntrinsicCall(ID, {Op->getType()}, ArrayRef{Op},
1380:                              "hlsl.wave.active.bit.and");
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1381-1400
```cpp
1381:   }
1382:   case Builtin::BI__builtin_hlsl_wave_active_ballot: {
1383:     [[maybe_unused]] Value *Op = EmitScalarExpr(E->getArg(0));
1384:     assert(Op->getType()->isIntegerTy(1) &&
1385:            "Intrinsic WaveActiveBallot operand must be a bool");
1386: 
1387:     return handleHlslWaveActiveBallot(*this, E);
1388:   }
1389:   case Builtin::BI__builtin_hlsl_wave_active_count_bits: {
1390:     Value *OpExpr = EmitScalarExpr(E->getArg(0));
1391:     Intrinsic::ID ID = CGM.getHLSLRuntime().getWaveActiveCountBitsIntrinsic();
1392:     return EmitIntrinsicCall(ID, ArrayRef{OpExpr});
1393:   }
1394:   case Builtin::BI__builtin_hlsl_wave_active_sum: {
1395:     // Due to the use of variadic arguments, explicitly retrieve argument
1396:     Value *OpExpr = EmitScalarExpr(E->getArg(0));
1397:     Intrinsic::ID IID = getWaveActiveSumIntrinsic(
1398:         getTarget().getTriple().getArch(), E->getArg(0)->getType());
1399: 
1400:     return EmitIntrinsicCall(IID, {OpExpr->getType()}, ArrayRef{OpExpr},
```
- **EN**: This block defines callable entry points like `handleHlslWaveActiveBallot`, `getTarget`; uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `handleHlslWaveActiveBallot`, `getTarget`；通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1401-1420
```cpp
1401:                              "hlsl.wave.active.sum");
1402:   }
1403:   case Builtin::BI__builtin_hlsl_wave_active_product: {
1404:     // Due to the use of variadic arguments, explicitly retrieve argument
1405:     Value *OpExpr = EmitScalarExpr(E->getArg(0));
1406:     Intrinsic::ID IID = getWaveActiveProductIntrinsic(
1407:         getTarget().getTriple().getArch(), E->getArg(0)->getType());
1408: 
1409:     return EmitIntrinsicCall(IID, {OpExpr->getType()}, ArrayRef{OpExpr},
1410:                              "hlsl.wave.active.product");
1411:   }
1412:   case Builtin::BI__builtin_hlsl_wave_active_max: {
1413:     // Due to the use of variadic arguments, explicitly retrieve argument
1414:     Value *OpExpr = EmitScalarExpr(E->getArg(0));
1415:     QualType QT = E->getArg(0)->getType();
1416:     Intrinsic::ID IID;
1417:     if (QT->isUnsignedIntegerType())
1418:       IID = CGM.getHLSLRuntime().getWaveActiveUMaxIntrinsic();
1419:     else
1420:       IID = CGM.getHLSLRuntime().getWaveActiveMaxIntrinsic();
```
- **EN**: This block defines callable entry points like `getTarget`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getTarget`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 1421-1440
```cpp
1421: 
1422:     return EmitIntrinsicCall(IID, {OpExpr->getType()}, ArrayRef{OpExpr},
1423:                              "hlsl.wave.active.max");
1424:   }
1425:   case Builtin::BI__builtin_hlsl_wave_active_min: {
1426:     // Due to the use of variadic arguments, explicitly retrieve argument
1427:     Value *OpExpr = EmitScalarExpr(E->getArg(0));
1428:     QualType QT = E->getArg(0)->getType();
1429:     Intrinsic::ID IID;
1430:     if (QT->isUnsignedIntegerType())
1431:       IID = CGM.getHLSLRuntime().getWaveActiveUMinIntrinsic();
1432:     else
1433:       IID = CGM.getHLSLRuntime().getWaveActiveMinIntrinsic();
1434: 
1435:     return EmitIntrinsicCall(IID, {OpExpr->getType()}, ArrayRef{OpExpr},
1436:                              "hlsl.wave.active.min");
1437:   }
1438:   case Builtin::BI__builtin_hlsl_wave_get_lane_index: {
1439:     // We don't define a SPIR-V intrinsic, instead it is a SPIR-V built-in
1440:     // defined in SPIRVBuiltins.td. So instead we manually get the matching name
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 1441-1460
```cpp
1441:     // for the DirectX intrinsic and the demangled builtin name
1442:     switch (CGM.getTarget().getTriple().getArch()) {
1443:     case llvm::Triple::dxil:
1444:       return EmitIntrinsicCall(Intrinsic::dx_wave_getlaneindex);
1445:     case llvm::Triple::spirv:
1446:       return EmitRuntimeCall(CGM.CreateRuntimeFunction(
1447:           llvm::FunctionType::get(IntTy, {}, false),
1448:           "__hlsl_wave_get_lane_index", {}, false, true));
1449:     default:
1450:       llvm_unreachable(
1451:           "Intrinsic WaveGetLaneIndex not supported by target architecture");
1452:     }
1453:   }
1454:   case Builtin::BI__builtin_hlsl_wave_is_first_lane: {
1455:     Intrinsic::ID ID = CGM.getHLSLRuntime().getWaveIsFirstLaneIntrinsic();
1456:     return EmitIntrinsicCall(ID);
1457:   }
1458:   case Builtin::BI__builtin_hlsl_wave_get_lane_count: {
1459:     Intrinsic::ID ID = CGM.getHLSLRuntime().getWaveGetLaneCountIntrinsic();
1460:     return EmitIntrinsicCall(ID);
```
- **EN**: This block defines callable entry points like `EmitIntrinsicCall`; uses control flow (switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitIntrinsicCall`；通过控制流（switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1461-1480
```cpp
1461:   }
1462:   case Builtin::BI__builtin_hlsl_wave_read_lane_at: {
1463:     // Due to the use of variadic arguments we must explicitly retrieve them and
1464:     // create our function type.
1465:     Value *OpExpr = EmitScalarExpr(E->getArg(0));
1466:     Value *OpIndex = EmitScalarExpr(E->getArg(1));
1467:     return EmitIntrinsicCall(CGM.getHLSLRuntime().getWaveReadLaneAtIntrinsic(),
1468:                              {OpExpr->getType()}, ArrayRef{OpExpr, OpIndex},
1469:                              "hlsl.wave.readlane");
1470:   }
1471:   case Builtin::BI__builtin_hlsl_wave_prefix_sum: {
1472:     Value *OpExpr = EmitScalarExpr(E->getArg(0));
1473:     Intrinsic::ID IID = getWavePrefixSumIntrinsic(
1474:         getTarget().getTriple().getArch(), E->getArg(0)->getType());
1475:     return EmitIntrinsicCall(IID, {OpExpr->getType()}, ArrayRef{OpExpr},
1476:                              "hlsl.wave.prefix.sum");
1477:   }
1478:   case Builtin::BI__builtin_hlsl_wave_prefix_product: {
1479:     Value *OpExpr = EmitScalarExpr(E->getArg(0));
1480:     Intrinsic::ID IID = getWavePrefixProductIntrinsic(
```
- **EN**: This block defines callable entry points like `getTarget`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getTarget`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 1481-1500
```cpp
1481:         getTarget().getTriple().getArch(), E->getArg(0)->getType());
1482:     return EmitIntrinsicCall(IID, {OpExpr->getType()}, ArrayRef{OpExpr},
1483:                              "hlsl.wave.prefix.product");
1484:   }
1485:   case Builtin::BI__builtin_hlsl_quad_read_across_x: {
1486:     Value *OpExpr = EmitScalarExpr(E->getArg(0));
1487:     Intrinsic::ID ID = CGM.getHLSLRuntime().getQuadReadAcrossXIntrinsic();
1488:     return EmitIntrinsicCall(ID, {OpExpr->getType()}, ArrayRef{OpExpr},
1489:                              "hlsl.quad.read.across.x");
1490:   }
1491:   case Builtin::BI__builtin_hlsl_quad_read_across_y: {
1492:     Value *OpExpr = EmitScalarExpr(E->getArg(0));
1493:     Intrinsic::ID ID = CGM.getHLSLRuntime().getQuadReadAcrossYIntrinsic();
1494:     return EmitIntrinsicCall(ID, {OpExpr->getType()}, ArrayRef{OpExpr},
1495:                              "hlsl.quad.read.across.y");
1496:   }
1497:   case Builtin::BI__builtin_hlsl_elementwise_sign: {
1498:     auto *Arg0 = E->getArg(0);
1499:     Value *Op0 = EmitScalarExpr(Arg0);
1500:     llvm::Type *Xty = Op0->getType();
```
- **EN**: This block defines callable entry points like `getTarget`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getTarget`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 1501-1520
```cpp
1501:     llvm::Type *retType = llvm::Type::getInt32Ty(this->getLLVMContext());
1502:     if (Xty->isVectorTy()) {
1503:       auto *XVecTy = Arg0->getType()->castAs<VectorType>();
1504:       retType = llvm::VectorType::get(
1505:           retType, ElementCount::getFixed(XVecTy->getNumElements()));
1506:     }
1507:     assert((Arg0->getType()->hasFloatingRepresentation() ||
1508:             Arg0->getType()->hasIntegerRepresentation()) &&
1509:            "sign operand must have a float or int representation");
1510: 
1511:     if (Arg0->getType()->hasUnsignedIntegerRepresentation()) {
1512:       Value *Cmp = Builder.CreateICmpEQ(Op0, ConstantInt::get(Xty, 0));
1513:       return Builder.CreateSelect(Cmp, ConstantInt::get(retType, 0),
1514:                                   ConstantInt::get(retType, 1), "hlsl.sign");
1515:     }
1516: 
1517:     return Builder.CreateIntrinsic(
1518:         retType, CGM.getHLSLRuntime().getSignIntrinsic(),
1519:         ArrayRef<Value *>{Op0}, nullptr, "hlsl.sign");
1520:   }
```
- **EN**: This block defines callable entry points like `getFixed`, `get`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getFixed`, `get`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1521-1540
```cpp
1521:   case Builtin::BI__builtin_hlsl_elementwise_radians: {
1522:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1523:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
1524:            "radians operand must have a float representation");
1525:     return Builder.CreateIntrinsic(
1526:         /*ReturnType=*/Op0->getType(),
1527:         CGM.getHLSLRuntime().getRadiansIntrinsic(), ArrayRef<Value *>{Op0},
1528:         nullptr, "hlsl.radians");
1529:   }
1530:   case Builtin::BI__builtin_hlsl_buffer_update_counter: {
1531:     Value *ResHandle = EmitScalarExpr(E->getArg(0));
1532:     Value *Offset = EmitScalarExpr(E->getArg(1));
1533:     Value *OffsetI8 = Builder.CreateIntCast(Offset, Int8Ty, true);
1534:     return Builder.CreateIntrinsic(
1535:         /*ReturnType=*/Offset->getType(),
1536:         CGM.getHLSLRuntime().getBufferUpdateCounterIntrinsic(),
1537:         ArrayRef<Value *>{ResHandle, OffsetI8}, nullptr);
1538:   }
1539:   case Builtin::BI__builtin_hlsl_elementwise_splitdouble: {
1540: 
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1541-1560
```cpp
1541:     assert((E->getArg(0)->getType()->hasFloatingRepresentation() &&
1542:             E->getArg(1)->getType()->hasUnsignedIntegerRepresentation() &&
1543:             E->getArg(2)->getType()->hasUnsignedIntegerRepresentation()) &&
1544:            "asuint operands types mismatch");
1545:     return handleHlslSplitdouble(E, this);
1546:   }
1547:   case Builtin::BI__builtin_hlsl_elementwise_clip:
1548:     assert(E->getArg(0)->getType()->hasFloatingRepresentation() &&
1549:            "clip operands types mismatch");
1550:     return handleHlslClip(E, this);
1551:   case Builtin::BI__builtin_hlsl_all_memory_barrier: {
1552:     Intrinsic::ID ID = CGM.getHLSLRuntime().getAllMemoryBarrierIntrinsic();
1553:     return EmitIntrinsicCall(ID);
1554:   }
1555:   case Builtin::BI__builtin_hlsl_all_memory_barrier_with_group_sync: {
1556:     Intrinsic::ID ID =
1557:         CGM.getHLSLRuntime().getAllMemoryBarrierWithGroupSyncIntrinsic();
1558:     return EmitIntrinsicCall(ID);
1559:   }
1560:   case Builtin::BI__builtin_hlsl_device_memory_barrier: {
```
- **EN**: This block defines callable entry points like `handleHlslSplitdouble`, `handleHlslClip`, `EmitIntrinsicCall`; uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `handleHlslSplitdouble`, `handleHlslClip`, `EmitIntrinsicCall`；通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1561-1580
```cpp
1561:     Intrinsic::ID ID = CGM.getHLSLRuntime().getDeviceMemoryBarrierIntrinsic();
1562:     return EmitIntrinsicCall(ID);
1563:   }
1564:   case Builtin::BI__builtin_hlsl_device_memory_barrier_with_group_sync: {
1565:     Intrinsic::ID ID =
1566:         CGM.getHLSLRuntime().getDeviceMemoryBarrierWithGroupSyncIntrinsic();
1567:     return EmitIntrinsicCall(ID);
1568:   }
1569:   case Builtin::BI__builtin_hlsl_group_memory_barrier: {
1570:     Intrinsic::ID ID = CGM.getHLSLRuntime().getGroupMemoryBarrierIntrinsic();
1571:     return EmitIntrinsicCall(ID);
1572:   }
1573:   case Builtin::BI__builtin_hlsl_group_memory_barrier_with_group_sync: {
1574:     Intrinsic::ID ID =
1575:         CGM.getHLSLRuntime().getGroupMemoryBarrierWithGroupSyncIntrinsic();
1576:     return EmitIntrinsicCall(ID);
1577:   }
1578:   case Builtin::BI__builtin_hlsl_elementwise_ddx_coarse: {
1579:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1580:     if (!E->getArg(0)->getType()->hasFloatingRepresentation())
```
- **EN**: This block defines callable entry points like `EmitIntrinsicCall`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `EmitIntrinsicCall`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 1581-1600
```cpp
1581:       llvm_unreachable("ddx_coarse operand must have a float representation");
1582:     Intrinsic::ID ID = CGM.getHLSLRuntime().getDdxCoarseIntrinsic();
1583:     return Builder.CreateIntrinsic(/*ReturnType=*/Op0->getType(), ID,
1584:                                    ArrayRef<Value *>{Op0}, nullptr,
1585:                                    "hlsl.ddx.coarse");
1586:   }
1587:   case Builtin::BI__builtin_hlsl_elementwise_ddy_coarse: {
1588:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1589:     if (!E->getArg(0)->getType()->hasFloatingRepresentation())
1590:       llvm_unreachable("ddy_coarse operand must have a float representation");
1591:     Intrinsic::ID ID = CGM.getHLSLRuntime().getDdyCoarseIntrinsic();
1592:     return Builder.CreateIntrinsic(/*ReturnType=*/Op0->getType(), ID,
1593:                                    ArrayRef<Value *>{Op0}, nullptr,
1594:                                    "hlsl.ddy.coarse");
1595:   }
1596:   case Builtin::BI__builtin_hlsl_elementwise_ddx_fine: {
1597:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1598:     if (!E->getArg(0)->getType()->hasFloatingRepresentation())
1599:       llvm_unreachable("ddx_fine operand must have a float representation");
1600:     Intrinsic::ID ID = CGM.getHLSLRuntime().getDdxFineIntrinsic();
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1601-1620
```cpp
1601:     return Builder.CreateIntrinsic(/*ReturnType=*/Op0->getType(), ID,
1602:                                    ArrayRef<Value *>{Op0}, nullptr,
1603:                                    "hlsl.ddx.fine");
1604:   }
1605:   case Builtin::BI__builtin_hlsl_elementwise_ddy_fine: {
1606:     Value *Op0 = EmitScalarExpr(E->getArg(0));
1607:     if (!E->getArg(0)->getType()->hasFloatingRepresentation())
1608:       llvm_unreachable("ddy_fine operand must have a float representation");
1609:     Intrinsic::ID ID = CGM.getHLSLRuntime().getDdyFineIntrinsic();
1610:     return Builder.CreateIntrinsic(/*ReturnType=*/Op0->getType(), ID,
1611:                                    ArrayRef<Value *>{Op0}, nullptr,
1612:                                    "hlsl.ddy.fine");
1613:   }
1614:   case Builtin::BI__builtin_get_spirv_spec_constant_bool:
1615:   case Builtin::BI__builtin_get_spirv_spec_constant_short:
1616:   case Builtin::BI__builtin_get_spirv_spec_constant_ushort:
1617:   case Builtin::BI__builtin_get_spirv_spec_constant_int:
1618:   case Builtin::BI__builtin_get_spirv_spec_constant_uint:
1619:   case Builtin::BI__builtin_get_spirv_spec_constant_longlong:
1620:   case Builtin::BI__builtin_get_spirv_spec_constant_ulonglong:
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1621-1640
```cpp
1621:   case Builtin::BI__builtin_get_spirv_spec_constant_half:
1622:   case Builtin::BI__builtin_get_spirv_spec_constant_float:
1623:   case Builtin::BI__builtin_get_spirv_spec_constant_double: {
1624:     llvm::Function *SpecConstantFn = getSpecConstantFunction(E->getType());
1625:     llvm::Value *SpecId = EmitScalarExpr(E->getArg(0));
1626:     llvm::Value *DefaultVal = EmitScalarExpr(E->getArg(1));
1627:     llvm::Value *Args[] = {SpecId, DefaultVal};
1628:     return Builder.CreateCall(SpecConstantFn, Args);
1629:   }
1630:   }
1631:   return nullptr;
1632: }
1633: 
1634: llvm::Function *clang::CodeGen::CodeGenFunction::getSpecConstantFunction(
1635:     const clang::QualType &SpecConstantType) {
1636: 
1637:   // Find or create the declaration for the function.
1638:   llvm::Module *M = &CGM.getModule();
1639:   std::string MangledName =
1640:       getSpecConstantFunctionName(SpecConstantType, getContext());
```
- **EN**: This block defines callable entry points like `getSpecConstantFunctionName`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getSpecConstantFunctionName`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 1641-1652
```cpp
1641:   llvm::Function *SpecConstantFn = M->getFunction(MangledName);
1642: 
1643:   if (!SpecConstantFn) {
1644:     llvm::Type *IntType = ConvertType(getContext().IntTy);
1645:     llvm::Type *RetTy = ConvertType(SpecConstantType);
1646:     llvm::Type *ArgTypes[] = {IntType, RetTy};
1647:     llvm::FunctionType *FnTy = llvm::FunctionType::get(RetTy, ArgTypes, false);
1648:     SpecConstantFn = llvm::Function::Create(
1649:         FnTy, llvm::GlobalValue::ExternalLinkage, MangledName, M);
1650:   }
1651:   return SpecConstantFn;
1652: }
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

## Key Concepts / 关键概念

- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getArg**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Builder**: Acts as a construction helper that incrementally assembles LLVM IR emission state. / 充当构建辅助器，逐步组装 LLVM IR 生成 状态。
- **getType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **EmitScalarExpr**: Suggests an entry point that materializes IR or helper objects for LLVM IR emission. / 暗示其是为 LLVM IR 生成 生成 IR 或辅助对象的入口。
- **CGF**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **Builtin**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGM**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGBuiltin.h`, `CGHLSLRuntime.h`, `CodeGenFunction.h`
- **LLVM libraries / LLVM 库**: `llvm/IR/MatrixBuilder.h`
