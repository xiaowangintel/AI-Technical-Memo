# BPFCheckAndAdjustIR.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFCheckAndAdjustIR.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Provides target-specific support code, declarations, or helper routines used by the backend.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===------------ BPFCheckAndAdjustIR.cpp - Check and Adjust IR -----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // Check IR and adjust IR for verifier friendly codes.
  10: // The following are done for IR checking:
  11: //   - no relocation globals in PHI node.
  12: // The following are done for IR adjustment:
  13: //   - remove __builtin_bpf_passthrough builtins. Target independent IR
  14: //     optimizations are done and those builtins can be removed.
  15: //   - remove llvm.bpf.getelementptr.and.load builtins.
  16: //   - remove llvm.bpf.getelementptr.and.store builtins.
  17: //   - for loads and stores with base addresses from non-zero address space
  18: //     cast base address to zero address space (support for BPF address spaces).
  19: //
  20: //===----------------------------------------------------------------------===//
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。

### Lines 21-40

```cpp
  21: 
  22: #include "BPF.h"
  23: #include "BPFCORE.h"
  24: #include "llvm/Analysis/LoopInfo.h"
  25: #include "llvm/IR/GlobalVariable.h"
  26: #include "llvm/IR/IRBuilder.h"
  27: #include "llvm/IR/Instruction.h"
  28: #include "llvm/IR/Instructions.h"
  29: #include "llvm/IR/IntrinsicInst.h"
  30: #include "llvm/IR/IntrinsicsBPF.h"
  31: #include "llvm/IR/Module.h"
  32: #include "llvm/IR/Type.h"
  33: #include "llvm/IR/Value.h"
  34: #include "llvm/Pass.h"
  35: #include "llvm/Transforms/Utils/BasicBlockUtils.h"
  36: 
  37: #define DEBUG_TYPE "bpf-check-and-opt-ir"
  38: 
  39: using namespace llvm;
  40: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 41-60

```cpp
  41: namespace {
  42: 
  43: class BPFCheckAndAdjustIR final : public ModulePass {
  44:   bool runOnModule(Module &F) override;
  45: 
  46: public:
  47:   static char ID;
  48:   BPFCheckAndAdjustIR() : ModulePass(ID) {}
  49:   void getAnalysisUsage(AnalysisUsage &AU) const override;
  50: 
  51: private:
  52:   void checkIR(Module &M);
  53:   bool adjustIR(Module &M);
  54:   bool removePassThroughBuiltin(Module &M);
  55:   bool removeCompareBuiltin(Module &M);
  56:   bool sinkMinMax(Module &M);
  57:   bool removeGEPBuiltins(Module &M);
  58:   bool insertASpaceCasts(Module &M);
  59: };
  60: } // End anonymous namespace
