# SystemZTDC.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZTDC.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZTDC.cpp - Utilize Test Data Class instruction --------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass looks for instructions that can be replaced by a Test Data Class
  10: // instruction, and replaces them when profitable.
  11: //
  12: // Roughly, the following rules are recognized:
  13: //
  14: // 1: fcmp pred X, 0 -> tdc X, mask
  15: // 2: fcmp pred X, +-inf -> tdc X, mask
  16: // 3: fcmp pred X, +-minnorm -> tdc X, mask
  17: // 4: tdc (fabs X), mask -> tdc X, newmask
  18: // 5: icmp slt (bitcast float X to int), 0 -> tdc X, mask [ie. signbit]
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 19-36 / 第 19-36 行
```cpp
  19: // 6: icmp sgt (bitcast float X to int), -1 -> tdc X, mask
  20: // 7: icmp ne/eq (call @llvm.s390.tdc.*(X, mask)) -> tdc X, mask/~mask
  21: // 8: and i1 (tdc X, M1), (tdc X, M2) -> tdc X, (M1 & M2)
  22: // 9: or i1 (tdc X, M1), (tdc X, M2) -> tdc X, (M1 | M2)
  23: // 10: xor i1 (tdc X, M1), (tdc X, M2) -> tdc X, (M1 ^ M2)
  24: //
  25: // The pass works in 4 steps:
  26: //
  27: // 1. All fcmp and icmp instructions in a function are checked for a match
  28: //    with rules 1-3 and 5-7.  Their TDC equivalents are stored in
  29: //    the ConvertedInsts mapping.  If the operand of a fcmp instruction is
  30: //    a fabs, it's also folded according to rule 4.
  31: // 2. All and/or/xor i1 instructions whose both operands have been already
  32: //    mapped are mapped according to rules 8-10.  LogicOpsWorklist is used
  33: //    as a queue of instructions to check.
  34: // 3. All mapped instructions that are considered worthy of conversion (ie.
  35: //    replacing them will actually simplify the final code) are replaced
  36: //    with a call to the s390.tdc intrinsic.
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 37-54 / 第 37-54 行
```cpp
  37: // 4. All intermediate results of replaced instructions are removed if unused.
  38: //
  39: // Instructions that match rules 1-3 are considered unworthy of conversion
  40: // on their own (since a comparison instruction is superior), but are mapped
  41: // in the hopes of folding the result using rules 4 and 8-10 (likely removing
  42: // the original comparison in the process).
  43: //
  44: //===----------------------------------------------------------------------===//
  45: 
  46: #include "SystemZ.h"
  47: #include "SystemZSubtarget.h"
  48: #include "llvm/ADT/MapVector.h"
  49: #include "llvm/CodeGen/TargetPassConfig.h"
  50: #include "llvm/IR/Constants.h"
  51: #include "llvm/IR/IRBuilder.h"
  52: #include "llvm/IR/InstIterator.h"
  53: #include "llvm/IR/Instructions.h"
  54: #include "llvm/IR/IntrinsicsS390.h"
