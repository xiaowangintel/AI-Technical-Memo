# NVPTXTargetTransformInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXTargetTransformInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): NVPTXTargetTransformInfo support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- NVPTXTargetTransformInfo.cpp - NVPTX specific TTI -----------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: #include "NVPTXTargetTransformInfo.h"
10: #include "NVVMProperties.h"
11: #include "llvm/ADT/STLExtras.h"
12: #include "llvm/Analysis/LoopInfo.h"
13: #include "llvm/Analysis/TargetTransformInfo.h"
14: #include "llvm/Analysis/ValueTracking.h"
15: #include "llvm/CodeGen/BasicTTIImpl.h"
16: #include "llvm/CodeGen/TargetLowering.h"
17: #include "llvm/IR/Constants.h"
18: #include "llvm/IR/IntrinsicInst.h"
19: #include "llvm/IR/Intrinsics.h"
20: #include "llvm/IR/IntrinsicsNVPTX.h"
21: #include "llvm/IR/Value.h"
22: #include "llvm/Support/Casting.h"
23: #include "llvm/Support/ErrorHandling.h"
24: #include "llvm/Support/NVPTXAddrSpace.h"
25: #include "llvm/Transforms/InstCombine/InstCombiner.h"
26: #include <optional>
27: using namespace llvm;
28:
29: #define DEBUG_TYPE "NVPTXtti"
30:
31: // Whether the given intrinsic reads threadIdx.x/y/z.
32: static bool readsThreadIndex(const IntrinsicInst *II) {
33:   switch (II->getIntrinsicID()) {
34:     default: return false;
35:     case Intrinsic::nvvm_read_ptx_sreg_tid_x:
36:     case Intrinsic::nvvm_read_ptx_sreg_tid_y:
37:     case Intrinsic::nvvm_read_ptx_sreg_tid_z:
38:       return true;
39:   }
40: }
41:
42: static bool readsLaneId(const IntrinsicInst *II) {
43:   return II->getIntrinsicID() == Intrinsic::nvvm_read_ptx_sreg_laneid;
44: }
45:
46: // Whether the given intrinsic is an atomic instruction in PTX.
47: static bool isNVVMAtomic(const IntrinsicInst *II) {
48:   switch (II->getIntrinsicID()) {
49:   default:
50:     return false;
51:   case Intrinsic::nvvm_atomic_add_gen_f_cta:
52:   case Intrinsic::nvvm_atomic_add_gen_f_sys:
53:   case Intrinsic::nvvm_atomic_add_gen_i_cta:
54:   case Intrinsic::nvvm_atomic_add_gen_i_sys:
55:   case Intrinsic::nvvm_atomic_and_gen_i_cta:
56:   case Intrinsic::nvvm_atomic_and_gen_i_sys:
57:   case Intrinsic::nvvm_atomic_cas_gen_i_cta:
58:   case Intrinsic::nvvm_atomic_cas_gen_i_sys:
59:   case Intrinsic::nvvm_atomic_dec_gen_i_cta:
60:   case Intrinsic::nvvm_atomic_dec_gen_i_sys:
```
- EN: This range implements operational logic in helpers such as readsThreadIndex, readsLaneId, isNVVMAtomic, translating backend policy into executable code.
- CN: 这一段实现了 readsThreadIndex、readsLaneId、isNVVMAtomic 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:   case Intrinsic::nvvm_atomic_inc_gen_i_cta:
 62:   case Intrinsic::nvvm_atomic_inc_gen_i_sys:
 63:   case Intrinsic::nvvm_atomic_max_gen_i_cta:
 64:   case Intrinsic::nvvm_atomic_max_gen_i_sys:
 65:   case Intrinsic::nvvm_atomic_min_gen_i_cta:
 66:   case Intrinsic::nvvm_atomic_min_gen_i_sys:
 67:   case Intrinsic::nvvm_atomic_or_gen_i_cta:
 68:   case Intrinsic::nvvm_atomic_or_gen_i_sys:
 69:   case Intrinsic::nvvm_atomic_exch_gen_i_cta:
 70:   case Intrinsic::nvvm_atomic_exch_gen_i_sys:
 71:   case Intrinsic::nvvm_atomic_xor_gen_i_cta:
 72:   case Intrinsic::nvvm_atomic_xor_gen_i_sys:
 73:     return true;
 74:   }
 75: }
 76:
 77: bool NVPTXTTIImpl::isSourceOfDivergence(const Value *V) const {
 78:   // Without inter-procedural analysis, we conservatively assume that arguments
 79:   // to __device__ functions are divergent.
 80:   if (const Argument *Arg = dyn_cast<Argument>(V))
 81:     return !isKernelFunction(*Arg->getParent());
 82:
 83:   if (const Instruction *I = dyn_cast<Instruction>(V)) {
 84:     // Without pointer analysis, we conservatively assume values loaded from
 85:     // generic or local address space are divergent.
 86:     if (const LoadInst *LI = dyn_cast<LoadInst>(I)) {
 87:       unsigned AS = LI->getPointerAddressSpace();
 88:       return AS == ADDRESS_SPACE_GENERIC || AS == ADDRESS_SPACE_LOCAL;
 89:     }
 90:     // Atomic instructions may cause divergence. Atomic instructions are
 91:     // executed sequentially across all threads in a warp. Therefore, an earlier
 92:     // executed thread may see different memory inputs than a later executed
 93:     // thread. For example, suppose *a = 0 initially.
 94:     //
 95:     //   atom.global.add.s32 d, [a], 1
 96:     //
 97:     // returns 0 for the first thread that enters the critical region, and 1 for
 98:     // the second thread.
 99:     if (I->isAtomic())
100:       return true;
101:     if (const IntrinsicInst *II = dyn_cast<IntrinsicInst>(I)) {
102:       // Instructions that read threadIdx are obviously divergent.
103:       if (readsThreadIndex(II) || readsLaneId(II))
104:         return true;
105:       // Handle the NVPTX atomic intrinsics that cannot be represented as an
106:       // atomic IR instruction.
107:       if (isNVVMAtomic(II))
108:         return true;
109:     }
110:     // Conservatively consider the return value of function calls as divergent.
111:     // We could analyze callees with bodies more precisely using
112:     // inter-procedural analysis.
113:     if (isa<CallInst>(I))
114:       return true;
115:   }
116:
117:   return false;
118: }
119:
120: // Convert NVVM intrinsics to target-generic LLVM code where possible.
```
- EN: This range implements operational logic in helpers such as NVPTXTTIImpl::isSourceOfDivergence, isKernelFunction, getPointerAddressSpace, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXTTIImpl::isSourceOfDivergence、isKernelFunction、getPointerAddressSpace 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121: static Instruction *convertNvvmIntrinsicToLlvm(InstCombiner &IC,
122:                                                IntrinsicInst *II) {
123:   // Each NVVM intrinsic we can simplify can be replaced with one of:
124:   //
125:   //  * an LLVM intrinsic,
126:   //  * an LLVM cast operation,
127:   //  * an LLVM binary operation, or
128:   //  * ad-hoc LLVM IR for the particular operation.
129:
130:   // Some transformations are only valid when the module's
131:   // flush-denormals-to-zero (ftz) setting is true/false, whereas other
132:   // transformations are valid regardless of the module's ftz setting.
133:   enum FtzRequirementTy {
134:     FTZ_Any,       // Any ftz setting is ok.
135:     FTZ_MustBeOn,  // Transformation is valid only if ftz is on.
136:     FTZ_MustBeOff, // Transformation is valid only if ftz is off.
137:   };
138:   // Classes of NVVM intrinsics that can't be replaced one-to-one with a
139:   // target-generic intrinsic, cast op, or binary op but that we can nonetheless
140:   // simplify.
141:   enum SpecialCase {
142:     SPC_Reciprocal,
143:     SCP_FunnelShiftClamp,
144:   };
145:
146:   // SimplifyAction is a poor-man's variant (plus an additional flag) that
147:   // represents how to replace an NVVM intrinsic with target-generic LLVM IR.
148:   struct SimplifyAction {
149:     // Invariant: At most one of these Optionals has a value.
150:     std::optional<Intrinsic::ID> IID;
151:     std::optional<Instruction::CastOps> CastOp;
152:     std::optional<Instruction::BinaryOps> BinaryOp;
153:     std::optional<SpecialCase> Special;
154:
155:     FtzRequirementTy FtzRequirement = FTZ_Any;
156:     // Denormal handling is guarded by different attributes depending on the
157:     // type (denormal-fp-math vs denormal-fp-math-f32), take note of halfs.
158:     bool IsHalfTy = false;
159:
160:     SimplifyAction() = default;
161:
162:     SimplifyAction(Intrinsic::ID IID, FtzRequirementTy FtzReq,
163:                    bool IsHalfTy = false)
164:         : IID(IID), FtzRequirement(FtzReq), IsHalfTy(IsHalfTy) {}
165:
166:     // Cast operations don't have anything to do with FTZ, so we skip that
167:     // argument.
168:     SimplifyAction(Instruction::CastOps CastOp) : CastOp(CastOp) {}
169:
170:     SimplifyAction(Instruction::BinaryOps BinaryOp, FtzRequirementTy FtzReq)
171:         : BinaryOp(BinaryOp), FtzRequirement(FtzReq) {}
172:
173:     SimplifyAction(SpecialCase Special, FtzRequirementTy FtzReq)
174:         : Special(Special), FtzRequirement(FtzReq) {}
175:   };
176:
177:   // Try to generate a SimplifyAction describing how to replace our
178:   // IntrinsicInstr with target-generic LLVM IR.
179:   const SimplifyAction Action = [II]() -> SimplifyAction {
180:     switch (II->getIntrinsicID()) {
```
- EN: This range defines or declares important types such as FtzRequirementTy, SpecialCase, SimplifyAction, IID, shaping the data model used by NVPTXTargetTransformInfo.cpp.
- CN: 这一段定义或声明了 FtzRequirementTy、SpecialCase、SimplifyAction、IID 等关键类型，构成 NVPTXTargetTransformInfo.cpp 使用的数据模型。

### Lines 181-240
```cpp
181:     // NVVM intrinsics that map directly to LLVM intrinsics.
182:     case Intrinsic::nvvm_ceil_d:
183:       return {Intrinsic::ceil, FTZ_Any};
184:     case Intrinsic::nvvm_ceil_f:
185:       return {Intrinsic::ceil, FTZ_MustBeOff};
186:     case Intrinsic::nvvm_ceil_ftz_f:
187:       return {Intrinsic::ceil, FTZ_MustBeOn};
188:     case Intrinsic::nvvm_floor_d:
189:       return {Intrinsic::floor, FTZ_Any};
190:     case Intrinsic::nvvm_floor_f:
191:       return {Intrinsic::floor, FTZ_MustBeOff};
192:     case Intrinsic::nvvm_floor_ftz_f:
193:       return {Intrinsic::floor, FTZ_MustBeOn};
194:     case Intrinsic::nvvm_fma_rn_d:
195:       return {Intrinsic::fma, FTZ_Any};
196:     case Intrinsic::nvvm_fma_rn_f:
197:       return {Intrinsic::fma, FTZ_MustBeOff};
198:     case Intrinsic::nvvm_fma_rn_ftz_f:
199:       return {Intrinsic::fma, FTZ_MustBeOn};
200:     case Intrinsic::nvvm_fma_rn_f16:
201:       return {Intrinsic::fma, FTZ_MustBeOff, true};
202:     case Intrinsic::nvvm_fma_rn_ftz_f16:
203:       return {Intrinsic::fma, FTZ_MustBeOn, true};
204:     case Intrinsic::nvvm_fma_rn_f16x2:
205:       return {Intrinsic::fma, FTZ_MustBeOff, true};
206:     case Intrinsic::nvvm_fma_rn_ftz_f16x2:
207:       return {Intrinsic::fma, FTZ_MustBeOn, true};
208:     case Intrinsic::nvvm_fma_rn_bf16:
209:       return {Intrinsic::fma, FTZ_MustBeOff, true};
210:     case Intrinsic::nvvm_fma_rn_bf16x2:
211:       return {Intrinsic::fma, FTZ_MustBeOff, true};
212:     case Intrinsic::nvvm_fmax_d:
213:       return {Intrinsic::maximumnum, FTZ_Any};
214:     case Intrinsic::nvvm_fmax_f:
215:       return {Intrinsic::maximumnum, FTZ_MustBeOff};
216:     case Intrinsic::nvvm_fmax_ftz_f:
217:       return {Intrinsic::maximumnum, FTZ_MustBeOn};
218:     case Intrinsic::nvvm_fmax_nan_f:
219:       return {Intrinsic::maximum, FTZ_MustBeOff};
220:     case Intrinsic::nvvm_fmax_ftz_nan_f:
221:       return {Intrinsic::maximum, FTZ_MustBeOn};
222:     case Intrinsic::nvvm_fmax_f16:
223:       return {Intrinsic::maximumnum, FTZ_MustBeOff, true};
224:     case Intrinsic::nvvm_fmax_ftz_f16:
225:       return {Intrinsic::maximumnum, FTZ_MustBeOn, true};
226:     case Intrinsic::nvvm_fmax_f16x2:
227:       return {Intrinsic::maximumnum, FTZ_MustBeOff, true};
228:     case Intrinsic::nvvm_fmax_ftz_f16x2:
229:       return {Intrinsic::maximumnum, FTZ_MustBeOn, true};
230:     case Intrinsic::nvvm_fmax_nan_f16:
231:       return {Intrinsic::maximum, FTZ_MustBeOff, true};
232:     case Intrinsic::nvvm_fmax_ftz_nan_f16:
233:       return {Intrinsic::maximum, FTZ_MustBeOn, true};
234:     case Intrinsic::nvvm_fmax_nan_f16x2:
235:       return {Intrinsic::maximum, FTZ_MustBeOff, true};
236:     case Intrinsic::nvvm_fmax_ftz_nan_f16x2:
237:       return {Intrinsic::maximum, FTZ_MustBeOn, true};
238:     case Intrinsic::nvvm_fmin_d:
239:       return {Intrinsic::minimumnum, FTZ_Any};
240:     case Intrinsic::nvvm_fmin_f:
```
- EN: This range implements operational logic in helpers such as backend logic, translating backend policy into executable code.
- CN: 这一段实现了 后端逻辑 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:       return {Intrinsic::minimumnum, FTZ_MustBeOff};
242:     case Intrinsic::nvvm_fmin_ftz_f:
243:       return {Intrinsic::minimumnum, FTZ_MustBeOn};
244:     case Intrinsic::nvvm_fmin_nan_f:
245:       return {Intrinsic::minimum, FTZ_MustBeOff};
246:     case Intrinsic::nvvm_fmin_ftz_nan_f:
247:       return {Intrinsic::minimum, FTZ_MustBeOn};
248:     case Intrinsic::nvvm_fmin_f16:
249:       return {Intrinsic::minimumnum, FTZ_MustBeOff, true};
250:     case Intrinsic::nvvm_fmin_ftz_f16:
251:       return {Intrinsic::minimumnum, FTZ_MustBeOn, true};
252:     case Intrinsic::nvvm_fmin_f16x2:
253:       return {Intrinsic::minimumnum, FTZ_MustBeOff, true};
254:     case Intrinsic::nvvm_fmin_ftz_f16x2:
255:       return {Intrinsic::minimumnum, FTZ_MustBeOn, true};
256:     case Intrinsic::nvvm_fmin_nan_f16:
257:       return {Intrinsic::minimum, FTZ_MustBeOff, true};
258:     case Intrinsic::nvvm_fmin_ftz_nan_f16:
259:       return {Intrinsic::minimum, FTZ_MustBeOn, true};
260:     case Intrinsic::nvvm_fmin_nan_f16x2:
261:       return {Intrinsic::minimum, FTZ_MustBeOff, true};
262:     case Intrinsic::nvvm_fmin_ftz_nan_f16x2:
263:       return {Intrinsic::minimum, FTZ_MustBeOn, true};
264:     case Intrinsic::nvvm_sqrt_rn_d:
265:       return {Intrinsic::sqrt, FTZ_Any};
266:     case Intrinsic::nvvm_sqrt_f:
267:       // nvvm_sqrt_f is a special case.  For  most intrinsics, foo_ftz_f is the
268:       // ftz version, and foo_f is the non-ftz version.  But nvvm_sqrt_f adopts
269:       // the ftz-ness of the surrounding code.  sqrt_rn_f and sqrt_rn_ftz_f are
270:       // the versions with explicit ftz-ness.
271:       return {Intrinsic::sqrt, FTZ_Any};
272:     case Intrinsic::nvvm_trunc_d:
273:       return {Intrinsic::trunc, FTZ_Any};
274:     case Intrinsic::nvvm_trunc_f:
275:       return {Intrinsic::trunc, FTZ_MustBeOff};
276:     case Intrinsic::nvvm_trunc_ftz_f:
277:       return {Intrinsic::trunc, FTZ_MustBeOn};
278:
279:     // NVVM intrinsics that map to LLVM cast operations.
280:     // Note - we cannot map intrinsics like nvvm_d2ll_rz to LLVM's
281:     // FPToSI, as NaN to int conversion with FPToSI is considered UB and is
282:     // eliminated. NVVM conversion intrinsics are translated to PTX cvt
283:     // instructions which define the outcome for NaN rather than leaving as UB.
284:     // Therefore, translate NVVM intrinsics to sitofp/uitofp, but not to
285:     // fptosi/fptoui.
286:     case Intrinsic::nvvm_i2d_rn:
287:     case Intrinsic::nvvm_i2f_rn:
288:     case Intrinsic::nvvm_ll2d_rn:
289:     case Intrinsic::nvvm_ll2f_rn:
290:       return {Instruction::SIToFP};
291:     case Intrinsic::nvvm_ui2d_rn:
292:     case Intrinsic::nvvm_ui2f_rn:
293:     case Intrinsic::nvvm_ull2d_rn:
294:     case Intrinsic::nvvm_ull2f_rn:
295:       return {Instruction::UIToFP};
296:
297:     // NVVM intrinsics that map to LLVM binary ops.
298:     case Intrinsic::nvvm_div_rn_d:
299:       return {Instruction::FDiv, FTZ_Any};
300:
```
- EN: This range implements operational logic in helpers such as backend logic, translating backend policy into executable code.
- CN: 这一段实现了 后端逻辑 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:     // The remainder of cases are NVVM intrinsics that map to LLVM idioms, but
302:     // need special handling.
303:     //
304:     // We seem to be missing intrinsics for rcp.approx.{ftz.}f32, which is just
305:     // as well.
306:     case Intrinsic::nvvm_rcp_rn_d:
307:       return {SPC_Reciprocal, FTZ_Any};
308:
309:     case Intrinsic::nvvm_fshl_clamp:
310:     case Intrinsic::nvvm_fshr_clamp:
311:       return {SCP_FunnelShiftClamp, FTZ_Any};
312:
313:       // We do not currently simplify intrinsics that give an approximate
314:       // answer. These include:
315:       //
316:       //   - nvvm_cos_approx_{f,ftz_f}
317:       //   - nvvm_ex2_approx(_ftz)
318:       //   - nvvm_lg2_approx_{d,f,ftz_f}
319:       //   - nvvm_sin_approx_{f,ftz_f}
320:       //   - nvvm_sqrt_approx_{f,ftz_f}
321:       //   - nvvm_rsqrt_approx_{d,f,ftz_f}
322:       //   - nvvm_div_approx_{ftz_d,ftz_f,f}
323:       //   - nvvm_rcp_approx_ftz_d
324:       //
325:       // Ideally we'd encode them as e.g. "fast call @llvm.cos", where "fast"
326:       // means that fastmath is enabled in the intrinsic.  Unfortunately only
327:       // binary operators (currently) have a fastmath bit in SelectionDAG, so
328:       // this information gets lost and we can't select on it.
329:       //
330:       // TODO: div and rcp are lowered to a binary op, so these we could in
331:       // theory lower them to "fast fdiv".
332:
333:     default:
334:       return {};
335:     }
336:   }();
337:
338:   // If Action.FtzRequirementTy is not satisfied by the module's ftz state, we
339:   // can bail out now.  (Notice that in the case that IID is not an NVVM
340:   // intrinsic, we don't have to look up any module metadata, as
341:   // FtzRequirementTy will be FTZ_Any.)
342:   if (Action.FtzRequirement != FTZ_Any) {
343:     // FIXME: Broken for f64
344:     DenormalMode Mode = II->getFunction()->getDenormalMode(
345:         Action.IsHalfTy ? APFloat::IEEEhalf() : APFloat::IEEEsingle());
346:     bool FtzEnabled = Mode.Output == DenormalMode::PreserveSign;
347:
348:     if (FtzEnabled != (Action.FtzRequirement == FTZ_MustBeOn))
349:       return nullptr;
350:   }
351:
352:   // Simplify to target-generic intrinsic.
353:   if (Action.IID) {
354:     SmallVector<Value *, 4> Args(II->args());
355:     // All the target-generic intrinsics currently of interest to us have one
356:     // type argument, equal to that of the nvvm intrinsic's argument.
357:     Type *Tys[] = {II->getArgOperand(0)->getType()};
358:     return CallInst::Create(
359:         Intrinsic::getOrInsertDeclaration(II->getModule(), *Action.IID, Tys),
360:         Args);
```
- EN: This range implements operational logic in helpers such as APFloat::IEEEhalf, Args, translating backend policy into executable code.
- CN: 这一段实现了 APFloat::IEEEhalf、Args 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:   }
362:
363:   // Simplify to target-generic binary op.
364:   if (Action.BinaryOp)
365:     return BinaryOperator::Create(*Action.BinaryOp, II->getArgOperand(0),
366:                                   II->getArgOperand(1), II->getName());
367:
368:   // Simplify to target-generic cast op.
369:   if (Action.CastOp)
370:     return CastInst::Create(*Action.CastOp, II->getArgOperand(0), II->getType(),
371:                             II->getName());
372:
373:   // All that's left are the special cases.
374:   if (!Action.Special)
375:     return nullptr;
376:
377:   switch (*Action.Special) {
378:   case SPC_Reciprocal:
379:     // Simplify reciprocal.
380:     return BinaryOperator::Create(
381:         Instruction::FDiv, ConstantFP::get(II->getArgOperand(0)->getType(), 1),
382:         II->getArgOperand(0), II->getName());
383:
384:   case SCP_FunnelShiftClamp: {
385:     // Canonicalize a clamping funnel shift to the generic llvm funnel shift
386:     // when possible, as this is easier for llvm to optimize further.
387:     if (const auto *ShiftConst = dyn_cast<ConstantInt>(II->getArgOperand(2))) {
388:       const bool IsLeft = II->getIntrinsicID() == Intrinsic::nvvm_fshl_clamp;
389:       if (ShiftConst->getZExtValue() >= II->getType()->getIntegerBitWidth())
390:         return IC.replaceInstUsesWith(*II, II->getArgOperand(IsLeft ? 1 : 0));
391:
392:       const unsigned FshIID = IsLeft ? Intrinsic::fshl : Intrinsic::fshr;
393:       return CallInst::Create(Intrinsic::getOrInsertDeclaration(
394:                                   II->getModule(), FshIID, II->getType()),
395:                               SmallVector<Value *, 3>(II->args()));
396:     }
397:     return nullptr;
398:   }
399:   }
400:   llvm_unreachable("All SpecialCase enumerators should be handled in switch.");
401: }
402:
403: // Returns true/false when we know the answer, nullopt otherwise.
404: static std::optional<bool> evaluateIsSpace(Intrinsic::ID IID, unsigned AS) {
405:   if (AS == NVPTXAS::ADDRESS_SPACE_GENERIC ||
406:       AS == NVPTXAS::ADDRESS_SPACE_ENTRY_PARAM)
407:     return std::nullopt; // Got to check at run-time.
408:   switch (IID) {
409:   case Intrinsic::nvvm_isspacep_global:
410:     return AS == NVPTXAS::ADDRESS_SPACE_GLOBAL;
411:   case Intrinsic::nvvm_isspacep_local:
412:     return AS == NVPTXAS::ADDRESS_SPACE_LOCAL;
413:   case Intrinsic::nvvm_isspacep_shared:
414:     // If shared cluster this can't be evaluated at compile time.
415:     if (AS == NVPTXAS::ADDRESS_SPACE_SHARED_CLUSTER)
416:       return std::nullopt;
417:     return AS == NVPTXAS::ADDRESS_SPACE_SHARED;
418:   case Intrinsic::nvvm_isspacep_shared_cluster:
419:     return AS == NVPTXAS::ADDRESS_SPACE_SHARED_CLUSTER ||
420:            AS == NVPTXAS::ADDRESS_SPACE_SHARED;
```
- EN: This range implements operational logic in helpers such as getArgOperand, getName, replaceInstUsesWith, args, translating backend policy into executable code.
- CN: 这一段实现了 getArgOperand、getName、replaceInstUsesWith、args 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:   case Intrinsic::nvvm_isspacep_const:
422:     return AS == NVPTXAS::ADDRESS_SPACE_CONST;
423:   default:
424:     llvm_unreachable("Unexpected intrinsic");
425:   }
426: }
427:
428: // Returns an instruction pointer (may be nullptr if we do not know the answer).
429: // Returns nullopt if `II` is not one of the `isspacep` intrinsics.
430: //
431: // TODO: If InferAddressSpaces were run early enough in the pipeline this could
432: // be removed in favor of the constant folding that occurs there through
433: // rewriteIntrinsicWithAddressSpace
434: static std::optional<Instruction *>
435: handleSpaceCheckIntrinsics(InstCombiner &IC, IntrinsicInst &II) {
436:
437:   switch (auto IID = II.getIntrinsicID()) {
438:   case Intrinsic::nvvm_isspacep_global:
439:   case Intrinsic::nvvm_isspacep_local:
440:   case Intrinsic::nvvm_isspacep_shared:
441:   case Intrinsic::nvvm_isspacep_shared_cluster:
442:   case Intrinsic::nvvm_isspacep_const: {
443:     Value *Op0 = II.getArgOperand(0);
444:     unsigned AS = Op0->getType()->getPointerAddressSpace();
445:     // Peek through ASC to generic AS.
446:     // TODO: we could dig deeper through both ASCs and GEPs.
447:     if (AS == NVPTXAS::ADDRESS_SPACE_GENERIC)
448:       if (auto *ASCO = dyn_cast<AddrSpaceCastOperator>(Op0))
449:         AS = ASCO->getOperand(0)->getType()->getPointerAddressSpace();
450:
451:     if (std::optional<bool> Answer = evaluateIsSpace(IID, AS))
452:       return IC.replaceInstUsesWith(II,
453:                                     ConstantInt::get(II.getType(), *Answer));
454:     return nullptr; // Don't know the answer, got to check at run time.
455:   }
456:   default:
457:     return std::nullopt;
458:   }
459: }
460:
461: std::optional<Instruction *>
462: NVPTXTTIImpl::instCombineIntrinsic(InstCombiner &IC, IntrinsicInst &II) const {
463:   if (std::optional<Instruction *> I = handleSpaceCheckIntrinsics(IC, II))
464:     return *I;
465:   if (Instruction *I = convertNvvmIntrinsicToLlvm(IC, &II))
466:     return I;
467:
468:   return std::nullopt;
469: }
470:
471: InstructionCost
472: NVPTXTTIImpl::getInstructionCost(const User *U,
473:                                  ArrayRef<const Value *> Operands,
474:                                  TTI::TargetCostKind CostKind) const {
475:   if (const auto *CI = dyn_cast<CallInst>(U))
476:     if (const auto *IA = dyn_cast<InlineAsm>(CI->getCalledOperand())) {
477:       // Without this implementation getCallCost() would return the number
478:       // of arguments+1 as the cost. Because the cost-model assumes it is a call
479:       // since it is classified as a call in the IR. A better cost model would
480:       // be to return the number of asm instructions embedded in the asm
```
- EN: This range implements operational logic in helpers such as llvm_unreachable, handleSpaceCheckIntrinsics, getArgOperand, getType, translating backend policy into executable code.
- CN: 这一段实现了 llvm_unreachable、handleSpaceCheckIntrinsics、getArgOperand、getType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-540
```cpp
481:       // string.
482:       StringRef AsmStr = IA->getAsmString();
483:       const unsigned InstCount =
484:           count_if(split(AsmStr, ';'), [](StringRef AsmInst) {
485:             // Trim off scopes denoted by '{' and '}' as these can be ignored
486:             AsmInst = AsmInst.trim().ltrim("{} \t\n\v\f\r");
487:             // This is pretty coarse but does a reasonably good job of
488:             // identifying things that look like instructions, possibly with a
489:             // predicate ("@").
490:             return !AsmInst.empty() &&
491:                    (AsmInst[0] == '@' || isAlpha(AsmInst[0]) ||
492:                     AsmInst.contains(".pragma"));
493:           });
494:       return InstCount * TargetTransformInfo::TCC_Basic;
495:     }
496:
497:   return BaseT::getInstructionCost(U, Operands, CostKind);
498: }
499:
500: InstructionCost NVPTXTTIImpl::getArithmeticInstrCost(
501:     unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
502:     TTI::OperandValueInfo Op1Info, TTI::OperandValueInfo Op2Info,
503:     ArrayRef<const Value *> Args, const Instruction *CxtI) const {
504:   // Legalize the type.
505:   std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(Ty);
506:
507:   int ISD = TLI->InstructionOpcodeToISD(Opcode);
508:
509:   switch (ISD) {
510:   default:
511:     return BaseT::getArithmeticInstrCost(Opcode, Ty, CostKind, Op1Info,
512:                                          Op2Info);
513:   case ISD::ADD:
514:   case ISD::MUL:
515:   case ISD::XOR:
516:   case ISD::OR:
517:   case ISD::AND:
518:     // The machine code (SASS) simulates an i64 with two i32. Therefore, we
519:     // estimate that arithmetic operations on i64 are twice as expensive as
520:     // those on types that can fit into one machine register.
521:     if (LT.second.SimpleTy == MVT::i64)
522:       return 2 * LT.first;
523:     // Delegate other cases to the basic TTI.
524:     return BaseT::getArithmeticInstrCost(Opcode, Ty, CostKind, Op1Info,
525:                                          Op2Info);
526:   }
527: }
528:
529: void NVPTXTTIImpl::getUnrollingPreferences(
530:     Loop *L, ScalarEvolution &SE, TTI::UnrollingPreferences &UP,
531:     OptimizationRemarkEmitter *ORE) const {
532:   BaseT::getUnrollingPreferences(L, SE, UP, ORE);
533:
534:   // Enable partial unrolling and runtime unrolling, but reduce the
535:   // threshold.  This partially unrolls small loops which are often
536:   // unrolled by the PTX to SASS compiler and unrolling earlier can be
537:   // beneficial.
538:   UP.Partial = UP.Runtime = true;
539:   UP.PartialThreshold = UP.Threshold / 4;
540: }
```
- EN: This range implements operational logic in helpers such as getAsmString, trim, contains, BaseT::getInstructionCost, translating backend policy into executable code.
- CN: 这一段实现了 getAsmString、trim、contains、BaseT::getInstructionCost 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-600
```cpp
541:
542: void NVPTXTTIImpl::getPeelingPreferences(Loop *L, ScalarEvolution &SE,
543:                                          TTI::PeelingPreferences &PP) const {
544:   BaseT::getPeelingPreferences(L, SE, PP);
545: }
546:
547: bool NVPTXTTIImpl::collectFlatAddressOperands(SmallVectorImpl<int> &OpIndexes,
548:                                               Intrinsic::ID IID) const {
549:   switch (IID) {
550:   case Intrinsic::nvvm_isspacep_const:
551:   case Intrinsic::nvvm_isspacep_global:
552:   case Intrinsic::nvvm_isspacep_local:
553:   case Intrinsic::nvvm_isspacep_shared:
554:   case Intrinsic::nvvm_isspacep_shared_cluster:
555:   case Intrinsic::nvvm_prefetch_tensormap: {
556:     OpIndexes.push_back(0);
557:     return true;
558:   }
559:   }
560:   return false;
561: }
562:
563: Value *NVPTXTTIImpl::rewriteIntrinsicWithAddressSpace(IntrinsicInst *II,
564:                                                       Value *OldV,
565:                                                       Value *NewV) const {
566:   const Intrinsic::ID IID = II->getIntrinsicID();
567:   switch (IID) {
568:   case Intrinsic::nvvm_isspacep_const:
569:   case Intrinsic::nvvm_isspacep_global:
570:   case Intrinsic::nvvm_isspacep_local:
571:   case Intrinsic::nvvm_isspacep_shared:
572:   case Intrinsic::nvvm_isspacep_shared_cluster: {
573:     const unsigned NewAS = NewV->getType()->getPointerAddressSpace();
574:     if (const auto R = evaluateIsSpace(IID, NewAS))
575:       return ConstantInt::get(II->getType(), *R);
576:     return nullptr;
577:   }
578:   case Intrinsic::nvvm_prefetch_tensormap: {
579:     IRBuilder<> Builder(II);
580:     const unsigned NewAS = NewV->getType()->getPointerAddressSpace();
581:     if (NewAS == NVPTXAS::ADDRESS_SPACE_CONST ||
582:         NewAS == NVPTXAS::ADDRESS_SPACE_ENTRY_PARAM)
583:       return Builder.CreateUnaryIntrinsic(Intrinsic::nvvm_prefetch_tensormap,
584:                                           NewV);
585:     return nullptr;
586:   }
587:   }
588:   return nullptr;
589: }
590:
591: bool NVPTXTTIImpl::isLegalMaskedStore(Type *DataTy, Align Alignment,
592:                                       unsigned AddrSpace,
593:                                       TTI::MaskKind MaskKind) const {
594:   if (MaskKind != TTI::MaskKind::ConstantMask)
595:     return false;
596:
597:   //  We currently only support this feature for 256-bit vectors, so the
598:   //  alignment must be at least 32
599:   if (Alignment < 32)
600:     return false;
```
- EN: This range implements operational logic in helpers such as BaseT::getPeelingPreferences, push_back, getIntrinsicID, getType, translating backend policy into executable code.
- CN: 这一段实现了 BaseT::getPeelingPreferences、push_back、getIntrinsicID、getType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 601-660
```cpp
601:
602:   if (!ST->has256BitVectorLoadStore(AddrSpace))
603:     return false;
604:
605:   auto *VTy = dyn_cast<FixedVectorType>(DataTy);
606:   if (!VTy)
607:     return false;
608:
609:   auto *ElemTy = VTy->getScalarType();
610:   return (ElemTy->getScalarSizeInBits() == 32 && VTy->getNumElements() == 8) ||
611:          (ElemTy->getScalarSizeInBits() == 64 && VTy->getNumElements() == 4);
612: }
613:
614: bool NVPTXTTIImpl::isLegalMaskedLoad(Type *DataTy, Align Alignment,
615:                                      unsigned /*AddrSpace*/,
616:                                      TTI::MaskKind MaskKind) const {
617:   if (MaskKind != TTI::MaskKind::ConstantMask)
618:     return false;
619:
620:   if (Alignment < DL.getTypeStoreSize(DataTy))
621:     return false;
622:
623:   // We do not support sub-byte element type masked loads.
624:   auto *VTy = dyn_cast<FixedVectorType>(DataTy);
625:   if (!VTy)
626:     return false;
627:   return VTy->getElementType()->getScalarSizeInBits() >= 8;
628: }
629:
630: unsigned NVPTXTTIImpl::getLoadStoreVecRegBitWidth(unsigned AddrSpace) const {
631:   // 256 bit loads/stores are currently only supported for global address space
632:   if (ST->has256BitVectorLoadStore(AddrSpace))
633:     return 256;
634:   return 128;
635: }
636:
637: unsigned NVPTXTTIImpl::getAssumedAddrSpace(const Value *V) const {
638:   if (isa<AllocaInst>(V))
639:     return ADDRESS_SPACE_LOCAL;
640:
641:   if (const Argument *Arg = dyn_cast<Argument>(V)) {
642:     if (isKernelFunction(*Arg->getParent())) {
643:       const NVPTXTargetMachine &TM =
644:           static_cast<const NVPTXTargetMachine &>(getTLI()->getTargetMachine());
645:       if (TM.getDrvInterface() == NVPTX::CUDA && !Arg->hasByValAttr())
646:         return ADDRESS_SPACE_GLOBAL;
647:     } else {
648:       // We assume that all device parameters that are passed byval will be
649:       // placed in the local AS. Very simple cases will be updated after ISel to
650:       // use the device param space where possible.
651:       if (Arg->hasByValAttr())
652:         return ADDRESS_SPACE_LOCAL;
653:     }
654:   }
655:
656:   return -1;
657: }
658:
659: void NVPTXTTIImpl::collectKernelLaunchBounds(
660:     const Function &F,
```
- EN: This range implements operational logic in helpers such as getScalarType, getScalarSizeInBits, NVPTXTTIImpl::getLoadStoreVecRegBitWidth, NVPTXTTIImpl::getAssumedAddrSpace, translating backend policy into executable code.
- CN: 这一段实现了 getScalarType、getScalarSizeInBits、NVPTXTTIImpl::getLoadStoreVecRegBitWidth、NVPTXTTIImpl::getAssumedAddrSpace 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 661-679
```cpp
661:     SmallVectorImpl<std::pair<StringRef, int64_t>> &LB) const {
662:   if (const auto Val = getMaxClusterRank(F))
663:     LB.push_back({"maxclusterrank", *Val});
664:
665:   const auto MaxNTID = getMaxNTID(F);
666:   if (MaxNTID.size() > 0)
667:     LB.push_back({"maxntidx", MaxNTID[0]});
668:   if (MaxNTID.size() > 1)
669:     LB.push_back({"maxntidy", MaxNTID[1]});
670:   if (MaxNTID.size() > 2)
671:     LB.push_back({"maxntidz", MaxNTID[2]});
672: }
673:
674: ValueUniformity NVPTXTTIImpl::getValueUniformity(const Value *V) const {
675:   if (isSourceOfDivergence(V))
676:     return ValueUniformity::NeverUniform;
677:
678:   return ValueUniformity::Default;
679: }
```
- EN: This range implements operational logic in helpers such as push_back, getMaxNTID, NVPTXTTIImpl::getValueUniformity, translating backend policy into executable code.
- CN: 这一段实现了 push_back、getMaxNTID、NVPTXTTIImpl::getValueUniformity 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include readsThreadIndex, readsLaneId, isNVVMAtomic, NVPTXTTIImpl::isSourceOfDivergence, isKernelFunction, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 readsThreadIndex, readsLaneId, isNVVMAtomic, NVPTXTTIImpl::isSourceOfDivergence, isKernelFunction，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXTargetTransformInfo.h`
  - `NVVMProperties.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/STLExtras.h`
  - `llvm/Analysis/LoopInfo.h`
  - `llvm/Analysis/TargetTransformInfo.h`
  - `llvm/Analysis/ValueTracking.h`
  - `llvm/CodeGen/BasicTTIImpl.h`
  - `llvm/CodeGen/TargetLowering.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsNVPTX.h`
  - `llvm/IR/Value.h`
  - `llvm/Support/Casting.h`
  - `llvm/Support/ErrorHandling.h`
  - `llvm/Support/NVPTXAddrSpace.h`
  - `llvm/Transforms/InstCombine/InstCombiner.h`
- System/standard headers / 系统或标准头文件:
  - `optional`