```

- EN: This chunk introduces interfaces or data structures such as BPFCheckAndAdjustIR, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 BPFCheckAndAdjustIR 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 61-80

```cpp
  61: 
  62: char BPFCheckAndAdjustIR::ID = 0;
  63: INITIALIZE_PASS(BPFCheckAndAdjustIR, DEBUG_TYPE, "BPF Check And Adjust IR",
  64:                 false, false)
  65: 
  66: ModulePass *llvm::createBPFCheckAndAdjustIR() {
  67:   return new BPFCheckAndAdjustIR();
  68: }
  69: 
  70: void BPFCheckAndAdjustIR::checkIR(Module &M) {
  71:   // Ensure relocation global won't appear in PHI node
  72:   // This may happen if the compiler generated the following code:
  73:   //   B1:
  74:   //      g1 = @llvm.skb_buff:0:1...
  75:   //      ...
  76:   //      goto B_COMMON
  77:   //   B2:
  78:   //      g2 = @llvm.skb_buff:0:2...
  79:   //      ...
  80:   //      goto B_COMMON
```

- EN: Function bodies or method definitions such as checkIR contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: checkIR 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 81-100

```cpp
  81:   //   B_COMMON:
  82:   //      g = PHI(g1, g2)
  83:   //      x = load g
  84:   //      ...
  85:   // If anything likes the above "g = PHI(g1, g2)", issue a fatal error.
  86:   for (Function &F : M)
  87:     for (auto &BB : F)
  88:       for (auto &I : BB) {
  89:         PHINode *PN = dyn_cast<PHINode>(&I);
  90:         if (!PN || PN->use_empty())
  91:           continue;
  92:         for (int i = 0, e = PN->getNumIncomingValues(); i < e; ++i) {
  93:           auto *GV = dyn_cast<GlobalVariable>(PN->getIncomingValue(i));
  94:           if (!GV)
  95:             continue;
  96:           if (GV->hasAttribute(BPFCoreSharedInfo::AmaAttr) ||
  97:               GV->hasAttribute(BPFCoreSharedInfo::TypeIdAttr))
  98:             report_fatal_error("relocation global in PHI node");
  99:         }
 100:       }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101: }
 102: 
 103: bool BPFCheckAndAdjustIR::removePassThroughBuiltin(Module &M) {
 104:   // Remove __builtin_bpf_passthrough()'s which are used to prevent
 105:   // certain IR optimizations. Now major IR optimizations are done,
 106:   // remove them.
 107:   bool Changed = false;
 108:   CallInst *ToBeDeleted = nullptr;
 109:   for (Function &F : M)
 110:     for (auto &BB : F)
 111:       for (auto &I : BB) {
 112:         if (ToBeDeleted) {
 113:           ToBeDeleted->eraseFromParent();
 114:           ToBeDeleted = nullptr;
 115:         }
 116: 
 117:         auto *Call = dyn_cast<CallInst>(&I);
 118:         if (!Call)
 119:           continue;
 120:         auto *GV = dyn_cast<GlobalValue>(Call->getCalledOperand());
```

- EN: Function bodies or method definitions such as removePassThroughBuiltin contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: removePassThroughBuiltin 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:         if (!GV)
 122:           continue;
 123:         if (!GV->getName().starts_with("llvm.bpf.passthrough"))
 124:           continue;
 125:         Changed = true;
 126:         Value *Arg = Call->getArgOperand(1);
 127:         Call->replaceAllUsesWith(Arg);
 128:         ToBeDeleted = Call;
 129:       }
 130:   return Changed;
 131: }
 132: 
 133: bool BPFCheckAndAdjustIR::removeCompareBuiltin(Module &M) {
 134:   // Remove __builtin_bpf_compare()'s which are used to prevent
 135:   // certain IR optimizations. Now major IR optimizations are done,
 136:   // remove them.
 137:   bool Changed = false;
 138:   CallInst *ToBeDeleted = nullptr;
 139:   for (Function &F : M)
 140:     for (auto &BB : F)
```

- EN: Function bodies or method definitions such as removeCompareBuiltin contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: removeCompareBuiltin 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141:       for (auto &I : BB) {
 142:         if (ToBeDeleted) {
 143:           ToBeDeleted->eraseFromParent();
 144:           ToBeDeleted = nullptr;
 145:         }
 146: 
 147:         auto *Call = dyn_cast<CallInst>(&I);
 148:         if (!Call)
 149:           continue;
 150:         auto *GV = dyn_cast<GlobalValue>(Call->getCalledOperand());
 151:         if (!GV)
 152:           continue;
 153:         if (!GV->getName().starts_with("llvm.bpf.compare"))
 154:           continue;
 155: 
 156:         Changed = true;
 157:         Value *Arg0 = Call->getArgOperand(0);
 158:         Value *Arg1 = Call->getArgOperand(1);
 159:         Value *Arg2 = Call->getArgOperand(2);
 160: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-180

```cpp
 161:         auto OpVal = cast<ConstantInt>(Arg0)->getValue().getZExtValue();
 162:         CmpInst::Predicate Opcode = (CmpInst::Predicate)OpVal;
 163: 
 164:         auto *ICmp = new ICmpInst(Opcode, Arg1, Arg2);
 165:         ICmp->insertBefore(Call->getIterator());
 166: 
 167:         Call->replaceAllUsesWith(ICmp);
 168:         ToBeDeleted = Call;
 169:       }
 170:   return Changed;
 171: }
 172: 
 173: struct MinMaxSinkInfo {
 174:   ICmpInst *ICmp;
 175:   Value *Other;
 176:   ICmpInst::Predicate Predicate;
 177:   CallInst *MinMax;
 178:   ZExtInst *ZExt;
 179:   SExtInst *SExt;
 180: 
```

- EN: This chunk introduces interfaces or data structures such as MinMaxSinkInfo, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 MinMaxSinkInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 181-200

```cpp
 181:   MinMaxSinkInfo(ICmpInst *ICmp, Value *Other, ICmpInst::Predicate Predicate)
 182:       : ICmp(ICmp), Other(Other), Predicate(Predicate), MinMax(nullptr),
 183:         ZExt(nullptr), SExt(nullptr) {}
 184: };
 185: 
 186: static bool sinkMinMaxInBB(BasicBlock &BB,
 187:                            const std::function<bool(Instruction *)> &Filter) {
 188:   // Check if V is:
 189:   //   (fn %a %b) or (ext (fn %a %b))
 190:   // Where:
 191:   //   ext := sext | zext
 192:   //   fn  := smin | umin | smax | umax
 193:   auto IsMinMaxCall = [=](Value *V, MinMaxSinkInfo &Info) {
 194:     if (auto *ZExt = dyn_cast<ZExtInst>(V)) {
 195:       V = ZExt->getOperand(0);
 196:       Info.ZExt = ZExt;
 197:     } else if (auto *SExt = dyn_cast<SExtInst>(V)) {
 198:       V = SExt->getOperand(0);
 199:       Info.SExt = SExt;
 200:     }
```

- EN: Function bodies or method definitions such as ICmp, sinkMinMaxInBB contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: ICmp, sinkMinMaxInBB 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 201-220

```cpp
 201: 
 202:     auto *Call = dyn_cast<CallInst>(V);
 203:     if (!Call)
 204:       return false;
 205: 
 206:     auto *Called = dyn_cast<Function>(Call->getCalledOperand());
 207:     if (!Called)
 208:       return false;
 209: 
 210:     switch (Called->getIntrinsicID()) {
 211:     case Intrinsic::smin:
 212:     case Intrinsic::umin:
 213:     case Intrinsic::smax:
 214:     case Intrinsic::umax:
 215:       break;
 216:     default:
 217:       return false;
 218:     }
 219: 
 220:     if (!Filter(Call))
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221:       return false;
 222: 
 223:     Info.MinMax = Call;
 224: 
 225:     return true;
 226:   };
 227: 
 228:   auto ZeroOrSignExtend = [](IRBuilder<> &Builder, Value *V,
 229:                              MinMaxSinkInfo &Info) {
 230:     if (Info.SExt) {
 231:       if (Info.SExt->getType() == V->getType())
 232:         return V;
 233:       return Builder.CreateSExt(V, Info.SExt->getType());
 234:     }
 235:     if (Info.ZExt) {
 236:       if (Info.ZExt->getType() == V->getType())
 237:         return V;
 238:       return Builder.CreateZExt(V, Info.ZExt->getType());
 239:     }
 240:     return V;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241:   };
 242: 
 243:   bool Changed = false;
 244:   SmallVector<MinMaxSinkInfo, 2> SinkList;
 245: 
 246:   // Check BB for instructions like:
 247:   //   insn := (icmp %a (fn ...)) | (icmp (fn ...)  %a)
 248:   //
 249:   // Where:
 250:   //   fn := min | max | (sext (min ...)) | (sext (max ...))
 251:   //
 252:   // Put such instructions to SinkList.
 253:   for (Instruction &I : BB) {
 254:     ICmpInst *ICmp = dyn_cast<ICmpInst>(&I);
 255:     if (!ICmp)
 256:       continue;
 257:     if (!ICmp->isRelational())
 258:       continue;
 259:     MinMaxSinkInfo First(ICmp, ICmp->getOperand(1),
 260:                          ICmpInst::getSwappedPredicate(ICmp->getPredicate()));
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 261-280

```cpp
 261:     MinMaxSinkInfo Second(ICmp, ICmp->getOperand(0), ICmp->getPredicate());
 262:     bool FirstMinMax = IsMinMaxCall(ICmp->getOperand(0), First);
 263:     bool SecondMinMax = IsMinMaxCall(ICmp->getOperand(1), Second);
 264:     if (!(FirstMinMax ^ SecondMinMax))
 265:       continue;
 266:     SinkList.push_back(FirstMinMax ? First : Second);
 267:   }
 268: 
 269:   // Iterate SinkList and replace each (icmp ...) with corresponding
 270:   // `x < a && x < b` or similar expression.
 271:   for (auto &Info : SinkList) {
 272:     ICmpInst *ICmp = Info.ICmp;
 273:     CallInst *MinMax = Info.MinMax;
 274:     Intrinsic::ID IID = MinMax->getCalledFunction()->getIntrinsicID();
 275:     ICmpInst::Predicate P = Info.Predicate;
 276:     if (ICmpInst::isSigned(P) && IID != Intrinsic::smin &&
 277:         IID != Intrinsic::smax)
 278:       continue;
 279: 
 280:     IRBuilder<> Builder(ICmp);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 281-300

```cpp
 281:     Value *X = Info.Other;
 282:     Value *A = ZeroOrSignExtend(Builder, MinMax->getArgOperand(0), Info);
 283:     Value *B = ZeroOrSignExtend(Builder, MinMax->getArgOperand(1), Info);
 284:     bool IsMin = IID == Intrinsic::smin || IID == Intrinsic::umin;
 285:     bool IsMax = IID == Intrinsic::smax || IID == Intrinsic::umax;
 286:     bool IsLess = ICmpInst::isLE(P) || ICmpInst::isLT(P);
 287:     bool IsGreater = ICmpInst::isGE(P) || ICmpInst::isGT(P);
 288:     assert(IsMin ^ IsMax);
 289:     assert(IsLess ^ IsGreater);
 290: 
 291:     Value *Replacement;
 292:     Value *LHS = Builder.CreateICmp(P, X, A);
 293:     Value *RHS = Builder.CreateICmp(P, X, B);
 294:     if ((IsLess && IsMin) || (IsGreater && IsMax))
 295:       // x < min(a, b) -> x < a && x < b
 296:       // x > max(a, b) -> x > a && x > b
 297:       Replacement = Builder.CreateLogicalAnd(LHS, RHS);
 298:     else
 299:       // x > min(a, b) -> x > a || x > b
 300:       // x < max(a, b) -> x < a || x < b
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 301-320

```cpp
 301:       Replacement = Builder.CreateLogicalOr(LHS, RHS);
 302: 
 303:     ICmp->replaceAllUsesWith(Replacement);
 304: 
 305:     Instruction *ToRemove[] = {ICmp, Info.ZExt, Info.SExt, MinMax};
 306:     for (Instruction *I : ToRemove)
 307:       if (I && I->use_empty())
 308:         I->eraseFromParent();
 309: 
 310:     Changed = true;
 311:   }
 312: 
 313:   return Changed;
 314: }
 315: 
 316: // Do the following transformation:
 317: //
 318: //   x < min(a, b) -> x < a && x < b
 319: //   x > min(a, b) -> x > a || x > b
 320: //   x < max(a, b) -> x < a || x < b
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 321-340

```cpp
 321: //   x > max(a, b) -> x > a && x > b
 322: //
 323: // Such patterns are introduced by LICM.cpp:hoistMinMax()
 324: // transformation and might lead to BPF verification failures for
 325: // older kernels.
 326: //
 327: // To minimize "collateral" changes only do it for icmp + min/max
 328: // calls when icmp is inside a loop and min/max is outside of that
 329: // loop.
 330: //
 331: // Verification failure happens when:
 332: // - RHS operand of some `icmp LHS, RHS` is replaced by some RHS1;
 333: // - verifier can recognize RHS as a constant scalar in some context;
 334: // - verifier can't recognize RHS1 as a constant scalar in the same
 335: //   context;
 336: //
 337: // The "constant scalar" is not a compile time constant, but a register
 338: // that holds a scalar value known to verifier at some point in time
 339: // during abstract interpretation.
 340: //
```

- EN: This range continues the implementation of the backend component described by BPFCheckAndAdjustIR.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 341-360

```cpp
 341: // See also:
 342: //   https://lore.kernel.org/bpf/20230406164505.1046801-1-yhs@fb.com/
 343: bool BPFCheckAndAdjustIR::sinkMinMax(Module &M) {
 344:   bool Changed = false;
 345: 
 346:   for (Function &F : M) {
 347:     if (F.isDeclaration())
 348:       continue;
 349: 
 350:     LoopInfo &LI = getAnalysis<LoopInfoWrapperPass>(F).getLoopInfo();
 351:     for (Loop *L : LI)
 352:       for (BasicBlock *BB : L->blocks()) {
 353:         // Filter out instructions coming from the same loop
 354:         Loop *BBLoop = LI.getLoopFor(BB);
 355:         auto OtherLoopFilter = [&](Instruction *I) {
 356:           return LI.getLoopFor(I->getParent()) != BBLoop;
 357:         };
 358:         Changed |= sinkMinMaxInBB(*BB, OtherLoopFilter);
 359:       }
 360:   }
```

- EN: Function bodies or method definitions such as sinkMinMax contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: sinkMinMax 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 361-380

```cpp
 361: 
 362:   return Changed;
 363: }
 364: 
 365: void BPFCheckAndAdjustIR::getAnalysisUsage(AnalysisUsage &AU) const {
 366:   AU.addRequired<LoopInfoWrapperPass>();
 367: }
 368: 
 369: static void unrollGEPLoad(CallInst *Call) {
 370:   auto [GEP, Load] = BPFPreserveStaticOffsetPass::reconstructLoad(Call);
 371:   GEP->insertBefore(Call->getIterator());
 372:   Load->insertBefore(Call->getIterator());
 373:   Call->replaceAllUsesWith(Load);
 374:   Call->eraseFromParent();
 375: }
 376: 
 377: static void unrollGEPStore(CallInst *Call) {
 378:   auto [GEP, Store] = BPFPreserveStaticOffsetPass::reconstructStore(Call);
 379:   GEP->insertBefore(Call->getIterator());
 380:   Store->insertBefore(Call->getIterator());
```

- EN: Function bodies or method definitions such as getAnalysisUsage, unrollGEPLoad, unrollGEPStore contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: getAnalysisUsage, unrollGEPLoad, unrollGEPStore 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 381-400

```cpp
 381:   Call->eraseFromParent();
 382: }
 383: 
 384: static bool removeGEPBuiltinsInFunc(Function &F) {
 385:   SmallVector<CallInst *> GEPLoads;
 386:   SmallVector<CallInst *> GEPStores;
 387:   for (auto &BB : F)
 388:     for (auto &Insn : BB)
 389:       if (auto *Call = dyn_cast<CallInst>(&Insn))
 390:         if (auto *Called = Call->getCalledFunction())
 391:           switch (Called->getIntrinsicID()) {
 392:           case Intrinsic::bpf_getelementptr_and_load:
 393:             GEPLoads.push_back(Call);
 394:             break;
 395:           case Intrinsic::bpf_getelementptr_and_store:
 396:             GEPStores.push_back(Call);
 397:             break;
 398:           }
 399: 
 400:   if (GEPLoads.empty() && GEPStores.empty())
```

- EN: Function bodies or method definitions such as removeGEPBuiltinsInFunc contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: removeGEPBuiltinsInFunc 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 401-420

```cpp
 401:     return false;
 402: 
 403:   for_each(GEPLoads, unrollGEPLoad);
 404:   for_each(GEPStores, unrollGEPStore);
 405: 
 406:   return true;
 407: }
 408: 
 409: // Rewrites the following builtins:
 410: // - llvm.bpf.getelementptr.and.load
 411: // - llvm.bpf.getelementptr.and.store
 412: // As (load (getelementptr ...)) or (store (getelementptr ...)).
 413: bool BPFCheckAndAdjustIR::removeGEPBuiltins(Module &M) {
 414:   bool Changed = false;
 415:   for (auto &F : M)
 416:     Changed = removeGEPBuiltinsInFunc(F) || Changed;
 417:   return Changed;
 418: }
 419: 
 420: // Wrap ToWrap with cast to address space zero:
```

- EN: Function bodies or method definitions such as removeGEPBuiltins contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: removeGEPBuiltins 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 421-440

```cpp
 421: // - if ToWrap is a getelementptr,
 422: //   wrap it's base pointer instead and return a copy;
 423: // - if ToWrap is Instruction, insert address space cast
 424: //   immediately after ToWrap;
 425: // - if ToWrap is not an Instruction (function parameter
 426: //   or a global value), insert address space cast at the
 427: //   beginning of the Function F;
 428: // - use Cache to avoid inserting too many casts;
 429: static Value *aspaceWrapValue(DenseMap<Value *, Value *> &Cache, Function *F,
 430:                               Value *ToWrap) {
 431:   auto It = Cache.find(ToWrap);
 432:   if (It != Cache.end())
 433:     return It->getSecond();
 434: 
 435:   if (auto *GEP = dyn_cast<GetElementPtrInst>(ToWrap)) {
 436:     Value *Ptr = GEP->getPointerOperand();
 437:     Value *WrappedPtr = aspaceWrapValue(Cache, F, Ptr);
 438:     auto *GEPTy = cast<PointerType>(GEP->getType());
 439:     auto *NewGEP = GEP->clone();
 440:     NewGEP->insertAfter(GEP->getIterator());
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 441-460

```cpp
 441:     NewGEP->mutateType(PointerType::getUnqual(GEPTy->getContext()));
 442:     NewGEP->setOperand(GEP->getPointerOperandIndex(), WrappedPtr);
 443:     NewGEP->setName(GEP->getName());
 444:     Cache[ToWrap] = NewGEP;
 445:     return NewGEP;
 446:   }
 447: 
 448:   IRBuilder IB(F->getContext());
 449:   if (Instruction *InsnPtr = dyn_cast<Instruction>(ToWrap))
 450:     IB.SetInsertPoint(*InsnPtr->getInsertionPointAfterDef());
 451:   else
 452:     IB.SetInsertPoint(F->getEntryBlock().getFirstInsertionPt());
 453:   auto *ASZeroPtrTy = IB.getPtrTy(0);
 454:   auto *ACast = IB.CreateAddrSpaceCast(ToWrap, ASZeroPtrTy, ToWrap->getName());
 455:   Cache[ToWrap] = ACast;
 456:   return ACast;
 457: }
 458: 
 459: // Wrap a pointer operand OpNum of instruction I
 460: // with cast to address space zero
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 461-480

