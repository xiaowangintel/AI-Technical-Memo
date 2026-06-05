# SystemZTargetTransformInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZTargetTransformInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides target-specific optimization cost modelling for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责提供目标相关的优化开销建模。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```cpp
   1: //===-- SystemZTargetTransformInfo.cpp - SystemZ-specific TTI -------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements a TargetTransformInfo analysis pass specific to the
  10: // SystemZ target machine. It uses the target's detailed information to provide
  11: // more precise answers to certain TTI queries, while letting the target
  12: // independent and default TTI implementations handle the rest.
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: #include "SystemZTargetTransformInfo.h"
  17: #include "llvm/Analysis/TargetTransformInfo.h"
  18: #include "llvm/CodeGen/BasicTTIImpl.h"
  19: #include "llvm/CodeGen/TargetLowering.h"
  20: #include "llvm/IR/DerivedTypes.h"
  21: #include "llvm/IR/InstIterator.h"
  22: #include "llvm/IR/IntrinsicInst.h"
  23: #include "llvm/IR/Intrinsics.h"
  24: #include "llvm/Support/Debug.h"
  25: #include "llvm/Support/InstructionCost.h"
  26: #include "llvm/Support/MathExtras.h"
  27: 
  28: using namespace llvm;
  29: 
  30: #define DEBUG_TYPE "systemztti"
  31: 
  32: //===----------------------------------------------------------------------===//
  33: //
  34: // SystemZ cost model.
  35: //
  36: //===----------------------------------------------------------------------===//
  37: 
  38: static bool isUsedAsMemCpySource(const Value *V, bool &OtherUse) {
  39:   bool UsedAsMemCpySource = false;
  40:   for (const User *U : V->users())
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZTargetTransformInfo.h`, `TargetTransformInfo.h`, `BasicTTIImpl.h`, `TargetLowering.h`, `DerivedTypes.h`, `InstIterator.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `isUsedAsMemCpySource`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZTargetTransformInfo.h`, `TargetTransformInfo.h`, `BasicTTIImpl.h`, `TargetLowering.h`, `DerivedTypes.h`, `InstIterator.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `isUsedAsMemCpySource` 等函数。

### Lines 41-80 / 第 41-80 行
```cpp
  41:     if (const Instruction *User = dyn_cast<Instruction>(U)) {
  42:       if (isa<BitCastInst>(User) || isa<GetElementPtrInst>(User)) {
  43:         UsedAsMemCpySource |= isUsedAsMemCpySource(User, OtherUse);
  44:         continue;
  45:       }
  46:       if (const MemCpyInst *Memcpy = dyn_cast<MemCpyInst>(User)) {
  47:         if (Memcpy->getOperand(1) == V && !Memcpy->isVolatile()) {
  48:           UsedAsMemCpySource = true;
  49:           continue;
  50:         }
  51:       }
  52:       OtherUse = true;
  53:     }
  54:   return UsedAsMemCpySource;
  55: }
  56: 
  57: static void countNumMemAccesses(const Value *Ptr, unsigned &NumStores,
  58:                                 unsigned &NumLoads, const Function *F) {
  59:   if (!isa<PointerType>(Ptr->getType()))
  60:     return;
  61:   for (const User *U : Ptr->users())
  62:     if (const Instruction *User = dyn_cast<Instruction>(U)) {
  63:       if (User->getParent()->getParent() == F) {
  64:         if (const auto *SI = dyn_cast<StoreInst>(User)) {
  65:           if (SI->getPointerOperand() == Ptr && !SI->isVolatile())
  66:             NumStores++;
  67:         } else if (const auto *LI = dyn_cast<LoadInst>(User)) {
  68:           if (LI->getPointerOperand() == Ptr && !LI->isVolatile())
  69:             NumLoads++;
  70:         } else if (const auto *GEP = dyn_cast<GetElementPtrInst>(User)) {
  71:           if (GEP->getPointerOperand() == Ptr)
  72:             countNumMemAccesses(GEP, NumStores, NumLoads, F);
  73:         }
  74:       }
  75:     }
  76: }
  77: 
  78: unsigned SystemZTTIImpl::adjustInliningThreshold(const CallBase *CB) const {
  79:   unsigned Bonus = 0;
  80:   const Function *Caller = CB->getParent()->getParent();
```
- **EN**: The range implements or declares functions including `countNumMemAccesses`, `SystemZTTIImpl::adjustInliningThreshold`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `countNumMemAccesses`, `SystemZTTIImpl::adjustInliningThreshold` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 81-120 / 第 81-120 行
```cpp
  81:   const Function *Callee = CB->getCalledFunction();
  82:   if (!Callee)
  83:     return 0;
  84: 
  85:   // Increase the threshold if an incoming argument is used only as a memcpy
  86:   // source.
  87:   for (const Argument &Arg : Callee->args()) {
  88:     bool OtherUse = false;
  89:     if (isUsedAsMemCpySource(&Arg, OtherUse) && !OtherUse) {
  90:       Bonus = 1000;
  91:       break;
  92:     }
  93:   }
  94: 
  95:   // Give bonus for globals used much in both caller and a relatively small
  96:   // callee.
  97:   unsigned InstrCount = 0;
  98:   SmallDenseMap<const Value *, unsigned> Ptr2NumUses;
  99:   for (auto &I : instructions(Callee)) {
 100:     if (++InstrCount == 200) {
 101:       Ptr2NumUses.clear();
 102:       break;
 103:     }
 104:     if (const auto *SI = dyn_cast<StoreInst>(&I)) {
 105:       if (!SI->isVolatile())
 106:         if (auto *GV = dyn_cast<GlobalVariable>(SI->getPointerOperand()))
 107:           Ptr2NumUses[GV]++;
 108:     } else if (const auto *LI = dyn_cast<LoadInst>(&I)) {
 109:       if (!LI->isVolatile())
 110:         if (auto *GV = dyn_cast<GlobalVariable>(LI->getPointerOperand()))
 111:           Ptr2NumUses[GV]++;
 112:     } else if (const auto *GEP = dyn_cast<GetElementPtrInst>(&I)) {
 113:       if (auto *GV = dyn_cast<GlobalVariable>(GEP->getPointerOperand())) {
 114:         unsigned NumStores = 0, NumLoads = 0;
 115:         countNumMemAccesses(GEP, NumStores, NumLoads, Callee);
 116:         Ptr2NumUses[GV] += NumLoads + NumStores;
 117:       }
 118:     }
 119:   }
 120: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 121-160 / 第 121-160 行
```cpp
 121:   for (auto [Ptr, NumCalleeUses] : Ptr2NumUses)
 122:     if (NumCalleeUses > 10) {
 123:       unsigned CallerStores = 0, CallerLoads = 0;
 124:       countNumMemAccesses(Ptr, CallerStores, CallerLoads, Caller);
 125:       if (CallerStores + CallerLoads > 10) {
 126:         Bonus = 1000;
 127:         break;
 128:       }
 129:     }
 130: 
 131:   // Give bonus when Callee accesses an Alloca of Caller heavily.
 132:   unsigned NumStores = 0;
 133:   unsigned NumLoads = 0;
 134:   for (unsigned OpIdx = 0; OpIdx != Callee->arg_size(); ++OpIdx) {
 135:     Value *CallerArg = CB->getArgOperand(OpIdx);
 136:     Argument *CalleeArg = Callee->getArg(OpIdx);
 137:     if (isa<AllocaInst>(CallerArg))
 138:       countNumMemAccesses(CalleeArg, NumStores, NumLoads, Callee);
 139:   }
 140:   if (NumLoads > 10)
 141:     Bonus += NumLoads * 50;
 142:   if (NumStores > 10)
 143:     Bonus += NumStores * 50;
 144:   Bonus = std::min(Bonus, unsigned(1000));
 145: 
 146:   LLVM_DEBUG(if (Bonus)
 147:                dbgs() << "++ SZTTI Adding inlining bonus: " << Bonus << "\n";);
 148:   return Bonus;
 149: }
 150: 
 151: InstructionCost
 152: SystemZTTIImpl::getIntImmCost(const APInt &Imm, Type *Ty,
 153:                               TTI::TargetCostKind CostKind) const {
 154:   assert(Ty->isIntegerTy());
 155: 
 156:   unsigned BitSize = Ty->getPrimitiveSizeInBits();
 157:   // There is no cost model for constants with a bit size of 0. Return TCC_Free
 158:   // here, so that constant hoisting will ignore this constant.
 159:   if (BitSize == 0)
 160:     return TTI::TCC_Free;
```
- **EN**: The range implements or declares functions including `LLVM_DEBUG`, `SystemZTTIImpl::getIntImmCost`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LLVM_DEBUG`, `SystemZTTIImpl::getIntImmCost` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 161-200 / 第 161-200 行
```cpp
 161:   // No cost model for operations on integers larger than 128 bit implemented yet.
 162:   if ((!ST->hasVector() && BitSize > 64) || BitSize > 128)
 163:     return TTI::TCC_Free;
 164: 
 165:   if (Imm == 0)
 166:     return TTI::TCC_Free;
 167: 
 168:   if (Imm.getBitWidth() <= 64) {
 169:     // Constants loaded via lgfi.
 170:     if (isInt<32>(Imm.getSExtValue()))
 171:       return TTI::TCC_Basic;
 172:     // Constants loaded via llilf.
 173:     if (isUInt<32>(Imm.getZExtValue()))
 174:       return TTI::TCC_Basic;
 175:     // Constants loaded via llihf:
 176:     if ((Imm.getZExtValue() & 0xffffffff) == 0)
 177:       return TTI::TCC_Basic;
 178: 
 179:     return 2 * TTI::TCC_Basic;
 180:   }
 181: 
 182:   // i128 immediates loads from Constant Pool
 183:   return 2 * TTI::TCC_Basic;
 184: }
 185: 
 186: InstructionCost SystemZTTIImpl::getIntImmCostInst(unsigned Opcode, unsigned Idx,
 187:                                                   const APInt &Imm, Type *Ty,
 188:                                                   TTI::TargetCostKind CostKind,
 189:                                                   Instruction *Inst) const {
 190:   assert(Ty->isIntegerTy());
 191: 
 192:   unsigned BitSize = Ty->getPrimitiveSizeInBits();
 193:   // There is no cost model for constants with a bit size of 0. Return TCC_Free
 194:   // here, so that constant hoisting will ignore this constant.
 195:   if (BitSize == 0)
 196:     return TTI::TCC_Free;
 197:   // No cost model for operations on integers larger than 64 bit implemented yet.
 198:   if (BitSize > 64)
 199:     return TTI::TCC_Free;
 200: 
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getIntImmCostInst`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getIntImmCostInst` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 201-240 / 第 201-240 行
```cpp
 201:   switch (Opcode) {
 202:   default:
 203:     return TTI::TCC_Free;
 204:   case Instruction::GetElementPtr:
 205:     // Always hoist the base address of a GetElementPtr. This prevents the
 206:     // creation of new constants for every base constant that gets constant
 207:     // folded with the offset.
 208:     if (Idx == 0)
 209:       return 2 * TTI::TCC_Basic;
 210:     return TTI::TCC_Free;
 211:   case Instruction::Store:
 212:     if (Idx == 0 && Imm.getBitWidth() <= 64) {
 213:       // Any 8-bit immediate store can by implemented via mvi.
 214:       if (BitSize == 8)
 215:         return TTI::TCC_Free;
 216:       // 16-bit immediate values can be stored via mvhhi/mvhi/mvghi.
 217:       if (isInt<16>(Imm.getSExtValue()))
 218:         return TTI::TCC_Free;
 219:     }
 220:     break;
 221:   case Instruction::ICmp:
 222:     if (Idx == 1 && Imm.getBitWidth() <= 64) {
 223:       // Comparisons against signed 32-bit immediates implemented via cgfi.
 224:       if (isInt<32>(Imm.getSExtValue()))
 225:         return TTI::TCC_Free;
 226:       // Comparisons against unsigned 32-bit immediates implemented via clgfi.
 227:       if (isUInt<32>(Imm.getZExtValue()))
 228:         return TTI::TCC_Free;
 229:     }
 230:     break;
 231:   case Instruction::Add:
 232:   case Instruction::Sub:
 233:     if (Idx == 1 && Imm.getBitWidth() <= 64) {
 234:       // We use algfi/slgfi to add/subtract 32-bit unsigned immediates.
 235:       if (isUInt<32>(Imm.getZExtValue()))
 236:         return TTI::TCC_Free;
 237:       // Or their negation, by swapping addition vs. subtraction.
 238:       if (isUInt<32>(-Imm.getSExtValue()))
 239:         return TTI::TCC_Free;
 240:     }
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 241-280 / 第 241-280 行
```cpp
 241:     break;
 242:   case Instruction::Mul:
 243:     if (Idx == 1 && Imm.getBitWidth() <= 64) {
 244:       // We use msgfi to multiply by 32-bit signed immediates.
 245:       if (isInt<32>(Imm.getSExtValue()))
 246:         return TTI::TCC_Free;
 247:     }
 248:     break;
 249:   case Instruction::Or:
 250:   case Instruction::Xor:
 251:     if (Idx == 1 && Imm.getBitWidth() <= 64) {
 252:       // Masks supported by oilf/xilf.
 253:       if (isUInt<32>(Imm.getZExtValue()))
 254:         return TTI::TCC_Free;
 255:       // Masks supported by oihf/xihf.
 256:       if ((Imm.getZExtValue() & 0xffffffff) == 0)
 257:         return TTI::TCC_Free;
 258:     }
 259:     break;
 260:   case Instruction::And:
 261:     if (Idx == 1 && Imm.getBitWidth() <= 64) {
 262:       // Any 32-bit AND operation can by implemented via nilf.
 263:       if (BitSize <= 32)
 264:         return TTI::TCC_Free;
 265:       // 64-bit masks supported by nilf.
 266:       if (isUInt<32>(~Imm.getZExtValue()))
 267:         return TTI::TCC_Free;
 268:       // 64-bit masks supported by nilh.
 269:       if ((Imm.getZExtValue() & 0xffffffff) == 0xffffffff)
 270:         return TTI::TCC_Free;
 271:       // Some 64-bit AND operations can be implemented via risbg.
 272:       const SystemZInstrInfo *TII = ST->getInstrInfo();
 273:       unsigned Start, End;
 274:       if (TII->isRxSBGMask(Imm.getZExtValue(), BitSize, Start, End))
 275:         return TTI::TCC_Free;
 276:     }
 277:     break;
 278:   case Instruction::Shl:
 279:   case Instruction::LShr:
 280:   case Instruction::AShr:
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-320 / 第 281-320 行
```cpp
 281:     // Always return TCC_Free for the shift value of a shift instruction.
 282:     if (Idx == 1)
 283:       return TTI::TCC_Free;
 284:     break;
 285:   case Instruction::UDiv:
 286:   case Instruction::SDiv:
 287:   case Instruction::URem:
 288:   case Instruction::SRem:
 289:   case Instruction::Trunc:
 290:   case Instruction::ZExt:
 291:   case Instruction::SExt:
 292:   case Instruction::IntToPtr:
 293:   case Instruction::PtrToInt:
 294:   case Instruction::BitCast:
 295:   case Instruction::PHI:
 296:   case Instruction::Call:
 297:   case Instruction::Select:
 298:   case Instruction::Ret:
 299:   case Instruction::Load:
 300:     break;
 301:   }
 302: 
 303:   return SystemZTTIImpl::getIntImmCost(Imm, Ty, CostKind);
 304: }
 305: 
 306: InstructionCost
 307: SystemZTTIImpl::getIntImmCostIntrin(Intrinsic::ID IID, unsigned Idx,
 308:                                     const APInt &Imm, Type *Ty,
 309:                                     TTI::TargetCostKind CostKind) const {
 310:   assert(Ty->isIntegerTy());
 311: 
 312:   unsigned BitSize = Ty->getPrimitiveSizeInBits();
 313:   // There is no cost model for constants with a bit size of 0. Return TCC_Free
 314:   // here, so that constant hoisting will ignore this constant.
 315:   if (BitSize == 0)
 316:     return TTI::TCC_Free;
 317:   // No cost model for operations on integers larger than 64 bit implemented yet.
 318:   if (BitSize > 64)
 319:     return TTI::TCC_Free;
 320: 
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getIntImmCostIntrin`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getIntImmCostIntrin` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 321-360 / 第 321-360 行
```cpp
 321:   switch (IID) {
 322:   default:
 323:     return TTI::TCC_Free;
 324:   case Intrinsic::sadd_with_overflow:
 325:   case Intrinsic::uadd_with_overflow:
 326:   case Intrinsic::ssub_with_overflow:
 327:   case Intrinsic::usub_with_overflow:
 328:     // These get expanded to include a normal addition/subtraction.
 329:     if (Idx == 1 && Imm.getBitWidth() <= 64) {
 330:       if (isUInt<32>(Imm.getZExtValue()))
 331:         return TTI::TCC_Free;
 332:       if (isUInt<32>(-Imm.getSExtValue()))
 333:         return TTI::TCC_Free;
 334:     }
 335:     break;
 336:   case Intrinsic::smul_with_overflow:
 337:   case Intrinsic::umul_with_overflow:
 338:     // These get expanded to include a normal multiplication.
 339:     if (Idx == 1 && Imm.getBitWidth() <= 64) {
 340:       if (isInt<32>(Imm.getSExtValue()))
 341:         return TTI::TCC_Free;
 342:     }
 343:     break;
 344:   case Intrinsic::experimental_stackmap:
 345:     if ((Idx < 2) || (Imm.getBitWidth() <= 64 && isInt<64>(Imm.getSExtValue())))
 346:       return TTI::TCC_Free;
 347:     break;
 348:   case Intrinsic::experimental_patchpoint_void:
 349:   case Intrinsic::experimental_patchpoint:
 350:     if ((Idx < 4) || (Imm.getBitWidth() <= 64 && isInt<64>(Imm.getSExtValue())))
 351:       return TTI::TCC_Free;
 352:     break;
 353:   }
 354:   return SystemZTTIImpl::getIntImmCost(Imm, Ty, CostKind);
 355: }
 356: 
 357: TargetTransformInfo::PopcntSupportKind
 358: SystemZTTIImpl::getPopcntSupport(unsigned TyWidth) const {
 359:   assert(isPowerOf2_32(TyWidth) && "Type width must be power of 2");
 360:   if (ST->hasPopulationCount() && TyWidth <= 64)
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getPopcntSupport`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getPopcntSupport` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 361-400 / 第 361-400 行
```cpp
 361:     return TTI::PSK_FastHardware;
 362:   return TTI::PSK_Software;
 363: }
 364: 
 365: void SystemZTTIImpl::getUnrollingPreferences(
 366:     Loop *L, ScalarEvolution &SE, TTI::UnrollingPreferences &UP,
 367:     OptimizationRemarkEmitter *ORE) const {
 368:   // Find out if L contains a call, what the machine instruction count
 369:   // estimate is, and how many stores there are.
 370:   bool HasCall = false;
 371:   InstructionCost NumStores = 0;
 372:   for (auto &BB : L->blocks())
 373:     for (auto &I : *BB) {
 374:       if (isa<CallInst>(&I) || isa<InvokeInst>(&I)) {
 375:         if (const Function *F = cast<CallBase>(I).getCalledFunction()) {
 376:           if (isLoweredToCall(F))
 377:             HasCall = true;
 378:           if (F->getIntrinsicID() == Intrinsic::memcpy ||
 379:               F->getIntrinsicID() == Intrinsic::memset)
 380:             NumStores++;
 381:         } else { // indirect call.
 382:           HasCall = true;
 383:         }
 384:       }
 385:       if (isa<StoreInst>(&I)) {
 386:         Type *MemAccessTy = I.getOperand(0)->getType();
 387:         NumStores += getMemoryOpCost(Instruction::Store, MemAccessTy, Align(),
 388:                                      0, TTI::TCK_RecipThroughput);
 389:       }
 390:     }
 391: 
 392:   // The z13 processor will run out of store tags if too many stores
 393:   // are fed into it too quickly. Therefore make sure there are not
 394:   // too many stores in the resulting unrolled loop.
 395:   unsigned const NumStoresVal = NumStores.getValue();
 396:   unsigned const Max = (NumStoresVal ? (12 / NumStoresVal) : UINT_MAX);
 397: 
 398:   if (HasCall) {
 399:     // Only allow full unrolling if loop has any calls.
 400:     UP.FullUnrollMaxCount = Max;
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getUnrollingPreferences`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getUnrollingPreferences` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 401-440 / 第 401-440 行
```cpp
 401:     UP.MaxCount = 1;
 402:     return;
 403:   }
 404: 
 405:   UP.MaxCount = Max;
 406:   if (UP.MaxCount <= 1)
 407:     return;
 408: 
 409:   // Allow partial and runtime trip count unrolling.
 410:   UP.Partial = UP.Runtime = true;
 411: 
 412:   UP.PartialThreshold = 75;
 413:   UP.DefaultUnrollRuntimeCount = 4;
 414: 
 415:   // Allow expensive instructions in the pre-header of the loop.
 416:   UP.AllowExpensiveTripCount = true;
 417: 
 418:   UP.Force = true;
 419: }
 420: 
 421: void SystemZTTIImpl::getPeelingPreferences(Loop *L, ScalarEvolution &SE,
 422:                                            TTI::PeelingPreferences &PP) const {
 423:   BaseT::getPeelingPreferences(L, SE, PP);
 424: }
 425: 
 426: bool SystemZTTIImpl::isLSRCostLess(
 427:     const TargetTransformInfo::LSRCost &C1,
 428:     const TargetTransformInfo::LSRCost &C2) const {
 429:   // SystemZ specific: check instruction count (first), and don't care about
 430:   // ImmCost, since offsets are checked explicitly.
 431:   return std::tie(C1.Insns, C1.NumRegs, C1.AddRecCost,
 432:                   C1.NumIVMuls, C1.NumBaseAdds,
 433:                   C1.ScaleCost, C1.SetupCost) <
 434:     std::tie(C2.Insns, C2.NumRegs, C2.AddRecCost,
 435:              C2.NumIVMuls, C2.NumBaseAdds,
 436:              C2.ScaleCost, C2.SetupCost);
 437: }
 438: 
 439: unsigned SystemZTTIImpl::getNumberOfRegisters(unsigned ClassID) const {
 440:   bool Vector = (ClassID == 1);
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getPeelingPreferences`, `SystemZTTIImpl::isLSRCostLess`, `SystemZTTIImpl::getNumberOfRegisters`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getPeelingPreferences`, `SystemZTTIImpl::isLSRCostLess`, `SystemZTTIImpl::getNumberOfRegisters` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 441-480 / 第 441-480 行
```cpp
 441:   if (!Vector)
 442:     // Discount the stack pointer.  Also leave out %r0, since it can't
 443:     // be used in an address.
 444:     return 14;
 445:   if (ST->hasVector())
 446:     return 32;
 447:   return 0;
 448: }
 449: 
 450: TypeSize
 451: SystemZTTIImpl::getRegisterBitWidth(TargetTransformInfo::RegisterKind K) const {
 452:   switch (K) {
 453:   case TargetTransformInfo::RGK_Scalar:
 454:     return TypeSize::getFixed(64);
 455:   case TargetTransformInfo::RGK_FixedWidthVector:
 456:     return TypeSize::getFixed(ST->hasVector() ? 128 : 0);
 457:   case TargetTransformInfo::RGK_ScalableVector:
 458:     return TypeSize::getScalable(0);
 459:   }
 460: 
 461:   llvm_unreachable("Unsupported register kind");
 462: }
 463: 
 464: unsigned SystemZTTIImpl::getMinPrefetchStride(unsigned NumMemAccesses,
 465:                                               unsigned NumStridedMemAccesses,
 466:                                               unsigned NumPrefetches,
 467:                                               bool HasCall) const {
 468:   // Don't prefetch a loop with many far apart accesses.
 469:   if (NumPrefetches > 16)
 470:     return UINT_MAX;
 471: 
 472:   // Emit prefetch instructions for smaller strides in cases where we think
 473:   // the hardware prefetcher might not be able to keep up.
 474:   if (NumStridedMemAccesses > 32 && !HasCall &&
 475:       (NumMemAccesses - NumStridedMemAccesses) * 32 <= NumStridedMemAccesses)
 476:     return 1;
 477: 
 478:   return ST->hasMiscellaneousExtensions3() ? 8192 : 2048;
 479: }
 480: 
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getRegisterBitWidth`, `SystemZTTIImpl::getMinPrefetchStride`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getRegisterBitWidth`, `SystemZTTIImpl::getMinPrefetchStride` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 481-520 / 第 481-520 行
```cpp
 481: unsigned SystemZTTIImpl::getMaxInterleaveFactor(ElementCount VF) const {
 482:   return VF.isVector() ? 8 : 1;
 483: }
 484: 
 485: bool SystemZTTIImpl::hasDivRemOp(Type *DataType, bool IsSigned) const {
 486:   EVT VT = TLI->getValueType(DL, DataType);
 487:   return (VT.isScalarInteger() && TLI->isTypeLegal(VT));
 488: }
 489: 
 490: static bool isFreeEltLoad(const Value *Op) {
 491:   if (isa<LoadInst>(Op) && Op->hasOneUse()) {
 492:     const Instruction *UserI = cast<Instruction>(*Op->user_begin());
 493:     return !isa<StoreInst>(UserI); // Prefer MVC
 494:   }
 495:   return false;
 496: }
 497: 
 498: InstructionCost SystemZTTIImpl::getScalarizationOverhead(
 499:     VectorType *Ty, const APInt &DemandedElts, bool Insert, bool Extract,
 500:     TTI::TargetCostKind CostKind, bool ForPoisonSrc, ArrayRef<Value *> VL,
 501:     TTI::VectorInstrContext VIC) const {
 502:   unsigned NumElts = cast<FixedVectorType>(Ty)->getNumElements();
 503:   InstructionCost Cost = 0;
 504: 
 505:   if (Insert && Ty->isIntOrIntVectorTy(64)) {
 506:     // VLVGP will insert two GPRs with one instruction, while VLE will load
 507:     // an element directly with no extra cost
 508:     assert((VL.empty() || VL.size() == NumElts) &&
 509:            "Type does not match the number of values.");
 510:     InstructionCost CurrVectorCost = 0;
 511:     for (unsigned Idx = 0; Idx < NumElts; ++Idx) {
 512:       if (DemandedElts[Idx] && !(VL.size() && isFreeEltLoad(VL[Idx])))
 513:         ++CurrVectorCost;
 514:       if (Idx % 2 == 1) {
 515:         Cost += std::min(InstructionCost(1), CurrVectorCost);
 516:         CurrVectorCost = 0;
 517:       }
 518:     }
 519:     Insert = false;
 520:   }
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getMaxInterleaveFactor`, `SystemZTTIImpl::hasDivRemOp`, `isFreeEltLoad`, `SystemZTTIImpl::getScalarizationOverhead`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getMaxInterleaveFactor`, `SystemZTTIImpl::hasDivRemOp`, `isFreeEltLoad`, `SystemZTTIImpl::getScalarizationOverhead` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 521-560 / 第 521-560 行
```cpp
 521: 
 522:   Cost += BaseT::getScalarizationOverhead(Ty, DemandedElts, Insert, Extract,
 523:                                           CostKind, ForPoisonSrc, VL);
 524:   return Cost;
 525: }
 526: 
 527: // Return the bit size for the scalar type or vector element
 528: // type. getScalarSizeInBits() returns 0 for a pointer type.
 529: static unsigned getScalarSizeInBits(Type *Ty) {
 530:   unsigned Size =
 531:     (Ty->isPtrOrPtrVectorTy() ? 64U : Ty->getScalarSizeInBits());
 532:   assert(Size > 0 && "Element must have non-zero size.");
 533:   return Size;
 534: }
 535: 
 536: // getNumberOfParts() calls getTypeLegalizationCost() which splits the vector
 537: // type until it is legal. This would e.g. return 4 for <6 x i64>, instead of
 538: // 3.
 539: static unsigned getNumVectorRegs(Type *Ty) {
 540:   auto *VTy = cast<FixedVectorType>(Ty);
 541:   unsigned WideBits = getScalarSizeInBits(Ty) * VTy->getNumElements();
 542:   assert(WideBits > 0 && "Could not compute size of vector");
 543:   return ((WideBits % 128U) ? ((WideBits / 128U) + 1) : (WideBits / 128U));
 544: }
 545: 
 546: static bool isFoldableRMW(const Instruction *I, Type *Ty) {
 547:   auto *BI = dyn_cast_or_null<BinaryOperator>(I);
 548:   if (!BI || !BI->hasOneUse())
 549:     return false;
 550: 
 551:   unsigned Opcode = BI->getOpcode();
 552:   unsigned BitWidth = Ty->getScalarSizeInBits();
 553: 
 554:   switch (Opcode) {
 555:   case Instruction::And:
 556:   case Instruction::Or:
 557:   case Instruction::Xor:
 558:     if (BitWidth != 8)
 559:       return false;
 560:     break;
```
- **EN**: The range implements or declares functions including `getScalarSizeInBits`, `getNumVectorRegs`, `isFoldableRMW`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getScalarSizeInBits`, `getNumVectorRegs`, `isFoldableRMW` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 561-600 / 第 561-600 行
```cpp
 561:   case Instruction::Add:
 562:   case Instruction::Sub:
 563:     if (BitWidth != 32 && BitWidth != 64)
 564:       return false;
 565:     break;
 566:   default:
 567:     return false;
 568:   }
 569: 
 570:   Value *Op0 = BI->getOperand(0), *Op1 = BI->getOperand(1);
 571:   if (!isa<ConstantInt>(Op0) && !isa<ConstantInt>(Op1))
 572:     return false;
 573: 
 574:   Value *V =
 575:       (Opcode == Instruction::Sub) ? Op0 : (isa<ConstantInt>(Op0) ? Op1 : Op0);
 576:   if (Opcode == Instruction::Sub && !isa<ConstantInt>(Op1))
 577:     return false;
 578: 
 579:   auto *LI = dyn_cast_or_null<LoadInst>(V);
 580:   // Already checked BI hasOneUse.
 581:   auto *SI = dyn_cast<StoreInst>(BI->user_back());
 582: 
 583:   return LI && SI && !LI->isVolatile() && !SI->isVolatile() &&
 584:          LI->hasOneUse() && LI->getPointerOperand() == SI->getPointerOperand();
 585: }
 586: 
 587: InstructionCost SystemZTTIImpl::getArithmeticInstrCost(
 588:     unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
 589:     TTI::OperandValueInfo Op1Info, TTI::OperandValueInfo Op2Info,
 590:     ArrayRef<const Value *> Args, const Instruction *CxtI) const {
 591: 
 592:   // TODO: Handle more cost kinds.
 593:   if (CostKind != TTI::TCK_RecipThroughput)
 594:     return BaseT::getArithmeticInstrCost(Opcode, Ty, CostKind, Op1Info,
 595:                                          Op2Info, Args, CxtI);
 596:   if (CxtI && Ty && !Ty->isVectorTy() && isFoldableRMW(CxtI, Ty))
 597:     return TTI::TCC_Free;
 598:   // TODO: return a good value for BB-VECTORIZER that includes the
 599:   // immediate loads, which we do not want to count for the loop
 600:   // vectorizer, since they are hopefully hoisted out of the loop. This
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getArithmeticInstrCost`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getArithmeticInstrCost` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 601-640 / 第 601-640 行
```cpp
 601:   // would require a new parameter 'InLoop', but not sure if constant
 602:   // args are common enough to motivate this.
 603: 
 604:   unsigned ScalarBits = Ty->getScalarSizeInBits();
 605: 
 606:   // There are thre cases of division and remainder: Dividing with a register
 607:   // needs a divide instruction. A divisor which is a power of two constant
 608:   // can be implemented with a sequence of shifts. Any other constant needs a
 609:   // multiply and shifts.
 610:   const unsigned DivInstrCost = 20;
 611:   const unsigned DivMulSeqCost = 10;
 612:   const unsigned SDivPow2Cost = 4;
 613: 
 614:   bool SignedDivRem =
 615:       Opcode == Instruction::SDiv || Opcode == Instruction::SRem;
 616:   bool UnsignedDivRem =
 617:       Opcode == Instruction::UDiv || Opcode == Instruction::URem;
 618: 
 619:   // Check for a constant divisor.
 620:   bool DivRemConst = false;
 621:   bool DivRemConstPow2 = false;
 622:   if ((SignedDivRem || UnsignedDivRem) && Args.size() == 2) {
 623:     if (const Constant *C = dyn_cast<Constant>(Args[1])) {
 624:       const ConstantInt *CVal =
 625:           (C->getType()->isVectorTy()
 626:                ? dyn_cast_or_null<const ConstantInt>(C->getSplatValue())
 627:                : dyn_cast<const ConstantInt>(C));
 628:       if (CVal && (CVal->getValue().isPowerOf2() ||
 629:                    CVal->getValue().isNegatedPowerOf2()))
 630:         DivRemConstPow2 = true;
 631:       else
 632:         DivRemConst = true;
 633:     }
 634:   }
 635: 
 636:   if (!Ty->isVectorTy()) {
 637:     // These FP operations are supported with a dedicated instruction for
 638:     // float, double and fp128 (base implementation assumes float generally
 639:     // costs 2).
 640:     if (Opcode == Instruction::FAdd || Opcode == Instruction::FSub ||
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 641-680 / 第 641-680 行
```cpp
 641:         Opcode == Instruction::FMul || Opcode == Instruction::FDiv)
 642:       return 1;
 643: 
 644:     // There is no native support for FRem.
 645:     if (Opcode == Instruction::FRem)
 646:       return LIBCALL_COST;
 647: 
 648:     // Give discount for some combined logical operations if supported.
 649:     if (Args.size() == 2) {
 650:       if (Opcode == Instruction::Xor) {
 651:         for (const Value *A : Args) {
 652:           if (const Instruction *I = dyn_cast<Instruction>(A))
 653:             if (I->hasOneUse() &&
 654:                 (I->getOpcode() == Instruction::Or ||
 655:                  I->getOpcode() == Instruction::And ||
 656:                  I->getOpcode() == Instruction::Xor))
 657:               if ((ScalarBits <= 64 && ST->hasMiscellaneousExtensions3()) ||
 658:                   (isInt128InVR(Ty) &&
 659:                    (I->getOpcode() == Instruction::Or || ST->hasVectorEnhancements1())))
 660:                 return 0;
 661:         }
 662:       }
 663:       else if (Opcode == Instruction::And || Opcode == Instruction::Or) {
 664:         for (const Value *A : Args) {
 665:           if (const Instruction *I = dyn_cast<Instruction>(A))
 666:             if ((I->hasOneUse() && I->getOpcode() == Instruction::Xor) &&
 667:                 ((ScalarBits <= 64 && ST->hasMiscellaneousExtensions3()) ||
 668:                  (isInt128InVR(Ty) &&
 669:                   (Opcode == Instruction::And || ST->hasVectorEnhancements1()))))
 670:               return 0;
 671:         }
 672:       }
 673:     }
 674: 
 675:     // Or requires one instruction, although it has custom handling for i64.
 676:     if (Opcode == Instruction::Or)
 677:       return 1;
 678: 
 679:     if (Opcode == Instruction::Xor && ScalarBits == 1) {
 680:       if (ST->hasLoadStoreOnCond2())
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 681-720 / 第 681-720 行
```cpp
 681:         return 5; // 2 * (li 0; loc 1); xor
 682:       return 7; // 2 * ipm sequences ; xor ; shift ; compare
 683:     }
 684: 
 685:     if (DivRemConstPow2)
 686:       return (SignedDivRem ? SDivPow2Cost : 1);
 687:     if (DivRemConst)
 688:       return DivMulSeqCost;
 689:     if (SignedDivRem || UnsignedDivRem)
 690:       return DivInstrCost;
 691:   }
 692:   else if (ST->hasVector()) {
 693:     auto *VTy = cast<FixedVectorType>(Ty);
 694:     unsigned VF = VTy->getNumElements();
 695:     unsigned NumVectors = getNumVectorRegs(Ty);
 696: 
 697:     // These vector operations are custom handled, but are still supported
 698:     // with one instruction per vector, regardless of element size.
 699:     if (Opcode == Instruction::Shl || Opcode == Instruction::LShr ||
 700:         Opcode == Instruction::AShr) {
 701:       return NumVectors;
 702:     }
 703: 
 704:     if (DivRemConstPow2)
 705:       return (NumVectors * (SignedDivRem ? SDivPow2Cost : 1));
 706:     if (DivRemConst) {
 707:       SmallVector<Type *> Tys(Args.size(), Ty);
 708:       return VF * DivMulSeqCost +
 709:              BaseT::getScalarizationOverhead(VTy, Args, Tys, CostKind);
 710:     }
 711:     if (SignedDivRem || UnsignedDivRem) {
 712:       if (ST->hasVectorEnhancements3() && ScalarBits >= 32)
 713:         return NumVectors * DivInstrCost;
 714:       else if (VF > 4)
 715:         // Temporary hack: disable high vectorization factors with integer
 716:         // division/remainder, which will get scalarized and handled with
 717:         // GR128 registers. The mischeduler is not clever enough to avoid
 718:         // spilling yet.
 719:         return 1000;
 720:     }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 721-760 / 第 721-760 行
```cpp
 721: 
 722:     // These FP operations are supported with a single vector instruction for
 723:     // double (base implementation assumes float generally costs 2). For
 724:     // FP128, the scalar cost is 1, and there is no overhead since the values
 725:     // are already in scalar registers.
 726:     if (Opcode == Instruction::FAdd || Opcode == Instruction::FSub ||
 727:         Opcode == Instruction::FMul || Opcode == Instruction::FDiv) {
 728:       switch (ScalarBits) {
 729:       case 32: {
 730:         // The vector enhancements facility 1 provides v4f32 instructions.
 731:         if (ST->hasVectorEnhancements1())
 732:           return NumVectors;
 733:         // Return the cost of multiple scalar invocation plus the cost of
 734:         // inserting and extracting the values.
 735:         InstructionCost ScalarCost =
 736:             getArithmeticInstrCost(Opcode, Ty->getScalarType(), CostKind);
 737:         SmallVector<Type *> Tys(Args.size(), Ty);
 738:         InstructionCost Cost =
 739:             (VF * ScalarCost) +
 740:             BaseT::getScalarizationOverhead(VTy, Args, Tys, CostKind);
 741:         // FIXME: VF 2 for these FP operations are currently just as
 742:         // expensive as for VF 4.
 743:         if (VF == 2)
 744:           Cost *= 2;
 745:         return Cost;
 746:       }
 747:       case 64:
 748:       case 128:
 749:         return NumVectors;
 750:       default:
 751:         break;
 752:       }
 753:     }
 754: 
 755:     // There is no native support for FRem.
 756:     if (Opcode == Instruction::FRem) {
 757:       SmallVector<Type *> Tys(Args.size(), Ty);
 758:       InstructionCost Cost =
 759:           (VF * LIBCALL_COST) +
 760:           BaseT::getScalarizationOverhead(VTy, Args, Tys, CostKind);
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 761-800 / 第 761-800 行
```cpp
 761:       // FIXME: VF 2 for float is currently just as expensive as for VF 4.
 762:       if (VF == 2 && ScalarBits == 32)
 763:         Cost *= 2;
 764:       return Cost;
 765:     }
 766:   }
 767: 
 768:   // Fallback to the default implementation.
 769:   return BaseT::getArithmeticInstrCost(Opcode, Ty, CostKind, Op1Info, Op2Info,
 770:                                        Args, CxtI);
 771: }
 772: 
 773: InstructionCost
 774: SystemZTTIImpl::getShuffleCost(TTI::ShuffleKind Kind, VectorType *DstTy,
 775:                                VectorType *SrcTy, ArrayRef<int> Mask,
 776:                                TTI::TargetCostKind CostKind, int Index,
 777:                                VectorType *SubTp, ArrayRef<const Value *> Args,
 778:                                const Instruction *CxtI) const {
 779:   Kind = improveShuffleKindFromMask(Kind, Mask, SrcTy, Index, SubTp);
 780:   if (ST->hasVector()) {
 781:     unsigned NumVectors = getNumVectorRegs(SrcTy);
 782: 
 783:     // TODO: Since fp32 is expanded, the shuffle cost should always be 0.
 784: 
 785:     // FP128 values are always in scalar registers, so there is no work
 786:     // involved with a shuffle, except for broadcast. In that case register
 787:     // moves are done with a single instruction per element.
 788:     if (SrcTy->getScalarType()->isFP128Ty())
 789:       return (Kind == TargetTransformInfo::SK_Broadcast ? NumVectors - 1 : 0);
 790: 
 791:     switch (Kind) {
 792:     case  TargetTransformInfo::SK_ExtractSubvector:
 793:       // ExtractSubvector Index indicates start offset.
 794: 
 795:       // Extracting a subvector from first index is a noop.
 796:       return (Index == 0 ? 0 : NumVectors);
 797: 
 798:     case TargetTransformInfo::SK_Broadcast:
 799:       // Loop vectorizer calls here to figure out the extra cost of
 800:       // broadcasting a loaded value to all elements of a vector. Since vlrep
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getShuffleCost`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getShuffleCost` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 801-840 / 第 801-840 行
```cpp
 801:       // loads and replicates with a single instruction, adjust the returned
 802:       // value.
 803:       return NumVectors - 1;
 804: 
 805:     default:
 806: 
 807:       // SystemZ supports single instruction permutation / replication.
 808:       return NumVectors;
 809:     }
 810:   }
 811: 
 812:   return BaseT::getShuffleCost(Kind, DstTy, SrcTy, Mask, CostKind, Index,
 813:                                SubTp);
 814: }
 815: 
 816: // Return the log2 difference of the element sizes of the two vector types.
 817: static unsigned getElSizeLog2Diff(Type *Ty0, Type *Ty1) {
 818:   unsigned Bits0 = Ty0->getScalarSizeInBits();
 819:   unsigned Bits1 = Ty1->getScalarSizeInBits();
 820: 
 821:   if (Bits1 >  Bits0)
 822:     return (Log2_32(Bits1) - Log2_32(Bits0));
 823: 
 824:   return (Log2_32(Bits0) - Log2_32(Bits1));
 825: }
 826: 
 827: // Return the number of instructions needed to truncate SrcTy to DstTy.
 828: unsigned SystemZTTIImpl::getVectorTruncCost(Type *SrcTy, Type *DstTy) const {
 829:   assert (SrcTy->isVectorTy() && DstTy->isVectorTy());
 830:   assert(SrcTy->getPrimitiveSizeInBits().getFixedValue() >
 831:              DstTy->getPrimitiveSizeInBits().getFixedValue() &&
 832:          "Packing must reduce size of vector type.");
 833:   assert(cast<FixedVectorType>(SrcTy)->getNumElements() ==
 834:              cast<FixedVectorType>(DstTy)->getNumElements() &&
 835:          "Packing should not change number of elements.");
 836: 
 837:   // TODO: Since fp32 is expanded, the extract cost should always be 0.
 838: 
 839:   unsigned NumParts = getNumVectorRegs(SrcTy);
 840:   if (NumParts <= 2)
```
- **EN**: The range implements or declares functions including `getElSizeLog2Diff`, `SystemZTTIImpl::getVectorTruncCost`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `getElSizeLog2Diff`, `SystemZTTIImpl::getVectorTruncCost` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 841-880 / 第 841-880 行
```cpp
 841:     // Up to 2 vector registers can be truncated efficiently with pack or
 842:     // permute. The latter requires an immediate mask to be loaded, which
 843:     // typically gets hoisted out of a loop.  TODO: return a good value for
 844:     // BB-VECTORIZER that includes the immediate loads, which we do not want
 845:     // to count for the loop vectorizer.
 846:     return 1;
 847: 
 848:   unsigned Cost = 0;
 849:   unsigned Log2Diff = getElSizeLog2Diff(SrcTy, DstTy);
 850:   unsigned VF = cast<FixedVectorType>(SrcTy)->getNumElements();
 851:   for (unsigned P = 0; P < Log2Diff; ++P) {
 852:     if (NumParts > 1)
 853:       NumParts /= 2;
 854:     Cost += NumParts;
 855:   }
 856: 
 857:   // Currently, a general mix of permutes and pack instructions is output by
 858:   // isel, which follow the cost computation above except for this case which
 859:   // is one instruction less:
 860:   if (VF == 8 && SrcTy->getScalarSizeInBits() == 64 &&
 861:       DstTy->getScalarSizeInBits() == 8)
 862:     Cost--;
 863: 
 864:   return Cost;
 865: }
 866: 
 867: // Return the cost of converting a vector bitmask produced by a compare
 868: // (SrcTy), to the type of the select or extend instruction (DstTy).
 869: unsigned SystemZTTIImpl::getVectorBitmaskConversionCost(Type *SrcTy,
 870:                                                         Type *DstTy) const {
 871:   assert (SrcTy->isVectorTy() && DstTy->isVectorTy() &&
 872:           "Should only be called with vector types.");
 873: 
 874:   unsigned PackCost = 0;
 875:   unsigned SrcScalarBits = SrcTy->getScalarSizeInBits();
 876:   unsigned DstScalarBits = DstTy->getScalarSizeInBits();
 877:   unsigned Log2Diff = getElSizeLog2Diff(SrcTy, DstTy);
 878:   if (SrcScalarBits > DstScalarBits)
 879:     // The bitmask will be truncated.
 880:     PackCost = getVectorTruncCost(SrcTy, DstTy);
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getVectorBitmaskConversionCost`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getVectorBitmaskConversionCost` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 881-920 / 第 881-920 行
```cpp
 881:   else if (SrcScalarBits < DstScalarBits) {
 882:     unsigned DstNumParts = getNumVectorRegs(DstTy);
 883:     // Each vector select needs its part of the bitmask unpacked.
 884:     PackCost = Log2Diff * DstNumParts;
 885:     // Extra cost for moving part of mask before unpacking.
 886:     PackCost += DstNumParts - 1;
 887:   }
 888: 
 889:   return PackCost;
 890: }
 891: 
 892: // Return the type of the compared operands. This is needed to compute the
 893: // cost for a Select / ZExt or SExt instruction.
 894: static Type *getCmpOpsType(const Instruction *I, unsigned VF = 1) {
 895:   Type *OpTy = nullptr;
 896:   if (CmpInst *CI = dyn_cast<CmpInst>(I->getOperand(0)))
 897:     OpTy = CI->getOperand(0)->getType();
 898:   else if (Instruction *LogicI = dyn_cast<Instruction>(I->getOperand(0)))
 899:     if (LogicI->getNumOperands() == 2)
 900:       if (CmpInst *CI0 = dyn_cast<CmpInst>(LogicI->getOperand(0)))
 901:         if (isa<CmpInst>(LogicI->getOperand(1)))
 902:           OpTy = CI0->getOperand(0)->getType();
 903: 
 904:   if (OpTy != nullptr) {
 905:     if (VF == 1) {
 906:       assert (!OpTy->isVectorTy() && "Expected scalar type");
 907:       return OpTy;
 908:     }
 909:     // Return the potentially vectorized type based on 'I' and 'VF'.  'I' may
 910:     // be either scalar or already vectorized with a same or lesser VF.
 911:     Type *ElTy = OpTy->getScalarType();
 912:     return FixedVectorType::get(ElTy, VF);
 913:   }
 914: 
 915:   return nullptr;
 916: }
 917: 
 918: // Get the cost of converting a boolean vector to a vector with same width
 919: // and element size as Dst, plus the cost of zero extending if needed.
 920: unsigned
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 921-960 / 第 921-960 行
```cpp
 921: SystemZTTIImpl::getBoolVecToIntConversionCost(unsigned Opcode, Type *Dst,
 922:                                               const Instruction *I) const {
 923:   auto *DstVTy = cast<FixedVectorType>(Dst);
 924:   unsigned VF = DstVTy->getNumElements();
 925:   unsigned Cost = 0;
 926:   // If we know what the widths of the compared operands, get any cost of
 927:   // converting it to match Dst. Otherwise assume same widths.
 928:   Type *CmpOpTy = ((I != nullptr) ? getCmpOpsType(I, VF) : nullptr);
 929:   if (CmpOpTy != nullptr)
 930:     Cost = getVectorBitmaskConversionCost(CmpOpTy, Dst);
 931:   if (Opcode == Instruction::ZExt || Opcode == Instruction::UIToFP)
 932:     // One 'vn' per dst vector with an immediate mask.
 933:     Cost += getNumVectorRegs(Dst);
 934:   return Cost;
 935: }
 936: 
 937: InstructionCost SystemZTTIImpl::getCastInstrCost(unsigned Opcode, Type *Dst,
 938:                                                  Type *Src,
 939:                                                  TTI::CastContextHint CCH,
 940:                                                  TTI::TargetCostKind CostKind,
 941:                                                  const Instruction *I) const {
 942:   // FIXME: Can the logic below also be used for these cost kinds?
 943:   if (CostKind == TTI::TCK_CodeSize || CostKind == TTI::TCK_SizeAndLatency) {
 944:     auto BaseCost = BaseT::getCastInstrCost(Opcode, Dst, Src, CCH, CostKind, I);
 945:     return BaseCost == 0 ? BaseCost : 1;
 946:   }
 947: 
 948:   unsigned DstScalarBits = Dst->getScalarSizeInBits();
 949:   unsigned SrcScalarBits = Src->getScalarSizeInBits();
 950: 
 951:   if (!Src->isVectorTy()) {
 952:     if (Dst->isVectorTy())
 953:       return BaseT::getCastInstrCost(Opcode, Dst, Src, CCH, CostKind, I);
 954: 
 955:     if (Opcode == Instruction::SIToFP || Opcode == Instruction::UIToFP) {
 956:       if (Src->isIntegerTy(128))
 957:         return LIBCALL_COST;
 958:       if (SrcScalarBits >= 32 ||
 959:           (I != nullptr && isa<LoadInst>(I->getOperand(0))))
 960:         return 1;
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getCastInstrCost`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getCastInstrCost` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 961-1000 / 第 961-1000 行
```cpp
 961:       return SrcScalarBits > 1 ? 2 /*i8/i16 extend*/ : 5 /*branch seq.*/;
 962:     }
 963: 
 964:     if ((Opcode == Instruction::FPToSI || Opcode == Instruction::FPToUI) &&
 965:         Dst->isIntegerTy(128))
 966:       return LIBCALL_COST;
 967: 
 968:     if ((Opcode == Instruction::ZExt || Opcode == Instruction::SExt)) {
 969:       if (Src->isIntegerTy(1)) {
 970:         if (DstScalarBits == 128) {
 971:           if (Opcode == Instruction::SExt && ST->hasVectorEnhancements3())
 972:             return 0;/*VCEQQ*/
 973:           return 5 /*branch seq.*/;
 974:         }
 975: 
 976:         if (ST->hasLoadStoreOnCond2())
 977:           return 2; // li 0; loc 1
 978: 
 979:         // This should be extension of a compare i1 result, which is done with
 980:         // ipm and a varying sequence of instructions.
 981:         unsigned Cost = 0;
 982:         if (Opcode == Instruction::SExt)
 983:           Cost = (DstScalarBits < 64 ? 3 : 4);
 984:         if (Opcode == Instruction::ZExt)
 985:           Cost = 3;
 986:         Type *CmpOpTy = ((I != nullptr) ? getCmpOpsType(I) : nullptr);
 987:         if (CmpOpTy != nullptr && CmpOpTy->isFloatingPointTy())
 988:           // If operands of an fp-type was compared, this costs +1.
 989:           Cost++;
 990:         return Cost;
 991:       }
 992:       else if (isInt128InVR(Dst)) {
 993:         // Extensions from GPR to i128 (in VR) typically costs two instructions,
 994:         // but a zero-extending load would be just one extra instruction.
 995:         if (Opcode == Instruction::ZExt && I != nullptr)
 996:           if (LoadInst *Ld = dyn_cast<LoadInst>(I->getOperand(0)))
 997:             if (Ld->hasOneUse())
 998:               return 1;
 999:         return 2;
1000:       }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1001-1040 / 第 1001-1040 行
```cpp
1001:     }
1002: 
1003:     if (Opcode == Instruction::Trunc && isInt128InVR(Src) && I != nullptr) {
1004:       if (LoadInst *Ld = dyn_cast<LoadInst>(I->getOperand(0)))
1005:         if (Ld->hasOneUse())
1006:           return 0;  // Will be converted to GPR load.
1007:       bool OnlyTruncatingStores = true;
1008:       for (const User *U : I->users())
1009:         if (!isa<StoreInst>(U)) {
1010:           OnlyTruncatingStores = false;
1011:           break;
1012:         }
1013:       if (OnlyTruncatingStores)
1014:         return 0;
1015:       return 2; // Vector element extraction.
1016:     }
1017:   }
1018:   else if (ST->hasVector()) {
1019:     // Vector to scalar cast.
1020:     auto *SrcVecTy = cast<FixedVectorType>(Src);
1021:     auto *DstVecTy = dyn_cast<FixedVectorType>(Dst);
1022:     if (!DstVecTy) {
1023:       // TODO: tune vector-to-scalar cast.
1024:       return BaseT::getCastInstrCost(Opcode, Dst, Src, CCH, CostKind, I);
1025:     }
1026:     unsigned VF = SrcVecTy->getNumElements();
1027:     unsigned NumDstVectors = getNumVectorRegs(Dst);
1028:     unsigned NumSrcVectors = getNumVectorRegs(Src);
1029: 
1030:     if (Opcode == Instruction::Trunc) {
1031:       if (Src->getScalarSizeInBits() == Dst->getScalarSizeInBits())
1032:         return 0; // Check for NOOP conversions.
1033:       return getVectorTruncCost(Src, Dst);
1034:     }
1035: 
1036:     if (Opcode == Instruction::ZExt || Opcode == Instruction::SExt) {
1037:       if (SrcScalarBits >= 8) {
1038:         // ZExt will use either a single unpack or a vector permute.
1039:         if (Opcode == Instruction::ZExt)
1040:           return NumDstVectors;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1041-1080 / 第 1041-1080 行
```cpp
1041: 
1042:         // SExt will be handled with one unpack per doubling of width.
1043:         unsigned NumUnpacks = getElSizeLog2Diff(Src, Dst);
1044: 
1045:         // For types that spans multiple vector registers, some additional
1046:         // instructions are used to setup the unpacking.
1047:         unsigned NumSrcVectorOps =
1048:           (NumUnpacks > 1 ? (NumDstVectors - NumSrcVectors)
1049:                           : (NumDstVectors / 2));
1050: 
1051:         return (NumUnpacks * NumDstVectors) + NumSrcVectorOps;
1052:       }
1053:       else if (SrcScalarBits == 1)
1054:         return getBoolVecToIntConversionCost(Opcode, Dst, I);
1055:     }
1056: 
1057:     if (Opcode == Instruction::SIToFP || Opcode == Instruction::UIToFP ||
1058:         Opcode == Instruction::FPToSI || Opcode == Instruction::FPToUI) {
1059:       // TODO: Fix base implementation which could simplify things a bit here
1060:       // (seems to miss on differentiating on scalar/vector types).
1061: 
1062:       // Only 64 bit vector conversions are natively supported before z15.
1063:       if (DstScalarBits == 64 || ST->hasVectorEnhancements2()) {
1064:         if (SrcScalarBits == DstScalarBits)
1065:           return NumDstVectors;
1066: 
1067:         if (SrcScalarBits == 1)
1068:           return getBoolVecToIntConversionCost(Opcode, Dst, I) + NumDstVectors;
1069:       }
1070: 
1071:       // Return the cost of multiple scalar invocation plus the cost of
1072:       // inserting and extracting the values. Base implementation does not
1073:       // realize float->int gets scalarized.
1074:       InstructionCost ScalarCost = getCastInstrCost(
1075:           Opcode, Dst->getScalarType(), Src->getScalarType(), CCH, CostKind);
1076:       InstructionCost TotCost = VF * ScalarCost;
1077:       bool NeedsInserts = true, NeedsExtracts = true;
1078:       // FP128 registers do not get inserted or extracted.
1079:       if (DstScalarBits == 128 &&
1080:           (Opcode == Instruction::SIToFP || Opcode == Instruction::UIToFP))
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1081-1120 / 第 1081-1120 行
```cpp
1081:         NeedsInserts = false;
1082:       if (SrcScalarBits == 128 &&
1083:           (Opcode == Instruction::FPToSI || Opcode == Instruction::FPToUI))
1084:         NeedsExtracts = false;
1085: 
1086:       TotCost += BaseT::getScalarizationOverhead(SrcVecTy, /*Insert*/ false,
1087:                                                  NeedsExtracts, CostKind);
1088:       TotCost += BaseT::getScalarizationOverhead(DstVecTy, NeedsInserts,
1089:                                                  /*Extract*/ false, CostKind);
1090: 
1091:       // FIXME: VF 2 for float<->i32 is currently just as expensive as for VF 4.
1092:       if (VF == 2 && SrcScalarBits == 32 && DstScalarBits == 32)
1093:         TotCost *= 2;
1094: 
1095:       return TotCost;
1096:     }
1097: 
1098:     if (Opcode == Instruction::FPTrunc) {
1099:       if (SrcScalarBits == 128)  // fp128 -> double/float + inserts of elements.
1100:         return VF /*ldxbr/lexbr*/ +
1101:                BaseT::getScalarizationOverhead(DstVecTy, /*Insert*/ true,
1102:                                                /*Extract*/ false, CostKind);
1103:       else // double -> float
1104:         return VF / 2 /*vledb*/ + std::max(1U, VF / 4 /*vperm*/);
1105:     }
1106: 
1107:     if (Opcode == Instruction::FPExt) {
1108:       if (SrcScalarBits == 32 && DstScalarBits == 64) {
1109:         // float -> double is very rare and currently unoptimized. Instead of
1110:         // using vldeb, which can do two at a time, all conversions are
1111:         // scalarized.
1112:         return VF * 2;
1113:       }
1114:       // -> fp128.  VF * lxdb/lxeb + extraction of elements.
1115:       return VF + BaseT::getScalarizationOverhead(SrcVecTy, /*Insert*/ false,
1116:                                                   /*Extract*/ true, CostKind);
1117:     }
1118:   }
1119: 
1120:   return BaseT::getCastInstrCost(Opcode, Dst, Src, CCH, CostKind, I);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1121-1160 / 第 1121-1160 行
```cpp
1121: }
1122: 
1123: // Scalar i8 / i16 operations will typically be made after first extending
1124: // the operands to i32.
1125: static unsigned getOperandsExtensionCost(const Instruction *I) {
1126:   unsigned ExtCost = 0;
1127:   for (Value *Op : I->operands())
1128:     // A load of i8 or i16 sign/zero extends to i32.
1129:     if (!isa<LoadInst>(Op) && !isa<ConstantInt>(Op))
1130:       ExtCost++;
1131: 
1132:   return ExtCost;
1133: }
1134: 
1135: InstructionCost SystemZTTIImpl::getCFInstrCost(unsigned Opcode,
1136:                                                TTI::TargetCostKind CostKind,
1137:                                                const Instruction *I) const {
1138:   if (CostKind != TTI::TCK_RecipThroughput)
1139:     return Opcode == Instruction::PHI ? TTI::TCC_Free : TTI::TCC_Basic;
1140:   // Branches are assumed to be predicted.
1141:   return TTI::TCC_Free;
1142: }
1143: 
1144: InstructionCost SystemZTTIImpl::getCmpSelInstrCost(
1145:     unsigned Opcode, Type *ValTy, Type *CondTy, CmpInst::Predicate VecPred,
1146:     TTI::TargetCostKind CostKind, TTI::OperandValueInfo Op1Info,
1147:     TTI::OperandValueInfo Op2Info, const Instruction *I) const {
1148:   if (CostKind != TTI::TCK_RecipThroughput)
1149:     return BaseT::getCmpSelInstrCost(Opcode, ValTy, CondTy, VecPred, CostKind,
1150:                                      Op1Info, Op2Info);
1151: 
1152:   if (!ValTy->isVectorTy()) {
1153:     switch (Opcode) {
1154:     case Instruction::ICmp: {
1155:       // A loaded value compared with 0 with multiple users becomes Load and
1156:       // Test. The load is then not foldable, so return 0 cost for the ICmp.
1157:       unsigned ScalarBits = ValTy->getScalarSizeInBits();
1158:       if (I != nullptr && (ScalarBits == 32 || ScalarBits == 64))
1159:         if (LoadInst *Ld = dyn_cast<LoadInst>(I->getOperand(0)))
1160:           if (const ConstantInt *C = dyn_cast<ConstantInt>(I->getOperand(1)))
```
- **EN**: The range implements or declares functions including `getOperandsExtensionCost`, `SystemZTTIImpl::getCFInstrCost`, `SystemZTTIImpl::getCmpSelInstrCost`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getOperandsExtensionCost`, `SystemZTTIImpl::getCFInstrCost`, `SystemZTTIImpl::getCmpSelInstrCost` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1161-1200 / 第 1161-1200 行
```cpp
1161:             if (!Ld->hasOneUse() && Ld->getParent() == I->getParent() &&
1162:                 C->isZero())
1163:               return 0;
1164: 
1165:       unsigned Cost = 1;
1166:       if (ValTy->isIntegerTy() && ValTy->getScalarSizeInBits() <= 16)
1167:         Cost += (I != nullptr ? getOperandsExtensionCost(I) : 2);
1168:       return Cost;
1169:     }
1170:     case Instruction::Select:
1171:       if (ValTy->isFloatingPointTy())
1172:         return 4; // No LOC for FP - costs a conditional jump.
1173: 
1174:       // When selecting based on an i128 comparison, LOC / VSEL is possible
1175:       // if i128 comparisons are directly supported.
1176:       if (I != nullptr)
1177:         if (ICmpInst *CI = dyn_cast<ICmpInst>(I->getOperand(0)))
1178:           if (CI->getOperand(0)->getType()->isIntegerTy(128))
1179:             return ST->hasVectorEnhancements3() ? 1 : 4;
1180: 
1181:       // Load On Condition / Select Register available, except for i128.
1182:       return !isInt128InVR(ValTy) ? 1 : 4;
1183:     }
1184:   }
1185:   else if (ST->hasVector()) {
1186:     unsigned VF = cast<FixedVectorType>(ValTy)->getNumElements();
1187: 
1188:     // Called with a compare instruction.
1189:     if (Opcode == Instruction::ICmp || Opcode == Instruction::FCmp) {
1190:       unsigned PredicateExtraCost = 0;
1191:       if (I != nullptr) {
1192:         // Some predicates cost one or two extra instructions.
1193:         switch (cast<CmpInst>(I)->getPredicate()) {
1194:         case CmpInst::Predicate::ICMP_NE:
1195:         case CmpInst::Predicate::ICMP_UGE:
1196:         case CmpInst::Predicate::ICMP_ULE:
1197:         case CmpInst::Predicate::ICMP_SGE:
1198:         case CmpInst::Predicate::ICMP_SLE:
1199:           PredicateExtraCost = 1;
1200:           break;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1201-1240 / 第 1201-1240 行
```cpp
1201:         case CmpInst::Predicate::FCMP_ONE:
1202:         case CmpInst::Predicate::FCMP_ORD:
1203:         case CmpInst::Predicate::FCMP_UEQ:
1204:         case CmpInst::Predicate::FCMP_UNO:
1205:           PredicateExtraCost = 2;
1206:           break;
1207:         default:
1208:           break;
1209:         }
1210:       }
1211: 
1212:       // Float is handled with 2*vmr[lh]f + 2*vldeb + vfchdb for each pair of
1213:       // floats.  FIXME: <2 x float> generates same code as <4 x float>.
1214:       unsigned CmpCostPerVector = (ValTy->getScalarType()->isFloatTy() ? 10 : 1);
1215:       unsigned NumVecs_cmp = getNumVectorRegs(ValTy);
1216: 
1217:       unsigned Cost = (NumVecs_cmp * (CmpCostPerVector + PredicateExtraCost));
1218:       return Cost;
1219:     }
1220:     else { // Called with a select instruction.
1221:       assert (Opcode == Instruction::Select);
1222: 
1223:       // We can figure out the extra cost of packing / unpacking if the
1224:       // instruction was passed and the compare instruction is found.
1225:       unsigned PackCost = 0;
1226:       Type *CmpOpTy = ((I != nullptr) ? getCmpOpsType(I, VF) : nullptr);
1227:       if (CmpOpTy != nullptr)
1228:         PackCost =
1229:           getVectorBitmaskConversionCost(CmpOpTy, ValTy);
1230: 
1231:       return getNumVectorRegs(ValTy) /*vsel*/ + PackCost;
1232:     }
1233:   }
1234: 
1235:   return BaseT::getCmpSelInstrCost(Opcode, ValTy, CondTy, VecPred, CostKind,
1236:                                    Op1Info, Op2Info);
1237: }
1238: 
1239: InstructionCost SystemZTTIImpl::getVectorInstrCost(
1240:     unsigned Opcode, Type *Val, TTI::TargetCostKind CostKind, unsigned Index,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1241-1280 / 第 1241-1280 行
```cpp
1241:     const Value *Op0, const Value *Op1, TTI::VectorInstrContext VIC) const {
1242:   if (Opcode == Instruction::InsertElement) {
1243:     // Vector Element Load.
1244:     if (Op1 != nullptr && isFreeEltLoad(Op1))
1245:       return 0;
1246: 
1247:     // vlvgp will insert two grs into a vector register, so count half the
1248:     // number of instructions as an estimate when we don't have the full
1249:     // picture (as in getScalarizationOverhead()).
1250:     if (Val->isIntOrIntVectorTy(64))
1251:       return ((Index % 2 == 0) ? 1 : 0);
1252:   }
1253: 
1254:   if (Opcode == Instruction::ExtractElement) {
1255:     int Cost = ((getScalarSizeInBits(Val) == 1) ? 2 /*+test-under-mask*/ : 1);
1256: 
1257:     // Give a slight penalty for moving out of vector pipeline to FXU unit.
1258:     if (Index == 0 && Val->isIntOrIntVectorTy())
1259:       Cost += 1;
1260: 
1261:     return Cost;
1262:   }
1263: 
1264:   return BaseT::getVectorInstrCost(Opcode, Val, CostKind, Index, Op0, Op1, VIC);
1265: }
1266: 
1267: // Check if a load may be folded as a memory operand in its user.
1268: bool SystemZTTIImpl::isFoldableLoad(const LoadInst *Ld,
1269:                                     const Instruction *&FoldedValue) const {
1270:   if (!Ld->hasOneUse())
1271:     return false;
1272:   FoldedValue = Ld;
1273:   const Instruction *UserI = cast<Instruction>(*Ld->user_begin());
1274:   unsigned LoadedBits = getScalarSizeInBits(Ld->getType());
1275:   unsigned TruncBits = 0;
1276:   unsigned SExtBits = 0;
1277:   unsigned ZExtBits = 0;
1278:   if (UserI->hasOneUse()) {
1279:     unsigned UserBits = UserI->getType()->getScalarSizeInBits();
1280:     if (isa<TruncInst>(UserI))
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::isFoldableLoad`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::isFoldableLoad` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1281-1320 / 第 1281-1320 行
```cpp
1281:       TruncBits = UserBits;
1282:     else if (isa<SExtInst>(UserI))
1283:       SExtBits = UserBits;
1284:     else if (isa<ZExtInst>(UserI))
1285:       ZExtBits = UserBits;
1286:   }
1287:   if (TruncBits || SExtBits || ZExtBits) {
1288:     FoldedValue = UserI;
1289:     UserI = cast<Instruction>(*UserI->user_begin());
1290:     // Load (single use) -> trunc/extend (single use) -> UserI
1291:   }
1292:   if ((UserI->getOpcode() == Instruction::Sub ||
1293:        UserI->getOpcode() == Instruction::SDiv ||
1294:        UserI->getOpcode() == Instruction::UDiv) &&
1295:       UserI->getOperand(1) != FoldedValue)
1296:     return false; // Not commutative, only RHS foldable.
1297:   // LoadOrTruncBits holds the number of effectively loaded bits, but 0 if an
1298:   // extension was made of the load.
1299:   unsigned LoadOrTruncBits =
1300:       ((SExtBits || ZExtBits) ? 0 : (TruncBits ? TruncBits : LoadedBits));
1301:   switch (UserI->getOpcode()) {
1302:   case Instruction::Add: // SE: 16->32, 16/32->64, z14:16->64. ZE: 32->64
1303:   case Instruction::Sub:
1304:   case Instruction::ICmp:
1305:     if (LoadedBits == 32 && ZExtBits == 64)
1306:       return true;
1307:     [[fallthrough]];
1308:   case Instruction::Mul: // SE: 16->32, 32->64, z14:16->64
1309:     if (UserI->getOpcode() != Instruction::ICmp) {
1310:       if (LoadedBits == 16 &&
1311:           (SExtBits == 32 ||
1312:            (SExtBits == 64 && ST->hasMiscellaneousExtensions2())))
1313:         return true;
1314:       if (LoadOrTruncBits == 16)
1315:         return true;
1316:     }
1317:     [[fallthrough]];
1318:   case Instruction::SDiv:// SE: 32->64
1319:     if (LoadedBits == 32 && SExtBits == 64)
1320:       return true;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1321-1360 / 第 1321-1360 行
```cpp
1321:     [[fallthrough]];
1322:   case Instruction::UDiv:
1323:   case Instruction::And:
1324:   case Instruction::Or:
1325:   case Instruction::Xor:
1326:     // This also makes sense for float operations, but disabled for now due
1327:     // to regressions.
1328:     // case Instruction::FCmp:
1329:     // case Instruction::FAdd:
1330:     // case Instruction::FSub:
1331:     // case Instruction::FMul:
1332:     // case Instruction::FDiv:
1333: 
1334:     // All possible extensions of memory checked above.
1335: 
1336:     // Comparison between memory and immediate.
1337:     if (UserI->getOpcode() == Instruction::ICmp)
1338:       if (ConstantInt *CI = dyn_cast<ConstantInt>(UserI->getOperand(1)))
1339:         if (CI->getValue().isIntN(16))
1340:           return true;
1341:     return (LoadOrTruncBits == 32 || LoadOrTruncBits == 64);
1342:     break;
1343:   }
1344:   return false;
1345: }
1346: 
1347: static bool isBswapIntrinsicCall(const Value *V) {
1348:   if (const Instruction *I = dyn_cast<Instruction>(V))
1349:     if (auto *CI = dyn_cast<CallInst>(I))
1350:       if (auto *F = CI->getCalledFunction())
1351:         if (F->getIntrinsicID() == Intrinsic::bswap)
1352:           return true;
1353:   return false;
1354: }
1355: 
1356: InstructionCost SystemZTTIImpl::getMemoryOpCost(unsigned Opcode, Type *Src,
1357:                                                 Align Alignment,
1358:                                                 unsigned AddressSpace,
1359:                                                 TTI::TargetCostKind CostKind,
1360:                                                 TTI::OperandValueInfo OpInfo,
```
- **EN**: The range implements or declares functions including `isBswapIntrinsicCall`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isBswapIntrinsicCall` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1361-1400 / 第 1361-1400 行
```cpp
1361:                                                 const Instruction *I) const {
1362:   assert(!Src->isVoidTy() && "Invalid type");
1363: 
1364:   // TODO: Handle other cost kinds.
1365:   if (CostKind != TTI::TCK_RecipThroughput)
1366:     return 1;
1367: 
1368:   if (I && Opcode == Instruction::Store && !Src->isVectorTy()) {
1369:     if (isFoldableRMW(dyn_cast<Instruction>(I->getOperand(0)), Src))
1370:       return TTI::TCC_Free;
1371:   }
1372: 
1373:   if (!Src->isVectorTy() && Opcode == Instruction::Load && I != nullptr) {
1374:     // Store the load or its truncated or extended value in FoldedValue.
1375:     const Instruction *FoldedValue = nullptr;
1376:     if (isFoldableLoad(cast<LoadInst>(I), FoldedValue)) {
1377:       const Instruction *UserI = cast<Instruction>(*FoldedValue->user_begin());
1378:       assert (UserI->getNumOperands() == 2 && "Expected a binop.");
1379: 
1380:       // UserI can't fold two loads, so in that case return 0 cost only
1381:       // half of the time.
1382:       for (unsigned i = 0; i < 2; ++i) {
1383:         if (UserI->getOperand(i) == FoldedValue)
1384:           continue;
1385: 
1386:         if (Instruction *OtherOp = dyn_cast<Instruction>(UserI->getOperand(i))){
1387:           LoadInst *OtherLoad = dyn_cast<LoadInst>(OtherOp);
1388:           if (!OtherLoad &&
1389:               (isa<TruncInst>(OtherOp) || isa<SExtInst>(OtherOp) ||
1390:                isa<ZExtInst>(OtherOp)))
1391:             OtherLoad = dyn_cast<LoadInst>(OtherOp->getOperand(0));
1392:           if (OtherLoad && isFoldableLoad(OtherLoad, FoldedValue/*dummy*/))
1393:             return i == 0; // Both operands foldable.
1394:         }
1395:       }
1396: 
1397:       return 0; // Only I is foldable in user.
1398:     }
1399:   }
1400: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1401-1440 / 第 1401-1440 行
```cpp
1401:   // Type legalization (via getNumberOfParts) can't handle structs
1402:   if (TLI->getValueType(DL, Src, true) == MVT::Other)
1403:     return BaseT::getMemoryOpCost(Opcode, Src, Alignment, AddressSpace,
1404:                                   CostKind);
1405: 
1406:   // FP128 is a legal type but kept in a register pair on older CPUs.
1407:   if (Src->isFP128Ty() && !ST->hasVectorEnhancements1())
1408:     return 2;
1409: 
1410:   unsigned NumOps =
1411:     (Src->isVectorTy() ? getNumVectorRegs(Src) : getNumberOfParts(Src));
1412: 
1413:   // Store/Load reversed saves one instruction.
1414:   if (((!Src->isVectorTy() && NumOps == 1) || ST->hasVectorEnhancements2()) &&
1415:       I != nullptr) {
1416:     if (Opcode == Instruction::Load && I->hasOneUse()) {
1417:       const Instruction *LdUser = cast<Instruction>(*I->user_begin());
1418:       // In case of load -> bswap -> store, return normal cost for the load.
1419:       if (isBswapIntrinsicCall(LdUser) &&
1420:           (!LdUser->hasOneUse() || !isa<StoreInst>(*LdUser->user_begin())))
1421:         return 0;
1422:     }
1423:     else if (const StoreInst *SI = dyn_cast<StoreInst>(I)) {
1424:       const Value *StoredVal = SI->getValueOperand();
1425:       if (StoredVal->hasOneUse() && isBswapIntrinsicCall(StoredVal))
1426:         return 0;
1427:     }
1428:   }
1429: 
1430:   return  NumOps;
1431: }
1432: 
1433: // The generic implementation of getInterleavedMemoryOpCost() is based on
1434: // adding costs of the memory operations plus all the extracts and inserts
1435: // needed for using / defining the vector operands. The SystemZ version does
1436: // roughly the same but bases the computations on vector permutations
1437: // instead.
1438: InstructionCost SystemZTTIImpl::getInterleavedMemoryOpCost(
1439:     unsigned Opcode, Type *VecTy, unsigned Factor, ArrayRef<unsigned> Indices,
1440:     Align Alignment, unsigned AddressSpace, TTI::TargetCostKind CostKind,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1441-1480 / 第 1441-1480 行
```cpp
1441:     bool UseMaskForCond, bool UseMaskForGaps) const {
1442:   if (UseMaskForCond || UseMaskForGaps)
1443:     return BaseT::getInterleavedMemoryOpCost(Opcode, VecTy, Factor, Indices,
1444:                                              Alignment, AddressSpace, CostKind,
1445:                                              UseMaskForCond, UseMaskForGaps);
1446:   assert(isa<VectorType>(VecTy) &&
1447:          "Expect a vector type for interleaved memory op");
1448: 
1449:   unsigned NumElts = cast<FixedVectorType>(VecTy)->getNumElements();
1450:   assert(Factor > 1 && NumElts % Factor == 0 && "Invalid interleave factor");
1451:   unsigned VF = NumElts / Factor;
1452:   unsigned NumEltsPerVecReg = (128U / getScalarSizeInBits(VecTy));
1453:   unsigned NumVectorMemOps = getNumVectorRegs(VecTy);
1454:   unsigned NumPermutes = 0;
1455: 
1456:   if (Opcode == Instruction::Load) {
1457:     // Loading interleave groups may have gaps, which may mean fewer
1458:     // loads. Find out how many vectors will be loaded in total, and in how
1459:     // many of them each value will be in.
1460:     BitVector UsedInsts(NumVectorMemOps, false);
1461:     std::vector<BitVector> ValueVecs(Factor, BitVector(NumVectorMemOps, false));
1462:     for (unsigned Index : Indices)
1463:       for (unsigned Elt = 0; Elt < VF; ++Elt) {
1464:         unsigned Vec = (Index + Elt * Factor) / NumEltsPerVecReg;
1465:         UsedInsts.set(Vec);
1466:         ValueVecs[Index].set(Vec);
1467:       }
1468:     NumVectorMemOps = UsedInsts.count();
1469: 
1470:     for (unsigned Index : Indices) {
1471:       // Estimate that each loaded source vector containing this Index
1472:       // requires one operation, except that vperm can handle two input
1473:       // registers first time for each dst vector.
1474:       unsigned NumSrcVecs = ValueVecs[Index].count();
1475:       unsigned NumDstVecs = divideCeil(VF * getScalarSizeInBits(VecTy), 128U);
1476:       assert (NumSrcVecs >= NumDstVecs && "Expected at least as many sources");
1477:       NumPermutes += std::max(1U, NumSrcVecs - NumDstVecs);
1478:     }
1479:   } else {
1480:     // Estimate the permutes for each stored vector as the smaller of the
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1481-1520 / 第 1481-1520 行
```cpp
1481:     // number of elements and the number of source vectors. Subtract one per
1482:     // dst vector for vperm (S.A.).
1483:     unsigned NumSrcVecs = std::min(NumEltsPerVecReg, Factor);
1484:     unsigned NumDstVecs = NumVectorMemOps;
1485:     NumPermutes += (NumDstVecs * NumSrcVecs) - NumDstVecs;
1486:   }
1487: 
1488:   // Cost of load/store operations and the permutations needed.
1489:   return NumVectorMemOps + NumPermutes;
1490: }
1491: 
1492: InstructionCost getIntAddReductionCost(unsigned NumVec, unsigned ScalarBits) {
1493:   InstructionCost Cost = 0;
1494:   // Binary Tree of N/2 + N/4 + ... operations yields N - 1 operations total.
1495:   Cost += NumVec - 1;
1496:   // For integer adds, VSUM creates shorter reductions on the final vector.
1497:   Cost += (ScalarBits < 32) ? 3 : 2;
1498:   return Cost;
1499: }
1500: 
1501: InstructionCost getFastReductionCost(unsigned NumVec, unsigned NumElems,
1502:                                      unsigned ScalarBits) {
1503:   unsigned NumEltsPerVecReg = (SystemZ::VectorBits / ScalarBits);
1504:   InstructionCost Cost = 0;
1505:   // Binary Tree of N/2 + N/4 + ... operations yields N - 1 operations total.
1506:   Cost += NumVec - 1;
1507:   // For each shuffle / arithmetic layer, we need 2 instructions, and we need
1508:   // log2(Elements in Last Vector) layers.
1509:   Cost += 2 * Log2_32_Ceil(std::min(NumElems, NumEltsPerVecReg));
1510:   return Cost;
1511: }
1512: 
1513: inline bool customCostReductions(unsigned Opcode) {
1514:   return Opcode == Instruction::FAdd || Opcode == Instruction::FMul ||
1515:          Opcode == Instruction::Add || Opcode == Instruction::Mul;
1516: }
1517: 
1518: InstructionCost
1519: SystemZTTIImpl::getArithmeticReductionCost(unsigned Opcode, VectorType *Ty,
1520:                                            std::optional<FastMathFlags> FMF,
```
- **EN**: The range implements or declares functions including `getIntAddReductionCost`, `getFastReductionCost`, `customCostReductions`.
- **CN**: 这一段实现或声明了 `getIntAddReductionCost`, `getFastReductionCost`, `customCostReductions` 等函数。

### Lines 1521-1560 / 第 1521-1560 行
```cpp
1521:                                            TTI::TargetCostKind CostKind) const {
1522:   unsigned ScalarBits = Ty->getScalarSizeInBits();
1523:   // The following is only for subtargets with vector math, non-ordered
1524:   // reductions, and reasonable scalar sizes for int and fp add/mul.
1525:   if (customCostReductions(Opcode) && ST->hasVector() &&
1526:       !TTI::requiresOrderedReduction(FMF) &&
1527:       ScalarBits <= SystemZ::VectorBits) {
1528:     unsigned NumVectors = getNumVectorRegs(Ty);
1529:     unsigned NumElems = ((FixedVectorType *)Ty)->getNumElements();
1530:     // Integer Add is using custom code gen, that needs to be accounted for.
1531:     if (Opcode == Instruction::Add)
1532:       return getIntAddReductionCost(NumVectors, ScalarBits);
1533:     // The base cost is the same across all other arithmetic instructions
1534:     InstructionCost Cost =
1535:         getFastReductionCost(NumVectors, NumElems, ScalarBits);
1536:     // But we need to account for the final op involving the scalar operand.
1537:     if ((Opcode == Instruction::FAdd) || (Opcode == Instruction::FMul))
1538:       Cost += 1;
1539:     return Cost;
1540:   }
1541:   // otherwise, fall back to the standard implementation
1542:   return BaseT::getArithmeticReductionCost(Opcode, Ty, FMF, CostKind);
1543: }
1544: 
1545: InstructionCost
1546: SystemZTTIImpl::getMinMaxReductionCost(Intrinsic::ID IID, VectorType *Ty,
1547:                                        FastMathFlags FMF,
1548:                                        TTI::TargetCostKind CostKind) const {
1549:   // Return custom costs only on subtargets with vector enhancements.
1550:   if (ST->hasVectorEnhancements1()) {
1551:     unsigned NumVectors = getNumVectorRegs(Ty);
1552:     unsigned NumElems = ((FixedVectorType *)Ty)->getNumElements();
1553:     unsigned ScalarBits = Ty->getScalarSizeInBits();
1554:     InstructionCost Cost = 0;
1555:     // Binary Tree of N/2 + N/4 + ... operations yields N - 1 operations total.
1556:     Cost += NumVectors - 1;
1557:     // For the final vector, we need shuffle + min/max operations, and
1558:     // we need #Elements - 1 of them.
1559:     Cost += 2 * (std::min(NumElems, SystemZ::VectorBits / ScalarBits) - 1);
1560:     return Cost;
```
- **EN**: The range implements or declares functions including `SystemZTTIImpl::getMinMaxReductionCost`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTTIImpl::getMinMaxReductionCost` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1561-1600 / 第 1561-1600 行
```cpp
1561:   }
1562:   // For other targets, fall back to the standard implementation
1563:   return BaseT::getMinMaxReductionCost(IID, Ty, FMF, CostKind);
1564: }
1565: 
1566: static int
1567: getVectorIntrinsicInstrCost(Intrinsic::ID ID, Type *RetTy,
1568:                             const SmallVectorImpl<Type *> &ParamTys) {
1569:   if (RetTy->isVectorTy() && ID == Intrinsic::bswap)
1570:     return getNumVectorRegs(RetTy); // VPERM
1571: 
1572:   return -1;
1573: }
1574: 
1575: InstructionCost
1576: SystemZTTIImpl::getIntrinsicInstrCost(const IntrinsicCostAttributes &ICA,
1577:                                       TTI::TargetCostKind CostKind) const {
1578:   InstructionCost Cost = getVectorIntrinsicInstrCost(
1579:       ICA.getID(), ICA.getReturnType(), ICA.getArgTypes());
1580:   if (Cost != -1)
1581:     return Cost;
1582:   return BaseT::getIntrinsicInstrCost(ICA, CostKind);
1583: }
1584: 
1585: bool SystemZTTIImpl::shouldExpandReduction(const IntrinsicInst *II) const {
1586:   // Always expand on Subtargets without vector instructions.
1587:   if (!ST->hasVector())
1588:     return true;
1589: 
1590:   // Whether or not to expand is a per-intrinsic decision.
1591:   switch (II->getIntrinsicID()) {
1592:   default:
1593:     return true;
1594:   // Do not expand vector.reduce.add...
1595:   case Intrinsic::vector_reduce_add:
1596:     auto *VType = cast<FixedVectorType>(II->getOperand(0)->getType());
1597:     // ...unless the scalar size is i64 or larger,
1598:     // or the operand vector is not full, since the
1599:     // performance benefit is dubious in those cases.
1600:     return VType->getScalarSizeInBits() >= 64 ||
```
- **EN**: The range implements or declares functions including `getVectorIntrinsicInstrCost`, `SystemZTTIImpl::getIntrinsicInstrCost`, `SystemZTTIImpl::shouldExpandReduction`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getVectorIntrinsicInstrCost`, `SystemZTTIImpl::getIntrinsicInstrCost`, `SystemZTTIImpl::shouldExpandReduction` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1601-1603 / 第 1601-1603 行
```cpp
1601:            VType->getPrimitiveSizeInBits() < SystemZ::VectorBits;
1602:   }
1603: }
```
- **EN**: This span continues the file's main responsibility: this file provides target-specific optimization cost modelling for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `SystemZTargetTransformInfo.h`
- `llvm/Analysis/TargetTransformInfo.h`
- `llvm/CodeGen/BasicTTIImpl.h`
- `llvm/CodeGen/TargetLowering.h`
- `llvm/IR/DerivedTypes.h`
- `llvm/IR/InstIterator.h`
- `llvm/IR/IntrinsicInst.h`
- `llvm/IR/Intrinsics.h`
- `llvm/Support/Debug.h`
- `llvm/Support/InstructionCost.h`
- `llvm/Support/MathExtras.h`