```
- **EN**: It imports dependencies such as `SystemZ.h`, `SystemZSubtarget.h`, `MapVector.h`, `TargetPassConfig.h`, `Constants.h`, `IRBuilder.h` that expose the LLVM and target interfaces used in later logic. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `SystemZ.h`, `SystemZSubtarget.h`, `MapVector.h`, `TargetPassConfig.h`, `Constants.h`, `IRBuilder.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55: #include "llvm/IR/LegacyPassManager.h"
  56: #include "llvm/IR/Module.h"
  57: #include "llvm/Target/TargetMachine.h"
  58: #include <set>
  59: 
  60: using namespace llvm;
  61: 
  62: namespace {
  63: 
  64: class SystemZTDCPass : public FunctionPass {
  65: public:
  66:   static char ID;
  67:   SystemZTDCPass() : FunctionPass(ID) {}
  68: 
  69:   bool runOnFunction(Function &F) override;
  70: 
  71:   void getAnalysisUsage(AnalysisUsage &AU) const override {
  72:     AU.addRequired<TargetPassConfig>();
```
- **EN**: It imports dependencies such as `LegacyPassManager.h`, `Module.h`, `TargetMachine.h`, `set` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `SystemZTDCPass`. The range implements or declares functions including `SystemZTDCPass`.
- **CN**: 它引入了 `LegacyPassManager.h`, `Module.h`, `TargetMachine.h`, `set` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `SystemZTDCPass` 等 TableGen 记录。 这一段实现或声明了 `SystemZTDCPass` 等函数。

### Lines 73-90 / 第 73-90 行
```cpp
  73:  }
  74: 
  75: private:
  76:   // Maps seen instructions that can be mapped to a TDC, values are
  77:   // (TDC operand, TDC mask, worthy flag) triples.
  78:   MapVector<Instruction *, std::tuple<Value *, int, bool>> ConvertedInsts;
  79:   // The queue of and/or/xor i1 instructions to be potentially folded.
  80:   std::vector<BinaryOperator *> LogicOpsWorklist;
  81:   // Instructions matched while folding, to be removed at the end if unused.
  82:   std::set<Instruction *> PossibleJunk;
  83: 
  84:   // Tries to convert a fcmp instruction.
  85:   void convertFCmp(CmpInst &I);
  86: 
  87:   // Tries to convert an icmp instruction.
  88:   void convertICmp(CmpInst &I);
  89: 
  90:   // Tries to convert an i1 and/or/xor instruction, whose both operands
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   // have been already converted.
  92:   void convertLogicOp(BinaryOperator &I);
  93: 
  94:   // Marks an instruction as converted - adds it to ConvertedInsts and adds
  95:   // any and/or/xor i1 users to the queue.
  96:   void converted(Instruction *I, Value *V, int Mask, bool Worthy) {
  97:     ConvertedInsts[I] = std::make_tuple(V, Mask, Worthy);
  98:     auto &M = *I->getFunction()->getParent();
  99:     auto &Ctx = M.getContext();
 100:     for (auto *U : I->users()) {
 101:       auto *LI = dyn_cast<BinaryOperator>(U);
 102:       if (LI && LI->getType() == Type::getInt1Ty(Ctx) &&
 103:           (LI->getOpcode() == Instruction::And ||
 104:            LI->getOpcode() == Instruction::Or ||
 105:            LI->getOpcode() == Instruction::Xor)) {
 106:         LogicOpsWorklist.push_back(LI);
 107:       }
 108:     }
```
- **EN**: The range implements or declares functions including `converted`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `converted` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   }
 110: };
 111: 
 112: } // end anonymous namespace
 113: 
 114: char SystemZTDCPass::ID = 0;
 115: INITIALIZE_PASS(SystemZTDCPass, "systemz-tdc",
 116:                 "SystemZ Test Data Class optimization", false, false)
 117: 
 118: FunctionPass *llvm::createSystemZTDCPass() {
 119:   return new SystemZTDCPass();
 120: }
 121: 
 122: void SystemZTDCPass::convertFCmp(CmpInst &I) {
 123:   Value *Op0 = I.getOperand(0);
 124:   auto *Const = dyn_cast<ConstantFP>(I.getOperand(1));
 125:   auto Pred = I.getPredicate();
 126:   // Only scalar comparisons with consts are interesting.
```
- **EN**: The range implements or declares functions including `SystemZTDCPass::convertFCmp`.
- **CN**: 这一段实现或声明了 `SystemZTDCPass::convertFCmp` 等函数。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   if (!Const || !Const->getType()->isFloatingPointTy())
 128:     return;
 129:   // Compute the smallest normal number (and its negation).
 130:   auto &Sem = Op0->getType()->getFltSemantics();
 131:   APFloat Smallest = APFloat::getSmallestNormalized(Sem);
 132:   APFloat NegSmallest = Smallest;
 133:   NegSmallest.changeSign();
 134:   // Check if Const is one of our recognized consts.
 135:   int WhichConst;
 136:   if (Const->isZero()) {
 137:     // All comparisons with 0 can be converted.
 138:     WhichConst = 0;
 139:   } else if (Const->isInfinity()) {
 140:     // Likewise for infinities.
 141:     WhichConst = Const->isNegative() ? 2 : 1;
 142:   } else if (Const->isExactlyValue(Smallest)) {
 143:     // For Smallest, we cannot do EQ separately from GT.
 144:     if ((Pred & CmpInst::FCMP_OGE) != CmpInst::FCMP_OGE &&
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:         (Pred & CmpInst::FCMP_OGE) != 0)
 146:       return;
 147:     WhichConst = 3;
 148:   } else if (Const->isExactlyValue(NegSmallest)) {
 149:     // Likewise for NegSmallest, we cannot do EQ separately from LT.
 150:     if ((Pred & CmpInst::FCMP_OLE) != CmpInst::FCMP_OLE &&
 151:         (Pred & CmpInst::FCMP_OLE) != 0)
 152:       return;
 153:     WhichConst = 4;
 154:   } else {
 155:     // Not one of our special constants.
 156:     return;
 157:   }
 158:   // Partial masks to use for EQ, GT, LT, UN comparisons, respectively.
 159:   static const int Masks[][4] = {
 160:     { // 0
 161:       SystemZ::TDCMASK_ZERO,              // eq
 162:       SystemZ::TDCMASK_POSITIVE,          // gt
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163:       SystemZ::TDCMASK_NEGATIVE,          // lt
 164:       SystemZ::TDCMASK_NAN,               // un
 165:     },
 166:     { // inf
 167:       SystemZ::TDCMASK_INFINITY_PLUS,     // eq
 168:       0,                                  // gt
 169:       (SystemZ::TDCMASK_ZERO |
 170:        SystemZ::TDCMASK_NEGATIVE |
 171:        SystemZ::TDCMASK_NORMAL_PLUS |
 172:        SystemZ::TDCMASK_SUBNORMAL_PLUS),  // lt
 173:       SystemZ::TDCMASK_NAN,               // un
 174:     },
 175:     { // -inf
 176:       SystemZ::TDCMASK_INFINITY_MINUS,    // eq
 177:       (SystemZ::TDCMASK_ZERO |
 178:        SystemZ::TDCMASK_POSITIVE |
 179:        SystemZ::TDCMASK_NORMAL_MINUS |
 180:        SystemZ::TDCMASK_SUBNORMAL_MINUS), // gt
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 181-198 / 第 181-198 行
```cpp
 181:       0,                                  // lt
 182:       SystemZ::TDCMASK_NAN,               // un
 183:     },
 184:     { // minnorm
 185:       0,                                  // eq (unsupported)
 186:       (SystemZ::TDCMASK_NORMAL_PLUS |
 187:        SystemZ::TDCMASK_INFINITY_PLUS),   // gt (actually ge)
 188:       (SystemZ::TDCMASK_ZERO |
 189:        SystemZ::TDCMASK_NEGATIVE |
 190:        SystemZ::TDCMASK_SUBNORMAL_PLUS),  // lt
 191:       SystemZ::TDCMASK_NAN,               // un
 192:     },
 193:     { // -minnorm
 194:       0,                                  // eq (unsupported)
 195:       (SystemZ::TDCMASK_ZERO |
 196:        SystemZ::TDCMASK_POSITIVE |
 197:        SystemZ::TDCMASK_SUBNORMAL_MINUS), // gt
 198:       (SystemZ::TDCMASK_NORMAL_MINUS |
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 199-216 / 第 199-216 行
```cpp
 199:        SystemZ::TDCMASK_INFINITY_MINUS),  // lt (actually le)
 200:       SystemZ::TDCMASK_NAN,               // un
 201:     }
 202:   };
 203:   // Construct the mask as a combination of the partial masks.
 204:   int Mask = 0;
 205:   if (Pred & CmpInst::FCMP_OEQ)
 206:     Mask |= Masks[WhichConst][0];
 207:   if (Pred & CmpInst::FCMP_OGT)
 208:     Mask |= Masks[WhichConst][1];
 209:   if (Pred & CmpInst::FCMP_OLT)
 210:     Mask |= Masks[WhichConst][2];
 211:   if (Pred & CmpInst::FCMP_UNO)
 212:     Mask |= Masks[WhichConst][3];
 213:   // A lone fcmp is unworthy of tdc conversion on its own, but may become
 214:   // worthy if combined with fabs.
 215:   bool Worthy = false;
 216:   if (CallInst *CI = dyn_cast<CallInst>(Op0)) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 217-234 / 第 217-234 行
```cpp
 217:     Function *F = CI->getCalledFunction();
 218:     if (F && F->getIntrinsicID() == Intrinsic::fabs) {
 219:       // Fold with fabs - adjust the mask appropriately.
 220:       Mask &= SystemZ::TDCMASK_PLUS;
 221:       Mask |= Mask >> 1;
 222:       Op0 = CI->getArgOperand(0);
 223:       // A combination of fcmp with fabs is a win, unless the constant
 224:       // involved is 0 (which is handled by later passes).
 225:       Worthy = WhichConst != 0;
 226:       PossibleJunk.insert(CI);
 227:     }
 228:   }
 229:   converted(&I, Op0, Mask, Worthy);
 230: }
 231: 
 232: void SystemZTDCPass::convertICmp(CmpInst &I) {
 233:   Value *Op0 = I.getOperand(0);
 234:   auto *Const = dyn_cast<ConstantInt>(I.getOperand(1));
```
- **EN**: The range implements or declares functions including `SystemZTDCPass::convertICmp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTDCPass::convertICmp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235:   auto Pred = I.getPredicate();
 236:   // All our icmp rules involve comparisons with consts.
 237:   if (!Const)
 238:     return;
 239:   if (auto *Cast = dyn_cast<BitCastInst>(Op0)) {
 240:     // Check for icmp+bitcast used for signbit.
 241:     if (!Cast->getSrcTy()->isFloatTy() &&
 242:         !Cast->getSrcTy()->isDoubleTy() &&
 243:         !Cast->getSrcTy()->isFP128Ty())
 244:       return;
 245:     Value *V = Cast->getOperand(0);
 246:     int Mask;
 247:     if (Pred == CmpInst::ICMP_SLT && Const->isZero()) {
 248:       // icmp slt (bitcast X), 0 - set if sign bit true
 249:       Mask = SystemZ::TDCMASK_MINUS;
 250:     } else if (Pred == CmpInst::ICMP_SGT && Const->isMinusOne()) {
 251:       // icmp sgt (bitcast X), -1 - set if sign bit false
 252:       Mask = SystemZ::TDCMASK_PLUS;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-270 / 第 253-270 行
```cpp
 253:     } else {
 254:       // Not a sign bit check.
 255:       return;
 256:     }
 257:     PossibleJunk.insert(Cast);
 258:     converted(&I, V, Mask, true);
 259:   } else if (auto *CI = dyn_cast<CallInst>(Op0)) {
 260:     // Check if this is a pre-existing call of our tdc intrinsic.
 261:     Function *F = CI->getCalledFunction();
 262:     if (!F || F->getIntrinsicID() != Intrinsic::s390_tdc)
 263:       return;
 264:     if (!Const->isZero())
 265:       return;
 266:     Value *V = CI->getArgOperand(0);
 267:     auto *MaskC = dyn_cast<ConstantInt>(CI->getArgOperand(1));
 268:     // Bail if the mask is not a constant.
 269:     if (!MaskC)
 270:       return;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 271-288 / 第 271-288 行
```cpp
 271:     int Mask = MaskC->getZExtValue();
 272:     Mask &= SystemZ::TDCMASK_ALL;
 273:     if (Pred == CmpInst::ICMP_NE) {
 274:       // icmp ne (call llvm.s390.tdc(...)), 0 -> simple TDC
 275:     } else if (Pred == CmpInst::ICMP_EQ) {
 276:       // icmp eq (call llvm.s390.tdc(...)), 0 -> TDC with inverted mask
 277:       Mask ^= SystemZ::TDCMASK_ALL;
 278:     } else {
 279:       // An unknown comparison - ignore.
 280:       return;
 281:     }
 282:     PossibleJunk.insert(CI);
 283:     converted(&I, V, Mask, false);
 284:   }
 285: }
 286: 
 287: void SystemZTDCPass::convertLogicOp(BinaryOperator &I) {
 288:   Value *Op0, *Op1;
```
- **EN**: The range implements or declares functions including `SystemZTDCPass::convertLogicOp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTDCPass::convertLogicOp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 289-306 / 第 289-306 行
```cpp
 289:   int Mask0, Mask1;
 290:   bool Worthy0, Worthy1;
 291:   std::tie(Op0, Mask0, Worthy0) = ConvertedInsts[cast<Instruction>(I.getOperand(0))];
 292:   std::tie(Op1, Mask1, Worthy1) = ConvertedInsts[cast<Instruction>(I.getOperand(1))];
 293:   if (Op0 != Op1)
 294:     return;
 295:   int Mask;
 296:   switch (I.getOpcode()) {
 297:     case Instruction::And:
 298:       Mask = Mask0 & Mask1;
 299:       break;
 300:     case Instruction::Or:
 301:       Mask = Mask0 | Mask1;
 302:       break;
 303:     case Instruction::Xor:
 304:       Mask = Mask0 ^ Mask1;
 305:       break;
 306:     default:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 307-324 / 第 307-324 行
```cpp
 307:       llvm_unreachable("Unknown op in convertLogicOp");
 308:   }
 309:   converted(&I, Op0, Mask, true);
 310: }
 311: 
 312: bool SystemZTDCPass::runOnFunction(Function &F) {
 313:   auto &TPC = getAnalysis<TargetPassConfig>();
 314:   if (TPC.getTM<TargetMachine>()
 315:           .getSubtarget<SystemZSubtarget>(F)
 316:           .hasSoftFloat())
 317:     return false;
 318: 
 319:   ConvertedInsts.clear();
 320:   LogicOpsWorklist.clear();
 321:   PossibleJunk.clear();
 322: 
 323:   // Look for icmp+fcmp instructions.
 324:   for (auto &I : instructions(F)) {
```
- **EN**: The range implements or declares functions including `SystemZTDCPass::runOnFunction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTDCPass::runOnFunction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 325-342 / 第 325-342 行
```cpp
 325:     if (I.getOpcode() == Instruction::FCmp)
 326:       convertFCmp(cast<CmpInst>(I));
 327:     else if (I.getOpcode() == Instruction::ICmp)
 328:       convertICmp(cast<CmpInst>(I));
 329:   }
 330: 
 331:   // If none found, bail already.
 332:   if (ConvertedInsts.empty())
 333:     return false;
 334: 
 335:   // Process the queue of logic instructions.
 336:   while (!LogicOpsWorklist.empty()) {
 337:     BinaryOperator *Op = LogicOpsWorklist.back();
 338:     LogicOpsWorklist.pop_back();
 339:     // If both operands mapped, and the instruction itself not yet mapped,
 340:     // convert it.
 341:     if (ConvertedInsts.count(dyn_cast<Instruction>(Op->getOperand(0))) &&
 342:         ConvertedInsts.count(dyn_cast<Instruction>(Op->getOperand(1))) &&
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 343-360 / 第 343-360 行
```cpp
 343:         !ConvertedInsts.count(Op))
 344:       convertLogicOp(*Op);
 345:   }
 346: 
 347:   // Time to actually replace the instructions.  Do it in the reverse order
 348:   // of finding them, since there's a good chance the earlier ones will be
 349:   // unused (due to being folded into later ones).
 350:   Module &M = *F.getParent();
 351:   auto &Ctx = M.getContext();
 352:   Value *Zero32 = ConstantInt::get(Type::getInt32Ty(Ctx), 0);
 353:   bool MadeChange = false;
 354:   for (auto &It : reverse(ConvertedInsts)) {
 355:     Instruction *I = It.first;
 356:     Value *V;
 357:     int Mask;
 358:     bool Worthy;
 359:     std::tie(V, Mask, Worthy) = It.second;
 360:     if (!I->user_empty()) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 361-378 / 第 361-378 行
```cpp
 361:       // If used and unworthy of conversion, skip it.
 362:       if (!Worthy)
 363:         continue;
 364:       // Call the intrinsic, compare result with 0.
 365:       IRBuilder<> IRB(I);
 366:       Value *MaskVal = ConstantInt::get(Type::getInt64Ty(Ctx), Mask);
 367:       Instruction *TDC =
 368:           IRB.CreateIntrinsic(Intrinsic::s390_tdc, V->getType(), {V, MaskVal});
 369:       Value *ICmp = IRB.CreateICmp(CmpInst::ICMP_NE, TDC, Zero32);
 370:       I->replaceAllUsesWith(ICmp);
 371:     }
 372:     // If unused, or used and converted, remove it.
 373:     I->eraseFromParent();
 374:     MadeChange = true;
 375:   }
 376: 
 377:   if (!MadeChange)
 378:     return false;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 379-387 / 第 379-387 行
```cpp
 379: 
 380:   // We've actually done something - now clear misc accumulated junk (fabs,
 381:   // bitcast).
 382:   for (auto *I : PossibleJunk)
 383:     if (I->user_empty())
 384:       I->eraseFromParent();
 385: 
 386:   return true;
 387: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `SystemZ.h`
- `SystemZSubtarget.h`
- `llvm/ADT/MapVector.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `llvm/IR/Constants.h`
- `llvm/IR/IRBuilder.h`
- `llvm/IR/InstIterator.h`
- `llvm/IR/Instructions.h`
- `llvm/IR/IntrinsicsS390.h`
- `llvm/IR/LegacyPassManager.h`
- `llvm/IR/Module.h`
- `llvm/Target/TargetMachine.h`
- `set`