```cpp
 461: static void aspaceWrapOperand(DenseMap<Value *, Value *> &Cache, Instruction *I,
 462:                               unsigned OpNum) {
 463:   Value *OldOp = I->getOperand(OpNum);
 464:   if (OldOp->getType()->getPointerAddressSpace() == 0)
 465:     return;
 466: 
 467:   Value *NewOp = aspaceWrapValue(Cache, I->getFunction(), OldOp);
 468:   I->setOperand(OpNum, NewOp);
 469:   // Check if there are any remaining users of old GEP,
 470:   // delete those w/o users
 471:   for (;;) {
 472:     auto *OldGEP = dyn_cast<GetElementPtrInst>(OldOp);
 473:     if (!OldGEP)
 474:       break;
 475:     if (!OldGEP->use_empty())
 476:       break;
 477:     OldOp = OldGEP->getPointerOperand();
 478:     OldGEP->eraseFromParent();
 479:   }
 480: }
```

- EN: Function bodies or method definitions such as aspaceWrapOperand contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: aspaceWrapOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 481-500

```cpp
 481: 
 482: static Value *wrapPtrIfASNotZero(DenseMap<Value *, Value *> &Cache,
 483:                                  CallInst *CI, Value *P) {
 484:   if (auto *PTy = dyn_cast<PointerType>(P->getType())) {
 485:     if (PTy->getAddressSpace() == 0)
 486:       return P;
 487:   }
 488:   return aspaceWrapValue(Cache, CI->getFunction(), P);
 489: }
 490: 
 491: static Instruction *aspaceMemSet(Intrinsic::ID ID,
 492:                                  DenseMap<Value *, Value *> &Cache,
 493:                                  CallInst *CI) {
 494:   auto *MI = cast<MemIntrinsic>(CI);
 495:   IRBuilder<> B(CI);
 496: 
 497:   Value *OldDst = CI->getArgOperand(0);
 498:   Value *NewDst = wrapPtrIfASNotZero(Cache, CI, OldDst);
 499:   if (OldDst == NewDst)
 500:     return nullptr;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 501-520

```cpp
 501: 
 502:   // memset(new_dst, val, len, align, isvolatile, md)
 503:   Value *Val = CI->getArgOperand(1);
 504:   Value *Len = CI->getArgOperand(2);
 505: 
 506:   auto *MS = cast<MemSetInst>(CI);
 507:   MaybeAlign Align = MS->getDestAlign();
 508:   bool IsVolatile = MS->isVolatile();
 509: 
 510:   if (ID == Intrinsic::memset)
 511:     return B.CreateMemSet(NewDst, Val, Len, Align, IsVolatile,
 512:                           MI->getAAMetadata());
 513:   else
 514:     return B.CreateMemSetInline(NewDst, Align, Val, Len, IsVolatile,
 515:                                 MI->getAAMetadata());
 516: }
 517: 
 518: static Instruction *aspaceMemCpy(Intrinsic::ID ID,
 519:                                  DenseMap<Value *, Value *> &Cache,
 520:                                  CallInst *CI) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 521-540

