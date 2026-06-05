# SPIRVCombinerHelper.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVCombinerHelper.cpp`
- Repository: `llvm-project`
- Purpose (EN): SPIRVCombinerHelper support code for the LLVM target backend.
- 目的（中文）: 该文件提供 LLVM 目标后端所需的辅助逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- SPIRVCombinerHelper.cpp -------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: #include "SPIRVCombinerHelper.h"
10: #include "SPIRVGlobalRegistry.h"
11: #include "SPIRVUtils.h"
12: #include "llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"
13: #include "llvm/CodeGen/GlobalISel/MIPatternMatch.h"
14: #include "llvm/IR/DerivedTypes.h"
15: #include "llvm/IR/IntrinsicsSPIRV.h"
16: #include "llvm/IR/LLVMContext.h" // Explicitly include for LLVMContext
17: #include "llvm/Target/TargetMachine.h"
18:
19: using namespace llvm;
20: using namespace MIPatternMatch;
21:
22: SPIRVCombinerHelper::SPIRVCombinerHelper(
23:     GISelChangeObserver &Observer, MachineIRBuilder &B, bool IsPreLegalize,
24:     GISelValueTracking *VT, MachineDominatorTree *MDT, const LegalizerInfo *LI,
25:     const SPIRVSubtarget &STI)
26:     : CombinerHelper(Observer, B, IsPreLegalize, VT, MDT, LI), STI(STI) {}
27:
28: /// This match is part of a combine that
29: /// rewrites length(X - Y) to distance(X, Y)
30: ///   (f32 (g_intrinsic length
31: ///           (g_fsub (vXf32 X) (vXf32 Y))))
32: /// ->
33: ///   (f32 (g_intrinsic distance
34: ///           (vXf32 X) (vXf32 Y)))
35: ///
36: bool SPIRVCombinerHelper::matchLengthToDistance(MachineInstr &MI) const {
37:   if (MI.getOpcode() != TargetOpcode::G_INTRINSIC ||
38:       cast<GIntrinsic>(MI).getIntrinsicID() != Intrinsic::spv_length)
39:     return false;
40:
41:   // First operand of MI is `G_INTRINSIC` so start at operand 2.
42:   Register SubReg = MI.getOperand(2).getReg();
43:   MachineInstr *SubInstr = MRI.getVRegDef(SubReg);
44:   if (SubInstr->getOpcode() != TargetOpcode::G_FSUB)
45:     return false;
46:
47:   return true;
48: }
49:
50: void SPIRVCombinerHelper::applySPIRVDistance(MachineInstr &MI) const {
51:   // Extract the operands for X and Y from the match criteria.
52:   Register SubDestReg = MI.getOperand(2).getReg();
53:   MachineInstr *SubInstr = MRI.getVRegDef(SubDestReg);
54:   Register SubOperand1 = SubInstr->getOperand(1).getReg();
55:   Register SubOperand2 = SubInstr->getOperand(2).getReg();
56:   Register ResultReg = MI.getOperand(0).getReg();
57:
58:   Builder.setInstrAndDebugLoc(MI);
59:   Builder.buildIntrinsic(Intrinsic::spv_distance, ResultReg)
60:       .addUse(SubOperand1)
```
- EN: This range implements operational logic in helpers such as CombinerHelper, SPIRVCombinerHelper::matchLengthToDistance, getIntrinsicID, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 CombinerHelper、SPIRVCombinerHelper::matchLengthToDistance、getIntrinsicID、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:       .addUse(SubOperand2);
 62:
 63:   MI.eraseFromParent();
 64: }
 65:
 66: /// This match is part of a combine that
 67: /// rewrites select(fcmp(dot(I, Ng), 0), N, -N) to faceforward(N, I, Ng)
 68: ///   (vXf32 (g_select
 69: ///             (g_fcmp
 70: ///                (g_intrinsic dot(vXf32 I) (vXf32 Ng)
 71: ///                 0)
 72: ///             (vXf32 N)
 73: ///             (vXf32 g_fneg (vXf32 N))))
 74: /// ->
 75: ///   (vXf32 (g_intrinsic faceforward
 76: ///             (vXf32 N) (vXf32 I) (vXf32 Ng)))
 77: ///
 78: /// This only works for Vulkan shader targets.
 79: ///
 80: bool SPIRVCombinerHelper::matchSelectToFaceForward(MachineInstr &MI) const {
 81:   if (!STI.isShader())
 82:     return false;
 83:
 84:   // Match overall select pattern.
 85:   Register CondReg, TrueReg, FalseReg;
 86:   if (!mi_match(MI.getOperand(0).getReg(), MRI,
 87:                 m_GISelect(m_Reg(CondReg), m_Reg(TrueReg), m_Reg(FalseReg))))
 88:     return false;
 89:
 90:   // Match the FCMP condition.
 91:   Register DotReg, CondZeroReg;
 92:   CmpInst::Predicate Pred;
 93:   if (!mi_match(CondReg, MRI,
 94:                 m_GFCmp(m_Pred(Pred), m_Reg(DotReg), m_Reg(CondZeroReg))))
 95:     return false;
 96:   if (Pred == CmpInst::FCMP_OGT || Pred == CmpInst::FCMP_UGT)
 97:     std::swap(DotReg, CondZeroReg);
 98:   else if (!(Pred == CmpInst::FCMP_OLT || Pred == CmpInst::FCMP_ULT))
 99:     return false;
100:
101:   // Check if FCMP is a comparison between a dot product and 0.
102:   MachineInstr *DotInstr = MRI.getVRegDef(DotReg);
103:   if (DotInstr->getOpcode() != TargetOpcode::G_INTRINSIC ||
104:       cast<GIntrinsic>(DotInstr)->getIntrinsicID() != Intrinsic::spv_fdot) {
105:     Register DotOperand1, DotOperand2;
106:     // Check for scalar dot product.
107:     if (!mi_match(DotReg, MRI,
108:                   m_GFMul(m_Reg(DotOperand1), m_Reg(DotOperand2))) ||
109:         !MRI.getType(DotOperand1).isScalar() ||
110:         !MRI.getType(DotOperand2).isScalar())
111:       return false;
112:   }
113:
114:   const ConstantFP *ZeroVal;
115:   if (!mi_match(CondZeroReg, MRI, m_GFCst(ZeroVal)) || !ZeroVal->isZero())
116:     return false;
117:
118:   // Check if select's false operand is the negation of the true operand.
119:   auto AreNegatedConstantsOrSplats = [&](Register TrueReg, Register FalseReg) {
120:     std::optional<FPValueAndVReg> TrueVal, FalseVal;
```
- EN: This range implements operational logic in helpers such as addUse, eraseFromParent, SPIRVCombinerHelper::matchSelectToFaceForward, m_GISelect, translating backend policy into executable code.
- CN: 这一段实现了 addUse、eraseFromParent、SPIRVCombinerHelper::matchSelectToFaceForward、m_GISelect 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:     if (!mi_match(TrueReg, MRI, m_GFCstOrSplat(TrueVal)) ||
122:         !mi_match(FalseReg, MRI, m_GFCstOrSplat(FalseVal)))
123:       return false;
124:     APFloat TrueValNegated = TrueVal->Value;
125:     TrueValNegated.changeSign();
126:     return FalseVal->Value.compare(TrueValNegated) == APFloat::cmpEqual;
127:   };
128:
129:   if (!mi_match(TrueReg, MRI, m_GFNeg(m_SpecificReg(FalseReg))) &&
130:       !mi_match(FalseReg, MRI, m_GFNeg(m_SpecificReg(TrueReg)))) {
131:     std::optional<FPValueAndVReg> MulConstant;
132:     MachineInstr *TrueInstr = MRI.getVRegDef(TrueReg);
133:     MachineInstr *FalseInstr = MRI.getVRegDef(FalseReg);
134:     if (TrueInstr->getOpcode() == TargetOpcode::G_BUILD_VECTOR &&
135:         FalseInstr->getOpcode() == TargetOpcode::G_BUILD_VECTOR &&
136:         TrueInstr->getNumOperands() == FalseInstr->getNumOperands()) {
137:       for (unsigned I = 1; I < TrueInstr->getNumOperands(); ++I)
138:         if (!AreNegatedConstantsOrSplats(TrueInstr->getOperand(I).getReg(),
139:                                          FalseInstr->getOperand(I).getReg()))
140:           return false;
141:     } else if (mi_match(TrueReg, MRI,
142:                         m_GFMul(m_SpecificReg(FalseReg),
143:                                 m_GFCstOrSplat(MulConstant))) ||
144:                mi_match(FalseReg, MRI,
145:                         m_GFMul(m_SpecificReg(TrueReg),
146:                                 m_GFCstOrSplat(MulConstant))) ||
147:                mi_match(TrueReg, MRI,
148:                         m_GFMul(m_GFCstOrSplat(MulConstant),
149:                                 m_SpecificReg(FalseReg))) ||
150:                mi_match(FalseReg, MRI,
151:                         m_GFMul(m_GFCstOrSplat(MulConstant),
152:                                 m_SpecificReg(TrueReg)))) {
153:       if (!MulConstant || !MulConstant->Value.isExactlyValue(-1.0))
154:         return false;
155:     } else if (!AreNegatedConstantsOrSplats(TrueReg, FalseReg))
156:       return false;
157:   }
158:
159:   return true;
160: }
161:
162: void SPIRVCombinerHelper::applySPIRVFaceForward(MachineInstr &MI) const {
163:   // Extract the operands for N, I, and Ng from the match criteria.
164:   Register CondReg = MI.getOperand(1).getReg();
165:   MachineInstr *CondInstr = MRI.getVRegDef(CondReg);
166:   Register DotReg = CondInstr->getOperand(2).getReg();
167:   CmpInst::Predicate Pred = cast<GFCmp>(CondInstr)->getCond();
168:   if (Pred == CmpInst::FCMP_OGT || Pred == CmpInst::FCMP_UGT)
169:     DotReg = CondInstr->getOperand(3).getReg();
170:   MachineInstr *DotInstr = MRI.getVRegDef(DotReg);
171:   Register DotOperand1, DotOperand2;
172:   if (DotInstr->getOpcode() == TargetOpcode::G_FMUL) {
173:     DotOperand1 = DotInstr->getOperand(1).getReg();
174:     DotOperand2 = DotInstr->getOperand(2).getReg();
175:   } else {
176:     DotOperand1 = DotInstr->getOperand(2).getReg();
177:     DotOperand2 = DotInstr->getOperand(3).getReg();
178:   }
179:   Register TrueReg = MI.getOperand(2).getReg();
180:   Register FalseReg = MI.getOperand(3).getReg();
```
- EN: This range implements operational logic in helpers such as mi_match, changeSign, getVRegDef, getNumOperands, translating backend policy into executable code.
- CN: 这一段实现了 mi_match、changeSign、getVRegDef、getNumOperands 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:   MachineInstr *TrueInstr = MRI.getVRegDef(TrueReg);
182:   if (TrueInstr->getOpcode() == TargetOpcode::G_FNEG ||
183:       TrueInstr->getOpcode() == TargetOpcode::G_FMUL)
184:     std::swap(TrueReg, FalseReg);
185:   MachineInstr *FalseInstr = MRI.getVRegDef(FalseReg);
186:
187:   Register ResultReg = MI.getOperand(0).getReg();
188:   Builder.setInstrAndDebugLoc(MI);
189:   Builder.buildIntrinsic(Intrinsic::spv_faceforward, ResultReg)
190:       .addUse(TrueReg)      // N
191:       .addUse(DotOperand1)  // I
192:       .addUse(DotOperand2); // Ng
193:
194:   SPIRVGlobalRegistry *GR =
195:       MI.getMF()->getSubtarget<SPIRVSubtarget>().getSPIRVGlobalRegistry();
196:   auto RemoveAllUses = [&](Register Reg) {
197:     SmallVector<MachineInstr *, 4> UsesToErase;
198:     for (auto &UseMI : MRI.use_instructions(Reg))
199:       UsesToErase.push_back(&UseMI);
200:
201:     // calling eraseFromParent to early invalidates the iterator.
202:     for (auto *MIToErase : UsesToErase)
203:       MIToErase->eraseFromParent();
204:   };
205:
206:   RemoveAllUses(CondReg); // remove all uses of FCMP Result
207:   GR->invalidateMachineInstr(CondInstr);
208:   CondInstr->eraseFromParent(); // remove FCMP instruction
209:   RemoveAllUses(DotReg);        // remove all uses of spv_fdot/G_FMUL Result
210:   GR->invalidateMachineInstr(DotInstr);
211:   DotInstr->eraseFromParent(); // remove spv_fdot/G_FMUL instruction
212:   RemoveAllUses(FalseReg);
213:   GR->invalidateMachineInstr(FalseInstr);
214:   FalseInstr->eraseFromParent();
215: }
216:
217: bool SPIRVCombinerHelper::matchMatrixTranspose(MachineInstr &MI) const {
218:   return MI.getOpcode() == TargetOpcode::G_INTRINSIC &&
219:          cast<GIntrinsic>(MI).getIntrinsicID() == Intrinsic::matrix_transpose;
220: }
221:
222: void SPIRVCombinerHelper::applyMatrixTranspose(MachineInstr &MI) const {
223:   Register ResReg = MI.getOperand(0).getReg();
224:   Register InReg = MI.getOperand(2).getReg();
225:   uint32_t Rows = MI.getOperand(3).getImm();
226:   uint32_t Cols = MI.getOperand(4).getImm();
227:
228:   Builder.setInstrAndDebugLoc(MI);
229:
230:   if (Rows == 1 && Cols == 1) {
231:     Builder.buildCopy(ResReg, InReg);
232:     MI.eraseFromParent();
233:     return;
234:   }
235:
236:   SmallVector<int, 16> Mask;
237:   for (uint32_t K = 0; K < Rows * Cols; ++K) {
238:     uint32_t R = K / Cols;
239:     uint32_t C = K % Cols;
240:     Mask.push_back(C * Rows + R);
```
- EN: This range implements operational logic in helpers such as getVRegDef, getOpcode, std::swap, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getVRegDef、getOpcode、std::swap、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:   }
242:
243:   Builder.buildShuffleVector(ResReg, InReg, InReg, Mask);
244:   MI.eraseFromParent();
245: }
246:
247: bool SPIRVCombinerHelper::matchMatrixMultiply(MachineInstr &MI) const {
248:   return MI.getOpcode() == TargetOpcode::G_INTRINSIC &&
249:          cast<GIntrinsic>(MI).getIntrinsicID() == Intrinsic::matrix_multiply;
250: }
251:
252: SmallVector<Register, 4>
253: SPIRVCombinerHelper::extractColumns(Register MatrixReg, uint32_t NumberOfCols,
254:                                     SPIRVTypeInst SpvColType,
255:                                     SPIRVGlobalRegistry *GR) const {
256:   // If the matrix is a single colunm, return that single column.
257:   if (NumberOfCols == 1)
258:     return {MatrixReg};
259:
260:   SmallVector<Register, 4> Cols;
261:   LLT ColTy = GR->getRegType(SpvColType);
262:   for (uint32_t J = 0; J < NumberOfCols; ++J)
263:     Cols.push_back(MRI.createGenericVirtualRegister(ColTy));
264:   Builder.buildUnmerge(Cols, MatrixReg);
265:   for (Register R : Cols) {
266:     setRegClassType(R, SpvColType, GR, &MRI, Builder.getMF());
267:   }
268:   return Cols;
269: }
270:
271: SmallVector<Register, 4>
272: SPIRVCombinerHelper::extractRows(Register MatrixReg, uint32_t NumRows,
273:                                  uint32_t NumCols, SPIRVTypeInst SpvRowType,
274:                                  SPIRVGlobalRegistry *GR) const {
275:   SmallVector<Register, 4> Rows;
276:   LLT VecTy = GR->getRegType(SpvRowType);
277:
278:   // If there is only one column, then each row is a scalar that needs
279:   // to be extracted.
280:   if (NumCols == 1) {
281:     assert(SpvRowType->getOpcode() != SPIRV::OpTypeVector);
282:     for (uint32_t I = 0; I < NumRows; ++I)
283:       Rows.push_back(MRI.createGenericVirtualRegister(VecTy));
284:     Builder.buildUnmerge(Rows, MatrixReg);
285:     for (Register R : Rows) {
286:       setRegClassType(R, SpvRowType, GR, &MRI, Builder.getMF());
287:     }
288:     return Rows;
289:   }
290:
291:   // If the matrix is a single row return that row.
292:   if (NumRows == 1) {
293:     return {MatrixReg};
294:   }
295:
296:   for (uint32_t I = 0; I < NumRows; ++I) {
297:     SmallVector<int, 4> Mask;
298:     for (uint32_t k = 0; k < NumCols; ++k)
299:       Mask.push_back(k * NumRows + I);
300:     Rows.push_back(Builder.buildShuffleVector(VecTy, MatrixReg, MatrixReg, Mask)
```
- EN: This range implements operational logic in helpers such as buildShuffleVector, eraseFromParent, SPIRVCombinerHelper::matchMatrixMultiply, getRegType, translating backend policy into executable code.
- CN: 这一段实现了 buildShuffleVector、eraseFromParent、SPIRVCombinerHelper::matchMatrixMultiply、getRegType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:                        .getReg(0));
302:   }
303:   for (Register R : Rows) {
304:     setRegClassType(R, SpvRowType, GR, &MRI, Builder.getMF());
305:   }
306:   return Rows;
307: }
308:
309: Register SPIRVCombinerHelper::computeDotProduct(Register RowA, Register ColB,
310:                                                 SPIRVTypeInst SpvVecType,
311:                                                 SPIRVGlobalRegistry *GR) const {
312:   bool IsVectorOp = SpvVecType->getOpcode() == SPIRV::OpTypeVector;
313:   SPIRVTypeInst SpvScalarType = GR->getScalarOrVectorComponentType(SpvVecType);
314:   bool IsFloatOp = SpvScalarType->getOpcode() == SPIRV::OpTypeFloat;
315:   LLT VecTy = GR->getRegType(SpvVecType);
316:
317:   Register DotRes;
318:   if (IsVectorOp) {
319:     LLT ScalarTy = VecTy.getElementType();
320:     Intrinsic::SPVIntrinsics DotIntrinsic =
321:         (IsFloatOp ? Intrinsic::spv_fdot : Intrinsic::spv_udot);
322:     DotRes = Builder.buildIntrinsic(DotIntrinsic, {ScalarTy})
323:                  .addUse(RowA)
324:                  .addUse(ColB)
325:                  .getReg(0);
326:   } else {
327:     if (IsFloatOp)
328:       DotRes = Builder.buildFMul(VecTy, RowA, ColB).getReg(0);
329:     else
330:       DotRes = Builder.buildMul(VecTy, RowA, ColB).getReg(0);
331:   }
332:   setRegClassType(DotRes, SpvScalarType, GR, &MRI, Builder.getMF());
333:   return DotRes;
334: }
335:
336: SmallVector<Register, 16>
337: SPIRVCombinerHelper::computeDotProducts(const SmallVector<Register, 4> &RowsA,
338:                                         const SmallVector<Register, 4> &ColsB,
339:                                         SPIRVTypeInst SpvVecType,
340:                                         SPIRVGlobalRegistry *GR) const {
341:   SmallVector<Register, 16> ResultScalars;
342:   for (uint32_t J = 0; J < ColsB.size(); ++J) {
343:     for (uint32_t I = 0; I < RowsA.size(); ++I) {
344:       ResultScalars.push_back(
345:           computeDotProduct(RowsA[I], ColsB[J], SpvVecType, GR));
346:     }
347:   }
348:   return ResultScalars;
349: }
350:
351: SPIRVTypeInst
352: SPIRVCombinerHelper::getDotProductVectorType(Register ResReg, uint32_t K,
353:                                              SPIRVGlobalRegistry *GR) const {
354:   // Loop over all non debug uses of ResReg
355:   Type *ScalarResType = nullptr;
356:   for (auto &UseMI : MRI.use_instructions(ResReg)) {
357:     if (UseMI.getOpcode() != TargetOpcode::G_INTRINSIC_W_SIDE_EFFECTS)
358:       continue;
359:
360:     if (!isSpvIntrinsic(UseMI, Intrinsic::spv_assign_type))
```
- EN: This range implements operational logic in helpers such as getReg, setRegClassType, getScalarOrVectorComponentType, getRegType, translating backend policy into executable code.
- CN: 这一段实现了 getReg、setRegClassType、getScalarOrVectorComponentType、getRegType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-402
```cpp
361:       continue;
362:
363:     Type *Ty = getMDOperandAsType(UseMI.getOperand(2).getMetadata(), 0);
364:     if (Ty->isVectorTy())
365:       ScalarResType = cast<VectorType>(Ty)->getElementType();
366:     else
367:       ScalarResType = Ty;
368:     assert(ScalarResType->isIntegerTy() || ScalarResType->isFloatingPointTy());
369:     break;
370:   }
371:   if (!ScalarResType)
372:     llvm_unreachable("Could not determine scalar result type");
373:   Type *VecType =
374:       (K > 1 ? FixedVectorType::get(ScalarResType, K) : ScalarResType);
375:   return GR->getOrCreateSPIRVType(VecType, Builder,
376:                                   SPIRV::AccessQualifier::None, false);
377: }
378:
379: void SPIRVCombinerHelper::applyMatrixMultiply(MachineInstr &MI) const {
380:   Register ResReg = MI.getOperand(0).getReg();
381:   Register AReg = MI.getOperand(2).getReg();
382:   Register BReg = MI.getOperand(3).getReg();
383:   uint32_t NumRowsA = MI.getOperand(4).getImm();
384:   uint32_t NumColsA = MI.getOperand(5).getImm();
385:   uint32_t NumColsB = MI.getOperand(6).getImm();
386:
387:   Builder.setInstrAndDebugLoc(MI);
388:
389:   SPIRVGlobalRegistry *GR =
390:       MI.getMF()->getSubtarget<SPIRVSubtarget>().getSPIRVGlobalRegistry();
391:
392:   SPIRVTypeInst SpvVecType = getDotProductVectorType(ResReg, NumColsA, GR);
393:   SmallVector<Register, 4> ColsB =
394:       extractColumns(BReg, NumColsB, SpvVecType, GR);
395:   SmallVector<Register, 4> RowsA =
396:       extractRows(AReg, NumRowsA, NumColsA, SpvVecType, GR);
397:   SmallVector<Register, 16> ResultScalars =
398:       computeDotProducts(RowsA, ColsB, SpvVecType, GR);
399:
400:   Builder.buildBuildVector(ResReg, ResultScalars);
401:   MI.eraseFromParent();
402: }
```
- EN: This range implements operational logic in helpers such as getMDOperandAsType, getElementType, assert, llvm_unreachable, translating backend policy into executable code.
- CN: 这一段实现了 getMDOperandAsType、getElementType、assert、llvm_unreachable 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include CombinerHelper, SPIRVCombinerHelper::matchLengthToDistance, getIntrinsicID, getOperand, getVRegDef, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 CombinerHelper, SPIRVCombinerHelper::matchLengthToDistance, getIntrinsicID, getOperand, getVRegDef，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVCombinerHelper.h`
  - `SPIRVGlobalRegistry.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/GlobalISel/GenericMachineInstrs.h`
  - `llvm/CodeGen/GlobalISel/MIPatternMatch.h`
  - `llvm/IR/DerivedTypes.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/IR/LLVMContext.h`
  - `llvm/Target/TargetMachine.h`