```cpp
 521:   auto *MI = cast<MemIntrinsic>(CI);
 522:   IRBuilder<> B(CI);
 523: 
 524:   Value *OldDst = CI->getArgOperand(0);
 525:   Value *OldSrc = CI->getArgOperand(1);
 526:   Value *NewDst = wrapPtrIfASNotZero(Cache, CI, OldDst);
 527:   Value *NewSrc = wrapPtrIfASNotZero(Cache, CI, OldSrc);
 528:   if (OldDst == NewDst && OldSrc == NewSrc)
 529:     return nullptr;
 530: 
 531:   // memcpy(new_dst, dst_align, new_src, src_align, len, isvolatile, md)
 532:   Value *Len = CI->getArgOperand(2);
 533: 
 534:   auto *MT = cast<MemTransferInst>(CI);
 535:   MaybeAlign DstAlign = MT->getDestAlign();
 536:   MaybeAlign SrcAlign = MT->getSourceAlign();
 537:   bool IsVolatile = MT->isVolatile();
 538: 
 539:   return B.CreateMemTransferInst(ID, NewDst, DstAlign, NewSrc, SrcAlign, Len,
 540:                                  IsVolatile, MI->getAAMetadata());
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 541-560

```cpp
 541: }
 542: 
 543: static Instruction *aspaceMemMove(DenseMap<Value *, Value *> &Cache,
 544:                                   CallInst *CI) {
 545:   auto *MI = cast<MemIntrinsic>(CI);
 546:   IRBuilder<> B(CI);
 547: 
 548:   Value *OldDst = CI->getArgOperand(0);
 549:   Value *OldSrc = CI->getArgOperand(1);
 550:   Value *NewDst = wrapPtrIfASNotZero(Cache, CI, OldDst);
 551:   Value *NewSrc = wrapPtrIfASNotZero(Cache, CI, OldSrc);
 552:   if (OldDst == NewDst && OldSrc == NewSrc)
 553:     return nullptr;
 554: 
 555:   // memmove(new_dst, dst_align, new_src, src_align, len, isvolatile, md)
 556:   Value *Len = CI->getArgOperand(2);
 557: 
 558:   auto *MT = cast<MemTransferInst>(CI);
 559:   MaybeAlign DstAlign = MT->getDestAlign();
 560:   MaybeAlign SrcAlign = MT->getSourceAlign();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 561-580

```cpp
 561:   bool IsVolatile = MT->isVolatile();
 562: 
 563:   return B.CreateMemMove(NewDst, DstAlign, NewSrc, SrcAlign, Len, IsVolatile,
 564:                          MI->getAAMetadata());
 565: }
 566: 
 567: // Support for BPF address spaces:
 568: // - for each function in the module M, update pointer operand of
 569: //   each memory access instruction (load/store/cmpxchg/atomicrmw)
 570: //   or intrinsic call insns (memset/memcpy/memmove)
 571: //   by casting it from non-zero address space to zero address space, e.g:
 572: //
 573: //   (load (ptr addrspace (N) %p) ...)
 574: //     -> (load (addrspacecast ptr addrspace (N) %p to ptr))
 575: //
 576: // - assign section with name .addr_space.N for globals defined in
 577: //   non-zero address space N
 578: bool BPFCheckAndAdjustIR::insertASpaceCasts(Module &M) {
 579:   bool Changed = false;
 580:   for (Function &F : M) {
```

- EN: Function bodies or method definitions such as insertASpaceCasts contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: insertASpaceCasts 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 581-600

```cpp
 581:     DenseMap<Value *, Value *> CastsCache;
 582:     for (BasicBlock &BB : F) {
 583:       for (Instruction &I : llvm::make_early_inc_range(BB)) {
 584:         unsigned PtrOpNum;
 585: 
 586:         if (auto *LD = dyn_cast<LoadInst>(&I)) {
 587:           PtrOpNum = LD->getPointerOperandIndex();
 588:           aspaceWrapOperand(CastsCache, &I, PtrOpNum);
 589:           continue;
 590:         }
 591:         if (auto *ST = dyn_cast<StoreInst>(&I)) {
 592:           PtrOpNum = ST->getPointerOperandIndex();
 593:           aspaceWrapOperand(CastsCache, &I, PtrOpNum);
 594:           continue;
 595:         }
 596:         if (auto *CmpXchg = dyn_cast<AtomicCmpXchgInst>(&I)) {
 597:           PtrOpNum = CmpXchg->getPointerOperandIndex();
 598:           aspaceWrapOperand(CastsCache, &I, PtrOpNum);
 599:           continue;
 600:         }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 601-620

```cpp
 601:         if (auto *RMW = dyn_cast<AtomicRMWInst>(&I)) {
 602:           PtrOpNum = RMW->getPointerOperandIndex();
 603:           aspaceWrapOperand(CastsCache, &I, PtrOpNum);
 604:           continue;
 605:         }
 606: 
 607:         auto *CI = dyn_cast<CallInst>(&I);
 608:         if (!CI)
 609:           continue;
 610: 
 611:         Function *Callee = CI->getCalledFunction();
 612:         if (!Callee || !Callee->isIntrinsic())
 613:           continue;
 614: 
 615:         // Check memset/memcpy/memmove
 616:         Intrinsic::ID ID = Callee->getIntrinsicID();
 617:         bool IsSet = ID == Intrinsic::memset || ID == Intrinsic::memset_inline;
 618:         bool IsCpy = ID == Intrinsic::memcpy || ID == Intrinsic::memcpy_inline;
 619:         bool IsMove = ID == Intrinsic::memmove;
 620:         if (!IsSet && !IsCpy && !IsMove)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 621-640

```cpp
 621:           continue;
 622: 
 623:         Instruction *New;
 624:         if (IsSet)
 625:           New = aspaceMemSet(ID, CastsCache, CI);
 626:         else if (IsCpy)
 627:           New = aspaceMemCpy(ID, CastsCache, CI);
 628:         else
 629:           New = aspaceMemMove(CastsCache, CI);
 630: 
 631:         if (!New)
 632:           continue;
 633: 
 634:         I.replaceAllUsesWith(New);
 635:         New->takeName(&I);
 636:         I.eraseFromParent();
 637:       }
 638:     }
 639:     Changed |= !CastsCache.empty();
 640:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 641-660

```cpp
 641:   // Merge all globals within same address space into single
 642:   // .addr_space.<addr space no> section
 643:   for (GlobalVariable &G : M.globals()) {
 644:     if (G.getAddressSpace() == 0 || G.hasSection())
 645:       continue;
 646:     SmallString<16> SecName;
 647:     raw_svector_ostream OS(SecName);
 648:     OS << ".addr_space." << G.getAddressSpace();
 649:     G.setSection(SecName);
 650:     // Prevent having separate section for constants
 651:     G.setConstant(false);
 652:   }
 653:   return Changed;
 654: }
 655: 
 656: bool BPFCheckAndAdjustIR::adjustIR(Module &M) {
 657:   bool Changed = removePassThroughBuiltin(M);
 658:   Changed = removeCompareBuiltin(M) || Changed;
 659:   Changed = sinkMinMax(M) || Changed;
 660:   Changed = removeGEPBuiltins(M) || Changed;
```

- EN: Function bodies or method definitions such as adjustIR contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: adjustIR 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 661-668

```cpp
 661:   Changed = insertASpaceCasts(M) || Changed;
 662:   return Changed;
 663: }
 664: 
 665: bool BPFCheckAndAdjustIR::runOnModule(Module &M) {
 666:   checkIR(M);
 667:   return adjustIR(M);
 668: }
```

- EN: Function bodies or method definitions such as runOnModule contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: runOnModule 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

## Key Concepts / 关键概念

- Target backend support code / 目标后端支持代码
- LLVM CodeGen integration / LLVM CodeGen 集成

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPF.h`, `BPFCORE.h`, `llvm/Analysis/LoopInfo.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`
